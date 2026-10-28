
它根据自定的spec文件描述函数原型生成Lib。spec比起手写空函数体加配套[模块定义文件 (.def)](https://bbs.kanxue.com/elink@2b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.)要简单的多：

|  |  |
| --- | --- |
| 1 | `@ stdcall CreateProcessInternalW(ptr wstr wstr ptr ptr` `long` `long` `ptr wstr ptr ptr` `long``)` |

据此生成汇编源文件再生成Stub Dll+Lib。卷入汇编器总比卷入C编译器生成得快。

- [MASM32](https://bbs.kanxue.com/elink@323K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5i4y4E0x3K6u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.)的`inc2l`工具

   它根据汇编`inc`文件里的过程声明生成汇编源文件然后生成Stub Dll+Lib。

- [MSVC 库管理器 (`Lib.exe`)](https://bbs.kanxue.com/elink@a94K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3I4A6j5W2)9J5k6s2u0W2k6X3g2J5k6h3&6U0k6b7`.`.)

   它虽然可以根据[模块定义文件 (.def)](https://bbs.kanxue.com/elink@833K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.)直接生成Lib，但其生成导入符号的名称类型总指定为`IMPORT_OBJECT_NAME`，即表示导入名称与导入符号名称完全相同，不适用于x86下`stdcall`调用约定的API（此场景下应用`IMPORT_OBJECT_NAME_UNDECORATE`，表示导入名称为导入符号名称去除调用约定的修饰后的结果）。

比起以上已有的轮子还能进一步卷的，是做到能用[spec2def](https://bbs.kanxue.com/elink@c30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3q4U0N6r3!0K6i4K6u0r3M7X3g2S2j5%4c8G2M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6k6r3E0Q4x3V1k6@1L8$3!0D9M7#2)9J5c8Y4y4H3k6h3x3J5k6r3g2X3i4K6u0r3M7%4m8W2j5K6u0V1k6h3k6Q4x3X3g2U0)这样简洁的函数定义，还能不卷入汇编器/编译器这类代码生成工具，甚至还能实现一个Lib包含多个Dll的导出符号。不妨从文件结构开始，从零构建Lib。

## 二、 生成包含Dll导出符号的Lib

### 1. Lib文件结构

先祭出Microsoft Learn的祖传文档——[PE Format - Archive (Library) File Format - Win32 apps | Microsoft Learn](https://bbs.kanxue.com/elink@16aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6V1k6h3u0#2k6#2)9J5c8Y4m8W2i4K6u0V1k6X3!0J5L8h3q4@1i4K6t1K6j5i4u0U0K9r3W2$3k6g2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3X3c8X3K9h3I4W2i4K6u0V1k6X3!0J5L8h3q4@1)。

我已将踩过的坑对应的文档错漏提交给[Microsoft Docs](https://bbs.kanxue.com/elink@be1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6)并得到合入([PR #1555](https://bbs.kanxue.com/elink@74cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0f1#2y4b7`.`.), [PR #1586](https://bbs.kanxue.com/elink@173K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0f1^5y4R3`.`.), [PR #1622](https://bbs.kanxue.com/elink@fe9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0j5J5x3R3`.`.))，此外需要注意的本文会提及。与某些单纯复制或者翻译微软文档就拿来发的博客相比，算是实践出真知了。

**Lib (Archive) 文件主要包含导入符号（Import Symbol），供链接器链接。这些符号可以指向外部Dll的导出，也可以存在于Lib自身夹带的Obj文件里。**

整体布局比较简单。开头固定的文件签名后，一系列成员（Archive Member）依次排列，每个成员都以成员头(`IMAGE_ARCHIVE_MEMBER_HEADER`)开始，后跟成员数据。

![](https://bbs.kanxue.com/upload/attach/202307/853701_WVNE465XHGGBXJM.png)

- Archive Member Header (成员头)

   Lib文件中所有成员都以`IMAGE_ARCHIVE_MEMBER_HEADER`结构体开头，实际有用且需要的只有`Name`和`Size`字段，分别表示名称和大小。`Name`也用以指示为上表签名后的三个特殊成员，如果名称不够16字节，剩余的用空格`' '`补齐，如果超过16字节（去除末尾固定的`'/'`只剩15字节），则名称字符串存放到 Longnames Member （长名称成员）里，`Name`字段指示其所在偏移。`Size`表示成员数据大小，不包含成员头自身大小60（`IMAGE_SIZEOF_ARCHIVE_MEMBER_HDR`或`sizeof(IMAGE_ARCHIVE_MEMBER_HEADER)`）字节，也不包含用于对齐的`'\n'`。

- 1st Linker Member 与 2nd Linker Member

   作用和结构都类似，记录符号的名称和所在文件偏移供链接器定位，信息上是冗余的。前者Unix Style，后者MS Style，并且后者按名称排序使得按名称查找符号更高效，MS链接器（[Link.exe](https://bbs.kanxue.com/elink@e9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3I4A6L8X3E0A6L8X3M7`.)）使用。

- Longnames Member

   后续各符号的成员名如果超长，则名称形式不再是`"xxxx/"`（16字节放不下），而是`"/n"`（`n`即为该名称字符串所在Longnames Member内容中的偏移）。

- Symbol

   Lib包含的各个符号实际内容。前面说过，可以是指向外部Dll的导出，也可以是夹带的Obj。

  - 指向外部Dll的导出

     成员名称为Dll名称，如`"KERNEL32.dll"`，内容为Import Header+导入符号名字符串+Dll名称字符串。虽然Dll名称重复出现，但后续MS链接器生成导入表时会分别使用，这两处不同或者有误会导致生成的导入表奇怪或不正确。要注意调用约定、Dll导出名、导入符号名的关系，内容示例：

    | 平台 | DLL导出名 | Import Header | 导入符号名 |
    | --- | --- | --- | --- |
    | x86 | CreateProcessInternalW | Type=`IMPORT_OBJECT_CODE`<br/>Name Type=`IMPORT_OBJECT_NAME_UNDECORATE` | `_CreateProcessInternalW@48` |
    | x64 | CreateProcessInternalW | Type=`IMPORT_OBJECT_CODE`<br/>Name Type=`IMPORT_OBJECT_NAME` | `_CreateProcessInternalW` |

    此外，还要同时导出带`"__imp_"`前缀的符号供链接器使用，避免调用外部Dll导出时生成不必要的Jmp Stub指令。即`_CreateProcessInternalW@48`与`__imp__CreateProcessInternalW@48`都指向同个Import Header。

  - 夹带Obj

     成员名称为Obj在Lib中的全路径，如果在根目录，等同于Obj文件名。内容直接为Obj文件内容。如果多个Lib导入符号都在同一个夹带的Obj内，则都指向同个Symbol，链接器会在这个Symbol夹带的Obj里寻找它们。

- 其它注意事项

  - 每个成员（Archive Member Header）所在偏移要2字节对齐，如果成员大小为奇数，则在其后插入`'\n'`(`IMAGE_ARCHIVE_PAD`)，使紧随其后的成员所在偏移得以对齐。插入的`'\n'`不计入成员大小。

  - 数值有的用十进制字符串表示，有的用大端存储，MS Style的2nd Linker Member里各数值用小端存储，要看清文档说明。

### 2. 用于链接Dll的Lib

按照上述基础的Lib文件结构，构造包含Dll导出符号（如`_CreateProcessInternalW@48`与`__imp__CreateProcessInternalW@48`）的Lib可供链接器链接，但MS链接器（[Link.exe](https://bbs.kanxue.com/elink@c1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3I4A6L8X3E0A6L8X3M7`.)）生成的二进制导入表却缺失了对应Dll的IID（`IMAGE_IMPORT_DESCRIPTOR`）。

想必从Windows SDK里最小的Dll导入Lib找线索更容易。用[7-Zip](https://bbs.kanxue.com/elink@c99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5%4i4K6u0V1P5X3W2H3i4K6u0W2L8%4u0Y4i4K6u0r3)打开SAS.lib里的1.txt（也就是1st Linker Member），看到其包含符号如下：

|  |  |
| --- | --- |
| 1  2  3  4  5 | `1.SAS``.dll    __IMPORT_DESCRIPTOR_SAS`  `2.SAS``.dll    __NULL_IMPORT_DESCRIPTOR`  `3.SAS``.dll    SAS_NULL_THUNK_DATA`  `4.SAS``.dll    _SendSAS@``4`  `4.SAS``.dll    __imp__SendSAS@``4` |

可见，除了包含Dll导出符号（`_SendSAS@4`与`__imp__SendSAS@4`），还有：

1. Dll的IID符号（`__IMPORT_DESCRIPTOR_SAS`）

2. Dll的空Thunk（`SAS_NULL_THUNK_DATA`）

3. 导入表末尾结束的空IID（`__NULL_IMPORT_DESCRIPTOR`）

它们分别位于3个不同夹带的Obj里，MS链接器（[Link.exe](https://bbs.kanxue.com/elink@a38K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3I4A6L8X3E0A6L8X3M7`.)）构造导入表时使用它们。

在一个用于链接Dll的Lib里`__NULL_IMPORT_DESCRIPTOR`只需要有1个，所以存放在单独的一个Obj会比较好。成员名无所谓，不用保留名称即可。

前二者名称分别为`"__IMPORT_DESCRIPTOR_[Dll名]"`和`"[Dll名]_NULL_THUNK_DATA"`（**注意开头字符为`'\x7F'`**），可以放在同个Obj里。如果Lib包含多个Dll的导出符号，它们也要对应地有多个，成员名也是Dll的名称。所在Obj需要有`__NULL_IMPORT_DESCRIPTOR`的外部引用记录，否则链接Dll时`__NULL_IMPORT_DESCRIPTOR`可能未被主动引入，导致的PE映像导入表没能以空IID结尾。

与Windows SDK的Lib里的Obj相比，我们可以去除其中的调试符号，也可以简化3个Obj之间的复杂关系。

### 3. 代码实现

我的代码实现：

[KNSoft/C4Lib - C4Lib/Source/PEImage/ObjectFile.cs](https://bbs.kanxue.com/elink@a33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6o6y4p5I4A6j5W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5h3W2F1i4K6u0r3f1$3!0#2M7X3y4W2i4K6u0r3f1p5g2u0L8h3q4Y4k6g2)9J5c8V1!0T1K9X3g2U0N6p5k6A6L8r3g2Q4x3X3g2U0M7H3`.)中`NewNullIIDObject`方法用于构造`__NULL_IMPORT_DESCRIPTOR`所在Obj，`NewDllImportStubObject`方法用于构造`__IMPORT_DESCRIPTOR_XXX`与`XXX_NULL_THUNK_DATA`所在Obj。
