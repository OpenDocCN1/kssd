

# Acprotect1.10 Build123

> 原文：[`www.pediy.com/kssd/pediy06/pediy6424.htm`](https://www.pediy.com/kssd/pediy06/pediy6424.htm)

【脱文作者】 weiyi75\[Dfcg\]   
  
【作者邮箱】 weiyi75@sohu.com   
  
【作者主页】 Dfcg 官方大本营   
  
【使用工具】 Ollydbg1.10b(反 Antidbg 版),ImportREC1.42   
  
【破解平台】 Win2000/XP   
  
【软件名称】 PE 文件分析器   
    
【下载地址】 附件下载   
  
【软件简介】 Acprotect1.10 Build123“注册”版本加密的一个加密的一个 PE 文件分析器。   
  
【软件大小】 553k   
  
【加壳方式】 UltraProtect 1.x -> RISCO Software Inc.   
  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享。   
  
【破解目的】 模拟跟踪寻找 Stolen Code 具体内容。   
  
前言，这个软件是我用 Acprotect1.10 Build123“注册”版本加密的，没有使用 RSA 生成 KEY 保护，使用了所有反跟踪选项，关于动态代码替换保护软件已说明只支持 C++和 Dephi，呵呵，对准敌人的死穴点吧，就是模拟跟踪+内存镜像断点。   
  
先 OD 载入原程序看看      
  
004B7220 > $  55            PUSH EBP    //典型 Dephi 语言入口点,当 Acprotect 的壳运行到 OEP 时，你会发现入口点附近的一些代码被修改，如果你直接脱壳会导致不能运行。经过分析几个 Acprotect 修改入口代码个数，暂时发现一般是 6 个字节，有待验证。    
  
稍微说明一下:                                                       
  
每一种编译工具例如 : VC++ , Delphi , Borland , etc..                   
  
在 OEP 有一个唯一的/相同的 PE 头                                 
  
其中的一些是这样的:                                                
  
Push EBP    //第一句就是我们模拟跟踪的对象，幸运的是，目前 Acprotect 还没有采用 Asprotect 的入口代码变形技术，变了也没用?!                                                 
MOV Ebp,Esp                                                             
  
Add ESP , -010     
  
004B7221   .  8BEC          MOV EBP,ESP   
  
004B7223   .  83C4 F0       ADD ESP,-10   
  
004B7226   .  B8 A86F4B00   MOV EAX,Pe.004B6FA8   
  
004B722B   .  E8 6CF4F4FF   CALL Pe.0040669C   
  
004B7230   .  A1 C4A04B00   MOV EAX,DWORD PTR DS:\[4BA0C4\]   
  
004B7235   .  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
  
004B7237   .  E8 C48CFCFF   CALL Pe.0047FF00   
  
004B723C   .  A1 C4A04B00   MOV EAX,DWORD PTR DS:\[4BA0C4\]   
  
004B7241   .  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
  
004B7243   .  BA 80724B00   MOV EDX,Pe.004B7280   
  
004B7248   .  E8 AB88FCFF   CALL Pe.0047FAF8   
  
004B724D   .  8B0D F4A14B00 MOV ECX,DWORD PTR DS:\[4BA1F4\]            ;  Pe.004BBCAC   
  
004B7253   .  A1 C4A04B00   MOV EAX,DWORD PTR DS:\[4BA0C4\]   
  
004B7258   .  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
  
004B725A   .  8B15 48E44A00 MOV EDX,DWORD PTR DS:\[4AE448\]            ;  Pe.004AE494   
  
004B7260   .  E8 B38CFCFF   CALL Pe.0047FF18   
  
..........................................................................................   
  
我们再次复习 Dephi 语言入口特征码。   
  
这次 EBP=12fff0 不管用了，程序运行到入口时，EBP 的值根本不是 12fff0。   
  
OD 异常设置不忽略内存异常，其余全部忽略,载入程序,用插件隐藏 OD。   
      
004EF000 >  60              pushad //加壳程序入口点，F9 运行。    
  
004EF001    87E9            xchg ecx, ebp   
  
004EF003    F9              stc   
  
004EF004    41              inc ecx   
  
004EF005    BF 08589F45     mov edi, 459F5808   
  
004EF00A    50              push eax   
  
004EF00B    E8 01000000     call Pey.004EF011   
  
004EF010    7E 58           jle short Pey.004EF06A   
  
004EF012    58              pop eax   
  
004EF013    C1ED E9         shr ebp, 0E9   
  
004EF016    EB 01           jmp short Pey.004EF019   
  
004EF018  - 76 D3           jbe short Pey.004EEFED   
  
004EF01A    ED              in eax, dx   
  
004EF01B    EB 01           jmp short Pey.004EF01E   
  
004EF01D  - 75 85           jnz short Pey.004EEFA4   
  
........................................................................   
  
内存异常   
  
00415719    CD 01           int 1   //典型 Acprotect 小于 1.20 版最后一次异常。    
  
004F771B    40              inc eax   
  
004F771C    40              inc eax   
  
004F771D    0BC0            or eax, eax   
  
004F771F    75 05           jnz short Pey.004F7726   
  
004F7721    90              nop   
  
004F7722    90              nop   
  
004F7723    90              nop   
  
004F7724    90              nop   
  
004F7725    61              popad   
  
004F7726    33C0            xor eax, eax   
  
004F7728    64:8F00         pop dword ptr fs:\[eax\]   
  
004F772B    58              pop eax   
  
004F772C    60              pushad   
  
004F772D    E8 00000000     call Pey.004F7732   
  
........................................................................   
  
堆栈内容   
  
0012FF58   0012FFE0  指针到下一个 SEH 记录   
  
0012FF5C   004F76FD  SE 句柄   //这里同原来不同，为了跟踪到 Stolen Code，选择合适的地点跟踪很关键，在跟踪 N 个 Acprotect 程序后，找到了关键点，这些没什么技术可言，只是不断的实际，无数次的失败总结。右键对 004F76FD 转存中跟随，下内存访问断点。    
  
0012FF60   00000042   
  
0012FF64   59D26602   
  
........................................................................   
  
004156FD    8B5C24 0C       mov ebx, dword ptr ss:\[esp+C\]     //下断点 Shift+F9 中断 1    
  
004F7701    8383 B8000000 0>add dword ptr ds:\[ebx+B8\], 2   
  
004F7708    33C0            xor eax, eax   
  
004F770A    C3              retn   
  
004F770B    64:67:FF36 0000 push dword ptr fs:\[0\]   
  
004F7711    64:67:8926 0000 mov dword ptr fs:\[0\], esp   
  
004F7717    33C0            xor eax, eax   
  
004F7719    CD 01           int 1   
  
004F771B    40              inc eax   
  
004F771C    40              inc eax   
  
004F771D    0BC0            or eax, eax   
  
004F771F    75 05           jnz short Pey.004F7726   
  
004F7721    90              nop   
  
004F7722    90              nop   
  
004F7723    90              nop   
  
004F7724    90              nop   
  
004F7725    61              popad   
  
........................................................................   
  
004F774D    8B048E          mov eax, dword ptr ds:\[esi+ecx\*4\] //下断点 Shift+F9 中断 2,清除内存断点。    
  
004F7750    8B5C8E 04       mov ebx, dword ptr ds:\[esi+ecx\*4+4\]   
  
004F7754    2BC3            sub eax, ebx   
  
004F7756    C1C8 11         ror eax, 11   
  
004F7759    03C2            add eax, edx   
  
004F775B    81F2 B0362B9F   xor edx, 9F2B36B0   
  
004F7761    89048E          mov dword ptr ds:\[esi+ecx\*4\], eax   
  
004F7764    49              dec ecx   
  
004F7765  ^ EB E1           jmp short Pey.004F7748   
  
004F7767    61              popad   
  
004F7768    61              popad   
  
004F7769    C3              retn   //F4 直接下来，这时是模拟跟踪的时候了。    
  
........................................................................   
  
Ctrl+T 设置暂停运行跟踪条件   
  
将命令是一个复选框打勾，填入  PUSH EBP    ,确定。   
  
这个条件断点如何在命令行设置，肯请诸位大侠不吝指教。为什么要设置这个命令，先前已说明这是 Acprotect 的死穴之一，对准它点吧。   
  
Ctrl+F11 跟踪进入每个 Call,1 分钟左右。   
  
004254C9    55              push ebp       //Stolen Code    
  
004254CA    8BEC            mov ebp, esp  //Stolen Code   //一切尽在掌握    
  
004254CC    83EC 44         ADD ESP,-10  //Stolen Code      
  
004254CF    60              pushad   
  
004254D0    60              pushad   
  
005074D1    E8 00000000     call Pey.005074D6   
  
也可以下个 12ffc0 的硬件写入 dword 断点，可以断在 4254CA 这句   
  
........................................................................   
  
这个程序抽了六个字节。   
  
ALT+M 打开内存镜像断点,对准它的第二个死穴点。     
  
 内存镜像，项目 21   
  
 地址=00401000     //Code 段下内存访问断点   
  
 大小=000B7000 (749568.)   
  
 Owner=Pey      00400000   
  
 区段=CODE   
  
 包含=code   
  
 类型=Imag 01001002   
  
 访问=R   
  
 初始访问=RWE   
  
    
F9 运行到达   
  
004B7220    BF C12D8DE2     mov edi, E28D2DC1   
  
004B7225    90              nop   
  
004B7226    B8 A86F4B00     mov eax, Pey.004B6FA8   //临时 Oep,滚动条向上看,如何判断抽掉多少字节，用滚动条是代码会混乱，然后用 Ctrl+向上下键配合保证看到临时 Oep 代码完整，并且上面的要有几行代码同时存在,如我现在调整的样式。    
  
004B722B    E8 6CF4F4FF     call Pey.0040669C   
  
004B7230    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7235    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B7237    E8 C48CFCFF     call Pey.0047FF00   
  
004B723C    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7241    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B7243    BA 80724B00     mov edx, Pey.004B7280   
  
004B7248    E8 AB88FCFF     call Pey.0047FAF8   
  
004B724D    8B0D F4A14B00   mov ecx, dword ptr ds:\[4BA1F4\]           ; Pey.004BBCAC   
  
004B7253    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7258    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B725A    8B15 48E44A00   mov edx, dword ptr ds:\[4AE448\]           ; Pey.004AE494   
  
004B7260    E8 B38CFCFF     call Pey.0047FF18   
  
........................................................................   
  
OD 直接修复找到的六个字节。   
  
004B7220    55              push ebp  //用 Od 插件修正入口为 B7220 直接脱壳吧，重建输入表的勾去掉    
  
004B7221    8BEC            mov ebp, esp   
  
004B7223    83C4 F0         add esp, -10   
  
004B7226    B8 A86F4B00     mov eax, Pey.004B6FA8   
  
004B722B    E8 6CF4F4FF     call Pey.0040669C   
  
004B7230    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7235    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B7237    E8 C48CFCFF     call Pey.0047FF00   
  
004B723C    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7241    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B7243    BA 80724B00     mov edx, Pey.004B7280   
  
004B7248    E8 AB88FCFF     call Pey.0047FAF8   
  
004B724D    8B0D F4A14B00   mov ecx, dword ptr ds:\[4BA1F4\]           ; Pey.004BBCAC   
  
004B7253    A1 C4A04B00     mov eax, dword ptr ds:\[4BA0C4\]   
  
004B7258    8B00            mov eax, dword ptr ds:\[eax\]   
  
004B725A    8B15 48E44A00   mov edx, dword ptr ds:\[4AE448\]           ; Pey.004AE494   
  
004B7260    E8 B38CFCFF     call Pey.0047FF18   
  
........................................................................   
  
修复 IAT 必须关闭 OD 单独开加壳程序修复，不然 ImportREC 停止响应，最后用 ImportREC Oep 填 B7220,跟踪等级三轻松修复,无法运行!?   
寻找出错原因。   
  
004F01F8   /78 01           js short dumped\_.004F01FB   
  
004F01FA   |F8              clc   
  
004F01FB   BF 58D5CC8F     mov edi, 8FCCD558   
  
004F0200    8B45 00         mov eax, dword ptr ss:\[ebp\]   
  
004F0203    7C 03           jl short dumped\_.004F0208   
  
004F0205    7D 01           jge short dumped\_.004F0208   
  
004F0207    E9 E9040000     jmp dumped\_.004F06F5   
  
004F020C    004E 66         add byte ptr ds:\[esi+66\], cl   
  
004F020F    1BD9            sbb ebx, ecx   
  
004F0211    33C2            xor eax, edx   
  
004F0213    50              push eax   
  
004F0214    E8 01000000     call dumped\_.004F021A //发现这个 Call 可能是新版 Acprotect 的解码技术，和完美卸载 Xp 里面的一样的，这一段修复一直没搞定，比原先的 Jmp xxxxx  那种语句难多了。    
  
004F0219    EA 58587B05 668>jmp far 8166:057B5858   
  
不好意思，文章没写完,卡住了。   
  
今天看到 Fly 大侠的文章，呵呵，明白是什么一会事。   
  
其实本版本 Acprotect 嵌有 Oep 处代码检验程序，发现 Oep 处代码被替换，立即拒绝解码，你失去了关键的代码当然程序无法运行，还要承认 N 次错误确定按钮。   
  
004254C9    55              push ebp       //Stolen Code    
  
004254CA    8BEC            mov ebp, esp  //Stolen Code   //一切尽在掌握    
  
004254CC    83EC 44         ADD ESP,-10  //Stolen Code    
  
我们已经掌握。   
  
004B7226    B8 A86F4B00     mov eax, Pey.004B6FA8   //临时 Oep 也知道。    
  
现在做什么,让程序认为你没有脱壳,乖乖将 perplex 段里面有用的内容全部解码。   
  
方法   
  
push ebp          //Stolen Code    
  
mov ebp, esp     //Stolen Code   //一切尽在掌握    
  
ADD ESP,-10     //Stolen Code   
  
jmp  004B7226  //直接跳向临时 Oep   
  
Od 载入脱壳修复后的程序,Ctrl+G 004EF000   
    
004EF000 >/$  68 420E715B   PUSH 5B710E42  //4EF000 是原来壳的入口点，狸猫换太子，将 Oep 入口代码复制到这里。    
  
004EF005  |.  813424 A27989>XOR DWORD PTR SS:\[ESP\],2C8979A2   
  
004EF00C  .  C3            RETN   
  
004EF00D   .  68 59266136   PUSH 36612659   
  
004EF012   .  813424 248C99>XOR DWORD PTR SS:\[ESP\],41998C24   
  
004EF019   .  C3            RETN   
  
004EF01A   .  68 24C959F5   PUSH F559C924   
  
004EF01F   .  813424 6863A1>XOR DWORD PTR SS:\[ESP\],82A16368   
  
004EF026   .  C3            RETN   
  
004EF027   .  68 CF904873   PUSH 734890CF   
  
004EF02C   .  813424 C01DAE>XOR DWORD PTR SS:\[ESP\],4AE1DC0   
  
004EF033   .  C3            RETN   
  
.........................................................   
  
004EF000 > .  55            PUSH EBP   
  
004EF001   .  8BEC          MOV EBP,ESP   
  
004EF003   .  83C4 F0       ADD ESP,-10   
  
004EF006   .- E9 1B82FCFF   JMP dumpedw4.004B7226   
  
004EF00B      90            NOP   //注意保持代码完整，填入一个 Nop    
  
004EF00C   .  C3            RETN   
  
004EF00D   .  68 59266136   PUSH 36612659   
  
004EF012   .  813424 248C99>XOR DWORD PTR SS:\[ESP\],41998C24   
  
004EF019   .  C3            RETN   
  
004EF01A   .  68 24C959F5   PUSH F559C924   
  
004EF01F   .  813424 6863A1>XOR DWORD PTR SS:\[ESP\],82A16368   
  
004EF026   .  C3            RETN   
  
004EF027   .  68 CF904873   PUSH 734890CF   
  
004EF02C   .  813424 C01DAE>XOR DWORD PTR SS:\[ESP\],4AE1DC0   
  
004EF033   .  C3            RETN   
  
.........................................................   
  
将替换的代码复制到程序中，另存为一个文件，用 PEedit 修正入口为 EF000,正常运行。   
  
    
【破解总结】发现 Dephi 语言的 Acprotect 的壳采用了 Stolen Code,动态解码，用 Acprotect 保护的它人没加上 RSAKEY 就有强劲的保护，加 RSAKEY 保护壳继续研究中,感谢你耐心看完.   
  
  
 4dd466d0e1e0cc123afbe504da2462ca