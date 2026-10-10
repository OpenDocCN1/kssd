

# AlfaClock 1.60a

> 原文：[`www.pediy.com/kssd/pediy06/pediy6775.htm`](https://www.pediy.com/kssd/pediy06/pediy6775.htm)

目标程序 AlfaClock 1.60a（华军下载）  
工具 OllyDbg 1.09d  
     LordPE  
     ImportRec 1.60  
一、脱壳：  
用 OD 加载后去掉调试器标志（IsDebuggerPresent），F9 执行，异常后按 21 次 SHIFT+F9，  
将 Hex Dump 窗口转到 4D6420，会看到如下数据，这里从 4D6420 到 4D6453 是 Aspr 要修改的  
地方，主要是：  
      1.4D6420  字符串指针，若这个指针指向空串，表示未注册  
      2.4D6428  字符串指针，这个指针不能指向空串，Aspr 让它指向 7Rk1OABg9ag=  
      3.4D6430－4D643B 是三个函数的入口地址，Aspr 会修改这三个指针，使其指向  
        壳中的三个函数！  
      4.4D6440  Share 版的使用天数，4D6444  剩下的天数  
      5.从 4D6448－4D6453 是程序验证壳存在的标记  
004D6420  00 00 00 00 00 00 00 00 00 00 00 00 00 8D 40 00  .............岪.  
004D6430  70 1B 4D 00 9C 1B 4D 00 9C 1B 4D 00 00 00 00 00  pM.?M.?M.....  
004D6440  FE FF FF FF FE FF FF FF 00 00 00 00 FE FF FF FF  ?��?��....?��  
004D6450  FE FF FF FF   
（从上面的叙述你应该知道修复程序时你要干什么了吧）go on  
再按 5 次 SHIFT+F9，程序停在这里：  
00FD3D03  XOR DWORD PTR DS:\[EAX\],EAX  
00FD3D05  POP DWORD PTR FS:\[0\]  
00FD3D0C  POP EAX  
00FD3D0D  CMP DWORD PTR DS:\[FD7EBC\],0  
00FD3D14  JE SHORT 00FD3D2A  
00FD3D16  PUSH 0C  
00FD3D18  MOV ECX,0FD7EBC  
00FD3D1D  LEA EAX,DWORD PTR SS:\[EBP-8\]  
00FD3D20  MOV EDX,4  
00FD3D25  CALL 00FD1010  
00FD3D2A  PUSH DWORD PTR SS:\[EBP-4\]  
00FD3D2D  PUSH DWORD PTR SS:\[EBP-8\]  
00FD3D30  MOV EAX,DWORD PTR SS:\[EBP-C\]  
00FD3D33  CMP DWORD PTR DS:\[EAX\],0  
00FD3D36  JE SHORT 00FD3D3A  
00FD3D38  PUSH DWORD PTR DS:\[EAX\]  
00FD3D3A  PUSH DWORD PTR SS:\[EBP-10\]  
00FD3D3D  PUSH DWORD PTR SS:\[EBP-14\]  
00FD3D40  RETN  
我们马上要进入原程序了，再看看 Hex Dump 窗口：  
004D6420  51 39 FC 00 00 00 00 00 30 A3 FE 00 00 8D 40 00  Q9?....0￣..岪.  
004D6430  A4 14 FD 00 40 14 FD 00 74 14 FD 00 FE FF FF FF  ??@?t??��  
004D6440  1E 00 00 00 0D 00 00 00 FE FF FF FF 00 00 00 00  .......?��....  
004D6450  00 00 00 00   
看到修改了？这时，可以 DUMP 程序了，用 LordPE，操作就不讲了吧。  
  
二、修复 IAT：  
在上面的 RETN 处设断，SHIFT+F9，断点处停下后 F2 清除断点再 F8，来到：  
00FE7CF0  PUSH EAX       ；停在这里  
00FE7CF1  POP ESI  
00FE7CF2  CALL 00FE7D01  
00FE7CF7  PUSH 5B  
00FE7CF9  CLC  
00FE7CFA  SAL DWORD PTR DS:\[ESI\],1  
  
稍往下滚动鼠标，看到  
00FE7D5F  MOV EDI,ECX  
00FE7D61  MOV DWORD PTR DS:\[EAX\],EBX  
00FE7D63  MOV DX,0DC32  
00FE7D67  SUB EAX,4  
00FE7D6A  MOV DX,CX  
00FE7D6D  SUB ECX,1  
00FE7D73  JNZ 00FE7D90  
00FE7D79  JPE 00FE7D84        <-光标移到这里按 F4  
00FE7D7F  OR DX,0DEAD  
00FE7D84  JMP 00FE7DB6  
  
光标移到 FE7D79（JPE 00FE7D84）再 F4，下面，Aspr 下面要执行是  
1.执行 Stolen code  
2.销毁来到这里的程序段  
有兴趣的可以跟，我选择的是直接进入原程序。于是再稍往下滚动鼠标，看到  
  
00FE7DC0  PUSHAD  
00FE7DC1  PUSHFD  
00FE7DC2  CLD  
00FE7DC3  MOV EDI,0  
00FE7DC8  MOV ECX,0  
00FE7DCD  REP STOS BYTE PTR ES:\[EDI\]  
00FE7DCF  POPFD  
00FE7DD0  POPAD  
00FE7DD1  RETN  
  
光标移到 RETN 处 F4，再 F8 就进入原程序了：  
  
00406324  JMP DWORD PTR DS:\[4D9280\]       <-停在这里  
0040632A  MOV EAX,EAX  
0040632C  JMP DWORD PTR DS:\[4D927C\]  
00406332  MOV EAX,EAX  
00406334  JMP DWORD PTR DS:\[4D9278\]  
  
显然，我们处在程序执行了一段代码的位置，但这并不妨碍修复 IAT，看看堆栈：  
  
0012FF6C  004063F9  RETURN to AlfaCloc.004063F9 from AlfaCloc.00406324  
0012FF70  00000000  
0012FF74  0000071D  
0012FF78  004D2FD5  RETURN to AlfaCloc.004D2FD5 from AlfaCloc.004063E8  
0012FF7C  005885C3  AlfaCloc.005885C3  
0012FF80  0000071D  
0012FF84  00000000  
0012FF88  00000000  
0012FF8C  00000000  
0012FF90  0000071D  
0012FF94  00B37CEE  
0012FF98  00FE7E99  
0012FF9C  00B37CEE  
0012FFA0  00B37CEE  
0012FFA4  00000000  
0012FFA8  00000000  
0012FFAC  0012FFF0  
0012FFB0  0012FFC4  
0012FFB4  7FFDF000  
0012FFB8  FFFFFFFF  
0012FFBC  00010101  
0012FFC0  00000000  
0012FFC4  77E67903  RETURN to kernel32.77E67903  
容易找到被删除了 Stolen code 的 oep，就是 4D2FD5 上一条语句的位置，鼠标选中  
堆栈区 0012FF78 处的 004D2FD5，回车后再向上滚动鼠标，看到  
  
004D2FCA  ADD BYTE PTR DS:\[EAX\],AL  
004D2FCC  ADD BYTE PTR DS:\[EAX\],AL  
004D2FCE  ADD BYTE PTR DS:\[EAX\],AL  
004D2FD0  CALL AlfaCloc.004063E8   
004D2FD5  MOV EBX,DWORD PTR DS:\[4D69C0\]              
004D2FDB  XOR EAX,EAX  
004D2FDD  PUSH EBP  
004D2FDE  PUSH AlfaCloc.004D3248  
  
好，就把 D2FD0 作为 oep 吧。启动 ImportREC，选中目标，就绪后在 OEP 处填上 D2FD0，  
点击 IAT AutoSearch 得到 RVA: 000D9274  Size: 0000000C，从 Size 值知道这里有误，  
把 Hex Dump 区转到 4D9000 往下翻页，看到  
  
004D9190  C7 E6 60 A5 5E 88 19 A0 D5 48 A7 3D BD A9 03 F3  擎\`?\_ 誋?僵   
004D91A0  78 92 22 D4 70 05 FE 00 A0 05 FE 00 AC 05 FE 00  x?詐 ?????  
004D91B0  5C 92 FE 00 B8 05 FE 00 D0 05 FE 00 EC 05 FE 00  \\揀.??????  
004D91C0  1C 06 FE 00 4C 06 FE 00 58 06 FE 00 28 15 FD 00    ?L ?X ?( ?  
004D91D0  68 06 FE 00 78 06 FE 00 84 06 FE 00 90 06 FE 00  h ?x ?????  
004D91E0  A8 06 FE 00 B8 06 FE 00 C8 06 FE 00 F4 06 FE 00  ????????  
004D91F0  24 07 FE 00 34 07 FE 00 44 07 FE 00 AC 10 FD 00  $?4?D???  
004D9200  00 15 FD 00 7C 07 FE 00 94 07 FE 00 74 15 FD 00  . ?|???t ?  
004D9210  A4 07 FE 00 B0 07 FE 00 C0 07 FE 00 F0 07 FE 00  ????????  
004D9220  1C 08 FE 00 4C 08 FE 00 70 08 FE 00 A0 08 FE 00    ?L ?p ???  
004D9230  CC 08 FE 00 DC 08 FE 00 EC 08 FE 00 C6 8B D1 6F  ??????茓裲  
004D9240  7E FB DF 77 AF E3 DF 77 68 3D E2 77 04 CF DF 77  ~w遷 h=鈝 线 w  
004D9250  38 5B C5 63 8E 85 D9 77 AC 82 D9 77 4D 7D D9 77  8\[與巺賥瑐賥 M}賥  
004D9260  02 5F 8A FD 33 15 99 77 A6 3F 99 77 59 14 99 77   \_ 婟 3 檞?檞 Y 檞  
004D9270  36 D7 A2 D9 00 09 FE 00 18 09 FE 00 2C 09 FE 00  6 注?.? .?,.?  
004D9280  00 15 FD 00 26 A3 03 CD 54 94 D9 77 8E 85 D9 77  . ?&?蚑斮 w 巺賥  
  
可以看出 D91A4 是 RVA 值，Size 值？再往下翻页直到  
  
004D9830  BA 22 99 77 94 23 99 77 39 99 00 34 B0 30 A3 77  ?檞?檞 9?4?  
004D9840  E0 4F A3 77 84 69 A3 77 1A 69 A3 77 D5 BF FC 06  郞刬 i湛?  
004D9850  54 10 FE 00 64 10 FE 00 70 10 FE 00 88 10 FE 00  T ?d ?p ???  
004D9860  A0 10 FE 00 AC 10 FE 00 BC 10 FE 00 CC 10 FE 00  ????????  
004D9870  E4 10 FE 00 F0 10 FE 00 00 11 FE 00 0C 11 FE 00  ????. ?. ?  
004D9880  1C 11 FE 00 28 11 FE 00 38 11 FE 00 48 11 FE 00    ?( ?8 ?H ?  
004D9890  58 11 FE 00 64 11 FE 00 70 11 FE 00 80 11 FE 00  X ?d ?p ?€ ?  
004D98A0  90 11 FE 00 9C 11 FE 00 A8 11 FE 00 C8 11 FE 00  ????????  
004D98B0  9C AE 54 F4 6C 5A 7C 77 5C 63 7C 77 81 59 7C 77  湲 T 鬺 Z|w\\c|w 乊|w  
004D98C0  CC 45 7C 77 02 61 67 9F 72 C3 5F 77 4B 71 19 5F  蘀|w ag 焤胈 wKq \_  
004D98D0  45 6D B0 76 60 EF AF 76 CD A5 AF 76 27 BC D1 4E  Em 皏\`锆 v 庭痸'佳 N  
004D98E0  A6 AF 4A 77 E8 12 4A 77 78 7D 79 13 E9 34 53 77  ΟJw?Jwx}y ?Sw  
004D98F0  B6 C6 36 8E 69 27 FB 74 6A 4A FB 74 24 58 FB 74  镀 6 巌'鹴 jJ 鹴$X 鹴  
004D9900  71 66 11 68 09 EB FB 3C 09 75 91 84 E5 61 CF D6  qf h.臌<.u 憚錫现  
  
显然 D9900 是中止值，于是 Size：900-1A4=75C  
把 D91A4 和 75C 分别填好后，就可以 Get Imports 了。对于 unresolved pointers，  
   1.Show Invalid 后 Trace Level1(Disasm)  
   2.Show Invalid 后 Plugin Tracers -> Asprotect 1.22  
以上两步之后就只有 rva: 000D93AC  ptr: 000FD158C 是真的 unresolved pointer 了。  
选中本行后选择 Disassemble/HexView，看到  
00FD158C   PUSH EBP  
00FD158D   MOV EBP,ESP  
00FD158F   MOV EAX,DWORD PTR DS:\[FD7CF0\]   // DWORD value : 00133938  
00FD1595   POP EBP  
00FD1596   RETN 4  
这是 Aspr 制造的一个的 unresolved 的"API"，  
（如果 RETN 4 换成 RETN，就是 GetCommandLineA，嘿嘿）  
因此这个函数应该 resolved 成 LockResource，为什么？嘿嘿！  
好了，对于剩下的 unresolved pointers 就全是 Invalid 了，Show Invalid，Cut Thunks。  
  
  
三、修复  
这是最麻烦的一步。  
（一）修改 Aspr 修改过的 D6420 开始的几个指针，主要思想是：  
      1.还原原来的三个函数指针  
      2.让两个字符串指针指向程序内部某处，我选定的是 589800 和 5797F0，在这两个  
        位置放入字符串 Pediy001 和 7Rk1OABg9ag=。呵呵  
下面是我修改过的情况  
004D6420  00 98 58 00 00 00 00 00 F0 97 58 00 00 8D 40 00  .榅.....饤 X..岪.  
004D6430  70 1B 4D 00 9C 1B 4D 00 9C 1B 4D 00 FE FF FF FF  pM.?M.?M.?��  
004D6440  1E 00 00 00 10 00 00 00 FE FF FF FF 00 00 00 00  ......?��....  
004D6450  00 00 00 00                                      ....  
（二）仿真 stolen code  
      主要是根据 EIP 的位置（00406324  JMP DWORD PTR DS:\[4D9280\]），参照此时程序  
到要执行的“明码”、堆栈结构和寄存器的情况来仿真  
明码是：  
004063E8  PUSH EBX  
004063E9  MOV EBX,EAX  
004063EB  XOR EAX,EAX  
004063ED  MOV DWORD PTR DS:\[4D40A0\],EAX  
004063F2  PUSH 0  
004063F4  CALL AlfaCloc.00406324         <- 正常情况下程序应从这里到达 00406324  
004063F9  MOV DWORD PTR DS:\[4D7664\],EAX  
  
堆栈结构前面已列出，寄存器的情况这时是：  
EAX=0 ECX=FE787F EDX=FE7F98 EBX=4D2B48   
ESP=12FF6C EBP=12FFA0 ESI=5885C3 EDI=1  
  
因此，程序在调用函数时，这几个量是重要的：  
EAX=4D2B48、ESI=58853、EBP=12FFA0，来到这里，应有相似的堆栈结构，怎样仿真我想这  
是一件仁者见仁的事吧。我在程序中构造了一个函数来完成，函数后面给出。  
  
（三）去除暗桩：暗桩的查找太麻烦就免了，只说修改吧。使用 Hex 编辑器作下列修改：  
      1.004D1E05 处的 FF 33 和 004D1E0A 的 8F 03 全改成 90 90  
      2.004D1DD4 的 74 改为 EB  
      3.004D63E3 的 FC 2D 8C FD 改为 8D 8D 8D 8D  
      4.004D63F7 的 88 B9 18 89 改为 19 19 19 19  
      5.004D6413 的 63 94 F3 64 改为 F4 F4 F4 F4  
（四）申请局部堆，仿真 Aspr 的验证数据  
  以前对付 Aspr 加壳程序都没有这一步，包括 AlfaClock 的 1.60 版，现在 Aspr 玩的花样是越  
来越多了！  
  在修复的前两步完成后程序仍然不能正常执行，发现它要到 139XXX 去取数据，再次跟踪加  
壳的程序，发现了这些“无聊”的数据，它们唯一的作用就是让你脱壳后的程序不能执行。  
于是你也得在局部堆有相似的数据结构，哈，就在堆中申请一点内存来仿真吧！这些工作一  
并交给我构造的函数来完成。不过说清这个函数的事就算了，函数如下  
004D3330  PUSH EBP  
004D3331  MOV EBP,ESP  
004D3333  PUSHAD  
004D3334  PUSH 0FF8                                 /Size = FF8 (4088.)  
004D3339  PUSH 40                                   |Flags = LPTR  
004D333B  CALL            \\LocalAlloc  
004D3340  PUSH EAX  
004D3341  PUSH 664                                  /Size = 664 (1636.)  
004D3346  PUSH 0                                    |Flags = LMEM\_FIXED  
004D3348  CALL            \\LocalAlloc  
004D334D  POP EDX  
004D334E  MOV DWORD PTR DS:\[4D7620\],EDX  
004D3354  MOV DWORD PTR DS:\[4D75E0\],EAX  
004D3359  MOV DWORD PTR DS:\[EAX\],0  
004D335F  MOV ECX,60  
004D3364  MOV EDI,DWORD PTR DS:\[4D75E0\]  
004D336A  ADD EDI,14  
004D336D  MOV EBX,EDI  
004D336F  SUB EBX,10  
004D3372  MOV DWORD PTR DS:\[EDI\],EBX  
004D3374  ADD EDI,10  
004D3377  ADD EBX,10  
004D337A  LOOPD SHORT dumped\_.004D3372  
004D337C  ADD EBX,20  
004D337F  MOV DWORD PTR DS:\[EDI\],EBX  
004D3381  ADD EDI,10  
004D3384  ADD EBX,10  
004D3387  MOV DWORD PTR DS:\[EDI\],EBX  
004D3389  ADD EDI,10  
004D338C  SUB EBX,30  
004D338F  MOV DWORD PTR DS:\[EDI\],EBX  
004D3391  MOV EDI,DWORD PTR DS:\[4D75E0\]  
004D3397  ADD EDI,614  
004D339D  MOV DWORD PTR DS:\[4D75E4\],EDI  
004D33A3  XOR EAX,EAX  
004D33A5  MOV DWORD PTR DS:\[4D75C8\],EAX  
004D33AA  MOV DWORD PTR SS:\[EBP+10\],EAX  
004D33AD  MOV DWORD PTR SS:\[EBP+14\],EAX  
004D33B0  MOV DWORD PTR SS:\[EBP+18\],EAX  
004D33B3  POPAD  
004D33B4  MOV EBX,71D  
004D33B9  MOV DWORD PTR SS:\[EBP+C\],EBX  
004D33BC  MOV DWORD PTR SS:\[EBP+1C\],EBX  
004D33BF  POP EBP  
004D33C0  ADD EBP,30  
004D33C3  MOV DWORD PTR SS:\[EBP-44\],EBP  
004D33C6  SUB EBP,50  
004D33C9  RETN  
  
注：CALL  是 CALL 401388  
  
最后在 4D2FBA 键入下面代码，再用 LordPE 把 EntryPoint 的值改成 000D2FBA，Over!  
004D2FBA  PUSH EBP  
004D2FBB  MOV EBP,ESP  
004D2FBD  ADD ESP,-40  
004D2FC0  MOV EAX,dumped\_.004D2B48  
004D2FC5  MOV ESI,dumped\_.005885C3  
004D2FCA  PUSH ESI  
004D2FCB  CALL dumped\_.004D3330  
  
这是我在 001 的第一篇，难免错误，请各位高手指点