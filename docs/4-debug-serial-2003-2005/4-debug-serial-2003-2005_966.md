
# 长天游戏加速器[超强版] 注册算法分析(入门级)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-403.htm`](https://www.pediy.com/kssd/pediy07/pediy7-403.htm)

日期：2005 年 9 月 28 日 破解人:Baby2008[BCG][FCG][PCG]

-------------------------------------------------------------------------------------------------------------------------

**『软件名称』**：长天游戏加速器[超强版]

**『软件大小』**：759 KB

**『下载地址』**：[`www.soreport.com/SpeedUp3.EXE`](http://www.soreport.com/SpeedUp3.EXE)

**『软件介绍』**：

该版本在原 2.0 基础之上，经过潜心开发完成。1）增加了进程加速模式，也就是只针对某个进程加速，而系统和其他应用程序速度不变，节省了系统资源。2）使用进程加速模式，解决了以前有个别机器无法加速的问题。3）保留 2.0 原来的系统加速模式，可以根据需要使用。

**『保护方式』**：注册码保护+试用期限 1 天

**『破解声明』**：初学 Crack，只是感兴趣,，失误之处敬请诸位大侠赐教!

**『破解工具』**：`OllyDbg.V1.10` 聆风听雨汉化第二版、`PeID` 0.93，`Dede` 3.50,`ASPackDie` v1.41.HH

**『破解过程』**：

`PeID` 0.93 查壳,`ASPack` 2.12 -> `Alexey` `Solodovnikov`，我懒人一个,用 `ASPackDie` v1.41.HH 搞定,默认另存为 `Unpacked.exe`,脱了直接能运行，

`OD` 载入，插件查找“注册成功！\n 注册信息为：\n 用户名：”字样，向上来到：

```
00481044 <>/.  55                 push ebp                                        ;  <-TRegSoftFrm@btnRegClick
00481045   |.  8BEC               mov ebp,esp
00481047   |.  6A 00              push 0
00481049   |.  6A 00              push 0
0048104B   |.  6A 00              push 0
0048104D   |.  53                 push ebx
0048104E   |.  8BD8               mov ebx,eax
00481050   |.  33C0               xor eax,eax
00481052   |.  55                 push ebp
00481053   |.  68 64114800        push <Unpacked.->System.@HandleFinally;>
00481058   |.  64:FF30            push dword ptr fs:[eax]
0048105B   |.  64:8920            mov dword ptr fs:[eax],esp
0048105E   |.  A1 CC2C4A00        mov eax,dword ptr ds:[4A2CCC]
00481063   |.  8B00               mov eax,dword ptr ds:[eax]
00481065   |.  8B40 58            mov eax,dword ptr ds:[eax+58]
00481068 <>|.  E8 6316FFFF        call Unpacked.004726D0                            ;  ->ActiveX.PROPSETHDR_OSVER_KIND
(DWORD):Word;<+>
0048106D   |.  84C0               test al,al
0048106F   |.  74 1D              je short Unpacked.0048108E
00481071   |.  6A 40              push 40
00481073   |.  B9 70114800        mov ecx,Unpacked.00481170                          ;  注册成功
00481078   |.  BA 7C114800        mov edx,Unpacked.0048117C                          ;  软件已经注册！
0048107D   |.  A1 B82B4A00        mov eax,dword ptr ds:[4A2BB8]
00481082   |.  8B00               mov eax,dword ptr ds:[eax]
00481084 <>|.  E8 6FE9FEFF        call Unpacked.0046F9F8                            ;  ->Forms.TApplication.MessageBox
(TApplication;PChar;PChar;Longint):Integer;
00481089   |.  E9 B3000000        jmp Unpacked.00481141
0048108E   |>  8D55 F8            lea edx,[local.2]
00481091 <>|.  8B83 34030000      mov eax,dword ptr ds:[ebx+334]                     ;  *edtCode:N.A.
00481097 <>|.  E8 94E7FCFF        call Unpacked.0044F830                            ;  ->Controls.TControl.GetText
(TControl):TCaption;
0048109C   |.  8B45 F8            mov eax,[local.2]                                  ;  试炼码
0048109F   |.  50                 push eax
004810A0   |.  8D55 F4            lea edx,[local.3]
004810A3 <>|.  8B83 30030000      mov eax,dword ptr ds:[ebx+330]                     ;  *edtDDHName:N.A.
004810A9 <>|.  E8 82E7FCFF        call Unpacked.0044F830                            ;  ->Controls.TControl.GetText
(TControl):TCaption;
004810AE   |.  8B55 F4            mov edx,[local.3]                                  ;  注册名
004810B1   |.  A1 CC2C4A00        mov eax,dword ptr ds:[4A2CCC]
004810B6   |.  8B00               mov eax,dword ptr ds:[eax]
004810B8   |.  8B40 58            mov eax,dword ptr ds:[eax+58]
004810BB   |.  33C9               xor ecx,ecx
004810BD   |.  E8 461AFFFF        call Unpacked.00472B08
004810C2   |.  84C0               test al,al
004810C4   |.  75 12              jnz short Unpacked.004810D8
004810C6   |.  BA 94114800        mov edx,Unpacked.00481194                          ;  注册失败！
004810CB <>|.  8B83 08030000      mov eax,dword ptr ds:[ebx+308]                     ;  *lblExpired:N.A.
004810D1 <>|.  E8 8AE7FCFF        call Unpacked.0044F860                            ;  ->Controls.TControl.SetText
(TControl;TCaption);
004810D6   |.  EB 69              jmp short Unpacked.00481141
004810D8   |>  68 A8114800        push Unpacked.004811A8                            ;  注册成功！\n 注册信息为：\n 用户名：
004810DD   |.  A1 CC2C4A00        mov eax,dword ptr ds:[4A2CCC]
004810E2   |.  8B00               mov eax,dword ptr ds:[eax]
004810E4   |.  8B58 58            mov ebx,dword ptr ds:[eax+58]
004810E7   |.  FF73 48            push dword ptr ds:[ebx+48]
004810EA   |.  68 D4114800        push Unpacked.004811D4                            ;  \n
004810EF   |.  68 D4114800        push Unpacked.004811D4                            ;  \n
004810F4   |.  68 E0114800        push Unpacked.004811E0                            ;  注册码：
004810F9   |.  A1 CC2C4A00        mov eax,dword ptr ds:[4A2CCC]
004810FE   |.  FF73 5C            push dword ptr ds:[ebx+5C]
00481101   |.  68 D4114800        push Unpacked.004811D4                            ;  \n
00481106   |.  68 F4114800        push Unpacked.004811F4                            ;  感谢您对我们的支持！请重新启动长天游戏加速器。
0048110B   |.  8D45 FC            lea eax,[local.1]
0048110E   |.  BA 08000000        mov edx,8
00481113 <>|.  E8 B032F8FF        call Unpacked.004043C8                            ;  ->System.@LStrCatN;
00481118   |.  6A 40              push 40
0048111A   |.  8B45 FC            mov eax,[local.1]
0048111D <>|.  E8 E633F8FF        call Unpacked.00404508                            ;  ->System.@LStrToPChar(String):PAnsiChar;
00481122   |.  8BD0               mov edx,eax
00481124   |.  B9 70114800        mov ecx,Unpacked.00481170                          ;  注册成功
00481129   |.  A1 B82B4A00        mov eax,dword ptr ds:[4A2BB8]
0048112E   |.  8B00               mov eax,dword ptr ds:[eax]
00481130 <>|.  E8 C3E8FEFF        call Unpacked.0046F9F8                            ;  ->Forms.TApplication.MessageBox
(TApplication;PChar;PChar;Longint):Integer;
00481135   |.  A1 B82B4A00        mov eax,dword ptr ds:[4A2BB8]
0048113A   |.  8B00               mov eax,dword ptr ds:[eax]
0048113C <>|.  E8 13E8FEFF        call Unpacked.0046F954                            ;  ->Forms.TApplication.Terminate
(TApplication);
00481141   |>  33C0               xor eax,eax
00481143   |.  5A                 pop edx
00481144   |.  59                 pop ecx
00481145   |.  59                 pop ecx
00481146   |.  64:8910            mov dword ptr fs:[eax],edx
00481149   |.  68 6B114800        push Unpacked.0048116B
0048114E   |>  8D45 F4            lea eax,[local.3]
00481151   |.  BA 02000000        mov edx,2
00481156 <>|.  E8 112FF8FF        call Unpacked.0040406C                            ;  ->System.@LStrArrayClr(void;void;Integer);
0048115B   |.  8D45 FC            lea eax,[local.1]
0048115E <>|.  E8 E52EF8FF        call Unpacked.00404048                            ;  ->System.@LStrClr(void;void);
00481163   \.  C3                 retn
00481164 <> .^ E9 E328F8FF        jmp Unpacked.00403A4C                             ;  ->System.@HandleFinally;
00481169    .^ EB E3              jmp short Unpacked.0048114E
0048116B     . 5B                 pop ebx
0048116C     . 8BE5               mov esp,ebp
0048116E     . 5D                 pop ebp
0048116F     .  C3                retn
```

很明显,`004810BD   |.  E8 461AFFFF    call Unpacked.00472B08` 是关键,跟进:

-------------------------------------------------------------------------------------------------------------------------

```
00472B08   /$  55                 push ebp                                         ;  (initial cpu selection)
00472B09   |.  8BEC               mov ebp,esp
00472B0B   |.  83C4 F0            add esp,-10
00472B0E   |.  53                 push ebx
00472B0F   |.  33DB               xor ebx,ebx
00472B11   |.  895D F0            mov [local.4],ebx
00472B14   |.  895D F4            mov [local.3],ebx
00472B17   |.  894D F8            mov [local.2],ecx
```



```
00472B1A   |.  8955 FC        mov [local.1],edx  

00472B1D   |.  8BD8           mov ebx,eax  

00472B1F   |.  8B45 FC        mov eax,[local.1]  

00472B22   |.  E8 D119F9FF    call Unpacked.004044F8  

00472B27   |.  8B45 F8        mov eax,[local.2]  

00472B2A   |.  E8 C919F9FF    call Unpacked.004044F8  

00472B2F   |.  8B45 08        mov eax,[arg.1]  

00472B32   |.  E8 C119F9FF    call Unpacked.004044F8  

00472B37   |.  33C0           xor eax,eax  

00472B39   |.  55             push ebp  

00472B3A   |.  68 F22B4700    push Unpacked.00472BF2  

00472B3F   |.  64:FF30        push dword ptr fs:[eax]  

00472B42   |.  64:8920        mov dword ptr fs:[eax],esp  

00472B45   |.  8B45 FC        mov eax,[local.1]                           ;  注册名  

00472B48   |.  E8 BB17F9FF    call Unpacked.00404308                     ;  System.@LStrLen(String):Integer;  

00472B4D   |.  3B43 4C        cmp eax,dword ptr ds:[ebx+4C]             ;  64  

00472B50   |.  7F 19          jg short Unpacked.00472B6B                 ;  注册名长度不能>64  

00472B52   |.  8B45 FC        mov eax,[local.1]                           ;  注册名  

00472B55   |.  E8 AE17F9FF    call Unpacked.00404308                     ;  System.@LStrLen(String):Integer;  

00472B5A   |.  3B43 50        cmp eax,dword ptr ds:[ebx+50]             ;  3  

00472B5D   |.  7C 0C          jl short Unpacked.00472B6B                 ;  注册名长度不能<3  

00472B5F   |.  8B45 08        mov eax,[arg.1]                             ;  试炼码  

00472B62   |.  E8 A117F9FF    call Unpacked.00404308                     ;  System.@LStrLen(String):Integer;  

00472B67   |.  85C0           test eax,eax  

00472B69   |.  75 04          jnz short Unpacked.00472B6F                ;  试炼码长度不能为 0  

00472B6B   |>  33DB           xor ebx,ebx  

00472B6D   |.  EB 60          jmp short Unpacked.00472BCF  

00472B6F   |>  8D55 F4        lea edx,[local.3]  

00472B72   |.  8B45 08        mov eax,[arg.1]                             ;  试炼码  

00472B75   |.  E8 A654F9FF    call Unpacked.00408020                     ;  转大写 UpperCase():AnsiString;  

00472B7A   |.  8B55 F4        mov edx,[local.3]                           ;  UpperCase(试炼码)  

00472B7D   |.  8D45 08        lea eax,[arg.1]  

00472B80   |.  E8 5B15F9FF    call Unpacked.004040E0  

00472B85   |.  8D4D F0        lea ecx,[local.4]  

00472B88   |.  8B55 FC        mov edx,[local.1]                           ;  注册名  

00472B8B   |.  8BC3           mov eax,ebx  

00472B8D   |.  E8 46FBFFFF    call Unpacked.004726D8                     ;  重要函数  

00472B92   |.  8B45 F0        mov eax,[local.4]                           ;  注册码  

00472B95   |.  8B55 08        mov edx,[arg.1]                             ;  试炼码  

00472B98   |.  E8 FB54F9FF    call Unpacked.00408098                     ;  比较 CompareStr():Integer;  

00472B9D   |.  85C0           test eax,eax  

00472B9F   |.  74 04          je short Unpacked.00472BA5                 ;  明码比较,内存注册机或爆破点  

00472BA1   |.  33DB           xor ebx,ebx  

00472BA3   |.  EB 2A          jmp short Unpacked.00472BCF  

00472BA5   |>  8D43 48        lea eax,dword ptr ds:[ebx+48]  

00472BA8   |.  8B55 FC        mov edx,[local.1]  

00472BAB   |.  E8 EC14F9FF    call Unpacked.0040409C  

00472BB0   |.  8D43 54        lea eax,dword ptr ds:[ebx+54]  

00472BB3   |.  8B55 F8        mov edx,[local.2]  

00472BB6   |.  E8 E114F9FF    call Unpacked.0040409C  

00472BBB   |.  8D43 5C        lea eax,dword ptr ds:[ebx+5C]  

00472BBE   |.  8B55 08        mov edx,[arg.1]  

00472BC1   |.  E8 D614F9FF    call Unpacked.0040409C  

00472BC6   |.  8BC3           mov eax,ebx  

00472BC8   |.  E8 5B020000    call Unpacked.00472E28  

00472BCD   |.  B3 01          mov bl,1                                    ;  标志  

00472BCF   |>  33C0           xor eax,eax  

00472BD1   |.  5A             pop edx  

00472BD2   |.  59             pop ecx  

00472BD3   |.  59             pop ecx  

00472BD4   |.  64:8910        mov dword ptr fs:[eax],edx  

00472BD7   |.  68 F92B4700    push Unpacked.00472BF9  

00472BDC   |>  8D45 F0        lea eax,[local.4]  

00472BDF   |.  BA 04000000    mov edx,4  

00472BE4   |.  E8 8314F9FF    call Unpacked.0040406C  

00472BE9   |.  8D45 08        lea eax,[arg.1]  

00472BEC   |.  E8 5714F9FF    call Unpacked.00404048  

00472BF1   \\.  C3             retn  

00472BF2     .^ E9 550EF9FF   jmp Unpacked.00403A4C  

00472BF7     .^ EB E3          jmp short Unpacked.00472BDC  

00472BF9      .  8BC3          mov eax,ebx  

00472BFB      .  5B             pop ebx  

00472BFC      .  8BE5          mov esp,ebp  

00472BFE      .  5D             pop ebp  

00472BFF      .  C2 0400        retn 4  
```

明码比较，我喜欢，继续跟进关键函数 `00472B8D   |.  E8 46FBFFFF    call Unpacked.004726D8`：

\-------------------------------------------------------------------------------------------------------------------------

```
004726D8   /$  55             push ebp  

004726D9   |.  8BEC           mov ebp,esp  

004726DB   |.  51             push ecx  

004726DC   |.  B9 04000000    mov ecx,4  

004726E1   |>  6A 00          /push 0  

004726E3   |.  6A 00          |push 0  

004726E5   |.  49             |dec ecx  

004726E6   |.^ 75 F9          \jnz short Unpacked.004726E1  

004726E8   |.  874D FC        xchg [local.1],ecx  

004726EB   |.  53             push ebx  

004726EC   |.  56             push esi  

004726ED   |.  57             push edi  

004726EE   |.  8BF9           mov edi,ecx  

004726F0   |.  8955 FC        mov [local.1],edx                           ;  注册名  

004726F3   |.  8BF0           mov esi,eax  

004726F5   |.  8B45 FC        mov eax,[local.1]  

004726F8   |.  E8 FB1DF9FF    call Unpacked.004044F8  

004726FD   |.  33C0           xor eax,eax  

004726FF   |.  55             push ebp  

00472700   |.  68 78284700    push Unpacked.00472878  

00472705   |.  64:FF30        push dword ptr fs:[eax]  

00472708   |.  64:8920        mov dword ptr fs:[eax],esp  

0047270B   |.  8D55 DC        lea edx,[local.9]  

0047270E   |.  8BC6           mov eax,esi  

00472710   |.  E8 430F0000    call Unpacked.00473658                     ;  产生机器码,没必要分析  

00472715   |.  8B45 DC        mov eax,[local.9]                           ;  机器码  

00472718   |.  8D55 EC        lea edx,[local.5]  

0047271B   |.  E8 505BF9FF    call Unpacked.00408270                     ;  SysUtils.Trim(AnsiString);  

00472720   |.  837D EC 00     cmp [local.5],0  

00472724   |.  75 0D          jnz short Unpacked.00472733                ;  机器码不为空  

00472726   |.  8D45 E0        lea eax,[local.8]  

00472729   |.  8B55 FC        mov edx,[local.1]  

0047272C   |.  E8 AF19F9FF    call Unpacked.004040E0  

00472731   |.  EB 5D          jmp short Unpacked.00472790  

00472733   |>  8B45 EC        mov eax,[local.5]                           ;  机器码  

00472736   |.  E8 CD1BF9FF    call Unpacked.00404308                     ;  Length(机器码)  

0047273B   |.  8BD8           mov ebx,eax                                 ;  长度  

0047273D   |.  8D45 E8        lea eax,[local.6]  

00472740   |.  50             push eax  

00472741   |.  8BCB           mov ecx,ebx                                 ;  EBX=机器码长度,记为 M  

00472743   |.  D1F9           sar ecx,1                                   ;  M/2  

00472745   |.  79 03          jns short Unpacked.0047274A  

00472747   |.  83D1 00        adc ecx,0  

0047274A   |>  BA 01000000    mov edx,1                                   ;  1  

0047274F   |.  8B45 EC        mov eax,[local.5]                           ;  机器码  
```



```
00472752   |.  E8 111EF9FF    call Unpacked.00404568                     ;  System.@LStrCopy(机器码,1,M/2)  
00472757   |.  8D45 E4        lea eax,[local.7]  
0047275A   |.  50             push eax  
0047275B   |.  8BC3           mov eax,ebx                                 ;  长度 M  
0047275D   |.  D1F8           sar eax,1                                     ;  M/2  
0047275F   |.  79 03          jns short Unpacked.00472764  
00472761   |.  83D0 00        adc eax,0  
00472764   |>  8BCB           mov ecx,ebx  
00472766   |.  2BC8           sub ecx,eax                                   ;  M-M/2  
00472768   |.  8BD3           mov edx,ebx                                 ;  长度 M  
0047276A   |.  D1FA           sar edx,1                                     ;  M/2  
0047276C   |.  79 03          jns short Unpacked.00472771  
0047276E   |.  83D2 00        adc edx,0  
00472771   |>  42             inc edx                                       ;  M/2+1  
00472772   |.  8B45 EC        mov eax,[local.5]                             ;  机器码  
00472775   |.  E8 EE1DF9FF    call Unpacked.00404568                     ;  System.@LStrCopy(机器码,M/2+1,M-M/2);  
0047277A   |.  FF75 E8        push [local.6]                               ;  机器码前 6 位  
0047277D   |.  FF75 FC        push [local.1]                               ;  注册名  
00472780   |.  FF75 E4        push [local.7]                               ;  机器码后 6 位  
00472783   |.  8D45 E0        lea eax,[local.8]  
00472786   |.  BA 03000000    mov edx,3  
0047278B   |.  E8 381CF9FF    call Unpacked.004043C8                     ;  连接函数 System.@LStrCatN;  
00472790   |>  C745 F0 000000>mov [local.4],0  
00472797   |.  C745 F4 000000>mov [local.3],0  
0047279E   |.  8B45 FC        mov eax,[local.1]                             ;  注册名  
004727A1   |.  E8 621BF9FF    call Unpacked.00404308                     ;  Length  
004727A6   |.  3B46 4C        cmp eax,dword ptr ds:[esi+4C]                 ;  64  
004727A9   |.  7F 0D          jg short Unpacked.004727B8  
004727AB   |.  8B45 FC        mov eax,[local.1]                             ;  注册名  
004727AE   |.  E8 551BF9FF    call Unpacked.00404308  
004727B3   |.  3B46 50        cmp eax,dword ptr ds:[esi+50]                 ;  3  
004727B6   |.  7D 0C          jge short Unpacked.004727C4  
004727B8   |>  8BC7           mov eax,edi  
004727BA   |.  E8 8918F9FF    call Unpacked.00404048  
004727BF   |.  E9 91000000    jmp Unpacked.00472855  
004727C4   |>  8B45 E0        mov eax,[local.8]                             ;  机器码前 6+注册名+机器码后 6 位  
004727C7   |.  E8 3C1BF9FF    call Unpacked.00404308                     ;  Length  
004727CC   |.  8BD8           mov ebx,eax                                 ;  长度  
004727CE   |.  EB 37          jmp short Unpacked.00472807  
004727D0   |>  8B45 F0        /mov eax,[local.4]                            ;  初始值 0  
004727D3   |.  8B55 F4        |mov edx,[local.3]                            ;  0  
004727D6   |.  0346 68        |add eax,dword ptr ds:[esi+68]                 ;  初始值 075BCD15  
004727D9   |.  1356 6C        |adc edx,dword ptr ds:[esi+6C]                 ;  0  
004727DC   |.  52             |push edx                                      ;  保存  
004727DD   |.  50             |push eax  
004727DE   |.  8B45 E0        |mov eax,[local.8]                            ;  机器码前 6+注册名+机器码后 6 位,记为 Mach  
004727E1   |.  0FB64418 FF    |movzx eax,byte ptr ds:[eax+ebx-1]             ;  从后面开始取字符  
004727E6   |.  50             |push eax                                      ;  ASCII 值入栈  
004727E7   |.  B8 59040000    |mov eax,459                                   ;  459  
004727EC   |.  5A             |pop edx                                       ;  ASCII  
004727ED   |.  8BCA           |mov ecx,edx  
004727EF   |.  33D2           |xor edx,edx  
004727F1   |.  F7F1           |div ecx  
004727F3   |.  8BC2           |mov eax,edx                                 ;  eax=459 MOD ASCII  
004727F5   |.  33D2           |xor edx,edx  
004727F7   |.  290424         |sub dword ptr ss:[esp],eax  
004727FA   |.  195424 04      |sbb dword ptr ss:[esp+4],edx  
004727FE   |.  58             |pop eax                                       ;  取出  
004727FF   |.  5A             |pop edx  
00472800   |.  8945 F0        |mov [local.4],eax  
00472803   |.  8955 F4        |mov [local.3],edx  
00472806   |.  4B             |dec ebx  
00472807   |>  8B45 E0        mov eax,[local.8]                             ;  机器码前 6+注册名+机器码后 6 位  
0047280A   |.  E8 F91AF9FF    |call Unpacked.00404308                     ;  Length  
0047280F   |.  3BD8           |cmp ebx,eax  
00472811   |.  7F 04          |jg short Unpacked.00472817  
00472813   |.  85DB           |test ebx,ebx  
00472815   |.^ 7F B9          \jg short Unpacked.004727D0                 ;  循环累加,64 位算法  
00472817   |>  8B5E 60        mov ebx,dword ptr ds:[esi+60]  
0047281A   |.  85DB           test ebx,ebx  
0047281C   |.  7F 11          jg short Unpacked.0047282F  
0047281E   |.  FF75 F4        push [local.3]                               ;  /Arg2  
00472821   |.  FF75 F0        push [local.4]                               ;  |Arg1  
00472824   |.  8BD7           mov edx,edi                                  ;  |  
00472826   |.  33C0           xor eax,eax                                  ;  |  
00472828   |.  E8 7F5CF9FF    call Unpacked.004084AC                     ;  \Unpacked.004084AC  
0047282D   |.  EB 26          jmp short Unpacked.00472855  
0047282F   |>  FF75 F4        push [local.3]                               ;  /Arg2  
00472832   |.  FF75 F0        push [local.4]                               ;  |Arg1  
00472835   |.  8BD7           mov edx,edi                                  ;  |  
00472837   |.  8BC3           mov eax,ebx                                  ;  |  
00472839   |.  E8 6E5CF9FF    call Unpacked.004084AC                     ;  \Unpacked.004084AC  
0047283E   |.  8B07           mov eax,dword ptr ds:[edi]  
00472840   |.  E8 C31AF9FF    call Unpacked.00404308                     ;  Length  
00472845   |.  8BC8           mov ecx,eax  
00472847   |.  2B4E 60        sub ecx,dword ptr ds:[esi+60]  
0047284A   |.  8B56 60        mov edx,dword ptr ds:[esi+60]  
0047284D   |.  42             inc edx  
0047284E   |.  8BC7           mov eax,edi  
00472850   |.  E8 531DF9FF    call Unpacked.004045A8  
00472855   |>  33C0           xor eax,eax  
00472857   |.  5A             pop edx  
00472858   |.  59             pop ecx  
00472859   |.  59             pop ecx  
0047285A   |.  64:8910        mov dword ptr fs:[eax],edx  
0047285D   |.  68 7F284700    push Unpacked.0047287F  
00472862   |>  8D45 DC        lea eax,[local.9]  
00472865   |.  BA 05000000    mov edx,5  
0047286A   |.  E8 FD17F9FF    call Unpacked.0040406C  
0047286F   |.  8D45 FC        lea eax,[local.1]  
00472872   |.  E8 D117F9FF    call Unpacked.00404048  
00472877   \.  C3             retn  
00472878    .^ E9 CF11F9FF    jmp Unpacked.00403A4C  
0047287D    .^ EB E3          jmp short Unpacked.00472862  
0047287F    .  5F             pop edi  
00472880    .  5E             pop esi  
00472881    .  5B             pop ebx  
00472882    .  8BE5           mov esp,ebp  
00472884    .  5D             pop ebp  
00472885    .  C3             retn  
```

看来已经到达目的地了，算法很简单，就一个 64 位数据类型计算。

### 算法总结

1. 机器码前半部分 + 注册名 + 机器码后半部分，记为 `Mach`；
2. 初始一个 `int64` 常数：`075BCD15`
3. 从 `Mach` 后面开始循环取字符 `Mach(i)`，`Sum = 075BCD15 + (Sum - 459 mod Mach(i));`
4. 转换 64 位数值 `Sum` 为 12 位 16 进制字符串即为注册码。

### Delphi 7.0 注册机源代码

```
Procedure TForm1.btn1Click(Sender: TObject);

Const
  Temp = $075BCD15; //常数

Var
  Sum: Int64; //累加
```



```
i: Integer; //循环变量  

Mach: String;  

Begin  

  Sum := 0;  

  Mach := LeftStr(edtMach.Text, Length(edtMach.Text) Div 2) + edtName.Text + RightStr(edtMach.Text, Length(edtMach.Text) - Length(edtMach.Text) Div 2);  

  Mach := ReverseString(Mach);  

  For i := 1 To Length(Mach) Do Sum := Temp + (Sum - $459 Mod Ord(Mach[i]));  

  edtSN.Text := IntToHex(Sum, 12);  

End;  
```

我的注册信息：

- 用户名：`Baby2008`
- 机器码：`00006705357E`
- 注册码：`0000932C0417`

测试通过!

--完--

`4dd466d0e1e0cc123afbe504da2462ca`
