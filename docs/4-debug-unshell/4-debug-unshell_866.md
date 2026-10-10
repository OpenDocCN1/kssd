

# 学习笔记：单步法手脱 PECompact 2.5 Retail - Jeremy Collake

> 原文：[`www.pediy.com/kssd/pediy10/94624.html`](https://www.pediy.com/kssd/pediy10/94624.html)

单步法手脱 PECompact 2.5 Retail - Jeremy Collake  
　　这是我的学习笔记，毕竟刚学破解，牛人们看了不要笑话我这入门级的水平，给我鼓励就好了。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  
　　在学习黑鹰三人行的视频教程“6、手脱 PECompact”中，当讲到脱 PECompact 2.5 这个壳时，老师用的方法是下断点 BP VirtualFree 和查找 push 8000（特征码）的方法，至于为什么要这样，老师说“现在也只能告诉你，脱壳时通过下这个断点，可以跳过壳的加密，现在也只能告诉大家这些。每一个壳都有一种套路。这个是经验。”  
  
　　作为新手的我，听了这话，只得不知其所以然地强记老师的方法，相信很多新手此时都是无奈的，知识断档了。可是看过之后，觉得不甘心，老师的经验也是通过跟踪得来的呀，作为新手，不是正需要从基础锻炼起吗？于是就单步法反复跟踪，终于成功了。高兴之余，就想百度查一下别人是不是和我的方法相同，一查才知道脱这个壳的教程不多，竟然全部是用下断点 BP VirtualFree 和查找 push 8000（特征码）的方法！所以，我将自己的学习笔记贴上来，也可给新手们一个参考。好了，借用老师的一句话：废话不多说，回到正题！  
  
　　带壳软件：加了 PECompact 2.5 Retail -> Jeremy Collake 壳的 Win98 下的记事本 NotePad.exe（这里命名为 PECompact 2.55.EXE）  
　　脱壳软件：OllyICE、PEiD、LordPE、ImportREC\_fix  
  
**一、查壳**  
　　PEiD 查出壳为 PECompact 2.5 Retail -> Jeremy Collake  
  
**二、找 OEP**  
　　设置 Ollydbg 忽略所有的异常选项，载入 PECompact 2.55.EXE 文件，提示“你仍要继续分析吗？”选“否”。  
  
01001000 >  B8 90BA0101         mov     eax, PECompac.0101BA90        ; 载入时停在这里,F8 单走  
01001005    50                  push    eax  
01001006    64:FF35 00000000    push    dword ptr fs:\[0\]  
0100100D    64:8925 00000000    mov     dword ptr fs:\[0\], esp  
01001014    33C0                xor     eax, eax  
01001016    8908                mov     dword ptr ds:\[eax\], ecx       ; 这行 F8 后自动跳转  
01001018    50                  push    eax                       
01001019    45                  inc     ebp  
0100101A    43                  inc     ebx  
  
7C92E480  |.  8B1C24            mov     ebx, dword ptr ss:\[esp\]       ; 自动跳转到这里。此处是系统空间，不可 Alt+F9 回到用户代码，否则后面会无法继续跟踪，加密惹的祸^\_^，继续 F8  
7C92E483  |.  51                push    ecx  
7C92E484  |.  53                push    ebx  
7C92E485  |.  E8 B35A0200       call    ntdll.7C953F3D                ; 可 F8 步过  
7C92E48A  |.  0AC0              or      al, al  
7C92E48C  |.  74 0C             je      short ntdll.7C92E49A  
7C92E48E  |.  5B                pop     ebx  
7C92E48F  |.  59                pop     ecx  
7C92E490  |.  6A 00             push    0  
7C92E492  |.  51                push    ecx  
7C92E493  |.  E8 C6EBFFFF       call    ntdll.ZwContinue              ; 必须 F7 跟进！否则一直停在“运行”状态无法继续  
7C92E498  |.  EB 0B             jmp     short ntdll.7C92E4A5  
7C92E49A  |>  5B                pop     ebx  
7C92E49B  |.  59                pop     ecx  
  
7C92D05E >/$  B8 20000000       mov     eax, 20                       ; 跟进到了这里，继续 F8  
7C92D063  |.  BA 0003FE7F       mov     edx, 7FFE0300  
7C92D068  |.  FF12              call    dword ptr ds:\[edx\]            ; 必须 F7 跟进！否则一直停在“运行”状态无法继续  
7C92D06A  \\.  C2 0800           retn    8  
7C92D06D      90                nop  
  
7C92E510 >/$  8BD4              mov     edx, esp                      ; 跟进到了这里  
7C92E512  |.  0F34              sysenter                              ; 这行 F8 后自动跳转  
7C92E514 >\\$  C3                retn  
7C92E515   .  8DA424 00000000   lea     esp, dword ptr ss:\[esp\]  
7C92E51C   .  8D6424 00         lea     esp, dword ptr ss:\[esp\]  
  
0101BAB3    B8 97A801F1         mov     eax, F101A897                 ; 跳到了这里，继续 F8  
0101BAB8    64:8F05 00000000    pop     dword ptr fs:\[0\]  
0101BABF    83C4 04             add     esp, 4  
0101BAC2    55                  push    ebp  
0101BAC3    53                  push    ebx  
0101BAC4    51                  push    ecx  
0101BAC5    57                  push    edi  
0101BAC6    56                  push    esi  
0101BAC7    52                  push    edx  
0101BAC8    8D98 D5110010       lea     ebx, dword ptr ds:\[eax+100011>  
0101BACE    8B53 18             mov     edx, dword ptr ds:\[ebx+18\]  
0101BAD1    52                  push    edx  
0101BAD2    8BE8                mov     ebp, eax  
0101BAD4    6A 40               push    40  
0101BAD6    68 00100000         push    1000  
0101BADB    FF73 04             push    dword ptr ds:\[ebx+4\]  
0101BADE    6A 00               push    0  
0101BAE0    8B4B 10             mov     ecx, dword ptr ds:\[ebx+10\]  
0101BAE3    03CA                add     ecx, edx  
0101BAE5    8B01                mov     eax, dword ptr ds:\[ecx\]  
0101BAE7    FFD0                call    eax                           ; F8 过  
0101BAE9    5A                  pop     edx  
0101BAEA    8BF8                mov     edi, eax  
0101BAEC    50                  push    eax  
0101BAED    52                  push    edx  
0101BAEE    8B33                mov     esi, dword ptr ds:\[ebx\]  
0101BAF0    8B43 20             mov     eax, dword ptr ds:\[ebx+20\]  
0101BAF3    03C2                add     eax, edx  
0101BAF5    8B08                mov     ecx, dword ptr ds:\[eax\]  
0101BAF7    894B 20             mov     dword ptr ds:\[ebx+20\], ecx  
0101BAFA    8B43 1C             mov     eax, dword ptr ds:\[ebx+1C\]  
0101BAFD    03C2                add     eax, edx  
0101BAFF    8B08                mov     ecx, dword ptr ds:\[eax\]  
0101BB01    894B 1C             mov     dword ptr ds:\[ebx+1C\], ecx  
0101BB04    03F2                add     esi, edx  
0101BB06    8B4B 0C             mov     ecx, dword ptr ds:\[ebx+C\]  
0101BB09    03CA                add     ecx, edx  
0101BB0B    8D43 1C             lea     eax, dword ptr ds:\[ebx+1C\]  
0101BB0E    50                  push    eax  
0101BB0F    57                  push    edi  
0101BB10    56                  push    esi  
0101BB11    FFD1                call    ecx                           ; F8 过  
0101BB13    5A                  pop     edx  
0101BB14    58                  pop     eax  
0101BB15    0343 08             add     eax, dword ptr ds:\[ebx+8\]  
0101BB18    8BF8                mov     edi, eax  
0101BB1A    52                  push    edx  
0101BB1B    8BF0                mov     esi, eax  
0101BB1D    8B46 FC             mov     eax, dword ptr ds:\[esi-4\]  
0101BB20    83C0 04             add     eax, 4  
0101BB23    2BF0                sub     esi, eax  
0101BB25    8956 08             mov     dword ptr ds:\[esi+8\], edx  
0101BB28    8B4B 10             mov     ecx, dword ptr ds:\[ebx+10\]  
0101BB2B    894E 24             mov     dword ptr ds:\[esi+24\], ecx  
0101BB2E    8B4B 14             mov     ecx, dword ptr ds:\[ebx+14\]  
0101BB31    51                  push    ecx  
0101BB32    894E 28             mov     dword ptr ds:\[esi+28\], ecx  
0101BB35    8B4B 0C             mov     ecx, dword ptr ds:\[ebx+C\]  
0101BB38    894E 14             mov     dword ptr ds:\[esi+14\], ecx  
0101BB3B    FFD7                call    edi                           ; F8 过,寄存器窗口有较大的变化  
0101BB3D    8985 C8120010       mov     dword ptr ss:\[ebp+100012C8\], >  
0101BB43    8BF0                mov     esi, eax  
0101BB45    59                  pop     ecx  
0101BB46    5A                  pop     edx  
0101BB47    EB 0C               jmp     short PECompac.0101BB55  
0101BB49    03CA                add     ecx, edx  
0101BB4B    68 00800000         push    8000                          ; 这里就是老师说的特征码！OEP 就在附近!  
0101BB50    6A 00               push    0  
0101BB52    57                  push    edi  
0101BB53    FF11                call    dword ptr ds:\[ecx\]  
0101BB55    8BC6                mov     eax, esi  
0101BB57    5A                  pop     edx  
0101BB58    5E                  pop     esi  
0101BB59    5F                  pop     edi  
0101BB5A    59                  pop     ecx  
0101BB5B    5B                  pop     ebx  
0101BB5C    5D                  pop     ebp  
0101BB5D  - FFE0                jmp     eax                           ; 跳往 OEP!  
0101BB5F    9D                  popfd  
0101BB60    73 00               jnb     short PECompac.0101BB62  
0101BB62    0100                add     dword ptr ds:\[eax\], eax  
0101BB64    0000                add     byte ptr ds:\[eax\], al  
0101BB66    0000                add     byte ptr ds:\[eax\], al  
  
0100739D    6A 70               push    70                            ; 这里就是 OEP!  
0100739F    68 98180001         push    PECompac.01001898  
010073A4    E8 BF010000         call    PECompac.01007568  
010073A9    33DB                xor     ebx, ebx  
010073AB    53                  push    ebx  
010073AC    8B3D CC100001       mov     edi, dword ptr ds:\[10010CC\]   ; kernel32.GetModuleHandleA  
010073B2    FFD7                call    edi  
  
**三、脱壳**  
　　分别用 OD 自带的脱壳功能（右键－Dump Degugged Process）及 LordPE 脱壳。其中 OD 自带的脱壳功能分别用方法 1 和方法 2 进行脱壳，保存文件名分别为 1.exe 和 2.exe；运行 LordPE，右键 PECompact 2.55.EXE 进程点“修正镜像大小”，右键 PECompact 2.55.EXE 进程点“完整转存”，保存为 dump.exe。  
  
**四、修复输入表**  
　　运行脱壳文件，用 OD 自带的脱壳功能方法 1 脱壳的文件 1.exe 能正常运行，方法 2 脱的文件 2.exe 运行无反应；LordPE 脱壳的文件 dump.exe 运行报初始化失败，因此需要对输入表修复。  
　　运行 ImportREC\_fix，选取 PECompact 2.55.EXE 进程，OEP 改为 739D，点“自动查找 IAT”，再点“获取输入表”，点“显示无效的”，发现全是有效。（顺便带一句，如果此时有个别无效指针的话，可剪切掉；若有大量无效指针，则需要手工查找本软件调用的 API 函数，找出 IAT 的起始地址和计算出大小，手工填入 IAT 的 RVA 和大小。这里就不讲了，三人行的教程里有）。分别抓取 1.exe、2.exe、dump.exe，保存新文件 1\_.exe、2\_.exe、dump\_.exe，都能正常运行，修复成功！用 PEiD 查出编程语言是 Microsoft Visual C++ 7.0 Method2。  
  
**小结：**  
　　1、新手都应从基础练起，不要怕难，既然别人已经跟踪出了经验，就说明通过单步还是能跟出来的，那自己就要试试看。本例看似很复杂，但跟起来很快就有了结果，所以要有信心。  
　　2、当跟到无法继续时，就要按老师说的，看看附近有没有未实现的跳转，跟过去。还不行时，就要跟进 call 去看看，尽管如本例这些 call 看似调用的是系统领空，正如老师所说，每个壳都有自己的套路，我们不要中了 SEH 暗桩的陷阱。  
  
**判断是否到了 OEP 的小技巧：**  
　　这是我目前学习脱壳的一点心得，是不是正确还不知道，如有错误请高手指正，非常感谢！  
　　对于老师所说的“大跨段跳转”，在实际操作上有些困难，到底哪些算是“大跨段”，老师也说了有些不一定是很大，自己对编程语言 OEP 的特征码不熟悉，就常常没信心是不是到了 OEP。怎么办呢？我通过观察得出了一个规律，因为刚学没多久，老师的初级教程我都还没看完，是不是老师已经讲过这个规律，我不清楚；如果大家在网上看到相同的说法，请相信我不是抄袭，只是巧合罢了。  
这个小技巧就是：OD 载入程度后，点“M”图标，即内存映像。在 PEheader 下面的那一段（有的是.text，有的是.code，总之是紧接 PEheader 下面的那个代码段），看一下起始地址，加上大小就是结束地址，记下这个代码段的地址范围。加壳程序入口点一般不在这个范围，所以当 OD 跟踪到了这个地址范围，就可判断是 OEP 了，可直接 dump 了。但偷代码的壳就不能直接 dump 这么简单了（目前我还没学到脱这类壳的技术）。  
　　上面这个技巧一直都行得通，直到今天脱上面这个 PECompact 2.55 壳（我目前接触的壳还很少，有待进一步学习），OD 一载入就是 01001000，正在那个范围之内，一下子信心动摇了，好在才运行了几行代码就跑到别的段了，再返回来时，正是 OEP！所以到目前为止，这个规律还是行得通的，呵呵。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PECompact 2.55.rar \[解压密码：pediy\]  
 |