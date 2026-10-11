
# 一个建站软件的注册算法分析（最新版）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-415.htm`](https://www.pediy.com/kssd/pediy07/pediy7-415.htm)

【破解作者】 落魄浪子  

【使用工具】 `FlyOD1.1` 修改版  

【破解平台】 `Win9x/NT/2000/XP`  

【软件名称】 `Crob FTP Server 中文专业版`  

【下载地址】 [`www.crob.net`](http://www.crob.net)  

【加壳方式】 `ASPack 2.1 -> Alexey Solodovnikov [Overlay]`  

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

---

【破解内容】

最近由于工作忙，很长时间没有来这里了。呵呵。今天下了个软件，主要是想在自己的电脑上建个网站，就找到了它“`Crob FTP Server 中文专业版`”，没有注册有 21 天的限制。注册码不对时点注册，注册框没有任何提示就退出。正确也没有提示，只会把标题栏的未注册字样去掉。

怎么找到下面的，呵呵![](img/d8960f6c7afa20ba0ca27920887f4540.jpg)

```
00445403    .  3978 F8      cmp dword ptr ds:[eax-8],edi   比较注册码是否为空
00445406    .  0F84 5E0800>je 1.00445C6A
0044540C    .  8B8E 640500>mov ecx,dword ptr ds:[esi+564]
00445412    .  3979 F8      cmp dword ptr ds:[ecx-8],edi   比较 email 是否为空
00445415    .  0F84 4F0800>je 1.00445C6A
0044541B    .  8B96 680500>mov edx,dword ptr ds:[esi+568]
00445421    .  8D9E 680500>lea ebx,dword ptr ds:[esi+568]
00445427    .  397A F8      cmp dword ptr ds:[edx-8],edi   比较注册名是否为空
0044542A    .  0F84 3A0800>je 1.00445C6A
00445430    .  8BCE         mov ecx,esi
00445432    .  E8 D9090000  call 1.00445E10
00445437    .  8B45 00      mov eax,dword ptr ss:[ebp]     注册码入 eax
0044543A    .  897C24 2C    mov dword ptr ss:[esp+2C],edi
0044543E    .  8B40 F8      mov eax,dword ptr ds:[eax-8]    取注册码长度入 eax
00445441    .  85C0         test eax,eax
00445443    .  7E 1C        jle short 1.00445461
00445445    >  8BCE         mov ecx,esi
00445447    .  E8 C4090000  call 1.00445E10                 
0044544C    .  8B45 00      mov eax,dword ptr ss:[ebp]
0044544F    .  803C38 2D    cmp byte ptr ds:[eax+edi],2D    查找注码里是否有字符"-"
00445453    .  75 04        jnz short 1.00445459
00445455    .  FF4424 2C    inc dword ptr ss:[esp+2C]
00445459    >  8B48 F8      mov ecx,dword ptr ds:[eax-8]
0044545C    .  47           inc edi
0044545D    .  3BF9         cmp edi,ecx
0044545F    .^ 7C E4        jl short 1.00445445             是否查找完成，没有则继续.
00445461    >  8BCE         mov ecx,esi
00445463    .  E8 A8090000  call 1.00445E10                 
00445468    .  837C24 2C 0>cmp dword ptr ss:[esp+2C],3      注册码中是否有 3 个字符"-"
0044546D    .  0F8C F70700>jl 1.00445C6A
00445473    .  6A 2D        push 2D
00445475    .  8BCD         mov ecx,ebp
00445477    .  E8 40510B00  call <jmp.&MFC42.#6874_CString::Remove>  把注册码中的字符"-"去掉
0044547C    .  8B4D 00      mov ecx,dword ptr ss:[ebp]
0044547F    .  8379 F8 1C   cmp dword ptr ds:[ecx-8],1C  去掉字符“-”后注册码长度是否小于 28
```

---

```
0044557F    .  68 649F5500  push 1.00559F64                       ;  ASCII "nover"
00445584    .  E8 F7490B00  call <jmp.&MFC42.#537_CString::CString>
00445589    .  8BCF         mov ecx,edi                             ; |
0044558B    .  C64424 60 0>mov byte ptr ss:[esp+60],5                ; |
00445590    .  E8 9BB8FEFF  call 1.00430E30                         ; \1.00430E30  算法 call 进入
00430E30   /$  6A FF        push -1     进入到这里
00430E32   |.  68 600F5000 push 1.00500F60                       ;  SE handler installation
00430E37   |.  64:A1 00000>mov eax,dword ptr fs:[0]
00430E3D   |.  50           push eax
00430E3E   |.  64:8925 000>mov dword ptr fs:[0],esp
```

---

```
00430DC9   |> \8B45 EC      mov eax,dword ptr ss:[ebp-14]
00430DCC   |.  8B4D F0      mov ecx,dword ptr ss:[ebp-10]
00430DCF   |.  8B16         mov edx,dword ptr ds:[esi]
00430DD1   |.  53           push ebx
00430DD2   |.  50           push eax
00430DD3   |.  51           push ecx
00430DD4   |.  8BCE         mov ecx,esi
00430DD6   |.  FF52 08      call dword ptr ds:[edx+8]              进入
```

---

```
004AD5A1     .  E8 5AF6FFFF  call 1.004ACC00                       ; \重点进入
004AD5A6     .  8B4D 08      mov ecx,dword ptr ss:[ebp+8]
004AD5A9     .  8B5424 10    mov edx,dword ptr ss:[esp+10]
004AD5AD     .  03F9         add edi,ecx
004AD5AF     .  03D9         add ebx,ecx
004AD5B1     .  42           inc edx
004AD5B2     .  8BC6         mov eax,esi
```

---

```
004ACC00   /$  64:A1 00000>mov eax,dword ptr fs:[0]
004ACD01   |.  E8 DAF3FFFF  call 1.004AC0E0                       ; \1.004AC0E0  进入
004ACD06   |.  E9 93020000  jmp 1.004ACF9E
004AC2BD   |> \8B4424 64    mov eax,dword ptr ss:[esp+64]    ;  字符“nover"入 eax 记为 a1
004AC2C1   |.  33D2         xor edx,edx                     ;  清空 edx
004AC2C3   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC2C5   |.  8B5D 38      mov ebx,dword ptr ss:[ebp+38]    ;  字符”M_hc“入 ebx 记为 b1"4D5F6863
004AC2C8   |.  8A10         mov dl,byte ptr ds:[eax]         ;  A1 的第一位 n 入 dl 记为 n1
004AC2CA   |.  8A48 01      mov cl,byte ptr ds:[eax+1]       ;  A1 的第二位 o 入 cl 记为 n2
004AC2CD   |.  C1E2 18      shl edx,18                      ;  n1 左移 18 次记为 n3
004AC2D0   |.  40           inc eax
004AC2D1   |.  8B7D 3C      mov edi,dword ptr ss:[ebp+3C]    ;  字符”PUHM“入 edi 记为 b2"5055484d
004AC2D4   |.  C1E1 10      shl ecx,10                      ;  n2 左移 10 次记为 n4
004AC2D7   |.  0BD1         or edx,ecx                      ;  n3 与 n4 作或运算记为 n5”6E6F0000"
004AC2D9   |.  40           inc eax
004AC2DA   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC2DC   |.  8A28         mov ch,byte ptr ds:[eax]         ;  A1 的第三位 v 入 ch 记为 n6
004AC2DE   |.  0BD1         or edx,ecx                      ;  n5 与 n6 作或运算记为 n7"6E6F76"
004AC2E0   |.  40           inc eax
004AC2E1   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC2E3   |.  8A08         mov cl,byte ptr ds:[eax]         ;  A1 的第四位 e 入 cl 记为 n8
004AC2E5   |.  0BD1         or edx,ecx                      ;  n7 与 n8 作或运算记为 n9"6E6F7665"
004AC2E7   |.  40           inc eax
004AC2E8   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC2EA   |.  33D3         xor edx,ebx                     ;  n9 与 b1 作异或运算记为 n10"0D072928"
004AC2EC   |.  8A08         mov cl,byte ptr ds:[eax]         ;  A1 的第五位 r 入 cl 记为 n11
004AC2EE   |.  33DB         xor ebx,ebx                     ;  清空 ebx
004AC2F0   |.  8BF1         mov esi,ecx                     ;  n11 入 esi="72"
004AC2F2   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC2F4   |.  8A48 01      mov cl,byte ptr ds:[eax+1]       ;  A1 的第六位 0 入 cl 记为 n12
004AC2F7   |.  895424 18    mov dword ptr ss:[esp+18],edx    ;  n10 放入内存地址 0012ec00"0D072928"
004AC2FB   |.  C1E6 18      shl esi,18                      ;  n11 左移 18 次记为 n13"72000000"
004AC2FE   |.  40           inc eax
004AC2FF   |.  C1E1 10      shl ecx,10                      ;  n12 左移 10 次记为 n14
004AC302   |.  0BF1         or esi,ecx                      ;  n13 与 n14 作或运算记为 n15“72000000”
004AC304   |.  40           inc eax
004AC305   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC307   |.  8A28         mov ch,byte ptr ds:[eax]         ;  A1 的第七位 0 入 ch 记为 n16
004AC309   |.  0BF1         or esi,ecx                      ;  n15 与 n16 作或运算记为 n17"72000000"
004AC30B   |.  40           inc eax
004AC30C   |.  33C9         xor ecx,ecx                     ;  清空 ecx
004AC30E   |.  8A08         mov cl,byte ptr ds:[eax]         ;  A1 的第八位 0 入 cl 记为 n18
004AC310   |.  0BF1         or esi,ecx                      ;  n17 与 n18 作或运算记为 n19"72000000"
004AC312   |.  40           inc eax
004AC313   |.  33C9         xor ecx,ecx                     ;  清空 ecx
```



### 汇编代码片段分析

```
004AC315   |.  33F7        xor esi,edi                 ;  n19 与 b2 作异或运算记为 n20 "3f485550"  

004AC317   |.  8A08        mov cl,byte ptr ds:[eax]     ;  A1 的第九位 0 入 cl 记为 n21  

004AC319   |.  8A58 01     mov bl,byte ptr ds:[eax+1]   ;  A1 的第十位 0 入 bl 记为 n22  

004AC31C   |.  C1E1 18     shl ecx,18                  ;  n21 左移 18 次记为 n23  

004AC31F   |.  40          inc eax  

004AC320   |.  897424 14   mov dword ptr ss:[esp+14],esi ;  n20 放入内存地址 0012ebfc "3F485550"  

004AC324   |.  C1E3 10     shl ebx,10                  ;  n22 左移 10 次记为 n24  

004AC327   |.  0BCB        or ecx,ebx                  ;  n23 与 n24 作或运算记为 n25  

004AC329   |.  40          inc eax  

004AC32A   |.  33DB        xor ebx,ebx                 ;  清空 ebx  

004AC32C   |.  8A38        mov bh,byte ptr ds:[eax]     ;  A1 的第十一位 0 入 bh 记为 n26  

004AC32E   |.  0BCB        or ecx,ebx                  ;  n25 与 n26 作或运算记为 n27  

004AC330   |.  40          inc eax  

004AC331   |.  33DB        xor ebx,ebx                 ;  清空 ebx  

004AC333   |.  8A18        mov bl,byte ptr ds:[eax]     ;  A1 的第十二位 0 入 bl 记为 n28  

004AC335   |.  0BCB        or ecx,ebx                  ;  n27 与 n28 作或运算记为 n29  

004AC337   |.  8B5D 40     mov ebx,dword ptr ss:[ebp+40]   ;  字符“92iO”入 ebx 记为 b3 "4F693239"  

004AC33A   |.  40          inc eax  

004AC33B   |.  33CB        xor ecx,ebx                 ;  N29 与 b3 作异或运算记为 n30 "4F693239"  

004AC33D   |.  33DB        xor ebx,ebx                 ;  清空 ebx  

004AC33F   |.  894C24 1C   mov dword ptr ss:[esp+1C],ecx  ;  n30 放入内存地址 0012ec04 "4F693239"  

004AC343   |.  8A18        mov bl,byte ptr ds:[eax]     ;  A1 的第十三位 0 入 bl 记为 n31  

004AC345   |.  8BF3        mov esi,ebx                 ;  n31 入 esi  

004AC347   |.  33DB        xor ebx,ebx                 ;  清空 ebx  

004AC349   |.  8A58 01     mov bl,byte ptr ds:[eax+1]   ;  A1 的第十四位 0 入 bl 记为 n32  

004AC34C   |.  C1E6 18     shl esi,18                  ;  n31 左移 18 次记为 n33  

004AC34F   |.  40          inc eax  

004AC350   |.  C1E3 10     shl ebx,10                  ;  n32 左移 10 次记为 n34  

004AC353   |.  0BF3        or esi,ebx                  ;  n33 与 n34 作或运算记为 n35  

004AC355   |.  40          inc eax  

004AC356   |.  33DB        xor ebx,ebx                 ;  清空 ebx  

004AC358   |.  8A38        mov bh,byte ptr ds:[eax]     ;  A1 的第十五位 0 入 bh 记为 n36  

004AC35A   |.  8A58 01     mov bl,byte ptr ds:[eax+1]   ;  A1 的第十六位 0 入 bl 记为 n37  

004AC35D   |.  8B45 44     mov eax,dword ptr ss:[ebp+44]   ;  字符”4873“放入 eax 记为 b4 "33373834"  

004AC360   |.  0BDE        or ebx,esi                  ;  ebx 与 n35 作或运算记为 n38  

004AC362   |.  8BB5 D80300> mov esi,dword ptr ss:[ebp+3D8]  ;  字符"E"入 esi 记为 n39  

004AC368   |.  33C3        xor eax,ebx                 ;  b4 与 n38 作异或运算记为 n40 "33373834"  

004AC36A   |.  897424 20   mov dword ptr ss:[esp+20],esi  ;  n39 放入内存地址 0012ec08 "E"  

004AC36E   |.  894424 10   mov dword ptr ss:[esp+10],eax  ;  n40 放入内存地址 0012ebf8 "33373834"  

004AC372   |.  83FE 01     cmp esi,1  

004AC375   |.  0F8E 350100> jle 1.004AC4B0  

004AC37B   |.  83C5 60     add ebp,60  

004AC37E   |.  4E          dec esi                     ;  n39 减一记为 n41  

004AC37F   |.  897424 64   mov dword ptr ss:[esp+64],esi  ;  n41 放入内存地址 0012ec4c "D"  

004AC383   |>  33DB        /xor ebx,ebx                ;  清空 ebx  

004AC385   |.  83C5 20     |add ebp,20  

004AC388   |.  8ADC        |mov bl,ah                   ;  n40 的第二位放入 bl 记为 n42 "38"  

004AC38A   |.  33C0        |xor eax,eax                ;  清空 eax  

004AC38C   |.  8A4424 1E   |mov al,byte ptr ss:[esp+1E]    ;  n30 的第二位”i“入 al 记为 n43 "69"  

004AC390   |.  8B349D 4867> |mov esi,dword ptr ds:[ebx*4+526748] ;  n42*4+526748 查表得"07090E07"记为 n44 入 esi  

004AC397   |.  8B3C85 4863> |mov edi,dword ptr ds:[eax*4+526348] ;  n30*4+526348 查表得"10E9F9F9"记为 n45 入 edi  

004AC39E   |.  33C0        |xor eax,eax                ;  清空 eax  

004AC3A0   |.  8A4424 17   |mov al,byte ptr ss:[esp+17]    ;  n20 的第一位”？“入 al 记为 n46 "3f"  

004AC3A4   |.  33F7        |xor esi,edi                ;  n44 与 n45 作异或运算记为 n47”17E0F7FE“  

004AC3A6   |.  8B7D DC     |mov edi,dword ptr ss:[ebp-24]   ;  b2 入 edi "4D485550"  

004AC3A9   |.  8B1C85 485F> |mov ebx,dword ptr ds:[eax*4+525f48] ;  n20*4+525f48 查表得"EA75759F"记为 n48 入 ebx  

004AC3B0   |.  8BC2        |mov eax,edx                 ;  n9 入 eax  

004AC3B2   |.  25 FF000000 |and eax,0FF                 ;  n9 与 0FF 作与运算记为 n49 "00000028"  

004AC3B7   |.  33F3        |xor esi,ebx                 ;  n47 与 n48 作异或运算记为 n50”FD958261“  

004AC3B9   |.  8B1C85 486B> |mov ebx,dword ptr ds:[eax*4+526b48] ;  n49*4+526b48 查表得”34345C68“记为 n51  

004AC3C0   |.  33C0        |xor eax,eax                 ;  清空 eax  

004AC3C2   |.  8A4424 12   |mov al,byte ptr ss:[esp+12]    ;  n40 的第二位”7“入 al 记为 n52  

004AC3C6   |.  33F3        |xor esi,ebx                 ;  n50 与 n51 作异或运算记为 n53”C9A1DE09“  

004AC3C8   |.  33DB        |xor ebx,ebx                 ;  清空 ebx  

004AC3CA   |.  33F7        |xor esi,edi                 ;  n53 与 b2 作异或运算记为 n54”84E98B59“  

004AC3CC   |.  8A5C24 1F   |mov bl,byte ptr ss:[esp+1F]    ;  B3 的第四位”O“入 bl 记为 n55  

004AC3D0   |.  8B3C85 4863> |mov edi,dword ptr ds:[eax*4+526348] ;  n52*4+526348 查表得”"B52F9A9A"记为 n56 入 edi  

004AC3D7   |.  8B049D 485F> |mov eax,dword ptr ds:[ebx*4+525f48] ;  n55*4+525f48 查表得“13848487”记为 n57 入 eax  

004AC3DE   |.  33F8        |xor edi,eax                 ;  n57 与 n56 作异或运算记为 n58 "A6AB1E0D"  

004AC3E0   |.  33C0        |xor eax,eax                 ;  清空 eax  

004AC3E2   |.  8AC6        |mov al,dh                   ;  n10 的第二位“）”入 al 记为 n58  

004AC3E4   |.  8B1485 4867> |mov edx,dword ptr ds:[eax*4+526748] ;  n58*4+526748 查表得“A5F451A5“记为 n59 入 edx  

004AC3EB   <>|.  8B4424 14   |mov eax,dword ptr ss:[esp+14]   ;  n20 入 eax  

004AC3EF   |.  33FA        |xor edi,edx                 ;  n58 与 n59 作异或运算记为 n60”035F4FA8“  

004AC3F1   |.  8BD0        |mov edx,eax                 ;  N20 入 edx  

004AC3F3   |.  81E2 FF0000> |and edx,0FF                 ;  n20 与 0FF 作与运算记为 n61 "00000050"  

004AC3F9   |.  8B1C95 486B> |mov ebx,dword ptr ds:[edx*4+526b48] ;  n61*4+526b48 查表得”5353F5A6“记为 N62 入 ebx  

004AC400   |.  33D2        |xor edx,edx                 ;  清空 edx  

004AC402   |.  8A5424 13   |mov dl,byte ptr ss:[esp+13]    ;  n40 的第四位”3“入 dl 记为 n63  

004AC406   |.  33FB        |xor edi,ebx                 ;  n60 与 n62 作异或运算记为 n64 "500CBA0E"  

004AC408   |.  8B5D E0     |mov ebx,dword ptr ss:[ebp-20]   ;  n30 入 ebx  

004AC40B   |.  33FB        |xor edi,ebx                 ;  n64 与 n30 作异或运算记为 n65 "1F658837"  

004AC40D   |.  33DB        |xor ebx,ebx                 ;  清空 ebx  

004AC40F   |.  8ADC        |mov bl,ah                   ;  n20 的第二位”U"入 bl 记为 n66  

004AC411   |.  8B0495 485F> |mov eax,dword ptr ds:[edx*4+525f48] ;  n63*4+525f48 查表得“9DC3C35E”记为 n67 入 eax  

004AC418   |.  8B149D 4867> |mov edx,dword ptr ds:[ebx*4+526748] ;  n66*4+526748 查表得“FC1FE3FC”记为 n68 入 edx  

004AC41F   |.  33C2        |xor eax,edx                 ;  n67 与 n68 作异或运算记为 n69“61DC20A2”  

004AC421   |.  33D2        |xor edx,edx                 ;  清空 edx  

004AC423   |.  8A5424 1A   |mov dl,byte ptr ss:[esp+1A]    ;  n10 的第二位“07”入 dl 记为 n70  

004AC427   |.  8B1C95 4863> |mov ebx,dword ptr ds:[edx*4+526348] ;  n70*4+526348 查表得“5491C5C5”记为 n71 入 ebx  

004AC42E   |.  8BD1        |mov edx,ecx                 ;  n30 入 edx  

004AC430   |.  81E2 FF0000> |and edx,0FF                 ;  n30 与 OFF 作与运算记为 n72 "39"  

004AC436   |.  33C3        |xor eax,ebx                 ;  n69 与 n71 作异或运算记为 n73 "354DE567"  

004AC438   |.  8B1C95 486B> |mov ebx,dword ptr ds:[edx*4+526b48] ;  n72*4+526b48 查表得“12123624”记为 n74 入 ebx  
```



004AC43F   |.  8B55 E4     |`mov edx,dword ptr ss:[ebp-1C]`   ;  n40 入 `edx`  

004AC442   |.  33C3        |`xor eax,ebx`                     ;  n73 与 n74 作异或运算记为 n75 “275FD343”  

004AC444   |.  33C2        |`xor eax,edx`                     ;  n75 与 N40 作异或运算记为 n76 "1468EB77“  

004AC446   |.  33D2        |`xor edx,edx`                     ;  清空 `edx`  

004AC448   |.  8AD5        |`mov dl,ch`                       ;  n30 的第二位”2“入 `dl` 记为 n77  

004AC44A   |.  33C9        |`xor ecx,ecx`                     ;  清空 `ecx`  

004AC44C   |.  8A4C24 16   |`mov cl,byte ptr ss:[esp+16]`     ;  n20 的第二位”H“入 `cl` 记为 n78  

004AC450   |.  897424 14   |`mov dword ptr ss:[esp+14],esi`   ;  n54 入内存地址 0012ebfc "84e98b59"  

004AC454   |.  8B1495 4867>|`mov edx,dword ptr ds:[edx*4+526`;  n77*4+526748 查表得”23654623“记为 n79 入 `edx`  

004AC45B   |.  8B7424 64   |`mov esi,dword ptr ss:[esp+64]`   ;  n41 入 `esi`  

004AC45F   |.  8B1C8D 4863>|`mov ebx,dword ptr ds:[ecx*4+526`;  n78*4+526348 查表得”F6A45252“记为 n80 入 `ebx`  

004AC466   |.  33C9        |`xor ecx,ecx`                     ;  清空 `ecx`  

004AC468   |.  8A4C24 1B   |`mov cl,byte ptr ss:[esp+1B]`     ;  n10 的第一位"0D"入 `cl` 记为 n81  

004AC46C   |.  33D3        |`xor edx,ebx`                     ;  n79 与 n80 作异或运算记为 n82”D5C11471“  

004AC46E   |.  8B1C8D 485F>|`mov ebx,dword ptr ds:[ecx*4+525`;  n81*4+525f48 查表得"B5D7D762"记为 n83 入 `ebx`  

004AC475   |.  8B4C24 10   |`mov ecx,dword ptr ss:[esp+10]`   ;  n40 入 `ecx`  

004AC479   |.  81E1 FF0000>|`and ecx,0FF`                     ;  n40 与 0FF 作与运算记为 n84  

004AC47F   |.  33D3        |`xor edx,ebx`                     ;  n82 与 n83 作异或运算记为 n85"6016C313"  

004AC481   |.  894424 10   |`mov dword ptr ss:[esp+10],eax`   ;  n76 入内存地址 0012ebf8  

004AC485   |.  8B1C8D 486B>|`mov ebx,dword ptr ds:[ecx*4+526`;  n84*4+526b48 查表得"18182830"记为 n86 入 `ebx`  

004AC48C   |.  8BCF        |`mov ecx,edi`                     ;  n65 入 `ecx`  

004AC48E   |.  33D3        |`xor edx,ebx`                     ;  n85 与 n86 作异或运算记为 n87”780EEB23"  

004AC490   |.  8B5D D8     |`mov ebx,dword ptr ss:[ebp-28]`   ;  b1 入 `ebx`  

004AC493   |.  33D3        |`xor edx,ebx`                     ;  n87 与 n86 作异或运算记为 n88“1B66B46E”  

004AC495   |.  4E          |`dec esi`                         ;  n41 减一  

004AC496   |.  895424 18   |`mov dword ptr ss:[esp+18],edx`   ;  n88“1B66B46E"入内存地址 0012ec00  

004AC49A   |.  894C24 1C   |`mov dword ptr ss:[esp+1C],ecx`   ;  n65"1F658837"入内存地址 0012ec04  

004AC49E   |.  897424 64   |`mov dword ptr ss:[esp+64],esi`   ;  n41 入内存地址 0012ec4c  

004AC4A2   |.^ 0F85 DBFEFF>\\`jnz 1.004AC383`                ;  没有完则继续。循环 14 次  

004AC4A8   |.  8B7424 20   `mov esi,dword ptr ss:[esp+20]`    ;  n41 入 `esi`  

004AC4AC   |.  8B6C24 24   `mov ebp,dword ptr ss:[esp+24]`  

004AC4B0   |>   C1E6 05    `shl esi,5`                         ;  n41 左移 5 次记为 m1  

004AC4B3   |.  33D2        `xor edx,edx`                       ;  清空 `edx`  

004AC4B5   |.  8B7C2E 38   `mov edi,dword ptr ds:[esi+ebp+38]`

----------------------------------------------------------------------------------------  

004AC643   |.  64:890D 000>`mov dword ptr fs:[0],ecx`  

004AC64A   |.  83C4 50     `add esp,50`  

004AC64D   |\\.  C2 0800   `retn 8`   以上都是移位查表操作  

得”D6A6ECDB19D8CD47AFC0134DA67B39F5“

-----------------------------------------------------------------------------------------  

00430EC9   |.  E8 42FEFFFF `call 1.00430D10`                  ;  \\1.00430D10  第二步  

00430ECE   |.  8D4C24 0C   `lea ecx,dword ptr ss:[esp+C]`  

00430ED2   |.  8D4424 3C   `lea eax,dword ptr ss:[esp+3C]`  

第二步也是以上的计算 `call`。a1 为户名+字符串“nover"。  

结果得："8B631E85CC05BFAAF9904D1A5ED93140"  

00430DD6   |.  FF52 08     `call dword ptr ds:[edx+8]`        ;  1.004AD380  第三步  

00430DD9   |.  8D7C1B 01   `lea edi,dword ptr ds:[ebx+ebx+1]`  

00430DDD   |.  8BC7        `mov eax,edi`  

00430DDF   |.  83C0 03     `add eax,3`  

第三步也是以上的计算 `call` 经过两次。a1 为注册的 Email+字符串“nover"。  

结果得："53A25FA9C297A638BE8B4610C03E88CC2D91F21842793A3A91BA2DBF7F2E0539"  

00430EFC   |.  E8 9B920C00 `call <jmp.&MFC42.#4129_CString::Left>` 从左到右取第三步的两位  

00430F01   |.  8BF8        `mov edi,eax`  

00430F03   |.  8D4424 24   `lea eax,dword ptr ss:[esp+24]`  

00430F07   |.  6A 02       `push 2`  

00430F09   |.  50         `push eax`  

00430F0A   |.  8D4C24 18   `lea ecx,dword ptr ss:[esp+18]`  

00430F0E   |.  C64424 3C 0>`mov byte ptr ss:[esp+3C],6`  

00430F13   |.  E8 84920C00 `call <jmp.&MFC42.#4129_CString::Left>` 从左到右取第二步的两位  

00430F18   |.  8BE8        `mov ebp,eax`  

00430F1A   |.  8D4C24 20   `lea ecx,dword ptr ss:[esp+20]`  

00430F1E   |.  6A 02       `push 2`  

00430F20   |.  51         `push ecx`  

00430F21   |.  8D4C24 1C   `lea ecx,dword ptr ss:[esp+1C]`  

00430F25   |.  C64424 3C 0>`mov byte ptr ss:[esp+3C],7`  

00430F2A   |.  E8 6D920C00 `call <jmp.&MFC42.#4129_CString::Left>` 从左到右取第一步的两位  

00430F2F   |.  55         `push ebp`

-------------------------------------------------------------------  

00430F46   |.  C64424 3C 0>`mov byte ptr ss:[esp+3C],9`  

00430F4B   |.  50         `push eax`  

00430F4C   |.  E8 E5910C00 `call <jmp.&MFC42.#922_operator+>`  连起来"D68B53"  

第四步也是以上的计算 `call`，a1 为"D68B53"，结果得“9F91F6123A3B244C29CB676A505B3329”  

00430FAF   |.  E8 E8910C00 `call <jmp.&MFC42.#4129_CString::Left>`   （1）  

00430FB4   |.  50         `push eax`  

00430FB5   |.  8BCF        `mov ecx,edi`  

00430FB7   |.  C64424 38 0>`mov byte ptr ss:[esp+38],0F`  

（1）从“9F91F6123A3B244C29CB676A505B3329”左取 7 位“9F91F61”第一组注册码  

---------------------------------------------------------------------------------  

004455B1    .  51         `push ecx`                                    ; /s2 = "9F91F61"  真码  

004455B2    .  50         `push eax`                                    ; |s1 = "1111111"  输入的假码  

004455B3    .  FF15 EC9051>`call dword ptr ds:[<&MSVCRT._mbscmp>]`     ; \\_mbscmp     比较  

004455D2    .  84C0        `test al,al`  

004455D4    .  74 0A      `je short 1.004455E0`  比较不通过这里就不跳，我这里让她跳。z=1  

004455D6    .  E8 49480B00 `call <jmp.&MFC42.#4853_CDialog::OnOK>`没反应  

004455DB    .  E9 20060000 `jmp 1.00445C00`  

-----------------------------------------------------------------------------------  

0044563E    .  68 649F5500 `push 1.00559F64`                           ;  ASCII "nover"  

00445643    .  E8 38490B00 `call <jmp.&MFC42.#537_CString::CString>`  

00445648    .  8BCF        `mov ecx,edi`                                ; |  

0044564A    .  C64424 60 0>`mov byte ptr ss:[esp+60],5`               ; |  

0044564F    .  E8 FCB9FEFF `call 1.00431050`                           ; \\1.00431050  

004310B2   |.  E8 4D950C00 `call <jmp.&MFC42.#4203_CString::MakeRevers>` 把字符"nover"换位  

004310B7   |.  8D4C24 40   `lea ecx,dword ptr ss:[esp+40]`  

004310BB   |.  E8 44950C00 `call <jmp.&MFC42.#4203_CString::MakeRevers>`把注册名换位  

004310C0   |.  8D4C24 44   `lea ecx,dword ptr ss:[esp+44]`  

004310C4   |.  E8 3B950C00 `call <jmp.&MFC42.#4203_CString::MakeRevers>`把注册 Email 换位  

第五步也是以上的计算 `call`,a1 为换位后的字符“revon",  

结果得”4234CE3C13C0914CF022453D51D882AF“  

004310FE   |.  E8 0DFCFFFF `call 1.00430D10`                           ; \\1.00430D10  

00431103   |.  8D4424 0C   `lea eax,dword ptr ss:[esp+C]`  

00431107   |.  8D5424 44   `lea edx,dword ptr ss:[esp+44]`  

第六步也是以上的计算 `call`，a1 为换位后的注册用户名  

结果得”3B83606EAC15FBA5C4879DCB1F65295B"  

第七步也是以上的计算 `call`,a1 为换位后的注册的 Email  

结果得“3EE94FA04CB773F06CCC53FF6DCC7F7F”  

0043112B   |.  E8 6C900C00 `call <jmp.&MFC42.#4129_CString::Left>` 左取第七步的两位  

00431130   |.  8BF8        `mov edi,eax`  

00431132   |.  8D4C24 24   `lea ecx,dword ptr ss:[esp+24]`  

00431136   |.  6A 02       `push 2`  

00431138   |.  51         `push ecx`  

00431139   |.  8D4C24 18   `lea ecx,dword ptr ss:[esp+18]`  

0043113D   |.  C64424 3C 0>`mov byte ptr ss:[esp+3C],6`  

00431142   |.  E8 55900C00 `call <jmp.&MFC42.#4129_CString::Left>`左取第六步的两位  

00431147   |.  8BE8        `mov ebp,eax`  

00431149   |.  8D5424 20   `lea edx,dword ptr ss:[esp+20]`  

0043114D   |.  6A 02       `push 2`  



### 汇编分析与注册码计算

0043114F   |.  52          push edx

00431150   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

00431154   |.  C64424 3C 0>mov byte ptr ss:[esp+3C],7

00431159   |.  E8 3E900C00 call <jmp.&MFC42.#4129\_CString::Left> 左取第五步的两位

-----------------------------------------------------------------------------------

0043116A   |.  E8 C78F0C00 call <jmp.&MFC42.#922\_operator+>   连起来

0043116F   |.  57          push edi

00431170   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

00431174   |.  50          push eax

00431175   |.  51          push ecx

00431176   |.  C64424 40 0>mov byte ptr ss:[esp+40],9

0043117B   |.  E8 B68F0C00 call <jmp.&MFC42.#922\_operator+>

004311D0   |.  E8 3BFBFFFF call 1.00430D10                     ; \\1.00430D10

004311D5   |.  8D4424 48   lea eax,dword ptr ss:[esp+48]

#### 第八步

也是以上的计算 `call`, `a1` 为连起来的字符 `"423B3E"`

结果得 `"392F62291D9B740975738C332B748AC6"`

00445667     .  E8 AE4B0B00 call <jmp.&MFC42.#4278\_CString::Mid> （2）

0044566C     .  8B5424 30   mov edx,dword ptr ss:[esp+30]

00445670     .  8B00        mov eax,dword ptr ds:[eax]

（2）左取字符串 `"392F62291D9B740975738C332B748AC6"` 的七位 `"392F622"` 为第二组注册码

00445672     .  52          push edx                             ; /s2 = "392F622"  真码

00445673     .  50          push eax                             ; |s1 = "2222222"  假码

00445674     .  FF15 EC9051>call dword ptr ds:[<&MSVCRT.\_mbscmp>]     ; \\\_mbscmp        比较

0044567A     .  83C4 08     add esp,8

0044567D     .  8D4C24 2C   lea ecx,dword ptr ss:[esp+2C]

00445681     .  85C0        test eax,eax

00445683     .  0F954424 13 setne byte ptr ss:[esp+13]

00445688     .  E8 E7480B00 call <jmp.&MFC42.#800\_CString::~CString>

0044568D     .  8A4424 13   mov al,byte ptr ss:[esp+13]

00445691     .  8BCE        mov ecx,esi

00445693     .  84C0        test al,al

00445695     .  74 6F       je short 1.00445706        这里我让她跳   z=1

00445697     .  E8 88470B00 call <jmp.&MFC42.#4853\_CDialog::OnOK> 没反应

0044574F     .  68 649F5500 push 1.00559F64                     ;   ASCII "nover"

00445754     .  E8 27480B00 call <jmp.&MFC42.#537\_CString::CString>

00445759     .  8BCF        mov ecx,edi                          ; |

0044575B     .  C64424 60 0>mov byte ptr ss:[esp+60],5           ; |

00445760     .  E8 1BBBFEFF call 1.00431280                     ; \\1.00431280

#### 第九步

重复第一步的计算，得 `"D6A6ECDB19D8CD47AFC0134DA67B39F5"`

#### 第十步

也是以上的计算 `call`, `a1` 为注册的用户名的大写。

结果得 `"297178FA0064B64F91E77F6F992D140E"`

#### 第十一步

也是以上的计算 `call`, `a1` 为注册的 Email 的大写

结果得 `"64E44D025F74192518B9E65AEC1DD4C7"`

0043135B   |.  E8 3C8E0C00 call <jmp.&MFC42.#4129\_CString::Left> 左取第十一步的两位

00431360   |.  8BF8        mov edi,eax

00431362   |.  8D4C24 24   lea ecx,dword ptr ss:[esp+24]

00431366   |.  6A 02       push 2

00431368   |.  51          push ecx

00431369   |.  8D4C24 18   lea ecx,dword ptr ss:[esp+18]

0043136D   |.  C64424 3C 0>mov byte ptr ss:[esp+3C],6

00431372   |.  E8 258E0C00 call <jmp.&MFC42.#4129\_CString::Left> 左取第十步的两位

00431377   |.  8BE8        mov ebp,eax

00431379   |.  8D5424 20   lea edx,dword ptr ss:[esp+20]

0043137D   |.  6A 02       push 2

0043137F   |.  52          push edx

00431380   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

00431384   |.  C64424 3C 0>mov byte ptr ss:[esp+3C],7

00431389   |.  E8 0E8E0C00 call <jmp.&MFC42.#4129\_CString::Left> 左取第九步的两位

0043139A   |.  E8 978D0C00 call <jmp.&MFC42.#922\_operator+>

0043139F   |.  57          push edi

004313A0   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

004313A4   |.  50          push eax

004313A5   |.  51          push ecx

004313A6   |.  C64424 40 0>mov byte ptr ss:[esp+40],9

004313AB   |.  E8 868D0C00 call <jmp.&MFC42.#922\_operator+>   连起来

00431400   |.  E8 0BF9FFFF call 1.00430D10                     ; \\1.00430D10

00431405   |.  8D4424 48   lea eax,dword ptr ss:[esp+48]

#### 第十二步

也是以上的计算 `call`, `a1` 为连起来的字符 `"D62964"`

结果得 `"3F943C4337ED46F167F6DEDEA82D9D0B"`

00445778     .  E8 9D4A0B00 call <jmp.&MFC42.#4278\_CString::Mid> （3）

0044577D     .  8B4C24 30   mov ecx,dword ptr ss:[esp+30]

00445781     .  8B00        mov eax,dword ptr ds:[eax]

（3）左取第十二步的 `"3F943C4337ED46F167F6DEDEA82D9D0B"` 七位 `"3F943C4"`

00445783     .  51          push ecx                             ; /s2 = "3F943C4"  真码

00445784     .  50          push eax                             ; |s1 = "3333333"  假码

00445785     .  FF15 EC9051>call dword ptr ds:[<&MSVCRT.\_mbscmp>]     ; \\\_mbscmp        比较

0044578B     .  83C4 08     add esp,8

0044578E     .  8D4C24 2C   lea ecx,dword ptr ss:[esp+2C]

00445792     .  85C0        test eax,eax

00445794     .  0F954424 13 setne byte ptr ss:[esp+13]

00445799     .  E8 D6470B00 call <jmp.&MFC42.#800\_CString::~CString>

0044579E     .  8A4424 13   mov al,byte ptr ss:[esp+13]

004457A2     .  8BCE        mov ecx,esi

004457A4     .  84C0        test al,al

004457A6     .  74 6F       je short 1.00445817    这里我让她跳      z=1

004457A8     .  E8 77460B00 call <jmp.&MFC42.#4853\_CDialog::OnOK>

----------------------------------------------------------------------------------------

00445859     .  68 649F5500 push 1.00559F64                     ;   ASCII "nover"

0044585E     .  E8 1D470B00 call <jmp.&MFC42.#537\_CString::CString>

00445863     .  8BCF        mov ecx,edi                           ; |

00445865     .  C64424 60 0>mov byte ptr ss:[esp+60],5           ; |

0044586A     .  E8 41BCFEFF call 1.004314B0                     ; \\1.004314B0

#### 第十三步

重复第一步的计算，得 `"D6A6ECDB19D8CD47AFC0134DA67B39F5"`

#### 第十四步

也是以上的计算 `call`, `a1` 为注册的用户名 + 这个软件的主页 `[www.crob.net](http://www.crob.net)`

结果得 `"1EB959796972DEB2BDA60700CABCBF37"`

#### 第十五步

也是经上的计算 `call`, `a1` 为注册的 Email

结果得 `"4F21187873592D0C9C4DF884A40336E1"`

00431576   |.  E8 218C0C00 call <jmp.&MFC42.#4129\_CString::Left>  左取第十五步的两位

0043157B   |.  8BF8        mov edi,eax

0043157D   |.  8D4C24 24   lea ecx,dword ptr ss:[esp+24]

00431581   |.  6A 02       push 2

00431583   |.  51          push ecx

00431584   |.  8D4C24 18   lea ecx,dword ptr ss:[esp+18]

00431588   |.  C64424 3C 0>mov byte ptr ss:[esp+3C],6

0043158D   |.  E8 0A8C0C00 call <jmp.&MFC42.#4129\_CString::Left>   左取第十四步的两位

00431592   |.  8BE8        mov ebp,eax

00431594   |.  8D5424 20   lea edx,dword ptr ss:[esp+20]

00431598   |.  6A 02       push 2

0043159A   |.  52          push edx

0043159B   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

0043159F   |.  C64424 3C 0>mov byte ptr ss:[esp+3C],7

004315A4   |.  E8 F38B0C00 call <jmp.&MFC42.#4129\_CString::Left> 左取第十三步的两位

004315B5   |.  E8 7C8B0C00 call <jmp.&MFC42.#922\_operator+>

004315BA   |.  57          push edi

004315BB   |.  8D4C24 1C   lea ecx,dword ptr ss:[esp+1C]

004315BF   |.  50          push eax

004315C0   |.  51          push ecx

004315C1   |.  C64424 40 0>mov byte ptr ss:[esp+40],9

004315C6   |.  E8 6B8B0C00 call <jmp.&MFC42.#922\_operator+>   连起来

#### 第十六步

也是以上的计算 `call`, `a1` 为连起来的字符 `"D61E4F"`

结果得 `"4666A01E1E4C445A68287B0A90E1578A"`

0044587C     .  E8 15490B00 call <jmp.&MFC42.#5710\_CString::Right>    (4)

00445881     .  8B00        mov eax,dword ptr ds:[eax]

00445883     .  57          push edi                             ; /s2 = "4666A01"

00445884     .  50          push eax                             ; |s1 = "4444444"

00445885     .  FF15 EC9051>call dword ptr ds:[<&MSVCRT.\_mbscmp>]     ; \\\_mbscmp  比较

0044588B     .  83C4 08     add esp,8

0044588E     .  8D4C24 30   lea ecx,dword ptr ss:[esp+30]

00445892     .  85C0        test eax,eax

00445894     .  0F954424 13 setne byte ptr ss:[esp+13]

00445899     .  E8 D6460B00 call <jmp.&MFC42.#800\_CString::~CString>

0044589E     .  8A4424 13   mov al,byte ptr ss:[esp+13]

004458A2     .  8BCE        mov ecx,esi



### 汇编分析

```
004458A4    .  84C0        test al,al  
004458A6    .  74 6F      je short 1.00445917    这里这让她跳 z=1  
004458A8    .  E8 77450B00 call <jmp.&MFC42.#4853_CDialog::OnOK>  
```

(4) 取第十六步的结果的七位“`4666A01`”，为最后一组注册码。

到这里已经完成了所有的计算过程，把正确的注册信息写入文件中。但是她还会重复一次以上所有的步骤。呵呵，真变态啊。

把 `00578048` 处一直置 0 则为未注册版反之嘿嘿。

=====================================================================

### 数据库注册

这个软件的数据库还要另外注册，晕。只好在跟了下。

注册算法 CALL 还是以上的注册算法 CALL。第一步 `A1` 为“`nover`”，`b1` 为“`745F6863`”，`B2` 为“`745E2424`”，`B3` 为“`46534466`”，`B4` 为“`59544447`”。

得结果为“`DFECF5F8BB082BC4EA49C08170503F87`”。

第二步：还是以上的注册算法 CALL。`A1` 为注册用户名+“`nover`”，

得结果“`29580E2BF8383152B09F6223A1D3887B`”

第三步：还是以上的注册算法 CALL。`A1` 为注册的 `EMAIL`+“`nover`”，

如果字节超过 16 位，则取前 16 位计算完后再取俐下的字节继续。

得结果为“`8A5758373B47216428BD0B8B BD2CC31B6D59DD97CB8859B0521594C37659C465`”。

以下的步骤和上面都差不多。

* 用户名：`落魄浪子`
* 注册 `Email`：`Pediy_Crack@21cn.net`
* 注册码：`2ED3601-8E5CF6E-111E8F8-1C9B4A0`

--------------------------------------------------------------------------------

【破解总结】

注册算法实在长，但却简单。注册机会了吧，内存注册也可。呵呵，写的不好，错误之处敬请指教。

给出一组注册码:
* 用户名：`落魄浪子`
* 注册 `Email`：`Pediy_Crack@21cn.net`
* 注册码：`57D0BF5-0C687AF-EF123D1-CE92408`

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

那上面一大段是 `AES`

```
4dd466d0e1e0cc123afbe504da2462ca
```
