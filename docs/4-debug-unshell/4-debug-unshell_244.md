

# HyperSnap-DX 5.60

> 原文：[`www.pediy.com/kssd/pediy06/pediy6724.htm`](https://www.pediy.com/kssd/pediy06/pediy6724.htm)

好久没怎么脱过壳，最近碰到一个软件是用 arm 加的壳，用到了 arm 的输入表乱序和远地址跳转，只有找些资料来补课，略有些心得，还望指正。  
  
下面以 HyperSnap-DX 5.60 为例，这个是 arm3.70a 加的壳，单进程，用了输入表乱序和远地址跳转。HyperSnap-DX 5.61 已经是双进程 CC 壳了，还不敢碰，:(   
  
第一部分：准备工作&抵达 OEP  
  
先需要找到一些基本数据  
1. OEP  
设断点 he SetProcessWorkingSetSize 断下后再设断点 he GetCurrentThreadId  
断下后,Ctrl+F9 返回，去除这两个断点，向下看，看到那个 call edi 了没，在按 F2 那里设个断点  
断下后，F7 跟入，就来到 OEP 了  
记住： OEP=004E8850  
  
2. magicjump  
好几个地方修改跳转都可以放置 arm 破坏输入表的，都可以称之为 magicjump  
来看 arm 处理输入表的流程  
先将一些预设的 api 经过处理  
然后在读入输入表时候，比较 api 名称是否为壳中处理过的 api 名称  
如果相同则将该地址指向壳中的函数地址，程序所得到的地址就指向壳里了,importRec 就识别不出该  
  
api 了  
  
这个虽然代码比较长，不过流程还算清楚，可以跟上几遍就大概知道是怎么回事了  
因为相关文章比较多，这里简单看一下  
  
00DC96FD    8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:\[EBP\-14C\]  
00DC9703    50              PUSH EAX  
00DC9704    FF15 C8B0DE00   CALL DWORD PTR DS:\[DEB0C8\]               ;   
  
kernel32.GetModuleHandleA  
00DC970A    8B0D C04CDF00   MOV ECX,DWORD PTR DS:\[DF4CC0\]  
00DC9710    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX  
00DC9713    A1 C04CDF00     MOV EAX,DWORD PTR DS:\[DF4CC0\]  
00DC9718    393C06          CMP DWORD PTR DS:\[ESI+EAX\],EDI  
00DC971B    75 16           JNZ SHORT 00DC9733  
00DC971D    8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:\[EBP\-14C\]  
00DC9723    50              PUSH EAX  
00DC9724    FF15 D0B0DE00   CALL DWORD PTR DS:\[DEB0D0\]               ;   
  
kernel32.LoadLibraryA  
00DC972A    8B0D C04CDF00   MOV ECX,DWORD PTR DS:\[DF4CC0\]  
00DC9730    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX  
00DC9733    A1 C04CDF00     MOV EAX,DWORD PTR DS:\[DF4CC0\]  
00DC9738    393C06          CMP DWORD PTR DS:\[ESI+EAX\],EDI  
00DC973B    0F84 AD000000   JE 00DC97EE                           //magicjump 记录该地址  
  
值  
00DC9741    33C9            XOR ECX,ECX  
  
如果是 dll 可能还需要更改这个地址，od 载入 dll 基地址为 10000000，而输入表在该地址以下  
造成在 ImportRec 设不了 IAT RVA 地址  
  
  
00DE489D    8B85 1CE8FFFF   MOV EAX,DWORD PTR SS:\[EBP\-17E4\]       //保存当前 IAT 指针  
00DE48A3    8B8D 80E2FFFF   MOV ECX,DWORD PTR SS:\[EBP\-1D80\]  
00DE48A9    8908            MOV DWORD PTR DS:\[EAX\],ECX            //一个 api 地址写入 IAT  
00DE48AB    8B85 1CE8FFFF   MOV EAX,DWORD PTR SS:\[EBP\-17E4\]  
00DE48B1    83C0 04         ADD EAX,4  
00DE48B4    8985 1CE8FFFF   MOV DWORD PTR SS:\[EBP\-17E4\],EAX  
对\[EBP\-17E4\]来设内存写断点，可以找到这里  
00DE453B    8B8D F0E6FFFF   MOV ECX,DWORD PTR SS:\[EBP\-1910\]       //保存 IAT 的首地址，记录  
  
该地址值 00E9A880  
00DE4541    8D0481          LEA EAX,DWORD PTR DS:\[ECX+EAX\*4\]  
00DE4544    8985 1CE8FFFF   MOV DWORD PTR SS:\[EBP\-17E4\],EAX       //载入下一个 dll 的 IAT 中  
  
首地址   
  
3.antidump  
OD 把原代码中一些代码，jmp 较高的地址了 如 jmp 03171000,造成 lordpe dump 文件时候，这段代码丢  
  
失，  
该地址是不是固定的。  
可以直接下断 VirtualAlloc，查看申请地址值，如果比较大就需要注意了，返回后代码类似下面  
00DE2612    6A 40           PUSH 40  
00DE2614    68 00200000     PUSH 2000  
00DE2619    FFB5 70E6FFFF   PUSH DWORD PTR SS:\[EBP\-1990\]  
00DE261F    FF35 3092DF00   PUSH DWORD PTR DS:\[DF9230\]  
00DE2625    FF15 A0B1DE00   CALL DWORD PTR DS:\[DEB1A0\]               ;   
  
kernel32.VirtualAlloc  
00DE262B    8985 78E6FFFF   MOV DWORD PTR SS:\[EBP\-1988\],EAX  
00DE2631    83BD 78E6FFFF 0>CMP DWORD PTR SS:\[EBP\-1988\],0  
00DE2638    74 33           JE SHORT 00DE266D  
00DE263A    6A 40           PUSH 40  
00DE263C    68 00100000     PUSH 1000  
00DE2641    FFB5 70E6FFFF   PUSH DWORD PTR SS:\[EBP\-1990\]  
00DE2647    FF35 3092DF00   PUSH DWORD PTR DS:\[DF9230\]  
00DE264D    FF15 A0B1DE00   CALL DWORD PTR DS:\[DEB1A0\]               ;   
  
kernel32.VirtualAlloc  
00DE2653    8985 78E6FFFF   MOV DWORD PTR SS:\[EBP\-1988\],EAX          //保存 codesplit 首地  
  
址 在这里断下修改 eax 值到一较低地址我选择写到 arm 的 adata 节  
  
00DE2659    83BD 78E6FFFF 0>CMP DWORD PTR SS:\[EBP\-1988\],0  
00DE2660    74 0B           JE SHORT 00DE266D  
  
记录 CodeSplit addr=00DE2653  
  
4. 用脚本来走到 OEP  
因为要修改的地方比较多，而且后面我们调试脚本难免要重复工作，所以还是写个 od 脚本吧，可以节省点体力 :)  
因为要向 rdata 段写 IAT 数据，所以先将该段内存设为可写  
Memory 窗口选中该段，右键 set access ->full access  

> ```
> 代码:
> //脚本 1，直达 OEP，顺便处理 magicjump 和 antidump
> var NewIatHead
> var NewSplitCodeHead
> var SetIatHead
> var SetSplitCodeHead
> var IatOver
> var MagicJmp
> var OEP
> 
> var bSplitCodeOver
> var bIatOver
> var pTempAddr
> 
> var VirtualAlloc
> 
> //需要填入的信息内容
> mov NewIatHead, 5CA000
> mov NewSplitCodeHead, 674000
> mov MagicJmp, 00DC973B
> mov SetIatHead, 00DE453B
> mov IatOver, 00DE498E
> mov SetSplitCodeHead, 00DE2653
> mov OEP, 004E8850
> 
> //变量初始化
> mov bIatOver, 0
> mov bSplitCodeOver, 0
> 
> //获得 VirtualAlloc 首地址
> gpa "VirtualAlloc", "kernel32.dll" 
> mov VirtualAlloc, $RESULT
> 
> BPHWS VirtualAlloc, "x"
> run
> BPHWC VirtualAlloc
> 
> //此时，壳内存代码已分配
> //开始设断点
> BPHWS MagicJmp, "x"  //magicjump 之处
> //BPHWS 00994704, "x"  写入内存时
> BPHWS SetIatHead, "x"  //载入输入表首地址时候
> BPHWS IatOver, "x"  //处理完所有 dll
> BPHWS SetSplitCodeHead, "x"  //申请高位内存处，需更改返回 eax 为一段低位内存 
> 
> eoe \_Exception
> eob \_Break
> run 
> 
> //遇例外继续执行
> \_Exception: 
> esto
>  
> //处理断点中断
> \_Break:
> cmp eip, SetIatHead
> je \_SetIatHead
> cmp eip, MagicJmp
> je \_MagicJmp
> cmp eip, IatOver
> je \_IATOver
> cmp eip, SetSplitCodeHead
> je \_SetSplitCodeHead
> jmp \_InvalidBreak
> 
> //设置新的 IAT 首地址
> /\*
> 00DE453B    8B8D F0E6FFFF   MOV ECX,DWORD PTR SS:\[EBP\-1910\]       //保存 IAT 的首地址
> 00DE4541    8D0481          LEA EAX,DWORD PTR DS:\[ECX+EAX\*4\]
> 00DE4544    8985 1CE8FFFF   MOV DWORD PTR SS:\[EBP\-17E4\],EAX       //当前 IAT 指针
> \*/
> \_SetIatHead:
> mov pTempAddr, ebp
> sub pTempAddr, 1910          //看上面数据
> mov \[pTempAddr\], NewIatHead
> log NewIatHead
> BPHWC SetIatHead
> run 
> 
> //修改 magicjump, 来得到原始的 IAT
> \_MagicJmp:
> mov !ZF, 1     //修改 magicjump
> run 
> 
> //maigcjump 处理完毕
> \_IATOver:
> BPHWC MagicJmp
> BPHWC IatOver
> mov bIatOver, 1
> cmp bSplitCodeOver,1
> je \_FixOver
> run 
> 
> //设置新的保存 CodeSplit 代码的首地址
> /\*
> 00DE263A    6A 40           PUSH 40
> 00DE263C    68 00100000     PUSH 1000
> 00DE2641    FFB5 70E6FFFF   PUSH DWORD PTR SS:\[EBP\-1990\]
> 00DE2647    FF35 3092DF00   PUSH DWORD PTR DS:\[DF9230\]
> 00DE264D    FF15 A0B1DE00   CALL DWORD PTR DS:\[DEB1A0\]               ; 
> 
> kernel32.VirtualAlloc
> 00DE2653    8985 78E6FFFF   MOV DWORD PTR SS:\[EBP\-1988\],EAX          //保存 antidump 首地址
> 00DE2659    83BD 78E6FFFF 0>CMP DWORD PTR SS:\[EBP\-1988\],0
> 00DE2660    74 0B           JE SHORT 00DE266D
> \*/
> \_SetSplitCodeHead:
> mov eax, NewSplitCodeHead
> mov bSplitCodeOver, 1
> BPHWC SetSplitCodeHead
> cmp bIatOver,1
> je \_FixOver
> run 
> 
> //其它未处理的断点
> \_InvalidBreak:
> log eip
> msg "Invalid Break"
> ret
> 
> //IAT,AntiDump 处理完毕
> //准备跳往 OEP
> \_FixOver:
> eoe \_Continue
> eob \_End
> BPHWS OEP, "x"
> run
> 
> \_Continue: 
> esto 
> 
> \_End:
> BPHWC OEP
> msg "Success!"
> ret
> 
> 
> ```
> 
>   

   
  
第二部分. 修复 IAT 乱序  
  
1, 关于 IAT 乱序  
arm 从 3.60 开始添加了新的功能输入表乱序(ImportTable Elimination)  
再我们修改了 magicjmp 后得到漂亮的 IAT, 可惜马上就被 arm 给破坏了，将顺序打乱，而且算法有些复  
  
杂，不好反求。  
  
00DE498E    8B85 FCE6FFFF   MOV EAX,DWORD PTR SS:\[EBP-1904\]         //上面的脚本中 IatOver  
  
地址  
00DE4994    8985 F4CFFFFF   MOV DWORD PTR SS:\[EBP-300C\],EAX  
00DE499A    FFB5 F4CFFFFF   PUSH DWORD PTR SS:\[EBP-300C\]  
00DE49A0    E8 855E0000     CALL 00DEA82A                            ; JMP to msvcrt.??  
  
3@YAXPAX@Z  
00DE49A5    59              POP ECX  
00DE49A6    83BD F0E6FFFF 0>CMP DWORD PTR SS:\[EBP-1910\],0  
...  
00DE4AA2    8985 64E1FFFF   MOV DWORD PTR SS:\[EBP-1E9C\],EAX  
00DE4AA8    8B85 60E1FFFF   MOV EAX,DWORD PTR SS:\[EBP-1EA0\]  
00DE4AAE    8D0485 04000000 LEA EAX,DWORD PTR DS:\[EAX\*4+4\]  
00DE4AB5    50              PUSH EAX  
00DE4AB6    8B85 F0E6FFFF   MOV EAX,DWORD PTR SS:\[EBP-1910\]  
00DE4ABC    83C0 04         ADD EAX,4  
00DE4ABF    50              PUSH EAX  
00DE4AC0    FFB5 F0E6FFFF   PUSH DWORD PTR SS:\[EBP-1910\]  
00DE4AC6    FF15 D4B2DE00   CALL DWORD PTR DS:\[DEB2D4\]               ; msvcrt.memmove  //  
  
将 IAT 向前循环移动 2 个 DWORD，打乱 IAT  
00DE4ACC    83C4 0C         ADD ESP,0C  
00DE4ACF    8B85 60E1FFFF   MOV EAX,DWORD PTR SS:\[EBP-1EA0\]  
00DE4AD5    8B8D F0E6FFFF   MOV ECX,DWORD PTR SS:\[EBP-1910\]  
00DE4ADB    8B95 64E1FFFF   MOV EDX,DWORD PTR SS:\[EBP-1E9C\]  
00DE4AE1    891481          MOV DWORD PTR DS:\[ECX+EAX\*4\],EDX  
00DE4AE4  ^ E9 47FFFFFF     JMP 00DE4A30  
  
使得我们在 ImportRec 中看到这样排列的 IAT，从而无法生成新的输入表  
1  001CA0D0  kernel32.dll  01A7  GetProfileIntA  
1  001CA0D4  comctl32.dll  002B  ImageList\_AddMasked  
1  001CA0D8  user32.dll  005B  CreateIconIndirect  
1  001CA0DC  user32.dll  00F7  GetClassInfoA  
1  001CA0E0  gdi32.dll  01CE  LineTo  
1  001CA0E4  user32.dll  008F  DefWindowProcA  
1  001CA0E8  kernel32.dll  015C  GetFileSize  
1  001CA0EC  gdi32.dll  01BA  GetTextFaceA  
1  001CA0F0  gdi32.dll  01C3  GetWindowExtEx  
  
  
2. 整理 IAT  
在 Ricardo Narvaja 的 203-208 教程中给出了一个很脚本，用来整理 IAT，生成一个新的排列整齐的 IAT,   
  
原来的 IAT 中的地址值则改为指针指向这个新 IAT 表中的项，这个脚脚本说明参见 Ricardo 的 207 教程。里面跳转有点多，我估摸着加了点注释。  
  
  
举例   
原先 IAT 中一项为  
005CA000  45 2D E1 77                                      E-醱今  
在运行脚本后  
005CA000  D0 AD 5C 00                                      协\\.尠  
005CADD0  45 2D E1 77                                      E-醱 p  
当然这样做以后，call 所调用的地址就不对了  
call \[005CA000\]     // 77E12D45call Kernel32.SetUnhandledExceptionFilter  
故而 Ricardo 在 OEP 处插入了一段程序来，将排列好的 IAT，再次还原到原先打乱的状态，不过这样出  
  
来的程序在静态反编译时，函数会识别不出，不便于分析。  
在下面 1 节将会采用一段脚本来处理这个问题  
在 OEP 处运行  
  

> ```
> 代码:
> //脚本 2，将 ARM 混乱的 IAT 重新整理
> //来自 Ricardo Narvaja 的 207 号教程，略做修改
> 
> var it
> var it2
> var x
> var y
> var pit
> var pit2
> var dll              
> var dll1            
> var pitt
> var it1\_end
> var base
> var savecode
> 
> //需要设定的内容
> mov it,5CA000        //混乱 IAT 首址
> mov it1\_end,5CAD90    //混乱 IAT 末址
> mov it2,5CADD0        //待存放整理后的 IAT 首址
> 
> //
> mov savecode,\[eip\]   //保存当前 eip 指向内容
> mov \[eip\], #EBFE#    //jmp eip，由于整理 IAT 比较慢，用于待会刷新界面，防界面假死 
> 
> gmi eip, MODULEBASE    //取主文件的基址
> log $RESULT
> mov base,$RESULT
> 
> INICIO:              //初始化
> mov pitt,it          //pitt 指向当前正在处理 api 地址，其前所有 api 均被处理完毕
>                      //pitt 每一轮增加 4，当其等于混乱后 IAT 末址时，则该脚本运行结束
> COMIENZO:
> add pit,it
> add pit2,it2
> 
> SEGUIMOS: //WE FOLLOWED  新的循环开始
> 
> add pit,x
> add pit2,y
> cmp pit,it1\_end
> log pit
> log x
> log y
> je FIN
> cmp pit,it1\_end
> ja FIN
> 
> gmi \[pit\], MODULEBASE    //取该 api 对应的模块基址
> log $RESULT
> log dll1
> cmp pit,pitt
> jne NOPRIMERA
> 
> cmp $RESULT,base       //该地址处 api,是否已被处理过，若被处理过则其基址为主文件基址 base
> je PIRULO              //处理过则跳转到下一轮
> 
> cmp $RESULT,dll1       //这两句好像没用，如果前 1 个 dll 已被处理过，这里根本不会跳转
> je IGUALES
> mov dll, $RESULT
> log dll
> jmp NOPRIMERA
> 
> NOPRIMERA:       //not first
> cmp $RESULT,dll
> jne NOGUARDO
> 
> //该地址处 api,是否已被处理过，若被处理过则其基址
> //例如,处理后该 005CA000 处内容为 005CADD0，取其对应模块基址为为主文件基址 base
> cmp $RESULT,base     
> je NOGUARDO          //处理过则跳转到下一轮
> mov \[pit2\], \[pit\]    //将混乱后 IAT 保存 api 移入新地址，原地址存放指针指向新地址 
> mov \[pit\], pit2
> mov x,4
> mov y,4
> jmp FINLOOP
> 
> NOGUARDO:    //I do not keep
> mov x,4
> mov y,0
> jmp FINLOOP
> 
> FINLOOP:      //1 轮小循环结束
> log pit
> log pit2
> jmp SEGUIMOS
> 
> 
> IGUALES: //EQUAL 没啥用？
> mov x,4
> mov y,0
> jmp FINLOOP
> 
> 
> FIN:              //1 个 dll 处理完毕
> mov \[pit2\],0
> mov dll1,dll
> sto              //防界面假死，刷新 od 界面
> xor x,x
> xor y,y
> add pitt,4
> cmp dll,base
> je SALTO
> 
> add pit2,4
> 
> SALTO:          //JUMP
> mov pit,pitt
> cmp pitt,it1\_end
> je FINISH
> cmp pitt,it1\_end
> ja FINISH
> log pit
> log pit2
> log pitt
> jmp SEGUIMOS
> 
> FINISH:            //所有 dll 都已被处理完毕
> mov \[eip\],savecode //保存当前 eip 指向内容
> MSG "TERMINAMOS"
> log pitt
> ret
> 
> PIRULO:         //该地址 api 所在 dll 已被处理过，故跳过该地址，重新开始一轮
> add pitt,4
> jmp NOPRIMERA
> 
> 
> ```
> 
>   

  
3. 修复代码段调用  
在 Ricardo 的教程中在 OEP 处插入了一段程序来，将排列好的 IAT，再次还原到原先打乱的状态，不过这  
  
样出来的程序在反编译时，不能自动识别出调用的函数，颇为不爽。  
我们来想办法来解决这个问题。  
  
我们在程序中随便找一个 api 调用，比如这个  
004E8873    8965 E8         MOV DWORD PTR SS:\[EBP-18\],ESP  
004E8876    FF15 E4AC5C00   CALL DWORD PTR DS:\[5CACE4\]               ;   
  
kernel32.GetVersion  
  
004E8876  FF 15 64 B5 E9 00                                �d 甸.3  
  
设硬件写 DWORD 断点 004E8878，来看看壳是什么时候在代码段写入的该 api 地址  
  
重新运行程序，第 1 次断下时是代码段解压，在第二次断下时候  
00DE553A    8B85 E4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F1C\]  
00DE5540    40              INC EAX  
00DE5541    8985 E4D0FFFF   MOV DWORD PTR SS:\[EBP-2F1C\],EAX  
00DE5547    8B85 E4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F1C\]  
00DE554D    8B8D 30E8FFFF   MOV ECX,DWORD PTR SS:\[EBP-17D0\]   //保存 call 调用表首地址  
  
00E9B628 在这里查看表格首地址  
00DE5553    833C81 00       CMP DWORD PTR DS:\[ECX+EAX\*4\],0    //取表中 1 项  
00DE5557    0F84 90000000   JE 00DE55ED  
00DE555D    8B85 E4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F1C\]  
00DE5563    8B8D 30E8FFFF   MOV ECX,DWORD PTR SS:\[EBP-17D0\]  
00DE5569    8B95 18E7FFFF   MOV EDX,DWORD PTR SS:\[EBP-18E8\]  
00DE556F    031481          ADD EDX,DWORD PTR DS:\[ECX+EAX\*4\]  
00DE5572    8995 D4D0FFFF   MOV DWORD PTR SS:\[EBP-2F2C\],EDX  
00DE5578    8B85 D4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F2C\]  
00DE557E    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
00DE5580    8985 D0D0FFFF   MOV DWORD PTR SS:\[EBP-2F30\],EAX  
00DE5586    81BD D0D0FFFF 9>CMP DWORD PTR SS:\[EBP-2F30\],90909090  
00DE5590    74 56           JE SHORT 00DE55E8  
00DE5592    8B85 D0D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F30\]  
00DE5598    2B85 E0D0FFFF   SUB EAX,DWORD PTR SS:\[EBP-2F20\]  
00DE559E    8985 D0D0FFFF   MOV DWORD PTR SS:\[EBP-2F30\],EAX  
00DE55A4    FFB5 D0D0FFFF   PUSH DWORD PTR SS:\[EBP-2F30\]  
00DE55AA    8B85 E4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F1C\]  
00DE55B0    33D2            XOR EDX,EDX  
00DE55B2    6A 10           PUSH 10  
00DE55B4    59              POP ECX  
00DE55B5    F7F1            DIV ECX  
00DE55B7    FF1495 48B7DE00 CALL DWORD PTR DS:\[EDX\*4+DEB748\]    //计算该 api 在 IAT 中的索引  
00DE55BE    59              POP ECX  
00DE55BF    8985 D0D0FFFF   MOV DWORD PTR SS:\[EBP-2F30\],EAX  
00DE55C5    8B85 D0D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F30\]  
00DE55CB    8B8D F0E6FFFF   MOV ECX,DWORD PTR SS:\[EBP-1910\]  
00DE55D1    8D0481          LEA EAX,DWORD PTR DS:\[ECX+EAX\*4\]   //得到该 api 在 IAT 中地址  
00DE55D4    8985 D0D0FFFF   MOV DWORD PTR SS:\[EBP-2F30\],EAX  
00DE55DA    8B85 D4D0FFFF   MOV EAX,DWORD PTR SS:\[EBP-2F2C\]  
00DE55E0    8B8D D0D0FFFF   MOV ECX,DWORD PTR SS:\[EBP-2F30\]  
00DE55E6    8908            MOV DWORD PTR DS:\[EAX\],ECX         //写入 call \[XXX\] 的调用地  
  
址  
00DE55E8  ^ E9 4DFFFFFF     JMP 00DE553A  //断在这里  
  
走到这里，看看\[EBP-17D0\]所指向的内存  
00DE554D    8B8D 30E8FFFF   MOV ECX,DWORD PTR SS:\[EBP-17D0\]   //00E9B628 保存 call 调用表首  
  
地址  
  
00E9B628  4B 10 00 00 92 10 00 00 A7 10 00 00 B8 10 00 00  K..?..?..?..  
00E9B638  EA 10 00 00 14 11 00 00 64 11 00 00 75 11 00 00  ?....d..u..  
00E9B648  88 11 00 00 AB 11 00 00 0C 12 00 00 30 12 00 00  ?..?.....0..  
00E9B658  B8 12 00 00 CB 12 00 00 E8 12 00 00 37 13 00 00  ?..?..?..7..  
00E9B668  AC 13 00 00 B8 14 00 00 C0 14 00 00 D6 14 00 00  ?..?..?..?..  
......  
00EA2B88  9B 77 1B 00 A8 77 1B 00 06 78 1B 00 79 78 1B 00  泈.╳.x.yx.  
00EA2B98  EB 78 1B 00 E0 79 1B 00 20 7A 1B 00 7F 7A 1B 00  離.鄖. z.z.  
00EA2BA8  00 00 00 00                                      ....  
^\_^，call 调用所在地址表，这正是我们需要的，call 所在的 RAV 地址，这样就可以用这张表来修复  
  
call\[XXX\]的地址值了，使之直接指向我们的整理后的 IAT，不需要原先的 IAT 做跳板。  
  
拷贝该表格到 WinHex 中，待会好再粘贴回来，运行脚本  
  
现在再次来到运行前面的脚本到达 OEP 之处，  
然后运行 Ricardo 的脚本，整理打乱的 IAT，然后随便找个空地将刚才得到的 call 调用地址表 binary   
  
paste，  
比如我粘贴到 00E92000 开始的内存里  
然后再运行一个小脚本  

> ```
> 代码:
> //脚本 3，修改代码段 call \[XXX\]，到整理后的 IAT
> var pCallTable  //存放 call 调用所在之处的 RVA 地址表
> var calladdr
> var base
> var iataddr
> var goodiat
> 
> //设置 Call RVA 地址表所在位置
> mov pCallTable ,0E92000
> 
> //获取当前模块基地址
> gmi eip, MODULEBASE 
> log $RESULT
> mov base, $RESULT
> 
> NEXT:
> mov calladdr, \[pCallTable\]
> cmp calladdr, 0
> je FIN
> add calladdr, base              //RVA+base=VA
> mov iataddr, \[calladdr\]         //得到原始 IAT 表中地址(间接指针,指向整理后得 IAT 中的地址)
> log calladdr
> mov goodiat, \[iataddr\]          //得到整理后得 IAT 中该 api 地址
> mov \[calladdr\], goodiat         //修改原始代码中 call XXXX 中 XXXX 的地址,指向整理后的 IAT
> add pCallTable,4
> JMP NEXT
> 
> FIN:
> MSG "Success!"
> ret
> 
> 
> ```
> 
>   

  
OK,再看看刚才的那个 api 调用已经直接指向那个整理后的 IAT 中了  
004E8873    8965 E8         MOV DWORD PTR SS:\[EBP-18\],ESP  
004E8876    FF15 68B05C00   CALL DWORD PTR DS:\[5CB068\]               ;   
  
kernel32.GetVersion  
  
4. dump&fix  
  
现在用 LordPE dump 下来，然后用 ImportRec 修复，填入 OEP，RVA 及 RVASIZE  
OEP=00E8850 IATRVA=1CADD0 大小 0D90，将 Invalid 的全部 cut。  
修复以后运行一下看看，ok 了，累 :)  
  
感谢 Ricardo，Eggi 的教程。已上传至[`bbs.pediy.com/showthread.php?s=&threadid=5962`](http://bbs.pediy.com/showthread.php?s=&threadid=5962)  4dd466d0e1e0cc123afbe504da2462ca