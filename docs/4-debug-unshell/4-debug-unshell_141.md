

# ACProtect\_122b\_pro 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6487.htm`](https://www.pediy.com/kssd/pediy06/pediy6487.htm)

【前    言】：我现在找到一个更加方便的方法.而且也破解了一个限制.所以重新写下了这篇.由于这个壳有很多特别的地方,所以写笔记的时候主要是用自己的思想去写的.希望读者可以看明白和学会!   
  
【软件名称】：ACProtect\_122b\_pro   
  
【下载页面】：不记得了   
  
【软件大小】：828,282 字节  ----->   3,567,616 字节   
  
【应用平台】：WINXP   
  
【软件简介】：一个加密器   
  
【软件限制】：超级 BT 的 OEP 抽走代码.SDK 保护.数据转移   
  
【文章作者】：辉仔 Yock   
  
【作者声明】：本人读书少,比较笨,表达也不好,有什么地方写的不好,还请大侠多多指出,我好及时改正!   
  
【破解工具】：Yock-OD    LordPE    ImportREC   
  
—————————————————————————————————    
【过    程】：   
  
  
//首先载入.忽略所有异常( 除内存异常之外 ) 还有就是隐藏 OD 了   
  
006D5000 >  60              PUSHAD   
//壳的 OEP   
//重要   
  
006D5001    87EA            XCHG    EDX, EBP   
006D5003    66:23EF         AND     BP, DI   
006D5006    66:8BD3         MOV     DX, BX   
006D5009    FC              CLD   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//这个壳有一个很有意思的地方,就是边走边解码,让我联想起一首歌的名字------边走边爱   
006D5BB3    81CA 4FF34B51   OR      EDX, 514BF34F   
006D5BB9    81C3 4B8BAC86   ADD     EBX, 86AC8B4B   
006D5BBF    BE 4F460000     MOV     ESI, 464F   
006D5BC4    8B0F            MOV     ECX, \[EDI\]   
006D5BC6    03CB            ADD     ECX, EBX   
006D5BC8    C1C9 07         ROR     ECX, 7   
006D5BCB    83EF FC         SUB     EDI, -4   
006D5BCE    2B0F            SUB     ECX, \[EDI\]   
006D5BD0    83C7 FC         ADD     EDI, -4   
006D5BD3    890F            MOV     \[EDI\], ECX   
006D5BD5    81F3 A4F8B360   XOR     EBX, 60B3F8A4   
006D5BDB    81EF FCFFFFFF   SUB     EDI, -4   
006D5BE1    83EE 01         SUB     ESI, 1   
006D5BE4  ^ 0F85 DAFFFFFF   JNZ     006D5BC4   
006D5BEA    E9 A9180100     JMP     006E7498   
//来到这里就要开始了.注意了.   
006D5BEF    0000            ADD     \[EAX\], AL   
006D5BF1    0000            ADD     \[EAX\], AL   
006D5BF3    0000            ADD     \[EAX\], AL   
006D5BF5    0000            ADD     \[EAX\], AL   
006D5BF7    0000            ADD     \[EAX\], AL   
\---------------------------------------------   
//上面跳到这里:   
//来到这里马上豁然开朗.   
//这里有很多 CALL,里面的用处都有不同的.我会在下面分别对重要的 CALL 进行简单分析.   
  
006E7498    E8 4ABBFFFF     CALL    006E2FE7   
//这个 CALL 主要是取得一个固定的值放进去 EBP 里面作为一个固定量.   
006E749D    E8 00000000     CALL    006E74A2   
006E74A2    5B              POP     EBX   
006E74A3    2B9D F6234000   SUB     EBX, \[EBP+4023F6\]   
006E74A9    81EB A2240100   SUB     EBX, 124A2   
006E74AF    899D 31DE4000   MOV     \[EBP+40DE31\], EBX   
//上面这四个指令是计算出 400000 这个值.保存起来   
006E74B5    E8 CCFDFFFF     CALL    006E7286   
//用 OEP 的值计算某某数值作为一个 Key,用处没有多大留意了!   
006E74BA    E8 28BBFFFF     CALL    006E2FE7   
006E74BF    E8 C1B8FFFF     CALL    006E2D85   
//根据之前的 Key 再次计算.   
006E74C4    8985 8CED4000   MOV     \[EBP+40ED8C\], EAX   
//最终的 Key 保存起来.其实是一个效验.对于密码学菜的我就没有花什么时间跟这里了.   
006E74CA    E8 18BBFFFF     CALL    006E2FE7   
006E74CF    C685 57DD4000 0>MOV     BYTE PTR \[EBP+40DD57\], 0   
006E74D6    E8 46DDFFFF     CALL    006E5221   
//这里是我问了一下 jeff 才知道是 ACP 这个系统的 AIP 来的.有意思...(jeff 说和跨平台有很大关系.所以在这里提提)   
006E74DB    E8 DEDAFFFF     CALL    006E4FBE   
//这里是代码解码处.解码完毕之后会把 006E4FBE 这个地址改成一个 C3(够心思,不过没有什么用)   
006E74E0    E8 5BFDFFFF     CALL    006E7240   
//这里就是转移数据到低位.(需要进去做做手脚,注意!!现在最新版 1.22d 版的对于这里有一下效验.这个版的没有)   
//再往下走就是 反跟踪 反调试 IAT 解码   
006E74E5    E8 FDBAFFFF     CALL    006E2FE7   
006E74EA    6A 00           PUSH    0   
006E74EC    E8 0A000000     CALL    006E74FB   
006E74F1    41              INC     ECX   
006E74F2    43              INC     EBX   
006E74F3    50              PUSH    EAX   
006E74F4    72 6F           JB      SHORT 006E7565   
006E74F6    74 65           JE      SHORT 006E755D   
006E74F8    637400 E8       ARPL    \[EAX+EAX\-18\], SI   
006E74FC    25 00000050     AND     EAX, 50000000   
006E7501    72 6F           JB      SHORT 006E7572   
..............   
..........省略 N 多的小花...   
.............   
  
//由于花指令太多.所以列出一些关键的 CALL   
//这里 CALL 都藏在小花里面的,跟踪时需要细心留神   
  
006E7795    E8 9DC3FFFF     CALL    006E3B37   
//里面是一段小花,然后到达小黑(黑名单)   
//然后返回   
..............   
..........省略 N 多的小花...   
.............   
  
006E79C7    E8 53D1FFFF     CALL    006E4B1F    
//进入之后是一段小花后到达下面检查调试器   
//kernel32.IsDebuggerPresent   
//如果发现调试器的话,不是杀掉,而是把程序代码改一下,比如 E9 改成 01.呵呵,后果可想而知(够阴险)   
//然后返回   
  
..............   
..........省略 N 多的小花...   
.............   
006E7B91    E8 10C3FFFF     CALL    006E3EA6   
//里面的东西由于自己水平低下而看不懂.就下面这三个 AIP   
//kernel32.GetCurrentThread   
//kernel32.GetThreadContext   
//kernel32.SetThreadContext   
//返回   
  
..............   
..........省略 N 多的小花...   
.............   
  
006E7BAC    E8 6ECFFFFF     CALL    006E4B1F   
//我靠,进去再次检查调试器!!!   
//如果发现调试器的话,不是杀掉,而是把程序代码改一下,比如 E9 改成 01.呵呵,后果可想而知(够阴险)   
  
..............   
..........省略 N 多的小花...   
.............   
  
006E7C57    E8 C3CEFFFF     CALL    006E4B1F                         ; ACProtec.006E4B1F   
//我靠,再次检查调试器!!!发神经!   
//如果发现调试器的话,不是杀掉,而是把程序代码改一下,比如 E9 改成 01.呵呵,后果可想而知(够阴险)   
//其实来到这里进入之后是一个 C3   
  
..............   
..........省略 N 多的小花...   
.............   
  
//来到下面这个 CALL 的时候就需要注意了.经过了上面的抢林弹雨,是时候进入这个 XX 的私密之处咯.   
006E7CA7    E8 40DDFFFF     CALL    006E59EC                         ; ACProtec.006E59EC   
//进入之后来到下面关键一   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\--->   
//006E7CA7 处的 CALL 来到这里!   
  
006E5B97    E8 4BD4FFFF     CALL    006E2FE7                         ; ACProtec.006E2FE7   
006E5B9C    80BD 472D4100 C>CMP     BYTE PTR \[EBP+412D47\], 0C3   
//这里是比较下面 006E5BA9 的 CALL 进入后的第一条指令是否 C3.是的话就跳过(多余)   
006E5BA3    74 09           JE      SHORT 006E5BAE                   ; ACProtec.006E5BAE   
006E5BA5    90              NOP   
006E5BA6    90              NOP   
006E5BA7    90              NOP   
006E5BA8    90              NOP   
006E5BA9    E8 99110000     CALL    006E6D47                         ; ACProtec.006E6D47   
//IAT 解码处.这里留在最后.因为这个壳的 IAT 没有难度.因此不详细说了!   
  
006E5BAE    66:C785 17DB400>MOV     WORD PTR \[EBP+40DB17\], 0   
006E5BB7    83BD E8AC4100 0>CMP     DWORD PTR \[EBP+41ACE8\], 0   
006E5BBE    0F84 BF000000   JE      006E5C83                         ; ACProtec.006E5C83   
006E5BC4    8DB5 93D94000   LEA     ESI, \[EBP+40D993\]   
006E5BCA    56              PUSH    ESI   
006E5BCB    68 FF000000     PUSH    0FF   
006E5BD0    FF95 46E34000   CALL    \[EBP+40E346\]   
//取得系统临时目录的路径   
006E5BD6    8DBD 93D94000   LEA     EDI, \[EBP+40D993\]   
006E5BDC    33C0            XOR     EAX, EAX   
006E5BDE    F2:AE           REPNE   SCAS BYTE PTR ES:\[EDI\]   
006E5BE0    8DB5 A3DA4000   LEA     ESI, \[EBP+40DAA3\]   
006E5BE6    4F              DEC     EDI   
006E5BE7    B9 0B000000     MOV     ECX, 0B   
006E5BEC    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR\>   
006E5BEE    8DB5 93D94000   LEA     ESI, \[EBP+40D993\]   
006E5BF4    6A 00           PUSH    0   
006E5BF6    6A 20           PUSH    20   
006E5BF8    6A 02           PUSH    2   
006E5BFA    6A 00           PUSH    0   
006E5BFC    6A 03           PUSH    3   
006E5BFE    68 000000C0     PUSH    C0000000   
006E5C03    56              PUSH    ESI   
006E5C04    FF95 16E34000   CALL    \[EBP+40E316\]   
//查找是否有 perplex.dll   
006E5C0A    0BC0            OR      EAX, EAX   
006E5C0C    74 75           JE      SHORT 006E5C83                   ; ACProtec.006E5C83   
//没有怎么办呢?自己分泌一个.我操   
006E5C0E    90              NOP   
006E5C0F    90              NOP   
006E5C10    90              NOP   
006E5C11    90              NOP   
006E5C12    50              PUSH    EAX   
006E5C13    6A 00           PUSH    0   
006E5C15    8DB5 9FDA4000   LEA     ESI, \[EBP+40DA9F\]   
006E5C1B    56              PUSH    ESI   
006E5C1C    FFB5 ECAC4100   PUSH    DWORD PTR \[EBP+41ACEC\]   
006E5C22    8B9D E8AC4100   MOV     EBX, \[EBP+41ACE8\]   
006E5C28    039D 31DE4000   ADD     EBX, \[EBP+40DE31\]   
006E5C2E    2B9D ECAC4100   SUB     EBX, \[EBP+41ACEC\]   
006E5C34    53              PUSH    EBX   
006E5C35    50              PUSH    EAX   
006E5C36    FF95 3EE34000   CALL    \[EBP+40E33E\]   
006E5C3C    FF95 12E34000   CALL    \[EBP+40E312\]   
//分泌就是那么简单!呵呵   
006E5C42    8DB5 93D94000   LEA     ESI, \[EBP+40D993\]   
006E5C48    56              PUSH    ESI   
006E5C49    FF95 88B04100   CALL    \[EBP+41B088\]                     ; kernel32.LoadLibraryA   
//......   
006E5C4F    0BC0            OR      EAX, EAX   
006E5C51    74 30           JE      SHORT 006E5C83                   ; ACProtec.006E5C83   
006E5C53    90              NOP   
006E5C54    90              NOP   
006E5C55    90              NOP   
006E5C56    90              NOP   
006E5C57    8985 29DE4000   MOV     \[EBP+40DE29\], EAX   
006E5C5D    8DB5 AFDA4000   LEA     ESI, \[EBP+40DAAF\]   
006E5C63    56              PUSH    ESI   
006E5C64    FFB5 29DE4000   PUSH    DWORD PTR \[EBP+40DE29\]   
006E5C6A    FF95 80B04100   CALL    \[EBP+41B080\]                     ; kernel32.GetProcAddress   
006E5C70    8985 AFDA4000   MOV     \[EBP+40DAAF\], EAX   
//EAX=10002DB0   
006E5C76    0BC0            OR      EAX, EAX   
006E5C78    74 09           JE      SHORT 006E5C83                   ; ACProtec.006E5C83   
//是否找到这个 DLL 的口子.是的话就进入   
006E5C7A    90              NOP   
006E5C7B    90              NOP   
006E5C7C    90              NOP   
006E5C7D    90              NOP   
006E5C7E    E8 20F8FFFF     CALL    006E54A3                         ; ACProtec.006E54A3   
//跟进去瞧瞧是啥样子的娘们   
006E5C83    60              PUSHAD   
006E5C84    E8 00000000     CALL    006E5C89                         ; ACProtec.006E5C89   
006E5C89    5E              POP     ESI                              ; ACProtec.006E7D5C   
006E5C8A    83EE 06         SUB     ESI, 6   
006E5C8D    B9 EC000000     MOV     ECX, 0EC   
006E5C92    29CE            SUB     ESI, ECX                         ; ACProtec.006E7C00   
006E5C94    BA 22F93191     MOV     EDX, 9131F922   
006E5C99    C1E9 02         SHR     ECX, 2   
006E5C9C    83E9 02         SUB     ECX, 2   
006E5C9F    83F9 00         CMP     ECX, 0   
006E5CA2    7C 1A           JL      SHORT 006E5CBE                   ; ACProtec.006E5CBE   
006E5CA4    8B048E          MOV     EAX, \[ESI+ECX\*4\]   
006E5CA7    8B5C8E 04       MOV     EBX, \[ESI+ECX\*4+4\]   
006E5CAB    2BC3            SUB     EAX, EBX   
006E5CAD    C1C8 0B         ROR     EAX, 0B   
006E5CB0    33C2            XOR     EAX, EDX   
006E5CB2    81EA B1E7B51F   SUB     EDX, 1FB5E7B1   
006E5CB8    89048E          MOV     \[ESI+ECX\*4\], EAX   
006E5CBB    49              DEC     ECX                              ; ACProtec.006E7C00   
006E5CBC  ^ EB E1           JMP     SHORT 006E5C9F                   ; ACProtec.006E5C9F   
006E5CBE    61              POPAD   
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\--->   
//006E5C7E 处的 CALL 进入之后是小花,然后来到下面:   
//下面这段我看不懂.比较晕...   
//仔细跟了一下是一个把程序自己的路径转换成为 Key.dat.   
//应该是准备计算注册 Key 是否合格了!   
  
006E5669    FFB5 FA234000   PUSH    DWORD PTR \[EBP+4023FA\]           ; kernel32.77E40000   
006E566F    FF95 80B04100   CALL    \[EBP+41B080\]                     ; kernel32.GetProcAddress   
006E5675    FFD0            CALL    EAX   
//kernel32.GetCommandLineA   
006E5677    8BF0            MOV     ESI, EAX   
006E5679    8DBD 93D94000   LEA     EDI, \[EBP+40D993\]   
006E567F    AC              LODS    BYTE PTR \[ESI\]   
006E5680    0AC0            OR      AL, AL   
006E5682    74 30           JE      SHORT 006E56B4                   ; ACProtec.006E56B4   
006E5684    90              NOP   
006E5685    90              NOP   
006E5686    90              NOP   
006E5687    90              NOP   
006E5688    3C 5C           CMP     AL, 5C   
006E568A    75 0B           JNZ     SHORT 006E5697                   ; ACProtec.006E5697   
006E568C    90              NOP   
006E568D    90              NOP   
006E568E    90              NOP   
006E568F    90              NOP   
006E5690    C685 68164100 0>MOV     BYTE PTR \[EBP+411668\], 1   
006E5697    817E FB 2E45584>CMP     DWORD PTR \[ESI\-5\], 4558452E   
006E569E    74 14           JE      SHORT 006E56B4                   ; ACProtec.006E56B4   
006E56A0    90              NOP   
006E56A1    90              NOP   
006E56A2    90              NOP   
006E56A3    90              NOP   
006E56A4    817E FB 2E65786>CMP     DWORD PTR \[ESI\-5\], 6578652E   
006E56AB    74 07           JE      SHORT 006E56B4                   ; ACProtec.006E56B4   
006E56AD    90              NOP   
006E56AE    90              NOP   
006E56AF    90              NOP   
006E56B0    90              NOP   
006E56B1    AA              STOS    BYTE PTR ES:\[EDI\]   
006E56B2  ^ EB CB           JMP     SHORT 006E567F                   ; ACProtec.006E567F   
006E56B4    4F              DEC     EDI   
006E56B5    8A07            MOV     AL, \[EDI\]   
006E56B7    0AC0            OR      AL, AL   
006E56B9    74 08           JE      SHORT 006E56C3                   ; ACProtec.006E56C3   
006E56BB    90              NOP   
006E56BC    90              NOP   
006E56BD    90              NOP   
006E56BE    90              NOP   
006E56BF    3C 5C           CMP     AL, 5C   
006E56C1  ^ 75 F1           JNZ     SHORT 006E56B4                   ; ACProtec.006E56B4   
006E56C3    47              INC     EDI   
006E56C4    80BD 68164100 0>CMP     BYTE PTR \[EBP+411668\], 1   
006E56CB    74 0A           JE      SHORT 006E56D7                   ; ACProtec.006E56D7   
006E56CD    90              NOP   
006E56CE    90              NOP   
006E56CF    90              NOP   
006E56D0    90              NOP   
006E56D1    8DBD 93D94000   LEA     EDI, \[EBP+40D993\]   
006E56D7    8DB5 93DA4000   LEA     ESI, \[EBP+40DA93\]   
006E56DD    B9 0C000000     MOV     ECX, 0C   
006E56E2    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR\>   
006E56E4    80BD 68164100 0>CMP     BYTE PTR \[EBP+411668\], 1   
006E56EB    74 0F           JE      SHORT 006E56FC                   ; ACProtec.006E56FC   
006E56ED    90              NOP   
006E56EE    90              NOP   
006E56EF    90              NOP   
006E56F0    90              NOP   
006E56F1    8DBD 93DA4000   LEA     EDI, \[EBP+40DA93\]   
006E56F7    EB 14           JMP     SHORT 006E570D                   ; ACProtec.006E570D   
006E56F9    90              NOP   
006E56FA    90              NOP   
006E56FB    90              NOP   
006E56FC    66:8B07         MOV     AX, \[EDI\]   
006E56FF    66:3D 3A5C      CMP     AX, 5C3A   
006E5703    74 07           JE      SHORT 006E570C                   ; ACProtec.006E570C   
006E5705    90              NOP   
006E5706    90              NOP   
006E5707    90              NOP   
006E5708    90              NOP   
006E5709    4F              DEC     EDI   
006E570A  ^ EB F0           JMP     SHORT 006E56FC                   ; ACProtec.006E56FC   
006E570C    4F              DEC     EDI   
006E570D    6A 00           PUSH    0   
006E570F    68 80000000     PUSH    80   
006E5714    6A 03           PUSH    3   
006E5716    6A 00           PUSH    0   
006E5718    6A 01           PUSH    1   
006E571A    68 00000080     PUSH    80000000   
006E571F    57              PUSH    EDI   
006E5720    FF95 16E34000   CALL    \[EBP+40E316\]                     ; kernel32.CreateFileA   
//打开 key.dat   
006E5726    0BC0            OR      EAX, EAX   
006E5728    75 15           JNZ     SHORT 006E573F                   ; ACProtec.006E573F   
006E572A    90              NOP   
006E572B    90              NOP   
006E572C    90              NOP   
006E572D    90              NOP   
006E572E    6A 00           PUSH    0   
006E5730    57              PUSH    EDI   
006E5731    57              PUSH    EDI   
006E5732    6A 00           PUSH    0   
006E5734    FF95 90B04100   CALL    \[EBP+41B090\]                     ; USER32.MessageBoxA   
006E573A    E9 26020000     JMP     006E5965                         ; ACProtec.006E5965   
006E573F    50              PUSH    EAX   
006E5740    8DB5 9FDA4000   LEA     ESI, \[EBP+40DA9F\]   
006E5746    8DBD 17DB4000   LEA     EDI, \[EBP+40DB17\]   
006E574C    6A 00           PUSH    0   
006E574E    56              PUSH    ESI   
006E574F    68 00020000     PUSH    200   
006E5754    57              PUSH    EDI   
006E5755    50              PUSH    EAX   
006E5756    FF95 3AE34000   CALL    \[EBP+40E33A\]                     ; kernel32.ReadFile   
006E575C    FF95 12E34000   CALL    \[EBP+40E312\]                     ; kernel32.CloseHandle   
006E5762    81BD 9FDA4000 0>CMP     DWORD PTR \[EBP+40DA9F\], 200   
//比较这个 Key.dat 是否 200 字节   
006E576C    0F85 F3010000   JNZ     006E5965                         ; ACProtec.006E5965   
006E5772    8DB5 E6D64000   LEA     ESI, \[EBP+40D6E6\]   
006E5778    8DBD E7DB4000   LEA     EDI, \[EBP+40DBE7\]   
006E577E    B9 28000000     MOV     ECX, 28   
006E5783    F3:A7           REPE    CMPS DWORD PTR ES:\[EDI\], DWORD P>   
006E5785    83F9 00         CMP     ECX, 0   
006E5788    74 31           JE      SHORT 006E57BB                   ; ACProtec.006E57BB   
006E578A    90              NOP   
006E578B    90              NOP   
006E578C    90              NOP   
006E578D    90              NOP   
006E578E    8DB5 A5174100   LEA     ESI, \[EBP+4117A5\]   
006E5794    6A 00           PUSH    0   
006E5796    56              PUSH    ESI   
006E5797    56              PUSH    ESI   
006E5798    6A 00           PUSH    0   
006E579A    FF95 90B04100   CALL    \[EBP+41B090\]                     ; USER32.MessageBoxA   
006E57A0    E9 C0010000     JMP     006E5965                         ; ACProtec.006E5965   
006E57A5    6C              INS     BYTE PTR ES:\[EDI\], DX            ; I/O command   
006E57A6    6963 65 6E73652>IMUL    ESP, \[EBX+65\], 2065736E   
006E57AD    76 65           JBE     SHORT 006E5814                   ; ACProtec.006E5814   
006E57AF    72 73           JB      SHORT 006E5824                   ; ACProtec.006E5824   
006E57B1    696F 6E 2065727>IMUL    EBP, \[EDI+6E\], 72726520   
006E57B8    6F              OUTS    DX, DWORD PTR ES:\[EDI\]           ; I/O command   
006E57B9    72 00           JB      SHORT 006E57BB                   ; ACProtec.006E57BB   
006E57BB    8DB5 93D74000   LEA     ESI, \[EBP+40D793\]   
006E57C1    8DBD 17DB4000   LEA     EDI, \[EBP+40DB17\]   
006E57C7    833E 00         CMP     DWORD PTR \[ESI\], 0   
006E57CA    74 54           JE      SHORT 006E5820                   ; ACProtec.006E5820   
006E57CC    90              NOP   
006E57CD    90              NOP   
006E57CE    90              NOP   
006E57CF    90              NOP   
006E57D0    813E 20202020   CMP     DWORD PTR \[ESI\], 20202020   
006E57D6    74 48           JE      SHORT 006E5820                   ; ACProtec.006E5820   
006E57D8    90              NOP   
006E57D9    90              NOP   
006E57DA    90              NOP   
006E57DB    90              NOP   
006E57DC    B9 20000000     MOV     ECX, 20   
006E57E1    F3:A6           REPE    CMPS BYTE PTR ES:\[EDI\], BYTE PTR\>   
006E57E3    0BC9            OR      ECX, ECX   
006E57E5    74 13           JE      SHORT 006E57FA                   ; ACProtec.006E57FA   
006E57E7    90              NOP   
006E57E8    90              NOP   
006E57E9    90              NOP   
006E57EA    90              NOP   
006E57EB    807E FF 00      CMP     BYTE PTR \[ESI\-1\], 0   
006E57EF    74 09           JE      SHORT 006E57FA                   ; ACProtec.006E57FA   
006E57F1    90              NOP   
006E57F2    90              NOP   
006E57F3    90              NOP   
006E57F4    90              NOP   
006E57F5    83C6 20         ADD     ESI, 20   
006E57F8  ^ EB CD           JMP     SHORT 006E57C7                   ; ACProtec.006E57C7   
006E57FA    8DB5 11184100   LEA     ESI, \[EBP+411811\]   
006E5800    6A 00           PUSH    0   
006E5802    56              PUSH    ESI   
006E5803    56              PUSH    ESI   
006E5804    6A 00           PUSH    0   
006E5806    FF95 90B04100   CALL    \[EBP+41B090\]                     ; USER32.MessageBoxA   
006E580C    E9 54010000     JMP     006E5965                         ; ACProtec.006E5965   
006E5811    73 74           JNB     SHORT 006E5887                   ; ACProtec.006E5887   
006E5813    6F              OUTS    DX, DWORD PTR ES:\[EDI\]           ; I/O command   
006E5814    6C              INS     BYTE PTR ES:\[EDI\], DX            ; I/O command   
006E5815    65:6E           OUTS    DX, BYTE PTR ES:\[EDI\]            ; I/O command   
006E5817    206C69 63       AND     \[ECX+EBP\*2+63\], CH   
006E581B    65:6E           OUTS    DX, BYTE PTR ES:\[EDI\]            ; I/O command   
006E581D    73 65           JNB     SHORT 006E5884                   ; ACProtec.006E5884   
006E581F    90              NOP   
006E5820    6A 10           PUSH    10   
006E5822    8DB5 67DD4000   LEA     ESI, \[EBP+40DD67\]   
006E5828    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5829    8DB5 F7DB4000   LEA     ESI, \[EBP+40DBF7\]   
006E582F    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5830    8DB5 E7DB4000   LEA     ESI, \[EBP+40DBE7\]   
006E5836    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5837    8DB5 87DC4000   LEA     ESI, \[EBP+40DC87\]   
006E583D    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E583E    FF95 AFDA4000   CALL    \[EBP+40DAAF\]   
//进入 DLL 里面.   
006E5844    83C4 14         ADD     ESP, 14   
006E5847    B9 10000000     MOV     ECX, 10   
006E584C    8DB5 17DB4000   LEA     ESI, \[EBP+40DB17\]   
006E5852    8DBD 67DD4000   LEA     EDI, \[EBP+40DD67\]   
006E5858    AD              LODS    DWORD PTR \[ESI\]   
006E5859    8B1F            MOV     EBX, \[EDI\]   
006E585B    83C7 04         ADD     EDI, 4   
006E585E    49              DEC     ECX   
006E585F    74 4D           JE      SHORT 006E58AE                          ; ACProtec.006E58AE   
006E5861    90              NOP   
006E5862    90              NOP   
006E5863    90              NOP   
006E5864    90              NOP   
006E5865    3BC3            CMP     EAX, EBX   
006E5867    75 06           JNZ     SHORT 006E586F                          ; ACProtec.006E586F   
//比较是否合法   
006E5869    90              NOP   
006E586A    90              NOP   
006E586B    90              NOP   
006E586C    90              NOP   
006E586D  ^ EB E9           JMP     SHORT 006E5858                          ; ACProtec.006E5858   
006E586F    8DB5 86184100   LEA     ESI, \[EBP+411886\]   
006E5875    6A 00           PUSH    0   
006E5877    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5878    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5879    6A 00           PUSH    0   
006E587B    FF95 90B04100   CALL    \[EBP+41B090\]                            ; USER32.MessageBoxA   
//不合法的话就弹出提示框!   
006E5881    E9 DF000000     JMP     006E5965                                ; ACProtec.006E5965   
006E5886    55              PUSH    EBP   
006E5887    73 65           JNB     SHORT 006E58EE                          ; ACProtec.006E58EE   
006E5889    72 20           JB      SHORT 006E58AB                          ; ACProtec.006E58AB   
006E588B    6F              OUTS    DX, DWORD PTR ES:\[EDI\]                  ; I/O command   
006E588C    72 20           JB      SHORT 006E58AE                          ; ACProtec.006E58AE   
006E588E    4D              DEC     EBP   
006E588F    61              POPAD   
006E5890    6368 69         ARPL    \[EAX+69\], BP   
006E5893    6E              OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
006E5894    65:49           DEC     ECX                                     ; Superfluous prefix   
006E5896    44              INC     ESP   
006E5897    2069 6E         AND     \[ECX+6E\], CH   
006E589A    206C69 63       AND     \[ECX+EBP\*2+63\], CH   
006E589E    65:6E           OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
006E58A0    73 65           JNB     SHORT 006E5907                          ; ACProtec.006E5907   
006E58A2    2066 69         AND     \[ESI+69\], AH   
006E58A5    6C              INS     BYTE PTR ES:\[EDI\], DX                   ; I/O command   
006E58A6    65:2065 72      AND     GS:\[EBP+72\], AH   
006E58AA    72 6F           JB      SHORT 006E591B                          ; ACProtec.006E591B   
006E58AC    72 00           JB      SHORT 006E58AE                          ; ACProtec.006E58AE   
006E58AE    8B9D 87DD4000   MOV     EBX, \[EBP+40DD87\]   
006E58B4    0BDB            OR      EBX, EBX   
006E58B6    74 5B           JE      SHORT 006E5913                          ; ACProtec.006E5913   
006E58B8    90              NOP   
006E58B9    90              NOP   
006E58BA    90              NOP   
006E58BB    90              NOP   
006E58BC    81FB 20202020   CMP     EBX, 20202020   
006E58C2    74 4F           JE      SHORT 006E5913                          ; ACProtec.006E5913   
006E58C4    90              NOP   
006E58C5    90              NOP   
006E58C6    90              NOP   
006E58C7    90              NOP   
006E58C8    53              PUSH    EBX   
006E58C9    6A 00           PUSH    0   
006E58CB    FF95 84B04100   CALL    \[EBP+41B084\]                            ; kernel32.GetModuleHandleA   
006E58D1    8DB5 86D74000   LEA     ESI, \[EBP+40D786\]   
006E58D7    60              PUSHAD   
006E58D8    AD              LODS    DWORD PTR \[ESI\]   
006E58D9    8BD8            MOV     EBX, EAX   
006E58DB    AD              LODS    DWORD PTR \[ESI\]   
006E58DC    3BC3            CMP     EAX, EBX   
006E58DE    74 0A           JE      SHORT 006E58EA                          ; ACProtec.006E58EA   
006E58E0    90              NOP   
006E58E1    90              NOP   
006E58E2    90              NOP   
006E58E3    90              NOP   
006E58E4    61              POPAD   
006E58E5    EB 09           JMP     SHORT 006E58F0                          ; ACProtec.006E58F0   
006E58E7    90              NOP   
006E58E8    90              NOP   
006E58E9    90              NOP   
006E58EA    61              POPAD   
006E58EB    EB 17           JMP     SHORT 006E5904                          ; ACProtec.006E5904   
006E58ED    90              NOP   
006E58EE    90              NOP   
006E58EF    90              NOP   
006E58F0    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E58F1    50              PUSH    EAX   
006E58F2    FF95 80B04100   CALL    \[EBP+41B080\]                            ; kernel32.GetProcAddress   
006E58F8    8985 86D74000   MOV     \[EBP+40D786\], EAX   
006E58FE    8985 8AD74000   MOV     \[EBP+40D78A\], EAX   
006E5904    FF95 86D74000   CALL    \[EBP+40D786\]   
006E590A    5B              POP     EBX                                     ; ACProtec.006E1C87   
006E590B    3BD8            CMP     EBX, EAX   
006E590D    75 56           JNZ     SHORT 006E5965                          ; ACProtec.006E5965   
006E590F    90              NOP   
006E5910    90              NOP   
006E5911    90              NOP   
006E5912    90              NOP   
006E5913    6A 10           PUSH    10   
006E5915    8DB5 67DD4000   LEA     ESI, \[EBP+40DD67\]   
006E591B    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E591C    8DB5 F7DB4000   LEA     ESI, \[EBP+40DBF7\]   
006E5922    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5923    8DB5 E7DB4000   LEA     ESI, \[EBP+40DBE7\]   
006E5929    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E592A    8DB5 57DB4000   LEA     ESI, \[EBP+40DB57\]   
006E5930    56              PUSH    ESI                                     ; ACProtec.006E1C87   
006E5931    FF95 AFDA4000   CALL    \[EBP+40DAAF\]   
006E5937    83C4 14         ADD     ESP, 14   
006E593A    E8 63D4FFFF     CALL    006E2DA2                                ; ACProtec.006E2DA2   
//这个 CALL 进去是干什么的没有搞懂   
006E593F    C685 57DD4000 0>MOV     BYTE PTR \[EBP+40DD57\], 1   
006E5946    8B8D F6AC4100   MOV     ECX, \[EBP+41ACF6\]   
006E594C    8DB5 17DB4000   LEA     ESI, \[EBP+40DB17\]   
006E5952    8BBD F2AC4100   MOV     EDI, \[EBP+41ACF2\]   
006E5958    03BD 31DE4000   ADD     EDI, \[EBP+40DE31\]                       ; ACProtec.00400000   
006E595E    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR \[ESI\]   
//转移 KEY.dat 里面的数据到 004CCE04 这个地址!   
006E5960    EB 47           JMP     SHORT 006E59A9                          ; ACProtec.006E59A9   
006E5962    90              NOP   
006E5963    90              NOP   
006E5964    90              NOP   
006E5965    66:C785 17DB400>MOV     WORD PTR \[EBP+40DB17\], 0FF00   
//未注册版的标志!   
//爆破点!!!(经过测试,只能爆破掉加壳后不会出现"未注册"字样,其他需要注册才能用的功能用不了)   
006E596E    8B9D 13DB4000   MOV     EBX, \[EBP+40DB13\]   
006E5974    83C3 02         ADD     EBX, 2   
006E5977    B9 FFFFFFFF     MOV     ECX, -1   
006E597C    41              INC     ECX   
006E597D    8B848D BBDA4000 MOV     EAX, \[EBP+ECX\*4+40DABB\]   
006E5984    0BC0            OR      EAX, EAX   
006E5986    74 21           JE      SHORT 006E59A9                          ; ACProtec.006E59A9   
006E5988    90              NOP   
006E5989    90              NOP   
006E598A    90              NOP   
006E598B    90              NOP   
006E598C    0385 31DE4000   ADD     EAX, \[EBP+40DE31\]                       ; ACProtec.00400000   
006E5992    3BC3            CMP     EAX, EBX   
006E5994  ^ 75 E6           JNZ     SHORT 006E597C                          ; ACProtec.006E597C   
006E5996    83E8 02         SUB     EAX, 2   
006E5999    8B8C8D E7DA4000 MOV     ECX, \[EBP+ECX\*4+40DAE7\]   
006E59A0    83C1 08         ADD     ECX, 8   
006E59A3    C600 E9         MOV     BYTE PTR \[EAX\], 0E9   
006E59A6    8948 01         MOV     \[EAX+1\], ECX   
006E59A9    60              PUSHAD   
.....   
//返回 006E5C83   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//返回到 006E5C83 之后继续走一段小花.然后会到这里   
  
006E5E69    E8 79D1FFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006E5E6E    66:81BD 17DB400>CMP     WORD PTR \[EBP+40DB17\], 0FF00   
//这里就是验证标志位了.把这里改成赋入值某某值就爆破了.   
006E5E77    0F85 F1000000   JNZ     006E5F6E                                ; ACProtec.006E5F6E   
006E5E7D    80BD D4E34000 C>CMP     BYTE PTR \[EBP+40E3D4\], 0CC   
006E5E84    0F84 E4000000   JE      006E5F6E                                ; ACProtec.006E5F6E   
006E5E8A    80BD D5E34000 C>CMP     BYTE PTR \[EBP+40E3D5\], 0CC   
006E5E91    0F84 D7000000   JE      006E5F6E                                ; ACProtec.006E5F6E   
006E5E97    80BD BB1E4100 0>CMP     BYTE PTR \[EBP+411EBB\], 0   
006E5E9E    0F84 CA000000   JE      006E5F6E                                ; ACProtec.006E5F6E   
006E5EA4    6A 00           PUSH    0   
006E5EA6    E8 0B000000     CALL    006E5EB6                                ; ACProtec.006E5EB6   
006E5EAB    4E              DEC     ESI   
006E5EAC    6F              OUTS    DX, DWORD PTR ES:\[EDI\]                  ; I/O command   
006E5EAD    204C69 63       AND     \[ECX+EBP\*2+63\], CL   
006E5EB1    65:6E           OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
006E5EB3    73 65           JNB     SHORT 006E5F1A                          ; ACProtec.006E5F1A   
006E5EB5    90              NOP   
006E5EB6    E8 AB000000     CALL    006E5F66                                ; ACProtec.006E5F66   
006E5EBB    006F 20         ADD     \[EDI+20\], CH   
006E5EBE    4C              DEC     ESP   
006E5EBF    6963 65 6E73652>IMUL    ESP, \[EBX+65\], 2065736E   
006E5EC6    6F              OUTS    DX, DWORD PTR ES:\[EDI\]                  ; I/O command   
006E5EC7    72 20           JB      SHORT 006E5EE9                          ; ACProtec.006E5EE9   
006E5EC9    4C              DEC     ESP   
006E5ECA    6963 65 6E73652>IMUL    ESP, \[EBX+65\], 2065736E   
006E5ED1    6973 20 6E6F742>IMUL    ESI, \[EBX+20\], 20746F6E   
006E5ED8    636F 72         ARPL    \[EDI+72\], BP   
006E5EDB    72 65           JB      SHORT 006E5F42                          ; ACProtec.006E5F42   
006E5EDD    637420 2C       ARPL    \[EAX+2C\], SI   
006E5EE1    73 6F           JNB     SHORT 006E5F52                          ; ACProtec.006E5F52   
006E5EE3    6D              INS     DWORD PTR ES:\[EDI\], DX                  ; I/O command   
006E5EE4    65:2066 75      AND     GS:\[ESI+75\], AH   
006E5EE8    6E              OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
006E5EE9    637469 6F       ARPL    \[ECX+EBP\*2+6F\], SI   
006E5EED    6E              OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
006E5EEE    2061 72         AND     \[ECX+72\], AH   
006E5EF1    65:206E 6F      AND     GS:\[ESI+6F\], CH   
006E5EF5    74 20           JE      SHORT 006E5F17                          ; ACProtec.006E5F17   
006E5EF7    61              POPAD   
006E5EF8    76 61           JBE     SHORT 006E5F5B                          ; ACProtec.006E5F5B   
006E5EFA    696C61 62 6C652>IMUL    EBP, \[ECX+62\], 21656C   
006E5F02    0000            ADD     \[EAX\], AL   
006E5F04    0000            ADD     \[EAX\], AL   
006E5F06    0000            ADD     \[EAX\], AL   
.................   
.............省略...   
.................   
006E5F62    0000            ADD     \[EAX\], AL   
006E5F64    0000            ADD     \[EAX\], AL   
006E5F66    6A 00           PUSH    0   
006E5F68    FF95 90B04100   CALL    \[EBP+41B090\]                            ; USER32.MessageBoxA   
006E5F6E    C685 EC194100 C>MOV     BYTE PTR \[EBP+4119EC\], 0C3   
006E5F75    60              PUSHAD   
006E5F76    E8 00000000     CALL    006E5F7B                                ; ACProtec.006E5F7B   
006E5F7B    5E              POP     ESI                                     ; ACProtec.006E7D5C   
006E5F7C    83EE 06         SUB     ESI, 6   
006E5F7F    B9 0C010000     MOV     ECX, 10C   
006E5F84    29CE            SUB     ESI, ECX   
006E5F86    BA 22F93191     MOV     EDX, 9131F922   
006E5F8B    C1E9 02         SHR     ECX, 2   
006E5F8E    83E9 02         SUB     ECX, 2   
006E5F91    83F9 00         CMP     ECX, 0   
006E5F94    7C 1A           JL      SHORT 006E5FB0                          ; ACProtec.006E5FB0   
006E5F96    8B048E          MOV     EAX, \[ESI+ECX\*4\]   
006E5F99    8B5C8E 04       MOV     EBX, \[ESI+ECX\*4+4\]   
006E5F9D    2BC3            SUB     EAX, EBX   
006E5F9F    C1C8 0B         ROR     EAX, 0B   
006E5FA2    33C2            XOR     EAX, EDX   
006E5FA4    81EA B1E7B51F   SUB     EDX, 1FB5E7B1   
006E5FAA    89048E          MOV     \[ESI+ECX\*4\], EAX   
006E5FAD    49              DEC     ECX   
006E5FAE  ^ EB E1           JMP     SHORT 006E5F91                          ; ACProtec.006E5F91   
006E5FB0    61              POPAD   
006E5FB1    61              POPAD   
006E5FB2    E8 4C050000     CALL    006E6503                                ; ACProtec.006E6503   
//这里有一个很特别的 CALL 哦,里面的东西没有花什么心思看了.验证另外一个 Key.dat 的合法标志而已.   
//还有一些验证 CC 什么的.   
006E5FB7    C3              RETN   
//返回 006E7CAC   
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//返回 006E7CAC 之后就做最后的冲刺准备了.但是离 OEP 的路还远呢.我就把剩下的简单说说了!   
..............   
..........省略 N 多 N 多的小花...   
.............   
//来到下面就已经到达了 BT 的 OEP 了.什么都吃掉了.   
//这个时候需要把各个寄存器的值都抄下来.包括堆栈的值.这样对于恢复 OEP 就方便多了.   
006ED442    61              POPAD   
006ED443    890D 095F6D00   MOV     \[6D5F09\], ECX   
006ED449    FF35 095F6D00   PUSH    DWORD PTR \[6D5F09\]   
006ED44F    C705 055F6D00 D>MOV     DWORD PTR \[6D5F05\], 406EDC   
006ED459    8B0D 055F6D00   MOV     ECX, \[6D5F05\]                           ; ntdll.77F944A8   
006ED45F    51              PUSH    ECX   
006ED460    8F05 195F6D00   POP     DWORD PTR \[6D5F19\]                      ; ntdll.77F944A8   
006ED466    8B0C24          MOV     ECX, \[ESP\]                              ; ntdll.77F944A8   
006ED469    8F05 015F6D00   POP     DWORD PTR \[6D5F01\]                      ; ntdll.77F944A8   
006ED46F    FF15 195F6D00   CALL    \[6D5F19\]   
//等于是 CALL 406EDC   
006ED475    A1 982F4F00     MOV     EAX, \[4F2F98\]   
006ED47A    8B00            MOV     EAX, \[EAX\]   
006ED47C    51              PUSH    ECX   
006ED47D    C70424 14264600 MOV     DWORD PTR \[ESP\], 462614   
006ED484    8F05 155F6D00   POP     DWORD PTR \[6D5F15\]                      ; ntdll.77F944A8   
006ED48A    FF15 155F6D00   CALL    \[6D5F15\]   
006ED490    90              NOP   
006ED491    90              NOP   
006ED492    60              PUSHAD   
..............   
..........省略...   
.............   
//下面也是被吃掉的 OEP 变形的代码   
006EDE3F    61              POPAD   
006EDE40    FF35 582D4F00   PUSH    DWORD PTR \[4F2D58\]                      ; ACProtec.004F5D14   
006EDE46    8F05 FD5E6D00   POP     DWORD PTR \[6D5EFD\]                      ; ntdll.77F944A8   
006EDE4C    8B0D FD5E6D00   MOV     ECX, \[6D5EFD\]   
006EDE52    A1 982F4F00     MOV     EAX, \[4F2F98\]   
006EDE57    8B00            MOV     EAX, \[EAX\]   
006EDE59    FF35 9CFE4700   PUSH    DWORD PTR \[47FE9C\]                      ; ACProtec.0047FEE8   
006EDE5F    8F05 F95E6D00   POP     DWORD PTR \[6D5EF9\]                      ; ntdll.77F944A8   
006EDE65    8B15 F95E6D00   MOV     EDX, \[6D5EF9\]   
006EDE6B    8915 F55E6D00   MOV     \[6D5EF5\], EDX   
006EDE71    FF35 F55E6D00   PUSH    DWORD PTR \[6D5EF5\]                      ; ntdll.77F944A8   
006EDE77    51              PUSH    ECX   
006EDE78    B9 2C264600     MOV     ECX, 46262C   
006EDE7D    8BD1            MOV     EDX, ECX   
006EDE7F    59              POP     ECX                                     ; ntdll.77F944A8   
006EDE80    52              PUSH    EDX   
006EDE81    8F05 115F6D00   POP     DWORD PTR \[6D5F11\]                      ; ntdll.77F944A8   
006EDE87    8B1424          MOV     EDX, \[ESP\]                              ; ntdll.77F944A8   
006EDE8A    8F05 F15E6D00   POP     DWORD PTR \[6D5EF1\]                      ; ntdll.77F944A8   
006EDE90    60              PUSHAD   
..............   
..........省略...   
.............   
//下面也是被吃掉的 OEP 变形的代码   
  
006EE848    61              POPAD   
006EE849    FF15 115F6D00   CALL    \[6D5F11\]                                ; ACProtec.0046262C   
006EE84F    FF35 8C2C4F00   PUSH    DWORD PTR \[4F2C8C\]                      ; ACProtec.004F5D0C   
006EE855    8B0C24          MOV     ECX, \[ESP\]                              ; ntdll.77F944A8   
006EE858    8F05 ED5E6D00   POP     DWORD PTR \[6D5EED\]                      ; ntdll.77F944A8   
006EE85E    A1 982F4F00     MOV     EAX, \[4F2F98\]   
006EE863    8B00            MOV     EAX, \[EAX\]   
006EE865    890D E95E6D00   MOV     \[6D5EE9\], ECX   
006EE86B    FF35 E95E6D00   PUSH    DWORD PTR \[6D5EE9\]                      ; ntdll.77F57D70   
006EE871    50              PUSH    EAX   
006EE872    B8 0CFB4700     MOV     EAX, 47FB0C   
006EE877    8BC8            MOV     ECX, EAX   
006EE879    58              POP     EAX                                     ; ntdll.77F944A8   
006EE87A    8B11            MOV     EDX, \[ECX\]   
006EE87C    59              POP     ECX                                     ; ntdll.77F944A8   
006EE87D    51              PUSH    ECX   
006EE87E    B9 2C264600     MOV     ECX, 46262C   
006EE883    890D 0D5F6D00   MOV     \[6D5F0D\], ECX   
006EE889    59              POP     ECX                                     ; ntdll.77F944A8   
006EE88A    FF15 0D5F6D00   CALL    \[6D5F0D\]   
006EE890    A1 982F4F00     MOV     EAX, \[4F2F98\]   
006EE895    8B00            MOV     EAX, \[EAX\]   
006EE897    90              NOP   
006EE898    90              NOP   
006EE899    60              PUSHAD   
  
..............   
..........省略...   
.............   
  
006EEC42    E8 A043FFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006EEC47    8B85 E4AC4100   MOV     EAX, \[EBP+41ACE4\]   
//EAX=CCDF7   
006EEC4D    0385 31DE4000   ADD     EAX, \[EBP+40DE31\]                       ; ACProtec.00400000   
//ADD      EAX,400000   
006EEC53    8985 E4AC4100   MOV     \[EBP+41ACE4\], EAX   
006EEC59    E8 8943FFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006EEC5E    C685 A1AC4100 E>MOV     BYTE PTR \[EBP+41ACA1\], 0E8   
006EEC65    E8 7D43FFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006EEC6A    C785 A2AC4100 F>MOV     DWORD PTR \[EBP+41ACA2\], 25FF   
006EEC74    8D85 E4AC4100   LEA     EAX, \[EBP+41ACE4\]   
006EEC7A    8985 A4AC4100   MOV     \[EBP+41ACA4\], EAX   
006EEC80    E8 6243FFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006EEC85    8DBD 80AA4100   LEA     EDI, \[EBP+41AA80\]   
006EEC8B    8D8D 96AC4100   LEA     ECX, \[EBP+41AC96\]   
006EEC91    2BCF            SUB     ECX, EDI   
006EEC93    C1E9 02         SHR     ECX, 2   
006EEC96    E8 EA40FFFF     CALL    006E2D85                                ; ACProtec.006E2D85   
006EEC9B    AB              STOS    DWORD PTR ES:\[EDI\]   
006EEC9C  ^ E2 F8           LOOPD   SHORT 006EEC96                          ; ACProtec.006EEC96   
006EEC9E    61              POPAD   
//一个插屁股的小循环!   
006EEC9F    EB 01           JMP     SHORT 006EECA2                          ; ACProtec.006EECA2   
006EECA1    90              NOP   
006EECA2  - FF25 E4EC6E00   JMP     \[6EECE4\]                                ; ACProtec.004CCDF7   
//JMP     OEP   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//到达的 OEP:   
//其实已经什么都没有了!   
//修补 OEP 的代码是一件非常烦人的事情.   
004CCDEA    39F1            CMP     ECX, ESI                                ; ntdll.77F57D70   
004CCDEC    0D 7EC3C7CC     OR      EAX, CCC7C37E   
004CCDF1    C2 2ECA         RETN    0CA2E   
004CCDF4    52              PUSH    EDX   
004CCDF5    92              XCHG    EAX, EDX   
004CCDF6    90              NOP   
004CCDF7    E8 B058F9FF     CALL    004626AC                                ; ACProtec.004626AC   
//停在这里   
004CCDFC    E8 D77BF3FF     CALL    004049D8                                ; ACProtec.004049D8   
004CCE01    8D40 00         LEA     EAX, \[EAX\]   
004CCE04    55              PUSH    EBP   
004CCE05    6E              OUTS    DX, BYTE PTR ES:\[EDI\]                   ; I/O command   
004CCE06    70 61           JO      SHORT 004CCE69                          ; ACProtec.004CCE69   
004CCE08    636B 65         ARPL    \[EBX+65\], BP   
004CCE0B    64:2041 6E      AND     FS:\[ECX+6E\], AL   
  
  
  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//IAT 解码处:   
//这里不打算写什么了.没有什么难度.就是有两个地方说说!   
  
  
006E6EF7    C685 472D4100 C>MOV     BYTE PTR \[EBP+412D47\], 0C3   
//这个指令.就是之前进入这里的 CALL 处第一跳指令.可以形容为堵住入口.关门啦   
  
  
006E7085    74 57           JE      SHORT 006E70DE                   ; ACProtec.006E70DE   
//把这里改成 JMP 就可以用 ImportREC 得到一份完整的 IAT 了.   
//需要更加完美的数据就多改三处地方.自己想,很简单的   
  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//在地址 006E74DB 处的数据解码 CALL 进入后越过一段小花来到这里:   
  
006E5169    E8 79DEFFFF     CALL    006E2FE7                         ; ACProtec.006E2FE7   
006E516E    C685 BE0F4100 C>MOV     BYTE PTR \[EBP+410FBE\], 0C3   
//把这个 CAKK 的入口放一个 C3 堵门   
006E5175    8DB5 39DE4000   LEA     ESI, \[EBP+40DE39\]   
006E517B    56              PUSH    ESI   
006E517C    AD              LODS    DWORD PTR \[ESI\]   
006E517D    0BC0            OR      EAX, EAX   
006E517F    74 49           JE      SHORT 006E51CA                   ; ACProtec.006E51CA   
006E5181    90              NOP   
006E5182    90              NOP   
006E5183    90              NOP   
006E5184    90              NOP   
006E5185    50              PUSH    EAX   
006E5186    AD              LODS    DWORD PTR \[ESI\]   
006E5187    91              XCHG    EAX, ECX                         ; ACProtec.006E51DA   
006E5188    51              PUSH    ECX                              ; ACProtec.006E51DA   
006E5189    51              PUSH    ECX                              ; ACProtec.006E51DA   
006E518A    6A 40           PUSH    40   
006E518C    FF95 EAE24000   CALL    \[EBP+40E2EA\]                     ; kernel32.GlobalAlloc   
006E5192    8985 35DE4000   MOV     \[EBP+40DE35\], EAX   
006E5198    59              POP     ECX                              ; ACProtec.006E7526   
006E5199    58              POP     EAX                              ; ACProtec.006E7526   
006E519A    0385 31DE4000   ADD     EAX, \[EBP+40DE31\]                ; ACProtec.00400000   
006E51A0    8BF0            MOV     ESI, EAX   
006E51A2    50              PUSH    EAX   
006E51A3    8BBD 35DE4000   MOV     EDI, \[EBP+40DE35\]   
006E51A9    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR\>   
006E51AB    58              POP     EAX                              ; ACProtec.006E7526   
006E51AC    50              PUSH    EAX   
006E51AD    FFB5 35DE4000   PUSH    DWORD PTR \[EBP+40DE35\]   
006E51B3    E8 699B0000     CALL    006EED21                         ; ACProtec.006EED21   
006E51B8    FFB5 35DE4000   PUSH    DWORD PTR \[EBP+40DE35\]   
006E51BE    FF95 EEE24000   CALL    \[EBP+40E2EE\]                     ; kernel32.GlobalFree   
006E51C4    5E              POP     ESI                              ; ACProtec.006E7526   
006E51C5    83C6 08         ADD     ESI, 8   
006E51C8  ^ EB B1           JMP     SHORT 006E517B                   ; ACProtec.006E517B   
//上面的循环是在低位申请一个内存,然后用来解码之用.   
006E51CA    5E              POP     ESI                              ; ACProtec.006E7526   
006E51CB    68 30750000     PUSH    7530   
006E51D0    6A 40           PUSH    40   
006E51D2    FF95 EAE24000   CALL    \[EBP+40E2EA\]                     ; kernel32.GlobalAlloc   
006E51D8    8985 35DE4000   MOV     \[EBP+40DE35\], EAX   
//这里比较特殊.我跟了好久才明白是放一个 Key 进去,作为程序运行之后返回壳中解码之用的.   
//这里是把这个 KEY 放入 006E1E35   
006E51DE    60              PUSHAD   
006E51DF    E8 00000000     CALL    006E51E4                         ; ACProtec.006E51E4   
006E51E4    5E              POP     ESI                              ; ACProtec.006E7526   
006E51E5    83EE 06         SUB     ESI, 6   
006E51E8    B9 75000000     MOV     ECX, 75   
006E51ED    29CE            SUB     ESI, ECX                         ; ACProtec.006E51DA   
006E51EF    BA 22F93191     MOV     EDX, 9131F922   
006E51F4    C1E9 02         SHR     ECX, 2   
006E51F7    83E9 02         SUB     ECX, 2   
006E51FA    83F9 00         CMP     ECX, 0   
006E51FD    7C 1A           JL      SHORT 006E5219                   ; ACProtec.006E5219   
006E51FF    8B048E          MOV     EAX, \[ESI+ECX\*4\]   
006E5202    8B5C8E 04       MOV     EBX, \[ESI+ECX\*4+4\]   
006E5206    2BC3            SUB     EAX, EBX   
006E5208    C1C8 0B         ROR     EAX, 0B   
006E520B    33C2            XOR     EAX, EDX   
006E520D    81EA B1E7B51F   SUB     EDX, 1FB5E7B1   
006E5213    89048E          MOV     \[ESI+ECX\*4\], EAX   
006E5216    49              DEC     ECX                              ; ACProtec.006E51DA   
006E5217  ^ EB E1           JMP     SHORT 006E51FA                   ; ACProtec.006E51FA   
006E5219    61              POPAD   
006E521A    61              POPAD   
006E521B    E8 191F0000     CALL    006E7139                         ; ACProtec.006E7139   
006E5220    C3              RETN   
//返回 006E74E0   
  
  
  
  
  
006E51D8    8985 35DE4000   MOV     \[EBP+40DE35\], EAX   
//关于这个 EAX=Key 的正确值怎么得到的我会在最后把自己的方法写上   
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//地址 006E74E0 处的转移数据至低位的 CALL 进入之后来到这里:   
  
006E7240    60              PUSHAD   
006E7241    E8 A1BDFFFF     CALL    006E2FE7                         ; ACProtec.006E2FE7   
006E7246    33D2            XOR     EDX, EDX                         ; ACProtec.0040E2FE   
006E7248    8BB495 251F4000 MOV     ESI, \[EBP+EDX\*4+401F25\]   
//取得需要转移的数据大小.   
006E724F    0BF6            OR      ESI, ESI   
//是否还有.没有就结束数据转移!   
006E7251    74 31           JE      SHORT 006E7284                   ; ACProtec.006E7284   
006E7253    90              NOP   
006E7254    90              NOP   
006E7255    90              NOP   
006E7256    90              NOP   
006E7257    03B5 31DE4000   ADD     ESI, \[EBP+40DE31\]                ; ACProtec.00400000   
006E725D    8B8C95 B5204000 MOV     ECX, \[EBP+EDX\*4+4020B5\]   
006E7264    60              PUSHAD   
006E7265    52              PUSH    EDX                              ; ACProtec.0040E2FE   
006E7266    51              PUSH    ECX                              ; ntdll.77F5F70F   
006E7267    6A 40           PUSH    40   
006E7269    FF95 EAE24000   CALL    \[EBP+40E2EA\]                     ; kernel32.GlobalAlloc   
006E726F    5A              POP     EDX                              ; ACProtec.006E74E5   
006E7270    898495 45224000 MOV     \[EBP+EDX\*4+402245\], EAX          ; ACProtec.0040E2CF   
006E7277    61              POPAD   
006E7278    8BBC95 45224000 MOV     EDI, \[EBP+EDX\*4+402245\]   
006E727F    F3:A4           REP     MOVS BYTE PTR ES:\[EDI\], BYTE PTR\>   
006E7281    42              INC     EDX                              ; ACProtec.0040E2FE   
006E7282  ^ EB C4           JMP     SHORT 006E7248                   ; ACProtec.006E7248   
006E7284    61              POPAD   
006E7285    C3              RETN   
//返回 006E74E5   
  
  
//解除这里最好的办法就是在地址 006E7248 的时候把赋值给 ESI 的地址里面的数据给清零,这样就不会有数据丢失了!   
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
//有了上面一串的分析之后,我就简单说说快我这个大懒人的方法吧!   
//我实在不喜欢手动修复那么丢失的 OEP 代码,所以我用了这个懒方法:   
  
载入之后:   
006D5000 >  60              PUSHAD   
006D5001    87EA            XCHG    EDX, EBP   
006D5003    66:23EF         AND     BP, DI   
006D5006    66:8BD3         MOV     DX, BX   
006D5009    FC              CLD   
006D500A    48              DEC     EAX   
006D500B    F8              CLC   
006D500C    F8              CLC   
006D500D    43              INC     EBX   
006D500E    EB 01           JMP     SHORT 006D5011                          ; ACProtec.006D5011   
  
  
  
直接输入 g 006E74DB  来到下面:   
  
  
  
006E7498    E8 4ABBFFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006E749D    E8 00000000     CALL    006E74A2                                ; ACProtec.006E74A2   
006E74A2    5B              POP     EBX                                     ; ntdll.77F944A8   
006E74A3    2B9D F6234000   SUB     EBX, \[EBP+4023F6\]   
006E74A9    81EB A2240100   SUB     EBX, 124A2   
006E74AF    899D 31DE4000   MOV     \[EBP+40DE31\], EBX                       ; USER32.77D10000   
006E74B5    E8 CCFDFFFF     CALL    006E7286                                ; ACProtec.006E7286   
006E74BA    E8 28BBFFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006E74BF    E8 C1B8FFFF     CALL    006E2D85                                ; ACProtec.006E2D85   
006E74C4    8985 8CED4000   MOV     \[EBP+40ED8C\], EAX                       ; ACProtec.0040E2CF   
006E74CA    E8 18BBFFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006E74CF    C685 57DD4000 0>MOV     BYTE PTR \[EBP+40DD57\], 0   
006E74D6    E8 46DDFFFF     CALL    006E5221                                ; ACProtec.006E5221   
006E74DB    E8 DEDAFFFF     CALL    006E4FBE                                ; ACProtec.006E4FBE   
//停在这里.这里是什么地方呢?对了,就是代码和数据的解码 CALL.   
//返回之后 006E4FBE 地址的里面已经是一个 C3 了!所以不用去理会了!   
006E74E0    E8 5BFDFFFF     CALL    006E7240                                ; ACProtec.006E7240   
//进去把 ESI 的值给清空然后返回出来!   
006E74E5    E8 FDBAFFFF     CALL    006E2FE7                                ; ACProtec.006E2FE7   
006E74EA    6A 00           PUSH    0   
006E74EC    E8 0A000000     CALL    006E74FB                                ; ACProtec.006E74FB   
006E74F1    41              INC     ECX                                     ; ntdll.77F5F70F   
006E74F2    43              INC     EBX                                     ; USER32.77D10000   
006E74F3    50              PUSH    EAX                                     ; ACProtec.0040E2CF   
  
  
//然后输入 G 006ED443    
  
//到达 006ED443 这个地址的时候就可以 Dump 了.   
006ED442    61              POPAD   
006ED443    890D 095F6D00   MOV     \[6D5F09\], ECX   
006ED449    FF35 095F6D00   PUSH    DWORD PTR \[6D5F09\]   
006ED44F    C705 055F6D00 D>MOV     DWORD PTR \[6D5F05\], 406EDC   
006ED459    8B0D 055F6D00   MOV     ECX, \[6D5F05\]                           ; ntdll.77F944A8   
006ED45F    51              PUSH    ECX   
006ED460    8F05 195F6D00   POP     DWORD PTR \[6D5F19\]                      ; ntdll.77F944A8   
006ED466    8B0C24          MOV     ECX, \[ESP\]                              ; ntdll.77F944A8   
006ED469    8F05 015F6D00   POP     DWORD PTR \[6D5F01\]                      ; ntdll.77F944A8   
006ED46F    FF15 195F6D00   CALL    \[6D5F19\]   
//等于是 CALL 406EDC   
006ED475    A1 982F4F00     MOV     EAX, \[4F2F98\]   
006ED47A    8B00            MOV     EAX, \[EAX\]   
006ED47C    51              PUSH    ECX   
006ED47D    C70424 14264600 MOV     DWORD PTR \[ESP\], 462614   
006ED484    8F05 155F6D00   POP     DWORD PTR \[6D5F15\]                      ; ntdll.77F944A8   
006ED48A    FF15 155F6D00   CALL    \[6D5F15\]   
006ED490    90              NOP   
006ED491    90              NOP   
006ED492    60              PUSHAD   
  
  
  
//Dump 出来之后,因为 Dump 出来的时候程序代码区和数据区的代码已经全部解码了.   
//所以把 006D5000 这个地址的代码改成这样,然后保存   
006D5000 >  55              PUSH    EBP   
006D5001    8BEC            MOV     EBP, ESP   
006D5003    83C4 F0         ADD     ESP, -10   
006D5006    B8 8CCB4C00     MOV     EAX, 4CCB8C   
006D500B    E9 33840100     JMP     006ED443                         ; Y\_ACProt.006ED443   
  
  
  
接着就是修复 IAT 了,这个就不所说了.   
  
  
  
  
  
  
\-------------------------------------------   
//现在运行看看.   
//晕,出错了.理论上来说应该是可以运行的哦.怎么回事呢?载入看看吧!   
//载入之后隐藏 OD,然后 F9,过一会程序就不动了,很明显出现了问题.按一下 F12 看看是怎么回事!   
0015350E    6E              OUTS    DX, BYTE PTR ES:\[EDI\]            ; I/O command   
0015350F    CF              IRETD   
00153510    93              XCHG    EAX, EBX                         ; dumped.004086D8   
00153511    6C              INS     BYTE PTR ES:\[EDI\], DX            ; I/O command   
00153512    50              PUSH    EAX             
  
  
\-------------------------------------------//堆栈   
//晕死.停在了一些乱七八糟的指令上,根据堆栈:   
//第一跳堆栈的位置就是问题所在了.   
00102310   00403D75  RETURN to dumped.00403D75 from 0015350E   
00102314   00000000   
00102318   00000000   
0010231C   004086D8  ASCII 0A,"EPrivilege"   
00102320   00403D0C  RETURN to dumped.00403D0C from dumped.00403D68   
  
  
\----------------------------------------------   
//我们回到程序的 00403D75 这个地址看看吧:   
00403D6D    89D7            MOV     EDI, EDX   
00403D6F    AB              STOS    DWORD PTR ES:\[EDI\]   
00403D70    E8 99F7D4FF     CALL    0015350E   
//原来是这个 CALL 进入之后出错的   
00403D75    51              PUSH    ECX   
  
\---------------------------------------------------------------------------------------   
//现在知道了出错的地方就简单多了.重新载入,然后对 00403D70 这个 CALL 的地址下断   
//按一下 F9 马上断了下来.进入之后发现是需要进入壳解码.   
  
006D65C6    8B4424 20       MOV     EAX, \[ESP+20\]                    ; dumped.00403D75   
006D65CA    33C9            XOR     ECX, ECX   
006D65CC    8B9C8D 69324000 MOV     EBX, \[EBP+ECX\*4+403269\]   
006D65D3    039D 31DE4000   ADD     EBX, \[EBP+40DE31\]                ; dumped.00400000   
006D65D9    3BC3            CMP     EAX, EBX   
006D65DB    74 07           JE      SHORT 006D65E4                   ; dumped.006D65E4   
006D65DD    90              NOP   
006D65DE    90              NOP   
006D65DF    90              NOP   
006D65E0    90              NOP   
006D65E1    41              INC     ECX   
006D65E2  ^ EB E8           JMP     SHORT 006D65CC                   ; dumped.006D65CC   
006D65E4    8DB5 49614000   LEA     ESI, \[EBP+406149\]   
006D65EA    B8 0A000000     MOV     EAX, 0A   
006D65EF    F7E1            MUL     ECX   
006D65F1    03F0            ADD     ESI, EAX                         ; dumped.006D5922   
006D65F3    8DBD EF1B4000   LEA     EDI, \[EBP+401BEF\]   
006D65F9    0FB6840D B12640>MOVZX   EAX, BYTE PTR \[EBP+ECX+4026B1\]   
006D6601    FEC0            INC     AL   
006D6603    88840D B1264000 MOV     \[EBP+ECX+4026B1\], AL   
006D660A    3C FF           CMP     AL, 0FF   
006D660C    75 13           JNZ     SHORT 006D6621                   ; dumped.006D6621   
006D660E    90              NOP   
006D660F    90              NOP   
006D6610    90              NOP   
006D6611    90              NOP   
006D6612    8BBD 35DE4000   MOV     EDI, \[EBP+40DE35\]   
//出错的地方在这里,当经过 255 次之后程序就会到这里   
//地址\[EBP+40DE35\]=006E1E35 所保存的 Key 不正确   
//这个 Key 的值不对而出了问题!   
006D6618    B8 0A000000     MOV     EAX, 0A   
006D661D    F7E1            MUL     ECX   
006D661F    03F8            ADD     EDI, EAX                         ; dumped.006D5922   
006D6621    8A9D 06244000   MOV     BL, \[EBP+402406\]   
006D6627    B9 0A000000     MOV     ECX, 0A   
//来到这里的时候 EDI=ESI 才对的.所以我们对上面的 Key 求逆就非常简单了.我计算出来 Key 的值=006DA149   
006D662C    AC              LODS    BYTE PTR \[ESI\]   
006D662D    32C3            XOR     AL, BL   
006D662F    AA              STOS    BYTE PTR ES:\[EDI\]   
006D6630  ^ E2 FA           LOOPD   SHORT 006D662C                   ; dumped.006D662C   
006D6632    83EF 0A         SUB     EDI, 0A   
006D6635    57              PUSH    EDI   
006D6636    8DB5 EF1B4000   LEA     ESI, \[EBP+401BEF\]   
006D663C    33F7            XOR     ESI, EDI   
006D663E    74 19           JE      SHORT 006D6659                   ; dumped.006D6659   
006D6640    90              NOP   
006D6641    90              NOP   
006D6642    90              NOP   
006D6643    90              NOP   
006D6644    8B7424 24       MOV     ESI, \[ESP+24\]                    ; dumped.004CCB94   
006D6648    83EE 04         SUB     ESI, 4   
006D664B    AD              LODS    DWORD PTR \[ESI\]   
006D664C    81EF 16244000   SUB     EDI, 402416   
006D6652    2BFD            SUB     EDI, EBP   
006D6654    03C7            ADD     EAX, EDI   
006D6656    8946 FC         MOV     \[ESI\-4\], EAX                     ; dumped.006D5922   
006D6659    5F              POP     EDI                              ; 00CE0444   
006D665A    57              PUSH    EDI   
006D665B    33C9            XOR     ECX, ECX   
006D665D    83F9 08         CMP     ECX, 8   
006D6660    74 0E           JE      SHORT 006D6670                   ; dumped.006D6670   
006D6662    90              NOP   
006D6663    90              NOP   
006D6664    90              NOP   
006D6665    90              NOP   
006D6666    8B448C 04       MOV     EAX, \[ESP+ECX\*4+4\]   
006D666A    89048C          MOV     \[ESP+ECX\*4\], EAX                 ; dumped.006D5922   
006D666D    41              INC     ECX   
006D666E  ^ EB ED           JMP     SHORT 006D665D                   ; dumped.006D665D   
006D6670    893C8C          MOV     \[ESP+ECX\*4\], EDI   
006D6673    60              PUSHAD   
  
  
  
  
//知道了 Key=006DA149 就简单了,退出.   
//然后重新脱一次.在数据解码的时候就把这个正确的 Key=006DA149 写进去.   
//当然,也可以把 006D660C 的那个条件跳转改成绝对跳转.   
//重新脱一次之后,可以运行了.   
  
  
\==============================================================   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
后记:   
  
这个程序的破解版我也弄出来了.不过是在地址 006E7498 的时候就 Dump 出来了.   
  
然后把下面的指令   
006E5E6E    66:81BD 17DB400>CMP     WORD PTR \[EBP+40DB17\], 0FF00   
  
改成了:   
006E5E6E    66:C785 17DB4000 4F4B  MOV     WORD PTR \[EBP+40DB17\], 4B4F   
  
然后保存起来就算是解除了加壳之后弹出的"未注册"框!   
  
  
  
  
  
\------------------------------------------------------------------   
【总    结】：   
  
  
  
以上只是我这个懒人的方法,每个人都有自己的方法.   
  
这个文章只能做一个参看.你脱的时候可别完全学我啊.不然出错了我也不知道怎么办.   
  
我自己感觉有很多地方写得很罗嗦,而且表达得也不好,如果有什么地方写错了,请大侠多多指正.   
  
最后在这里感谢你花了那么多时间看这篇文章!   
  
  
辉仔 Yock   
2004.04.11  
  
 4dd466d0e1e0cc123afbe504da2462ca