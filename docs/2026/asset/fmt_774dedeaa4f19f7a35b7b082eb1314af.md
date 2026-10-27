
```import pefile
# 通过pefile模块读取PE文件
pe = pefile.PE('notepad.exe')
for exp in pe.DIRECTORY\_ENTRY\_EXPORT.symbols:
print hex(pe.OPTIONAL\_HEADER.ImageBase + exp.address), exp.name, exp.ordinal
```

## 附件说明

因为示例DLL序号表与函数入口地址表本身就是0、1、2、3对应,附件多给几个DLL进行对比。以下是附件DLL的说明与`dumpbin.exe`结果对比

请留意DLL序号表对应关系：

```【PEdemo.dll】
NumberOfFunctions 0x00000004 unsigned long
NumberOfNames 0x00000004 unsigned long
Base 0x00000001 unsigned long
AddressOfFunctions AddressOfNames -> name AddressOfNameOrdinal 实际上为(WORD)
E6 10 00 00 2B 7F 00 00 fnPEDemoFun 00 00
87 10 00 00 37 7F 00 00 fnPEDemoFunA 01 00
8C 10 00 00 44 7F 00 00 fnPEDemoFunB 02 00
38 91 00 00 51 7F 00 00 nPEDemo 03 00
dumpbin result:
0.00 version
1 ordinal base
4 number of functions
4 number of names
ordinal hint RVA name
1 0 000010E6 fnPEDemoFun
2 1 00001087 fnPEDemoFunA
3 2 0000108C fnPEDemoFunB
4 3 00009138 nPEDemo
【PEDemofnPEDemoFunB@2fnPEDemoFunA@3.dll(指定序号)】
NumberOfFunctions 0x00000004 unsigned long
NumberOfNames 0x00000004 unsigned long
Base 0x00000002 unsigned long
AddressOfFunctions AddressOfNames -> name AddressOfNameOrdinal 实际上为(WORD)
8C 10 00 00 2B 7F 00 00 fnPEDemoFun 02 00
87 10 00 00 37 7F 00 00 fnPEDemoFunA 01 00
E6 10 00 00 44 7F 00 00 fnPEDemoFunB 00 00
38 91 00 00 51 7F 00 00 nPEDemo 03 00
dumpbin result:
0.00 version
2 ordinal base
4 number of functions
4 number of names
ordinal hint RVA name
4 0 000010E6 fnPEDemoFun = @ILT+225(\_fnPEDemoFun)
3 1 00001087 fnPEDemoFunA = @ILT+130(\_fnPEDemoFunA)
2 2 0000108C fnPEDemoFunB = @ILT+135(\_fnPEDemoFunB)
5 3 00009138 nPEDemo = \_nPEDemo
结论：先找函数名称，查看序号，得到函数入口地址
【D:\\PEDemofnPEDemoFunB@2NONAME.dll(隐藏函数名以序号导出)】
NumberOfFunctions 0x00000004 unsigned long
NumberOfNames 0x00000003 unsigned long
Base 0x00000002 unsigned long
AddressOfFunctions AddressOfNames -> name AddressOfNameOrdinal 实际上为(WORD)
8C 10 00 00 25 7F 00 00 fnPEDemoFun 01 00
E6 10 00 00 31 7F 00 00 fnPEDemoFunA 02 00
87 10 00 00 3E 7F 00 00 nPEDemo 03 00
38 91 00 00
dumpbin result:
0.00 version
2 ordinal base
4 number of functions
3 number of names
ordinal hint RVA name
3 0 000010E6 fnPEDemoFun
4 1 00001087 fnPEDemoFunA
5 2 00009138 nPEDemo
2 0000108C [NONAME]
结论：查询函数入口地址的数组序号，序号加基数等于导出函数序号(ordinal)
【D:\\PEDemofnPEDemoFunB@2private.dll（private关键字）】
NumberOfFunctions 0x00000004 unsigned long
NumberOfNames 0x00000004 unsigned long
Base 0x00000002 unsigned long
AddressOfFunctions AddressOfNames -> name AddressOfNameOrdinal 实际上为(WORD)
8C 10 00 00 2B 7F 00 00 fnPEDemoFun 00 01
E6 10 00 00 37 7F 00 00 fnPEDemoFunA 00 02
87 10 00 00 44 7F 00 00 fnPEDemoFunB 00 00
38 91 00 00 51 7F 00 00 nPEDemo 00 03
dumpbin result:
0.00 version
2 ordinal base
4 number of functions
4 number of names
ordinal hint RVA name
3 0 000010E6 fnPEDemoFun
4 1 00001087 fnPEDemoFunA
2 2 0000108C fnPEDemoFunB
5 3 00009138 nPEDemo
结论：遍历函数入口地址得到序号，遍历名称个数获取对应的序号，得出函数入口地址对应的函数名称
```

## 5 参考

1、《黑客免杀攻防》-7.4

2、[原创]手查PE导出表

http://bbs.pediy.com/thread-205989.htm

3、Portable Executable

<mark class="encrypted">b32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6b7L8%4u0@1j5h3u0D9k6g2)9#2k6V1g2^5k6h3y4#2N6r3q4T1L8r3f1`.</mark>

4、PE文件的相对虚拟地址（RVA）和文件偏移地址（FOA）的转换

<mark class="encrypted">264K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3P5X3S2S2L8K6l9^5x3e0p5I4x3e0t1I4y4e0N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0b7J5x3e0V1J5z5o6R3I4</mark>

5、导出表一课

<mark class="encrypted">5a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2F1k6i4c8Q4x3V1k6U0L8%4g2J5M7$3g2Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3V1j5K6x3o6l9J5i4K6u0r3y4o6V1@1x3e0g2Q4x3@1k6S2N6i4c8G2i4K6g2X3M7%4c8S2M7Y4c8Q4x3@1b7I4</mark>

6、[原创]解析PE结构之-----导出表

http://bbs.pediy.com/thread-122632.htm

7、[原创]PE学习之手工解析导出表

http://bbs.pediy.com/thread-217229.htm

8、`IMAGE_DATA_DIRECTORY` 数据目录详解

<mark class="encrypted">ad9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6G2K9r3q4U0K9%4y4Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3X3b7#2z5q4)9J5k6h3S2@1L8h3H3`.</mark>

9、Windows Pe 第三章 PE头文件-EX-相关编程-2(RVA_FOA转换)

<mark class="encrypted">035K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6e0l9I4x3K6M7$3x3e0l9K6y4W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0t1%4y4e0p5%4x3U0p5`.</mark>

10、the-export-directory

<mark class="encrypted">e7eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2M7#2)9J5k6h3W2F1k6X3!0K6k6h3y4A6L8Y4y4@1K9i4c8#2N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2Q4x3X3c8W2P5s2m8G2M7Y4c8Q4x3X3c8V1K9i4u0W2j5%4c8G2M7Y4W2Q4x3V1j5`.</mark>

11、PE文件结构详解（三）PE导出表

<mark class="encrypted">29fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3k6i4k6A6L8r3g2S2k6$3I4W2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3U0p5%4y4U0M7&6y4H3`.`.</mark>

12、WindowsPE 第五章 导出表

<mark class="encrypted">300K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6e0l9I4x3K6M7$3x3e0l9K6y4W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3J5y4o6p5#2x3e0f1`.</mark>

13、小甲鱼PE详解之区块描述、对齐值以及RVA详解

<mark class="encrypted">c24K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9r3E0Q4y4h3j5#2y4K6R3^5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5@1z5o6t1J5y4e0l9H3y4H3`.`.</mark>

14、15PB学习参考资料

## 0 前言

此篇文章想写如何通过工具手查导出表、PE文件代码编程过程中的原理。文笔不是很好，内容也是查阅了很多的资料后整合出来的。希望借此加深对PE文件格式的理解，也希望可以对看雪论坛有所贡献。因为了解PE文件格式知识点对于逆向破解还是病毒分析都是很重要的，且基于对PE文件格式的深入理解还可以延伸出更多非常有意思的攻防思维。另外看雪能支持Markdown真是太好了了！发主题更加方便了

## 1 导出表查询工具

- 1 ) dumpbin

VS自带的工具，有很多的功能。但用来查询程序的导出表也非常方便，使用例子如下：

> `dumpbin.exe /EXPORTS D:\PEDemo.dll`

- 2 ) DLL Export Viewer (DLL导出表查看工具)

一款免费的dll查看工具，可以帮助查看DLL链接库文件中的输出函数，COM类型库及相应的偏移地址。

- 3）010 Editor

十六进制编辑器010 Editor，有大量格式的解析模板，用EXE模板来解析二进制文件，辅助我们读懂和编辑。

- 4）LordPE

LordPE是查看PE格式文件信息的工具，并且可以修改相关信息。里面有个位置计算器的功能可以用于计算相对虚拟地址(RVA)转换文件偏移地址(FOA)。

## 2 Windows导出表相关的结构

### 2.1 导出表所处位置

在Visual Studio里有一个名为`WINNT.H`的头文件，里面定义了Windows系统里的PE内部结构。

PE结构中有一个NT头（`IMAGE_NT_HEADERS` `NtHeader`），NT头里包含了扩展头（`IMAGE_OPTIONAL_HEADER`），扩展头中包含数据目录表（`IMAGE_DATA_DIRECTORY_ARRAY`）。

扩展头定义：

```
    typedef struct _IMAGE_OPTIONAL_HEADER {

    //

    // Standard fields.

    //



    ...省略....

    //

    // NT additional fields.

    //



    ....省略...



         IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];  //数据目录表

    } IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;

```

`IMAGE_NUMBEROF_DIRECTORY_ENTRIES`是个宏定义，值是0x16。

宏定义：

```
    #define IMAGE_NUMBEROF_DIRECTORY_ENTRIES 16

```

数据目录是一个有16个（`WINNT.H`中定义为`IMAGE_NUMBEROF_DIRECTORY_ENTRIES`）元素的结构数组。每个数组元素所指定的内容已经被预先定义好了。

`WINNT.H`文件中的这些`IMAGE_DIRECTORY_ENTRY_xxx`定义就是数据目录的索引（从0到15）。导出表相对虚拟地址(RVA)就在数据目录表中的第0个数组里。

数据目录表有两个结构体成员分别存有数据的相对虚拟地址和数据的大小，定义如下：

```
    typedef struct _IMAGE_DATA_DIRECTORY {



        DWORD   VirtualAddress; // 数据的相对虚拟地址（RVA）



        DWORD   Size;   // 数据的大小



    } IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;

```

下表描述了每个`IMAGE_DIRECTORY_ENTRY_xxx`值每个数组的意义。

![image](https://bbs.kanxue.com/upload/attach/202002/214974_BEG7VN3HX9K9739.jpg)

图1 `IMAGE_OPTIONAL_HEADER`中数据目录表结构体数组含义

### 2.2 导出表结构

下面是导出表的数据结构定义说明：
