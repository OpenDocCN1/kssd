

# 幻影 2.0 测试版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50054.htm`](https://www.pediy.com/kssd/pediy05/pediy50054.htm)

幻影 2。0 测试版的 Anti-debug   
  
    幻影 2。0 编写得真不错，它的 anti-debug 很有特色，我追它可是害惨了我的爱机。系统崩溃   
不下 20 次，反复重启－－晕！！虽然在半月前就彻底追过了，但是仍有很多问题没有明白。今天将它写出来，望各位大虾多多指教，为本人解惑。   
我觉得它有如下特点：   
1、花指令   
2、SEH 反跟踪   
3、在 Ring3、Ring0 级清除断点寄存器   
4、利用 createfile 查找调试工具（如 sice、trw、bw.superbpm 等等）   
5、断点可不能乱下，不然系统会崩溃（d.boy 大侠说：“设计中应该一触发断点   
  
就会崩溃；崩溃是因为　int3　被幻影占领了”。）   
  
以下的代码都是 d.boy 大侠用幻影 2。0 测试版加壳的 exescope,我已改过的（原是花指令）：   
167:004AC094  PUSH    DWORD \[WORD FS:00\]   
0167:004AC09A  PUSHF    
0167:004AC09B  PUSH    BYTE +03   
0167:004AC0C7  MOV      \[WORD FS:00\],ESP    《————建立 SEH 链   
0167:004AC0CD  PUSHF    
0167:004AC0CE  PUSH    BYTE +03   
0167:004AC0FA  XOR      EAX,EAX      
0167:004AC0FC  PUSHF    
0167:004AC0FD  JC      004AC109   
0167:004AC113  MOV      \[EAX\],EAX            《————产生异常   
0167:004AC115  PUSHF    
0167:004AC116  PUSH    BYTE +03   
0167:004AC118  JNC      004AC125   
0167:004AC11A  JMP      SHORT 004AC1   
  
当光标停在 4ac0fa 处时，看看 stack 中，异常处理入口为 4dca77,这时断点可以下在那里。接下去会有   
很多类似的异常，异常处理入口也在 4dca77 处，但是后面的两个异常的断点就不能设在这里了，不然系统   
就会崩溃的。我胡乱猜测是第一次异常时，读入 4dca77 后的代码，产生页面保护错误，于是就在这里做手脚，就像   
上面 d.boy 大侠说的那样，“int3　被幻影占领了”（各位大侠，我乱猜的，请指教！）   
  
下面我们就进入了异常处理代码中：   
0167:004DCA77 C8000000        ENTER    00,00   
0167:004DCA7B 9C              PUSHF    
0167:004DCA7C 6A03            PUSH    BYTE +03   
0167:004DCA7E 730B            JNC      004DCA8B   
0167:004DCA80 EB02            JMP      SHORT 004DCA84   
  
0167:004DCB94  MOV      EAX,\[EBP+10\]  《————eax 中为 context 的指针   
0167:004DCB97  JPE      004DCB9C   
  
0167:004DCBA1  MOV      EBX,\[EAX+B8\]  《————取出 context.eip   
0167:004DCBA7  JZ      004DCBAC   
  
0167:004DCBAC  ADD      EBX,BYTE +02    《————context.eip+2   
0167:004DCBAF  PUSHF    
  
0167:004DCBC6  ADD      EBX,BYTE +03    《————再加   
  
0167:004DCBCE  ADD      EBX,BYTE +2D      《————再加   
0167:004DCBD1  JC      004DCBD6   
  
0167:004DCBD6  MOV      \[EAX+B8\],EBX      《————送回   
0167:004DCBDC  PUSHF    
  
0167:004DCBF3  XOR      EBX,EBX          《————清零   
0167:004DCBF5  JC      004DCBFA   
  
0167:004DCBFA  MOV      \[EAX+04\],EBX      《————修改   
0167:004DCBFD  PUSHF    
  
0167:004DCC14          MOV      \[EAX+08\],EBX  《————修改   
0167:004DCC17          JPE      004DCC1C   
  
0167:004DCC1C 89580C          MOV      \[EAX+0C\],EBX    《————修改   
0167:004DCC1F 7203            JC      004DCC24   
  
0167:004DCC24 895810          MOV      \[EAX+10\],EBX    《————修改   
0167:004DCC27 9C              PUSHF    
  
0167:004DCC3E 895814          MOV      \[EAX+14\],EBX    《————修改   
0167:004DCC41 7403            JZ      004DCC46   
  
0167:004DCC46 C7401855010000  MOV      DWORD \[EAX+18\],0155  《————修改   
0167:004DCC4D 7403            JZ      004DCC52   
  
0167:004DCC8A C9              LEAVE    
0167:004DCC8B C21000          RET      10   
  
  
经过三次上面的异常处理后，会来到下面的地方：   
0167:004AFD61  SIDT    \[ESI\]        《————esi==4dc9d8   
0167:004AFD64  JZ      004AFD69   
0167:004AFD66  JNZ      004AFD69   
  
0167:004AFD69  MOV      ESI,\[ESI+02\]  《————\[esi+02\]==800b0000   
0167:004AFD6C  JZ      004AFD71   
0167:004AFD6E  JNZ      004AFD71   
  
0167:004AFD76  MOV      AX,\[ESI+18\]    《————AX＝F5E4   
0167:004AFD7A  JC      004AFD7F   
0167:004AFD7C  JNC      004AFD7F   
  
0167:004AFD7F  MOV      BX,\[ESI+1E\]    《————BX=C18C   
0167:004AFD83  JC      004AFD88   
0167:004AFD85  JNC      004AFD88   
  
0167:004AFD88  MOV      \[EBP+00433C35\],AX  《————F5E4 送入 4dc9de 中   
0167:004AFD8F  JC      004AFD94   
0167:004AFD91  JNC      004AFD94   
  
0167:004AFD94  MOV      \[EBP+00433C37\],BX  《————C18C 送入 4dc9e0 中   
0167:004AFD9B  PUSHF    
0167:004AFD9C  PUSH    BYTE +03   
0167:004AFD9E  JNC      004AFDAB   
  
0167:004AFE18  MOV      \[ESI+18\],AX      《————这时 eax==4dcca5,esi\=800b0000，AX=CCA5   
0167:004AFE1C  PUSHF    
0167:004AFE1D  PUSH    BYTE +03   
0167:004AFE1F  JNC      004AFE2C   
  
0167:004AFE49  SHR      EAX,10          
0167:004AFE4C  JZ      004AFE51   
  
0167:004AFE51  MOV      \[ESI+1E\],AX      《————esi=800b0000，AX＝004D   
0167:004AFE55  JPE      004AFE5A   
  
这中间有一段解码，不断写页面。。。。当来到下面   
0167:004B00BF  INT3      
0167:004B00C0  PUSHF    
0167:004B00C1  JC      004B00CD   
  
这时会通过 int3 进入 Ring0.入口就在 4dcca5 处。进入 Ring0 后，会来到以下地方：   
004DCFB9  XOR  EAX,EAX   
004DCFBB  PUSHF   
  
004DCFE8  MOV DR0,EAX   
004DD018  MOV DR1,EAX   
004DD048  MOV DR2,EAX   
004DD050  MOV DR3,EAX   
004DD06A  MOV EAX,0155   
004DD09C  MOV DR7,EAX   
  
004DD0D2  IRET          《————从这里出来回到 Ring3   
  
出来后，会反复循环在 Ring0 和 Ring3 之间，不断解码写页面，最后好像是造成页面错误，产生异常又进入上面的异常处理例程中 4dca77   
回到正常例程后，还有一些异常处理，跟上面的差不多。最后来到这里：   
  
0167:004D4F72 C3              RET      
看看 stack 中，程序将会到 00748f8f 中，当到这里：   
0167:0074930C 837A08FF        CMP      DWORD \[EDX+08\],BYTE -01  《————edx=816477c8,此时可不能让\[EDX+08\]==-1   
0167:00749310 9C              PUSHF    
0167:00749311 6A03            PUSH    BYTE +03   
0167:00749313 730B            JNC      00749320   
  
0167:0074933D 7563            JNZ      007493A2          《————这里必须跳   
  
之后会来到：   
  
0167:007494A3 803E00          CMP      BYTE \[ESI\],00  《————注意这里   
0167:007494A6 7403            JZ      007494AB   
0167:007494A8 7501            JNZ      007494AB   
在 7494a3 处，d esi,你会看到“\\bw2k.\\\\.\\SUPERBPM.\\\\.\\ICEDUMP.\\\\.\\TRW.\\\\.\\TRWDEBUG.\\\\.\\REGVXD.\\\\.\\NTICE.\\\\.\\SIWVID.\\\\.\\SICE.\\\\.\\FILEVXD”   
在接下来的 749449 处，你千万不能跳的。   
  
在接下去的代码中还有类似第一处的异常处理，很是麻烦的。需要耐心！当你来到 74a56e 处时：   
0167:0074A56E 83F800          CMP      EAX,BYTE +00   
0167:0074A571 7403            JZ      0074A576   
0167:0074A573 7501            JNZ      0074A576   
这里以下的代码就是重定位、破坏 import.   
这些搞完后，还有一些异常处理，需要耐心啊。最后来到下面：   
0167:0074BCC3  JMP      EAX  <----eax\==4887ec   
0167:0074BCC5  JC      0074BCCA   
0167:0074BCC7  JNC      0074BCCA   
0167:0074BCC9  JNZ      0074BD33   
0167:0074BCCB  CDQ      
0167:0074BCCC  ADD      \[EAX\],EAX   
呵呵。。总算到达 oep 了。   
0167:004887EC 55              PUSH    EBP   
0167:004887ED 8BEC            MOV      EBP,ESP   
0167:004887EF 83C4EC          ADD      ESP,BYTE -14   
0167:004887F2 33C0            XOR      EAX,EAX   
0167:004887F4 8945F0          MOV      \[EBP\-10\],EAX   
0167:004887F7 8945EC          MOV      \[EBP\-14\],EAX   
0167:004887FA B8CC854800      MOV      EAX,004885CC   
0167:004887FF E854D2F7FF      CALL    00405A58   
  
啊，总是走完了，人也搞得糊里糊涂的。。。   
谢谢你看完这篇罗里罗嗦地帖子。   
望 d.boy 大侠及各位多多指教！！！  4dd466d0e1e0cc123afbe504da2462ca