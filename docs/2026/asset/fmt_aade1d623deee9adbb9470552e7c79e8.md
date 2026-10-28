
```java
if (elfFile.file\_type == ElfFile.FT\_DYN) { // not executable
int init = dynamicStructure.getInit();
if (init != 0) {
initFunctionList.add(new LinuxInitFunction(load\_base, soName, init));
//new LinuxInitFunction(load\_base, soName, init).call(emulator);
}
int initArraySize = dynamicStructure.getInitArraySize();
int count = initArraySize / emulator.getPointerSize();
if (count > 0) {
Pointer pointer = UnidbgPointer.pointer(emulator, load\_base + dynamicStructure.getInitArrayOffset());
if (pointer == null) {
throw new IllegalStateException("DT\_INIT\_ARRAY is null");
}
for (int i = 0; i < count; i++) {
Pointer func = pointer.getPointer((long) i \* emulator.getPointerSize());
if (func != null) {
initFunctionList.add(new AbsoluteInitFunction(load\_base, soName, ((UnidbgPointer) func).peer));
}
}
}
}
```

如果我们细心的阅读`Linker`的源码，就会发现`Unidbg`这里处理的是不恰当的。在本文的最后，我们看到了初始化函数的调用，是`DT_INIT`函数先被执行，后面再处理`DT_INIT_ARRAY`，而`Unidbg`这里就是将他们都添加到一个`List`，再一起调用。这样就会产生一个问题，在某些加壳的`So`中，它的`DT_INIT_ARRAY`是在`DT_INIT`函数执行之后，才会有值的`(`进行修复`)`，所以按照`Unidbg`这个写法就无法执行`INIT_ARRAY`或部分`INIT_ARRAY`无法执行。处理方法也很简单，注释在上面了，只需要让`DT_INIT`先执行就可以了。

那么本篇文章对`Linker`的讲解就到这里了，如果您觉得有用，可以加个`VX`一起学习呀:`roy5ue`

# Android Linker详解(二)

- [Android Linker详解(二)](https://bbs.kanxue.com/#android-linker详解二)
  - [本文目的](https://bbs.kanxue.com/#本文目的)
  - [So的链接](https://bbs.kanxue.com/#so的链接)
  - [So重定位](https://bbs.kanxue.com/#so重定位)
  - [总结](https://bbs.kanxue.com/#总结)

## 本文目的

接上篇`Linker`源码详解`(一)`，本文继续来分析`Linker`的链接过程。为了更好的理解`Unidbg`的原理，我们需要了解很多细节。虽然一个模拟二进制执行框架的弊端很多，但也是未来二进制分析的一个很好的思路。

上篇文章我们讲解了`Linker`的装载，将`So`文件按`PT_LOAD`段的指示来将`So`加载到内存，那么我们这篇文章就来分析一下加载完之后又干了什么呢？

## So的链接

> 4cfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6t1K6y4K6l9J5

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `static soinfo*` `load_library(const char*` `name) {`  `//...`  `ElfReader elf_reader(name, fd);`  `if` `(!elf_reader.Load()) {`  `return` `NULL;`  `}`    `const char*` `bname` `=` `strrchr(name,` `'/'` `);`  `soinfo*` `si` `=` `soinfo_alloc(bname ? bname` `+` `1` `: name);`  `if` `(si` `==` `NULL) {`  `return` `NULL;`  `}`  `si->base` `=` `elf_reader.load_start();`  `si->size` `=` `elf_reader.load_size();`  `si->load_bias` `=` `elf_reader.load_bias();`  `si->flags` `=` `0`;`  `si->entry` `=` `0`;`  `si->dynamic` `=` `NULL;`  `si->phnum` `=` `elf_reader.phdr_count();`  `si->phdr` `=` `elf_reader.loaded_phdr();`  `return` `si;`  `}` |

上篇我们进入了`elf_reader.Load()`函数，阅读了`Linker`的装载源码，当装载结束后，对`soinfo`结构体进行赋值(`So`文件的头信息/装载的结果)，并插入到链表，接着我们回到上层函数继续看

> 78eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6t1K6y4K6f1I4

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `static soinfo*` `find_library_internal(const char*` `name) {`  `//...`  `si` `=` `load_library(name);`  `if` `(si` `==` `NULL) {`  `return` `NULL;`  `}`    `//` `At this point we know that whatever`  `is` `loaded @ base` `is` `a valid ELF`  `shared library whose segments are properly mapped`  `in.`  `TRACE("[ init_library base=0x%08x sz=0x%08x name='%s' ]",`  `si->base, si->size, si->name);`    `if` `(!soinfo_link_image(si)) {`  `munmap(reinterpret_cast<void*>` `(si->base), si->size);`  `soinfo_free(si);`  `return` `NULL;`  `}`    `return` `si;`  `}` |

我们从上面这个函数中看到，当调用了`load_library`函数之后，又调用了`soinfo_link_image`这个函数。这个函数也就是我们今天分析的一个主要入口--链接

下面的这个函数很长，我给大家把不相关的代码去掉，大家先通过注释来看一遍这个函数在干什么

> 999K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6t1K6x3e0x3H3x3H3`.

|  |  |
| --- | --- |
