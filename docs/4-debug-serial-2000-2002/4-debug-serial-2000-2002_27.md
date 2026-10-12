
# CacheX

> 原文：[`www.pediy.com/kssd/pediy02/forum130.htm`](https://www.pediy.com/kssd/pediy02/forum130.htm)

如何在 `TRW2000v1.22` 中抓如下样式的屏幕

```
:004F4055 55                   push ebp
:004F4056 8BEC                 mov ebp, esp
:004F4058 83EC0C               sub esp, 0000000C
:004F405B 8B4D08               mov ecx, dword ptr [ebp+08]
:004F405E 53                   push ebx
:004F405F 56                   push esi
:004F4060 33F6                 xor esi, esi
:004F4062 8B4104               mov eax, dword ptr [ecx+04]
:004F4065 57                   push edi
:004F4066 40                   inc eax
:004F4067 C745FC79786573       mov [ebp-04], 73657879
:004F406E C745F865626162       mov [ebp-08], 62616265
:004F4075 8945F4               mov dword ptr [ebp-0C], eax
```

2. 运行 `TRW2000` 后，运行 `CacheX for Internet Explorer v3.00`

（download: [`http://www.mwso.com/`](http://www.mwso.com/)）

会弹出"Debug mode not allowed"对话框，如何解决，请高手指点。

这个软件没有加壳，部分结果如下（启动的时候）：

1. 调用 `kernel32.dll` 中的 `IsDebuggerInstalled()` 判断是否驻留了系统调试器。
2. 调用 `CreateFileA()` 三次，检查 `SoftICE` 的 `Driver` 是否驻留在内存。
3. 调用 `CreateFileA()` 一次，是用来检查 `CRC` 的。
4. 调用 `int 3` 两次，用来检查 `debugger`。
5. `FrogsICE` 还报告检测到它用 `SEH` 来发现 `debugger`。
6. 程序在执行过程中会经常用 `GlobalAlloc()` 或 `LocalAlloc()` 分配一段内存，然后动态解出一段指令来放到刚分配的空间中并执行，执行完毕后又将内存释放掉。

该程序的偏移地址一般为 `004XXXXX`，如果当前 `EIP` 为 `006XXXXX`，则说明在执行动态解压出来的代码。

我将上述 1～4 打补丁之后，结果发现有一处动态解压出来的代码始终是错误的，结果总是非法操作。如下：

```
......
CS:0043C7D4  CALL ESI
......
```

即如果改了程序文件之后，`CS:ESI` 处的指令是乱七八糟的指令，说明解压出错，导致非法操作。

猜测该程序动态解码时可能与程序的其它部分相关。如果真是这样，这个玩意儿比较难搞定。

安装时间记录在

`HKEY_CLASSES_ROOT\CLSID\{5EFB6220-0DBD-11D4-841A-0060978867CF}\AutoConvertTo\@` `4dd466d0e1e0cc123afbe504da2462ca`
