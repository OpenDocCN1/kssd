

# 瓦解 PELock 1.0x -> Bartosz Wojcik 秀

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-627.htm`](https://www.pediy.com/kssd/pediy08/pediy8-627.htm)

\[**主题**\]:瓦解 PELock 1.0x -> Bartosz Wojcik 秀  
\[**作者**\]:PeaceClub  
\[**工具**\]:Ollydbg,OllyScript v9.2,OllyDump,LoadPE,Imprec  
\[**文件**\]:演示文件为 PELock 1.06 保护的 WinRoot 的 Dll\_LoadEx  
\[**说明**\]:PELock 的壳代码混淆和保护能力还是挺优秀的,我向来对复杂的壳不愿意花时间去弄,但今天是个特殊的纪念日子,所以……,Action!  
**第一步:进入解压段**  
为了配合脚本及过程,请您忽略所有异常,除了 C000001D (ILLEGAL INSTRUCTION)。  
\[动作\]:ollydbg 载入,停留在 40605C  

引用:    
0040605C >  F9              STC  
0040605D    72 01           JB SHORT Dll\_Load.00406060  
0040605F    D6              SALC  
00406060    F9              STC  
00406061    72 01           JB SHORT Dll\_Load.00406064  
00406063    5A              POP EDX  
00406064    E8 02000000     CALL Dll\_Load.0040606B  
00406069    5A              POP EDX  
0040606A    2D 5880F10C     SUB EAX,0CF18058  
0040606F    EB 02           JMP SHORT Dll\_Load.00406073  
00406071    CD 20           INT 20  

  
\[动作\]:F9 运行  
出现异常提示,点击"确定"  
\[动作\]:Alt+M,查看内存,在 DLL\_Load 模块的上两行 00380000 段下访问断点,Shift+F9  
断点生效后停在:  

引用:    
00382E54   /EB 02           JMP SHORT 00382E58  
00382E56   |0F5EEB          DIVPS XMM5,XMM3  
00382E59    0361 79         ADD ESP,DWORD PTR DS:\[ECX+79\]  
00382E5C    4B              DEC EBX  
00382E5D    C1F2 00         SAL EDX,0                                ; Shift constant out of range 1..31  
00382E60    EB 03           JMP SHORT 00382E65  
00382E62    17              POP SS                                   ; Modification of segment register  
00382E63    BE 24EB0263     MOV ESI,6302EB24  
00382E68    D5 EB           AAD 0EB  
00382E6A    020F            ADD CL,BYTE PTR DS:\[EDI\]  
00382E6C    D0EB            SHR BL,1  

  
  
**第二步:IAT 脚本修补**:  
调用 ollyScript 来运行下面的脚本,根据表的长度大小,需要点时间,当寄存器和代码不变化的时候,F12 暂停执行  
此例中代码显示窗停在  
0038602D    F3:AB           REP STOS DWORD PTR ES:\[EDI\]  
  

引用:    
/\*  
PELock 1.0x -> Bartosz Wojcik IAT Fix script v0.1  
Author: peaceclub  
Email:   5261314@sohu.com  
OS:      Winxp Sp2  
Date:   2006-7-29  
Action: Auto fix IAT,With Remote jmp Patch code  
\*/  
var iat1  
var nextstop  
dbh  
  
l1:  
find eip,#8919#  
bprm $RESULT,2  
mov iat1,$RESULT  
  
find eip,#C602E9#  
mov nextstop,$RESULT  
mov \[nextstop\],#ebfe#  
eob I2  
esto  
  
I2:  
cmp eip,iat1  
jne I3  
mov \[ecx\],eax  
add eip,2  
eob I2  
esto  
  
I3:  
eob I2  
esto  
  
I4:  
msg "IAT Fix OK.Please Ctrl+G,then input EDI-1 to Patch Code Area!"  
cob  
dbs  
ret  

  
**第三步:RemoteJump 代码混淆修补**:  
脚本停止后,提示:  
IAT Fix OK.Please Ctrl+G,then input EDI-1 to Patch Code Area!  
CPU 窗口显示:  

引用:    
003880EB  - EB FE           JMP SHORT 003880EB  
003880ED  - E9 8BC72BC2     JMP C264487D  
003880F2    83E8 05         SUB EAX,5  
003880F5    8942 01         MOV DWORD PTR DS:\[EDX+1\],EAX  

  
\[动作\]:CPU 窗口中 Ctrl+G,输入 Edi-1  
不同机器申请空间不一样,我机器上是来到 970000,粘贴以下二进制代码:  

引用:    
EB 05 8B 16 83 C6 04 8B FA 0F B6 06 46 80 3E 8D 74 15 50 8B C8 83 E0 03 C1 E9 02 F3 A5 8B C8 F3  
A4 5A 46 90 90 EB 47 50 33 D2 33 C9 B1 06 F7 F1 8B C8 0F B6 46 01 8A E0 68 25 2D 35 3D 68 05 0D  
15 1D B0 B8 33 D2 38 24 14 74 07 90 90 FE C0 42 EB F4 5A 5A 25 FF 00 00 00 50 33 C0 8B 56 02 03  
C2 83 C6 06 E2 F6 5A 88 17 89 47 01 46 5A 4B 75 91 5F 8D 4D 66 2B CF F3 AA 61 C3 

  
在 970000 新建 EIP(光标移到 970000,然后 Ctrl+\*)  
\[动作\]:F4 到修补代码最后的 retn 上  
  
**第四步:StolenCode 修补**:  
  
在 401000 上下访问断点,F9 运行,我们停留在:  

引用:    
00401D8D   ?  FF15 C8314000 CALL DWORD PTR DS:\[4031C8\]               ;  msvcrt.\_\_set\_app\_type  
00401D93   .  59            POP ECX  
00401D94   .  830D 70424000>OR DWORD PTR DS:\[404270\],FFFFFFFF  
00401D9B   .  830D 74424000>OR DWORD PTR DS:\[404274\],FFFFFFFF  
00401DA2   .  FF15 C4314000 CALL DWORD PTR DS:\[4031C4\]               ;  msvcrt.\_\_p\_\_fmode  
00401DA8   .  8B0D 64424000 MOV ECX,DWORD PTR DS:\[404264\]  
00401DAE   .  8908          MOV DWORD PTR DS:\[EAX\],ECX  

  
看堆栈区域和 cpu 窗口代码,我们依照 VC 程序入口点进行修补,oep 为 401d60:  

引用:    
00401D60  /.  55            PUSH EBP  
00401D61  |.  8BEC          MOV EBP,ESP  
00401D63  |.  6A 00         PUSH -1         \*\*注意:虽然堆栈区是 0,但 401d86 处清 0 的,回溯 oep 代码的时候会发现,此处 push 的值是-1\*\*\*  
00401D65  |.  68 B0354000   PUSH pel.004035B0  
00401D6A  |.  68 E61E4000   PUSH pel.00401EE6                        ;  JMP to msvcrt.\_except\_handler3; SE handler installation  
00401D6F  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
00401D75  |.  50            PUSH EAX  
00401D76  |.  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP  
00401D7D  |.  83EC 68       SUB ESP,68  
00401D80  |.  53            PUSH EBX  
00401D81  |.  56            PUSH ESI  
00401D82  |.  57            PUSH EDI  
00401D83  |.  8965 E8       MOV DWORD PTR SS:\[EBP-18\],ESP  
00401D86  |.  33DB          XOR EBX,EBX  
00401D88  |.  895D FC       MOV DWORD PTR SS:\[EBP-4\],EBX  
00401D8B  |.  6A 02         PUSH 2  

  
  
**第五步:Dump & Fix IAT**:  
使用 OllyDump 进行脱壳,指定 oep 为 1d60  
\[提示\]:如果你是追求完美的人,请跳到第六部  
Imprec 中请选中"Use PE head From Disk(从磁盘获得 PE 头部)"选项,oep:1d60,自动获得输入表,Fix Dump 即可  
  
**第六步:瘦身减肥,天天好心情**:  
经过观察,最后的 section 段是壳的代码,我们删除它,并删除从 6000 到文件结尾的数据,接着用第五步的 imprec 修复 iat.  
用工具 Rebuild 一下,最后大小 13405 字节,运行功能测试完全正常。  
  
凑够六步,祝各位**六六大顺**! 

这么修复很慢，  
克隆一份代码 patch CRC 那里，然后可以直接修改 patch IAT。  
  
并且试试 delphi 程序，有个表被抽走了。  
  
给你个参考代码，写得很乱：  
  
          ; clone decoder to fuck crc  
          pusha  
          mov  ebx, s\_size\[ebp\]  
          push  4  
          push  3000h  
          push  ebx  
          push  0  
          callX  VirtualAlloc  
          mov  s\_clone\[ebp\], eax  
          xchg  edi, eax  
          mov  esi, s\_start\[ebp\]  
          xchg  ecx, ebx  
          rep  movsb  
          popa  
  
          ; find CRC patch  
          ; 00374351    2B848D 143B0000 SUB     EAX, \[EBP+ECX\*4+3B14\]  
          ; 2B 84 8D 14 3B 00 00  
          .while  b \[eax\] != 2Bh || \\  
            d \[eax+1\] != 3B148D84h || \\  
            w \[eax+5\] != 0  
            inc  eax  
          .endw  
          ; save the addr to decide is last exception  
          mov  iatproc\[ebp\], eax  
          ; patch CRC  
          add  eax, 3  
          mov  ecx, s\_clone\[ebp\]  
          sub  ecx, s\_start\[ebp\]  
          add  ecx, \[eax\]  
          mov  \[eax\], ecx  
          ; patch IAT encrypt  
          ; 00374661    8919            MOV     \[ECX\], EBX  
          ; 00374663    EB 03           JMP     SHORT 00374668  
          ; 8919EB03  
          .while  d \[eax\] != 03EB1989h  
            inc  eax  
          .endw  
          inc  eax  
          mov  b \[eax\], 01h  ; mov \[ecx\], eax  
          ; wait alloc mem to fuck Codereplace  
          mov  eax, za\[ebp\]  
          call  \_bp