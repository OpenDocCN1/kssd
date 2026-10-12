
# 简论程序是如何动态修改内存或指令的

> 原文：[`www.pediy.com/kssd/pediy12/100106.html`](https://www.pediy.com/kssd/pediy12/100106.html)

自打学破解以来，经常听人说指令修改内存、动态解压指令什么的，看得我是一头雾水。在研究一个壳是如何隐藏`OEP`的时候，无意间有了一些发现。现在总结一下，希望能为和我一样的菜鸟们提供些帮助，高手莫要见笑哈![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

(老大啥时候能把我升级为会员啊，还是临时会员呢，哎....)

请看这段跳到`OEP`之前的一段代码

代码:

```
0040D39A    B8 CC100000    mov eax,10CC
0040D39F    50             push eax
0040D3A0    0385 22040000  add eax,dword ptr ss:\[ebp+422\]
0040D3A6    59             pop ecx
0040D3A7    0BC9          or ecx,ecx
0040D3A9    8985 A8030000 mov dword ptr ss:\[ebp+3A8\],eax
0040D3AF    61             popad
0040D3B0    75 08         jnz short NotePad\_.0040D3BA
0040D3B2    B8 01000000    mov eax,1
0040D3B7    C2 0C00        retn 0C
0040D3BA    68 00000000    push 0
0040D3BF    C3             retn
```

在执行过`0040D3A9    8985 A8030000  mov dword ptr ss:\[ebp+3A8\],eax`这条指令后，该段代码变为：

代码:

```
0040D39A    B8 CC100000    mov eax,10CC
0040D39F    50             push eax
0040D3A0    0385 22040000  add eax,dword ptr ss:\[ebp+422\]
0040D3A6    59             pop ecx
0040D3A7    0BC9          or ecx,ecx
0040D3A9    8985 A8030000 mov dword ptr ss:\[ebp+3A8\],eax
0040D3AF    61             popad
0040D3B0    75 08         jnz short NotePad\_.0040D3BA
0040D3B2    B8 01000000    mov eax,1
0040D3B7    C2 0C00        retn 0C
0040D3BA    68 CC104000    push NotePad\_.004010CC
0040D3BF    C3             retn
```

我们看到`0040D3BA`这句指令发生了变化，由`PUSH 0`变为`push NotePad\_.004010CC`,为什么呢？这当然是`mov dword ptr ss:\[ebp+3A8\],eax`这条指令的缘故了。请看分析：

代码:

```
0040D39A    B8 CC100000    MOV EAX,10CC        ；10CC 赋给 EAX
0040D39F    50             PUSH EAX            ;EAX 入栈，即 10CC
0040D3A0    0385 22040000   ADD EAX,DWORD PTR SS:\[EBP+422\] 
；将【EBP+422】内存地址的内容赋值给 EAX,此时 EBP=0040D013,因此【EBP+422】=0040D435,Crtl+G 跟随到该地址，在数据窗口中跟随，在数据窗口
查看该地址的内存值（16 进制显示），此时该值为 00   00   40   00，（反序）即 00400000，该值与 EAX(10CC)相加为 004010CC 即将 004010CC 赋给 EAX 
0040D3A6    59             POP ECX             ；10CC 出栈赋给 ECX
0040D3A7    0BC9          OR ECX,ECX         ；或，ECX 不变
0040D3A9    8985 A8030000   MOV DWORD PTR SS: \[EBP+3A8\],EAX  
；将 EAX 赋给内存地址\[EBP+3A8\]（经同上计算，该值为 0040D3BB）,即将 004010CC 赋给内存地址 0040D3BB,可以看到该地址即是 PUSH 0 这句指令。
由此可知这条指令其实是把 0040D3BA 处的这句指令修改为 PUSH  004010CC，由此实现了动态修改指令，隐藏入口点的目的。
0040D3AF    61             POPAD
0040D3B0    75 08         JNZ SHORT 0040D3BA
0040D3B2    B8 01000000    MOV EAX,1
0040D3B7    C2 0C00        RETN 0C
0040D3BA    68 00000000    PUSH 0   
；执行完 0040D3A9 处的指令后该指令变为 PUSH  004010CC
0040D3BF    C3             RETN  ；返回到入口点
```

下面是内存地址`0040D3BB`处，在执行`0040D3BA   mov dword ptr ss:\[ebp+3A8\],eax`这条指令前后发生的变化：

（1） 执行`0040D3A9, MOV`指令之前的内存状态（数据窗口）注意看`0040D3BB`处的指令变化

代码:

```
0040D3BA  68 00 00 00 00 C3 8B 85 26 04 00 00 8D 8D 3B 04  h....? ..; 
0040D3CA  00 00 51 50 FF 95 49 0F 00 00 89 85 55 05 00 00  ..QP ..U ..
0040D3DA  8D 85 47 04 00 00 50 FF 95 51 0F 00 00 89 85 2A  G ..P 9Q .\*
```

反序即为`00000000`

（2） 执行`MOV`指令之后的内存状态，注意看`0040D3BB`处的指令变化

代码:

```
0040D3BA  68 CC 10 40 00 C3 8B 85 26 04 00 00 8D 8D 3B 04  h?@.?? 
```

反序即为`004010CC`

看到了吧，程序正是通过在执行过程中动态修改内存来实现对指令的修改的，具体到此处则实现了隐藏`OEP`的目的。其实，如果我们对汇编比较熟悉的话，就很容易知道这句话的含意了。

好的，就说这么多，下面是截的图片...

![](img/8e629a03e78aa0d757ec4c36acae1f0d.jpg)

![](img/bee67a97c4ce68e055c82f3f9d3aa60f.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `NotePad.98.E.rar`

 |
