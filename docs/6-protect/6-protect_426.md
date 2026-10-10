

# 使用异常处理反跟踪调试的例子【分享】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-471.htm`](https://www.pediy.com/kssd/pediy08/pediy8-471.htm)

调试环境 windows 2000，  
调试工具 softice。  
软件 skype 2.0.47 版\[现在版本估计是兼容性的问题 把这部分程序去掉了\]  
    如果你找不到可以去下列地址  
    http://www.netexpert.cn/thread-12119-1-1.html  
    \[不过这个我把执行这段程序通过前面无条件转移跳过了：\]  
    修改这个部分对付：它的异常处理过程．否则它会出错在 softice 下．  
0072CD9A  7505                JNZ       0072CDA1  ；修改为无条件转移\[现在程序中是无条件转移改过来就可以了\]  
0072CD9C  E85FFAFFFF          CALL      0072C800 ；这个部分每过一段时间就会调用，在 softice 下出异常。  
0072CDA1  A1E839B200          MOV       EAX,\[00B239E8\]  
  
喜欢用 softice 调试软件，见过一些，反 softice 跟踪一些软件。过去也看到过一些  
检测 softice 是否存在运行的文章。  
前些日子，跟踪 skype 的时候，发现它用了以前没有见过的方式。  
具体方法是：  
1  使用 SetUnhandledExceptionFilter 修改为自己的异常处理程序。  
2  申请一块内存，随机添入数据，再随机产生一个地址在申请内存中，  
   加入几条指令。指令如下：  
001B:3F330241  0F31                RDTSC  
001B:3F330243  CD01                INT       1  
001B:3F330245  CC                  INT       3  
001B:3F330246  8DC0                LEA       EAX,EAX  
001B:3F330248  0F31                RDTSC  
001B:3F33024A  CD01                INT       1  
001B:3F33024C  8DC0                LEA       EAX,EAX  
001B:3F33024E  CC                  INT       3  
001B:3F33024F  CD01                INT       1  
001B:3F330251  CF                  IRETD  
001B:3F330252  90                  NOP  
3  保存当前的积存器值，给各积存器赋予新的值。然后转入执行刚才申请  
   内存中的指令。  
4。使用异常处理程序判断执行的异常指令，取得它的异常码，且保存  
5。执行完上面的指令后，与正确的执行过程得到的异常码是否相同。  
   注意如果有 softice 则上面的指令执行时就被 softice 捕获，无法正确执行。  
     
   下面是涉及到相关程序部分。  
0072C800  53                  PUSH      EBX  
0072C801  56                  PUSH      ESI  
0072C802  57                  PUSH      EDI  
0072C803  FF15A0ADA000        CALL      \[KERNEL32!GetCurrentThreadId\] ；取得的进程号  
0072C809  A37036B200          MOV       \[00B23670\],EAX  
0072C80E  E83DF2FFFF          CALL      0072BA50　　；取得当前的异常处理过程  
0072C813  8B3DA4ADA000        MOV       EDI,\[KERNEL32!SetUnhandledExceptionFilter\]  
0072C819  6880C47200          PUSH      0072C480  
0072C81E  8BF0                MOV       ESI,EAX  
0072C820  FFD7                CALL      EDI　　　　  ；设置异常处理之程序  
0072C822  A36C36B200          MOV       \[00B2366C\],EAX  
0072C827  B904000000          MOV       ECX,00000004  
0072C82C  A1B839B200          MOV       EAX,\[00B239B8\]  
0072C831  8B1DD839B200        MOV       EBX,\[00B239D8\]  
0072C837  8D1440              LEA       EDX,\[EAX\*2+EAX\]  
0072C83A  8D1490              LEA       EDX,\[EDX\*4+EAX\]  
0072C83D  C1E204              SHL       EDX,04  
0072C840  03D0                ADD       EDX,EAX  
0072C842  C1E208              SHL       EDX,08  
0072C845  2BD0                SUB       EDX,EAX  
0072C847  8D8490C39E2600      LEA       EAX,\[EDX\*4+EAX+00269EC3\]  
0072C84E  8B15DC39B200        MOV       EDX,\[00B239DC\]  
0072C854  2BD1                SUB       EDX,ECX  
0072C856  83C104              ADD       ECX,04  
0072C859  A3B839B200          MOV       \[00B239B8\],EAX  
0072C85E  83F920              CMP       ECX,20  
0072C861  89041A              MOV       \[EBX+EDX\],EAX  
0072C864  7EC6                JLE       0072C82C　　　　　；设置 POPAD 的寄存器值  
0072C866  E8F5FBFFFF          CALL      0072C460　　　　　；调用那段乱七八糟的代码  
0072C86B  A16C36B200          MOV       EAX,\[00B2366C\]  
0072C870  50                  PUSH      EAX  
0072C871  FFD7                CALL      EDI　　　　　　　；恢复异常处理链  
0072C873  56                  PUSH      ESI  
0072C874  E8C7F1FFFF          CALL      0072BA40　　　　；设置原来的异常处理之程序  
0072C879  A16836B200          MOV       EAX,\[00B23668\]  
0072C87E  83C404              ADD       ESP,04  
0072C881  83F80B              CMP       EAX,0B               ;比较  
0072C884  7544                JNZ       0072C8CA             ;异常  
0072C886  33D2                XOR       EDX,EDX              ;处理  
0072C888  B9A036B200          MOV       ECX,00B236A0         ;得到  
0072C88D  B898B2A000          MOV       EAX,00A0B298         ;数据  
0072C892  668B31              MOV       SI,\[ECX\]             ;是否  
0072C895  663B30              CMP       SI,\[EAX\]             ;正确  
0072C898  7517                JNZ       0072C8B1             ;  
0072C89A  0FBF7002            MOVSX     ESI,WORD PTR \[EAX+02\];  
0072C89E  397108              CMP       \[ECX+08\],ESI         ;  
0072C8A1  750E                JNZ       0072C8B1             ;  
0072C8A3  83C004              ADD       EAX,04               ;  
0072C8A6  42                  INC       EDX                  ;  
0072C8A7  83C10C              ADD       ECX,0C               ;  
0072C8AA  3DC4B2A000          CMP       EAX,00A0B2C4         ;  
0072C8AF  7CE1                JL        0072C892             ;  
0072C8B1  83FA0B              CMP       EDX,0B               ;  
0072C8B4  7514                JNZ       0072C8CA             ;  
0072C8B6  8B0DE839B200        MOV       ECX,\[00B239E8\]       ;  
0072C8BC  81E116840400        AND       ECX,00048416         ;  
0072C8C2  81F916840400        CMP       ECX,00048416         ;  
0072C8C8  740C                JZ        0072C8D6             ;  
0072C8CA  A1F839B200          MOV       EAX,\[00B239F8\]  
0072C8CF  0C02                OR        AL,02                ；不正确时设置  
0072C8D1  A3F839B200          MOV       \[00B239F8\],EAX  
0072C8D6  5F                  POP       EDI  
0072C8D7  5E                  POP       ESI  
0072C8D8  5B                  POP       EBX  
0072C8D9  C3                  RET  
\==========skype 的程序异常处理过程============================================  
0072C480  53                  PUSH      EBX  
0072C481  FF15A0ADA000        CALL      \[KERNEL32!GetCurrentThreadId\]  
0072C487  3B057036B200        CMP       EAX,\[00B23670\]  
0072C48D  741C                JZ        0072C4AB       ；通过进程判断是否是自己执行时的异常  
0072C48F  A16C36B200          MOV       EAX,\[00B2366C\]  
0072C494  33DB                XOR       EBX,EBX  
0072C496  3BC3                CMP       EAX,EBX  
0072C498  740B                JZ        0072C4A5  
0072C49A  8B4C2408            MOV       ECX,\[ESP+08\]  
0072C49E  51                  PUSH      ECX  
0072C49F  FFD0                CALL      EAX            ；执行原来的异常处理过程  
0072C4A1  5B                  POP       EBX  
0072C4A2  C20400              RET       0004  
0072C4A5  33C0                XOR       EAX,EAX  
0072C4A7  5B                  POP       EBX  
0072C4A8  C20400              RET       0004  
0072C4AB  8B0D6836B200        MOV       ECX,\[00B23668\]  
0072C4B1  55                  PUSH      EBP  
0072C4B2  56                  PUSH      ESI  
0072C4B3  8B742410            MOV       ESI,\[ESP+10\]  
0072C4B7  8D1449              LEA       EDX,\[ECX\*2+ECX\]  
0072C4BA  57                  PUSH      EDI  
0072C4BB  41                  INC       ECX  
0072C4BC  B801000000          MOV       EAX,00000001  
0072C4C1  8D3C95A036B200      LEA       EDI,\[EDX\*4+00B236A0\]  
0072C4C8  890D6836B200        MOV       \[00B23668\],ECX  
0072C4CE  C74708FFFFFFFF      MOV       DWORD PTR \[EDI+08\],FFFFFFFF  
0072C4D5  8B0E                MOV       ECX,\[ESI\]  
0072C4D7  8B29                MOV       EBP,\[ECX\]  
0072C4D9  F7C500FFFF00        TEST      EBP,00FFFF00  
0072C4DF  7516                JNZ       0072C4F7  
0072C4E1  8BD5                MOV       EDX,EBP  
0072C4E3  C1EA10              SHR       EDX,10  
0072C4E6  8ADA                MOV       BL,DL  
0072C4E8  3219                XOR       BL,\[ECX\]  
0072C4EA  81E3FF000000        AND       EBX,000000FF  
0072C4F0  33DA                XOR       EBX,EDX  
0072C4F2  66891F              MOV       \[EDI\],BX    ；异常标志号  
0072C4F5  EB05                JMP       0072C4FC  
0072C4F7  66C707FFFF          MOV       WORD PTR \[EDI\],FFFF  
0072C4FC  8B0E                MOV       ECX,\[ESI\]  
0072C4FE  33DB                XOR       EBX,EBX  
0072C500  8B510C              MOV       EDX,\[ECX+0C\]  
0072C503  895704              MOV       \[EDI+04\],EDX  
0072C506  8B0E                MOV       ECX,\[ESI\]  
0072C508  8B510C              MOV       EDX,\[ECX+0C\]  
0072C50B  33C9                XOR       ECX,ECX  
0072C50D  668B0F              MOV       CX,\[EDI\]  
0072C510  81F91DC00000        CMP       ECX,0000C01D  
0072C516  0F8F4E020000        JG        0072C76A  
0072C51C  0F84C4010000        JZ        0072C6E6  
0072C522  81E903800000        SUB       ECX,00008003  
0072C528  0F847D010000        JZ        0072C6AB  
0072C52E  49                  DEC       ECX  
0072C52F  0F845E010000        JZ        0072C693  
0072C535  81E901400000        SUB       ECX,00004001  
0072C53B  0F8534020000        JNZ       0072C775  
0072C541  803ACD              CMP       BYTE PTR \[EDX\],CD    ;是否是  
0072C544  0F8578020000        JNZ       0072C7C2             ;int 1   
0072C54A  807A0101            CMP       BYTE PTR \[EDX+01\],01 ;指令  
0072C54E  0F856E020000        JNZ       0072C7C2  
0072C554  8B7A01              MOV       EDI,\[EDX+01\]  
0072C557  32C9                XOR       CL,CL  
0072C559  81FF01CC8DC0        CMP       EDI,C08DCC01  
0072C55F  7540                JNZ       0072C5A1  
0072C561  8B5604              MOV       EDX,\[ESI+04\]  
0072C564  8B82B0000000        MOV       EAX,\[EDX+000000B0\]  
0072C56A  A3A839B200          MOV       \[00B239A8\],EAX  
0072C56F  8B4E04              MOV       ECX,\[ESI+04\]  
0072C572  8B91A8000000        MOV       EDX,\[ECX+000000A8\]  
0072C578  8915AC39B200        MOV       \[00B239AC\],EDX  
0072C57E  8B4604              MOV       EAX,\[ESI+04\]  
0072C581  8B88B8000000        MOV       ECX,\[EAX+000000B8\]  
0072C587  83C105              ADD       ECX,05  
0072C58A  8988B8000000        MOV       \[EAX+000000B8\],ECX  
0072C590  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C595  0C02                OR        AL,02  
0072C597  A3E839B200          MOV       \[00B239E8\],EAX  
0072C59C  83C8FF              OR        EAX,-01  
0072C59F  EB6E                JMP       0072C60F  
0072C5A1  81FF018DC0CC        CMP       EDI,CCC08D01  
0072C5A7  7568                JNZ       0072C611  
0072C5A9  8B15E839B200        MOV       EDX,\[00B239E8\]  
0072C5AF  83CA04              OR        EDX,04  
0072C5B2  8915E839B200        MOV       \[00B239E8\],EDX  
0072C5B8  8B4604              MOV       EAX,\[ESI+04\]  
0072C5BB  F6C202              TEST      DL,02  
0072C5BE  8B80B0000000        MOV       EAX,\[EAX+000000B0\]  
0072C5C4  A3A039B200          MOV       \[00B239A0\],EAX  
0072C5C9  8B4E04              MOV       ECX,\[ESI+04\]  
0072C5CC  8B89A8000000        MOV       ECX,\[ECX+000000A8\]  
0072C5D2  890DA439B200        MOV       \[00B239A4\],ECX  
0072C5D8  7428                JZ        0072C602  
0072C5DA  8B2DA839B200        MOV       EBP,\[00B239A8\]  
0072C5E0  8B15AC39B200        MOV       EDX,\[00B239AC\]  
0072C5E6  2BC5                SUB       EAX,EBP  
0072C5E8  A3F039B200          MOV       \[00B239F0\],EAX  
0072C5ED  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C5F2  1BCA                SBB       ECX,EDX  
0072C5F4  80CC80              OR        AH,80  
0072C5F7  890DF439B200        MOV       \[00B239F4\],ECX  
0072C5FD  A3E839B200          MOV       \[00B239E8\],EAX  
0072C602  8B4E04              MOV       ECX,\[ESI+04\]  
0072C605  83C8FF              OR        EAX,-01  
0072C608  8381B800000004      ADD       DWORD PTR \[ECX+000000B8\],04  
0072C60F  B101                MOV       CL,01  
0072C611  81E7FFFF0000        AND       EDI,0000FFFF  
0072C617  81FF01CF0000        CMP       EDI,0000CF01  
0072C61D  7565                JNZ       0072C684  
0072C61F  8B5604              MOV       EDX,\[ESI+04\]  
0072C622  8B82C4000000        MOV       EAX,\[EDX+000000C4\]  
0072C628  8B00                MOV       EAX,\[EAX\]  
0072C62A  F6C408              TEST      AH,08  
0072C62D  A3EC39B200          MOV       \[00B239EC\],EAX  
0072C632  750A                JNZ       0072C63E  
0072C634  810DE839B20000000100OR        DWORD PTR \[00B239E8\],00010000  
0072C63E  F6C401              TEST      AH,01  
0072C641  750A                JNZ       0072C64D  
0072C643  810DE839B20000000200OR        DWORD PTR \[00B239E8\],00020000  
0072C64D  8B5604              MOV       EDX,\[ESI+04\]  
0072C650  B9F0C77200          MOV       ECX,0072C7F0  
0072C655  41                  INC       ECX  
0072C656  898AB8000000        MOV       \[EDX+000000B8\],ECX  
0072C65C  8B4604              MOV       EAX,\[ESI+04\]  
0072C65F  8B0DE039B200        MOV       ECX,\[00B239E0\]  
0072C665  8988C4000000        MOV       \[EAX+000000C4\],ECX  
0072C66B  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C670  0D00000400          OR        EAX,00040000  
0072C675  A3E839B200          MOV       \[00B239E8\],EAX  
0072C67A  E881FDFFFF          CALL      0072C400  
0072C67F  E93B010000          JMP       0072C7BF  
0072C684  3ACB                CMP       CL,BL  
0072C686  0F8536010000        JNZ       0072C7C2  
0072C68C  33C0                XOR       EAX,EAX  
0072C68E  E92F010000          JMP       0072C7C2  
0072C693  391DE439B200        CMP       \[00B239E4\],EBX  
0072C699  0F8520010000        JNZ       0072C7BF  
0072C69F  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C6A4  0C01                OR        AL,01  
0072C6A6  E90F010000          JMP       0072C7BA  
0072C6AB  803ACC              CMP       BYTE PTR \[EDX\],CC    ；是否 int 3  
0072C6AE  0F850E010000        JNZ       0072C7C2  
0072C6B4  807A018D            CMP       BYTE PTR \[EDX+01\],8D ;是否是  
0072C6B8  750F                JNZ       0072C6C9             ;8D C0 LEA EAX,EAX  
0072C6BA  807A02C0            CMP       BYTE PTR \[EDX+02\],C0 ;代码  
0072C6BE  7509                JNZ       0072C6C9  
0072C6C0  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C6C5  0C08                OR        AL,08  
0072C6C7  EB07                JMP       0072C6D0  
0072C6C9  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C6CE  0C10                OR        AL,10  
0072C6D0  A3E839B200          MOV       \[00B239E8\],EAX  
0072C6D5  8B4E04              MOV       ECX,\[ESI+04\]  
0072C6D8  83C8FF              OR        EAX,-01  
0072C6DB  FF81B8000000        INC       DWORD PTR \[ECX+000000B8\]  
0072C6E1  E9DC000000          JMP       0072C7C2  
0072C6E6  803A0F              CMP       BYTE PTR \[EDX\],0F  
0072C6E9  B902000000          MOV       ECX,00000002  
0072C6EE  7539                JNZ       0072C729  
0072C6F0  807A010B            CMP       BYTE PTR \[EDX+01\],0B  
0072C6F4  7533                JNZ       0072C729  
0072C6F6  8B4604              MOV       EAX,\[ESI+04\]  
0072C6F9  8BA8B8000000        MOV       EBP,\[EAX+000000B8\]  
0072C6FF  03E9                ADD       EBP,ECX  
0072C701  89A8B8000000        MOV       \[EAX+000000B8\],EBP  
0072C707  8B4604              MOV       EAX,\[ESI+04\]  
0072C70A  8BB8C0000000        MOV       EDI,\[EAX+000000C0\]  
0072C710  81CF00090000        OR        EDI,00000900  
0072C716  89B8C0000000        MOV       \[EAX+000000C0\],EDI  
0072C71C  83C8FF              OR        EAX,-01  
0072C71F  C705E439B20001000000MOV       DWORD PTR \[00B239E4\],00000001  
0072C729  803A8D              CMP       BYTE PTR \[EDX\],8D  
0072C72C  0F8590000000        JNZ       0072C7C2  
0072C732  807A01C0            CMP       BYTE PTR \[EDX+01\],C0  
0072C736  0F8586000000        JNZ       0072C7C2  
0072C73C  8B4604              MOV       EAX,\[ESI+04\]  
0072C73F  8B90B8000000        MOV       EDX,\[EAX+000000B8\]  
0072C745  03D1                ADD       EDX,ECX  
0072C747  8990B8000000        MOV       \[EAX+000000B8\],EDX  
0072C74D  8B4604              MOV       EAX,\[ESI+04\]  
0072C750  8B88C0000000        MOV       ECX,\[EAX+000000C0\]  
0072C756  80E5F6              AND       CH,F6  
0072C759  8988C0000000        MOV       \[EAX+000000C0\],ECX  
0072C75F  83C8FF              OR        EAX,-01  
0072C762  891DE439B200        MOV       \[00B239E4\],EBX  
0072C768  EB58                JMP       0072C7C2  
0072C76A  81E995C00000        SUB       ECX,0000C095  
0072C770  7440                JZ        0072C7B2  
0072C772  49                  DEC       ECX  
0072C773  7411                JZ        0072C786  
0072C775  8B0DE839B200        MOV       ECX,\[00B239E8\]  
0072C77B  80CD08              OR        CH,08  
0072C77E  890DE839B200        MOV       \[00B239E8\],ECX  
0072C784  EB3C                JMP       0072C7C2  
0072C786  803A0F              CMP       BYTE PTR \[EDX\],0F  
0072C789  7537                JNZ       0072C7C2  
0072C78B  807A0134            CMP       BYTE PTR \[EDX+01\],34  
0072C78F  7531                JNZ       0072C7C2  
0072C791  8B3DE839B200        MOV       EDI,\[00B239E8\]  
0072C797  83C8FF              OR        EAX,-01  
0072C79A  81CF00100000        OR        EDI,00001000  
0072C7A0  893DE839B200        MOV       \[00B239E8\],EDI  
0072C7A6  8B4E04              MOV       ECX,\[ESI+04\]  
0072C7A9  8381B800000002      ADD       DWORD PTR \[ECX+000000B8\],02  
0072C7B0  EB10                JMP       0072C7C2  
0072C7B2  A1E839B200          MOV       EAX,\[00B239E8\]  
0072C7B7  80CC04              OR        AH,04  
0072C7BA  A3E839B200          MOV       \[00B239E8\],EAX  
0072C7BF  83C8FF              OR        EAX,-01  
0072C7C2  8B5604              MOV       EDX,\[ESI+04\]  
0072C7C5  5F                  POP       EDI  
0072C7C6  895A10              MOV       \[EDX+10\],EBX  
0072C7C9  8B4E04              MOV       ECX,\[ESI+04\]  
0072C7CC  89590C              MOV       \[ECX+0C\],EBX  
0072C7CF  8B5604              MOV       EDX,\[ESI+04\]  
0072C7D2  895A08              MOV       \[EDX+08\],EBX  
0072C7D5  8B4E04              MOV       ECX,\[ESI+04\]  
0072C7D8  5E                  POP       ESI  
0072C7D9  5D                  POP       EBP  
0072C7DA  895904              MOV       \[ECX+04\],EBX  
0072C7DD  5B                  POP       EBX  
0072C7DE  C20400              RET       0004 

调试环境：    windows 2000  
调试工具：    softice  
调试软件名称： PRTG Paessler Router Traffic Grapher V5.2.0.574  
下载地址： http://www.xdowns.com/soft/1/3/2006/soft\_17315.html  \[或自己搜索\]  
  
  
这个程序在前面还有对 softice 软件的测试，下面我们要看到是后面的一个利用执行异常  
代码，来测试有没有跟踪。 代码部分如下：  
  
001B:01A915C3  A1C89FAA01          MOV       EAX,\[KERNEL32!VirtualAlloc\]  
001B:01A915C8  681116A901          PUSH      01A91611  
001B:01A915CD  64FF3500000000      PUSH      DWORD PTR FS:\[00000000\]  
001B:01A915D4  64892500000000      MOV       FS:\[00000000\],ESP  
001B:01A915DB  33F6                XOR       ESI,ESI  
001B:01A915DD  6A04                PUSH      04  
001B:01A915DF  6800100000          PUSH      00001000  
001B:01A915E4  6800100000          PUSH      00001000  
001B:01A915E9  6A00                PUSH      00  
001B:01A915EB  FFD0                CALL      EAX   ；VirtualAlloc 申请内存  
001B:01A915ED  669C                PUSHF  
001B:01A915EF  0FBA2C2410          BTS       DWORD PTR \[ESP\],10  
001B:01A915F4  0FBA2C2408          BTS       DWORD PTR \[ESP\],08  
001B:01A915F9  0E                  PUSH      CS         ；设置执行 IRET  
001B:01A915FA  680216A901          PUSH      01A91602   ；指令的堆栈  
001B:01A915FF  0FD0                INVALID------------------->异常指令  
异常时系统中堆栈情况：  
0010:BC832DCC 01A915FF  0000001B  00210246  0012950A    
0010:BC832DDC 00000023  804692A2  BBC89414  FDE08E48    
  
00)   BPMB KERNEL32!UnhandledExceptionFilter RW DR3  
01) \* BPX KERNEL32!VirtualAlloc  
  
001B:01A91601  CF                  IRETD   ------>返回时单步执行标志将被置 1  
001B:01A91602  FF00                INC       DWORD PTR \[EAX\]  
返回单步后在系统中的堆栈情况：  
0010:BC97ADC8 00000006  01A91602  0000001B  00040346   
0010:BC97ADD8 00129516  00000023  804692A2  BBC89414   
001B:01A91604  50                  PUSH      EAX  
001B:01A91605  A1CC9FAA01          MOV       EAX,\[KERNEL32!VirtualFree\]  
001B:01A9160A  FFD0                CALL      EAX  
001B:01A9160C  8975FC              MOV       \[EBP-04\],ESI------->保存 ESI 的值  
001B:01A9160F  EB78                JMP       01A91689----->转到后面判断  
\=====================下面是异常处理的程序部分==================================  
001B:01A91611  60                  PUSHAD  
001B:01A91612  8D7C2424            LEA       EDI,\[ESP+24\]  
001B:01A91616  8B37                MOV       ESI,\[EDI\]  
001B:01A91618  8B7F08              MOV       EDI,\[EDI+08\]  
001B:01A9161B  8B06                MOV       EAX,\[ESI\]  
001B:01A9161D  3D04000080          CMP       EAX,80000004 ; 是不是单步标志  
001B:01A91622  7439                JZ        01A9165D  
001B:01A91624  3D1D0000C0          CMP       EAX,C000001D ; 指令异常码  
001B:01A91629  7405                JZ        01A91630  
001B:01A9162B  61                  POPAD  
001B:01A9162C  33C0                XOR       EAX,EAX  
001B:01A9162E  40                  INC       EAX  
001B:01A9162F  C3                  RET  
001B:01A91630  8B87B8000000        MOV       EAX,\[EDI+000000B8\]  
001B:01A91636  6681380FD0          CMP       WORD PTR \[EAX\],D00F     ；是不是 d00f 指令产生的异常  
001B:01A9163B  7402                JZ        01A9163F  
001B:01A9163D  EBED                JMP       01A9162C  
001B:01A9163F  33C0                XOR       EAX,EAX  
001B:01A91641  894714              MOV       \[EDI+14\],EAX  
001B:01A91644  C747040216A901      MOV       DWORD PTR \[EDI+04\],01A91602   
001B:01A9164B  C7471801050000      MOV       DWORD PTR \[EDI+18\],00000501  ；单步中断标志置位  
001B:01A91652  8387B800000002      ADD       DWORD PTR \[EDI+000000B8\],02  ；修改返回地址  
001B:01A91659  61                  POPAD  
001B:01A9165A  33C0                XOR       EAX,EAX  
001B:01A9165C  C3                  RET  
001B:01A9165D  0FBA671400          BT        DWORD PTR \[EDI+14\],00   ；单步中断标志  
FFFF0FF1 ：在 softice 下连续连续执行的时候---------------------->\[EDI+14\]=FFFF0FF1  
FFFF4FF0 ：在地址 01A915FF 使用单步，然后再连续时标志------------>\[EDI+14\]=FFFF4FF0  
所以这部分如果有 softice 在且连续执行时是错误的。可以起到反跟踪的目的。  
001B:01A91662  7314                JAE       01A91678  
001B:01A91664  C787A000000001000000MOV       DWORD PTR \[EDI+000000A0\],00000001  ；设置 ESI 值  
001B:01A9166E  C787B80000000416A901MOV       DWORD PTR \[EDI+000000B8\],01A91604  
001B:01A91678  33C0                XOR       EAX,EAX  
001B:01A9167A  894704              MOV       \[EDI+04\],EAX  
001B:01A9167D  894714              MOV       \[EDI+14\],EAX  
001B:01A91680  C7471800040000      MOV       DWORD PTR \[EDI+18\],00000400  ；清掉单步中断标志  
001B:01A91687  EBD0                JMP       01A91659  
\======================转入到此判断执行情况返回=====================================  
001B:01A91689  648F0500000000      POP       DWORD PTR FS:\[00000000\]  
001B:01A91690  83C404              ADD       ESP,04  
001B:01A91693  837DFC00            CMP       DWORD PTR \[EBP-04\],00  ；判断程序是否被正确执行  
001B:01A91697  0F95C0              SETNZ     AL  
001B:01A9169A  5F                  POP       EDI  
001B:01A9169B  5E                  POP       ESI  
001B:01A9169C  5B                  POP       EBX  
001B:01A9169D  C9                  LEAVE  
001B:01A9169E  C3                  RET