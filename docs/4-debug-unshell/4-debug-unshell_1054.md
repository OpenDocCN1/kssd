

# 逆向\]Telock98 加密过程分析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6543.htm`](https://www.pediy.com/kssd/pediy06/pediy6543.htm)

Telock98 加密过程分析(上)  
  
作者：lordor  
Mail:lordor2#hotmail.com  
来自：NukeGroup  
网站：[www.digitalnuke.com](http://www.digitalnuke.com)  
逆向对象：Telock98  
  
不知大家有没有兴趣研究 PE 加壳技术，我没编写过加壳器，但想了解，所以逆向了一下 Telock 的加壳过程。  
在看本文前，最好了解一下 pe32 的格式。如果大家有什么好的想法，Please let me know.  
  
我们开始：  
看一下加密过程用到什么 call,定位 createfilea 函数，来到如下:  
  
  
00404A85  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404A8A  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404A90  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404A95  PUSH dumped\_.00410B61                               ; /FileName = "C:\\Documents and Settings\\lordor\\桌面\\TELock098  
  
\\WinPE V1.0.exe.bak"  
00404A9A  CALL <JMP.&kernel32.GetFileAttributesA>             ; \\GetFileAttributesA  ==>文件属性  
00404A9F  CMP EAX,-1  
00404AA2  JE SHORT dumped\_.00404AD8  
00404AA4  AND EAX,1  
00404AA7  JE SHORT dumped\_.00404AD8  
00404AA9  PUSH 24                                             ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
00404AAB  PUSH dumped\_.0040B0B1                               ; |Title = "确认"  
00404AB0  PUSH dumped\_.0040AF18                               ; |Text = "文件被写保护。您仍要加锁吗？"  
00404AB5  PUSH DWORD PTR DS:\[40EF28\]                          ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
00404ABB  CALL <JMP.&user32.MessageBoxA>                      ; \\MessageBoxA  
00404AC0  CMP EAX,6  
00404AC3  JNZ dumped\_.0040568F  
00404AC9  PUSH 80                                             ; /FileAttributes = NORMAL  
00404ACE  PUSH dumped\_.00410B61                               ; |FileName = "C:\\Documents and Settings\\lordor\\桌面\\TELock098  
  
\\WinPE V1.0.exe.bak"  
00404AD3  CALL <JMP.&kernel32.SetFileAttributesA>             ; \\SetFileAttributesA  ==>设置文件属性  
00404AD8  XOR EAX,EAX  
00404ADA  PUSH EAX                                            ; /hTemplateFile => NULL  
00404ADB  PUSH 80                                             ; |Attributes = NORMAL  
00404AE0  PUSH 3                                              ; |Mode = OPEN\_EXISTING  
00404AE2  PUSH EAX                                            ; |pSecurity => NULL  
00404AE3  PUSH EAX                                            ; |ShareMode => 0  
00404AE4  PUSH C0000000                                       ; |Access = GENERIC\_READ|GENERIC\_WRITE  
00404AE9  PUSH dumped\_.00410B61                               ; |FileName = "C:\\Documents and Settings\\lordor\\桌面\\TELock098  
  
\\WinPE V1.0.exe.bak"  
00404AEE  CALL <JMP.&kernel32.CreateFileA>                    ; \\CreateFileA  
00404AF3  MOV DWORD PTR DS:\[40EFF0\],EAX  
00404AF8  CMP EAX,-1  
00404AFB  JNZ SHORT dumped\_.00404B35  
00404AFD  PUSH dumped\_.0040AC83                               ; /lParam = 40AC83  
00404B02  PUSH 0                                              ; |wParam = 0  
00404B04  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404B09  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404B0F  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404B14  JMP dumped\_.0040568F  
00404B19  PUSH dumped\_.0040AEB6                               ; /lParam = 40AEB6  
00404B1E  PUSH 0                                              ; |wParam = 0  
00404B20  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404B25  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404B2B  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404B30  JMP dumped\_.0040568F  
00404B35  PUSH 0                                              ; /pFileSizeHigh = NULL  
00404B37  PUSH DWORD PTR DS:\[40EFF0\]                          ; |hFile = 000000A4 (window)  
00404B3D  CALL <JMP.&kernel32.GetFileSize>                    ; \\GetFileSize  
00404B42  TEST EAX,EAX  
00404B44  JG SHORT dumped\_.00404B6D  
00404B46  PUSH dumped\_.0040ACB1                               ; /lParam = 40ACB1  
00404B4B  PUSH 0                                              ; |wParam = 0  
00404B4D  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404B52  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404B58  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404B5D  PUSH DWORD PTR DS:\[40EFF0\]                          ; /hObject = 000000A4 (window)  
00404B63  CALL <JMP.&kernel32.CloseHandle>                    ; \\CloseHandle  
00404B68  JMP dumped\_.0040568F  
00404B6D  MOV DWORD PTR DS:\[40F028\],EAX  
00404B72  MOV DWORD PTR DS:\[40F02C\],EAX  
00404B77  CALL dumped\_.00404864                               ;  据取得的文件大小，再加 0x1000 大小进行分配内存  
00404B7C  JE SHORT dumped\_.00404B97  
00404B7E  PUSH dumped\_.0040ACDC                               ; /lParam = 40ACDC  
00404B83  PUSH 0                                              ; |wParam = 0  
00404B85  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404B8A  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404B90  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404B95  JMP SHORT dumped\_.00404B5D  
00404B97  CALL dumped\_.0040483A                               ;  把文件读入内存  
00404B9C  JE SHORT dumped\_.00404BF4  
00404B9E  PUSH dumped\_.0040AD0E                               ; /lParam = 40AD0E  
00404BA3  PUSH 0                                              ; |wParam = 0  
00404BA5  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404BAA  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404BB0  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404BB5  CALL dumped\_.0040488E  
00404BBA  PUSH 0                                              ; /lParam = 0  
00404BBC  PUSH DWORD PTR DS:\[40EFE8\]                          ; |wParam = C503D0  
00404BC2  PUSH 170                                            ; |Message = STM\_SETICON  
00404BC7  PUSH DWORD PTR DS:\[40EF98\]                          ; |hWnd = 9D03EE  
00404BCD  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404BD2  CMP BYTE PTR DS:\[40F0AD\],0  
00404BD9  JNZ SHORT dumped\_.00404B5D  
00404BDB  PUSH 0                                              ; /lParam = 0  
00404BDD  PUSH 0                                              ; |wParam = 0  
00404BDF  PUSH 402                                            ; |Message = WM\_USER+2  
00404BE4  PUSH DWORD PTR DS:\[40EFC8\]                          ; |hWnd = 29038E  
00404BEA  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404BEF  JMP dumped\_.00404B5D  
00404BF4  MOV EDI,DWORD PTR DS:\[40F014\]  
00404BFA  PUSH dumped\_.00410B61                               ; /lParam = 410B61  
00404BFF  PUSH 0                                              ; |wParam = 0  
00404C01  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404C06  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404C0C  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404C11  PUSH dumped\_.00409FE7                               ; /lParam = 409FE7  
00404C16  PUSH 0                                              ; |wParam = 0  
00404C18  PUSH 180                                            ; |Message = LB\_ADDSTRING  
00404C1D  PUSH DWORD PTR DS:\[40EFA8\]                          ; |hWnd = 7B0392  
00404C23  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404C28  PUSH 0                                              ; /lParam = 0  
00404C2A  PUSH 64                                             ; |wParam = 64  
00404C2C  PUSH 402                                            ; |Message = WM\_USER+2  
00404C31  PUSH DWORD PTR DS:\[40EFC8\]                          ; |hWnd = 29038E  
00404C37  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404C3C  PUSH 0                                              ; /lParam = 0  
00404C3E  PUSH DWORD PTR DS:\[40EFE4\]                          ; |wParam = 4A03F4  
00404C44  PUSH 170                                            ; |Message = STM\_SETICON  
00404C49  PUSH DWORD PTR DS:\[40EF98\]                          ; |hWnd = 9D03EE  
00404C4F  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404C54  PUSH 0                                              ; /lParam = 0  
00404C56  PUSH 0                                              ; |wParam = 0  
00404C58  PUSH 0F0                                            ; |Message = BM\_GETCHECK  
00404C5D  PUSH DWORD PTR DS:\[40EF4C\]                          ; |hWnd = 5E02DE  
00404C63  CALL <JMP.&user32.SendMessageA>                     ; \\SendMessageA  
00404C68  CMP EAX,1  
00404C6B  JE SHORT dumped\_.00404C72  
00404C6D  CALL dumped\_.00406465                               ;  备份  
00404C72  PUSH EDI                                            ; /Arg1 = 00D50000  
00404C73  CALL dumped\_.00405905                               ; \\是否加密判断，请看下面分析  
00404C78  JB dumped\_.00404BB5  
00404C7E  PUSH dumped\_.00409FE7                                    ; /lParam = 409FE7  
00404C83  PUSH 0                                                   ; |wParam = 0  
00404C85  PUSH 180                                                 ; |Message = LB\_ADDSTRING  
00404C8A  PUSH DWORD PTR DS:\[40EFA8\]                               ; |hWnd = 7B0392  
00404C90  CALL <JMP.&user32.SendMessageA>                          ; \\SendMessageA  
00404C95  MOV EAX,DWORD PTR DS:\[EDI+3C\]  
00404C98  MOV DWORD PTR SS:\[EBP\-10\],EAX  
00404C9B  ADD EDI,EAX  
00404C9D  MOV DWORD PTR SS:\[EBP\-18\],EDI  
00404CA0  MOV EAX,DWORD PTR DS:\[EDI+3C\]  
00404CA3  CMP EAX,200  
00404CA8  JE SHORT dumped\_.00404CF5  
00404CAA  PUSHAD  
00404CAB  PUSH EAX                                                 ; /<%.4lX>  
00404CAC  PUSH dumped\_.0040AF77                                    ; |Format = "已调整文件队列: %.4lXh -> 0200h"  
00404CB1  PUSH dumped\_.00410D2D                                    ; |s = dumped\_.00410D2D  
00404CB6  CALL <JMP.&user32.wsprintfA>                             ; \\wsprintfA  
00404CBB  ADD ESP,0C  
00404CBE  PUSH dumped\_.00410D2D                                    ; /lParam = 410D2D  
00404CC3  PUSH 0                                                   ; |wParam = 0  
00404CC5  PUSH 180                                                 ; |Message = LB\_ADDSTRING  
00404CCA  PUSH DWORD PTR DS:\[40EFA8\]                               ; |hWnd = 7B0392  
00404CD0  CALL <JMP.&user32.SendMessageA>                          ; \\SendMessageA  
  
...(待续)....  
  
  
\-------------------------------------------------  
00404C73  CALL dumped\_.00405905:  
  
00405905  ENTER 4,0  
00405909  PUSHAD  
0040590A  AND DWORD PTR SS:\[EBP\-4\],0  
0040590E  AND DWORD PTR DS:\[40F020\],0  
00405915  AND DWORD PTR DS:\[40F024\],0  
0040591C  MOV EDI,DWORD PTR SS:\[EBP+8\]             ;  映射内存的地址  
0040591F  CMP WORD PTR DS:\[EDI\],5A4D               ;  是 dos 格式吗  
00405924  JE SHORT dumped\_.00405935  
00405926  PUSH dumped\_.0040B404                    ; /Arg1 = 0040B404  
0040592B  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405930  JMP dumped\_.00405CEE  
00405935  MOV EAX,DWORD PTR DS:\[EDI+3C\]  
00405938  CMP EAX,DWORD PTR DS:\[40F028\]            ;  是否到文件尾  
0040593E  JL SHORT dumped\_.00405942  
00405940  JMP SHORT dumped\_.00405926  
00405942  ADD EDI,EAX                              ;  加基址，定位到 pe 头  
00405944  CMP DWORD PTR DS:\[EDI\],4550              ;  是否为 pe 文件  
0040594A  JE SHORT dumped\_.0040594E  
0040594C  JMP SHORT dumped\_.00405926  
0040594E  CMP DWORD PTR DS:\[EDI+3C\],200            ;  文件对齐是否为 200  
00405955  JGE SHORT dumped\_.00405966  
00405957  PUSH dumped\_.0040B606                    ; /Arg1 = 0040B606  
0040595C  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405961  JMP dumped\_.00405CEE  
00405966  TEST DWORD PTR DS:\[EDI+F4\],100000        ;  pe 头+f4 处，此为保留值，看是否为 100000,加密标志  
00405970  JE SHORT dumped\_.004059A8  
00405972  PUSH dumped\_.0040B57C                    ; /Arg1 = 0040B57C  
00405977  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
0040597C  PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
0040597E  PUSH dumped\_.0040B297                    ; |Title = "确认"  
00405983  PUSH dumped\_.0040B29F                    ; |Text = "该文件似乎已被压缩或加密。  
您真要继续吗？"  
00405988  PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
0040598E  CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00405993  CMP EAX,6  
00405996  JNZ dumped\_.00405CEE  
0040599C  MOV DWORD PTR DS:\[EDI+F4\],0  
004059A6  JMP SHORT dumped\_.004059F6  
004059A8  TEST DWORD PTR DS:\[EDI+F4\],200000        ;  PE 头+0xf4 处是否为 200000,是则出错,这个就是 telock 的加密标志  
004059B2  JE SHORT dumped\_.004059C3  
004059B4  PUSH dumped\_.0040B555                    ; /Arg1 = 0040B555  
004059B9  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
004059BE  JMP dumped\_.00405CEE  
004059C3  CMP DWORD PTR DS:\[EDI+F4\],0              ;  是否为 0，未加密  
004059CA  JE SHORT dumped\_.004059F6  
004059CC  PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
004059CE  PUSH dumped\_.0040B297                    ; |Title = "确认"  
004059D3  PUSH dumped\_.0040B29F                    ; |Text = "该文件似乎已被压缩或加密。  
您真要继续吗？"  
004059D8  PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
004059DE  CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
004059E3  CMP EAX,6  
004059E6  JNZ dumped\_.00405CEE  
004059EC  MOV DWORD PTR DS:\[EDI+F4\],0  
004059F6  CMP DWORD PTR DS:\[EDI+9C\],0              ;  certificate table size 是否为 0  
004059FD  JE SHORT dumped\_.00405A0E  
004059FF  PUSH dumped\_.0040B483                    ; /Arg1 = 0040B483  
00405A04  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405A09  JMP dumped\_.00405CEE  
00405A0E  CMP DWORD PTR DS:\[EDI+8\],4F434550        ;  比较 Timedatestamp,是否为 2012 年  
00405A15  JNZ SHORT dumped\_.00405A37  
00405A17  PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
00405A19  PUSH dumped\_.0040B297                    ; |Title = "确认"  
00405A1E  PUSH dumped\_.0040B384                    ; |Text = "该文件已被其他工具加密或压缩。  
您真要继续吗？(不推荐)"  
00405A23  PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
00405A29  CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00405A2E  CMP EAX,6  
00405A31  JNZ dumped\_.00405CEE  
00405A37  MOVZX EDX,WORD PTR DS:\[EDI+14\]           ;  可选头大小  
00405A3B  ADD EDX,18                               ;  加 Filehead  
00405A3E  LEA ESI,DWORD PTR DS:\[EDI+EDX\]           ;  定位到节表啦  
00405A41  MOV EAX,DWORD PTR DS:\[EDI+28\]            ;  入口地址  
00405A44  TEST EAX,EAX  
00405A46  JE SHORT dumped\_.00405A93  
00405A48  CMP EAX,DWORD PTR DS:\[ESI+C\]             ;  esi+c 为节表的起始 roffset  
00405A4B  JGE SHORT dumped\_.00405A93  
00405A4D  PUSH dumped\_.0040B41E                    ; /Arg1 = 0040B41E  
00405A52  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405A57  JMP dumped\_.00405CEE  
00405A5C  CMP DWORD PTR SS:\[EBP\-4\],0  
00405A60  JNZ dumped\_.00405B93  
00405A66  PUSH ECX  
00405A67  PUSH EDX  
00405A68  PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
00405A6A  PUSH dumped\_.0040B297                    ; |Title = "确认"  
00405A6F  PUSH dumped\_.0040B384                    ; |Text = "该文件已被其他工具加密或压缩。  
您真要继续吗？(不推荐)"  
00405A74  PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
00405A7A  CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00405A7F  POP EDX  
00405A80  POP ECX  
00405A81  CMP EAX,6  
00405A84  JNZ dumped\_.00405CEE  
00405A8A  OR DWORD PTR SS:\[EBP\-4\],1  
00405A8E  JMP dumped\_.00405B93  
00405A93  CMP EAX,DWORD PTR DS:\[ESI+34\]            ;  与下一节的内存偏移比较,这里是判断入口点是否在第一个节中  
00405A96  JBE SHORT dumped\_.00405AB8  
00405A98  PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
00405A9A  PUSH dumped\_.0040B297                    ; |Title = "确认"  
00405A9F  PUSH dumped\_.0040B2F5                    ; |Text = "该文件的入口点大于区段 2 的 RVA。原因可能是该文件  
已被加密或压缩。您真要继续吗？"  
00405AA4  PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00340288 ('tElock v0.98',class='tEWinClass')  
00405AAA  CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00405AAF  CMP EAX,6  
00405AB2  JNZ dumped\_.00405CEE  
00405AB8  MOVZX ECX,WORD PTR DS:\[EDI+6\]            ;  节表的个数  
00405ABC  XOR EBX,EBX  
00405ABE  CMP DWORD PTR DS:\[ESI\],7073612E  
00405AC4  JE SHORT dumped\_.00405A5C                ;  节表名字是：.asp 吗,以下判断节表名字，看是否加过密  
00405AC6  CMP DWORD PTR DS:\[ESI\],30585055          ;  是 upx0?  
00405ACC  JE SHORT dumped\_.00405A5C  
00405ACE  CMP DWORD PTR DS:\[ESI\],21585055          ;  upx!  ?  
00405AD4  JE SHORT dumped\_.00405A5C  
00405AD6  CMP DWORD PTR DS:\[ESI\],6C6B702E          ;  .pkl  
00405ADC  JE dumped\_.00405A5C  
00405AE2  CMP DWORD PTR DS:\[ESI\],7268732E          ;  .shr  
00405AE8  JE dumped\_.00405A5C  
00405AEE  CMP DWORD PTR DS:\[ESI\],5057572E          ;  .wwp  
00405AF4  JE dumped\_.00405A5C  
00405AFA  CMP DWORD PTR DS:\[ESI\],7972432E          ;  .cry  
00405B00  JE dumped\_.00405A5C  
00405B06  CMP DWORD PTR DS:\[ESI\],7268732E          ;  .shr  
00405B0C  JE dumped\_.00405A5C  
00405B12  CMP DWORD PTR DS:\[ESI\],5057572E          ;  .wwp  
00405B18  JE dumped\_.00405A5C  
00405B1E  CMP DWORD PTR DS:\[ESI\],31636570          ;  pec1  
00405B24  JE dumped\_.00405A5C  
00405B2A  CMP DWORD PTR DS:\[ESI\],48534550          ;  pesh  
00405B30  JE dumped\_.00405A5C  
00405B36  CMP DWORD PTR DS:\[ESI\],4F4C4550          ;  pelo  
00405B3C  JE dumped\_.00405A5C  
00405B42  CMP DWORD PTR DS:\[ESI\],464A422E          ;  .BJF  
00405B48  JE dumped\_.00405A5C  
00405B4E  CMP DWORD PTR DS:\[ESI\],6369662E          ;  .fic  
00405B54  JE dumped\_.00405A5C  
00405B5A  CMP DWORD PTR DS:\[ESI\],41504550          ;  PEPA  
00405B60  JE dumped\_.00405A5C  
00405B66  CMP DWORD PTR DS:\[ESI\],41746942          ;  BitA  
00405B6C  JE dumped\_.00405A5C  
00405B72  CMP DWORD PTR DS:\[ESI\],6F656E2E          ;  .neo  
00405B78  JE dumped\_.00405A5C  
00405B7E  CMP DWORD PTR DS:\[ESI\],30455354          ;  TSE0  
00405B84  JE dumped\_.00405A5C  
00405B8A  CMP DWORD PTR DS:\[ESI\],0                 ;  节表名字是 0 吗  
00405B8D  JE dumped\_.00405A5C  
00405B93  MOV EAX,DWORD PTR DS:\[ESI+10\]            ;  节在文件中大小 Rsize  
00405B96  CMP DWORD PTR DS:\[ESI+8\],EAX             ;  与内存中的大小比较 Vsize  
00405B99  JGE SHORT dumped\_.00405B9E  
00405B9B  |>MOV DWORD PTR DS:\[ESI+8\],EAX             ;  改成与文件的大小一样  
00405B9E  |>MOV EAX,DWORD PTR DS:\[ESI+C\]             ;  文件偏移  
00405BA1  |>ADD EAX,DWORD PTR DS:\[ESI+10\]            ;  加上文件中节的大小  
00405BA4  |>CMP EAX,DWORD PTR DS:\[EDI+50\]            ;  比较 sizeofimage  
00405BA7  |>JLE SHORT dumped\_.00405BB8  
00405BA9  PUSH dumped\_.0040B4BC                    ; /Arg1 = 0040B4BC  
00405BAE  CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405BB3  JMP dumped\_.00405CEE  
00405BB8  MOV EAX,DWORD PTR DS:\[ESI+14\]            ;  文件偏移  
00405BBB  TEST EAX,EAX  
00405BBD  JE SHORT dumped\_.00405BD8  
00405BBF  ADD EAX,DWORD PTR DS:\[ESI+10\]            ;  加上 Rsize  
00405BC2  CMP EBX,EAX  
00405BC4  JGE SHORT dumped\_.00405BD8  
00405BC6  PUSH ECX  
00405BC7  MOV ECX,DWORD PTR DS:\[EDI+3C\]            ;  文件对齐  
00405BCA  XOR EDX,EDX  
00405BCC  DIV ECX  
00405BCE  TEST EDX,EDX  
00405BD0  JE SHORT dumped\_.00405BD3  
00405BD2  INC EAX  
00405BD3  MUL ECX  
00405BD5  POP ECX  
00405BD6  MOV EBX,EAX  
00405BD8  ADD ESI,28                               ;  下一节  
00405BDB  DEC ECX  
00405BDC  JG dumped\_.00405ABE  
00405BE2  |>CMP EBX,DWORD PTR DS:\[40F028\]  
00405BE8  |>JNB dumped\_.00405CCC  
00405BEE  |>PUSH 0                                   ; /lParam = 0  
00405BF0  |>PUSH 0                                   ; |wParam = 0  
00405BF2  |>PUSH 0F0                                 ; |Message = BM\_GETCHECK  
00405BF7  |>PUSH DWORD PTR DS:\[40EF48\]               ; |hWnd = 1027E  
00405BFD  |>CALL <JMP.&user32.SendMessageA>          ; \\SendMessageA  
00405C02  |>CMP EAX,1  
00405C05  |>JNZ SHORT dumped\_.00405C45  
00405C07  |>SUB EBX,DWORD PTR DS:\[40F028\]  
00405C0D  |>/NEG EBX  
00405C0F  |>\\JS SHORT dumped\_.00405C0D  
00405C11  |>PUSH EBX                                 ; /<%d>  
00405C12  |>PUSH dumped\_.0040AF50                    ; |Format = "已找到文件重复占位段，截去了 %d 字节。"  
00405C17  |>PUSH dumped\_.00410D2D                    ; |s = dumped\_.00410D2D  
00405C1C  |>CALL <JMP.&user32.wsprintfA>             ; \\wsprintfA  
00405C21  |>ADD ESP,0C  
00405C24  |>PUSH dumped\_.00410D2D                    ; /lParam = 410D2D  
00405C29  |>PUSH 0                                   ; |wParam = 0  
00405C2B  |>PUSH 180                                 ; |Message = LB\_ADDSTRING  
00405C30  |>PUSH DWORD PTR DS:\[40EFA8\]               ; |hWnd = 102A4  
00405C36  |>CALL <JMP.&user32.SendMessageA>          ; \\SendMessageA  
00405C3B  |>CALL dumped\_.004067EB  
00405C40  |>JMP dumped\_.00405CCC  
00405C45  |>MOV EAX,EBX  
00405C47  |>XOR EDX,EDX  
00405C49  |>MOV ECX,DWORD PTR DS:\[EDI+38\]  
00405C4C  |>DIV ECX  
00405C4E  |>TEST EDX,EDX  
00405C50  |>JE SHORT dumped\_.00405C53  
00405C52  |>INC EAX  
00405C53  |>MUL ECX  
00405C55  |>CMP EAX,DWORD PTR DS:\[40F028\]  
00405C5B  |>JNZ SHORT dumped\_.00405C79  
00405C5D  |>PUSH 24                                  ; /Style = MB\_YESNO|MB\_ICONQUESTION|MB\_APPLMODAL  
00405C5F  |>PUSH dumped\_.0040B28F                    ; |Title = "确认"  
00405C64  |>PUSH dumped\_.0040B1C0                    ; |Text = "已检测到重复占位段 - 大小匹配文件的物理结尾与  
下一个队列边界之间的差距。原因可能是使用了一  
个边界链接器且在大多数情况下能被截去...  
您要删除该重复占位段吗？"  
00405C69  |>PUSH DWORD PTR DS:\[40EF28\]               ; |hOwner = 00030262 ('tElock v0.98',class='tEWinClass')  
00405C6F  |>CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00405C74  |>CMP EAX,6  
00405C77  |>JE SHORT dumped\_.00405CCC  
00405C79  |>MOV ESI,EBX  
00405C7B  |>SUB EBX,DWORD PTR DS:\[40F028\]  
00405C81  |>/NEG EBX  
00405C83  |>\\JS SHORT dumped\_.00405C81  
00405C85  |>MOV DWORD PTR DS:\[40F024\],EBX  
00405C8B  |>PUSH 4                                   ; /Protect = PAGE\_READWRITE  
00405C8D  |>PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00405C92  |>PUSH EBX                                 ; |Size  
00405C93  |>PUSH 0                                   ; |Address = NULL  
00405C95  |>CALL <JMP.&kernel32.VirtualAlloc>        ; \\VirtualAlloc  
00405C9A  |>TEST EAX,EAX  
00405C9C  |>JNZ SHORT dumped\_.00405CAA  
00405C9E  |>PUSH dumped\_.0040B4BC                    ; /Arg1 = 0040B4BC  
00405CA3  |>CALL dumped\_.00405D08                    ; \\dumped\_.00405D08  
00405CA8  |>JMP SHORT dumped\_.00405CEE  
00405CAA  |>MOV DWORD PTR DS:\[40F020\],EAX  
00405CAF  |>MOV EDI,EAX  
00405CB1  |>MOV ECX,DWORD PTR DS:\[40F024\]  
00405CB7  |>ADD ESI,DWORD PTR DS:\[40F014\]  
00405CBD  |>MOV EAX,ECX  
00405CBF  |>AND EAX,3  
00405CC2  |>SHR ECX,2  
00405CC5  |>CLD  
00405CC6  |>REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS\>  
00405CC8  |>MOV ECX,EAX  
00405CCA  |>REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>  
00405CCC  |>PUSH dumped\_.0040B5EF                    ; /lParam = 40B5EF  
00405CD1  |>PUSH 0                                   ; |wParam = 0  
00405CD3  |>PUSH 180                                 ; |Message = LB\_ADDSTRING  
00405CD8  |>PUSH DWORD PTR DS:\[40EFA8\]               ; |hWnd = 102A4  
00405CDE  |>CALL <JMP.&user32.SendMessageA>          ; \\SendMessageA  
00405CE3  |>CALL dumped\_.004067EB  
00405CE8  |>POPAD  
00405CE9  |>CLC  
00405CEA  |>LEAVE  
00405CEB  |>RETN 4  
\----------------------------------------------  
  
总结：这是 Telock 加密的第一部分，判断是否加密等，为后面进一步加密作准备。  
  
By lordor  6.24

 4dd466d0e1e0cc123afbe504da2462ca