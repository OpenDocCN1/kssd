
# 销售王客户管理软件 单机多用户版 2005.01 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-255.htm`](https://www.pediy.com/kssd/pediy07/pediy7-255.htm)

- **【破文标题】**：销售王客户管理软件 单机多用户版 2005.01 算法分析
- **【破文作者】**：`KuNgBiM[DFCG]`
- **【作者邮箱】**：`gb_1227@163.com`
- **【保护方式】**：注册码+功能限制+启动注册提示
- **【编译语言】**：`Borland Delphi 6.0 - 7.0`
- **【调试环境】**：`WinXP`、`PEiD`、`W32Dasm`、`Ollydbg`
- **【破解日期】**：`2005-05-28`
- **【破解目的】**：研究算法分析
- **【作者声明】**：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 【破解过程】

### 侦测
用 `PEiD` 查壳，无壳，`Borland Delphi 6.0 - 7.0` 编译。

### 试探
运行主程序注册，输入试炼信息，确认！程序提示"注册码错误!"

### 初步下药
使出法宝，用 `W32Dasm` 进行静态反汇编，查找"注册码错误!"字符串，找到 `006583E3` 处！

### 对症下药
`Ollydbg` 载入主程序，加载完毕后，找到 `006583E3` 处！

向上来到 `006582A7` 处下断，`F9` 运行，输入注册信息：

```
***** 试炼信息 *****
机器码：3HSCCTSM
用户名：KuNgBiM[DFCG]
注册码：9876543210
********************************
```

点击确定后 `OD` 断下:（这里我采用的是 `W32Dasm` 的反汇编信息，比较干净清楚！）

```
:006582A7 8B83F8020000        mov eax, dword ptr [ebx+000002F8]

:006582AD E83E44E0FF         call 0045C6F0                          //取用户名

:006582B2 8B45F8             mov eax, dword ptr [ebp-08]            //把取得的用户名转存给 EAX

:006582B5 8D55FC             lea edx, dword ptr [ebp-04]

:006582B8 E82F12DBFF         call 004094EC

:006582BD 837DFC00           cmp dword ptr [ebp-04], 00000000      //用户名是否为空

:006582C1 741F               je 006582E2                            //为 0 则跳死！

:006582C3 8D55F0             lea edx, dword ptr [ebp-10]

:006582C6 8B83FC020000       mov eax, dword ptr [ebx+000002FC]

:006582CC E81F44E0FF         call 0045C6F0                          //取试炼码

:006582D1 8B45F0             mov eax, dword ptr [ebp-10]            //把取得的试炼码转存给 EAX

:006582D4 8D55F4             lea edx, dword ptr [ebp-0C]

:006582D7 E81012DBFF         call 004094EC

:006582DC 837DF400           cmp dword ptr [ebp-0C], 00000000      //试炼码是否为空

:006582E0 751D               jne 006582FF                           //为 0 则跳死！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:006582C1(C)
|

:006582E2 6A40               push 00000040

* Possible StringData Ref from Code Obj ->"信息提示"
                          |

:006582E4 B980846500         mov ecx, 00658480

* Possible StringData Ref from Code Obj ->"用户名和序列号不能为空!"      //注册信息输入非法的提示
                          |

:006582E9 BA8C846500         mov edx, 0065848C

:006582EE A168A57000         mov eax, dword ptr [0070A568]

:006582F3 8B00               mov eax, dword ptr [eax]

:006582F5 E8E2D9DFFF         call 00455CDC

:006582FA E9F5000000         jmp 006583F4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:006582E0(C)
|

:006582FF 8D45EC             lea eax, dword ptr [ebp-14]

:00658302 50                 push eax

:00658303 8D55E8             lea edx, dword ptr [ebp-18]

:00658306 8B83F4020000       mov eax, dword ptr [ebx+000002F4]

:0065830C E8DF43E0FF         call 0045C6F0                          //取机器码

:00658311 8B45E8             mov eax, dword ptr [ebp-18]            //把取得的机器码转存给 EAX

:00658314 50                 push eax                               //机器码压栈

:00658315 8D55E0             lea edx, dword ptr [ebp-20]

:00658318 8B83F8020000       mov eax, dword ptr [ebx+000002F8]

:0065831E E8CD43E0FF         call 0045C6F0                          //取用户名

:00658323 8B45E0             mov eax, dword ptr [ebp-20]            //把取得的用户名转存给 EAX

:00658326 8D55E4             lea edx, dword ptr [ebp-1C]

:00658329 E8BE11DBFF         call 004094EC

:0065832E 8B55E4             mov edx, dword ptr [ebp-1C]

:00658331 A1B09D7000         mov eax, dword ptr [00709DB0]

:00658336 8B00               mov eax, dword ptr [eax]

:00658338 8B80C4000000       mov eax, dword ptr [eax+000000C4]

:0065833E 59                 pop ecx

:0065833F E8D0A7F0FF         call 00562B14                          //算法 CALL，F7 跟进！

:00658344 8B45EC             mov eax, dword ptr [ebp-14]            //把真码赋值给 EAX

:00658347 50                 push eax                               //真码压栈，"4F646F5F5F706F691C1C1C1C62"

:00658348 8D55D8             lea edx, dword ptr [ebp-28]            //取用户名，转存给 EDX

:0065834B 8B83FC020000       mov eax, dword ptr [ebx+000002FC]

:00658351 E89A43E0FF         call 0045C6F0                          //取试炼码位数

:00658356 8B45D8             mov eax, dword ptr [ebp-28]            //试炼码赋值给 EAX，EAX=0A

:00658359 8D55DC             lea edx, dword ptr [ebp-24]

:0065835C E88B11DBFF         call 004094EC                          //取试炼码，真码

:00658361 8B55DC             mov edx, dword ptr [ebp-24]            //假码赋值给 EDX，ASCII "9876543210"

:00658364 58                 pop eax                                //真码赋值给 EAX，ASCII "4F646F5F5F706F691C1C1C1C62"

:00658365 E852C9DAFF         call 00404CBC                          //比对 CALL（经典）

:0065836A 7570               jne 006583DC                           //爆破点

:0065836C 8D55D0             lea edx, dword ptr [ebp-30]

:0065836F 8B83FC020000       mov eax, dword ptr [ebx+000002FC]

:00658375 E87643E0FF         call 0045C6F0

:0065837A 8B45D0             mov eax, dword ptr [ebp-30]

:0065837D 8D55D4             lea edx, dword ptr [ebp-2C]

:00658380 E86711DBFF         call 004094EC

:00658385 8B45D4             mov eax, dword ptr [ebp-2C]

:00658388 50                 push eax

:00658389 8D55C8             lea edx, dword ptr [ebp-38]

:0065838C 8B83F8020000       mov eax, dword ptr [ebx+000002F8]

:00658392 E85943E0FF         call 0045C6F0

:00658397 8B45C8             mov eax, dword ptr [ebp-38]

:0065839A 8D55CC             lea edx, dword ptr [ebp-34]

:0065839D E84A11DBFF         call 004094EC

:006583A2 8B55CC             mov edx, dword ptr [ebp-34]

:006583A5 A1B09D7000         mov eax, dword ptr [00709DB0]

:006583AA 8B00               mov eax, dword ptr [eax]

:006583AC 8B80C4000000       mov eax, dword ptr [eax+000000C4]

:006583B2 59                 pop ecx

:006583B3 E84494F0FF         call 005617FC

:006583B8 6A40               push 00000040

* Possible StringData Ref from Code Obj ->"信息提示"
                          |

:006583BA B980846500         mov ecx, 00658480

* Possible StringData Ref from Code Obj ->"注册成功,请重新启动程序!"      //注册信息输入正确的提示
                          |

:006583BF BAA4846500         mov edx, 006584A4

:006583C4 A168A57000         mov eax, dword ptr [0070A568]

:006583C9 8B00               mov eax, dword ptr [eax]

:006583CB E80CD9DFFF         call 00455CDC

:006583D0 C7834C02000001000000 mov dword ptr [ebx+0000024C], 00000001

:006583DA EB18               jmp 006583F4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0065836A(C)
|

:006583DC 6A40               push 00000040

* Possible StringData Ref from Code Obj ->"信息提示"
                          |

:006583DE B980846500         mov ecx, 00658480

* Possible StringData Ref from Code Obj ->"注册码错误!"                //注册失败的信息提示
                          |

:006583E3 BAC0846500         mov edx, 006584C0

:006583E8 A168A57000         mov eax, dword ptr [0070A568]

:006583ED 8B00               mov eax, dword ptr [eax]

:006583EF E8E8D8DFFF         call 00455CDC
```

`=========== 跟进：0065833F E8D0A7F0FF call 00562B14 ===========`

`00562B14 55 push ebp`



```
00562B15    8BEC            mov ebp,esp  
00562B17    83C4 F4         add esp,-0C  
00562B1A    53              push ebx  
00562B1B    56              push esi  
00562B1C    57              push edi  
00562B1D    33DB            xor ebx,ebx  
00562B1F    895D F4         mov dword ptr ss:[ebp-C],ebx  
00562B22    894D F8         mov dword ptr ss:[ebp-8],ecx                //提取机器码，ASCII "3HSCCTSM"  
00562B25    8955 FC         mov dword ptr ss:[ebp-4],edx                //提取用户名，ASCII "KuNgBiM[DFCG]"  
00562B28    8B75 08         mov esi,dword ptr ss:[ebp+8]  
00562B2B    8B45 FC         mov eax,dword ptr ss:[ebp-4]  
00562B2E    E8 2D22EAFF     call PersonMU.00404D60                      //同时取机器码和用户名  
00562B33    8B45 F8         mov eax,dword ptr ss:[ebp-8]                //全部入栈，准备开始计算  
00562B36    E8 2522EAFF     call PersonMU.00404D60  
00562B3B    33C0            xor eax,eax                                   //计数器清零  
00562B3D    55              push ebp  
00562B3E    68 A42B5600     push PersonMU.00562BA4  
00562B43    64:FF30         push dword ptr fs:[eax]  
00562B46    64:8920         mov dword ptr fs:[eax],esp  
00562B49    8BC6            mov eax,esi  
00562B4B    E8 601DEAFF     call PersonMU.004048B0                      //提取用户名，ASCII "KuNgBiM[DFCG]"  
00562B50    8B45 FC         mov eax,dword ptr ss:[ebp-4]                //把取得的用户名转存给 EAX  
00562B53    E8 2020EAFF     call PersonMU.00404B78                      //计算用户名的位数  
00562B58    8BD8            mov ebx,eax                                   //用户名为 13 位，EAX 赋值给 EBX，EAX=0D  
00562B5A    85DB            test ebx,ebx                                  //EBX=0D  
00562B5C    7E 2B           jle short PersonMU.00562B89  
00562B5E    BF 01000000     mov edi,1                                     //EDI=6  
00562B63    8B45 F8         mov eax,dword ptr ss:[ebp-8]                //开始取机器码的 HEX 值，并且计算 13 次  
00562B66    0FB64438 FF     movzx eax,byte ptr ds:[eax+edi-1]           //33 ('3')  
                                                                          //48 ('H')  
                                                                          //53 ('S')  
                                                                          //43 ('C')  
                                                                          //43 ('C')  
                                                                          //54 ('T')  
                                                                          //53 ('S')  
                                                                          //4D ('M')  
                                                                          //00 ('')  
                                                                          //00 ('')  
                                                                          //00 ('')  
                                                                          //00 ('')  
                                                                          //46 ('F')  <---随机数  
00562B6B    83C0 1C         add eax,1C                                    //把求得的每位机器码 HEX 值都加上 1C  
00562B6E    8D4D F4         lea ecx,dword ptr ss:[ebp-C]  
00562B71    BA 02000000     mov edx,2                                     //edx=013F9496, (ASCII "69")  
                                                                          //edx=013F9498, (ASCII "1C")  
                                                                          //edx=013F949A, (ASCII "1C")  
                                                                          //edx=013F949C, (ASCII "1C")  
                                                                          //edx=013F949E, (ASCII "1C")  
00562B76    E8 B96DEAFF     call PersonMU.00409934  
00562B7B    8B55 F4         mov edx,dword ptr ss:[ebp-C]                //4F ('3'+1C)  
                                                                          //64 ('H'+1C)  
                                                                          //6F ('S'+1C)  
                                                                          //5F ('C'+1C)  
                                                                          //5F ('C'+1C)  
                                                                          //70 ('T'+1C)  
                                                                          //6F ('S'+1C)  
                                                                          //69 ('M'+1C)  
                                                                          //1C (''+1C)  
                                                                          //1C (''+1C)  
                                                                          //1C (''+1C)  
                                                                          //1C (''+1C)  
                                                                          //62 ('F'+1C)  <---随机数  
00562B7E    8BC6            mov eax,esi  
00562B80    E8 FB1FEAFF     call PersonMU.00404B80  
00562B85    47              inc edi                                       //EDI 自加 1，指向下一位  
00562B86    4B              dec ebx                                       //EBX 自减 1，运算次数减少 1 次  
00562B87  ^ 75 DA           jnz short PersonMU.00562B63                   //向上作循环运算  
00562B89    33C0            xor eax,eax                                   //EAX=2  
00562B8B    5A              pop edx  
00562B8C    59              pop ecx  
00562B8D    59              pop ecx  
00562B8E    64:8910         mov dword ptr fs:[eax],edx  
00562B91    68 AB2B5600     push PersonMU.00562BAB  
00562B96    8D45 F4         lea eax,dword ptr ss:[ebp-C]  
00562B99    BA 03000000     mov edx,3  
00562B9E    E8 311DEAFF     call PersonMU.004048D4  
00562BA3    C3              retn  
00562BA4  ^ E9 AB15EAFF     jmp PersonMU.00404154  
00562BA9  ^ EB EB           jmp short PersonMU.00562B96  
00562BAB    5F              pop edi                                       //EDI=0E  
00562BAC    5E              pop esi  
00562BAD    5B              pop ebx  
00562BAE    8BE5            mov esp,ebp  
00562BB0    5D              pop ebp  
00562BB1    C2 0400         retn 4                                        //计算完毕，返回  
```

---

## 【算法总结】

注册码与机器码和用户名位数有关：

1. 取用户名的位数作为机器码运算的次数（用户名位数最好小于等于 12 位，否则计算出的注册码视为非法注册码）
2. 再计算用户名位数所对应的机器码的 `HEX` 值，并且每一位机器码的 `HEX` 值加上 16 进制数 `1C` 组合成新的 `HEX` 值字符串，作为注册码

例如：（本机计算信息）

```
机器码：（3HSCCTSM）      334853434354534D
用户名：（KuNgBiM）（7 位）33485343435453
运算码：                  1C1C1C1C1C1C1C
注册码：                  4F646F5F5F706F
```

## 【注册信息】

### 【合法注册码】

机器码：`3HSCCTSM`  
用户名：`KuNgBiM`  
注册码：`4F646F5F5F706F`

### 【非法注册码】

机器码：`3HSCCTSM`  
用户名：`KuNgBiM[DFCG]`  
注册码：`4F646F5F5F706F691C1C1C1C62`

---

〓本文完〓

---

版权所有（C）2005 `KuNgBiM[DFCG]`   Copyright (C) 2005 `KuNgBiM[DFCG]`

---

   Cracked By `KuNgBiM[DFCG]`  
         2005-05-28  
         16:18:18 PM

`4dd466d0e1e0cc123afbe504da2462ca`
