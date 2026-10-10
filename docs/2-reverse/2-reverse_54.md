

# ollydbg 破解教学

> 原文：[`www.pediy.com/kssd/pediy06/pediy6388.htm`](https://www.pediy.com/kssd/pediy06/pediy6388.htm)

ollydbg 破解教学之--------万能断点篇   
  
    终于有一点时间总结一下 OD 破解的经验,有许多大侠们都认为 OD 不能下万能断点,可是有些软件的破解需要用到万能断   
点,以前一直没有这方面的资料,有的只是 OD 不能下万能断点的断言,一时心恢之极,可是我原来用 TRW 和 SICE 用的挺好,谁知   
系统又跟我闹矛盾,一用 TRW 和 SICE 就死机,没办法我现在只能用 OD,我写的一些破解文章都是用 OD 破的,反复试验,我终于发   
现 OD 也能下万能断点,在我以前写的一些文章中好象有一篇简单提到过,但没有说的很详细,这次静下心,想写一点东西,送给   
我心爱的组织 DFCG 吧,我的破解是从 DFCG 组织起步的,很感谢 DFCG 的高手们的帮助,在此说声谢谢!为了更明确,这次破解所使   
用的调试器 OD 是从看雪论坛临时下载的 OD1.09 的汉化版,没有使用我自己手头经过改造的 OD,在此也感谢看雪论坛,这里真的   
有很多高手,在这个论坛我也学到了很多.这次破解的对象是超级 XX 王,为保护国产软件我隐去了软件相关信息,我不想给自己   
带来麻烦,现在工作很忙,能少一点麻烦就尽量少一点吧.   
    破解作者   
             yzez\[DFCG\]   
    破解工具   
             ollydbg1.09 汉化版,看雪论坛下载.   
    破解目的   
             本不为破解而破解,只因为技术而破解   
    破解环境   
             WINDOWS XP,这个我已经在 98 和 XP 下各做了两次,贴图和我这篇文章是在 XP 系统下完成的.   
    破解过程   
1.检查了一下,该程序无壳,C++编译,用 W32DSM 反汇编一堆乱码,我倒!那就动态调试吧!用 ollydbg1.09 载入程序,这次我将教   
你们如何用 OD 下万能断点,过程我尽可能详细一点,下面请看.载入程序选 OD 菜单栏上的插件-----命令行(快捷键是 ALT+F1),   
在弹出的窗口中输入万能断点命令:bpx hmemcpy,按 ENTER 键,结果又出现一个对话框:   
Intermodular calls   
00401164 CALL DWORD PTR DS:\[<&USER32.GetWindowRect>\]  USER32.GetWindowRect   
0040118F CALL DWORD PTR DS:\[<&USER32.GetClientRect>\]  USER32.GetClientRect   
00401279 CALL DWORD PTR DS:\[<&USER32.GetClientRect>\]  USER32.GetClientRect    
004013E1 CALL DWORD PTR DS:\[<&USER32.GetClientRect>\]  USER32.GetClientRect   
0040190A CALL DWORD PTR DS:\[<&KERNEL32.GetThreadLocale>\]   kernel32.GetThreadLocale   
0040191C CALL DWORD PTR DS:\[<&KERNEL32.GetLocaleInfoA>\]    kernel32.GetLocaleInfoA   
    还有很多,我就省略了,在这个框里右键单击,出现一个对话框,选中在每个命令中设置断点(热键是 S),单击,你看每一行   
都变成了红色吧,这表示全部设下断点了,后面的过程很烦燥,你要一边按 F9,边按 F2 把一些无用的断点去掉,有一点耐心吧,   
当然也可以不这样做,但遇到断点跳不过的时候,你就得按 F2 把这个断点去掉.   
感觉在这一点上 OD 比不上 TRW 和 SICE,到出现注册信息框后,你要输入注册信息,然后按确定,程序被断下来,我的序列号是:   
GK342QZ0C6RE03L,我输入试验码:123456789098765.程序中断在下面:   
  
00471F71  CALL DWORD PTR DS:\[<&USER32.GetWindowTex>\*\*\*\*按确定后回到 OD 程序中断在此,按 F8 走,在此还没有到关键处!   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*注意这里会循环两次,分别处理序列号和试验码,按 F8 走   
00471F77  LEA ECX,DWORD PTR DS:\[EAX+1\]   
00471F7A  PUSH ECX   
00471F7B  MOV ECX,DWORD PTR SS:\[EBP+10\]   
00471F7E  PUSH EAX   
00471F7F  CALL SuperPIM.0043E15D   
00471F84  PUSH EAX                                    
00471F85  PUSH ESI                                    
00471F86  CALL DWORD PTR DS:\[<&USER32.GetWindowTex>   
00471F8C  MOV ECX,DWORD PTR SS:\[EBP+10\]   
00471F8F  PUSH -1   
00471F91  CALL SuperPIM.00401D6A   
00471F96  JMP SHORT SuperPIM.00471FA3   
00471F98  MOV EAX,DWORD PTR SS:\[EBP+10\]   
00471F9B  PUSH DWORD PTR DS:\[EAX\]                     
00471F9D  PUSH ESI                                    
00471F9E  CALL SuperPIM.00470B61                     
00471FA4  POP ESI   
00471FA5  POP EBP   
00471FA6  RETN 0C\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*程序第二次循环后最后会返回到 0040FE99 下面看代码:   
\----------------------------------------------------------------------------------------------------------   
0040FE99  LEA EAX,DWORD PTR SS:\[EBP-14\]\*\*\*\*第二次循环后会返回到这里,注意这就是我们要找的关键地方,按 F8 往下!   
0040FE9C  PUSH EAX   
0040FE9D  CALL SuperPIM.00433D92\*\*\*\*\*\*\*\*\*\*\*这就是关键 CALL,按 F7 追进,一定要进,因为算法就在这里面!   
0040FEA2  POP ECX   
0040FEA3  AND DWORD PTR SS:\[EBP-4\],0   
0040FEA7  LEA ESI,DWORD PTR DS:\[EDI+74\]   
0040FEAA  MOV EAX,DWORD PTR DS:\[ESI\]\*\*\*\*\*\*试验码入 EAX   
0040FEAC  CMP DWORD PTR DS:\[EAX-C\],0\*\*\*\*\*\*比较试验码输入了吗?   
0040FEB0  JE SHORT SuperPIM.0040FF2F\*\*\*\*\*\*没有输入就跳走,一跳就失败!   
0040FEB2  PUSH 28   
0040FEB4  LEA EAX,DWORD PTR SS:\[EBP-18\]   
0040FEB7  PUSH ESI   
0040FEB8  PUSH EAX   
0040FEB9  CALL SuperPIM.0042CEC8\*\*\*\*\*\*\*\*\*\*此 CALL 对输入的注册码进行处理,得到一个 40 位的长字符串,有兴趣自己跟,   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*我已是头晕的很,不想跟进!   
0040FEBE  LEA ECX,DWORD PTR SS:\[EBP-14\]   
0040FEC1  PUSH ECX   
0040FEC2  PUSH EAX   
0040FEC3  CALL SuperPIM.0040F9B7\*\*\*\*\*\*\*\*\*\*此 CALL 对序列号处理,也是得到一个 40 位的长字符串   
0040FEC8  MOV ECX,DWORD PTR SS:\[EBP-18\]   
0040FECB  ADD ESP,14   
0040FECE  ADD ECX,-10   
0040FED1  MOV BYTE PTR SS:\[EBP-D\],AL   
0040FED4  CALL SuperPIM.00401B5D\*\*\*\*\*\*\*\*\*\*此 CALL 进行比较,注册码不对,值为 0   
0040FED9  CMP BYTE PTR SS:\[EBP-D\],0\*\*\*\*\*\*\*比较是 0 吗?   
0040FEDD  JE SHORT SuperPIM.0040FF2F\*\*\*\*\*\*相等就跳,跳就失败,所以一定不能跳!   
0040FEDF  PUSH 0\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*不跳往下你就成功了!下面代码省略!   
...............................................................省略若干代码!.........................   
\=============================================================================================================   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*关键 CALL 的代码!\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
00433D92  MOV     EAX, SuperPIM.0049844E\*\*\*\*追进关键 CALL 后我们来到这里!   
00433D97  CALL    SuperPIM.00450900   
00433D9C  PUSH    ECX   
00433D9D  PUSH    ECX   
00433D9E  AND     \[LOCAL.5\], 0   
00433DA2  LEA     EAX, \[LOCAL.5\]   
00433DA5  PUSH    EAX   
00433DA6  CALL    SuperPIM.00433CA9\*\*\*\*\*\*\*\*此 CALL 根据电脑硬件信息得到你的序列号,有兴趣跟吧!我是没有兴趣!   
00433DAB  AND     \[LOCAL.1\], 0   
00433DAF  PUSH    14   
00433DB1  LEA     EAX, \[LOCAL.5\]   
00433DB4  PUSH    EAX   
00433DB5  LEA     EAX, \[LOCAL.4\]   
00433DB8  PUSH    EAX   
00433DB9  CALL    SuperPIM.0042CEC8\*\*\*\*\*\*\*算法 CALL(1),按 F7 跟进!   
00433DBE  PUSH    28   
00433DC0  LEA     EAX, \[LOCAL.4\]   
00433DC3  PUSH    EAX   
00433DC4  PUSH    \[ARG.1\]                             
00433DC7  MOV     BYTE PTR SS:\[EBP-4\], 1   
00433DCB  CALL    SuperPIM.0042CEC8   
00433DD0  MOV     ECX, \[LOCAL.4\]                      
00433DD3  ADD     ESP, 1C   
00433DD6  ADD     ECX, -10   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*省略 N 行代码!\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
0043454B   RETN   
\===========================================================================================================   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*算法 CALL!\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
0042CEC8  MOV     EAX, SuperPIM.00497B52\*\*\*\*\*\*\*\*追进算法 CALL 我们在这里!   
0042CECD  CALL    SuperPIM.00450900   
0042CED2  SUB     ESP, 14   
0042CED5  PUSH    EBX   
0042CED6  PUSH    ESI                                 
0042CED7  XOR     EBX, EBX   
0042CED9  PUSH    EDI   
0042CEDA  MOV     \[LOCAL.8\], EBX   
0042CEDD  CALL    SuperPIM.004639B5   
0042CEE2  MOV     EDX, DWORD PTR DS:\[EAX\]   
0042CEE4  MOV     ECX, EAX                           
0042CEE6  CALL    DWORD PTR DS:\[EDX+C\]   
0042CEE9  LEA     EDI, DWORD PTR DS:\[EAX+10\]   
0042CEEC  MOV     \[LOCAL.4\], EDI   
0042CEEF  MOV     EAX, \[ARG.2\]   
0042CEF2  MOV     EAX, DWORD PTR DS:\[EAX\]\*\*\*\*\*\*\*序列号:GK342QZ0C6RE03L 移入 EAX   
0042CEF4  MOV     ESI, DWORD PTR DS:\[EAX-C\]\*\*\*\*\*序列号的位数 15(十六进制值是 F)送入 ESI   
0042CEF7  CMP     ESI, EBX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*比较 ESI 和 EBX,ESI 的值是 F 即序列号位数,EBX 的初始值是 0   
0042CEF9  MOV     \[LOCAL.1\], EBX   
0042CEFC  MOV     \[LOCAL.8\], ESI                    
0042CEFF  JE      SuperPIM.0042CFB2\*\*\*\*\*\*\*\*\*\*\*\*\*相等就跳,这里当然不相等,所以就不会跳!   
0042CF05  MOV     EAX, \[ARG.3\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*常数 14(十进制值是 20)送入 EAX   
0042CF08  CMP     EAX, ESI\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*比较 EAX 和 ESI,即 14 和 F 比较!                            
0042CF0A  MOV     \[LOCAL.6\], EAX                  
0042CF0D  JG      SHORT SuperPIM.0042CF12\*\*\*\*\*\*\*大于就跳!   
0042CF0F  MOV     \[LOCAL.6\], ESI                 
0042CF12  CMP     \[LOCAL.6\], EBX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*跳到这里!比较 14 和 0   
0042CF15  JLE     SuperPIM.0042CFB2\*\*\*\*\*\*\*\*\*\*\*\*\*小于就跳,这里当然不会小!所以不跳!   
0042CF1B  MOV     EAX, EBX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*0 移入 EAX   
0042CF1D  CDQ\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EDX 清 0,准备计算!   
0042CF1E  IDIV    ESI\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*除,EAX/ESI ,EAX 的值是 0,ESI 的值存放序列号的位数 F                             
0042CF20  MOV     EAX, \[ARG.2\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*赋 EAX 地址值   
0042CF23  MOV     EAX, DWORD PTR DS:\[EAX\]\*\*\*\*\*\*\*序列号:GK342QZ0C6RE03L 移入 EAX   
0042CF25  PUSH    EBX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EBX 入栈   
0042CF26  MOV     AL, BYTE PTR DS:\[EDX+EAX\]\*\*\*\*\*序列号的第一位 G(ASCII 码值 47)入 AL   
0042CF29  MOV     BYTE PTR SS:\[EBP-1C\], AL\*\*\*\*\*\*保存值 47   
0042CF2C  PUSH    \[LOCAL.7\]   
0042CF2F  CALL    SuperPIM.0042CD8E\*\*\*\*\*\*\*\*\*\*\*\*\*算法 CALL(2),按 F7 跟进!代码直接在下面给出   
\==============================================================================================================   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*算法 CALL(2)\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
0042CD8E  PUSH    EBP   
0042CD8F  MOV     EBP, ESP   
0042CD91  PUSH    ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*G 的 ASCII 码值 47 入 ECX   
0042CD92  MOVZX   EAX, BYTE PTR SS:\[EBP+8\]\*\*\*\*扩展成 00000047   
0042CD96  MOV     ECX, \[ARG.2\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*赋 ECX 的初始值为 0   
0042CD99  MOV     \[LOCAL.1\], EAX   
0042CD9C  MOV     EAX, ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*ECX 的值移入 EAX   
0042CD9E  IMUL    EAX, ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EAX=EAX\*ECX=0\*0   
0042CDA1  LEA     EAX, DWORD PTR DS:\[EAX+EAX\*2+7\]\*\*EAX+EAX\*2+7 的值 7 给 EAX   
0042CDA5  IMUL    EAX, ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EAX=EAX\*ECX=7\*0   
0042CDA8  ADD     EAX, 0D\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EAX=EAX+0D=D   
0042CDAB  IMUL    EAX, ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EAX=EAX\*ECX=D\*0=0   
0042CDAE  PUSH    ESI\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*序列号位数 F 入栈!   
0042CDAF  LEA     ESI, DWORD PTR DS:\[ECX+5\]   
0042CDB2  PUSH    ESI   
0042CDB3  MOV     \[ARG.1\], EAX   
0042CDB6  LEA     EAX, \[ARG.1\]   
0042CDB9  PUSH    4   
0042CDBB  PUSH    EAX   
0042CDBC  CALL    SuperPIM.0042CD2E\*\*\*\*\*\*\*\*\*\*\*这个 CALL 也在计算,我实在不想跟进!   
0042CDC1  PUSH    ESI   
0042CDC2  LEA     EAX, \[LOCAL.1\]   
0042CDC5  PUSH    4   
0042CDC7  PUSH    EAX   
0042CDC8  CALL    SuperPIM.0042CD5E\*\*\*\*\*\*\*\*\*\*这个 CALL 也是计算 CALL,烦!不跟了!   
0042CDCD  MOV     EAX, \[ARG.1\]   
0042CDD0  ADD     ESP, 18   
0042CDD3  XOR     EAX, \[LOCAL.1\]\*\*\*\*\*\*\*\*\*\*\*\*\*这里赋 EAX 的值 38000002   
0042CDD6  POP     ESI                                
0042CDD7  LEAVE   
0042CDD8  RETN   
\=========================================================================算法 CALL(2)结束==================   
  
0042CF34  POP     ECX                                
0042CF35  POP     ECX                              
0042CF36  PUSH    2B\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*常数 2B 入栈!   
0042CF38  XOR     EDX, EDX   
0042CF3A  POP     ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*把常数 2B 赋给 ECX   
0042CF3B  DIV     ECX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*除,EAX/ECX=38000002/2B=014D6535,余数 1B 入 EDX   
0042CF3D  MOV     ECX, EDX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*结果 1B 入 ECX   
0042CF3F  ADD     CL, 30\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*CL=CL+30=1B+30=4B(对应的字符串是 K)   
0042CF42  CMP     CL, 39\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*比较是不是数字 9   
0042CF45  MOV     BYTE PTR SS:\[EBP-14\], CL\*\*\*\*\*保存字符串 K   
0042CF48  JLE     SHORT SuperPIM.0042CF55\*\*\*\*\*\*小于就跳走!   
0042CF4A  CMP     CL, 41\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*比较是不是 A   
0042CF4D  JGE     SHORT SuperPIM.0042CF55\*\*\*\*\*\*大于就跳走   
0042CF4F  ADD     CL, 0F6   
0042CF52  MOV     BYTE PTR SS:\[EBP-14\], CL   
0042CF55  CMP     EBX, \[ARG.3\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*跳到这里!比较 0 和 14   
0042CF58  JGE     SHORT SuperPIM.0042CF67\*\*\*\*\*\*大于等于就跳走!   
0042CF5A  PUSH    \[LOCAL.5\]                          
0042CF5D  LEA     ECX, \[LOCAL.4\]   
0042CF60  CALL    SuperPIM.00417EAF   
0042CF65  JMP     SHORT SuperPIM.0042CFA5\*\*\*\*\*\*无条件跳   
0042CF67  MOV     EAX, EBX   
0042CF69  CDQ   
0042CF6A  IDIV    \[ARG.3\]   
0042CF6D  MOVSX   ECX, CL   
0042CF70  PUSH    2B   
0042CF72  MOV     ESI, EDX   
0042CF74  MOVSX   EAX, BYTE PTR DS:\[ESI+EDI\]   
0042CF78  LEA     EAX, DWORD PTR DS:\[EAX+ECX-60\]   
0042CF7C  CDQ   
0042CF7D  POP     ECX                               
0042CF7E  IDIV    ECX   
0042CF80  ADD     DL, 30   
0042CF83  CMP     DL, 39   
0042CF86  MOV     BYTE PTR SS:\[EBP-14\], DL   
0042CF89  JLE     SHORT SuperPIM.0042CF96   
0042CF8B  CMP     DL, 41   
0042CF8E  JGE     SHORT SuperPIM.0042CF96   
0042CF90  ADD     DL, 0F6   
0042CF93  MOV     BYTE PTR SS:\[EBP-14\], DL   
0042CF96  PUSH    \[LOCAL.5\]                         
0042CF99  LEA     ECX, \[LOCAL.4\]   
0042CF9C  PUSH    ESI                                
0042CF9D  CALL    SuperPIM.0042CE4B   
0042CFA2  MOV     ESI, \[LOCAL.8\]                     
0042CFA5  MOV     EDI, \[LOCAL.4\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*上面跳到这里!                     
0042CFA8  INC     EBX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*EBX 加 1   
0042CFA9  CMP     EBX, \[LOCAL.6\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*比较 1 和 14                     
0042CFAC  JL      SuperPIM.0042CF1B\*\*\*\*\*\*\*\*\*\*\*小于就跳,循环,序列号只有 15 位,这里循环 20 次,取完后又从第一位取出!   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*循环结束后得到的值是:K3L2LPBDW1F2H4B8S0UY 这就是我们要的注册码!   
0042CFB2  MOV     ECX, \[ARG.1\]   
0042CFB5  LEA     EAX, \[LOCAL.4\]   
0042CFB8  PUSH    EAX                                
0042CFB9  CALL    SuperPIM.00401F36   
0042CFBE  LEA     ECX, DWORD PTR DS:\[EDI-10\]   
0042CFC1  CALL    SuperPIM.00401B5D   
0042CFC6  MOV     ECX, \[LOCAL.3\]   
0042CFC9  MOV     EAX, \[ARG.1\]   
0042CFCC  POP     EDI                                 
0042CFCD  POP     ESI                                 
0042CFCE  POP     EBX                                 
0042CFCF  MOV     DWORD PTR FS:\[0\], ECX   
0042CFD6  LEAVE   
0042CFD7  RETN   
  
     写这篇文章的目的是想说明如何在 OD 下万能断点,算法过程太烦琐,就不想多跟了!希望能对大家有点启发,如果觉得这篇   
破文还行帮我顶一下吧!也是希望有更多的人能看到,对更多的人有益,我也是一只菜鸟,请大家多多指点!   
     我的序列号:GK342QZ0C6RE03L   
         注册码:K3L2LPBDW1F2H4B8S0UY  
 4dd466d0e1e0cc123afbe504da2462ca