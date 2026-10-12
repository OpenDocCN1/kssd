
# 逆向工程文档排版

## 1. 案例解析：hello.exe 的章节数据

已 `hello.exe` 为例，我们查表，看到 `205c` 落入 `.rdata` 节，该节的虚拟地址（就是内存地址）`0x2000` 对应文件偏移 `0600`，那么 `205c`，则对应文件偏移的 `065c`。用 `ultraedit` 观看，如下图示。跟 `ollydbg` 看到的内容一样。

```
0000650: 0000 0000 5c20 0000 0000 0000 9d01 4d65  ....\ ........Me
0000660: 7373 6167 6542 6f78 4100 7573 6572 3332  ssageBoxA.user32
0000670: 2e64 6c6c 0000 8000 4578 6974 5072 6f63  .dll....ExitProc
0000680: 6573 7300 6b65 726e 656c 3332 2e64 6c6c  ess.kernel32.dll
```

**代学生**：顺便问一下，那些查虚拟地址到文件偏移转换的工具是这样查的吗？

**代老师**：是的。

### 问 3.29

`.rdata` 节中大部分数据都明白了，但从 `610-65d` 那一段数据是干什么的？

### 答 3.29

这段数据，当然也是动态加载用的。

前面讲的 `link` 与 `loader` 对话，确实如上所说，但那只是问题的一半，还有一半就是，当 `loader` 把程序加载到内存，它要修改数据，它怎样找到修改数据的地址，也就是说，那个存储着 `RVA 205c` 的地址 `00402008` `loader` 是怎样得到的？还有，它怎么知道 `MessageBoxA` 在 `user32.dll` 里面？

### 问 3.30

平时都是我问，现在忽然被反问。翻翻前面讲的 。。。

啊！ 是 `2.18` 时提出来的，程序要 `call 41c`, `41c` 要向 `402008` 地址所装的内容处跳。

哦，这是我们的读法。`loader` 是不会这么读的，`loader` 是死的，`loader` 是一段程序，它只会干机械的事。它怎么找到 `402008` 的，它怎么知道 `MessageBoxA` 在 `user32.dll` 里面？ 还是听你讲吧。

### 答 3.30

讲清这个问题，我们还要再看 `option header`. 坚持住，动态加载导入部分也就差这一点点了。

在 `1.2` 中提到 `option header` 的数据结构，在他的底部有一个成员。

```
IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
```

看着眼晕，还不如写简单点,它的数组就是 16 个元素

```
IMAGE_DATA_DIRECTORY DataDirectory[16];
```

前面那个结构叫数据目录，如下定义

```
//
// Directory format.
//

typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   RelativeVirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

挺简单的，其实就是 8 个字节。 16 个目录吗，就是 `16*8 = 108` 字节。占 8 整行

别担心，大部分没有用，微软在这里搞捉迷藏。我们只关心几个就行了。

这里首先介绍的一个叫导入表(import table)。

```
0000130:          0000 0000 0000 0000   ................
0000140: 1020 0000 3c00 0000 0040 0000 a003 0000   . ...<....@......
0000150: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0000160: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0000170: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0000180: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0000190: 0000 0000 0000 0000 0020 0000 1000 0000   . . . . . . . . . . . .
00001a0: 0000 0000 0000 0000 0000 0000 0000 0000   ................
00001b0: 0000 0000 0000 0000 2e74 6578 7400 0000   . . . . . . . . . . . .
```

为什么还留下那个 `.text`, 一看就知道，`.text` 代表的是 `section header table` 的开始地址

其上的 8 个整行就是 16 个目录了。真要让你像计算机一样从上到下数偏移，我们还真不行，找 `ascii` 字，我们还在行。

第一个目录叫导出表，这里为空

第二个目录叫导入表，这里虚拟地址是 `0x2010`, 大小是 `3c`.

`2010` `RVA` 对应的文件偏移是 `610` （算法我想你已经掌握了,看 `3.28`）

第三个目录叫资源表。`RVA=0x4000`, `size=0x3a0` (暂时先不讨论）

第十三个目录叫 `IAT` 表，英文全称为：`Import Address Table`. `RVA=0x2000`,`size=0x10`

其它的目录都是空的，没用，不理它们。

**代老师**：哦，是不是讲的有点多了？

**代学生**： 还行，反正讲得挺清楚的。想不到这个小小的 `hello.exe` 还有这么多事情。

不过再看看 `hello.exe` 文件，大部分内容都讲了，我想也不会有太多东西了。

**代老师**：正是，坚持一下就是胜利。

## 2. 深入解析：导入表与 IAT

接问题 3.30 后的第 31 个问题

### 问 4.31：

上回谈到目录项，有两个目录项要关心。

1. 导入表：`RVA=0x2010`, `size=0x3c`
2. 导入地址表（俗称 `IAT)`: `RVA=0x2000`,`size=0x10`

具体什么用途呢？

### 答 4.31：

先看 `IAT` 表，`loader` 要做的工作是把 `RVA=0x2000`,`size=0x10` 这么大范围的 `DWORD` 都要重新定址。如果是 `00 00 00 00` 就不用了。

对 `hello.exe` 而言文件偏移是：（今后 `RVA` 和 `OFFSET` 的转换我就不再提了）

```
0000600: 7620 0000 0000 0000 5c20 0000 0000 0000   v . . . . . . \ . . . .
```

再看导入表 `RVA=0x2010`, `offset=610`. `size=0x3c`,内容如下：

```
0000610: 5420 0000 0000 0000 0000 0000 6a20 0000   T . . . . . . . j . .
0000620: 0820 0000 4c20 0000 0000 0000 0000 0000   . . . L . . . . . . .
0000630: 8420 0000 0020 0000 0000 0000 0000 0000   . . . . . . . . . . .
0000640: 0000 0000 0000 0000 0000 0000
```

这里是一个结构数组。该结构叫导入表的描述符

```
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;            // 0 for terminating null import descriptor
        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    };
    DWORD   TimeDateStamp;                  // 0 if not bound,
    DWORD   ForwarderChain;
    DWORD   Name;
    DWORD   FirstThunk;                     // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
```

每个结构包含 5 个 `DWORD` 成员,最后一个结构，成员全是 0

结构中第二个 `TimeDateStamp`，第三个成员 `ForwarderChain` 不用关心，它们总是 0

`name` 变量：

* 第一个结构中为 `206A`, 对应文件偏移 `66A`, `"user32.dll"`
* 第二个结构中为 `2084`, 对应文件偏移 `684`  `"kernel32.dll"`

`FirstThunk` 地址：

* 第一个结构中为 `2008`, 对应文件偏移 `608`, //我们看到这个就是 `IAT` 表的内容了
* 第二个结构中为 `2000`, 对应文件偏移 `600`，//我们看到这个就是 `IAT` 表的内容了

`OriginalFirstThunk` 地址：

* 第一个结构中为 `2054`, 对应文件偏移 `654`, //`654` 与 `608` 存储的内容是一样的
* 第二个结构中为 `204c`, 对应文件偏移 `64c`，//`64c` 与 `600` 存储的内容是一样的

怪不得一个叫 `first thunk`, 一个叫 `OriginalFirstThunk`，原来所指地址处包含的内容是一样的.

```
0000640: 0000 0000 0000 0000 0000 0000 7620 0000   . . . . . . . . v . .
0000650: 0000 0000 5c20 0000 0000 0000 9d01 4d65   . . . . \ . . . . . . . M e
0000660: 7373 6167 6542 6f78 4100 7573 6572 3332   s s a g e B o x A . u s e r 3 2
0000670: 2e64 6c6c 0000 8000 4578 6974 5072 6f63   . d l l . . . . E x i t P r o c
0000680: 6573 7300 6b65 726e 656c 3332 2e64 6c6c   e s s . k e r n e l 3 2 . d l l
```

导入描述符有 2 个，说明它有两个 `DLL` 要导入，从 `Name` 能看出来，第一个叫`"user32.dll"`, 第二个叫`"Kernel32.dll"`

描述符的 `FirstThunk`, 指向一个导入函数地址数组，就叫 `thunk data` 数组吧，该数组最后一项为 `00000000`

描述符的 `OriginalFirstThunk` ，虽然与 `FirstThunk` 所指地址不同，但该地址所包含的内容却一样。

以第一个结构为例：

这个 `DWORD` 数组为 `5c20 0000 0000 0000` ，这个数组只有两项，去掉末尾标志项，只有一项，说明只有一个函数要导入。怎么导入，在这个地方，我们以 `0x205c` 为例， 把 `3.23` 中叙述过的导入过程补充完整。

1. `loader` 从目录第二项得到 `Import Table(导入表）`
2. 导入表是一个导入描述符结构数组，以第一个结构为例
   * 从名字项中它知道，这个动态库的名字叫 `"user32.dll"` 。
3. `first thunk` 指向 `thunnk data` 数组，是一个以全 0 结尾的 `DWORD` 数组，
   * 非全 0 的元素或者是一个 `RVA`(最高位为 0），指向一个 `Import_BY_NAME` 结构，
   * 或者是一个 `Ordinal`(序号，（其最高位为 1，使用时把最高位去掉就成序号啦）。
   * `first thunk` 指针属于 `IAT` 表的地址范围。
   * `hello.exe` 中 import 第一个描述符 `first thunk` 表偏移是 `608`，`608` 处存 `205c`（`RVA)`
   * 对应 `Import_BY_NAME` `MessageBoxA`, 下一个 `DWORD=0` 数组就结束了
   * 说明 `user32.dll` 只导入了一个函数。

从上面分析可以看出。

* 目录项中的导入地址表：指向一个大的 `IAT`。
* 目录项中的导入表：指向一个导入描述符数组。每一个描述符，描述一个 `DLL`。 而 `FirstThunk` 指向那个大的 `IAT` 中的一个部分。我们称它为 `thunk data` 数组，数组最后一个元素 `DWORD` 为全 0.

这样的一个设计结构，导入表和导入地址表的信息是有冗余的。我们不管它，知道就行了。

### 问：4.32

既然有了 `FirstThunk`， 还要 `OriginalFirstThunk` 干什么？ 它们指向的 `IAT` 内容都是一样的。

### 答：4.32

`FirstThunk` 所指向的 `IAT`, 会在加载时被 `loader` 修改为真实的函数地址。而 `OriginalFirstThunk` 所指向的 `IAT` 是不会被修改的 。其实我也认为，留着这个 `OriginalFirstThunk` 没有用途，可能是微软认为那个 `IAT` 已经被修改了， 万一你要再用你到拿找哇。 其实我看这是多虑了， 第一 , `IAT` 用完了我不会再用它。 第二, 万一被我们改了回头又要用，我们可以先把没改之前的备份一下呀。

不过，这都是个人意见，人家这么定义了，我们知道就行了。

### 问：4.33

从运行的角度来看，好像没有问题了，`.text` 段依靠`.rdata` 段的帮助，由 `loader` 修改为可执行代码。代码执行时读取或存入数据段数据。`hello` 就可以运行了。

那`.rsrc` 段是什么呢 ？ 是资源段吗？

### 答：4.33

是的。 本来我是没想加这个资源段的。可是不小心给加上了。这个其实你可以不用关心它的。

解开附件 `hello.rar`, 图标是一个笑脸，这就是那个资源段的功能。如果没有那个资源段。默认的图标是一个 `WINDOWS ICON`.

### 问：4.34

要想显示需要的图标，需要我们做什么呢？

### 答：4.34

你只要把 `ICON` 资源文件连进去就行了，代码并不需要做任何事情。在创建 `MessageBox` 窗口时，系统发现你文件里有一个 `ICON` 资源项，就替你把它画出来，如果你没有包含 `ICON` 资源，它就把自己手边的那个叫默认 `ICON`,画到 `MessagBox` 左上角了。

### 问：4.35

`hello.exe` 文件都分析完了，只是 `option header` 中还有一些项没有提到，它们重要吗？

### 大：4.35

有些项还是很重要的，例如程序入口点，但它们都已经很好理解了。

这里，我就把 `option` 做一个标注，此时再浏览 `option header` 的各个项，已经是时候了。

```
//
// Optional header format.
//

typedef struct _IMAGE_OPTIONAL_HEADER {
    //
    // Standard fields.
    //

    WORD    Magic;      // NT HDR32 定义为 010b
    BYTE    MajorLinkerVersion;    //link version 不重要
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;      //代码段大小
    DWORD   SizeOfInitializedData;  //初始化数据大小
    DWORD   SizeOfUninitializedData;  //未初始化数据大小
    DWORD   AddressOfEntryPoint;  //程序入口点：  重要
    DWORD   BaseOfCode;      //代码基址（RVA)
    DWORD   BaseOfData;      //数据基址(RVA)

    //
    // NT additional fields.
    //

    DWORD   ImageBase;      //模块基址
    DWORD   SectionAlignment;    //内存对齐调整
    DWORD   FileAlignment;    //文件对齐调整
    WORD    MajorOperatingSystemVersion;  //版本信息，不重要
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;    //模块的大小
    DWORD   SizeOfHeaders;    //header 的大小
    DWORD   CheckSum;      //未使用
    WORD    Subsystem;      //02 为 gui, 03 是 console
    WORD    DllCharacteristics;    //dll 用
```
