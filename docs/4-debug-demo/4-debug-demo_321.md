

# 网页特效小精灵 V2.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue106.htm`](https://www.pediy.com/kssd/pediy04/kanxue106.htm)

这个东西从天空软件站拉下来的，因为没处注册，所以只能改代码。   
首先用 FI 查看，是 PECompact 加的壳，自动脱壳成功。（在此感谢柯南兄，因为这个东西没脱壳时用 Exescope 能正常查看，我以为是没加壳的，搞了半天才知道）   
把时间调到过期，出现出错信息。用 W32DASM 查找，无果。   
只能用 TRW 跟踪。   
因为程序有时间限制，所以下断点 bpx getlocaltime，g，嘿嘿拦住了。一看领空，啊,KAV9X，妈呀，怎么把我的金山毒霸给拦住了？把金山关了，再拦，拦住了，找不到切入点。   
因为程序会弹出一个框告诉你过期，所以下断点 messageboxindirect，拦住了。   
015F:00420A4C 83C414          ADD      ESP,BYTE +14   
015F:00420A4F 663BDF          CMP      BX,DI   
015F:00420A52 0F8495000000    JZ      NEAR 00420AED      就是这里了，不跳就过期，所以改成 JMP 00420AED   
015F:00420A58 8B3534114000    MOV      ESI,\[00401134\]   
015F:00420A5E B90A000000      MOV      ECX,0A   
015F:00420A63 B804000280      MOV      EAX,80020004   
015F:00420A68 894D9C          MOV      \[EBP\-64\],ECX   
015F:00420A6B 894DAC          MOV      \[EBP\-54\],ECX   
015F:00420A6E BB08000000      MOV      EBX,08   
015F:00420A73 8D954CFFFFFF    LEA      EDX,\[EBP+FFFFFF4C\]   
015F:00420A79 8D4DBC          LEA      ECX,\[EBP\-44\]   
015F:00420A7C 8945A4          MOV      \[EBP\-5C\],EAX   
015F:00420A7F 8945B4          MOV      \[EBP\-4C\],EAX   
015F:00420A82 C78554FFFFFF68DE+MOV      DWORD \[EBP+FFFFFF54\],0041DE68   
015F:00420A8C 899D4CFFFFFF    MOV      \[EBP+FFFFFF4C\],EBX   
015F:00420A92 FFD6            CALL    ESI   
015F:00420A94 8D955CFFFFFF    LEA      EDX,\[EBP+FFFFFF5C\]   
015F:00420A9A 8D4DCC          LEA      ECX,\[EBP\-34\]   
015F:00420A9D C78564FFFFFF28DE+MOV      DWORD \[EBP+FFFFFF64\],0041DE28   
015F:00420AA7 899D5CFFFFFF    MOV      \[EBP+FFFFFF5C\],EBX   
015F:00420AAD FFD6            CALL    ESI   
015F:00420AAF 8D4D9C          LEA      ECX,\[EBP\-64\]   
015F:00420AB2 8D55AC          LEA      EDX,\[EBP\-54\]   
015F:00420AB5 51              PUSH    ECX   
015F:00420AB6 8D45BC          LEA      EAX,\[EBP\-44\]   
015F:00420AB9 52              PUSH    EDX   
015F:00420ABA 50              PUSH    EAX   
015F:00420ABB 8D4DCC          LEA      ECX,\[EBP\-34\]   
015F:00420ABE 6A10            PUSH    BYTE +10   
015F:00420AC0 51              PUSH    ECX   
015F:00420AC1 FF1560104000    CALL    \`MSVBVM60!rtcMsgBox\`  pmodule 以后，程序停在下一行，说明就是这一行叫出过期提示框的，往上看，看看有什么地方可以跳过这里   
用十六进制编辑器打开脱壳后的程序，把 0F8498000000   
                                改成 E99600000000   
至于为什么要这样改我也不知道，感谢 crack007 和 ccak 的帮助！   
测试：把时间调到 2002 年，也能用了，破解成功。  4dd466d0e1e0cc123afbe504da2462ca