
# Billy Belceb 病毒编写教程 for Win32

> 原文：[`www.pediy.com/kssd/pediy06/pediy65789.htm`](https://www.pediy.com/kssd/pediy06/pediy65789.htm)

## Win32 反调试

~~~~~~~~~~~~~~~

下面我将给出一些花招用来保护你的病毒或者程序不被调试(所有级别的，应用级和系统级)。我希望你将喜欢它。

### Win98/NT: 用 IsDebuggerPresent 检测应用级调试器

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个 API 函数在 Win95 中没有，所以你不得不自己检测它的存在，并和应用级调试器(如 `TD32`)一起工作。而且它工作得很好。让我们看看在 Win32 API 参考列表里面是怎么写的。

--------------------------------------------

`IsDebuggerPresent` 函数表明调用的进程是否是在一个调试器下运行。这个函数从 `KERNEL32.DLL` 中导出。

`BOOL IsDebuggerPresent(VOID)`

参数

====

这个函数没有参数。

返回值

======

- 如果当前进程是在一个调试器下运行，返回值是非 0 值。
- 如果当前进程不在调试器下运行，返回值是 0。

--------------------------------------------

所以演示这个的例子很简单。下面就是。

```
;--------从这儿开始剪切------------------------------------------------------
    .586p
    .model flat
extrn   GetProcAddress:PROC
extrn   GetModuleHandleA:PROC
extrn   MessageBoxA:PROC
extrn   ExitProcess:PROC

            .data
szTitle         db      "IsDebuggerPresent Demonstration",0
msg1            db      "Application Level Debugger Found",0
msg2            db      "Application Level Debugger NOT Found",0
msg3            db      "Error: Couldn't get IsDebuggerPresent.",10
            db      "We're probably under Win95",0
@IsDebuggerPresent db   "IsDebuggerPresent",0
K32             db      "KERNEL32",0
    .code

antidebug1:
    push    offset K32                     ; Obtain KERNEL32 base address
    call    GetModuleHandleA
    or      eax,eax                     ; Check for fails
    jz      error
    push    offset @IsDebuggerPresent     ; Now search for the existence
    push    eax                         ; of IsDebuggerPresent. If
    call    GetProcAddress              ; GetProcAddress returns an
    or      eax,eax                     ; error, we assume we're in
    jz      error                       ; Win95
    call    eax                         ; Call IsDebuggerPresent
    or      eax,eax                     ; If it's not 0, we're being
    jnz     debugger_found              ; debugged

debugger_not_found:
    push    0                           ; Show "Debugger not found"
    push    offset szTitle
    push    offset msg2
    push    0
    call    MessageBoxA
    jmp     exit

error:
    push    00001010h                   ; Show "Error! We're in Win95"
    push    offset szTitle
    push    offset msg3
    push    0
    call    MessageBoxA
    jmp     exit

debugger_found:
    push    00001010h                   ; Show "Debugger found!"
    push    offset szTitle
    push    offset msg1
    push    0
    call    MessageBoxA

exit:
    push    00000000h                   ; Exit program
    call    ExitProcess

end     antidebug1
;--------到这儿为止剪切------------------------------------------------------
```

很美妙吧?Micro$oft 为我们做了这个工作:)但是，毫无疑问，不要期望这个方法对 `SoftICE` 有效，上帝;)

## Win32:知道我们是否被一个调试器调试的另外一个方法

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果你看了由 `Murkry/iKX` 写的在 `Xine-3` 中发表的"Win95 Structures and Secrets"这篇文章的话，你将意识到在 `FS` 寄存器中有一个非常酷的结构。看看 `FS:[20h]` 域...它是 '`DebugContext'`。只要这么做:

```
        mov     ecx,fs:[20h]
        jecxz   not_being_debugger
        [...]   <--- do whatever, we're being debugged :)
```

所以，如果 `FS:[20h]` 是 0，我们就没有被调试。只要享受这个小而简单的方法来检测调试器!当然了，这个不能对 `SoftICE` 有效...

## Win32:用 SEH 来停止应用级调试器

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我仍然不知道为什么，但是如果程序简单地使用了 `SEH`，应用级调试器就死了。而且如果我们制造错误，代码模拟器也死了:)`SEH`，正如我在我的发表在 `DDT#1` 中的一篇文章所说的，可以用来达到很多有意思的目的。你可以看看“高级 Win32 技术”(`Advanced Win32 techniques`)这一章的 `SEH` 部分。

你所必须做的是使 `SEH handler` 指向你想继续执行代码的地方，而当 `SEH handler` 被安装了，你激活了一个标志(一个好的选择是在 `00000000h` 内存地址试图做些事情);)

我希望你看懂了这个。如果没有...恩，忘记它:)而且，正如以前其它的方法一样，这个对 `SoftICE` 没有用。

## Win9X:检测 SoftICE (I)

~~~~~~~~~~~~~~~~~~~~~~~~

这里，我必须向 `Super/29A` 致敬，因为他是告诉我这个方法的人。我把这个分成两个部分:在这个部分中，我们将看到从一个 `Ring-0` 病毒的角度该怎么做。我不会给出整个例子程序，因为它将占一些不必要的行，但是你必须知道这个方法必须是在 `Ring-0` 下执行，而且因为 `Call-back` 问题(你还记得吗？)，`VxDCall` 必须重建。

我们将使用 `Virtual Machine Manager (VMM)` 的 `Get_DDB` 服务，所以这个服务将为 `00010146h (VMM_Get_DDB)`。让我们看看 `SDK` 中关于这个服务的信息。

-------------------------------------

```
     mov    eax, Device_ID
     mov    edi, Device_Name
     int    20h                       ; VMMCall Get_DDB
     dd     00010146h
     mov    [DDB], ecx
```

- 确定一个 `VxD` 是否对特定设备安装了，如果安装了就会返回一个那个设备的 `DDB`。
- 使用 `ECX`，`flags`(标志)。
- 如果函数成功会返回指定设备的 `DDB`；
- 否则，返回 0。

`?Device_ID`: 设备标志符。对于基于名字的设备，这个参数可以为 0。

`?Device_Name`: 一个 8－字符的设备名，不够用空字符填充。如果 `Device_ID` 为 0 的时候，这个参数才被需要。设备名大小写敏感。

-------------------------------------

现在，你想要知道为什么了，非常简单，`SoftICE VxD` 的 `Device_ID` 域对于所以程序来说是一个常量，正如它在 `Micro$oft` 注册的，所以我们就有了对付不可思议的 `SoftICE` 的武器了。它的 `Device_ID` 总是 `202h`。所以我们应该使用如下的代码:

```
        mov     eax,00000202h
        VxDCall VMM_Get_DDB
        xchg    eax,ecx
        jecxz   NotSoftICE
        jmp     DetectedSoftICE
```

`NotSoftICE` 应该是继续我们的病毒代码的地方，而 `DetectedSoftICE` 标记应该是既然我们已经知道我们的敌人还活着，该采取一些行动的地方，我不建议任何破坏性的事情，因为，例如，将会伤害我的电脑，因为我总是使得 `SoftICE` 处于激活状态:)

## Win9X: 检测 SoftICE (II)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~

下面是另外一种方法来检测我所钟爱得 `SoftICE` 的存在，但是基于以前的同样的观点： `202h` ;) 我必须再次对 `Super` 致敬:)好了，在 `Ralph Brown` 的中断列表中，我们可以看到一个在中断 `2Fh`(多元)的 `1684h` 服务。

----------------------------------------------

```
 Inp.:
        AX = 1684h
        BX = virtual device (VxD) ID (看 #1921)
        ES:DI = 0000h:0000h
 返回:  ES:DI -> VxD API  入口, 或者 0:0 如果这个 VxD 不支持一个 API
 说明:  一些 Windows 增强－模式虚拟设备提供了一些应用程序可以访问的服务。例如，Virtual  Display Device(VDD)提供了由 WINOLDAP 轮流使用的 API。
```

----------------------------------------------

所以，你在 `BX` 中放一个 `202h`，并指向这个函数。然后你要说了...“嗨，Billy... 我用于中断多傻呀?"。我的回答是...使用 `VxDCALL0`!!!

## Win32: 检测 SoftICE (III)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你正等待的是比较权威的和令人惊奇的招...同时在 `Win9x` 和 `WinNT` 环境寻找 `SoftICE`!它非常简单，100%基于 API，而且没有"脏"招来进行兼容性。这个答案并没有你想的那么隐蔽...关键是在你肯定以前已经用过的 API 函数中:`CreateFile`。是的，那个 API...不迷人吗?好了，我得试图打开下面的东西：

- `SoftICE for Win9x : "\\.\SICE"`
- `SoftICE for WinNT : "\\.\NTICE"`

如果这个 API 返回给我们和 `-1 (INVALID_HANDLE_VALUE)` 不同的东西，`SoftICE` 就是处于激活状态!下面是演示程序:

```
;--------从这里开始剪切--------------------------------------------------------
        .586p
        .model  flat
extrn   CreateFileA:PROC
extrn   CloseHandle:PROC
extrn   MessageBoxA:PROC
extrn   ExitProcess:PROC
        .data
szTitle         db      "SoftICE detection",0
szMessage       db      "SoftICE for Win9x : "
answ1           db      "not found!",10
                db      "SoftICE for WinNT : "
answ2           db      "not found!",10
                db      "(c) 1999 Billy Belcebu/iKX",0
nfnd            db      "found!    ",10
SICE9X          db      "\\\\.\\SICE",0
SICENT          db      "\\\\.\\NTICE",0
        .code

DetectSoftICE:
        push    00000000h                   ; Check for the presence of
        push    00000080h                   ; SoftICE for Win9x envirome-
        push    00000003h                   ; nts...
        push    00000000h
  push    00000001h
        push    0C0000000h
        push    offset SICE9X
        call    CreateFileA
        inc     eax
        jz      NoSICE9X
        dec     eax
        push    eax                         ; Close opened file
        call    CloseHandle
        lea     edi,answ1                   ; SoftICE found!
        call    PutFound

NoSICE9X:        
        push    00000000h                   ; And now try to open SoftICE
        push    00000080h                   ; for WinNT...
```



```
push    00000003h  

push    00000000h  

  push    00000001h  

        push    0C0000000h  

        push    offset SICENT  

        call    CreateFileA  

        inc     eax  

        jz      NoSICENT  

        dec     eax  

        push    eax                             ; Close file handle  

        call    CloseHandle  

        lea     edi,answ2                       ; SoftICE for WinNT found!  

        call    PutFound  

NoSICENT:  

        push    00h                             ; Show a MessageBox with the  

        push    offset szTitle                  ; results  

        push    offset szMessage  

        push    00h  

        call    MessageBoxA                      

        push    00h                             ; Terminate program  

        call    ExitProcess  

PutFound:  

        mov     ecx,0Bh                         ; Change "not found" by  

        lea     esi,nfnd                        ; "found"; address of where  

        rep     movsb                           ; to do the change is in EDI  

        ret  

end     DetectSoftICE  

;--------到这里为止剪切--------------------------------------------------------  
```

    这个真的起作用了，相信我:)这个同样的方法可以应用于其它"敌对"驱动，只要对它研究一点点就可以了。 

    这里使用`Win9X`来杀掉调试器硬件断点。 

`Win9X`：杀掉调试器硬件断点 

你是否在想调试寄存器(`DR?`)，我们有一个小问题:它们在`WinNT`下是特权级指令。这一招由这些简单的事情组成:注意`DR0`、`DR1`、`DR2`和`DR3`(它们由调试器用来作为硬件断点的)。所以，简单的使用这个代码，你就可以避开调试器: 

```
xor     edx,edx 

mov     dr0,edx 

mov     dr1,edx 

mov     dr2,edx 

mov     dr3,edx 
```

    哈哈，是不是很有意思呀?:) 

最后的话 

    这是一些简单的反调试招。我希望你能够在你的病毒中没有任何问题的使用它们，看你了! 

    `4dd466d0e1e0cc123afbe504da2462ca`
