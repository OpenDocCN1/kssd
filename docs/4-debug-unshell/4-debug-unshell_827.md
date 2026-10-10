

# hying 0.7x 的脱壳过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-580.htm`](https://www.pediy.com/kssd/pediy07/pediy7-580.htm)

;前两天拿到一个外挂,还不错,就想研究一下. 遂欲脱壳.  
;按照 peid 的检测结果,这应该是一个某人 0.7x 的壳. 我没用过那个壳,不知道到底是不是 0.7x,不过跟踪起来,发现与以前的老版本区别似乎不是很大.不过这个帖子发在 pediy 不知道会不会被删 ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)   
;跟踪的流程大致如下  
;============================================  
;1.在程序段的壳中运行,一路很多类似于"add reg, 4";"xor \[reg\], key" 的代码,把后面的代码进行解码. 这一段直接 F4 过去就是了.   
;最后 VirtualAlloc 申请一段内存,把解码后的壳代码放入. 然后跳到缓冲区执行. 这一步基本上对调试没有任何的阻力.  
;============================================  
;2.在缓冲区中执行. 一段很长的垃圾 seh 代码,对解码没有任何实质作用,仅仅是反跟踪. 我们直接找:MOV DWORD PTR FS:\[0\],ESP"这样的用来设置 seh 函数的语句,  
;一路上 F4 过去就是.  
;============================================  
;3.在这个版本的壳中,最巧妙的就是它的 4 个关键 SEH. 接下来我们遇到了第一个.  
;这个 seh 对反跟踪效果非常的好. 它首先利用一个 int3 中断进入 seh, 设置 4 个单步断点于 dr 寄存器中. 然后分别在四个单步中断处完成解码的相应操作,  
;最后经过一个除 0 异常进入 seh,设置好 dr 寄存器的值后,拆除 seh.  
;    我们如果在 4 个单步断点之间进行软硬件中断调试,是不太可行的.   
;    a.软件: 很多代码没有解密,无法对其进行软件中断  
;    b.硬件: dr 寄存器被占用,根本无法使用硬件中断  
;所以,我们唯一的办法只有在它最后一次进入 seh 之后的代码上下软件断点"CMP EAX,CD000094". 幸好,这段代码部分没有加密,不然就根本无法跟踪下去了.  
;然后,出第一个 seh 中断. 这个时候,我们必须把 dr 寄存器中的值记录下来,因为在第二个 seh 中要用到. 除非,你在第一个和第二个 seh 之间不进行中断.  
;============================================  
;4.经过了第一个 seh,长征才刚刚开始.接下来我们来到了第二个 seh.  
;这个 seh 就是利用上个 seh 最后的寄存器值,对一段代码进行解码. 在一个循环当中,n 次进入这个 seh. 没什么太大的难度,头一次进入这个 seh 时  
;恢复好 dr 寄存器值,然后找到拆除这个 seh 的代码,软件中断,F9 过去就是了.  
;============================================  
;5.接下来,壳要对原始程序进行解压缩  
;    a. 它利用一段 20 字节的代码对原始程序进行解压缩,而这 20 字节的内容是加密过的. 所以首先的把这 20 字节内容给解密了.  
;    在这个过程中,它用到了一个全局变量--在全局 heap\_addr+0c 处的一个字节. 它用这个字节来对上面提到的 20 字节内容进行了解码.  
;    b. 然后,对原始程序进行解码.  
;    c. 再用那 20 字节,对解码后的原始程序进行解压缩. 完毕.  
;============================================  
;6.然后是对 iat 的处理.  
;经过壳的 iat 处理,形成了下面的一个调用过程.  
;  
;iat 中地址    -->    |Hook\_proc:  
;Hook\_proc        |PUSH DWORD PTR DS:\[Hook\_proc+1C\]  
;            |XOR DWORD PTR SS:\[ESP\],key  
;            |ret;                    ->    |Stub\_proc:  
;            |                        |api\_start\_code  
;            |Hook\_proc+1C:                    |...  
;            |Stub\_proc\_xor\_key                |api\_some\_code  
;                                    |push api\_next\_code\_addr  
;                                    |ret  
;经过我们对壳代码的修改,应该使其变成下面的一个形式  
;  
;iat 中地址    -->    |Hook\_proc:  
;Hook\_proc        |PUSH DWORD PTR DS:\[Hook\_proc+1C\]  
;            |XOR DWORD PTR SS:\[ESP\],key  
;            |ret;                    ->    |Stub\_proc:  
;            |                        |push api\_start\_addr  
;            |Hook\_proc+1C:                    |ret  
;            |Stub\_proc                      
;  
;那么我们就可以通过自己编写一段代码来遍历 iat, 找回每一个 api\_start\_addr  
;  
;============================================  
;7.对程序文件的校验  
;这个没什么好说的,两个值改成相等就行  
;============================================  
;8.对 mainform 的处理  
;壳把 mainform 的数据移到壳当中去了.包括 VMT 和 Field\_Table,Class\_Info 之类的 delphi 数据结构. 我们需要把它移回原始程序代码段.   
;详见 hexer 的脱壳文章.我这里只有过程而没有讲原因.  
;============================================  
;9.对代码段中部分的 "mov 寄存器, \[地址 A\]"语句的替换.  
;我是在所有其他工作都做完后,脱壳发现运行不了,才注意到壳还有这么一手的.  
;    它把代码段中部分的 "mov 寄存器, \[地址 A\]"语句给替换成了"call \[地址 B\]", 语句都为 6 个字节长度  
;    地址 B 处于 400000--401000 文件头部范围内. 而在 地址 B 处是这样的代码 " jmp \[地址 C\]"  
;    地址 C 处则是原来的"mov 寄存器, \[地址 A\]""ret"  
;这个也好办,在 oep 处,写一段代码,遍历程序代码段,替换回来就行了. 它能弄过去,我们就能弄回来  
;============================================  
;10.第三个 seh  
;这个 seh 没有什么特别的,只是需要把第二个 seh 最后的 dr 寄存器值保留下来到这里,不然会出错.  
;============================================  
;11.第四个 seh. 这也是壳中最精彩的一个 seh  
;没有用到前面的 seh 留下来的 dr 寄存器值了.但其中通过修改 eip 的值,来控制程序的走向--从一个异常走向另一个异常. 一共经过了 7 个异常(另外一个程序好像是 8 个),  
;才拆掉此 seh. 对付此 seh 的办法是在此 seh 返回的语句下软件断点,然后每次运行到此处时,查看 TIB 中的 regEIP 值,如果 seh 出去的地方附近有拆除 seh 的代码,就跟出去,  
;如果没有,还是不要浪费时间了.  
;============================================  
;12.stolen code  
;写过 delphi 程序的人都知道,delphi 程序有一个框架. Application.Initialize; Application.CreateForm; Application.run. (好像是这样的,不太记得了)  
;hying 壳就是把那个框架代码给偷到壳里头来了. 可能是因为这个框架每个 delphi 程序都类似,所以比较好偷.  
;这个其实也很简单,找到壳内相当于框架入口(也就是 oep)的代码位置,把它后面的代码等效还原就好了. 然后我们直接跳到框架处,也就是 oep 处执行.  
;  
;ok 原理就到这里了,下面我们开始脱...  
  
;1.在程序段的壳中运行  
007970F4    5D              POP EBP                                  ; main.00797000  
  
007971A7    E8 94010000     CALL main.00797340            ;代码解密函数么?  
  
00797162    FFE2            JMP EDX                ;跳到 370000  
;2.进入缓冲区执行  
00370018    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]        ;开始一段很长的 seh,没有任何实质作用  
0037001F    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
00370026    74 03           JE SHORT 0037002B  
  
  
003709CA    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
0037132F    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
0037191C    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
00372269    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
00372A2E    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
00372B8D        64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
00372C8F        E8 03000000     CALL 00372C97  
  
00372CF0    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
  
00372D55    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]  
;垃圾 SEH 结束  
  
;这里遇到了一个 SEH 异常  
003744A8    F8              CLC  
;堆栈  
;0012FF9C  /0012FFE0  Pointer to next SEH record  
;0012FFA0  |003733AC  SE handler  
;调试寄存器  
;DR0 003744A8  
;DR1 0037473C  
;DR2 003748E5  
;DR3 00374A7E  
;DR6 FFFF0FF0  
;DR7 00000555  
  
;3733AC ★第一个 SEH 函数================================================  
  
003733F9    3D 03000080     CMP EAX,80000003  
003733FE    0F85 A9020000   JNZ 003736AD  
  
003736AD    3D 1D0000C0     CMP EAX,C000001D  
003736B2    0F85 8E010000   JNZ 00373846  
  
00373846    3D 04000080     CMP EAX,80000004  
0037384B    0F85 B6050000   JNZ 00373E07  
  
00373E07    3D 940000CD     CMP EAX,CD000094                ;这里应该是最后一次进入这个 SEH 函数,除 0 错误  
00373E0C    23ACAB 121617A9 AND EBP,DWORD PTR DS:\[EBX+EBP\*4+A9171612>  
...  
00373F9E    C3              RETN  
;结束时,寄存器的值  
;FLAG      00010017  
;dr0       0FFF0123  
;dr1       0FFF4567  
;dr2       0FFF89AB  
;dr3       0FFFCDEF  
;dr6       FFFF0FF0  
;dr7       00000555  
;EIP    374C25  
  
;第一个 SEH 函数结束跳到这里  
00374C26    64:8F05 0000000>POP DWORD PTR FS:\[0\]                      
00374C2D    58              POP EAX  
  
00374C4B    33D2            XOR EDX,EDX                              
00374C4D    F7E6            MUL ESI  
00374C4F    05 78563412     ADD EAX,12345678  
00374C54    83D2 00         ADC EDX,0  
00374C57    F7F3            DIV EBX  
00374C59    58              POP EAX  
00374C5A    32C2            XOR AL,DL  
00374C5C    50              PUSH EAX  
00374C5D    4C              DEC ESP                    ;<--用 ESP 当做指针  
00374C5E    8BC2            MOV EAX,EDX  
00374C60  ^ E2 E9           LOOPD SHORT 00374C4B            ;对后面一段代码解码 374C62--...  
00374C62    8BE7            MOV ESP,EDI                    ;恢复 ESP  
;========================================================  
00374C64    E8 5B030000     CALL 00374FC4                ;准备安装 SEH 函数(★第二个 SEH 函数)  
;★第二个 SEH 函数  
00374C69    8B4C24 0C       MOV ECX,DWORD PTR SS:\[ESP+C\]        ;其中要对 dr 寄存器进行操作,所以之前要恢复到第一个 SEH 函数最后的 dr 寄存器值  
00374C6D    9C              PUSHFD  
00374C6E    6A 03           PUSH 3  
...  
00374CC7    8B81 B0000000   MOV EAX,DWORD PTR DS:\[ECX+B0\]        ;regEAX  
00374CCD    8B51 04         MOV EDX,DWORD PTR DS:\[ECX+4\]        ;dr0  
  
00374CFD    F6D0            NOT AL  
00374D16    32C2            XOR AL,DL  
00374D1F    66:25 FF00      AND AX,0FF  
00374D4C    66:03D0         ADD DX,AX  
00374D7A    66:C1CA 03      ROR DX,3  
00374DA8    66:8951 04      MOV WORD PTR DS:\[ECX+4\],DX            ;设置 dr0  
  
00374DD6    66:3151 08      XOR WORD PTR DS:\[ECX+8\],DX            ;dr1  
;出第二个 SEH 函数  
  
  
00374FC4    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]  
00374FCB    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
00374FD2    8DB5 C5414000   LEA ESI,DWORD PTR SS:\[EBP+4041C5\]  
00374FD8    B9 1F050000     MOV ECX,51F  
00374FDD    8A0431          MOV AL,BYTE PTR DS:\[ECX+ESI\]  
00374FE0    CC              INT3  
00374FE1    90              NOP  
00374FE2    880431          MOV BYTE PTR DS:\[ECX+ESI\],AL  
00374FE5  ^ E2 F6           LOOPD SHORT 00374FDD            ;循环. 实际上是利用第二个 SEH 函数对一段代码解码  
00374FE7    64:8F05 0000000>POP DWORD PTR FS:\[0\]            ;37550D--375A2C  
00374FEE    58              POP EAX  
;最后一次出第二个 SEH 函数,调试寄存器的值为  
;DR0 0FFF0195  
;DR1 0FFF35B6  
;DR2 0FFF8C3D  
;DR3 0FFFCDEF  
;DR6 FFFF0FF0  
;DR7 00000555  
  
...  
;接下来开始出现代码入栈执行  
...  
;下面要取得一些函数的地址,看来是壳要用到=============================  
0037514C    8DB5 94184100   LEA ESI,DWORD PTR SS:\[EBP+411894\]  
00375152    56              PUSH ESI                    ;ESI 382BDC ASCII "KERNEL32.dll"  
  
00375173    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
  
0037517F   /E9 3AD00000     JMP 003821BE                ;看来这里就是'调用 jmp'了  
  
  
00375331   /E9 88CE0000     JMP 003821BE                ;调用 38290F,类似于 GetProcAddress  
;堆栈  
;0012FF94   00375337  
;0012FF98   7C800000  kernel32.7C800000  
;0012FF9C   003830E4  ASCII "CloseHandle"  
;0012FFA0   00000022  
  
0037533B    8903            MOV DWORD PTR DS:\[EBX\],EAX  
0037533D    03D9            ADD EBX,ECX  
0037533F    43              INC EBX  
00375340    59              POP ECX  
00375341  ^\\E2 A2           LOOPD SHORT 003752E5            ;下一个函数  
;这里的函数列表: CloseHandle, CreateFileA, CreateFileMappingA, CreateThread, DeleteFileA, ExitProcess, FindResourceA...  
  
00375343    8DB5 E4174100   LEA ESI,DWORD PTR SS:\[EBP+4117E4\]  
;Address=00382B2C, (ASCII "USER32.dll")  
003754B9    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
003754C5   /E9 F4CC0000     JMP 003821BE                ;这里没有 Get 到...  
  
003754F0    8B85 44174100   MOV EAX,DWORD PTR SS:\[EBP+411744\]        ; kernel32.LoadLibraryA  
003754FC   /E9 BDCC0000     JMP 003821BE  
;下面又是和上面的 kernel32 相同,从中取得一些 api 的地址  
...  
00375598    49              DEC ECX  
00375599  ^ 0F85 6FFFFFFF   JNZ 0037550E                ;下一个函数          
  
0037559F    8DB5 BA1A4100   LEA ESI,DWORD PTR SS:\[EBP+411ABA\]  
  
...  
00375599  ^\\0F85 6FFFFFFF   JNZ 0037550E                ;还单独取得了 wsprintfA 的地址,为什么?  
  
0037559F    8DB5 BA1A4100   LEA ESI,DWORD PTR SS:\[EBP+411ABA\]  
;Address=00382E02, (ASCII "WS2\_32.DLL")  
...  
003757C9  ^\\E2 A1           LOOPD SHORT 0037576C  
003757CB    8DB5 E31A4100   LEA ESI,DWORD PTR SS:\[EBP+411AE3\]  
;Address=00382E2B, (ASCII "advapi32.dll")  
...  
003759F2    59              POP ECX  
003759F3  ^ E2 A1           LOOPD SHORT 00375996  
  
;======================================================  
00375A15    8B85 5E194100   MOV EAX,DWORD PTR SS:\[EBP+41195E\]        ; kernel32.GetCurrentThread  
  
00375A4B    8B85 321A4100   MOV EAX,DWORD PTR SS:\[EBP+411A32\]        ; kernel32.SetThreadPriority  
  
00375AE0    8B85 5C174100   MOV EAX,DWORD PTR SS:\[EBP+41175C\]        ;400000  
00375AE6    0340 3C         ADD EAX,DWORD PTR DS:\[EAX+3C\]        ;pe 头  
  
00375DC0    8B85 96194100   MOV EAX,DWORD PTR SS:\[EBP+411996\]        ; kernel32.GetProcessHeap    噢,看来要开始对原程序解码了  
;返回值 00140000  
00375DFD    8A40 0C         MOV AL,BYTE PTR DS:\[EAX+C\]            ;  
;DS:\[0014000C\]=62 ('b')        又到了这个解码的关键地方了,难道还要把 14000C 改为 02 么?  
00375E2A    24 40           AND AL,40  
00375E56    F6D0            NOT AL  
00375E82    8DB5 591D4100   LEA ESI,DWORD PTR SS:\[EBP+411D59\]  
00375EB2    B9 20000000     MOV ECX,20  
00375F0A    3006            XOR BYTE PTR DS:\[ESI\],AL  
00375F35    46              INC ESI  
00375F60  ^\\E2 80           LOOPD SHORT 00375EE2            ;对这一段 20byte 的数据解码,然后用这一段数据作为原程序代码解码的 key  
  
00375F86    8DB5 9A1C4100   LEA ESI,DWORD PTR SS:\[EBP+411C9A\]  
00375F8C    46              INC ESI  
  
;开始检测 debugger=============================================  
003760E4    51              PUSH ECX                    ;9  
003760E5    56              PUSH ESI                    ;00382FE3  
003760E6    AC              LODS BYTE PTR DS:\[ESI\]  
  
003760E9    F6D0            NOT AL                    ;噢~ \\\\.\\NTICE  
003760EB    8846 FF         MOV BYTE PTR DS:\[ESI\-1\],AL  
003760EE    AC              LODS BYTE PTR DS:\[ESI\]  
003760EF    0AC0            OR AL,AL  
003760F1  ^ 75 F6           JNZ SHORT 003760E9  
  
00376154    8B85 AF184100   MOV EAX,DWORD PTR SS:\[EBP+4118AF\]        ; kernel32.CreateFileA  
  
003761F6    F6D0            NOT AL  
003761F8    8846 FF         MOV BYTE PTR DS:\[ESI\-1\],AL  
003761FB    AC              LODS BYTE PTR DS:\[ESI\]  
003761FC    0AC0            OR AL,AL  
003761FE  ^ 75 F6           JNZ SHORT 003761F6                ;又把\\\\.\\NTICE 加密了  
  
                push 9  
                pop ecx  
0037628C    0BC9            OR ECX,ECX  
0037628E  ^ 0F85 50FEFFFF   JNZ 003760E4  
  
;准备调用 ZwSetInformationThread==============================  
00376B52    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
;堆栈  
;0012FF9C   00376B64    'ntdll.dll'  
;0012FFA0   00376B28  RETURN to 00376B28 from 00376B32  
  
00376CFC   /E9 BDB40000     JMP 003821BE                ;调用 38290F  
;堆栈  
;0012FF98   00376D02  'ZwSetInformationThread'  
;0012FF9C   7C920000  ntdll.7C920000  
;0012FFA0   00376CB8  RETURN to 00376CB8 from 00376CCF      
  
00376EF5    8B85 5E194100   MOV EAX,DWORD PTR SS:\[EBP+41195E\]        ; kernel32.GetCurrentThread  
  
00376F17   /E9 13B80000     JMP 0038272F                ;调用 ZwSetInformationThread?  
;堆栈  
;0012FF90   00376F1D  
;0012FF94   FFFFFFFE  
;0012FF98   00000011  
;0012FF9C   00000000  
;0012FFA0   00000000  
  
;准备开始对程序代码解码====================================  
00376F1D    8D1D 3F204100   LEA EBX,DWORD PTR DS:\[41203F\]  
00376F23    833C2B 00       CMP DWORD PTR DS:\[EBX+EBP\],0  
00376F3E  ^\\73 EF           JNB SHORT 00376F2F  
00376F2F   /0F84 4F0C0000   JE 00377B84                    ;解码完毕,从这里跳走  
  
00376FE8    03B5 5C174100   ADD ESI,DWORD PTR SS:\[EBP+41175C\]        ; main.00400000  
0037701B    8BFE            MOV EDI,ESI                              ; main.00401000  
  
003770F9    E8 0AA60000     CALL 00381708                ;调用原程序代码解码函数  
  
00377306    8B85 54174100   MOV EAX,DWORD PTR SS:\[EBP+411754\]        ; kernel32.VirtualAlloc  
;返回值 C90000  
  
0037792E   /E9 8BA80000     JMP 003821BE                ;调用 00797165, 解压缩函数  
;堆栈  
;0012FF98   00377934  
;0012FF9C   00382A98  
;0012FFA0   00C90000    解压到 C90000  
  
00377934   /EB 49           JMP SHORT 0037797F                ;返回到这里  
  
003779B5    51              PUSH ECX  
003779B6    C1E9 02         SHR ECX,2  
003779B9    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
;ECX=000C5C00 (decimal 809984.)  
;DS:\[ESI\]=\[00C90000\]=00000000  
;ES:\[EDI\]=\[00401000\]=0000DCD8  
003779BB    59              POP ECX  
003779BC    83E1 03         AND ECX,3  
003779BF    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\];把解压缩后的数据移回代码段  
  
00377AEB    8B85 58174100   MOV EAX,DWORD PTR SS:\[EBP+411758\]                 ; kernel32.VirtualFree  
  
00377B58    83C3 0C         ADD EBX,0C  
;EBX=0041203F (main.0041203F)  
;=============================================================  
;程序原代码解压缩完毕,跳到这里  
00377B84    8DB5 0B6A4000   LEA ESI,DWORD PTR SS:\[EBP+406A0B\]  
  
00377CD9    87E6            XCHG ESI,ESP            ;又准备用 ESP 进行解码了, 377D54--3800B7  
00377CDB    B9 64830000     MOV ECX,8364  
                pop eax  
00377CE7    F6D0            NOT AL  
00377CF1    83C4 04         ADD ESP,4  
00377CF4    50              PUSH EAX  
00377CFB    44              INC ESP  
00377D01  ^\\E2 DD           LOOPD SHORT 00377CE0  
00377D03    87E6            XCHG ESI,ESP            ;解码完毕到这里  
  
;下面开始处理 iat=======================================  
00377D33    8B85 54174100   MOV EAX,DWORD PTR SS:\[EBP+411754\]        ; kernel32.VirtualAlloc  
;返回值 EAX=C90000  
  
003788C7    0BFF            OR EDI,EDI                    ;6  
003788C9    75 05           JNZ SHORT 003788D0                  
003788CB    E9 8F370000     JMP 0037C05F                ;iat 处理完毕后这里跳出去  
  
00378A45    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
;堆栈  
;0012FF9C   00378A58  
;0012FFA0   003837A4  ASCII "advapi32.dll"  
  
00379194    33C0            XOR EAX,EAX  
00379196    AC              LODS BYTE PTR DS:\[ESI\]  
00379197    EB 07           JMP SHORT 003791A0  
00379199    C0C0 03         ROL AL,3  
0037919C    F6D0            NOT AL  
0037919E    AA              STOS BYTE PTR ES:\[EDI\]  
0037919F    AC              LODS BYTE PTR DS:\[ESI\]  
003791A0    0BC0            OR EAX,EAX  
003791A2  ^ 75 F5           JNZ SHORT 00379199  
003791A4    AA              STOS BYTE PTR ES:\[EDI\]            ;把 api 的名字解码到 3830E4  
003791A5    9C              PUSHFD  
  
00379340   /E9 798E0000     JMP 003821BE                ;调用 38290F, GetProcAddress  
;0012FF90   00379347  
;0012FF94   77DA0000  ADVAPI32.77DA0000  
;0012FF98   003830E4  ASCII "RegQueryValueExA"  
;0012FF9C   003837B6  
;0012FFA0   00000003  
  
00379347    E8 F0960000     CALL 00382A3C                ;检查是否函数头是否为 CC  
  
0037949B    8907            MOV DWORD PTR DS:\[EDI\],EAX               ; ADVAPI32.RegQueryValueExA  
0037949D    5A              POP EDX  
0037949E    0FB642 FF       MOVZX EAX,BYTE PTR DS:\[EDX\-1\]  
003794A2    03D0            ADD EDX,EAX  
003794A4    42              INC EDX                    ;到下一个函数  
003794A5    59              POP ECX  
003794A6    49              DEC ECX                    ;ECX=3  
003794A7  ^ 0F85 B1F9FFFF   JNZ 00378E5E                ;这里的 3 个函数都是注册表相关函数..  
003794AD    E9 A82B0000     JMP 0037C05A  
  
00378A45    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
;0012FF9C   00378A58  
;0012FFA0   003837F0  ASCII "oleaut32.dll"  
  
0037949B    8907            MOV DWORD PTR DS:\[EDI\],EAX               ; oleaut32.SysFreeString -> 6F726C  
0037949D    5A              POP EDX  
0037949E    0FB642 FF       MOVZX EAX,BYTE PTR DS:\[EDX\-1\]  
003794A2    03D0            ADD EDX,EAX  
003794A4    42              INC EDX  
003794A5    59              POP ECX  
003794A6    49              DEC ECX  
003794A7  ^ 0F85 B1F9FFFF   JNZ 00378E5E  
003794AD    E9 A82B0000     JMP 0037C05A  
  
;上面是不要加密的 api,下面是需要加密的 api  
003797A6    8B3A            MOV EDI,DWORD PTR DS:\[EDX\]  
003797A8    03BD 5C174100   ADD EDI,DWORD PTR SS:\[EBP+41175C\]  
003797AE    891F            MOV DWORD PTR DS:\[EDI\],EBX  
003797B0    83C3 20         ADD EBX,20  
003797B3    0FB642 04       MOVZX EAX,BYTE PTR DS:\[EDX+4\]  
003797B7    0AC0            OR AL,AL  
003797B9    75 02           JNZ SHORT 003797BD  
  
003797BD    03D0            ADD EDX,EAX  
003797BF    83C2 06         ADD EDX,6  
003797C2    49              DEC ECX  
003797C3    0BC9            OR ECX,ECX  
003797C5  ^ 75 DF           JNZ SHORT 003797A6            ;这一段,在 iat 中写入 hook 函数的地址  
  
;开始构造 hook 函数  
0037982E    66:C707 FF35    MOV WORD PTR DS:\[EDI\],35FF        ;PUSH DWORD PTR DS:\[0\]  
00379982    C747 06 8134240>MOV DWORD PTR DS:\[EDI+6\],243481    ;XOR DWORD PTR SS:\[ESP\],0  
00379AD8    8947 02         MOV DWORD PTR DS:\[EDI+2\],EAX    ;hook\_proc+1C  
00379B08    C647 0D C3      MOV BYTE PTR DS:\[EDI+D\],0C3        ;ret  
00379B0D    0F31            RDTSC                ;取时间戳作为加密 stub\_proc 地址的 key  
  
00379C81    8947 09         MOV DWORD PTR DS:\[EDI+9\],EAX    ;把 key 作为 xor 的一个操作数  
  
00379C85    83C7 20         ADD EDI,20  
00379C88    49              DEC ECX  
00379C89    0BC9            OR ECX,ECX  
00379C8B  ^ 0F85 6DFBFFFF   JNZ 003797FE            ;构造下一个 hook 函数  
  
;===============计算 stub\_proc 的地址  
00379C93    83C2 04         ADD EDX,4  
00379C96    51              PUSH ECX  
00379C97    0FB602          MOVZX EAX,BYTE PTR DS:\[EDX\]  
00379C9A    0BC0            OR EAX,EAX  
00379C9C    0F85 EF090000   JNZ 0037A691  
  
0037A955    8D95 9C1D4100   LEA EDX,DWORD PTR SS:\[EBP+411D9C\]        ;得到了当前要的到地址的 api 名  
;Address=003830E4, (ASCII "WSAStartup")  
0037A95B    52              PUSH EDX                    ;从这里跳到★★,不作字符串比较,不从壳内取 api 地址  
;...  
0037AC22    8B85 C0194100   MOV EAX,DWORD PTR SS:\[EBP+4119C0\]        ; kernel32.lstrcmpA  做字符串比较,如果相同,就从壳内取 api 地址  
  
0037AC35    85C0            TEST EAX,EAX                 ;.. n 次比较  
0037AC37    75 0B           JNZ SHORT 0037AC44  
  
0037AF0B    8B85 C0194100   MOV EAX,DWORD PTR SS:\[EBP+4119C0\]        ; kernel32.lstrcmpA  
  
;...  
0037B905    5A              POP EDX                    ;最后一次 strcmp 出来到这里  
0037B906    85C0            TEST EAX,EAX  
0037B908    75 0B           JNZ SHORT 0037B915  
  
;★★:  
0037B915    52              PUSH EDX                    ;EDX=3830E4  
0037B916    56              PUSH ESI  
  
;调用一个函数,取得 api 的地址  
0037B943   /E9 76680000     JMP 003821BE                ;调用 0038290F  
;0012FF90   0037B949  
;0012FF94   71A40000    module handle  
;0012FF98   003830E4    ASCII "WSACleanup"  
;0012FF9C   0038449A  
;0012FFA0   00000019  
;返回值 EAX=71A24428  WS2\_32.WSACleanup  
;返回到这里  
0037B949    8B9D C7204100   MOV EBX,DWORD PTR SS:\[EBP+4120C7\]  
  
0037B95A    E8 C5690000     CALL 00382324                ;建立 stub\_proc  
;0012FF8C   00C90000  |Arg1 = 00C90000                    ;返回值 EAX=stub\_proc+1  
;0012FF90   71A24428  |Arg2 = 71A24428  
;0012FF94   00000000  \\Arg3 = 00000000  
;===========  
;进入这个函数  
    00382324    60              PUSHAD  
    00382325    8B7C24 24       MOV EDI,DWORD PTR SS:\[ESP+24\]  
    00382329    8B7424 28       MOV ESI,DWORD PTR SS:\[ESP+28\]  
    0038232D    66:8B06         MOV AX,WORD PTR DS:\[ESI\]            ;从这里跳到▲  
  
;构造 stub\_proc 的时候,最后到这里  
;▲  
    003826DD    C607 68         MOV BYTE PTR DS:\[EDI\],68            ;push xx  
    003826E0    8977 01         MOV DWORD PTR DS:\[EDI+1\],ESI        ;push WS2\_32.71A24437  
    003826E3    C647 05 C3      MOV BYTE PTR DS:\[EDI+5\],0C3            ;ret  
    003826E7    83C7 06         ADD EDI,6  
    003826EA    897C24 FC       MOV DWORD PTR SS:\[ESP\-4\],EDI  
    003826EE    837C24 2C 00    CMP DWORD PTR SS:\[ESP+2C\],0  
;===========  
  
0037B95F    2B85 C7204100   SUB EAX,DWORD PTR SS:\[EBP+4120C7\]  
0037B965    8985 CB204100   MOV DWORD PTR SS:\[EBP+4120CB\],EAX  
0037B96B    60              PUSHAD  
0037B96C    3D C01F0000     CMP EAX,1FC0                ;看是不是 buffer 不够  
0037B971    0F86 8A010000   JBE 0037BB01                ;够就跳走  
  
0037BC52    8BC3            MOV EAX,EBX                    ;EBX 为 stub\_proc 的地址  
0037BC54    3347 09         XOR EAX,DWORD PTR DS:\[EDI+9\]          
0037BDA6    8947 1C         MOV DWORD PTR DS:\[EDI+1C\],EAX        ;加密后写入 hook\_proc  
  
0037BEF9    0FB642 FF       MOVZX EAX,BYTE PTR DS:\[EDX\-1\]  
0037BEFD    03D0            ADD EDX,EAX  
0037BEFF    42              INC EDX  
0037BF00    83C7 20         ADD EDI,20  
  
0037C052    59              POP ECX  
0037C053    49              DEC ECX  
0037C054  ^ 0F85 39DCFFFF   JNZ 00379C93                ;下一个  
0037C05A  ^ E9 17C7FFFF     JMP 00378776  
  
;==========================================================  
;iat 处理完毕  
;上面的 iat 处理,实际上是形成了这样的一个调用过程  
;  
;iat 中地址    -->    |Hook\_proc:  
;            |PUSH DWORD PTR DS:\[Hook\_proc+1C\]  
;            |XOR DWORD PTR SS:\[ESP\],key  
;            |ret;                    ->    |Stub\_proc:  
;            |                        |api\_start\_code  
;            |Hook\_proc+1C:                    |...  
;            |Stub\_proc\_xor\_key                |api\_some\_code  
;                                    |push api\_next\_code\_addr  
;                                    |ret  
;  
;  
;  
;  
;调用 ZwQueryInformationProcess=============================  
0037C05F    E8 0A000000     CALL 0037C06E  
  
0037C08E    8B85 48174100   MOV EAX,DWORD PTR SS:\[EBP+411748\]        ; kernel32.GetModuleHandleA  
;0012FF9C   0037C0A0  
;0012FFA0   0037C064  ASCII "ntdll.dll"  
  
  
0037C243   /E9 765F0000     JMP 003821BE  
;0012FF98   0037C249  
;0012FF9C   7C920000  ntdll.7C920000  
;0012FFA0   0037C1FC  ASCII "ZwQueryInformationProcess"  
  
0037C398    0BC0            OR EAX,EAX                               ; ntdll.ZwQueryInformationProcess  
0037C39A    0F84 45080000   JE 0037CBE5                    ;在这里干掉它, 我们把 EAX 置 0  
  
;开始进行校验和验证============================================  
0037D2AE    8B85 82194100   MOV EAX,DWORD PTR SS:\[EBP+411982\]        ; kernel32.GetModuleFileNameA  
;0012FE94   0037D2C0  
;0012FE98   00400000  ASCII "MZP"  
;0012FE9C   0012FEA4     
  
0037D429    8B85 AF184100   MOV EAX,DWORD PTR SS:\[EBP+4118AF\]        ; kernel32.CreateFileA  
  
0037D468    8B85 29194100   MOV EAX,DWORD PTR SS:\[EBP+411929\]        ; kernel32.GetFileSize  
  
0037D4AF    8B85 BC184100   MOV EAX,DWORD PTR SS:\[EBP+4118BC\]        ; kernel32.CreateFileMappingA  
  
0037D4DC    8B85 D8194100   MOV EAX,DWORD PTR SS:\[EBP+4119D8\]        ; kernel32.MapViewOfFile  
  
0037D509    E8 983C0000     CALL 003811A6                ;计算校验和  
;返回值 EAX=F219B48A  
0037D50F    3385 70174100   XOR EAX,DWORD PTR SS:\[EBP+411770\]  
0037D515    C1C8 03         ROR EAX,3  
0037D518    8BF0            MOV ESI,EAX                    ;最后的结果 DE5AB7CE  
0037D51A    8B85 68174100   MOV EAX,DWORD PTR SS:\[EBP+411768\]  
  
0037D66F    0340 3C         ADD EAX,DWORD PTR DS:\[EAX+3C\]        ;EAX 指向 pe 头  
0037D672    8B78 FC         MOV EDI,DWORD PTR DS:\[EAX\-4\]        ;pe 头-4,原校验和 DE5AB7CE  
  
0037D69B    8B85 561A4100   MOV EAX,DWORD PTR SS:\[EBP+411A56\]        ; kernel32.UnmapViewOfFile  
  
0037D6D3    8B85 A2184100   MOV EAX,DWORD PTR SS:\[EBP+4118A2\]        ; kernel32.CloseHandle  
  
0037D706    8B85 A2184100   MOV EAX,DWORD PTR SS:\[EBP+4118A2\]        ; kernel32.CloseHandle  
  
0037D872    3BF7            CMP ESI,EDI                    ;在这里比较校验结果  
0037D874    0F85 F65E0000   JNZ 00383770  
  
;这一段好像对代码段进行了处理,什么意思呢=========================================  
0037DB40    66:8B06         MOV AX,WORD PTR DS:\[ESI\]            ;ESI=401000  
0037DB43    3C E8           CMP AL,0E8  
0037DB45    0F85 C0010000   JNZ 0037DD0B  
  
0037DD0B    3C E9           CMP AL,0E9  
0037DD0D    0F85 C0010000   JNZ 0037DED3  
  
0037DED3    3C 0F           CMP AL,0F  
0037DED5    0F85 D0010000   JNZ 0037E0AB  
  
  
  
  
0037E0AB    46              INC ESI  
0037E0AC    49              DEC ECX  
0037E0AD    81F9 00000080   CMP ECX,80000000  
0037E0B3  ^ 0F82 87FAFFFF   JB 0037DB40  
;========================================  
  
0037E0B9    8D85 DCCE4000   LEA EAX,DWORD PTR SS:\[EBP+40CEDC\]  
  
0037E20F    8B85 B4194100   MOV EAX,DWORD PTR SS:\[EBP+4119B4\]        ; kernel32.GetVersion  
  
0037E24A    8B85 36194100   MOV EAX,DWORD PTR SS:\[EBP+411936\]        ; kernel32.GetCurrentProcess  
  
0037E3D1    8B85 49194100   MOV EAX,DWORD PTR SS:\[EBP+411949\]        ; kernel32.GetCurrentProcessId  
  
0037E53F    8B85 18194100   MOV EAX,DWORD PTR SS:\[EBP+411918\]        ; kernel32.GetCommandLineA  
  
0037E57E    8B85 70194100   MOV EAX,DWORD PTR SS:\[EBP+411970\]        ; kernel32.GetModuleHandleA  
;0012FF9C   0037E590  
;0012FFA0   00000000  
  
0037E843    E8 DC3A0000     CALL 00382324                ;干吗的? 先放着  
;0012FF98   00383307  |Arg1 = 00383307  
;0012FF9C   71A26233  |Arg2 = 71A26233        WS2\_32.WSASend  
;0012FFA0   00000000  \\Arg3 = 00000000  
  
0037E84A    FFB5 CF1A4100   PUSH DWORD PTR SS:\[EBP+411ACF\]           ; WS2\_32.WSARecv  
  
0037E857    E8 C83A0000     CALL 00382324  
;0012FF98   00383307  |Arg1 = 00383347  
;0012FF9C   71A26233  |Arg2 =            WS2\_32.WSARecv  
;0012FFA0   00000000  \\Arg3 = 00000000  
  
0037E890    8B85 BC184100   MOV EAX,DWORD PTR SS:\[EBP+4118BC\]        ; kernel32.CreateFileMappingA  
;0012FF88   0037E8A2  
;0012FF8C   FFFFFFFF  
;0012FF90   00000000  
;0012FF94   00000004  
;0012FF98   00000000  
;0012FF9C   00000100  
;0012FFA0   0038311C  ASCII "ShellTmpMap"  
  
0037E8DD    8B85 D8194100   MOV EAX,DWORD PTR SS:\[EBP+4119D8\]        ; kernel32.MapViewOfFile  
;返回值 EAX=F70000    去看看,是空的?  
  
0037E911    64:FF35 3000000>PUSH DWORD PTR FS:\[30\]            ;检测应用程序级 debugger  
0037E918    58              POP EAX  
0037E919    0FB658 02       MOVZX EBX,BYTE PTR DS:\[EAX+2\]  
0037E91D    0ADB            OR BL,BL  
0037E91F    0F85 4B4E0000   JNZ 00383770  
0037E925    E9 77010000     JMP 0037EAA1  
  
;SEH 又来了===============================================  
0037EF24    E8 0E000000     CALL 0037EF37  
0037EF29    8B4C24 0C       MOV ECX,DWORD PTR SS:\[ESP+C\]  
0037EF2D    8381 B8000000 0>ADD DWORD PTR DS:\[ECX+B8\],2            ;仅仅是改变 eip  
0037EF34    33C0            XOR EAX,EAX  
0037EF36    C3              RETN  
  
0037EF37    64:FF35 0000000>PUSH DWORD PTR FS:\[0\]  
0037EF3E    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
0037EF45    33C0            XOR EAX,EAX  
0037EF47    CD 01           INT 1  
0037EF49    40              INC EAX  
0037EF4A    40              INC EAX  
0037EF4B    0BC0            OR EAX,EAX  
0037EF4D    64:8F05 0000000>POP DWORD PTR FS:\[0\]  
0037EF54    58              POP EAX  
  
;对 mainform 的处理=================================  
0037F253    8B1E            MOV EBX,DWORD PTR DS:\[ESI\]            ;  
0037F25A    039D 5C174100   ADD EBX,DWORD PTR SS:\[EBP+41175C\]        ;EBX=00537B00  
  
0012FF94    31C9            XOR ECX,ECX  
0012FF96    890E            MOV DWORD PTR DS:\[ESI\],ECX  
0012FF98    83C6 04         ADD ESI,4  
0012FF9B    8933            MOV DWORD PTR DS:\[EBX\],ESI            ;我们要做的是吧 3891F3--38A7DE,复制到 53904D--53A638,长度为 15EB,  再把 ESI 这个指针给替换掉  
;ESI=003891F3  
;DS:\[00537B00\]=00000000  
0012FF9D    0FB70E          MOVZX ECX,WORD PTR DS:\[ESI\]  
0012FFA0    46              INC ESI  
0012FFA1    46              INC ESI  
0012FFA2    C3              RETN  
;  
;为 mainform 中的数据做重定位处理?  
0037F504    8B95 80174100   MOV EDX,DWORD PTR SS:\[EBP+411780\]  
0037F50A   /EB 0C           JMP SHORT 0037F518  
  
0037F50C    2956 02         SUB DWORD PTR DS:\[ESI+2\],EDX                    ; main.00400000    ESI=3891F3  
0037F50F    015E 02         ADD DWORD PTR DS:\[ESI+2\],EBX                    ; main.00400000  
0037F512    0FB706          MOVZX EAX,WORD PTR DS:\[ESI\]  
0037F515    03F0            ADD ESI,EAX  
0037F517    49              DEC ECX  
0037F518    0BC9            OR ECX,ECX  
0037F51A  ^ 75 F0           JNZ SHORT 0037F50C  
;mainform 处理结束======================================  
  
0037F54A    8B85 54174100   MOV EAX,DWORD PTR SS:\[EBP+411754\]               ; kernel32.VirtualAlloc  
;返回值 F80000  
  
0037F56C    64:FF35 3000000>PUSH DWORD PTR FS:\[30\]  
0037F573    58              POP EAX  
  
0037F6C7    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]  
0037F6CA    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]  
0037F6CD    C740 20 0010000>MOV DWORD PTR DS:\[EAX+20\],1000            ;anti dump  
  
  
0037F8B9    8B85 771A4100   MOV EAX,DWORD PTR SS:\[EBP+411A77\]               ; kernel32.VirtualProtect  
  
  
;==================================  
0037FBA0    0FB706          MOVZX EAX,WORD PTR DS:\[ESI\]  
  
0037FBCB    03C3            ADD EAX,EBX                    ;EBX=385EE3 表基址?  
0037FBF8    0110            ADD DWORD PTR DS:\[EAX\],EDX            ;EDX=386EA3 表中每个被索引到的元素都要加上一个值  
  
0037FC22    83C1 02         ADD ECX,2  
0037FC50    83C6 02         ADD ESI,2                    ;word 数组, 数组元素为表的索引  
  
0037FC77    3B8D C4174100   CMP ECX,DWORD PTR SS:\[EBP+4117C4\]        ;数组的 size  
0037FC7D  ^ 0F82 15FFFFFF   JB 0037FB98  
  
  
0037FC9F    66:813E FF35    CMP WORD PTR DS:\[ESI\],35FF  
0037FCA4    75 0B           JNZ SHORT 0037FCB1  
0037FCA6    0146 02         ADD DWORD PTR DS:\[ESI+2\],EAX  
0037FCA9    0146 07         ADD DWORD PTR DS:\[ESI+7\],EAX  
0037FCAC    83C6 0C         ADD ESI,0C  
0037FCAF    EB 06           JMP SHORT 0037FCB7  
0037FCB1    0146 02         ADD DWORD PTR DS:\[ESI+2\],EAX                    ; main.00400000  
0037FCB4    83C6 07         ADD ESI,7  
0037FCB7    49              DEC ECX  
0037FCB8    0BC9            OR ECX,ECX  
0037FCBA  ^ 75 E3           JNZ SHORT 0037FC9F  
;上面一个循环之后,形成了这样的一段代码=========================================  
;00386EAA    8B1D E4855700   MOV EBX,DWORD PTR DS:\[5785E4\]  
;00386EB0    C3              RETN  
;00386EB1    8B35 E4855700   MOV ESI,DWORD PTR DS:\[5785E4\]  
;00386EB7    C3              RETN  
;00386EB8    8B15 1C865700   MOV EDX,DWORD PTR DS:\[57861C\]  
;00386EBE    C3              RETN  
;00386EBF    8B1D 1C865700   MOV EBX,DWORD PTR DS:\[57861C\]  
;...  
;=========================================  
  
0037FD1B    8B85 54174100   MOV EAX,DWORD PTR SS:\[EBP+411754\]               ; kernel32.VirtualAlloc  
;返回值 00F90000  
  
0037FD58    8BF8            MOV EDI,EAX  
  
0037FD84    8BB5 5C174100   MOV ESI,DWORD PTR SS:\[EBP+41175C\]               ; main.00400000  
  
0037FDB4    2B85 5C174100   SUB EAX,DWORD PTR SS:\[EBP+41175C\]               ; main.00400000  
  
0037FDE5    8746 3C         XCHG DWORD PTR DS:\[ESI+3C\],EAX            ;恩?这是干吗,把 pe 头指针给改了?改到缓冲区去了  
;EAX=00B90000          
;DS:\[0040003C\]=00000100  
0037FE3D    B9 00010000     MOV ECX,100  
0037FE6D    C1E9 02         SHR ECX,2  
0037FE70    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
;ECX=00000040 (decimal 64.)  
;DS:\[ESI\]=\[00400100\]=00004550  
;ES:\[EDI\]=\[00F90000\]=00000000  
0037FE72    59              POP ECX  
0037FE73    83E1 03         AND ECX,3  
0037FE76    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
0037FE78    EB 4E           JMP SHORT 0037FEC8  
  
0037FF92    51              PUSH ECX  
0037FF93    C1E9 02         SHR ECX,2  
0037FF96    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]        ;把文件头给改了?  
;ECX=000003F0 (decimal 1008.)  
;DS:\[ESI\]=\[00385EE3\]=00386EA3  
;ES:\[EDI\]=\[00400040\]=0E0010BA  
0037FF98    59              POP ECX  
0037FF99    83E1 03         AND ECX,3  
0037FF9C    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
0037FF9E    EB 22           JMP SHORT 0037FFC2  
;=============================  
;上面这段代码 37FBA0--37FF9E, 把代码段中部分的 "mov 寄存器, \[地址 A\]"语句给替换成了"call \[地址 B\]", 语句都为 6 个字节长度  
;地址 B 处于 400000--401000 文件头部范围内. 而在 地址 B 处是这样的代码 " jmp \[地址 C\]"  
;地址 C 处则是原来的"mov 寄存器, \[地址 A\]""ret"  
;处理方法是在 oep 处写一段代码,搜索整个代码段,把被替换掉的代码给找回来就行了.  
;=============================  
  
0037FFEC    8B85 E7194100   MOV EAX,DWORD PTR SS:\[EBP+4119E7\]               ; kernel32.OutputDebugStringA  
;0012FF9C   0037FFFF  
;0012FFA0   0037FFC7  RETURN to 0037FFC7 from 0037FFCC  
0037FFF8   /E9 C1210000            JMP 003821BE  
  
  
  
0038000E    AC              LODS BYTE PTR DS:\[ESI\]  
0038000F    2C 80           SUB AL,80  
00380011    AA              STOS BYTE PTR ES:\[EDI\]  
00380012    49              DEC ECX  
00380013    0BC9            OR ECX,ECX  
00380015  ^ 75 F7           JNZ SHORT 0038000E                ;开始解出 i am....了  
00380017    8DB5 051D4100   LEA ESI,DWORD PTR SS:\[EBP+411D05\]              
  
003800AF    010C38          ADD DWORD PTR DS:\[EAX+EDI\],ECX        ;ECX=0 ?  
003800B2    AD              LODS DWORD PTR DS:\[ESI\]  
003800B3    0BC0            OR EAX,EAX  
003800B5  ^ 75 F8           JNZ SHORT 003800AF  
003800B7    8B85 CC174100   MOV EAX,DWORD PTR SS:\[EBP+4117CC\]  
  
  
  
0038042D    B9 0FFD0000     MOV ECX,0FD0F  
0038045F    C1E9 02         SHR ECX,2  
00380464    AD              LODS DWORD PTR DS:\[ESI\]            ;00372D79  
00380465    3185 A0174100   XOR DWORD PTR SS:\[EBP+4117A0\],EAX        ;SS:\[382AE8\]  
0038046B    49              DEC ECX  
0038046C    0BC9            OR ECX,ECX  
0038046E  ^ 75 F4           JNZ SHORT 00380464  
;最后出循环时  
;00382AE8  61 60 B8 09              a\`?.  
  
0038049A    8948 02         MOV DWORD PTR DS:\[EAX+2\],ECX  
;ECX=003816A6  
;DS:\[004C356B\]=00909313  
  
003804A3    E8 48000000     CALL 003804F0  
;★第三个 SEH 函数=========================  
;其中用到了 dr 寄存器,要把上次 SEH 末尾时的 dr 寄存器值设置好  
003804A8    8B4C24 0C       MOV ECX,DWORD PTR SS:\[ESP+C\]  
003804AC    FF81 B8000000   INC DWORD PTR DS:\[ECX+B8\]  
  
;...  
;第三个 SEH 函数出去之后,没有改变 dr 寄存器内的值,与出第二个 SEH 函数时相同  
;=========================  
003804F2    64:FF31         PUSH DWORD PTR FS:\[ECX\]  
003804F5    64:8921         MOV DWORD PTR FS:\[ECX\],ESP            ;安装第三个 SEH 函数  
003804F8    CC              INT3  
003804F9    90              NOP  
003804FA    64:8F01         POP DWORD PTR FS:\[ECX\]            ;从第三个 SEH 函数出来到这里  
003804FD    83C4 04         ADD ESP,4  
  
  
003807C6    83F9 01                CMP ECX,1  
003807C9    0F85 EB020000          JNZ 00380ABA  
  
  
00380C30    8BB5 B71F4100          MOV ESI,DWORD PTR SS:\[EBP+411FB7\]  
00380C36    03B5 5C174100          ADD ESI,DWORD PTR SS:\[EBP+41175C\]    ;401000  
00380C3C    8B8D BB1F4100          MOV ECX,DWORD PTR SS:\[EBP+411FBB\]    ;171000  
00380C42    E8 5F050000            CALL 3811A6                ;计算校验和  
  
  
00380C47    8985 74174100          MOV DWORD PTR SS:\[EBP+411774\],EAX  
;返回值 7EA93C42  
  
  
;第四个 SEH 出现... =====================================  
0038354B    8BD4                   MOV EDX,ESP  
0038354D    60                     PUSHAD  
0038354E    8B7A 0C                MOV EDI,DWORD PTR DS:\[EDX+C\]  
00383551    8BAF B4000000          MOV EBP,DWORD PTR DS:\[EDI+B4\]  
  
00383561    3B03                   CMP EAX,DWORD PTR DS:\[EBX\]        ;这个地方一定要相等,如果不相等就是出错了  
00383563    0F85 70010000          JNZ 003836D9  
;Stack DS:\[0012FCB4\]=C0000005  
;EAX=C0000005  
  
003836DD    C3                     RETN  
;第一次 C0000005 返回时 dr0=380DF9 eip=380DF8  
;第二次 80000004 返回时 dr 全为 0 eip=380DF9  
;第三次 80000003 返回时 dr0=37E54C eip=37E54B  
;第四次 80000004 返回时 dr 全为 0 eip=37E54C  
;第五次 80000004 返回时 dr 全为 0 eip=37E21B  
;第六次 C0000094 返回时 dr 全为 0 eip=37D4E9    除零错误  
;第六次 C0000095 返回时 dr 全为 0 eip=37D435  
;第七次 C000008C 返回时 dr 全为 0 eip=37B998    数组越界  
;===========================================  
  
00380DE6    64:FF35 00000000       PUSH DWORD PTR FS:\[0\]  
00380DED    64:8925 00000000       MOV DWORD PTR FS:\[0\],ESP        ;安装 SEH  
;0012FF9C   0012FFE0  Pointer to next SEH record  
;0012FFA0   0038354B  SE handler  
00380DF4    33C0                   XOR EAX,EAX  
00380DF6    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]  
00380DF8    90                     NOP  
00380DF9    90                     NOP  
00380DFA    CC                     INT3  
00380DFB  ^ EB FB                  JMP SHORT 00380DF8  
;...  
;第七次从 SEH 函数返回时,到这里  
0037B998    64:8F05 00000000       POP DWORD PTR FS:\[0\]            ;解除 SEH  
0037B99F    58                     POP EAX  
;==========================================  
  
0037B5CC    B8 E3204100            MOV EAX,4120E3  
0037B5D1    8906                   MOV DWORD PTR DS:\[ESI\],EAX                       ; main.004120E3  
;EAX=004120E3 (main.004120E3)  
;DS:\[00383427\]=0038352B  
  
0037B5F3    8B85 5E194100          MOV EAX,DWORD PTR SS:\[EBP+41195E\]                ; kernel32.GetCurrentThread  
  
0037B777    8B85 321A4100          MOV EAX,DWORD PTR SS:\[EBP+411A32\]                ; kernel32.SetThreadPriority  
  
0037733A    8B85 25184100          MOV EAX,DWORD PTR SS:\[EBP+411825\]                ; USER32.EnableWindow  
  
003774C2    61                     POPAD                        ;噢噢~ 好像快到 oep 了  
003774C3    50                     PUSH EAX                        ;dump 时,要从这里跳到★oep  
  
;stolen code, 一般 delphi 程序为 5 个 call. 这个 bt 有 7 个 call  
0038A7E2    55                     PUSH EBP                        ;push ebp  
0038A7E6    54                     PUSH ESP  
0038A7E7    5D                     POP EBP                        ;mov ebp, esp  
  
0038A7EC    54                     PUSH ESP  
0038A7ED    830424 F0              ADD DWORD PTR SS:\[ESP\],-10  
0038A7F1    5C                     POP ESP                        ;add esp, -10  
  
0038A7F4    68 54175700            PUSH 571754  
0038A7F9    58                     POP EAX                        ;mov eax, 571754  
  
0038A7FF    E8 02000000            CALL 0038A806                    ;把返回地址 38A804 压栈  
0038A804    EB 11                  JMP SHORT 0038A817  
  
0038A806    50                     PUSH EAX                        ;为下面的 xchg 作准备  
0038A807    E8 04000000            CALL 0038A810                      
0038A80C    C471 40                LES ESI,FWORD PTR DS:\[ECX+40\]            ;   
  
0038A810    58                     POP EAX                        ;第一 proc ptr \[38A80C\]  
0038A811    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                  
0038A813    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A816    C3                     RETN                            ;call 4071C4  
;=============  
  
0038A81C    E8 04000000            CALL 0038A825  
0038A821    4C                     DEC ESP  
0038A822    73 57                  JNB SHORT 0038A87B  
  
0038A825    58                     POP EAX                        ;\[38A821\]  
0038A826    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, 57734C  
0038A828    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, \[eax\]  
0038A82C    FF30                   PUSH DWORD PTR DS:\[EAX\]  
0038A82E    58                     POP EAX                        ;mov eax, \[eax\]  
      
0038A832    E8 02000000            CALL 0038A839                    ;把返回地址 38A837 压栈  
0038A837    EB 11                  JMP SHORT 0038A84A  
  
0038A839    50                     PUSH EAX  
0038A83A    E8 04000000            CALL 0038A843  
0038A83F    60                     PUSHAD  
0038A840    B7 46                  MOV BH,46  
  
0038A843    58                     POP EAX                        ;第二 proc ptr \[0038A83F\]  
0038A844    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]  
0038A846    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A849    C3                     RETN                            ;call 46B760  
;=============  
0038A84C    FF35 A86D5700          PUSH DWORD PTR DS:\[576DA8\]                         
0038A852    59                     POP ECX                        ;mov ecx, \[576DA8\]  
  
0038A858    E8 04000000            CALL 0038A861  
0038A85D    4C                     DEC ESP  
0038A85E    73 57                  JNB SHORT 0038A8B7  
  
0038A861    58                     POP EAX                        ;  
0038A862    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, 57734C  
0038A864    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, \[eax\]  
0038A869    FF30                   PUSH DWORD PTR DS:\[EAX\]  
0038A86B    58                     POP EAX                        ;mov eax, \[eax\]  
  
0038A86E    FF35 E87A5300          PUSH DWORD PTR DS:\[537AE8\]                  
0038A874    5A                     POP EDX                        ;mov edx, \[537AE8\]  
  
0038A879    E8 02000000            CALL 0038A880                    ;把返回地址 38A87E 压栈  
0038A87E    EB 11                  JMP SHORT 0038A891  
  
0038A880    50                     PUSH EAX  
0038A881    E8 04000000            CALL 0038A88A  
0038A886  ^ 78 B7                  JS SHORT 0038A83F                    ;第三 proc ptr  
0038A888    46                     INC ESI  
  
0038A88A    58                     POP EAX  
0038A88B    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                  
0038A88D    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A890    C3                     RETN                            ;call 46B778  
;=============  
0038A895    FF35 DC6B5700          PUSH DWORD PTR DS:\[576BDC\]          
0038A89B    59                     POP ECX                        ;mov ecx, \[576BDC\]  
  
0038A8A1    E8 04000000            CALL 0038A8AA  
0038A8A6    4C                     DEC ESP  
0038A8A7    73 57                  JNB SHORT 0038A900  
  
0038A8AA    58                     POP EAX  
0038A8AB    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, 57734C  
0038A8AD    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, \[eax\]  
0038A8B3    FF30                   PUSH DWORD PTR DS:\[EAX\]  
0038A8B5    58                     POP EAX                        ;mov eax, \[eax\]  
  
0038A8BB    FF35 74405200          PUSH DWORD PTR DS:\[524074\]                  
0038A8C1    5A                     POP EDX                        ;mov edx, \[524074\]  
  
0038A8C5    E8 02000000            CALL 0038A8CC                    ;把返回地址 38A8CA 压栈  
0038A8CA    EB 11                  JMP SHORT 0038A8DD  
  
0038A8CD    E8 04000000            CALL 0038A8D6  
0038A8D2  ^ 78 B7                  JS SHORT 0038A88B                    ;第四 proc ptr  
0038A8D4    46                     INC ESI  
  
0038A8D6    58                     POP EAX                              
0038A8D7    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]  
0038A8D9    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A8DC    C3                     RETN                            ;call 46B778  
;=============  
0038A8E0    FF35 30765700          PUSH DWORD PTR DS:\[577630\]                  
0038A8E6    59                     POP ECX                        ;mov ecx, \[577630\]  
  
0038A8EC    E8 04000000            CALL 0038A8F5  
0038A8F1    4C                     DEC ESP  
0038A8F2    73 57                  JNB SHORT 0038A94B  
  
0038A8F5    58                     POP EAX  
0038A8F6    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, 57734C  
0038A8F8    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, \[eax\]  
0038A8FD    FF30                   PUSH DWORD PTR DS:\[EAX\]  
0038A8FF    58                     POP EAX                        ;mov eax, \[eax\]  
  
0038A903    FF35 A0755300          PUSH DWORD PTR DS:\[5375A0\]                  
0038A909    5A                     POP EDX                        ;mov edx, \[5375A0\]  
  
0038A90D    E8 02000000            CALL 0038A914                    ;把返回地址 38A912 压栈  
0038A912    EB 11                  JMP SHORT 0038A925                      
  
0038A914    50                     PUSH EAX  
0038A915    E8 04000000            CALL 0038A91E  
0038A91A  ^ 78 B7                  JS SHORT 0038A8D3                    ;第五 proc ptr  
0038A91C    46                     INC ESI  
  
0038A91E    58                     POP EAX                          
0038A91F    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                  
0038A921    870424                 XCHG DWORD PTR SS:\[ESP\],EAX                  
0038A924    C3                     RETN                            ;call 46B778  
  
;=============  
0038A928    E8 04000000            CALL 0038A931  
0038A92D    4C                     DEC ESP  
0038A92E    73 57                  JNB SHORT 0038A987  
  
0038A931    58                     POP EAX                          
0038A932    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, 57734C  
0038A934    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]                ;mov eax, \[eax\]  
0038A939    FF30                   PUSH DWORD PTR DS:\[EAX\]  
0038A93B    58                     POP EAX                        ;mov eax, \[eax\]  
  
0038A93E    E8 02000000            CALL 0038A945                    ;把返回地址 38A943 压栈  
0038A943    EB 11                  JMP SHORT 0038A956  
  
0038A945    50                     PUSH EAX  
0038A946    E8 04000000            CALL 0038A94F  
0038A94B    F8                     CLC                            ;第六 proc ptr  
0038A94C    B7 46                  MOV BH,46  
  
0038A94F    58                     POP EAX                          
0038A950    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]  
0038A952    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A955    C3                     RETN                            ;call 46B7F8  
;=============  
0038A959    E8 02000000            CALL 0038A960                    ;把返回地址 38A95E 压栈  
0038A95E    EB 11                  JMP SHORT 0038A971                      
  
0038A960    50                     PUSH EAX  
0038A961    E8 04000000            CALL 0038A96A  
0038A966    184B 40                SBB BYTE PTR DS:\[EBX+40\],CL                ;第七 proc ptr  
  
0038A96A    58                     POP EAX                          
0038A96B    8B00                   MOV EAX,DWORD PTR DS:\[EAX\]  
0038A96D    870424                 XCHG DWORD PTR SS:\[ESP\],EAX  
0038A970    C3                     RETN                            ;call 404B18  
  
  
;======================最后还原出来的 stolen code 是这样的  
;★oep:  
00578000    55               PUSH EBP  
00578001    8BEC             MOV EBP,ESP  
00578003    83C4 F0          ADD ESP,-10  
00578006    B8 54175700      MOV EAX,main.00571754  
0057800B    E8 B4F1E8FF      CALL main.004071C4  
00578010    B8 4C735700      MOV EAX,main.0057734C  
00578015    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578017    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578019    E8 4237EFFF      CALL main.0046B760  
0057801E    8B0D A86D5700    MOV ECX,DWORD PTR DS:\[576DA8\]                    ; main.005DCC84  
00578024    B8 4C735700      MOV EAX,main.0057734C  
00578029    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
0057802B    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
0057802D    8B15 E87A5300    MOV EDX,DWORD PTR DS:\[537AE8\]                    ; main.00537B34  
00578033    E8 4037EFFF      CALL main.0046B778  
00578038    8B0D DC6B5700    MOV ECX,DWORD PTR DS:\[576BDC\]                    ; main.005DC990  
0057803E    B8 4C735700      MOV EAX,main.0057734C  
00578043    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578045    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578047    8B15 74405200    MOV EDX,DWORD PTR DS:\[524074\]                    ; main.005240C0  
0057804D    E8 2637EFFF      CALL main.0046B778  
00578052    8B0D 30765700    MOV ECX,DWORD PTR DS:\[577630\]                    ; main.005DCC7C  
00578058    B8 4C735700      MOV EAX,main.0057734C  
0057805D    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
0057805F    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578061    8B15 A0755300    MOV EDX,DWORD PTR DS:\[5375A0\]                    ; main.005375EC  
00578067    E8 0C37EFFF      CALL main.0046B778  
0057806C    B8 4C735700      MOV EAX,main.0057734C  
00578071    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578073    8B00             MOV EAX,DWORD PTR DS:\[EAX\]  
00578075    E8 7E37EFFF      CALL main.0046B7F8                    ;Application.Run  
0057807A    E8 99CAE8FF      CALL main.00404B18  
;======================  
  
;最后,在 oep 处完成我们上面原理部分第 9 步的处理,就一切 ok 了. dump 再 importrec 修复.  
  
  
  
;附上处理 iat 和"mov 替换"的代码  
;1.iat  
    pushad  
    pushfd  
    mov eax, 6F7190        ;iat 起始处  
    mov ecx, 6F79C4        ;iat 结尾处  
@@loop:  
    add eax, 4  
    cmp eax, ecx  
    ja @@end  
    cmp \[eax\], 0  
    je @@loop  
    cmp \[eax\], DA0000  
    ja @@loop  
    mov esi, \[eax\]  
    mov esi, \[esi+1c\]  
    mov esi, \[esi+1\]  
    mov \[eax\], esi  
    jmp @@loop  
@@end:  
    popfd  
    popad  
  
;2.mov 替换  
    pushad  
    pushfd  
  
    mov eax, 401000  
    sub eax, 1  
    mov edx, 718000  
    sub edx, 2  
@@loop:  
    add eax, 1  
    cmp eax, ecx  
    ja @@end  
    cmp WORD ptr \[eax\], 15FF  
    jne @@loop  
    mov edi, eax  
    mov esi, \[eax+2\]  
    add eax, 1  
    cmp esi, 401000  
    jnb @@loop  
    mov esi, \[esi\]  
    mov ecx, 6  
    rep movsb \[edi\], \[esi\]  
    jmp @@loop  
@@end:  
    popfd  
    popad  
  
  
  
;special thanx to heXer, 老菜鸟, loveboom, csjwaman && 老罗的代码着色器. 

 4dd466d0e1e0cc123afbe504da2462ca