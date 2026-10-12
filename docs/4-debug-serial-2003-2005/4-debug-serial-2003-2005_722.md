
# ezConverter 2.1 -- 注册算法破解分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-243.htm`](https://www.pediy.com/kssd/pediy07/pediy7-243.htm)

【破文标题】：ezConverter 2.1 -- 注册算法破解分析  
【破文作者】：KuNgBiM\[DFCG\]  
【作者邮箱】：gb\_1227@163.com  
【软件名称】：ezConverter 2.1  
【软件大小】：732KB  
【整理时间】：2005-5-21  
【软件类别】：国外软件/共享版/音频工具  
【下载地址】：[`www.goldlimit.com/product/ezceng.exe`](http://www.goldlimit.com/product/ezceng.exe)  
【软件简介】：支持的源格式：asf，wmv，wma，wav，mp3，mpeg，dat，avi，cd 音频，磁带，话筒等等.目标格式包括 wma，wav，mp3，asf 等等在内多达 10 余种，更自动支持标准第三方格式。转换速度极快。就 4 分钟的 mp3 为例，目前的一些转换工具一般需要 40-45 秒，而 ezConverter 只需 30 秒左右。  
【保护方式】：注册码 + 30 次试用限制 + 重启验证  
【编译语言】：Microsoft Visual C++ 6.0  
【调试环境】：WinXP、PEiD、Ollydbg  
【破解日期】：2005-05-21  
【破解目的】：研究算法分析  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

—————————————————————————————————

## 破解过程

### 侦测

用 `PEiD` 查壳，无壳，`Microsoft Visual C++ 6.0` 编译。

### 试探

运行主程序注册，输入 `Register name`、`Register code`，确认！程序提示"Please restart ezConverter now."（重启验证）。

输入试炼信息：

`Register name`：KuNgBiM  
`Register code`：78787878787878

### 对症下药

这个软件有 `ANTI-Loader` 功能，所以看来 `W32Dasm` 是没戏的了，我们用 `Ollydbg` 载入主程序，加载完毕后，在调试窗口内单击鼠标右键，选择“搜索”-->“当前模块中的名称（标签）”。找到 `MFC42.#1199_AfxMessageBox` 和 `MFC.#1200_AfxMessageBox`，在这两项上选择“在每个参考上设置断点”，这两个函数就是杀掉启动提示窗口的 `MFC42` 程序断点。 `F9` 运行程序：

很快就在 `00405791` 处中断： 

```
0040578E    57                push edi  
0040578F    57                push edi  
00405790    50                push eax  
00405791    E8 DC7D0000       call <jmp.&MFC42.#1200>                 //提示未注册信息  
00405796    8D4C24 08         lea ecx,dword ptr ss:[esp+8]  
0040579A    C78424 C40A0000   mov dword ptr ss:[esp+AC4],-1  
004057A5    E8 467B0000       call <jmp.&MFC42.#800>  
004057AA    8BCE              mov ecx,esi  
004057AC    E8 5F020000       call ezConver.00405A10  
004057B1    57                push edi  
004057B2    8D4C24 7C         lea ecx,dword ptr ss:[esp+7C]  
........
```

程序被 `OD` 中断后，向上翻看可疑跳转：

```
00405696    6A FF             push -1  
00405698    68 C1E14000       push ezConver.0040E1C1  
0040569D    50               push eax  
0040569E    64:8925 00000000 mov dword ptr fs:[0],esp  
004056A5    81EC B40A0000     sub esp,0AB4  
004056AB    56               push esi  
004056AC    57               push edi  
004056AD    33FF             xor edi,edi  
004056AF    8BF1             mov esi,ecx  
004056B1    57               push edi  
004056B2    E8 D37E0000       call <jmp.&MFC42.#1134>                  //可疑的 CALL  
004056B7    83C4 04          add esp,4  
004056BA    E8 C57E0000       call <jmp.&MFC42.#1205>  
004056BF    8BCE              mov ecx,esi  
004056C1    E8 B87E0000       call <jmp.&MFC42.#2621>  
004056C6    8D86 C4000000     lea eax,dword ptr ds:[esi+C4]  
004056CC    50               push eax  
004056CD    68 FF000000       push 0FF  
004056D2    FF15 68F04000     call dword ptr ds:[<&KERNEL32.GetCurrentDir>  
004056D8    8BCE              mov ecx,esi  
004056DA    E8 41070000       call ezConver.00405E20                   //在此设断，关键 CALL，F7 跟进去   
004056DF    84C0              test al,al  
004056E1    0F85 C3000000      jnz ezConver.004057AA                   //关键的跳转，不跳则死  
004056E7    57               push edi  
004056E8    8BCE              mov ecx,esi  
004056EA    E8 C10D0000       call ezConver.004064B0  
004056EF    B9 1E000000       mov ecx,1E  
004056F4    2BC8              sub ecx,eax  
004056F6    83F9 9C          cmp ecx,-64  
004056F9    898E C4010000    mov dword ptr ds:[esi+1C4],ecx  
004056FF    0F8C B9010000     jl ezConver.004058BE  
00405705    83F9 01          cmp ecx,1  
00405708    7D 53            jge short ezConver.0040575D  
0040570A    57               push edi  
0040570B    8D4C24 14         lea ecx,dword ptr ss:[esp+14]  
0040570F    E8 8C780000       call ezConver.0040CFA0  
00405714    8D4C24 10         lea ecx,dword ptr ss:[esp+10]  
00405718    89BC24 C40A0000   mov dword ptr ss:[esp+AC4],edi  
0040571F    E8 947A0000       call <jmp.&MFC42.#2514>  
00405724    8D4C24 74         lea ecx,dword ptr ss:[esp+74]  
00405728    C78424 C40A0000   mov dword ptr ss:[esp+AC4],2  
00405733    E8 B87B0000       call <jmp.&MFC42.#800>  
00405738    8D4C24 70         lea ecx,dword ptr ss:[esp+70]  
0040573C    C68424 C40A0000   mov byte ptr ss:[esp+AC4],1  
00405744    E8 A77B0000       call <jmp.&MFC42.#800>  
00405749    C78424 C40A0000   mov dword ptr ss:[esp+AC4],-1  
00405754    8D4C24 10         lea ecx,dword ptr ss:[esp+10]  
00405758    E9 5C010000       jmp ezConver.004058B9  
0040575D    8D4C24 08         lea ecx,dword ptr ss:[esp+8]  
00405761    E8 A27B0000       call <jmp.&MFC42.#540>  
00405766    8B8E C4010000     mov ecx,dword ptr ds:[esi+1C4]  
0040576C    8D5424 08         lea edx,dword ptr ss:[esp+8]  
00405770    51               push ecx  
00405771    68 92000000       push 92  
00405776    52               push edx  
00405777    C78424 D00A0000   mov dword ptr ss:[esp+AD0],3  
00405782    E8 F17D0000       call <jmp.&MFC42.#2817>  
00405787    8B4424 14         mov eax,dword ptr ss:[esp+14]  
0040578B    83C4 0C          add esp,0C  
0040578E    57               push edi  
0040578F    57               push edi  
00405790    50               push eax  
00405791    E8 DC7D0000       call <jmp.&MFC42.#1200>                  //提示未注册信息  
00405796    8D4C24 08         lea ecx,dword ptr ss:[esp+8]  
0040579A    C78424 C40A0000   mov dword ptr ss:[esp+AC4],-1  
004057A5    E8 467B0000       call <jmp.&MFC42.#800>  
004057AA    8BCE              mov ecx,esi  
004057AC    E8 5F020000       call ezConver.00405A10  
004057B1    57               push edi  
004057B2    8D4C24 7C         lea ecx,dword ptr ss:[esp+7C]  
004057B6    E8 E5110000       call ezConver.004069A0  
004057BB    8D4C24 78         lea ecx,dword ptr ss:[esp+78]  
004057BF    C78424 C40A0000   mov dword ptr ss:[esp+AC4],4  
004057CA    894E 20           mov dword ptr ds:[esi+20],ecx  
004057CD    8D4C24 78         lea ecx,dword ptr ss:[esp+78]  
004057D1    E8 E2790000       call <jmp.&MFC42.#2514>  
004057D6    57               push edi  
004057D7    FF15 64F04000     call dword ptr ds:[<&KERNEL32.GetCurrentPro>  
004057DD    50               push eax  
004057DE    FF15 44F04000     call dword ptr ds:[<&KERNEL32.TerminateProc>  
004057E4    8D8C24 780A0000   lea ecx,dword ptr ss:[esp+A78]  
004057EB    C78424 C40A0000   mov dword ptr ss:[esp+AC4],9  
004057F6    E8 677B0000       call <jmp.&MFC42.#693>  
004057FB    8D9424 5C0A0000   lea edx,dword ptr ss:[esp+A5C]  
00405802    C78424 5C0A0000   mov dword ptr ss:[esp+A5C],ezConver.0040FDE>  
0040580D    895424 0C         mov dword ptr ss:[esp+C],edx  
00405811    8B8424 600A0000   mov eax,dword ptr ss:[esp+A60]  
00405818    C68424 C40A0000   mov byte ptr ss:[esp+AC4],0A  
00405820    3BC7              cmp eax,edi  
00405822    74 06            je short ezConver.0040582A  
00405824    8B00              mov eax,dword ptr ds:[eax]  
00405826    3BC7              cmp eax,edi  
00405828    ^ 75 FA          jnz short ezConver.00405824  
0040582A    8B8C24 700A0000   mov ecx,dword ptr ss:[esp+A70]  
00405831    89BC24 680A0000   mov dword ptr ss:[esp+A68],edi  
00405838    89BC24 6C0A0000   mov dword ptr ss:[esp+A6C],edi  
0040583F    89BC24 640A0000   mov dword ptr ss:[esp+A64],edi  
00405846    89BC24 600A0000   mov dword ptr ss:[esp+A60],edi  
0040584D    E8 1A7D0000       call <jmp.&MFC42.#2841>  
00405852    8D8C24 80070000   lea ecx,dword ptr ss:[esp+780]  
00405859    89BC24 700A0000   mov dword ptr ss:[esp+A70],edi  
00405860    C78424 5C0A0000   mov dword ptr ss:[esp+A5C],ezConver.0040F78>  
0040586B    C68424 C40A0000   mov byte ptr ss:[esp+AC4],7  
00405873    E8 08D4FFFF       call ezConver.00402C80  
00405878    68 E0AF4000       push ezConver.CFlyBtn::~CFlyBtn  
0040587D    6A 0F            push 0F  
0040587F    8D8424 EC000000  lea eax,dword ptr ss:[esp+EC]
```



```
00405886    6A 70             push 70

00405888    50               push eax

00405889    C68424 D40A0000 >mov byte ptr ss:[esp+AD4],6

00405891    E8 687E0000      call ezConver.0040D6FE

00405896    8D8C24 DC000000  lea ecx,dword ptr ss:[esp+DC]

0040589D    C68424 C40A0000 >mov byte ptr ss:[esp+AC4],5

004058A5    E8 BC7C0000      call <jmp.&MFC42.#686>

004058AA    C78424 C40A0000 >mov dword ptr ss:[esp+AC4],-1

004058B5    8D4C24 78        lea ecx,dword ptr ss:[esp+78]

004058B9    E8 3E7A0000      call <jmp.&MFC42.#641>

004058BE    8B8C24 BC0A0000  mov ecx,dword ptr ss:[esp+ABC]

004058C5    5F               pop edi

004058C6    33C0             xor eax,eax

004058C8    5E               pop esi

004058C9    64:890D 00000000 mov dword ptr fs:[0],ecx

004058D0    81C4 C00A0000    add esp,0AC0

004058D6    C3               retn

\================  跟进：004056DA    E8 41070000      call ezConver.00405E20  ================

00405E20    55               push ebp

00405E21    8BEC             mov ebp,esp

00405E23    83E4 F8          and esp,FFFFFFF8

00405E26    6A FF            push -1

00405E28    68 EEE24000      push ezConver.0040E2EE

00405E2D    64:A1 00000000   mov eax,dword ptr fs:[0]

00405E33    50               push eax

00405E34    64:8925 00000000 mov dword ptr fs:[0],esp

00405E3B    81EC 18010000    sub esp,118

00405E41    53               push ebx

00405E42    55               push ebp

00405E43    56               push esi

00405E44    8B35 34F04000    mov esi,dword ptr ds:[<&KERNEL32.GetProcAdd>; kernel32.GetProcAddress

00405E4A    8BC6             mov eax,esi

00405E4C    57               push edi

00405E4D    2D 00000070      sub eax,70000000

00405E52    894C24 14        mov dword ptr ss:[esp+14],ecx

00405E56    0F88 77030000    js ezConver.004061D3

00405E5C    68 08344100      push ezConver.00413408                  ; ASCII "kernel32.dll"

00405E61    FF15 30F04000    call dword ptr ds:[<&KERNEL32.LoadLibraryA>>; kernel32.LoadLibraryA

00405E67    8BE8             mov ebp,eax

00405E69    68 FC334100      push ezConver.004133FC                  //读取文件  ASCII "ReadFile"

00405E6E    55               push ebp

00405E6F    FFD6             call esi

00405E71    68 F0334100      push ezConver.004133F0                  //创建文件  "CreateFileA"

00405E76    55               push ebp

00405E77    8BD8             mov ebx,eax

00405E79    FFD6             call esi

00405E7B    68 E4334100      push ezConver.004133E4                  //写入文件  ASCII "WriteFile"

00405E80    55               push ebp

00405E81    8BF8             mov edi,eax

00405E83    FFD6             call esi

00405E85    8D4C24 10        lea ecx,dword ptr ss:[esp+10]

00405E89    E8 7A740000      call <jmp.&MFC42.#540>

00405E8E    8B4C24 14        mov ecx,dword ptr ss:[esp+14]

00405E92    8D5424 10        lea edx,dword ptr ss:[esp+10]

00405E96    81C1 C4000000    add ecx,0C4

00405E9C    C78424 30010000 >mov dword ptr ss:[esp+130],0

00405EA7    51               push ecx

00405EA8    68 D8334100      push ezConver.004133D8                  //读取注册文件中的注册信息   ASCII "%s\\erf.dat"

00405EAD    52               push edx

00405EAE    E8 7F740000      call <jmp.&MFC42.#2818>

00405EB3    8B4424 1C        mov eax,dword ptr ss:[esp+1C]

00405EB7    83C4 0C          add esp,0C

00405EBA    6A 00            push 0

00405EBC    6A 00            push 0

00405EBE    6A 03            push 3

00405EC0    6A 00            push 0

00405EC2    6A 00            push 0

00405EC4    68 00000080      push 80000000

00405EC9    50               push eax                                //ASCII "C:\\Program Files\\GoldLimit\\ezConverter\\erf.dat"

00405ECA    FFD7             call edi

00405ECC    8BF0             mov esi,eax

00405ECE    83FE FF          cmp esi,-1

00405ED1    75 0C            jnz short ezConver.00405EDF

00405ED3    898424 30010000  mov dword ptr ss:[esp+130],eax

00405EDA    E9 EB020000      jmp ezConver.004061CA

00405EDF    B9 08000000      mov ecx,8

00405EE4    33C0             xor eax,eax

00405EE6    8D7C24 64        lea edi,dword ptr ss:[esp+64]

00405EEA    6A 00            push 0

00405EEC    F3:AB            rep stos dword ptr es:[edi]

00405EEE    66:AB            stos word ptr es:[edi]

00405EF0    AA               stos byte ptr es:[edi]

00405EF1    B9 08000000      mov ecx,8

00405EF6    33C0             xor eax,eax

00405EF8    8D7C24 44        lea edi,dword ptr ss:[esp+44]

00405EFC    6A 00            push 0

00405EFE    F3:AB            rep stos dword ptr es:[edi]

00405F00    66:AB            stos word ptr es:[edi]

00405F02    68 20030000      push 320

00405F07    56               push esi

00405F08    AA               stos byte ptr es:[edi]

00405F09    FF15 3CF04000    call dword ptr ds:[<&KERNEL32.SetFilePointe>; kernel32.SetFilePointer

00405F0F    8D4C24 24        lea ecx,dword ptr ss:[esp+24]

00405F13    6A 00            push 0

00405F15    51               push ecx

00405F16    8D5424 6C        lea edx,dword ptr ss:[esp+6C]

00405F1A    6A 1E            push 1E

00405F1C    52               push edx

00405F1D    56               push esi

00405F1E    FFD3             call ebx

00405F20    8D4424 24        lea eax,dword ptr ss:[esp+24]

00405F24    6A 00            push 0

00405F26    50               push eax

00405F27    8D4C24 48        lea ecx,dword ptr ss:[esp+48]

00405F2B    6A 1E            push 1E

00405F2D    51               push ecx

00405F2E    56               push esi

00405F2F    FFD3             call ebx

00405F31    8D5424 14        lea edx,dword ptr ss:[esp+14]

00405F35    8D4424 1C        lea eax,dword ptr ss:[esp+1C]

00405F39    52               push edx

00405F3A    8D4C24 2C        lea ecx,dword ptr ss:[esp+2C]

00405F3E    50               push eax

00405F3F    51               push ecx

00405F40    56               push esi

00405F41    FF15 88F04000    call dword ptr ds:[<&KERNEL32.GetFileTime>]   //读去文件创建时间

00405F47    8D5424 1C        lea edx,dword ptr ss:[esp+1C]

00405F4B    8D4424 30        lea eax,dword ptr ss:[esp+30]

00405F4F    52               push edx

00405F50    50               push eax

00405F51    66:C74424 38 CF0>mov word ptr ss:[esp+38],7CF

00405F58    66:C74424 3A 050>mov word ptr ss:[esp+3A],5

00405F5F    66:C74424 3E 160>mov word ptr ss:[esp+3E],16

00405F66    FF15 8CF04000    call dword ptr ds:[<&KERNEL32.SystemTimeToF>  //检查注册文件和系统时间是否更改

00405F6C    8D4C24 14        lea ecx,dword ptr ss:[esp+14]

00405F70    8D5424 1C        lea edx,dword ptr ss:[esp+1C]

00405F74    51               push ecx

00405F75    8D4424 2C        lea eax,dword ptr ss:[esp+2C]

00405F79    52               push edx

00405F7A    50               push eax

00405F7B    56               push esi

00405F7C    FF15 80F04000    call dword ptr ds:[<&KERNEL32.SetFileTime>] ; kernel32.SetFileTime

00405F82    56               push esi

00405F83    FF15 78F04000    call dword ptr ds:[<&KERNEL32.CloseHandle>]   //如果检查失败则退出

00405F89    55               push ebp

00405F8A    FF15 38F04000    call dword ptr ds:[<&KERNEL32.FreeLibrary>] ; kernel32.FreeLibrary

00405F90    83CB FF          or ebx,FFFFFFFF

00405F93    8D7C24 64        lea edi,dword ptr ss:[esp+64]                 //读取注册文件中的用户名 ASCII "KuNgBiM"

00405F97    8BCB             mov ecx,ebx

00405F99    33C0             xor eax,eax

00405F9B    F2:AE            repne scas byte ptr es:[edi]

00405F9D    F7D1             not ecx                                       //取用户名的长度   ecx=7

00405F9F    49               dec ecx
```



```asm
00405FA0    83F9 01           cmp ecx,1                                  //取用户名的长度与 1 相比  
00405FA3    0F82 1A020000     jb ezConver.004061C3                       //小于 1 则跳死  
00405FA9    8D7C24 40         lea edi,dword ptr ss:[esp+40]             //读取注册文件中的注册码 ASCII "78787878787878"  
00405FAD    8BCB              mov ecx,ebx  
00405FAF    F2:AE             repne scas byte ptr es:[edi]  
00405FB1    F7D1              not ecx  
00405FB3    49                dec ecx  
00405FB4    83F9 01           cmp ecx,1                                  //取注册码的长度与 1 相比  
00405FB7    0F82 06020000     jb ezConver.004061C3                       //小于 1 则跳死  
00405FBD    BF CC334100       mov edi,ezConver.004133CC                //取 ASCII "ezConverter"  
00405FC2    8BCB              mov ecx,ebx                                //注册码位数为 14 位 ecx=0E  
00405FC4    F2:AE             repne scas byte ptr es:[edi]  
00405FC6    F7D1              not ecx  
00405FC8    49                dec ecx  
00405FC9    8D7C24 64         lea edi,dword ptr ss:[esp+64]             //ASCII "KuNgBiM"  ASCII "%s\erf.dat"  
00405FCD    8BE9              mov ebp,ecx  
00405FCF    8BCB              mov ecx,ebx  
00405FD1    F2:AE             repne scas byte ptr es:[edi]  
00405FD3    F7D1              not ecx  
00405FD5    49                dec ecx  
00405FD6    C68424 30010000   >mov byte ptr ss:[esp+130],1  
00405FDE    03E9              add ebp,ecx  
00405FE0    8D4C24 14         lea ecx,dword ptr ss:[esp+14]  
00405FE4    8BF3              mov esi,ebx  
00405FE6    E8 1D730000       call <jmp.&MFC42.#540>  
00405FEB    8D4C24 64         lea ecx,dword ptr ss:[esp+64]             //再次读取用户名  ASCII "KuNgBiM"  
00405FEF    C68424 30010000   >mov byte ptr ss:[esp+130],2  
00405FF7    51                push ecx                                   //注册名入 ECX  
00405FF8    8D4C24 18         lea ecx,dword ptr ss:[esp+18]  
00405FFC    E8 01730000       call <jmp.&MFC42.#860>  
00406001    68 CC334100       push ezConver.004133CC                     //取 ASCII "ezConverter"  
00406006    8D4C24 18         lea ecx,dword ptr ss:[esp+18]  
0040600A    E8 7D730000       call <jmp.&MFC42.#941>  
0040600F    8D4C24 14         lea ecx,dword ptr ss:[esp+14]  
00406013    E8 9C750000       call <jmp.&MFC42.#4204>                   //把的用户名与"ezConverter"连接并转换成大写  
00406018    6A 00             push 0  
0040601A    8D4C24 18         lea ecx,dword ptr ss:[esp+18]             //ASCII "KUNGBIMEZCONVERTER"  
0040601E    E8 75730000       call <jmp.&MFC42.#2915>  
00406023    33C9              xor ecx,ecx                                //清零  
00406025    8BF8              mov edi,eax                                  //ASCII "KUNGBIMEZCONVERTER"  
00406027    85ED              test ebp,ebp                                 //ebp=12  
00406029    7E 68             jle short ezConver.00406093  
0040602B    8BC1              mov eax,ecx                                  //开始循环取字符串的 HEX 值  
0040602D    BB 03000000       mov ebx,3  
00406032    99                cdq  
00406033    F7FB              idiv ebx  
00406035    46                inc esi  
00406036    85D2              test edx,edx  
00406038    75 16             jnz short ezConver.00406050  
0040603A    8A0439            mov al,byte ptr ds:[ecx+edi]               //取首字符的 HEX 值  
0040603D    0FBED0            movsx edx,al  
00406040    83EA 05           sub edx,5  
00406043    83FA 41           cmp edx,41  
00406046    7E 04             jle short ezConver.0040604C  
00406048    2C 05             sub al,5  
0040604A    EB 38             jmp short ezConver.00406084  
0040604C    04 05             add al,5  
0040604E    EB 34             jmp short ezConver.00406084  
00406050    83FA 01           cmp edx,1  
00406053    75 16             jnz short ezConver.0040606B  
00406055    8A0439            mov al,byte ptr ds:[ecx+edi]  
00406058    0FBED0            movsx edx,al  
0040605B    83C2 07           add edx,7  
0040605E    83FA 5A           cmp edx,5A  
00406061    7D 04             jge short ezConver.00406067  
00406063    04 07             add al,7  
00406065    EB 1D             jmp short ezConver.00406084  
00406067    2C 07             sub al,7  
00406069    EB 19             jmp short ezConver.00406084  
0040606B    83FA 02           cmp edx,2  
0040606E    75 1B             jnz short ezConver.0040608B  
00406070    8A0439            mov al,byte ptr ds:[ecx+edi]  
00406073    0FBED0            movsx edx,al  
00406076    83EA 09           sub edx,9  
00406079    83FA 41           cmp edx,41  
0040607C    7E 04             jle short ezConver.00406082  
0040607E    2C 09             sub al,9  
00406080    EB 02             jmp short ezConver.00406084  
00406082    04 09             add al,9  
00406084    888434 88000000   mov byte ptr ss:[esp+esi+88],al  
0040608B    41                inc ecx  
0040608C    3BCD              cmp ecx,ebp  
0040608E    ^ 7C 9B           jl short ezConver.0040602B  
00406090    83CB FF           or ebx,FFFFFFFF  
00406093    DD05 00FE4000     fld qword ptr ds:[40FE00]  
00406099    33C0              xor eax,eax  
0040609B    85F6              test esi,esi  
0040609D    7E 17             jle short ezConver.004060B6  
0040609F    0FBE8C04 8800000>movsx ecx,byte ptr ss:[esp+eax+88]  
004060A7    894C24 1C         mov dword ptr ss:[esp+1C],ecx  
004060AB    40                inc eax  
004060AC    DB4424 1C         fild dword ptr ss:[esp+1C]  
004060B0    3BC6              cmp eax,esi  
004060B2    DEC1              faddp st(1),st  
004060B4    ^ 7C E9           jl short ezConver.0040609F  
004060B6    D9C0              fld st  
004060B8    D9FE              fsin                                      //浮点运算  
004060BA    D9FF              fcos                                      //浮点运算  
004060BC    D9FE              fsin                                      //浮点运算               
004060BE    D9FF              fcos                                      //浮点运算  
004060C0    D9FE              fsin                                      //浮点运算  
004060C2    DD5424 1C         fst                                       //浮点运算  
004060C6    DC1D 00FE4000     fcomp qword ptr ds:[40FE00]  
004060CC    DFE0              fstsw ax  
004060CE    F6C4 01           test ah,1  
004060D1    74 23             je short ezConver.004060F6  
004060D3    DC0D F8FD4000     fmul qword ptr ds:[40FDF8]  
004060D9    D9C0              fld st  
004060DB    D9FE              fsin                                       //浮点运算  
004060DD    D9FF              fcos                                       //浮点运算  
004060DF    D9FE              fsin                                       //浮点运算  
004060E1    D9FF              fcos                                       //浮点运算  
004060E3    D9FE              fsin                                       //浮点运算  
004060E5    DD5424 1C         fst qword ptr ss:[esp+1C]  
004060E9    DC1D 00FE4000     fcomp qword ptr ds:[40FE00]  
004060EF    DFE0              fstsw ax  
004060F1    F6C4 01           test ah,1  
004060F4    ^ 75 DD           jnz short ezConver.004060D3  
004060F6    8B5424 20         mov edx,dword ptr ss:[esp+20]  
004060FA    8B4424 1C         mov eax,dword ptr ss:[esp+1C]  
004060FE    52                push edx  
004060FF    50                push eax  
00406100    8D8C24 E0000000   lea ecx,dword ptr ss:[esp+E0]  
00406107    68 C4334100       push ezConver.004133C4                     //ASCII "%.14f"  
0040610C    51                push ecx  
0040610D    DDD8              fstp st                                    //st=1279.0000000000000000  
0040610F    FF15 38F44000     call dword ptr ds:[<&MSVCRT.sprintf>]  
00406115    8DBC24 E8000000   lea edi,dword ptr ss:[esp+E8]           //ASCII "0.63924307380512"  //ASCII "KUNGBIMEZCONVERTER"  
0040611C    8BCB              mov ecx,ebx  
```



### 汇编代码片段

```asm
0040611E    33C0             xor eax,eax                                 //eax=10  

00406120    83C4 10          add esp,10  

00406123    33D2             xor edx,edx  

00406125    F2:AE            repne scas byte ptr es:[edi]  

00406127    F7D1             not ecx  

00406129    49               dec ecx  

0040612A    83E9 02          sub ecx,2  

0040612D    74 27            je short ezConver.00406156  

0040612F    8A8414 DA000000  mov al,byte ptr ss:[esp+edx+DA]             //开始循环取"0.63924307380512"的 HEX 值  

00406136    8DBC24 D8000000  lea edi,dword ptr ss:[esp+D8]  

0040613D    04 41            add al,41  

0040613F    8BCB             mov ecx,ebx  

00406141    888414 88000000  mov byte ptr ss:[esp+edx+88],al  

00406148    33C0             xor eax,eax  

0040614A    42               inc edx  

0040614B    F2:AE            repne scas byte ptr es:[edi]  

0040614D    F7D1             not ecx  

0040614F    83C1 FD          add ecx,-3  

00406152    3BD1             cmp edx,ecx  

00406154    72 D9            jb short ezConver.0040612F  

00406156    8D8C24 88000000  lea ecx,dword ptr ss:[esp+88]  

0040615D    C68434 88000000 mov byte ptr ss:[esp+esi+88],0  

00406165    51               push ecx  

00406166    FF15 ACF44000    call dword ptr ds:[<&USER32.CharUpperA>]    ; USER32.CharUpperA  

0040616C    8D4C24 14        lea ecx,dword ptr ss:[esp+14]  

00406170    C68424 30010000 mov byte ptr ss:[esp+130],1  

00406178    E8 73710000      call <jmp.&MFC42.#800>  

0040617D    8D7C24 40        lea edi,dword ptr ss:[esp+40]  

00406181    8BCB             mov ecx,ebx  

00406183    33C0             xor eax,eax                                 //清零  

00406185    8DB424 88000000  lea esi,dword ptr ss:[esp+88]              //注册码   ASCII "WTZSUTQXTYQVRSIOL"  

0040618C    F2:AE            repne scas byte ptr es:[edi]  

0040618E    F7D1             not ecx  

00406190    49               dec ecx  

00406191    8D7C24 40        lea edi,dword ptr ss:[esp+40]  

00406195    33D2             xor edx,edx  

00406197    899C24 30010000  mov dword ptr ss:[esp+130],ebx  

0040619E    F3:A6            repe cmps byte ptr es:[edi],byte ptr ds:[esi]  

004061A0    8D4C24 10        lea ecx,dword ptr ss:[esp+10]  

004061A4    75 28            jnz short ezConver.004061CE                 //爆破点，nop 掉，即为无功能限制注册版！  

004061A6    E8 45710000      call <jmp.&MFC42.#800>  

004061AB    B0 01            mov al,1  

004061AD    8B8C24 28010000  mov ecx,dword ptr ss:[esp+128]  

004061B4    64:890D 00000000 mov dword ptr fs:[0],ecx  

004061BB    5F               pop edi  

004061BC    5E               pop esi  

004061BD    5D               pop ebp  

004061BE    5B               pop ebx  

004061BF    8BE5             mov esp,ebp  

004061C1    5D               pop ebp  

004061C2    C3               retn  

004061C3    899C24 30010000  mov dword ptr ss:[esp+130],ebx  

004061CA    8D4C24 10        lea ecx,dword ptr ss:[esp+10]  

004061CE    E8 1D710000      call <jmp.&MFC42.#800>  

004061D3    8B8C24 28010000  mov ecx,dword ptr ss:[esp+128]  

004061DA    5F               pop edi  

004061DB    32C0             xor al,al  

004061DD    64:890D 00000000 mov dword ptr fs:[0],ecx  

004061E4    5E               pop esi                                     //ASCII "TZSUTQXTYQVRSIOL"  

004061E5    5D               pop ebp  

004061E6    5B               pop ebx  

004061E7    8BE5             mov esp,ebp                                 //ASCII "Pe8"  

004061E9    5D               pop ebp  

004061EA    C3               retn                                        //返回
```

## 算法总结

1.  用户名 + 固定字符 `ezConverter` 组合成一个新字符串并全部转换成大写，得到字符串 `A`。
2.  分别取字符串 `A` 中字符的 HEX 值转换成为 `B`。
3.  `B` 值再通过一系列的浮点运算，最终生成一个新的字符串作为注册码。

## 完美爆破点

```asm
004061A4    75 28            jnz short ezConver.004061CE                //nop 掉，即为无功能限制注册版！
```

### 注册信息

`Register name`: `KuNgBiM`

`Register code`: `WTZSUTQXTYQVRSIOL`

***

（本文完）

版权所有（C）2005 `KuNgBiM[DFCG]` &nbsp;&nbsp; Copyright (C) 2005 `KuNgBiM[DFCG]`

***

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cracked BY `KuNgBiM[DFCG]`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2005-05-21
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;07:37:57 AM

`4dd466d0e1e0cc123afbe504da2462ca`
