

# WinRAR 2.71

> 原文：[`www.pediy.com/kssd/pediy03/forum646.htm`](https://www.pediy.com/kssd/pediy03/forum646.htm)

WinRAR 2.71 的初级破解   
  
前两天装上 WinRAR 2.71，为的是解开下载的 Revirgin。发现只能用 40 天，那个 NagDialog 始终在提醒我注册。   
  
此段时间一直在向 hying 兄学习脱壳的技术，看看以前破解 WinRAR 教程似乎不太难，想着来练练吧。   
  
Dasm 反汇编了一把，发现和以前教程里写的不同，晕倒......醒来过后只好自己开刀了。   
  
首先发现\[466CCC\]这个地方很可疑，于是用 trw2000 下 bpm 466CCC 试验了一下，看到只要\[466CCC\]=01，   
启动后 WinRAR 就没有了 evaluation copy 字样。看看 About 框，也有了 Registered to 字样。   
(当然，只要用 Add Files to archive 就会发现 NagDialog 由弹了出来，然后又成了 evaluation copy 了。)   
  
既然如此，就看看何时\[466CCC\]被程序变成了 00。检查到了几处，还好不多。   
  
经过分析，发现 43C0E8 往下的这个 Call 很有问题，它在检查 rarreg.key 文件，看来首先对准这里没错。   
  
检查调用它的三个 Call，以下是其中一个调用：   
\* Reference To: USER32.FlashWindow, Ord:0000h   
|   
:00418373 E81F9B0400              Call 00461E97   
:00418378 6A01                    push 00000001   
:0041837A E8693D0200              call 0043C0E8   
:0041837F A2CC6C4600              mov byte ptr \[00466CCC\], al <------------ al\=00   
:00418384 6A00                    push 00000000   
:00418386 8B0D24B94600            mov ecx, dword ptr \[0046B924\]   
:0041838C 51                      push ecx   
  
那么如何将 al 变成 01 呢？从 43C0E8 往下走可疑看到   
  
:0043C10C BE25814600              mov esi, 00468125   
:0043C111 A5                      movsd   
:0043C112 A5                      movsd   
:0043C113 A4                      movsb   
:0043C114 5F                      pop edi   
:0043C115 8D95BCFBFFFF            lea edx, dword ptr \[ebp+FFFFFBBC\]   
:0043C11B 52                      push edx   
:0043C11C E8AB030000              call 0043C4CC <--------- 检查 rarreg.key 是否存在   
:0043C121 84C0                    test al, al   
:0043C123 7511                    jne 0043C136  <--------- 有的话就跳了   
:0043C125 33C0                    xor eax, eax   
:0043C127 8B55C8                  mov edx, dword ptr \[ebp\-38\]   
:0043C12A 64891500000000          mov dword ptr fs:\[00000000\], edx   
:0043C131 E950030000              jmp 0043C486  <---------- 这里直接就走到函数结束去了。   
  
好吧好吧，既然我从来没有试过 SMC，这次来这里试试看吧。   
(大虾可不要笑话哦，焉用牛刀哉! 俺学习嘛。)   
  
首先在文件的 offset 0xF52 找到了一堆的 00(应该在哪里找？大虾可否指点？)，   
赶紧写个 patchcode 在这里(详细请看 toye 兄主页的 SMC 教程)，在 TRW2000 里找到位置为 401952。   
我的想法是这样的，将 43C131 处先跳向 401952，在这里 mov eax, 01, 然后再跳回 43C486，就可以了。   
  
说干就干，立刻把 43C131 处的代码改为 jmp 00401952，(E91C58FCFF)   
以下就是在 401952 下汇编的代码   
mov eax, 01 (B801000000)  可能改为 mov al,01 就可以了，不过地方宽裕嘛......   
jmp 43C486 (E923AB0300)   
注意记下机械码哦。   
  
当然，只改这里是不行的，看看跳 NagDialog 的地方吧：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011B2(C)   
|   
:00401506 833DC0CC460000          cmp dword ptr \[0046CCC0\], 00000000   
:0040150D 7546                    jne 00401555   
:0040150F 803D7430460000          cmp byte ptr \[00463074\], 00  <----- 这里干脆改为 mov byte ptr \[00463074\], 01   
:00401516 753D                    jne 00401555                  
:00401518 803DCC6C460000          cmp byte ptr \[00466CCC\], 00   
:0040151F 7534                    jne 00401555   
:00401521 A1A8F54600              mov eax, dword ptr \[0046F5A8\]   
:00401526 83F828                  cmp eax, 00000028  <-------- 40 天   
:00401529 7F04                    jg 0040152F <------ 这里显然不能让跳去。   
:0040152B 85C0                    test eax, eax   
:0040152D 7D26                    jge 00401555   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401529(C)   
|   
:0040152F C6057430460001          mov byte ptr \[00463074\], 01   
:00401536 6A00                    push 00000000   
:00401538 68A8BA4000              push 0040BAA8   
:0040153D 8B1524B94600            mov edx, dword ptr \[0046B924\]   
:00401543 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"REMINDER"    <--------- NagDialog。   
                                  |   
:00401544 68E73B4600              push 00463BE7   
:00401549 8B0D20CC4600            mov ecx, dword ptr \[0046CC20\]   
:0040154F 51                      push ecx   
\======================================================================================   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040EE78(C)   
|   
:0040EEBC 803DCC6C460000          cmp byte ptr \[00466CCC\], 00   
:0040EEC3 7410                    je 0040EED5   
:0040EEC5 803D9096470000          cmp byte ptr \[00479690\], 00   
:0040EECC 7507                    jne 0040EED5   
:0040EECE C605CC6C460000          mov byte ptr \[00466CCC\], 00  <------ 怎么能让它放进 00 呢，改成 mov byte ptr \[00466CCC\], 01   
  
不足的是 About 对话框里没有俺的名字。(当然，对 WinRAR 没怎么用，不知道是否改彻底了。)

标题: 要显示名字只要建立一个 rarreg.123(任意扩展名)，里面第二三行名字和公司。这个软件关键是几个功能限制，我还没看到一篇功能完全破解的教程。   
  
内容:  4dd466d0e1e0cc123afbe504da2462ca