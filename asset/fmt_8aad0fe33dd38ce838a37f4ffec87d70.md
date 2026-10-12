
# 绿鹰 PC 万能精灵 V3.92 注册算法分析（同样适用 V3.95）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-312.htm`](https://www.pediy.com/kssd/pediy07/pediy7-312.htm)

绿鹰 PC 万能精灵 V3.92 注册算法分析（同样适用 V3.95）

先说点废话。^\_^ 就是这个软件带我进入 Cracker 世界，感谢~ 当时由于中了卑鄙小人的木马，病急乱投医，找到这个软件，结果要注册，呵呵，当时很想搞掉它，可惜功力不够。最近闲来无事，下载了个最新版 V3.92 再研究研究，也算是了却我一个心愿吧~ 谁知道噩梦开始了，该软件虽没有用密码学算法（还不如用呢，还好定位和分析），可是却层层嵌套，费尽心思隐藏代码生成过程，真是不堪回首，不过如果只是想要自己的注册码的话就不用这么麻烦了，可以利用软件帮 ourselves 算，如果分析... 以下我只贴出主要代码，细了太长，我也不想再回头看这段可怕的路了，祝大家好运~

\[Cracker\] ： prince  
\[时间\] ： 2005.07.16  
\[声明\] ： 只做技术交流，不做商业用途，如果你手头宽裕并喜欢这个软件的话请支持正版软件。  
\[E-mail\] ： Cracker_prince@163.com  
\[软件信息\]  
\[软件说明\] ：

功能判断，可杀未来木马。可实时查杀：各种网络游戏盗号木马(传奇、天堂等)、邮件木马、聊天软件盗密木马(如 OICQ、MSN 等)、股票大盗...

\[保护方式\] ： 序列号 + 次数限制 + 功能限制（只提示，不杀毒，并且不管有没有病毒都提示有可疑文件） + 反调试 + 自校验

\[外壳保护\] ： `PECompact 2.x` -> `Jeremy Collake`

\[编译器/语言\]： `Microsoft Visual C++ 6.0` / `C/C++`

\[下载地址\] ： `http://www.onlinedown.net/soft/6396.htm` （3.95 版）

\[分析过程\] ：

文件脱壳、自校验和反调试我前几天已经写过，这里不再赘述，可以参考这里：`http://bbs.pediy.com/showthread.php?s=&threadid=14860`

软件使用 `SendMessage` 获取用户输入的注册码，下断 `bpx SendMessageA`，将我们没有点击注册按钮之前的所有断点,添好假码 `8765432133333333`（格式后面分析），点确定，断下来：

```
------------------------------------------------------------------
004248AC   >call dword ptr ds:[<&user32.SendMessageA>] ; \SendMessageA
004248B2   >test eax,eax
004248B4   >mov eax,dword ptr ds:[47B0F0]
004248B9   >jnz UN2_++自.00424AA8
004248BF   >mov dword ptr ss:[esp+94],eax
004248C6   >mov dword ptr ss:[esp+1364],0
004248D1   >mov dword ptr ss:[esp+10],eax
004248D5   >mov dword ptr ss:[esp+11C],eax
004248DC   >lea ecx,dword ptr ss:[esp+94]
004248E3   >mov byte ptr ss:[esp+1364],2
004248EB   >push ecx
004248EC   >push 456
004248F1   >mov ecx,esi
004248F3   >call UN2_++自.0044BE75                     ;  EAX=0012F7C0 “tDF”
004248F8   >mov ecx,eax
004248FA   >call UN2_++自.0044A113                     ;  取用户输入的假码送 ECX，EAX=假码个数
004248FF   >mov edx,dword ptr ss:[esp+94]              ;  假码写入内存 00130608
00424906   >cmp dword ptr ds:[edx-8],10                ;  假码个数同 0x10 比较
0042490A   >je short UN2_++自.0042494F                 ;  不跳就死
0042490C   >mov ecx,esi
------------------------------------------------------------------
```

主要判断代码：

```
004100A0  /$  6A>push -1
004100A2  |.  68>push UN2_++自.004591AC                   ;  SE 句柄安装
004100A7  |.  64>mov eax,dword ptr fs:[0]
004100AD  |.  50 push eax
004100AE  |.  64>mov dword ptr fs:[0],esp
004100B5  |.  81>sub esp,814
004100BB  |.  53 push ebx
004100BC  |.  55 push ebp
004100BD  |.  56 push esi
004100BE  |.  57 push edi
004100BF  |.  8B>mov edi,ecx
004100C1  |.  8B>mov eax,dword ptr ss:[esp+834]           ;  取假码的前 8 位“87654321”送 EAX
004100C8  |.  8B>mov ecx,dword ptr ss:[esp+838]           ;  取假码的后 8 位“33333333”送 ECX
004100CF  |.  C7>mov dword ptr ss:[esp+82C],3             ;  3 写入[0012DB10]
004100DA  |.  8B>mov edx,dword ptr ds:[eax-8]             ;  EDX=8
004100DD  |.  8B>mov eax,dword ptr ds:[ecx-8]             ;  EAX=8
004100E0  |.  3B>cmp edx,eax
004100E2  |.  0F>jnz UN2_++自.0041033E
004100E8  |.  85>test eax,eax
004100EA  |.  0F>je UN2_++自.0041033E
004100F0  |.  33>xor esi,esi                              ;  ESI 清零
004100F2  |.  85>test eax,eax
004100F4  |.  7E>jle short UN2_++自.0041011E
004100F6  |>  8A>/mov cl,byte ptr ds:[esi+ecx]             ;  循环取假码后 8 位的单个字符
004100F9  |.  88>|mov byte ptr ss:[esp+10],cl             
004100FD  |.  8B>|mov edx,dword ptr ss:[esp+10]           ;  送 EDX
00410101  |.  52 |push edx                                  ;  压栈
00410102  |.  8B>|mov ecx,edi
00410104  |.  E8>|call UN2_++自.00410620
00410109  |.  85>|test eax,eax
0041010B  |.  0F>|jl UN2_++自.0041033E
00410111  |.  8B>|mov ecx,dword ptr ss:[esp+838]
00410118  |.  46 |inc esi
00410119  |.  3B>|cmp esi,dword ptr ds:[ecx-8]
0041011C  |.^ 7C>\jl short UN2_++自.004100F6
0041011E  |>  8B>mov eax,dword ptr ss:[esp+834]            ;  取假码的前 8 位送 EAX
00410125  |.  33>xor esi,esi                               ;  ESI 清零
00410127  |.  8B>mov ecx,dword ptr ds:[eax-8]              ;  ECX=8
0041012A  |.  85>test ecx,ecx
0041012C  |.  0F>jle UN2_++自.004101F2
00410132  |.  B3>mov bl,4                                  
00410134  |>  8B>/mov ecx,dword ptr ds:[47B0F0]             ;  UN2_++自.0047B104
0041013A  |.  89>|mov dword ptr ss:[esp+14],ecx
0041013E  |.  56 |push esi                                  ;  ESI==0
0041013F  |.  56 |push esi
00410140  |.  8D>|lea ecx,dword ptr ss:[esp+83C]           
00410147  |.  88>|mov byte ptr ss:[esp+834],bl
0041014E  |.  E8>|call UN2_++自.004103B0                    ;  取假码的前 8 位单个字符
00410153  |.  50 |push eax                                  ;  假码的第 1 位压栈
00410154  |.  51 |push ecx                                  ;  ECX==0
00410155  |.  8D>|lea edx,dword ptr ss:[esp+848]
0041015C  |.  8B>|mov ecx,esp
0041015E  |.  89>|mov dword ptr ss:[esp+2C],esp
00410162  |.  52 |push edx
00410163  |.  E8>|call UN2_++自.0044C6BF
00410168  |.  8D>|lea eax,dword ptr ss:[esp+1C]             ;|
0041016C  |.  8B>|mov ecx,edi                                ;|
0041016E  |.  50 |push eax                                   ;|Arg1
0041016F  |.  E8>|call UN2_++自.004103C0                    ; \关键，确定后 8 位字符，跟进
00410174  |.  50 |push eax                                  
00410175  |.  8D>|lea ecx,dword ptr ss:[esp+18]             
00410179  |.  C6>|mov byte ptr ss:[esp+830],5                
00410181  |.  E8>|call UN2_++自.0044CA83
00410186  |.  8D>|lea ecx,dword ptr ss:[esp+10]
0041018A  |.  88>|mov byte ptr ss:[esp+82C],bl
00410191  |.  E8>|call UN2_++自.0044C94A
00410196  |.  8B>|mov ecx,dword ptr ss:[esp+14]
0041019A  |.  8B>|mov eax,dword ptr ss:[esp+838]            ;  取假码的后 8 位
004101A1  |.  8A>|mov dl,byte ptr ds:[ecx]                  ;  查表
004101A3  |.  8A>|mov cl,byte ptr ds:[esi+eax]              ;  取假码后 8 位单个字符
004101A6  |.  88>|mov byte ptr ss:[esp+1C],cl              
004101AA  |.  88>|mov byte ptr ss:[esp+18],dl               ;  查表结果写入内存
004101AE  |.  8B>|mov edx,dword ptr ss:[esp+1C]             
004101B2  |.  8B>|mov ecx,edi
004101B4  |.  52 |push edx
004101B5  |.  E8>|call UN2_++自.00410620
004101BA  |.  8B>|mov ebp,eax                                
004101BC  |.  8B>|mov eax,dword ptr ss:[esp+18]             
004101C0  |.  50 |push eax
004101C1  |.  8B>|mov ecx,edi                                
004101C3  |.  E8>|call UN2_++自.00410620
004101C8  |.  3B>|cmp ebp,eax                               ;  判断查表结果和计算结果是否相同
004101CA  |.  C6>|mov byte ptr ss:[esp+82C],3
004101D2  |.  8D>|lea ecx,dword ptr ss:[esp+14]
004101D6  |.  0F>|jnz UN2_++自.00410339                     ;  跳走即失败！！！
004101DC  |.  E8>|call UN2_++自.0044C94A
004101E1  |.  8B>|mov ecx,dword ptr ss:[esp+834]            ;  取假码前 8 个字符准备下次循环
004101E8  |.  46 |inc esi
004101E9  |.  3B>|cmp esi,dword ptr ds:[ecx-8]
004101EC  |.^ 0F>\jl UN2_++自.00410134
004101F2  |>  8B>mov eax,dword ptr ss:[esp+844]
004101F9  |.  85>test eax,eax
004101FB  |.  0F>jnz UN2_++自.004102DF
00410201  |.  A1>mov eax,dword ptr ds:[47E420]
00410206  |.  8D>lea edx,dword ptr ss:[esp+24]
0041020A  |.  68>push 800                                    ;/Count = 800 (2048.)
------------------------------------------------------------------
```

我就只贴出这些了，剩下的所有都可以从这里跟下去，很长，很复杂，而且利用多次指针转移，不容易跟踪，大家如果有兴趣分析，可以顺藤摸瓜，逆着往回查，可以省点工夫。

我贴出注册机源码，过程中的浮点操作我不知道如何用高级语言描述，于是大多采用了程序中的汇编源码，大家可以对照源码查看，基本过程在我的注册机里很清楚。

另外，我是边跟踪边写注册机，所以变量使用混乱，代码繁杂，也不想再修正了，大家凑合看吧，请勿深究变量使用及设计方面的问题。
