

# EZIP1.0 脱壳手记

> 原文：[`www.pediy.com/kssd/pediy04/kanxue243.htm`](https://www.pediy.com/kssd/pediy04/kanxue243.htm)

EZIP1.0 脱壳手记   
                                  Wrote by 娃娃/\[CCG\]   
  
  
    好久没有写有关于脱壳的文章了 主要是最近一直在研究 Keygen 的编写和密码学  而且也没有什么感兴趣的壳可写   
小弟功力微薄  太难的写不出来 比如 DBPE1.5B4 或者 ASProtect1.3  太简单的又没有  ^\_^ 今天到 http://www.programmerstools.org/   
溜达了一圈  乱七八糟的东西弄下来一大堆  其中就有一个这个东西 叫做 EZIP 是个动态压缩软件 使用 Generic Unpacker Win32 脱壳居   
然非法操作  还以为会有点乐趣值得研究一下 没有想到使用 TRW 脱壳居然出奇的简单  ：（  功夫不能白费 写出来给一些初学者参考一   
下吧  不对的地方请多多执教！   
  
脱壳的目标就是 EZIP 程序本身。   
  
用 TRW 载入程序 按两下 F10 后来到外壳部分代码段：   
  
017F:004582DC  INT3   
017F:004582DD  MOV  EBP,ESP  /TRW 的光条停在这里   
017F:004582DF  SUB  EBP,0428   
017F:004582E5  PUSH  EBX   
017F:004582E6  PUSH  ESI    
017F:004582E7  PUSH  EDI   
017F:004582E8  LEA  EAX,\[EBP+FFFFFC94\]   
017F:004582EE  PUSH  EAX   
017F:004582EF  CALL  004550F0   
017F:004582F4  POP  ECX   
017F:004582F5  TEST  EAX,EAX   
017F:004582F7  JNZ  0045868A   
  
这个东西的壳相当简单 比我所见过的任何壳都简单许多 一直按 F10 单步跟踪就到达了这里：   
  
017F:00458539  PUSH  EAX   
017F:0045853A  PUSH  DWORD \[EBP+FFFFFDEC\]   
017F:00458540  LEA  EAX,\[EBP+FFFFFC3C\]   
017F:00458546  PUSH  EAX   
017F:00458547  LEA  EAX,\[EBP+FFFFFC94\]   
017F:0045854D  PUSH  EAX   
017F:0045854E  CALL  00455005   
017F:00458553  ADD  ESP,BYTE +10   
017F:00458556  AND  DWORD\[EBP+FFFFFC34\]   
017F:0045855D  JMP  SHORT 0045856C  /跳到脱壳代码段   
017F:0045855F  MOV  EAX,\[EBP+FFFFFC34\]  /标志位－EAX   
017F:00458565  INC  EAX  /循环一次后减一   
017F:00458566  MOV  \[EBP+FFFFFC34\],EAX  /压入地址 EBP+FFFFFC34 中   
017F:0045856C  MOV  EAX,\[EBP+FFFFFBE4\]  /从这里开始程序开始进行循环 并在内存中脱壳   
017F:00458572  MOVZX  EAX,WORD \[EAX+06\] /取值   
017F:00458576  CMP  \[EBP+FFFFFC34\],EAX  /对比循环标志   
017F:0045857C  JNC  NEAR 0045861F  /跳出到 45861F 的话则循环结束   
017F:00458582  MOV  EAX,\[EBP+FFFFFDEC\]   
017F:00458588  MOV  EAX,\[EAX+0C\]   
017F:0045858B  ADD  EAX,\[EBP+FFFFFBE0\]   
  
若没有在 0045857C 处跳出，你就会按 F10 跟到这里：   
  
017F:004585F5  LEA  EAX,\[EBP+FFFFFC3C\]   
017F:004585FB  PUSH  EAX   
017F:004585FC  LEA  EAX,\[EBP+FFFFFC94\]   
017F:00458602  PUSH  EAX   
017F:00458603  CALL  00455050   
017F:00458608  ADD  ESP,BYTE +18   
017F:0045860B  MOV  EAX,\[EBP+FFFFFDEC\]   
017F:00458611  ADD  EAX,BYTE +28   
017F:00458614  MOV  \[EBP+FFFFFDEC\],EAX   
017F:0045861A  JMP  0045855F  /跳到上面继续进行循环脱壳   
017F:0045861F  PUSH  DWORD \[EBP+FFFFFDE8\]   
  
因此我们知道了这个壳的第一个关键处 就是 45857C  只要这个时候下 BPX XXXXXXX （45857C 中 JNC 语句后面的地址）  然后按   
一下 F5 程序就会自动停到循环后的第一个语句处  这时按 F10 继续跟踪会很快来到这里：   
  
017F:00458662  POP  ECX   
017F:00458663  POP  ECX   
017F:00458664  MOV  EAX,\[EBP+FFFFFC2C\]   
017F:0045866A  MOV  EAX,\[EAX+10\]   
017F:0045866D  MOV  ECX,\[EBP+FFFFFC2C\]   
017F:00458673  ADD  EAX,\[ECX+1C\]   
017F:00458676  MOV  \[EBP+FFFFFC38\],EAX   
017F:0045867C  MOV  EAX,\[EBP+FFFFFC38\]   
017F:00458682  POP  EDI   
017F:00458683  POP  ESI   
017F:00458684  POP  EBX   
017F:00458685  MOV  ESP,EBP   
017F:00458687  POP  EBP   
017F:00458688  JMP  EAX  /这个就是跳到程序的 OEP 的 JMP   
017F:0045868A  POP  EDI   
  
从这个 JMP 跳出来 哦～～～  整个世界清净了 重要找到了 OEP  ^\_^ 赶快下命令 MAKEPE ————  脱壳成功  程序没有对   
输入表加密 脱壳后可以夸平台运行了  呵呵   
脱壳后的程序 370 多 K  看来这个东西的压缩比还是很高的阿    
  
  
  
仅以此文献给我们可爱的组织 CCG，希望它能蒸蒸日上。   
  
  
  
                                                  娃娃(NYDoll)   
                                              CHiNA CrACKiNG GrOUp 

\[NYDoll's EZIP1.0\]   
L1=OBJR   
L2=LOOK EB,0D   
L3=BP   
L4=WALK   
L5=OBJR   
L6=LOOK FF,E0   
L7=BP   
L8=STEP   
OPTL1=00000000   
OPTL2=01010001   
OPTL3=01010001   
OPTL4=00030000   
OPTL5=00000000   
  
完美脱壳 ^\_^ 测试了 3 个不同的程序 ～～～～   
  
  
娃娃(NYDoll)   
CHiNA CrACKiNG GrOUp  4dd466d0e1e0cc123afbe504da2462ca