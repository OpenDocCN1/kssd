
#### 隐式链接

### 基本步骤

1. 将`.dll` `.lib`放到工程目录中

2. 使用`#pragma comment(lib,"dllname.lib")`导入`lib`文件

   静态库的`.lib`文件保存二进制代码,而`dll`中的`.lib`文件仅仅是指示函数位置

3. 加入函数声明

   `extern "C" __declspec(dllexport) void func();`

### 具体示例

首先将`.dll`和`.lib`放到工程目录中

![](https://bbs.kanxue.com/upload/attach/202308/968342_X4UJSSGR8YQCFCH.png)

程序代码

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `#pragma comment(lib,"DllTest.lib")          //导入.lib`  `extern` `"C"` `__declspec``(``dllimport``)` `void` `func();``//导入函数声明`  `int` `main() {`  `func();``//直接调用`  `return` `0;`  `}` |

运行结果

![](https://bbs.kanxue.com/upload/attach/202308/968342_YBYERE3AERWXVP2.png)

#### 显式链接

### 基本使用方法

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `//1. 定义函数指针`  `typedef` `int` `(__stdcall *lpPlus)(``int``,``int``);`  `//2. 声明函数指针`  `lpPlus plus;`  `//3. 动态加载dll`  `HINSTANCE` `hModule=LoadLibrary(``"Dllname.dll"``);`  `//4. 获取函数地址`  `plus=(lpPlus)GetProcAddress(hModule,``"_Plus@8"``);`  `//默认的cdecl可以直接用函数名 如果是__stdcall会导致函数名改变`  `//5. 调用函数`  `int` `a=plus(1,2);` |

### 示例程序代码

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `#include<Windows.h>//包含了win32的函数和数据结构`  `#include<stdio.h>`  `int` `main() {`  `typedef` `int``(*lpPlus)(``int``,` `int``);`  `lpPlus plus;`  `HINSTANCE` `hModule = LoadLibrary(L``"DllTest.dll"``);``//L是代表宽字符`  `plus = (lpPlus)GetProcAddress(hModule,` `"plus"``);`  `printf``(``"%d"``, plus(1, 2));`  `return` `0;`  `}` |

运行结果

![](https://bbs.kanxue.com/upload/attach/202308/968342_QEU6FT7F253VBB3.png)

# 数据目录表

### 定义

|  |  |
| --- | --- |
| 1  2  3  4 | `typedef` `struct` `_IMAGE_DATA_DIRECTORY {`  `DWORD`   `VirtualAddress;``//虚拟地址RVA，数据目录表的起始位置`  `DWORD`   `Size;``//大小`  `} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;` |

### 表项定义

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `#define IMAGE_DIRECTORY_ENTRY_EXPORT        //0 导出表`  `#define IMAGE_DIRECTORY_ENTRY_IMPORT        //1 导入表`  `#define IMAGE_DIRECTORY_ENTRY_RESOURCE      //2 资源目录`  `#define IMAGE_DIRECTORY_ENTRY_EXCEPTION     //3 异常目录`  `#define IMAGE_DIRECTORY_ENTRY_SECURITY      //4 安全目录`  `#define IMAGE_DIRECTORY_ENTRY_BASERELOC     //5 重定位基本表`  `#define IMAGE_DIRECTORY_ENTRY_DEBUG         //6 调试目录`  `#define IMAGE_DIRECTORY_ENTRY_COPYRIGHT     //7 描述字串 64位为ARCHITECTURE`  `#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR     //8 机器值`  `#define IMAGE_DIRECTORY_ENTRY_TLS           //9 TLS目录`  `#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG   //10 载入配值目录`  `#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT  //11 绑定输入表`  `#define IMAGE_DIRECTORY_ENTRY_IAT           //12 导入地址表`  `#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT  //13 延迟载入描述`  `#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR//14 COM信息`  `//第16个保留` |

可选头的最后两个成员分别定义了数据目录表的个数和数据目录表数组,指向了一些关键表格

|  |  |
| --- | --- |
| 1  2 | `DWORD`   `NumberOfRvaAndSizes;`  `IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];` `//数据目录表,结构体数组` |

比较重要的有`导出表,导入表,重定位表,IAT表`

### 打印数据目录表

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60 | `//打印数据目录表`  `void` `PrintDirectory() {`  `PIMAGE_DATA_DIRECTORY pDirectory = pOptionalHeader->DataDirectory;`  `printf``(``"\n**********数据目录表**********\n"``);`  `for` `(``DWORD` `i = 0; i < pOptionalHeader->NumberOfRvaAndSizes; i++) {`  `switch` `(i) {`  `case` `IMAGE_DIRECTORY_ENTRY_EXPORT:`  `printf``(``"\n==========导出表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_IMPORT:`  `printf``(``"\n==========导入表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_RESOURCE:`  `printf``(``"\n==========资源目录==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_EXCEPTION:`  `printf``(``"\n==========异常目录==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_SECURITY:`  `printf``(``"\n==========安全目录=========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_BASERELOC:`  `printf``(``"\n==========重定位基本表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_DEBUG:`  `printf``(``"\n==========调试目录==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_ARCHITECTURE:`  `printf``(``"\n==========描述字串==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_GLOBALPTR:`  `printf``(``"\n==========机器值==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_TLS:`  `printf``(``"\n==========TLS目录==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG:`  `printf``(``"\n==========载入配置目录==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT:`  `printf``(``"\n==========绑定输入表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_IAT:`  `printf``(``"\n==========导入地址表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT:`  `printf``(``"\n==========延迟导入表==========\n"``);`  `break``;`  `case` `IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR:`  `printf``(``"\n==========COM信息==========\n"``);`  `break``;`  `case` `15:`  `printf``(``"\n==========保留表==========\n"``);`  `break``;`  `}`  `printf``(``"VirtualAddress=%x\nSize=%x\nFOA=%x\n"``, pDirectory[i].VirtualAddress, pDirectory[i].Size,RVA2FOA(pDirectory[i].VirtualAddress));`    `}`  `printf``(``"\n**********数据目录表打印完毕**********\n\n"``);`  `}` |

# 导出表

导出表记录了`pe`文件导出的函数,所以`.exe`和`.dll`程序都可以导出函数

数据目录表中记录了导出表的地址和偏移 这个地址是`RVA`,需要转换为`FOA`

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `typedef` `struct` `_IMAGE_EXPORT_DIRECTORY {`  `DWORD`   `Characteristics;`  `DWORD`   `TimeDateStamp;`  `WORD`    `MajorVersion;`  `WORD`    `MinorVersion;`  `DWORD`   `Name;` `// 指向导出表文件名 RVA-> FOA + FileBuffer=char *name`  `DWORD`   `Base;` `// 导出函数起始序号`  `DWORD`   `NumberOfFunctions;` `// 所有导出函数的个数`  `DWORD`   `NumberOfNames;` `// 以函数名称导出的函数个数`  `DWORD`   `AddressOfFunctions;` `// 导出函数地址表首地址RVA 记录了所有导出函数的地址 每个表项大小4字节`  `DWORD`   `AddressOfNames;` `// 导出函数名称表首地址RVA 每个表项都是函数名的字符串指针RVA 每个表项大小4字节`  `DWORD`   `AddressOfNameOrdinals;` `// 导出函数序号表首地址RVA 其中存储的序号为-Base后的值  每个表项大小2字节`  `} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;` |

通过`NONAME`关键字可以使部分导出函数没有名称仅有地址

### 关键成员

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `Name`  `Base`  `NumberOfFunctions`  `NumberOfNames`  `AddressOfFunctions`  `AddressOfNames`  `AddressOfNameOrdinals` |

### 具体示例

假设:

导出函数名称表 `FuncNameTable`

导出函数序号表 `FuncOridinalTable`

导出函数地址表 `FuncAddressTable`

函数地址`RVA` `FuncAddress`

使用上一节的`Dll`函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `//DllTest.def`  `LIBRARY` `"DllTest"`  `EXPORTS`    `func1 @15`  `plus @1`  `sub  @3 NONAME` `//sub序号为3 无导出函数名`    `//DllTest.cpp`  `#include<stdio.h>`  `#include<windows.h>`  `void` `func1() {`  `printf``(``"HelloDynamicLib!"``);`  `}`  `int` `plus(``int` `x,` `int` `y) {`  `return` `x + y;`  `}`    `int` `sub(``int` `x,` `int` `y) {`  `return` `x - y;`  `}` |

`DLL`的导出表信息

![](https://bbs.kanxue.com/upload/attach/202308/968342_RVPNWQDASXGKJ9Z.png)

## 序号导出

假设已知导出函数序号`OridinalNum`

那么`FuncAddress=FuncAddressTable[OridinalNum-Base]`

即导出函数序号-`Base`值可以直接作为下标查找导出函数地址表得到导出函数地址

已知函数`sub`的导出序号为`3` 所以`3-1=2`直接查找得到其地址

![](https://bbs.kanxue.com/upload/attach/202308/968342_ST32PN3WYR2HX69.png)

所以无名函数的序号可以通过遍历导出函数地址表来得到

## 名称导出

通过函数名称查找函数地址的过程

1. 首先查找导出函数名称表,判断数组中哪个字符串和目的函数名称相同

2. 将该元素的下标作为索引,查找导出函数序号表

3. 将导出函数序号表中该下标元素的内容作为下标查找导出函数地址表,该值即为函数地址

即

|  |  |
| --- | --- |
| 1  2 | `if``(``strcmp``(name,FuncNameTable[i])==0)`  `FuncAddress=FuncAddressTable[FuncOridinalTable[i]];` |

假设要查找`plus`函数

`plus`这个函数名在函数名称表中的下标为`1`

而`FuncOridinalTable[1]=0`

所以`plus Address=FuncAddressTable[0]=1125d`

![](https://bbs.kanxue.com/upload/attach/202308/968342_56Z7RVVFDM4W9CW.png)

## 注意

1. 导出函数地址表中有很多地址为`0`的项目

由于导出函数地址表的大小=`NumberOfFunctions=`导出函数最大序号-最小序号

当序号不是连续时,就会用`0`地址填充多余表项

![](https://bbs.kanxue.com/upload/attach/202308/968342_GX4WXK9DW6SDCGJ.png)

2. 导出函数序号表存储的序号是真实序号-`Base`

   所以序号最小的导出函数对应的存储序号是`0`

3. 导出函数序号表中不保存无名称函数的序号

   通过序号查找函数时,将序号值-`Base`直接作为下标查找导出函数地址表
