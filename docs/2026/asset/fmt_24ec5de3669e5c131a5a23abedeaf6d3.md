
使用`FindResourceW`、`SizeofResource`、`LoadResource`和`LockResource`等API实现资源的释放。

程序会在同级目录下释放后续功能测试所需的DLL。

### 注入技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_CYZ6HTKMEKBUNQJ.png)

#### 全局钩子注入

使用Windows提供的`SetWindowsHookEx`将DLL注入到系统。测试DLL为`MyHookDll.dll`。可通过资源释放技术释放所需dll。也可自行编译。HOOK的消息是`WH_GETMESSAGE`。dll中只写了一个象征性的`MessageBox`。即拖拽dll点击开始注入后，当系统中有任意进程接收到消息会弹出一个`MessageBox`。如果电脑中启动的应用较多可能导致蓝屏卡死等情况。

#### 远程线程注入

1. 拖拽`TestDll.dll`(使用资源释放技术释放)或者自行编译
2. 输入需要注入的进程ID
3. 点击开始注入后会将测试DLL注入到目标进程 测试DLL只有一个`MessageBox`
4. 如果注入的进程权限较高或者检测较严格(如`QQ.exe`) 则可能失败

**实现原理**

使用`ZwCreateThreadEx`函数创建远程线程 实现DLL注入。之所以没有`CreateRemoteThread`是因为`CreateRemoteThread`函数内部是被`CreateRemoteThread`调用的。相对来说这个函数的权限会更高。

#### APC注入

1. 拖拽`TestDll.dll`(使用资源释放技术释放)或自行编译
2. 输入需要注入的进程名
3. 点击开始注入后会将测试DLL注入到目标进程 测试DLL只有一个`MessageBox`
4. 如果注入的进程权限较高或者检测较严格(如`QQ.exe`) 则可能失败

**实现原理**

通过`QueueUserAPC`把一个函数添加到指定线程的`APC`队列中

### 启动技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_NYE46ZQTZRY37HG.png)

#### 三种方式创建进程

1. 任意拖拽一个exe到窗口，点击按钮后会使用三种不同的方式创建进程
2. 分别使用`CreateProcess`、`ShellExecute`和`WinExec`三种方式创建进程

#### 内存加载运行dll

1. 拖拽`TestDll.dll`(使用资源释放技术释放)
2. 点击按钮后程序会将`TestDll`模拟PE加载器加载到本程序内存后直接运行
3. 如果将DLL隐藏到资源可以不将DLL释放到本地直接运行
4. 病毒WannaCry和BadRabbit都用了这种方式来运行自己的主逻辑程序

### 自启动技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_ESZS2MHYTR6JUUN.png)

#### 注册表

1. 拖拽任意exe到窗口
2. 点击按钮后会将程序的完整路径写到注册表启动项
3. 目标程序随系统启动而启动
4. 可自行查看提示窗口的注册表路径进行查看或删除
5. 原理：通过`RegOpenKeyEx`和`RegSetValueEx`将程序写入到`\HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

#### 快速启动目录

1. 拖拽任意exe到窗口
2. 点击按钮后会将程序的复制到系统的快速启动目录
3. 目标程序随用户登录而自行启动
4. 可自行查看提示窗口的快速启动目录路径进行查看或删除
5. 实现原理：使用`SHGetSpecialFolderPath`函数获取Windows系统中快速启动目录的路径 然后，使用`CopyFile`函数，将想要自启动的程序复制到快速启动目录下

#### 计划任务

1. 此功能需使用管理员运行
2. 拖拽任意exe到窗口
3. 点击按钮后会将目标程序写入到系统的计划任务
4. 程序随用户登录而启动
5. 可自行在计划任务中删除添加的事件
6. 实现原理：调用COM组件将程序添加到计划任务实现开机自启动

#### 系统服务

1. 此功能需使用管理员运行
2. 拖拽`ServiceExeTest.exe`(服务应用程序 需自行编译)到窗口
3. 点击按钮后会将`ServiceExeTest.exe`注册为系统服务
4. `ServiceExeTest.exe`将会在后台运行
5. `ServiceExeTest.exe`不含任何代码 只是一个空的服务应用程序
6. 可自行使用任务管理器删除
7. 实现原理：使用`CreateService`将编写的服务应用程序注册为系统服务实现开机自启动

### 提权技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_VNHWX87GYR7CVEA.png)

#### 提升为Debug权限

1. 此功能需使用管理员运行
2. 点击按钮后程序将当前进程提升为为Debug权限

#### BypassUAC

1. 首先将用户账户控制设置为默认
2. 点击按钮后会绕过UAC的弹窗提升为管理员权限
3. 不弹窗开启一个管理员权限的命令行
4. 实现原理：使用COM组件提权

#### 查看当前进程权限

1. 点击按钮后将在List控件上显示当前程序的所有权限
2. 可使用此功能查看提升为Debug权限是否生效

### 隐藏技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_X3NAXD6VCZXCAH2.png)

#### 进程伪装

1. 点击按钮后会将本进程伪装为`Explorer.exe`
2. 本程序为32位 在32为系统下此功能才有效
3. 实现原理：使用`NtQueryInformationProcess`获取进程环境块。修改指定进程环境块的进程路径以及命令行信息，从而达到进程伪装的效果。

#### 傀儡进程

1. 拖拽任意一个exe到窗口
2. 点击按钮后会将一段MessageBox的ShellCode注入到目标exe
3. 实现原理：修改某一进程的内存数据，向内存数据写入Shellcode代码，并修改该进程的执行流程，使其执行Shellcode代码。这样，进程还是原来的进程，但执行的操作却替换了。

#### DLL劫持

1. 此功能没有写在界面上 需要自行手动操作
2. 我这里劫持的是`FileCleaner2.0.exe`的`oledlg.dll`
3. 找到`DllHijackTest.dll`(使用资源释放技术释放)或自行编译
4. 将DLL重命名为`oledlg.dll`
5. 找到`C:\Windows\SysWOW64`目录下的`oledlg.dll`将其重命名为`Old\_oledlg`(如果无法重命名请自行百度解决方案)
6. 将`oledlg.dll`放到`SysWOW64`目录下
7. 运行`FileCleaner2.0.exe`
8. 劫持的DLL在`DllMian`中写了一个`MessageBox`函数 表示劫持成功
9. 实现原理：在PE加载器加载DLL时 使用自制的DLL替换掉原DLL ，在自制的DLL中转发原DLL的导出函数并添加代码

### 压缩技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_6KGXWBV53A6Z37Y.png)

#### 数据解压缩

1. 在文本框中输入数据 点击按钮，即可对输入的数据压缩和解压缩
2. 实现原理：通过Windows提供的`RtlDecompressBuffer`和`RtlGetCompressionWorkSpaceSize`实现对数据的解压缩

#### 文件解压缩

1. 拖拽文件到窗口，点击按钮 即可对文件进行压缩和解压缩
2. 使用`ZLIB`压缩库实现对文件的解压缩

### 加密技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_84NRJPVTV8Q7T99.png)

#### HASH

拖拽文件到窗口，点击按钮 即可求得文件的哈希值

#### AES加/解密

在数据窗口输入数据 点击按钮 即可求得数据经过AES加解密后的值

#### RSA加/解密

在数据窗口输入数据 点击按钮 即可求得数据经过RAS加解密后的值

#### 实现原理

以上加解密都是使用Windows自带的加密库来实现的，使用的是`cryptoAPI`。

### 功能技术

![](https://bbs.kanxue.com/upload/attach/201904/833057_32J58AHSKC7KUQM.png)

#### 进程遍历

对当前所有进程进行遍历

实现原理：使用`Process32First`和`Process32Next`

#### 文件遍历

1. 对当前系统所有文件进行遍历 可查看文件属性和磁盘信息
2. 可查看文件属性和磁盘信息 双击可进入下一层目录
3. 实现原理：使用`FindFirstFile`和`FindNextFile`

#### 桌面截屏

1. 对当前屏幕进行截屏 截图存放在程序同级目录
2. 实现原理：通过GDI的方式实现屏幕画面抓取

#### 按键记录

1. 实现了一个键盘记录器 记录下所有的按键记录
2. 按键记录存储在D盘下的`log.txt`
3. 实现原理：利用`GetAsyncKeyState`函数。该函数可以判断按键状态，根据是否为按下状态来判断用户是否进行了按键操作，从而记录
4. 之所以没有用全局键盘钩子和原始输入模型是因为这两种方式都有各自的限制，MFC实现起来不太容易

#### 显示CMD结果

1. 在命令框输入CMD命令 例如`ping 127.0.0.0`
2. 等命令执行完成会在数据窗口会显示CMD命令的执行结果
3. 由于环境不一致 这个功能可能会执行失败
4. 实现原理：使用命名管道进行进程通讯，从而获得`cmd`的执行结果

#### U盘小偷

1. 在功能窗口此功能默认开启
2. 对U盘的插入和拔出进行监控
3. 拷贝U盘里所有的`.txt`文件到程序的当前目录下,如需拷贝其他文件则修改源码
4. 实现原理:响应`WM_DEVICECHANGE`监控U盘，获取盘符之后进行文件遍历 即可拷贝所需要的文件

#### 文件监控

1. 对D盘上的文件操作进行监控
2. 实现原理：Windows提供了一个文件监控的接口函数`ReadDirectoryChangesW`，这个监控函数可以对计算机上的所有文件操作进行监控。

#### 自删除

1. 点击后程序将退出进程 并删除自身
2. 实现原理：使用批处理

## 说明

1. 本程序仅用于交流和研究，请勿用于其他用途。有任何后果与本人无关
2. 由于环境不一样 所以不能保证每个功能在每台机器上都是可行的。如果程序有bug还请多多体谅
3. 如果发现缺dll或者没有测试文件的情况在工程项目下都可找到，可自行编译。
4. 本程序编写环境为W10 64 VS2017 最后附上源码
5. Githun:e9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1L8$3&6&6b7$3S2W2L8U0f1$3i4K6u0r3d9r3q4U0K9$3g2J5g2r3!0G2L8s2x3`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2019-7-16 10:53

被鬼手56编辑

，原因： 图片失效

上传的附件：

- [HackerTools.rar](javascript:void(0);)

  （2.90MB，1509次下载）
