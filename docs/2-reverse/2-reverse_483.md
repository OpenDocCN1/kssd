
# 利用 call 传递静态参数

> 原文：[`www.pediy.com/kssd/pediy10/95985.html`](https://www.pediy.com/kssd/pediy10/95985.html)

今天逆向一个木马的时候有点小心得，希望能对我这样的菜们有所帮助哈～

出现了一段代码，貌似很乱无意义：

```
00402E28    E8 09000000      CALL t.00402E36

00402E2D    61              POPAD

00402E2E    64:76 61        JBE SHORT t.00402E92

00402E31    70 69           JO SHORT t.00402E9C

00402E33    3332            XOR ESI,DWORD PTR DS:[EDX]

00402E35    00FF            ADD BH,BH

00402E37    95              XCHG EAX,EBP

00402E38    21F1            AND ECX,ESI

00402E3A    FFFF            ???

00402E3C    8985 57FBFFFF   MOV DWORD PTR SS:[EBP-4A9],EAX
```

执行的时候变成：

…………（看不到，向上翻就变回去了。以为是花指令，没管它继续向下执行）

```
00402E36    FF95 21F1FFFF   CALL DWORD PTR SS:[EBP-EDF] ;kernel32.LoadLibraryA

00402E3C    8985 57FBFFFF   MOV DWORD PTR SS:[EBP-4A9],EAX
```

而且居然还能顺利运行，并返回结果`advapi32.77DA0000`

觉得非常奇怪，为什么没有`push`参数却能执行呢？

重新执行一遍，观察堆栈。发现`call LoadLibraryA`之前`ESP`为：

```
0012EFB4   00402E2D  返回到 t.00402E2D 来自 t.00402E36
```

这是没错的，因为前面的`call`会将其下一句压栈。难道返回地址也能做参数？

……

突然灵光一闪（不是一下子就想出来的，惭愧啊惭愧）

中间那段根本不是代码也不是花指令，而是字符串！

数据与代码的混合使反汇编出现无意义代码。`00`其实是字符串的结尾。`LoadLibrary`的参数是指向字符串的指针，而那个返回值就指向字符串的首地址。所有的都解释得通了。

在`IDA`中修正后的代码（`OD`中要怎么做？请指导一下……）：

```
DATA:00402E28             call    loc_402E36

DATA:00402E28 ; ---------------------------------------------------------------------------

DATA:00402E2D aAdvapi32       db 'advapi32',0

DATA:00402E36 ; ---------------------------------------------------------------------------

DATA:00402E36

DATA:00402E36 loc_402E36:                ; CODE XREF: DATA:00402E28 p

DATA:00402E36             call    dword ptr [ebp-0EDFh]

DATA:00402E3C             mov     [ebp-4A9h], eax
```

这是一种利用`call`传递参数的方法。终于可以上网搜了（一开始根本无从下手，不知道要搜什么……）。原来在`shellcode`编写中是常用技巧。

虽然可能是很基本的东西，但在不知道的情况下，能自己分析解决，并学到一些东西，还是小有成就感的。

呼呼

```
push 40h

call @f

db 'charme',0

@@:

push 0

push 0

call MessageBoxA
```

不就是个这么，，呼呼，，，
