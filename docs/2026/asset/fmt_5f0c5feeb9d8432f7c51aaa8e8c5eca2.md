
```cpp
struct DexFile {

501    /* directly-mapped "opt" header */

502    const DexOptHeader* pOptHeader;

503

504    /* pointers to directly-mapped structs and arrays in base DEX */

505    const DexHeader*    pHeader;

506    const DexStringId*  pStringIds;

507    const DexTypeId*    pTypeIds;

508    const DexFieldId*   pFieldIds;

509    const DexMethodId*  pMethodIds;

510    const DexProtoId*   pProtoIds;

511    const DexClassDef*  pClassDefs;

512    const DexLink*      pLinkData;

513

514    /*
515     * These are mapped out of the "auxillary" section, and may not be
516     * included in the file.
517     */

518    const DexClassLookup* pClassLookup;

519    const void*         pRegisterMapPool;       // RegisterMapClassPool

520

521    /* points to start of DEX file data */

522    const u1*           baseAddr;

523

524    /* track memory overhead for auxillary structures */

525    int                 overhead;

526

527    /* additional app-specific data structures associated with the DEX */

528    //void*               auxData;

529};
```

`ART` 下 `DexFile` 类的代码较长，只贴出片段：

```cpp
class DexFile {

 public:

  static const uint8_t kDexMagic[];

  static const uint8_t kDexMagicVersion[];

  static constexpr size_t kSha1DigestSize = 20;

  static constexpr uint32_t kDexEndianConstant = 0x12345678;

  // name of the DexFile entry within a zip archive

  static const char* kClassesDex;

  // The value of an invalid index.

  static const uint32_t kDexNoIndex = 0xFFFFFFFF;

  // The value of an invalid index.

  static const uint16_t kDexNoIndex16 = 0xFFFF;

  // The separator charactor in MultiDex locations.

  static constexpr char kMultiDexSeparator = ':';

  // A string version of the previous. This is a define so that we can merge string literals in the
  // preprocessor.

  #define kMultiDexSeparatorString ":"

  // Raw header_item.

  struct Header {

    uint8_t magic_[8];

    uint32_t checksum_;  // See also location_checksum_

    uint8_t signature_[kSha1DigestSize];

    uint32_t file_size_;  // size of entire file

    uint32_t header_size_;  // offset to start of next section

    uint32_t endian_tag_;

    uint32_t link_size_;  // unused

    uint32_t link_off_;  // unused

    uint32_t map_off_;  // unused

    uint32_t string_ids_size_;  // number of StringIds

    uint32_t string_ids_off_;  // file offset of StringIds array

    uint32_t type_ids_size_;  // number of TypeIds, we don't support more than 65535

    uint32_t type_ids_off_;  // file offset of TypeIds array

    uint32_t proto_ids_size_;  // number of ProtoIds, we don't support more than 65535

    uint32_t proto_ids_off_;  // file offset of ProtoIds array

    uint32_t field_ids_size_;  // number of FieldIds

    uint32_t field_ids_off_;  // file offset of FieldIds array

    uint32_t method_ids_size_;  // number of MethodIds

    uint32_t method_ids_off_;  // file offset of MethodIds array

    uint32_t class_defs_size_;  // number of ClassDefs

    uint32_t class_defs_off_;  // file offset of ClassDef array

    uint32_t data_size_;  // unused

    uint32_t data_off_;  // unused

   private:

    DISALLOW_COPY_AND_ASSIGN(Header);

  };

  /

........
```

可以看到，`ART` 下 `DexFile` 类中定义了两个关键的变量：`begin_`、`size_`，以及用于获取这两个变量的 `Begin()` 和 `Size()` 函数。这两个变量分别代表着当前 `DexFile` 对象对应的内存中的 dex 文件加载的起始位置和大小。当然也有其他的方法来获取到内存中的 dex 加载的起始位置和大小。可以说，只要有了这两个值，我们就可以完成对这个 dex 的 dump。

## 二、ART 下基于关键字的快速定位脱壳点方法

在上一小节，我对当前 Android 加固 app 脱壳的本质进行了总结。其中，ART 下影响脱壳的关键的一个类就是 `DexFile`，那么我们便可以围绕这个类，实现在 Android 庞大的系统源码中快速定位脱壳点，从而能够找到“海量”的脱壳点。这里再总结给出两种快速定位脱壳点的方法。

### 1. 直接查找法

这里的直接查找法就是指以 `DexFile` 为关键字，在庞大的源码库中检索定位可能的脱壳点。如参数中出现 `DexFile` 类型的、返回值为 `DexFile` 类型的、函数流程中出现 `DexFile` 类型的源码位置。在获取到 `DexFile` 对象以后，然后再通过该对象的 `Begin()` 和 `Size()` 函数获取到该 `DexFile` 对象对应的内存中的 dex 的起始地址和大小即可进行 dex 的 dump。如下图的检索过程，可以轻松定位出网上公开的基于 `dex2oat` 流程进行脱壳的脱壳点，同时，也可以看到那个脱壳点只是 `dex2oat` 流程中的一个脱壳点而已。

![](https://bbs.kanxue.com/upload/attach/201909/632473_WW9YVHSCS69M4PY.png)

### 2. 间接查找法

间接法就是指以 `DexFile` 为出发点，寻找能够间接获取到 `DexFile` 对象的。如通过 `ArtMethod` 对象的 `getDexFile()` 获取到 `ArtMethod` 所属的 `DexFile` 对象的这种一级间接法等；然后再在海量源码中以 `ArtMethod` 为关键字进行检索，检索那些参数中出现 `ArtMethod` 类型的、返回值为 `ArtMethod` 类型的、函数流程中出现 `ArtMethod` 类型的源码位置。

再比如通过 `Thread` 的 `getCurrentMethod()` 函数首先获取到 `ArtMethod`，或者通过 `ShadowFrame` 的 `getMethod` 获取到 `ArtMethod` 对象，然后再通过 `getDexFile` 获取到 `ArtMethod` 对象所属的 `DexFile` 的二级间接法；以及通过 `ShadowFrame` 对象的 `GetMethod()` 函数获取到当前栈中执行的 `ArtMethod` 对象，然后再获取 `DexFile` 对象等二级间接法。

好了，上面已经给出了如何快速在庞大的源码库中定位可能的脱壳点的方法了，大家就可以开始自己的“挖宝”行动了！接下来，便进入了本文的彩蛋部分了。

## 三、彩蛋：送出一个未公开的脱壳点

### 1. 原理分析

众所周知，`ART` 下引入了 `dex2oat` 来对 dex 进行编译，生成每一个 Java 函数对应的 native 代码，来提高运行效率。但是，`dex2oat` 并不是对 dex 中的所有函数进行编译。通过对 `dex2oat` 的源码进行分析，最终可以到达 `CompilerDriver` 类的 `CompileMethod` 函数，可以看到 `dex2oat` 对 dex 进行编译的过程中是按照函数粒度进行编译的。
