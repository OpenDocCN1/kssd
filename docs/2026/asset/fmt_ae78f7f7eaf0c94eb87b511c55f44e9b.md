
导出表事实上并不只是存在于DLL中，同时也可能存在于exe中。当一个PE文件被装载的时候，装载器会将PE文件中所有被登记的DLL一起载入，然后根据DLL的导出表对导入表中的IAT进行修正。

## 0x2.2 导出表结构

导出表描述信息位于可选头的数据目录中的第一项。4000是导出表的RVA值，需要转化为FOA值。利用导入表的公式`section[i].PointOfRawData+(RVA-VirtuallAddress)`进行计算，得到FOA为C00.

![](upload/attach/201904/739734_XBFPH9MDKUJKD6J.jpg)

![](upload/attach/201904/739734_VVSUS8C9TVRCFDN.jpg)

这时候，我们在文件中的C00处就可以找到我们导出目录了。我们现在需要了解一下导出目录`IMPORT_EXPORT_DIRECTORY`这个结构.其中我们需要重点关心的成员有以下几个。

* `Name`:导出函数的文件名RVA
* `Base`:导出函数起始需要，在导出函数序号表中的值，需要加上此值才是导出函数真正的序号
* `NumberOfFunctions`：导出函数个数
* `NumberOfNames`：名称导出函数个数
* `AddressOfFunctions`; //指向到处函数地址表的RVA
* `AddressOfNames`; //指向函数名地址表的AVA
* `AddressOfNameOrdinals`; //指向函数名序号表的RVA

```
typedef struct _IMAGE_EXPORT_DIRECTORY {
DWORD Characteristics;
DWORD TimeDateStamp; //输出表的创建时间
WORD MajorVersion; //输出表的主版本号。未使用设置为0
WORD MinorVersion; //输出表的次版本号。未使用设置为0
DWORD Name; //指向一个与输出函数关联的文件名的RVA
DWORD Base; //导出函数的起始序号
DWORD NumberOfFunctions; //导出函数的总数
DWORD NumberOfNames; //以名称导出的函数总数
DWORD AddressOfFunctions; //指向到处函数地址表的RVA
DWORD AddressOfNames; //指向函数名地址表的AVA
DWORD AddressOfNameOrdinals; //指向函数名序号表的RVA
} IMAGE_EXPORT_DIRECTORYM, *pIMAGE_EXPORT_DIRECTORY
```

如下图是导出表的数据。可以看到`32 40 00 00`对应的是`Name`这个成员变量的RVA，我们调整端序，计算FOA(32400000--->00004032--->C32)可到Name在文件中的位置是C32.正好指向DllDemo.dll这个字符串。根据上述方法解析导出表数据如下

![](upload/attach/201904/739734_5NZ5DNJXE8KEJS4.jpg)

* `Name`：[C06]=C32-->"DllDemo.dll" (rva)
* `Base`：[C10]="00000001"
* `NumberOfFunctions`:[C14]="00000001"
* `NumberOfNames`:[C18]="00000001"
* `AddressOfFunctions`:[C1C]=C28--->08100000 (RVA)
* `AddressOfNames`:[C20]=C2C--->"MsgBox" (RVA)
* `AddressOfNameOrdinals`[C24]=C30--->0000 (RVA)

【重点】PE装载器调用`GetProcAddress`来填充IAT，这时候需要了解`GetProcAddress`原理。

* 定位到`IMAGE_EXPORT_DIRECTORY`结构
* 判断`Name`是否是传入的`DllName`
* 获取`ENT`数组的起始地址，寻找`FuncName`，如果找到记录在`ENT`的数组索引。
* 然后读取导出函数序号数据的第一项的序号值
* 使用`Base+序号`的值到`EAT`中寻找地址

## 0x2.3 导出表编程

首先需要在数据目录中获取导出表地址，需要注意的是这个地址是个RVA的值，需要加上`BaseAddress`。

```
DWORD Rav_Export_Table = pNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;
PIMAGE_EXPORT_DIRECTORY ExportTable = (PIMAGE_EXPORT_DIRECTORY)((ULONG_PTR)lpBaseAddress + Rav_Export_Table);
```

由于`AddressOfNames`,`AddressOfFunctions`,和`AddressOfNameOrdinals`都是RVA值，且都需要进程RVA->FOA的转化。所以如果要获取这三个成员变量的FOA的话需要进行两部，第一将RVA转化为FOA，然后加上BaseAddress。

```
//获取导出函数名数组
DWORD* dwAddressOfNames =(DWORD*)((ULONG_PTR)lpBaseAddress+ RvaToOffset(ExportTable->AddressOfNames,lpBaseAddress));
//获取导出函数数组
DWORD* dwAddressOfFunctions = (DWORD*)((ULONG_PTR)lpBaseAddress + RvaToOffset(ExportTable->AddressOfFunctions, lpBaseAddress));
//获取导出函数索引数组
DWORD* dwAddressOfNameOrdinals = (DWORD*)((ULONG_PTR)lpBaseAddress + RvaToOffset(ExportTable->AddressOfNameOrdinals, lpBaseAddress));
```

因为`NumberOfFunctions>=NumberOfNames`,适合放在外部循环进行比较。

```
for (i = 0; i < dwNumberOfFunctions; i++){...}
```

在提到关于`GetProcAddress`用法时说道，如果目标函数在`AddressOfFuns`被找到记录其数组索引。然后在导出函数序号数组中寻找对应序号。

```
if (*(WORD*)(dwAddressOfNameOrdinals + j * sizeof(WORD)) == i)
{}
```

接着取出我们的需要序号，以及在导出函数地址数组对应的序号所对应的导出函数地址

```
//函数名称
FunName = (LPCSTR)((ULONG_PTR)lpBaseAddress + dwAddressOfNames[j * sizeof(WORD)]); //VA值
//函数索引
FunOrdinal = *(WORD*)(dwAddressOfNameOrdinals + j * sizeof(WORD));
//函数地址 i=(dwAddressOfNameOrdinals + j * sizeof(WORD)) 其实是索引值
//这里需要取其值，注意*(DWORD*)
FunAddress = *(DWORD*)(dwAddressOfFunctions + FunOrdinal * sizeof(DWORD));
```

![](upload/attach/201904/739734_JDYJPVCQ9NZ2DF2.jpg)

# 0x3 重定位表

## 0x3.1 重定位原理

PE装载的时候，大多选用相对地址RVA，这样做的原因有二，第一是为了方便装载器，二是为了重定位。那么什么时候需要重定位呢，或者说重定位的条件是什么呢，我们应该知道每个进程内存是相互独立的，也就是说exe文件发生重定位的可能性不高，这样看来由于DLL的装载位置的不同，DLL文件发生重定位的可能性就比较高了，因为同一个进程空间里面可能存在多个DLL文件，有时候多个dll设定的原始的基地址是相同的，但是对于操作系统来说这是不允许存在的，所以需要对他们进行重定位。

这时候就需要将那些需要被重定位的数据保存在一张表里面，然后取出里面的地址，利用某一个特定的公式，重新计算加载地址即可。

## 0x3.2 重定位表结构

重定位表位于数据目录项中第6项，通过上述的方法可以定位到文件中重定位表地址为0xE00

![](upload/attach/201904/739734_KQN5YKEJYMRY9TR.jpg)

![](upload/attach/201904/739734_8Y5CZEYTRJR6J58.jpg)

接下来，我们开始解析重定位表，在解析之前，需要了解重定位表结构。重定位表是由多个`IMAGE_BASE_RELOCATION`结构体构成的。有三个成员变量

```
typedef struct _IMAGE_BASE_RELOCATION {
DWORD VirtualAddress;//RVA
DWORD SizeOfBlock; //重定位数据大小
WORD TypeOffset; // 重定位项数组
} IMAGE_BASE_RELOCATION,* PIMAGE_BASE_RELOCATION;
```

* `VirtualAddress`是重定位数组的RVA，**但是需要每个重定位数组地址加上这个RVA才是真正的重定位数组的地址**
* `SizeofBlock`：重定位结构大小
* `TypeOffset`：两个字节，16位，高4位表示重定位类型，低12位表示重定位地址

如图所有，解析如下：

* `VirtualAddress`：00001000
* `sizeofBlock`：10H-->(10H-8H)/2=4,一共有四个重定位数组，8H指的是`VirtualAddress`和`sizeofBlock`所占的字节数为8，除以2H，表示一个`TypeOffset`为2个字节。
* `Data1`:0F30-->300F-->Type:3-->Addr:00F-->RVA:100Fh--->FOA:60F
* `Data2`:2330-->3023-->Type:3-->Addr:023-->RVA:1023h--->FOA:623
* `Data3`:0000
* `Data4`:0000

得到需要重定位的数据为00402000和00403030，假设当前基地址为00400000，目标基地址为01000000 则重定位后的地址为01002000和01003030

## 0x3.3 重定位表编程

首先获取重定位表地址

```
//重定位表地址
DWORD RelocTableRva = pNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress;
// printf("\t\t [*]RelocTableRva:%p\n", RelocTableRva);
PIMAGE_BASE_RELOCATION RelocTable =(PIMAGE_BASE_RELOCATION)((ULONG)lpBaseAddress+RvaToOffset(RelocTableRva,lpBaseAddress));
printf("\t\t [*]RelocTable:%p\n", RelocTable);
```

获取每个重定位数组的RVA地址和大小，原因有二，第一，获取完`TypeOffset`的第12位需要加上`VirtualAddress`才是真正的RVA，第二，`sizeofBlock`是指向下一个重定位数组(块)的重要参数。

```
DWORD VirtualAddress = RelocTable->VirtualAddress;
printf("\t\t [*]VirtualAddress:%p", VirtualAddress);
DWORD Cout = (RelocTable->SizeOfBlock - 8) / 2;
```

定位到重定位数组，每个`Typeoffset`在偏移处第八个字节，所以需要加8。

```
WORD* RecAddr = (WORD*)((BYTE*)RelocTable + 8);
```

解析`TypeOffset`

```
//取第三位地址，并加上VirtualAddress才是真的RVA
DWORD offset = VirtualAddress + (RecAddr[j] & 0x0FFF);
//TYPE
DWORD type = RecAddr[j] >> 12;
printf("\t\t Type:[%d] \t RVA:[%p]\n", type, offset);
```

利用`sizeofblock`定位下一个重定位表

```
RelocTable = (IMAGE_BASE_RELOCATION *)((BYTE *)RelocTable + RelocTable->SizeOfBlock);
```

# 0x0 前言

刚刚结束春招，投了好几家公司，结果不是很理想，原因无外乎自身实力和行业寒冬。

这次春招面试题主要集中在PE相关，HOOK技术和DLL注入技术，还有一些杂七杂八的问题上面。从中也暴露处自己技能栈上的不足，通过这一段时间的沉淀希望可以补足。

四月初就开始谋划写一系列的文章，但是加上在校实习比较忙碌，所以进展很慢，这些文章主要面向受众是那些入坑新人，借此希望能够让那些小伙伴能够少走弯路。同时也能多多总结自身的不足，共同进步。

这系列文章取啥名？想了很久，决定参考某一师傅的系列文章---<一篇文章带你·····>,主要希望总结PE文件，HOOK,DLL注入，以及其他方面的知识，主要的参考文献是看雪**加密解密第四版**，以及其他资料。(打了广告，希望相关师傅记得打点广告费)

这系列是我边总结技术边写文章，可能部分内容会以后补全技能栈，例如R0下的DLL注入等。**但是尽量做到不鸽**，关于代码，不提供自己写的代码，原因有二，第一，这些代码网上都有现成的，我只是理解修改部分罢了，也怕自己的代码误导小伙伴们。**第二，拒绝伸手党**。

由于自身能力有限，文章中难免出现错误，希望各位师傅少喷我。

# 0x1 PE导入表

## 0x1.1 输入函数的调用

`DLL`动态链接库文件主要实现代码的复用。当一个程序调用`DLL`文件中的数据和代码的时候，有两种链接方式，第一种是**隐式链接**，这个过程是由windows装载器完成的，另外一种是**显式链接**，通过使用`LoadLibrary`和`GetProcAddress`这两个API函数实现的。

当隐式的调用一个API函数的时候，同样也存在类似于`LoadLibrary`和`GetProcAddress`函数的功能实现，但是，这个操作是由windows装载器完成的，所以称为**隐式链接**，当程序使用隐式链接调用`DLL`代码的时候，装载器需要完成以下几个步骤(IAT填充):

- 首先将所需要的`DLL`文件载入内存，`Kernel32.dll`等是通过映射的方式载入的
- 定位IID，寻找IID的第四个字段Name。
- 接着根据`OrginalFirstThunk`指向，获取`INT`。
