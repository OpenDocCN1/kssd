

# \*\*\*\*\*管理专家 V1.05

> 原文：[`www.pediy.com/kssd/pediy05/pediy50319.htm`](https://www.pediy.com/kssd/pediy05/pediy50319.htm)

\*\*\*\*\*管理专家 V１.０５版破解实录   
时        间：2002-10-10   
破解对象： \*\*\*\*\* V 1.05 版   
破解工具：Language、TRW2000\_ 娃娃版、Keymake v1.73   
破解原因：近日从 网上拉下一款工资管理类软件，安装后发现其功能一般般。看看它的注册说明，哇噻，这东东竟敢索要 1000 大洋！这作者未免太那个了吧，俺就拿它来练练手，比破 Crackme 要强。   
保护方式：序列号！   
破  解  者：duhe（张湘平）   
破解过程：   
1、用 Language 侦测该软件是用 C 语言编写，并且无壳，很好！干掉它。   
  
2、运行 TRW2000 v1.23，然后再运行\*\*\*\*\*管理专家，点击“帮助/注册”菜单，在注册窗口内分别填入：   
    用户码：duhe   
    注册码：1111-2222-3333-4444   
  
3、呼出 TRW2000，下万能断点：bpx hmemcpy，按 F5 键切换到“\*\*\*\*\*管理专家”界面，点击“完成注册 ”按钮。   
……   
怎么回事？当机啦？晦气！重来！   
  
4、重启机器后，重复前面的第 2、3 步骤。怎么又当机了？！我靠！看样子软件作者运用反跟踪技术（Anti-Debug）了，狡猾狡猾的。怎么办？当然用 TRW2000\_ 娃娃版啦！   
  
5、从头再来。点击“完成注册 ”按钮后，程序很快被拦截下来，下命令：pmoudle ，来到“\*\*\*\*\*管理专家”的领空，按二下（3 下就会蹦出错误窗口）F12 键和一次 F10 键，程序来到这里：   
015F:0043D970 55              PUSH    EBP   
015F:0043D971 8BEC            MOV      EBP,ESP   
015F:0043D973 6AFF            PUSH    BYTE -01   
015F:0043D975 68D8E44400      PUSH    DWORD 0044E4D8   
015F:0043D97A 64A100000000    MOV      EAX,\[FS:00\]   
015F:0043D980 50              PUSH    EAX   
015F:0043D981 64892500000000  MOV      \[FS:00\],ESP   
015F:0043D988 83EC6C          SUB      ESP,BYTE +6C   
015F:0043D98B 53              PUSH    EBX   
015F:0043D98C 56              PUSH    ESI   
015F:0043D98D 57              PUSH    EDI   
015F:0043D98E 8BD9            MOV      EBX,ECX   
015F:0043D990 8965F0          MOV      \[EBP\-10\],ESP   
015F:0043D993 6A01            PUSH    BYTE +01   
015F:0043D995 E8288E0000      CALL    \`MFC42!ord\_000018BE\`   
015F:0043D99A 8D8390020000    LEA      EAX,\[EBX+0290\]      <---程序停在这里   
015F:0043D9A0 8D8B8C020000    LEA      ECX,\[EBX+028C\]   
015F:0043D9A6 50              PUSH    EAX   
015F:0043D9A7 8D55E0          LEA      EDX,\[EBP\-20\]   
015F:0043D9AA 51              PUSH    ECX   
015F:0043D9AB 52              PUSH    EDX   
015F:0043D9AC E8AB8A0000      CALL    \`MFC42!ord\_0000039A\`   
015F:0043D9B1 8D8B94020000    LEA      ECX,\[EBX+0294\]   
015F:0043D9B7 8D55E4          LEA      EDX,\[EBP\-1C\]   
015F:0043D9BA 51              PUSH    ECX   
015F:0043D9BB 50              PUSH    EAX   
015F:0043D9BC 52              PUSH    EDX   
015F:0043D9BD C745FC00000000  MOV      DWORD \[EBP\-04\],00   
015F:0043D9C4 E8938A0000      CALL    \`MFC42!ord\_0000039A\`   
015F:0043D9C9 8D8B98020000    LEA      ECX,\[EBX+0298\]   
015F:0043D9CF 8D55E8          LEA      EDX,\[EBP\-18\]   
015F:0043D9D2 51              PUSH    ECX   
015F:0043D9D3 50              PUSH    EAX   
015F:0043D9D4 52              PUSH    EDX   
015F:0043D9D5 C645FC01        MOV      BYTE \[EBP\-04\],01   
015F:0043D9D9 E87E8A0000      CALL    \`MFC42!ord\_0000039A\`   
015F:0043D9DE 8D4DE4          LEA      ECX,\[EBP\-1C\]   
015F:0043D9E1 C645FC04        MOV      BYTE \[EBP\-04\],04   
015F:0043D9E5 E8E2890000      CALL    \`MFC42!ord\_00000320\`   
015F:0043D9EA 8D4DE0          LEA      ECX,\[EBP\-20\]   
015F:0043D9ED C645FC03        MOV      BYTE \[EBP\-04\],03   
015F:0043D9F1 E8D6890000      CALL    \`MFC42!ord\_00000320\`   
015F:0043D9F6 A15C264600      MOV      EAX,\[0046265C\]   
015F:0043D9FB 8B0D60264600    MOV      ECX,\[00462660\]   
015F:0043DA01 8B1564264600    MOV      EDX,\[00462664\]   
015F:0043DA07 89459C          MOV      \[EBP\-64\],EAX   
015F:0043DA0A A168264600      MOV      EAX,\[00462668\]   
015F:0043DA0F 894DA0          MOV      \[EBP\-60\],ECX   
015F:0043DA12 8A0D6C264600    MOV      CL,\[0046266C\]   
015F:0043DA18 8945A8          MOV      \[EBP\-58\],EAX   
015F:0043DA1B A14C264600      MOV      EAX,\[0046264C\]   
015F:0043DA20 884DAC          MOV      \[EBP\-54\],CL   
015F:0043DA23 8B0D50264600    MOV      ECX,\[00462650\]   
015F:0043DA29 89458C          MOV      \[EBP\-74\],EAX   
015F:0043DA2C A058264600      MOV      AL,\[00462658\]   
015F:0043DA31 894D90          MOV      \[EBP\-70\],ECX   
015F:0043DA34 8B0DA03C4600    MOV      ECX,\[00463CA0\]   
015F:0043DA3A 8955A4          MOV      \[EBP\-5C\],EDX        〈---ECX 指向我的用户名：“duhe”   
015F:0043DA3D 8B1548264600    MOV      EDX,\[00462648\]   
015F:0043DA43 884598          MOV      \[EBP\-68\],AL   
015F:0043DA46 8B41F8          MOV      EAX,\[ECX\-08\]   
015F:0043DA49 895588          MOV      \[EBP\-78\],EDX   
015F:0043DA4C 8B1554264600    MOV      EDX,\[00462654\]   
015F:0043DA52 83F810          CMP      EAX,BYTE +10   
015F:0043DA55 895594          MOV      \[EBP\-6C\],EDX   
015F:0043DA58 7E63            JNG      0043DABD   
015F:0043DA5A 8D55DC          LEA      EDX,\[EBP\-24\]   
015F:0043DA5D 6A10            PUSH    BYTE +10   
015F:0043DA5F 52              PUSH    EDX   
015F:0043DA60 B9A03C4600      MOV      ECX,00463CA0   
015F:0043DA65 E8E0890000      CALL    \`MFC42!ord\_00001021\`   
015F:0043DA6A 8B00            MOV      EAX,\[EAX\]   
015F:0043DA6C 83C9FF          OR      ECX,BYTE -01   
015F:0043DA6F 8BF8            MOV      EDI,EAX   
015F:0043DA71 33C0            XOR      EAX,EAX   
015F:0043DA73 F2AE            REPNE SCASB   
015F:0043DA75 F7D1            NOT      ECX   
015F:0043DA77 2BF9            SUB      EDI,ECX   
015F:0043DA79 8D559C          LEA      EDX,\[EBP\-64\]   
015F:0043DA7C 8BC1            MOV      EAX,ECX   
015F:0043DA7E 8BF7            MOV      ESI,EDI   
015F:0043DA80 8BFA            MOV      EDI,EDX   
015F:0043DA82 C1E902          SHR      ECX,02   
015F:0043DA85 F3A5            REP MOVSD   
015F:0043DA87 8BC8            MOV      ECX,EAX   
015F:0043DA89 83E103          AND      ECX,BYTE +03   
015F:0043DA8C F3A4            REP MOVSB   
015F:0043DA8E 8D4DDC          LEA      ECX,\[EBP\-24\]   
015F:0043DA91 E836890000      CALL    \`MFC42!ord\_00000320\`   
015F:0043DA96 8B0DA03C4600    MOV      ECX,\[00463CA0\]   
015F:0043DA9C 33C0            XOR      EAX,EAX   
015F:0043DA9E 8B51F8          MOV      EDX,\[ECX\-08\]   
015F:0043DAA1 83C2F0          ADD      EDX,BYTE -10   
015F:0043DAA4 85D2            TEST    EDX,EDX   
015F:0043DAA6 7E42            JNG      0043DAEA   
015F:0043DAA8 8A540110        MOV      DL,\[ECX+EAX+10\]   
015F:0043DAAC 0054059C        ADD      \[EBP+EAX\-64\],DL   
015F:0043DAB0 8B51F8          MOV      EDX,\[ECX\-08\]   
015F:0043DAB3 40              INC      EAX   
015F:0043DAB4 83C2F0          ADD      EDX,BYTE -10   
015F:0043DAB7 3BC2            CMP      EAX,EDX   
015F:0043DAB9 7CED            JL      0043DAA8   
015F:0043DABB EB2D            JMP      SHORT 0043DAEA   
015F:0043DABD 50              PUSH    EAX   
015F:0043DABE B9A03C4600      MOV      ECX,00463CA0   
015F:0043DAC3 E822890000      CALL    \`MFC42!ord\_00000B63\`   
015F:0043DAC8 8BF8            MOV      EDI,EAX      〈---EAX 指向我的用户名：“duhe”   
015F:0043DACA 83C9FF          OR      ECX,BYTE -01      
015F:0043DACD 33C0            XOR      EAX,EAX   
015F:0043DACF 8D559C          LEA      EDX,\[EBP\-64\]   
015F:0043DAD2 F2AE            REPNE SCASB   
015F:0043DAD4 F7D1            NOT      ECX   
015F:0043DAD6 2BF9            SUB      EDI,ECX   
015F:0043DAD8 8BC1            MOV      EAX,ECX      〈---EDI 指向我的用户名：“duhe”   
015F:0043DADA 8BF7            MOV      ESI,EDI   
015F:0043DADC 8BFA            MOV      EDI,EDX      〈---ESI 指向我的用户名：“duhe”   
015F:0043DADE C1E902          SHR      ECX,02   
015F:0043DAE1 F3A5            REP MOVSD   
015F:0043DAE3 8BC8            MOV      ECX,EAX      〈---EDX 指向我的用户名：“duhe”   
015F:0043DAE5 83E103          AND      ECX,BYTE +03   
015F:0043DAE8 F3A4            REP MOVSB   
015F:0043DAEA 33C9            XOR      ECX,ECX      〈---EDX 指向我的用户名：“duhe”   
015F:0043DAEC 8A440D9C        MOV      AL,\[EBP+ECX\-64\]    ←——————   
015F:0043DAF0 84C0            TEST    AL,AL                            |   
015F:0043DAF2 7D06            JNL      0043DAFA                        |   
015F:0043DAF4 F6D8            NEG      AL                              |   
015F:0043DAF6 88440D9C        MOV      \[EBP+ECX\-64\],AL                  |   
015F:0043DAFA 0FBE440D88      MOVSX    EAX,BYTE \[EBP+ECX\-78\]              
015F:0043DAFF 0FBE540D9C      MOVSX    EDX,BYTE \[EBP+ECX\-64\]            计   
015F:0043DB04 0BC2            OR      EAX,EDX                          算        
015F:0043DB06 BE1A000000      MOV      ESI,1A                          注   
015F:0043DB0B 99              CDQ                                      册   
015F:0043DB0C F7FE            IDIV    ESI                              码   
015F:0043DB0E 80C241          ADD      DL,41                              
015F:0043DB11 88540D9C        MOV      \[EBP+ECX\-64\],DL                  |   
015F:0043DB15 41              INC      ECX                              |   
015F:0043DB16 83F910          CMP      ECX,BYTE +10                    |   
015F:0043DB19 7CD1            JL      0043DAEC          ←——————   
015F:0043DB1B 8D459C          LEA      EAX,\[EBP\-64\]   
015F:0043DB1E 8D4DEC          LEA      ECX,\[EBP\-14\]              <---EAX 指向真注册码   
015F:0043DB21 50              PUSH    EAX   
015F:0043DB22 E8C9880000      CALL    \`MFC42!ord\_00000219\`      <---EAX 指向真注册码   
015F:0043DB27 8B75EC          MOV      ESI,\[EBP\-14\]   
015F:0043DB2A 8B45E8          MOV      EAX,\[EBP\-18\]   
015F:0043DB2D C645FC05        MOV      BYTE \[EBP\-04\],05   
015F:0043DB31 8A10            MOV      DL,\[EAX\]   
015F:0043DB33 8ACA            MOV      CL,DL   
015F:0043DB35 3A16            CMP      DL,\[ESI\]   
015F:0043DB37 751C            JNZ      0043DB55   
015F:0043DB39 84C9            TEST    CL,CL   
015F:0043DB3B 7414            JZ      0043DB51   
015F:0043DB3D 8A5001          MOV      DL,\[EAX+01\]   
015F:0043DB40 8ACA            MOV      CL,DL   
015F:0043DB42 3A5601          CMP      DL,\[ESI+01\]   
015F:0043DB45 750E            JNZ      0043DB55   
015F:0043DB47 83C002          ADD      EAX,BYTE +02   
015F:0043DB4A 83C602          ADD      ESI,BYTE +02   
015F:0043DB4D 84C9            TEST    CL,CL   
015F:0043DB4F 75E0            JNZ      0043DB31   
015F:0043DB51 33C0            XOR      EAX,EAX   
015F:0043DB53 EB05            JMP      SHORT 0043DB5A   
015F:0043DB55 1BC0            SBB      EAX,EAX   
015F:0043DB57 83D8FF          SBB      EAX,BYTE -01   
015F:0043DB5A 85C0            TEST    EAX,EAX   
015F:0043DB5C 743F            JZ      0043DB9D              <---此处跳转就能成功注册   
015F:0043DB5E 6A00            PUSH    BYTE +00   
015F:0043DB60 68B4364600      PUSH    DWORD 004636B4   
015F:0043DB65 6894364600      PUSH    DWORD 00463694   
015F:0043DB6A 8BCB            MOV      ECX,EBX   
015F:0043DB6C E8398C0000      CALL    \`MFC42!ord\_00001080\`    <---call 出出错窗口   
015F:0043DB71 8D4DEC          LEA      ECX,\[EBP\-14\]   
015F:0043DB74 C645FC03        MOV      BYTE \[EBP\-04\],03   
015F:0043DB78 E84F880000      CALL    \`MFC42!ord\_00000320\`   
015F:0043DB7D 8D4DE8          LEA      ECX,\[EBP\-18\]   
015F:0043DB80 C745FCFFFFFFFF  MOV      DWORD \[EBP\-04\],FFFFFFFF   
015F:0043DB87 E840880000      CALL    \`MFC42!ord\_00000320\`   
015F:0043DB8C 8B4DF4          MOV      ECX,\[EBP\-0C\]   
015F:0043DB8F 5F              POP      EDI   
015F:0043DB90 5E              POP      ESI   
015F:0043DB91 64890D00000000  MOV      \[FS:00\],ECX   
015F:0043DB98 5B              POP      EBX   
015F:0043DB99 8BE5            MOV      ESP,EBP   
015F:0043DB9B 5D              POP      EBP   
  
    当我们来到 0043DB22 处，D EAX 就能瞧见我们的注册码（当然是真的啦）。再往下继续走，发现 0043DB6C E8398C0000  CALL  \`MFC42!ord\_00001080\`  是 call 出出错窗口的地方，我们回过头往前看，在其不远处恰巧有一跳转，能够跳过这个该死的 call ，据此我们可以肯定地判断：这个跳转就是关键的一跳。   
  
6、好了，现在关键的地方已经找到，那么我们应该如何处置它呢？办法当然有多个，即要么爆破，要么用 Keymake 做出它的注册机：   
  ⑴、爆破：   
    从上面的分析中我们得知，在 0043DB5C 743F  JZ  0043DB9D 处只要反跳就能成功注册，所以，我们将跳转命令 JZ 改成 JNZ ，即将机器码 743F 改成 753F 就行。因此我们先备份该软件的主文件，再用 16 进制编辑工具（如 UltraEdit）把 0043DB5C 743F  JZ  0043DB9D 改成 0043DB5C 743F  JNZ  0043DB9D ，也即将将机器码 743F 改成 753F，然后存盘退出，修改后的程序，注册时我们随意填写注册码（当然是假的哦）均可成功注册。   
    非常有趣的是，只要当你跳过 0043DB6C E8398C0000  CALL  \`MFC42!ord\_00001080\`  （call 出出错窗口），不但能够让你成功注册，而且它还自动地将正确的注册码填入注册码的窗口中！   
  ⑵、做内存注册机：   
    首先我们得感谢刘建英为我等 Cracker 打造了一个巨酷的工具软件：Keymake。运行 Keymake V1.73   
    我们知道当程序运行到在 0043DB22 E8C9880000  CALL  \`MFC42!ord\_00000219\` 处，D EAX 就可在 TRW2000 的数据窗口内瞧见你所需要的真注册码，因此我们点击 Keymake 菜单：“其他/内存注册机”，在中断地址列表中填上如下内容：   
    中断地址：43Db22   
    次    数：1   
    指    令：E8   
    长    度：5   
    注 册 码：内存方式--->EAX   
    然后点击“生成”按钮即可生成我们需要的内存注册机了。   
  
  
    当你注册成功后，软件会在下列地方做些改动：   
㈠、在注册表中增加如下键值：   
\[HKEY\_USERS\\.DEFAULT\\Software\\爬山虎软件工作室\\爬山虎工资管理专家\\Options\]   
"zcm"="HVTH-TLBP-XHBK-DGYT"   
"dwmc"="duhe"   
㈡、在其数据库文件（GZ.mdb）中填入你的注册码相关信息。   
  
  
问题：   
由于本人太菜而看不懂上面的算法。如果我要用 Keymake 做个它的算法注册机，请问各位高手，我该如何编写它的 rek 文件呢？   
先谢了！  4dd466d0e1e0cc123afbe504da2462ca