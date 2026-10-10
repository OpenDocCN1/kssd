

# 时间限制二

> 原文：[`www.pediy.com/kssd/pediy03/forum389.htm`](https://www.pediy.com/kssd/pediy03/forum389.htm)

注：GetTickCount( )的破解 

X-win Pro 4.3 的 demon 版有 31 分钟的时间限制.   
在运行到第 30 分钟的时候,弹出一个对话框,告诉你只能再用 1 分钟.到第 31 分钟的   
时候,又弹出一个对话框,告诉你已经没时间了,要你重新运行它.它已经销毁了它   
所启动的所有窗口.   
我的破解方法如下:   
用 softice symbol loader 加载 xserver.exe,按 F5 运行.等连上 UNIX 主机后,   
用 bpx messageboxa 设置断点,然后出去玩半个小时(我是去逛书店了).   
等 30 分钟过后,就发现进入了 softice.按 F11 从 MessageBoxA 函数返回.   
再按几下 F10,经过一条 ret 指令后,就可以看见如下的程序段:   
0041228D: CALL  CS:\[KERNEL32!GetTickCount\]       
          MOV  ECX,\[0050D184\]   
          MOV  EBX,EAX   
          TEST  ECX,ECX   
          JZ    004122D3   
004122A0: CMP  EAX,ECX   
          JLE  004122D3   
          MOV  EDX,0000EA60              ;60000 毫秒=1 分钟   
          MOV  EBX,\[0050D17C\]   
          MOV  EAX,\[0050D174\]   
          MOV  \[0050D184\],ESI   
          CALL  \[0050D178\]                  ;显示"只剩 1 分钟了"   
          CALL  CS:\[KERNEL32!GetTickCount\]  ;取时间   
          ADD  EAX,0000EA60                ;加上 1 分钟(60000 毫秒)   
          MOV  \[0050D188\],EAX              ;保存起来   
          JMP  004122F8   
004122D3: CMP  EBX,\[0050D188\]   
          JLE  004122F8   
          MOV  EBX,\[0050D17C\]   
          MOV  EAX,\[0050D174\]   
          XOR  EDX,EDX                    ;0 分钟   
          CALL  \[0050D178\]                  ;显示"剩下 0 分钟"   
          MOV  DWORD PTR \[0050D18C\],00000001   
004122F8: MOV  EAX,ESI   
          POP  ESI   
          POP  EDX   
          POP  ECX   
          POP  EBX   
          RET   
  通过用 softice 改变程序执行流程,对上面的程序段进行分析,知道程序是这么   
作的:   
  程序不断用 GetTickCount( )得到系统时间(系统启动以来所经过的毫秒数),   
和预定的值比较,看有没有到 30 分钟.到了 30 分钟的时候,先显示对话框,告诉你   
只能再用 1 分钟了.然后用 GetTickCount( )得到系统时间,将其加上 60000,   
也就是加上 1 分钟,保存在一个变量中.此后不断将 GetTickCount( )返回的数和这   
个变量比较,如果大于这个变量,就说明过了 31 分钟,就弹出个对话框,告诉你剩下   
0 分钟,然后销毁窗口.   
  如果让 GetTickCount( )返回固定值,不就可以骗过它了吗?由于 GetTickCount   
返回值在 EAX 寄存器中,所以只需要把 CALL  CS:\[KERNEL32!GetTickCount\]这条   
指令改为   
            MOV EAX,00000001   
            NOP   
            NOP   
为了保险起见,我把上面的两条 CALL  CS:\[KERNEL32!GetTickCount\]都改了.   
find:      2E FF 15 4C 71 52 00 8B 0D   
replace:    B8 01 00 00 00 90 90 -- --   
  
find:      2E FF 15 4C 71 52 00 05 60 EA   
replace:    B8 01 00 00 00 90 90 -- -- --   
由于 30 分钟时间很长,我只检验过一次,不知道还有什么 bug.   
当然也可以把 30 分钟改短点儿. 

 4dd466d0e1e0cc123afbe504da2462ca