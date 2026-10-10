

# RegHance v1.1

> 原文：[`www.pediy.com/kssd/pediy03/forum746.htm`](https://www.pediy.com/kssd/pediy03/forum746.htm)

前言：考试在即。还是忍不住写了出来。写这篇实录花了一下午，这下考试悬了。。。   
  
  
破解者: midi   
级别：初级   
目标: RegHance v1.1   
下载: http://www.lavasoft.de/binary/awbin/regh.exe (632k)   
工具： TRW2000 v1.22, W32Dasm, Hex WorkShop, UnPECompact 1.31, eXeScope,etc   
作者对软件的描述:   
  
The standard registry editor that comes with windows, lacks functionality.   
Reghance was designed to give you more overview and control, making it easy   
to navigate and walk through your registry.   
  
保护：a).开始弹出" About” nag 窗口，须按“close this window" 才能回到主程序；   
b).About 窗口中有"This tool is a shareware...";   
c).退出程序时，弹出"About"的 nag 窗口，须等三、四秒才能关闭退出;   
d).About 窗口的 image 中有"unregistered"字样.   
  
过程实录：   
  
1）。记得 SunBird 大哥常说凡事都要 info 一下。果不出所料，主程序 RegHance.exe 是用 PECompact 加的壳。UnPECompact 1.31 脱之。。。   
  
2）。   
  
对 a)====)   
  
运行脱壳后的主程序，看到那个"About”nag 窗口停在那里，此时启动 TRW。对付这类 nag 窗口， 我懒得去考虑下什么断点，因为可能的断点太多了。我常常采用一种最傻、但却最有效的方法"左右手配合"--就是右手在点击那个“close this window" 钮的同时，左手 CTL+D （我觉得按 CTL+D 比较舒服）激活 TRW。（为此法我练了整整一个月的时间，多亏我有弹六弦琴的功底，哈哈。。）。TRW 做起这活来真是爽极了！！   
  
  
——KERNEL32！CancelWaitableTimer+011F\_\_\_\_\_\_\_\_\_\_   
  
0177:BFF99A75 685C002A00 PUSH DWORD 002A005C   
0177:BFF99A7A E85579FDFF CALL \`KERNEL32!ord\_00000001\`   
0177:BFF99A7F 3DC0000000 CMP EAX,C0   
0177:BFF99A84 8BF0 MOV ESI,EAX<------光棒在这！   
0177:BFF99A86 7505 JNZ BFF99A8D   
0177:BFF99A88 E8FDB7FEFF CALL BFF8528A   
0177:BFF99A8D 8BC6 MOV EAX,ESI   
0177:BFF99A8F 5E POP ES   
  
pmodule 返回主程：   
  
0177:0044D044 E84F66FBFF CALL 00403698   
0177:0044D049 807DFB00 CMP BYTE \[EBP\-05\],00   
0177:0044D04D 7405 JZ 0044D054   
0177:0044D04F E8E0A3FBFF CALL \`USER32!WaitMessage\`   
0177:0044D054 33C0 XOR EAX,EAX<----光棒在这里！   
0177:0044D056 5A POP EDX   
0177:0044D057 59 POP ECX   
0177:0044D058 59 POP ECX   
0177:0044D059 648910 MOV \[FS:EAX\],EDX   
0177:0044D05C 6876D04400 PUSH DWORD 0044D076   
0177:0044D061 8D45F0 LEA EAX,\[EBP\-10\]   
0177:0044D064 BA02000000 MOV EDX,02   
0177:0044D069 E80E6BFBFF CALL 00403B7C   
  
开始一次一次按 F12+F10，目的是到底看看哪个 call 出这个 nag 窗口来。   
需要用笔记下可疑的 call 或判断的地址的，（？具体哪个地址可疑要凭感觉了）并按 F9 设断。退出并重新运行主程，在 nag 窗口弹出以前会被 TRW 拦住去路。试着将判断语句改向或将可疑的 call nop 掉。。。一次次尝试。。终于眼前一亮：   
  
:004948A9 8BC3 mov eax, ebx   
:004948AB E8342CFBFF call 004474E4   
:004948B0 8BC3 mov eax, ebx   
:004948B2 8B10 mov edx, dword ptr \[eax\]   
:004948B4 FF92D8000000 call dword ptr \[edx+000000D8\]===>This will bring up the start nag screen, JUST nop it!   
:004948BA 8BC3 mov eax, ebx   
:004948BC E8CFE5F6FF call 00402E90   
:004948C1 6850494900 push 00494950   
:004948C6 E88953F7FF call 00409C54   
:004948CB 83C4F8 add esp, FFFFFFF8   
:004948CE DD1C24 fstp qword ptr \[esp\]   
:004948D1 9B wait   
  
至此，第一步完成！   
  
  
  
对 b)----)   
  
用 W32Dasm 反汇编脱壳后的主程序，奇怪的是竟然没有 SDR，只有 Data Hex 和 Imp Fn.幸运的是在 Hex Workshop 中能查到"This tool is a shareware". 会有两处，其中一处后面有"Thank you for your licensing reghance!"   
记下它的 offset:9A550,并在 W32Dasm 中找到相应的 offset 的地址：0049B150, Search 它后来到：   
  
  
:0049B0E5 A1F0D64900 mov eax, dword ptr \[0049D6F0\]   
:0049B0EA 83381A cmp dword ptr \[eax\], 0000001A   
:0049B0ED 7512 jne 0049B101===(if license jmp)   
:0049B0EF BA50B14900 mov edx, 0049B150======"This tool is shareware..."   
:0049B0F4 8B83DC020000 mov eax, dword ptr \[ebx+000002DC\]   
:0049B0FA E8A537F9FF call 0042E8A4   
:0049B0FF EB10 jmp 0049B111   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0049B0ED(C)   
|   
:0049B101 BA84B14900 mov edx, 0049B184   
:0049B106 8B83DC020000 mov eax, dword ptr \[ebx+000002DC\]   
:0049B10C E89337F9FF call 0042E8A4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0049B0FF(U)   
|   
:0049B111 80BBF402000000 cmp byte ptr \[ebx+000002F4\], 00   
:0049B118 742A je 0049B144   
:0049B11A BAB8B14900 mov edx, 0049B1B8====WAIT 3 秒   
:0049B11F 8B83D8020000 mov eax, dword ptr \[ebx+000002D8\]   
:0049B125 E87A37F9FF call 0042E8A4   
:0049B12A B201 mov dl, 01   
:0049B12C 8B83E8020000 mov eax, dword ptr \[ebx+000002E8\]   
:0049B132 E8ED56FBFF call 00450824   
:0049B137 33D2 xor edx, edx   
:0049B139 8B83D8020000 mov eax, dword ptr \[ebx+000002D8\]   
  
只须在:0049B0ED 处打补丁就可去掉 About 窗口中的“This tool is shareware..."   
  
对 c)---)   
  
方法和 a)一样。点一下关闭程序，等 delay 窗口稳定下来后，运用"左右手配合"法。最后会来到：   
  
......   
  
:004464EA 8BC0 mov eax, eax   
:004464EC 53 push ebx   
:004464ED 6683B87A02000000 cmp word ptr \[eax+0000027A\], 0000   
:004464F5 7412 je 00446509==if license, jmp and exit windows   
:004464F7 8BCA mov ecx, edx   
:004464F9 8BD8 mov ebx, eax   
:004464FB 8BD0 mov edx, eax   
:004464FD 8B837C020000 mov eax, dword ptr \[ebx+0000027C\]   
:00446503 FF9378020000 call dword ptr \[ebx+00000278\] ==refer to the nag shareware reminder   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004464F5(C)   
|   
:00446509 5B pop ebx   
:0044650A C3 ret   
  
可以看到只须将:004464F5 改改流向或将:00446503 的 call 给 nop 掉，讨厌的退出 nag 就没了！   
  
d)---)   
  
最后一项我就不写了，看一看前面的帖子就行了！（在此，midi 向 SunBird 大哥致敬！--有机会小弟请客！^\_^）   
  
3）\*\*总结\*\*我的补丁是：   
  
:004948B4 FF92D8000000-->909090909090;   
:0049B0ED 7512-->EB12;   
:004464F5 7412-->EB12   
  
  
  
后语：这是我应 peterchen 兄的要求写的第一篇破解实录。各位大虾莫见笑，多多指教！！   
还望大家将破解的 tip 贴出来共同交流。。。   
我的 email: bestwishes66@ h o t m a i l . c o m  4dd466d0e1e0cc123afbe504da2462ca