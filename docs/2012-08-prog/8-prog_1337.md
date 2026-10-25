
# Ring3 下另一种结束卡巴斯基进程的新技术

> 原文：[`www.pediy.com/kssd/pediy10/66063.html`](https://www.pediy.com/kssd/pediy10/66063.html)

原创，转载请保留全部信息

卡巴斯基<1>启动的时候，两个进程内部都会创建一个事件用于同步退出，当显示关闭这个事件时，`AVP` 进程会退出。这个事件内核对象名字是一个 `GUID`，一开始我以为是动态生成的，但据我逆向及在其他机器观察，这个事件是硬编码，`6953EA60-8D5F-4529-8710-42F8ED3E8CDA`(为了方便，在本文，我把这个事件写成 `A` 事件，我猜想卡巴斯基开发人员肯定读过 `windows` 核心编程，`Jeffrey Richter` 建议用 `GUID` 来命名事件^\_^)。打开 `procexp.exe` 查找两个 `avp.exe`<2>进程内核对象，关闭 `A` 事件，这时 `AVP` 进程就会退出，各位看官可以在自己机器上测试^\_^，`AVP` 进程守护功能<3>在此不起作用。`Procexp.exe` 是如何做到关闭其他进程内核对象的呢？逆向之后发现 `procexp.exe` 是在 `ring0` 下完成的，偶不用 `ring0`，直接 `ring3`。

卡巴斯基有 `Hook NtOpenProcess`，不过当我们以 `PROCESS_QUERY_INFORMATION` 打开 `AVP` 时，仍然可以成功<4>，同样卡巴斯基也有 `Hook NtDuplicateObject`，一开始我以为如果调用 `DuplicateHandle` 来改变 `desire access` 会失效，据牛人张翼的话：很多 `HIPS` 挂了 `NtDuplicateObject` 也是白挂。于是我尝试用 `Duplicate(… PROCESS_DUP_HANDLE….)`改变进程的 `desire access`，卡巴斯基有两个进程，其中一个以 `SYSTEM` 账号运行的 `AVP` 进程不能打开，只能打开另一个以当前用户运行的进程，足够了。当我们终止以当前用户运行的 `AVP` 进程时，卡巴就失效了。

成功打开了句柄后，采用 `NtQuerySystemInformation SystemHandleInformation` 功能号枚举句柄，当枚举到的句柄是 `A` 事件时，我们用刚才打开的进程句柄，用 `DuplicateHandle(..DUPLICATE_CLOSE_SOURCE)`, `DUPLICATE_CLOSE_SOURCE` 标志位关闭 source 进程内的句柄，`AVP` 进程感应到退出事件，退出。

代码详见附件，如果发现有 `BUGS`，请 mail me : `wangyongxina@gmail.com`

感谢猪头三的技术支持~

注<1>：我开发用的版本如下：
`kis8.0.0.402en.exe`，`kav8.0.0.402en.exe`，`kis7_0_1_325sch.exe`。其他版本可能会运行失败。早期的 `KIS7.0.2`(我忘记版本号了)，这种方法无效。

注<2>：卡巴斯基在启动的时候，可能会启动三个进程，其中有一个用于升级，本文不考虑这种情况。

注<3>：卡巴斯基有进程守护功能，当结束其中的一个时，如果是以当前用户运行的 `AVP` 进程，会被另一个 `AVP` 重启；如果是以 `SYSTEM` 账号运行的进程，会被 `services.exe` 启动

注<4>:我在测试 `KIS7.0` 版本时，发现以 `PROCESS_DUP_HANDLE | PROCESS_QUERY_INFORMATION | PROCESS_VM_READ` 可以打开句柄，而 `KIS8.0` 进程只能以 `PROCESS_QUERY_INFORMATION` 打开。

测试时，请注意卡巴版本，不是所有版本都存在这样方式来终止

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Kill_KIS8.rar` \[解压密码：pediy\]   
|
