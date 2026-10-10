

# 迷你歌词插件

> 原文：[`www.pediy.com/kssd/pediy06/pediy6770.htm`](https://www.pediy.com/kssd/pediy06/pediy6770.htm)

Winamp 有个迷你歌词插件，可以方便地下载、管理歌词。但用了总觉得有个地方不顺手，就是如果 winamp 显示歌曲为“xxx -  无标题 ”时，总是要将后边的词去掉才能正常下载。偏偏我的大多数歌曲都是这种情况。于是就向插件下手了，目标是在搜索歌词前自动去掉“ -  无标题”。  
经过跟踪得知 winamp 是调用 MiniLyrics.dll，如果搜索不到歌词，则弹出对话框“不能找到相匹配的歌词，请您试试用其他歌曲名搜索！”，打开中文菜单文件 mschs.ini，找到这句，前面的英文是：Can't find any match lyrics, please try other name!  
好了，用 OD 载入 winamp，运行，查看 MiniLyrics.dll 模块，搜索 Can't find any match lyrics, 设断点。然后在下载对话框中点击“搜索”按钮，中断在 OD 中。仔细跟踪找到如下代码：  
00FF2E40  SUB ESP,100  
00FF2E46  PUSH ESI  
00FF2E47  PUSH 100  
00FF2E4C  LEA EAX,DWORD PTR SS:\[ESP+8\]  
00FF2E50  PUSH EAX  
00FF2E51  MOV ESI,ECX  
00FF2E53  MOV ECX,DWORD PTR DS:\[ESI+4\]  
00FF2E56  PUSH 44E  
00FF2E5B  PUSH ECX  
00FF2E5C  CALL DWORD PTR DS:\[<&USER32.GetDlgItemTe>; USER32.GetDlgItemTextA  
00FF2E62  MOV AL,BYTE PTR SS:\[ESP+4\]   --->歌曲名  
00FF2E66  TEST AL,AL  
00FF2E68  JNZ SHORT MiniLyri.00FF2EA0  --->有歌名就跳  
00FF2E6A  MOV EDX,DWORD PTR DS:\[ESI+4\]  
00FF2E6D  PUSH 0  
00FF2E6F  PUSH 40  
00FF2E71  PUSH EDX  
00FF2E72  PUSH MiniLyri.010368F0                    ASCII "Please input the song name!"  
00FF2E77  MOV ECX,MiniLyri.01043898  
00FF2E7C  CALL MiniLyri.00FE96D0  
00FF2E81  PUSH EAX  
00FF2E82  CALL MiniLyri.00FE7BC0  
程序跳到这：  
00FF2EA0  PUSH EBX  
00FF2EA1  PUSH EBP  
00FF2EA2  PUSH EDI  
00FF2EA3  PUSH 7F02  
00FF2EA8  PUSH 0  
00FF2EAA  CALL DWORD PTR DS:\[<&USER32.LoadCursorA>>; USER32.LoadCursorA  
00FF2EB0  MOV EDI,EAX  
00FF2EB2  CALL DWORD PTR DS:\[<&USER32.GetCursor>\]   USER32.GetCursor  
00FF2EB8  MOV EBP,DWORD PTR DS:\[<&USER32.SetCursor>; USER32.SetCursor  
00FF2EBE  PUSH EDI  
00FF2EBF  MOV EBX,EAX  
00FF2EC1  CALL EBP  
00FF2EC3  PUSH 1  
00FF2EC5  LEA ECX,DWORD PTR SS:\[ESP+14\]  
00FF2EC9  PUSH ECX  
00FF2ECA  MOV ECX,ESI  
00FF2ECC  CALL MiniLyri.00FF2B40  
00FF2ED1  TEST EAX,EAX  
00FF2ED3  PUSH EBX  
00FF2ED4  JNZ SHORT MiniLyri.00FF2F03  
00FF2ED6  CALL EBP  
00FF2ED8  MOV EDX,DWORD PTR DS:\[ESI+4\]  
00FF2EDB  PUSH 0  
00FF2EDD  PUSH 40  
00FF2EDF  PUSH EDX  
00FF2EE0  PUSH MiniLyri.010368BC                    ASCII "Can't find any match lyrics, please try other name!"  
00FF2EE5  MOV ECX,MiniLyri.01043898  
00FF2EEA  CALL MiniLyri.00FE96D0  
  
修改如下：  
00FF2EA0  JMP MiniLyri.01033490     --->跳到修改的代码  
00FF2EA5  NOP                       --->补充字节，下同  
00FF2EA6  NOP  
00FF2EA7  NOP  
跳到这修改：  
01033490  PUSHAD                    --->保存堆栈数据  
01033491  MOV EBX,25                --->esp 减小了 20，故应加回  
01033496  MOV AL,BYTE PTR DS:\[ESP+EBX\]  --->歌曲名第二字节  
0103349A  CMP AL,0                      --->没有？  
0103349C  JE SHORT MiniLyri.010334D0    --->跳到结束  
0103349E  INC BL                          
010334A0  CMP AL,20                     --->等于 20？(空格)  
010334A2  JE SHORT MiniLyri.010334A6    --->进一步判断  
010334A4  JMP SHORT MiniLyri.01033496   --->循环  
010334A6  MOV AL,BYTE PTR SS:\[ESP+EBX\]  --->下一字节  
010334A9  INC BL  
010334AB  CMP AL,2D                     --->等于 2D？即"-"号  
010334AD  JE SHORT MiniLyri.010334B1  
010334AF  JMP SHORT MiniLyri.01033496  
010334B1  MOV AL,BYTE PTR SS:\[ESP+EBX\]  
010334B4  INC BL  
010334B6  CMP AL,20  
010334B8  JE SHORT MiniLyri.010334BC  
010334BA  JMP SHORT MiniLyri.01033496  
010334BC  MOV AL,BYTE PTR SS:\[ESP+EBX\]  
010334BF  INC BL  
010334C1  CMP AL,20  
010334C3  JE SHORT MiniLyri.010334C7  
010334C5  JMP SHORT MiniLyri.01033496  
010334C7  SUB BL,4                      --->下面不想判断了，有“ -  "四个字节就当作无用的，将其置 0，先回到第一个空格  
010334CA  NOP  
010334CB  NOP  
010334CC  NOP  
010334CD  NOP  
010334CE  NOP  
010334CF  NOP  
010334D0  NOP  
010334D1  NOP  
010334D2  NOP  
010334D3  MOV DWORD PTR SS:\[ESP+EBX\],0      --->四个字节置 0  
010334DA  POPAD                             --->恢复堆栈数据  
010334DB  PUSH EBX                    --->恢复原代码，下同  
010334DC  PUSH EBP  
010334DD  PUSH EDI  
010334DE  PUSH 7F02  
010334E3  JMP MiniLyri.00FF2EA8       --->返回原程序继续执行  
  
  
再次运行，OK!  
首次用汇编代码修改 PE 功能，花了 5 个多小时，中间走了些弯路，真的好累!  
  
  
\--------------------------------------------------------------------------------  
  
标 题:顺便帖一下让 winamp 任务栏滚动文字正常显示中文 (1 千字)  
发信人:lzqgj    
时 间:2003-08-17 23:26:06  
详细信息:  
  
  
  
winamp 任务栏可以滚动显示歌名，但如果是中文，将会时而出现乱码，因为中文是双字节，而 winamp 并没有考虑，仍当作单字节处理。当滚过第一字节后，第二字节就无法正常显示。  
用 OD 载入 winamp，下断点：setwindowtexta，中断后回到这里：  
0042EE53  LEA EAX,DWORD PTR \[EBP\-1000\]    --->\[EBP\-1000\]为任务栏中要显示的文字  
0042EE59  PUSH EAX                          
0042EE5A  PUSH DWORD PTR DS:\[453320\]      --->HWnd  
0042EE60  CALL DWORD PTR DS:\[<&USER32.SetWindo>; \\SetWindowTextA  
向上分析文字是怎么滚动的：  
0042ED40  PUSH ESI                             --->文字    
0042ED41  STOS BYTE PTR ES:\[EDI\]                 
0042ED42  CALL             --->长度  
0042ED47  POP ECX  
0042ED48  MOV ECX,DWORD PTR DS:\[451248\]        --->重要：取第几字节的记数器  
0042ED4E  CMP ECX,EAX  
0042ED50  JGE SHORT Winamp.0042EDA9            --->大于就跳，计数器重新置 0  
0042ED52  LEA EAX,DWORD PTR DS:\[ECX+4538C0\]    --->\[4538C0\]中放置完整的文字  
0042ED58  PUSH EAX                               
0042ED59  LEA EAX,DWORD PTR SS:\[EBP\-1000\]        
0042ED5F  PUSH EAX                               
0042ED60  CALL             --->取得文字复制到\[EBP\-1000\]中  
下略。  
明显程序用\[451248\]记忆取到第几位数，然后取得文字复制到\[EBP\-1000\]中让任务栏显示，只要判断取得的第一个文字是否中文，是则将计数器加 1，就可跳过该中文的第二字节。  
  
修改程序如下：  
0042EE53  JMP Winamp.0043D4E0        --->在任务栏显示前先处理文字  
0043D4E0  MOV AL,BYTE PTR SS:\[EBP\-1000\]   --->取第一字节  
0043D4E6  CMP AL,9F                  --->中文一般大于 9F?我猜的  
0043D4E8  JB SHORT Winamp.0043D4F0   --->是则跳  
0043D4EA  INC DWORD PTR DS:\[451248\]  --->计数器加 1  
0043D4F0  LEA EAX,DWORD PTR SS:\[EBP\-1000\]   --->恢复原代码  
0043D4F6  JMP Winamp.0042EE59        --->回原程序继续执行   看雪学院技术论坛([www.pediy.com](http://www.pediy.com)) 并保持文章的完整性！   
  
  
\--------------------------------------------------------------------------------  

 4dd466d0e1e0cc123afbe504da2462ca