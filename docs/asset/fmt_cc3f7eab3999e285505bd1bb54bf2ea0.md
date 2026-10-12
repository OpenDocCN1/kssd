
# Billy Belceb 病毒编写教程 for Win32

> 原文：[`www.pediy.com/kssd/pediy06/pediy657811.htm`](https://www.pediy.com/kssd/pediy06/pediy657811.htm)

## 【高级 Win32 技术】

在这一章，我将讨论一些那些不需要一整章来讨论的技术，但是，不是很容易忘记的:)所以，下面我将讨论这些东西:

- `Structured Exception Handler(SEH)`
- `MultiThreading(多线程)`
- `CRC32 (IT/ET)`
- `AntiEmulators(反模拟)`
- `Overwriting .reloc section(写.reloc 节)`

### Structured Exception Handler

结构化异常处理(Structured Exception Handler),简称 `SEH`，是所有 `Win32` 环境的一个非常酷的特点。它所做的非常容易理解:如果一个一般保护错误(简称 `GPF`)发生了，控制会自动传到当前存在的 `SEH` handler。你看到了它的辅助作用了吗?如果你把所有东西弄乱了，你将能够(仍然能)保持你的病毒没法被发现:)指向 `SEH` handler 的指针是在 `FS:[0000]`中的。所以，你可以很容易地设置你自己的新 `SEH` handler(但是要记住保存旧的!)如果一个错误发生了，控制将会传给你的 `SEH` handler 例程，但是堆栈将会混乱。幸运的是，`Micro$oft` 已经在设置我们的 `SEH` handler 之前把堆栈放到 `ESP+8` 的地方了:)所以，简单的我们只要恢复它并把旧的 `SEH` handler 设置回去就可以了:)让我们看看一个 `SEH` 使用的一个简单例子:

```
;--------从这里开始剪切-------------------------------------------------------
        .386p
        .model  flat                          ; Good good... 32 bit r0x0r
extrn   MessageBoxA:PROC                      ; Defined APIs
extrn   ExitProcess:PROC

        .data
szTitle        db      "Structured Exception Handler [SEH]",0
szMessage      db      "Intercepted General Protection Fault!",0
        .code
start:
        push    offset exception_handler      ; Push our exception handler
                                             ; offset
        push    dword ptr fs:[0000h]         ;
        mov     dword ptr fs:[0000h],esp

errorhandler:
        mov     esp,[esp+8]                 ; Put the original SEH offset
                                             ; Error gives us old ESP
                                             ; in [ESP+8]
        pop     dword ptr fs:[0000h]        ; Restore old SEH handler
        push    1010h                       ; Parameters for MessageBoxA
        push    offset szTitle
        push    offset szMessage
        push    00h
        call    MessageBoxA                 ; Show message :)
        push    00h

        call    ExitProcess                 ; Exit Application
setupSEH:
        xor     eax,eax                     ; Generate an exception
        div     eax
end     start
;--------到这里为止剪切-------------------------------------------------------
```

正如在"Win32 反调试"那一章所看到的，除此之外 `SEH` 还有另外一个特色:)它愚弄了大多数应用级的调试器。为了使你的设置一个新的 `SEH` handler 更简单，这里你可以用一些宏来做做这个(hi,Jacky!):

```
; Put SEH - Sets a new SEH handler
pseh    macro   what2do
        local   @@over_seh_handler
        call    @@over_seh_handler
        mov     esp,[esp+08h]
        what2do
@@over_seh_handler:
        xor     edx,edx
        push    dword ptr fs:[edx]
        mov     dword ptr fs:[edx],esp
        endm

; Restore SEH - Restore old SEH handler
rseh    macro
        xor     edx,edx
        pop     dword ptr fs:[edx]
        pop     edx
        endm
```

它的用法非常简单。例如:

```
        pseh    <jmp SEH_handler>
        div     edx
        push    00h
        call    ExitProcess

SEH_handler:
        rseh
        [...]
```

下面的代码，如果执行了，将会在`rseh`宏之后继续，而不是终止进程。清楚了吗?:)

### 多线程

当我被告知这个可以在 `Win32` 环境很容易实现的时候，在我的脑海中的是许多它的用处:执行代码而其它的代码(也是我们病毒的)也在执行是一个美梦，因为你节约了时间:)

一个多任务的过程的主要算法是:

1. 创建你想要运行的相关线程的代码
2. 在父进程的代码中等待子进程结束

这个看起来很难，但是有两个 `API` 可以救我们。它们的名字：`CreateThread` 和 `WaitForSingleObject`。让我们看看 `Win32` `API` 列表关于这两个 `API` 是怎么说的...

`CreateThread` 函数在调用进程的地址空间中创建一个线程执行。

```
HANDLE CreateThread(
  LPSECURITY_ATTRIBUTES lpThreadAttributes,  // ptr to thread security attrs

  DWORD dwStackSize,                 // initial thread stack size, in bytes
  LPTHREAD_START_ROUTINE lpStartAddress,      // pointer to thread function
  LPVOID lpParameter,                     // argument for new thread
  DWORD dwCreationFlags,                      // creation flags
  LPDWORD lpThreadId              // pointer to returned thread identifier
);
```

参数

`?lpThreadAttributes`:指向一个确定返回句柄可以由子进程继承的 `SECURITY_ATTRIBUTES` 结构。如果 `lpThreadAttributes` 是 `NULL`，这个句柄不能被继承。

`Windows NT`: 这个结构的 `lpSecurityDescriptor` 成员指定新线程的安全描述。如果 `lpThreadAttributes` 是 `NULL`，这个线程获得一个缺省安全描述。

`Windows 95`: 这个结构的 `lpSecurityDescriptor` 成员被忽略了。

`?dwStackSize`: 以字节数指定新线程的堆栈大小。如果指定了 0，堆栈的大小缺省的和进程的主线程的堆栈大小一样。堆栈是在进程的内存空间中自动开辟的，并在线程终止的时候释放。注意如果需要的话，堆栈大小会增加。`CreateThread` 试图把由 `dwStackSize` 指定的大小提交字节数，如果大小超过了可利用的内存的话，就会失败。

`?lpStartAddress`:新线程的开始地址。这个通常是一个用 `WINAPI` 调用惯例声明的函数，这个函数接受一个 32-bit 的指针的参数，并返回一个 32-bit 的退出码。它的原型是:

```
DWORD WINAPI ThreadFunc( LPVOID );
```

`?lpParameter`: 指定一个传给线程的 32-bit 的参数。

`?dwCreationFlags`:指定控制线程创建的额外标志。如果 `CREATE_SUSPENDED` 标志被指定了，线程将以一个挂起状态创建，除非 `ResumeThread` 函数调用，将不会运行。如果这个值是 0，线程在创建之后立即运行。这次，没有其它的支持的值。

`?lpThreadId`: 指向一个接受线程标志的 32bit 变量。

返回值

`?` 如果函数成功了，返回值是一个新线程的句柄。

`?` 如果函数失败了，返回值是 `NULL`。为了获得详细的错误信息，调用 `GetLastError`。

`Windows 95`: `CreateThread` 仅仅是在一个 32-bit 的上下文中的时候才成功。一个 32-bit `DLL` 不能创建一个额外的线程，当那个 `DLL` 正在被一个 16-bit 程序调用的时候。

`WaitForSingleObject` 函数当如下的情况发生的时候返回:

`?` 指定的对象在 signaled 状态。

`?` 过期间隔逝去了。

```
DWORD WaitForSingleObject(
  HANDLE hHandle,                       // handle of object to wait for
  DWORD dwMilliseconds             // time-out interval in milliseconds
);
```

参数

`?hHandle`:识别对象。对一个对象类型的列表，它的句柄可以指定，看看接下来的评论。

`Windows NT`:句柄必须有 `SYNCHRONIZE` 访问。想知道更多的信息，看看 `Access Masks and Access Rights`(访问标志和访问权限)。

`?dwMilliseconds`: 指定过期间隔，以毫秒形式。如果间隔过了，甚至对象的状态是 nonsignaled，这个函数就返回。如果 `dwMilliseconds` 是 0，这个函数就测试对象的状态并立即返回。如果 `dwMilliseconds` 是 `INFINITE` 这个函数从不会过期。

返回值

`?` 如果函数成功了，返回值表明了导致函数返回的事件。

`?` 如果函数失败了，返回值是 `WAIT_FAILED`。为了获得详细的错误信息，调用 `GetLastError`。

如果这个对你来说还不够，或者你不懂试图解释给你听的子句的话，下面给出一个多线程的 ASM 例子。

```
;-------从这里开始剪切------------------------------------------------------
       .586p
       .model flat
extrn   CreateThread:PROC
extrn   WaitForSingleObject:PROC
extrn   MessageBoxA:PROC
extrn   ExitProcess:PROC

       .data
tit1            db      "Parent Process",0
msg1            db      "Spread your wings and fly away...",0
tit2            db      "Child Process",0
msg2            db      "Billy's awesome bullshit!",0
lpParameter     dd      00000000h
lpThreadId      dd      00000000h
       .code
multitask:
        push    offset lpThreadId               ; lpThreadId
        push    00h                             ; dwCreationFlags
        push    offset lpParameter              ; lpParameter
        push    offset child_process            ; lpStartAddress
        push    00h                             ; dwStackSize
        push    00h                             ; lpThreadAttributes
        call    CreateThread
; EAX = Thread handle
        push    00h                             ; 'Parent Process' blah blah
        push    offset tit1
        push    offset msg1
        push    00h
        call    MessageBoxA
        push    0FFh                            ; Wait infinite seconds
        push    eax                             ; Handle to wait (thread)
        call    WaitForSingleObject
        push    00h                             ; Exit program
        call    ExitProcess
child_process:
        push    00h                             ; 'Child Process' blah blah
        push    offset tit2
        push    offset msg2
        push    00h
        call    MessageBoxA
        ret
end     multitask
```
