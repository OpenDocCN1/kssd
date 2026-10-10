

# EXECryptor1.53

> 原文：[`www.pediy.com/kssd/pediy06/pediy6458.htm`](https://www.pediy.com/kssd/pediy06/pediy6458.htm)

以理服人,巧脱 EXECryptor1.53 的壳   
  
工具：OllyDBG1.1B 中文版;Lordpe;   
操作系统：Win2K/WINXP   
    近来关于 EXECryptor1.53 的脱壳很是热闹，有的在虚拟机里脱；有的靠猜测脱。那么偶呢？偶是"以理服人,巧脱 EXECryptor1.53 的壳"。   
  
一、脱壳过程：   
用 Ollydbg 加载后,停留载程序入口：   
00401000 >CALL EXECrypt.00401029   
00401005  MOV ECX,DWORD PTR SS:\[ESP+C\]   
00401009  MOV DWORD PTR DS:\[ECX\],10017   
0040100F  MOV DWORD PTR DS:\[ECX+B8\],EXECrypt.004FC000   
00401019  XOR EAX,EAX   
0040101B  MOV DWORD PTR DS:\[ECX+14\],EAX   
0040101E  MOV DWORD PTR DS:\[ECX+18\],EAX   
00401021  AND BYTE PTR DS:\[ECX+C1\],0FE   
00401028  RETN   
00401029  XOR EAX,EAX   
0040102B  PUSH DWORD PTR FS:\[EAX\]   
0040102E  MOV DWORD PTR FS:\[EAX\],ESP   
00401031  INT3   
00401032  RETN   
在调试设置(按 ALT-O)中，异常除"内存访问异常"外全部忽略(其它全打勾),在同时忽略以下指定的异常或者范围中添加 C000001E 并勾上。F9 运行，出现提示后选 N(不执行),然后 Ctr-G:004FC74B 到异常处:   
004FC74B  LOCK CMPXCHG8B EAX   ; 非法使用寄存器   
修改 004FC74B 处的值 F0 为 CC(int3)后,按 shift-F9 继续。出现异常:   
004FDFF4  MOV SS,BX  ; 修正的段位寄存器   
004FDFF6  PUSH 0   
004FDFF8  JMP SHORT EXECrypt.004FDFFB   
(弄懂原因后，就在这里下硬件断点，到数据转存窗口按 Ctrl-G:0012ff84 然后在 0012FF84 处下硬件断点：硬件访问->Dword)   
  
shift-f9 继续。出现：   
004FF225  MOV SS,AX     ; 修正的段位寄存器   
堆栈窗口显示：   
0012FF60  /0012FF7C  指针到下一个 SEH 记录   
0012FF64  |004FF00D  SE 句柄   
0012FF68  |004FC35C  返回到 EXECrypt.004FC35C 来自 EXECrypt.004FEF70   
0012FF6C  |004FC347  返回到 EXECrypt.004FC347 来自 EXECrypt.004FC354   
0012FF70  |004FE0BE  返回到 EXECrypt.004FE0BE 来自 EXECrypt.004FE0CB   
0012FF74  |004FE113  返回到 EXECrypt.004FE113 来自 EXECrypt.004FC327   
0012FF78  |004FE076  返回到 EXECrypt.004FE076 来自 EXECrypt.004FE081   
0012FF7C   012FFE0  指针到下一个 SEH 记录   
0012FF80   004FDD6E  SE 句柄   
0012FF84   77E80978  kernel32.GetModuleHandleA   
0012FF88   77E80B09  kernel32.GetProcAddress   
0012FF8C   77E8007F  kernel32.LoadLibraryA   
0012FF90   77E806A9  kernel32.FreeLibrary   
0012FF94   77E7E815  kernel32.VirtualProtect   
上面第 2 行指示 SEH 异常处理地址:004FF00D,于是到 004FF00D 处按 F2 设断。(熟练后，这步可以省略，主要是为了给大家弄清原因)   
再看 0012FF84 为函数 GetModuleHandleA 入口地址。到数据转存窗口按 Ctrl-G:0012ff84 然后在 0012FF84 处下硬件断点：硬件访问->Dword(后面会告诉你为什么,如果在前一异常处下了硬件断点这里就不要再下)   
Shift-F9 继续运行到 004FF00D 断下后再 F2 取消断点。   
004FF00D  MOV ECX,DWORD PTR SS:\[ESP+C\]   
004FF011  MOV DWORD PTR DS:\[ECX\],10017   
按 F7 一路下来到 004FF027 后开始分析(每行//后面为分析说明)。   
004FF027  MOV EAX,DWORD PTR DS:\[ECX+B0\]   
//\[ecx+b0\]为异常时的 EAX,EAX=0:GetModuleHandleA;EAX=1:GetProcAddress;EAX=2:LoadLibraryA;EAX=3:FreeLibrary;EAX=4:VirtualProtect   
004FF02D  SHL EAX,2     //EAX 乘以 4,每个函数的入口地址为 4 字节。   
004FF030  ADD EAX,DWORD PTR DS:\[ECX+4\]  //\[ECX+4\]=DR0,通过 DR0 来传送指向 5 个函数入口地址的起始地址。没有被调试时\[ecx+4\]=DR0=0012FF84,程序正常运行.如果程序被调试时，win2k/winxp 下 OllyDBG 会置 DR0=0,那么就得不到函数正确地址。由于前面设置了硬件断点为 0012FF84 而让调试器 Ollydbg 给 DR0 附值=0012ff84，即\[ecx+4\]=0012FF84,这就是前面为什么要设置硬件断点的原因。而且这点也在 win98 下得到验证,win98 下 OllyDBG 不支持硬件断点,所以\ecx+4\就是对应的该堆栈的值(不过不是 12ff84,而是 72fdfc,同样指向该 5 个函数的地址).   
004FF033  MOV DWORD PTR DS:\[ECX+B0\],EAX //\[EAX\]单元为相应函数的入口地址,并保存到\ECX+B0\   
004FF039  ADD DWORD PTR DS:\[ECX+B8\],7   //改变 EIP 为异常时的 eip 加 7.   
004FF040  CALL EXECrypt.004FF050   
此时在命令窗口设置断点：bp VirtualProtect+1,为什么加 1？因为壳会判断函数入口指令是否为 0CC(int3)/0CD(INT)/68(PUSH)/0E9(JMP)等.   
再次修改调试设置(按 ALT-O)，把异常全部忽略(全打勾)。F9 继续。断在 VirtualProtect+1 处：   
77E7E816  MOV EBP,ESP   
77E7E818  PUSH DWORD PTR SS:\[EBP+14\]   
77E7E81B  PUSH DWORD PTR SS:\[EBP+10\]   
77E7E81E  PUSH DWORD PTR SS:\[EBP+C\]   
77E7E821  PUSH DWORD PTR SS:\[EBP+8\]   
77E7E824  PUSH -1   
77E7E826  CALL kernel32.VirtualProtectEx   
F9 继续，第 2 次断下后，一路 F8 直到返回到程序的领空。   
然后 F7 到下面代码开始打扫战场。   
004FEDBD  PUSH EDI   
004FEDBE  MOV EDI,EXECrypt.004FEBAD   
004FEDC3  PUSH ECX   
004FEDC4  MOV ECX,222   
004FEDC9  CLD   
004FEDCA  MOV AL,68   
004FEDCC  STOS BYTE PTR ES:\[EDI\]   
004FEDCD  MOV EAX,EXECrypt.004FEE03   
004FEDD2  STOS DWORD PTR ES:\[EDI\]   
004FEDD3  MOV AL,0C3   
004FEDD5  REPNE STOS BYTE PTR ES:\[EDI\]   
004FEDD7  POP ECX   
004FEDD8  POP EDI   
004FEDD9  JMP SHORT EXECrypt.004FEE03   
再一路 F7 到：   
004FEE26   >ADD ESP,4   
004FEE29   >RETN   //返回到 004B58E5,程序入口+1：   
  
004B58E5  MOV EBP,ESP   
004B58E7  ADD ESP,-18   
004B58EA  PUSH EBX   
    明白的原因大家完全可以简化以上步骤。偶就不罗嗦了。   
  
二、WIN9X 下用 twx2002 脱壳   
   WIN9X 用 twx2002 脱壳更简单，只要用 BPX VirtualProtect+2，在第 2 次断下后一步一步跟,就到了 OEP.所以 EXECryptor1.53 算是弱壳。  
 4dd466d0e1e0cc123afbe504da2462ca