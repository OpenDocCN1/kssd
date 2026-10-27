
### `IMAGE_RESOURCE_DIRECTORY_ENTRY.DUMMYUNIONNAME2`

在资源目录项中该字段是一个联合体类型，大小为4个字节，它决定这个资源目录的目录中子节点的类型(是目录还是节点)。如果这个字段的最高位是1，则表示该资源的子节点是一个目录类型，该字段的低31位是子目录的资源目录头结构的偏移，但这个偏移既不是FOA也不是RVA，它是以首个资源表的地址为基址，加上低31位的值才是资源目录头结构的地址。如果最高位为0，则表示该资源的子节点是一个节点，它也以首个资源表的地址为基址，整个字段的值就是该资源节点的偏移。这个节点是`IMAGE_RESOURCE_DATA_ENTRY`类型的结构体。(一般在三级目录中该字段的最高位位0，而在其他两个目录中该字段的最高位为1)

> 注：为了编程方便，`IMAGE_RESOURCE_DIRECTORY_ENTRY`的联合体中出现了一组特殊的struct结构体，其成员声明格式为：[类型] [变量名] : [位宽表达式]， 这个格式就是C语言中位段的声明格式。`NameOffset`字段的值等于该联合体的低31位，`NameIsString`字段的值等于该联合体的最高位。将一个4字节的类型拆成这样两个字段就可以方便的避免了繁琐的位操作了，而且该结构的总大小不会发生变化。

### `IMAGE_RESOURCE_DATA_ENTRY`

这个结构体就是目录资源的三级目录下的子目录，里面存储的就是资源文件的信息，如`OffsetToData`字段存储的就是资源文件的RVA地址，它指向了资源的二进制信息，`Size`字段存储的就是资源文件的大小，`CodePage`字段存储资源的代码页但大多数情况为0。

注：在其指向的资源数据中，字符串都是Unicode的编码方式，每个字符都是由一个16位(一个单字)的值表示，并且都是以`UNICODE_NULL`结束(其实就是两个0x00)。

### `IMAGE_RESOURCE_DIR_STRING_U`

该结构体就是目录资源的名称结构，里面存在两个字段，都是2个字节，`Length`字段存储的是目录资源名称的长度，以2个字节为单位。`NameString`字段是一个Unicode字符串的第一个字符，并不以0结尾，其长度是由`Length`字段限制。该结构的总大小并不是表面上的4个字节，而是根据名字长度变化的，计算方式为：`Size = SizeOf(WCHAR) * (Length + 1)`; 这里的1是`Length`字段的大小。

**资源表的结构图：**

![资源表结构图](upload/attach/202002/813468_WP2WGCATM99FP73.png)

**资源表的结构图(简图)：**

![资源表结构简图](upload/attach/202002/813468_EZ7BH56WEKXXGBV.png)

**手动寻找资源数据(为了方便程序的两个对齐方式的值都是0x1000)：**

**1、通过可选PE头定位资源表，解析第一个资源目录项：**

![资源表1](upload/attach/202002/813468_Q27P98YX88V8ZX5.png)

(1)、可以得到资源目录头中一共有9个目录项。
(2)、在第一个目录项中以ID号命名，资源类型位2也就是位图资源。
(3)、在目录项的二个字段可以得知该目录的子节点也是目录，偏移是0x58，`RVA = 0x2B000 + 0x58 = 0x2B058`。

**2、定位二级资源目录并解析：**

![资源表2](upload/attach/202002/813468_2Z34MHFK7FJ9YV5.png)

(1)、可以得到资源目录头中一共有1个目录项。
(2)、目录项以ID号命名，ID号为0x80。
(3)、在目录项的二个字段可以得知该目录的子节点也是目录，偏移是0x268，`RVA = 0x2B000 + 0x268 = 0x2B268`。

**3、定位三级资源目录并解析：**

![资源表3](upload/attach/202002/813468_HW6Y7VVCA64J3F2.png)

(1)、可以得到资源目录头中一共有1个目录项。
(2)、目录项以ID号命名，ID号为0x804，表示使用简体中文的代码页。
(3)、在目录项的二个字段可以得知该目录的子节点是数据项，偏移是0x6E8，`RVA = 0x2B000 + 0x6E8 = 0x2B6E8`。

**4、定位数据项：**

![资源表4](upload/attach/202002/813468_X2M8G7YKYJZH56F.png)

(1)、资源数据的RVA地址：0x2C260.
(2)、资源数据的大小：0x680.

***

## 第十一节：资源表小练习

> 1.通过编写控制台程序，打印资源表的信息。(可以考虑递归实现)

> 2.在PE文件中创建一个新节，然后将资源表移动到新节中。最后将文件写入硬盘，并可以正确解析资源表。

**练习小提示:**

```
1、打印流程：
(1)定位资源表
(2)打印资源表目录头信息，并循环打印目录项。(一级目录)
(3)定位二级目录头，并打印资源表目录头信息，循环打印目录项。(二级目录)
(4)定位三级目录头，并打印资源表目录头信息，循环打印目录项。(三级目录)
(5)定位资源数据项，并打印资源数据项信息。
2、移动资源表的步骤：
　　(1)在PE文件中创建一个新节
　　(2)将资源表的一级目录全部copy到新节
　　(3)循环一级资源表信息，定位二级目录。
　　(4)将资源表的二级目录全部copy到指定位置。
　　(5)循环二级资源表信息，定位三级目录。
　　(6)将资源表的三级目录全部copy到指定位置。
　　(7)定位资源数据项，将资源数据项全部copy到指定位置。
　　(8)将资源数据copy到新节中，并修复数据项。
　　(9)修复目录项信息。
注：由于资源表都是通过首个资源目录头定位数据的，而且都是在资源目录头之后，所以可以直接按照目录项中的资源大小，将资源从首个资源目录头开始全部copy到新节中。但是不推荐这样完成。
```

***

# 第三章：打造自己的PE解析器——全文总结

> 文章写到这里，我们打造PE解析器的基础理论已经全部介绍完毕了，相信大家也都跟着文章完成了其中的练习题，那么接下来的事情就非常简单了，我们只要把所有的代码整合起来就可以实现一个简单PE解析器了，当然只是一个控制台程序。如果想要打造窗口程序就要具备一定的Windows编程经验了，然后将对应的内容输出出来就大功告成了。

> 希望同学们完成这最后一个任务！

写了一个月的文章完成，第一次发帖语言上有点生涩，如果文章有错误的地方希望能够指出，谢谢!

目录
- [前言](https://bbs.kanxue.com/#前言)
- [贯穿全文的概念](https://bbs.kanxue.com/#贯穿全文的概念)
- [第一章：打造自己的PE解析器——PE文件头结构](https://bbs.kanxue.com/#第一章：打造自己的pe解析器——pe文件头结构)
- [第一节：PE文件结构](https://bbs.kanxue.com/#第一节：pe文件结构)
- [第二节：IMAGE_DOS_HEADER](https://bbs.kanxue.com/#第二节：image_dos_header)
- 第三节：IMAGE_DOS_STUB (了解)
- [第四节：IMAGE_NT_HEADERS32](https://bbs.kanxue.com/#第四节：image_nt_headers32)
- [第五节：IMAGE_FILE_HEADER](https://bbs.kanxue.com/#第五节：image_file_header)
- [第六节：IMAGE_OPTIONAL_HEADER32](https://bbs.kanxue.com/#第六节：image_optional_header32)
- [第七节：IMAGE_SECTION_HEADER](https://bbs.kanxue.com/#第七节：image_section_header)
- [第八节：章节练习](https://bbs.kanxue.com/#第八节：章节练习)
- [第二章：打造自己的PE解析器——目录信息](https://bbs.kanxue.com/#第二章：打造自己的pe解析器——目录信息)
- [第一节：IMAGE_DATA_DIRECTORY——数据目录结构](https://bbs.kanxue.com/#第一节：image_data_directory——数据目录结构)
- [第二节：IMAGE_EXPORT_DIRECTORY——导出表](https://bbs.kanxue.com/#第二节：image_export_directory——导出表)
- [第三节：导出表小练习](https://bbs.kanxue.com/#第三节：导出表小练习)
- [第四节：IMAGE_BASE_RELOCATION——重定位表](https://bbs.kanxue.com/#第四节：image_base_relocation——重定位表)
- [第五节：重定位表小练习](https://bbs.kanxue.com/#第五节：重定位表小练习)
- [第六节：IMAGE_IMPORT_DESCRIPTOR——导入表](https://bbs.kanxue.com/#第六节：image_import_descriptor——导入表)
- [第七节：导入表小练习](https://bbs.kanxue.com/#第七节：导入表小练习)
- [第八节：IMAGE_BOUND_IMPORT_DESCRIPTOR——绑定导入表](https://bbs.kanxue.com/#第八节：image_bound_import_descriptor——绑定导入表)
- [第九节：绑定导入表小练习](https://bbs.kanxue.com/#第九节：绑定导入表小练习)
- [第十节：IMAGE_RESOURCE_DIRECTORY——资源表](https://bbs.kanxue.com/#第十节：image_resource_directory——资源表)
- [第十一节：资源表小练习](https://bbs.kanxue.com/#第十一节：资源表小练习)
- [第三章：打造自己的PE解析器——全文总结](https://bbs.kanxue.com/#第三章：打造自己的pe解析器——全文总结)

---

# 前言

> - **文章概述：**写本篇文章的宗旨就是同学可以完全通过阅读本篇文章打造一个自己的PE文件解释器。让同学们能够更加深入的理解并掌握PE文件的基本结构，少走一些弯路。
> - **阅读方法：**本篇文章的结构是一章理论加一章代码实现的结构，这样可以完美的达到理论与实践相结合的学习效果。同时也希望同学们在看完理论部分后可以动手实践完成本章的课后练习，在文章的附件中包含每一个练习的参考代码。文章的图片也会附加在附件当中。
> - **具备基础：**
>   (1)掌握C语言
> - **学习环境：**VS2017 或 VC++6.0，Windows操作系统，LordPE(PE解释器)

## 贯穿全文的概念

**地址空间：**这个地址空间指的是PE文件被加载到内存的空间，是一个虚拟的地址空间，之所以不是物理空间是因为数据在内存中的位置经常在变，这样既可以节约内存开支又可以避开错误的内存位置。这个地址空间的大小为4G，但其中供程序装载的空间只有2G而且还是低2G空间，高2G空间则被用于装载内核DLL文件，所以也被称作内核空间。

**文件映射：**PE文件在磁盘上的状态和在内存中的状态是不一样的，我们把PE文件在磁盘上的状态称作`FileBuffer`，在内存中的状态称为`ImageBuffer`。当PE文件通过装载器装入内存是会经过“拉伸”的过程，所以它在`FileBuffer`状态下和`ImageBuffer`状态下的大小是不一样的。这个拉伸的具体过程会在讲完PE头结构后进行介绍。大致的图解如下：

![PE拉伸](https://bbs.kanxue.com/upload/attach/202002/813468_WGD426WABYGFDKB.png)

**VA：**英文全称是`Virual Address`，简称VA，中文意思是虚拟地址。指的是文件被载入虚拟空间后的地址。

**ImageBase：**中文意思是基址，指的是程序在虚拟空间中被装载的位置。

**RVA：**英文全称是`Relative Virual Address`，简称RVA，中文意思是相对虚拟地址。可以理解为文件被装载到虚拟空间(拉伸)后先对于基址的偏移地址。计算方式：`RVA = VA(虚拟地址) - ImageBase(基址)`。它的对齐方式一般是以1000h为单位在虚拟空间中对齐的(传说中的4K对齐)，具体对齐需要参照`IMAGE_OPTIONAL_HEADER32`中的`SectionAlignment`成员。

**FOA：**英文全称是`File Offset Address`，简称FOA，中文意思是文件偏移地址。可以理解为文件在磁盘上存放时相对于文件开头的偏移地址。它的对齐方式一般是以200h为单位在硬盘中对齐的(512对齐)，具体对齐需要参照`IMAGE_OPTIONAL_HEADER32`中的`FileAlignment`成员。

![图解RVA和FOA](https://bbs.kanxue.com/upload/attach/201907/813468_4CFY4EN8FSRAU3K.png)

---

# 第一章：打造自己的PE解析器——PE文件头结构

> - **本章目的：**通过简单的讲解PE文件头结构及其基本概念，让刚开始学习PE的同学基本了解PE文件头的结构和相关理论知识，并通过学到的知识获取PE文件头的结构信息。
> - **重点掌握：**
>   (1)`IMAGE_DOS_HEADER`
>   (2)`IMAGE_NT_HEADERS32`
>   (3)`IMAGE_FILE_HEADER`
>   (4)`IMAGE_OPTIONAL_HEADER32`
>   (5)`IMAGE_SECTION_HEADER`

**废话太多了。。。开始正文！！！**

---

## 第一节：PE文件结构

![PE文件结构图](https://bbs.kanxue.com/upload/attach/202002/813468_PZCYNX9G8HN2YUB.png)

PE文件是由许许多多的结构体组成的，程序在运行时就会通过这些结构快速定位到PE文件的各种资源，其结构大致如图所示，从上到下依次是Dos头、Nt头、节表、节区和调试信息(可选)。其中Dos头、Nt头和节表在本文中统称为PE文件头(因为SizeOfHeaders就是这三个头的总大小)、节区则称为节,所以也可以说PE文件是由PE文件头和节组成。

PE文件头保存着整个PE文件的索引信息，可以帮助PE装载器定位资源，而节则保存着整个PE文件的所有资源。正因为如此，所以存在着这样的说法：头是节的描述，节是头的具体化。

---

## 第二节：IMAGE_DOS_HEADER

> `IMAGE_DOS_HEADER`的结构体定义如下：

```
typedef struct _IMAE_DOS_HEADER {        
    WORD e_magic;        **重要成员 相对该结构的偏移0x00**
    WORD e_cblp; 
    WORD e_cp;
    WORD e_crlc;
    WORD e_cparhdr;
    WORD e_minalloc;
    WORD e_maxalloc;
    WORD e_ss;
    WORD e_sp;
    WORD e_csum;
    WORD e_ip;
    WORD e_cs;
    WORD e_lfarlc;
    WORD e_ovno;
    WORD e_res[4];
    WORD e_oemid;
    WORD e_oeminfo;
    WORD e_res2[10];
    LONG e_lfanew;        **重要成员 相对该结构的偏移0x3C**
} IMAGE_DOS-HEADER, *PIMAGE_DOS_HEADER;
```

当我们用16进制编辑器打开一个PE文件时，就会发现所有PE文件的前两个字节都是`MZ`,用十六进制表示是`4D 5A`，这两个字母就是`Mark Zbikowski`的姓名缩写，他是最初的MS-DOS设计者之一。如果把PE文件的这两个字节修改成其他数据，运行该PE文件就会无法正常运行(跳出黑窗口打印`Program too big to fit in memory`然后闪退，有兴趣的朋友可以尝试下)。这里可以证明当PE文件运行时，首先就会检测这两个字节，如果不是`MZ`则会退出运行。

上图

![修改MZ后运行](https://bbs.kanxue.com/upload/attach/202002/813468_9YW6Q4YBMKRXRQH.png)

在该结构体中另一个重要成员就是最后一个成员`e_lfanew`。该成员的大小是`LONG`类型4个字节。之所以说它重要是因为它保存着`IMAGE_NT_HEADERS32`这个结构体在PE文件中的偏移地址，PE文件运行时只有通过该成员才能定位到PE签名(也就是`IMAGE_NT_HEADERS32`结构体的起始位置)。

咱们看图说话

![Dos头图解](https://bbs.kanxue.com/upload/attach/202002/813468_TRX7SVZGNCJ6KMP.png)

---

## 第三节：IMAGE_DOS_STUB (了解)
