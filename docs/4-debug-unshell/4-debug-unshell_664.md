

# 【翻译】ExeCryptor 2.3.9 - Unpacking

> 原文：[`www.pediy.com/kssd/pediy10/60035.html`](https://www.pediy.com/kssd/pediy10/60035.html)

作者:haggar  
原文链接:http://www.reversing.be/article.php?story=20061206203632615  
译者:Cyg07(cyg0x7 at 0wei.com)  
译者注:本人翻译不佳,望路过的兄弟多多指正!  
感谢 machenglin 和 KuNgBiM 两位前辈的耐心指导!  
  
  
本文是针对 ExeCryptor 2.3.9 小部分更新而写的脱文。本文主要目的是脱壳,故没有做后期的破解。脱壳前你需要用到的工具有:OllyDbg(包含脚本插件和一些隐藏插件),LordPE,ImpREC 和 一个稳定 Windows XP 平台。  
  
\- 调试前的准备 -  
  
首先打开 Olly 中的 "Debugging options" 选项,在 "events" 中设置 "Make first pause at:System breakpoint"。这样一来可以防止 ExeCryptor 直接在 TLS 回调上做的手脚。好了,我们开始把目标程序载入 Olly 。  
  
打开断点管理窗口,清除断点:  
  
0076E427 EXECrypt One-shot CALL EXECrypt.0076E323  
  
使用一些插件对 Olly 进行隐藏设置(或者参看以前的教程进行手工清除校验):  
IsDebuggerPresent/BeingDebugged  
NtGlobalFlag  
HeapFlag  
CheckRemoteDebuggerPresent  
  
不要在插件中设置任何检查,因为 EC(ExeCryptor)都有可能发现它的行为。EC 一般都会检测一些 HOOK 行为。  
  
Ctrl+G 查找 OutputDebugStringA 函数,并把函数的开始部分用 "RETN 4" 操作码替换。此举是为了防止 Olly 处理 "s%s%s" 的 Bug。  
  
\- 巧用硬件异常 -  
  
这是个新的技巧。该版的 EC 好象或多或少的设置了些硬件异常断点。我不知道这点是如此体现出来的,但可以使用 Olly 来检测。  
  
来到 Olly 的异常选项窗口，打上所有复选框。清除所有自定义异常。确保你没有修改任何内存，软件或者硬件断点。现在 Shift+F9 运行程序。你将停在第一个异常处：  
  
007637D0    F0:F1           LOCK INT1  
  
连续 Shift+F9(3 次以上),然后检查下硬件断点(Debug menu -> Hardware breakpoints)。你将看到硬件断点窗口中有个断点:  
  
Hardware breakpoints  
# Base Size Stop on  
1 0076098F Temporary  
  
这是个 EC 脱壳的新技巧。如果你带着这个硬件断点继续调试,最终程序将会挂掉。所以我们应该删除该断点。另外回到异常选项框里在自定义栏中添加入最后异常(C000001E INVALID LOCK SEQUENCE)。  
  
\- FindWindowA 检测 -  
  
硬件断点检测问题被解决了。接下来的 IsDebuggerPresent 检测也已经被我们的插件解决了。该是解决 FindWindowA 检测的时候,EC 会搜索当前进程中含有"OLLYDBG" 类名的窗口。这里我们还是 Ctrl+G->FindWindowA,在函数入口处设置断点。Shift+F9 后停在断点处,EC 对 API 的函数的检测行为真的很烦琐(开始清理下代码中的校验):  
  
0055A3DB    8A00            MOV AL,BYTE PTR DS:\[EAX\]  
00595A6F    2C 99           SUB AL,99  
00595A71    8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  
00595A74    F62A            IMUL BYTE PTR DS:\[EDX\]  
005DEE5C    3C A4           CMP AL,0A4  
005DEE5E  ^ 0F85 2BFFF7FF   JNZ EXECrypt.0055ED8F  
005DEE64    E9 598A0000     JMP EXECrypt.005E78C2  
  
有的地方 EC 鉴于某些原因检测函数的第一个操作码:  
  
005F8C87    0FB600          MOVZX EAX,BYTE PTR DS:\[EAX\]  
005F8C8A    833C85 00004E00>CMP DWORD PTR DS:\[EAX\*4+4E0000\],0  
005F8C92  ^ 0F84 D4A2F1FF   JE EXECrypt.00512F6C  
005F8C98    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\] ; user32.FindWindowA  
005F8C9B    8038 CF         CMP BYTE PTR DS:\[EAX\],0CF  
005F8C9E  ^ 0F84 C1A2F1FF   JE EXECrypt.00512F65  
005F8CA4  ^ E9 8571FFFF     JMP EXECrypt.005EFE2E  
  
检测 API 是否被 HOOK 或做重定向处理:  
  
005EFE2E    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\] ; user32.FindWindowA  
005EFE31    8038 E9         CMP BYTE PTR DS:\[EAX\],0E9 ; Is JMP?  
005EFE34  ^ 0F84 2B31F2FF   JE EXECrypt.00512F65  
005EFE3A    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\] ; user32.FindWindowA  
005EFE3D    8038 EB         CMP BYTE PTR DS:\[EAX\],0EB ; Is JMP SHORT?  
005EFE40  ^ 0F84 1F31F2FF   JE EXECrypt.00512F65  
005EFE46    E9 38770100     JMP EXECrypt.00607583  
  
检测是否被涂改了补丁代码  RETN XXXX:  
  
004EB930    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\] ; user32.FindWindowA  
004EB933    8A00            MOV AL,BYTE PTR DS:\[EAX\]  
004EB935    24 F6           AND AL,0F6  
004EB937    3C C2           CMP AL,0C2  
0060CCB1  ^ 0F84 AE62F0FF   JE EXECrypt.00512F65  
  
查看第一条操作码 JMP DWORD \[EAX\] 是否被重定向:  
  
0060CCBA    66:8B00         MOV AX,WORD PTR DS:\[EAX\]  
0060CCBD    66:25 FF38      AND AX,38FF  
005F0C95    66:3D FF20      CMP AX,20FF  
005F0C99  ^ 0F84 C622F2FF   JE EXECrypt.00512F65  
  
检查 call xxxxxxxx 类型的重定向行为:  
  
00546361    8038 E8         CMP BYTE PTR DS:\[EAX\],0E8  
00546364    0F85 DBED0300   JNZ EXECrypt.00585145  
0054636A  ^ E9 5DF7FCFF     JMP EXECrypt.00515ACC  
  
一些更深入的研究是执行后调用 FindWindowA 函数寻找 "OLLYDBG" 窗口。OK,我们清零 EAX 后返回就可以避开这个检测了。  
  
EC 将会进一步的遍历所有进程来寻找是否被 Olly 调试。EC 使用 GetWindowThreadProcessId 来获取所有进程的 PID,然后依次使用 ReadProcessMomery 读取分析。(EC 对该 API 上有检测)我们不能在 ReadProcessMomery 上设置断点,我们可以使用内存断点的方法,但这样太慢了。实际上 ReadProcessMemory 函数也是调用 ZwReadVirtualMemory 函数的,我们可以在 ZwReadVirtualMemory 上设置断点避开 EC 的检测。现在我们可以 F9 运行了。当停在了我们设置的断点处,跳出系统领空并返回到 EC 的代码中跟踪检测代码(如果你在本次调用没有发现到检测,F9 后继续观察):  
  
005ABBE2    81BD 38FEFFFF 45>CMP DWORD PTR SS:\[EBP-1C8\],2B584245  
005ABBEC  ^ 0F85 8A8BFAFF    JNZ EXECrypt.0055477C  
005ABBF2  ^ E9 6371F6FF      JMP EXECrypt.00512D5A  
  
跟随到数据窗口中:  
  
005ABBE2 81 BD 38 FE FF FF 45 42 58 2B ..8...EBX+  
  
可以看出 EC 在检测 Olly 的字符 "EBX+"。我们把 2B584245 修改下(比如 12345678)让它的检测失效。EC 没有对此设置校验。  
  
如果查看 EC 的所有区段,你会发现有不少类似的检测。查找下 CMP DWORD\[EAX\],CONST 你就知道了:  
  
Found commands  
Address Disassembly Comment  
00514CDB CMP DWORD PTR DS:\[EAX\],47424454 ; TDBG  
0053A13B CMP DWORD PTR DS:\[EAX\],47424454  
0055DF80 CMP DWORD PTR DS:\[EAX\],4742444F ; ODBG  
0057FD31 CMP DWORD PTR DS:\[EAX\],47424454  
00599B86 CMP DWORD PTR DS:\[EAX\],4742444F  
005A6EC7 CMP DWORD PTR DS:\[EAX\],4742444F  
005AF1DF CMP DWORD PTR DS:\[EAX\],4742444F  
005BA618 CMP DWORD PTR DS:\[EAX\],47424454  
005C88BD CMP DWORD PTR DS:\[EAX\],47424454  
005C96FF CMP DWORD PTR DS:\[EAX\],4742444F  
005D77C5 CMP DWORD PTR DS:\[EAX\],4742444F  
005E0B5A CMP DWORD PTR DS:\[EAX\],4742444F  
005EC439 CMP DWORD PTR DS:\[EAX\],4742444F  
005EEDDE CMP DWORD PTR DS:\[EAX\],47424454  
005F6512 CMP DWORD PTR DS:\[EAX\],47424454  
006004FA CMP DWORD PTR DS:\[EAX\],47424454  
  
还有对 sysinternals tools 的检测:  
  
Found commands  
Address Disassembly Comment  
00525A2A CMP DWORD PTR SS:\[EBP-10C\],656C6946 ;File  
0056A41F CMP DWORD PTR SS:\[EBP-10C\],6D676552 ;Regm  
00593828 CMP DWORD PTR SS:\[EBP-10C\],6D676552  
005BE5A2 CMP DWORD PTR SS:\[EBP-10C\],76676264 ;dbgv  
005DF3F9 CMP DWORD PTR SS:\[EBP-10C\],36343831 ;1846  
005E95A7 CMP DWORD PTR SS:\[EBP-10C\],36343831  
0060BCE9 CMP DWORD PTR SS:\[EBP-10C\],76676264  
\- 多线程保护 -  
  
启用多线程是用来持续保护壳程序的。可以直接 NOP 掉整个 CreateThread API (保留 RETN xxxx)。  
  
\- OEP -  
  
EC 使用 Delphi 编写的,所以 OEP 应该在代码段的末尾处。但是 OEP 已经被"偷"了,并且原 OEP 处的代码已经被 EC 涂改得怪凌乱的。我们使用以前教程提及过的内存断点大法在代码段上设置断点。OEP 应该就是这里:  
  
004D2AB8  - E9 81560D00     JMP EXECrypt.005A813E  
004D2ABD    45              INC EBP  
004D2ABE    91              XCHG EAX,ECX ; EXECrypt.005A8100  
004D2ABF    3D 6F43525F     CMP EAX,5F52436F  
004D2AC4    35 92C3E6E9     XOR EAX,E9E6C392  
004D2AC9    DA87 0900D6AD   FIADD DWORD PTR DS:\[EDI+ADD60009\]  
004D2ACF    9E              SAHF  
004D2AD0    D7              XLAT BYTE PTR DS:\[EBX+AL\]  
...  
  
飞向光明之颠--OEP:  
  
005A813E    55              PUSH EBP  
005A813F  ^ E9 7C52FAFF     JMP EXECrypt.0054D3C0  
005A8144  ^ FF25 58D74D00   JMP DWORD PTR DS:\[4DD758\] ; EXECrypt.0051256B  
005A814A  ^ 0F88 670CF7FF   JS EXECrypt.00518DB7  
...  
  
现在我们开始修复 IAT。  
  
\- 修复 IAT -  
  
修复 IAT 在以前的教程里都有说明了。这里直接用脚本修复它们,然后使用 LordPE 转储文件,重建输入表即可。  
  
//-------------------------------- SCRIPT START -----------------------------------------  
//ExeCryptor 2.x IAT for asm/Delphi/BorlandC++ type - by haggar  
var addr  
var oep  
var pointer  
var counter  
var esp\_ref  
var temp  
  
mov addr,401000  
mov oep,eip  
  
LABEL\_01:  
find addr,#ff25????4D00#  
cmp $RESULT,0  
je END\_01  
mov addr,$RESULT  
add addr,2  
mov pointer,addr  
mov pointer,\[pointer\]  
mov pointer,\[pointer\]  
cmp pointer,10000000 //Check is import placed in thunk, or redirection.  
ja LABEL\_01  
cmp pointer,0 //For delphi!!!!!!!!!!!!!!!!  
je LABEL\_01  
  
sub addr,2  
mov eip,addr  
add addr,2  
  
mov esp\_ref,esp //Stack reference.  
mov counter,0  
LABEL\_02: //Trace some code.  
sti  
add counter,1  
cmp counter,30  
jne LABEL\_02  
  
mov temp,esp  
LABEL\_03: //Find referenced stack value.  
add temp,4  
cmp temp,esp\_ref  
jne LABEL\_03  
sub temp,4  
  
mov temp,\[temp\] //Go to "Magic address".  
bp temp  
esto  
bc eip  
  
mov temp,\[eip\]  
and temp,0ffff  
cmp temp,025ff //SelfWriting import type? No need to fix it then.  
je LABEL\_01  
  
cmp eax,10000000 //If EAX=!IMPORT, then it is a first type.  
jb LABEL\_01  
  
mov temp,addr //In this case EAX=IMPORT.  
mov temp,\[temp\]  
mov \[temp\],eax  
jmp LABEL\_01  
  
END\_01:  
mov eip,oep  
ret  
//------------------------- END SCRIPT ------------------------------------------------  
  
\- 脱壳修复 -  
  
使用 LordPE 修改 TLS 指针为 000E2000。处理后应该可以正常运行,可能还会有点小问题。最大的问题是可能出现一系列的异常,存在一定的兼容性问题(比如程序无法跨机器运行)。 EC 的脱壳修复方面可以通过分析官网的 Excryptor crackme 了解到更多。  
  
\- 结语 -  
  
没有了。