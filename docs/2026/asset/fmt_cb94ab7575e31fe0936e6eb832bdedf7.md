
```asm
  .text:00417042                 mov     [ebx+4], eax

  .text:00417045                 mov     [ebx+21h], al

  .text:00417048                 mov     [ebp+Memory], eax

  .text:0041704B                 mov     ecx, [ebp+arg\_0] ; 注册码

  .text:0041704E                 push    ecx

  .text:0041704F                 lea     edi, [eax+19h]  ; 0x19

  .text:00417052                 mov     byte ptr [ebp+var\_4], 3

  .text:00417056                 call    sub\_416CB0      ; 去"-"

  .text:0041705B                 add     esp, 0Ch

  .text:0041705E                 lea     ecx, [ebp+Memory]

  .text:00417061                 mov     [ebp+Memory], eax

  .text:00417064                 call    sub\_40C990      ; 转换为大写

  .text:00417069                 mov     esi, [ebp+Memory]

  .text:0041706C                 mov     eax, esi

  .text:0041706E                 lea     edx, [eax+1]

  .text:00417071

  .text:00417071 loc\_417071:                             ; CODE XREF: sub\_417000+78j

  .text:00417071                 mov     cl, [eax]

  .text:00417073                 add     eax, 1

  .text:00417076                 test    cl, cl

  .text:00417078                 jnz     short loc\_417071

  .text:0041707A                 sub     eax, edx        ; 计算长度

  .text:0041707C                 cmp     eax, 19h        ; 长度=25跳

  .text:0041707F                 jz      short loc\_4170A1

  .text:00417081                 push    offset aBadKeyLength ; "Bad key length"

  .text:00417086                 push    offset dword\_473FA0 ; int

  .text:0041708B                 call    sub\_40C650

  .text:00417090                 mov     edx, [ebp+Memory]

  .text:00417093                 add     esp, 8

  .text:00417096                 push    edx             ; Memory

  .text:00417097                 call    j\_j\_\_free

  .text:0041709C                 add     esp, 4

  .text:0041709F                 jmp     short loc\_417108

  .text:004170A1 ; ---------------------------------------------------------------------------

  .text:004170A1

  .text:004170A1 loc\_4170A1:                             ; CODE XREF: sub\_417000+7Fj

  .text:004170A1                 push    -1

  .text:004170A3                 push    esi

  .text:004170A4                 call    sub\_40C730      ; strcpy()

  .text:004170A9                 mov     esi, eax

  .text:004170AB                 add     esp, 8

  .text:004170AE                 mov     [ebp+arg\_0], esi

  .text:004170B1                 mov     eax, [ebp+Memory]

  .text:004170B4                 push    esi

  .text:004170B5                 push    eax

  .text:004170B6                 mov     byte ptr [ebp+var\_4], 4

  .text:004170BA                 call    sub\_416FB0      ; 注册码处理,字符按位置进行替换,形成一个新的25长字符串

  .text:004170BF                 add     esp, 8

  .text:004170C2                 push    esi             ; 计算后的字符串指针

  .text:004170C3                 mov     ecx, ebx

  .text:004170C5                 call    sub\_416E50      ; 按位计算后组成注册信息

  .text:004170CA                 push    esi

  .text:004170CB                 mov     ecx, ebx

  .text:004170CD                 call    sub\_416D60      ; 前13字符+特征码计算sha1值,经过处理后与后12字符相等

  .text:004170D2                 push    esi             ; Memory

  .text:004170D3                 mov     [ebx+21h], al   ; 返回值放入C[0x21],1为成功,0为失败

  .text:004170D6                 call    j\_j\_\_free

  .text:004170DB                 mov     ecx, [ebp+Memory]

  .text:004170DE                 push    ecx             ; Memory

  .text:004170DF                 call    j\_j\_\_free

  .text:004170E4                 add     esp, 8

  .text:004170E7                 jmp     short loc\_417108

  .text:004170E9 ; ---------------------------------------------------------------------------

  .text:004170E9                 mov     edx, dword ptr [ebp+Args]

  .text:004170EC                 push    edx             ; Args

  .text:004170ED                 push    offset aS       ; "%s"

  .text:004170F2                 push    offset dword\_473FA0 ; int

  .text:004170F7                 call    sub\_4089A0

  .text:004170FC                 add     esp, 0Ch

  .text:004170FF                 mov     eax, offset loc\_417105

  .text:00417104                 retn

  .text:00417105 ; ---------------------------------------------------------------------------

  .text:00417105

  .text:00417105 loc\_417105:                             ; DATA XREF: sub\_417000+FFo

  .text:00417105                 mov     ebx, [ebp+var\_14]

  .text:00417108

  .text:00417108 loc\_417108:                             ; CODE XREF: sub\_417000+9Fj

  .text:00417108                                         ; sub\_417000+E7j

  .text:00417108                 mov     eax, ebx

  .text:0041710A                 mov     ecx, [ebp+var\_C]

  .text:0041710D                 mov     large fs:0, ecx

  .text:00417114                 pop     ecx

  .text:00417115                 pop     edi

  .text:00417116                 pop     esi

  .text:00417117                 pop     ebx

  .text:00417118                 mov     esp, ebp

  .text:0041711A                 pop     ebp

  .text:0041711B                 retn    8

  .text:0041711B sub\_417000      endp
```

这部分的功能在于：

1. 验证注册码长度是否等于25，不等于，报错
2. 验证注册码是否仅包含“2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ”（字串A）中的字符，超出范围，报错
2. 注册码与“2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ”（字串A）进行运算、换位，得出一个新的25字节字符串（B）
2. 利用新的25字节字符串（B）与字串A计算出结构C
3. 利用新的25字符的串与特征码连接，做sha1运算，返回值写了C结构

`00416FB0`处`call`

```asm
  .text:00416FB0 ; =============== S U B R O U T I N E =======================================

  .text:00416FB0

  .text:00416FB0

  .text:00416FB0 sub\_416FB0      proc near               ; CODE XREF: sub\_417000+BAp

  .text:00416FB0                                         ; sub\_417120+A7p

  .text:00416FB0

  .text:00416FB0 arg\_0           = dword ptr  4

  .text:00416FB0 arg\_4           = dword ptr  8

  .text:00416FB0

  .text:00416FB0                 push    ebx

  .text:00416FB1                 mov     ebx, [esp+4+arg\_0]

  .text:00416FB5                 push    esi

  .text:00416FB6                 push    edi

  .text:00416FB7                 mov     edi, [esp+0Ch+arg\_4]

  .text:00416FBB                 xor     esi, esi        ; n=0

  .text:00416FBD                 lea     ecx, [ecx+0]

  .text:00416FC0

  .text:00416FC0 loc\_416FC0:                             ; CODE XREF: sub\_416FB0+32j

  .text:00416FC0                 movzx   edx, byte ptr [ebx+esi\*2+1] ; 注册码2n+1位

  .text:00416FC5                 lea     eax, [esi+edi+0Eh] ; 指向注册码第n+14位

  .text:00416FC9                 push    eax

  .text:00416FCA                 movzx   eax, byte ptr [ebx+esi\*2] ; 注册码2n位

  .text:00416FCE                 lea     ecx, [esi+edi]  ; 批向注册码第n位

  .text:00416FD1                 push    ecx

  .text:00416FD2                 push    edx

  .text:00416FD3                 push    eax

  .text:00416FD4                 call    sub\_4168B0      ; 处理

  .text:00416FD9                 add     esi, 1

  .text:00416FDC                 add     esp, 10h

  .text:00416FDF                 cmp     esi, 0Bh        ; n<11

  .text:00416FE2                 jl      short loc\_416FC0 ; 注册码2n+1位

  .text:00416FE4                 mov     cx, [ebx+16h]   ; 取注册码第22-23位

  .text:00416FE8                 mov     [edi+0Bh], cx   ; 替换计算后的第11-12位

  .text:00416FEC                 mov     dl, [ebx+18h]   ; 取注册码第24位

  .text:00416FEF                 mov     [edi+0Dh], dl   ; 替换计算后的第13位

  .text:00416FF2                 pop     edi

  .text:00416FF3                 pop     esi

  .text:00416FF4                 pop     ebx

  .text:00416FF5                 retn

  .text:00416FF5 sub\_416FB0      endp
```

`004168B0`处的`call`

```asm
  .text:004168B0 ; =============== S U B R O U T I N E =======================================

  .text:004168B0

  .text:004168B0 ; 转换字符串

  .text:004168B0 ; (arg\_0&0x55|arg\_4&0xAA)->arg\_8

  .text:004168B0 ; (arg\_0&0xAA|arg\_4&0x55)->arg\_c
```
