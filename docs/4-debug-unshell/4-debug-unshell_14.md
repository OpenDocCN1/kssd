

# 来个新手学习脱壳的入门篇吧

> 原文：[`www.pediy.com/kssd/pediy06/pediy6395.htm`](https://www.pediy.com/kssd/pediy06/pediy6395.htm)

前  言    
细细回忆,学习 Crack 技术已经快 2 个月了,期间我学会的东西远比我以前任何一年内学的东西都多(专指计算机程序及系统了解情况)    
  
说到学脱壳也是最近一个月的时间,开始总是到处询问学习脱壳的方法,大多的答案就是看雪老大的书,谁也没有正面回答过.于是就自己摸索,略有一点思路,老大们估计是认为理所当然,对于我们小菜来说还是说白了比较合适.    
在这里我就班门弄斧一回,其实主要也是帮助一些朋友能更快的摸到门,不至于对只会照猫画虎,这些也是我当初我想问的那些问题的答案，都是些很基础的东西，说得不好还请高手指正.    
  
在此感谢看雪老大提供了如此之好论坛供大家学习交流，还写了一本很好的书指导像我这样的菜鸟，也感谢网上众多高手贡献出自己的脱壳手记,特别感谢 heXer 老大对本菜鸟的细心指点!    
  
顺便说一句,脱壳特别需要的是兴趣和耐心，如果没有耐心就......;)    
  
废话了一堆,我们下面进入正题:)    
结合娃娃 Wom 的新 KG 说说一般面对一个壳的简单分析方法    
  
一、找 OEP    
脱壳的一般流程是:查壳->寻找 OEP->Dump->修复    
查壳没什么好说的，FI 和 PEiD，不幸的是 FI 和 PEiD 都不能识别出这个壳:)    
  
我找 OEP 的一般思路如下：    
    先看壳是加密壳还是压缩壳，压缩壳相对来说容易些，一般是没有异常，找到对应的 popad 后就能到入口，跳到入口的方式一般为    
1、    jmp OEP    
2、    push OEP    
      ret    
3、    call OEP    
当然也有其他的，如 je OEP 等等，一般都是段之间的大跳转，OD 的反汇编窗口里都是同一个段的内容，所以更好区别是否是段间跳转    
    『这里我说说关于 F8(Step Over)和 F7(Step in)的一般方法，粗跟的时候一般都是常用 F8 走，但是有些 call 是变形的 Jmp，此时就需要 F7 代过，区别是否是变形 Jmp 的一个简单方法是比较 call 的目标地址和当前地址，如果两者离的很近，一般就是变形 Jmp 了，用 F7 走 』    
    对于加密壳，我的方法一般是用 OD 载入，钩掉所有异常（不忽略任何异常，有时由于异常过多可以适当忽略一些异常），运行，数着用了多少次 Shift+F9 程序运行，显然最后一次异常后，程序会从壳跳到 OEP 开始执行，这就是我们寻找 OEP 的一个关键，而对于这个壳可知是加密壳，Shift+F9 16 次后运行，重来，Shift+F1 15 次后到这    
0042CBD3  ⁷³ DC              jnb short kill\_luj.0042CBB1    
0042CBD5    CD 20              int 20    
0042CBD7    64:67:8F06 0000    pop dword ptr fs:\[0\]  <---这里    
一般的处理方法是找到 pop fs:\[0\]处下断点或者是\[esp+4\]处下断点，如果学习一下 SEH 的处理会更好些    
0012FF58  0012FFE0  Pointer to next SEH record    
0012FF5C  0042CBB7  SE handler            <---这里一般包含 Anti-Debug 的代码    
0012FF60  0042CB8A  RETURN to kill\_luj.0042CB8A from kill\_luj.0042CB90    
我首选第一种,在 0042CBD7 处下断之后,耐心的用 F7 走,因为变形 call 太多了...,关键注意一下往回跳的跳转,寻找合适的位置跳出循环,有耐心的话你就能这样走到 OEP 了;)    
当然你也可以用另一个好东西——Trace,在 Command 里来个 tc eip<42b000  （42b000 是当前段的起始位置，滚动条拖到最上面就能看到了,一般程序编译的基地址为 400000），OK，稍等一会我们就会停在这了（如果是 ASPr 可能要几分钟了)    
00419F68    55                push ebp            <----停在这    
00419F69    8BEC              mov ebp, esp    
00419F6B    83C4 F0            add esp, -10    
呵呵，OEP 找到了;)    
当然，这个用 PEiD 就能找到 OEP，比较一下，是不是一样:D，对于用 PEiD 找到的 OEP 可以直接 G 到 OEP    
  
此时就可以 Dump 了，用 LordPE 来 Dump 吧，由于有 Anti－Dump，所以要先 Correct Imagesize 一下再 Dump    
  
二、修复 IAT    
    由于加密壳对输入表进行了重定位，所以现在的文件还不能正常使用，我们的目标就是恢复它，先用 ImportREC，填入正确的 OEP, IAT AutoSearch, Get Import，无效的不少，Show Invalid, 右键->Trace Level1(Disasm)，失败！！看来是只有自己恢复了。记下 IAT 的地址 0001C128    
    好，重新运行，在 OD 的 Dump 窗口里 Go to Expression,到 0041C128，盯着它默默数着按了多少次 Shift+F9，在第 13 次后发现 0041C128 变了，而且和最后到 OEP 处一样，可见刚才壳修复了 IAT 也顺便破坏了 IAT，于是我们重来，Shift+F9 12 次以后按照前面说的方法跳过异常，这里我们可以在 0041C128 处选择用内存断点或硬件断点，如 Memory, On Write, On Access，Hardware，On Access 等，我用 Memory On Write，在壳几次解码 IAT 以后（0041C128 处变了好几次），我们可以到这里    
0042C7C7    60                pushad    
0042C7C8    8327 00            and dword ptr ds:\[edi\], 0        <----停在这    
0042C7CB    8B85 5B314000      mov eax, dword ptr ss:\[ebp+40315B\]    
0042C7D1    40                inc eax    
0042C7D2    0F84 A8000000      je kill\_luj.0042C880             
0042C7D8    80A5 04294000 FF  and byte ptr ss:\[ebp+402904\], 0FF    
0042C7DF    0F84 9B000000      je kill\_luj.0042C880    <----这个地址跳转就不被破坏    
0042C7E5    80A5 05294000 FF  and byte ptr ss:\[ebp+402905\], 0FF    
0042C7EC    0F84 8E000000      je kill\_luj.0042C880                
0042C7F2    8B85 5F314000      mov eax, dword ptr ss:\[ebp+40315F\]    
0042C7F8    8907              mov dword ptr ds:\[edi\], eax    <-----看这里,被破坏了;)    
再看一下 eax 的值，    
    eax 77F525CA ntdll.DeleteCriticalSection    
很不错的样子:D，IAT 的重定向已经找到了，正常修复 IAT 在哪里呢？？多跟几回，就会发现是在这    
0042C880    61                popad        <---前面的 je 就是跳到这    
0042C881    3107              xor dword ptr ds:\[edi\], eax      <---OK，到这就好了    
0042C883    58                pop eax    
于是乎，修改 je 为 jmp，IAT 就不会被破坏了，再到 OEP 处用 ImportREC 修复就好了    
这里我说的似乎很轻松，其实这些东西要靠跟出来的，下面是一些我找到的关键点的分析    
  
首次解码 IAT      0042c3ef    
解码 Dll Name    0042c4ea ->  解码出正确 Dll Name    
Clear dll Name     0042c645 ->  清除 Dll 的名字    
        0042c6ac ->  解码出正确 First Thunk    
        0042c66d ->  解码出正确 First Thunk    
最后解码 IAT    0042c693 ->  解码出正确的 RVA 指向函数名    
ReDirect          0042c7c8 ->  Import 重定向, 这里会置 0  必经路线    
        0042c7f8 ->  Import 重定向, 这里指向壳里    
Normal        0042c881 ->  不重定向时正常输入表地址    
Clear Function Name 0042c88d -> 清除 函数 名字    
Repair IAT Over    0042c8fa -> BreakPoint  到这里 IAT 修复完成    
  
解码 dll 名的那段代码没仔细找;),你如果有兴趣可以找找    
  
当然再有兴趣的朋友可以自己添加代码修复 IAT，这样就可以不用 ImportREC，手动修复 IAT    
  
一般来说壳到这就差不多了，不过也会有某些变态壳要修修补补，这就没法说了也超出了本文的范围了    
  
                后    记    
    我本人对于对于脱壳的思路大致就是如此，当然对于某些特殊壳还需要特殊待遇:D，我希望本文代给朋友的不是娃娃这个 KG 的壳如何脱，而是能更好的理解壳，理解如何分析壳，脱壳。也能更好的理解大侠手记里一些不易理解的地方。例如本壳实际使 telock 的变形壳，如果忽略单步异常，5 次 Shift+F9 运行，第 4 次按 Shift+F9 时就会修复和破坏 IAT，从而能理解为什么要在 5-2=3 次后查找 IAT（这个问题我就迷茫过一阵子）    
  
    最后列一列我未玩过的壳，哪位大侠有相关的经验交流交流    
      DBPE        国产精品，一直未动过，据说常会让机器重启……    
      SVKP        有时间玩玩    
      Armadillo    新的 3.40 CopyMem II 没玩过，据说难度比以前更难了:(    
      Obsidium    分段加密的猛壳，好像还没见过相关的手记    
      ACProtect    感觉和 ASPr 有点象，另一个国产精品，不过现在怎么没有中文的界面了？    
      这么多没玩过......，不写了，没前途    
         
    我的 e-mail:shinegood@163.com，欢迎多多指教交流    
            QQ:31536111  Cracker 为暗号:)  
 4dd466d0e1e0cc123afbe504da2462ca