
# [原创]初识堆喷射及事例(暴雷漏洞)分析

# 暴雷漏洞CVE-2012-1889

## 一．堆喷简介

打开Poc利用文件，发现使用了HeapSpray（堆喷技术），堆喷射实现的主要因素为以下两点（堆喷射将shellcode放置在了堆中，在堆中执行代码）：

1. 使用浏览器程序打开我们的poc样本时，它会执行我们样本文件中的JavaScript代码
2. 控制程序eip，使其指向`0x0C0C0C0C`地址

下面我们来了解一下堆喷射的实现流程：

1. 我们首先创建一个大小为1mb的堆块，并使用`0x0C0C0C0C`填充

1.1 为什么使用`0x0C0C0C0C`填充呢？有两点因素：

（1）`0x0C0C0C0C`会被程序解释成`OR AL,0C`，可以作为滑板指令（即执行此种指令不会对程序的后续行为产生影响），有人说为什么不使用`0x90`（也是滑板指令）呢？请看下一条

（2）之前说过，我们的shellcode会被放到堆中去执行，也就是所谓的使eip指向`0x0C0C0C0C`这个地址，而`0x0C0C0C0C`这个地址从0计算的话，大概在192mb左右，但`0x90909090`就不言而喻了，需要申请的堆空间那就相当大了。又有人会说为甚麽一定要让我们的eip指向`0x0C0C0C0C`呢？继续往下看

2. 计算好shellcode的字节数，将shellcode的代码贴到我们申请的1mb堆块的尾部，控制总大小刚好为1MB

![](https://bbs.kanxue.com/upload/attach/201811/833287_QPG9PKDW9ECUT76.jpg)

3. 创建一个成员数为200的数组，数组的每个成员都是这样的堆块（为什么是200MB呢？因为可以保证`0x0C0C0C0C`这个地址指向我们所构造的这个数组中）

![](https://bbs.kanxue.com/upload/attach/201811/833287_3UB2BEHVRZKUEYN.jpg)

4. 为什么有很大概率指向我们的滑块指令`0x0C0C0C0C`（此处作为滑块指令）？而不是shellcode代码呢？

一般我们的shellcode的大小在500字节左右（也许会更小），而一个块的大小为1MB，相当于我们的shellcode只占了整个块1/2000。当eip指向我们的滑板指令`0x0C`时，会一点点地往后执行到我们的shellcode，此时我们的漏洞就算利用成功了。我们从事例中讲解堆喷的实现。

## 二．事例分析

### 1. 暴雷漏洞环境

虚拟机环境：WinXpSP3

漏洞软件：ie6.0

调试工具：OD、IDA

样本文件缀至文末

### 2. Poc样本中堆喷块的详细构造流程

![](https://bbs.kanxue.com/upload/attach/201811/833287_EHUR6JTTT4NFP3H.jpg)

### 3. 漏洞分析

#### 3.1 配置环境

虚拟机环境就不多说了

ie的环境大家可以下载一个`IECollection`，里面有IE各版本的集合

#### 3.2 触发漏洞

（1）修改Poc样本，不构造堆块，直接触发漏洞溢出（换言之就是将poc样本中的前三个步骤删掉直接触发漏洞），同时修改OD的异常选项，使其不忽略任何异常。

![](https://bbs.kanxue.com/upload/attach/201811/833287_X5EWSUUR8KQ5G2S.jpg)

（2）OD附加，拖入样本，中断至下图所示

![](https://bbs.kanxue.com/upload/attach/201811/833287_PJWCKDXAN9VEUNM.jpg)

（3）追栈，查找溢出点

![](https://bbs.kanxue.com/upload/attach/201811/833287_AUFWRMT7R2V3T2H.jpg)

（4）`77f5f87b`下软断，`0x13df98`下硬断（凡是被覆盖的点都可以，最好选择中间位置下断），查找何时溢出

（5）查看此时的`edi`（`0x13DEFA`被我们覆盖的栈），`esi`（`0x1885D8A`好像是个堆地址）

![](https://bbs.kanxue.com/upload/attach/201811/833287_SWB4V87DUN9QQ8R.jpg)

（6）IDA查看相关代码

![](https://bbs.kanxue.com/upload/attach/201811/833287_ABDJEGMHZF99PPJ.jpg)

（7）我们继续往上追`esi`的来源，即`ebp - 0x94`（ida中的`pszUrl`）（`0x13A090`），硬断写入`0x13A090`，当该地址内被填充为一个明显堆地址时断下，此时，堆地址已被填充为`0x1000`字节的图片原路径

![](https://bbs.kanxue.com/upload/attach/201811/833287_JCRYWMN5SEYW3B6.jpg)

（8）至此，我们便可以下结论了，由于程序员的疏忽，并未对图片源路径的长度做限制，在对图片源路径拷贝时，造成了栈溢出，覆盖了栈中的重要指针，从而使eip指向`0x0C0C0C0C`

#### 3.3 利用流程如下

![](https://bbs.kanxue.com/upload/attach/201811/833287_C7WVCNXEAVDWGF4.jpg)

① 栈被覆盖后，从栈中取出指针（`0x0C0C0C0C`）赋值给`eax`

② 由于堆喷射原理，所以`0x0C0C0C0C`地址内的数据大概率还是`0x0C0C0C0C`（滑块），此时`ecx = 0x0C0C0C0C`

③ 同理，`ecx + 0x18`该地址内大概率也是`0x0C0C0C0C`（滑块），从而滑向我们的shellcode

### 4. 漏洞利用（具体利用流程参照Poc样本）

(1) 先将自己的shellcode由ascii格式转换成unicode格式，具体转换方式如图

![](https://bbs.kanxue.com/upload/attach/201811/833287_VR44QYK9UDD6YRK.jpg)

(2) 制作滑板数据

每个滑板块均为1mb

申请`1024*1024 - 32 - 4 -2`字节的空间

具体填充方式如图

![](https://bbs.kanxue.com/upload/attach/201811/833287_MWC6SQZGEPGMHTH.jpg)

注意：填充内容为`0x0C`，而不是图中的`0x90`

(3) 制造200个这样的数据，形成堆喷区

(4) 触发漏洞，使eip指向`0x0C0C0C0C`

![](https://bbs.kanxue.com/upload/attach/201811/833287_6GZWJSJX2XJAETK.jpg)

(5) 利用成功，哈哈哈，窗口全部拆解（个人shellcode功能为拆解桌面所有窗口...）

![](https://bbs.kanxue.com/upload/attach/201811/833287_RMFXYM68TKN4P6B.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2018-11-23 11:20 被F4our444编辑，原因：修改

上传的附件：

- [直接触发漏洞\_poc.htm](javascript:void(0);)（1.06kb，47次下载）
- [ie6.0+xpsp3\_poc.htm](javascript:void(0);)（3.58kb，37次下载）
