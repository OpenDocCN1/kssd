
```
00454C71   |.  FF15 44E44500      call dword ptr ds:[<&USER32.GetWindowTextLe>; \GetWindowTextLengthA

00454C77   |.  8D48 01           lea ecx,dword ptr ds:[eax+1]

00454C7A   |.  51               push ecx

00454C7B   |.  8B4D 10          mov ecx,dword ptr ss:[ebp+10]

00454C7E   |.  50               push eax

00454C7F   |.  E8 4DC6FFFF      call CCProxy.004512D1

00454C84   |.  50               push eax                                 ; |Buffer

00454C85   |.  56               push esi                                 ; |hWnd

00454C86   |.  FF15 48E44500    call dword ptr ds:[<&USER32.GetWindowTextA>>; \GetWindowTextA

00454C8C   |.  8B4D 10          mov ecx,dword ptr ss:[ebp+10]    //返回到这里

00454C8F   |.  6A FF            push -1

00454C91   |.  E8 13C6FFFF      call CCProxy.004512A9

00454C96   |.  EB 0B            jmp short CCProxy.00454CA3

00454C98   |>  8B45 10          mov eax,dword ptr ss:[ebp+10]

00454C9B   |.  FF30             push dword ptr ds:[eax]                        ; /Arg2

00454C9D   |.  56               push esi                                 ; |Arg1

00454C9E   |.  E8 86F0FFFF      call CCProxy.00453D29                        ; \CCProxy.00453D29

00454CA3   |>  5F               pop edi

00454CA4   |.  5E               pop esi

00454CA5   |.  5D               pop ebp

00454CA6   \.  C2 0C00          retn 0C
```

跟进 `call CCProxy.004512A9`，计算注册码的长度。

```
004512A9   /$  56               push esi

004512AA   |.  8BF1             mov esi,ecx

004512AC   |.  E8 88FBFFFF      call CCProxy.00450E39

004512B1   |.  8B4424 08        mov eax,dword ptr ss:[esp+8]

004512B5   |.  83F8 FF          cmp eax,-1

004512B8   |.  75 08            jnz short CCProxy.004512C2

004512BA   |.  FF36             push dword ptr ds:[esi]                        ; /String

004512BC   |.  FF15 0CE24500    call dword ptr ds:[<&KERNEL32.lstrlenA>]    ; \lstrlenA

            //计算注册码的长度

004512C2   |>  8B0E             mov ecx,dword ptr ds:[esi]

004512C4   |.  8941 F8          mov dword ptr ds:[ecx-8],eax

004512C7   |.  8B0E             mov ecx,dword ptr ds:[esi]

004512C9   |.  5E               pop esi

004512CA   |.  802401 00        and byte ptr ds:[ecx+eax],0

004512CE   \.  C2 0400          retn 4
```

从地址 `00454CA6` 还会后，来到 `0040b619`，类似的操作，分别处理特征码和用户名。

```
0040B619     .  8D96 20010000    lea edx,dword ptr ds:[esi+120]

0040B61F     .  52               push edx                                 ; /Arg3

0040B620     .  68 2C040000      push 42C                                 ; |Arg2 = 0000042C

0040B625     .  57               push edi                                 ; |Arg1

0040B626     .  E8 2C960400      call CCProxy.00454C57                        ; \CCProxy.00454C57

0040B62B     .  81C6 24010000    add esi,124

0040B631     .  56               push esi                                 ; /Arg3

0040B632     .  68 2D040000      push 42D                                 ; |Arg2 = 0000042D

0040B637     .  57               push edi                                 ; |Arg1

0040B638     .  E8 1A960400      call CCProxy.00454C57                        ; \CCProxy.00454C57

0040B63D     .  5F               pop edi

0040B63E     .  5E               pop esi

0040B63F     .  C2 0400          retn 4
```

从 `0040B63F` 最后返回后到 `0040d633`，原来是将注册码和用户名保存到 `CCProxy.ini` 文件。

```
0040D633     .  8B86 1C010000    mov eax,dword ptr ds:[esi+11C]

0040D639     .  8B3D 88E24500    mov edi,dword ptr ds:[<&KERNEL32.WritePriva>;  kernel32.WritePrivateProfileStringA

0040D63F     .  68 E83B4700      push CCProxy.00473BE8                        ; /FileName = "D:\\CCProxy\\CCProxy.ini"

0040D644     .  50               push eax                                 ; |String

0040D645     .  68 5CF04600      push CCProxy.0046F05C                        ; |Key = "RegCode"

0040D64A     .  68 54EF4600      push CCProxy.0046EF54                        ; |Section = "System"

0040D64F     .  FFD7             call edi                                 ; \WritePrivateProfileStringA

0040D651     .  8B86 24010000    mov eax,dword ptr ds:[esi+124]

0040D657     .  68 E83B4700      push CCProxy.00473BE8                        ; /FileName = "D:\\CCProxy\\CCProxy.ini"

0040D65C     .  50               push eax                                 ; |String

0040D65D     .  68 E4F04600      push CCProxy.0046F0E4                        ; |Key = "UserName"

0040D662     .  68 54EF4600      push CCProxy.0046EF54                        ; |Section = "System"

0040D667     .  FFD7             call edi                                 ; \WritePrivateProfileStringA

0040D669     .  8BCE             mov ecx,esi

0040D66B     .  E8 E62F0400      call CCProxy.00450656

0040D670     .  E8 3BC6FFFF      call CCProxy.00409CB0

0040D675     .  8BCE             mov ecx,esi

0040D677     .  E8 EF2F0400      call CCProxy.0045066B
```

从 `0040D670` 跟进

```
00409CB0   /$  B8 8C210000       mov eax,218C

00409CB5   |.  E8 46110300       call CCProxy.0043AE00

00409CBA   |.  55               push ebp

00409CBB   |.  56               push esi

00409CBC   |.  57               push edi

00409CBD   |.  68 E03A4700       push CCProxy.00473AE0                        ; /pCriticalSection = CCProxy.00473AE0

00409CC2   |.  FF15 78E24500     call dword ptr ds:[<&KERNEL32.EnterCritical>; \EnterCriticalSection

00409CC8   |.  A0 B03A4700       mov al,byte ptr ds:[473AB0]

00409CCD   |.  B9 FF000000       mov ecx,0FF

00409CD2   |.  888424 90050000   mov byte ptr ss:[esp+590],al

00409CD9   |.  33C0               xor eax,eax

00409CDB   |.  8DBC24 91050000   lea edi,dword ptr ss:[esp+591]

00409CE2   |.  8B35 98E24500     mov esi,dword ptr ds:[<&KERNEL32.GetPrivate>;  kernel32.GetPrivateProfileStringA

00409CE8   |.  F3:AB             rep stos dword ptr es:[edi]

00409CEA   |.  68 E83B4700       push CCProxy.00473BE8                        ; /IniFileName = "D:\\CCProxy\\CCProxy.ini"

00409CEF   |.  8D8C24 940D0000   lea ecx,dword ptr ss:[esp+D94]                 ; |

00409CF6   |.  68 00040000       push 400                                 ; |BufSize = 400 (1024.)

00409CFB   |.  51               push ecx                                 ; |ReturnBuffer

00409CFC   |.  66:AB             stos word ptr es:[edi]                   ; |

00409CFE   |.  68 B03A4700       push CCProxy.00473AB0                        ; |Default = ""

00409D03   |.  68 5CF04600       push CCProxy.0046F05C                        ; |Key = "RegCode"

00409D08   |.  68 54EF4600       push CCProxy.0046EF54                        ; |Section = "System"

00409D0D   |.  AA               stos byte ptr es:[edi]                   ; |

00409D0E   |.  FFD6             call esi                                 ; \GetPrivateProfileStringA
                   //取注册码

00409D10   |.  68 E83B4700       push CCProxy.00473BE8                        ; /IniFileName = "D:\\CCProxy\\CCProxy.ini"

00409D15   |.  8D9424 90010000   lea edx,dword ptr ss:[esp+190]                 ; |

00409D1C   |.  68 00040000       push 400                                 ; |BufSize = 400 (1024.)

00409D21   |.  52               push edx                                 ; |ReturnBuffer

00409D22   |.  68 B03A4700       push CCProxy.00473AB0                        ; |Default = ""

00409D27   |.  68 E4F04600       push CCProxy.0046F0E4                        ; |Key = "UserName"

00409D2C   |.  68 54EF4600       push CCProxy.0046EF54                        ; |Section = "System"

00409D31   |.  FFD6             call esi                                 ; \GetPrivateProfileStringA
              //取用户名

00409D33   |.  8D8424 8C010000   lea eax,dword ptr ss:[esp+18C]        //eax 指向用户名
```
