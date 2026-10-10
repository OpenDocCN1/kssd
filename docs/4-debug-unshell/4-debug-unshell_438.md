

# ASPR1.3 脱壳初步探讨

> 原文：[`www.pediy.com/kssd/pediy06/pediy6748.htm`](https://www.pediy.com/kssd/pediy06/pediy6748.htm)

这里就以 Win2K,Ollydbg 寻找 System cleaner(4.90 build 122) OEP 为例：  
  
新版的 ASPR 最早让我发现异样的就是用 IsDebuggerPresent 来检查是否在被调试，Ollydbg 里下断点 IsDebuggerPresent..经过一系列 Shift+F9 后断点来到 IsDebuggerPresent 的函数里:  
  
77E6F4E9 > 64:A1 18000000   MOV EAX,DWORD PTR FS:\[18\]  
77E6F4EF   8B40 30          MOV EAX,DWORD PTR DS:\[EAX+30\]  
77E6F4F2   0FB640 02        MOVZX EAX,BYTE PTR DS:\[EAX+2\]  
77E6F4F6   C3               RETN  
  
此函数利用 eax 返回值，1 被调试，０否．．运行到 MOVZX EAX,BYTE PTR DS:\[EAX+2\]，看一下 DS:\[EAX+2\]所指内存区域，把内存中的数值改为 0(必须)，同时 eax 也返回了 0．  
  
继续 Shift+F9，经过十数个后就来到这里(当中会发现 ASPR 比以前的 access memory 错误还多了一些 Int3)：  
  
01363D5F   3100             XOR DWORD PTR DS:\[EAX\],EAX                 <-----Ollydbg 停在这里  
01363D61   64:8F05 00000000 POP DWORD PTR FS:\[0\]                       <-----我们在这里下断点  
01363D68   58               POP EAX  
01363D69   833D BC7E3601 00 CMP DWORD PTR DS:\[1367EBC\],0  
01363D70   74 14            JE SHORT 01363D86  
01363D72   6A 0C            PUSH 0C  
01363D74   B9 BC7E3601      MOV ECX,1367EBC  
01363D79   8D45 F8          LEA EAX,DWORD PTR SS:\[EBP\-8\]  
01363D7C   BA 04000000      MOV EDX,4  
01363D81   E8 8ED2FFFF      CALL 01361014  
01363D86   FF75 FC          PUSH DWORD PTR SS:\[EBP\-4\]  
01363D89   FF75 F8          PUSH DWORD PTR SS:\[EBP\-8\]  
01363D8C   8B45 F4          MOV EAX,DWORD PTR SS:\[EBP\-C\]  
01363D8F   8338 00          CMP DWORD PTR DS:\[EAX\],0  
01363D92   74 02            JE SHORT 01363D96  
01363D94   FF30             PUSH DWORD PTR DS:\[EAX\]  
01363D96   FF75 F0          PUSH DWORD PTR SS:\[EBP\-10\]  
01363D99   FF75 EC          PUSH DWORD PTR SS:\[EBP\-14\]  
01363D9C   C3               RETN  
01363D9D   5F               POP EDI  
01363D9E   5E               POP ESI  
01363D9F   5B               POP EBX  
01363DA0   8BE5             MOV ESP,EBP  
01363DA2   5D               POP EBP  
01363DA3   C3               RETN  
  
Shift+F9，ollydbg 就停在我们的断点上．．接下去已经没多少路了，手动吧．F8 一直到 Retn，然后就在这里：  
  
01379330   68 D3F8FD02      PUSH 2FDF8D3  
01379335   B4 4F            MOV AH,4F  
01379337   5B               POP EBX  
01379338   66:81D3 27F5     ADC BX,0F527  
0137933D   E8 09000000      CALL 0137934B         <------F7 进去  
  
接着一直到：  
013793EA   5B               POP EBX  
013793EB   81C7 5075A77E    ADD EDI,7EA77550  
013793F1   E8 0B000000      CALL 01379401         <------F7 进去  
  
接下去小心一点，用 ollydbg 的 trace eip 很快就能返回程序的领空，首先就是停在这里：  
  
00407278   $-FF25 2CF35600  JMP DWORD PTR DS:\[56F32C\] 　　<-----我们停在这里  
0040727E     8BC0           MOV EAX,EAX  
00407280   $-FF25 28F35600  JMP DWORD PTR DS:\[56F328\]  
00407286     8BC0           MOV EAX,EAX  
00407288   $-FF25 24F35600  JMP DWORD PTR DS:\[56F324\]  
0040728E     8BC0           MOV EAX,EAX  
00407290   $-FF25 20F35600  JMP DWORD PTR DS:\[56F320\]  
  
对于 1.2x 和大多数的 1.3+来说，trace eip 停下来就是 OEP 里，但新版本可以看到明显不一样，F7 跟进去，这里有一个技巧，如果发现停下来的代码都是 DB FF 之类的，大多数情况下选择右键菜单中的 analysis/analyze code 就可以看到正常的 code．有时也不行，下面会看到  
  
01361504   55               PUSH EBP  
01361505   8BEC             MOV EBP,ESP  
01361507   8B45 08          MOV EAX,DWORD PTR SS:\[EBP+8\]  
0136150A   85C0             TEST EAX,EAX  
0136150C   75 13            JNZ SHORT 01361521  
0136150E   813D 70793601 00>CMP DWORD PTR DS:\[1367970\],400000         ASCII "MZP"  
01361518   75 07            JNZ SHORT 01361521  
0136151A   A1 70793601      MOV EAX,DWORD PTR DS:\[1367970\]  
0136151F   EB 06            JMP SHORT 01361527  
01361521   50               PUSH EAX  
01361522   E8 853FFFFF      CALL 013554AC                             JMP to kernel32.GetModuleHandleA  
01361527   5D               POP EBP  
01361528   C2 0400          RETN 4  
  
看到上面两个指令是不是很心动．．．继续走  
  
0040734D   . A3 68B65600    MOV DWORD PTR DS:\[56B668\],EAX            <-----看到 40734d 这个 EIP 么，很重要  
00407352   . A1 68B65600    MOV EAX,DWORD PTR DS:\[56B668\]  
00407357   . A3 D8205600    MOV DWORD PTR DS:\[5620D8\],EAX  
0040735C   . 33C0           XOR EAX,EAX  
0040735E   . A3 DC205600    MOV DWORD PTR DS:\[5620DC\],EAX  
00407363   . 33C0           XOR EAX,EAX  
00407365   . A3 E0205600    MOV DWORD PTR DS:\[5620E0\],EAX  
0040736A   . E8 C1FFFFFF    CALL SystemCl.00407330  
0040736F   . BA D4205600    MOV EDX,SystemCl.005620D4  
00407374   . 8BC3           MOV EAX,EBX  
00407376   . E8 75D8FFFF    CALL SystemCl.00404BF0  
0040737B   . 5B             POP EBX  
0040737C   . C3             RETN  
  
继续走，会停在  
  
005613EB     E8             DB E8  
005613EC     4C             DB 4C                                      CHAR 'L'  
005613ED     5F             DB 5F                                      CHAR '\_'  
005613EE     EA             DB EA  
005613EF     FF             DB FF  
005613F0     FF             DB FF                <----我们停在这里  
005613F1     15             DB 15  
005613F2     84A15600       DD SystemCl.0056A184  
005613F6     E8             DB E8  
005613F7     BD             DB BD  
005613F8     39             DB 39                                      CHAR '9'  
005613F9     EA             DB EA  
005613FA     FF             DB FF  
  
这里就不用 analyse 了，检查了也没用．．．但分析一下上面的 binary ，什么意思呢？反汇编一下就知道了  
  
005613EB     E8 4C5FEAFF    CALL SystemCl.0040734D  
005613F0     FF15 84A15600  CALL DWORD PTR DS:\[56A184\]                 SystemCl.005608F0  
005613F6     E8 BD39EAFF    CALL SystemCl.00404DB8  
  
看到了么，最上面那个 cALL 我们已经运行过了，很显然，aSPR 更狡猾了，直接在壳里运行了第一个 cALL，所以 oep 就在第一个 CALL 位置:5613EB  
  
剩下要做的工作就是恢复 11 个 STOLEN BYTES，分析一下刚才走过的程序就可以得到，还有 IAT，这些就不在这里详述了  
  
这篇东西是在下午上班时抽空半小时写的．．．若有错误，请多多包涵．．．  

 4dd466d0e1e0cc123afbe504da2462ca