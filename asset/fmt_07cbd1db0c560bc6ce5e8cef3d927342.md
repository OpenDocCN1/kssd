
# 逆向分析与代码混淆分析

## 汇编代码段 00410B8C

```asm
:00410B8C B907030000           mov ecx, 00000307

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410B9E(C)
|
:00410B91 0FBE90B4454700      movsx edx, byte ptr [eax+g_szUserPro]
:00410B98 2BCA               sub ecx, edx
:00410B9A 40                 inc eax
:00410B9B 83F810             cmp eax, 00000010
:00410B9E 7CF1               jl 00410B91
:00410BA0 A0DD454700        mov al, byte ptr [004745DD]
:00410BA5 83E11F             and ecx, 0000001F
```

对应的伪代码逻辑：
```c
dwSum = 0x307;
for (i = 0; i < 0x10; i++)
    dwSum -= szUserPro[i];
```

## 汇编代码段 0041331A

```asm
:0041331A B986000000         mov ecx, 00000086
:0041331F 33C0               xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00413339(C)
|
:00413321 0FBE90B4454700     movsx edx, byte ptr [eax+g_szUserPro]
:00413328 A801               test al, 01
:0041332A 7404               je 00413330
:0041332C 03CA               add ecx, edx
:0041332E EB05               jmp 00413335

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041332A(C)
|
:00413330 F7DA               neg edx
:00413332 8D0C51             lea ecx, dword ptr [ecx+2*edx]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041332E(U)
|
:00413335 40                 inc eax
:00413336 83F810             cmp eax, 00000010
:00413339 7CE6               jl 00413321
:0041333B A0DE454700        mov al, byte ptr [004745DE]
:00413340 83E11F             and ecx, 0000001F
```

## 汇编代码段 0040D75E

```asm
:0040D75E BA07000000         mov edx, 00000007
:0040D763 33C0               xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D785(C)
L_Label1:
:0040D765 0FBE88B4454700     movsx ecx, byte ptr [eax+g_szUserPro]
:0040D76C 83F808             cmp eax, 00000008
:0040D76F 7D04               jge L_Label2
:0040D771 8BF1               mov esi, ecx
:0040D773 EB07               jmp L_Label3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D76F(C)
L_Label2:
:0040D775 BE01000000         mov esi, 00000001
:0040D77A 2BF1               sub esi, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D773(U)
L_Label3:
:0040D77C 0FAFF1             imul esi, ecx
:0040D77F 03D6               add edx, esi
:0040D781 40                 inc eax
:0040D782 83F810             cmp eax, 00000010
:0040D785 7CDE               jl L_Label1
:0040D787 A0DF454700        mov al, byte ptr [004745DF]
:0040D78C 83E21F             and edx, 0000001F
```

## 汇编代码段 0040CFF4

```asm
:0040CFF4 B98C030000         mov ecx, 0000038C
:0040CFF9 33C0               xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D020(C)
L_Label7:
:0040CFFB 0FBE90B4454700     movsx edx, byte ptr [eax+g_szUserPro]
:0040D002 83F809             cmp eax, 00000009
:0040D005 7D08               jge L_Label8
:0040D007 8D1452             lea edx, dword ptr [edx+2*edx]
:0040D00A 8D0C51             lea ecx, dword ptr [ecx+2*edx]
:0040D00D EB0D               jmp L_Label9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D005(C)
L_Label8:
:0040D00F 8D349500000000     lea esi, dword ptr [4*edx+00000000]
:0040D016 2BF2               sub esi, edx
:0040D018 F7DE               neg esi
:0040D01A 03CE               add ecx, esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D00D(U)
L_Label9:
:0040D01C 40                 inc eax
:0040D01D 83F810             cmp eax, 00000010
:0040D020 7CD9               jl L_Label7
:0040D022 A0E0454700        mov al, byte ptr [004745E0]
:0040D027 83E11F             and ecx, 0000001F
```

## C 语言代码实现

```c
// 有的地方很难转成 C 代码，可能本来就是 ASM 写的
// 所以直接嵌入 ASM
BOOL GenSN(LPCSTR szUser, LPSTR lpSN)
{
    if (strlen(szUser) <= 0)
        return FALSE;

    const char* puc = szUser;
    while (*puc != '\0')
    {
        if (*puc == '\x14' || (*puc & 0x80) != 0)
            return FALSE;
        puc++;
    }

    char szProTable[] = "23456789ABCDEFGHJKLMNPQRSTUVWXYZ";
    char szUserPro[0x100];
    char szSNPro[0x15];

    int i = 0;
    puc = szUser;
    char* ppuc = szUserPro;
    int nCount = 0;

    // 产生处理后用户名数据
    ZeroMemory(szUserPro, sizeof(szUserPro));
    for (i = 0; i < 0x6f; i++)
    {
        if (*puc == '\0')
        {
            puc = szUser;
            ppuc++;
        }
        if (*puc != '\x20' && (*puc & 0x80) == 0)
        {
            *ppuc = *ppuc + *puc;
            ppuc++;
            nCount++;
            if (nCount == 0x10)
            {
                nCount = 0;
                ppuc = szUserPro;
            }
        }
        else
            i--;
        puc++;
    }

    srand(::GetTickCount());
    for (i = 5; i < 0x11; i++)
    {
        szSNPro[i] = (char)(rand() % 0x20);
    }

    // 0
    DWORD dwSum = 0x4d;
    _asm
    {
        push eax
        push ecx
        push edx
        xor eax, eax
        mov ecx, 0000004dh
    L_Loop1:
        movsx edx, byte ptr [eax + szUserPro]
        inc eax
        cmp eax, 00000010h
        lea ecx, dword ptr [ecx + 2 * edx]
        jl L_Loop1
        and ecx, 0000001fh
        mov dwSum, ecx
        pop edx
        pop ecx
        pop eax
    }
    szSNPro[0x0] = (char)dwSum;

    // 1
    dwSum = 0x307;
    for (i = 0; i < 0x10; i++)
        dwSum -= szUserPro[i];
    szSNPro[0x1] = (char)(dwSum & 0x1f);

    // 2
    _asm
    {
        push eax
        push ecx
        push edx
        mov ecx, 00000086h
        xor eax, eax
    L_Label1:
        movsx edx, byte ptr [eax+szUserPro]
        test al, 01
        je L_Label3
        add ecx, edx
        jmp L_Label2
    L_Label3:
        neg edx
        lea ecx, dword ptr [ecx+2*edx]
    L_Label2:
        inc eax
        cmp eax, 00000010h
        jl L_Label1
        and ecx, 0000001Fh
        mov dwSum, ecx
        pop edx
        pop ecx
        pop eax
    }
    szSNPro[0x2] = (char)dwSum;

    _asm
    {
        push eax
        push ecx
        push edx
        push esi
        mov edx, 00000007h
        xor eax, eax
    L_Label4:
        movsx ecx, byte ptr [eax+szUserPro]
        cmp eax, 00000008h
        jge L_Label5
        mov esi, ecx
        jmp L_Label6
    L_Label5:
        mov esi, 00000001h
        sub esi, ecx
    L_Label6:
        imul esi, ecx
        add edx, esi
        inc eax
        cmp eax, 00000010h
        jl L_Label4
    }
}
```
