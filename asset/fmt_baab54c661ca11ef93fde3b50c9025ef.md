
# 汇编代码片段

## 第一段代码

```asm
:00419EFD 64890D00000000           mov dword ptr fs:[00000000], ecx
:00419F04 5B                       pop ebx
:00419F05 81C468030000            add esp, 00000368
:00419F0B C3                       ret
```

`**********************************************************************`

此处的子程序根据输入的序列号作 RSA 运算，产生一个新的字符长串。

感谢娃娃 `[CCG]`，《看雪论坛精华 5》。

`* Referenced by a CALL at Addresses:`

`:00415E36 , :00419C55`

## 第二段代码

```asm
:00405DE0 6AFF                    push FFFFFFFF
:00405DE2 68ED704400              push 004470ED
:00405DE7 64A100000000            mov eax, dword ptr fs:[00000000]
:00405DED 50                      push eax
:00405DEE 64892500000000          mov dword ptr fs:[00000000], esp
:00405DF5 81ECA8000000            sub esp, 000000A8
:00405DFB 56                      push esi
:00405DFC 57                      push edi
:00405DFD 8BF1                    mov esi, ecx
:00405DFF C744240800000000        mov [esp+08], 00000000
:00405E07 8B8424C4000000          mov eax, dword ptr [esp+000000C4]----------->输入的注册码
:00405E0E BF01000000              mov edi, 00000001
:00405E13 68A83B4600              push 00463BA8------------------------------->空
:00405E18 50                      push eax
:00405E19 89BC24C0000000          mov dword ptr [esp+000000C0], edi
```

`* Reference To: MSVCRT._mbscmp, Ord:0159h------------------------------------->测试输入是否为空`

`|`

```asm
:00405E20 FF15ECC94400            Call dword ptr [0044C9EC]
:00405E26 83C408                  add esp, 00000008
:00405E29 85C0                    test eax, eax
:00405E2B 7532                    jne 00405E5F-------------------------------->不为空则跳
:00405E2D 8BB424C0000000          mov esi, dword ptr [esp+000000C0]
:00405E34 68A83B4600              push 00463BA8
:00405E39 8BCE                    mov ecx, esi
```

`* Reference To: MFC42.Ordinal:0219, Ord:0219h`

`|`

```asm
:00405E3B E842B60200              Call 00431482
:00405E40 897C2408                mov dword ptr [esp+08], edi
:00405E44 8D8C24C4000000          lea ecx, dword ptr [esp+000000C4]
:00405E4B C68424B800000000        mov byte ptr [esp+000000B8], 00
```

`* Reference To: MFC42.Ordinal:0320, Ord:0320h`

`|`

```asm
:00405E53 E8BEB50200              Call 00431416
:00405E58 8BC6                    mov eax, esi
:00405E5A E9C2000000              jmp 00405F21------------------------------->为空则直接返回
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:00405E2B(C)`

`|`

```asm
:00405E5F 53                      push ebx
:00405E60 55                      push ebp
:00405E61 8D8C24CC000000          lea ecx, dword ptr [esp+000000CC]----------->注册码地址
:00405E68 8DBEBC000000            lea edi, dword ptr [esi+000000BC]----------->变换后的地址
:00405E6E 6A10                    push 00000010
:00405E70 51                      push ecx------------------------------------>注册码的地址
:00405E71 8BCF                    mov ecx, edi
:00405E73 E838E0FFFF              call 00403EB0------------------------------->把输入的注册码变换(内存中为 14 30 01 21 98)
:00405E78 8D6E08                  lea ebp, dword ptr [esi+08]
```

`* Possible StringData Ref from Data Obj ->"7B2EEC1F7CB07AEB8026B9F83B4470BB71CA19182E7BC3"`

`->"E2E38867EBD0E84FD5108B083C037DCCA4CB7FB1113043"`

`->"EA424C241DD0AEDE517518CC428DFDF1D6A5"-----------------/*N*/`

`|`

```asm
:00405E7B 688CD04500              push 0045D08C------------------------------->上面的字符串
:00405E80 8BCD                    mov ecx, ebp
```

`* Reference To: MFC42.Ordinal:035C, Ord:035Ch`

`|`

```asm
:00405E82 E801B60200              Call 00431488
:00405E87 8D9E60010000            lea ebx, dword ptr [esi+00000160]
:00405E8D 6A10                    push 00000010
:00405E8F 55                      push ebp
:00405E90 8BCB                    mov ecx, ebx
:00405E92 E819E0FFFF              call 00403EB0--------------------------------->上把面的字符串变换
```

`* Possible StringData Ref from Data Obj ->"10001"------------------------------------------------/*E*/`

`|`

```asm
:00405E97 6810D14500              push 0045D110
:00405E9C 8BCE                    mov ecx, esi
```

`* Reference To: MFC42.Ordinal:035C, Ord:035Ch`

`|`

```asm
:00405E9E E8E5B50200              Call 00431488
:00405EA3 8DAE04020000            lea ebp, dword ptr [esi+00000204]
:00405EA9 6A10                    push 00000010
:00405EAB 56                      push esi
:00405EAC 8BCD                    mov ecx, ebp
:00405EAE E8FDDFFFFF              call 00403EB0------------------------------->"10001"变换后保存于内存
:00405EB3 53                      push ebx
:00405EB4 8D542418                lea edx, dword ptr [esp+18]
:00405EB8 55                      push ebp
:00405EB9 52                      push edx
:00405EBA 8BCF                    mov ecx, edi
:00405EBC E82FE9FFFF              call 004047F0------------------------------>??
:00405EC1 8D7E18                  lea edi, dword ptr [esi+18]
:00405EC4 50                      push eax
:00405EC5 8BCF                    mov ecx, edi
:00405EC7 C68424C400000002        mov byte ptr [esp+000000C4], 02
:00405ECF E87CD5FFFF              call 00403450------------------------------>??
:00405ED4 8D4C2414                lea ecx, dword ptr [esp+14]
:00405ED8 C68424C000000001        mov byte ptr [esp+000000C0], 01
:00405EE0 E81B1E0100              call 00417D00------------------------------>??
:00405EE5 83C60C                  add esi, 0000000C
:00405EE8 6A10                    push 00000010
:00405EEA 56                      push esi
:00405EEB 8BCF                    mov ecx, edi
:00405EED E88EE0FFFF              call 00403F80------------------------------->??
:00405EF2 56                      push esi------------------------------------>长串字符
:00405EF3 8BB424CC000000          mov esi, dword ptr [esp+000000CC]----------->输入的注册码的地址的地址
:00405EFA 8BCE                    mov ecx, esi-------------------------------->保存于 ECX
```

`* Reference To: MFC42.Ordinal:0217, Ord:0217h`

`|`

```asm
:00405EFC E893B50200              Call 00431494
:00405F01 8D8C24CC000000          lea ecx, dword ptr [esp+000000CC]
:00405F08 C744241001000000        mov [esp+10], 00000001
:00405F10 C68424C000000000        mov byte ptr [esp+000000C0], 00
```

`* Reference To: MFC42.Ordinal:0320, Ord:0320h`

`|`

```asm
:00405F18 E8F9B40200              Call 00431416-------------------------------->?
:00405F1D 5D                      pop ebp
:00405F1E 8BC6                    mov eax, esi
:00405F20 5B                      pop ebx
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:00405E5A(U)`

`|`

```asm
:00405F21 8B8C24B0000000          mov ecx, dword ptr [esp+000000B0]------------>
:00405F28 5F                      pop edi
:00405F29 5E                      pop esi
:00405F2A 64890D00000000          mov dword ptr fs:[00000000], ecx
:00405F31 81C4B4000000            add esp, 000000B4
:00405F37 C20800                  ret 0008
```

`**********************************************************************`

`* Referenced by a CALL at Address:`

`:00405EED`

`|`

```asm
:00403F80 64A100000000            mov eax, dword ptr fs:[00000000]
:00403F86 6AFF                    push FFFFFFFF
:00403F88 68266F4400              push 00446F26
```
