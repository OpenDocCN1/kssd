

# ASProtect v0.9x 保护

> 原文：[`www.pediy.com/kssd/tutorial/chap8-5-5-3.htm`](https://www.pediy.com/kssd/tutorial/chap8-5-5-3.htm)

**ASProtect v0.9x 保护**  

 Advanced Zip Password Recovery 3.0 的脱壳   
  
教程写作: 冰毒   
作者信箱: break\_ice@hotmail.com   
写作日期: 2000 年 3 月 25 日   
版权声明: 本文没有版权,允许任意转贴和修改. 但如果只引用文中部分内容时,请最好注明原文出处,以表示对一位 Cracker 同行的劳动的尊重.   
  
使用工具:   
TRW2000 1.03   
ProcDump 1.6.2   
Hexworkshop 3.02 

下载：Advanced Zip Password Recovery 3.0  
  
比起其 Beta 版来,AZPR 3.0 正式版的保护更为加强. 1. 对 Softice 的多处 Check,用 FrogsICE 不能完全骗过; 2. CRC 的校验; 3. 动态地址(好象是这个名吧?); 4.对 Loader 的防范,这回用 Process Patch 不行了.   
  
用 Softice 跟踪它会是一种痛苦 (当然完全可以用 Softice,只是你要了解程序的 Anti-debugger 技巧,在关键的 Check 后改变跳转方向). 这次祭出我们中国人的骄傲 TRW2000 来对付它.   
  
先用 Procdump 的 PE Editor 查看一下程序的.idata section 记下数值. Virtual size=2000, Virtual offset=21000. 另外,记下程序的 image Base: 400000.   
  
1\. 运行 TRW, Load 程序. 程序中断在入口处. 但接下来无论是否 BPX 设断只要 G 继续运行程序,便会出错. 下 Faults off, G, OK 程序退出.   
2\. 试 BPM 421000 (.idata 的 offset), 再 Load 程序. G, 程序可以被正常中断, COOL!   
3\. G,当程序第三次(? 记不清了. 原则是 d 421000 显示的 data 区见到 XXXX0200 000000 字样)中断时,BD\*,F10 直到下面的语句   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
注意: 程序每次 Load 时 Offset 都不一样,你的机器中的 XXXX:YYYYYYYY 肯定和下面的不同. 下 Code on 指令,这样才好参照下面的代码. 另外,下面所摘的几段代码是引用高手 tiamath 的.因为我不清楚在 TRW 下如何 dump 屏幕 (谁能指教一下:);而用 ICEDump 的 pagein n 命令来 Dump Softice 的屏幕,我的机器会死机. :(   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
0167:004F34CD 50         PUSH EAX   
0167:004F34CE B890274F00     MOV EAX,004F2790   
0167:004F34D3 50         PUSH EAX   
0167:004F34D4 B8A4274F00     MOV EAX,004F27A4   
0167:004F34D9 50         PUSH EAX   
0167:004F34DA B8A0284F00     MOV EAX,004F28A0   
0167:004F34DF 50         PUSH EAX   
0167:004F34E0 B8AC274F00     MOV EAX,004F27AC   
0167:004F34E5 50         PUSH EAX   
0167:004F34E6 B8646C4E00     MOV EAX,004E6C64   
0167:004F34EB 50         PUSH EAX   
0167:004F34EC 8B4508         MOV EAX,\[EBP+08\]   
0167:004F34EF 8D4824         LEA ECX,\[EAX+24\]   
0167:004F34F2 8B4508         MOV EAX,\[EBP+08\]   
0167:004F34F5 8B500C         MOV EDX,\[EAX+0C\]   
0167:004F34F8 8B4508         MOV EAX,\[EBP+08\]   
0167:004F34FB 8B4008         MOV EAX,\[EAX+08\]   
0167:004F34FE E899F4FFFF     CALL 004F299C   
0167:004F3503 33C0         XOR EAX,EAX <--在这里 Dump .idata   
  
下指令 W 421000 L 2000 azprdata.bin   
  
OK. 再往下,直到   
  
0167:004F3637 8D4818         LEA ECX,\[EAX+18\]   
0167:004F363A 8B4508         MOV EAX,\[EBP+08\]   
0167:004F363D 8B10         MOV EDX,\[EAX\]   
0167:004F363F 8B4508         MOV EAX,\[EBP+08\]   
0167:004F3642 8B401C         MOV EAX,\[EAX+1C\]   
0167:004F3645 E8EAF6FFFF     CALL 004F2D34 <--F8 进入   
0167:004F364A 5F         POP EDI   
0167:004F364B 5E         POP ESI   
  
到了   
  
0167:004F2D72 E9148B1DA8     JMP A86CB88B   
0167:004F2D77 8E4F00         MOV CS,\[EDI+00\]   
0167:004F2D7A EB01         JMP 004F2D7D   
0167:004F2D7C EB89         JMP 004F2D07   
0167:004F2D7E 041C         ADD AL,1C   
0167:004F2D80 EB02         JMP 004F2D84   
0167:004F2D82 EBE8         JMP 004F2D6C   
0167:004F2D84 61         POPAD   
0167:004F2D85 EB01         JMP 004F2D88   
0167:004F2D87 E850EB02E9     CALL E95218DC   
0167:004F2D8C 17         POP SS   
0167:004F2D8D E802000000     CALL 004F2D94   
0167:004F2D92 E91758C35E     JMP 5F1285AE   
0167:004F2D97 5B         POP EBX   
0167:004F2D98 59         POP ECX   
  
此时小心地跟踪,碰到 JMP 时按 F8 而不要按 F10   
  
直到显示变成   
  
0167:004F2D72 E9148B1DA8     JMP A86CB88B   
0167:004F2D77 8E4F00         MOV CS,\[EDI+00\]   
0167:004F2D7A EB01         JMP 004F2D7D   
0167:004F2D7C EB89         JMP 004F2D07   
0167:004F2D7E 041C         ADD AL,1C   
0167:004F2D80 EB02         JMP 004F2D84   
0167:004F2D82 EBE8         JMP 004F2D6C   
0167:004F2D84 61         POPAD   
0167:004F2D85 EB01         JMP 004F2D88   
0167:004F2D87 E850EB02E9     CALL E95218DC   
0167:004F2D8C 17         POP SS   
0167:004F2D8D E802000000     CALL 004F2D94   
0167:004F2D92 E91758C35E     JMP 5F1285AE   
0167:004F2D94 58         POP EAX <-- 光标位于此行时, EAX=401000   
0167:004F2D95 C3         RET <--这里   
  
下 Suspend 指令. 回到 Windows. 用 ProcDump 来 Dump(full)脱壳的程序,得到 azprdump.exe. 你也可以用 TRW 的 PEDUMP 命令来得到脱壳程序,但我个人的经验很容易死机,所以我宁愿用 ProcDump 来做.   
  
4\. 用 PE Editor 修改程序的 Entry Point 为 1000. 并查看脱壳后程序的.idata section. 此时 Raw size=1670, Raw offset=1FC00. 修改 Directory 中 Import Table 的 RVA=21000,SIZE=1670.   
5\. 用 Hexworkshop 打开 azprdump.exe 和 azprdata.bin. Goto 到 exe 文件的 1FC00 偏移处,Select Block 大小为 1670. 拷贝.bin 文件的同样大小(1670)的 Block,粘贴到 exe 文件中以替换掉不正确的.idata section.   
  
现在,再试着运行程序,应该可以正常运行了. 如果程序出错,再做一件事: 把 .bss section 的 raw size 值改为 00000000 (高手 tiamath 的建议).   
  
有了脱壳的程序,大家应该会 Patch 它成为注册版了吧. 只需改一个字节.   
  
结语: 用本文所描述的方法,可以对绝大多数 Asprotect+Aspack 保护的程序进行成功的脱壳. 大家读完这篇教程,不妨找几个程序开刀. 比方说,The Bat! 1.39 现在不应该再难住大家了.   
  
好了,这次就到这儿了,下回见.   
  
致谢:   
  
感谢 SV, Hobgoblin, tiamath. 没有他们的帮助,我不可能做到这一步. 

 4dd466d0e1e0cc123afbe504da2462ca