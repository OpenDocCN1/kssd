

# SynchroMagic

> 原文：[`www.pediy.com/kssd/pediy06/pediy6967.htm`](https://www.pediy.com/kssd/pediy06/pediy6967.htm)

【脱文作者】 weiyi75\[Dfcg\]   
  
【作者邮箱】 weiyi75@sohu.com   
  
【作者主页】 Dfcg 官方大本营   
  
【使用工具】 Ollydbg1.09d,ImportREC1.42,Peid   
  
【破解平台】 Win2000/XP   
  
【软件名称】 SynchroMagic V3.0.557   
  
【下载地址】 http://www.skycn.com/soft/2889.html   
  
【软件简介】  SynchroMagic 是一个文件备份同步系统，对指定的文件、文件夹做同步、备份、多馀删除的动作，并支持 FTP 功能，可以将电脑与 FTP Server 中的文件同步，不论你走到哪里，时时确保都有最新的文件在手边。   
    
【软件大小】 2022 KB   
  
【加壳方式】 ASProtect 1.23 RC4 Registered -> Alexey Solodovnikov   
  
【脱壳目的】 查找 Asprotect1.23 RC4 的 Stolen Code,由于 Asprotect 动态地址分配,初学者查 Stolen Code 的具体位置不是很容易的,针对此点，详细写出脱壳过程。   
  
【破解声明】 本文感谢 csjwaman\[DFCG\]和 JustICE\[DFCG\]技术指导，我是一只小菜鸟，偶得一点心得，愿与大家分享。   
  
  
\--------------------------------------------------------------------------------   
  
【破解内容】   
  
    
首先 Peid 查壳，脱壳，为 ASProtect 1.23 RC4 Registered -> Alexey Solodovnikov。Ollydbg 载入程序，插件隐藏 OD，OD 异常设置忽略除内存异常外的全部异常。   
  
由于 Asprotect 动态地址分配,大家看到本文内存地址将和你不同，但看到的代码基本一样的，脱壳开始。   
  
00401000 >  68 01005500     push Synchro.00550001      //程序开始点，F9 运行。    
00401005    E8 01000000     call Synchro.0040100B   
0040100A    C3              retn   
0040100B    C3              retn   
0040100C    27              daa   
0040100D    60              pushad   
0040100E    FB              sti   
0040100F    8ADB            mov bl, bl   
00401011    A7              cmps dword ptr ds:\[esi\], dword ptr es:\[e>   
00401012    0D BFAC50FC     or eax, FC50ACBF   
00401017    853B            test dword ptr ds:\[ebx\], edi   
00401019    F3:             prefix rep:   
0040101A    FE              ???                                      ; 未知命令   
0040101B    39CA            cmp edx, ecx   
.....................................................................   
  
程序异常。   
  
00EB4524    3100            xor dword ptr ds:\[eax\], eax   
00EB4526    EB 01           jmp short 00EB4529   
00EB4528    68 648F0500     push 58F64   
00EB452D    0000            add byte ptr ds:\[eax\], al   
00EB452F    00EB            add bl, ch   
00EB4531    02E8            add ch, al   
00EB4533    0158 68         add dword ptr ds:\[eax+68\], ebx   
00EB4536    34 B4           xor al, 0B4   
00EB4538    EA 00689445 EB0>jmp far 00EB:45946800   
00EB453F    68 FC39EB00     push 0EB39FC   
00EB4544    68 C436EB00     push 0EB36C4   
00EB4549    68 7030EB00     push 0EB3070   
00EB454E    68 582AEB00     push 0EB2A58   
00EB4553    68 BC3DEB00     push 0EB3DBC   
00EB4558    C3              retn   
.....................................................................   
  
然后按 Shift+F9 29 次到程序最后一次异常处。   
  
00EB3D5F    3100            xor dword ptr ds:\[eax\], eax  //典型最后一次内存异常。    
00EB3D61    64:8F05 0000000>pop dword ptr fs:\[0\]   
00EB3D68    58              pop eax   
00EB3D69    833D BC7EEB00 0>cmp dword ptr ds:\[EB7EBC\], 0   
00EB3D70    74 14           je short 00EB3D86   
00EB3D72    6A 0C           push 0C   
00EB3D74    B9 BC7EEB00     mov ecx, 0EB7EBC   
00EB3D79    8D45 F8         lea eax, dword ptr ss:\[ebp\-8\]   
00EB3D7C    BA 04000000     mov edx, 4   
00EB3D81    E8 8ED2FFFF     call 00EB1014   
00EB3D86    FF75 FC         push dword ptr ss:\[ebp\-4\]   
00EB3D89    FF75 F8         push dword ptr ss:\[ebp\-8\]   
00EB3D8C    8B45 F4         mov eax, dword ptr ss:\[ebp\-C\]   
00EB3D8F    8338 00         cmp dword ptr ds:\[eax\], 0   
00EB3D92    74 02           je short 00EB3D96   
00EB3D94    FF30            push dword ptr ds:\[eax\]   
00EB3D96    FF75 F0         push dword ptr ss:\[ebp\-10\]   
00EB3D99    FF75 EC         push dword ptr ss:\[ebp\-14\]   
00EB3D9C    C3              retn    //在这里下断点，Shift+F9 运行到这里后取消断点。    
.....................................................................   
  
这里没有用内存镜像断点和模拟跟踪到达伪 Oep,因为前者跳过了 Stolen Code 的具体位置,后者我目前找不到 Stolen Code 的具体位置。   
  
下面有无数小跳转，小循环，可疑 Call，用 F8 极其容易跑飞，没特别说明，一律 F7 步过,F4 直接运行到指定位置。   
  
00EC6368    B7 C0           mov bh, 0C0   
00EC636A    B5 DF           mov ch, 0DF   
00EC636C    66:8BD9         mov bx, cx   
00EC636F    E8 07000000     call 00EC637B  //Btw，遇到 Call 较近距离的 Call 一般用 F7 步过，我的理解是 相距 10 个代码左右视为近距离 Call。    
  
00EC637B    81DE 74BBB366   sbb esi, 66B3BB74   
00EC6381    5A              pop edx   
00EC6382    66:81EB 3FA8    sub bx, 0A83F   
00EC6387    66:81D3 3760    adc bx, 6037   
00EC638C    81C2 0B160000   add edx, 160B   
00EC6392    BE D3C8BF65     mov esi, 65BFC8D3   
00EC6397    33FF            xor edi, edi   
00EC6399    66:B9 C526      mov cx, 26C5   
00EC639D    E9 10000000     jmp 00EC63B2   
  
00EC63B2    8BF3            mov esi, ebx   
00EC63B4    FF3417          push dword ptr ds:\[edi+edx\]   
00EC63B7    B9 E9199279     mov ecx, 799219E9   
00EC63BC    81DE 7A4B4D4B   sbb esi, 4B4D4B7A   
00EC63C2    E9 0D000000     jmp 00EC63D4   
  
00EC63D4    58              pop eax                                  ; Synchro.005508E6   
00EC63D5    51              push ecx   
00EC63D6    66:81CB CDA4    or bx, 0A4CD   
00EC63DB    68 9300546A     push 6A540093   
00EC63E0    0FB7CF          movzx ecx, di   
00EC63E3    68 DA0EF02D     push 2DF00EDA   
00EC63E8    5E              pop esi   
00EC63E9    66:81D9 94A4    sbb cx, 0A494   
00EC63EE    59              pop ecx   
00EC63EF    5B              pop ebx   
00EC63F0    66:8BCA         mov cx, dx   
00EC63F3    8AE8            mov ch, al   
00EC63F5    81F0 68E86160   xor eax, 6061E868   
00EC63FB    E8 07000000     call 00EC6407   
  
00EC6407    8AFD            mov bh, ch   
00EC6409    5B              pop ebx   
00EC640A    81F0 81D4EF06   xor eax, 6EFD481   
00EC6410    8AFC            mov bh, ah   
00EC6412    81C0 26F9A223   add eax, 23A2F926   
00EC6418    80C5 B6         add ch, 0B6   
00EC641B    8BD9            mov ebx, ecx   
00EC641D    BE AFC6AD5E     mov esi, 5EADC6AF   
00EC6422    89043A          mov dword ptr ds:\[edx+edi\], eax   
00EC6425    57              push edi   
00EC6426    51              push ecx   
00EC6427    68 FD0CF236     push 36F20CFD   
00EC642C    59              pop ecx   
00EC642D    0F82 00000000   jb 00EC6433   
00EC6433    66:81CB B5DE    or bx, 0DEB5   
00EC6438    5E              pop esi   
00EC6439    5B              pop ebx   
00EC643A    66:BE 6955      mov si, 5569   
00EC643E    53              push ebx   
00EC643F    66:BE ABDE      mov si, 0DEAB   
00EC6443    59              pop ecx   
00EC6444    83EF 03         sub edi, 3   
00EC6447    81E6 AA144070   and esi, 704014AA   
00EC644D    E8 0D000000     call 00EC645F   
  
00EC645F   /0F8B 08000000   jpo 00EC646D   
00EC6465   |0FBFDE          movsx ebx, si   
00EC6468   |66:81D1 AC23    adc cx, 23AC   
00EC646D   8AD8            mov bl, al   
00EC646F    81CB F3B3043D   or ebx, 3D04B3F3   
00EC6475    5E              pop esi   
00EC6476    4F              dec edi   
00EC6477    8BF7            mov esi, edi   
00EC6479    E8 0D000000     call 00EC648B   
  
00EC648B    0FB7CB          movzx ecx, bx   
00EC648E    5B              pop ebx   
00EC648F    E8 12000000     call 00EC64A6   
  
00EC64A6    BB 6C5C2657     mov ebx, 57265C6C   
00EC64AB    5E              pop esi   
00EC64AC    81FF 60EBFFFF   cmp edi, -14A0   
00EC64B2    0F85 1D000000   jnz 00EC64D5 //Btw:注意这里，这里即将跳到某处，慢慢又循环回这里，往下看。    
00EC64B8    8BF0            mov esi, eax   
00EC64BA    8AF9            mov bh, cl   
00EC64BC    E9 22000000     jmp 00EC64E3  //F4 直接到这里，我个人认为如果发现 Jnz 等跳转附近如果有 jmp xxxx 语句，直接 F4 到这里,很可能是循环出口。    
00EC64C1    9C              pushfd   
00EC64C2    A5              movs dword ptr es:\[edi\], dword ptr ds:\[e>   
00EC64C3    7A 2B           jpe short 00EC64F0   
.................................................................   
  
00EC64E3    5B              pop ebx     成功到达这里。   
00EC64E4    58              pop eax   
00EC64E5    05 7C40E3C3     add eax, C3E3407C   
00EC64EA    5C              pop esp   
00EC64EB    EB 3F           jmp short 00EC652C   
  
00EC652C    03C3            add eax, ebx                             ; Synchro.00400000   
00EC652E    BB B1000000     mov ebx, 0B1   
00EC6533    0BDB            or ebx, ebx   
00EC6535    75 02           jnz short 00EC6539   
  
00EC6539    E8 00000000     call 00EC653E   
  
00EC653E    5D              pop ebp   
00EC653F    81ED 23014B00   sub ebp, 4B0123   
00EC6545    8D85 D2004B00   lea eax, dword ptr ss:\[ebp+4B00D2\]   
00EC654B    8D8D 69014B00   lea ecx, dword ptr ss:\[ebp+4B0169\]   
00EC6551    03CB            add ecx, ebx   
00EC6553    8941 01         mov dword ptr ds:\[ecx+1\], eax   
00EC6556    8D85 11014B00   lea eax, dword ptr ss:\[ebp+4B0111\]   
00EC655C    8D8D D6004B00   lea ecx, dword ptr ss:\[ebp+4B00D6\]   
00EC6562    8901            mov dword ptr ds:\[ecx\], eax   
00EC6564    B8 58140000     mov eax, 1458   
00EC6569    8D8D DB004B00   lea ecx, dword ptr ss:\[ebp+4B00DB\]   
00EC656F    8901            mov dword ptr ds:\[ecx\], eax   
00EC6571    8D8D 69014B00   lea ecx, dword ptr ss:\[ebp+4B0169\]   
00EC6577    8D85 69134B00   lea eax, dword ptr ss:\[ebp+4B1369\]   
00EC657D    51              push ecx   
00EC657E    50              push eax   
00EC657F    E8 7BFFFFFF     call 00EC64FF  //这个 Call 往回 Call,而且距离超过 10 个,F8 放心步过。    
00EC6584    3E:EB 01        jmp short 00EC6588   
  
00EC6588    66:8105 9165EC0>add word ptr ds:\[EC6591\], 0CCFC   
00EC6591    F61E            neg byte ptr ds:\[esi\]  //F7 到这里发现语句变化，F8 没试过。    
  
00EC8871    83EC 1C         sub esp, 1C   
00EC8874    83C4 18         add esp, 18   
00EC8877    F2:             prefix repne:  //发现这样的语句实际是变形 jmp    
  
00EC887C    894C24 00       mov dword ptr ss:\[esp\], ecx   
00EC8880    66:812D 8988EC0>sub word ptr ds:\[EC8889\], 0EC49   
00EC8889    AE              scas byte ptr es:\[edi\]  //F7 到这里发现语句变化，F8 没试过。    
  
00EC8889   /65:EB 02        jmp short 00EC888E   
  
00EC888E    23C8            and ecx, eax   
00EC8890    64:EB 02        jmp short 00EC8895   
  
00EC8895    334C24 00       xor ecx, dword ptr ss:\[esp\]   
00EC8899    EB 02           jmp short 00EC889D   
  
00EC889D    F2:             prefix repne:   
00EC889E    EB 02           jmp short 00EC88A2   
  
00EC88A2    0FC9            bswap ecx   
00EC88A4    66:8135 AE88EC0>xor word ptr ds:\[EC88AE\], 223D   
00EC88AD    F2:             prefix repne:  //F7 到这里发现语句变化，F8 没试过。    
  
00EC88B1    B9 B288EC00     mov ecx, 0EC88B2   
00EC88B6    FF51 08         call dword ptr ds:\[ecx+8\]   
  
00EC88C0    59              pop ecx                                  ; 00EC88B9   
00EC88C1    26:EB 02        jmp short 00EC88C6   
  
00EC88C6    034C24 18       add ecx, dword ptr ss:\[esp+18\]   
00EC88CA    59              pop ecx   
00EC88CB    3E:EB 01        jmp short 00EC88CF   
  
00EC88CE    F2:             prefix repne:  //未知用途语句，有待研究，但发现这些语句,Stolen Code 就在附近。    
00EC88CF    55              push ebp  //这里是，往下看看，不难看出是 C++的入口特征码，就是 Stolen Code 的具体位置，看看被 Stolen Code 多少代码,整理结果在下面。    
00EC88D0    8BEC            mov ebp, esp   
00EC88D2    6A FF           push -1   
00EC88D4    68 A8384A00     push 4A38A8   
00EC88D9    68 6CC54600     push 46C56C   
00EC88DE    64:A1 00000000  mov eax, dword ptr fs:\[0\]   
00EC88E4    3E:EB 01        jmp short 00EC88E8  //这是干扰 Unpack 的指令，略过。    
00EC88E7    F2:             prefix repne:       //这是干扰 Unpack 的指令，略过。    
00EC88E8    50              push eax   
00EC88E9    64:8925 0000000>mov dword ptr fs:\[0\], esp   
00EC88F0    83EC 58         sub esp, 58   
00EC88F3    3E:EB 01        jmp short 00EC88F7   //这是干扰 Unpack 的指令，略过。    
00EC88F6    F2:             prefix repne:        //这是干扰 Unpack 的指令，略过。    
00EC88F7    53              push ebx   
00EC88F8    3E:EB 01        jmp short 00EC88FC   //这是干扰 Unpack 的指令，略过。    
00EC88FB    F2:             prefix repne:        //这是干扰 Unpack 的指令，略过。    
00EC88FC    56              push esi   
00EC88FD    3E:EB 01        jmp short 00EC8901   //这是干扰 Unpack 的指令，略过。    
00EC8900    F2:             prefix repne:        //这是干扰 Unpack 的指令，略过。    
00EC8901    57              push edi   
00EC8902    8965 E8         mov dword ptr ss:\[ebp\-18\], esp   
00EC8905    3E:EB 01        jmp short 00EC8909   //这是干扰 Unpack 的指令，略过。    
00EC8908    F2:             prefix repne:        //这是干扰 Unpack 的指令，略过。    
00EC8909    EB 01           jmp short 00EC890C   
00EC890B    9A 687D7746 006>call far 6800:46777D68   
00EC8912    C9              leave   
00EC8913    87EC            xchg esp, ebp   
00EC8915    00C3            add bl, al   
.......................................................................   
  
这是整理出的 Stolen Code 共 38 个字节。   
  
00EC88CF    55              push ebp   
00EC88D0    8BEC            mov ebp, esp   
00EC88D2    6A FF           push -1   
00EC88D4    68 A8384A00     push 4A38A8   
00EC88D9    68 6CC54600     push 46C56C   
00EC88DE    64:A1 00000000  mov eax, dword ptr fs:\[0\]   
00EC88E8    50              push eax   
00EC88E9    64:8925 0000000>mov dword ptr fs:\[0\], esp   
00EC88F0    83EC 58         sub esp, 58   
00EC88F7    53              push ebx   
00EC88FC    56              push esi   
00EC8901    57              push edi   
00EC8902    8965 E8         mov dword ptr ss:\[ebp\-18\], esp   
  
找到了 Stolen Code 字节，继续前进,全部用 F8 走。   
00EC890C    68 7D774600     push 46777D   
00EC8911    68 C987EC00     push 0EC87C9   
00EC8916    C3              retn   
  
00EC87C9    60              pushad   
00EC87CA    9C              pushfd   
00EC87CB    FC              cld   
00EC87CC    BF 0888EC00     mov edi, 0EC8808   
00EC87D1    B9 58140000     mov ecx, 1458   
00EC87D6    F3:AA           rep stos byte ptr es:\[edi\]  执行 Stolen Code   
00EC87D8    9D              popfd   
00EC87D9    61              popad   
00EC87DA    C3              retn   
  
00467756    C3              retn   
00467757    0000            add byte ptr ds:\[eax\], al  //这里是真的 Oep,Asprotect 一共抽掉 38 个字节,这里有个问题是 Asprotect 有时抽 11 个字节,有时抽 38 个字节,有什么规律，应该是从 push ebp 到第一个 Call 指令中间的代码，有多少抽多少。     
00467759    0000            add byte ptr ds:\[eax\], al   //好啦，直接双击这句补上刚才整理的 Stolen Code,Od 必须去掉用 Nop 填充选项。    
0046775B    0000            add byte ptr ds:\[eax\], al   
0046775D    0000            add byte ptr ds:\[eax\], al   
0046775F    0000            add byte ptr ds:\[eax\], al   
00467761    0000            add byte ptr ds:\[eax\], al   
00467763    0000            add byte ptr ds:\[eax\], al   
00467765    0000            add byte ptr ds:\[eax\], al   
00467767    0000            add byte ptr ds:\[eax\], al   
00467769    0000            add byte ptr ds:\[eax\], al   
0046776B    0000            add byte ptr ds:\[eax\], al   
0046776D    0000            add byte ptr ds:\[eax\], al   
0046776F    0000            add byte ptr ds:\[eax\], al   
00467771    0000            add byte ptr ds:\[eax\], al   
00467773    0000            add byte ptr ds:\[eax\], al   
00467775    0000            add byte ptr ds:\[eax\], al   
00467777    0000            add byte ptr ds:\[eax\], al   
00467779    0000            add byte ptr ds:\[eax\], al   
0046777B    0000            add byte ptr ds:\[eax\], al   
0046777D    FF15 94934900   call dword ptr ds:\[499394\]  //伪 Oep 地址。    
00467783    33D2            xor edx, edx   
00467785    8AD4            mov dl, ah   
00467787    8915 04154C00   mov dword ptr ds:\[4C1504\], edx   
0046778D    8BC8            mov ecx, eax   
0046778F    81E1 FF000000   and ecx, 0FF   
00467795    890D 00154C00   mov dword ptr ds:\[4C1500\], ecx   
0046779B    C1E1 08         shl ecx, 8   
0046779E    03CA            add ecx, edx   
.......................................................................   
  
补好代码结果。   
  
00467757    55              push ebp      
00467758    8BEC            mov ebp, esp   
0046775A    6A FF           push -1   
0046775C    68 A8384A00     push Synchro.004A38A8   
00467761    68 6CC54600     push Synchro.0046C56C   
00467766    64:A1 00000000  mov eax, dword ptr fs:\[0\]   
0046776C    50              push eax   
0046776D    64:8925 0000000>mov dword ptr fs:\[0\], esp   
00467774    83EC 58         sub esp, 58   
00467777    53              push ebx   
00467778    56              push esi   
00467779    57              push edi   
0046777A    8965 E8         mov dword ptr ss:\[ebp\-18\], esp   
0046777D    FF15 94934900   call dword ptr ds:\[499394\]   
00467783    33D2            xor edx, edx   
00467785    8AD4            mov dl, ah   
00467787    8915 04154C00   mov dword ptr ds:\[4C1504\], edx   
0046778D    8BC8            mov ecx, eax   
0046778F    81E1 FF000000   and ecx, 0FF   
00467795    890D 00154C00   mov dword ptr ds:\[4C1500\], ecx   
0046779B    C1E1 08         shl ecx, 8   
0046779E    03CA            add ecx, edx   
.......................................................................   
  
Od 插件修改入口为 67757 重建输入表选择 2 脱壳。   
  
然后用 ImportREC 用跟踪等级 1 和 Asprotect1.3X 插件轻松修复，正常运行。   
使用 Active Registry Monitor 备份注册表，修改系统时间，发现原程序早已不能运行，脱壳程序正常运行。还原系统时间，然后用 Active Registry Monitor 还原注册表。   
  
点 About 按钮，程序崩溃。   
  
好，继续努力。   
  
Od 载入脱壳修复的程序，直接运行,点关于按钮。   
  
  
  
004012DD   .  E8 37C10700   call Unpack\_.0047D419   
004012E2   .  8B3D 00714B00 mov edi, dword ptr ds:\[4B7100\]     
004012E8   .  83C9 FF       or ecx, FFFFFFFF   
004012EB   .  33C0          xor eax, eax   
004012ED   .  F2:AE         repne scas byte ptr es:\[edi\]  //程序在这里异常。    
004012EF   .  F7D1          not ecx   
004012F1   .  49            dec ecx   
004012F2   .  8D4409 02     lea eax, dword ptr ds:\[ecx+ecx+2\]   
004012F6   .  50            push eax   
004012F7   .  E8 83F80700   call Unpack\_.00480B7F   
  
看代码窗口下面的信息窗口。   
  
ecx\=FFFFFFFF   
al\=00   
es:\[edi\]=\[00EA3951\]=???  //edi 中的地址不存在，程序异常，我们看右边寄存 Edi 中的值 00EA3951，这个地址是没脱壳的程序中的某地址，而且是存储的是注册用户名字。    
  
解决方法很简单，我们在脱壳修复程序转存窗口尾部为 0 处，注意空间要够你填入个人信息,随意找，我选择的是 4C3FE8 处，右键编辑转存-二进制,去掉保留大小选项，选 ASCII 方式填入个人破解信息,如 Crack By weiyi75\[Dfcg\]。选定刚才修改的代码，右键复制到可执行文件－然后保存文件。   
  
然后让错误的指针指向我们指定的位置,继续分析原程序。   
  
  
004012DD   .  E8 37C10700   call Unpack\_.0047D419   
004012E2   .  8B3D 00714B00 mov edi, dword ptr ds:\[4B7100\]  //这里将壳中地址给 edi,点一下这句，看信息窗口提示。    
004012E8   .  83C9 FF       or ecx, FFFFFFFF   
004012EB   .  33C0          xor eax, eax   
004012ED   .  F2:AE         repne scas byte ptr es:\[edi\]  //程序在这里异常。    
004012EF   .  F7D1          not ecx   
004012F1   .  49            dec ecx   
004012F2   .  8D4409 02     lea eax, dword ptr ds:\[ecx+ecx+2\]   
004012F6   .  50            push eax   
004012F7   .  E8 83F80700   call Unpack\_.00480B7F   
  
信息窗口提示。   
  
ds:\[004B7100\]=00EA3951  //这个是壳中地址。   
  
我们在转存窗口，Ctrl+G，填入 4b7100   
  
004B7100  51 39 EA 00        
  
点右键－二进制编辑,HEX+00   
  
004B7100  E8 3F 4C 00     
  
复制到程序，保存文件，破解成功。   
  
  
  
谨以本文献给困惑中的脱友，如果对你有所帮助是我的心愿。   
  
    
\--------------------------------------------------------------------------------    
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!   
 4dd466d0e1e0cc123afbe504da2462ca