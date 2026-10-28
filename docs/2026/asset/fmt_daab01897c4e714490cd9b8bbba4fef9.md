
# [原创]忙里偷闲，分析了一个病毒，希望高手指点

最近比较忙，在学习研究软件漏洞，在公司里借了本`failwest`的《Oday安全：软件漏洞分析技术》，感觉写的不错，对于像我种漏洞菜鸟来说，入门的绝好材料~~读那本书很流畅，明白了很多道理，原来是这样找漏洞， 说点题外话：如果你拥有扎实的汇编功底，不管以后学什么，你都会比别人学的快，学的更透彻~~

所以有时间的话，建议去好好学学汇编，绝对不是浪费时间的事情~~

对了差点忘说了，听说第二版快要出来了，很期待~~好像还有内核漏洞方面的哦，正好可以补一下，自己的内核方面的知识，快快出版吧，让我等菜鸟等的花儿也谢了~~

昨天在群里有位朋友说有个病毒样本，于是就`DOWN`下来做了些研究，有点碰运气吧，样本被我草草研究了一下~~大家就将就的看看吧！

病毒名称：`server.exe`

分析工具：`flyODBG`

壳：不知道什么鬼壳

开始吧~~~

先查壳，这是我的习惯，查得如图所示：

查不到~~一般都这样！！！！

再来看看`EP`区段吧

后面四个区段很明显有问题，同一个偏移地址中间三个大小全为零，最后一个大小为`00004400`，我想`.cdata`肯定是外壳（只是猜想）

呵呵，用`Exeinfo PE`查得是用如下结果，好像是`Asprotect`的一个老版本

样本入口代码如下：

```
00401000 >  B8 D507D0B0        mov eax,B0D007D5
00401005    B8 6FA04000        mov eax,server.0040A06F
0040100A    8BC0               mov eax,eax                                          ; EAX = server.0040A06F
0040100C    8BD2               mov edx,edx                                          ; ntdll.KiFastSystemCallRet
0040100E    55                 push ebp                                             ; EBP = 0013FFF0
0040100F    8BE9               mov ebp,ecx                                          ; EBP = ECX = 0013FFB0
00401011    5D                 pop ebp                                              ; EBP = 0013FFF0
00401012    50                 push eax                                             ; EAX = server.0040A06F
00401013    51                 push ecx                                             ; ECX = 0013FFB0
00401014    8BC8               mov ecx,eax                                          ; ECX = EAX = server.0040A06F
00401016    59                 pop ecx                                              ; ECX = 0013FFB0
00401017    C3                 retn                                                 ; 返回
```

真不知道上面是作什么用的~~~返回再说吧~~

```
0040A06F    60                PUSHAD
0040A070    83EC 38           SUB ESP,38                                     ; ESP = 0012FFA4
0040A073    33C0              XOR EAX,EAX                                    ; EAx = 00000000
0040A075    C745 D8 47657450  MOV DWORD PTR SS:[EBP-28],50746547
0040A07C    C745 DC 726F6341  MOV DWORD PTR SS:[EBP-24],41636F72
0040A083    C745 E0 64647265  MOV DWORD PTR SS:[EBP-20],65726464
0040A08A    C745 E4 73730000  MOV DWORD PTR SS:[EBP-1C],7373                 ; GetProcAddress
0040A091    8945 FC           MOV DWORD PTR SS:[EBP-4],EAX
0040A094    8945 F8           MOV DWORD PTR SS:[EBP-8],EAX
0040A097    60                PUSHAD
0040A098    64:A1 30000000    MOV EAX,DWORD PTR FS:[30]                      ; TEB+30h ---->PEB
0040A09E    8B50 0C           MOV EDX,DWORD PTR DS:[EAX+C]                   ; PEB+C---->PEB\_LDR\_DATA
0040A0A1    8B42 1C           MOV EAX,DWORD PTR DS:[EDX+1C]                  ; 模块头指针InInitializationOrderModuleList
0040A0A4    8B00              MOV EAX,DWORD PTR DS:[EAX]                     ; 找到kernel32.dll链表结点
0040A0A6    8B40 08           MOV EAX,DWORD PTR DS:[EAX+8]                   ; kernel32.dll的基址加偏移0x08处是kernel32.dll在内存中加载地址
0040A0A9    8945 FC           MOV DWORD PTR SS:[EBP-4],EAX                   ; kernel32.dll加载地址存入堆栈[EBP-4]处
0040A0AC    8BD0              MOV EDX,EAX                                    ; kernel32.7C800000
0040A0AE    83C0 23           ADD EAX,23
0040A0B1    83C0 19           ADD EAX,19                                     ; EAX = 7C80003C
0040A0B4    8B00              MOV EAX,DWORD PTR DS:[EAX]                     ; 000000F0
0040A0B6    8D4410 78         LEA EAX,DWORD PTR DS:[EAX+EDX+78]              ; EAX = 7C800168 ASCII ",&"
0040A0BA    8B00              MOV EAX,DWORD PTR DS:[EAX]                     ; EAX = 262C
0040A0BC    8B4C02 18         MOV ECX,DWORD PTR DS:[EDX+EAX+18]              ; ECX = 000003B9
0040A0C0    8B5C02 20         MOV EBX,DWORD PTR DS:[EDX+EAX+20]              ; EBX = 00003538
0040A0C4    03DA              ADD EBX,EDX                                    ; kernel32.7C800000
0040A0C6    49                DEC ECX                                        ; ECX = 3B9
0040A0C7    90                NOP
0040A0C8    85C9              TEST ECX,ECX
0040A0CA    90                NOP
0040A0CB    74 38             JE SHORT 0040A105
0040A0CD    8D7D D8           LEA EDI,DWORD PTR SS:[EBP-28]                  ; [EBP-28] = GetProcAddress
0040A0D0    8B348B            MOV ESI,DWORD PTR DS:[EBX+ECX*4]               ; ESI = [EBX+ECX*4] = [7C804418] = 000008FEC
0040A0D3    03F2              ADD ESI,EDX                                    ; ESI = 000008FEC + EDX = 7C800000 = 7C808FEC
0040A0D5    51                PUSH ECX                                       ; 7C808FEC =>lstrlenW      ECX = 000003B8
0040A0D6    50                PUSH EAX                                       ; EAX = 0000262C
0040A0D7    B8 09000000       MOV EAX,9                                      ; EAX = 00000009
0040A0DC    83C0 06           ADD EAX,6                                      ; EAX = 0000000F
0040A0DF    8BC8              MOV ECX,EAX                                    ; ECX = EAX = 0000000F
0040A0E1    58                POP EAX                                        ; EAX = 0000262C
0040A0E2    F3:A6             REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]  ; 比较名称
0040A0E4    85C9              TEST ECX,ECX
0040A0E6    59                POP ECX
0040A0E7  ^ 75 DD             JNZ SHORT 0040A0C6                             ; ESI = 7C806AAF = "GetProcessAffinityMask"
0040A0E9    8B7402 24         MOV ESI,DWORD PTR DS:[EDX+EAX+24]              ; ESI = 0000441C
0040A0ED    03F2              ADD ESI,EDX                                    ; ESI = 7C80441C
0040A0EF    8BF9              MOV EDI,ECX                                    ; EDI = ECX = 00000198
0040A0F1    0FB7347E          MOVZX ESI,WORD PTR DS:[ESI+EDI*2]              ; ESI = 00000198
0040A0F5    8B7C02 1C         MOV EDI,DWORD PTR DS:[EDX+EAX+1C]              ; EDI = 00002654
0040A0F9    03FA              ADD EDI,EDX                                    ; EDI = 7C802654
0040A0FB    8B3CB7            MOV EDI,DWORD PTR DS:[EDI+ESI*4]               ; EDI = 0000AE30
0040A0FE    03FA              ADD EDI,EDX                                    ; EDI = 7C80AE30 kernel32.GetProcAddress
0040A100    897D F8           MOV DWORD PTR SS:[EBP-8],EDI                   ; SS:[0012FFE8] = EDI = GetProcAddress
0040A103    EB 07             JMP SHORT 0040A10C
0040A105    C745 F8 00000000  MOV DWORD PTR SS:[EBP-8],0
0040A10C    61                POPAD
0040A10D    8B4D FC           MOV ECX,DWORD PTR SS:[EBP-4]                   ; ECX = kernel32.7C800000
0040A110    33FF              XOR EDI,EDI                                    ; EDI = 00000000
0040A112    3BCF              CMP ECX,EDI
0040A114    0F84 DA000000     JE 0040A1F4
0040A11A    8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]                   ; EAX = SS:[EBP-8] = GetProcAddress
0040A11D    3BC7              CMP EAX,EDI
0040A11F    0F84 CF000000     JE 0040A1F4
0040A125    8D55 E0           LEA EDX,DWORD PTR SS:[EBP-20]                  ; SS:[EBP-20] = "ddress"
0040A128    52                PUSH EDX                                       ; EDX = 0012FFD0 ASCII "ddress"
0040A129    51                PUSH ECX                                       ; ECX = kernel32.7C800000
0040A12A    C645 E0 4C        MOV BYTE PTR SS:[EBP-20],4C
0040A12E    C645 E1 6F        MOV BYTE PTR SS:[EBP-1F],6F
0040A132    C645 E2 61        MOV BYTE PTR SS:[EBP-1E],61
0040A136    66:C745 E3 644C   MOV WORD PTR SS:[EBP-1D],4C64
0040A13C    90                NOP
0040A13D    90                NOP
0040A13E    66:C745 E5 6962   MOV WORD PTR SS:[EBP-1B],6269
0040A144    90                NOP
```
