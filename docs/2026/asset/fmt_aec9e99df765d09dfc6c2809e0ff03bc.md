
### 004263D0

```
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
004263F5 >|>  8>lea     eax, dword ptr [ebp+8]
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
00426429 >|>  8>lea     eax, dword ptr [ebp+8]
0042642C  |.  E>call    <System::AnsiString::c_str(void)>
00426431  |.  0>movsx   edx, byte ptr [eax+2B]
00426435  |.  8>cmp     edx, 33
00426438  |.  7>je      short <loc_42645D>
```

### 4.另外两处的字符明码比较：

第二位判断：

```
00426472  |.  0FBE>movsx   ecx, byte ptr [ebp-63]
00426476  |.  83F9>cmp     ecx, 32                       ;  第二位为32=2
00426479  |.  0F85>jnz     004265BB
```

第十一位的判断：

```
004265AE  |> \0FBE>movsx   edx, byte ptr [ebp-5A]
004265B2  |.  83FA>cmp     edx, 59
004265B5      74 0>je      short 004265BB
```

5.，推断第1、3-10为任意，可自己指定。

### 循环块（一）根据注册码3-10位和“1z1h+2a0n-0g8y\*9a1n|”的3-10位计算出12-19位：

```
00426377  |.  B>mov     esi, 005237CA                               ;  ASCII "1z1h+2a0n-0g8y\*9a1n|"
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
004263C1 >|>  8>lea     eax, dword ptr [ebp+8]
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
004263F5 >|>  8>lea     eax, dword ptr [ebp+8]
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
00426429 >|>  8>lea     eax, dword ptr [ebp+8]
0042642C  |.  E>call    <System::AnsiString::c_str(void)>
00426431  |.  0>movsx   edx, byte ptr [eax+2B]
00426435  |.  8>cmp     edx, 33
00426438  |.  7>je      short <loc_42645D>
0042643A  |.  3>xor     eax, eax
0042643C  |.  5>push    eax
0042643D  |.  F>dec     dword ptr [ebp-C]
00426440  |.  8>lea     eax, dword ptr [ebp+8]
00426443  |.  B>mov     edx, 2
00426448  |.  E>call    <sub_509664>
0042644D  |.  5>pop     eax
0042644E  |.  8>mov     edx, dword ptr [ebp-28]
00426451  |.  6>mov     dword ptr fs:[0], edx
00426458  |.  E>jmp     <loc_4265DA>
0042645D >|>  8>lea     eax, dword ptr [ebp+8]
00426460  |.  E>call    <System::AnsiString::c_str(void)>
00426465  |.  5>push    eax                                         ; /Arg2
00426466  |.  8>lea     edx, dword ptr [ebp-64]                     ; |
00426469  |.  5>push    edx                                         ; |Arg1
0042646A  |.  E>call    <_strcpy>                                   ; \001.004FE1B4
0042646F  |.  8>add     esp, 8
00426472  |.  0>movsx   ecx, byte ptr [ebp-63]
00426476  |.  8>cmp     ecx, 32
00426479  |.  0>jnz     <loc_4265BB>
0042647F  |.  C>mov     byte ptr [ebp-63], 23
00426483  |.  C>mov     byte ptr [ebp-29], 1
00426487  |.  C>mov     dword ptr [ebp-30], 2
0042648E >|>  8>/mov     eax, dword ptr [ebp-30]
00426491  |.  0>|movsx   edx, byte ptr [ebp+eax-7C]
00426496  |.  8>|mov     ecx, dword ptr [ebp-30]
00426499  |.  0>|movsx   eax, byte ptr [ebp+ecx-65]
0042649E  |.  0>|add     edx, eax
004264A0  |.  8>|mov     ecx, dword ptr [ebp-30]
004264A3  |.  0>|movsx   eax, byte ptr [ebp+ecx-64]
004264A8  |.  3>|xor     edx, eax
004264AA  |.  8>|mov     ecx, dword ptr [ebp-30]
004264AD  |.  0>|movsx   eax, byte ptr [ebp+ecx-7C]
004264B2  |.  3>|xor     edx, eax
004264B4  |.  5>|push    edx                                        ; /Arg1
004264B5  |.  E>|call    <sub_40E1E4>                               ; \001.0040E1E4
004264BA  |.  5>|pop     ecx
004264BB  |.  B>|mov     ecx, 1A
004264C0  |.  9>|cdq
004264C1  |.  F>|idiv    ecx
004264C3  |.  8>|add     edx, 41
004264C6  |.  8>|mov     eax, dword ptr [ebp-30]
004264C9  |.  0>|movsx   ecx, byte ptr [ebp+eax-5B]
004264CE  |.  3>|cmp     edx, ecx
004264D0  |.  7>|je      short <loc_4264D8>
004264D2  |.  C>|mov     byte ptr [ebp-29], 0
004264D6  |.  E>|jmp     short <loc_4264E1>
004264D8 >|>  F>|inc     dword ptr [ebp-30]
004264DB  |.  8>|cmp     dword ptr [ebp-30], 0A
004264DF  |.^ 7>\jl      short <loc_42648E>
```

### 6. 循环块（二）

```
004264FE >|> /8>/mov     edx, dword ptr [ebp-34]
00426501  |. |0>|movsx   eax, byte ptr [ebp+edx-7B]
00426506  |. |B>|mov     ecx, 6
0042650B  |. |9>|cdq
0042650C  |. |F>|idiv    ecx
0042650E  |. |8>|mov     ecx, edx
00426510  |. |8>|mov     eax, dword ptr [ebp-34]
00426513  |. |0>|movsx   edx, byte ptr [ebp+eax-7A]
00426518  |. |D>|shl     edx, cl
0042651A  |. |8>|mov     eax, dword ptr [ebp-34]
0042651D  |. |0>|movsx   ecx, byte ptr [ebp+eax-79]
00426522  |. |0>|or      edx, ecx
00426524  |. |5>|push    edx                                        ; /Arg1
00426525  |. |E>|call    <sub_40E1E4>                               ; \001.0040E1E4
0042652A  |. |5>|pop     ecx
0042652B  |. |B>|mov     ecx, 1A
00426530  |. |9>|cdq
00426531  |. |F>|idiv    ecx
00426533  |. |8>|add     dl, 61
00426536  |. |8>|mov     eax, dword ptr [ebp-34]
00426539  |. |8>|mov     byte ptr [ebp+eax-A8], dl
00426540  |. |F>|inc     dword ptr [ebp-34]
00426543  |. |8>|cmp     dword ptr [ebp-34], 28
00426547  |.^\7>\jl      short <loc_4264FE>
```

### 循环块（三）

```
00426564 >|> /8>/mov     edx, dword ptr [ebp-38]
00426567  |. |0>|movsx   eax, byte ptr [ebp+edx-A8]
0042656F  |. |C>|shl     eax, 4
00426572  |. |8>|mov     edx, dword ptr [ebp-38]
00426575  |. |0>|movsx   ecx, byte ptr [ebp+edx-A7]
0042657D  |. |D>|sar     ecx, 1
0042657F  |. |3>|xor     eax, ecx
00426581  |. |5>|push    eax                                        ; /Arg1
```
