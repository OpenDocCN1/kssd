

# AsProtect Loader v0.c 壳盲脱壳【分享】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-571.htm`](https://www.pediy.com/kssd/pediy07/pediy7-571.htm)

【Target】: AsLoad (c) 2003 by GlObAl & \[NtSC\]  
  
简介：作用解决试用日期限制注册，对 ASProtect v1.2 试用日期限制的有效，readme 中说可以 high 点，我没有试  
  
【小注】：想了解一下 Loader，找到了这个；这个壳我未玩过，不等于这个壳很先进，但它的处理方式有些哈哈！这个可能是私人壳或者是一些壳的早期版本。。。有人知道可以通知我。。。呵呵  
  
PEid 没检到什么，它的输入表 就只有那么一个 API GetModuleHandleA，一定有动作的。。。  
开头你不能用 Esp 定律  
开头你不能用硬件断点，被它占用作小解码，占用很长的一段非法指令时间，全跟要吐血的...  
  
开头用 int3 断点要小心噢，在 se 入口处下断要当心；  
API 断点要小心下到准确无误的地址。  
  
bp GetProcAddress  
bp GetModuleHandleA  
bp LoadLibraryA  
  
这样你会死得很惨，它并不检你是否有 bp api，由于它做惯“小偷”，你不能这样去下断，否则自食其果！  
bp api + some offset 就可以了，offset 要定位准确噢!  
  
用 OD 调试，你无需隐藏调试器标志，它不检这个！  
  
废话连篇，动手呀。。。  
我用的是自己的 DIY 版 OD，在 XP 下调试。。。  
OD 设置忽略所有异常，全部打勾。。。  
bp LoadLibraryA +9  
  
  
00401A82   pushad   // 停在入口  
00401A83   call 00401A88                    ; AsLoad.00401A88  
00401A88   pop ebp  
00401A89   sub ebp,4078BB  
00401A8F   pushad  
00401A90   xor eax,eax  
00401A92   call 00401B8B                     ; AsLoad.00401B8B  
  
Shift + F9  
中断后堆栈报告如下：  
0012FF6C    00404BEC   ASLOAD.00404BEC  
0012FF70    00401FA9   ASLOAD.00401FA9  
0012FF74    003705B4   返回到 003705B4               // Ctrl + G 此地址下断，Shift + F9  
0012FF78    003705A3   返回到 003705A3 来自 003705AE  
  
// 下面组织得比较乱。。。  
// call 370020 是解密数据  
// 特殊的 api 被壳 Hook 和 Stolen 了，并加入了花花，而且很铺张地每个用上 64 K 内存 -> 即 10000h  
// 壳会动态解码后运行完又加密，小花样，加密解密都用同一个 call 00371147  
  
00370556   push 4  
00370558   push 1000  
0037055D   mov eax,dword ptr ss:\[ebp+409533\]  
00370563   add eax,10000                        ; UNICODE "=::=::\\"  
00370568   push eax  
00370569   push 0  
0037056B   call dword ptr ss:\[ebp+4096AF\]     //\[ebp+4096AF\] = 390000  实际是 kernel32.VirtualAlloc 最后 jmp 77E5981B  
  
00370585   call 00371147                     // 公用的 加密/解码 Call -> 不可轻易 F8  
0037058A   popad  
0037058B   pushad  
0037058C   lea eax,dword ptr ss:\[ebp+408378\]  
00370592   mov ecx,47  
00370597   call 00371147  
0037059C   popad  
0037059D   pushad  
0037059E   call 003705AE  
  
003705AE   call dword ptr ss:\[ebp+409553\]  //\[ebp+409553\] = 3B0000 实际是 LoadLibraryA  
003705B4   mov dword ptr ss:\[ebp+409547\],eax  
003705BA   call 003705CB  
003705BF   dec ebp  
003705C0   jnb short 00370636  
  
003705CB  push eax                          // 模块入口  
003705CC  call dword ptr ss:\[ebp+40954F\]    // \[ebp+40954F\] = 003D0000 实际是 GetProcAddress  
003705D2  pushad  
003705D3  lea eax,dword ptr ss:\[ebp+408378\]  
003705D9  mov ecx,47  
003705DE  call 00371147  //加密壳代码  
  
003705E4  cmp dword ptr ss:\[ebp+4097E7\],1  // ??  
003705EB  jnz 00370721  
003705F1  mov dword ptr ss:\[ebp+40954B\],eax  
003705F7  mov eax,dword ptr ss:\[ebp+40954B\]  
003705FD  lea edi,dword ptr ss:\[ebp+40954B\]  
00370603  call 003711CB  
  
// 块解码好像会比较块数据开头是否为 "JC"  
00370784   >popad  
00370785   >xchg eax,ecx  
00370786   >cmp dword ptr ss:\[ebp+4096A7\],0  
0037078D   >je short 003707A1  
0037078F   >mov edi,dword ptr ss:\[ebp+4096A7\]  
00370795   >add edi,dword ptr ss:\[ebp+4097DF\]  
0037079B   >mov ecx,dword ptr ss:\[ebp+4096AB\]  
003707A1   >rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\] 迁回解码后的 Code 节 401000  
003707A3   >pushad  
003707A4   >mov ecx,30  
003707A9   >lea eax,dword ptr ss:\[ebp+408560\]  
003707AF   >call 00371147  
  
003708B1  popad  
003708B2  push 4  
003708B4  push 1000  
003708B9  push 10000                                        ; UNICODE "=::=::\\"  
003708BE  push 0  
003708C0  call dword ptr ss:\[ebp+4096AF\] // 申请空间  
003708C6  xchg eax,edi  
003708C7  call 00371110  
003708CC  pushad  
003708CD  xchg eax,esi  
003708CE  mov ecx,11380  
003708D3  mov eax,F6305196  
003708D8  xor byte ptr ds:\[esi+ecx\],al  
003708DB  rol eax,cl  
003708DD  xor eax,ecx  
003708DF  loopd short 003708D8  
003708E1  popad  
003708E2  add eax,800  
003708E7  push edi  
003708E8  push edi  
003708E9  push eax  
003708EA  call 00370020 // 过了这个 Call 可以看到 edi 地址是 API 信息  
003708EF  pushad  
003708F0  mov ecx,4F  
003708F5  lea eax,dword ptr ss:\[ebp+40868D\]   
003708FB  call 00371147  //加密  
  
00370949  test eax,80000000  
0037094E  jnz short 00370986  
00370950  push edi  
00370951  push edi  
00370952  push dword ptr ss:\[ebp+409517\]  
00370958  call dword ptr ss:\[ebp+40954F\]  
0037095E  mov dword ptr ds:\[ebx\],eax  
00370960  pushad  
00370961  cmp dword ptr ss:\[ebp+40969F\],1 // 是否需要加密 API ？当然不加密，\[ebp+40969F\] -> set zero ★★★  
00370968  jnz short 00370971  
0037096A  xchg edi,ebx  
0037096C  call 00370B9A  
00370971  popad  
  
  
00370D62   pop dword ptr fs:\[0\]  // se 出口，下面是 Iat 处理  
00370D68   add esp,4  
00370D6B   pushad  
  
00370D90   lods dword ptr ds:\[esi\]          // 不用加密的处理？  
00370D91   and dword ptr ds:\[esi-4\],0  
00370D95   or eax,eax  
00370D97   je short 00370DB7  
00370D99   xchg eax,ebx  
00370D9A   add ebx,dword ptr ss:\[ebp+4097DF\]  
00370DA0   lods dword ptr ds:\[esi\]  
00370DA1   and dword ptr ds:\[esi-4\],0  
00370DA5   add eax,dword ptr ss:\[ebp+4097DF\]  
00370DAB   add eax,dword ptr ss:\[ebp+4096D3\]  
00370DB1   mov ebx,dword ptr ds:\[ebx\]  
00370DB3   mov dword ptr ds:\[eax\],ebx  
00370DB5   jmp short 00370D90  
  
  
00370DDD   lods dword ptr ds:\[esi\]         //  FF25 Jmp \[\] 加密的处理，这里壳处理成 Jmp api 地址偏移量，即：E9 + offset 的 5 字节跳转形式  
00370DDE   and dword ptr ds:\[esi-4\],0  
00370DE2   or eax,eax  
00370DE4   je short 00370E09  
00370DE6   xchg eax,ebx  
00370DE7   add ebx,dword ptr ss:\[ebp+4097DF\]  
00370DED   lods dword ptr ds:\[esi\]  
00370DEE   and dword ptr ds:\[esi-4\],0  
00370DF2   add eax,dword ptr ss:\[ebp+4097DF\]  
00370DF8   add eax,dword ptr ss:\[ebp+4096D3\]  
00370DFE   mov ebx,dword ptr ds:\[ebx\]  
00370E00   sub ebx,eax  
00370E02   sub ebx,4  
00370E05   mov dword ptr ds:\[eax\],ebx  
00370E07   jmp short 00370DDD  
  
  
00370E2F   lods dword ptr ds:\[esi\]         //  FF15  Call \[\] 加密的处理 ? 我这里没有这情况。。。  
00370E30   and dword ptr ds:\[esi-4\],0  
00370E34   or eax,eax  
00370E36   je short 00370E63  
00370E38   xchg eax,ebx  
00370E39   add ebx,dword ptr ss:\[ebp+4097DF\]  
00370E3F   lods dword ptr ds:\[esi\]  
00370E40   and dword ptr ds:\[esi-4\],0  
00370E44   add eax,dword ptr ss:\[ebp+4097DF\]  
00370E4A   add eax,dword ptr ss:\[ebp+4096D3\]  
00370E50   mov byte ptr ds:\[eax-1\],0E8  
00370E54   mov ebx,dword ptr ds:\[ebx\]  
00370E56   sub ebx,eax  
00370E58   sub ebx,4  
00370E5B   mov dword ptr ds:\[eax\],ebx  
00370E5D   mov byte ptr ds:\[eax+4\],90  
00370E61   jmp short 00370E2F  
  
  
// 可以改良一下，就能还原正常可识别的 Jmp \[\] Iat ★★★  
00370DEE   add eax,dword ptr ss:\[ebp+4097DF\]    
00370DF4   add eax,dword ptr ss:\[ebp+4096D3\]  
00370DFA   inc eax  
00370DFB   mov dword ptr ds:\[eax\],ebx  
00370DFD   dec eax  
00370DFE   dec eax  
00370DFF   mov word ptr ds:\[eax\],25FF  
00370E04   nop  
00370E05   nop  
00370E06   nop  
00370E07   jmp short 00370DDD  
  
03 85 DF 97 40 00 03 85 D3 96 40 00 40 89 18 48 48 66 C7 00 FF 25 90 90 90  
  
// StolenCode 形式为 5 字节长度，呵呵比较简单，可以完全还原。。。  
// 下面过程是壳查表修复，因为壳是动态申请 Stolen 地址的。。。所以它要做这个工作  
00370E88   lods dword ptr ds:\[esi\]           // 处理 StolenCode  
00370E89   or eax,eax  
00370E8B   je short 00370ED3  
00370E8D   mov ebx,eax  
00370E8F   lods dword ptr ds:\[esi\]  
00370E90   add eax,ebx  
00370E92   add eax,dword ptr ss:\[ebp+4097DF\]  
00370E98   add eax,dword ptr ss:\[ebp+4096D3\]  
00370E9E   add eax,5  
00370EA1   cmp dword ptr ds:\[esi+4\],1      // 是否需要修正？  
00370EA5   jnz short 00370EC5  
00370EA7   cmp dword ptr ds:\[eax\],5355434B // 是否吻合要处理偏移位置的特征码？  
00370EAD   je short 00370EB2  
00370EAF   inc eax  
00370EB0   jmp short 00370EA7  
00370EB2   add esi,8  
00370EB5   push esi  
00370EB6   push eax  
00370EB7   dec eax  
00370EB8   sub eax,esi  
00370EBA   neg eax  
00370EBC   mov esi,eax  
00370EBE   pop eax  
00370EBF   sub esi,5  
00370EC2   mov dword ptr ds:\[eax\],esi  
00370EC4   pop esi  
00370EC5   inc esi  
00370EC6   cmp dword ptr ds:\[esi\],58585858 // 这里全是这种模式，是 StolenCode 标志？  
00370ECC   jnz short 00370EC5  
00370ECE   add esi,4  
00370ED1   jmp short 00370E88  
  
  
57 48 8B F8 33 C9 B1 05 F3 A4 5F 90 90 90  
// 顺便补丁还原代码：  
00370EB5   push esi  
00370EB6   push edi                       // ★★★  
00370EB7   dec eax  
00370EB8   mov edi,eax  
00370EBA   xor ecx,ecx  
00370EBC   mov cl,5  
00370EBE   rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
00370EC0   pop edi  
00370EC1   nop  
00370EC2   nop  
00370EC3   nop                            // ★★★  
00370EC4   pop esi  
00370EC5   inc esi  
  
  
003710D5  stos dword ptr es:\[edi\]  
003710D6  popad  
003710D7  mov dword ptr ss:\[esp+1C\],edi  
003710DB  popad  
003710DC  jmp eax  // 前往 OEP 处理的花样 00AA0000  
003710DE  popad  
003710DF  retn  
  
  
// 里面有很多花花，可以用 Esp 定律找 Stolen Code，也可以根据堆栈 “我猜我猜我猜猜猜”  
00AA0000   push 401CDD //★★★  
00AA0005   neg esi  
00AA0008   cmp edi,edi  
00AA000A   shld edi,eax,cl  
  
00AA08E9   push dword ptr fs:\[0\]  //★★★  
00AA08EF   mov esi,AB3919AB  
00AA08F5   inc esi  
00AA08F7   ror esi,3E  
  
00AA11E6   mov dword ptr fs:\[0\],esp  //★★★  
00AA11EC   not esi  
00AA11EE   test esi,8BBE848D  
00AA11F4   imul edi,esi,1DC84EAC  
  
00AA1AEB   mov ecx,edi  
00AA1AED   and edi,FFFF0000  
00AA1AF3   sub ecx,edi  
00AA1AF5   rep stos byte ptr es:\[edi\] // 移“花”植“草”  
00AA1AF7   popad  
00AA1AF8   popfd  
00AA1AF9   push 401011               // 压入伪 OEP  
00AA1AFE   retn  
  
// 当 eip 等于 401011 时堆栈的情况，明显是安装了 seh 401CDD  
  
0012FFBC    0012FFE0   指针到下一个 SEH 记录  
0012FFC0    00401CDD   SE 句柄  
0012FFC4    77E5EB69   返回到 kernel32.77E5EB69  
  
  
// OEP 定为 401000 吧，上面找到的长度要大，套不了，作如下修复刚好用完 00 字节  
00401000    >push 401CDD    
00401005    >xor eax,eax  
00401007    >push dword ptr fs:\[eax\]  
0040100A    >mov dword ptr fs:\[0\],esp  
  
哈哈，这次脱壳后又使我再次从壳盲的人群中挤了出来！  
另外，我很想知道很多早期的壳在 98 下检测出 OD 调试器来，是怎么做的？  
00401C05   and eax,FFFF0000  
00401C0A   mov dword ptr ss:\[ebp+4097DF\],eax  
00401C10   mov eax,dword ptr ss:\[ebp+407889\]  
00401C16   and eax,FFFF0000  
00401C1B   cmp word ptr ds:\[eax\],5A4D         //到达这里的指令， 很明显是只适用 NT 的获取模块基址的方式，好像不适用 98  
00401C20   je short 00401C29                                  ; ASLOAD.00401C29  
00401C22   sub eax,10000  
00401C27   jmp short 00401C1B                                 ; ASLOAD.00401C1B  
00401C29   mov dword ptr ss:\[ebp+409543\],eax  
00401C2F   mov dword ptr ss:\[ebp+40952B\],eax  
  
Written by askformore  
 06.01.2005  
  
Andy Law Advertising: 想要...就系你既...![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

 4dd466d0e1e0cc123afbe504da2462ca