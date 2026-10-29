
# [原创]二进制漏洞利用中的ROP技术研究与实例分析

（备注：之前有给先知投稿）

随着网络安全的发展，在网络攻方对抗中，漏洞利用的难度在不断增大，为了绕过各种漏洞缓解措施，掌握ROP技术势在必行。在很多时候漏洞程序往往都开启了堆栈代码执行保护NX（windows下叫DEP），这样早期我们直接把SHELLCODE放到栈上执行的方法就完全失效了。ROP技术通俗的讲就是在控制了函数地址指针后，通过不断地跳转到程序加载的地址空间去执行代码的方式，想办法获得我们想要执行的函数的地址并执行，中途或许为了跳转或给寄存器赋值还需要执行一些代码片段（类似`pop ebp,retn`，该代码片段的地址通常被称为gadget），最终获得一个可以交互的SHELL的技术。

本文将以linux x64系统作为调试环境，用IDA作为调试器，以一个实例来分析通过ROP技术来绕过堆栈代码执行保护，最终获得一个交互式SHELL的过程。

## 一、漏洞简要分析

漏洞程序下载地址：<mark class="encrypted">eadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4k6g2u0f1g2$3k6E0K9b7`.`.</mark> 密码: wxd9

漏洞原理分析并不是本文的重点，本文将通过一个简单的存在溢出漏洞的linux x64平台下的二进制文件的分析，来研究在开启了NX保护时，通过ROP技术来实现溢出漏洞利用的技术。下文中该二进制文件统一命名为`rop`，用64位IDA打开`rop`，定位到`vuln`函数，并按F5，看到其反汇编代码如下:

![图片描述](upload/attach/202002/39732_udieqanzim0gtkt.png)

从汇编代码可以看出，进入该`vuln`后 `sub rsp-0x40`，堆栈开辟了`0x40`字节空间，然后调用`gets`函数读入数据到`edi`所指向的空间，`edi`此时实际上是等于`rsp`的指向栈顶的位置，`gets`函数读入数据以换行符号为结束标志，在遇到换行符号前，会读取任意数据到栈里，这样当读入超长字符串后，就会覆盖函数的返回地址，在该函数执行`retn`时就会可以返回到任意我们指定的地方去执行代码。这是一个很典型的缓冲溢出漏洞。

## 二、漏洞调试

下面我们用调试器调试一下这个漏洞发生的过程。本文调试将借助于python代码进行，并需依赖pwn库。

首先在win系统设置IDA，在调试器栏选择remote linux debugger，在菜单debugger下拉菜单里选择process option选项，设置如下：

![图片描述](upload/attach/202002/39732_ltmnplpuyluvdcs.png)

其中目录就选择`rop`所在目录，hostname填上linux虚拟机的ip，同时填上该虚拟机密码，端口默认选择23946。

然后打开linux虚拟机，并把ida目录下的`linux_serverX64`拷贝到虚拟机里并执行，如图：

![图片描述](upload/attach/202002/39732_igjsyfe3ampibut.png)

`rop`的功能比较简单，就是执行的时候，输入字符并打印出来。用py编写调试代码`rexp.py`如下：

```python
#! /usr/bin/python

from pwn import *

import pdb

context.log_level = 'debug'

target = process('./rop')

elf=ELF('./rop') #这个会显示rop用了哪些保护技术

pdb.set_trace()#这里设置一个pdb断点，可以让ida附加rop进程

target.sendline('a'*64+'b'*8+'c'*8)

target.interactive()
```

然后在linux系统上新开一个终端，执行`rexp.py`如下:

![图片描述](upload/attach/202002/39732_jbiicq40jwj7k1d.png)

可以看到py代码中断了，通过调试信息可以看到`rop`开启了nx保护，因为堆栈只开辟了`0x40`字节空间，那么我们用`0x40`个字符a覆盖此空间，再用8个字符b覆盖`ebp`，后面用8个字符c就可以覆盖返回地址了，函数返回时将会跳转到`cccccccc`指向的空间去执行，从而崩溃。下面跟踪调试一下这一过程。

在ida `vuln`函数里的`gets`函数后面下好断点，然后点击debugger，附加远程进程，找到`./rop`打开：

![图片描述](upload/attach/202002/39732_11htp2xc2t37rz1.png)

打开后按F9执行。在linux中断里按n并回车，可以看到栈里的数据被覆盖，并且返回地址恰好被`cccccccc`覆盖掉。

![图片描述](upload/attach/202002/39732_jpz9j5af2vjgccj.png)

当函数执行到`retn`时，`rsp`指向的地址为`cccccccc`，程序将跳转到该地址处去执行，我们成功控制了返回地址，可以跳转到任何地方去执行代码。

![图片描述](upload/attach/202002/39732_u57zd6vcvxdjnqe.png)

但是因为开启了NX保护，我们不可以把shellcode放到栈上来执行了，因此我们就需要用到ROP技术来迂回获得SHELL。

## 三、漏洞利用方式分析

如果要最终执行SHELLCODE来获得SHELL，就要通过`mmap`函数在可执行代码的区域来申请一片空间，然后再执行`gets`函数拷贝shellcode到该空间，最后把`EIP`指向shellcode的空间去执行。这样就需要在`rop`的内存里找到一个可执行可以写的空间，我们在IDA里按ctrl+s如图:

![图片描述](upload/attach/202002/39732_dtc86foprjkbg1m.png)

从上图可以看到所有`rop`空间，凡是可写空间，都不可以执行；凡是可以执行空间都是只读的，所以上面提到的获得SHELL的方式就无法利用了。但是上图也可以看出`rop`内存里加载了libc库，这个库里面有`system`函数，执行`system('/bin/sh')`同样可以获得一个SHELL，因此我们将通过该方式获得SHELL。

然后问题就来了，`rop`虽然没有开启PIE（内存地址随机化），其内存加载基地址虽然是固定的，但是`rop`里面并没有调用`system`函数，无法直接把`system`函数拿来用。我们只好到libc地址空间去找`system`函数地址，但是如果linux系统开启了地址随机化，那么每次加载libc的基地址就会不同。现在该怎么办呢？我们观察到`rop`漏洞函数部分有一个`printf`函数，打印来自终端输入的字符串，是否可以用`printf`函数来获得`system`函数的地址呢？我们看到`rop`里相关代码如下：

![图片描述](upload/attach/202002/39732_3g71oc6uqrwt1jv.png)

从上图可以看出在执行`printf`函数前，`edi`指向的是格式化串，`rsi`指向的是被打印串的地址。如果控制了`rsi`，那么我们就可以打印任何地址的内容。

另外我们看到`rop`调用了几个libc里面的函数如图：

![图片描述](upload/attach/202002/39732_oeyvws4203ab2o3.png)

其中有`printf`、`gets`、`setvbuf`，在`rop`内存这几个函数的GOT表地址是固定的，并且在`rop`执行后，GOT表地址指向的内容就变成了该函数在libc空间的地址。因此如果我们改变`rsi`为某个函数GOT表的地址，那么就可以打印出其在libc空间的地址，然后再根据该函数和`system`函数的偏移来计算出`system`函数的地址就可以了。然后想办法传入`/bin/sh`，再找机会执行`system`就达到目的。下面就来通过ROP实现这一过程。

## 四、ROP链构造获得system函数地址

接下来我们就要去研究如何控制`printf`打印前`rsi`的值。既然我们控制了函数的返回地址，那么是否可以去跳转到一个代码片段，这里给`rsi`赋值后返回，然后再跳转到`printf`那里执行呢？实际上，这就是获得gadget的过程，已经有现成的工具来搜索这样的代码片段了，读者可以自行查找。在IDA里直接搜索文本`pop rsi`也是可以的，但是经过搜索发现并没有这样的代码片段。不过看到了如下代码片段：

![图片描述](upload/attach/202002/39732_u48tdwvoabj3k8j.png)

我们看到我们在栈里布置好数据，先执行`40075A`处代码，然后再跳转到`400740`处去执行，这样就可以控制`r12`、`edi`、`rsi`，然后执行`call`的时候让`call`地址变成`printf` GOT表里的地址，这样就可以打印任何地址指向的内存值了。我们接下来就来调试用`printf`打印`printf`在libc里的地址。

那接下来怎么布置堆栈呢？

1. 返回地址处用`0x40075a`覆盖。
2. 因为上图中`call`执行完后要判断`rbx rbp`是否相等，不等要继续循环，所以这里让`rbx`为`0`，`rbp`为`1`就绕过了循环判断。所以覆盖完返回地址后继续用`0x0`和`0x1`填充，这样`pop`时就可以覆盖掉`rbx,rbp`。
3. 接下来用`printf`的GOT地址填充，这样可以`pop`给`r12`，刚好`rbx`为`0`，然后`call`的时候执行`printf`。
4. `r13`传给了`edx`，这里并没什么作用，因此就用`0x0`填充。
5. `r14`传给了`rsi`，是我们想要打印的地址，这里打印`printf` GOT表地址指向的内容，所以用`printf` GOT表地址覆盖。
6. `r15`传给了`edi`，对于`printf`来说它是一个格式化串，直接就用`rop`里该串的地址`0x400784`填充。
7. `pop r15`后执行的是`retn`，要返回到`rsp`指向的空间，这里因为我们要跳转到`0x400740`去执行`call`，所以后面就用`0x400740`来填充。
8. `call`执行完后，还要执行一个`add rsp, 8`和6次`pop`，然后返回，那么我们在后面再布置7个地址，然后返回地址用`vuln`的地址，然后`rop`继续跳转到漏洞函数里去，方便后面再操作。所以接下来填充7个`0x0`和`0x400656`。

然后利用测试代码如下：

```python
#! /usr/bin/python

from pwn import *

import pdb

context.log_level = 'debug'

target = process('./rop')

elf=ELF('./rop')

printf_got_addr=elf.got['printf']

print hex(printf_got_addr)

rop='a'*72

rop+=p64(0x40075a)# 上面描述的rop链 这里开始覆盖返回地址

rop+=p64(0x0)#->rbx

rop+=p64(0x1)#->rbp

rop+=p64(printf_got_addr)#exec printf

rop+=p64(0x0)#

rop+=p64(printf_got_addr)#

rop+=p64(0x400784)#

rop+=p64(0x400740)#

rop+=p64(0x0)*7#

rop+=p64(0x400656)# return to vul

pdb.set_trace()

target.sendline(rop)

target.recvuntil(':')#正常执行vul时会收到you said:

target.recvuntil(': ') #劫持执行vul时也会收到you said: 所以接收两次

addr=target.recvline()[:-1]

addr = u64(addr+'\x00'*(8-len(addr)))

print 'printfs addr is:'

print hex(addr)

target.interactive()
```

然后我们利用前面提到的调试方法来调试，py代码执行后如图：

![图片描述](upload/attach/202002/39732_yk54miyqeatm6h7.png)

这里打印出了`printf` GOT地址为`0x600af0`，按n后，IDA在`gets`后断下，如图所示：

![图片描述](upload/attach/202002/39732_bv0m6gwqn29wfe9.png)

我们看到堆栈里，返回地址被成功覆盖，然后`0`、`1`都覆盖成功了，接下来本来应该覆盖成`printf_got_addr=0x600af0`，现在却成了`0x400f0`，然后后面的地址都没有被按我们想象的填充，而是堆栈里原本的值。这又是为什么呢？

百思不得其解？经过仔细研究分析，前面提到过`gets`遇到换行符号就结束，后面的内容不再读取了，再看看`printf_got_addr=0x600af0`这个数据里面刚好有个`0x0a`，这个就是换行符号对应的内存值，因此在读取`0xf0`后`gets`就结束读取了，所以后面的就无法正常覆盖了。那接下来怎么办呢？我们只好再整理一下思路，继续突破。能否执行`gets`函数，然后读入数据到某个地方，最后执行呢？答案在前面就否定了，能写的都不能执行。`rop`里用的函数就只有`printf`、`gets`、`setvbuf`，仔细查过`setvbuf`貌似也没啥用，`gets`只能读入又不能读出。这下感觉身陷困境，有什么办法呢？

我们再来分析一下前面找到的用来控制寄存器和执行函数的代码片段：

![图片描述](upload/attach/202002/39732_9irooq46glvhdhz.png)

在痛苦地思索了良久后，突然发现，不用直接传`0x600af0`进去啊，可以配合`rbx*8`变换，然后`r12`变成一个其他的没有`0x0a`字节的值就可以了啊。这时`r12=0x600af0-rbx*8`，如果要改变到`0x0a`字节，那么需要`rbx*8`大于`0xf0`，于是`rbx > 0xf0/8`，即`rbx > 0x1e`，这里不妨取`rbx=0x1f`，那么`r12=0x600af0-0xf8`。为了让`cmp rbx rbp`返回为`0`，直接往下执行，不再循环，那么需要让`rbp=rbx+1`（在`cmp`前`rbx`增加了`1`）。为了不有任何`0x0a`字节，接下来我们把要打印的地址变成`gets`的地址，通过它和`system`的偏移也可以计算出`system`地址来，然后我们的`rop`链就变成如下了：

```python
rop+=p64(0x40075a)#

rop+=p64(0x1f)#->rbx

rop+=p64(0x20)#->rbp

rop+=p64(printf_got_addr-0xf8)#exec printf

rop+=p64(0x0)#

rop+=p64(gets_got_addr)#

rop+=p64(0x400784)#

rop+=p64(0x400740)#

rop+=p64(0x0)*7#

rop+=p64(0x400656)# return to vul
```

替换上面代码`rop`链部分，然后我们再来调试执行，如图：

![图片描述](upload/attach/202002/39732_p3vu0yeo887qz4d.png)

现在堆栈里的数据已经完全按照我们的要求布置好了，然后我们单步继续运行，当执行`retn`后，程序将跳转到`0x40075A`处去执行，单步步过`retn`后如图：

![图片描述](upload/attach/202002/39732_zvlidlqolf0gx7u.png)

接下来，现在的堆栈里的数据将会一个一个被`pop`到寄存器里，然后跳转到`0x400740`处去执行，步过`retn`后再单步几次来到`0x400749`处如图：

![图片描述](upload/attach/202002/39732_r1f8skg71nmjwep.png)

可以看到此时`edi`为格式化串的地址了，`rsi`为`gets` GOT地址（存放的是其在libc里的地址）将要被打印，`r12+rbx*8=0x600af0`是`printf` GOT地址，`call`执行后将打印`gets`在libc里的地址。同时看到堆栈里有7个地址都是存放的`0`，在执行完`call`后会`add rsp, 8`，再6次`pop`刚好把`0`都`pop`完后返回到地址`0x400656`处（`vuln`函数地址）执行。可是当在`call`处按下F8单步执行后，让人意外的事情再次发生了：

![图片描述](upload/attach/202002/39732_sr4latpcfc14hgh.png)

`rop`发出了一个它准备退出的信号，好吧不跟我们玩了，漏洞分析调试就是一个让人兴奋与崩溃并存的事情，保持耐心，收拾好心情，我们再来分析分析！

这时重新调试`rop`在上面`call`的地方，我们单步进去，如图：

![图片描述](upload/attach/202002/39732_d0pmphohncy1scx.png)

进来之后发现这里有处跳转，但是`rax`为`0x56`，跳转不能实现：

![图片描述](upload/attach/202002/39732_b1tcqbl9k2aa749.png)

会不会正常执行`printf`时这里跳转实现了呢，我们在正常`call _printf`处时单步进去，然后F8一直跟踪到上述代码处，如图：

![图片描述](upload/attach/202002/39732_kgbzb6hi4phkyzp.png)

发现这里`rax`为`0`，也就是要正常执行`printf`，还要保证`rax`为`0`。

前面我们覆盖返回地址为`0x40075a`，在该处没有代码如下：

![图片描述](upload/attach/202002/39732_kgbzb6hi4phkyzp.png)

该处完全没有能控制`rax`的地方，那么怎么办呢？这样我们在返回执行`0x40075a`前，还要找到一处代码片段（gadget）控制`rax`为`0`，然后再跳到`0x40075a`处执行。通过搜索，发现`0x4005f3`处有如下代码：

![图片描述](upload/attach/202002/39732_2yo1u2lc59csd7f.png)

这里把`eax`置`0`，然后直接跳转到`pop rbp`处然后返回，可以满足我们的需求。然后我们把返回地址覆盖为`0x4005f3`，然后再填充一个`0x0`让其`pop ebp`用，接着`retn`时返回地址我们就布置为上面的`0x40075a`，这样就完成了`rax`指令操作后又跳回去执行`printf`函数了。现在我们的`rop`链如下：

```python
rop='a'*72

rop+=p64(0x4005f3)# eax=0 or printf can't run normer
```
