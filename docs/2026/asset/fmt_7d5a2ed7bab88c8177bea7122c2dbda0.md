
# 代码片段

```asm
0041B031 JNZ 0x3d(0041B074)
0041B043 MOV DWORD PTR [EAX],EBX
0041B0B6 POP EDX
```

```
//-------分割线---（以下代码都是连续的，我人为的分开是方便大家阅读）-----------------------------
//以下这段不经常用到，就是如果opcode中设置另外个flag的话，是可以读取计算中的EFLAGS寄存器的结果的
```

```asm
0041B0C4 MOV ECX,DWORD PTR [EBP+0x8]
0041B0D9 XOR DWORD PTR [EBP+0x53],0x1e4ffb66
0041B10A SUB DWORD PTR [EBP+0xa7],0x1d2ace37
0041B112 ADD ECX,0x6
0041B129 MOVZX ECX,BYTE PTR [ECX]
0041B12C CMP ECX,0x0
0041B132 JE 0x85(0041B1BD)
0041B180 MOV EDI,DWORD PTR [EBP+0x8]
0041B182 ADD EDI,0x0
0041B196 MOV CX,WORD PTR [EDI]
0041B19E ADD ECX,EBP
0041B1AD MOV DWORD PTR [ECX],EDX
```

```
//-------分割线---（以下代码都是连续的，我人为的分开是方便大家阅读）-----------------------------
//最后这段，每个handle都会有，就是重新设置opcode的偏移，读取下一个handler号并跳转，大家可以自己阅读一下
```

```asm
0041B1DF MOV EDX,DWORD PTR [EBP+0x8]
0041B1E8 MOV ESI,DWORD PTR [EBP+0x4f]//这里是handler指针表
0041B1EC ADD EDX,0x7
0041B1FA MOVZX EDX,WORD PTR [EDX]
0041B20E ADD EDX,DWORD PTR [EBP+0x53]
0041B242 XOR EDX,0xf833c89
0041B274 XOR DWORD PTR [EBP+0x53],EDX
0041B278 AND EDX,0xffff
0041B280 SHL EDX,0x2
0041B28D ADD ESI,EDX
0041B298 MOV EDI,DWORD PTR [ESI]
0041B2A4 ADD DWORD PTR [EBP+0x8],0xb
0041B2AA JMP EDI
```

看完这段代码，我想大家也应该对`type1 handler`有了一个大概了理解，我在这里总结一下

## handler 结构总结

1、 `handler` 代码通常由读取`opcode`，处理`opcode`，保存结果和最后的跳转四部分组成。

2、 加密寄存器无时不刻在对`opcode` 处理过程中的中间数和结果加密，可以说走一步加密一下，直到最后使用这些结果的时候才解密，真是结果出现的时间被竟可能的压缩。而加密寄存器本身也在不断的被加密，所以想要爆破是不可能的。

3、 `Opcode`指令序列可以由不同的几个部分组成的，包括 `handleid` `subhandle` `flag` 立即数虚拟寄存器号参数大小的控制参数等，长度是可变的，但具体到哪个偏移量存放那种参数却不一定，完全由`handler`决定，最长应该不超过12位。

4、 `Opcode`中的`flag`说明了后面参数的类型和大小。高位控制类型，1为虚拟寄存器，2为虚拟地址，3为立即数。低位控制大小，3为`DWORD`,2为`WORD`,1为`BYRE`。

## flag 意义示例

最后，我们通过阅读一段从`OU`中截取的`type1 handler`处理`opcode`后的中间代码，来对应的比较和学习一下`flag`的意义。我们从前到后依次分析。

```text
{001B} [00000039-7D54B913-0000001A-00000000-00000000] NOT EBP
```

1、前面的`001B`是`handler` 的`ID`,也可以说是`handler`在`handler`表中的序号。

2、`00000039`是`subhandle` ,其实在`Opcode`中它只占1位。对应前面代码中的`0041AE06`位置，可以看到，这是个`NOT`操作。

3、`7D54B913`这个其实就是`flag`，我们只需要看最后1位 `0x13`，高位的1表示操作数是虚拟寄存器,低位的3代表`DOWRD`。前面的是解密过程中产生的垃圾代码，当然也不能说完全垃圾，他们是还会被用来加密加密寄存器的。

4、`0000001A` 这个就是虚拟寄存器的代号了，这里的`1A`代表`EBP`，你要问这个代号是哪里来的，抱歉，这个是虚拟机生成时随机产生的，不会有两个加密的程序会完全一样。

5、以上合起来就是后面的一个简单的操作，`NOT EBP`。化简为繁`themida`是做的很到位的。

到这里，本文的第二部分也差不多结束了，后面其实我还想把`VM FISH`中最麻烦的关于`EFLAGS`的操作和处理，连带条件跳转的判断处理一起整理一下，毕竟不把这块说清楚就不算彻底弄明白`FISH VM`,当然这也不属于初探的范畴了。只可惜笔记时间久远，而且比较分散，可能需要各位耐心等待一段时间，我有时间一定尽快整理发布。![](https://bbs.kanxue.com/view/img/face/41.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
