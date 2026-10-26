
# 智能开关机 V3.20算法分析笔记

- 【破解作者】 落魄浪子
- 【使用工具】 `FlyOD1.1`
- 【破解平台】 Win9x/NT/2000/XP
- 【软件名称】 智能开关机 V3.20
- 【下载地址】 `4afK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6%4j5Y4y4G2k6Y4c8Q4x3X3g2U0L8$3@1`
- 【软件简介】
  - **软件概述**：智能开关机能够在关机状态自动开启你的计算机，以及在待机状态下唤醒计算机执行指定任务。
  - **主要功能**：
    - 控制计算机从关机状态自动开启。
    - 在待机状态下自动唤醒计算机。
    - 在指定的时间运行指定程序、打开指定文档、打开指定网址以及关闭指定程序。
    - 在指定的时间调用系统默认程序播放mp3、wma、wav以及flash等声音文件。
    - 计算机开启后自动登陆系统以及锁定系统。
    - 在指定的时间关闭、重启系统，以及使系统进入待机、休眠状态。
    - 与互联网上多个原子钟服务器校正时间。
    - 其他。
- 【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

因为我有时在星期六下班时电脑还在工作，而星期天又没有来。那电脑就这样一直开着，又不想自已来关，所以下了这个软件好让它在完成任务后自已关机。启动软件，30天限制.

开启`OD`载入程序，找到出错的地方并在关键点下断：

```
004B7C73   |.  8B45 E0           mov eax,dword ptr ss:[ebp-20]    断在这里，机器码入EAX
004B7C76   |.  8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
004B7C79   |.  BA 987E4B00       mov edx,autoss.004B7E98    ;  ASCII "DS$%^TG"预置数入EDX
004B7C7E   |.  E8 51CA0200       call autoss.004E46D4             进入看看
```

---

```
004E46D4   /$  55                push ebp         进入后来到这里
004E46D5   |.  8BEC              mov ebp,esp
004E46D7   |.  83C4 F0           add esp,-10
004E46DA   |.  53                push ebx
004E46DB   |.  56                push esi
004E46DC   |.  57                push edi
004E46DD   |.  894D F4           mov dword ptr ss:[ebp-C],ecx
004E46E0   |.  8955 F8           mov dword ptr ss:[ebp-8],edx   预置数
004E46E3   |.  8945 FC           mov dword ptr ss:[ebp-4],eax   机器码
004E46E6   |.  8B45 FC           mov eax,dword ptr ss:[ebp-4]   机器码
004E46E9   |.  E8 8200F2FF       call autoss.00404770
004E46EE   |.  8B45 F8           mov eax,dword ptr ss:[ebp-8]   预置数
004E46F1   |.  E8 7A00F2FF       call autoss.00404770
004E46F6   |.  33C0              xor eax,eax
004E46F8   |.  55                push ebp
004E46F9   |.  68 9E474E00       push autoss.004E479E
004E46FE   |.  64:FF30           push dword ptr fs:[eax]
004E4701   |.  64:8920           mov dword ptr fs:[eax],esp
004E4704   |.  837D F8 00        cmp dword ptr ss:[ebp-8],0      比较预置数是否为空
004E4708   |.  75 0D             jnz short autoss.004E4717       为空则预置数为"TG^u"
004E470A   |.  8D45 F8           lea eax,dword ptr ss:[ebp-8]
004E470D   |.  BA B4474E00       mov edx,autoss.004E47B4               ;  ASCII "TG^u"
004E4712   |.  E8 41FCF1FF       call autoss.00404358
004E4717   |>  BE 01000000       mov esi,1                       初始计数器1
004E471C   |.  8B45 FC           mov eax,dword ptr ss:[ebp-4]    机器码入EAX
004E471F   |.  E8 5CFEF1FF       call autoss.00404580            取长度
004E4724   |.  8BF8              mov edi,eax
004E4726   |.  85FF              test edi,edi
004E4728   |.  7E 4E             jle short autoss.004E4778
004E472A   |.  BB 01000000       mov ebx,1                       初始计数器2
004E472F   |>  8B45 FC           /mov eax,dword ptr ss:[ebp-4]   机器码入EAX
004E4732   |.  8A4418 FF         |mov al,byte ptr ds:[eax+ebx-1] 取机器码的一位的HEX入AL记为S1
004E4736   |.  24 0F             |and al,0F                      S1与0F作与运算结果记为N1
004E4738   |.  8B55 F8           |mov edx,dword ptr ss:[ebp-8]   预置数"DS$%^TG"入EDX
004E473B   |.  8A5432 FF         |mov dl,byte ptr ds:[edx+esi-1] 取预置数的一位的HEX入DL记为S2
004E473F   |.  80E2 0F           |and dl,0F                      S2与0F作与运算结果记为N2
004E4742   |.  32C2              |xor al,dl                      N1与N2异或运算结果记为N3
004E4744   |.  8845 F3           |mov byte ptr ss:[ebp-D],al
004E4747   |.  8D45 FC           |lea eax,dword ptr ss:[ebp-4]
004E474A   |.  E8 8900F2FF       |call autoss.004047D8
004E474F   |.  8B55 FC           |mov edx,dword ptr ss:[ebp-4]   机器码入EDX
004E4752   |.  8A541A FF         |mov dl,byte ptr ds:[edx+ebx-1] 取机器码的一位的HEX记为S3
004E4756   |.  80E2 F0           |and dl,0F0                     S3与0F0作与运算结果记为N4
004E4759   |.  8A4D F3           |mov cl,byte ptr ss:[ebp-D]     N3入CL
004E475C   |.  02D1              |add dl,cl                      N4加N3结果记为N5
004E475E   |.  885418 FF         |mov byte ptr ds:[eax+ebx-1],dl
004E4762   |.  46                |inc esi                        计数器1加1
004E4763   |.  8B45 F8           |mov eax,dword ptr ss:[ebp-8]   预置数"DS$%^TG"入EAX
004E4766   |.  E8 15FEF1FF       |call autoss.00404580           取长度
004E476B   |.  3BF0              |cmp esi,eax
004E476D   |.  7E 05             |jle short autoss.004E4774      是否取完
004E476F   |.  BE 01000000       |mov esi,1                      取完则再初始计数器1
004E4774   |>  43                |inc ebx                        计数器2加1
004E4775   |.  4F                |dec edi                        机器码长度减1
004E4776   |.^ 75 B7             \jnz short autoss.004E472F      没有完则继续
004E4778   |>  8B45 F4           mov eax,dword ptr ss:[ebp-C]
004E477B   |.  8B55 FC           mov edx,dword ptr ss:[ebp-4]    这就是计算完的后9位注册码
004E477E   |.  E8 91FBF1FF       call autoss.00404314
004E4783   |.  33C0              xor eax,eax
004E4785   |.  5A                pop edx
004E4786   |.  59                pop ecx
004E4787   |.  59                pop ecx
004E4788   |.  64:8910           mov dword ptr fs:[eax],edx
004E478B   |.  68 A5474E00       push autoss.004E47A5
004E4790   |>  8D45 F8           lea eax,dword ptr ss:[ebp-8]
004E4793   |.  BA 02000000       mov edx,2
004E4798   |.  E8 47FBF1FF       call autoss.004042E4
004E479D   \.  C3                retn
004E479E    .^ E9 7DF4F1FF       jmp autoss.00403C20
004E47A3    .^ EB EB             jmp short autoss.004E4790
004E47A5    .  5F                pop edi
004E47A6    .  5E                pop esi
004E47A7    .  5B                pop ebx
004E47A8    .  8BE5              mov esp,ebp
004E47AA    .  5D                pop ebp
004E47AB    .  C3                retn                             返回
```

---

```
004B7C83   |.  8B45 E4           mov eax,dword ptr ss:[ebp-1C]    返回到这里
004B7C86   |.  50                push eax
004B7C87   |.  8D55 D8           lea edx,dword ptr ss:[ebp-28]
004B7C8A   |.  8B83 48030000     mov eax,dword ptr ds:[ebx+348]
004B7C90   |.  E8 DBDFF9FF       call autoss.00455C70
004B7C95   |.  8B45 D8           mov eax,dword ptr ss:[ebp-28]   机器码入EAX
004B7C98   |.  8D55 DC           lea edx,dword ptr ss:[ebp-24]
004B7C9B   |.  E8 1CCB0200       call autoss.004E47BC            进入看看
```

---

```
004E47BC   /$  55                push ebp                        进入后来到这里
004E47BD   |.  8BEC              mov ebp,esp
004E47BF   |.  33C9              xor ecx,ecx
004E47C1   |.  51                push ecx
004E47C2   |.  51                push ecx
004E47C3   |.  51                push ecx
004E47C4   |.  51                push ecx
004E47C5   |.  51                push ecx
004E47C6   |.  53                push ebx
004E47C7   |.  56                push esi
004E47C8   |.  57                push edi
004E47C9   |.  8955 F8           mov dword ptr ss:[ebp-8],edx
004E47CC   |.  8945 FC           mov dword ptr ss:[ebp-4],eax
004E47CF   |.  8B45 FC           mov eax,dword ptr ss:[ebp-4]
004E47D2   |.  E8 99FFF1FF       call autoss.00404770
004E47D7   |.  33C0              xor eax,eax
004E47D9   |.  55                push ebp
004E47DA   |.  68 91484E00       push autoss.004E4891
004E47DF   |.  64:FF30           push dword ptr fs:[eax]
004E47E2   |.  64:8920           mov dword ptr fs:[eax],esp
004E47E5   |.  837D FC 00        cmp dword ptr ss:[ebp-4],0
004E47E9   |.  75 0A             jnz short autoss.004E47F5
004E47EB   |.  8B45 F8           mov eax,dword ptr ss:[ebp-8]
004E47EE   |.  E8 CDFAF1FF       call autoss.004042C0
004E47F3   |.  EB 79             jmp short autoss.004E486E
004E47F5   |>  8D45 F4           lea eax,dword ptr ss:[ebp-C]
```
