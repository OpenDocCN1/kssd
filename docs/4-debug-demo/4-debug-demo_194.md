

# Periodic Table SE v3.55

> 原文：[`www.pediy.com/kssd/pediy05/pediy50196.htm`](https://www.pediy.com/kssd/pediy05/pediy50196.htm)

Periodic Table SE v3.55 （元素周期表 3.55）破解手记   
  
Cracked by  WksWlj999@sohu.com   
  
软件说明:   
    目前市场上最好的电子元素周期表(是作者主页上讲的@\_#)，功能多多，你想知道的有关某个元素的信息都可以在上面找到。更多说明参见其主页 http://www.smisoftware.com/。   
    本人刚刚涉足加密与解密，水平有限，文中有不足之处还望各位大虾不吝指教！此文仅供初学者参考。   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
目    的：去除 NAG 和使用次数限制   
下载地址：http://www.smisoftware.com/ftplib/ptab32.zip   
破解工具：TRW2000 v1.23 For Win9X；RegMon   
日    期：2002 年 7 月 6 日   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
该程序有 30 天或 60 次试用限制，用 RegMon 检测注册表发现此软件在 HKEY\_CLASS\_ROOT\\PT350KeyInfo\\下设有以下几个主键：   
    SmiBuildLevel    ==>默认值为已使用的次数   
    SmiGoData    ==>默认值为程序安装时的日期数据   
当试用日期及次数全部过了时，你如果在 NAG 上点"Agree",程序会弹出"Order Form"窗口请你订购，点"Cancel"后程序出现个对话框告诉你"你获得了额外的 15 天或 15 次的试用期"；并建立 3 个主键：   
    SmiRunData    ==>默认值为 0    （表示在额外试用期）   
    SmiRunLevel    ==>默认值为 1    （表示在第一次额外试用期）   
    SmiRunType    ==>默认值为可试用的总天数   
当这个期限也过了时，次序又会弹出"Order Form"窗口请你订购，点"Cancel"后程序再次告诉你"你获得了额外的 15 天或 15 次的试用期",此时：        SmiRunData    ==>默认值为 0    （表示在额外试用期）   
    SmiRunLevel    ==>默认值为 2    （表示在第二次额外试用期）   
    SmiRunType    ==>默认值为可试用的总天数   
又过期后程序提醒你"我们已经给了你 2 次额外的试用期"然后，弹出"Order Form"窗口请你汇款订购，然后退出。此时：   
    SmiRunData    ==>默认值为 1    （表示两次额外试用期均过了）   
    SmiRunLevel    ==>默认值为 2       
    SmiRunType    ==>默认值为可试用的总天数   
  
破解思路：跳过 NAG 窗口，并阻止程序改动注册表的 SmiBuildLevel 主键默认键值。   
  
好了，用 TRW2000 加载主程序 ptoe.exe，停在入口处，用 F10 大块跟踪......   
  
017F:0044BD36 50              PUSH    EAX   
017F:0044BD37 FF759C          PUSH    DWORD \[EBP\-64\]   
017F:0044BD3A 56              PUSH    ESI   
017F:0044BD3B 56              PUSH    ESI   
017F:0044BD3C FF15C4834800    CALL    \`KERNEL32!GetModuleHandleA\`   
017F:0044BD42 50              PUSH    EAX   
017F:0044BD43 E8BFF10000      CALL    0045AF07    <==跳过就出现 NAG 窗口，F8 跟进   
017F:0044BD48 8945A0          MOV      \[EBP\-60\],EAX   
017F:0044BD4B 50              PUSH    EAX   
017F:0044BD4C E81A150000      CALL    0044D26B   
017F:0044BD51 8B45EC          MOV      EAX,\[EBP\-14\]   
017F:0044BD54 8B08            MOV      ECX,\[EAX\]   
  
来到这里...   
  
017F:0045AF07 FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF0B FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF0F FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF13 FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF17 E8A2A60000      CALL    004655BE    <==跳过就出现 NAG 窗口，F8 跟进   
017F:0045AF07 FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF0B FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF0F FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF13 FF742410        PUSH    DWORD \[ESP+10\]   
017F:0045AF17 E8A2A60000      CALL    004655BE   
017F:0045AF1C C21000          RET      10   
  
来到这里，用 F10 继续跟...   
  
017F:004655BE 53              PUSH    EBX   
017F:004655BF 56              PUSH    ESI   
017F:004655C0 57              PUSH    EDI   
017F:004655C1 83CBFF          OR      EBX,BYTE -01   
017F:004655C4 E818ECFFFF      CALL    004641E1   
......   
  
017F:00435D79 8B17            MOV      EDX,\[EDI\]   
017F:00435D7B 8BCF            MOV      ECX,EDI   
017F:00435D7D 889C24E0040000  MOV      \[ESP+04E0\],BL   
017F:00435D84 FF92B8000000    CALL    NEAR \[EDX+B8\]    <==跟到这里，跳过就出现 NAG 窗口   
017F:00435D8A 85FF            TEST    EDI,EDI   
017F:00435D8C 8BE8            MOV      EBP,EAX   
017F:00435D8E 7409            JZ      00435D99   
017F:00435D90 8B07            MOV      EAX,\[EDI\]   
017F:00435D92 6A01            PUSH    BYTE +01   
017F:00435D94 8BCF            MOV      ECX,EDI   
017F:00435D96 FF5004          CALL    NEAR \[EAX+04\]   
017F:00435D99 83FD02          CMP      EBP,BYTE +02    <==比较是否按了"Quit"按钮   
017F:00435D9C 754E            JNZ      00435DEC      <==不是"Quit"，跳到运行主程序处；否则不跳，就 over 了   
017F:00435D9E 8D4C241C        LEA      ECX,\[ESP+1C\]   
017F:00435DA2 C68424E004000008 MOV      BYTE \[ESP+04E0\],08   
017F:00435DAA E8ADC30200      CALL    0046215C   
  
因此，把 00435D84 的 Call 去掉，直接跳到 00435DEC 运行主程序处即可跳过 NAG 窗口。   
分析发现，NAG 出现后按"Agree"后程序才会改动 SmiBuildLevel 主键，使其值加 1，于是按"Agree"之前下断点 BPX RegSetValuea,按下"Agree"后 TRW2000 中断，往上看会发现 00413846 处是关键：   
  
017F:0041381B 51              PUSH    ECX   
017F:0041381C 52              PUSH    EDX   
017F:0041381D 6814384A00      PUSH    DWORD 004A3814    <==\[4A3814\]="SmiBuildLevel"   
017F:00413822 50              PUSH    EAX   
017F:00413823 C744241480000000 MOV      DWORD \[ESP+14\],80   
017F:0041382B FF150C804800    CALL    \`ADVAPI32!RegQueryValueA\`    <==取 SmiBuildLevel 的值   
017F:00413831 85C0            TEST    EAX,EAX   
017F:00413833 754B            JNZ      00413880   
017F:00413835 8D4C2408        LEA      ECX,\[ESP+08\]   
017F:00413839 57              PUSH    EDI   
017F:0041383A 51              PUSH    ECX   
017F:0041383B E839630300      CALL    00449B79   
017F:00413840 8D542410        LEA      EDX,\[ESP+10\]   
017F:00413844 6A0A            PUSH    BYTE +0A   
017F:00413846 40              INC      EAX    <==就是这里把使用次数加 1   
017F:00413847 52              PUSH    EDX   
017F:00413848 50              PUSH    EAX   
017F:00413849 C744241880000000 MOV      DWORD \[ESP+18\],80   
017F:00413851 E877A20300      CALL    0044DACD   
017F:00413856 8D7C241C        LEA      EDI,\[ESP+1C\]   
017F:0041385A 83C9FF          OR      ECX,BYTE -01   
017F:0041385D 33C0            XOR      EAX,EAX   
017F:0041385F 83C410          ADD      ESP,BYTE +10   
017F:00413862 F2AE            REPNE SCASB   
017F:00413864 F7D1            NOT      ECX   
017F:00413866 49              DEC      ECX   
017F:00413867 8D44240C        LEA      EAX,\[ESP+0C\]   
017F:0041386B 51              PUSH    ECX   
017F:0041386C 8B4C2408        MOV      ECX,\[ESP+08\]   
017F:00413870 50              PUSH    EAX   
017F:00413871 6A01            PUSH    BYTE +01   
017F:00413873 6814384A00      PUSH    DWORD 004A3814   
017F:00413878 51              PUSH    ECX   
017F:00413879 FF1508804800    CALL    \`ADVAPI32!RegSetValueA\`    <==断在这里，把加过 1 的次数写回注册表   
017F:0041387F 5F              POP      EDI   
017F:00413880 8B542400        MOV      EDX,\[ESP+00\]   
017F:00413884 52              PUSH    EDX   
017F:00413885 FF1504804800    CALL    \`ADVAPI32!RegCloseKey\`   
017F:0041388B 81C488000000    ADD      ESP,88   
017F:00413891 C3              RET   
  
继续 F12 加 F10 若干次后发现回到了 00435D8A 处，可见 NAG 的产生和把加过 1 的次数写回注册表的程序代码都在   
017F:00435D84 FF92B8000000    CALL    NEAR \[EDX+B8\]这个 Call 里，因此只要把这里改为 jmp 00435DEC 直接跳到主程序运行处，即可一举两得，破除 NAG 和次数限制。下命令 bpx 435D84,重新运行程序，中断在 017F:00435D84 处，下命令 A EIP，输入下列代码   
jmp 00435DEC   
nop   
记下更改前后机器码变化，F5 运行，不再有 NAG 出现，查看 SmiBuildLevel 键值，亦保持不变。   
用 16 进制编辑器打开 ptoe.exe 查找"FF92B800000085EF"改为"E9630000009085EF"就 OK 了。  4dd466d0e1e0cc123afbe504da2462ca