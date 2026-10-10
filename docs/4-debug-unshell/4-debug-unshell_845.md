

# PE Compact 1.4 beta6

> 原文：[`www.pediy.com/kssd/pediy02/forum278.htm`](https://www.pediy.com/kssd/pediy02/forum278.htm)

 用天意+Procdump 脱 PE Compact 1.4 beta6   
  
本人对 PE 的文件结构也不是很了解！   
1、如果你和我一样使用 s3 的早期显卡，那么你需要安装 sdd v6.53 或者   
sdd v7.0 alpha3   
  
2、运行天意的 test 程序，如果你的显卡支持的话，会生成一个 Trackit.dat 的   
文件！   
  
3、我们这次脱 PE Compact 1.4 beta6 的壳，用天意脱这个软件的壳，   
用天意似乎比 trw2000 更快来到关键处！   
  
4、此篇文章由于我的水平有限无法涉及让他脱壳后跨平台运行的问题！   
不过在本机应该可以使用！主要是介绍天意为 cracker 们特设的 suspend 命令！   
  
好开始，首先用 load 载入 pecompact.exe 文件！   
来到如下程序段：   
015F:00419800  JMP      SHORT 00419808 -->中断与此，按 f10 继续   
015F:00419802  PUSH    DWORD 00019000   
015F:00419807  RET      
015F:00419808  PUSHF  -->按一下 f10 来到了这里，继续用 f10 单步执行！      
015F:00419809  PUSHA    
015F:0041980A  CALL    00419811 -->来到这里，不要用 f10,改用 f8,f10 继续！   
015F:0041980F  XOR      EAX,EAX   
015F:00419811  MOV      EAX,ESP   
015F:00419813  ADD      EAX,BYTE +04   
015F:00419816  XCHG    EAX,EBX   
015F:00419817  MOV      ESP,EBX   
015F:00419819  MOV      EBX,\[EBX\-04\]   
015F:0041981C  SUB      EBX,0040A00F   
015F:00419822  XCHG    EBX,EBP   
    .   
    .   
    .   
    .   
    .   
    .   
..过不了多久就来到这里！   
015F:00419850  PUSH    EBX   
015F:00419851  PUSH    EBX   
015F:00419852  PUSH    EBX   
015F:00419853  PUSH    EBX   
015F:00419854  POP      EAX   
015F:00419855  SUB      EAX,0040A070   
015F:0041985A  MOV      \[EBP+0040A071\],EAX   
015F:00419860  POP      EDI   
015F:00419861  LEA      ESI,\[EBP+0040A070\]   
015F:00419867  MOV      ECX,0447   
015F:0041986C  REP MOVSD -->看见这条指令了吧！在天意里不管用 f10 或 f8 都是   
                            要运行很长时间的（在 trw2000 里可以用 f10 带过）   
                            其实，在这里，我们可以用天意的 g 命令跳过！   
                            马上会来到这里...   
      .   
      .   
015F:0041B40B  NOP      
015F:0041B40C  MOV      ESI,\[EBP+0040A5DC\]   
015F:0041B412  MOV      EDI,\[EBP+0040A5E0\]   
015F:0041B418  CALL    0041C0A4   
015F:0041B41D  POPA      
015F:0041B41E  POPF      
015F:0041B41F  PUSH    EAX   
015F:0041B420  PUSH    DWORD 00419000 -->在 41986c 处，使用 g 命令后来到了这里   
                                          在按 f10   
015F:0041B425  RET      04  -->来到这里用 f10 带过，就来到程序脱壳的   
                              关键处！   
..关键处代码：   
015F:00419000  PUSHF  -->来到这里，用 f10 跑到 419002 处   
015F:00419001  PUSHA    
015F:00419002  CALL    00419009 -->来到这里就可以脱壳了，这里我们用   
                                    suspend 命令回到 windows 环境，启动   
                                    procdump，然后在就可以脱壳了！   
在 procdump 里可以选种我们刚才准备脱壳文件的那个进程！点鼠标右键，选择   
dump(full)   
  
  
注：本人昨天和人打架，眼睛伤了，这篇文章只是简单介绍一下它是如何脱壳的！希望大家   
能看明白，天意的功能十分强大，我无法说的清楚！希望大家关注这款国产的调试器！   
  
                                                      wind\[CCG\]   4dd466d0e1e0cc123afbe504da2462ca