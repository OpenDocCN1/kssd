

# JRun Studio 3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum630.htm`](https://www.pediy.com/kssd/pediy03/forum630.htm)

JRun Studio 3.0 是编辑、调试 JSP 程序的优秀软件，但有 30 天使用限期！   
win98 下很简单，但 Win2000 下我也想到了办法！哈哈。。。   
  
\------------NAG window 和时间限制（脱壳）只能用于 WIN98，在 NT 下见第二部分-----------   
(1).用 W32Dasm  Load jrs3.exe 死机！原来它用 Aspack2.001 加壳，但 Unaspack   
    后不能用。只能手动去壳。   
  
(2)Blast Wave 2000 v0.2 知入口为 0077C2B0.   
(3)用 Trw2000 Load，   
bpx 0077C2B0   
  0077C361  je -----            ;jump 下面出现 Flash 窗口   
  0077C380  je 0077C3D2            ;若在 30 天内，jump 跳 0077C3D2 处 jrs3 正常运行   
  0077C39E  call ---            ;否则弹出注册窗口,退出   
  0077C3D2  -----        ;jrs3 运行   
  
(4)再用 Trw2000 Load，   
bpx 0077C2B0   
pedump d:\\a.exe   
makepe d:\\b.exe      ;无效！   
suspend   
  
(5)用 ProcDump,Dump All >b.exe   
PE Editor 观察 a.exe 入口地址为 0037C2B0   
于是将 b.exe 入口地址改为 0037C2B0   
  
(6)用 HView Search (3)的 Decode 码，将其改为 jne ??? jne ???   
也可以在（4）时修改内存后 Dump All.   
OK!   
  
\----------------------WIN2000 下手动脱壳-----------------------   
  
⑴ 用上述方法 softice 4.05 和 prodump 在 WIN2000 下   
a   
jmp eip   
ProcDump:Dump All >b.exe 后不知如何改 PE,Stop here.   
  
㈡ 只能编写 Patch 程序,用 HView 反汇编原压缩程序 jrstudio3.exe：   
?009664ED: 89856F2E4400    mov        \[ebp\]\[000442E6F\],eax  ;   
?009664F3: 61              popad                    ;   
\* 009664F4: 75??            jne 0009664FE                      ;改为 jne        0009664F6   
?009664F6: mov eax,??                                        ;改为 C60580C3770073  mov b,\[00077C380\],073   
?009664FD: mov esp,??                                        ;改为 90              nop   
?009664FE: 6800000000      push        00   
  00966503: C3              retn   
上述为 ASPACK 在内存中解压缩的最后一段，   
  00966503： retn                              ;到 jrs3.exe 的程序入口 0077C2B0   
注意到 009664F6-009664FE 这一段没有用到（可用 SI 下断点，不会中断），因此可用这个空间写 Patch 程序。   
也可以用 UltraEdit 看 jrs3.exe 程序开头有无可利用的空间，如 ASPACK 版本信息处写 Patch 程序。   
  
㈢ 任务是:解压缩时,把   
    0077C380: 74??? je  0077C3D2          ;若在 30 天内，跳 0077C3D2 处 jrs3 正常运行   
改为 0077C380: 75??? jne 0077C3D2，            ;若不在 30 天内，也跳 0077C3D2 处正常运行   
因而在 HVIEW 改 009664F6:mov byte \[0077C380\],75  ;  (即在内存中把\[0077C380\]处 je 改为 jne)   
            009664FD:nop   
            009664FE:                  ;以下不改动，进入 jrs3.exe 的程序入口   
          ?00966503:   
㈣ 在 WIN2000 下修改时间过期，运行修改过的 jrstudio3.exe,OK！   
出现 FLASH 窗口（...30 days period has expired!)后，可以正常运行了！！！哈哈！   
再把时间改在期限内，运行修改过的 jrstudio3.exe，出现 Flash(You have 25 days left to run...)   
    下面却弹出注册窗口，点 NO 后程序退出！！！我倒！   
仔细想想，原来在期限内   
  0077C380: jne 0077C3D2              ;No jump!在 30 天内，不跳到 0077C3D2 处正常运行   
  
㈤ 曾设法改   
  0077C380: jmp 0077C3D2 ，但 009664F6-009664FE 这一段装不下 jmp 语句！   
想了好多方法都不行，灵机一动，用 SI 看 0077C380:时 FLAGS 标志，Z 标志确实随时间过期与否变，   
但 C 标志不变！都为 0，既只要改为：0077C380: 73???    jnc 0077C3D2   
于是用 HVIEW 改   
?009664F6:  C60580C3770073  mov b,\[00077C380\],073  ;让它把内存 0077C380 处改为 73?? jnc 0077C3D2   
?009664FD:  90              nop   
这下不管时间是否过期，在 win98 和 win2000 下都能用了！:-)   
  
如有人要 Patch 程序,mail to me!   
hurrah@wx88.net   
http://hurrah@go.163.com  4dd466d0e1e0cc123afbe504da2462ca