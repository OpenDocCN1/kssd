

# Aston 1.8

> 原文：[`www.pediy.com/kssd/pediy06/pediy6764.htm`](https://www.pediy.com/kssd/pediy06/pediy6764.htm)

  
aston1.8 为例此程序是 asprotec1.23 加壳。  
工具 trw2000＋supbmp  
  
运行 Thwizard.exe 后，设断 bpx exitprocess do "dd esp"拦下来后   
  
  
堆栈值为 0040D40A，ok！bpm 0040d40a (其实你 u 这个地址的会发现上面的 call 就是调 exitprocess）。  
  
从新运行 thwizard.exe，又停下来。d esp 后向下翻一屏后看到：  
  
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
016F:006AFDCC 00012600 00C1538C 00BF51D6 00C02207 .&..孲?諵?."?   
016F:006AFDDC 00BF0000 00BD0000 00C03374 00400000 ..?..?t3?..@.   
016F:006AFDEC 00000000 00000000 01003499 00401000 ........?....@.   
016F:006AFDFC 006AFE10 00C02708 00C02A08 00C03450 .㳠..'?.\*?P4?   
016F:006AFE0C 00BFA9B8 0043142B 00000000 004317DD 俯?+.C.....?C.   
016F:006AFE1C 00000000 8194150C 006AFF78 006AFE3C ......攣 x�j.<㳠.   
016F:006AFE2C 005A0000 8194156C 8194152C 00401000 ..Z.l.攣,.攣..@.   
016F:006AFE3C BFF8B86C 00000000 8194150C 005A0000 l 给?.....攣..Z.   
016F:006AFE4C 69776854 6472617A 45584500 00000000 Thwizard.EXE....   
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
注意 zthwizard.exe 上面 006afe39 处的 00401000，是不是很有想法！！！ok！我们 bpm 006afe39 这个时候清除  
掉以前的断点后按一次 f5 就来到了这里。（此法在对付 1.22 壳时是 bpm 006afdf8 )  
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  
0167:00C150EE E800000000       CALL     00C150F3  
0167:00C150F3 5D               POP      EBP  
0167:00C150F4 81EDBD6F4900     SUB      EBP,00496FBD  
0167:00C150FA 8D8DD16F4900     LEA      ECX,\[EBP+00496FD1\]  
0167:00C15100 03CB             ADD      ECX,EBX  
0167:00C15102 894101           MOV      \[ECX+01\],EAX<-注意通过这里后 00c15112 的变化。  
0167:00C15105 EB00             JMP      SHORT 00C15107  
0167:00C15107 55               PUSH     EBP            \\  
0167:00C15108 8BEC             MOV      EBP,ESP         \\入口处被抽掉的字节。  
0167:00C1510A 83C4F4           ADD      ESP,BYTE -0C    /  
0167:00C1510D B81C334100       MOV      EAX,0041331C   /  
0167:00C15112 6890909090       PUSH     DWORD 90909090  
0167:00C15117 C3               RET       
?br> 0167:00C150EE E800000000       CALL     00C150F3  
0167:00C150F3 5D               POP      EBP  
0167:00C150F4 81EDBD6F4900     SUB      EBP,00496FBD  
0167:00C150FA 8D8DD16F4900     LEA      ECX,\[EBP+00496FD1\]  
0167:00C15100 03CB             ADD      ECX,EBX  
0167:00C15102 894101           MOV      \[ECX+01\],EAX  
0167:00C15105 EB00             JMP      SHORT 00C15107  
0167:00C15107 55               PUSH     EBP  
0167:00C15108 8BEC             MOV      EBP,ESP  
0167:00C1510A 83C4F4           ADD      ESP,BYTE -0C  
0167:00C1510D B81C334100       MOV      EAX,0041331C  
0167:00C15112 68FF334100       PUSH     DWORD 004133FF<-入口地址别忘记在其地址上加上被抽掉的代码。  
0167:00C15117 C3               RET       
    
至于输入表的修复，看雪第二版写的很详细了。唯一不同的是输入表间被非 0 的字节添充了，改之就可以用 ImportREC OK 了！  
此方法公布后我猜不会有俄国间谍把情报汇报回国吧???  

 4dd466d0e1e0cc123afbe504da2462ca