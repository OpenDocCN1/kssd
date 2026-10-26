
```text
.text:004168B0
.text:004168B0 sub_4168B0      proc near               ; CODE XREF: sub_416040+20p
.text:004168B0                                         ; sub_416FB0+24p
.text:004168B0 arg_0           = dword ptr  4
.text:004168B0 arg_4           = dword ptr  8
.text:004168B0 arg_8           = dword ptr  0Ch
.text:004168B0 arg_C           = dword ptr  10h
.text:004168B0                 mov     eax, [esp+arg_0]
.text:004168B4                 push    esi
.text:004168B5                 push    edi
.text:004168B6                 push    eax
.text:004168B7                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:004168BC                 mov     ecx, [esp+0Ch+arg_4] ; 2n+1位
.text:004168C0                 push    ecx
.text:004168C1                 mov     esi, eax
.text:004168C3                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:004168C8                 mov     edx, eax
.text:004168CA                 mov     ecx, esi
.text:004168CC                 and     edx, 0AAh       ; 2n+1位索引取偶数位
.text:004168D2                 and     ecx, 55h        ; 2n位索引取奇数位
.text:004168D5                 or      edx, ecx        ; 合成
.text:004168D7                 add     esp, 8
.text:004168DA                 and     edx, 8000001Fh  ; 取低5位,并校验高位
.text:004168E0                 jns     short loc_4168E7 ; 高位未置跳转
.text:004168E2                 dec     edx
.text:004168E3                 or      edx, 0FFFFFFE0h
.text:004168E6                 inc     edx
.text:004168E7 loc_4168E7:                             ; CODE XREF: sub_4168B0+30j
.text:004168E7                 mov     ecx, off_470DC0
.text:004168ED                 mov     dl, [edx+ecx]   ; 取合成数在字符串A中相应字符
.text:004168F0                 mov     edi, [esp+8+arg_8] ; 替换n处字符
.text:004168F4                 and     esi, 0AAh       ; 2n位索引取偶数位
.text:004168FA                 and     eax, 55h        ; 2n+1位索引取奇数位
.text:004168FD                 or      esi, eax
.text:004168FF                 and     esi, 8000001Fh  ; 合成后取低5位,并检测高位
.text:00416905                 mov     [edi], dl
.text:00416907                 jns     short loc_41690E
.text:00416909                 dec     esi
.text:0041690A                 or      esi, 0FFFFFFE0h
.text:0041690D                 inc     esi
.text:0041690E loc_41690E:                             ; CODE XREF: sub_4168B0+57j
.text:0041690E                 mov     al, [esi+ecx]
.text:00416911                 mov     ecx, [esp+8+arg_C] ; 取合成数在字符串中相应字符替换n+14处字符
.text:00416915                 pop     edi
.text:00416916                 mov     [ecx], al
.text:00416918                 pop     esi
.text:00416919                 retn
.text:00416919 sub_4168B0      endp
```

以上部分是注册码与“2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ”（字符串A）运算出新的25字节长字符串B的过程

00416E50处的Call

```text
.text:00416E50 ; =============== S U B R O U T I N E =======================================
.text:00416E50 ; 按位计算后组成注册信息
.text:00416E50 sub_416E50      proc near               ; CODE XREF: sub_417000+C5p
.text:00416E50                                         ; sub_417120+E1p
.text:00416E50 arg_0           = dword ptr  4
.text:00416E50                 push    ebx
.text:00416E51                 push    ebp
.text:00416E52                 push    esi
.text:00416E53                 mov     esi, [esp+0Ch+arg_0] ; 第一次运算后的字符串
.text:00416E57                 movzx   eax, byte ptr [esi+3] ; 取第3个字符
.text:00416E5B                 push    edi
.text:00416E5C                 push    eax
.text:00416E5D                 mov     edi, ecx        ; 栈指针C
.text:00416E5F                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416E64                 movzx   ecx, byte ptr [esi+2] ; 取第2个字符
.text:00416E68                 mov     ebp, eax
.text:00416E6A                 push    ecx
.text:00416E6B                 shl     ebp, 5          ; 第3字符索引<<5
.text:00416E6E                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416E73                 movzx   edx, byte ptr [esi+1] ; 取第1个字符
.text:00416E77                 mov     ebx, eax
.text:00416E79                 or      ebx, ebp
.text:00416E7B                 push    edx
.text:00416E7C                 shl     ebx, 5
.text:00416E7F                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416E84                 mov     ebp, eax
.text:00416E86                 movzx   eax, byte ptr [esi] ; 取第0个字符
.text:00416E89                 or      ebp, ebx
.text:00416E8B                 push    eax
.text:00416E8C                 shl     ebp, 5
.text:00416E8F                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416E94                 or      eax, ebp        ; 产生dw,从低到高每5位为第0,1,2,3字符在字符串中的索引
.text:00416E96                 mov     [edi+8], eax    ; 放入C[0x8]
.text:00416E99                 movzx   ecx, byte ptr [esi+6] ; 取第6个字符
.text:00416E9D                 push    ecx
.text:00416E9E                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416EA3                 movzx   edx, byte ptr [esi+5] ; 取第5个字符
.text:00416EA7                 mov     ebp, eax
.text:00416EA9                 push    edx
.text:00416EAA                 shl     ebp, 5
.text:00416EAD                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416EB2                 mov     ebx, eax
.text:00416EB4                 movzx   eax, byte ptr [esi+4] ; 取第4个字符
.text:00416EB8                 or      ebx, ebp
.text:00416EBA                 push    eax
.text:00416EBB                 shl     ebx, 5
.text:00416EBE                 call    sub_416840      ; 取在字符串中"2ABC3DEF4GHJ5KLM6NPQ7RST8UVW9XYZ"位置
.text:00416EC3                 or      eax, ebx        ; 产生dw,从低到高每5位为第4,5,6字符在字符串中的索引
.text:00416EC5                 cdq                     ; eax高位值扩展入edx
.text:00416EC6                 mov     [edi+18h], eax  ; 放入C[0x18]
.text:00416EC9                 and     eax, 4000h
.text:00416ECE                 xor     ecx, ecx
.text:00416ED0                 add     esp, 1Ch
.text:00416ED3                 or      eax, ecx        ; 判断eax第14位是否为0
.text:00416ED5                 mov     [edi+1Ch], edx  ; 放入C[0x1C]
.text:00416ED8                 jz      short loc_416EDF
.text:00416EDA                 lea     eax, [ecx+1]    ; eax=1
.text:00416EDD                 jmp     short loc_416EE1 ; al->C[0x20]
.text:00416EDF ; ---------------------------------------------------------------------------
.text:00416EDF loc_416EDF:                             ; CODE XREF: sub_416E50+88j
.text:00416EDF                 xor     eax, eax
.text:00416EE1 loc_416EE1:                             ; CODE XREF: sub_416E50+8Dj
.text:00416EE1                 mov     [edi+20h], al   ; al->C[0x20]
.text:00416EE4                 call    sub_416610      ; 取时间值,设置和判断470DC8和470DCC的值
.text:00416EE9                 mov     ecx, [edi+18h]  ; 取C[0x18]
.text:00416EEC                 push    0
.text:00416EEE                 mov     ebp, edx
.text:00416EF0                 mov     edx, [edi+1Ch]  ; 取C[0x1C]
.text:00416EF3                 push    15180h
.text:00416EF8                 and     ecx, 0FFFFBFFFh ; 取第14位以外各位
.text:00416EFE                 push    edx             ; C[0x1C]
.text:00416EFF                 push    ecx             ; C[0x18]
.text:00416F00                 mov     ebx, eax
```
