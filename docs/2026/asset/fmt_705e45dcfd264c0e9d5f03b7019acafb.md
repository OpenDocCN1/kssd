
# [原创]X64下的虚拟地址到物理地址的转换

早就知道传上来排版会全乱掉，把pdf直接传上来吧

[x64结构体系寻址.pdf](https://bbs.kanxue.com/attach-download-99453.htm)

发现安大的关于x86启用PAE下的虚拟地址转物理地址的帖子，大家可以参考一下<http://bbs.pediy.com/showthread.php?t=180989>

## X64结构体系内存寻址

在阅读NewBluePill源码的时候，看内存的那一块简直头疼，全是x64下的寻址，之前根本就没有接触过x64的内存寻址上的内容，看的晕头转向，决定先把x64下的寻址给弄明白了再回过头来看NewBluePill的源码，然后在网上一顿找，居然没有找到关于x64寻址的博客或者文章，简直痛苦啊，然后找了好久才发现一本不错的书《x86/x64 体系探索及编程》，然后终于把x64的寻址问题弄清楚了，总结出来分享一下学习历程。

## 0x01 x64寻址简介

在保护模式下，CPU发出的线性地址，由内存管理单元（MMU）根据当前`CR3`寄存器所指向的页表物理地址，将该线性地址翻译成物理地址进行内存访问，该过程称为地址翻译。

在x64体系结构中，线性地址的结构如图

![](https://bbs.kanxue.com/upload/attach/201508/632449_y2p0rthhzj0x0rq.png)

在x64体系中只实现了48位的virtual address，高16位被用作符号扩展，这高16位要么全是0，要么全是1。

不同于x86体系结构，每级页表寻址长度变成9位，由于在x64体系结构中，普通页大小仍为4KB，然而数据位宽却是64位，因此一个4KB页在x64体系结构下只能包含512项内容，所以为了保证页对齐和以页为单位的页表内容换入换出，在x64下每级页表寻址部分长度定为9位。

为了正确翻译x64的线性地址，其页表也从x86的2级变成了4级，翻译过程如图所示，在x64体系结构中，每级页表包含512项（2^9）指向下级目录的指针，该指针称为页表项，描述了存储下级

![](https://bbs.kanxue.com/upload/attach/201508/632449_8k9zl2c59lnaqsn.png)

- `PML4T`（Page Map Level4 Table）及表内的`PML4E`结构，每个表为4K，内含512个`PML4E`结构，每个8字节
- `PDPT`（Page Directory Pointer Table）及表内的`PDPTE`结构，每个表4K，内含512个`PDPTE`结构，每个8字节
- `PDT`（Page Directory Table）及表内的`PDE`结构，每个表4K，内含512个`PDE`结构，每个8字节
- `PT`（Page Table）及表内的`PTE`结构，每个表4K，内含512个`PTE`结构，每个8字节

每个table entry的结构都是8个字节64位宽，而virtual address中每个索引值都是9位，因此每个table都是512 x 8 = 4K字节。

## 0x02 页转换模型

X64，准确的说应该是IA32e paging模型提供了三种页转换模型：

1. 4K页面的转换表结构；
2. 2M页面的转换结构；
3. 1G页面的转换结构。

在64位模式下，处理器将48位虚拟地址转化为物理地址，在兼容模式下，转化32位的虚拟地址。

![](https://bbs.kanxue.com/upload/attach/201508/632449_ef9rxtwfgz45sc2.png)

三种模型都是物理页帧的基地址加上页偏移得到物理地址，不同只是在于页帧的大小划分不同：

1. 4K页面：使用`PML4T`、`PDPT`、`PDT`和`PT`四级页转化表结构；
2. 2M页面：使用`PML4T`、`PDPT`和`PDT`三级页转化表结构；
3. 1G页面：使用`PML4T`和`PDPT`二级页表转化结构。

而在这里我们主要讨论的是4K页面大小的寻址方式，因为在个人计算机上，普遍都是4K页面寻址，其他的方式也主要就是页面大小的差异。

## 0x03 最大物理地址

在Intel中使用`MAXPHYADDR`来表示最大的物理地址，我们可以通过`CPUID`指令来获得处理器支持的最大物理地址，然而这已经不在此次的讨论范围之内，我们需要知道的只是：

- 当`MAXPHYADDR`为36位，在Intel平台的桌面处理器上普遍实现了36位的最高物理地址值，也就是我们普通的个人计算机，可寻址64G空间；
- 当`MAXPHYADDR`为40位，在Intel的服务器产品和AMD的平台上普遍实现40位的最高物理地址，可寻址达1TB；
- 当`MAXPHYADDR`为52位，这是x64体系结构描述的最高实现值，目前尚未有处理器实现。

而4K页面寻址对下级表的物理地址的存储遵循如下规则：

1. 当`MAXPHYADDR`为52位时，上一级table entry的12~51位提供下一级table物理基地址的高40位，低12位补零，达到基地址在4K边界对齐；
2. 当`MAXPHYADDR`为40位时，上一级table entry的12~39位提供下一级table物理基地址的高28位，此时40~51是保留位，必须置0，低12位补零，达到基地址在4K边界对齐；
3. 当`MAXPHYADDR`为36位时，上一级table entry的12~35位提供下一级table物理基地址的高24位，此时36~51是保留位，必须置0，低12位补零，达到基地址在4K边界对齐。

## 0x04 实际转化

### `CR3`

当`CR4.PCIDE = 0`时，`CR3`的结构如图

![](https://bbs.kanxue.com/upload/attach/201508/632449_6asxlf6ojbbx9zn.png)

`CR3`可以使用64位宽，但是它表示的`PML4T`的物理基地址同样受到之前所说的`MAXPHYADDR`的约束，图示的只是理想的`MAXPHYADDR`为52位时的情况。

而当`CR4.PCIDE = 1`时：

![](https://bbs.kanxue.com/upload/attach/201508/632449_fm71twv8p6oge9b.png)

`CR3`的低12位提供一个`PCID`值，用来定义当前Process Context ID。

当对`CR3`进行更新时，`CR3`第63位决定是否需要处理器的TLB和paging-struct cache，这不在我们此次谈论的范围之内。

### `PML4E`

接着再看`PML4E`的结构，如图：

![](https://bbs.kanxue.com/upload/attach/201508/632449_srsqnaqyf9vnutj.png)

`PML4E`并没有`PS`标志位，因此第7位是保留的，而`PML4E`提供的`PDPT`的物理基地址也受之前的`MAXPHYADDR`规则的约束。

### `PDPTE`

然后就是`PDPTE`结构：

由于新增了1G页面，因此在`PDPTE`结构里将控制1G的页面转化，由`PDPTE.PS`标志位进行转换，如图：

![](https://bbs.kanxue.com/upload/attach/201508/632449_7az929ymvpeey2m.png)

当`PDPTE.PS = 1`，也就是`PDPTE`的第7位为1时，`PDPTE`将提供1G的物理页面地址；当`PDPTE.PS = 0`，也就是`PDPTE`的第7位为0时，使用非1G的页面，将提供下一级的`PDT`的物理基地址，同样受`MAXPHYADDR`规则的约束。

1G页面下的`PDPTE`结构解析如下：

![](https://bbs.kanxue.com/upload/attach/201508/632449_avsaxcjol48jmf9.png)

同样地，`PDPTE`提供的1G页面的物理地址也遵守`MAXPHYADDR`规则，1G页面的地址低30将补0，意味着在1G边界上对齐。

4K和2M页面下的`PDPTE`结构解析如下：

![](https://bbs.kanxue.com/upload/attach/201508/632449_y5nfqmc734wj02g.png)

将提供下一级`PDT`的物理基地址，同样也遵循`MAXPHYADDR`规则，那么再根据`PDE.PS`决定是使用2M页面还是4K页面。

### `PDE`

`PDE`的结构和`PDPTE`类似，也是用`PS`（第7位）表示是使用2M的页面还是4K的页面，下面是2M页面的`PDE`结构解析：

![](https://bbs.kanxue.com/upload/attach/201508/632449_1fzon3urlct35y3.png)

同样，页面的物理基地址也遵循`MAXPHYADDR`原则。

接下来是4K页面的`PDE`结构解析：

![](https://bbs.kanxue.com/upload/attach/201508/632449_l0tlsqdyvol9fh1.png)

也遵循`MAXPHYADDR`规则。

### `PTE`

`PTE`的结构解析：

![](https://bbs.kanxue.com/upload/attach/201508/632449_s3k04d0ivojc4ug.png)

同样遵循`MAXPHYADDR`规则。

## 0x05 实际例子

上面写了很多原理性的东西，可能看完之后对于x64还没有很清晰的认识，我们以一个很简单的例子来加深对于x64结构体系的寻址的认识。

```
#include "stdafx.h"
#include <Windows.h>

int _tmain(int argc, _TCHAR* argv[])
{
  char szName[20] = "HelloWorld";
  printf("szName:%x\n",szName);
  getchar();
  return 0;
}
```

很简单的一个程序，就是打印出`szName`的虚拟地址，运行结果如下：

![](https://bbs.kanxue.com/upload/attach/201508/632449_1d7o7oageu13rax.png)

我们接下来要做的就是将`0x2ffde8`这个虚拟地址转换成物理地址，在物理页上找到我们的`"HelloWorld"`。

`0x2ffde8` ===> 转换二进制：

```
000000000     000 0000 00     00 0000 001      0 1111 1111        1101 1110 1000
0              0              1              0xff                0xde8
PML4E索引     PDPTE索引     PDE索引         PTE索引            页内偏移
```

![](https://bbs.kanxue.com/upload/attach/201508/632449_5j2t4t54cqga6ka.png)

目标进程的`DirBase`为`0x7d838000`，根据我们之前学习的寻址方式，应该是按照`MAXPHYADDR`为36位的规则，即上一级table entry的12~35位提供下一级table物理基地址的高24位，此时36~51是保留位，必须置0，低12位补零。

![](https://bbs.kanxue.com/upload/attach/201508/632449_05vok8jmvrfwgqb.png)

因为`PML4E`的索引为0，所以我们的目标`PML4E`项的值为`0x02b00000~7d274867`，12~35位为`0x07d274`，低12位补零，则：

![](https://bbs.kanxue.com/upload/attach/201508/632449_37f8g5ku8stttg5.png)

`PDPTE`的索引也为0，目标`PDPTE`项的值为`0x03000000~7d737867`，`PS`位（第7位）为0，12~35位为`0x07d737`，低12位补零，则：

![](https://bbs.kanxue.com/upload/attach/201508/632449_cpfmsbjbhkjhjxh.png)

因为`PDE`的索引为1，所以我们要加上`8`，目标`PDE`项的值为`0x01500000~7d7bb867`，`PS`位（第7位）为0，12~35位为`0x07d7bb`，低12位补零，则：

![](https://bbs.kanxue.com/upload/attach/201508/632449_i81qz90yjsoyft3.png)

`PTE`的索引为`0xff`，所以要加上`0xff * 8`，得到目标`PTE`项的值为`0x89a00000~7d084867`，12~35位为`0x07d084`，低12位补零，得到页面物理基地址，再加上页面偏移，我们是`0xde8`，则：

![](https://bbs.kanxue.com/upload/attach/201508/632449_nm2wkju1dg06emx.png)

终于在物理页上看到了我们熟悉的“HelloWorld”。

[x64结构体系寻址.pdf](https://bbs.kanxue.com/attach-download-99453.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- r1.png（41.98kb，37次下载）
- r2.png（233.22kb，29次下载）
- r3.png（22.05kb，10次下载）
- r4.png（34.59kb，8次下载）
- r5.png（43.11kb，8次下载）
- r6.png（53.16kb，8次下载）
- r7.png（56.86kb，10次下载）
- r8.png（39.23kb，8次下载）
- r9.png（52.76kb，8次下载）
- r10.png（57.89kb，8次下载）
- r11.png（51.84kb，16次下载）
- r12.png（57.52kb，15次下载）
- addr.PNG（13.33kb，8次下载）
- ProcessInfor.PNG（8.31kb，29次下载）
- 1.PNG（12.56kb，8次下载）
- 2.PNG（12.87kb，8次下载）
- 3.PNG（12.47kb，7次下载）
- 4.PNG（17.28kb，6次下载）
- 5.PNG（18.83kb，15次下载）
- x64结构体系寻址.pdf（570.10kb，941次下载）
