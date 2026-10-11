
# 关于 Securom 8 保护机制的一些尝试

> 原文：[`www.pediy.com/kssd/pediy12/137566.html`](https://www.pediy.com/kssd/pediy12/137566.html)

仙剑 5 出了之后迅速购入了一款，毕竟是少有的几款喜欢的国产游戏品牌（从仙剑 3 开始接触的～），

通关之后便对它的保护机制有了兴趣，，。

于是首先去网上搜罗了一下，发现加密方式不是 `Securom 7.x` 的，貌似已经升级成了 `8`。

但是从网上的一个关于 `Securom7.x` 的资料中还是发现了一些有用的东西的，，，这是后话了，现在开始。

此文章中涉及到的技术和资料用于研究和学习。

## 尝试 1：

首先我尝试用 `OllyDbg` 载入，然后从开始一步一步的跟着程序走，没多远走到这里

```
02F3EB9C    90               nop
02F3EB9D    895424 08        mov     [dword esp+8], edx
02F3EBA1    0FA4DF 00        shld    edi, ebx, 0
02F3EBA5    8B10             mov     edx, [dword eax]
02F3EBA7    015424 14        add     [dword esp+14], edx
02F3EBAB    90               nop
02F3EBAC    83C0 04          add     eax, 4
02F3EBAF    66:FF4C24 10     dec     [word esp+10]
02F3EBB4    90               nop
02F3EBB5   ^ 75 EA           jnz     short 02F3EBA1                  ; 对程序的一小块计算校验和
02F3EBB7    804C24 14 01     or      [byte esp+14], 1
02F3EBBC    53               push    ebx
02F3EBBD    5B               pop     ebx
02F3EBBE    817424 14 42A90>xor     [dword esp+14], 0CA942
02F3EBC6    817C24 14 03A70>cmp     [dword esp+14], 0CA703          ; 对比前面的计算结果，准确则跳
02F3EBCE    76 03            jbe     short 02F3EBD3
02F3EBD0    EB 32            jmp     short 02F3EC04
```

由此可以看到如果在前面一段设了 `int3` 断电就会出错，应该是很典型的 `Anti-Debug` 了。

然后继续跟

```
02F3EE9C    50               push    eax
02F3EE9D    16               push    ss
02F3EE9E    17               pop     ss                            ; 这里利用 CPU 的机制强制 EFLAG 寄存器入栈，
02F3EE9F    9C               pushfd                                  ; 所以如果用 F7 跟踪的话就露了马脚了，
02F3EEA0    8B0424           mov     eax, [dword esp]
02F3EEA3    F6C4 01          test    ah, 1                         ; 这里就是要比较 T FLAG 了，这里我们将 ah 修改为 2 后继续
02F3EEA6    74 05            je      short 02F3EEAD
02F3EEA8    B9 BE070000      mov     ecx, 7BE
02F3EEAD    33C0             xor     eax, eax
02F3EEAF    74 02            je      short 02F3EEB3
```

然后到这里又是计算校验和，但是附近没有判断，继续跟进

```
02F3EEC1    90               nop
02F3EEC2    8B5C24 20        mov     ebx, [dword esp+20]           ; 计算 2f3ee2ch 开始到 2FBEEF8 的校验和
02F3EEC6    8B1B             mov     ebx, [dword ebx]
02F3EEC8    33F3             xor     esi, ebx                      ; 后面会根据这个结果算出程序的返回地址
02F3EECA    834424 20 04     add     [dword esp+20], 4
02F3EECF    56               push    esi
02F3EED0    5E               pop     esi
02F3EED1    66:FF4C24 1C     dec     [word esp+1C]
02F3EED6    90               nop
02F3EED7   ^ 75 E8           jnz     short 02F3EEC1
02F3EED9    90               nop
02F3EEDA    8B5C24 14        mov     ebx, [dword esp+14]
02F3EEDE    90               nop
02F3EEDF    896424 14        mov     [dword esp+14], esp
02F3EEE3    834424 14 24     add     [dword esp+14], 24
02F3EEE8    2BEE             sub     ebp, esi
02F3EEEA    8B7424 18        mov     esi, [dword esp+18]
02F3EEEE    8D1B             lea     ebx, [dword ebx]
02F3EEF0    8B6424 14        mov     esp, [dword esp+14]
02F3EEF4    872C24           xchg    [dword esp], ebp              ; 在此处 将算出的地址入栈
02F3EEF7    90               nop
02F3EEF8    C74424 04 C2040>mov     [dword esp+4], D50004C2
02F3EF00    C1EB 00          shr     ebx, 0    ；这里 OllyDbg 被混淆了，应为“retn 4”
02F3EF03   ^ EB F7           jmp     short 02F3EEFC     
```

此处没有判断，但是后面会根据这个结果得出程序的返回地址，因此可以在靠下一些的位置设硬件断点，，，

## 尝试 2：

![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")这么一步一步分析太慢了，并且一旦放跑就会结束进程，后来从资料中发现，`Securom7` 的保护模式中不是只有一个进程的，而是会另外生成一个进程，这个才是最重要的进程，前面的那个进程主要是计算出新创建进程需要的参数的，那么 `8` 是不是也是这样呢？，（相关信息可以看 `wzanthony` 的 `http://bbs.pediy.com/showthread.php?t=93932`），其中涉及到很重要的一些突破方式甚至涉及到了 `Hook` 系统 `DLL`，![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")，我只是刚入逆向的新手啊，，这从何下手，。。= =，

继续 `Google`，发现了国外的一些成果，关键字“`Special_Issue_for_SecuRom7.3`”，有时间的话我再翻译它![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")。

在那个资料里面提到了一个工具 `OEP Finder` 可以完成这个任务！，如文档里所说勾上 `Kill Low Alloc,Hook Virtual Alloc,Get TickCount +1` 即可，然后在参数用在原先 `OllyDbg` 载入的 `Pal5` 设 `CreateProcesinternalA` 建立的断点得到，填入之后即可，但是有一点不足，无法自动找到 `OEP`，看来有了一定的应对措施，![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")我也懒得找了，直接用 `OllyDbg` `Attach` 进去看看，这里发现一个规律，`Attach` 进去之后要尽快 `F9` 恢复程序的运行，这样才能保证不会死掉，

由于我只是个新手，因此在 `Attach` 之后尚无明确的方法找到 `OEP` 并且有效脱壳，比较有趣的是发现貌似 `Pal5` 内含有很多调试信息，甚至一些参数信息，

![](img/74db49b470316289cd57dd2da08e6768.jpg)

![](img/6f72a6a966937f734a2e68b86f2430c7.jpg)

以上就是到目前为止的一些半成品了，如果有什么好的方法的话希望能够多多交流～，![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")
