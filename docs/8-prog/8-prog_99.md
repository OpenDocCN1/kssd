
# 在汇编程序中调用 C 语言的库函数

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-671.htm`](https://www.pediy.com/kssd/pediy08/pediy8-671.htm)

在汇编程序中调用 C 语言的库函数

作者：chaykovsky

E-mail: chaykovsky@163.com

http://JinYawei.MyCool.Net

在汇编语言中如果能够调用 C 语言的库函数，有时会很方便

先给个示例程序：

```asm
.386
.model flat,stdcall
option casemap:none
;Include 定义
include stdio.inc
includelib msvcrt.lib

.data
szMsg db "在汇编程序中调用 C 的 puts 函数!",0

.code
start:
invoke puts,offset szMsg
ret
end start
```

是不是有点像 C 中的 `Hello,world` 程序。

`stdio.inc` 怎么来的？

`stdio.inc` 为标准输入输出 C 函数的声明。

`MASM32` 软件包中没有，`Microsoft` 也没有提供，这个是我从 `VC` 中的 `stdio.h` 转换得来的。

怎么转换？

`Microsoft` 提供了一个 `.H` 文件转换为 `.inc` 文件的工具 `H2INC`，在 `VC` 的目录中可以找到。

转换命令：

```bash
h2inc /WIN32 \*.h
```

`stdio.h` 直接转换时，会出错，要做一些修改。

1) 在最前面加一句：

```c
#define _WIN32
```

2) 把文件中的 `#pragma pack(push,8)` 与 `//#pragma pack(pop)` 注释掉

转换成功之后，只要在程序开头加上

```asm
include stdio.inc
includelib msvcrt.lib
```

就可以调用 C 语言的输入输出函数了。

建议将 `msvcrt.lib` 所在的目录设置到 `lib` 环境变量中，并将 `stdio.inc` 所在的目录设置到 `include` 环境变量中

`msvcrt.lib` 为 `msvcrt.dll` 或者 `msvcr71.dll`(VC7.1)的库文件，`VC` 的 `lib` 目录中有

使用 `h2inc` 转换头文件时需要注意一些事项：

1) `/WIN32` 参数一定要加上，不然就会按默认的 16 位进行转换
2) `h` 文件最后一行要有个回车
3) 函数类型声明会先转化为 `@proto_n` 的形式，然后再声明，如果转换了好几个关文件，那么几个文件中都是从 `@proto_0` 开始的，如果你同时包含这几文件时编译器就会提示：`symbol type conflict`(符号类型冲突)。

解决方法：

转换后，用文本编辑器的全部替换功能，将不同文件中的“`proto_`”全部替换为不同的字符串。

4) 结构和联合也有与 3) 一样的问题。

5) 如果 `h` 文件中的函数声名中没有加 `__cdecl` 修饰，就会被转换为默认的 `STDCALL` 类型

6) `h2inc` 不会转换 `h` 文件中的程序，有很多 C 语法也不支持。

附件中我转换的 `stdio.inc` 等头文件和示例程序附件:`c_include.rar`

4dd466d0e1e0cc123afbe504da2462ca
