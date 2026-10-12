
# OD 被 Anti 的原因分析及应对之道

> 原文：[`www.pediy.com/kssd/pediy10/102366.html`](https://www.pediy.com/kssd/pediy10/102366.html)

本文链接：`http://bbs.pediy.com/showthread.php?t=102366`

OD 载入程序就自动退出是比较恼人的，还没开始调试呢就退出了，这可让人如何是好。初学破解的人一定会遇到这个问题,怎样解决呢，网上虽然有零星的介绍但都不全面，以下是我总结的一些，希望能对各位初学者有所帮助。

（比如`Peid`、`FI`查壳查不到，`OD`一载入就退出，这极有可能是`VMProtect`的保护（虚拟机保护），用`EXEinfo`可以查出来一些版本的`VMP`，如果有这个提示那就更确定无疑了“A debugger has been found running in your system.Please, unload it from memory and restart your program”。）

### 1. 更换几个 OD 试试，OllyICE、Shadow、加强版等

### 2. 用附加的方式加载程序，文件-->附加,能解决很多问题

### 3. OD 目录下，将 475K 的 DbgHelp.dll 文件换成近 1M 大小的 DbgHelp.dll 文件，475K 的有溢出漏洞，这条比较关键

### 4. 使用 StrongOD 插件，（StrongOD+原版 OD 试试），这条比较关键

### 5. StrongOD 中选择 CreateAsRestrict

### 6. 尝试命令 bp ExitProcess，看能否发现什么线索

### 7. 改变 ollydbg.ini 中的驱动名称，修改版的 OD 不需要自己改

- `DriverName` - 驱动文件名，设备对象名
- `DriverKey` - 和驱动通信的 key
- `HideWindow` - 是否隐藏窗口，1 为隐藏，0 为不隐藏
- `HideProcess` - 是否隐藏 od 进程，1 为隐藏，0 为不隐藏
- `ProtectProcess` - 是否隐藏保护 Od 进程，1 为保护，0 为不保护

### 8. 改 OD 窗体类名，用的修改版的话一般都改过了，不需要自己再改

方法如下：

主窗体类名:

引用:
- `VA:004B7218`
- `Offset:000B6018`

各子窗体类名：

引用:
- `VA:004B565B ~ 004B568A`
- `Offset:000B445B ~ 000B448A`

改成任意，可以过`GetWindow`检测

### 9. 手动修改程序“导出表”中的“函数名数目”值

上面方法不管用再试试它

方法：使用“LordPE”打开要编辑的 PE 程序，然后依次选择\[目录\]->\[导出表对应的“..”按钮\]，把“函数名数目”的值减 1，并点击“保存”按钮，就 OK 了。为了好看些，也可以把“函数数目”和“函数名数目”的值都同时减 1 并保存，效果一样。

解释：一般情况下 EXE 不会加“导出表”，如果加了，就应该给出所导出的 API 函数。当我们打开这类 PE 程序(EXE 版)时，会发现它存在“导出表”，但“导出表”中并没有导出的 API 函数。同时“函数数目”和“函数名数目”的值都比原 PE 程序设置的值大了 1(如：EXE 版“导出表”列表中显示了 0 个导出的 API 函数，壳将其“函数数目”和“函数名数目”的值都设置成了 1；DLL 版“导出表”列表中显示了 0xD 个导出的 API 函数，壳将其“函数数目”和“函数名数目”的值都设置成了 0xE。)。所以我们将其减 1，就 OK 了。被修改过的 PE 程序，可以正常运行，不会有任何影响。

这只是我的一点总结，附加方式加载、替换 `DBGHELP.DLL`、使用 `StrongOD` 插件和修改导出表函数名数目的方法是可行的，能够解决一些问题。当然这些方法可能并不全面。

### ANTI-OD 原因解读：

概括来说：`TLS` 回调函数在入口点之前执行，并进行了 `ANTI-OD` 的操作.

具体请看：

`TLS` 数据初始化和 `TLS` 回调函数都会在入口点之前执行，也就是说 `TLS` 是程序最开始运行的地方，因此可以在这里防止 `ANTI-OD` 的代码，检测并关闭 `OD`。

应对方法：

默认情况下 `OllyDbg` 载入程序将会暂停在入口点，应该配置一下 `OllyDbg` 使其在 `TLS` 回调被调用之前中断在实际的 loader。

通过“选项->调试选项->事件->第一次中断于->系统断点”来设置中断于 `ntdll.dll` 内的实际 loader 代码。这样设置以后，`OllyDbg` 将会中断在位于执行 `TLS` 回调的 `ntdll!LdrpRunInitializeRoutines()`之前的 `ntdll!\_LdrpInitializeProcess()`，这时就可以在回调例程中下断并跟踪了。例如：在内存映像的 `.text` 代码段上设置内存访问断点，就可以断在 `TLS` 回调函数里。

实例代码：程序见附件，用原版 `OD` 测试，参考了某位大虾的代码。

```asm
.386

.model flat,stdcall
option casemap:none
include windows.inc
include user32.inc
include kernel32.inc

includelib user32.lib
includelib kernel32.lib

.data?
dwTLS\_Index dd  ?

OPTION DOTNAME

;; 定义一个 TLS 节          
.tls  SEGMENT                 
TLS\_Start LABEL  DWORD
 dd   0100h   dup ("slt.")
TLS\_End  LABEL  DWORD
.tls  ENDS
OPTION NODOTNAME

.data
TLS\_CallBackStart  dd  TlsCallBack0
TLS\_CallBackEnd    dd  0
szTitle            db  "Hello TLS",0
szInTls            db  "我在 TLS 里",0
szInNormal         db  "我在正常代码内",0
szClassName        db  "ollydbg"        ; OD 类名

;这里需要注意的是，必须要将此结构声明为 PUBLIC,用于让连接器连接到指定的位置，
;其次结构名必须为\_tls\_uesd 这是微软的一个规定。编译器引入的位置名称也如此。

PUBLIC _tls\_used

_tls\_used IMAGE\_TLS\_DIRECTORY <TLS\_Start, TLS\_End, dwTLS\_Index, TLS\_CallBackStart, 0, ?>

.code

;******************************************************************
;; TLS 的回调函数
TlsCallBack0 proc Dllhandle:LPVOID,dwReason:DWORD,lpvReserved:LPVOID   
 mov    eax,dwReason ;判断 dwReason 发生的条件
 cmp    eax,DLL\_PROCESS\_ATTACH  ; 在进行加载时被调用
 jnz    ExitTlsCallBack0
 invoke FindWindow,addr szClassName,NULL  ;通过类名进行检测
 .if    eax     ;找到
      invoke   SendMessage,eax,WM\_CLOSE,NULL,NULL
 .endif
 invoke MessageBox,NULL,addr szInTls,addr szTitle,MB\_OK
 mov    dword ptr\[TLS\_Start\],0   
 xor    eax,eax
 inc    eax
ExitTlsCallBack0: 
 ret
TlsCallBack0  ENDP
;******************************************************************

Start:
 invoke  MessageBox,NULL,addr szInNormal,addr szTitle,MB\_OK
 invoke  ExitProcess, 1
 end  Start
```

关于反调试的实现方法请看“[脱壳的艺术](http://bbs.pediy.com/showthread.php?t=50119)”一文。

参考文献：

`http://www.unpack.cn/viewthread.php?tid=35013&extra=page%3D1`

......

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `dbghelp.rar` |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `StrongOD.v0.2.6.415.rar` |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `终结版.rar` |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `TLS\_Antiod.rar` |

\[注：附件请到论坛上下载：[`bbs.pediy.com/showthread.php?t=102366`](http://bbs.pediy.com/showthread.php?t=102366)\]
