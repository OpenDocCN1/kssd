
# 加密与解密二版菜鸟学习笔记（1）-Anti-spy

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-54.htm`](https://www.pediy.com/kssd/pediy07/pediy7-54.htm)

**【文章标题】** 菜鸟《加密与解密》二版学习笔记（1）－反调试，及 `OD` 中 `API` 断点的使用

**【软件名称】** `Anti－spy.exe`

**【下载地址】** 《加密与解密》二版配套光盘

**【使用工具】** `flyod` 1.10

**【保护方式】** `Anti-SPY` : `regmon` `filemon`

**【开发语言】** `MasM`

**【本文写于】** 2005-2-1

**【软件简介】** 《加密与解密》二版，第十章反 Anti 练习程序。

---

### 【破解分析】

**1.**

`Od` 载入，来到入口代码：

```
00401000 A>/$  6A 01                push 1
00401002   |.  6A 00                push 0
00401004   |.  6A 00                push 0
00401006   |.  6A 00                push 0                                   ; /pModule = NULL
00401008   |.  FF15 00204000        call dword ptr ds:[<&KERNEL32.GetModuleHandle>; \GetModuleHandleA
0040100E   |.  50                   push eax
0040100F   |.  E8 0C000000          call Anti-Spy.00401020 //应用程序入口

00401020   /$  8B4424 04            mov eax,dword ptr ss:[esp+4]              ;  Anti-Spy.00400000
00401024   |.  6A 00                push 0                                    ; /lParam = NULL
00401026   |.  68 A0104000          push Anti-Spy.004010A0                    ; |DlgProc = Anti-Spy.004010A0
0040102B   |.  6A 00                push 0                                    ; |hOwner = NULL
0040102D   |.  6A 65                push 65                                   ; |pTemplate = 65
0040102F   |.  50                   push eax                                  ; |hInst
00401030   |.  A3 08314000          mov dword ptr ds:[403108],eax             ;
00401035   |.  FF15 20204000        call dword ptr ds:[<&USER32.DialogBoxParamA>] ; \DialogBoxParamA //产生对话
0040103B   |.  33C0                xor eax,eax
0040103D   \.  C2 1000             retn 10

00401014   |.  50                   push eax                                  ; /ExitCode //结束进程
00401015   \.  FF15 0C204000        call dword ptr ds:[<&KERNEL32.ExitProcess>]  ; \ExitProcess
0040101B     .  C3                   retn
0040101C       90                   nop
```

**2.**

学习一下设置 `api` 断点：

参看一下当前模块中的和消息有关的 `api` 函数：

名称位于 `Anti-Spy`

地址 `区段` `类型` `名称` `注释`

`00402004` `.rdata` `导入` `KERNEL32.CloseHandle`
`00402008` `.rdata` `导入` `KERNEL32.CreateFileA`
`00402020` `.rdata` `导入` `USER32.DialogBoxParamA`
`00402014` `.rdata` `导入` `USER32.EndDialog`
`0040200C` `.rdata` `导入` `KERNEL32.ExitProcess`
`00402040` `.rdata` `导入` `USER32.FindWindowA`
`00402024` `.rdata` `导入` `USER32.GetDesktopWindow`
`00402038` `.rdata` `导入` `USER32.GetDlgItem`
`00402000` `.rdata` `导入` `KERNEL32.GetModuleHandleA`
`00402028` `.rdata` `导入` `USER32.GetWindowRect`
`00402030` `.rdata` `导入` `USER32.LoadIconA`
`00402034` `.rdata` `导入` `USER32.MessageBoxA`
`00401000` `.text`  `导出`  `<ModuleEntryPoint>`
`0040201C` `.rdata` `导入` `USER32.MoveWindow`
`00402018` `.rdata` `导入` `USER32.PostMessageA` `//******1`
`0040202C` `.rdata` `导入` `USER32.SendDlgItemMessageA` `//是给对话里的控件发消息的,我们不需要,不管它.`
`0040203C` `.rdata` `导入` `USER32.SendMessageA` `//******2`

---

#### **1** 看一下 `PostMessageA` 函数原型：

`The PostMessage function places (posts) a message in the message queue (消息队列) associated with the thread that created the specified window and then returns without waiting for the thread to process the message. Messages in a message queue are retrieved by calls to the GetMessage or PeekMessage function.`

翻译: 把消息放到当前创建了一个特定窗口的线程的消息队列里,然后不等待次线程处理这个消息就返回.
消息队列里的消息通过调用 `GetMessage` 或 `PeekMessage` 函数来获得.

```
BOOL PostMessage(
    HWND hWnd,  // handle of destination window //当前窗口的句柄
    UINT Msg,  // message to post              //要送走的消息
    WPARAM wParam,  // first message parameter  //消息附加信息
    LPARAM lParam   // second message parameter //消息附加信息
   );
```

**Return Values 返回值**
`If the function succeeds, the return value is nonzero. //成功,返回非 0`
`If the function fails, the return value is zero.       //失败返回 0`

---

#### **2** 看一下 `sendMessageA` 函数原型：

`The SendMessage function sends the specified message to a window or windows. The function calls the window procedure for the specified window and does not return until the window procedure has processed the message. The PostMessage function, in contrast, posts a message to a thread's message queue and returns immediately.`

翻译: `SendMessage` 函数给一个窗口或多个串口发送一个特定的消息.函数调用这个特定窗口的窗口过程,
直到这个窗口过程处理了这个消息才返回.相对应的 `PostMessage` 函数是把消息发送到线程的消息队列里,并马上返回.

```
LRESULT SendMessage(
    HWND hWnd,  // handle of destination window //目标窗口的句柄
    UINT Msg,  // message to send               //要发送的消息
    WPARAM wParam,  // first message parameter//消息附加信息
    LPARAM lParam   // second message parameter//消息附加信息
   );
```

**Return Values 返回值**
`The return value specifies the result of the message processing and depends on the message sent.`

---

好了,看了这两个函数,设断点试试.

首先 `PostMessageA`,好了开始:

`00402018` `.rdata` `导入` `USER32.PostMessageA` `//******` `//在这里,按 enter 键`

来到参考位置:

参考位于 `Anti-Spy:.text` 到 `USER32.PostMessageA`

```
00401060   call dword ptr ds:[<&USER32.PostMessageA>]         USER32.PostMessageA
```

右键,选在每个命令上设断点,即 `bp PostMessageA` 则发现任何操作都会中断到系统领空中.

中断太多了,不好用.

再试试 `sendMessageA`,看解释它是给窗口过程发消息,并等待窗口过程处理消息,这正是我们需要的.

我们的目标是: 点了"检测"按钮,然后让程序处理这个消息,不就知道程序在干什么了吗.^\_^

方法同上:

找到:

参考位于 `Anti-Spy:.text` 到 `USER32.SendMessageA` `//有 3 处`

```
00401103   call dword ptr ds:[<&USER32.SendMessageA>]         USER32.SendMessageA // 会中断在系统领空
00401133   mov ebp,dword ptr ds:[<&USER32.SendMessageA>]      USER32.SendMessageA // 停在此指令处
00401217   mov edi,dword ptr ds:[<&USER32.SendMessageA>]      USER32.SendMessageA // 停在此指令处
```

右键,选在每个命令上设断点,即相当于在命令行 `bp SendMessageA`,区别在于 `bp SendMessageA` 肯定会停在系统领空（即 `sendmessagea` 函数的第一条指令处）,
而我们这种设断点的方法,会中断在那里,看上面的注释.这就是这两中设法的区别.

```
00401118    > \8B7424 34         mov esi,dword ptr ss:[esp+34]
0040111C    .  8B1D 38204000      mov ebx,dword ptr ds:[<&USER32.GetDlgItem>]   ;  USER32.GetDlgItem
00401122    .  6A 00              push 0                                   ; /lParam = 0
00401124    .  6A 00              push 0                                   ; |wParam = 0
00401126    .  68 F0000000        push 0F0                                 ; |Message = BM\_GETCHECK
0040112B    .  68 FE030000       push 3FE                                 ; |/ControlID = 3FE (1022.) //是那个可选钮的 ID
00401130    .  56                 push esi                                 ; ||hWnd
00401131    .  FFD3              call ebx                                 ; |\GetDlgItem
00401133    .  8B2D 3C204000     mov ebp,dword ptr ds:[<&USER32.SendMessageA>] ; |USER32.SendMessageA //停在这里
00401139    .  50                 push eax                                 ; |hWnd
0040113A    .  FFD5              call ebp                                 ; \SendMessageA
0040113C    .  8B3D 34204000     mov edi,dword ptr ds:[<&USER32.MessageBoxA>]  ;  USER32.MessageBoxA
00401142    .  85C0              test eax,eax
00401144    .  74 24             je short Anti-Spy.0040116A
00401146    .  E8 95010000       call Anti-Spy.004012E0 //进入
{
```



## 汇编代码分析

```
004012E0   /$  6A 00                 push 0                            ; /hTemplateFile = NULL  
004012E2   |.  68 80000000          push 80                             ; |Attributes = NORMAL  
004012E7   |.  6A 03                push 3                              ; |Mode = OPEN_EXISTING  
004012E9   |.  6A 00                push 0                              ; |pSecurity = NULL  
004012EB   |.  6A 03                push 3                              ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE  
004012ED   |.  68 000000C0          push C0000000                       ; |Access = GENERIC_READ|GENERIC_WRITE  
004012F2   |.  68 BC304000         push Anti-Spy.004030BC              ; |FileName = "\\\\.\\REGVXD"  
004012F7   |.  FF15 08204000       call dword ptr ds:[<&KERNEL32.CreateFileA>] ; \CreateFileA //打开驱动文件,返回句柄  
004012FD   |.  83F8 FF             cmp eax,-1  //找到会返回此文件句柄  
00401300   |.  74 0D               je short Anti-Spy.0040130F //找到驱动,就不跳  
00401302   |.  50                push eax                          ; /hObject  
00401303   |.  FF15 04204000       call dword ptr ds:[<&KERNEL32.CloseHandle>] ; \CloseHandle //关闭文件  
00401309   |.  B8 01000000         mov eax,1  
0040130E   |.  C3                retn  
0040130F   |>  68 84304000         push Anti-Spy.00403084              ; /Title = "Registry Monitor - Sysinternals: www.sysinternals.com"  
00401314   |.  6A 00                push 0                              ; |Class = 0  
00401316   |.  FF15 40204000       call dword ptr ds:[<&USER32.FindWindowA>]  ; \FindWindowA //实际是检查窗口标题  
                                                                          //找到符合条件的顶级窗口,就返回此窗口句柄  
                                                                          //没找到,返回 0  
0040131C   |.  F7D8                neg eax     //求补码  
0040131E   |.  1BC0                sbb eax,eax //带借位减法  
00401320   |.  F7D8                neg eax     //求补码  
00401322   \.  C3                retn  
```

### 举例分析

`eax=0x00000000` 时:

- `neg eax` ---> `0x00000000` `CF=0`
- `sbb eax,eax`---> `0x00000000 - 0x00000000 -0=0x00000000`
- `neg eax` ---> `0x00000000` `CF=0`

`eax=0x00000001` 时:

- `neg eax` ---> `0x00000001` `CF=1`
- `sbb eax,eax`---> `0x00000001 - 0x00000001 -1=0xffffffff`
- `neg eax` ---> `0x00000001` `CF=1`

就是说,返回 `0` 时,运算结果为 `0`  
返回非 `0` 时,运行结果为 `1`

}

```
0040114B   .  85C0               test eax,eax               //判断一下  
0040114D   .  6A 30               push 30  
0040114F   .  74 0C               je short Anti-Spy.0040115D  //=0 就跳,表示没找到 regmon 或 filemon  
00401151   .  68 80304000         push Anti-Spy.00403080                ;  ASCII "OK"  
00401156   .  68 6C304000         push Anti-Spy.0040306C                ;  ASCII "RegMON is Running!  
"  
0040115B   .  EB 0A               jmp short Anti-Spy.00401167  
0040115D   >  68 64304000         push Anti-Spy.00403064                ;  ASCII "Error"  
00401162   .  68 40304000         push Anti-Spy.00403040                ;  ASCII "Can't find RegMON with this method!"  
00401167   >  56                push esi  
00401168   .  FFD7               call edi //USER32.MessageBoxA  
0040116A   >  6A 00               push 0  
0040116C   .  6A 00               push 0  
0040116E   .  68 F0000000         push 0F0  
00401173   .  68 FF030000         push 3FF  
00401178   .  56                push esi  
00401179   .  FFD3               call ebx  
0040117B   .  50                push eax  
0040117C   .  FFD5               call ebp  
0040117E   .  85C0               test eax,eax  
00401180   .  0F84 40010000      je Anti-Spy.004012C6  
00401186   .  E8 A5010000         call Anti-Spy.00401330  
0040118B   .  85C0               test eax,eax  
0040118D   .  6A 30               push 30  
0040118F   .  74 1C               je short Anti-Spy.004011AD  
00401191   .  68 80304000         push Anti-Spy.00403080                ;  ASCII "OK"  
00401196   .  68 28304000         push Anti-Spy.00403028                ;  ASCII "FileMON is Running!  
"  
0040119B   .  56                push esi  
0040119C   .  FFD7               call edi  
0040119E   .  5F                pop edi  
0040119F   .  5E                pop esi  
004011A0   .  5D                pop ebp  
004011A1   .  B8 01000000         mov eax,1  
004011A6   .  5B                pop ebx  
004011A7   .  83C4 20            add esp,20  
004011AA   .  C2 1000            retn 10  
```

## 相关 API 函数

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

The `CloseHandle` function closes an open object handle. 关闭一个内核对象

```
BOOL CloseHandle(  
    HANDLE hObject   // handle to object to close  
);  
```

### Parameters

`hObject`

Identifies an open object handle. // 一个打开的对象的句柄标识

### Return Values

If the function succeeds, the return value is nonzero. //成功返回非 0

If the function fails, the return value is zero. //失败返回 0

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

The `CreateFile` function creates or opens the following objects and returns a handle that can be used to access the object: 创建或打开一个文件

```
HANDLE CreateFile(  
    LPCTSTR lpFileName,          // pointer to name of the file //指向文件名  
    DWORD dwDesiredAccess,       // access (read-write) mode  
    DWORD dwShareMode,           // share mode  
    LPSECURITY_ATTRIBUTES lpSecurityAttributes, // pointer to security attributes  
    DWORD dwCreationDistribution,          // how to create  
    DWORD dwFlagsAndAttributes,  // file attributes  
    HANDLE hTemplateFile   // handle to file with attributes to copy  
);  
```

### Return Values

//成功返回句柄

If the function succeeds, the return value is an open handle to the specified file.

If the specified file exists before the function call and `dwCreationDistribution` is `CREATE_ALWAYS` or `OPEN_ALWAYS`, a call to `GetLastError` returns `ERROR_ALREADY_EXISTS` (even though the function has succeeded).

If the file does not exist before the call, `GetLastError` returns zero.

If the function fails, the return value is `INVALID_HANDLE_VALUE`. To get extended error information, call `GetLastError`.

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

The `FindWindow` function retrieves the handle to the top-level window whose class name and window name match the specified strings. This function does not search child windows.

//翻译: `FindWindow` 函数获得符合指定的类名和窗口名的顶级窗口的句柄.

```
HWND FindWindow(  
    LPCTSTR lpClassName,  // pointer to class name  指向类名的指针  
    LPCTSTR lpWindowName   // pointer to window name 指向窗口名的指针  
);  
```

### Return Values

//成功,返回这个窗口的句柄,失败返回 NULL

If the function succeeds, the return value is the handle to the window that has the specified class name and window name.

If the function fails, the return value is NULL. To get extended error information, call `GetLastError`.

—————————————————————————————————

## 总结

仅以学习为目的，知识是宝贵的，破解是次要的。

有些程序，不经过消息循环，通过 `sendmessagea` 发送消息后，就开始执行相应的处理代码，对于窗口的操作(如点按钮)，在不知道设什么断点时，可以使用一下 `BP sendmessagea` 试试！当然这个断点也并不是总是好用的！

分析程序时，多查查 api 函数，会了解更多的底层的东西。

另：这个程序不能设置消息断点，请教大侠们指点一下。

我认为按下一个按钮就会，发送一个 `WM_COMMAND` 消息，在通过消息循环处理。这个程序怎么不行呢？不会是没经过消息循环吧，那么怎么拦截这个按钮？![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) 

请大侠们多指点了！

感谢你看完此菜文！

—————————————————————————————————

## 附件

`Anti-Spy.rar` 

`4dd466d0e1e0cc123afbe504da2462ca`
