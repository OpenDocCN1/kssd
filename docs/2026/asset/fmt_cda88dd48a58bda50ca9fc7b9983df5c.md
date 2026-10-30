
```
0:000> kb

ChildEBP RetAddr  Args to Child              

WARNING: Stack unwind information not available. Following frames may be wrong.

002c3520 6f1c1797 00000000 002c3594 00000000 wwlib!DllGetLCID+0x2cc78c

002c3574 706beb68 0000ffff 0000001a 06b6d1dc wwlib!GetAllocCounters+0x3a9cf

002c35b4 7065d79e 02566b8c 06b6dad8 00000027 mso!Ordinal682+0x54e

002c3604 6dddf281 80000005 06b6dad8 00000027 mso!Ordinal4348+0x1da

002c3670 6dddf25f 00000004 06b6d1e4 06b6d1d8 msxml6!DllRegisterServer+0x29ce4

002c36bc 6dddf25f 00000003 06b6d1e4 06b6d1d8 msxml6!DllRegisterServer+0x29cc2

002c3708 6dddf25f 00000002 06b6d1e4 06b6d1d8 msxml6!DllRegisterServer+0x29cc2

002c3754 6dddf25f 00000001 06b6d1e4 06b6d1d8 msxml6!DllRegisterServer+0x29cc2

002c37a0 6dddf25f 06b6d1e4 06b6d1d8 00000000 msxml6!DllRegisterServer+0x29cc2

002c37ec 6dddd02f 06b6d1d8 00000000 00000000 msxml6!DllRegisterServer+0x29cc2

002c3838 6dddfc6b d732b0ad 002c3894 002c38e4 msxml6!DllRegisterServer+0x27a92
```

在`ida`中跳到`DllGetLCID+0x2cc78c`，通过左下角的`Graph overview`可以看到这个函数还是比较复杂的，先大致浏览一下整个函数，因为崩溃处的代码受`ecx`的影响，而`ecx`是从`esi`的某个偏移处取得的，所以`esi`的值就变得很重要，我们盯着`esi`向上找，就会变得比较容易

![](https://bbs.kanxue.com/upload/attach/201805/682107_QE62JJ9FP334ZWJ.jpg)

一直向上找，发现，`esi`的值是传进来的第一个参数，

![](https://bbs.kanxue.com/upload/attach/201805/682107_JNJ9QPZG43K5MA5.jpg)![](https://bbs.kanxue.com/upload/attach/201805/682107_HGEGRWSS2428PNJ.jpg)

我们在这个函数首部下断，发现会断下很多次，对其参数进行观察，

```
0:000> t

eax=00123494 ebx=03e1fc14 ecx=00000001 edx=00000000 esi=02e40000 edi=0000ffff

eip=312a7650 esp=001233c0 ebp=0012341c iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000206

wwlib!FMain+0x630cf:

312a7650 8b5018          mov     edx,dword ptr [eax+18h] ds:0023:001234ac=03e66526

0:000> du poi(eax+0x18) lpoi(eax+1Ch)

03e66526  "shapedefaults"
```

这是将要处理的标签的名字，我们将其处理的所有标签名打印出来

```
0:000> bp wwlib!FMain+0x630c6 "dd poi(esp + 4) + 224 L1; du poi(poi(esp + 8) + 18) Lpoi(poi(esp + 8) + 1c); g;"

breakpoint 0 redefined

0:000> bl

 0 e 312a7647     0001 (0001)  0:**** wwlib!FMain+0x630c6 "dd poi(esp + 4) + 224 L1; du poi(poi(esp + 8) + 18) Lpoi(poi(esp + 8) + 1c); g;"

0:000> g

03c3c224  00000000

03e66526  "shapedefaults"

03c3c224  00000000

03e66526  "shapelayout"

03c3c224  00000000

03e66540  "idmap"

03c3c224  0008c000

03e2dcb8  "shapetype"

03c3c224  0008c000

03e2dcce  "stroke"

03c3c224  0008c000

03e2dcce  "formulas"

03c3c224  0008c000

03e2dce2  "f"

...共13个...

03e2dce2  "f"

03c3c224  0008c000

03e2dcce  "path"

03c3c224  0008c000

03e2dcce  "lock"

03c3c224  0008c000

03e2dcb8  "shape"

03c3c224  0008c000

03e2dcc6  "imagedata"

 ... 省略38对 ...

03e2dcb8  "shape"

03c3c224  0008c000

03e2dcc6  "imagedata"

(250.64c): Unknown exception - code e0000002 (first chance)

ModLoad: 0eca0000 0ecd9000   C:\Program Files\Common Files\Microsoft Shared\TextConv\mswrd632.wpc

ModLoad: 0b840000 0b86f000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\WPFT532.CNV

ModLoad: 0ecb0000 0ecce000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\msconv97.dll

ModLoad: 0eca0000 0ecdc000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\WPFT632.CNV

ModLoad: 0b850000 0b86e000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\msconv97.dll

ModLoad: 0eca0000 0ecd9000   C:\Program Files\Common Files\Microsoft Shared\TextConv\mswrd632.wpc

ModLoad: 0b840000 0b86f000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\WPFT532.CNV

ModLoad: 0ecb0000 0ecce000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\msconv97.dll

ModLoad: 0eca0000 0ecdc000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\WPFT632.CNV

ModLoad: 0b850000 0b86e000   C:\Program Files\Common Files\Microsoft Shared\TEXTCONV\msconv97.dll

(250.64c): Unknown exception - code e0000002 (first chance)

(250.64c): Unknown exception - code e0000002 (first chance)

03c3c224  00018000

03e6652c  "shapedefaults"

03c3c224  00000000

03e66526  "shapedefaults"

03c3c224  00000000

03e66526  "shapelayout"

03c3c224  00000000

03e66540  "idmap"

03c3c224  00018000

03e2dcba  "OLEObject"

03c3c224  00018002

03e2dcdc  "idmap"

(250.64c): Access violation - code c0000005 (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=088888ec ebx=00000000 ecx=e8bce4f5 edx=00000004 esi=00e245ec edi=0ae461dc

eip=3176aa33 esp=0012335c ebp=001233c8 iopl=0         nv up ei pl nz na po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010202

wwlib!DllGetLCID+0x2ccba9:

3176aa33 ff5104          call    dword ptr [ecx+4]    ds:0023:e8bce4f9=????????
```

可以看到，堆喷已经入场，并且在处理`idmap`的时候发生崩溃，此时`eax`是`088888ec`，但是在样本文件中，这个地址是在`font`字段的，`idmap`在`font`字段外层，这就意味着在处理`idmap`时还是在对`font`的数据进行操作，所以才会看到这个`088888ec`地址。

分析就到这儿吧，关于`exp`的编写这里就不说了，，，[这篇文章](https://bbs.kanxue.com/elink@558K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3%4x3e0t1J5)写的非常详细（我就是跟着后面写的。。。）。

## 3.总结

1)对于有嵌入对象的文档，得用工具（例如`oletools`）将其分解开再根据其内容去分析，很多时候通过样本就能知道很多信息，之后再去用调试器加载分析会相对容易许多；

2)在选用不开启`aslr`的模块时要考虑其兼容性（本例样本中就只能在`xp`中找到，而`win7` `win10`上找不到，不知道是不是我的机器的问题）；如果条件允许，不妨使用多种方法结合结合起来绕过`aslr`；

3)要尝试去写`exp`（虽然我是看别人的文章跟着写的。。。。）

## 4.参考

### [CVE-2017-11826 样本分析](https://bbs.pediy.com/thread-221995.htm)（分析的很精彩）

[CVE-2017-11826漏洞分析、利用及动态检测](https://bbs.kanxue.com/elink@5d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3%4x3e0t1J5)（写的非常详细）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2018-5-14 09:58

被`vlinkstone`编辑

，原因：
