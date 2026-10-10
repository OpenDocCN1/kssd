

# 用 OD 手脱 Armadillo v4.40 DLL 壳 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-555.htm`](https://www.pediy.com/kssd/pediy08/pediy8-555.htm)

【脱壳作者】 刹那恍惚  
【作者邮箱】 RegKiller2002@yahoo.com.cn  
【作者 QQ 】 14403147  
【使用工具】 OllyDBG 汉化第二版,LordPE,ImportREC v1.6F,PEiD v0.94,PE Tools v1.5.700  
【脱壳平台】 WinXP SP2  
【脱壳目标】 Armadillo v4.40 加过壳的 EdrLib.dll 文件  
【保护选项】 Standard protection only 仅仅标准保护 (单进程)  
【加壳方式】 Armadillo v4.40  
  
\--------------------------------------------------------------------------------  
【脱壳内容】  
  
  
一、准备工作  
  
1 侦壳：用 PEiD 查壳 Armadillo 2.51 - 3.xx DLL Stub -&gt; Silicon Realms Toolworks  
  
这里如何判断 Arm 的版本呢？记得 FLY 大狭说过 Armadillo V4.0 新增的反跟踪手段:  
  
OllyDbg 在处理调式包含格式串的消息时存在问题，被跟踪的应用程序可以使 OllyDbg 崩溃，或可能以进程权限执行任意指令。OutputDebugString 函数可发送字符串到调试器上，然后 OllyDbg 会在底端显示相关状态消息，但是如果包含格式串消息，就可能使 OllyDbg 崩溃。Armadillo 以前的版本没有此种 Anti，自 V4.0 始才有。  
  
有他这句话我们就可以做如下判断了：  
  
OD 载入  
  
下断点 HE OutputDebugStringA   
  
Shift+F9 运行，中断下来。看堆栈：  
  
0006EA98   00B8580F  /CALL 到 OutputDebugStringA 来自 00B85809  
0006EA9C   0006F410  \\String = "%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s"  
  
出现这个说明这个 DLL 是经过 Armadillo 4.X 压缩过  
  
2 判断进程：DLL 文件加壳应该是不可以双进程，所以这里是单进程方式。  
  
二 、脱壳   
  
1 寻找 Magic Jump   
  
先用 LordPE 看看加壳后的 DLL 信息：基址＝00400000，入口点＝0003FE97  
  
设置 Ollydbg 忽略所有的异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志  
  
清除断点后 OD 重新载入  
  
008AFE97 &gt;/$  55            PUSH EBP                                 ;  停在这里  
008AFE98  |.  8BEC          MOV EBP,ESP  
008AFE9A  |.  53            PUSH EBX  
008AFE9B  |.  8B5D 08       MOV EBX,DWORD PTR SS:\[EBP+8\]  
008AFE9E  |.  56            PUSH ESI  
  
下断点 BP GetModuleHandleA+5，Shift+F9 运行中断后，注意看堆栈：  
  
这里用 BP API+5 是为了躲过壳检查是否下过 API 断点，相关资料看雪论坛上面可以找到。  
  
0006E458  /0006E574  
0006E45C  |74683BEE  返回到 74683BEE 来自 kernel32.GetModuleHandleA  
0006E460  |0006E464  ASCII "D:\\WINDOWS\\system32\\ntdll.dll"  
  
0006E460  /0006E57C  
0006E464  |74683BEE  返回到 74683BEE 来自 kernel32.GetModuleHandleA  
0006E468  |0006E46C  ASCII "D:\\WINDOWS\\system32\\imm32.dll"  
  
0006E3AC  /0006E4C8  
0006E3B0  |74683BEE  返回到 74683BEE 来自 kernel32.GetModuleHandleA  
0006E3B4  |0006E3B8  ASCII "D:\\WINDOWS\\system32\\KERNEL32"  
  
0006E504  /0006E620  
0006E508  |7365D4A4  返回到 msctfime.7365D4A4 来自 kernel32.GetModuleHandleA  
0006E50C  |0006E510  ASCII "D:\\WINDOWS\\system32\\ntdll.dll"  
  
0006ED60  /0006ED7C  
0006ED64  |77F45BD8  返回到 77F45BD8 来自 kernel32.GetModuleHandleA  
0006ED68  |77F4501C  ASCII "KERNEL32.DLL"  
  
00069364  /0006EAAC  
00069368  |00B86DF3  返回到 00B86DF3 来自 kernel32.GetModuleHandleA  
0006936C  |00B9BC1C  ASCII "kernel32.dll"  
00069370  |00B9CEC4  ASCII "VirtualAlloc"  
  
00069364  /0006EAAC  
00069368  |00B86E10  返回到 00B86E10 来自 kernel32.GetModuleHandleA  
0006936C  |00B9BC1C  ASCII "kernel32.dll"  
00069370  |00B9CEB8  ASCII "VirtualFree"  
  
000690C8  /00069368  
000690CC  |00B75CE1  返回到 00B75CE1 来自 kernel32.GetModuleHandleA  
000690D0  |0006921C  ASCII "kernel32.dll"                              ; ★ 注意！在这里清除断点后 Alt+F9 返回程序  
  
这里说下我判断返回的经验，我的经验是一般出现下面这两句就快到返回点了  
  
00069364  /0006EAAC  
00069368  |00B86DF3  返回到 00B86DF3 来自 kernel32.GetModuleHandleA  
0006936C  |00B9BC1C  ASCII "kernel32.dll"  
00069370  |00B9CEC4  ASCII "VirtualAlloc"                              ; ★  注意这句  
  
  
00069364  /0006EAAC  
00069368  |00B86E10  返回到 00B86E10 来自 kernel32.GetModuleHandleA  
0006936C  |00B9BC1C  ASCII "kernel32.dll"  
00069370  |00B9CEB8  ASCII "VirtualFree"                               ; ★  注意这句  
  
  
00B75CE1    8B0D AC40BA00   MOV ECX,DWORD PTR DS:\[BA40AC\]            ; 返回到这里  
00B75CE7    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX  
00B75CEA    A1 AC40BA00     MOV EAX,DWORD PTR DS:\[BA40AC\]  
00B75CEF    391C06          CMP DWORD PTR DS:\[ESI+EAX\],EBX  
00B75CF2    75 16           JNZ SHORT 00B75D0A  
00B75CF4    8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:\[EBP\-14C\]  
00B75CFA    50              PUSH EAX  
00B75CFB    FF15 BC62B900   CALL DWORD PTR DS:\[B962BC\]               ; kernel32.LoadLibraryA  
00B75D01    8B0D AC40BA00   MOV ECX,DWORD PTR DS:\[BA40AC\]  
00B75D07    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX  
00B75D0A    A1 AC40BA00     MOV EAX,DWORD PTR DS:\[BA40AC\]  
00B75D0F    391C06          CMP DWORD PTR DS:\[ESI+EAX\],EBX  
00B75D12    0F84 2F010000   JE 00B75E47                              ; Magic Jump 改 JE 为 JMP 避开 IAT 加密  
00B75D18    33C9            XOR ECX,ECX  
00B75D1A    8B07            MOV EAX,DWORD PTR DS:\[EDI\]  
00B75D1C    3918            CMP DWORD PTR DS:\[EAX\],EBX  
00B75D1E    74 06           JE SHORT 00B75D26  
00B75D20    41              INC ECX  
00B75D21    83C0 0C         ADD EAX,0C  
00B75D24  ^ EB F6           JMP SHORT 00B75D1C  
  
把 00B75D12 这句的 JE 00B75E47 改成 JMP 00B75E47  
  
2 获得重定位信息  
  
下断点 bp GetTickCount，Shift+F9 运行中断后，注意看堆栈：  
  
00069370   00B8C009  /CALL 到 GetTickCount 来自 00B8C003  
  
00069370   00B8C3C8  /CALL 到 GetTickCount 来自 00B8C3C2             ; ★ 注意！在这里清除断点后 Alt+F9 返回程序  
  
  
00B8C3C8    2B85 A4D4FFFF   SUB EAX,DWORD PTR SS:\[EBP\-2B5C\]          ; 返回到这里  
00B8C3CE    8B8D A8D4FFFF   MOV ECX,DWORD PTR SS:\[EBP\-2B58\]  
00B8C3D4    6BC9 32         IMUL ECX,ECX,32  
00B8C3D7    81C1 D0070000   ADD ECX,7D0  
00B8C3DD    3BC1            CMP EAX,ECX  
00B8C3DF    76 07           JBE SHORT 00B8C3E8  
00B8C3E1    C685 34D9FFFF 0&gt;MOV BYTE PTR SS:\[EBP\-26CC\],1  
00B8C3E8    83BD E4D7FFFF 0&gt;CMP DWORD PTR SS:\[EBP\-281C\],0  
00B8C3EF    0F85 8A000000   JNZ 00B8C47F  
  
在 CPU 窗口按 Ctrl+S 查找如下代码  
  
PUSH EAX  
XCHG CX,CX  
POP EAX  
STC  
  
找到代码如下：  
  
00B8CF54    50              PUSH EAX  
00B8CF55    66:87C9         XCHG CX,CX  
00B8CF58    58              POP EAX  
00B8CF59    F9              STC  
  
我们在 00B8CF54 行按 F2 设置断点，然后 F9 执行后取消断点到这里后窗口里出现了红色代码  
  
00B8CF59    C705 E0C0B900 6&gt;MOV DWORD PTR DS:\[B9C0E0\],0B9CB60     ; ★ 从这里开始出现红色代码  
00B8CF63    A1 E49FBA00     MOV EAX,DWORD PTR DS:\[BA9FE4\]  
00B8CF68    8B00            MOV EAX,DWORD PTR DS:\[EAX\]               ; ★ 这个 00006000 就是重定位表的 RVA  
00B8CF6A    8985 3CD9FFFF   MOV DWORD PTR SS:\[EBP\-26C4\],EAX  
00B8CF70    A1 E49FBA00     MOV EAX,DWORD PTR DS:\[BA9FE4\]  
00B8CF75    83C0 04         ADD EAX,4  
00B8CF78    A3 E49FBA00     MOV DWORD PTR DS:\[BA9FE4\],EAX  
00B8CF7D    A1 E49FBA00     MOV EAX,DWORD PTR DS:\[BA9FE4\]  
00B8CF82    8B00            MOV EAX,DWORD PTR DS:\[EAX\]               ; ★ 这个 000003B0 就是重定位表的大小  
00B8CF84    8985 78D9FFFF   MOV DWORD PTR SS:\[EBP\-2688\],EAX  
00B8CF8A    A1 E49FBA00     MOV EAX,DWORD PTR DS:\[BA9FE4\]  
00B8CF8F    83C0 04         ADD EAX,4  
00B8CF92    A3 E49FBA00     MOV DWORD PTR DS:\[BA9FE4\],EAX  
00B8CF97    83BD 3CD9FFFF 0&gt;CMP DWORD PTR SS:\[EBP\-26C4\],0            ; ★ 重定位表的 RVA 为 0 吗？  
00B8CF9E    74 6F           JE SHORT 00B8D00F                        ; ★ 为 0 则重定位处理  
00B8CFA0    83BD 78D9FFFF 0&gt;CMP DWORD PTR SS:\[EBP\-2688\],0            ; ★ 重定位表的大小为 0 吗？  
00B8CFA7    74 66           JE SHORT 00B8D00F                        ; ★ 为 0 则重定位处理  
00B8CFA9    8B85 FCD7FFFF   MOV EAX,DWORD PTR SS:\[EBP\-2804\]  
00B8CFAF    8B8D 0CD8FFFF   MOV ECX,DWORD PTR SS:\[EBP\-27F4\]  
00B8CFB5    3B48 34         CMP ECX,DWORD PTR DS:\[EAX+34\]  
00B8CFB8    74 55           JE SHORT 00B8D00F                        ; ★ 如与映像基址不符则重定位处理！  
00B8CFBA    FFB5 78D9FFFF   PUSH DWORD PTR SS:\[EBP\-2688\]  
00B8CFC0    8B85 0CD8FFFF   MOV EAX,DWORD PTR SS:\[EBP\-27F4\]  
00B8CFC6    0385 3CD9FFFF   ADD EAX,DWORD PTR SS:\[EBP\-26C4\]  
00B8CFCC    50              PUSH EAX  
00B8CFCD    8B85 FCD7FFFF   MOV EAX,DWORD PTR SS:\[EBP\-2804\]  
00B8CFD3    FF70 34         PUSH DWORD PTR DS:\[EAX+34\]  
00B8CFD6    FFB5 0CD8FFFF   PUSH DWORD PTR SS:\[EBP\-27F4\]  
00B8CFDC    E8 3C150000     CALL 00B8E51D                            ; ★ 重定位处理 CALL  
00B8CFE1    83C4 10         ADD ESP,10  
00B8CFE4    0FB6C0          MOVZX EAX,AL  
00B8CFE7    85C0            TEST EAX,EAX  
00B8CFE9    75 24           JNZ SHORT 00B8D00F  
00B8CFEB    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]  
00B8CFEE    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
00B8CFF0    C700 07000000   MOV DWORD PTR DS:\[EAX\],7  
00B8CFF6    68 50CBB900     PUSH 0B9CB50                             ; ASCII "Location CPG"  
00B8CFFB    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]  
00B8CFFE    FF70 04         PUSH DWORD PTR DS:\[EAX+4\]  
00B8D001    E8 24800000     CALL 00B9502A                            ; JMP 到 msvcrt.strcpy  
00B8D006    59              POP ECX  
00B8D007    59              POP ECX  
00B8D008    33C0            XOR EAX,EAX  
00B8D00A    E9 A5070000     JMP 00B8D7B4  
  
我们把 00B8CFB8 的 JE 00B8D00F 改成 JMP 00B8D00F 跳过重定位处理，这样就不需要修改 DLL 的基址了，否则修改基址为 OEP 处看到的基址，如这里为 00870000  
  
现在我们 Alt+M 打开内存查看窗口，看到这个 DLL 的给个区段  
  
00870000   00001000   EdrLib                PE 文件头        Imag   R         RWE  
00871000   00003000   EdrLib     .text                    Imag   R         RWE  
00874000   00001000   EdrLib     .rdata     输出表           Imag   R         RWE  
00875000   00001000   EdrLib     .data      数据            Imag   R         RWE  
00876000   00001000   EdrLib     .reloc                   Imag   R         RWE  
00877000   00040000   EdrLib     .text1     代码            Imag   R         RWE  
008B7000   00010000   EdrLib     .adata     代码            Imag   R         RWE  
008C7000   00010000   EdrLib     .data1                   Imag   R         RWE  
008D7000   00010000   EdrLib     .reloc1    重定位           Imag   R         RWE  
008E7000   00030000   EdrLib     .pdata     输入表           Imag   R         RWE   
  
在 00871000   00003000   EdrLib     .text ★  这里设置内存访问断点 F9 运行，中断在 OEP  
  
008711C9    55              PUSH EBP                                 ; 等待已久的 OEP 终于出现了  
008711CA    8BEC            MOV EBP,ESP  
008711CC    53              PUSH EBX  
008711CD    8B5D 08         MOV EBX,DWORD PTR SS:\[EBP+8\]  
008711D0    56              PUSH ESI  
008711D1    8B75 0C         MOV ESI,DWORD PTR SS:\[EBP+C\]  
008711D4    57              PUSH EDI  
008711D5    8B7D 10         MOV EDI,DWORD PTR SS:\[EBP+10\]  
008711D8    85F6            TEST ESI,ESI  
008711DA    75 09           JNZ SHORT EdrLib.008711E5  
  
用 LordPE 选中 Ollydbg 的 loaddll.exe 的进程，在下面的列表里选择 EdrLib.dll，然后完整脱壳，得到 dumped.dll。  
  
我们用 LordPE 打开 dumped.dll 到目录中修改 重定位:RVA=6000 大小=3B0 然后保存修改  
  
3 搞定输入表  
  
因为已经修改了 Magic Jump，所以现在可以得到完整的输入表。随便从程序找个 API 调用：  
  
00871383    FF15 24404000   CALL DWORD PTR DS:\[404024\]  
  
由于我们刚才已经跳过了重定位处理所以这里是未经过重定位处理的地址  
我们在命令行里输入 D 874024 上下看到许多函数地址，很明显的可以找到 IAT 开始和结束的地址：  
  
00874024  A2 CA 81 7C 16 1E 80 7C 0D E0 80 7C 37 97 80 7C  ...|...|...|7..|  
00874034  F5 9B 80 7C 0F 2B 81 7C 53 34 81 7C 50 97 80 7C  ...|.+.|S4.|P..|  
00874044  CF C6 80 7C A9 2C 81 7C 69 10 81 7C EE 1E 80 7C  ...|?.|i..|...|  
00874054  8A 18 93 7C 57 B3 80 7C 3F DC 81 7C E0 C6 80 7C  ...|W..|?..|...|  
  
我们在数据窗口中用长型地址的方式显示数据将看到许多函数  
  
00874024  7C81CAA2  kernel32.ExitProcess  
00874028  7C801E16  kernel32.TerminateProcess  
0087402C  7C80E00D  kernel32.GetCurrentProcess  
00874030  7C809737  kernel32.GetCurrentThreadId  
00874034  7C809BF5  kernel32.TlsSetValue  
00874038  7C812B0F  kernel32.TlsAlloc  
0087403C  7C813453  kernel32.TlsFree  
00874040  7C809750  kernel32.TlsGetValue  
00874044  7C80C6CF  kernel32.SetHandleCount  
00874048  7C812CA9  kernel32.GetStdHandle  
0087404C  7C811069  kernel32.GetFileType  
00874050  7C801EEE  kernel32.GetStartupInfoA  
  
现在上下滚动窗口就可以很容易的找到 IAT 的开始和结束的地址  
  
开始地址＝00874000  
结束地址＝008740CB  
  
但是现在直接用 ImportREC 选取 EdrLib.dll,填入 RVA=00004000、大小＝CB，却提示“不能载入当前进程相关数据信息!”  
看看 ImportREC 的日志：  
映像基地址:00400000 大小:00097000  
\-&gt;&gt; 模块被选择! : e:\\试炼场\\脱壳学习\\dll 脱壳\\armadillo\\edrlib.dll\\edrlib.dll   
原来 ImportREC 显示 EdrLib.dll 的基址还是 00400000，呵呵  
          
如果填入 RVA=00474000、大小＝CB，可以得到输入表，却无法完成修复抓取文件。为何？都是重定位惹的祸啦。  
于是利用 FLY 大狭的移花接木的办法：再打开一个 Ollydbg，载入 Win98 的 NotePad.EXE，然后把 00874000－008740CB  的数据复制、粘贴进 NotePad.EXE 的 00404000－004040CB，然后用 ImportREC 选择 NotePad.EXE 进程，填入 RVA=00004000、大小＝CB，得到输入表，CUT 掉垃圾指针，改 OEP=000011C9，就可以 FixDump 啦！  
  
三 善后工作  
  
1 优化  
  
用 LordPE 删除 dumped\_.dll 的 text1、adata、data1、reloc1、pdata 共 5 个区段，然后去掉“转存修正”和“清除重定位表”选项，重建 PE 简单优化一下脱壳后的文件，672K-&gt;16.9K，比加壳前的原文件还小了。  
  
2 修复查壳显示错误  
  
脱壳后的 DLL 用 PEiD 看依旧显示“Armadillo 2.51 - 3.xx DLL Stub -&gt; Silicon Realms Toolworks”，用 FI 看显示“MS VC++ v6.0 {DLL} ”。用 PE Tools 打开 dumped\_.dll 在可选头部中修改主连接器版本为 06 副连接器版本 00，现在再用 PEiD 查看显示为：Microsoft Visual C++ 6.0 DLL  
  
3 测试脱壳文件  
  
把 dumped\_.dll 名字改成 EdrLib.dll 然后运行 EdrTest.exe 看是否正确调用脱壳后的 DLL 文件  
  
\--------------------------------------------------------------------------------  
【脱壳总结】  
  
Armadillo v4.40 这个版本的获得重定位信息的位置与以前的版本有点出入，我的获得方法是在修改了 Magic Jump 后对 00870000 段设置内存访问断点中断 1 次后一步步手动跟踪后才发现用 bp GetTickCount 这个断点断 2 次后返回并 F8 单步向下走到下面这个特征码  
  
PUSH EAX  
XCHG CX,CX  
POP EAX  
STC  
  
这里教大家一个判断重定位位置和大小的方法。不知道是否通用。  
  
我们可以把 DUMP 后的文件用 LoadPE 打开。然后在区段里查看 reloc 的 VOffset 来确定重定位的 RAV，然后在目录-&gt;重定位里点"H"按钮。把从 6000 开始到后面全是 00000000 的这段全部选中后看状态行提示的大小就可以了。  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  
本代码由 xTiNt 自动着色 [`kbadboy.yeah.net`](http://kbadboy.yeah.net) 

 4dd466d0e1e0cc123afbe504da2462ca