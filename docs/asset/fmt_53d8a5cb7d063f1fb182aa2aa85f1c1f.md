
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100108C5(C)
|
:100108B8 0FBE9018F91210          movsx edx, byte ptr [eax+g_strUserPro]
:100108BF 03CA                    add ecx, edx
:100108C1 40                      inc eax
:100108C2 83F810                  cmp eax, 00000010
:100108C5 7CF1                    jl 100108B8
:100108C7 33C0                    xor eax, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100108DD(C)
|
:100108C9 0FBE90F8F81210          movsx edx, byte ptr [eax+g_strSNPro]
:100108D0 0FBE92B0CB0210          movsx edx, byte ptr [edx+g_strProTable]
:100108D7 03CA                    add ecx, edx
:100108D9 40                      inc eax
:100108DA 83F811                  cmp eax, 00000011
:100108DD 7CEA                    jl 100108C9
:100108DF A009F91210              mov al, byte ptr [1012F909]
:100108E4 83E11F                  and ecx, 0000001F
:100108E7 3AC1                    cmp al, cl
:100108E9 7563                    jne 1001094E
```

```
********************************************************************/
dwSum = 0x49;
for(int i = 0; i < 0x10; i++)
  dwSum += g_strUserPro[i];
for(int i = 0; i < 0x11; i++)
{
  dwIndex = g_strSNPro[i];
  dwSum += g_strProTable[dwIndex];
}
dwSum &= 0x1f;
if (g_strSNPro[0x11] == (BYTE)dwSum)
{
```

********************************************************************
:100108EB B932000000              mov ecx, 00000032
:100108F0 33C0                    xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:100108FF(C)
|
:100108F2 0FBE9018F91210          movsx edx, byte ptr [eax+g_strUserPro]
:100108F9 2BCA                    sub ecx, edx
:100108FB 40                      inc eax
:100108FC 83F810                  cmp eax, 00000010
:100108FF 7CF1                    jl 100108F2
:10010901 33C0                    xor eax, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:10010917(C)
|
:10010903 0FBE90F8F81210          movsx edx, byte ptr [eax+g_strSNPro]
:1001090A 0FBE92B0CB0210          movsx edx, byte ptr [edx+g_strProTable]
:10010911 03CA                    add ecx, edx
:10010913 40                      inc eax
:10010914 83F812                  cmp eax, 00000012
:10010917 7CEA                    jl 10010903
:10010919 A00AF91210              mov al, byte ptr [1012F90A]
:1001091E 83E11F                  and ecx, 0000001F
:10010921 3AC1                    cmp al, cl
:10010923 7529                    jne 1001094E
```

********************************************************************
//DWORD dwT2 = 0x32;
dwSum = 0x32;
for (int i = 0; i < 0x10; i++)
  dwSum -= g_strUserPro[i];
for (int i = 0; i < 0x12; i++)
{
  dwIndex = g_strSNPro[i];
  dwSum += g_strProTable[dwIndex];
}
dwSum &= 0x1f;
if (g_strSNPro[0x12] == (BYTE)dwSum) //0x10010921
{

```
********************************************************************
:10010925 B979000000              mov ecx, 00000079
:1001092A 33C0                    xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10010940(C)
|
:1001092C 0FBE90F8F81210          movsx edx, byte ptr [eax+g_strSNPro]
:10010933 0FBE92B0CB0210          movsx edx, byte ptr [edx+g_strProTable]
:1001093A 2BCA                    sub ecx, edx
:1001093C 40                      inc eax
:1001093D 83F813                  cmp eax, 00000013
:10010940 7CEA                    jl 1001092C
```

********************************************************************/
dwSum = 0x79;
for (int i = 0; i < 0x13; i++)
{
  dwIndex = g_strSNPro[i];
  dwSum -= g_strProTable[dwIndex];
}

```
********************************************************************
:10010942 A00BF91210              mov al, byte ptr [1012F90B]
:10010947 83E11F                  and ecx, 0000001F
:1001094A 3AC1                    cmp al, cl
:1001094C 7416                    je 10010964
```

********************************************************************/
dwSum &= 0x1f;
if (g_strSNPro[0x13] == (BYTE)dwSum)
{

```
********************************************************************
:10010964 B801000000              mov eax, 00000001
:10010969 A338F91210              mov dword ptr [1012F938], eax
:1001096E C3                      ret
```

********************************************************************/
//g_bRegistered addr 1012F938
return g_bRegistered = TRUE;
}
else
{

```
********************************************************************
:1001094E 57                      push edi
:1001094F B91A000000              mov ecx, 0000001A
:10010954 33C0                    xor eax, eax
:10010956 BF38A40310              mov edi, g_strUser
:1001095B F3                      repz
:1001095C AB                      stosd
:1001095D A338F91210              mov dword ptr [1012F938], eax
:10010962 5F                      pop edi
:10010963 C3                      ret
```

********************************************************************/
//错误处理
_ZeroMemory(g_strUser, sizeof(g_strUser));
return g_bRegistered = FALSE;
}
}
}
}

////////////////////////////////////////////////////////////////////////////////////////////////////
//////////////////////////////////////
//004745DC 的比较 00410857

```
********************************************************************
:00410857 B94D000000              mov ecx, 0000004D
:0041085C 5E                      pop esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041086B(C)
|
:0041085D 0FBE90B4454700          movsx edx, byte ptr [eax+g_szUserPro]
:00410864 40                      inc eax
:00410865 83F810                  cmp eax, 00000010
:00410868 8D0C51                  lea ecx, dword ptr [ecx+2*edx]
:0041086B 7CF0                    jl 0041085D
:0041086D A0DC454700              mov al, byte ptr [g_szSNPro]
:00410872 83E11F                  and ecx, 0000001F
:00410875 3AC1                    cmp al, cl
:00410877 7408                    je 00410881
```

********************************************************************/
_asm
{
  push eax
  push ecx
  push edx
  xor eax, eax
L_Loop1:
  movsx edx, byte ptr [eax + g_szUserPro]
  inc eax
  cmp eax, 00000010h
  lea ecx, dword ptr [ecx + 2*edx]
  jl L_Loop1;
  and ecx, 0000001fh
  mov dwSum, ecx
}

//////////////////////////////////////
//////////////////////////////////////
004745DD 的比较 00410B8C
