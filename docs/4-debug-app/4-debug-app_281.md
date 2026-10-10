

# Microsoft 网上红心大战看牌分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-134.htm`](https://www.pediy.com/kssd/pediy09/pediy09-134.htm)

> 【文章标题】: Microsoft 网上红心大战看牌分析  
> 【文章作者】: PKLANG  
> 【软件名称】: mshearts.exe  
> 【使用工具】: OD,IDA,ResHACK。  
> 【下载地址】: WINXP 自带有,其它系统自己找看看  
> 【保护方式】: 无  
> 【作者声明】: 只是感兴趣，练习之用,没有其他目的。失误之处敬请诸位大侠赐教! 5.1 节快乐，GIS 工作者辛苦了！  
>   
> windows XP 自带有个<Microsoft 网上红心大战>,说网上其实和电脑玩的,不过还是蛮好玩的,具体规则是:  
> 选出三张牌传给其他对手（第四局不传牌）。如要选牌，单击即可。如要取消选定的牌，请再次单击。   
> 抓有梅花 2 的玩家首先出梅花 2。   
> 然后按顺时针方向出牌。每位玩家依次必须跟同花色牌。如果已经没有与发牌花色相同的牌，则可以出任何一张牌。唯一例外是不能在第一圈牌中出红桃或黑桃皇后。  
> 每一轮结束时，每张红心计 1 分，黑桃皇后计 13 分。游戏将持续到有人得 100 分或更多分或者庄家退出游戏时结束。   
> 如果在一轮牌中赢得了所有的红心和黑桃皇后（称之为“全收”），则您得零分，其他人每人得 26 分。   
> “红心大战”游戏的目标是争取游戏结束时得分最低。   
>   
> 现在我们来分析它,并显示电脑玩家的底牌.  
> (先声明,我是菜鸟,学破解刚入门不入,在看雪学到不少东东,在此先感谢看雪论坛,祝看雪越来越好!  
>  这是我的处女作,高手们不要见笑啊,有什么不对的多指点和批评,也希望能结识对破解分析有辟好的朋友:~~)  
>   
> OK,Let's go  
> 用 PEID 看了一下,竟然什么都没有,不会是加壳了吧?不管那么多用打开 OD 装入红心大战,也没提示什么警告,看了一下  
> 应该是用 MFC 写的,那怎么用 PEID 查不出呢?莫非威软用自己的编译器来编译 MFC???? 废话少说...用 IDA 打开反汇编导入  
> map 然后在 OD 用插件装入 map 文件,这样就容易看点,不然在 OD 里 N 多 call MFC42u.#XXXX 的  
> 打开 OD 装入红心大战,F9 运行,看了游戏一些菜单,界面等,在菜单有声音选项,把声音给沟上,玩几把就知道有两种  
> 声音,出黑桃 Q 和第一次出红桃时各播放放一种声音,用 右键->查找->所以模块间的调用,找到两个 sndPlaySoundW,在它  
> 设置断点,然后在游戏里出牌到黑桃 Q 时就断到这里来了:  
> 010098D5 >/$  57            push    edi  
> 010098D6  |.  33FF          xor     edi, edi  
> 010098D8  |.  39B9 18010000 cmp     dword ptr ds:\[ecx+118\], edi  
> 010098DE  |.  74 26         je      short <mshearts.loc\_1009906>  
> 010098E0  |.  397C24 08     cmp     dword ptr ss:\[esp+8\], edi  
> 010098E4  |.  75 18         jnz     short <mshearts.loc\_10098FE>  
> 010098E6  |.  393D 0CD70001 cmp     dword ptr ds:\[100D70C\], edi  
> 010098EC  |.  74 18         je      short <mshearts.loc\_1009906>  
> 010098EE  |.  57            push    edi  
> 010098EF  |.  57            push    edi  
> 010098F0  |.  FF15 00140001 call    dword ptr ds:\[<&WINMM.sndPlaySou>;  WINMM.sndPlaySoundW  
> 010098F6  |.  893D 0CD70001 mov     dword ptr ds:\[100D70C\], edi  
> 010098FC  |.  EB 08         jmp     short <mshearts.loc\_1009906>  
> 010098FE >|>  39B9 20010000 cmp     dword ptr ds:\[ecx+120\], edi  
> 01009904  |.  75 05         jnz     short <mshearts.loc\_100990B>  
> 01009906 >|>  33C0          xor     eax, eax  
> 01009908  |.  40            inc     eax  
> 01009909  |.  EB 57         jmp     short <mshearts.loc\_1009962>  
> 0100990B >|>  393D 0CD70001 cmp     dword ptr ds:\[100D70C\], edi  
> 01009911  |.  74 06         je      short <mshearts.loc\_1009919>  
> 01009913  |.  57            push    edi  
> 01009914  |.  E8 BCFFFFFF   call    <mshearts.sub\_10098D5>  
> 01009919 >|>  56            push    esi  
> 0100991A  |.  E8 C90D0000   call    <mshearts.AfxGetModuleState(void>;  jmp 到 MFC42u.#1165  
> 0100991F  |.  8B70 08       mov     esi, dword ptr ds:\[eax+8\]  
> 01009922  |.  0FB74424 0C   movzx   eax, word ptr ss:\[esp+C\]  
> 01009927  |.  68 6C230001   push    <mshearts.Type>                  ; /ResourceType = "WAVE"  
> 0100992C  |.  50            push    eax                              ; |ResourceName  
> 0100992D  |.  56            push    esi                              ; |hModule  
> 0100992E  |.  FF15 7C100001 call    dword ptr ds:\[<&KERNEL32.FindRes>; \\FindResourceW  
> 01009934  |.  3BC7          cmp     eax, edi  
> 01009936  |.  74 27         je      short <mshearts.loc\_100995F>  
> 01009938  |.  50            push    eax                              ; /hResource  
> 01009939  |.  56            push    esi                              ; |hModule  
> 0100993A  |.  FF15 80100001 call    dword ptr ds:\[<&KERNEL32.LoadRes>; \\LoadResource  
> 01009940  |.  3BC7          cmp     eax, edi  
> 01009942  |.  A3 0CD70001   mov     dword ptr ds:\[100D70C\], eax  
> 01009947  |.  74 16         je      short <mshearts.loc\_100995F>  
> 01009949  |.  50            push    eax                              ; /nHandles  
> 0100994A  |.  FF15 84100001 call    dword ptr ds:\[<&KERNEL32.LockRes>; \\SetHandleCount  
> 01009950  |.  3BC7          cmp     eax, edi  
> 01009952  |.  74 0B         je      short <mshearts.loc\_100995F>  
> 01009954  |.  6A 07         push    7  
> 01009956  |.  50            push    eax  
> 01009957  |.  FF15 00140001 call    dword ptr ds:\[<&WINMM.sndPlaySou>;  WINMM.sndPlaySoundW 断在这里  
> 0100995D  |.  EB 02         jmp     short <mshearts.loc\_1009961>  
> 0100995F >|>  33C0          xor     eax, eax  
> 01009961 >|>  5E            pop     esi  
> 01009962 >|>  5F            pop     edi  
> 01009963  \\.  C2 0400       retn    4  
>   
> 从代码看得出声音文件是从资源读取的,用 ResHacker 看资源有两个声音资源文件,分别为 401 和 402  
> CTRL+F9 执行到返回,F8 后到  
>   
> 010079CD >/$  B8 CBB30001   mov     eax, <mshearts.loc\_100B3CB>  
> 010079D2  |.  E8 99320000   call    <mshearts.sub\_100AC70>  
> 010079D7  |.  51            push    ecx  
> 010079D8  |.  53            push    ebx  
> 010079D9  |.  56            push    esi  
> 010079DA  |.  8BF1          mov     esi, ecx  
> 010079DC  |.  83BE 68010000>cmp     dword ptr ds:\[esi+168\], 0  
> 010079E3  |.  8B86 54010000 mov     eax, dword ptr ds:\[esi+154\]  
> 010079E9  |.  57            push    edi  
> 010079EA  |.  8BBC86 580100>mov     edi, dword ptr ds:\[esi+eax\*4+158>  
> 010079F1  |.  75 23         jnz     short <mshearts.loc\_1007A16>  
> 010079F3  |.  8B07          mov     eax, dword ptr ds:\[edi\]  
> 010079F5  |.  6A 04         push    4  
> 010079F7  |.  99            cdq  
> 010079F8  |.  59            pop     ecx  
> 010079F9  |.  F7F9          idiv    ecx  
> 010079FB  |.  83FA 02       cmp     edx, 2                ;这个好像是否是第一次出红桃的  
> 010079FE  |.  75 16         jnz     short <mshearts.loc\_1007A16>  
> 01007A00  |.  68 91010000   push    191        ;191 十进制就是 401  
> 01007A05  |.  8BCE          mov     ecx, esi  
> 01007A07  |.  C786 68010000>mov     dword ptr ds:\[esi+168\], 1  
> 01007A11  |.  E8 BF1E0000   call    <mshearts.sub\_10098D5>   ;调用刚才的函数  
> 01007A16 >|>  833F 2F       cmp     dword ptr ds:\[edi\], 2F   ;是否为黑桃 Q,F2 十进制是 47  
> 01007A19  |.  75 16         jnz     short <mshearts.loc\_1007A31>  
> 01007A1B  |.  68 92010000   push    192                      ;192 十进制就是 402  
> 01007A20  |.  8BCE          mov     ecx, esi  
> 01007A22  |.  C786 6C010000>mov     dword ptr ds:\[esi+16C\], 1  
> 01007A2C  |.  E8 A41E0000   call    <mshearts.sub\_10098D5>  ;调用刚才的函数  
> 01007A31 >|>  8B8E 54010000 mov     ecx, dword ptr ds:\[esi+154\] ;F8 后回到这里  
> 01007A37  |.  8BC1          mov     eax, ecx  
> 01007A39  |.  2B86 40010000 sub     eax, dword ptr ds:\[esi+140\]  
> 01007A3F  |.  6A 04         push    4  
> 01007A41  |.  83C0 04       add     eax, 4  
> 在这里我们知道黑桃 Q 是值为 2F(47),我们可以在 01007A16 设置断点来分析各个牌的数值  
> 通过分析知道(梅花 A)->0 (方块 A)->1 (红桃 A)->2 (黑桃 A)->3 (方块 2)->4 到(黑桃 K)->51   
>   
> 接下来我们要知道 cmp     dword ptr ds:\[edi\], 2F 里 edi 的地址  
> 我们可以通过在 edi 地址写入时中断下来,我在这里 EDI 是 00039AAC,右键->数据窗口跟随   
> 在数据窗口看到  
> 00039AAC  2F 00 00 00 09 00 00 00 4B 00 00 00 02 00 00 00  .......K......  
> 00039ABC  10 00 00 00 09 00 00 00 5A 00 00 00 00 00 00 00  .......Z.......  
> 00039ACC  1D 00 00 00 09 00 00 00 69 00 00 00 00 00 00 00  .......i.......  
> 00039ADC  31 00 00 00 09 00 00 00 78 00 00 00 00 00 00 00  1.......x.......  
> 我们在上面 2F 00 00 00 设置硬件写入断点(DWORD),F9 运行  
> 在这里断下来  
> 010086A3  |.  834D FC FF    |or      dword ptr ss:\[ebp-4\], FFFFFFFF ;写入 FFFFFFFF  
> 010086A7  |.  8D4D C4       |lea     ecx, dword ptr ss:\[ebp-3C\]  
> 这个不是我们要找的,不理,继续 F9 运行,当打完一局后重新发牌时  
> 在这里断下  
> 01007F89 >/$  B8 93B40001   mov     eax, <mshearts.loc\_100B493>  
> 01007F8E  |.  E8 DD2C0000   call    <mshearts.sub\_100AC70>  
> 01007F93  |.  81EC 04010000 sub     esp, 104  
> 01007F99  |.  53            push    ebx  
> 01007F9A  |.  56            push    esi  
> 01007F9B  |.  57            push    edi  
> 01007F9C  |.  6A 34         push    34                 ;把 0x34(52)个位置  
> 01007F9E  |.  8BF1          mov     esi, ecx           ;esi 从 ECX 来  
> 01007FA0  |.  33C0          xor     eax, eax  
> 01007FA2  |.  59            pop     ecx  
> 01007FA3 >|>  898485 F0FEFF>/mov     dword ptr ss:\[ebp+eax\*4-110\], eax  
> 01007FAA  |.  40            |inc     eax  
> 01007FAB  |.  3BC1          |cmp     eax, ecx  
> 01007FAD  |.^ 7C F4         \\jl      short <mshearts.loc\_1007FA3>  ;循环把 0x34(52)个位置置 0  
> 01007FAF  |.  8365 EC 00    and     dword ptr ss:\[ebp-14\], 0  
> 01007FB3  |.  894D E8       mov     dword ptr ss:\[ebp-18\], ecx      ;保存牌的数量  
> 01007FB6 >|>  FF15 08140001 /call    dword ptr ds:\[<unk\_100D408>\]   ; \[rand 产生随机数  
> 01007FBC  |.  99            |cdq  
> 01007FBD  |.  F77D E8       |idiv    dword ptr ss:\[ebp-18\]          ;rand()%52==得到 0-51  
> 01007FC0  |.  8B45 EC       |mov     eax, dword ptr ss:\[ebp-14\]  
> 01007FC3  |.  6A 0D         |push    0D  
> 01007FC5  |.  5F            |pop     edi  
> 01007FC6  |.  6A 04         |push    4  
> 01007FC8  |.  5B            |pop     ebx  
> 01007FC9  |.  8BCA          |mov     ecx, edx  
> 01007FCB  |.  99            |cdq  
> 01007FCC  |.  F7FF          |idiv    edi  
> 01007FCE  |.  2B86 40010000 |sub     eax, dword ptr ds:\[esi+140\]  
> 01007FD4  |.  8BFA          |mov     edi, edx  
> 01007FD6  |.  83C0 04       |add     eax, 4  
> 01007FD9  |.  99            |cdq  
> 01007FDA  |.  F7FB          |idiv    ebx  
> 01007FDC  |.  8D848D F0FEFF>|lea     eax, dword ptr ss:\[ebp+ecx\*4-110\]  
> 01007FE3  |.  8B18          |mov     ebx, dword ptr ds:\[eax\]  
> 01007FE5  |.  C1E7 04       |shl     edi, 4       ;每后一个偏移 16   
> 01007FE8  |.  8D8C96 300100>|lea     ecx, dword ptr ds:\[esi+edx\*4+130\] ;edx 为玩家位置(0 为自己)  
> 01007FEF  |.  8B11          |mov     edx, dword ptr ds:\[ecx\]          ;放入地址  
> 01007FF1  |.  895C17 1C     |mov     dword ptr ds:\[edi+edx+1C\], ebx  ;断在这里  
> 01007FF5  |.  8B09          |mov     ecx, dword ptr ds:\[ecx\]  
> 01007FF7  |.  33DB          |xor     ebx, ebx  
> 01007FF9  |.  FF4D E8       |dec     dword ptr ss:\[ebp-18\]  
> 01007FFC  |.  FF45 EC       |inc     dword ptr ss:\[ebp-14\]            ;发完一个后加 1  
> 01007FFF  |.  837D EC 34    |cmp     dword ptr ss:\[ebp-14\], 34       
> 01008003  |.  895C0F 28     |mov     dword ptr ds:\[edi+ecx+28\], ebx  
> 01008007  |.  8B4D E8       |mov     ecx, dword ptr ss:\[ebp-18\]  
> 0100800A  |.  8B8C8D F0FEFF>|mov     ecx, dword ptr ss:\[ebp+ecx\*4-110\]  
> 01008011  |.  8908          |mov     dword ptr ds:\[eax\], ecx  
> 01008013  |.^ 7C A1         \\jl      short <mshearts.loc\_1007FB6>   ;还没有发完继续发牌  
> 01008015  |.  83BE 44010000>cmp     dword ptr ds:\[esi+144\], 3  
>   
> 在这里应该是发牌的了,看到 rand 来随机发牌,先把硬件断点清除,然后慢慢分析...  
> 通过分析后知道牌放的地址为 \[esi+n\*4+0x130\]+0x1C+i\*16 n 为玩家序号,i 为牌序号  
> 现在关键是看 esi 地址是怎么得来呢?向上翻看到 esi 是从开始的 ECX 得到的,那 ECX 又从哪里来的呢  
> 能过跟踪后是从 new int 来的,就是动态分配来的,并放到一个全局变量里去  
> 这个全局变量地址是 100D514,具体是怎么找到留给那些有兴趣的朋友去分析吧,自己动手比单纯看要  
> 学到的东西很多的  
>   
> 地址基本都找到了下来就可以读出来显示出来作弊了:)  
>   
> 我用 VC 写了一个显示看牌的程序  
> 主要代码如下  
> void CDemoVCDlg::OnButtonTest()   
> {  
>   
>   DWORD dword\_100D514=0x100D514;  
>   DWORD addr;  
>   DWORD addData\[4\];  
>   BYTE isOut;  
>   
>   DWORD dwProcessId;  
>   HWND hWnd = ::FindWindow(NULL,"Microsoft 网上红心大战");  
>   if(hWnd == NULL)  
>   {  
>     KillTimer(110);  
>     m\_bAutoRefresh=FALSE;  
>     UpdateData(FALSE);  
>     MessageBox("Microsoft 网上红心大战程序没有运行","错误", MB\_ICONERROR);  
>     return ;  
>   }  
>   
>   GetWindowThreadProcessId(hWnd,&dwProcessId);  
>   HANDLE hProcess = ::OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwProcessId);  
>   if(hProcess == NULL)  
>   {  
>     KillTimer(110);  
>     m\_bAutoRefresh=FALSE;  
>     UpdateData(FALSE);  
>     MessageBox("打开进程失败","错误", MB\_ICONERROR);  
>     return ;  
>   }  
>   //读取全局牌的地址    
>   ReadProcessMemory(hProcess, (LPVOID)dword\_100D514, (LPVOID)&addr, 4, NULL);  
>   
>   //通过全局地址 addr+n\*4+0x130   
>   //分别读取四个玩家牌的地址  
>   ReadProcessMemory(hProcess, (LPVOID)(addr+0x130), (LPVOID)&addData\[0\], 4, NULL);  
>   ReadProcessMemory(hProcess, (LPVOID)(addr+4+0x130), (LPVOID)&addData\[1\], 4, NULL);  
>   ReadProcessMemory(hProcess, (LPVOID)(addr+8+0x130), (LPVOID)&addData\[2\], 4, NULL);  
>   ReadProcessMemory(hProcess, (LPVOID)(addr+12+0x130), (LPVOID)&addData\[3\], 4, NULL);  
>   
>   //每个玩家有 13 张牌  
>   for( int i=0;i<13;i++ )  
>   {  
>     //偏移 0x1C 才是牌号真正开始位置,每后一个偏移 16(0x10)个字节  
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[0\]+i\*16+0x1c), (LPVOID)&m\_paiData\[0\]\[i\], 4, NULL);  
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[1\]+i\*16+0x1c), (LPVOID)&m\_paiData\[1\]\[i\], 4, NULL);  
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[2\]+i\*16+0x1c), (LPVOID)&m\_paiData\[2\]\[i\], 4, NULL);  
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[3\]+i\*16+0x1c), (LPVOID)&m\_paiData\[3\]\[i\], 4, NULL);  
>       
>   
>     //牌好只用 4 个字节,后面还有 12 个字节,我还不知道有什么用  
>     //从分析可以看出,当出牌时后 4 个字节会有变化的,而牌号还没变,  
>     //当一轮完后才变成 0XFFFFFFFF,后面 12 个字节也发生变化  
>     //我经过比较当出牌后四个玩家的第五个有规律的,分别是 E1,C8,EB,04  
>     //所以我把当是已经出过的牌就设置为 0XFFFFFFFFF,这样看起来就同步了  
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[0\]+i\*16+0x1c+4), (LPVOID)&isOut, 1, NULL);  
>     if( isOut==0xE1 )m\_paiData\[0\]\[i\]=0xFFFFFFFF;  
>   
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[1\]+i\*16+0x1c+4), (LPVOID)&isOut, 1, NULL);  
>     if( isOut==0xC8 )m\_paiData\[1\]\[i\]=0xFFFFFFFF;  
>   
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[2\]+i\*16+0x1c+4), (LPVOID)&isOut, 1, NULL);  
>     if( isOut==0xEB )m\_paiData\[2\]\[i\]=0xFFFFFFFF;  
>     
>     ReadProcessMemory(hProcess, (LPVOID)(addData\[3\]+i\*16+0x1c+4), (LPVOID)&isOut, 1, NULL);  
>     if( isOut==0x04 )m\_paiData\[3\]\[i\]=0xFFFFFFFF;  
>   
>   
>   }  
>    ::CloseHandle(hProcess);  
>    m\_bRead=TRUE;  
>    //读出的牌除了自己是排序好外其他都没有排序的,所以还要把它排序才容易看牌  
>    //Sort(&m\_paiData\[0\]\[0\]);//自己的牌已经帮排好了!  
>    Sort(&m\_paiData\[1\]\[0\]);  
>    Sort(&m\_paiData\[2\]\[0\]);  
>    Sort(&m\_paiData\[3\]\[0\]);  
>    RedrawWindow();  
>   
> }  
>   
> void CDemoVCDlg::Sort(DWORD \*data)//排序  
> {  
>   int i,j,k,t;  
>   for( i = 0 ; i < 13-1 ; i++ )  
>   {  
>     for( k = i ,j = i+1 ; j <13 ; j++ )  
>     {  
>             if( data\[k\]%4 >= data\[j\]%4 ) //先把牌的类型排序  
>       {    
>         if( data\[k\]%4 == data\[j\]%4)//类型相等时  
>         {  
>           if(  data\[k\]/4==0 || data\[j\]/4==0 )//如果是遇到 A 时把 A 排在后面  
>           {  
>             if( data\[k\]/4 < data\[j\]/4 )  
>               k=j;  
>           }  
>           else  //不然按小大排  
>           {  
>             if( data\[k\]/4 > data\[j\]/4  )  
>               k = j;  
>           }  
>         }  
>   
>         else  
>         {  
>           //把黑桃排在红桃前面  
>           if( (data\[k\]%4==2 && data\[j\]%4==3) || (data\[k\]%4==3 && data\[j\]%4==2) )  
>           {  
>             if( data\[k\]%4 < data\[j\]%4)  
>               k = j;  
>           }  
>           else  
>             k = j;  
>         }  
>       }  
>       //把黑桃排在红桃前面  
>       else if( (data\[k\]%4==2 && data\[j\]%4==3) || (data\[k\]%4==3 && data\[j\]%4==2) )  
>       {  
>         if( data\[k\]%4 < data\[j\]%4)  
>           k = j;  
>       }  
>   
>           
>       
>     }  
>     if( k != i )  
>     {  
>       t =  data\[i\];  
>       data\[i\]=data\[k\];  
>       data\[k\]= t;  
>     }  
>   }  
>   
>   
> }  
>   
>   
>   
> ![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif")

 4dd466d0e1e0cc123afbe504da2462ca