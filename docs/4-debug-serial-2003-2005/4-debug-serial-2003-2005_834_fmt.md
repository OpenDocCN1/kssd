
# 编程文档手册 V3.6 简单算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-361.htm`](https://www.pediy.com/kssd/pediy07/pediy7-361.htm)

【破文标题】：编程文档手册 V3.6 简单算法

【破解目标】：编程文档手册 V3.6

【破解作者】：`jney2`

【破解日期】：2005 年 4 月 6 日

【作者声明】：破解，我是菜鸟，更要学习，练习！！！一点心得，大家分享。如有错误，蒙请老大指正。

【破解平台】：`XP`＋`OD`

【软件介绍】：编程文档手册- 集编程文档的收集、查询、浏览、编辑等功能于一体的辅助编程类软件。通过日积月累，相信本软件将给广大程序开发人员以及编程爱好者们带来很大的帮助。

【简要说明】：软件下载来自天空软件站。

【破解过程】：

1. `PEID` 查壳，`DBPE 2.33` -> `Ding Boy` 的壳，刚好手头有 `DBPE 2.33` 的脱壳机，脱之，再查，`Borland C++ 1999`，用 `W32Dasm` 不能反汇编，可能使用了 `Anti` 手段，那就用 `OD` 解决吧；
2. 注册对话框中有“完成注册”按钮为灰色，只有输入正确的注册码才会被激活，看来在这里找不到有价值的信息
3. 用 `OD` 载入脱壳后的程序，搜索字符串参考“`Unregistered`”（因为程序运行后在任务栏上有“`Promanual[Unregistered]`”字样，这里就是我们解决这个软件的突破口），可找到：

```
00401D94   .  E8 BF642200   call unpacked.00628258
00401D99   .  66:C745 C8 08>mov word ptr ss:[ebp-38],8
00401D9F   .  BA 06B16200   mov edx,unpacked.0062B106              ;  ASCII "Software\\Kooborp"
00401DA4   .  8D45 F4       lea eax,dword ptr ss:[ebp-C]
00401DA7   .  E8 40652200   call unpacked.006282EC
00401DAC   .  FF45 D4       inc dword ptr ss:[ebp-2C]
00401DAF   .  8B10          mov edx,dword ptr ds:[eax]
00401DB1   .  B1 01         mov cl,1
00401DB3   .  8BC3          mov eax,ebx
00401DB5   .  E8 822B1700   call unpacked.0057493C
00401DBA   .  FF4D D4       dec dword ptr ss:[ebp-2C]
00401DBD   .  8D45 F4       lea eax,dword ptr ss:[ebp-C]
00401DC0   .  BA 02000000   mov edx,2
00401DC5   .  E8 C6662200   call unpacked.00628490
00401DCA   .  66:C745 C8 14>mov word ptr ss:[ebp-38],14
00401DD0   .  BA 17B16200   mov edx,unpacked.0062B117              ;  ASCII "RegYesNo"
00401DD5   .  8D45 F0       lea eax,dword ptr ss:[ebp-10]
00401DD8   .  E8 0F652200   call unpacked.006282EC
00401DDD   .  FF45 D4       inc dword ptr ss:[ebp-2C]
00401DE0   .  8D4D EC       lea ecx,dword ptr ss:[ebp-14]
00401DE3   .  8B10          mov edx,dword ptr ds:[eax]
00401DE5   .  33C0          xor eax,eax
00401DE7   .  8945 EC       mov dword ptr ss:[ebp-14],eax
00401DEA   .  8BC3          mov eax,ebx
00401DEC   .  FF45 D4       inc dword ptr ss:[ebp-2C]
00401DEF   .  E8 F82F1700   call unpacked.00574DEC
00401DF4   .  8D55 EC       lea edx,dword ptr ss:[ebp-14]
00401DF7   .  B8 C0B06200   mov eax,unpacked.0062B0C0
00401DFC   .  E8 BF662200   call unpacked.006284C0
00401E01   .  FF4D D4       dec dword ptr ss:[ebp-2C]
00401E04   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]
00401E07   .  BA 02000000   mov edx,2
00401E0C   .  E8 7F662200   call unpacked.00628490
00401E11   .  FF4D D4       dec dword ptr ss:[ebp-2C]
00401E14   .  8D45 F0       lea eax,dword ptr ss:[ebp-10]
00401E17   .  BA 02000000   mov edx,2
00401E1C   .  E8 6F662200   call unpacked.00628490
00401E21   .  8BC3          mov eax,ebx
00401E23   .  E8 E4AA1800   call unpacked.0058C90C
00401E28   .  66:C745 C8 20>mov word ptr ss:[ebp-38],20
00401E2E   .  8B15 54726400 mov edx,dword ptr ds:[647254]         ;  unpacked.0064C824
00401E34   .  8B02          mov eax,dword ptr ds:[edx]
00401E36   .  E8 E5A41C00   call unpacked.005CC320
00401E3B   .  8B15 54726400 mov edx,dword ptr ds:[647254]         ;  unpacked.0064C824
00401E41   .  8B0A          mov ecx,dword ptr ds:[edx]
00401E43   .  B2 01         mov dl,1
00401E45   .  A1 047B6300   mov eax,dword ptr ds:[637B04]
00401E4A   .  E8 252F0200   call unpacked.00424D74
00401E4F   .  8BD8          mov ebx,eax
00401E51   .  A1 AC6C6400   mov eax,dword ptr ds:[646CAC]
00401E56   .  8918          mov dword ptr ds:[eax],ebx
00401E58   .  8BC3          mov eax,ebx
00401E5A   .  E8 E96E1C00   call unpacked.005C8D48
00401E5F   .  8B15 AC6C6400 mov edx,dword ptr ds:[646CAC]         ;  unpacked._LogoForm
00401E65   .  8B02          mov eax,dword ptr ds:[edx]
00401E67   .  8B10          mov edx,dword ptr ds:[eax]
00401E69     FF92 88000000 call dword ptr ds:[edx+88]
00401E6F   .  66:C745 C8 2C>mov word ptr ss:[ebp-38],2C
00401E75   .  BA 20B16200   mov edx,unpacked.0062B120              ;  ASCII "blowjj"
00401E7A   .  8D45 E8       lea eax,dword ptr ss:[ebp-18]
00401E7D   .  E8 6A642200   call unpacked.006282EC
00401E82   .  FF45 D4       inc dword ptr ss:[ebp-2C]
00401E85   .  8D55 E8       lea edx,dword ptr ss:[ebp-18]
00401E88   .  B8 C0B06200   mov eax,unpacked.0062B0C0
00401E8D   .  E8 FA662200   call unpacked.0062858C
00401E92   .  50            push eax                                 ; /Arg1
00401E93   .  FF4D D4       dec dword ptr ss:[ebp-2C]                ; |
00401E96   .  8D45 E8       lea eax,dword ptr ss:[ebp-18]            ; |
00401E99   .  BA 02000000   mov edx,2                                ; |
00401E9E   .  E8 ED652200   call unpacked.00628490                   ; \unpacked.00628490
00401EA3   .  59            pop ecx
00401EA4   .  84C9          test cl,cl
00401EA6   .  74 36         je short unpacked.00401EDE
00401EA8   .  66:C745 C8 38>mov word ptr ss:[ebp-38],38
00401EAE   .  BA 27B16200   mov edx,unpacked.0062B127              ;  ASCII "Promanual[Unregistered]"
00401EB3   .  8D45 E4       lea eax,dword ptr ss:[ebp-1C]
00401EB6   .  E8 31642200   call unpacked.006282EC
00401EBB   .  FF45 D4       inc dword ptr ss:[ebp-2C]
00401EBE   .  8B10          mov edx,dword ptr ds:[eax]
00401EC0   .  A1 54726400   mov eax,dword ptr ds:[647254]
00401EC5   .  8B00          mov eax,dword ptr ds:[eax]
00401EC7   .  E8 60A01C00   call unpacked.005CBF2C
00401ECC   .  FF4D D4       dec dword ptr ss:[ebp-2C]
00401ECF   .  8D45 E4       lea eax,dword ptr ss:[ebp-1C]
00401ED2   .  BA 02000000   mov edx,2
00401ED7   .  E8 B4652200   call unpacked.00628490
00401EDC   .  EB 34         jmp short unpacked.00401F12
00401EDE   >  66:C745 C8 44>mov word ptr ss:[ebp-38],44
00401EE4   .  BA 3FB16200   mov edx,unpacked.0062B13F              ;  ASCII "Promanual"
00401EE9   .  8D45 E0       lea eax,dword ptr ss:[ebp-20]
00401EEC   .  E8 FB632200   call unpacked.006282EC
```

4. 通过 `OD` 动态调试该段代码，很容易知道，只要注册表中：`RegYesNo` 键值为 `blowjj` 即注册成功。所以 `blowjj` 是注册成功标志。

```
[HKEY_LOCAL_MACHINE\SOFTWARE\Kooborp]

"RegYesNo"="blowjj"
```

5. 到这就可以说这个软件搞定了，但我们要找出注册算法来。于是我再找写注册表的地方：

```
00437C30  /.  55             push ebp
00437C31  |.  8BEC           mov ebp,esp
00437C33  |.  83C4 B0        add esp,-50
00437C36  |.  53             push ebx
00437C37  |.  56             push esi
00437C38  |.  57             push edi
00437C39  |.  8945 B0        mov dword ptr ss:[ebp-50],eax
00437C3C  |.  8D75 B4        lea esi,dword ptr ss:[ebp-4C]
00437C3F  |.  8D7D F4        lea edi,dword ptr ss:[ebp-C]
00437C42  |.  B8 2CC06300    mov eax,unpacked.0063C02C
00437C47  |.  E8 D0FC1C00    call unpacked.0060791C
00437C4C  |.  66:C746 10 14> mov word ptr ds:[esi+10],14
00437C52  |.  33D2           xor edx,edx
00437C54  |.  8955 FC        mov dword ptr ss:[ebp-4],edx
00437C57  |.  8D55 FC        lea edx,dword ptr ss:[ebp-4]
00437C5A  |.  FF46 1C        inc dword ptr ds:[esi+1C]
00437C5D  |.  8B45 B0        mov eax,dword ptr ss:[ebp-50]
00437C60  |.  8B80 18030000  mov eax,dword ptr ds:[eax+318]
00437C66  |.  E8 A1201A00    call unpacked.005D9D0C
00437C6B  |.  66:C746 10 08> mov word ptr ds:[esi+10],8
00437C71  |.  66:C746 10 20> mov word ptr ds:[esi+10],20
00437C77  |.  33D2           xor edx,edx
00437C79  |.  8955 F8        mov dword ptr ss:[ebp-8],edx
00437C7C  |.  FF46 1C        inc dword ptr ds:[esi+1C]
00437C7F  |.  66:C746 10 08> mov word ptr ds:[esi+10],8
00437C85  |.  BB 01000000    mov ebx,1
00437C8A  |.  EB 48          jmp short unpacked.00437CD4
00437C8C  |>  66:C746 10 2C>/ mov word ptr ds:[esi+10],2C
00437C92  |.  33C0           |xor eax,eax
00437C94  |.  B9 01000000    |mov ecx,1
```



```
00437C99  .|  8907            |mov dword ptr ds:[edi],eax

00437C9B  .|  57              |push edi                        ; /Arg1

00437C9C  .|  FF46 1C        |inc dword ptr ds:[esi+1C]       ; |

00437C9F  .|  B8 9C756400    |mov eax,unpacked.0064759C        ; |

00437CA4  .|  8BD3           |mov edx,ebx                      ; |

00437CA6  .|  E8 450A1F00    |call unpacked.006286F0           ; \unpacked.006286F0

00437CAB  .|  8BD7           |mov edx,edi

00437CAD  .|  8D45 F8       |lea eax,dword ptr ss:[ebp-8]

00437CB0  .|  B9 01000000    |mov ecx,1

00437CB5  .|  E8 CA091F00    |call unpacked.00628684

00437CBA  .|  8BD0           |mov edx,eax

00437CBC  .|  8D45 F8       |lea eax,dword ptr ss:[ebp-8]

00437CBF  .|  E8 FC071F00    |call unpacked.006284C0

00437CC4  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]

00437CC7  .|  8BC7           |mov eax,edi

00437CC9  .|  BA 02000000    |mov edx,2

00437CCE  .|  E8 BD071F00    |call unpacked.00628490

00437CD3  .|  43              |inc ebx

00437CD4  >|  833D 9C756400> cmp dword ptr ds:[64759C],0       ; 实际上，ds:[64759C]处所指的便是真码，在此可作内存注册机。

00437CDB  .|  74 0B           |je short unpacked.00437CE8

00437CDD  .|  8B0D 9C756400  |mov ecx,dword ptr ds:[64759C]

00437CE3  .|  8B41 FC       |mov eax,dword ptr ds:[ecx-4]

00437CE6  .|  EB 02           |jmp short unpacked.00437CEA

00437CE8  >|  33C0            |xor eax,eax

00437CEA  >|  3BD8            |cmp ebx,eax

00437CEC  .^ 7E 9E           \jle short unpacked.00437C8C

00437CEE  .|  66:C746 10 08> mov word ptr ds:[esi+10],8

00437CF4  .|  33DB            |xor ebx,ebx

00437CF6  .|  BF 01000000    |mov edi,1

00437CFB  .|  E9 83000000    |jmp unpacked.00437D83

00437D00  >|  33C0            /xor eax,eax

00437D02  .|  8D55 EC       |lea edx,dword ptr ss:[ebp-14]

00437D05  .|  8945 EC       |mov dword ptr ss:[ebp-14],eax

00437D08  .|  52              |push edx

00437D09  .|  FF46 1C       |inc dword ptr ds:[esi+1C]

00437D0C  .|  837D FC 00     |cmp dword ptr ss:[ebp-4],0

00437D10  .|  74 08           |je short unpacked.00437D1A

00437D12  .|  8B4D FC       |mov ecx,dword ptr ss:[ebp-4]

00437D15  .|  8B51 FC       |mov edx,dword ptr ds:[ecx-4]

00437D18  .|  EB 02           |jmp short unpacked.00437D1C

00437D1A  >|  33D2            |xor edx,edx

00437D1C  >|  2BD7            |sub edx,edi                      ; |

00437D1E  .|  42              |inc edx                            ; |

00437D1F  .|  8D45 F8       |lea eax,dword ptr ss:[ebp-8]      ; |

00437D22  .|  B9 01000000    |mov ecx,1                           ; |

00437D27  .|  E8 C4091F00    |call unpacked.006286F0             ; \unpacked.006286F0

00437D2C  .|  8D45 EC       |lea eax,dword ptr ss:[ebp-14]

00437D2F  .|  33D2            |xor edx,edx

00437D31  .|  50              |push eax

00437D32  .|  8D4D F0       |lea ecx,dword ptr ss:[ebp-10]

00437D35  .|  66:C746 10 38> mov word ptr ds:[esi+10],38

00437D3B  .|  8955 F0       |mov dword ptr ss:[ebp-10],edx

00437D3E  .|  51              |push ecx                           ; /Arg1

00437D3F  .|  FF46 1C       |inc dword ptr ds:[esi+1C]          ; |

00437D42  .|  B9 01000000    |mov ecx,1                           ; |

00437D47  .|  8BD7            |mov edx,edi                         ; |

00437D49  .|  8D45 FC       |lea eax,dword ptr ss:[ebp-4]      ; |

00437D4C  .|  E8 9F091F00    |call unpacked.006286F0             ; \unpacked.006286F0

00437D51  .|  8D45 F0       |lea eax,dword ptr ss:[ebp-10]

00437D54  .|  5A              |pop edx

00437D55  .|  E8 32081F00    |call unpacked.0062858C

00437D5A  .|  50              |push eax

00437D5B  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]

00437D5E  .|  8D45 EC       |lea eax,dword ptr ss:[ebp-14]

00437D61  .|  BA 02000000    |mov edx,2

00437D66  .|  E8 25071F00    |call unpacked.00628490

00437D6B  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]          ; |

00437D6E  .|  8D45 F0       |lea eax,dword ptr ss:[ebp-10]      ; |

00437D71  .|  BA 02000000    |mov edx,2                          ; |

00437D76  .|  E8 15071F00    |call unpacked.00628490            ; \unpacked.00628490

00437D7B  .|  59              |pop ecx

00437D7C  .|  84C9            |test cl,cl

00437D7E  .|  74 02           |je short unpacked.00437D82

00437D80  .|  B3 01           |mov bl,1

00437D82  >|  47              |inc edi

00437D83  >|  837D F8 00     |cmp dword ptr ss:[ebp-8],0

00437D87  .|  74 08           |je short unpacked.00437D91

00437D89  .|  8B45 F8       |mov eax,dword ptr ss:[ebp-8]

00437D8C  .|  8B50 FC       |mov edx,dword ptr ds:[eax-4]

00437D8F  .|  EB 02           |jmp short unpacked.00437D93

00437D91  >|  33D2            |xor edx,edx

00437D93  >|  3BFA            |cmp edi,edx

00437D95  .^ 0F8E 65FFFFFF  \jle unpacked.00437D00

00437D9B  .|  80FB 01        |cmp bl,1

00437D9E  .|  0F84 88010000 |je unpacked.00437F2C
                             ; 通过动态分析这两个循环发现，这只是一个搞得很复杂的比较功能，与机器码无关

00437DA4  .|  66:C746 10 44> mov word ptr ds:[esi+10],44

00437DAA  .|  33C0            |xor eax,eax

00437DAC  .|  8945 E8       |mov dword ptr ss:[ebp-18],eax

00437DAF  .|  8D55 E8       |lea edx,dword ptr ss:[ebp-18]

00437DB2  .|  FF46 1C       |inc dword ptr ds:[esi+1C]

00437DB5  .|  8B45 B0       |mov eax,dword ptr ss:[ebp-50]

00437DB8  .|  8B80 18030000 |mov eax,dword ptr ds:[eax+318]

00437DBE  .|  E8 491F1A00    |call unpacked.005D9D0C

00437DC3  .|  837D E8 00     |cmp dword ptr ss:[ebp-18],0

00437DC7  .|  74 08           |je short unpacked.00437DD1

00437DC9  .|  8B55 E8       |mov edx,dword ptr ss:[ebp-18]

00437DCC  .|  8B4A FC       |mov ecx,dword ptr ds:[edx-4]

00437DCF  .|  EB 02           |jmp short unpacked.00437DD3

00437DD1  >|  33C9            |xor ecx,ecx

00437DD3  >|  833D 9C756400> cmp dword ptr ds:[64759C],0

00437DDA  .|  74 0A           |je short unpacked.00437DE6

00437DDC  .|  A1 9C756400    |mov eax,dword ptr ds:[64759C]

00437DE1  .|  8B50 FC       |mov edx,dword ptr ds:[eax-4]

00437DE4  .|  EB 02           |jmp short unpacked.00437DE8

00437DE6  >|  33D2            |xor edx,edx

00437DE8  >|  3BCA            |cmp ecx,edx

00437DEA  .|  BA 02000000    |mov edx,2

00437DEF  .|  0F95C0         |setne al

00437DF2  .|  83E0 01        |and eax,1

00437DF5  .|  50              |push eax                        ; /Arg1

00437DF6  .|  8D45 E8       |lea eax,dword ptr ss:[ebp-18]    ; |

00437DF9  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]        ; |

00437DFC  .|  E8 8F061F00    |call unpacked.00628490          ; \unpacked.00628490

00437E01  .|  59              |pop ecx

00437E02  .|  84C9            |test cl,cl

00437E04  .|  74 2E           |je short unpacked.00437E34      ; 为 0 则注册码正确

00437E06  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]

00437E09  .|  8D45 F8       |lea eax,dword ptr ss:[ebp-8]

00437E0C  .|  BA 02000000    |mov edx,2

00437E11  .|  E8 7A061F00    |call unpacked.00628490

00437E16  .|  FF4E 1C       |dec dword ptr ds:[esi+1C]

00437E19  .|  8D45 FC       |lea eax,dword ptr ss:[ebp-4]

00437E1C  .|  BA 02000000    |mov edx,2

00437E21  .|  E8 6A061F00    |call unpacked.00628490

00437E26  .|  8B0E            |mov ecx,dword ptr ds:[esi]

00437E28  .|  64:890D 00000> mov dword ptr fs:[0],ecx

00437E2F  .|  E9 21010000    |jmp unpacked.00437F55

00437E34  >|  8B45 B0       |mov eax,dword ptr ss:[ebp-50]

00437E37  .|  8B80 00030000 |mov eax,dword ptr ds:[eax+300]

00437E3D  .|  B2 01           |mov dl,1

00437E3F  .|  8B08            |mov ecx,dword ptr ds:[eax]

00437E41  .|  FF51 64       |call dword ptr ds:[ecx+64]

00437E44  .|  8B45 B0       |mov eax,dword ptr ss:[ebp-50]
```



```
00437E47  |.  8B80 04030000 mov eax,dword ptr ds:\[eax+304\]
00437E4D  |.  33D2          xor edx,edx
00437E4F  |.  8B08          mov ecx,dword ptr ds:\[eax\]
00437E51  |.  FF51 64       call dword ptr ds:\[ecx+64\]
00437E54  |.  66:C746 10 50>mov word ptr ds:\[esi+10\],50
00437E5A  |.  BA C7BB6300   mov edx,unpacked.0063BBC7                 ; 按钮激活 ASCII "true"
00437E5F  |.  8D45 E4       lea eax,dword ptr ss:\[ebp-1C\]
00437E62  |.  E8 85041F00   call unpacked.006282EC
00437E67  |.  FF46 1C       inc dword ptr ds:\[esi+1C\]
00437E6A  |.  8D55 E4       lea edx,dword ptr ss:\[ebp-1C\]
00437E6D  |.  B8 C0B06200   mov eax,unpacked.0062B0C0
00437E72  |.  E8 49061F00   call unpacked.006284C0
00437E77  |.  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437E7A  |.  8D45 E4       lea eax,dword ptr ss:\[ebp-1C\]
00437E7D  |.  BA 02000000   mov edx,2
00437E82  |.  E8 09061F00   call unpacked.00628490
00437E87  |.  B2 01         mov dl,1
00437E89  |.  A1 38475700   mov eax,dword ptr ds:\[574738\]
00437E8E  |.  E8 A5C91300   call unpacked.00574838
00437E93  |.  8BD8          mov ebx,eax
00437E95  |.  BA 02000080   mov edx,80000002
00437E9A  |.  8BC3          mov eax,ebx
00437E9C  |.  E8 B7031F00   call unpacked.00628258
00437EA1  |.  66:C746 10 5C>mov word ptr ds:\[esi+10\],5C
00437EA7  |.  BA CCBB6300   mov edx,unpacked.0063BBCC                 ;  ASCII "Software\\Kooborp"
00437EAC  |.  8D45 E0       lea eax,dword ptr ss:\[ebp-20\]
00437EAF  |.  E8 38041F00   call unpacked.006282EC
00437EB4  |.  FF46 1C       inc dword ptr ds:\[esi+1C\]
00437EB7  |.  8B10          mov edx,dword ptr ds:\[eax\]
00437EB9  |.  B1 01         mov cl,1
00437EBB  |.  8BC3          mov eax,ebx
00437EBD  |.  E8 7ACA1300   call unpacked.0057493C
00437EC2  |.  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437EC5  |.  8D45 E0       lea eax,dword ptr ss:\[ebp-20\]
00437EC8  |.  BA 02000000   mov edx,2
00437ECD  |.  E8 BE051F00   call unpacked.00628490
00437ED2  |.  BA E6BB6300   mov edx,unpacked.0063BBE6                 ;  ASCII "blowjj"
00437ED7  |.  8D45 D8       lea eax,dword ptr ss:\[ebp-28\]
00437EDA  |.  E8 0D041F00   call unpacked.006282EC
00437EDF  |.  FF46 1C       inc dword ptr ds:\[esi+1C\]
00437EE2  |.  BA DDBB6300   mov edx,unpacked.0063BBDD                 ;  ASCII "RegYesNo"
00437EE7  |.  8B08          mov ecx,dword ptr ds:\[eax\]
00437EE9  |.  8D45 DC       lea eax,dword ptr ss:\[ebp-24\]
00437EEC  |.  51            push ecx
00437EED  |.  66:C746 10 68>mov word ptr ds:\[esi+10\],68
00437EF3  |.  E8 F4031F00   call unpacked.006282EC                     ;  以上将注册成功标志 blowjj 写入到注册表中
00437EF8  |.  FF46 1C       inc dword ptr ds:\[esi+1C\]
00437EFB  |.  8B10          mov edx,dword ptr ds:\[eax\]
00437EFD  |.  8BC3          mov eax,ebx
00437EFF  |.  59            pop ecx
00437F00  |.  E8 BBCE1300   call unpacked.00574DC0
00437F05  |.  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437F08  |.  8D45 D8       lea eax,dword ptr ss:\[ebp-28\]
00437F0B  |.  BA 02000000   mov edx,2
00437F10  |.  E8 7B051F00   call unpacked.00628490
00437F15  |.  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437F18  |.  8D45 DC       lea eax,dword ptr ss:\[ebp-24\]
00437F1B  |.  BA 02000000   mov edx,2
00437F20  |.  E8 6B051F00   call unpacked.00628490
00437F25  |.  8BC3          mov eax,ebx
00437F27  |.  E8 E0491500   call unpacked.0058C90C
00437F2C  |>  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437F2F  |.  8D45 F8       lea eax,dword ptr ss:\[ebp-8\]
00437F32  |.  BA 02000000   mov edx,2
00437F37  |.  E8 54051F00   call unpacked.00628490
00437F3C  |.  FF4E 1C       dec dword ptr ds:\[esi+1C\]
00437F3F  |.  8D45 FC       lea eax,dword ptr ss:\[ebp-4\]
00437F42  |.  BA 02000000   mov edx,2
00437F47  |.  E8 44051F00   call unpacked.00628490
00437F4C  |.  8B0E          mov ecx,dword ptr ds:\[esi\]
00437F4E  |.  64:890D 00000>mov dword ptr fs:\[0\],ecx
00437F55  |>  5F            pop edi
00437F56  |.  5E            pop esi
00437F57  |.  5B            pop ebx
00437F58  |.  8BE5          mov esp,ebp
00437F5A  |.  5D            pop ebp
00437F5B  \\.  C3            retn
```

### 6、对内存地址断点分析

对 `ds:[64759C]` 处所指的内存地址下硬件写入断点。`F2` 重新载入，`F9` 运行，经过数次硬件写入断点后，直到内存单元出现真码第一位时，按 `F8` 依次返回。可找到如下子程序：

```
00437778  /.  55            push ebp
00437779  |.  8BEC          mov ebp,esp
0043777B  |.  81C4 3CFFFFFF add esp,-0C4
........略........
00437816  |.  33F6          xor esi,esi
00437818  |.  E9 98000000   jmp unpacked.004378B5
0043781D  |>  66:C743 10 20>/mov word ptr ds:\[ebx+10\],20
00437823  |.  33C0          |xor eax,eax
00437825  |.  8D55 F0       |lea edx,dword ptr ss:\[ebp-10\]
00437828  |.  8945 F0       |mov dword ptr ss:\[ebp-10\],eax
0043782B  |.  52            |push edx                          ; /Arg1
0043782C  |.  FF43 1C       |inc dword ptr ds:\[ebx+1C\]                 ; |
0043782F  |.  8BD6          |mov edx,esi                           ;从第 SI 位取 |
00437831  |.  B8 98756400   |mov eax,unpacked.00647598               ;机器码首地址 |
00437836  |.  B9 03000000   |mov ecx,3                             ;连续取 3 位 |
0043783B  |.  E8 B00E1F00   |call unpacked.006286F0                  ; \unpacked.006286F0
00437840  |.  8D45 F0       |lea eax,dword ptr ss:\[ebp-10\]         ;  取得的 3 位字符首地址
00437843  |.  E8 8C0F1F00   |call unpacked.006287D4                  ;  转换为 16 进制，返回到 EAX
00437848  |.  B9 0D000000   |mov ecx,0D
0043784D  |.  99            |cdq
0043784E  |.  F7F9          |idiv ecx                              ;  除以 13
00437850  |.  8D45 F4       |lea eax,dword ptr ss:\[ebp-C\]
00437853  |.  E8 B00B1F00   |call unpacked.00628408                  ;  将余数转换为十进制
00437858  |.  FF43 1C       |inc dword ptr ds:\[ebx+1C\]
0043785B  |.  FF4B 1C       |dec dword ptr ds:\[ebx+1C\]
0043785E  |.  8D45 F0       |lea eax,dword ptr ss:\[ebp-10\]
00437861  |.  BA 02000000   |mov edx,2
00437866  |.  E8 250C1F00   |call unpacked.00628490
0043786B  |.  66:C743 10 14>|mov word ptr ds:\[ebx+10\],14
00437871  |.  837D F4 00    |cmp dword ptr ss:\[ebp-C\],0
00437875  |.  74 08         |je short unpacked.0043787F
00437877  |.  8B4D F4       |mov ecx,dword ptr ss:\[ebp-C\]           ;  余数的十进制字符
0043787A  |.  8B41 FC       |mov eax,dword ptr ds:\[ecx-4\]           ;  长度
0043787D  |.  EB 02         |jmp short unpacked.00437881
0043787F  |>  33C0          |xor eax,eax
00437881  |>  03F8          |add edi,eax
00437883  |.  8D55 F4       |lea edx,dword ptr ss:\[ebp-C\]
00437886  |.  B8 9C756400   |mov eax,unpacked.0064759C
0043788B  |.  8BCF          |mov ecx,edi
0043788D  |.  E8 F20D1F00   |call unpacked.00628684
00437892  |.  8BD0          |mov edx,eax
00437894  |.  B8 9C756400   |mov eax,unpacked.0064759C
00437899  |.  E8 220C1F00   |call unpacked.006284C0                   ;  字符串联，EDX 所指即为真码首地址
0043789E  |.  FF4B 1C       |dec dword ptr ds:\[ebx+1C\]
004378A1  |.  8D45 F4       |lea eax,dword ptr ss:\[ebp-C\]
004378A4  |.  BA 02000000   |mov edx,2
004378A9  |.  E8 E20B1F00   |call unpacked.00628490
004378AE  |.  66:C743 10 00>|mov word ptr ds:\[ebx+10\],0
004378B4  |.  46            |inc esi                                 ;  计数器加 1
004378B5  |>  833D 98756400> cmp dword ptr ds:\[647598\],0             ;  从这里进入循环体
004378BC  |.  74 0B         |je short unpacked.004378C9
004378BE  |.  8B0D 98756400 |mov ecx,dword ptr ds:\[647598\]
004378C4  |.  8B41 FC       |mov eax,dword ptr ds:\[ecx-4\]
004378C7  |.  EB 02         |jmp short unpacked.004378CB
004378C9  |>  33C0          |xor eax,eax
004378CB  |>  3BF0          |cmp esi,eax                              ;  EAX＝＝9
004378CD  |.^ 0F8E 4AFFFFFF \\jle unpacked.0043781D                    ;  不大于则循环继续
```



## 004378D3 |. 66:C743 10 2C>mov word ptr ds:[ebx+10],2C
## 004378D9 |. 33D2 xor edx,edx
## ........略........
## 00437B90 |. 5F pop edi
## 00437B91 |. 5E pop esi
## 00437B92 |. 5B pop ebx
## 00437B93 |. 8BE5 mov esp,ebp
## 00437B95 |. 5D pop ebp
## 00437B96 \\. C3 retn

### 7、根据我的机器码总结一下算法

我的机器码为：`139984676020`，但参与运算的只有 `139984676` 九位，循环次数共 10 次，`ESI` 取 0－9：

`ESI` 为 `0`，也从第 1 位取 3 位，即 `139`，然后 `139MOD13＝9`;

`ESI` 为 `1`，从第 1 位取 3 位，即 `139`，然后 `139MOD13＝9`;

`ESI` 为 `2`，从第 2 位取 3 位，即 `399`，然后 `399MOD13＝9`;

`ESI` 为 `3`，从第 3 位取 3 位，即 `998`，然后 `998MOD13＝10`;

`ESI` 为 `4`，从第 4 位取 3 位，即 `984`，然后 `984MOD13＝9`;

`ESI` 为 `5`，从第 5 位取 3 位，即 `846`，然后 `846MOD13＝1`;

`ESI` 为 `6`，从第 6 位取 3 位，即 `467`，然后 `467MOD13＝12`;

`ESI` 为 `7`，从第 7 位取 3 位，即 `676`，然后 `676MOD13＝0`;

`ESI` 为 `8`，从第 8 位取 3 位，后面只有 2 位，就只取 2 位，即 `76`，然后 `76MOD13＝11`;

`ESI` 为 `9`，从第 9 位取 3 位，后面只有 1 位，就只取 1 位，即 `6`，然后 `6MOD13＝6`;

把余数联到一起即为注册码：`9991091120116`

### 8、后话

程序的算法很简单，但我觉得破解思路比较重要。作者虽然用了灰色按钮与机器码的保护方式，但相当脆弱，且注册表键值与机器码无关，并且内存中明码比较。所以破解难度易。

本文完

破解这个软件太简单,思路不要太麻烦!

1、先用 `Dede` 反编译一下，它是用 `bcb6` 编的

2、找到 `tregisterform`，这是注册窗口的

3、找到 `edRegId` 控件，这是用来输入注册码的！

   由于它只有输入了正确的注册码才能使注册按钮生效，就一定会

   有一个检测的过程，要么是使用了 `edit` 控件的 `onkeypress` 事件，要么就是在

   外部进行检测，查看 `edRegId` 控件没有任何事件函数。说明是从外部进行

   检测。

4、可以看到还有一个 `timer` 控件，很简单，就是用 `timer` 时钟定时进行检查了！

   进行 `ontimer` 事件函数，果不其然，第一个就是 `gettext` 函数调用，用 `olldbg`

   装入，在这个函数上下断点。运行程序，打开注册窗口，不用输入注册码。

   程序就会停在这个断点上，“没事走二步”（本山语录），看到注册码了

   了吧！

完成，时间三分钟

想看注册码是如何生成的，就是 `gettext` 上面的 `call` 了

这个软件真不错,可以当个学习笔记文档了.

```
sjqm=m_jqm;

  strcpy(cjqm,sjqm.Mid(0,3));

  jqm=atoi( cjqm );

  code[0]=jqm%13;

    _itoa( code[0], tmp, 10 );

   m_code=tmp;

  scode=sjqm.Mid(0,9);

  for(ESI=0;ESI<9;ESI++)

  {

    if(ESI==7)

    {

      code[ESI]=atoi( scode.Mid(ESI,2) )%13;

      _itoa( code[ESI], tmp, 10 );

            m_code+=tmp;

    }

    else if(ESI==8)

    {

      code[ESI]=atoi( scode.Mid(ESI,1) )%13;

        _itoa( code[ESI], tmp, 10 );

            m_code+=tmp;

    }

    else

    {

      code[ESI]=atoi( scode.Mid(ESI,3) )%13;

        _itoa( code[ESI], tmp, 10 );

            m_code+=tmp;

    }

  }

4dd466d0e1e0cc123afbe504da2462ca
```
