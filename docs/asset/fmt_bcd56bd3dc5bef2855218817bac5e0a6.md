
:0041BBEC 2BCA    sub `ecx, edx`<-------------------------`0x73dc15df-0==0x73dc15df`

:0041BBEE 99    `cdq`

:0041BBEF 8BF0    `mov esi, eax`<-------------------------`0x1aa72435`

:0041BBF1 51    `push ecx`

:0041BBF2 8D4B04    `lea ecx, dword ptr [ebx+04]`

:0041BBF5 33F2    `xor esi, edx`

\* Possible StringData Ref from Data Obj ->`"%ld"`

    |

:0041BBF7 68F07B4700    `push 00477BF0`

:0041BBFC 51    `push ecx`

:0041BBFD 2BF2    `sub esi, edx`

:0041BBFF E824D70100    `call 00439328`<------------------------转换成 10 进制,`0x73dc15df==1943803359`

:0041BC04 83C40C    `add esp, 0000000C`

:0041BC07 83C308    `add ebx, 00000008`

:0041BC0A 56    `push esi`

\* Possible StringData Ref from Data Obj ->`"%ld"`

    |

:0041BC0B 68F07B4700    `push 00477BF0`

:0041BC10 53    `push ebx`

:0041BC11 E812D70100    `call 00439328`<------------------------转换成 10 进制,`0x1aa72435==447161397`

 ... ... 略 ... ...

:0041BC82 C21800    `ret 0018`

========================================================

\================================================================================================

\* Referenced by a CALL at Addresses:<----------------------------`call (1.1.1)(2.1.1)(3.1.1.2)`

:0041BB39  , :0041BB53

|

\* Possible Reference to Dialog: `DialogID_01BF`, `CONTROL_ID:00FF`, `""`

    |

:0041BC90 6AFF    `push FFFFFFFF`

:0041BC92 68D8CC4500    `push 0045CCD8`

:0041BC97 64A100000000    `mov eax, dword ptr fs:[00000000]`

:0041BC9D 50    `push eax`

:0041BC9E 64892500000000    `mov dword ptr fs:[00000000], esp`

:0041BCA5 83EC0C    `sub esp, 0000000C`

:0041BCA8 53    `push ebx`

:0041BCA9 56    `push esi`

:0041BCAA A190814700    `mov eax, dword ptr [00478190]`

:0041BCAF 33F6    `xor esi, esi`

:0041BCB1 8974241C    `mov dword ptr [esp+1C], esi`

:0041BCB5 8944240C    `mov dword ptr [esp+0C], eax`

:0041BCB9 C644241C01    `mov [esp+1C], 01`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:0041BD21(C)

|

:0041BCBE 6A01    `push 00000001`

:0041BCC0 8D4C2414    `lea ecx, dword ptr [esp+14]`

:0041BCC4 56    `push esi`

:0041BCC5 51    `push ecx`

:0041BCC6 8D4C2430    `lea ecx, dword ptr [esp+30]`

:0041BCCA E827D10100    `call 00438DF6`

:0041BCCF 8B00    `mov eax, dword ptr [eax]`<---------------注册码第 1 位`'l'==0x6c`

:0041BCD1 8D4C2410    `lea ecx, dword ptr [esp+10]`

:0041BCD5 8A18    `mov bl, byte ptr [eax]`

:0041BCD7 E89B360200    `call 0043F377`

:0041BCDC 8B1590814700    `mov edx, dword ptr [00478190]`

:0041BCE2 89542408    `mov dword ptr [esp+08], edx`

:0041BCE6 0FBEC3    `movsx eax, bl`

:0041BCE9 50    `push eax`

:0041BCEA 8D4C240C    `lea ecx, dword ptr [esp+0C]`

\* Possible StringData Ref from Data Obj ->`"%d"`

    |

:0041BCEE 6820784700    `push 00477820`

:0041BCF3 51    `push ecx`

:0041BCF4 C644242802    `mov [esp+28], 02`

:0041BCF9 E82AD60100    `call 00439328`<------------------------转换成 10 进制,`0x6c==108`

:0041BCFE 83C40C    `add esp, 0000000C`

:0041BD01 8D542408    `lea edx, dword ptr [esp+08]`<----------`108`

:0041BD05 8D4C240C    `lea ecx, dword ptr [esp+0C]`

:0041BD09 52    `push edx`

:0041BD0A E8803A0200    `call 0043F78F`

:0041BD0F 8D4C2408    `lea ecx, dword ptr [esp+08]`

:0041BD13 C644241C01    `mov [esp+1C], 01`

:0041BD18 E85A360200    `call 0043F377`

:0041BD1D 46    `inc esi`

:0041BD1E 83FE04    `cmp esi, 00000004`<-----------------循环计算,并把结果放在一起

:0041BD21 7C9B    `jl 0041BCBE`<-----------------------`1089711099`

:0041BD23 8B44240C    `mov eax, dword ptr [esp+0C]`

:0041BD27 50    `push eax`

:0041BD28 E88CD30000    `call 004290B9`<---------------------转换成 16 进制,`1089711099==0x40f3abfb`

:0041BD2D 83C404    `add esp, 00000004`

:0041BD30 8D4C240C    `lea ecx, dword ptr [esp+0C]`

:0041BD34 8BF0    `mov esi, eax`<-----------------------`0x40f3abfb`

:0041BD36 C644241C00    `mov [esp+1C], 00`

:0041BD3B E837360200    `call 0043F377`

:0041BD40 8D4C2424    `lea ecx, dword ptr [esp+24]`

:0041BD44 C744241CFFFFFFFF    `mov [esp+1C], FFFFFFFF`

:0041BD4C E826360200    `call 0043F377`

:0041BD51 8B4C2414    `mov ecx, dword ptr [esp+14]`

:0041BD55 8BC6    `mov eax, esi`

:0041BD57 5E    `pop esi`

:0041BD58 5B    `pop ebx`

:0041BD59 64890D00000000    `mov dword ptr fs:[00000000], ecx`

:0041BD60 83C418    `add esp, 00000018`

:0041BD63 C20400    `ret 0004`

\================================================================================================

\* Referenced by a CALL at Address:<-------------------------------------`call (3)`

:0041C4EE

|

\* Possible Reference to Dialog: `DialogID_01BF`, `CONTROL_ID:00FF`, `""`

    |

:0041C250 6AFF    `push FFFFFFFF`

 ... ... 略 ... ...

:0041C2D3 57    `push edi`

:0041C2D4 C644243003    `mov [esp+30], 03`

:0041C2D9 E8A2FCFFFF    `call 0041BF80`<------------(3.1)--------又是它,算法同 `call(1)`

:0041C2DE C744241001000000    `mov [esp+10], 00000001`    参数不同,而已

:0041C2E6 8D4C2408    `lea ecx, dword ptr [esp+08]`

:0041C2EA C644242402    `mov [esp+24], 02`

:0041C2EF E883300200    `call 0043F377`

:0041C2F4 8D4C240C    `lea ecx, dword ptr [esp+0C]`

:0041C2F8 C644242401    `mov [esp+24], 01`

:0041C2FD E875300200    `call 0043F377`

:0041C302 8D4C2430    `lea ecx, dword ptr [esp+30]`

:0041C306 C644242400    `mov [esp+24], 00`

:0041C30B E867300200    `call 0043F377`

:0041C310 8B4C241C    `mov ecx, dword ptr [esp+1C]`

:0041C314 8BC7    `mov eax, edi`

:0041C316 5F    `pop edi`

:0041C317 64890D00000000    `mov dword ptr fs:[00000000], ecx`

:0041C31E 5E    `pop esi`

:0041C31F 83C420    `add esp, 00000020`

:0041C322 C20800    `ret 0008`

\====================================================================================================

用户名：`lancelot[CCG]`

注册码：`299343965474195577`

\=================================================================================================

```
,;~;,
    /\\_
   ( /
   (())   //)
   | \\ ,;;'\\
 __ \_( )m=(lancelot(================--------
```
