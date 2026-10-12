
# 在堆栈中运行代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-817.htm`](https://www.pediy.com/kssd/pediy07/pediy7-817.htm)

参考`VCASM`最初在`http://bbs.pediy.com//showthread.php?threadid=8264`发表的技术，把一条汇编指令变形，在堆栈中运行，比如`xor eax,eax`可以变形为：

```
PUSH    0234365h  
PUSH    07EC2C033h  
XOR     DWORD PTR [ESP+03h], 023436576h  
CALL    ESP
```

在`call esp`后，堆栈中的指令为：

```
33 c0  xor eax,eax  
c2 08 00  ret 8
```

这样变形后的代码分析起来是很困难的。如果我们把重要函数代码都用这种方式变形，就可以防止代码被逆向分析。

在实际使用中，发现这种方法在某些老的赛扬`CPU`上运行不正常。问题出在`call esp`这条指令上。

我们期望`call esp`这条指令是这样运行的：

```
mov tem,esp
push offset L__1
jmp tem         
L__1:
```

但在某些`CPU`上，它却是这样运行的：

```
push L__1
jmp esp
L__1:
```

这样上面的代码运行起来就错了，程序会崩溃。

既然`call esp`不能用，只好用`call eax`了，我这样写：

```
PUSH    EAX  
PUSH    0234365h  
PUSH    066C2C033h  
PUSH    01024448Bh  
XOR     DWORD PTR [ESP+07h], 023436576h  
MOV     EAX, ESP  
CALL    EAX
```

在`call esp`后，堆栈中的指令为：

```
8B 44 24 10         mov         eax,dword ptr [esp+10h]  
33 C0             xor         eax,eax  
C2 10 00          ret         10h
```

这样就解决了某些`CPU`不兼容`call esp`的问题。缺点是代码膨胀过于厉害，原来的一条指令变成了这么长，运行效率也大大降低。

类似地，`push ebx`可以变形为：

```
PUSH    EAX  
PUSH    0234FA7h  
PUSH    066245C89h  
PUSH    01024448Bh  
XOR     DWORD PTR [ESP+07h], 023436576h  
MOV     EAX, ESP  
CALL    EAX
```

我使用这个方法，再加上一些不可逆变换，再加上跳转打乱，作出了一个“代码变形器”，对自己软件的关键函数加密，起到了良好的效果。

作者：`bookaa@rorsoft.com` `2005.12.19`

**这种方式导致运行速度下降。把 CPU 预取的指令都给冲掉了。不过加密来说毕竟不错。**

要求高速运行的模块不能采用这种方式加密,和界面相关的代码或者是运行次数很少的代码可以采用.

`4dd466d0e1e0cc123afbe504da2462ca`
