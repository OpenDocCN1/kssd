
通过调试发现`backing_store`指针位于偏移`0x20`的位置。

# 7. 利用方式1：将`free_hook`改写为`system`

信息泄露：通过`float Array`的`map`指针泄露`map`的基地址，然后通过`map`偏移`0x18`的位置泄露堆地址，然后读取堆地址的内容，泄露`d8`的基地址。最后通过`puts`函数的`got`表泄露`libc`地址，从而泄露`system`函数和`free_hook`地址。

调试release版本，获取偏移的过程如下：

|  |  |
| --- | --- |
