
```asm
.text:006F3B7E                 call    ds:ATL::CStringT<char,StrTraitMFC\_DLL<char,ATL::ChTraitsCRT<char>>>::Compare(char const \*)

.text:006F3B84                 test    eax, eax        ; 和注册码的最后4个字符比较

.text:006F3B86                 jnz     loc\_6F3E52      ; 不等则校验失败

.text:006F3B8C                 mov     [esp+5Ch+var\_34\_sa2], ebp ; 构造一个动态数组

.text:006F3B90                 mov     [esp+5Ch+var\_30\_pStr], edi

.text:006F3B94                 mov     [esp+5Ch+var\_24], edi

.text:006F3B98                 mov     [esp+5Ch+var\_28], edi

.text:006F3B9C                 mov     [esp+5Ch+var\_2C\_Len], edi

.text:006F3BA0                 push    0FFFFFFFFh

.text:006F3BA2                 push    60              ; 数组大小

.text:006F3BA4                 lea     ecx, [esp+64h+var\_34\_sa2]

.text:006F3BA8                 mov     byte ptr [esp+64h+var\_4], 8

.text:006F3BAD                 call    new\_6F3410      ; 分配60个字节空间

.text:006F3BB2                 lea     ecx, [esp+5Ch+var\_4C\_r21] ; r21

.text:006F3BB6                 xor     esi, esi

.text:006F3BB8                 xor     ebx, ebx

.text:006F3BBA                 call    ds:ATL::CSimpleStringT<char,1>::GetLength(void)

.text:006F3BC0                 test    eax, eax

.text:006F3BC2                 jle     loc\_6F3C4E

.text:006F3BC8                 xor     ebp, ebp

.text:006F3BCA                 lea     ebx, [ebx+0]

.text:006F3BD0

.text:006F3BD0 loc\_6F3BD0:

.text:006F3BD0                 push    ebx

.text:006F3BD1                 lea     ecx, [esp+60h+var\_4C\_r21] ; 取r21一个字符

.text:006F3BD5                 call    ds:ATL::CSimpleStringT<char,1>::operator[](int)

.text:006F3BDB                 or      edi, 0FFFFFFFFh ; 没找到则返回-1

.text:006F3BDE                 xor     ecx, ecx

.text:006F3BE0

.text:006F3BE0 loc\_6F3BE0:

.text:006F3BE0                 cmp     al, byte ptr ds:Table1\_865058[ecx] ; "CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8"

.text:006F3BE6                 jz      short loc\_6F3BF0

.text:006F3BE8                 inc     ecx

.text:006F3BE9                 cmp     ecx, 32

.text:006F3BEC                 jl      short loc\_6F3BE0

.text:006F3BEE                 jmp     short loc\_6F3BF2 ; 查表

.text:006F3BF0

.text:006F3BF0 loc\_6F3BF0:

.text:006F3BF0                 mov     edi, ecx        ; 取得在表中的索引

.text:006F3BF2

.text:006F3BF2 loc\_6F3BF2:

.text:006F3BF2                 xor     ecx, ecx

.text:006F3BF4

.text:006F3BF4 loc\_6F3BF4:

.text:006F3BF4                 lea     edx, [ecx+ebp]

.text:006F3BF7                 mov     eax, offset Table2\_864FE0

.text:006F3BFC                 lea     esp, [esp+0]

.text:006F3C00

.text:006F3C00 loc\_6F3C00:

.text:006F3C00                 cmp     edx, [eax]

.text:006F3C02                 jz      short loc\_6F3C34

.text:006F3C04                 add     eax, 4

.text:006F3C07                 cmp     eax, offset Table1\_865058 ; "CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8"

.text:006F3C0C                 jl      short loc\_6F3C00

.text:006F3C0E                 test    esi, esi

.text:006F3C10                 jl      failed\_6F3EA9

.text:006F3C16                 cmp     esi, [esp+5Ch+var\_2C\_Len]

.text:006F3C1A                 jge     failed\_6F3EA9

.text:006F3C20                 mov     edx, 1

.text:006F3C25                 shl     edx, cl

.text:006F3C27                 test    edx, edi

.text:006F3C29                 mov     edx, [esp+5Ch+var\_30\_pStr]

.text:006F3C2D                 setnz   al

.text:006F3C30                 mov     [edx+esi], al

.text:006F3C33                 inc     esi

.text:006F3C34

.text:006F3C34 loc\_6F3C34:

.text:006F3C34                 inc     ecx

.text:006F3C35                 cmp     ecx, 5

.text:006F3C38                 jl      short loc\_6F3BF4

.text:006F3C3A                 lea     ecx, [esp+5Ch+var\_4C\_r21]

.text:006F3C3E                 inc     ebx

.text:006F3C3F                 add     ebp, 5

.text:006F3C42                 call    ds:ATL::CSimpleStringT<char,1>::GetLength(void)

.text:006F3C48                 cmp     ebx, eax

.text:006F3C4A                 jl      short loc\_6F3BD0

.text:006F3C4C                 xor     edi, edi

.text:006F3C4E

.text:006F3C4E loc\_6F3C4E:

.text:006F3C4E                 mov     esi, [esp+5Ch+var\_30\_pStr]

.text:006F3C52                 mov     ebx, [esp+5Ch+var\_2C\_Len]

.text:006F3C56                 xor     edx, edx

.text:006F3C58                 mov     eax, 21

.text:006F3C5D                 lea     ecx, [ecx+0]

.text:006F3C60

.text:006F3C60 loc\_6F3C60:

.text:006F3C60                 cmp     eax, edi

.text:006F3C62                 jl      failed\_6F3EA9

.text:006F3C68                 cmp     eax, ebx

.text:006F3C6A                 jge     failed\_6F3EA9

.text:006F3C70                 cmp     byte ptr [esi+eax], 0

.text:006F3C74                 jz      short loc\_6F3C82

.text:006F3C76                 lea     ecx, [eax-21]

.text:006F3C79                 mov     ebp, 1

.text:006F3C7E                 shl     ebp, cl

.text:006F3C80                 or      edx, ebp

.text:006F3C82

.text:006F3C82 loc\_6F3C82:

.text:006F3C82                 inc     eax

.text:006F3C83                 cmp     eax, 26

.text:006F3C86                 jle     short loc\_6F3C60 ; 21-26,必须为0

.text:006F3C88                 cmp     edx, edi

.text:006F3C8A                 mov     eax, [esp+5Ch+var\_40]

.text:006F3C8E                 mov     [eax], edx      ; 保存结果

.text:006F3C90                 jnz     loc\_6F3E40

.text:006F3C96                 xor     eax, eax

.text:006F3C98                 xor     ecx, ecx

.text:006F3C9A                 lea     ebx, [ebx+0]

.text:006F3CA0

.text:006F3CA0 loc\_6F3CA0:

.text:006F3CA0                 cmp     ecx, edi

.text:006F3CA2                 jl      failed\_6F3EA9

.text:006F3CA8                 cmp     ecx, ebx

.text:006F3CAA                 jge     failed\_6F3EA9

.text:006F3CB0                 cmp     byte ptr [esi+ecx], 0

.text:006F3CB4                 jz      short loc\_6F3CBF

.text:006F3CB6                 mov     edx, 1

.text:006F3CBB                 shl     edx, cl

.text:006F3CBD                 or      eax, edx

.text:006F3CBF

.text:006F3CBF loc\_6F3CBF:

.text:006F3CBF                 inc     ecx

.text:006F3CC0                 cmp     ecx, 4          ; 0-4

.text:006F3CC3                 jle     short loc\_6F3CA0

.text:006F3CC5                 mov     ebp, [esp+5Ch+var\_40]

.text:006F3CC9                 mov     [ebp+1Ch], eax  ; 保存结果

.text:006F3CCC                 xor     edi, edi

.text:006F3CCE                 mov     eax, 5

.text:006F3CD3

.text:006F3CD3 loc\_6F3CD3:

.text:006F3CD3                 test    eax, eax

.text:006F3CD5                 jl      failed\_6F3EA9

.text:006F3CDB                 cmp     eax, ebx

.text:006F3CDD                 jge     failed\_6F3EA9

.text:006F3CE3                 cmp     byte ptr [esi+eax], 0

.text:006F3CE7                 jz      short loc\_6F3CF5

.text:006F3CE9                 lea     ecx, [eax-5]

.text:006F3CEC                 mov     edx, 1

.text:006F3CF1                 shl     edx, cl

.text:006F3CF3                 or      edi, edx

.text:006F3CF5

.text:006F3CF5 loc\_6F3CF5:

.text:006F3CF5                 inc     eax

.text:006F3CF6                 cmp     eax, 8          ; 5-8，主版本号

.text:006F3CF9                 jle     short loc\_6F3CD3

.text:006F3CFB                 xor     esi, esi

.text:006F3CFD                 mov     eax, 9

.text:006F3D02

.text:006F3D02 loc\_6F3D02:

.text:006F3D02                 test    eax, eax

.text:006F3D04                 jl      failed\_6F3EA9

.text:006F3D0A                 cmp     eax, ebx

.text:006F3D0C                 jge     failed\_6F3EA9

.text:006F3D12                 mov     ecx, [esp+5Ch+var\_30\_pStr]

.text:006F3D16                 cmp     byte ptr [ecx+eax], 0

.text:006F3D1A                 jz      short loc\_6F3D28

.text:006F3D1C                 lea     ecx, [eax-9]
```
