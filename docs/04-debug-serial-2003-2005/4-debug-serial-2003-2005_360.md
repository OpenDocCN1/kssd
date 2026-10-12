
# 粉色情人 V3.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6262.htm`](https://www.pediy.com/kssd/pediy06/pediy6262.htm)

VC++6.0,无壳

反汇编,字符串参考

使用 RSA 算法!!输入的注册码做 RSA 运算,得到的结果与序列号相等则注册成功。

```
:00419C00 6AFF                    push FFFFFFFFF
:00419C02 68DC934400             push 004493DC
:00419C07 64A100000000           mov eax, dword ptr fs:[00000000]
:00419C0D 50                    push eax
:00419C0E 64892500000000        mov dword ptr fs:[00000000], esp
:00419C15 81EC5C030000          sub esp, 0000035C
:00419C1B 53                    push ebx
:00419C1C 55                    push ebp
:00419C1D 56                    push esi
:00419C1E 57                    push edi
:00419C1F 8BF1                  mov esi, ecx
:00419C21 6A01                  push 00000001

* Reference To: MFC42.Ordinal:18BE, Ord:18BEh
                              |
:00419C23 E820790100           Call 00431548
:00419C28 8D4C2418             lea ecx, dword ptr [esp+18]
:00419C2C E88FBEFEFF          call 00405AC0
:00419C31 51                   push ecx
:00419C32 8D7E70               lea edi, dword ptr [esi+70]--------------------->输入的注册码的地址
:00419C35 8BCC                 mov ecx, esp
:00419C37 89642418             mov dword ptr [esp+18], esp
:00419C3B 57                   push edi
:00419C3C C784247C03000000000000 mov dword ptr [esp+0000037C], 00000000

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                              |
:00419C47 E848780100          Call 00431494----------------------------------->取输入的注册码
:00419C4C 8D442418             lea eax, dword ptr [esp+18]--------------------->输入的注册码地址
:00419C50 8D4C241C             lea ecx, dword ptr [esp+1C]
:00419C54 50                   push eax
:00419C55 E886C1FEFF          call 00405DE0----------------------------------->这个是关键!!
:00419C5A 8BE8                 mov ebp, eax\------------------------------------>生成的新字符长串的地址的地址
:00419C5C 8D4C2410             lea ecx, dword ptr [esp+10]
:00419C60 C684247403000001      mov byte ptr [esp+00000374], 01
:00419C68 51                   push ecx
:00419C69 8D4C241C             lea ecx, dword ptr [esp+1C]
:00419C6D E88EBFFEFF          call 00405C00----------------------------------->取序列号 348297013
:00419C72 8B6D00               mov ebp, dword ptr [ebp+00]--------------------->新的长串字符
:00419C75 8B00                 mov eax, dword ptr [eax]------------------------>序列号
:00419C77 55                   push ebp\---------------------------------------->新的字符串
:00419C78 50                   push eax\---------------------------------------->序列号

* Reference To: MSVCRT._mbscmp, Ord:0159h----------------------------------------------->比较
                              |
:00419C79 FF15ECC94400         Call dword ptr [0044C9EC]
:00419C7F 83C408               add esp, 00000008
:00419C82 8D4C2410             lea ecx, dword ptr [esp+10]
:00419C86 85C0                 test eax, eax
:00419C88 0F94C3               sete bl

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                              |
:00419C8B E886770100          Call 00431416------------------------------------------>???
:00419C90 8D4C2414             lea ecx, dword ptr [esp+14]---------------------------->新的数字串的地址的地址
:00419C94 C684247403000000      mov byte ptr [esp+00000374], 00

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                              |
:00419C9C E875770100          Call 00431416------------------------------------------>???
:00419CA1 84DB                 test bl, bl\-------------------------------------------->BL 作为标志
:00419CA3 0F846F010000         je 00419E18-------------------------------------------->跳则出错!!!!!!!!!
:00419CA9 8D542410             lea edx, dword ptr [esp+10]
:00419CAD 52                   push edx

* Possible StringData Ref from Data Obj ->"SOFTWAREMicrosoftWindowsCurrentVersion"
                              |
:00419CAE 6860D04500           push 0045D060
:00419CB3 6802000080           push 80000002

* Reference To: ADVAPI32.RegOpenKeyA, Ord:0171h
                              |
:00419CB8 FF1504C04400         Call dword ptr [0044C004]
:00419CBE 85C0                 test eax, eax
:00419CC0 0F8588000000         jne 00419D4E------------------------------------------>跳到打开注册表失败提示
:00419CC6 8B3F                 mov edi, dword ptr [edi]
:00419CC8 8B47F8               mov eax, dword ptr [edi-08]
:00419CCB 50                   push eax
:00419CCC 8B442414             mov eax, dword ptr [esp+14]
:00419CD0 57                   push edi
:00419CD1 6A01                  push 00000001
:00419CD3 6A00                  push 00000000

* Possible StringData Ref from Data Obj ->"RegistInfo"
                              |
:00419CD5 6854D04500           push 0045D054
:00419CDA 50                   push eax

* Reference To: ADVAPI32.RegSetValueExA, Ord:0186h
                              |
:00419CDB FF1500C04400         Call dword ptr [0044C000]
:00419CE1 85C0                 test eax, eax
:00419CE3 7554                 jne 00419D39----------------------------------------->跳到写注册表失败提示
:00419CE5 8B4C2410             mov ecx, dword ptr [esp+10]
:00419CE9 51                   push ecx

* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh
                              |
:00419CEA FF1508C04400         Call dword ptr [0044C008]
:00419CF0 A1803C4600           mov eax, dword ptr [00463C80]------------------------>标志
:00419CF5 85C0                 test eax, eax

* Possible StringData Ref from Data Obj ->"注册成功"
                              |
:00419CF7 B894E64500           mov eax, 0045E694
:00419CFC 7505                 jne 00419D03

* Possible StringData Ref from Data Obj ->"Register Success!"
                              |
:00419CFE B880E64500           mov eax, 0045E680

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419CFC(C)
|
:00419D03 6A00                  push 00000000
:00419D05 6A00                  push 00000000
:00419D07 50                   push eax
:00419D08 8BCE                 mov ecx, esi

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                              |
:00419D0A E84D7C0100          Call 0043195C
:00419D0F C705883C460001000000 mov dword ptr [00463C88], 00000001

* Reference To: MFC42.Ordinal:0490, Ord:0490h
                              |
:00419D19 E8AC770100          Call 004314CA
:00419D1E 8B4004               mov eax, dword ptr [eax+04]
:00419D21 8B4820               mov ecx, dword ptr [eax+20]
:00419D24 E8873EFFFF          call 0040DBB0
:00419D29 8B5020               mov edx, dword ptr [eax+20]
:00419D2C 6A01                  push 00000001
:00419D2E 6A00                  push 00000000
:00419D30 52                   push edx

* Reference To: USER32.InvalidateRect, Ord:017Ah
                              |
:00419D31 FF1590CA4400         Call dword ptr [0044CA90]
:00419D37 EB34                 jmp 00419D6D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419CE3(C)
|
:00419D39 A1803C4600           mov eax, dword ptr [00463C80]
:00419D3E 85C0                 test eax, eax

* Possible StringData Ref from Data Obj ->"写注册表失败"
                              |
```



```asm
:00419D40 B870E64500             mov eax, 0045E670
:00419D45 751A                  jne 00419D61

* Possible StringData Ref from Data Obj ->"Write Registry Failure!"
                    |
:00419D47 B858E64500             mov eax, 0045E658
:00419D4C EB13                  jmp 00419D61

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419CC0(C)-->设置注册表时的提示
|
:00419D4E A1803C4600             mov eax, dword ptr [00463C80]
:00419D53 85C0                  test eax, eax

* Possible StringData Ref from Data Obj ->"打开注册表失败"
                    |
:00419D55 B848E64500             mov eax, 0045E648
:00419D5A 7505                  jne 00419D61

* Possible StringData Ref from Data Obj ->"Open Registry Failure!"
                    |
:00419D5C B830E64500             mov eax, 0045E630

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00419D45(C), :00419D4C(U), :00419D5A(C)
|
:00419D61 6A00                  push 00000000
:00419D63 6A00                  push 00000000
:00419D65 50                    push eax
:00419D66 8BCE                  mov ecx, esi

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                    |
:00419D68 E8EF7B0100             Call 0043195C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419D37(U)
|
:00419D6D 8BCE                  mov ecx, esi

* Reference To: MFC42.Ordinal:12F5, Ord:12F5h
                    |
:00419D6F E840750100             Call 004312B4
:00419D74 8D8C24C0020000         lea ecx, dword ptr [esp+000002C0]
:00419D7B C784247403000013000000 mov dword ptr [esp+00000374], 13
:00419D86 E875DFFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419D8B 8D8C241C020000         lea ecx, dword ptr [esp+0000021C]
:00419D92 C684247403000012        mov byte ptr [esp+00000374], 12
:00419D9A E861DFFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419D9F 8D8C2478010000         lea ecx, dword ptr [esp+00000178]
:00419DA6 C684247403000011        mov byte ptr [esp+00000374], 11
:00419DAE E4DDFFFFF              call 00417D00--------------------------------->直接返回,没有任何操作
:00419DB3 8D8C24D4000000         lea ecx, dword ptr [esp+000000D4]
:00419DBA C684247403000010        mov byte ptr [esp+00000374], 10
:00419DC2 E839DFFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419DC7 8D4C2430              lea ecx, dword ptr [esp+30]
:00419DCB C68424740300000F        mov byte ptr [esp+00000374], 0F
:00419DD3 E828DFFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419DD8 8D4C2428              lea ecx, dword ptr [esp+28]
:00419DDC C68424740300000E        mov byte ptr [esp+00000374], 0E

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419DE4 E82D760100             Call 00431416
:00419DE9 8D4C2424              lea ecx, dword ptr [esp+24]
:00419DED C68424740300000D        mov byte ptr [esp+00000374], 0D

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419DF5 E81C760100             Call 00431416
:00419DFA 8D4C2420              lea ecx, dword ptr [esp+20]
:00419DFE C68424740300000C        mov byte ptr [esp+00000374], 0C

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419E06 E80B760100             Call 00431416
:00419E0B C68424740300000B        mov byte ptr [esp+00000374], 0B
:00419E13 E9BE000000             jmp 00419ED6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419CA3(C)
|
:00419E18 A1803C4600             mov eax, dword ptr [00463C80]
:00419E1D 85C0                  test eax, eax

* Possible StringData Ref from Data Obj ->"注册码无效"--------------------------->
                    |
:00419E1F B824E64500             mov eax, 0045E624
:00419E24 7505                  jne 00419E2B

* Possible StringData Ref from Data Obj ->"Invalid Register Code!"
                    |
:00419E26 B80CE64500             mov eax, 0045E60C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419E24(C)
|
:00419E2B 6A00                  push 00000000
:00419E2D 6A00                  push 00000000
:00419E2F 50                    push eax
:00419E30 8BCE                  mov ecx, esi

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                    |
:00419E32 E8257B0100             Call 0043195C
:00419E37 8D8C24C0020000         lea ecx, dword ptr [esp+000002C0]
:00419E3E C78424740300000A000000 mov dword ptr [esp+00000374], 0000000A
:00419E49 E8B2DEFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419E4E 8D8C241C020000         lea ecx, dword ptr [esp+0000021C]
:00419E55 C684247403000009        mov byte ptr [esp+00000374], 09
:00419E5D E89EDEFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419E62 8D8C2478010000         lea ecx, dword ptr [esp+00000178]
:00419E69 C684247403000008        mov byte ptr [esp+00000374], 08
:00419E71 E88ADEFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419E76 8D8C24D4000000         lea ecx, dword ptr [esp+000000D4]
:00419E7D C684247403000007        mov byte ptr [esp+00000374], 07
:00419E85 E876DEFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419E8A 8D4C2430              lea ecx, dword ptr [esp+30]
:00419E8E C684247403000006        mov byte ptr [esp+00000374], 06
:00419E96 E865DEFFFF             call 00417D00--------------------------------->直接返回,没有任何操作
:00419E9B 8D4C2428              lea ecx, dword ptr [esp+28]
:00419E9F C684247403000005        mov byte ptr [esp+00000374], 05

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419EA7 E86A750100             Call 00431416
:00419EAC 8D4C2424              lea ecx, dword ptr [esp+24]
:00419EB0 C684247403000004        mov byte ptr [esp+00000374], 04

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419EB8 E859750100             Call 00431416
:00419EBD 8D4C2420              lea ecx, dword ptr [esp+20]
:00419EC1 C684247403000003        mov byte ptr [esp+00000374], 03

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419EC9 E848750100             Call 00431416
:00419ECE C684247403000002        mov byte ptr [esp+00000374], 02

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419E13(U)
|
:00419ED6 8D4C241C              lea ecx, dword ptr [esp+1C]

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419EDA E837750100             Call 00431416
:00419EDF 8D4C2418              lea ecx, dword ptr [esp+18]
:00419EE3 C7842474030000FFFFFFFF mov dword ptr [esp+00000374], FFFFFFFF

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                    |
:00419EEE E823750100             Call 00431416
:00419EF3 8B8C246C030000         mov ecx, dword ptr [esp+0000036C]
:00419EFA 5F                    pop edi
:00419EFB 5E                    pop esi
:00419EFC 5D                    pop ebp
```



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



:00403F8D `50` push eax   

:00403F8E `64892500000000` mov dword ptr `fs:[00000000]`, esp   

:00403F95 `81EC50010000` sub esp, `00000150`   

:00403F9B `56` push esi   

:00403F9C `8BF1` mov esi, ecx   

:00403F9E `833E01` cmp dword ptr `[esi]`, `00000001`   

:00403FA1 `751D` jne `00403FC0`   

:00403FA3 `8B4604` mov eax, dword ptr `[esi+04]`   

:00403FA6 `85C0` test eax, eax   

:00403FA8 `7516` jne `00403FC0`   

:00403FAA `8B8C2464010000` mov ecx, dword ptr `[esp+00000164]`   

* Possible `StringData` Ref from `Data` Obj ->`"0"`   

                         |   

:00403FB1 `682CD04500` push `0045D02C`   

* Reference To: `MFC42.Ordinal:035C`, `Ord:035Ch`   

                         |   

:00403FB6 `E8CDD40200` Call `00431488`   

:00403FBB `E9D0000000` jmp `00404090`   

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Addresses:   

|:00403FA1(C), :00403FA8(C)   

|   

:00403FC0 `57` push edi   

:00403FC1 `8BBC2468010000` mov edi, dword ptr `[esp+00000168]`   

:00403FC8 `68A83B4600` push `00463BA8`   

:00403FCD `8BCF` mov ecx, edi   

* Reference To: `MFC42.Ordinal:035C`, `Ord:035Ch`   

                         |   

:00403FCF `E8B4D40200` Call `00431488`   

* Possible `StringData` Ref from `Data` Obj ->`"0123456789ABCDEF"`   

                         |   

:00403FD4 `6834D04500` push `0045D034`   

:00403FD9 `8D4C240C` lea ecx, dword ptr `[esp+0C]`   

* Reference To: `MFC42.Ordinal:0219`, `Ord:0219h`   

                         |   

:00403FDD `E8A0D40200` Call `00431482`   

:00403FE2 `8D4C2410` lea ecx, dword ptr `[esp+10]`   

:00403FE6 `C784246001000000000000` mov dword ptr `[esp+00000160]`, `00000000`   

:00403FF1 `E8EAF3FFFF` call `004033E0`   

:00403FF6 `56` push esi   

:00403FF7 `8D4C2414` lea ecx, dword ptr `[esp+14]`   

:00403FFB `C684246401000001` mov byte ptr `[esp+00000164]`, `01`   

:00404003 `E848F4FFFF` call `00403450`   

:00404008 `8B442410` mov eax, dword ptr `[esp+10]`   

:0040400C `8B4C8410` mov ecx, dword ptr `[esp+4*eax+10]`   

:00404010 `85C9` test ecx, ecx   

:00404012 `765E` jbe `00404072`   

:00404014 `8BB4246C010000` mov esi, dword ptr `[esp+0000016C]`   

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:   

|:00404070(C)   

|   

:0040401B `56` push esi   

:0040401C `8D4C2414` lea ecx, dword ptr `[esp+14]`   

:00404020 `E82BFEFFFF` call `00403E50`   

:00404025 `8B4C2408` mov ecx, dword ptr `[esp+08]`   

:00404029 `8A1408` mov dl, byte ptr `[eax+ecx]`   

:0040402C `8BCF` mov ecx, edi   

:0040402E `8854240C` mov byte ptr `[esp+0C]`, dl   

:00404032 `8B44240C` mov eax, dword ptr `[esp+0C]`   

:00404036 `50` push eax   

:00404037 `6A00` push `00000000`   

* Reference To: `MFC42.Ordinal:1A7A`, `Ord:1A7Ah`   

                         |   

:00404039 `E83ED40200` Call `0043147C`   

:0040403E `8D8C24B4000000` lea ecx, dword ptr `[esp+000000B4]`   

:00404045 `56` push esi   

:00404046 `51` push ecx   

:00404047 `8D4C2418` lea ecx, dword ptr `[esp+18]`   

:0040404B `E830FBFFFF` call `00403B80`   

:00404050 `50` push eax   

:00404051 `8D4C2414` lea ecx, dword ptr `[esp+14]`   

:00404055 `E8F6F3FFFF` call `00403450`   

:0040405A `8D8C24B4000000` lea ecx, dword ptr `[esp+000000B4]`   

:00404061 `E89A3C0100` call `00417D00`   

:00404066 `8B542410` mov edx, dword ptr `[esp+10]`   

:0040406A `8B449410` mov eax, dword ptr `[esp+4*edx+10]`   

:0040406E `85C0` test eax, eax   

:00404070 `77A9` ja `0040401B`   

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:   

|:00404012(C)   

|   

:00404072 `8D4C2410` lea ecx, dword ptr `[esp+10]`   

:00404076 `E8853C0100` call `00417D00`   

:0040407B `8D4C2408` lea ecx, dword ptr `[esp+08]`   

:0040407F `C7842460010000FFFFFFFF` mov dword ptr `[esp+00000160]`, `FFFFFFFF`   

* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`   

                         |   

:0040408A `E887D30200` Call `00431416` ->?   

:0040408F `5F` pop edi   

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Address:   

|:00403FBB(U)   

|   

:00404090 `8B8C2454010000` mov ecx, dword ptr `[esp+00000154]`   

:00404097 `5E` pop esi   

:00404098 `64890D00000000` mov dword ptr `fs:[00000000]`, ecx   

:0040409F `81C45C010000` add esp, `0000015C`   

:004040A5 `C20800` ret `0008`   

根据娃娃的文章，先假设上面两处得到 `N` 和 `E`，求 `D`   

`N`:   

`7B2EEC1F7CB07AEB8026B9F83B4470BB71CA19182E7BC3E2E38867EBD0E84FD5108B083C037DCCA4CB7FB1113043EA424C241DD0AEDE517518CC428DFDF1D6A5`   

`E`: `10001(H)`   

最后请教各位高手：`N` 怎么分解啊? 我用 `tE!` 的 `RSA tool 2 V1.7` 分解了一个多小时，最后一个出错提示框就什么都没有了……悲惨……   

`4dd466d0e1e0cc123afbe504da2462ca`
