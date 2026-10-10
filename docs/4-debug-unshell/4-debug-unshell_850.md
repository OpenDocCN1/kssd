

# 看雪学苑

> 原文：[`www.pediy.com/kssd/pediy02/forum185.htm`](https://www.pediy.com/kssd/pediy02/forum185.htm)

 脱壳----对用 pecompact 加壳的程序进行手动脱壳   
前言：偶然发现 pecompact 升级到 1.4b1,拿下来看看有何变化，结果令我很是失望。脱壳方法和以前的版本一样。在这里简单的介绍一下，希望大家对它有个了解。   
目标文件：    pecompact（1.4b1）.exe   
加壳方式：    pecompact   
所用工具：    trw2000 v1.22   
作者：        iis   
  
1.用 trw2000 的 Loader 加载加壳程序。   
2.将停在下面的地方，   
0167:0041E800  JMP      SHORT 0041E808《——停在这里   
0167:0041E802  PUSH    DWORD 0001E000   
0167:0041E807  RET      
0167:0041E808  PUSHF    
0167:0041E809  PUSHA    
0167:0041E80A  CALL    0041E811  《——按 f8 进入   
0167:0041E80F  XOR      EAX,EAX   
  
3.一直按 f10,经过三段连续的空指令（nop)   
0167:00420147  MOV      EDI,ESI   
0167:00420149  ADD      EDI,1000   
0167:0042014F  NOP      《——第一段开始   
        .   
        .   
        .   
0167:00420246  POP      EDI   
0167:00420247  JMP      SHORT 004201D1   
0167:00420249  NOP    《——第二段开始   
0167:0042024A  NOP            
        .   
        .   
        .   
0167:004202AC  NOP      
0167:004202AD  NOP    《——第二段结束   
0167:004202AE  CALL    00420440   
0167:004202B3  NOP    《——第三段开始   
0167:004202B4  NOP              
        .   
        .   
        .   
0167:0041E000  PUSHF  《——这是个重要标志   
0167:0041E001  PUSHA  《——这是个重要标志   
0167:0041E002  CALL    0041E009  《——执行命令 makepe 文件名   
  
好了，脱壳已经成功，我们该做个总结了。下面是我对破壳的一点心得，希望对大家能有帮助，   
        用 pecompact 加壳的程序：   
        用 trw2000 加载后，经过三段连续的空指令（nop)后，发现的 PUSHF,PUSHA 指令后面的调用子函数（  call 指令),就应该是脱壳的入口了。  

 请到我的主页来下载完美破解版，在 NT 和 ME 下都可以正常使用！   
http://iceworld.126.com/   
http://home.etang.com/iceworld   4dd466d0e1e0cc123afbe504da2462ca