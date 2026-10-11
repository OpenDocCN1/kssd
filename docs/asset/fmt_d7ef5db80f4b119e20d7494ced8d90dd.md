
## 重定位表构造

标题`"hello"`（`00402000h`）在文件中的偏移为`60Eh`，转化成`RVA`为`60E+(1000-600)=100Eh`，减去这组重定位数据开始的`RVA`地址为`100E-1000=00Eh`，于是得到`TypeOffset`低`12`位地址`00Eh`，`TypeOffset`的高`4`位类型为`3`，所以`TypeOffset`的值为`300Eh`，在`16`进制编辑器里输入`"0E 30"`。

文本`"hello,pediy!!!"`（`00402010`）按标题`"hello"`的方法计算，得到`TypeOffset`的值为`3013h`，在`16`进制编辑器里输入`"13 30"`。

函数`MessageBoxA`地址（`00403060h`）按标题`"hello"`的方法计算，得到`TypeOffset`的值为`301Bh`，在`16`进制编辑器里输入`"1B 30"`。

最后，我们得到`VirtualAddress`为`00001000h`，在`16`进制编辑器里输入`"00 10 00 00"`。

`SizeOfBlock`为`00000010h`（有四个重定位项，其中一个是为了数据对齐，`(10-8)/2=4h`），在`16`进制编辑器里输入`"10 0 00 00"`。

构造完成的重定位表区块`16`进制源码如下：

```
00000E00   00 10 00 00 10 00 00 00  0E 30 13 30 1B 30 00 00   .........0.0.0..
```

好，革命快要成功了，现在就差输出表区块（`.edata`）整个`DLL`文件就完成了。

### 输出表结构

输出表是个指向`IMAGE_EXPORT_DIRECTORY`（`IED`）结构，`IED`结构如下所示：

```c
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;
    DWORD   TimeDateStamp;
    WORD    MajorVersion;
    WORD    MinorVersion;
    DWORD   Name;
    DWORD   Base;
    DWORD   NumberOfFunctions;
    DWORD   NumberOfNames;
    DWORD   AddressOfFunctions;     // RVA from base of image
    DWORD   AddressOfNames;         // RVA from base of image
    DWORD   AddressOfNameOrdinals;  // RVA from base of image
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

#### 字段说明

1. `Characteristics`：四个字节，表示输出表的属性，保留。设置为`"00 00 00 00"`，在`16`进制编辑器里输入`"00 00 00 00"`。
2. `TimeDateStamp`：四个字节，表示输出表创建时间，设置为`"00 00 00 00"`，在`16`进制编辑器里输入`"00 00 00 00"`。
3. `MajorVersion`：两个字节，输出表的版本号，设置为`"00 00"`，在`16`进制编辑器里输入`"00 00"`。
4. `MinorVersion`：两个字节，输出表的次版本号，设置为`"00 00"`，在`16`进制编辑器里输入`"00 00"`。
5. `Name`：四个字节，指向一个`ASCII`码的`RVA`。这个字符串是本输出函数所指向的`DLL`名。这个值后面才论述。
6. `Base`：四个字节，包含这个文件输出表的起始序数值。一般来说，这个值为`1`。设置为`"00 00 00 01"`，在`16`进制编辑器里输入`"01 00 00 00"`。
7. `NumberOfFunctions`：四个字节，输出地址表（`EAT`）中的条目数量。因为只有一个输出函数`ShowMesBox`，所以这个值为`1`，设置为`"00 00 00 01"`，在`16`进制编辑器里输入`"01 00 00 00"`。
8. `NumberOfNames`：四个字节，输出名称表（`ENT`）的条目数量。因为只有一个输出函数`ShowMesBox`，所以这个值为`1`，设置为`"00 00 00 01"`，在`16`进制编辑器里输入`"01 00 00 00"`。
9. `AddressOfFunctions`：四个字节，`EAT`的`RVA`。数组中的每一个非零的`RVA`都对应于一个被输出的符号。
10. `AddressOfNames`：四个字节，`ENT`的`RVA`。`ENT`是一个指向`ASCII`码字符串的`RVA`数组，每个`ASCII`字符串对应于一个被输出的符号。
11. `AddressOfNameOrdinals`：四个字节，输出序数表的`RVA`。这个表将`ENT`中的数组索引映射到相应的输出地址表条目。

现在我们来看，这个`IMAGE_EXPORT_DIRECTORY`结构共用了`40`个字节，也就是`C00`到`C27`。

### 输出表数据构造

那么，我们在`C30`构造`Name`属性的所指向的`DLL`名字的`ASCII`码的。`"BinaryDll.dll"`转化成的`ASCII`码是`"42696E617279446C6C2E646C6C"`，在`16`进制编辑器里输入`"42 69 6E 61 72 79 44 6C 6C 2E 64 6C 6C"`。

把`C30`转化成`RVA`就是`C30+(4000-C00)=4030h`，所以把属性`Name`设置为`"00 00 40 30"`，在`16`进制编辑器里输入`"30 40 00 00"`。

在`C40`构造`EAT`表，`C40`转化为`RVA`为`C40+(4000-C00)=4040h`，所以把属性`AddressOfFunctions`设置为`"00 00 40 40"`，在`16`进制编辑器里输入`"40 40 00 00"`。

在`C50`构造`ENT`表，`C50`转化为`RVA`为`C50+(4000-C00)=4050h`，所以把属性`AddressOfNames`设置为`"00 00 40 50"`，在`16`进制编辑器里输入`"50 40 00 00"`。

在`C60`构造输出序数表，`C60`转化为`RVA`为`C60+(4000-C00)=4060h`，所以把属性`AddressOfNameOrdinals`设置为`"00 00 40 60"`，在`16`进制编辑器里输入`"60 40 00 00"`。

在`C40`处输入输出函数`ShowMesBox`的`RVA`，有代码区可得`DLL`的入口函数用了`8`个字节的空间，则`ShowMesBox`的地址为`1000+8=1008h`，在`16`进制编辑器里输入`"08 10 00 00"`。

在`C70`处输入输出函数`ShowMesBox`的`ASCII`码，在`16`进制编辑器里输入`"53 68 6F 77 4D 65 73 42 6F 78"`。

在`C50`处输入`C70`的`RVA`，为`C70+(4000-C00)=4070`，在`16`进制编辑器里输入`"70 40 00 00"`。

在`C60`处构造输出序数表，由于只有一个输出函数，所以设置为`"00 00"`，在`16`进制编辑器里输入`"00 00"`。

完成后输出表`16`进制源码如下：

```
00000C00   00 00 00 00 00 00 00 00  00 00 00 00 30 40 00 00   ............0@..
00000C10   01 00 00 00 01 00 00 00  01 00 00 00 40 40 00 00   ............@@..
00000C20   50 40 00 00 60 40 00 00  00 00 00 00 00 00 00 00   P@..`@..........
00000C30   42 69 6E 61 72 79 44 6C  6C 2E 64 6C 6C 00 00 00   BinaryDll.dll...
00000C40   08 10 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000C50   70 40 00 00 00 00 00 00  00 00 00 00 00 00 00 00   p@..............
00000C60   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000C70   53 68 6F 77 4D 65 73 42  6F 78 00 00 00 00 00 00   ShowMesBox......
```

### 数据目录表填写

最后，我们来填写数据目录表的内容。

数据目录表中的输出表`RVA`是`4000`，大小是`80h`，我们在`16`进制编辑器输入`"00 40 00 00 80 00 00 00"`。

数据目录表中的输入表`RVA`是`3000`，大小是`70h`，我们在`16`进制编辑器输入`"00 30 00 00 70 00 00 00"`。

数据目录表中的重定位表`RVA`是`5000`，大小是`10h`，我们在`16`进制编辑器输入`"00 50 00 00 10 00 00 00"`。

### 测试程序

最后写了个小测试程序，代码如下：

```c
#include <iostream>
#include <windows.h>

using namespace std;

int main() {
    typedef void(*pShowMsg)(void);
    HINSTANCE hDLL;
    pShowMsg ShowMeg;
    hDLL=LoadLibrary("BinaryDll.dll");
    ShowMeg=(pShowMsg)GetProcAddress(hDLL,"ShowMesBox");
    ShowMeg();
    ::FreeLibrary(hDLL);
    return 0;
}
```

运行后出现下图：

![](img/36b209d3544f69ad6fabcf1cb73db6e1.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) |
| :--- |
| `手写 DLL.rar` |
