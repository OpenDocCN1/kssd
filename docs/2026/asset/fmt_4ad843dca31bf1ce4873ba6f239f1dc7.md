
# [原创]魔改CobaltStrike：beacon浅析(下)

## 一、概述

前文再续书接上回，之前写了1-50号功能分析，这次来看看50-100功能号，若有错还请大伙指出，谢谢大伙。

## 二、50-100功能号分析

### `Rportfwd`

任务号50，端口转发数据

先获取到监听的端口号：

![](upload/attach/202107/718877\_BHQ65SD3QGR2AJB.png)
![](upload/attach/202107/718877\_VT4QEH5QNM6VH8G.png)

然后`bind sockattr 0.0.0.0:8888`和开始`listen`：

![](upload/attach/202107/718877\_W9AAWCMZPCVUEGD.png)
![](upload/attach/202107/718877\_QXJGFVFFGAST2KB.png)

任务号51，`rportfwd stop port`停止指定端口转发

有关于`rportfwd`所涉及到的任务号有15、16、50、51，cs和msf应该是根据portfwd开源项目<mark class="encrypted">075K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5M7%4y4F1M7$3A6Q4x3V1k6H3L8%4u0@1k6Y4N6V1i4K6u0r3i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1$3i4K6V1#2i4@1t1@1i4@1f1#2i4K6V1H3i4K6R3^5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1K6i4K6R3H3i4K6R3J5</mark>

### `Ls`

任务号53，调用`FindFirstFileA()`、`FindNextFileA()`相关Api遍历当前文件夹，调用`FileTimeToSystemTime()`获取文件的文件修改时间大小等信息并通过`SystemTimeToTzSpecificLocalTime()`转换，最后进行相应的拼接返回给teamserver端：

![](upload/attach/202107/718877\_ZD9D74A6JVAWH8W.png)
![](upload/attach/202107/718877\_CVW9KHQA9FMYMAD.png)

把获取到的相关文件信息返回给teamserver：

![](upload/attach/202107/718877\_7MXH8CB8UGX5TH3.png)

### `Mkdir`

任务号54，`createDirectoryA()`创建目录

![](upload/attach/202107/718877\_MJ3SM94NNCSC8DS.png)

### `drives`

任务号55，`GetLogicalDrivers()`列出目标机上所有的磁盘盘符

![](upload/attach/202107/718877\_ZUTBA9UDMVAA7XK.png)
![](upload/attach/202107/718877\_38X4WKK8MRXY64U.png)

### `Rm`

任务号56，删除文件

先用`GetFileAttributesA()`获取到文件属性，进行判断是文件还是文件夹：

![](upload/attach/202107/718877\_2U2JSYQ68Z4XCV4.png)

如果是文件夹就遍历删除里面文件再删文件夹：

![](upload/attach/202107/718877\_NX6WNMERUZWZ7J2.png)
![](upload/attach/202107/718877\_8XPVW54K8ZB5E6H.png)

如果是文件则直接删：

![](upload/attach/202107/718877\_N8746VX4S8KF5BU.png)

### `One-liner`

任务号59，简单来说就是开启服务器，让执行相应的ps命令，返回一个会话。

绑定端口，新建线程进行监听：

![](upload/attach/202107/718877\_WA4PVMA2JKVQM25.png)
![](upload/attach/202107/718877\_C758APWQNHNGTGF.png)
![](upload/attach/202107/718877\_G6VU7NH972JEKZU.png)
![](upload/attach/202107/718877\_8BGJQQ7JVUZRCEC.png)

执行ps脚本返回会话：

![](upload/attach/202107/718877\_6EYBZC2ADAR9GQC.png)

用管道执行`%COMSPEC% /c echo`配合`pth hash`发送给对应dll

首先调用`CreateaNamePipeA()`创建管道：

![](upload/attach/202107/718877\_TBE666HA86A2SD6.png)
![](upload/attach/202107/718877\_JCFFQNZFMQENDXM.png)

创建新线程，等待另一端连接该管道：

![](upload/attach/202107/718877\_UW6K22XDCMQKJJA.png)

调用`connectNamePipe()`等待另一端来连接该管道，同时线程阻塞：

![](upload/attach/202107/718877\_87UARA5KFCSXXYQ.png)

接着aggressor端会发送`mimikatz.dll`进行挂起注入并调用执行，过程与`spawn`一致，下面是`mimikatz.dll`内容：

![](upload/attach/202107/718877\_PUG6NXM3FXSHKDS.png)

`mimikatz.dll`进行挂起注入并调用执行：

![](upload/attach/202107/718877\_M267FF8HRKJHJ4T.png)

恢复线程并执行：

![](upload/attach/202107/718877\_JHHUZWX97JVZCRS.png)
![](upload/attach/202107/718877\_AGG6YJVTEV3R84J.png)

在内存中Dump下来`mimikatz.dll`，里面保存着两条管道名字，负责mimikatz与beacon通信用的：

`\pipe\8f5879`是beacon生成的，mimikatz调用`peekNamedPipe`去连接：

![](upload/attach/202107/718877\_WJUZMDAC6487WHV.png)

`\pipe\2c67bfba`是mimikatz生成的，beacon调用`peekNamedPipe`去连接：

![](upload/attach/202107/718877\_5RRZEEH62DD3S3B.png)
![](upload/attach/202107/718877\_TBBJM2QJBUUVSKG.png)

aggressor会发送任务号61来建立进行与`mimikatz.dll`的通信，并进行后续的模拟高权限token。

mimikatz执行后aggressor回传通信管道信息，任务号61：

![](upload/attach/202107/718877\_H3KC8YC4NZ93KGR.png)

调用`peekNamedPipe`链接mimikatz生成的管道`\pipe\2c67bfba`：

![](upload/attach/202107/718877\_YYM6P259HCWRKXA.png)

一旦停止阻塞，就会调用`ImpersonateNamedPipeClient`来模拟高权限客户端的token，并调用`openTreadToken()`获得当前线程token：

![](upload/attach/202107/718877\_C3SUN2NFBTQKD3J.png)

`ImpersonateLoggedOnUser()`让当前线程模拟登陆用户进行操作，并保存`token`值为全局`tokenHandle`：

![](upload/attach/202107/718877\_EGDHSN948P78YSZ.png)
![](upload/attach/202107/718877\_VEJS7RWJ6CDG5UN.png)

### Mimikatz Job执行后数据的回传

任务号62，流程与任务号40一样。

### `link` 连接SMB Beacon

任务号68，`link`通过管道连接SMB形式的Beacon

Aggress端在生成`link`方式的stagless 时，会加载`windows/beacon_bind_pipe`中`pviot.dll`的数据：

![](upload/attach/202107/718877\_TY69NVQP73K2GCF.png)
![](upload/attach/202107/718877\_6ACH3FSSS22YABV.png)

在`link`之前，需要建立连接：

```
shell net use \\10.10.10.165\c$ /user:"administrator" "!@#Q123"
```

![](upload/attach/202107/718877\_UNQ3B7RS2SBR9EN.png)

再执行`jump psexec64 10.10.10.165 Test_SMB`：

![](upload/attach/202107/718877\_RRY8AWPSQE5XARC.png)

在执行`jump`命令过程中会调用到`link`，下面看`jump`指令的执行过程分析：

先调用任务号9 `upload`上传文件：

![](upload/attach/202107/718877\_34RF2PB7NU98JPF.png)

将文件保存在目标ip的`ADMIN$`下，名字问`f9febc5.exe`

![](upload/attach/202107/718877\_9YJZQ46VAMQUMYW.png)

然后通过任务号100 `inline-execute`调用该文件。

再调用55任务号`rm`去删除上传的文件：

![](upload/attach/202107/718877\_9VW6NCDTUXK869E.png)

接着再调用47任务号`pause`暂停1秒：

![](upload/attach/202107/718877\_RH7CJY67CAT34GN.png)

最后调用任务号68 `link` Beacon：

![](upload/attach/202107/718877\_DKHHN64X2AZJR7K.png)

调用`createFIle()`打开命名管道：

![](upload/attach/202107/718877\_79BWCYNKP2AHWAZ.png)

调用`SetNamedPipeHandleState()`切换管道为读模式：

![](upload/attach/202107/718877\_VP4TNX4EQ8P7Y65.png)

循环调用`ReadFile()`读服务端返回的数据：

![](upload/attach/202107/718877\_C6MS9HYF6ATHM6U.png)

将数据返回给teamserver端上线：

![](upload/attach/202107/718877\_ZQJVQS2W2S4QZNQ.png)

### `Spawnto (x64)`

任务号69，`spawnto x64`，设置Beacon派生会话时使用的程序，执行流程与任务号13 `spawnto x86`一样。

### `execute-assembly (x86)`

任务号70，内存执行C#的可执行文件，

首先传输`invokeassembly.dll`并注入到`rundll32.exe`，实现了在其内存中创建CLR环境，然后通过管道再将C#可执行文件读取到内存中,最后执行。

![](upload/attach/202107/718877\_S3Y55AFK8PAXD4X.png)

Msf、cs的`execute-assembly`与开源的<mark class="encrypted">ffbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1y4s2u0@1K9h3E0Q4x3V1k6E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0V1k6i4S2W2j5%4g2@1k6g2)9J5k6r3q4K6M7$3g2E0j5X3I4&6i4K6u0r3</mark> 大同小异，我们大概了解下其执行流程：

当`execute-assembly`的dll注入到相应进程后

先后调用`ICLRMetaHost::EnumerateLoadedRuntimes`、`ICLRMetaHost::GetRuntime`方法以获取有效的`ICLRRuntimeInfo`指针：

![](upload/attach/202107/718877\_FBFTMEWWV4J3SAH.png)
![](upload/attach/202107/718877\_MFF8M53XC6UJW2K.png)

调用`ICLRRuntimeInfo::GetInterface`方法获取接口并使用：

这里使用的是`ICorRuntimeHost`：

- 需指定`CLSID_CorRuntimeHost`为`rclsid`参数
- 需指定`IID_ICorRuntimeHost`为`RIID`参数

![](upload/attach/202107/718877\_RGTSF465K79BBM6.png)

当获取到`_AppDomainPtr`后，使用其`Load_3(...)`从内存中读取并加载.NET程序集

![](upload/attach/202107/718877\_NJWMBJBCK2EUKV9.png)

获取参数后调用静态方法。

![](upload/attach/202107/718877\_9TUMK25SRGYWPPN.png)

### `execute-assembly (x64)`

任务号71，流程与任务号70一样，区别只是传输dll的版本是x64的。

### `Setenv`

任务号72，`putenv()`设置环境变量

![](upload/attach/202107/718877\_WP24BV47EYGPQ25.png)

### `Cp`

任务号73，调用`CopyFileA()`复制文件

![](upload/attach/202107/718877\_94UBDC8MX5QJZ9U.png)

### `Mv`

任务号74，调用`MoveFileA()`移动文件:

![](upload/attach/202107/718877\_SFEY6K8UJFCHNGZ.png)

### `ppid`

任务号75，伪造子进程（jobs）的父进程为指定进程，jobs是指`portscan`等操作，不是派生会话操作。

![](upload/attach/202107/718877\_W23XHP5JWT4PWTP.png)

首先我们设置指定jobs的父进程为18360“

![](upload/attach/202107/718877\_E48AZU3GZDEWGDA.png)
![](upload/attach/202107/718877\_SKGTN9UWB948V8K.png)

我们设置了注入`calc.exe`，该进程pid为35752：

![](upload/attach/202107/718877\_5CKC65CEC9YZTGP.png)

查询`calc.exe`的父进程pid为我们设置好的父进程：

![](upload/attach/202107/718877\_WWNNS2V4B6ZPRF2.png)

### `runu`

任务号76，父进程欺骗，即指定所创建程序的父进程pid值，并执行该进程：

我们设置伪装的父进程pid18360，先用`openprocess()`以`PROCESS_ALL_ACCESS`全部权限根据pid打开要伪装的父进程，获取其进程句柄：

![](upload/attach/202107/718877\_M9UWSGWDFZ8B7TA.png)
![](upload/attach/202107/718877\_TJ56NYTGNHB4G94.png)

调用`UpdateProcThreadAttribute()`，传入父进程的句柄，指定可继承的句柄：

![](upload/attach/202107/718877\_Y44AK5SQDEEM4XM.png)

接着用已构造的属性结构体更新属性表：

![](upload/attach/202107/718877\_EEFUD23ZMT69HAS.png)

最后使用`createprocess()`传入已跟换父进程属性表`STARTUPINFOEX`来创建新程序：

![](upload/attach/202107/718877\_39Y2BM2C7964FH6.png)
![](upload/attach/202107/718877\_G36XXKSUJ9PPM2V.png)

### `getprivs`

任务号77，`getprivs`，启用当前访问令牌所拥有的特权

命令窗口输入`getprivs`后，受控端会受到一系列的特权信息：

![](upload/attach/202107/718877\_9E3RT5VHR345QV5.png)

当使用hash传递后，`tokenhandle`会有相应的令牌信息，否则走`else`流程，流程中先使用`GetCurrentProcess()`获取当前进程的句柄，再使用`OpenProcessToken()`打开与进程相关的令牌：

![](upload/attach/202107/718877\_CH8XMFZMEAJKZGE.png)

传入指定特权的名称调用`LookupPrivilegeValue()`函数查看系统权限的特权值，函数调用成功后，信息存入第三个类型为`LUID`的结构体中，并且函数返回非0。接着调用`AdjustTokenPrivileges()`并传入新特权信息的指针`PTOKEN_PRIVILEGES`启用当前访问令牌所拥有的特权。

![](upload/attach/202107/718877\_GVH7X8MZUDRM8KH.png)
![](upload/attach/202107/718877\_4X3E2GAHMD3SJME.png)
