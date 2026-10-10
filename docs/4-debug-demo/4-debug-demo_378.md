

# BrickShooter 2.

> 原文：[`www.pediy.com/kssd/pediy03/forum690.htm`](https://www.pediy.com/kssd/pediy03/forum690.htm)

BrickShooter 2.1 破解心得（新手看看吧）   
  
  
这个软件是一个打砖块的小游戏，我个人感觉还不错，   
  
在海阔天空处下载，Unregistered 版本 有 30 天的   
  
时间限制，每局 30 分钟的限制，不能存盘，退出时还   
  
有 NAG，就拿它练习练习吧。   
  
  
工具：CASPR 1.00,Soft-ice 4.01,UltraEdit 8.0   
  
      Exescope 6.00,icedump,FileInfo   
  
用 FileInfo 查看是用 Asprotect 1.00 加的壳，但是   
  
我试着手工脱壳总是不能成功（哪位高手能指点一   
  
下？），Import Table 总是不能正确还原，看来我   
  
要好好学习一下 PE 文件结构了。还好最近出了个 CAS，   
  
太棒了，对我这样的菜鸟来说真是一个福音:)   
  
下面正式开始工作：   
  
1、打开一个 DOS 窗口，下命令   
  
　　caspr bshoot.exe bs.exe   
  
  这样把壳给脱了。   
  
2、用 ice 载入 bs.exe　F5　运行，单击“SAVE”按钮，   
  
　　它会弹出一个框，说“You are unable.....”   
  
　　ctrl-D 切入 sice,下断点   
  
　　bpx destroywindow   
    
    F5 运行，返回 BShooter,单击确定，被 sice 拦下，   
      
    bc \*　清除断点，F11 返回到 BS 中，下命令   
  
    s 30:0 l ffffffff 'You are unable'   
  
    找到如下地址：   
  
0030:0045B9B8 59 6F 75 20 61 72 65 20-75 6E 61 62 6C 65 20 74  You are unable t   
0030:0045B9C8 6F 20 6C 6F 61 64 20 6F-72 20 73 61 76 65 20 67  o load or save g   
0030:0045B9D8 61 6D 65 20 62 65 63 61-75 73 65 20 79 6F 75 72  ame because your   
0030:0045B9E8 20 63 6F 70 79 20 6F 66-20 42 72 69 63 6B 53 68  copy of BrickSh   
0030:0045B9F8 6F 6F 74 65 72 20 69 73-20 55 4E 52 45 47 49 53  ooter is UNREGIS   
0030:0045BA08 54 45 52 45 44 2E 0D 43-6C 69 63 6B 20 74 68 65  TERED..Click the   
0030:0045BA18 20 52 65 67 69 73 74 65-72 20 6C 61 62 65 6C 20  Register label   
0030:0045BA28 66 6F 72 20 6D 6F 72 65-20 69 6E 66 6F 72 6D 61  for more informa   
  
3、设断　　bpm 0045b9b8   
    
  F5 运行，单击“SAVE”按钮，被 sice 拦下，按 F12 一层层地返回到 BS 中，   
  
  如一直按 F10，最终会在几个 ret 处跳来跳去，那窗口总是不出现，   
  
  没关系，按 F12 二下,窗口弹出了，单击确定后又被 sice 拦下，   
  
  到达下面的地方：   
  
015F:00449F6F  CALL      004400D0                                                
015F:00449F74  MOV      EAX,\[EBP\-0C\]                                            
015F:00449F77  MOV      EDX,\[EAX\]                                                
015F:00449F79  CALL      \[EDX+000000CC\]                                          
015F:00449F7F  MOV      \[EBP\-08\],EAX                                            
015F:00449F82  XOR      EAX,EAX                                                  
015F:00449F84  POP      EDX                                                      
015F:00449F85  POP      ECX                                                      
015F:00449F86  POP      ECX                                                      
015F:00449F87  MOV      FS:\[EAX\],EDX                                            
015F:00449F8A  PUSH      00449F9F                                                
015F:00449F8F  MOV      EAX,\[EBP\-0C\]                                            
015F:00449F92  CALL      00402E54                                                
015F:00449F97  RET                                                                
015F:00449F98  JMP      004034AC                                                
015F:00449F9D  JMP      00449F8F                                                
015F:00449F9F  MOV      EAX,\[EBP\-08\]                                            
015F:00449FA2  POP      ESI                                                      
015F:00449FA3  POP      EBX                                                      
015F:00449FA4  MOV      ESP,EBP                                                  
015F:00449FA6  POP      EBP                                                      
015F:00449FA7  RET      0010                                                    
015F:00449FAA  MOV      EAX,EAX                                                  
015F:00449FAC  OR        ECX,-01                                                  
015F:00449FAF  OR        EDX,-01                                                  
015F:00449FB2  CALL      00449FB8                                                
015F:00449FB7  RET                                                                
015F:00449FB8  PUSH      00                                                      
015F:00449FBA  PUSH      EDX                                                      
015F:00449FBB  PUSH      ECX                                                      
015F:00449FBC  MOV      DL,04                                                    
015F:00449FBE  MOV      CX,\[00449FCC\]                                            
015F:00449FC5  CALL      00449ED4                                                
015F:00449FCA  RET                                                                
  
返回几个 CALL 之后，就可到达下面的关键之处   
  
015F:0045B153  MOV      EDI,\[00461FD4\]                                          
015F:0045B159  ADD      EDI,00000100                                            
015F:0045B15F  MOV      EAX,EDI                                                  
015F:0045B161  ADD      EAX,32                                                  
015F:0045B164  PUSH      EAX                                                      
015F:0045B165  PUSH      EBX                                                      
015F:0045B166  PUSH      ESI                                                      
015F:0045B167  MOV      ECX,\[00461FD8\]                                          
015F:0045B16D  ADD      ECX,000001EF                                            
015F:0045B173  ADD      ECX,23                                                  
015F:0045B176  ADD      ECX,09                                                  
015F:0045B179  MOV      EDX,EDI                                                  
015F:0045B17B  MOV      EAX,\[00461FD8\]                                          
015F:0045B180  ADD      EAX,000001E5                                            
015F:0045B185  ADD      EAX,09                                                  
015F:0045B188  CALL      00458BC8                                                
015F:0045B18D  TEST      AL,AL                                                    
015F:0045B18F  JZ        0045B1C6                                                
015F:0045B191  CMP      BYTE PTR \[0046208C\],00    ;看见这个比较了吧   
015F:0045B198  JNZ      0045B1B4                  ;这个跳转是关键   
015F:0045B19A  PUSH      00                                                      
015F:0045B19C  MOV      CX,\[0045B9AC\]                                            
015F:0045B1A3  MOV      DL,02                                                    
015F:0045B1A5  MOV      EAX,0045B9B8                                            
015F:0045B1AA  CALL      00449EB4                                                
015F:0045B1AF  JMP      0045B9A0                                  
015F:0045B1B4  MOV      EAX,\[EBP\-04\]                                            
015F:0045B1B7  MOV      EAX,\[EAX+00000318\]                                      
015F:0045B1BD  MOV      ECX,ESI                                                  
015F:0045B1BF  MOV      EDX,EBX                                                  
015F:0045B1C1  MOV      EDI,\[EAX\]                                                
015F:0045B1C3  CALL      \[EDI+3C\]                                                
015F:0045B1C6  MOV      EDI,\[00461FD4\]                                          
015F:0045B1CC  ADD      EDI,00000155                                            
015F:0045B1D2  MOV      EAX,EDI                                                  
015F:0045B1D4  ADD      EAX,30                                                  
015F:0045B1D7  PUSH      EAX                                                      
015F:0045B1D8  PUSH      EBX                                                    
015F:0045B1D9  PUSH      ESI                                                
  
　　显然，\[0046208C\]处存放的是注册标志，   
  
4、清除以前所有断点，重新设断   
  
bpm 0046208c   
  
重新载入，在如下处拦下：   
  
015F:00458B94  INVALID                                  
015F:00458B96  INVALID                                  
015F:00458B98  ADD      AL,00                          
015F:00458B9A  ADD      \[EAX\],AL                      
015F:00458B9C  INSD                                    
015F:00458B9E  IMUL      ESP,\[EAX+EAX+00\],05C60000      
015F:00458BA6  MOV      \[EAX\],FS                      
015F:00458BA8  INC      ESI                            
015F:00458BA9  ADD      \[EAX\],AL                      
015F:00458BAB  JMP      00458BB1              ;在这儿被拦下，        
015F:00458BAD  JMP      00458BB4                      
015F:00458BAF  MOV      \[ECX+000007E9\],ECX                                      
015F:00458BB5  ADD      \[ESI+53F23D5B\],AL                                        
015F:00458BBB  MOV      ECX,EB04EB5A                                            
015F:00458BC0  ADD      EAX,C3C39999                                            
015F:00458BC5  LEA      EAX,\[EAX+00\]                                            
015F:00458BC8  PUSH      EBP                                                      
015F:00458BC9  MOV      EBP,ESP                                                  
015F:00458BCB  PUSH      ESI                                                      
015F:00458BCC  PUSH      EDI                                                      
015F:00458BCD  MOV      EDI,\[EBP+08\]                                            
015F:00458BD0  MOV      ESI,\[EBP+0C\]                                            
015F:00458BD3  CMP      EAX,ESI                                                  
015F:00458BD5  JG        00458BE4                                                
015F:00458BD7  CMP      ECX,ESI                                                  
015F:00458BD9  JL        00458BE4                                                
015F:00458BDB  CMP      EDX,EDI                                                  
015F:00458BDD  JG        00458BE4                                                
015F:00458BDF  CMP      EDI,\[EBP+10\]                                            
015F:00458BE2  JLE      00458BE8                                                
015F:00458BE4  XOR      EAX,EAX                                                  
015F:00458BE6  JMP      00458BEA                                                
015F:00458BE8  MOV      AL,01                                                    
015F:00458BEA  POP      EDI                                                      
015F:00458BEB  POP      ESI                                                      
015F:00458BEC  POP      EBP                                                      
015F:00458BED  RET      000C      
  
00458bab 处的上一条指令应该是写注册标志的指令，   
  
但是上面的指令已经“花”了，没关系，F5 运行，   
  
同样，在相同地方被拦下，按 f10 一步一步返回到   
  
上一层,如下   
  
015F:00458F09  MOV      EBP,ESP                        
015F:00458F0B  PUSH      00                              
015F:00458F0D  PUSH      00                              
015F:00458F0F  PUSH      EBX                            
015F:00458F10  PUSH      ESI                            
015F:00458F11  PUSH      EDI                            
015F:00458F12  MOV      EBX,EAX                        
015F:00458F14  XOR      EAX,EAX                        
015F:00458F16  PUSH      EBP                            
015F:00458F17  PUSH      00459227                        
015F:00458F1C  PUSH      DWORD PTR FS:\[EAX\]              
015F:00458F1F  MOV      FS:\[EAX\],ESP                    
015F:00458F22  CALL      00458BA4    ;就是这个 CALL                      
015F:00458F27  MOV      EAX,\[0045FEBC\]                  
015F:00458F2C  MOV      EAX,\[EAX\]                      
015F:00458F2E  MOV      EAX,\[EAX+000002D0\]              
015F:00458F34  MOV      EDX,\[EBX+00000360\]              
015F:00458F3A  CALL      004495F0                        
015F:00458F3F  MOV      ECX,00459240                    
015F:00458F44  MOV      DL,01                                                    
015F:00458F46  MOV      EAX,\[0044C404\]                                          
015F:00458F4B  CALL      0044C4AC                                                
015F:00458F50  MOV      ESI,EAX                                                  
015F:00458F52  PUSH      01                                                      
015F:00458F54  MOV      ECX,00459254                                            
015F:00458F59  MOV      EDX,00459264                                            
015F:00458F5E  MOV      EAX,ESI                                                  
015F:00458F60  MOV      EDI,\[EAX\]                                                
015F:00458F62  CALL      \[EDI+10\]                                                
015F:00458F65  MOV      EDX,EAX                                                  
015F:00458F67  MOV      EAX,\[0045FEBC\]                                          
015F:00458F6C  MOV      EAX,\[EAX\]                                                
015F:00458F6E  MOV      EAX,\[EAX+000002C4\]                                      
015F:00458F74  MOV      ECX,\[EAX\]                                                
015F:00458F76  CALL      \[ECX+000000BC\]                                          
015F:00458F7C  PUSH      01                                                      
015F:00458F7E  MOV      ECX,00459274                                            
015F:00458F83  MOV      EDX,00459264                                            
015F:00458F88  MOV      EAX,ESI                                            
  
  
在 015F:00458F22 处设断，并清除以前断点，   
  
重新载入，运行被拦下后，按 F8 进入该 CALL   
  
015F:00458BA4  MOV      BYTE PTR \[0046208C\],00  ；就是这个指令                              
015F:00458BAB  JMP      00458BB1                                                
015F:00458BAD  JMP      00458BB4                                                
015F:00458BAF  MOV      \[ECX+000007E9\],ECX                                      
015F:00458BB5  ADD      \[ESI+53F23D5B\],AL                                        
015F:00458BBB  MOV      ECX,EB04EB5A                                            
015F:00458BC0  ADD      EAX,C3C39999                                            
015F:00458BC5  LEA      EAX,\[EAX+00\]                                            
015F:00458BC8  PUSH      EBP                                                      
015F:00458BC9  MOV      EBP,ESP                                                  
015F:00458BCB  PUSH      ESI                                                      
015F:00458BCC  PUSH      EDI                                                      
015F:00458BCD  MOV      EDI,\[EBP+08\]                                            
015F:00458BD0  MOV      ESI,\[EBP+0C\]                                            
015F:00458BD3  CMP      EAX,ESI                
  
这回看到了“花”指令的真面目了吧，下 code on 看一下机器码   
  
015F:00458BA4  C6058C20460000      MOV      BYTE PTR \[0046208C\],00              
015F:00458BAB  EB04                JMP      00458BB1                            
015F:00458BAD  EB05                JMP      00458BB4                            
015F:00458BAF  8989E9070000        MOV      \[ECX+000007E9\],ECX                  
015F:00458BB5  00865B3DF253        ADD      \[ESI+53F23D5B\],AL                    
015F:00458BBB  B95AEB04EB          MOV      ECX,EB04EB5A                        
015F:00458BC0  059999C3C3          ADD      EAX,C3C39999                        
015F:00458BC5  8D4000              LEA      EAX,\[EAX+00\]                        
015F:00458BC8  55                  PUSH      EBP                                  
015F:00458BC9  8BEC                MOV      EBP,ESP                              
015F:00458BCB  56                  PUSH      ESI                                  
015F:00458BCC  57                  PUSH      EDI                                  
015F:00458BCD  8B7D08              MOV      EDI,\[EBP+08\]                        
015F:00458BD0  8B750C              MOV      ESI,\[EBP+0C\]                        
015F:00458BD3  3BC6                CMP      EAX,ESI                              
015F:00458BD5  7F0D                JG        00458BE4                            
015F:00458BD7  3BCE                CMP      ECX,ESI              
  
 4dd466d0e1e0cc123afbe504da2462ca