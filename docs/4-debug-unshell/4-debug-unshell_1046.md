

# te!lock 加壳

> 原文：[`www.pediy.com/kssd/pediy05/pediy50077.htm`](https://www.pediy.com/kssd/pediy05/pediy50077.htm)

用 ollydbg 跟踪 te!lock 加壳的软件   
  
作者：zczc   
写文章只是为交流技术，转载请保持完整。   
  
打开 ollydbg,将 options 菜单下的 debugging options 中的 Exceptions 的 int3 break   
和 single break 勾选上，好了，开始吧。   
  
这次以 XXXXXX 为目标，加载 XXXXX ,按 F9  嘭.....   
中断在第一个异常处：   
005570A7  F7F3              DIV EBX  <---   
005570A9  64:67:8F06 0000  POP DWORD PTR FS:\[0\]   
005570AF  83C4 04          ADD ESP,4   
005570B2  66:BE 4746        MOV SI,4647   
005570B6  66:BF 4D4A        MOV DI,4A4D   
按 shift+F9 忽略异常，继续执行  又断下:   
005576A8  8DC0              LEA EAX,EAX ;Illegal use of register <---    
005576AA  74 03            JE SHORT JG.005576AF   
005576AC  CD 20            INT 20   
005576AE  64:67:8F06 0000  POP DWORD PTR FS:\[0\]   
005576B4  EB 02            JMP SHORT JG.005576B8   
还有：   
00557B94  66:B8 0043        MOV AX,4300   
00557B98  EB 02            JMP SHORT JG.00557B9C   
00557B9A  CD 20            INT 20   
00557B9C  81B5 291C0000 B86>XOR DWORD PTR SS:\[EBP+1C29\],FB969B8   
00557BA6  CD 68            INT 68   
都要按 shift+F9 忽略异常，继续执行   
005587B1  8B6424 08        MOV ESP,DWORD PTR SS:\[ESP+8\]   
005587B5  EB 0D            JMP SHORT JG.005587C4   
005587B7  33C9              XOR ECX,ECX   
005587B9  64:FF31          PUSH DWORD PTR FS:\[ECX\]   
005587BC  64:8921          MOV DWORD PTR FS:\[ECX\],ESP   
005587BF  F1                ???    Unknown command ; 按 shift+F9 通过      
005587C0  F7F1              DIV ECX <--异常   
看一下右下角的 esp 数据窗，在代码窗点右键，go to -->Expression ,把 esp+4 的内容填入，F2 设个断点，F9 执行。   
005587C2  EB E8            JMP SHORT JG.005587AC   
  
来到：   
0055880B  9C                PUSHFD   
0055880C  810C24 00010000  OR DWORD PTR SS:\[ESP\],100   
00558813  9D                POPFD   
00558814  F8                CLC  <---异常,c 处理方法同上   
00558815  73 DC            JNB SHORT JG.005587F3   
.   
.   
.   
.   
005588EE  E8 09000000      CALL JG.005588FC   
005588F3  48                DEC EAX   
005588F4  E9 0B000000      JMP JG.00558904   
005588F9  98                CWDE   
005588FA  2BC1              SUB EAX,ECX   
005588FC  1D 8A66E96A      SBB EAX,6AE9668A   
00558901  C3                RETN   
00558902  8BC1              MOV EAX,ECX   
00558904  51                PUSH ECX   
00558905  8BCF              MOV ECX,EDI   
00558907  E3 03            JECXZ SHORT JG.0055890C   
00558909  59                POP ECX   
0055890A  EB 90            JMP SHORT JG.0055889C   
0055890C  59                POP ECX  <--在这设个断点，跳出循环。   
0055890D  EB 02            JMP SHORT JG.00558911   
go go go ..   
  
00558778  66:AB            STOS WORD PTR ES:\[EDI\]   
0055877A  EB 02            JMP SHORT JG.0055877E   
0055877C  CD 20            INT 20   
0055877E  61                POPAD   
0055877F  FF6424 D0        JMP DWORD PTR SS:\[ESP\-30\] <--到站。  4dd466d0e1e0cc123afbe504da2462ca