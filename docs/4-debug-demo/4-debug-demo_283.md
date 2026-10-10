

# Turbo Note+ V4.4

> 原文：[`www.pediy.com/kssd/pediy03/forum996.htm`](https://www.pediy.com/kssd/pediy03/forum996.htm)

解除 Turbo Note+ 时间限制   
课题:    解除 Turbo Note+ V4.4 时间限制   
工具:    TRW 2000   
    TYP   
    Ultra Editer 32   
下载：http://turbopress.com/tnplus44.exe   
  
关键字:    破解 解压缩   
  
TurboNote+是一个方便的"即时帖"软件,可以工作在 Win95/98, WinNT 和 Win2000 环境下.它可以即时在自己或通过 IP 地址向其他人的屏幕上贴上一个可定义尺寸和颜色的"留言条",也可以按预定的时间启动备忘提示功能.   
通过"留言条",可以方便地启动可执行程序、互联网址、E-mail 地址,也可以把它们作为附件发送给指定的 IP 地址.它可以隐藏,也可以再现,甚至关机后信息也不会丢失.使用十分方便.   
  
TurboNote V4.4 是有 30 天的时间限制.过期后会随时出现延时的注册窗口,程序仍能使用.   
  
TurboNote+是一个被压缩了的可执行软件,用一般的方法无法修改.   
  
以下是破解其时间限制的方法:   
  
先在 DOS 环境下键入:C:>MYPATH\\TYP TBNOTE.EXE   
  
屏幕出现对运行环境和文件的分析结果,观察最后一行,TurboNote+的压缩外壳是 SHRINKER V3.3.   
  
解压缩（脱壳）:   
  
启动 TRW2000,点击 OK 使其最小化.右击 TRW 图标,选择 TRnewTCB(截获新线程).   
  
双击 TurboNote+图标,TurboNote+被 TRW 截获.   
  
下指令:BPX GETModuleFileNameA,按 F5,TurboNote+再次被 TRW 截获.   
  
按 F12、F10 各一次,程序回到 TurboNote+域:   
  
  
0167:0049944B        JMP    SHORT 00499450    ;光标停在这里                                
0167:0049944D        MOV    ESI,\[EBP+08\]                          
0167:00499450        CALL    00498000                            
0167:00499455        PUSH     DWORD \[EBP+10\]                              
0167:00499458        PUSH     DWORD \[EBP+0C\]                              
0167:0049945B        PUSH    ESI                                  
0167:0049945C        CALL    00499467    ;追到这里,按 F8 追进去   
  
按 F10 追到 CS:49945C,按 F8 追进去,再按 F10 一直追到:   
  
0167:004994BF        CMP    WORD \[00496E84\],BYTE +00   
0167:004994C6        JZ    004994E7   
0167:004994C8        MOV    EAX,\[00496E84\]   
0167:004994CD        ADD    EAX,\[00496F98\]   
0167:004994D3        MOV    \[EBP\-20\],EAX   
0167:004994D6        PUSH     WORD \[EBP+10\]   
0167:004994D9        PUSH     WORD \[EBP+0C\]   
0167:004994DC        PUSH    WORD \[EBP+08\]   
0167:004994DF        CALL     NEAR \[EBP\-20\] ;追到这里,按 F8 追进去   
0167:004994E2        MOV    \[EBP\-1C\],EAX   
0167:004994E5        JMP     SHORT 004994EE   
  
追到 CS:4994DF,按 F8 追进去:   
  
XXXX:XXXXXXXX    BD    ??   
XXXX:0045B8F3    BD    ??   
XXXX:XXXXXXXX    BD    ??   
.......   
  
当前代码页不可见.但光标停留在地址 CS:0045B8F3,这是解压缩后的程序入口.   
  
下指令:    MAKEPE   
  
TRW 在当前目录下生成被还原了的程序.程序名为 NEWPE.EXE.   
  
按 F5,TRW 退出,回到 WINDOWS 环境下.在屏幕右下脚生成一个 TurboNote+图标.   
  
右击 TurboNote+图标,出现 TurboNote+主菜单,点击"Exit",退出.   
  
在资源管理器中双击 NEWPE.EXE,在屏幕右下脚生成一个 TurboNote+图标.   
  
点击该图标,可运行且功能齐全,解压缩成功.   
  
破解时间限制:   
  
按 Ctrl+N,进入 TRW 环境.   
  
下指令:    BPX GetLocalTime   
  
按 F5,TRW 退出,回到 WINDOWS 环境下.   
  
右击 TurboNote+图标,TurboNote+被 TRW 截获.   
  
按 F12、F10 各一次,程序回到 TurboNote+域:   
  
0167:0040BD3B      CALL    KERNEL32!GetLocalTime   
0167:0040BD41      MOV    CL,\[ESP+06\];          光标停在这里   
0167:0040BD45      MOV    DL,\[ESP+02\]   
0167:0040BD49      MOV    AX,\[ESP+00\]   
0167:0040BD4E      MOV    \[ESP+90\],CL   
0167:0040BD55      MOV    CX,\[ESP+08\]   
0167:0040BD5A      MOV    \[ESP+91\],DL   
0167:0040BD61      MOV    DX,\[ESP+0A\]   
0167:0040BD66      MOV    \[ESP+92\],AX   
0167:0040BD6E      MOV    AX,\[ESP+0C\]   
0167:0040BD73      MOV    \[ESP+94\],CX   
0167:0040BD7B      MOV    CL,\[00477178\]   
0167:0040BD81      MOV    \[ESP+96\],DX   
0167:0040BD89      MOV    DL,\[00477179\]   
0167:0040BD8F      MOV    \[ESP+98\],AX   
0167:0040BD97      MOV    AX,\[0047717A\]   
0167:0040BD9D      MOV    \[ESP+10\],CL   
0167:0040BDA1      MOV    CX,\[0047717C\]   
0167:0040BDA8      MOV    \[ESP+11\],DL   
0167:0040BDAC      MOV    DX,\[0047717E\]   
0167:0040BDB3      MOV    \[ESP+12\],AX   
0167:0040BDB8      MOV    AX,\[00477180\]   
0167:0040BDBE      MOV    \[ESP+14\],CX   
0167:0040BDC3      PUSH    BYTE +01   
0167:0040BDC5      LEA    ECX,\[ESP+14\]   
0167:0040BDC9      MOV    \[ESP+1A\],DX   
0167:0040BDCE      MOV    \[ESP+1C\],AX   
0167:0040BDD3      CALL    0042D040   
0167:0040BDD8      LEA    ECX,\[ESP+10\]  ;TurboNote+安装日期   
0167:0040BDDC      PUSH    ECX   
0167:0040BDDD      LEA    ECX,\[ESP+94\]    ;当前日期   
0167:0040BDE4      CALL    0042D220    ;分析比较   
0167:0040BDE9      TEST    EAX,EAX   
0167:0040BDEB      JNZ    NEAR 40BEDE    ;EAX≠0,成功跳转   
0167:0040BDf1                ;EAX\=0,向下打出延时的注册窗口   
  
向下看 0040BDD8 处的两句,\[ESP+10\]是 TurboNote+安装日期,\[ESP+94\]是当前日期,在下面的子调用中进行比较,   
根据比较结果决定是否调用延时的注册窗口.   
  
若始终用同一个日期,例如 TurboNote+安装日期,进行比较,则比较的结果就一定是真,将不再显示延时的注册窗   
口.   
  
下指令:    code on   
  
抄下 0040BDDD 处的代码:   
  
8D 8C 24 94 00 00 00   
\-- -- -- 10 -- -- --   
  
用 Ultra Editer 32 按上面显示的方法修改 TurboNote+,--意为保留.   
  
存盘。   
  
如果能找到 SHRINKER V3.3(谁有呢?),还可以将修改过的 TurboNote+重新处理成压缩形态.   
  
TurboNote+有注册码,没有去破解,有谁破出来了,请与我联系.  4dd466d0e1e0cc123afbe504da2462ca