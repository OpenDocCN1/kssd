
# [原创]PE知识汇总

# PE知识汇总（三）

## 前文：

[PE知识汇总（一）](https://bbs.pediy.com/thread-262189.htm)

[PE知识汇总（二）](https://bbs.pediy.com/thread-262217.htm)

### 5、输出表

* 创建一个`DLL`时，实际上创建了一组能让`EXE`或其他`DLL`调用的函数，此时`PE`装载器根据`DLL`文件中输出的信息修改正在被执行文件的`IAT`。当一个`DLL`函数能被`EXE`或其他`DLL`文件使用是，它就被“输出了（Exported）”。其中，输出信息被保存在输出表中，`DLL`文件通过输出表向系统提供输出函数名、序号和入口地址等信息。

> `EXE`文件一般不存在输出表，而大部分`DLL`文件存在，但也不是绝对的。

* 输出表结构：输出表（Export Table）的主要内容是一个表格，其中包括函数名称、函数输出序号等信息。输出表示数据目录表的第一个成员，指向`IMAGE_EXPORT_DIRECTORY`(`IED`)结构。

* 结构：

```typedef struct _IMAGE_EXPORT_DIRECTORY {
DWORD Characteristics; // 未使用，总为0
DWORD TimeDateStamp; // 文件创建时间戳
WORD MajorVersion; // 未使用，总为0
WORD MinorVersion; // 未使用，总为0
DWORD Name; // 指向一个代表此 DLL名字的 ASCII字符串的 RVA
DWORD Base; // 函数的起始序号
DWORD NumberOfFunctions; // 导出函数的总数
DWORD NumberOfNames; // 以名称方式导出的函数的总数
DWORD AddressOfFunctions; // 指向输出函数地址的RVA
DWORD AddressOfNames; // 指向输出函数名字的RVA
DWORD AddressOfNameOrdinals; // 指向输出函数序号的RVA
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

* `Name`:模块的真实名称。

* `Base`:基数，序数减基数就是函数地址数组的索引值

* `NumberOfFunctions`:`AddressOfFunctions`阵列中函数个数

* `NumberOfNames`:`AddressOfNames`阵列中函数个数

* `AddressOfFunctions`：指向函数地址数组

* `AddressOfNames`：指向函数名称数组

* `AddressOfNameOrdinals`：指向输出序号数组

* 输出表指向方式

![图片描述](upload/attach/202009/889500_H6Y2BDV23MC7KYH.png)

* 实例：

> 这里所使用的`DLL` 是自己所编写的，带有输出函数的`DLL`

```// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#include <math.h>
int myAdd(int a, int b, int* result)
{
*result = a + b;
return 1;
}
// 导出函数定义
int myDec(int a, int b, int* result)
{
*result = abs(a - b);
return 1;
}
BOOL APIENTRY DllMain( HMODULE hModule,
DWORD ul_reason_for_call,
LPVOID lpReserved
)
{
switch (ul_reason_for_call)
{
case DLL_PROCESS_ATTACH:
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}
```

```
// Source.def 定义导出函数以及函数序号
LIBRARY
EXPORTS
myAdd @ 1
myDec @ 2
LIBRARY
```

1. `PETool`打开`dll` 文件，查看数据目录表的第一个值（Export Table）

![图片描述](upload/attach/202009/889500_XNX4KPXZJEQFS8V.png)

2. 计算位于哪个区块

![图片描述](upload/attach/202009/889500_29A5Q4S9PDGVMJ7.png)

3. 16进制编辑器打开`dll` 文件，并且跳转到`16C0`处

![图片描述](upload/attach/202009/889500_DPF2V6QZWD7U8XS.png)

`00 00 00 00` ——> `Characteristics`

`FF FF FF FF` ——> `TimeDateStamp`

`00 00` ——> `MajorVersion`

`00 00` ——> `MinorVersion`

`FC 24 00 00` ——> `Name=24FC=>16FC(FileOffset)`

`01 00 00 00` ——> `Base=01`

`02 00 00 00` ——> `NumberOfFunctions=02(存在两个函数地址)`

`02 00 00 00` ——> `NumberOfNames=02(存在两个函数名称)`

`E8 24 00 00` ——> `AddressOfFunctions=24E8=>16E8(FileOffset)`

`F0 24 00 00` ——> `AddressOfNames=24F0=>16F0(FileOffset)`

`F8 24 00 00` ——> `AddressOfNameOrdinals=24F8=>16F8(FileOffset)`

4. 跳转到这些地址查看

`Name`:

![图片描述](upload/attach/202009/889500_4B7GZG5BNQMJZ9J.png)

`AddressOfFunctions`：

![图片描述](upload/attach/202009/889500_PG3V7YKYJV6WFE8.png)

`AddressOfNames`：

![图片描述](upload/attach/202009/889500_GVTFKS9E8HGB9JM.png)

`AddressOfNameOrdinals`：

![图片描述](upload/attach/202009/889500_JNM7UDZJQ7Q9T4G.png)

`1:myAdd:0000 1000`

`2:myDec:0000 1020`

5. `IDA` 加载查看一下

![图片描述](upload/attach/202009/889500_KGUSQGWQ4MH5W3D.png)

> 注释：`1000 0000` 是`ImageBase`，上面得出的`0000 1000`可以理解为偏移

未完待续....

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
