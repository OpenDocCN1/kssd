
# [原创]vmp 过代码校验

# 代码校验HANDLER

简化后的代码

```
START：

mov edx,dword ptr ss:[ebp]

add ebp,0x4

LOOP：

xor eax,eax

mov ecx,eax

shl eax,0x7

shr ecx,0x19

or eax,ecx

xor al,byte ptr ds:[edx]

inc edx

dec dword ptr ss:[ebp]

jnz LOOP

mov dword ptr ss:[ebp],eax

END
```

这个`handler`需要两个参数，分别是代码校验的地址和大小，通过`xor`指令生成校验码，最后将堆栈的两个参数弹出，将校验码压栈，执行下一个`handler`

# 调用分析

以通过`OD`的条件断点，在校验`handler`下断，分别打印出 `vmcode(ESI)` - 1 、传入的两个参数进行分析

![](https://bbs.kanxue.com/upload/attach/201912/779774_JSE5JT64DWPMYHV.jpg)

日志冗长，就不贴了，经分析后一共有4处调用了校验`handler`，校验的部分分别是文件校验，代码校验，内存校验，随机校验

这4处校验`handler`调用之后的校验码比对部分有一个共同点，就是它们都是对`ZF`标志位进行判断

还有一个点就是`hash`比较后的结果会存储在一个寄存器中的`BL`位，也就是低字节，由堆栈弹出1字节并赋值，这里说的寄存器指的是`vm context`中的数据，虽然没有在每个`hash handler`中验证，但我猜测是一致的

在前3次处理中，会在`vm context`中保存循环的次数：

![](https://bbs.kanxue.com/upload/attach/201912/779774_QMZRMX5VSDCPVEE.jpg)

上图中的堆栈`33F608`的位置保存了循环次数为4次

这个值的位置是不固定的，需要自己手动分析

通过这个循环次数，和`ZF`的标志位判断就可以跳过校验部分了

# 代码分析

校验码计算是使用刚刚计算的校验码与原校验码相减，然后判断`ZF`标志位

运算过程如下：

![](https://bbs.kanxue.com/upload/attach/201912/779774_Y8M9B9HXD58VAGA.jpg)

化简后是这样的：

`a-b = ~(~a + b)`

`f1 = eflags of (~a + b)`

`f2 = eflags of ~(~a + b)`

`eflags = (f1 & 0x815) + (f2 & ~0x815)`

`M1 = eflags & 40`

通过对`M1`进行右移6位取出`ZF`标志位并保存到`vR13BL(vm context)`中

上图为以前分析的一个`vmp` 的 `hash`过程的记录，从初始化到一次循环的记录，文末有下载链接，由于每一次`jmp`都会对`vmcontext`进行打乱，阅读会有些不连贯，只能作为参考

# 跳过校验部分

前三次很好跳过，直接在代码校验的`handler`上下断，修改循环次数为1，不要修改为0，因为在`handler`中还要进行减一操作，如果你不知道循环次数在`vmcontext`中的保存的位置，可以用条件断点相对`esi`的值来判断

第四次并没有保存循环次数，但是它用到了右移指令，通常情况下，`vmp`会使用右移四位来判断`zf`位，如果`zf`位为1，那么右移4位后结果为4，否则为0

那么在第4次处理时首先在校验`handler`下断，断下后在右移`handler`下断，一般来说，第一次的右移是对`hash`校验码的判断，第二次右移是对循环的判断，所以当右移`handler`第二次断下，这个`handler`的结束会将生成的数值压栈，我们要做的就是把这个值修改为0，就可以跳过代码校验了

如果还是被检查出来了，问题就出在了每次处理的第一次检查的数据对比中，可以参考第四次的处理，在`shr`的`handler`中处理这个问题

`vmp`分析记录：

`096K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5i4y4A6j5$3u0A6N6q4)9J5k6h3y4F1i4K6u0r3k6r3!0U0i4K6u0r3g2X3#2H3i4K6t1#2x3U0m8Z5j5i4y4Z5i4K6t1#2x3U0m8A6L8X3W2@1i4K6t1#2x3U0m8S2L8X3q4D9P5i4y4A6M7#2)9J5k6h3c8G2j5%4R3`.`

`363K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5i4y4A6j5$3u0A6N6q4)9J5k6h3y4F1i4K6u0r3k6r3!0U0i4K6u0r3d9r3q4K6K9q4)9J5y4e0t1H3L8r3!0G2M7q4)9J5k6h3c8G2j5%4R3`.`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-12-17 22:15

被`Hasic`编辑

，原因：

[#VM保护](https://bbs.kanxue.com/forum-4-1-4.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
