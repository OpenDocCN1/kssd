
# 学习 OBJ 文件格式

> 原文：[`www.pediy.com/kssd/pediy11/121556.html`](https://www.pediy.com/kssd/pediy11/121556.html)

PE 文件格式研究的人越来越多了，`obj` 文件格式貌似一直是冷门，跟大家分享学习成果。

如有错误之处，还请大家指出，以免误导他人。

## 学习 Obj 文件格式

说明：此 Obj 是指用我们 `VC` 或者 `masm` 等编译器编译源文件（`.cpp` `.c` `.asm`）产生的 `COFF`(Common Object File Format)格式的 Obj 目标文件。而非网上漫天飞的什么 3d 的什么玩意，那个咱不懂~

### （一）整体构架

`IMAGE_FILE_HEADER`

`IMAGE_SECTION_HEADER`(多个)

各个段的内容和重定位信息头（这个重定位信息头不一定每个段都有，貌似只有`.text` 段才有，这个不太确定），此部分的偏移地址是由`section` 头指定的。

`Symbol` 表(字符串表，`symbol` 表重定位字符串数据的时候会用到)

### （二）详解

① `IMAGE_FILE_HEADER` 结构，从名字也猜个八九不离十

```c
typedef struct _IMAGE_FILE_HEADER {

    WORD    Machine;        //一般我们的 intel cpu 是 0x014c

    WORD    NumberOfSections;     //section 段个数

    DWORD   TimeDateStamp;        //一般是此文档生成的时间

    DWORD   PointerToSymbolTable;   //symbol table 表的文件偏移地址（注意：本文的偏移一律指绝对偏移，RVA 的概念在 obj 文件里用不到）

    DWORD   NumberOfSymbols;     //symbol 表个数

    WORD    SizeOfOptionalHeader;    //0 no used in obj file

    WORD    Characteristics;         //0no used in obj file

} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

② `IMAGE_SECTION_HEADER` 结构，应该非常熟悉才对，分析 `PE` 的基础

```c
typedef struct _IMAGE_SECTION_HEADER {

    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];

    union {

        DWORD   PhysicalAddress;

        DWORD   VirtualSize;

    } Misc;            //no used in obj file

    DWORD   VirtualAddress;

    DWORD   SizeOfRawData;    //大小

    DWORD   PointerToRawData;    //偏移

    DWORD   PointerToRelocations;  //relocations 头 文件偏移

    DWORD   PointerToLinenumbers;  //unknown

    WORD    NumberOfRelocations;  //recolations 个数

    WORD    NumberOfLinenumbers;  //unkonwn

    DWORD   Characteristics;      //section 属性

} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

③ 重定位信息头 `IMAGE_RELOCATION`

```c
typedef struct _IMAGE_RELOCATION {

    union {

        DWORD   VirtualAddress;

        DWORD   RelocCount;         //

    };  //指相对于 section 内容的偏移地址

DWORD   SymbolTableIndex; //symbol 表的索引

WORD    Type;  //这个指 类型我云里雾里的 暂时没搞明白

//大侠知道的指点一下

//0x0060 代表 DIR32，0x0014 代表 REL32 具体是、//啥自己体会吧 我体会不好 就不误导别人了

} IMAGE_RELOCATION;

typedef IMAGE_RELOCATION UNALIGNED *PIMAGE_RELOCATION;
```

④ `IMAGE_SYMBOL` 结构，前面不是说过一个`symbol` 表的索引吗，就是指这个的索引

```c
typedef struct _IMAGE_SYMBOL {

    union {

        BYTE    ShortName[8];

        struct {

            DWORD   Short;     // if 0, use LongName

            DWORD   Long;      // offset into string table

        } Name;

        PBYTE   LongName[2];

} N;  //存储一个字符串。如果太长了就令前 4 字节为 0，//后 4 字节保存一个偏移来指向字符串首地址，这//个偏移起始地址是 string table

    DWORD   Value;          //通常为 0

    SHORT   SectionNumber;      //指向哪个 section，1 为第一个

    WORD    Type;          //这个说不明白

    BYTE    StorageClass;      //这个也很就麻烦不说了

    BYTE    NumberOfAuxSymbols;  //指示后面还有几个本结构大小的 buffer 存储数据

} IMAGE_SYMBOL;

typedef IMAGE_SYMBOL UNALIGNED *PIMAGE_SYMBOL;
```

说明：我并没有找到哪一个结构指示了`string table` 的文件偏移，那么我们姑且认为`string table` 的起始文件偏移为`PointerToSymbolTable + NumberOfSymbols*sizeof(IMAGE_SYMBOL)`

参考：

`http://www.mouseos.com/assembly/07.html`（他写的比我好多啦）

By 看雪`blueapplez`

2010 年 10 月 3 日 9:35:53

排版部太好，貌似这样看很吃力，还是下载附件看吧

### 上传的附件

| `img/58e0728fcb44eb7297cf87d68539ed4e.jpg` | `obj.doc` |
