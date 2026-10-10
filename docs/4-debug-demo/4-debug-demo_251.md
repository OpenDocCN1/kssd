

# lingoes 词霸时间限制的爆破原理

> 原文：[`www.pediy.com/kssd/pediy12/128996.html`](https://www.pediy.com/kssd/pediy12/128996.html)

lingoes2.7.0 时间限制的突破                      
                            天易 love   2011-2-6  
    今天偶尔打开该软件，竟然过期了，简单在网上搜了下没找到答案，那就只好自己动手，还好不是很难。感觉掌握点破解知识还是有备无患。  
   原理：试用时间加密后保存在 system.inc 文件中，软件运行时解密该信息并与当前时间比较，如果过期了就弹出强制性升级窗口。  
    一些关键的地方直接给出：  
0046EF54  |.  57            PUSH EDI  
0046EF55  |.  51            PUSH ECX  
0046EF56  |.  53            PUSH EBX  
0046EF57  |.  8D8C24 480700>LEA ECX,DWORD PTR SS:\[ESP+748\]  
0046EF5E  |.  E8 BD150500   CALL Lingoes.004C0520      
上面 call 时堆栈情况如下：  
0012DF18   00F90048  |Arg1 = 00F90048                 //解密数据存放的起始地址  
0012DF1C   0039D020  |Arg2 = 0039D020 ASCII "?OCDD""  //system.inc 中的加密数据  
0012DF20   00002250  \\Arg3 = 00002250  
解密得到到期时间(UNICODE "2011-01-01 01:02:03")  
00F904E8  32 00 30 00 31 00 31 00 2D 00 30 00 31 00 2D 00  2.0.1.1.-.0.1.-.  
00F904F8  30 00 31 00 20 00 30 00 31 00 3A 00 30 00 32 00  0.1. .0.1.:.0.2.  
00F90508  3A 00 30 00 33 00                                :.0.3.  
在内存中把这里的 2011 改为 2012 已经可以正常使用了  
这里直接给出弹框的关键位置  
00480FAC  |.  B9 D8EE5D00   MOV ECX,Lingoes.005DEED8  
00480FB1  |.  E8 AA16FFFF   CALL Lingoes.00472660  //比较是否过期的关键 call          
00480FB6  |.  84C0          TEST AL,AL             // al=0 ok  
00480FB8  |.  0F84 BC000000 JE Lingoes.0048107A  //暴力法 1 改成    JMP 0048107A    
00480FBE  |.  6A 00         PUSH 0  
............................  
00480FF9  |.  C78424 300100>MOV DWORD PTR SS:\[ESP+130\],0  
00481004  |.  E8 DB320900   CALL <JMP.&MFC42u.#2506>  //弹出过期框  
F7 进入关键 call:  
00472660  /$  6A FF         PUSH -1  
00472662  |.  68 52C85600   PUSH Lingoes.0056C852                        
00472667  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
0047266D  |.  50            PUSH EAX  
...............  
0047270F  |.  FF15 AC9A5700 CALL DWORD PTR DS:\[<&MSVCRT.time>\]           
00472715  |.  8B4424 08     MOV EAX,DWORD PTR SS:\[ESP+8\]      
00472719  |.  8B4C24 2C     MOV ECX,DWORD PTR SS:\[ESP+2C\]  
0047271D  |.  83C4 04       ADD ESP,4  
00472720  |.  3BC6          CMP EAX,ESI      //esi 与到期时间有关   eax 与当前时间有关  
00472722  |.  0F9FC0        SETG AL          //al=1 就过期了 暴力法 2 改成  xor al,al  
00472725  |.  5E            POP ESI  
00472726  |.  64:890D 00000>MOV DWORD PTR FS:\[0\],ECX  
0047272D  |.  83C4 30       ADD ESP,30  
00472730  \\.  C3            RETN  
    查了几个单词，内存使用就飙升了，心里很不爽。