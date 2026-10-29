
# 010 Editor 模板解析 dex 文件格式

在学习《Android软件安全与逆向分析》中的 dex 文件格式时写的基于 010 Editor 的模板，算是重新温习了一下 010 模板编写，希望刚接触 android 的可以 "**可视化方式**" 来学习 DEX 文件格式

## 模板的使用方法

1. 将下面的代码复制并保存到文本文件, 文件后缀名可以是任意或默认的 `*.bt` 后缀。
2. 菜单栏—>模板—>打开模板, 选择保存的文本文件, 按 `F5` 运行模板。

![](https://bbs.kanxue.com/upload/attach/201603/271698_jzxc484wl6mkdqf.jpg)

## 一、010 文件格式模板缺点

1. 语法类似于伪 C 代码不能使用指针和 `new`/`delete`/`malloc`/`free` 等 C/c++ 运行库。
2. 目前不支持调试，仅支持打印日志。
3. 对打印日志支持不够完美，例如: `struct {int xxx}aaa<read=ReadEventFunc,read=WriteEnentFunc>;` 对于读写结构事件回调函数中不能声明结构体和使用 `Print` 函数打印日志。

## 二、010 文件格式模板介绍

### 1. 基本类型

8-Bit Signed Integer – `char`, `byte`, `CHAR`, `BYTE`

8-Bit Unsigned Integer – `uchar`, `ubyte`, `UCHAR`, `UBYTE`

16-Bit Signed Integer – `short`, `int16`, `SHORT`, `INT16`

16-Bit Unsigned Integer – `ushort`, `uint16`, `USHORT`, `UINT16`, `WORD`

32-Bit Signed Integer – `int`, `int32`, `long`, `INT`, `INT32`, `LONG`

32-Bit Unsigned Integer – `uint`, `uint32`, `ulong`, `UINT`, `UINT32`, `ULONG`, `DWORD`

64-Bit Signed Integer – `int64`, `quad`, `QUAD`, `INT64`, `__int64`

64-Bit Unsigned Integer – `uint64`, `uquad`, `UQUAD`, `UINT64`, `QWORD`, `__uint64`

32-Bit Floating Point Number – `float`, `FLOAT`

64-Bit Floating Point Number – `double`, `DOUBLE`

string type: `string`, `wstring`

### 2. 特有的数据结构

Date Types – `DOSDATE`, `DOSTIME`, `FILETIME`, `OLETIME`, `time_t` (for more information on date types, see Using the Inspector)

声明和使用几乎和 C 语言没有什么区别，`char var[8]` 和 `string var` 长度相同情况下几乎是相等的，能相互赋值.

### 3. 结构类型

`enum` 能指定枚举成员长度 `byte`/`word`/`dword`/`float`…..等基本类型长度，在解析文件时是非常非常有用的。

```
enum MYENUM
{
   COMP_1,
   COMP_2 = 5,
   COMP_3
} var1;

//enum指定成员数据长度
enum <ushort> MYENUM
{
   COMP_1,
   COMP_2 = 5,
   COMP_3
} var1;
```

`struct` 基本和 C 语言一致:

```
typedef struct
{
  int xxx;
}strVar;

struct
{
  int xxx;
}strVar;
```

### 4. 属性

这里的属性可以针对整个结构或结构成员，关于结构读/写事件回调注意的地方请参考 010 缺点章节.

`< format=hex|decimal|octal|binary, //按?进制显示`

`fgcolor=<color>, //前景色`

`bgcolor=<color>, //背景色`

`comment="<string>", //注释`

`open=true|false|suppress,`

`hidden=true|false,`

`read=<function_name>, //读取数据结构体回调,(仅用于结构变量, 回调原型参数是跟据结构类型来决定,string ReadCallback(structType &structVar);)`

`write=<function_name> >//hex编辑窗口写入数据回调(仅用于结构变量, 回调原型参数是跟据结构类型来决定, void WriteCallback(structType &structVar, string hexEditData);)`

### 5. 当文件的数据结构被定义时

结构成员以变量名来命名, 逐个从第一个成员开始读取, 每读取一个成员文件蕴含的文件指针将被自动增加, 增加的步长以数据类型长度为基准，读取完毕之后调用读取回调函数，回调函数执行完毕之后返回模板的值

```
//**************被定义的数据结构**************
typedef struct
{
   int myInt;
   short myShort;
   char myCharArray[6];
}MyStruct<read=ReadCallback,write=WriteCallback>;

//**************回调函数**************
//当MyStruct结构被定义时(实例化时:使用结构MyStruct myVar)，ReadCallback将被调用，参数为读取该结构的引用.
string ReadCallback(MyStruct &myStruct)
{
   local string value = "value";
   return value;  //返回值见下图，返回给检查器的值列的
}
```

当一个数据结构设置读回调时, 模板可以有返回值并显示在 "值" 列

![](https://bbs.kanxue.com/upload/attach/201603/271698_cg8vgq1ycgwqoim.png)

### 6. 文件指针

在模板执行中会蕴含一个文件指针，刚执行模板时默认文件指针偏移 0，**当结构模板被定义后变量值默认为全局**, 局部变量可以加 `local` 前缀定义。

### 7. 010 模板和脚本函数库

函数库可以适当查看帮助文档, 在此列出函数库类型:

Interface Functions : 包含了书签、剪贴板、文件信息获取、结构成员的属性设置等杂类函数

I/O Functions : 大小尾设置、文件读/写、文件指针等等

String Functions : 字符串比较、复制、转换等

Math Functions : 数学计算相关的函数

Tool Functions : 范围内的数据 HASH 计算

## DEX 文件格式解析脚本

以下脚本在 `010 Editor v3.1.3` 测试通过，其中 DEX 中的数据结构部分来自 Android 源码。
