
# [开源]快速转换dll到shellcode工具

好久之前的代码了，当做新婚礼物送给大家把 o \_o

上次开源了个`ObjToShellCode`，好像被华丽丽的无视了 QAQ

## 功能说明

1. 支持x86与x64
2. 支持压缩功能，支持aplib压缩以及windows自带的压缩函数两种模式
3. 支持直接调用dllmain，dllmain最后的保留参数可以拿来传参数。支持返回导出表函数，调用完shellcode之后会返回导出表的函数。

具体代码见github吧，修改起来很方便

生成完的shellcode大概结构如下

=========================
ShellCode主体
=========================
ShellCode Param

`uint8_t` 调用模式 [ 0 dllmain, 1 返回导出函数地址 ]

`uint32_t` 解压代码偏移

`uint32_t` 压缩前大小

`uint32_t` 压缩后大小

`uint32_t` dll数据偏移

char dllmain参数或导出函数地址[最大长度100]
=========================
解压代码(不压缩就不会有它了)
=========================
dll数据
=========================

github地址[e08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8K9h3I4D9k6i4k6W2L8W2)9J5c8V1c8D9L8q4c8G2f1$3S2W2L8r3I4o6L8$3c8W2](https://bbs.kanxue.com/elink@d33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8K9h3I4D9k6i4k6W2L8W2)9J5c8V1c8D9L8q4c8G2f1$3S2W2L8r3I4o6L8$3c8W2)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-3-18 18:38

被小夜的夜编辑，原因：
