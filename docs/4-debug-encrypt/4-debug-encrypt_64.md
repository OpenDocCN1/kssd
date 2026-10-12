
# 网络叔叔 1.0 简单 MD5 注册算法分析 + VB 注册机

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-305.htm`](https://www.pediy.com/kssd/pediy07/pediy7-305.htm)

## 基本信息

- **破文标题**：网络叔叔 1.0 简单 MD5 注册算法分析 + VB 注册机
- **破文作者**：KuNgBiM\[DFCG\]
- **作者邮箱**：gb\_1227@163.com
- **软件名称**：网络叔叔 1.0
- **软件大小**：2099 KB
- **软件类别**：国产软件 / 共享版 / 网络监测
- **整理时间**：2005-07-04
- **开发商**：[`www.netuncle.com`](http://www.netuncle.com)
- **下载地址**：[`www.shareware.cn/pub/12419.html`](http://www.shareware.cn/pub/12419.html)

## 软件简介

只要在一台机器上安装网络叔叔，便可以监控记录全网机器上网网址、收发邮件内容及可种网络活动。用户可规定那些网址可以上，那些网址阻拦，那些邮箱可以收发邮件，那些邮箱阻拦及各种网络活动如（聊天、游戏等）的完全控制，适用于家庭、公司、企业、学校等用户。

网络叔叔是规划网络的好帮手！

- 精确控制上网时间，上网网站，禁止搜指定不良信息。
- 精确控制邮件收发，指定可收发邮箱，不可收发邮箱。
- 精确控制邮件主题关键字。
- 精确控制 `TCP` 协议。
- 记录上网网址，收发邮件内容，`FTP` 命令，`TELNET` 命令等。

家庭用户使用网络叔叔，防止小孩受不良网站、游戏、聊天的影响。

公司用户使用网络叔叔，合理利用网络，提高工作效率，免受游戏、邮件、聊天的影响、防止公司机密外泄。

学校用户使用网络叔叔，提高学习效率，免受不良信息。

## 破解环境

- **保护方式**：注册码 + 启动 NAG + 功能限制
- **编译语言**：Microsoft Visual C++ 6.0
- **调试环境**：`WinXP`、`PEiD`、`W32Dasm`、`Ollydbg`
- **破解日期**：2005-07-04
- **破解目的**：研究算法分析
- **作者声明**：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 破解过程

1.  **侦测**：用 `PEiD` 查壳，无壳，`Microsoft Visual C++ 6.0` 编译。
2.  **试探**：运行主程序注册，输入试炼码，确认！程序提示："注册信息不正确！"
3.  **初步下药**：使出法宝，用 `W32Dasm` 进行静态反汇编，查找" 注册信息不正确！"字符串，结果找到 `00408109` 处，向上来到 `00407E50` 处。
4.  **对症下药**：`Ollydbg` 载入主程序，来到 `00407E50` 处下断，一路 `F9` 运行，直到程序运行，先取消注册，然后再手动点击选项，输入试炼信息：

```
******** 试炼信息 ********

注册码：1cb7fbd5
监控机器上限：999999999
序列号：7878787878787878
************************
```

```
00407E50    6A FF            push -1
00407E52    68 C89B4500      push NetManag.00459BC8
00407E57    64:A1 00000000   mov eax,dword ptr fs:[0]
00407E5D    50              push eax
00407E5E    64:8925 0000000>mov dword ptr fs:[0],esp
00407E65    81EC A0020000    sub esp,2A0
00407E6B    53              push ebx
00407E6C    55              push ebp
00407E6D    56              push esi
00407E6E    57              push edi
00407E6F    8BE9            mov ebp,ecx
00407E71    6A 00           push 0
00407E73    6A 00           push 0
00407E75    68 04484700     push NetManag.00474804
00407E7A    8D8C24 14010000 lea ecx,dword ptr ss:[esp+114]
00407E81    E8 B1770400     call NetManag.0044F637
00407E86    8D8C24 90010000 lea ecx,dword ptr ss:[esp+190]
00407E8D    C78424 B802000>mov dword ptr ss:[esp+2B8],0
00407E98    E8 63320100     call NetManag.0041B100
00407E9D    8D4C24 24       lea ecx,dword ptr ss:[esp+24]
00407EA1    C68424 B802000>mov byte ptr ss:[esp+2B8],1
00407EA9    E8 122A0100     call NetManag.0041A8C0
00407EAE    8D8424 90010000 lea eax,dword ptr ss:[esp+190]
00407EB5    8D8C24 08010000 lea ecx,dword ptr ss:[esp+108]
00407EBC    50              push eax
00407EBD    C68424 BC02000>mov byte ptr ss:[esp+2BC],2
00407EC5    E8 B97B0400     call NetManag.0044FA83
00407ECA    8D4C24 24       lea ecx,dword ptr ss:[esp+24]
00407ECE    51              push ecx
00407ECF    8D8C24 0C010000 lea ecx,dword ptr ss:[esp+10C]
00407ED6    E8 A87B0400     call NetManag.0044FA83                ; 检测系统是否已注册
00407EDB    A1 D4C84700     mov eax,dword ptr ds:[47C8D4]         ; 取寄存器 eax 中的值
00407EE0    85C0            test eax,eax                          ; 为空则继续
00407EE2    74 4D           je short NetManag.00407F31            ; 如果程序已注册则跳走！
00407EE4    68 F8474700     push NetManag.004747F8
00407EE9    8D8C24 04010000 lea ecx,dword ptr ss:[esp+104]
00407EF0    E8 474D0300     call NetManag.0043CC3C
00407EF5    68 DCC84700     push NetManag.0047C8DC
00407EFA    8D8C24 00010000 lea ecx,dword ptr ss:[esp+100]
00407F01    E8 E64C0300     call NetManag.0043CBEC
00407F06    68 D8C84700     push NetManag.0047C8D8
00407F0B    8D8C24 FC000000 lea ecx,dword ptr ss:[esp+FC]
00407F12    E8 D54C0300     call NetManag.0043CBEC
00407F17    8B15 E0C84700   mov edx,dword ptr ds:[47C8E0]
00407F1D    C78424 B0000000>mov dword ptr ss:[esp+B0],1
00407F28    899424 04010000 mov dword ptr ss:[esp+104],edx
00407F2F    EB 2D           jmp short NetManag.00407F5E
00407F31    68 F0C74700     push NetManag.0047C7F0
00407F36    8D8C24 00010000 lea ecx,dword ptr ss:[esp+100]
00407F3D    E8 FA4C0300     call NetManag.0043CC3C
00407F42    68 D8C84700     push NetManag.0047C8D8
00407F47    8D8C24 FC000000 lea ecx,dword ptr ss:[esp+FC]
00407F4E    E8 994C0300     call NetManag.0043CBEC                ; 赋予初始值
00407F53    C78424 04010000>mov dword ptr ss:[esp+104],5          ; 向 esp+104 中赋值 5
00407F5E    8D8C24 08010000 lea ecx,dword ptr ss:[esp+108]
00407F65    E8 11790400     call NetManag.0044F87B                ; F9 到这里程序就运行了
00407F6A    83F8 01         cmp eax,1                             ; 取消以上的断点，然后这里下断
                                                                  ; 比较“监控机器上限”数量是否小于 1
00407F6D    0F85 14020000   jnz NetManag.00408187                 ; 小于 1 则直接跳死
00407F73    A1 34634700     mov eax,dword ptr ds:[476334]         ; eax=1
00407F78    894424 10       mov dword ptr ss:[esp+10],eax
00407F7C    8B0D D4C84700   mov ecx,dword ptr ds:[47C8D4]
00407F82    C68424 B802000>mov byte ptr ss:[esp+2B8],3
00407F8A    85C9            test ecx,ecx
00407F8C    74 27           je short NetManag.00407FB5
00407F8E    8B8424 B4000000 mov eax,dword ptr ss:[esp+B4]
00407F95    8B8C24 F4000000 mov ecx,dword ptr ss:[esp+F4]
00407F9C    50              push eax
00407F9D    51              push ecx
00407F9E    8D5424 18       lea edx,dword ptr ss:[esp+18]
00407FA2    68 CC474700     push NetManag.004747CC
00407FA7    52              push edx
00407FA8    E8 41DC0200     call NetManag.00435BEE
00407FAD    83C4 10         add esp,10
00407FB0    E9 A6010000     jmp NetManag.0040815B
00407FB5    894424 14       mov dword ptr ss:[esp+14],eax
00407FB9    8B8424 04010000 mov eax,dword ptr ss:[esp+104]       ; 取“监控机器上限”数的 ASCII 值,堆栈 ss:[0012F544]=3B9AC9FF
00407FC0    8D4C24 14       lea ecx,dword ptr ss:[esp+14]         ; ecx 清零
00407FC4    50              push eax                              ;“监控机器上限”数的 ASCII 值压栈，eax=3B9AC9FF
00407FC5    68 B0454700     push NetManag.004745B0                ; ASCII "%d"
00407FCA    51              push ecx
00407FCB    C68424 C402000>mov byte ptr ss:[esp+2C4],4
00407FD3    E8 16DC0200     call NetManag.00435BEE                ; 取“监控机器上限”数值是否合法
00407FD8    83C4 0C         add esp,0C
00407FDB    8D5424 14       lea edx,dword ptr ss:[esp+14]         ; edx=071509B9, (ASCII "99999999")
00407FDF    8D4424 1C       lea eax,dword ptr ss:[esp+1C]         ; 取“监控机器上限”字符长度，eax=9
00407FE3    52              push edx
00407FE4    68 D8C84700     push NetManag.0047C8D8
00407FE9    50              push eax
00407FEA    E8 F34C0300     call NetManag.0043CCE2
00407FEF    50              push eax
00407FF0    8D4C24 18       lea ecx,dword ptr ss:[esp+18]
00407FF4    C68424 BC02000>mov byte ptr ss:[esp+2BC],5
00407FFC    E8 EB4B0300     call NetManag.0043CBEC                ; F7 跟进，程序连接“注册码”和“监控机器上限”
```

========================= 跟进 `00407FFC E8 EB4B0300 call NetManag.0043CBEC` =========================

```
0043CBEC    56              push esi
0043CBED    57              push edi
0043CBEE    8B7C24 0C       mov edi,dword ptr ss:[esp+C]
0043CBF2    8BF1            mov esi,ecx
0043CBF4    8B0E            mov ecx,dword ptr ds:[esi]
0043CBF6    8B07            mov eax,dword ptr ds:[edi]            ; ASCII "1cb7fbd5999999999"
0043CBF8    3BC8            cmp ecx,eax
0043CBFA    74 39           je short NetManag.0043CC35
0043CBFC    8379 F4 00      cmp dword ptr ds:[ecx-C],0
0043CC00    7D 0B           jge short NetManag.0043CC0D
0043CC02    83C1 F4         add ecx,-0C
0043CC05    3B0D 38634700   cmp ecx,dword ptr ds:[476338]         ; NetManag.0047633C
```



```
0043CC0B    75 06           jnz short NetManag.0043CC13  
0043CC0D    8378 F4 00      cmp dword ptr ds:[eax-C],0  
0043CC11    7D 0D           jge short NetManag.0043CC20  
0043CC13    50              push eax  
0043CC14    8BCE            mov ecx,esi  
0043CC16    FF70 F8         push dword ptr ds:[eax-8]  
0043CC19    E8 A1FFFFFF     call NetManag.0043CBBF  
0043CC1E    EB 15           jmp short NetManag.0043CC35  
0043CC20    8BCE            mov ecx,esi  
0043CC22    E8 0FFEFFFF     call NetManag.0043CA36  
0043CC27    8B07            mov eax,dword ptr ds:[edi]          ; ASCII "1cb7fbd5999999999"  
0043CC29    8906            mov dword ptr ds:[esi],eax          ; eax=07150B98, (ASCII "1cb7fbd5999999999")  
0043CC2B    83C0 F4         add eax,-0C  
0043CC2E    50              push eax  
0043CC2F    FF15 80F34500   call dword ptr ds:[<&KERNEL32.InterlockedIncrement>; kernel32.InterlockedIncrement  
0043CC35    8BC6            mov eax,esi  
0043CC37    5F              pop edi  
0043CC38    5E              pop esi  
0043CC39    C2 0400         retn 4                              ; 连接完毕返回程序  
```

\============================================================================================================

```
00408001    8D4C24 1C       lea ecx,dword ptr ss:[esp+1C]       ; ecx=07150B8C  
00408005    C68424 B8020000>mov byte ptr ss:[esp+2B8],4  
0040800D    E8 ED4A0300     call NetManag.0043CAFF              ; 检测连接后字符串的合法性  
00408012    8D4C24 14       lea ecx,dword ptr ss:[esp+14]  
00408016    8D5424 18       lea edx,dword ptr ss:[esp+18]  
0040801A    51              push ecx  
0040801B    52              push edx  
0040801C    E8 9F030000     call NetManag.004083C0  
00408021    83C4 08         add esp,8  
00408024    6A 10           push 10  
00408026    8D4424 20       lea eax,dword ptr ss:[esp+20]  
0040802A    6A 06           push 6  
0040802C    50              push eax  
0040802D    8D4C24 24       lea ecx,dword ptr ss:[esp+24]  
00408031    C68424 C4020000>mov byte ptr ss:[esp+2C4],6  
00408039    E8 7ED60200     call NetManag.004356BC              ; 算法 CALL1，F7 跟进！  
```

### 跟进 `NetManag.004356BC`

```
004356BC    B8 17DC4500     mov eax,NetManag.0045DC17  
004356C1    E8 4235FFFF     call NetManag.00428C08  
004356C6    51              push ecx  
004356C7    8B55 0C         mov edx,dword ptr ss:[ebp+C]  
004356CA    8365 F0 00      and dword ptr ss:[ebp-10],0  
004356CE    85D2            test edx,edx  
004356D0    7D 02           jge short NetManag.004356D4  
004356D2    33D2            xor edx,edx  
004356D4    8B45 10         mov eax,dword ptr ss:[ebp+10]  
004356D7    85C0            test eax,eax  
004356D9    7D 02           jge short NetManag.004356DD  
004356DB    33C0            xor eax,eax  
004356DD    56              push esi  
004356DE    8B31            mov esi,dword ptr ds:[ecx]          ; (ASCII "6ab601a68849197afe5b20a21e217ba8")  
                                                                ; 这里就是连接后字符串的标准 MD5 值  
004356E0    57              push edi  
004356E1    8D3C02          lea edi,dword ptr ds:[edx+eax]      ; 地址=00000016，edi=009F4C88  
004356E4    8B76 F8         mov esi,dword ptr ds:[esi-8]  
004356E7    3BFE            cmp edi,esi                         ; 比较 edi 和 esi 的值，esi=20，edi=16  
004356E9    7E 04           jle short NetManag.004356EF         ; edi 的值小于 esi 的值则跳，必需跳！  
004356EB    8BC6            mov eax,esi  
004356ED    2BC2            sub eax,edx  
004356EF    3BD6            cmp edx,esi                         ; 保险起见，再次比较 edi 和 esi 的值，esi=20，edi=16  
004356F1    7E 02           jle short NetManag.004356F5         ; edi 的值小于 esi 的值则跳，必需跳！  
004356F3    33C0            xor eax,eax  
004356F5    85D2            test edx,edx                        ; edx=6  
004356F7    75 0F           jnz short NetManag.00435708  
004356F9    3BC6            cmp eax,esi  
004356FB    75 0B           jnz short NetManag.00435708  
004356FD    51              push ecx  
004356FE    8B4D 08         mov ecx,dword ptr ss:[ebp+8]  
00435701    E8 6E710000     call NetManag.0043C874  
00435706    EB 37           jmp short NetManag.0043573F  
00435708    8B35 34634700   mov esi,dword ptr ds:[476334]      ; NetManag.00476348  
0043570E    8975 0C         mov dword ptr ss:[ebp+C],esi  
00435711    6A 01           push 1  
00435713    5E              pop esi  
00435714    6A 00           push 0  
00435716    52              push edx  
00435717    50              push eax  
00435718    8D45 0C         lea eax,dword ptr ss:[ebp+C]  
0043571B    50              push eax  
0043571C    8975 FC         mov dword ptr ss:[ebp-4],esi  
0043571F    E8 05740000     call NetManag.0043CB29              ; 算法 CALL2，F7 跟进！  
```

### 跟进 `NetManag.0043CB29`

```
0043CB29    55              push ebp  
0043CB2A    8BEC            mov ebp,esp  
0043CB2C    8B45 14         mov eax,dword ptr ss:[ebp+14]  
0043CB2F    53              push ebx  
0043CB30    8B5D 0C         mov ebx,dword ptr ss:[ebp+C]  
0043CB33    57              push edi  
0043CB34    03C3            add eax,ebx  
0043CB36    8BF9            mov edi,ecx  
0043CB38    85C0            test eax,eax  
0043CB3A    75 0D           jnz short NetManag.0043CB49  
0043CB3C    8B45 08         mov eax,dword ptr ss:[ebp+8]  
0043CB3F    8B0D 34634700   mov ecx,dword ptr ds:[476334]      ; NetManag.00476348  
0043CB45    8908            mov dword ptr ds:[eax],ecx  
0043CB47    EB 1E           jmp short NetManag.0043CB67  
0043CB49    56              push esi  
0043CB4A    8B75 08         mov esi,dword ptr ss:[ebp+8]  
0043CB4D    50              push eax  
0043CB4E    8BCE            mov ecx,esi  
0043CB50    E8 17FEFFFF     call NetManag.0043C96C  
0043CB55    8B07            mov eax,dword ptr ds:[edi]  
0043CB57    53              push ebx  
0043CB58    0345 10         add eax,dword ptr ss:[ebp+10]      ; 从第 6 位后开始取，数据为新的 MD5 值中的字符  
0043CB5B    50              push eax                            ; eax=07150D2E, (ASCII "a68849197afe5b20a21e217ba8")  
0043CB5C    FF36            push dword ptr ds:[esi]  
0043CB5E    E8 5DC2FEFF     call NetManag.00428DC0  
0043CB63    83C4 0C         add esp,0C  
0043CB66    5E              pop esi  
0043CB67    5F              pop edi                             ; 读去 edi 中的值，连续取 16 位，edi=16  
0043CB68    5B              pop ebx  
0043CB69    5D              pop ebp  
0043CB6A    C2 1000         retn 10                             ; 返回算法 CALL1  
```

\============================================================================================================

```
00435724    8B4D 08         mov ecx,dword ptr ss:[ebp+8]        ; 返回到这里  
00435727    8D45 0C         lea eax,dword ptr ss:[ebp+C]        ; 真序列号出现，(ASCII "a68849197afe5b20")  
0043572A    50              push eax                            ; 数据压栈给 eax，以待以后比较用  
0043572B    E8 44710000     call NetManag.0043C874  
00435730    8065 FC 00      and byte ptr ss:[ebp-4],0  
00435734    8D4D 0C         lea ecx,dword ptr ss:[ebp+C]  
00435737    8975 F0         mov dword ptr ss:[ebp-10],esi  
0043573A    E8 C0730000     call NetManag.0043CAFF  
0043573F    8B4D F4         mov ecx,dword ptr ss:[ebp-C]  
00435742    8B45 08         mov eax,dword ptr ss:[ebp+8]  
00435745    5F              pop edi  
00435746    5E              pop esi  
00435747    64:890D 0000000>mov dword ptr fs:[0],ecx  
0043574E    C9              leave  
0043574F    C2 0C00         retn 0C                             ; 返回程序空间  
```

\============================================================================================================



# 汇编代码分析

```
0040803E    50                 push eax                      ; 返回到这里
0040803F    8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
00408043    C68424 BC020000>   mov byte ptr ss:[esp+2BC],7
0040804B    E8 9C4B0300        call NetManag.0043CBEC       ; 检测 CALL，无需跟进！
00408050    8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
00408054    C68424 B8020000>   mov byte ptr ss:[esp+2B8],6
0040805C    E8 9E4A0300        call NetManag.0043CAFF       ; 比对 CALL
00408061    8BBC24 FC000000    mov edi,dword ptr ss:[esp+FC] ; 假序列号出现，ASCII "7878787878787878"
00408068    8B47 F8            mov eax,dword ptr ds:[edi-8] ; 计算序列号长度是否为 16 位，ds:[07150AA0]=00000010
0040806B    85C0               test eax,eax
0040806D    0F8E A4000000      jle NetManag.00408117         ; 小于 16 位则跳死！
00408073    8B4424 18          mov eax,dword ptr ss:[esp+18] ; 取真码长度，ASCII "a68849197afe5b20"，eax=10
00408077    8BF7               mov esi,edi                  ; 从这里检测真假序列号的合法性
00408079    8A10               mov dl,byte ptr ds:[eax]      ; 取每个字符，做逐字比较
0040807B    8A1E               mov bl,byte ptr ds:[esi]
0040807D    8ACA               mov cl,dl
0040807F    3AD3               cmp dl,bl
00408081    75 1E              jnz short NetManag.004080A1   ; 不等则跳向下一种检测方法（作者怕出错！）
00408083    84C9               test cl,cl
00408085    74 16              je short NetManag.0040809D
00408087    8A50 01            mov dl,byte ptr ds:[eax+1]
0040808A    8A5E 01            mov bl,byte ptr ds:[esi+1]
0040808D    8ACA               mov cl,dl
0040808F    3AD3               cmp dl,bl
00408091    75 0E              jnz short NetManag.004080A1
00408093    83C0 02            add eax,2
00408096    83C6 02            add esi,2
00408099    84C9               test cl,cl
0040809B    ^ 75 DC            jnz short NetManag.00408079
0040809D    33C0               xor eax,eax
0040809F    EB 05              jmp short NetManag.004080A6
004080A1    1BC0               sbb eax,eax                  ; 真假序列号相减
004080A3    83D8 FF            sbb eax,-1
004080A6    85C0               test eax,eax
004080A8    75 58              jnz short NetManag.00408102   ; 不等则跳向深渊！
004080AA    8B8424 04010000    mov eax,dword ptr ss:[esp+104]
004080B1    8D4C24 10          lea ecx,dword ptr ss:[esp+10]
004080B5    50                 push eax
004080B6    57                 push edi
004080B7    68 9C474700        push NetManag.0047479C
004080BC    51                 push ecx
004080BD    E8 2CDB0200        call NetManag.00435BEE
004080C2    83C4 10            add esp,10
004080C5    8D9424 FC000000    lea edx,dword ptr ss:[esp+FC]
004080CC    B9 DCC84700        mov ecx,NetManag.0047C8DC
004080D1    52                 push edx
004080D2    E8 154B0300        call NetManag.0043CBEC
004080D7    8B8424 04010000    mov eax,dword ptr ss:[esp+104]
004080DE    6A 00              push 0
004080E0    68 D0444700        push NetManag.004744D0
004080E5    68 88474700        push NetManag.00474788
004080EA    8BCD               mov ecx,ebp
004080EC    A3 E0C84700        mov dword ptr ds:[47C8E0],eax
004080F1    C705 D4C84700 0>   mov dword ptr ds:[47C8D4],1
004080FB    E8 58280300        call NetManag.0043A958
00408100    EB 37              jmp short NetManag.00408139
00408102    6A 00              push 0                       ; 全部清零
00408104    68 D0444700        push NetManag.004744D0
00408109    68 74474700        push NetManag.00474774
0040810E    8BCD               mov ecx,ebp
00408110    E8 43280300        call NetManag.0043A958       ; 到这里就宣布 Game Over 咯~~~
00408115    EB 22              jmp short NetManag.00408139
.........
```

---

## 算法总结

注册验证非常简单：

（监控机器上限数量最大值：`999999999`）

1. 连接“注册码”和“监控机器上限”，组合成一个“新字符串”。
2. 把“新字符串”转换成“`MD5` 值”。
3. 把计算得出的“`MD5` 值”从第 7 位开始取，连续取 16 位，然后再转换为小写输出，作为序列号。

即：`序列号 = LCase（Mid（MD5（注册码+监控机器上限），7，16））`

## VB6 算法注册机源码 (中文版)

```vb
'//////////////////////////////////////////////////////////////////////////////
'/                     /
'/    Program Disassembler & Debugger & Cracked        /
'/    Author: KuNgBiM[DFCG]                            /
'/    E-mail: gb_1227@163.com                          /
'/    OS : WinXP、PEiD、W32Dasm、Ollydbg、Visual Basic 6 /
'/    Date :  2005-07-04                               /
'/                                                    /
'//////////////////////////////////////////////////////////////////////////////
'/  Note : If you have one or more question, email me please,thank you!    /
'//////////////////////////////////////////////////////////////////////////////

'窗体部分
Private Sub Command1_Click()
    Dim RegCode, RegNumber, SerialNumber
    RegCode = Text1.Text
    RegNumber = Text2.Text

    If RegCode = "" And RegNumber = "" Then
        Text3.Text = "【请输入相关的注册信息】"
    Else
        If RegCode = "" Then
            Text3.Text = "【请输入注册码】"
        Else
            If RegNumber = "" Then
                Text3.Text = "【请输入监控机器上限数量】"
            Else
                If RegNumber > 999999999 Then
                    Text3.Text = "【你输入的监控机器上限数量已超过最大上限】"
                Else
                    Set c1 = New clsMD5
                    SerialNumber = LCase(Mid(c1.Md5_String_Calc(RegCode + RegNumber), 7, 16))
                    Text3.Text = SerialNumber
                End If
            End If
        End If
    End If
End Sub

'/////////////////////////////////////////////////////////////////////////
'类模块部分（名称:clsMD5）
Private Const OFFSET_4 = 4294967296#
Private Const MAXINT_4 = 2147483647

Private State(4) As Long
Private ByteCounter As Long
Private ByteBuffer(63) As Byte

Private Const S11 = 7
Private Const S12 = 12
Private Const S13 = 17
Private Const S14 = 22

Private Const S21 = 5
Private Const S22 = 9
Private Const S23 = 14
Private Const S24 = 20

Private Const S31 = 4
Private Const S32 = 11
Private Const S33 = 16
Private Const S34 = 23

Private Const S41 = 6
Private Const S42 = 10
Private Const S43 = 15
Private Const S44 = 21

Property Get RegisterA() As String
    RegisterA = State(1)
End Property

Property Get RegisterB() As String
    RegisterB = State(2)
End Property

Property Get RegisterC() As String
    RegisterC = State(3)
End Property

Property Get RegisterD() As String
    RegisterD = State(4)
End Property

Public Function Md5_String_Calc(SourceString As String) As String
    MD5Init
    MD5Update LenB(StrConv(SourceString, vbFromUnicode)), StringToArray(SourceString)
    MD5Final
    Md5_String_Calc = GetValues
End Function

Public Function Md5_File_Calc(InFile As String) As String
    GoSub begin
begin:
    Dim FileO As Integer
    FileO = FreeFile
    Call FileLen(InFile)
    Open InFile For Binary Access Read As #FileO
    MD5Init
    Do While Not EOF(FileO)
        Get #FileO, , ByteBuffer
        If Loc(FileO) < LOF(FileO) Then
            ByteCounter = ByteCounter + 64
            MD5Transform ByteBuffer
        End If
    Loop
    ByteCounter = ByteCounter + (LOF(FileO) Mod 64)
    Close #FileO
    MD5Final
    Md5_File_Calc = GetValues
End Function

Private Function StringToArray(InString As String) As Byte()
    Dim I As Integer, bytBuffer() As Byte
    ReDim bytBuffer(LenB(StrConv(InString, vbFromUnicode)))
    bytBuffer = StrConv(InString, vbFromUnicode)
    StringToArray = bytBuffer
End Function

Public Function GetValues() As String
    GetValues = LongToString(State(1)) & LongToString(State(2)) & LongToString(State(3)) & LongToString(State(4))
End Function
```



```vb
End Function

Private Function LongToString(Num As Long) As String

        Dim A As Byte, B As Byte, C As Byte, D As Byte

        A = Num And &HFF&

        If A < 16 Then LongToString = "0" & Hex(A) Else LongToString = Hex(A)

        B = (Num And &HFF00&) \ 256

        If B < 16 Then LongToString = LongToString & "0" & Hex(B) Else LongToString = LongToString & Hex(B)

        C = (Num And &HFF0000) \ 65536

        If C < 16 Then LongToString = LongToString & "0" & Hex(C) Else LongToString = LongToString & Hex(C)

        If Num < 0 Then D = ((Num And &H7F000000) \ 16777216) Or &H80& Else D = (Num And &HFF000000) \ 16777216

        If D < 16 Then LongToString = LongToString & "0" & Hex(D) Else LongToString = LongToString & Hex(D)

End Function

Public Sub MD5Init()

    ByteCounter = 0

    State(1) = UnsignedToLong(1732584193#)

    State(2) = UnsignedToLong(4023233417#)

    State(3) = UnsignedToLong(2562383102#)

    State(4) = UnsignedToLong(271733878#)

End Sub

Public Sub MD5Final()

    Dim dblBits As Double, padding(72) As Byte, lngBytesBuffered As Long

    padding(0) = &H80

    dblBits = ByteCounter * 8

    lngBytesBuffered = ByteCounter Mod 64

    If lngBytesBuffered <= 56 Then MD5Update 56 - lngBytesBuffered, padding Else MD5Update 120 - ByteCounter, padding

    padding(0) = UnsignedToLong(dblBits) And &HFF&

    padding(1) = UnsignedToLong(dblBits) \ 256 And &HFF&

    padding(2) = UnsignedToLong(dblBits) \ 65536 And &HFF&

    padding(3) = UnsignedToLong(dblBits) \ 16777216 And &HFF&

    padding(4) = 0

    padding(5) = 0

    padding(6) = 0

    padding(7) = 0

    MD5Update 8, padding

End Sub

Public Sub MD5Update(InputLen As Long, InputBuffer() As Byte)

    Dim II As Integer, I As Integer, J As Integer, K As Integer, lngBufferedBytes As Long, lngBufferRemaining As Long, lngRem As Long

    lngBufferedBytes = ByteCounter Mod 64

    lngBufferRemaining = 64 - lngBufferedBytes

    ByteCounter = ByteCounter + InputLen

    If InputLen >= lngBufferRemaining Then

        For II = 0 To lngBufferRemaining - 1

            ByteBuffer(lngBufferedBytes + II) = InputBuffer(II)

        Next II

        MD5Transform ByteBuffer

        lngRem = (InputLen) Mod 64

        For I = lngBufferRemaining To InputLen - II - lngRem Step 64

            For J = 0 To 63

                ByteBuffer(J) = InputBuffer(I + J)

            Next J

            MD5Transform ByteBuffer

        Next I

        lngBufferedBytes = 0

    Else

      I = 0

    End If

    For K = 0 To InputLen - I - 1

        ByteBuffer(lngBufferedBytes + K) = InputBuffer(I + K)

    Next K

End Sub

Private Sub MD5Transform(Buffer() As Byte)

    Dim x(16) As Long, A As Long, B As Long, C As Long, D As Long

    A = State(1)

    B = State(2)

    C = State(3)

    D = State(4)

    Decode 64, x, Buffer

    FF A, B, C, D, x(0), S11, -680876936

    FF D, A, B, C, x(1), S12, -389564586

    FF C, D, A, B, x(2), S13, 606105819

    FF B, C, D, A, x(3), S14, -1044525330

    FF A, B, C, D, x(4), S11, -176418897

    FF D, A, B, C, x(5), S12, 1200080426

    FF C, D, A, B, x(6), S13, -1473231341

    FF B, C, D, A, x(7), S14, -45705983

    FF A, B, C, D, x(8), S11, 1770035416

    FF D, A, B, C, x(9), S12, -1958414417

    FF C, D, A, B, x(10), S13, -42063

    FF B, C, D, A, x(11), S14, -1990404162

    FF A, B, C, D, x(12), S11, 1804603682

    FF D, A, B, C, x(13), S12, -40341101

    FF C, D, A, B, x(14), S13, -1502002290

    FF B, C, D, A, x(15), S14, 1236535329

    GG A, B, C, D, x(1), S21, -165796510

    GG D, A, B, C, x(6), S22, -1069501632

    GG C, D, A, B, x(11), S23, 643717713

    GG B, C, D, A, x(0), S24, -373897302

    GG A, B, C, D, x(5), S21, -701558691

    GG D, A, B, C, x(10), S22, 38016083

    GG C, D, A, B, x(15), S23, -660478335

    GG B, C, D, A, x(4), S24, -405537848

    GG A, B, C, D, x(9), S21, 568446438

    GG D, A, B, C, x(14), S22, -1019803690

    GG C, D, A, B, x(3), S23, -187363961

    GG B, C, D, A, x(8), S24, 1163531501

    GG A, B, C, D, x(13), S21, -1444681467

    GG D, A, B, C, x(2), S22, -51403784

    GG C, D, A, B, x(7), S23, 1735328473

    GG B, C, D, A, x(12), S24, -1926607734

    HH A, B, C, D, x(5), S31, -378558

    HH D, A, B, C, x(8), S32, -2022574463

    HH C, D, A, B, x(11), S33, 1839030562

    HH B, C, D, A, x(14), S34, -35309556

    HH A, B, C, D, x(1), S31, -1530992060

    HH D, A, B, C, x(4), S32, 1272893353

    HH C, D, A, B, x(7), S33, -155497632

    HH B, C, D, A, x(10), S34, -1094730640

    HH A, B, C, D, x(13), S31, 681279174

    HH D, A, B, C, x(0), S32, -358537222

    HH C, D, A, B, x(3), S33, -722521979

    HH B, C, D, A, x(6), S34, 76029189

    HH A, B, C, D, x(9), S31, -640364487

    HH D, A, B, C, x(12), S32, -421815835

    HH C, D, A, B, x(15), S33, 530742520

    HH B, C, D, A, x(2), S34, -995338651

    II A, B, C, D, x(0), S41, -198630844

    II D, A, B, C, x(7), S42, 1126891415

    II C, D, A, B, x(14), S43, -1416354905

    II B, C, D, A, x(5), S44, -57434055

    II A, B, C, D, x(12), S41, 1700485571

    II D, A, B, C, x(3), S42, -1894986606

    II C, D, A, B, x(10), S43, -1051523

    II B, C, D, A, x(1), S44, -2054922799

    II A, B, C, D, x(8), S41, 1873313359

    II D, A, B, C, x(15), S42, -30611744

    II C, D, A, B, x(6), S43, -1560198380

    II B, C, D, A, x(13), S44, 1309151649

    II A, B, C, D, x(4), S41, -145523070

    II D, A, B, C, x(11), S42, -1120210379

    II C, D, A, B, x(2), S43, 718787259

    II B, C, D, A, x(9), S44, -343485551

    State(1) = LongOverflowAdd(State(1), A)

    State(2) = LongOverflowAdd(State(2), B)

    State(3) = LongOverflowAdd(State(3), C)

    State(4) = LongOverflowAdd(State(4), D)

End Sub

Private Sub Decode(Length As Integer, OutputBuffer() As Long, InputBuffer() As Byte)

    Dim intDblIndex As Integer, intByteIndex As Integer, dblSum As Double

    For intByteIndex = 0 To Length - 1 Step 4

        dblSum = InputBuffer(intByteIndex) + InputBuffer(intByteIndex + 1) * 256# + InputBuffer(intByteIndex + 2) * 65536# + InputBuffer(intByteIndex + 3) * 16777216#

        OutputBuffer(intDblIndex) = UnsignedToLong(dblSum)

        intDblIndex = intDblIndex + 1

    Next intByteIndex

End Sub

Private Function FF(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long

    A = LongOverflowAdd4(A, (B And C) Or (Not (B) And D), x, ac)

    A = LongLeftRotate(A, S)

    A = LongOverflowAdd(A, B)

End Function

Private Function GG(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long

    A = LongOverflowAdd4(A, (B And D) Or (C And Not (D)), x, ac)

    A = LongLeftRotate(A, S)

    A = LongOverflowAdd(A, B)

End Function

Private Function HH(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long

    A = LongOverflowAdd4(A, B Xor C Xor D, x, ac)

    A = LongLeftRotate(A, S)

    A = LongOverflowAdd(A, B)

End Function

Private Function II(A As Long, B As Long, C As Long, D As Long, x As Long, S As Long, ac As Long) As Long

    A = LongOverflowAdd4(A, C Xor (B Or Not (D)), x, ac)

    A = LongLeftRotate(A, S)

    A = LongOverflowAdd(A, B)

End Function

Function LongLeftRotate(value As Long, Bits As Long) As Long

    Dim lngSign As Long, lngI As Long

    Bits = Bits Mod 32

    If Bits = 0 Then LongLeftRotate = value: Exit Function

    For lngI = 1 To Bits

        lngSign = value And &HC0000000

        value = (value And &H3FFFFFFF) * 2

        value = value Or ((lngSign < 0) And 1) Or (CBool(lngSign And &H40000000) And &H80000000)

    Next

    LongLeftRotate = value
```



```vb
End Function  

Private Function LongOverflowAdd(Val1 As Long, Val2 As Long) As Long  

    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long  

    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&)  

    lngOverflow = lngLowWord \ 65536  

    lngHighWord = (((Val1 And &HFFFF0000) \ 65536) + ((Val2 And &HFFFF0000) \ 65536) + lngOverflow) And &HFFFF&  

    LongOverflowAdd = UnsignedToLong((lngHighWord * 65536#) + (lngLowWord And &HFFFF&))  

End Function  

Private Function LongOverflowAdd4(Val1 As Long, Val2 As Long, val3 As Long, val4 As Long) As Long  

    Dim lngHighWord As Long, lngLowWord As Long, lngOverflow As Long  

    lngLowWord = (Val1 And &HFFFF&) + (Val2 And &HFFFF&) + (val3 And &HFFFF&) + (val4 And &HFFFF&)  

    lngOverflow = lngLowWord \ 65536  

    lngHighWord = (((Val1 And &HFFFF0000) \ 65536) + ((Val2 And &HFFFF0000) \ 65536) + ((val3 And &HFFFF0000) \ 65536) + ((val4 And &HFFFF0000) \ 65536) + lngOverflow) And &HFFFF&  

    LongOverflowAdd4 = UnsignedToLong((lngHighWord * 65536#) + (lngLowWord And &HFFFF&))  

End Function  

Private Function UnsignedToLong(value As Double) As Long  

    If value < 0 Or value >= OFFSET_4 Then Error 6  

    If value <= MAXINT_4 Then UnsignedToLong = value Else UnsignedToLong = value - OFFSET_4  

End Function  

Private Function LongToUnsigned(value As Long) As Double  

    If value < 0 Then LongToUnsigned = value + OFFSET_4 Else LongToUnsigned = value  

End Function  

'/////////////////////////////////////////////////////////////////////////
```

============================================================================================

【注册信息】：

注册码：`1cb7fbd5`

监控机器上限：`999999999`

序列号：`a68849197afe5b20`

--------------------------------------------------------------------------------------------

（本文完）

版权所有（C）2005 `KuNgBiM[DFCG]`         Copyright (C) 2005 `KuNgBiM[DFCG]`

--------------------------------------------------------------------------------------------

          `Cracked By KuNgBiM[DFCG]`

                `2005-07-04`

                `18:53:18 PM`

`4dd466d0e1e0cc123afbe504da2462ca`
