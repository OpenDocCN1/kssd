
# [原创]网络安全大赛之Key-enc

这是一个信安比赛的题目，有兴趣的可以拿来练练手，题目如下：

![](https://bbs.kanxue.com/upload/attach/201309/497514_v3t4758i657empn.png)

`key.txt`的内容为：`NDE4OUM3MUVBNUZFOTFBM0NCRUI5OTcyMDUzN0VBNEIzMjIxRUIxNDhCOTU3Mzcw=`

好，现在开始逆向该程序

运行后，界面如下：

![](https://bbs.kanxue.com/upload/attach/201309/497514_0kxs1ada3ro016q.png)

这里，原始数据输入`12345678`，然后加密文本

![](https://bbs.kanxue.com/upload/attach/201309/497514_o7wmfz3bb2er138.png)

这个提示很关键，得到的密文在`encrypted.txt`中，密文为：`MDhGMkQ5RDA0RTU2MzY3NkRFRDRCQUFCODREMjU0OTY=`

使用IDA搜索字符串，即可得到调用这个字符串的位置。然后找到这个位置的函数头，在`407AB0`处设断，用OD载入运行。

``` 
.text:00407AB0                 push    ebp

.text:00407AB1                 mov     ebp, esp

.text:00407AB3                 push    0FFFFFFFFh

.text:00407AB5                 push    offset SEH\_407AB0
```

当点击加密文本，就会在断点处停下。说明断点没有问题，但是这个函数里面有大量的函数调用，要是一个个分析还是挺繁琐的。所以当我运行到某个函数调用时我会想用IDA看看子函数的功能，考虑要不要进入。关键是要找个与自己输入数据有关的算法即可。

``` 
00407AE4  |.  59            pop ecx

00407AE5  |.  894D F0       mov [local.4],ecx

00407AE8  |.  6A 01         push 0x1

00407AEA  |.  8B4D F0       mov ecx,[local.4]

00407AED  |.  E8 89CA0800   call Key-enc.0049457B

00407AF2  |.  8B4D F0       mov ecx,[local.4]

00407AF5  |.  83C1 5C       add ecx,0x5C                                        ；输入数据的偏移

00407AF8  |.  E8 63A60D00   call Key-enc.004E2160                    ;  关键函数 判断输入数据长度

00407AFD  |.  85C0          test eax,eax

00407AFF  |.  7F 13         jg XKey-enc.00407B14                                ；若输入数据为0则结束

00407B01  |.  6A 00         push 0x0

00407B03  |.  6A 00         push 0x0

00407B05  |.  68 B8D55C00   push Key-enc.005CD5B8                    ;  加密数据不能为空!

00407B0A  |.  E8 12C80A00   call Key-enc.004B4321

00407B0F  |.  E9 32020000   jmp Key-enc.00407D46

00407B14  |>  8D4D EC       lea ecx,[local.5]                        ;  18f630

00407B17  |.  E8 A097FFFF   call Key-enc.004012BC                    ;  将5cd01c写入18f630

00407B1C  |.  C745 FC 00000>mov [local.1],0x0

00407B23  |.  8D4D DC       lea ecx,[local.9]                        ;  18f620

00407B26  |.  E8 A795FFFF   call Key-enc.004010D2                    ;  将5cd094写入18f620

00407B2B  |.  C645 FC 01    mov byte ptr ss:[ebp-0x4],0x1

00407B2F  |.  8D45 DC       lea eax,[local.9]

00407B32  |.  50            push eax                                 ;  18f620

00407B33  |.  E8 3596FFFF   call Key-enc.0040116D                    ;  获取关键信息到 [18f620+0c]

00407B38  |.  83C4 04       add esp,0x4

00407B3B  |.  8D4D DC       lea ecx,[local.9]

00407B3E  |.  E8 EF94FFFF   call Key-enc.00401032                    ;  关键函数这里会进行一些算法的转换，但与输入数据无关
```

为了节省时间 若后来用的到只需用其结果即可。

``` 
00407B43  |.  8D4D DC       lea ecx,[local.9]

00407B46  |.  E8 E695FFFF   call Key-enc.00401131                    ;  [18f620+0c]处的数据会变

00407B4B  |.  8D4D CC       lea ecx,[local.13]                       ;  18f610

00407B4E  |.  E8 7F95FFFF   call Key-enc.004010D2                    ;  将5cd094写入ecx

00407B53  |.  C645 FC 02    mov byte ptr ss:[ebp-0x4],0x2

00407B57  |.  8D4D CC       lea ecx,[local.13]

00407B5A  |.  51            push ecx

00407B5B  |.  8B4D F0       mov ecx,[local.4]

00407B5E  |.  83C1 5C       add ecx,0x5C                             ;  终于看到输入数据啦嘿嘿

00407B61  |.  E8 FAA50D00   call Key-enc.004E2160

00407B66  |.  50            push eax                                 ;  size

00407B67  |.  8B4D F0       mov ecx,[local.4]

00407B6A  |.  83C1 5C       add ecx,0x5C                             ;  buffer

00407B6D  |.  E8 38A60D00   call Key-enc.004E21AA

00407B72  |.  50            push eax                                 ;  输入数据的buffer

00407B73  |.  8D4D DC       lea ecx,[local.9]

00407B76  |.  E8 5F97FFFF   call Key-enc.004012DA

00407B7B  |.  50            push eax                                 ;  size2

00407B7C  |.  8D4D DC       lea ecx,[local.9]

00407B7F  |.  E8 7096FFFF   call Key-enc.004011F4

00407B84  |.  50            push eax                                 ;  相关数据的buffer

00407B85  |.  E8 7E96FFFF   call Key-enc.00401208                   ;  关键函数 变第一次
```

这个函数应该比较关键 有5个压栈参数这里有输入数据的偏移，大小，关键信息的偏移，大小，和一个地址，在函数返回后即算法结束后得到的结果应该与该地址有关

这里的地址为`18f610`，在`18f610+c`的地方存的就是数据的地址

``` 
0018F610   005CD094  斝\.   Key-enc.005CD094

0018F614   00000008  ...

0018F618   00000801  ..

0018F61C   02127798  榳

02127798  92 A8 01 86 FB BA 80 4B 00 00 00 00 00 00 00 00  挩嘂簚K........
```

这个数据应该就是进行第一次算法变换后得到的数据，然后就密切关注该数据怎么样变化才能得到最终的密文，这是我是先找关键地方的代码，最后再分析具体流程，上面红色字就会第一次关键变换。

接着分析主函数：

``` 
00407B85  |.  E8 7E96FFFF   call Key-enc.00401208                    ;  关键函数 变一次

00407B8A  |.  83C4 14       add esp,0x14

00407B8D  |.  8D4D CC       lea ecx,[local.13]

00407B90  |.  E8 9D94FFFF   call Key-enc.00401032                    ;  关键函数 变第二次 kqgBhvu6gEs=
```

这里的地址用`ecx`传的，还是为`18f610` 在`18f610+c`处存的地址有变化

``` 
0018F610   005CD094  斝\.   Key-enc.005CD094

0018F614   0000000C  ....

0018F618   00000801  ..

0018F61C   02126728  (g  ASCII "kqgBhvu6gEs="

02126728  6B 71 67 42 68 76 75 36 67 45 73 3D 00 00 00 00  kqgBhvu6gEs=....
```

这个数据应该就是进行第二次算法变换后得到的数据，然后就密切关注该数据怎么样变化才能得到最终的密文。

接着分析主函数 这里也有很多函数调用，可以用IDA分析具体功能，一下是我自己分析的一些作用的标注

``` 
00407B95  |.  8D4D C8       lea ecx,[local.14]

00407B98  |.  E8 FC96FFFF   call Key-enc.00401299

00407B9D  |.  C645 FC 03    mov byte ptr ss:[ebp-0x4],0x3

00407BA1  |.  8D4D B8       lea ecx,[local.18]

00407BA4  |.  E8 2995FFFF   call Key-enc.004010D2                    ;  18f5fc 不用跟进

00407BA9  |.  C645 FC 04    mov byte ptr ss:[ebp-0x4],0x4

00407BAD  |.  8D55 B8       lea edx,[local.18]

00407BB0  |.  52            push edx

00407BB1  |.  E8 C096FFFF   call Key-enc.00401276

00407BB6  |.  83C4 04       add esp,0x4

00407BB9  |.  8D4D B8       lea ecx,[local.18]

00407BBC  |.  E8 7095FFFF   call Key-enc.00401131                    ;  [18f5fc+0x0c]的数据与输入无关

00407BC1  |.  8D4D A8       lea ecx,[local.22]

00407BC4  |.  E8 0995FFFF   call Key-enc.004010D2

00407BC9  |.  C645 FC 05    mov byte ptr ss:[ebp-0x4],0x5            ;  18f5ec 不用跟进

00407BCD  |.  8D45 A8       lea eax,[local.22]

00407BD0  |.  50            push eax

00407BD1  |.  8D4D CC       lea ecx,[local.13]

00407BD4  |.  E8 0197FFFF   call Key-enc.004012DA                    ;  取[18f610+0x0c]数据的长度

00407BD9  |.  50            push eax

00407BDA  |.  8D4D CC       lea ecx,[local.13]

00407BDD  |.  E8 1296FFFF   call Key-enc.004011F4                    ;  不用管

00407BE2  |.  50            push eax

00407BE3  |.  8D4D B8       lea ecx,[local.18]

00407BE6  |.  E8 EF96FFFF   call Key-enc.004012DA

00407BEB  |.  50            push eax                                 ;  [18f5fc+0x0c]的数据长度

00407BEC  |.  8D4D B8       lea ecx,[local.18]

00407BEF  |.  E8 0096FFFF   call Key-enc.004011F4

00407BF4  |.  50            push eax

00407BF5  |.  E8 0E96FFFF   call Key-enc.00401208                    ;  关键函数 第三次变
```

这个函数和第一个关键函数一样 有5个压栈参数这里有输入数据的偏移，大小，关键信息的偏移，大小，和一个地址，在函数返回后即算法结束后得到的结果应该与该地址有关

这里的地址为`18f5ec`，在`18f5ec+c`的地方存的就是数据的地址

``` 
0018F5EC   005CD094  斝\.   Key-enc.005CD094

0018F5F0   00000010  ...

0018F5F4   00000801  ..

0018F5F8   02128808  ?

02128808  08 F2 D9 D0 4E 56 36 76 DE D4 BA AB 84 D2 54 96  蛸蠳V6v拊韩勔T?
```

这个数据应该就是进行第三次算法变换后得到的数据，然后就密切关注该数据怎么样变化才能得到最终的密文

接着分析主函数：

``` 
00407BFA  |.  83C4 14       add esp,0x14

00407BFD  |.  8D4D A8       lea ecx,[local.22]

00407C00  |.  E8 0094FFFF   call Key-enc.00401005                    ;  要进去 第四次
```

这里有`ecx`传参数 为`18f5ec`，运行后，

这里的地址为`18f5ec`，在`18f5ec+c`的地方存的就是数据的地址

``` 
0018F5EC   005CD094  斝\.   Key-enc.005CD094

0018F5F0   00000020   ...

0018F5F4   00000800  ...

0018F5F8   02127798  榳   ASCII "08F2D9D04E563676DED4BAAB84D25496"

02127798  30 38 46 32 44 39 44 30 34 45 35 36 33 36 37 36  08F2D9D04E563676

021277A8  44 45 44 34 42 41 41 42 38 34 44 32 35 34 39 36  DED4BAAB84D25496
```

这个数据应该就是进行第四次算法变换后得到的数据 不过这功能很明显，和第三次算法后得到的数据对比 即可知晓。

 然后就密切关注该数据怎么样变化才能得到最终的密文

接着分心主函数：

``` 
00407C05  |.  8D4D A8       lea ecx,[local.22]

00407C08  |.  E8 2594FFFF   call Key-enc.00401032                    ;  关键函数 第五次

00407C0D  |.  8D4D 98       lea ecx,[local.26]                       ;  最终数据已经出来了
```

这里有`ecx`传参数 为`18f5ec`，运行后，
