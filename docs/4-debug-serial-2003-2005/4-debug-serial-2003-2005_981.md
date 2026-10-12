
# 最优秀的排课软件分析详细过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-81.htm`](https://www.pediy.com/kssd/pediy07/pediy7-81.htm)

【破解作者】 `jsliyangsj`

【作者邮箱】 `sjcrack@yahoo.com.cn`

【使用工具】 `peid` `OllyDbg1.10`

【破解平台】 `Winxp`

【软件名称】 `自明排课 9.0` 最新版本

【下载地址】 `http://www.zimingsoft.com/soft/zmrj.arj`

【软件介绍】一个贼好的排课软件，名副其实可以算最好的了，我们单位就用它，（以前一直用排课王，没它好）速度快，40 个班级，全部优化好，约 1 个小时。

## 分析

程序是用 `C++` 编写的。

运行软件后输入序列号，有一个错误信息。

首先用 `OD` 的插件 `ustrref.dll` 查找有一个错误信息！

：您输入的序列号有误，您述而的名单将不能保存

双击到了：

```
00431E30   |.  68 B0354600   push ZMPK.004635B0                  ; |Title = "注册许可证号"
00431E35   |.  68 80354600   push ZMPK.00463580                  ; |Text = "您输入的许可证号有误，您输入的名单将不能保存。"
00431E3A   |.  8B48 08       mov ecx,dword ptr ds:[eax+8]         ; |
```

再向上看：

```
00431D77   |.  6A 00         push 0                              ; /lParam = NULL
00431D79   |.  8B51 08       mov edx,dword ptr ds:[ecx+8]         ; |
00431D7C   |.  8B0D CCF54600 mov ecx,dword ptr ds:[46F5CC]        ; |
00431D82   |.  68 201F4300   push ZMPK.00431F20                  ; |DlgProc = ZMPK.00431F20
00431D87   |.  8B8432 900B00> mov eax,dword ptr ds:[edx+esi+B90]  ; |
00431D8E   |.  50            push eax                              ; |hOwner
00431D8F   |.  68 C0354600   push ZMPK.004635C0                  ; |pTemplate = "CheckUse"
00431D94   |.  51            push ecx                              ; |hInst => NULL
00431D95   |.  FF15 4CB24500 call dword ptr ds:[<&USER32.DialogBoxPar>; \此处是得到输入的输入码
00431D9B   |.  85C0          test eax,eax
00431D9D   |.  8B06          mov eax,dword ptr ds:[esi]
00431D9F   |.  8B3D ECB14500 mov edi,dword ptr ds:[<&USER32.SendMessa>;  USER32.SendMessageA
00431DA5   |.  0F95C2        setne dl
00431DA8   |.  8B48 08       mov ecx,dword ptr ds:[eax+8]
00431DAB   |.  885424 10     mov byte ptr ss:[esp+10],dl
00431DAF   |.  8BD0          mov edx,eax
00431DB1   |.  6A 00         push 0                              ; /lParam = 0
00431DB3   |.  8D1C31        lea ebx,dword ptr ds:[ecx+esi]       ; |
00431DB6   |.  68 33750000   push 7533                           ; |压入一个错误参数 7533 为了得到一个错误的注册码
00431DBB   |.  8B42 08       mov eax,dword ptr ds:[edx+8]         ; |
00431DBE   |.  68 11010000   push 111                            ; |Message = WM_COMMAND
00431DC3   |.  8B8C30 900B00> mov ecx,dword ptr ds:[eax+esi+B90]  ; |
00431DCA   |.  51            push ecx                              ; |hWnd
00431DCB   |.  FFD7          call edi                              ; \产生一个错误的注册码  这里也可以跟进的一样的
00431DCD   |.  3983 B4000000 cmp dword ptr ds:[ebx+B4],eax         ;  比较你的输入码与错误的注册码是否相等
00431DD3   |.  0F84 81000000 je ZMPK.00431E5A                    ;  相等就完了（可能是迷惑破解吧）
00431DD9   |.  8B16          mov edx,dword ptr ds:[esi]
00431DDB   |.  6A 00         push 0                              ; /lParam = 0
00431DDD   |.  68 32750000   push 7532                           ; |压入一个错误参数 7532 为了得到一个错误的注册码
00431DE2   |.  68 11010000   push 111                            ; |Message = WM_COMMAND
00431DE7   |.  8B42 08       mov eax,dword ptr ds:[edx+8]         ; |
00431DEA   |.  8B8C30 900B00> mov ecx,dword ptr ds:[eax+esi+B90]  ; |
00431DF1   |.  8D1C30        lea ebx,dword ptr ds:[eax+esi]       ; |
00431DF4   |.  51            push ecx                              ; |hWnd
00431DF5   |.  FFD7          call edi                              ; \产生一个错误的注册码
00431DF7   |.  3983 B4000000 cmp dword ptr ds:[ebx+B4],eax         ;  比较你的输入码与错误的注册码是否相等
00431DFD   |.  74 5B         je short ZMPK.00431E5A                ;  相等就完了
00431DFF   |.  8B93 900B0000 mov edx,dword ptr ds:[ebx+B90]
00431E05   |.  6A 00         push 0                              ; /lParam = 0
00431E07   |.  68 31750000   push 7531                           ; |压入一个正确参数 7531 为了得到一个正确的注册码
00431E0C   |.  68 11010000   push 111                            ; |Message = WM_COMMAND
00431E11   |.  52            push edx                              ; |hWnd
00431E12   |.  FFD7          call edi                              ; \产生一个正确的注册码
00431E14   |.  8B0E          mov ecx,dword ptr ds:[esi]
00431E16   |.  8B51 08       mov edx,dword ptr ds:[ecx+8]
00431E19   |.  398432 B40000> cmp dword ptr ds:[edx+esi+B4],eax     ;  正确的注册码与你的输入码进行比较
00431E20   |.  74 38         je short ZMPK.00431E5A                ;  此处一定要跳了
00431E22   |.  C705 88344600> mov dword ptr ds:[463488],1
00431E2C   |>  8B06          mov eax,dword ptr ds:[esi]
00431E2E   |.  6A 30         push 30                             ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
00431E30   |.  68 B0354600   push ZMPK.004635B0                  ; |Title = "注册许可证号"
00431E35   |.  68 80354600   push ZMPK.00463580                  ; |Text = "您输入的许可证号有误，您输入的名单将不能保存。"
00431E3A   |.  8B48 08       mov ecx,dword ptr ds:[eax+8]         ; |
00431E3D   |.  8B9431 900B00> mov edx,dword ptr ds:[ecx+esi+B90]  ; |
00431E44   |.  52            push edx                              ; |hOwner
00431E45   |.  FF15 90B24500 call dword ptr ds:[<&USER32.MessageBoxA>>; \显示出错信息
00431E4B   |.  8B4424 10     mov eax,dword ptr ss:[esp+10]
00431E4F   |.  5F            pop edi
00431E50   |.  5B            pop ebx
00431E51   |.  25 FF000000   and eax,0FF
00431E56   |.  5E            pop esi
00431E57   |.  C2 0400       retn 4
00431E5A   |>  C705 88344600> mov dword ptr ds:[463488],0            ;  跳这里再次计算注册码
00431E64   |.  8B06          mov eax,dword ptr ds:[esi]
00431E66   |.  8BD0          mov edx,eax
00431E68   |.  6A 00         push 0
00431E6A   |.  8B48 08       mov ecx,dword ptr ds:[eax+8]
00431E6D   |.  68 31750000   push 7531                           ;  再压入一个正确参数 7531 为了得到一个正确的注册码
00431E72   |.  8B42 08       mov eax,dword ptr ds:[edx+8]
00431E75   |.  68 11010000   push 111
00431E7A   |.  8D1C31        lea ebx,dword ptr ds:[ecx+esi]
00431E7D   |.  8B8C30 900B00> mov ecx,dword ptr ds:[eax+esi+B90]
00431E84   |.  51            push ecx
00431E85   |.  FFD7          call edi                              ;  产生一个正确的注册码 关键 F7 进入
00431E87   |.  3983 B4000000 cmp dword ptr ds:[ebx+B4],eax         ;  比较了
00431E8D   |.  74 4D         je short ZMPK.00431EDC
00431E8F   |.  8B16          mov edx,dword ptr ds:[esi]
00431E91   |.  6A 00         push 0
00431E93   |.  68 32750000   push 7532                           ;  错误参数
00431E98   |.  68 11010000   push 111
00431E9D   |.  8B42 08       mov eax,dword ptr ds:[edx+8]
00431EA0   |.  8B8C30 900B00> mov ecx,dword ptr ds:[eax+esi+B90]
00431EA7   |.  8D1C30        lea ebx,dword ptr ds:[eax+esi]
00431EAA   |.  51            push ecx
00431EAB   |.  FFD7          call edi                              ;  错误注册码
00431EAD   |.  3983 B4000000 cmp dword ptr ds:[ebx+B4],eax
00431EB3   |.  74 27         je short ZMPK.00431EDC
00431EB5   |.  8B93 900B0000 mov edx,dword ptr ds:[ebx+B90]
00431EBB   |.  6A 00         push 0
00431EBD   |.  68 33750000   push 7533
00431EC2   |.  68 11010000   push 111
00431EC7   |.  52            push edx
00431EC8   |.  FFD7          call edi                              ;  错误注册码
00431ECA   |.  8B0E          mov ecx,dword ptr ds:[esi]
00431ECC   |.  8B51 08       mov edx,dword ptr ds:[ecx+8]
00431ECF   |.  398432 B40000> cmp dword ptr ds:[edx+esi+B4],eax
00431ED6   |.^ 0F85 50FFFFFF jnz ZMPK.00431E2C                     ;  如果是错误注册码就向上跳到错误中去
```



#### 代码块 1: ZMPK 模块汇编

```
00431EDC   |>  C705 9C254700>mov dword ptr ds:[47259C],1                ;  走到这里成功  

00431EE6   |.  8B06            mov eax,dword ptr ds:[esi]  

00431EE8   |.  68 38314600     push ZMPK.00463138                        ;  ASCII "1  

1"  

00431EED   |.  68 74354600     push ZMPK.00463574  

00431EF2   |.  8B48 08         mov ecx,dword ptr ds:[eax+8]  

00431EF5   |.  68 18354600     push ZMPK.00463518  

00431EFA   |.  68 0C354600     push ZMPK.0046350C  

00431EFF   |.  8B9431 900B00>  mov edx,dword ptr ds:[ecx+esi+B90]  

00431F06   |.  B9 28244700     mov ecx,ZMPK.00472428  

00431F0B   |.  52              push edx  

00431F0C   |.  E8 2F92FEFF     call ZMPK.0041B140                        ;  显示正确对话框  

00431F11   |.  8B4424 10       mov eax,dword ptr ss:[esp+10]  

00431F15   |.  5F              pop edi  

00431F16   |.  5B              pop ebx  

00431F17   |.  25 FF000000     and eax,0FF  

00431F1C   |.  5E              pop esi  

00431F1D   \.  C2 0400         retn 4  
```

F7 进入关键 `00431E12` `|. FFD7 call edi` 其他错误的 CALL 也可以跟进的，一样的 CALL

#### 代码块 2: USER32 模块汇编

```
77D15F22 U>  55              push ebp  
77D15F23      8BEC            mov ebp,esp  
77D15F25      56              push esi  
77D15F26      8B75 0C         mov esi,dword ptr ss:[ebp+C]  
77D15F29      F7C6 0000FEFF   test esi,FFFE0000  
77D15F2F      0F85 0AFD0200   jnz USER32.77D45C3F  
77D15F35      8B4D 08         mov ecx,dword ptr ss:[ebp+8]  
77D15F38      83F9 FF         cmp ecx,-1  
77D15F3B      0F84 BF320200   je USER32.77D39200  
77D15F41      81F9 FFFF0000   cmp ecx,0FFFF  
77D15F47      0F84 B3320200   je USER32.77D39200  
77D15F4D      E8 D2DAFFFF     call USER32.77D13A24  
77D15F52      85C0            test eax,eax  
77D15F54      0F84 90280100   je USER32.77D287EA  
77D15F5A      6A 01           push 1  
77D15F5C      FF75 14         push dword ptr ss:[ebp+14]  
77D15F5F      FF75 10         push dword ptr ss:[ebp+10]  
77D15F62      56              push esi  
77D15F63      50              push eax  
77D15F64      E8 32F4FFFF     call USER32.77D1539B    F7 进入它，走过之后就计算好了！  

77D15F69      5E              pop esi  
77D15F6A      5D              pop ebp  
77D15F6B      C2 1000         retn 10  
```

`77D15F64 E8 32F4FFFF call USER32.77D1539B`

#### 代码块 3: USER32.77D1539B 详细汇编

```
77D1539B      55              push ebp  
77D1539C      8BEC            mov ebp,esp  
77D1539E      51              push ecx  
77D1539F      51              push ecx  
77D153A0      8065 FF 00      and byte ptr ss:[ebp-1],0  
77D153A4      53              push ebx  
77D153A5      56              push esi  
77D153A6      8B75 08         mov esi,dword ptr ss:[ebp+8]  
77D153A9      8B06            mov eax,dword ptr ds:[esi]  
77D153AB      8065 0B 00      and byte ptr ss:[ebp+B],0  
77D153AF      57              push edi  
77D153B0      8B7D 0C         mov edi,dword ptr ss:[ebp+C]  
77D153B3      81FF E0030000   cmp edi,3E0  
77D153B9      8945 F8         mov dword ptr ss:[ebp-8],eax  
77D153BC      72 0C           jb short USER32.77D153CA  
77D153BE      81FF E8030000   cmp edi,3E8  
77D153C4      0F86 3F150000   jbe USER32.77D16909  
77D153CA      E8 AAEDFFFF     call USER32.77D14179  
77D153CF      3B46 08         cmp eax,dword ptr ds:[esi+8]  
77D153D2      0F85 31150000   jnz USER32.77D16909  
77D153D8      F646 16 04      test byte ptr ds:[esi+16],4  
77D153DC      0F85 27150000   jnz USER32.77D16909  
77D153E2      64:A1 18000000  mov eax,dword ptr fs:[18]  
77D153E8      8B88 E4060000   mov ecx,dword ptr ds:[eax+6E4]  
77D153EE      8B49 0C         mov ecx,dword ptr ds:[ecx+C]  
77D153F1      0B88 F0060000   or ecx,dword ptr ds:[eax+6F0]  
77D153F7      66:F7C1 2020    test cx,2020  
77D153FC      0F85 07150000   jnz USER32.77D16909  
77D15402      33C0            xor eax,eax  
77D15404      8A46 16         mov al,byte ptr ds:[esi+16]  
77D15407      C0E8 03         shr al,3  
77D1540A      F6D0            not al  
77D1540C      33C9            xor ecx,ecx  
77D1540E      33DB            xor ebx,ebx  
77D15410      83E0 01         and eax,1  
77D15413      395D 18         cmp dword ptr ss:[ebp+18],ebx  
77D15416      0F94C1          sete cl  
77D15419      3BC8            cmp ecx,eax  
77D1541B      0F85 92090000   jnz USER32.77D15DB3  
77D15421      8B5E 64         mov ebx,dword ptr ds:[esi+64]  
77D15424      2B5E 10         sub ebx,dword ptr ds:[esi+10]  
77D15427      03DE            add ebx,esi  
77D15429      895D 0C         mov dword ptr ss:[ebp+C],ebx  
77D1542C      E8 D0E5FFFF     call USER32.77D13A01  
77D15431      85C0            test eax,eax  
77D15433      75 0E           jnz short USER32.77D15443  
77D15435      66:8B43 08      mov ax,word ptr ds:[ebx+8]  
77D15439      66:3D A102      cmp ax,2A1  
77D1543D      0F83 95FE0000   jnb USER32.77D252D8  
77D15443      807D FF 00      cmp byte ptr ss:[ebp-1],0  
77D15447      0F85 BC140000   jnz USER32.77D16909  
77D1544D      8D5E 14         lea ebx,dword ptr ds:[esi+14]  
77D15450      6A 01           push 1  
77D15452      53              push ebx  
77D15453      FF75 14         push dword ptr ss:[ebp+14]  
77D15456      FF75 10         push dword ptr ss:[ebp+10]  
77D15459      57              push edi  
77D1545A      FF75 F8         push dword ptr ss:[ebp-8]  
77D1545D      FF76 60         push dword ptr ds:[esi+60]  
77D15460      FFB6 9C000000   push dword ptr ds:[esi+9C]  
77D15466      E8 15E6FFFF     call USER32.77D13A80       F7 进入它，走过之后就计算好了！  

77D1546B      8BC8            mov ecx,eax  
77D1546D      A1 60C0D677     mov eax,dword ptr ds:[77D6C060]  
77D15472      F640 02 04      test byte ptr ds:[eax+2],4  
77D15476      0F85 03DA0200   jnz USER32.77D42E7F  
77D1547C      8BC1            mov eax,ecx  
77D1547E      5F              pop edi  
77D1547F      5E              pop esi  
77D15480      5B              pop ebx  
77D15481      C9              leave  
77D15482      C2 1400         retn 14  
```

进入 `77D15466 E8 15E6FFFF call USER32.77D13A80`

#### 代码块 4: USER32.77D13A80 详细汇编

```
77D13A80      6A 30           push 30  
77D13A82      68 704FD677     push USER32.77D64F70  
77D13A87      E8 1DFFFFFF     call USER32.77D139A9  
77D13A8C      33DB            xor ebx,ebx  
77D13A8E      895D E4         mov dword ptr ss:[ebp-1C],ebx  
77D13A91      64:A1 18000000  mov eax,dword ptr fs:[18]  
77D13A97      8B80 08070000   mov eax,dword ptr ds:[eax+708]  
77D13A9D      3BC3            cmp eax,ebx  
77D13A9F      74 09           je short USER32.77D13AAA  
77D13AA1      F600 04         test byte ptr ds:[eax],4  
77D13AA4      0F85 C95E0000   jnz USER32.77D19973  
77D13AAA      895D E0         mov dword ptr ss:[ebp-20],ebx  
77D13AAD      33F6            xor esi,esi  
77D13AAF      46              inc esi  
77D13AB0      C745 CC 1400000>mov dword ptr ss:[ebp-34],14  
77D13AB7      8975 D0         mov dword ptr ss:[ebp-30],esi  
77D13ABA      33C0            xor eax,eax  
77D13ABC      8D7D D4         lea edi,dword ptr ss:[ebp-2C]  
77D13ABF      AB              stos dword ptr es:[edi]  
77D13AC0      AB              stos dword ptr es:[edi]  
77D13AC1      AB              stos dword ptr es:[edi]  
77D13AC2      895D C8         mov dword ptr ss:[ebp-38],ebx  
77D13AC5      395D E0         cmp dword ptr ss:[ebp-20],ebx  
77D13AC8      75 15           jnz short USER32.77D13ADF  
77D13ACA      FF75 08         push dword ptr ss:[ebp+8]  
77D13ACD      8D45 CC         lea eax,dword ptr ss:[ebp-34]  
77D13AD0      50              push eax  
77D13AD1      FF15 7010D177   call dword ptr ds:[<&ntdll.RtlActivateAc>; ntlld.RtlActivateActivationContextUnsafeFast  
77D13AD7      E8 06FFFFFF     call USER32.77D139E2  
```



```
77D13ADC     8945 C8         mov dword ptr ss:[ebp-38],eax  

77D13ADF     895D FC         mov dword ptr ss:[ebp-4],ebx  

77D13AE2     395D C8         cmp dword ptr ss:[ebp-38],ebx  

77D13AE5     74 1A           je short USER32.77D13B01  

77D13AE7     395D 24        cmp dword ptr ss:[ebp+24],ebx  

77D13AEA     74 15           je short USER32.77D13B01  

77D13AEC     68 D8C1D677     push USER32.77D6C1D8  

77D13AF1     FF75 14         push dword ptr ss:[ebp+14]  

77D13AF4     E8 62050000     call USER32.77D1405B  

77D13AF9     85C0            test eax,eax  

77D13AFB     0F85 186E0000   jnz USER32.77D1A919  

77D13B01     33C0            xor eax,eax  

77D13B03     8945 C4         mov dword ptr ss:[ebp-3C],eax  

77D13B06     8B75 0C         mov esi,dword ptr ss:[ebp+C]  

77D13B09     8975 0C         mov dword ptr ss:[ebp+C],esi  

77D13B0C     3BC3            cmp eax,ebx  

77D13B0E     0F85 A36D0000   jnz USER32.77D1A8B7  

77D13B14     8BCE            mov ecx,esi  

77D13B16     B8 000000C0     mov eax,C0000000  

77D13B1B     23C8            and ecx,eax  

77D13B1D     3BC8            cmp ecx,eax  

77D13B1F     0F84 8CB50300   je USER32.77D4F0B1  

77D13B25     FF75 1C         push dword ptr ss:[ebp+1C]  

77D13B28     FF75 18         push dword ptr ss:[ebp+18]  

77D13B2B     FF75 14         push dword ptr ss:[ebp+14]  

77D13B2E     FF75 10         push dword ptr ss:[ebp+10]  

77D13B31     56              push esi  

77D13B32     E8 16FFFFFF     call USER32.77D13A4D         F7 进入它，走过之后就计算好了！          

77D13B37     8945 E4         mov dword ptr ss:[ebp-1C],eax  

77D13B3A     834D FC FF      or dword ptr ss:[ebp-4],FFFFFFFF  

77D13B3E     E8 0B000000     call USER32.77D13B4E  

77D13B43     8B45 E4         mov eax,dword ptr ss:[ebp-1C]  

77D13B46     E8 4DFEFFFF     call USER32.77D13998  

77D13B4B     C2 2000         retn 20
```

…………………………………………………………………………………………………………………………

进入 `77D13B32` `E8` `16FFFFFF` `call` `USER32.77D13A4D`

………………………………………………………………………………………………………………

```
77D13A4D     55              push ebp  

77D13A4E     8BEC            mov ebp,esp  

77D13A50     56              push esi  

77D13A51     57              push edi  

77D13A52     53              push ebx  

77D13A53     68 CDABBADC     push DCBAABCD  

77D13A58     56              push esi  

77D13A59     FF75 18         push dword ptr ss:[ebp+18]  

77D13A5C     FF75 14         push dword ptr ss:[ebp+14]  

77D13A5F     FF75 10         push dword ptr ss:[ebp+10]  

77D13A62     FF75 0C         push dword ptr ss:[ebp+C]  

77D13A65     FF55 08         call dword ptr ss:[ebp+8]     F7 进入它，发现已经从系统出来了！         

77D13A68     817C24 04 CDABB>cmp dword ptr ss:[esp+4],DCBAABCD  

77D13A70     0F85 CD150300   jnz USER32.77D45043  

77D13A76     83C4 08         add esp,8  

77D13A79     5B              pop ebx  

77D13A7A     5F              pop edi  

77D13A7B     5E              pop esi  

77D13A7C     5D              pop ebp  

77D13A7D     C2 1400         retn 14
```

…………………………………………………………………………………………………………

进入 `77D13A65` `FF55` `08` `call` `dword` `ptr` `ss:[ebp+8]`

………………………………………………………………………………………………………………

```
00427310     .   8B4424 10     mov eax,dword ptr ss:[esp+10]  

00427314     .   8B4C24 0C     mov ecx,dword ptr ss:[esp+C]  

00427318     .   8B5424 08     mov edx,dword ptr ss:[esp+8]  

0042731C     .   50            push eax                         ; /Arg5  

0042731D     .   8B4424 08     mov eax,dword ptr ss:[esp+8]         ; |  

00427321     .   51            push ecx                         ; |Arg4  

00427322     .   8B0D F8274700 mov ecx,dword ptr ds:[4727F8]       ; |  

00427328     .   52            push edx                         ; |Arg3  

00427329     .   50            push eax                         ; |Arg2  

0042732A     .   51            push ecx                         ; |Arg1 => 003D59D0  

0042732B     .   E8 40F6FFFF   call ZMPK.00426970       F7 进入           ; \ZMPK.00426970  

00427330     .   C2 1000       retn 10
```

…………………………………………………………………………………………………………………

F7 进入 `0042732B` `.` `E8` `40F6FFFF` `call` `ZMPK.00426970` 算法了！！

……………………………………………………………………………………………………………………

```
00426970   /$   83EC 08         sub esp,8  

00426973   |.   53              push ebx  

00426974   |.   55              push ebp  

00426975   |.   56              push esi  

00426976   |.   57              push edi  

00426977   |.   C74424 10 000>mov dword ptr ss:[esp+10],0  

0042697F   |.   E8 BC02FEFF     call ZMPK.00406C40  

00426984   |.   8B6C24 24       mov ebp,dword ptr ss:[esp+24]  

00426988   |.   8B7C24 2C       mov edi,dword ptr ss:[esp+2C]  

0042698C   |.   8B7424 28       mov esi,dword ptr ss:[esp+28]  

00426990   |.   8B5C24 1C       mov ebx,dword ptr ss:[esp+1C]  

00426994   |.   81FD 11010000   cmp ebp,111                       ;  Switch (cases 0..111)  

0042699A   |.   894424 14       mov dword ptr ss:[esp+14],eax  

0042699E   |.   0F85 13010000   jnz ZMPK.00426AB7  

004269A4   |.   8B4424 20       mov eax,dword ptr ss:[esp+20]        ;  Case 111 (WM_COMMAND) of switch 00426994  

004269A8   |.   57              push edi  

004269A9   |.   56              push esi  

004269AA   |.   50              push eax  

004269AB   |.   8BCB            mov ecx,ebx  

004269AD   |.   E8 8E270000     call ZMPK.00429140  

004269B2   |.   81FE 33750000   cmp esi,7533  

004269B8   |.   74 14           je short ZMPK.004269CE  

004269BA   |.   81FE 31750000   cmp esi,7531  

004269C0   |.   74 0C           je short ZMPK.004269CE  

004269C2   |.   81FE 32750000   cmp esi,7532  

004269C8   |.   0F85 B7010000   jnz ZMPK.00426B85  

004269CE   |>   E8 6D02FEFF     call ZMPK.00406C40  

004269D3   |.   2B4424 14       sub eax,dword ptr ss:[esp+14]  

004269D7   |.   3D D0070000     cmp eax,7D0  

004269DC   |.   7E 1D           jle short ZMPK.004269FB         这里一定要用 F4 直接来到，不能一步一步  

004269DE   |.   8B4C24 20       mov ecx,dword ptr ss:[esp+20]  

004269E2   |.   6A 00           push 0                           ; /lParam = 0  

004269E4   |.   6A 00           push 0                           ; |wParam = 0  

004269E6   |.   6A 10           push 10                          ; |Message = WM_CLOSE  

004269E8   |.   51              push ecx                         ; |hWnd  

004269E9   |.   FF15 E0B24500   call dword ptr ds:[<&USER32.PostMessageA>; \PostMessageA  

004269EF   |.   5F              pop edi  

004269F0   |.   5E              pop esi  

004269F1   |.   5D              pop ebp  

004269F2   |.   33C0            xor eax,eax  

004269F4   |.   5B              pop ebx  

004269F5   |.   83C4 08         add esp,8  

004269F8   |.   C2 1400         retn 14  

004269FB   |>   81FE 33750000   cmp esi,7533                 如果是参数 7533 就安下面的算法  

00426A01   |.   75 33           jnz short ZMPK.00426A36  

00426A03   |.   8B13            mov edx,dword ptr ds:[ebx]  

00426A05   |.   B9 00CA9A3B     mov ecx,3B9ACA00  

00426A0A   |.   5F              pop edi  

00426A0B   |.   5E              pop esi  

00426A0C   |.   8B42 08         mov eax,dword ptr ds:[edx+8]  

00426A0F   |.   33D2            xor edx,edx  

00426A11   |.   5D              pop ebp  

00426A12   |.   8B8418 B00000>mov eax,dword ptr ds:[eax+ebx+B0]  

00426A19   |.   5B              pop ebx  

00426A1A   |.   05 23057719     add eax,19770523  

00426A1F   |.   35 2D220F33     xor eax,330F222D  

00426A24   |.   F7D0            not eax  

00426A26   |.   69C0 88366387   imul eax,eax,87633688  

00426A2C   |.   F7F1            div ecx  

00426A2E   |.   8BC2            mov eax,edx  

00426A30   |.   83C4 08         add esp,8  

00426A33   |.   C2 1400         retn 14  

00426A36   |>   81FE 32750000   cmp esi,7532                 如果是参数 7532 就安下面的算法  

00426A3C   |.   75 35           jnz short ZMPK.00426A73  

00426A3E   |.   8B13            mov edx,dword ptr ds:[ebx]  

00426A40   |.   B9 00CA9A3B     mov ecx,3B9ACA00  

00426A45   |.   5F              pop edi  

00426A46   |.   5E              pop esi
```



```
00426A47   |.  8B42 08       mov eax,dword ptr ds:[edx+8]  
00426A4A   |.  33D2         xor edx,edx  
00426A4C   |.  5D           pop ebp  
00426A4D   |.  8B8418 B0000>mov eax,dword ptr ds:[eax+ebx+B0]  
00426A54   |.  5B           pop ebx  
00426A55   |.  05 23057719  add eax,19770523  
00426A5A   |.  35 2D220F33  xor eax,330F222D  
00426A5F   |.  F7D0         not eax  
00426A61   |.  69C0 33321387 imul eax,eax,87133233  
00426A67   |.  F7F1         div ecx  
00426A69   |.  8BC2         mov eax,edx  
00426A6B   |.  03C1         add eax,ecx  
00426A6D   |.  83C4 08      add esp,8  
00426A70   |.  C2 1400      retn 14  
00426A73   |>  81FE 31750000 cmp esi,7531                 如果是参数 7531 就按下面的算法（正确的）  
00426A79   |.  0F85 06010000 jnz ZMPK.00426B85  
00426A7F   |.  8B13         mov edx,dword ptr ds:[ebx]  
00426A81   |.  B9 00CA9A3B  mov ecx,3B9ACA00               用于计算的参数 1000000000  
00426A86   |.  5F           pop edi  
00426A87   |.  5E           pop esi  
00426A88   |.  8B42 08      mov eax,dword ptr ds:[edx+8]   指针用于得到机器码  
00426A8B   |.  33D2         xor edx,edx                    清零  
00426A8D   |.  5D           pop ebp  
00426A8E   |.  8B8418 B0000>mov eax,dword ptr ds:[eax+ebx+B0]   得到我的机器码 1010405983  
00426A95   |.  5B           pop ebx  
00426A96   |.  05 23057719  add eax,19770523               机器码加上 19770523（作者的生日？）  
00426A9B   |.  35 2D220F33  xor eax,330F222D               结果在与 330F222D 异或  
00426AA0   |.  F7D0         not eax                        结果取反  
00426AA2   |.  69C0 66922987 imul eax,eax,87299266         结果与 87299266 相乘  
00426AA8   |.  F7F1         div ecx                        结果除以 1000000000  
00426AAA   |.  8BC2         mov eax,edx                    取它的余数  
00426AAC   |.  05 00943577  add eax,77359400               余数与 77359400 相加就是真正的注册码！  
00426AB1   |.  83C4 08      add esp,8  
00426AB4   |.  C2 1400      retn 14  
```

### 算法总结

得到机器码加上 `19770523` 的结果，再与 `330F222D` 异或，把结果取反，取反的结果与 `87299266` 相乘。

结果除以 `1000000000` 取它的余数，余数与 `77359400` 相加就是真正的注册码。

算法写出来了，注册机你自己搞定吧。

我的机器码：`1010405983`

我的注册码：`2503242208`

### 明码比较

也可以作内存注册机！

```
4dd466d0e1e0cc123afbe504da2462ca
```
