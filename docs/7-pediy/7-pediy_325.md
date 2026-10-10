

# 为 Loader 增加命令行功能

> 原文：[`www.pediy.com/kssd/pediy05/pediy50320.htm`](https://www.pediy.com/kssd/pediy05/pediy50320.htm)

一、问题的提出：   
为了 SendTo 的客观需要。   
  
二、分析原文件：   
004017A6  |. C685 B0FBFFFF 00        MOV BYTE PTR SS:\[EBP\-450\],0   
004017AD  |. C785 ECFAFFFF AC104000  MOV DWORD PTR SS:\[EBP\-514\],LOADER.004010AC        ;  ASCII "exe file"   
004017B7  |. C785 10FBFFFF CC104000  MOV DWORD PTR SS:\[EBP\-4F0\],LOADER.004010CC        ;  ASCII "Choose a exe..."   
004017C1  |. 8D8D B0FBFFFF            LEA ECX,DWORD PTR SS:\[EBP\-450\]   
004017C7  |. 898D FCFAFFFF            MOV DWORD PTR SS:\[EBP\-504\],ECX   
004017CD  |. C785 00FBFFFF 04010000  MOV DWORD PTR SS:\[EBP\-500\],104   
004017D7  |. C785 14FBFFFF 00180000  MOV DWORD PTR SS:\[EBP\-4EC\],1800   
004017E1  |. C785 F8FAFFFF 01000000  MOV DWORD PTR SS:\[EBP\-508\],1   
004017EB  |. C785 1CFBFFFF DC104000  MOV DWORD PTR SS:\[EBP\-4E4\],LOADER.004010DC        ;  ASCII "exe"   
004017F5  |. C785 0CFBFFFF E0104000  MOV DWORD PTR SS:\[EBP\-4F4\],LOADER.004010E0   
004017FF  |. C785 E0FAFFFF 4C000000  MOV DWORD PTR SS:\[EBP\-520\],4C   
00401809  |. 8D95 E0FAFFFF            LEA EDX,DWORD PTR SS:\[EBP\-520\]   
0040180F  |. 52                      PUSH EDX                                          ; /pOpenFileName   
00401810  |. FF15 70104000            CALL DWORD PTR DS:\[<&comdlg32.GetOpenFileNameA>\]  ; \\GetOpenFileNameA      ;如果在 GetOpenFileNameA 之前先判断一下参数，如果有参数则直接 Load 参数所代表的 EXE，否则再打开这个对话框。   
00401816  . 85C0                    TEST EAX,EAX   
00401818  . 75 05                    JNZ SHORT LOADER.0040181F   
0040181A  . E9 8F0B0000              JMP LOADER.004023AE   
0040181F  > 6A 00                    PUSH 0                                            ; /hTemplateFile =  NULL    ;如果带有参数，直接跳到这里。   
00401821  . 6A 01                    PUSH 1                                            ; |Attributes = READONLY   
00401823  . 6A 03                    PUSH 3                                            ; |Mode = OPEN\_EXISTING   
00401825  . 6A 00                    PUSH 0                                            ; |pSecurity = NULL   
00401827  . 6A 01                    PUSH 1                                            ; |ShareMode = FILE\_SHARE\_READ   
00401829  . 68 00000080              PUSH 80000000                                    ; |Access = GENERIC\_READ   
0040182E  . 8D85 B0FBFFFF            LEA EAX,DWORD PTR SS:\[EBP\-450\]                    ; |   
00401834  . 50                      PUSH EAX                                          ; |FileName   
00401835  . FF15 40104000            CALL DWORD PTR DS:\[<&KERNEL32.CreateFileA>\]      ; \\CreateFileA   
  
  
三、解决方案   
1、准备工作   
要想取得参数，得用到 GetCommandLineA。原想用 LordPE 加入这个函数到引入表。可是 LordPE 显示没有足够的空间!!!!!打开目标程序一看，“乖乖，真的有效”，老大可真是惜字如金，怪不得功能这么强大的东东才只有 8K!!!   
  
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
000001C0  2E 74 65 78 74 00 00 00  00 1D 00 00 00 10 00 00  .text...........   
000001D0  00 1D 00 00 00 02 00 00  00 00 00 00 00 00 00 00  ................   
000001E0  00 00 00 00 20 00 00 E0  00 00 00 00 00 00 00 00  .... ..?.......   
000001F0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................   
  
程序只用到了一个 SECTION，里面包括所有的数据和代码，想再把文件头扩大 200 字节，有些麻烦，而且原文件增大不少，最重要的是有可能出现未知错误(反正我没有尝试)。手动增加引入函数吧。什么？没有空间？想办法呀：P   
  
2、身体力行之手动增加引入函数   
不管三七二十一，先在程序末尾加入 GetCommandLineA 的描述字符串：   
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
00001DE0              00 00 47 65  74 43 6F 6D 6D 61 6E 64      ..GetCommand   
00001DF0  4C 69 6E 65 41 00 4B 65  72 6E 65 6C 33 32 2E 64  LineA.Kernel32.d   
00001E00  6C 6C 00 E4 2B 00 00                              ll.?..   
                    ^^^^^^^^^^^=1DE4+1000-200   
  
看程序的引入表块指针：   
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
00000140                            18 29 00 00                      .)..   
  
从 2918h-1000h+200h=1b18h 处向前看，这不是有很多 0 吗？这不就是空间？可是，呵呵，这是数据区，可不是那么便宜就给你用的。如果没用，凭老大这么吝啬的人，会给你留着？：P   
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
00001AE0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................   
00001AF0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................   
00001B00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................   
00001B10  00 00 00 00 00 00 00 00  84 29 00 00 00 00 00 00  ........?......   
00001B20  00 00 00 00 70 2B 00 00  08 10 00 00 E0 29 00 00  ....p+......?..   
  
没办法，暂时借用一下，等你用时再还给你，反正我只用一次，而且是在你用之前(我是瞎猜的，但运气好得很，在我用这块数据区之前，程序暂时没用到它)，我把它还原成“00”不就得了嘛。在这里面增加引入表的结构块：   
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
00001B00              03 2C 00 00  00 00 00 00 00 00 00 00      .,..........   
00001B10  F6 2B 00 00 03 2C 00 00                            ?...,..   
  
修改 OPTIONAL HEADER 里面的引入表偏移，   
从原来的 002918 改为 002918-14=002904：   
Offset      0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F   
  
00000140                            04 29 00 00                      .)..   
  
3、身体力行之补丁文件：   
然后呢，就是在 GetOpenFileName 之前判断是否带有命令行参数，如果有，就不显示打开文件的对话框，直接 Load 它.......   
  
a)修改 GetOpenFileName 之前语句，转向我们的判断：   
004017A6    . 68 102C400>PUSH CMDLNLOA.00402C10   
004017AB    . C3        RETN                                              ;  RET used as a jump to 00402C10   
004017AC      90        NOP   
  
b)我们的判断部分：   
GetCommandLine 没有用到其他参数，返回值 eax 指向命令行参数的起始地址。没得说了。   
  
00402C10    > 60        PUSHAD   
00402C11    . FF15 032C4>CALL DWORD PTR DS:\[<&Kernel32.GetCommandLineA>\]  ; \[GetCommandLineA\]   
00402C17    . 8BF8      MOV EDI,EAX   
00402C19    . 57        PUSH EDI          ;保存备用，下次查找参数的起始地址   
00402C1A    . 33C0      XOR EAX,EAX        ;查找命令行结尾，ecx 存放命令行长度。   
00402C1C    . 33C9      XOR ECX,ECX   
00402C1E    . 49        DEC ECX   
00402C1F    . F2:AE      REPNE SCAS BYTE PTR ES:\[EDI\]   
00402C21    . F7D9      NEG ECX   
00402C23    . 49        DEC ECX   
00402C24    . 49        DEC ECX   
00402C25    . 5F        POP EDI            ;查找参数的起始地址，以第一个空格为标志。   
00402C26    . B0 20      MOV AL,20   
00402C28    . F2:AE      REPNE SCAS BYTE PTR ES:\[EDI\]   
00402C2A    . 51        PUSH ECX          ;ECX 为参数的长度，入栈。   
00402C2B    . 90        NOP   
00402C2C    . 90        NOP   
00402C2D    . 8BF7      MOV ESI,EDI        ;从原文的 GetOpenFileName 来看，EBP-450 处是文件名存放的地方   
00402C2F    . 8DBD B0FBF>LEA EDI,DWORD PTR SS:\[EBP\-450\]  ;复制参数过去。   
00402C35    . 41        INC ECX            ;把结尾的 0 也复制过去   
00402C36    . F3:A4      REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]   
00402C38    . BF 0429400>MOV EDI,CMDLNLOA.00402904      ;还记得我们当初借用的数据空间吗？把它还原为 14H 个字长的 0。   
00402C3D    . B9 0500000>MOV ECX,5   
00402C42    . 33C0      XOR EAX,EAX   
00402C44    . F3:AB      REP STOS DWORD PTR ES:\[EDI\]   
00402C46    . 59        POP ECX            ;判断命令行长度，如果为 0，出现打开文件对话框；否则直接 Load...   
00402C47    . 85C9      TEST ECX,ECX   
00402C49    . 74 07      JE SHORT CMDLNLOA.00402C52   
00402C4B    . 61        POPAD   
00402C4C    . 68 1F18400>PUSH CMDLNLOA.0040181F   
00402C51    . C3        RETN                                              ;  RET used as a jump to 0040181F   
00402C52    > 61        POPAD   
00402C53    . 68 AD17400>PUSH CMDLNLOA.004017AD   
00402C58    . C3        RETN                                              ;  RET used as a jump to 004017AD   
  
  
四、完成啦。太过弱智化，高手莫笑。   
  
wscn   
02/10/15  4dd466d0e1e0cc123afbe504da2462ca