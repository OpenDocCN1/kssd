
# 《图章制作系统 V3.63》注册验证破解分析[算法篇]

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-331.htm`](https://www.pediy.com/kssd/pediy07/pediy7-331.htm)

【破文标题】:《图章制作系统 V3.63》注册验证破解分析[算法篇]  

【破文作者】: `KuNgBiM[DFCG]`  

【作者邮箱】: `gb_1227@163.com`  

【软件名称】: 图章制作系统 V3.63  

【整理时间】: 2005-07-29  

【下载地址】: `http://www.downreg.com/Software/View-Software-4587.html`  

【保护方式】: 注册码 + 试用功能限制  

【加密保护】: `ASPack 2.12` + 脱壳自校验 + 程序自杀代码(调用系统 `autoexec.bat` 命令删除校验失败的程序) + `Anti-Loader`(反加载)  

【编译语言】: `Borland Delphi 6.0 - 7.0`  

【调试环境】: `WinXP`、`PEiD`、`Ollydbg`、`LordPE`、`ImportREC`  

【破解目的】: 推广使用 `ESP` 定律脱壳，去除自校验，以及研究算法分析  

【作者声明】: 初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

—————————————————————————————————

\\ 【破解过程】 \\

试炼信息：

用户 id：`05809b6d5f477dc666db808b52e86086`

注册 id：`98765432109876543210987654321098`

—————————————————————————————————

●上篇我们脱壳去校验的主程序我从新命名为了“`脱壳去校验_MakeSign.exe`”便于区分●

`OD` 载入脱壳去校验的主程序后，使用“`Ultra String Reference`”插件的“`Find ASCII`”功能项查找“注册失败！注册码错误”：

```
0057E6BC    55                     push ebp                             ; 来到此处 F2 下断，F9 运行，添入试炼信息
0057E6BD    68 B0E75700            push 脱壳去校.0057E7B0
0057E6C2    64:FF30               push dword ptr fs:[eax]
0057E6C5    64:8920               mov dword ptr fs:[eax],esp
0057E6C8    8D55 F8               lea edx,dword ptr ss:[ebp-8]
0057E6CB    8BB3 00030000         mov esi,dword ptr ds:[ebx+300]
0057E6D1    8BC6                  mov eax,esi
0057E6D3    E8 08FBECFF           call 脱壳去校.0044E1E0                 ; 取“注册 id”，长度送 EAX
0057E6D8    8B45 F8               mov eax,dword ptr ss:[ebp-8]           ;“注册 id”送 EAX，eax=00000020
0057E6DB    8D55 FC               lea edx,dword ptr ss:[ebp-4]
0057E6DE    E8 19A8E8FF           call 脱壳去校.00408EFC                 ; 检测“注册 id”是否合法
0057E6E3    8B55 FC               mov edx,dword ptr ss:[ebp-4]           ;“注册 id”送 EDX，ASCII "98765432109876543210987654321098"
0057E6E6    8BC6                  mov eax,esi
0057E6E8    E8 23FBECFF           call 脱壳去校.0044E210
0057E6ED    8D55 F4               lea edx,dword ptr ss:[ebp-C]
0057E6F0    8B83 00030000         mov eax,dword ptr ds:[ebx+300]
0057E6F6    E8 E5FAECFF           call 脱壳去校.0044E1E0
0057E6FB    837D F4 00            cmp dword ptr ss:[ebp-C],0             ; 注册码是否为空
0057E6FF    0F84 88000000         je 脱壳去校.0057E78D                   ; 为空则跳死！
0057E705    B9 C8E75700           mov ecx,脱壳去校.0057E7C8              ; 否则"HsjSoft.ini"送 ECX，ASCII "HsjSoft.ini"
0057E70A    B2 01                 mov dl,1                               ; DL 置 1
0057E70C    A1 04084700           mov eax,dword ptr ds:[470804]
0057E711    E8 9E21EFFF           call 脱壳去校.004708B4
0057E716    8BF0                  mov esi,eax
0057E718    8D55 F0               lea edx,dword ptr ss:[ebp-10]
0057E71B    8B83 00030000         mov eax,dword ptr ds:[ebx+300]
0057E721    E8 BAFAECFF           call 脱壳去校.0044E1E0
0057E726    8B45 F0               mov eax,dword ptr ss:[ebp-10]          ;“注册 id”送 EAX
0057E729    50                    push eax                               ;“注册 id”压栈，ASCII "98765432109876543210987654321098"
0057E72A    B9 DCE75700           mov ecx,脱壳去校.0057E7DC              ; "reg_code"送 ECX，ASCII "reg_code"
0057E72F    8B93 10030000         mov edx,dword ptr ds:[ebx+310]
0057E735    8BC6                  mov eax,esi
0057E737    8B38                  mov edi,dword ptr ds:[eax]
0057E739    FF57 04               call dword ptr ds:[edi+4]
0057E73C    8BC6                  mov eax,esi
0057E73E    E8 0951E8FF           call 脱壳去校.0040384C
0057E743    8B83 10030000         mov eax,dword ptr ds:[ebx+310]
0057E749    E8 F60D0000          call 脱壳去校.0057F544                 ; ★验证关键 CALL，跟进！★
0057E74E    84C0                  test al,al                             ; AL 是否为 0，（如果刚才比较条件为真则 AL 为 1）
0057E750    75 1B                 jnz short 脱壳去校.0057E76D            ; 关键跳转！注册码比较后若正确则跳向"注册成功" 0057E76D 处
0057E752    6A 00                 push 0
0057E754    68 E8E75700           push 脱壳去校.0057E7E8
0057E759    68 F0E75700           push 脱壳去校.0057E7F0                 ; "注册失败！\n 注册码错误"   <-- 双击来到这里
0057E75E    8BC3                  mov eax,ebx
0057E760    E8 BF62EDFF           call 脱壳去校.00454A24
0057E765    50                    push eax
0057E766    E8 D990E8FF           call 脱壳去校.00407844                 ; jmp to user32.MessageBoxA   <-- 注册失败提示框！
0057E76B    EB 20                 jmp short 脱壳去校.0057E78D
0057E76D    6A 00                 push 0
0057E76F    68 08E85700           push 脱壳去校.0057E808
0057E774    68 10E85700           push 脱壳去校.0057E810                 ; "注册成功！"
0057E779    8BC3                  mov eax,ebx
0057E77B    E8 A462EDFF           call 脱壳去校.00454A24
0057E780    50                    push eax
0057E781    E8 BE90E8FF           call 脱壳去校.00407844                 ; jmp to user32.MessageBoxA   <-- 注册成功提示框！
0057E786    8BC3                  mov eax,ebx
0057E788    E8 3FD3EEFF           call 脱壳去校.0046BACC
0057E78D    33C0                  xor eax,eax
0057E78F    5A                    pop edx
0057E790    59                    pop ecx
0057E791    59                    pop ecx
0057E792    64:8910               mov dword ptr fs:[eax],edx
0057E795    68 B7E75700           push 脱壳去校.0057E7B7
0057E79A    8D45 F0               lea eax,dword ptr ss:[ebp-10]
0057E79D    BA 03000000           mov edx,3
0057E7A2    E8 B95EE8FF           call 脱壳去校.00404660
0057E7A7    8D45 FC               lea eax,dword ptr ss:[ebp-4]
0057E7AA    E8 8D5EE8FF           call 脱壳去校.0040463C
0057E7AF    C3                    retn                                   ; 返回程序
\================= 跟进：0057E749    E8 F60D0000           call 脱壳去校.0057F544 ================
0057F544    55                    push ebp
0057F545    8BEC                  mov ebp,esp
0057F547    33C9                  xor ecx,ecx
0057F549    51                    push ecx
0057F54A    51                    push ecx
0057F54B    51                    push ecx
0057F54C    51                    push ecx
0057F54D    51                    push ecx
0057F54E    53                    push ebx
0057F54F    56                    push esi
0057F550    8945 FC               mov dword ptr ss:[ebp-4],eax           ; 取固定字符串, (ASCII "HsjMakeSign")
0057F553    8B45 FC               mov eax,dword ptr ss:[ebp-4]
0057F556    E8 9155E8FF           call 脱壳去校.00404AEC
0057F55B    33C0                  xor eax,eax
0057F55D    55                    push ebp
0057F55E    68 F8F55700           push 脱壳去校.0057F5F8
0057F563    64:FF30               push dword ptr fs:[eax]
0057F566    64:8920               mov dword ptr fs:[eax],esp
0057F569    E8 7AF6FFFF           call 脱壳去校.0057EBE8
0057F56E    84C0                  test al,al                             ; al=00
0057F570    74 0E                 je short 脱壳去校.0057F580
0057F572    A1 A48B5900           mov eax,dword ptr ds:[598BA4]
0057F577    8B00                  mov eax,dword ptr ds:[eax]
0057F579    E8 76FDEEFF           call 脱壳去校.0046F2F4
0057F57E    EB 5D                 jmp short 脱壳去校.0057F5DD
0057F580    8D55 F4               lea edx,dword ptr ss:[ebp-C]           ; edx=2
0057F583    8B45 FC               mov eax,dword ptr ss:[ebp-4]           ; 固定字符串送 EAX
0057F586    E8 C1FDFFFF           call 脱壳去校.0057F34C
0057F58B    8D55 F8               lea edx,dword ptr ss:[ebp-8]
0057F58E    8B45 F4               mov eax,dword ptr ss:[ebp-C]           ; “用户 id”送 EAX，ASCII "05809b6d5f477dc666db808b52e86086"
```



```
0057F591    E8 BEFEFFFF           call 脱壳去校.0057F454                      ; ★验证关键 CALL，跟进！★  

0057F596    B9 10F65700           mov ecx,脱壳去校.0057F610                   ; 返回信息到"HsjSoft.ini"送 ECX，ASCII "HsjSoft.ini"  

0057F59B    B2 01                 mov dl,1                                   ; DL 置 1  

0057F59D    A1 04084700           mov eax,dword ptr ds:\[470804\]  

0057F5A2    E8 0D13EFFF           call 脱壳去校.004708B4  

0057F5A7    8BD8                  mov ebx,eax  

0057F5A9    6A 00                 push 0  

0057F5AB    8D45 EC               lea eax,dword ptr ss:\[ebp\-14\]  

0057F5AE    50                    push eax  

0057F5AF    B9 24F65700           mov ecx,脱壳去校.0057F624                   ; ASCII "reg\_code"  

0057F5B4    8B55 FC               mov edx,dword ptr ss:\[ebp\-4\]  

0057F5B7    8BC3                  mov eax,ebx  

0057F5B9    8B30                  mov esi,dword ptr ds:\[eax\]  

0057F5BB    FF16                  call dword ptr ds:\[esi\]  

0057F5BD    8B45 EC               mov eax,dword ptr ss:\[ebp\-14\]           ; 假码送 EAX，ASCII "98765432109876543210987654321098"  

0057F5C0    8D55 F0               lea edx,dword ptr ss:\[ebp\-10\]  

0057F5C3    E8 3499E8FF           call 脱壳去校.00408EFC  

0057F5C8    8BC3                  mov eax,ebx  

0057F5CA    E8 7D42E8FF           call 脱壳去校.0040384C  

0057F5CF    8B45 F8               mov eax,dword ptr ss:\[ebp\-8\]           ; 真码送 EAX，ASCII "2115997e981b8713f3b6f7124b631a14"  

0057F5D2    8B55 F0               mov edx,dword ptr ss:\[ebp\-10\]          ; 假码送 EAX，ASCII "98765432109876543210987654321098"  

0057F5D5    E8 6E54E8FF           call 脱壳去校.00404A48                     ; 假码和真码比较   ★验证爆破点★  

0057F5DA    0F94C3                sete bl                                    ; 置 BL 值，条件为假 FALSE  

0057F5DD    33C0                  xor eax,eax                                ; EAX 清零  

0057F5DF    5A                    pop edx  

0057F5E0    59                    pop ecx  

0057F5E1    59                    pop ecx  

0057F5E2    64:8910               mov dword ptr fs:\[eax\],edx  

0057F5E5    68 FFF55700           push 脱壳去校.0057F5FF  

0057F5EA    8D45 EC               lea eax,dword ptr ss:\[ebp\-14\]  

0057F5ED    BA 05000000           mov edx,5  

0057F5F2    E8 6950E8FF           call 脱壳去校.00404660  

0057F5F7    C3                    retn  

0057F5F8   ^\E9 E349E8FF           jmp 脱壳去校.00403FE0  

0057F5FD   ^ EB EB                 jmp short 脱壳去校.0057F5EA  

0057F5FF    8BC3                  mov eax,ebx  

0057F601    5E                    pop esi  

0057F602    5B                    pop ebx  

0057F603    8BE5                  mov esp,ebp  

0057F605    5D                    pop ebp  

0057F606    C3                    retn                                     ; 返回程序
```

\================= 跟进：`0057F591    E8 BEFEFFFF           call 脱壳去校.0057F454` =================

```
0057F454    55                    push ebp  

0057F455    8BEC                  mov ebp,esp  

0057F457    83C4 DC               add esp,-24  

0057F45A    53                    push ebx  

0057F45B    56                    push esi  

0057F45C    33C9                  xor ecx,ecx  

0057F45E    894D DC               mov dword ptr ss:\[ebp\-24\],ecx  

0057F461    894D E0               mov dword ptr ss:\[ebp\-20\],ecx  

0057F464    894D F8               mov dword ptr ss:\[ebp\-8\],ecx  

0057F467    894D F4               mov dword ptr ss:\[ebp\-C\],ecx  

0057F46A    8BF2                  mov esi,edx  

0057F46C    8945 FC               mov dword ptr ss:\[ebp\-4\],eax           ; 读取保存在 EAX 中的“用户 id”  

0057F46F    8B45 FC               mov eax,dword ptr ss:\[ebp\-4\]           ; “用户 id”送 EAX，ASCII "05809b6d5f477dc666db808b52e86086"  

0057F472    E8 7556E8FF           call 脱壳去校.00404AEC  

0057F477    33C0                  xor eax,eax  

0057F479    55                    push ebp  

0057F47A    68 1AF55700           push 脱壳去校.0057F51A  

0057F47F    64:FF30               push dword ptr fs:\[eax\]  

0057F482    64:8920               mov dword ptr fs:\[eax\],esp  

0057F485    8BC6                  mov eax,esi  

0057F487    E8 B051E8FF           call 脱壳去校.0040463C  

0057F48C    8D45 E0               lea eax,dword ptr ss:\[ebp\-20\]  

0057F48F    8B4D FC               mov ecx,dword ptr ss:\[ebp\-4\]           ; “用户 id”送 ECX，ASCII "05809b6d5f477dc666db808b52e86086"  

0057F492    BA 30F55700           mov edx,脱壳去校.0057F530  

0057F497    E8 B454E8FF           call 脱壳去校.00404950  

0057F49C    8B45 E0               mov eax,dword ptr ss:\[ebp\-20\]          ; “用户 id”位数送 EAX，eax=00000020  

0057F49F    8D55 E4               lea edx,dword ptr ss:\[ebp\-1C\]  

0057F4A2    E8 71EEFFFF           call 脱壳去校.0057E318  

0057F4A7    8D45 E4               lea eax,dword ptr ss:\[ebp\-1C\]  

0057F4AA    8D55 F8               lea edx,dword ptr ss:\[ebp\-8\]  

0057F4AD    E8 DAEEFFFF           call 脱壳去校.0057E38C                     ; ★计算关键 CALL，跟进！★  

0057F4B2    8D45 F4               lea eax,dword ptr ss:\[ebp\-C\]           ; 运算值返回到这里  

0057F4B5    E8 8251E8FF           call 脱壳去校.0040463C  

0057F4BA    8B45 F8               mov eax,dword ptr ss:\[ebp\-8\]           ; 运算值送 EAX，ASCII "41a136b4217f6b3f3178b189e7995112"  

0057F4BD    E8 4254E8FF           call 脱壳去校.00404904                     ; 检测运算值是否为空，不是则取运算值长度并送 EAX  

0057F4C2    8BD8                  mov ebx,eax                                ; EAX 送 EBX  

0057F4C4    83FB 01               cmp ebx,1                                  ; EBX 和 1 比较，ebx=00000020  

0057F4C7    7C 1F                 jl short 脱壳去校.0057F4E8                 ; 若值小于 1 则直接跳死  

0057F4C9    8D45 DC               lea eax,dword ptr ss:\[ebp\-24\]           ; 否则，进行下一步计算  ★ 循环开始 ★  

0057F4CC    8B55 F8               mov edx,dword ptr ss:\[ebp\-8\]           ; 运算值送 EDX，ASCII "41a136b4217f6b3f3178b189e7995112"  

0057F4CF    8A541A FF             mov dl,byte ptr ds:\[edx+ebx\-1\]         ; \[edx+ebx-1\]送 DL，字符串倒序运算  

0057F4D3    E8 4453E8FF           call 脱壳去校.0040481C  

0057F4D8    8B55 DC               mov edx,dword ptr ss:\[ebp\-24\]  

0057F4DB    8D45 F4               lea eax,dword ptr ss:\[ebp\-C\]  

0057F4DE    E8 2954E8FF           call 脱壳去校.0040490C  

0057F4E3    4B                    dec ebx                                    ; EBX 自减 1，指向下一位，确保计算正常  

0057F4E4    85DB                  test ebx,ebx                               ; 检测 EBX 值是否减去 1，并且是否为 0  

0057F4E6   ^ 75 E1                 jnz short 脱壳去校.0057F4C9              ; 不为 0 就继续  

0057F4E8    8BC6                  mov eax,esi  

0057F4EA    8B55 F4               mov edx,dword ptr ss:\[ebp\-C\]           ; 运算完毕，真码出现，ASCII "2115997e981b8713f3b6f7124b631a14"  

0057F4ED    E8 9E51E8FF           call 脱壳去校.00404690  

0057F4F2    33C0                  xor eax,eax  

0057F4F4    5A                    pop edx  

0057F4F5    59                    pop ecx  

0057F4F6    59                    pop ecx  

0057F4F7    64:8910               mov dword ptr fs:\[eax\],edx  

0057F4FA    68 21F55700           push 脱壳去校.0057F521  

0057F4FF    8D45 DC               lea eax,dword ptr ss:\[ebp\-24\]  

0057F502    BA 02000000           mov edx,2  

0057F507    E8 5451E8FF           call 脱壳去校.00404660  

0057F50C    8D45 F4               lea eax,dword ptr ss:\[ebp\-C\]  

0057F50F    BA 03000000           mov edx,3  

0057F514    E8 4751E8FF           call 脱壳去校.00404660  

0057F519    C3                    retn  

0057F51A   ^\E9 C14AE8FF           jmp 脱壳去校.00403FE0  

0057F51F   ^ EB DE                 jmp short 脱壳去校.0057F4FF  

0057F521    5E                    pop esi  

0057F522    5B                    pop ebx  

0057F523    8BE5                  mov esp,ebp  

0057F525    5D                    pop ebp                                    ; 返回验证程序  

0057F526    C3                    retn
```

\================= 跟进：`0057F4AD    E8 DAEEFFFF           call 脱壳去校.0057E38C` =================



0057E38C 55 push ebp
0057E38D 8BEC mov ebp,esp
0057E38F 83C4 E8 add esp,-18
0057E392 53 push ebx
0057E393 56 push esi
0057E394 57 push edi
0057E395 33C9 xor ecx,ecx
0057E397 894D EC mov dword ptr ss:[ebp-14],ecx
0057E39A 894D E8 mov dword ptr ss:[ebp-18],ecx
0057E39D 8BF0 mov esi,eax
0057E39F 8D7D F0 lea edi,dword ptr ss:[ebp-10]
0057E3A2 A5 movs dword ptr es:[edi],dword ptr ds:[esi] ; ds:[esi]=stack [0012F80C]=B436A141
0057E3A3 A5 movs dword ptr es:[edi],dword ptr ds:[esi] ; ds:[esi]=stack [0012F810]=3F6B7F21
0057E3A4 A5 movs dword ptr es:[edi],dword ptr ds:[esi] ; ds:[esi]=stack [0012F814]=89B17831
0057E3A5 A5 movs dword ptr es:[edi],dword ptr ds:[esi] ; ds:[esi]=stack [0012F818]=125199E7
0057E3A6 8BFA mov edi,edx
0057E3A8 33C0 xor eax,eax
0057E3AA 55 push ebp
0057E3AB 68 27E45700 push 脱壳去校.0057E427
0057E3B0 64:FF30 push dword ptr fs:[eax]
0057E3B3 64:8920 mov dword ptr fs:[eax],esp
0057E3B6 8BC7 mov eax,edi
0057E3B8 E8 7F62E8FF call 脱壳去校.0040463C
0057E3BD B3 10 mov bl,10
0057E3BF 8D75 F0 lea esi,dword ptr ss:[ebp-10]
0057E3C2 FF37 push dword ptr ds:[edi] ; ★★★ 循环开始 ★★★
0057E3C4 8D45 EC lea eax,dword ptr ss:[ebp-14]
0057E3C7 33D2 xor edx,edx ; 异或清零，edx=00000000
0057E3C9 8A16 mov dl,byte ptr ds:[esi] ; [esi]送 DL
0057E3CB C1EA 04 shr edx,4 ; EDX 右移 4 位
0057E3CE 83E2 0F and edx,0F ; 和 0F 与
0057E3D1 8A92 C8875900 mov dl,byte ptr ds:[edx+5987C8] ; [edx+5987C8]送 DL
0057E3D7 E8 4064E8FF call 脱壳去校.0040481C
0057E3DC FF75 EC push dword ptr ss:[ebp-14]
0057E3DF 8D45 E8 lea eax,dword ptr ss:[ebp-18]
0057E3E2 8A16 mov dl,byte ptr ds:[esi] ; [esi]送 DL
0057E3E4 80E2 0F and dl,0F ; DL 和 0F 与
0057E3E7 81E2 FF000000 and edx,0FF ; EDX 和 0FF 与
0057E3ED 8A92 C8875900 mov dl,byte ptr ds:[edx+5987C8] ; [edx+5987C8]送 DL
0057E3F3 E8 2464E8FF call 脱壳去校.0040481C
0057E3F8 FF75 E8 push dword ptr ss:[ebp-18]
0057E3FB 8BC7 mov eax,edi ; EDI 送 EAX
0057E3FD BA 03000000 mov edx,3
0057E402 E8 BD65E8FF call 脱壳去校.004049C4
0057E407 46 inc esi ; ESI 自加 1 指向下一位
0057E408 FECB dec bl ; BL 自减 1，确保计算正常
0057E40A 75 B6 jnz short 脱壳去校.0057E3C2 ; BL 不为 0 则继续，循环完后得字符串
0057E40C 33C0 xor eax,eax
0057E40E 5A pop edx
0057E40F 59 pop ecx
0057E410 59 pop ecx
0057E411 64:8910 mov dword ptr fs:[eax],edx
0057E414 68 2EE45700 push 脱壳去校.0057E42E
0057E419 8D45 E8 lea eax,dword ptr ss:[ebp-18]
0057E41C BA 02000000 mov edx,2
0057E421 E8 3A62E8FF call 脱壳去校.00404660
0057E426 C3 retn
0057E42E 5F pop edi
0057E42F 5E pop esi
0057E430 5B pop ebx
0057E431 8BE5 mov esp,ebp
0057E433 5D pop ebp
0057E434 C3 retn ; 返回运算值"41a136b4217f6b3f3178b189e7995112"

---

### 算法总结

“用户 id”通过变形 `MD5` 算法运算得到一字符串，再将此字符串倒序作为注册码

注意：由于该软件使用了“`Anti-Loader`(反加载)”技术，所以，想做内存注册机的话，那是不可能的了，它同样会调用系统 `autoexec.bat` 命令删除被加载的程序。

=============================================================================

### 注册信息

用户 id：`05809b6d5f477dc666db808b52e86086`

注册 id：`2115997e981b8713f3b6f7124b631a14`

#### 注册信息保存位置

`x:\WINDOWS\HsjSoft.ini` （“`x`”为系统盘）删除后又可以玩一次！

`HsjSoft.ini` 内容：

```ini
[HsjMakeSign]
reg_code=2115997e981b8713f3b6f7124b631a14
```

### 完美验证爆破点

0057F5D5 E8 6E54E8FF call 脱壳去校.00404A48 // nop 掉

改为：

```
0057F5D5 90 nop
0057F5D6 90 nop
0057F5D7 90 nop
0057F5D8 90 nop
0057F5D9 90 nop
```

---

版权所有（C）2005 `KuNgBiM[DFCG]` Copyright (C) 2005 `KuNgBiM[DFCG]`

---

UnPacked & Cracked By `KuNgBiM[DFCG]`
2005-08-02
12:15:18 PM
