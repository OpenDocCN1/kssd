

# ScreenRecorder1.2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue172.htm`](https://www.pediy.com/kssd/pediy04/kanxue172.htm)

ScreenRecorder1.2 注册码破解及注册机制作   
  
  
作者：maomao   
软件：ScreenRecorder1.2   
    ScreenRecorder 是一个屏幕抓录软件，可自定义窗口大小和录制速度，并可同时录制声音，有多种音频属性调整，支持预览，生成 AVI 文件。   
下载：http://www.interkodex.com/screenrecorder12.zip(1.03MB)   
工具：Trw2000 v1.23 注册机编写器 1.32   
日期：2001-10-25   
  
    一直没好意思写破文，主要是因为本人自感水平较菜，解密高手如云。现在看到大家纷纷加入 BCG 和 CCG，我再也坐不住了，赶紧写下第一篇申请加入组织的破文兼注册机制作过程，不足之处，望各位高手多多指点…… :)   
一、注册码破解   
　　首先，用 Language 2000 侦测(纯属条件反射的结果)，居然没有加壳, 还是老外比较友好。   
    现在当然是运行 ScreenRecorder 了,出来一个未注册的对话框，完全在情理之中，不要理它，按下工具条上的 Help 按钮(居然没有菜单)，选择 Enter registration code，会弹出一个注册框，OK，观察一下，有 Name 和 Key 两个再熟悉不过的输入框，看来它是根据用户名生成注册码。二话不说，在 Name 中输入大名 maomao，Key 中填 1111-1111-1111-1111(从头一直按住 1 不放啦，省事多了\*^\_^\* )，先不要急着按“OK”，还没到火候。   
　　下面，当然是祭起神兵利器 TRW2000 了，运行后按 Ctrl+N 激活，下 s 30:0 l ffffffff 'maomao',在内存中寻找输入的用户名，找到地址 80f6fcf2，再下 bpm 30:80f6fcf2，只要注册过程一读取这个值，嘿嘿......F5 退出 TRW，现在按“OK”，一下子被 TRW 拦截，各位，请看……   
0157:9EE6 F36766A5        REP MOVSD   
0157:9EEA 6659            POP      ECX  <=========光标停在这里   
0157:9EEC 6683E103        AND      ECX,BYTE +03   
0157:9EF0 F367A4          REP MOVSB   
0157:9EF3 33D2            XOR      DX,DX   
0157:9EF5 33C0            XOR      AX,AX   
0157:9EF7 EB43            JMP      SHORT 9F3C   
  
  下命令 bd \*，中断所有断点，再下 pmodule，现在我们已经来到 ScreenRecorder 的领空了。   
  一直按着 F10，大约经过 7 个 RET 后，现出注册错误对话框，下面是出错一段的代码：   
0177:0048A4E0  CALL    0048A75C    <=====比较注册码的 call   
0177:0048A4E5  TEST    AL,AL   
0177:0048A4E7  JZ      NEAR 0048A572<=====这里跳到出错点   
0177:0048A4ED  LEA      EAX,\[EBP\-0C\]   
0177:0048A4F0  MOV      ECX,\[ESI+24\]   
0177:0048A4F3  MOV      EDX,0048A5D8   
0177:0048A4F8  CALL    00403E1C   
0177:0048A4FD  MOV      ECX,\[EBP\-0C\]   
0177:0048A500  MOV      DL,01   
0177:0048A502  MOV      EAX,\[00470288\]   
0177:0048A507  CALL    00471510   
0177:0048A50C  MOV      ESI,EAX   
0177:0048A50E  LEA      EDX,\[EBP\-10\]   
0177:0048A511  MOV      EAX,\[EBX+02DC\]   
0177:0048A517  CALL    00488554   
0177:0048A51C  MOV      EAX,\[EBP\-10\]   
0177:0048A51F  PUSH    EAX   
0177:0048A520  MOV      ECX,0048A5E8   
0177:0048A525  MOV      EDX,0048A5F8   
0177:0048A52A  MOV      EAX,ESI   
0177:0048A52C  MOV      EDI,\[EAX\]   
0177:0048A52E  CALL    NEAR \[EDI+04\]   
0177:0048A531  LEA      EDX,\[EBP\-14\]   
0177:0048A534  MOV      EAX,\[EBX+02E0\]   
0177:0048A53A  CALL    0042E070   
0177:0048A53F  MOV      EAX,\[EBP\-14\]   
0177:0048A542  PUSH    EAX   
0177:0048A543  MOV      ECX,0048A5E8   
0177:0048A548  MOV      EDX,0048A604   
0177:0048A54D  MOV      EAX,ESI   
0177:0048A54F  MOV      EDI,\[EAX\]   
0177:0048A551  CALL    NEAR \[EDI+04\]   
0177:0048A554  PUSH    BYTE +00   
0177:0048A556  MOV      CX,\[0048A60C\]   
0177:0048A55D  MOV      DL,03   
0177:0048A55F  MOV      EAX,0048A618   
0177:0048A564  CALL    004517A0   
0177:0048A569  MOV      EAX,ESI   
0177:0048A56B  CALL    00402E88   
0177:0048A570  JMP      SHORT 0048A587   
0177:0048A572  PUSH    BYTE +00<========出错点   
0177:0048A574  MOV      CX,\[0048A634\]   
0177:0048A57B  MOV      DL,01   
0177:0048A57D  MOV      EAX,0048A640   
0177:0048A582  CALL    004517A0 <=======F10 经过这个 call 就出错了   
0177:0048A587  MOV      EAX,EBX   
0177:0048A589  CALL    00402E88   
  用 F9 对 0177:0048A4E0  CALL  0048A75C 设断点，按 F5 回到主程序，重新填写注册内容后按“OK”，光标停在这一行上，用 F8 跟进去，不久就跟到这里：   
0177:0048A7A1  LEA      EDX,\[EBP\-18\]   
0177:0048A7A4  MOV      EAX,\[EBP\-04\]   
0177:0048A7A7  CALL    0040848C   
0177:0048A7AC  MOV      EDX,\[EBP\-18\]<=====取用户名到 EDX   
0177:0048A7AF  LEA      ECX,\[EBP\-14\]   
0177:0048A7B2  MOV      EAX,EBX   
0177:0048A7B4  CALL    0048A890    <=====计算真正注册码   
0177:0048A7B9  MOV      EAX,\[EBP\-14\]   
0177:0048A7BC  LEA      EDX,\[EBP\-10\]   
0177:0048A7BF  CALL    0040848C   
0177:0048A7C4  MOV      EDX,\[EBP\-10\]<=====真正注册码到 EDX，下 D EDX 就可看到我的注册码   
                                          8H0G-C58T-VQAS-TT8U   
0177:0048A7C7  POP      EAX        <=====弹出用户输入的注册码   
0177:0048A7C8  CALL    00403EE0    <=====进行比较，不同则 ZF=0   
0177:0048A7CD  SETZ    AL          <=====设置标志 AL=ZF，1 表示注册成功，0 失败   
0177:0048A7D0  MOV      EBX,EAX    <=====存储标志   
0177:0048A7D2  XOR      EAX,EAX   
0177:0048A7D4  POP      EDX   
0177:0048A7D5  POP      ECX   
0177:0048A7D6  POP      ECX   
0177:0048A7D7  MOV      \[FS:EAX\],EDX   
0177:0048A7DA  PUSH    DWORD 0048A7F4   
0177:0048A7DF  LEA      EAX,\[EBP\-18\]   
0177:0048A7E2  MOV      EDX,06   
0177:0048A7E7  CALL    00403B74   
0177:0048A7EC  RET      
0177:0048A7ED  JMP      004035E8   
0177:0048A7F2  JMP      SHORT 0048A7DF   
0177:0048A7F4  MOV      EAX,EBX    <=====取回标志   
0177:0048A7F6  POP      EBX   
0177:0048A7F7  MOV      ESP,EBP   
0177:0048A7F9  POP      EBP   
0177:0048A7FA  RET   
    虽然可以在“取回标志”时用 mov al,1 来爆破，但不知程序的其它代码段有没有陷阱，所以嘛，还是做个注册机。请看......   
  
二、注册机的制作   
    运行注册机编写器(作者主页 http://www.365hz.net)，按 F8 或选择菜单“其它”->“另类注册机”，在弹出的设置框中，进行如下设置：   
1、按“浏览”按钮，选择 ScreenRecorder.exe；   
2、按下方“添加”铵钮，添加如下数据：   
      中断地址：48A7C7      <=====指定在正确注册码算出后中断(请参阅上边代码)   
      中断次数：2          <=====程序启动时要计算已保存的注册信息，我们忽略它，   
                                  在第二次(重新注册)时中断   
      第一字节：58          <=====POP EAX 的指令代码第一字节   
      指令长度：1          <=====被中断的指令的代码长度(POP EAX 是单字节代码)   
3、在右边注册码选项中选择“内存方式”，并勾选其下方的单选框，选择 EDX(注册码被放在 EDX 所指的内存中)；   
4、设置用户信息(你的相关资料啦)；   
5、按“生成”，输入生成的 EXE 文件名(放到主程序同一目录下)。   
    运行一下你生成的文件，哇，没有出现注册码，而 ScreenRecorde 已经运行了!不要紧张，现在我们来注册一下试试：输入你的大名\*\*\*\*\*，Key 中随便输入一些数字或字母，按“OK”，哈哈，你的注册码已经完整的躺在注册机编写器的对话框中了!还不快使用 copy->Paste 大法？!   
    解密 10 分钟，写作 1.5 小时......现在是凌晨 2:01，为了明天更好的工作，收工~~~~  4dd466d0e1e0cc123afbe504da2462ca