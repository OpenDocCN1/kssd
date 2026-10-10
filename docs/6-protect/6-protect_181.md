

# N188 破解过程(DOS 下的狗)

> 原文：[`www.pediy.com/kssd/pediy04/kanxue420.htm`](https://www.pediy.com/kssd/pediy04/kanxue420.htm)

N188 破解过程   
  
PART A：   
  
0A21:7786  PUSH    BP   
0A21:7787  MOV      BP,SP   
0A21:7789  NOP      
0A21:778A  PUSH    CS   
0A21:778B  CALL    75F3  //这里是关键，F8 进入 PART B   
0A21:778E  JC      77AA   
0A21:7790  MOV      DI,\[BP+02\]   
0A21:7793  ADD      DI,BYTE +16   
0A21:7796  MOV      BX,\[CS:0D\]   
0A21:779B  MOV      BX,BA46   
0A21:779E  SUB      BX,BA32   
0A21:77A2  MOV      \[CS:DI\],BX   
0A21:77A5  MOV      AL,00   
0A21:77A7  JMP      SHORT 77AC   
0A21:77A9  NOP      
0A21:77AA  MOV      AL,01   
0A21:77AC  POP      BP   
  
PART B：   
0A21:75F3  PUSH    BP   
0A21:75F4  MOV      BP,SP   
0A21:75F6  PUSH    SI   
0A21:75F7  PUSH    DS   
0A21:75F8  MOV      AX,1883   
0A21:75FB  MOV      DS,AX   
0A21:75FD  CMP      BYTE \[CS:6286\],01 //我没有改这里，因为在下面的那个 CALL 里会退出。    
                                      //要从退出的 CALL 入手   
0A21:7603  JZ      7613   
0A21:7605  CALL    6D23  //这里是关键的过程，因为下面的 JZ 可以跟到新的片段，而不是结束。    
                          //F8 进入后来到 PART C   
0A21:760A  OR      AX,DX  //注意这里是 AX 和 DX 进入或。所以，从 6D23 中返回的值只有 AX 和 DX 要用到。    
0A21:760C  JZ      7613  //这里的跳转很关键.   
0A21:760E  XOR      AX,AX   
0A21:7610  STC      
0A21:7611  JMP      SHORT 7637   
0A21:7613  MOV      AX,\[CS:67D9\] //下面的这几段代码通常是用来作标志或初始化的。    
0A21:7617  MOV      \[CS:67D5\],AX   
0A21:761B  MOV      AX,\[CS:67DB\]   
0A21:761F  MOV      \[CS:67D7\],AX   
0A21:7623  MOV      BX,6B21   
0A21:7626  MOV      AL,\[CS:67DC\]   
0A21:762A  XOR      AX,23   
  
PART C：   
119B:6D27  MOV      WORD \[BP\-02\],119B   
119B:6D2C  MOV      WORD \[BP\-04\],68C2   
119B:6D31  MOV      WORD \[BP\-06\],119B   
119B:6D36  MOV      WORD \[BP\-08\],6892   
119B:6D3B  PUSH    WORD \[BP\-06\]   
119B:6D3E  PUSH    WORD \[BP\-08\]   
119B:6D41  PUSH    WORD \[BP\-02\]   
119B:6D44  PUSH    WORD \[BP\-04\]   
119B:6D47  PUSH    BYTE +01   
119B:6D49  PUSH    CS   
119B:6D4A  CALL    4484  //当走过这个 CALL 时会退出。所以，我要把这里改成向 AX 和 DX 中赋 0    
119B:6D4D  ADD      SP,BYTE +0A   
119B:6D50  JMP      SHORT 6D52   
119B:6D52  LEAVE    
  
CALL 4484 的 CODE 是 4 个字节，改成 XOR AX，AX 和 XOR DX，DX 正好，改过后就完成破解了。   
  
      罗降神   
  2002.2.18  4dd466d0e1e0cc123afbe504da2462ca