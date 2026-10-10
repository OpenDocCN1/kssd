

# ARMadillo V4.1 Copy MEM II Code Inj

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-557.htm`](https://www.pediy.com/kssd/pediy07/pediy7-557.htm)

【目     标】:自己写的一个小程序  
【工     具】:Olydbg1.1(diy 版)   
【任     务】:不脱壳直接 Inline patch   
【操作平台】:Windows 2003 server   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】: [`www.pediy.com/bbs/pediy7/pediy7-557.rar`](http://www.pediy.com/bbs/pediy7/pediy7-557.rar)  
【简要说明】： Armadillo 是加密强度好的应该就是 CopyMEM2+CC，我测试的目标没有 CC,因为有 CC 和没有 CC 对使用代码注入技术来破解来说没有很大的影响，我用 armv4.1 public 加的壳，对于 CopyMEM2 的加密方式的代码注入的前提是：你必须会脱 copymemII 的标准壳。我不打算在这篇文章里写上怎么脱壳，CodeInjection 之前其实差不多算是脱了一个壳。  
【详细过程】:  
OD 设置:忽略全部异常，加上 C000001E 这个异常范围。去除 OutputDebugStringA 这个 OD 漏洞。OD 载入目标，然后下断 BP DebugActiveProcess,Shift+F9 运行中断在该 API 入口处：  
77E602C8 >  E8 05DD0100     CALL <JMP.&ntdll.DbgUiConnectToDbg>      ; 中断在这里  
77E602CD    85C0            TEST EAX,EAX  
77E602CF    7D 0A           JGE SHORT 77E602DB  
断下后，在堆栈中看看新进程的进程 ID：  
0012BC98   0042FDEA  /CALL to DebugActiveProcess from Packed.0042FDE4  
0012BC9C   000006E8  \\ProcessId = 6E8  
0012BCA0   0012FF04  
新进程 ID 为 6E8(注意这个值，每次调试时是不同的，因此不能看死的),开一个新的 OD,附加该新进程，附加后，让父进程(也就是我们下断 API 的那个进程)继续运行，这样的话，字进程的入口会被父进程修改，在新进程的调试窗口里下断 bp OpenMutexA,然后让子进程运行，在这过程中，OD 会提示已经挂起，打开线程窗口让线程继续运行(我在 win2003 下要很久才会有反应的, 我是用 Process Explorer 来处理进程 )。中间会有一次异常提示，点确定后 SHIFT+F9 让程序继续运行，最后中断在 OpenMutexA 的入口处。中断后在一空白处写上，把双进程改为单进程的代码：  
pushad  
push edx      ;Mutex Name  
push 0  
push 0  
call CreateMutexA  
popad  
jmp OpenMutexA  
然后把 EIP 改为我们写的代码开始处，写完后取消该 API 入口处的断点，然后下断 BP SetFilePointer,F9 运行-->中断-->返回程序代码。  
返回后直接向下找到这里：  
00AC4384    50              PUSH EAX                                 ; 向下找到这里  
00AC4385    FF15 0C32AD00   CALL DWORD PTR DS:\[AD320C\]               ; kernel32.OutputDebugStringA  
00AC438B    C705 7090AD00 D>MOV DWORD PTR DS:\[AD9070\],0AD99D0        ; ASCII "CC6"  
00AC4395    8B0D 20CFAD00   MOV ECX,DWORD PTR DS:\[ADCF20\]            ; Packed.00464370  
00AC439B    8B41 70         MOV EAX,DWORD PTR DS:\[ECX+70\]  
00AC439E    3341 64         XOR EAX,DWORD PTR DS:\[ECX+64\]  
00AC43A1    3341 60         XOR EAX,DWORD PTR DS:\[ECX+60\]  
00AC43A4    3345 D8         XOR EAX,DWORD PTR SS:\[EBP-28\]            ; \[EBP-28\]处保存正确的 CRC 值 Patch 点 0 \*\*\*\*  
00AC43A7    8985 C8F7FFFF   MOV DWORD PTR SS:\[EBP-838\],EAX  
  
\[EBP-28\]=\[0012D6FC\]=94B6086A     ;CRC Value  
  
PATCH 点 0 为 00AC43A4  
  
同时记下这个段是的起始地址，然后计算这个 PATCH 地址的相对地址，这里计算出来的值大小为 243A1，  
我们要知道这个段是什么时候动态申请的,这个实际上和我上一篇的单进程的找这个申请段的方法一样。  
转成单进程后，直接下断 bp VirtualAlloc,第二次就是申请 这个段。也就是下面这个地方：  
  
0042B223   .  8945 E0       MOV DWORD PTR SS:\[EBP-20\],EAX            ;第二次申请后返回这里,这里 patch 点 1 \*\*\*\*  
0042B226   >  837D E0 00    CMP DWORD PTR SS:\[EBP-20\],0  
  
返回后根据单进程的 CODE Injection 那篇文章，直接 CTRL+S 查找以下命令：  
PUSH EBP  
MOV EBP,ESP  
SUB ESP,5C  
会找到以下几个地方：  
0040732E    55              PUSH EBP  
0040732F    8BEC            MOV EBP,ESP  
00407331    83EC 5C         SUB ESP,5C  
00407334    833D C0424600 0>CMP DWORD PTR DS:\[4642C0\],0              ; 这里就是我们将要 PATCH 的地方,这里 PATCH 点 2 \*\*\*\*  
0040733B    0F85 54010000   JNZ 00407495  
......  
0042A8BB    55              PUSH EBP  
0042A8BC    8BEC            MOV EBP,ESP  
0042A8BE    83EC 5C         SUB ESP,5C  
0042A8C1    A1 BC434600     MOV EAX,DWORD PTR DS:\[4643BC\]  
0042A8C6    3305 7C434600   XOR EAX,DWORD PTR DS:\[46437C\]  
......  
00439BAC    55              PUSH EBP  
00439BAD    8BEC            MOV EBP,ESP  
00439BAF    83EC 5C         SUB ESP,5C  
00439BB2    C745 E8 FFFFFFF>MOV DWORD PTR SS:\[EBP-18\],-1  
  
第一次找到的地方就是我们的 PATCH 点 2  
  
现在用 LORDPE 看看以那两个 PATCH 点发现，代码是加密后保存到文件的，再次重来，在 0042B223 处下内存访问和写入断点，下访问断点的原因是考虑壳可能会用到这里的代码解别处的代码，当然实际是上是没有的。下断后，运行就会停在以下位置：  
0045A499    D20B            ROR BYTE PTR DS:\[EBX\],CL                 ; 第一次中断这里,记下 ECX 的值先  
0045A49B    300B            XOR BYTE PTR DS:\[EBX\],CL  
0045A49D    43              INC EBX  
0045A49E    49              DEC ECX  
0045A49F    85C9            TEST ECX,ECX  
0045A4A1  ^ 75 F6           JNZ SHORT 0045A499  
......  
0045A5F5    3003            XOR BYTE PTR DS:\[EBX\],AL                 ; 第二次中断,这里记下 AL 的值  
0045A5F7    43              INC EBX  
0045A5F8    49              DEC ECX  
0045A5F9    85C9            TEST ECX,ECX  
0045A5FB  ^ 75 F8           JNZ SHORT 0045A5F5  
0045A5FD    8B85 FA950000   MOV EAX,DWORD PTR SS:\[EBP+95FA\]          ; 到了这里程序就全部解压完了  
  
解密算法很简单，为这样子:  
   ROR BYTE PTR \[CKEY\],9C(这两个值是可变的)  
   XOR byte ptr \[CKEY\],DD  
   XOR byte ptr \[CKEY\],AL(AL 总是为 0B0)  
  
解密算法为：  
   XOR BYTE PTR \[DKEY\],0B0  
   XOR BYTE PTR \[DKEY\],CL  
   ROL BYTE PTR \[DKEY\],CL  
  
Patch 点 1 解压时 ECX=0001A1DD  
Patch 点 2 解压时 ECX=0003E0CC  
  
现在壳的 CRC 的 PATCH 点已经找全了，现在开始找程序的真正要破解的地方，OD 载入重来，先 F9 运行，OD 会报有异常的提示，点确定，然后下断 bp WriteProcessMemory,这时会中断几次：  
0012B994   004343D6  /CALL to WriteProcessMemory from Packed.004343D0  
0012B998   00000044  |hProcess = 00000044  
0012B99C   00454000  |Address = 454000  
0012B9A0   0012BC84  |Buffer = 0012BC84  
0012B9A4   00000002  |BytesToWrite = 2  
0012B9A8   0012BC88  \\pBytesWritten = 0012BC88  
......  
0012B994   004343FE  /CALL to WriteProcessMemory from Packed.004343F8  
0012B998   00000044  |hProcess = 00000044  
0012B99C   00454000  |Address = 454000  
0012B9A0   0046B20C  |Buffer = Packed.0046B20C  
0012B9A4   00000002  |BytesToWrite = 2  
0012B9A8   0012BC88  \\pBytesWritten = 0012BC88  
......  
0012BB34   00433EA8  /CALL to WriteProcessMemory from Packed.00433EA2  
0012BB38   00000044  |hProcess = 00000044  
0012BB3C   00401000  |Address = 401000  
0012BB40   003A1F30  |Buffer = 003A1F30  
0012BB44   00001000  |BytesToWrite = 1000 (4096.)  
0012BB48   0012BC50  \\pBytesWritten = 0012BC50  
  
第三个 WriteProcessMemory 时说明壳已经解压出正确程序的代码，\[ESP+8\]为程序的 CODE Section 的 VA，\[ESP+C\]为与 CODE Section 对应的 VA，因为我们已经有源代码所以不用脱壳就知道改哪里了^\_^:  
003A1F30    BB 01000000      MOV EBX,1  
003A1F35    83FB 01          CMP EBX,1                                ; 我选择这里为 PATCH 点,改成 cmp ebx,0 程序 Crack 点$$$$  
003A1F38    75 15            JNZ SHORT 003A1F4F  
003A1F3A    6A 00            PUSH 0  
  
程序 CRAK 点为 003A1F35  
过了这个 WriteProcessMemory 的话，程序已经把代码复制过去了，再改就没有意思了，因此，我们要在没有复制代码过去之前修改它，ALT+F9 返回到程序代码：  
00433E89   .  52             PUSH EDX                                 ; /pBytesWritten  
00433E8A   .  68 00100000    PUSH 1000                                ; |BytesToWrite = 1000 (4096.)  
00433E8F   .  A1 34B34600    MOV EAX,DWORD PTR DS:\[46B334\]            ; |  
00433E94   .  50             PUSH EAX                                 ; |Buffer => 003A1F30  我选择这里作为 Patch 点 3\*\*\*\*  
00433E95   .  8B4D EC        MOV ECX,DWORD PTR SS:\[EBP-14\]            ; |  
00433E98   .  51             PUSH ECX                                 ; |Address  
00433E99   .  8B15 08B34600  MOV EDX,DWORD PTR DS:\[46B308\]            ; |  
00433E9F   .  8B02           MOV EAX,DWORD PTR DS:\[EDX\]               ; |  
00433EA1   .  50             PUSH EAX                                 ; |hProcess  
00433EA2   .  FF15 10414600  CALL DWORD PTR DS:\[<&KERNEL32.WriteProce>; \\WriteProcessMemory  
00433EA8   .  85C0           TEST EAX,EAX                             ;  返回到这里  
00433EAA   .  75 4B          JNZ SHORT 00433EF7  
  
Patch 点 3 的为：00433E94,同样这里的代码也是动态还原的，加密方法和上面的一下，再次重来，然后记下 ECX 的值.  
ECX==0001156C  
到这里，全部的 patch 点都找回来了，自己动手在相关的位置写上自己的代码。动态还原的那几处加密后的值，自己去算算。现在动手写一下以下代码就可以了：  
  
  
0045DEAE    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045DEB0    0000            ADD BYTE PTR DS:\[EAX\],AL                 ; 这里保存动态申请的那个段起始地址  
0045DEB2    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045DEB4    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045DEB6    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045DEB8    0000            ADD BYTE PTR DS:\[EAX\],AL  
0045DEBA    90              NOP  
0045DEBB    90              NOP  
0045DEBC    90              NOP  
0045DEBD    90              NOP  
0045DEBE    90              NOP  
0045DEBF    90              NOP  
0045DEC0    50              PUSH EAX                                 ; patch 点 1 的 Patch 代码  
0045DEC1    A3 B0DE4500     MOV DWORD PTR DS:\[45DEB0\],EAX            ; 保存动态申请的地址  
0045DEC6    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0045DECA    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0045DECD    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
0045DECF    C700 8945E083   MOV DWORD PTR DS:\[EAX\],83E04589          ; 还原代码防止壳检测  
0045DED5    C640 04 7D      MOV BYTE PTR DS:\[EAX+4\],7D  
0045DED9    53              PUSH EBX                                 ; 修改 patch 点 2 的代码  
0045DEDA    BB 00DF4500     MOV EBX,0045DF00  
0045DEDF    B8 34734000     MOV EAX,00407334  
0045DEE4    2BD8            SUB EBX,EAX  
0045DEE6    83EB 05         SUB EBX,5  
0045DEE9    C600 E8         MOV BYTE PTR DS:\[EAX\],0E8  
0045DEEC    8958 01         MOV DWORD PTR DS:\[EAX+1\],EBX  
0045DEEF    5B              POP EBX                                  ; 修改完跳回去执行原代码  
0045DEF0    58              POP EAX  
0045DEF1    C3              RETN  
0045DEF2    90              NOP  
0045DEF3    90              NOP  
0045DEF4    90              NOP  
0045DEF5    90              NOP  
0045DEF6    90              NOP  
0045DEF7    90              NOP  
0045DEF8    90              NOP  
0045DEF9    90              NOP  
0045DEFA    90              NOP  
0045DEFB    90              NOP  
0045DEFC    90              NOP  
0045DEFD    90              NOP  
0045DEFE    90              NOP  
0045DEFF    90              NOP  
0045DF00    50              PUSH EAX                                 ; Patch 点 2 的代码  
0045DF01    A1 B0DE4500     MOV EAX,DWORD PTR DS:\[45DEB0\]  
0045DF06    53              PUSH EBX  
0045DF07    BB 40DF4500     MOV EBX,0045DF40  
0045DF0C    05 A1430200     ADD EAX,243A1                            ; 修改 Patch 点 0 的代码  
0045DF11    2BD8            SUB EBX,EAX  
0045DF13    83EB 05         SUB EBX,5  
0045DF16    C600 E8         MOV BYTE PTR DS:\[EAX\],0E8  
0045DF19    8958 01         MOV DWORD PTR DS:\[EAX+1\],EBX  
0045DF1C    5B              POP EBX  
0045DF1D    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0045DF21    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0045DF24    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
0045DF26    C700 833DC042   MOV DWORD PTR DS:\[EAX\],42C03D83          ; 还原代码防止壳检测  
0045DF2C    C640 04 46      MOV BYTE PTR DS:\[EAX+4\],46  
0045DF30    58              POP EAX  
0045DF31    C3              RETN  
0045DF32    90              NOP  
0045DF33    90              NOP  
0045DF34    90              NOP  
0045DF35    90              NOP  
0045DF36    90              NOP  
0045DF37    90              NOP  
0045DF38    90              NOP  
0045DF39    90              NOP  
0045DF3A    90              NOP  
0045DF3B    90              NOP  
0045DF3C    90              NOP  
0045DF3D    90              NOP  
0045DF3E    90              NOP  
0045DF3F    90              NOP  
0045DF40    50              PUSH EAX                                 ; Patch 点 0 处的处理代码  
0045DF41    C745 D8 6A08B69>MOV DWORD PTR SS:\[EBP-28\],94B6086A  
0045DF48    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0045DF4C    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0045DF4F    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
0045DF51    C700 33416033   MOV DWORD PTR DS:\[EAX\],33604133          ; 还原并执行原代码  
0045DF57    C640 04 45      MOV BYTE PTR DS:\[EAX+4\],45  
0045DF5B    58              POP EAX  
0045DF5C    C3              RETN  
  
修改完毕，运行一下，YUP，搞定了。patch 成功后看了一下 arm v4.1 的主程序，发现 patch 方法有所不同，可惜我不会破解主程序，所以我看也没看懂:-(,哪位知道的指教一下俺。  
  
  
  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
  
By loveboom\[DFCG\]\[FCG\]\[US\]  
http://blog.csdn.net/bmd2chen  
Email:loveboom#163.com  
Date:5/11/2005 3:38:00 PM 

 4dd466d0e1e0cc123afbe504da2462ca