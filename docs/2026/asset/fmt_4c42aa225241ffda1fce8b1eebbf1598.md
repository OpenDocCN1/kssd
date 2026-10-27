
```
0040AAD0  /$  8B5424 04     mov edx,dword ptr ss:[esp+0x4]                   ;  数据包首地址edx
0040AAD4  |.  81EC FC050000 sub esp,0x5FC                                    ;  抬高栈顶
0040AADA  |.  85D2          test edx,edx
0040AADC  |.  0F848F000000 je 1_2_9破?0040AB71
0040AAE2  |.  53            push ebx
0040AAE3  |.  56            push esi
0040AAE4  |.  57            push edi
0040AAE5  |.  B9 77010000   mov ecx,0x177
0040AAEA  |.  33C0          xor eax,eax
0040AAEC  |.  8D7C242C     lea edi,dword ptr ss:[esp+0x2C]
0040AAF0  |.  F3:AB         rep stos dword ptr es:[edi]                      ;  清空堆栈
0040AAF2  |.  8B8424 100600>mov eax,dword ptr ss:[esp+0x610]                 ;  数据包总长度放入eax
0040AAF9  |.  8BF2          mov esi,edx
0040AAFB  |.  8BC8          mov ecx,eax
0040AAFD  |.  8D7C242C     lea edi,dword ptr ss:[esp+0x2C]
0040AB01  |.  8BD9          mov ebx,ecx
0040AB03  |.  83C2 02       add edx,0x2                                      ;  加2后，就可以从第3位开始
0040AB06  |.  C1E9 02       shr ecx,0x2                                      ;  除4，ecx用作计数器。
0040AB09  |.  F3:A5         rep movs dword ptr es:[edi],dword ptr ds:[esi]   ;  将内容复制到刚开辟的堆栈空间中去。用的是movs，这也是为什么上一步要将总长度除4、先给edx加2了。
0040AB0B  |.  8BCB          mov ecx,ebx
0040AB0D  |.  83E1 03       and ecx,0x3                                      ;  处理当长度不是2的整数倍的情况
0040AB10  |.  F3:A4         rep movs byte ptr es:[edi],byte ptr ds:[esi]
0040AB12  |.  8B0A          mov ecx,dword ptr ds:[edx]
0040AB14  |.  894C240C     mov dword ptr ss:[esp+0xC],ecx                   ;  1
0040AB18  |.  8B4A04       mov ecx,dword ptr ds:[edx+0x4]
0040AB1B  |.  894C24 10     mov dword ptr ss:[esp+0x10],ecx                  ;  2
0040AB1F  |.  8B4A08       mov ecx,dword ptr ds:[edx+0x8]
0040AB22  |.  894C24 14     mov dword ptr ss:[esp+0x14],ecx                  ;  3
0040AB26  |.  33C9          xor ecx,ecx
0040AB28  |.  8B520C       mov edx,dword ptr ds:[edx+0xC]
0040AB2B  |.  894C24 2E     mov dword ptr ss:[esp+0x2E],ecx                  ;  将保存到堆栈中的3-6位清零
0040AB2F  |.  895424 18     mov dword ptr ss:[esp+0x18],edx                  ;  4，这四步刚好将数据包中3-18位保存到堆栈中
0040AB33  |.  8D54241C     lea edx,dword ptr ss:[esp+0x1C]
0040AB37  |.  52            push edx
0040AB38  |.  894C24 36     mov dword ptr ss:[esp+0x36],ecx                  ;  将保存到堆栈中的7-10位清零
0040AB3C  |.  50            push eax
0040AB3D  |.  8D4424 34     lea eax,dword ptr ss:[esp+0x34]                  ;  堆栈中数据包的地址
0040AB41  |.  894C24 3E     mov dword ptr ss:[esp+0x3E],ecx                  ;  将保存到堆栈中的11-14位清零
0040AB45  |.  50            push eax
0040AB46  |.  894C24 46     mov dword ptr ss:[esp+0x46],ecx                  ;  将保存到堆栈中的15-18位清零
0040AB4A  |.  E8 C1AEFFFF   call 1_2_9破?00405A10
0040AB4F  |.  83C40C       add esp,0xC
0040AB52  |.  B9 04000000   mov ecx,0x4
0040AB57  |.  8D7C240C     lea edi,dword ptr ss:[esp+0xC]
0040AB5B  |.  8D74241C     lea esi,dword ptr ss:[esp+0x1C]
0040AB5F  |.  33D2          xor edx,edx
0040AB61  |.  F3:A7         repe cmps dword ptr es:[edi],dword ptr ds:[esi]
0040AB63  |.  5F            pop edi
0040AB64  |.  5E            pop esi
0040AB65  |.  5B            pop ebx
0040AB66  |.  75 09         jnz short 1_2_9破?0040AB71                        ;  这个跳转作为设置al的，可以作为去除MD5校验位处
0040AB68  |.  B0 01         mov al,0x1
0040AB6A  |.  81C4 FC050000 add esp,0x5FC
0040AB70  |.  C3            retn
0040AB71  |>  32C0          xor al,al
0040AB73  |.  81C4 FC050000 add esp,0x5FC
0040AB79  \.  C3            retn
```

所以，瞬间`0040AB4A`处的`call`，就显得“高大”了起来。其中必有蹊跷，我们跟进去看看。还有一点值得怀疑的是，他为什么要把3-18位清零呢？难道是对清零后的数据包进行MD5，然后将生成的值与原数据包的3-18位进行比对？应该是这样的，那，我们直接去网上找了一段代码，然后对这个清零后的数据包做一次MD5，可得到的结果明显不是我们想要的（时间关系我这里就不演示了，不例证了），这该如何是好呢？也罢也罢，我们还是继续调试吧，或许进行MD5的并不是我们的数据包呢。恩，继续跟进，这里有一个小技巧（当我们看到如下所示有多个指定地址的`call`时，我们可以直接`enter`进去，看看缘由）。

![](https://bbs.kanxue.com/upload/attach/201311/475374_10b7f3a8354a4903.png)

当看到的都是一些系统调用和`retn`时，就可以果断选择过掉这个`call`了，如下图。

![](https://bbs.kanxue.com/upload/attach/201311/475374_b85de628e3109ccf.png)

于是，我们的第一个`call`，直接pass掉。

我们直接F4到第二个`call`上，`enter`看到了如下代码：

```
004064F0  /$  8BD1          mov edx,ecx
004064F2  |.  57            push edi
004064F3  |.  B9 10000000   mov ecx,0x10
004064F8  |.  33C0          xor eax,eax
004064FA  |.  8D7A04       lea edi,dword ptr ds:[edx+0x4]
004064FD  |.  C702 B00D4300 mov dword ptr ds:[edx],1_2_9破?00430DB0
00406503  |.  F3:AB         rep stos dword ptr es:[edi]
00406505  |.  8942 48       mov dword ptr ds:[edx+0x48],eax
00406508  |.  8942 44       mov dword ptr ds:[edx+0x44],eax
0040650B  |.  C7424C01234>mov dword ptr ds:[edx+0x4C],0x67452301
00406512  |.  C742 50 89ABC>mov dword ptr ds:[edx+0x50],0xEFCDAB89
00406519  |.  C742 54 FEDCB>mov dword ptr ds:[edx+0x54],0x98BADCFE
00406520  |.  C742 58 76543>mov dword ptr ds:[edx+0x58],0x10325476
00406527  |.  8BC2          mov eax,edx
00406529  |.  5F            pop edi
0040652A  \.  C3            retn
```

里面并没有`call`，但是呢，出现了`012345678ABCDEFFEDCBA9876543210`。还记得前面百度百科里说的初始链值（幻数）么？就是他了。由于没有其他的内容，我们可以选择不进行跟踪。暂时看看它放到哪里去了就行。本程序中将初始链值放入`00F2F0D`处。

接着看第三个`call`，发现其中还是不同的，有那么两个`call`：

```
……
0040664F  |.  E8 7CF4FFFF   call 1_2_9破?00405AD0                             ; \1_2_9破?00405AD0
00406654  |.  8BFD          mov edi,ebp
00406656  |.  8D753F       lea esi,dword ptr ss:[ebp+0x3F]
00406659  |.  8B6C24 18     mov ebp,dword ptr ss:[esp+0x18]
0040665D  |.  3BF5          cmp esi,ebp
0040665F  |.  731A         jnb short 1_2_9破?0040667B
00406661  |>  8B5424 14     /mov edx,dword ptr ss:[esp+0x14]
00406665  |.  8BCB          |mov ecx,ebx
00406667  |.  8D4432 C1     |lea eax,dword ptr ds:[edx+esi-0x3F]
0040666B  |.  50            |push eax                                        ; /Arg1
0040666C  |.  E8 5FF4FFFF   |call 1_2_9破?00405AD0                            ; \1_2_9破?00405AD0
00406671  |.  83C6 40       |add esi,0x40
00406674  |.  83C7 40       |add edi,0x40
00406677  |.  3BF5          |cmp esi,ebp
00406679  |.^ 72 E6         \jb short 1_2_9破?00406661
```

而且，你会发现，这两个`call`居然是调用一个函数。那么我们`enter`进入`00405AD0`，看看它是干嘛的（用`enter`进入，`Alt+c`返回当前执行处）。

看到有下面摸样的代码：

```
00405B2C  |.  8D8C0178A46A>lea ecx,dword ptr ds:[ecx+eax+0xD76AA478]
```

等等，`D76AA478`？不就是我们前面原理里面的`FF(a,b,c,d,M0,7,0xd76aa478）`中的么？于是，我们可以判定，`00405AD0`处的函数是进行MD5运算的。其中必定包含4大轮16小轮全部算法（数了下类似上面的代码总共64个，也就证实了我们的猜想)。将其生产的128位结果用作下一512位数据分组的初始链值。那，照这么说，在`0040664F`处`call`后面的种种判定，必然是为了看看整个数据包被分成了多少个512位的分组，然后有多少个分组就进行多少次循环咯。最终输出的4组32位链值，结合起来就是最终的MD5值了。

嗯，我们再来就先不进这个`call`了（因为在调试的时候，F7跟入只是为了更好的确定这个`call`的作用，既然我们现在知道了这个`call`的作用了，我们也就不用再去单步调试了）。

我们数据包总长度为`B8`，化作10进制是184字节。也就是1472位。除以512，等于2，余数为448。瞬间恍惚了，还记得原理里面说的么？要进行填充，使其位长度与512求与等于448。这里居然就刚好~~~然后，通过看雪中发的那个帖子，当求余结果刚好为448的时候，需要再增加一个512分组，并且，从448那里开始一直用`100000`填充，到最后八个字节（64bits）再用数据长度填充。

单步后，发现正如我们推测那样，第一个`call`是用来执行对前512位操作的，要是长度小于等于512，则只要进行着一步就行了，后面的循环就是其他对剩余的各512位分组操作了。在这里，我们发现，也就只循环了一次，所以，我们还剩448位没进行处理。

继续单步跟踪下去。看看有没有其他的发现。

返回后来到前面图中的第四个`call`处：

```
00405A60  |.  E8 1B0B0000   call 1_2_9破?00406580                             ; \1_2_9破?00406580
```

还是先`enter`键进去看看：
