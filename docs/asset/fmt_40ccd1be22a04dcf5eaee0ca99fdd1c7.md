
# MPEG-2 0day 漏洞：缓冲区溢出覆盖 SEH

> 原文：[`www.pediy.com/kssd/pediy10/92912.html`](https://www.pediy.com/kssd/pediy10/92912.html)

作者 blog：[`hi.baidu.com/yicong2007`](http://hi.baidu.com/yicong2007)

目的：纯属学习，请勿用于恶意用途。

系统环境：XP SP3 简体中文版，IE 7

工具：WinDbg+IDA

感谢 `mj0011` 指正了我原先分析的错误，感谢云砉生指出了出现漏洞的实质原因。

MPEG-2 0day 是一个 `ReadFile` 时的缓冲区溢出，通过覆盖 `SEH` 配合 `heap spray` 使得 `shellcode` 获得执行机会。

漏洞利用网页 `http://vip762.3322.org/aa/index.htm`

相关畸形文件 `logo.gif` 内容：

Offset 0 1 2 3 4 5 6 7 8 9 A B C D E F

00000000 00 03 00 00 11 20 34 00 00 00 00 00 00 00 00 00 ..... 4.........

00000010 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................

00000020 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................

00000030 00 00 00 00 00 00 FF FF FF FF 0C 0C 0C 0C 00 ...........

因为我不清楚这个编码规范，仅凭后面调试过程中看到的结果，猜测这个结构。

第 0-3 个字节 `0x00000300h`，某个属性标识，意义不详

第 4-5 个字节 `0x2011h`，这似乎又是一个属性值（代码中根据这个属性值不同去到不同的分支）

第 6-9 个字节 `0x00000034h`, 这个似乎指的是该 DWORD 后面的内容的长度。

第 10-13 个字节 与第 6-9 个字节一起被读入，用途不详。

第 14 个字节及以后：由于代码对缓冲区地址以及指针的处理有误，这部分数据被读入栈中，造成缓冲区溢出覆盖掉 `SEH`，使 `shellcode` 获得执行机会。

## 触发过程

在 `msvidctl.dll` 的 `ATL::AtlIPersistStreamInit_Load` 调用 `ATL::CComVariant::ReadFromStream` 的时候，后者通过读取文件中的这个标识内容长度的 `0x00000034h`，并调用 `mshtml!FatStream::Read`（是对 `ReadFile` 的封装）将内容读入内存。

`ATL::CComVariant::ReadFromStream` 成功申请了内存，但是在调用 `mshtml!FatStream::Read` 的时候犯了一个程序员经常犯的错误（感谢云砉生指出这一点），即将保存缓冲区指针的堆栈地址当成缓冲区指针本身传入（将传入参数 `buff` 误写为 `&buff`），使得读入的文件内容覆盖掉了属于 `ATL::AtlIPersistStreamInit_Load` 的局部变量以及其设置的 `SEH`，其中 `0xFFFFFFFF` 与 `0x0C0C0C0C` 刚好覆盖掉 `SEH`，`0x0C0C0C0C` 这个地址已通过 `Heap spray` 被 `shellcode` 前面的 `nop` 占据，结果 `ATL::AtlIPersistStreamInit_Load` 在后续操作中触发了异常，就跳到 `shellcode` 中执行了。

`ATL::AtlIPersistStreamInit_Load` 的代码如下，注意注释的标号说明顺序。

代码:
