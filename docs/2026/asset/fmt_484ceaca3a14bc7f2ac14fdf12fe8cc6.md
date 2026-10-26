
# 菜鸟看懂算法以后之三_____快刀英语V1.15算法分析

【破文标题】：菜鸟看懂算法以后之三\_\_\_\_\_快刀英语V1.15算法分析

【破解目标】：快刀英语V1.15

【破解作者】：jney2

【破解日期】：2005年4月26日

【作者声明】：破解，我是菜鸟，更要学习，练习！！！一点心得，大家分享。如有错误，蒙请老大指正。

【破解平台】：XP＋OD

【软件介绍】：软件来自天空软件站
主要功能：“快刀英语”是一款英语辅助学习软件，使用者可以通过单机和网络游戏的形式学习英语单词。
主要特点：把学习变成游戏，寓教于乐。学习效果：快速明显。学习方式：轻松愉快。
*致力解决兴趣淡漠、疲劳厌倦、容易遗忘、士气低落等英语学习者的常见问题。
*基于快速浏览法，一月可认识上千单词，每天对目标词库熟悉程度可提高5个百分点。
*内含妙手空空、多米洛、俄罗斯、闭关修炼等多个游戏。
*个人单机、二人同机、人机对战等多种游戏方式。
*词汇、拼写、语音、句子，都有针对训练，全面提高英语能力。
*强调动手，手脑并用，加深记忆印象。
*生词、易错词自动重复出现，循环记忆，战胜遗忘。
*以目标词库为学习对象，强调全面整体掌握。
*即时统计对目标词库的熟悉程度，每天的进步一目了然。
*大学四级词库分为高中已学未学两类，突出重点，避免重复劳动。
*单词可选择随机学习、字母顺序、字母多少三种学习方式。
*单词可选动词、名词、形容词、副词、其他、所有，可按词类分别学习。
*包括小学、初中、高中、四级、六级、大学词组、研究生入学、托福、GRE、GMAT、新概念英语、职称考试ABC级等词库，词汇总量4万。
*全程语音，自动发音，无需手动操作。
*v1.15版新增自建用户词库功能，快速简便，把需要学习的文章拷贝入文本框内，按一下按钮，就可以把文章里面的所有单词轻松转化为用户词库。
*v1.15版新增大学词组、新概念英语、职称考试英语词库。

## 算法分析

### 一

```
005070E0   .  55             push ebp
005070E1   .  8BEC           mov ebp,esp
005070E3   .  B9 15000000    mov ecx,15
005070E8   >  6A 00          push 0
005070EA   .  6A 00          push 0
005070EC   .  49             dec ecx
005070ED   .^ 75 F9          jnz short 快刀英语.005070E8
005070EF   .  53             push ebx
005070F0   .  56             push esi
005070F1   .  57             push edi
005070F2   .  8BD8           mov ebx,eax
005070F4   .  33C0           xor eax,eax
005070F6   .  55             push ebp
005070F7   .  68 BC745000    push 快刀英语.005074BC
005070FC   .  64:FF30        push dword ptr fs:[eax]
005070FF   .  64:8920        mov dword ptr fs:[eax],esp
00507102   .  BE D0745000    mov esi,快刀英语.005074D0
00507107   .  8D7D BA        lea edi,dword ptr ss:[ebp-46]
0050710A   .  B9 08000000    mov ecx,8
0050710F   .  F3:A5          rep movs dword ptr es:[edi],dword ptr ds:[esi]
00507111   .  A4             movs byte ptr es:[edi],byte ptr ds:[esi]
00507112   .  8D55 B4        lea edx,dword ptr ss:[ebp-4C]
00507115   .  8B83 04030000  mov eax,dword ptr ds:[ebx+304]
0050711B   .  E8 3CD3F5FF    call 快刀英语.0046445C
00507120   .  8B45 B4        mov eax,dword ptr ss:[ebp-4C]                   ；eax=01472814, (ASCII "F4E8-C8D4") 机器码
00507123   .  50             push eax
00507124   .  8D55 B0        lea edx,dword ptr ss:[ebp-50]
00507127   .  8B83 FC020000  mov eax,dword ptr ds:[ebx+2FC]
0050712D   .  E8 2AD3F5FF    call 快刀英语.0046445C
00507132   .  8B55 B0        mov edx,dword ptr ss:[ebp-50]                  ；EDX 0146E75C ASCII "jney2@21cn.com" 邮件地址
00507135   .  8D45 FC        lea eax,dword ptr ss:[ebp-4]
00507138   .  59             pop ecx                                        ；ECX 01472814 ASCII "F4E8-C8D4"
00507139   .  E8 B2D4EFFF    call 快刀英语.004045F0
0050713E   .  6A 20          push 20
00507140   .  8D45 BA        lea eax,dword ptr ss:[ebp-46]
00507143   .  50             push eax
00507144   .  8B45 FC        mov eax,dword ptr ss:[ebp-4]                    ；EAX 0146E778 ASCII "jney2@21cn.comF4E8-C8D4"
00507147   .  E8 58D6EFFF    call 快刀英语.004047A4
0050714C   .  50             push eax
0050714D   .  E8 86FFFFFF    call <jmp.&mcd.MakeCode>                        ；看函数名就知道是关键CALL了，F7跟进
00507152   .  8D45 AC        lea eax,dword ptr ss:[ebp-54]
00507155   .  8D55 BA        lea edx,dword ptr ss:[ebp-46]
00507158   .  B9 21000000    mov ecx,21
0050715D   .  E8 F2D3EFFF    call 快刀英语.00404554
00507162   .  8B45 AC        mov eax,dword ptr ss:[ebp-54]
00507165   .  50             push eax
00507166   .  8D55 A8        lea edx,dword ptr ss:[ebp-58]
00507169   .  8B83 00030000  mov eax,dword ptr ds:[ebx+300]
0050716F   .  E8 E8D2F5FF    call 快刀英语.0046445C
00507174   .  8B55 A8        mov edx,dword ptr ss:[ebp-58]                   ；假码
00507177   .  58             pop eax                                         ；真码
00507178   .  E8 73D5EFFF    call 快刀英语.004046F0                          ；在此可作内存注册机
0050717D   .  74 1D          je short 快刀英语.0050719C
0050717F   .  6A 10          push 10
00507181   .  B9 F4745000    mov ecx,快刀英语.005074F4
00507186   .  BA FC745000    mov edx,快刀英语.005074FC
0050718B   .  A1 18EE5000    mov eax,dword ptr ds:[50EE18]
00507190   .  8B00           mov eax,dword ptr ds:[eax]
00507192   .  E8 45D5F7FF    call 快刀英语.004846DC
00507197   .  E9 77020000    jmp 快刀英语.00507413
0050719C   >  A1 A8EC5000    mov eax,dword ptr ds:[50ECA8]
005071A1   .  8B00           mov eax,dword ptr ds:[eax]
005071A3   .  FFB0 60050000  push dword ptr ds:[eax+560]
005071A9   .  A1 A8EC5000    mov eax,dword ptr ds:[50ECA8]
```

### 二

```
00382371    8BEC             mov ebp,esp
00382373    6A 00            push 0
00382375    6A 00            push 0
00382377    53               push ebx
00382378    56               push esi
00382379    57               push edi
0038237A    33C0             xor eax,eax
0038237C    55               push ebp
0038237D    68 F2233800      push mcd.003823F2
00382382    64:FF30          push dword ptr fs:[eax]
00382385    64:8920          mov dword ptr fs:[eax],esp
00382388    33DB             xor ebx,ebx
0038238A    33C0             xor eax,eax
0038238C    55               push ebp
0038238D    68 C6233800      push mcd.003823C6
00382392    64:FF30          push dword ptr fs:[eax]
00382395    64:8920          mov dword ptr fs:[eax],esp
00382398    8D45 F8          lea eax,dword ptr ss:[ebp-8]
0038239B    8B55 08          mov edx,dword ptr ss:[ebp+8]
0038239E    E8 DD17FFFF      call mcd.00373B80
003823A3    8B45 F8          mov eax,dword ptr ss:[ebp-8]                ；EAX 01350B0C ASCII "jney2@21cn.comF4E8-C8D4"
003823A6    8D55 FC          lea edx,dword ptr ss:[ebp-4]
003823A9    E8 8AFEFFFF      call mcd.00382238                           ；关键CALL，F7跟进
003823AE    8B4D 10          mov ecx,dword ptr ss:[ebp+10]
003823B1    8B55 FC          mov edx,dword ptr ss:[ebp-4]
003823B4    8B45 0C          mov eax,dword ptr ss:[ebp+C]
003823B7    E8 F84BFFFF      call mcd.00376FB4
003823BC    33C0             xor eax,eax
003823BE    5A               pop edx
003823BF    59               pop ecx
003823C0    59               pop ecx
003823C1    64:8910          mov dword ptr fs:[eax],edx
003823C4    EB 11            jmp short mcd.003823D7
003823C6  ^ E9 310EFFFF      jmp mcd.003731FC
003823CB    E8 F0EDFEFF      call mcd.003711C0                              ; jmp to ntdll.RtlGetLastWin32Error
003823D0    8BD8             mov ebx,eax
003823D2    E8 0510FFFF      call mcd.003733DC
003823D7    33C0             xor eax,eax
003823D9    5A               pop edx
003823DA    59               pop ecx
003823DB    59               pop ecx
003823DC    64:8910          mov dword ptr fs:[eax],edx
003823DF    68 F9233800      push mcd.003823F9
003823E4    8D45 F8          lea eax,dword ptr ss:[ebp-8]
003823E7    BA 02000000      mov edx,2
003823EC    E8 BB15FFFF      call mcd.003739AC
003823F1    C3               retn
003823F2  ^ E9 310FFFFF      jmp mcd.00373328
003823F7  ^ EB EB            jmp short mcd.003823E4
003823F9    8BC3             mov eax,ebx
003823FB    5F               pop edi
003823FC    5E               pop esi
003823FD    5B               pop ebx
003823FE    59               pop ecx
003823FF    59               pop ecx
00382400    5D               pop ebp
00382401    C2 0C00          retn 0C
```

### 三

```
00382238    55               push ebp
00382239    8BEC             mov ebp,esp
0038223B    83C4 EC          add esp,-14
0038223E    53               push ebx
0038223F    56               push esi
00382240    57               push edi
00382241    33C9             xor ecx,ecx
00382243    894D EC          mov dword ptr ss:[ebp-14],ecx
00382246    894D F0          mov dword ptr ss:[ebp-10],ecx
00382249    8BF2             mov esi,edx
0038224B    8945 F8          mov dword ptr ss:[ebp-8],eax
0038224E    8B45 F8          mov eax,dword ptr ss:[ebp-8]
00382251    E8 BE1BFFFF      call mcd.00373E14
```
