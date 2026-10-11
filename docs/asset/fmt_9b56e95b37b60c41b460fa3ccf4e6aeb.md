
# 汇编代码片段

```
00426540  |.  F>|inc     dword ptr [ebp-34]  
00426543  |.  8>|cmp     dword ptr [ebp-34], 28  
00426547  |.^ 7>\jl      short <loc_4264FE>  
00426549  |>   C>mov     byte ptr [ebp-80], 5A  
0042654D  |.  C>mov     byte ptr [ebp-7F], 59  
00426551  |.  C>mov     dword ptr [ebp-38], 18  
00426558  |.  6>mov     word ptr [ebp-18], 8  
0042655E  |.  8>cmp     dword ptr [ebp-38], 28  
00426562  |.  7>jge     short <loc_4265AE>  
00426564  |>   8>/mov    edx, dword ptr [ebp-38]  
00426567  |.  0>|movsx  eax, byte ptr [ebp+edx-A8]  
0042656F  |.  C>|shl     eax, 4  
00426572  |.  8>|mov     edx, dword ptr [ebp-38]  
00426575  |.  0>|movsx   ecx, byte ptr [ebp+edx-A7]  
0042657D  |.  D>|sar     ecx, 1  
0042657F  |.  3>|xor     eax, ecx  
00426581  |.  5>|push    eax                     ; /Arg1  
00426582  |.  E>|call    <sub_40E1E4>           ; \001.0040E1E4  
00426587  |.  5>|pop     ecx  
00426588  |.  B>|mov     ecx, 1A  
0042658D  |.  9>|cdq  
0042658E  |.  F>|idiv    ecx  
00426590  |.  8>|add     edx, 41  
00426593  |.  8>|mov     eax, dword ptr [ebp-38]  
00426596  |.  0>|movsx   eax, byte ptr [ebp+eax-64]  
0042659B  |.  3>|cmp     edx, eax  
0042659D  |.  7>|je      short <loc_4265A5>  
0042659F  |.  C>|mov     byte ptr [ebp-29], 0  
004265A3  |.  E>|jmp     short <loc_4265AE>  
004265A5  |>   F>|inc     dword ptr [ebp-38]  
004265A8  |.  8>|cmp     dword ptr [ebp-38], 28  
004265AC  |.^ 7>\jl      short <loc_426564>  
004265AE  |>   0>movsx   edx, byte ptr [ebp-5A]  
004265B2  |.  8>cmp     edx, 59  
004265B5  |.  7>je      short <loc_4265BB>  
004265B7  |.  C>mov     byte ptr [ebp-29], 0  
004265BB  |>   8>mov     al, byte ptr [ebp-29]  
004265BE  |.  5>push    eax  
004265BF  |.  F>dec     dword ptr [ebp-C]  
004265C2  |.  8>lea     eax, dword ptr [ebp+8]  
004265C5  |.  B>mov     edx, 2  
004265CA  |.  E>call    <sub_509664>  
004265CF  |.  5>pop     eax  
004265D0  |.  8>mov     edx, dword ptr [ebp-28]  
004265D3  |.  6>mov     dword ptr fs:[0], edx  
004265DA  |>   5>pop     edi  
004265DB  |.  5>pop     esi  
004265DC  |.  8>mov     esp, ebp  
004265DE  |.  5>pop     ebp  
004265DF  \\.  C>retn  
```

## 第二十章 关键 CALL 的功能模块简要分析

静态汇编后，粗粗看一遍，大致的理解是单线程、顺序判断结构，程序的流程很简单，明码比较和暗码比较相交错：

### 1. 注册码的长度限制条件

```
00426369  |.  E>call    <System::AnsiString::Length(void)>
0042636E  |.  8>cmp     eax, 2C
00426371  |.  0>jnz     <loc_4265BB>
```

### 2. 参与算法分析的关键常量的赋值处

```
00426377  |.  B>mov     esi, 005237CA           ;  ASCII "1z1h+2a0n-0g8y*9a1n|"
```

判断常量的依据：

### 3. 四位相连的明码比较

后来经过动态调试证实是最后四位。

```
00426390  |.  E>call    <System::AnsiString::c_str(void)>
00426395  |.  0>movsx   edx, byte ptr [eax+28]
00426399  |.  8>cmp     edx, 50
0042639C  |.  7>je      short <loc_4263C1>
0042639E  |.  3>xor     eax, eax
004263A0  |.  5>push    eax
004263A1  |.  F>dec     dword ptr [ebp-C]
004263A4  |.  8>lea     eax, dword ptr [ebp+8]
004263A7  |.  B>mov     edx, 2
004263AC  |.  E>call    <sub_509664>
004263B1  |.  5>pop     eax
004263B2  |.  8>mov     edx, dword ptr [ebp-28]
004263B5  |.  6>mov     dword ptr fs:[0], edx
004263BC  |.  E>jmp     <loc_4265DA>

004263C1  |>   8>lea     eax, dword ptr [ebp+8]
004263C4  |.  E>call    <System::AnsiString::c_str(void)>
004263C9  |.  0>movsx   edx, byte ptr [eax+29]
004263CD  |.  8>cmp     edx, 32
004263D0  |.  7>je      short <loc_4263F5>
004263D2  |.  3>xor     eax, eax
004263D4  |.  5>push    eax
004263D5  |.  F>dec     dword ptr [ebp-C]
004263D8  |.  8>lea     eax, dword ptr [ebp+8]
004263DB  |.  B>mov     edx, 2
004263E0  |.  E>call    <sub_509664>
004263E5  |.  5>pop     eax
004263E6  |.  8>mov     edx, dword ptr [ebp-28]
004263E9  |.  6>mov     dword ptr fs:[0], edx
004263F0  |.  E>jmp     <loc_4265DA>

004263F5  |>   8>lea     eax, dword ptr [ebp+8]
004263F8  |.  E>call    <System::AnsiString::c_str(void)>
004263FD  |.  0>movsx   edx, byte ptr [eax+2A]
00426401  |.  8>cmp     edx, 49
00426404  |.  7>je      short <loc_426429>
00426406  |.  3>xor     eax, eax
00426408  |.  5>push    eax
00426409  |.  F>dec     dword ptr [ebp-C]
0042640C  |.  8>lea     eax, dword ptr [ebp+8]
0042640F  |.  B>mov     edx, 2
00426414  |.  E>call    <sub_509664>
00426419  |.  5>pop     eax
0042641A  |.  8>mov     edx, dword ptr [ebp-28]
0042641D  |.  6>mov     dword ptr fs:[0], edx
00426424  |.  E>jmp     <loc_4265DA>

00426429  |>   8>lea     eax, dword ptr [ebp+8]
0042642C  |.  E>call    <System::AnsiString::c_str(void)>
00426431  |.  0>movsx   edx, byte ptr [eax+2B]
00426435  |.  8>cmp     edx, 33
00426438  |.  7>je      short <loc_42645D>
```

### 4. 另外两处的字符明码比较

#### 第二位判断

```
00426472  |.  0FBE>movsx ecx, byte ptr [ebp-63]
00426476  |.  83F9>cmp  ecx, 32                 ;  第二位为 32=2
00426479  |.  0F85>jnz  004265BB
```

#### 第十一位的判断

```
004265AE  |>  \0FBE>movsx edx, byte ptr [ebp-5A]
004265B2  |.  83FA>cmp  edx, 59
004265B5       74 0>je  short 004265BB
```

### 5. 推断第 1、3-10 为任意，可自己指定

循环块（一）根据注册码 3-10 位和 `1z1h+2a0n-0g8y*9a1n|` 的 3-10 位计算出 12-19 位：

```
00426377  |.  B>mov     esi, 005237CA           ;  ASCII "1z1h+2a0n-0g8y*9a1n|"
0042637C  |.  8>lea     edi, dword ptr [ebp-7C]
0042637F  |.  B>mov     ecx, 5
00426384  |.  F>rep     movs dword ptr es:[edi], dword ptr [esi]
00426386  |.  A>movs    byte ptr es:[edi], byte ptr [esi]
00426387  |.  6>mov     word ptr [ebp-18], 8
0042638D  |.  8>lea     eax, dword ptr [ebp+8]
00426390  |.  E>call    <System::AnsiString::c_str(void)>
00426395  |.  0>movsx   edx, byte ptr [eax+28]
00426399  |.  8>cmp     edx, 50
0042639C  |.  7>je      short <loc_4263C1>
0042639E  |.  3>xor     eax, eax
004263A0  |.  5>push    eax
004263A1  |.  F>dec     dword ptr [ebp-C]
004263A4  |.  8>lea     eax, dword ptr [ebp+8]
004263A7  |.  B>mov     edx, 2
004263AC  |.  E>call    <sub_509664>
004263B1  |.  5>pop     eax
004263B2  |.  8>mov     edx, dword ptr [ebp-28]
004263B5  |.  6>mov     dword ptr fs:[0], edx
004263BC  |.  E>jmp     <loc_4265DA>

004263C1  |>   8>lea     eax, dword ptr [ebp+8]
004263C4  |.  E>call    <System::AnsiString::c_str(void)>
004263C9  |.  0>movsx   edx, byte ptr [eax+29]
004263CD  |.  8>cmp     edx, 32
004263D0  |.  7>je      short <loc_4263F5>
004263D2  |.  3>xor     eax, eax
004263D4  |.  5>push    eax
004263D5  |.  F>dec     dword ptr [ebp-C]
004263D8  |.  8>lea     eax, dword ptr [ebp+8]
004263DB  |.  B>mov     edx, 2
004263E0  |.  E>call    <sub_509664>
004263E5  |.  5>pop     eax
004263E6  |.  8>mov     edx, dword ptr [ebp-28]
004263E9  |.  6>mov     dword ptr fs:[0], edx
004263F0  |.  E>jmp     <loc_4265DA>

004263F5  |>   8>lea     eax, dword ptr [ebp+8]
004263F8  |.  E>call    <System::AnsiString::c_str(void)>
004263FD  |.  0>movsx   edx, byte ptr [eax+2A]
00426401  |.  8>cmp     edx, 49
00426404  |.  7>je      short <loc_426429>
00426406  |.  3>xor     eax, eax
00426408  |.  5>push    eax
00426409  |.  F>dec     dword ptr [ebp-C]
0042640C  |.  8>lea     eax, dword ptr [ebp+8]
0042640F  |.  B>mov     edx, 2
00426414  |.  E>call    <sub_509664>
00426419  |.  5>pop     eax
0042641A  |.  8>mov     edx, dword ptr [ebp-28]
0042641D  |.  6>mov     dword ptr fs:[0], edx
00426424  |.  E>jmp     <loc_4265DA>

00426429  |>   8>lea     eax, dword ptr [ebp+8]
0042642C  |.  E>call    <System::AnsiString::c_str(void)>
00426431  |.  0>movsx   edx, byte ptr [eax+2B]
00426435  |.  8>cmp     edx, 33
00426438  |.  7>je      short <loc_42645D>
0042643A  |.  3>xor     eax, eax
```
