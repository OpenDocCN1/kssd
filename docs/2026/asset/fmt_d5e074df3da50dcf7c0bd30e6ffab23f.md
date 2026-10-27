
压缩实现之后的大小变化：因为只是压缩了一个`.text`区段所以压缩率并不高，只是少了`600KB`左右
![](upload/attach/202012/881392_E7PBEYU6VKUK6VJ.png)

## 2.7 IAT加密

我们先看一下程序从编译到运行之间的流程

**编译**：将单个的`.c`或`.cpp`编译成中间文件 (`.obj`)，在`VS`下，这个过程由`cl.exe`程序完成。

**链接**：将编译出的`.obj`中间文件、系统的启动文件和用到的库文件链接成一个可执行文件 (`.exe`)。`VS`下由`link.exe`程序完成

**装载**：将一个可执行文件映射到虚拟地址空间并执行，由操作系统完成。在装载的过程中，完了让程序正常被执行，会有下列几个步骤：
1. 判断是否开启重定位，如果开启了，将 PE 文件加载到指定位置，并且修复目标PE 文件的重定位。
2. 遍历导入表，加载使用到的所有模块到内存，修复模块相关的信息，并根据导入表中的函数名称，填充所有的`IAT`地址项。
3. 查看当前是否存在`TLS`回调函数，如果存在，则传入进程创建事件，依次调用所有的`TLS`回调函数。
4. 以`PE`文件中的`AddressOfEntryPoint`为起始位置，创建线程并运行。

我们的`IAT`加密是在壳代码中完成的，但是为了能让壳代码顺利接管到`IAT`表，需要在令程序在装载的时候不填充`IAT`表

```C++
//令壳代码接管IAT表

VOID Pack::SetIatTo0() {

// 在重定位之前设置

pShareData->ImpTableVA = pOptionHead(PeBase)->DataDirectory[1].VirtualAddress;//备份原导入表RVA

pOptionHead(PeBase)->DataDirectory[1].VirtualAddress = 0; //清零

pOptionHead(PeBase)->DataDirectory[12].VirtualAddress = 0; //第[12]项也是导入表，也需要清零

}
```

## 2.8 处理TLS

一个程序如果有`TLS`会在程序运行之前就调用`TLS`里面的回调函数，由于我们运行的是加壳后的程序，原程序的代码段已经被加密，直接运行`TLS`肯定不行，我们需要手动处理`TLS`，在原程序中备份`TLS`以及`TLS`的回调函数数组，然后在壳代码中，程序解密后运行前，自己手动遍历`TLS`回调函数，手动循环调用一次，然后在恢复`TLS`。

```C++
//备份TLS后清空TLS

VOID Pack::SetTls() {

pShareData->TlsVirtualAddress = pOptionHead(PeBase)->DataDirectory[9].VirtualAddress;

//如果存在TLS表

if (pShareData->TlsVirtualAddress) {

pOptionHead(PeBase)->DataDirectory[9].VirtualAddress = 0;

DWORD TlsFoa = RvaToOffset(PeBase, pShareData->TlsVirtualAddress);

auto TlsTable = (PIMAGE_TLS_DIRECTORY)(TlsFoa + PeBase);

//备份TLS的回调函数

pShareData->TlsCallBackTableVa = TlsTable->AddressOfCallBacks;

}

}
```

## 2.9 修复重定位

重定位是一个比较麻烦的问题，还需要考虑到随机基址的问题。
首先，`dll`合并后的`.text`区段由于`ImageBase`和`区段RVA`改变了，需要先修复`dll`的`.text`区段的重定位项
![](upload/attach/202012/881392_FBKX2A6SUM3U599.png)

```C++
// 对壳代码进行重定位

VOID Pack::FixReloc(){

DWORD OldImageBase = pOptionHead(DllBase)->ImageBase;

DWORD NewImageBase = pOptionHead(PeBase)->ImageBase;

DWORD OldSectionBase = GetSecHeadAddr(DllBase, ".text")->VirtualAddress;

DWORD NewSectionBase = GetSecHeadAddr(PeBase, ".pack")->VirtualAddress;

auto Relocs = (PIMAGE_BASE_RELOCATION)(DllBase + pOptionHead(DllBase)->DataDirectory[5].VirtualAddress);

// 遍历重定位表

while (Relocs->SizeOfBlock) {

DWORD OldProtect = 0;

VirtualProtect((LPVOID)(DllBase + Relocs->VirtualAddress), 0x1000, PAGE_READWRITE, &OldProtect);

TypeOffset* items = (TypeOffset*)(Relocs + 1);

//重定位项数量

int count = (Relocs->SizeOfBlock - 8) / 2;

//遍历并修复重定位项

for (int i = 0; i < count; ++i) {

if (items[i].Type == 3) {

DWORD* item = (DWORD*)(pOptionHead(DllBase)->ImageBase + Relocs->VirtualAddress + items[i].Offset);

*item = *item - OldImageBase - OldSectionBase + NewImageBase + NewSectionBase;

}

}

VirtualProtect((LPVOID)(DllBase + Relocs->VirtualAddress), 0x1000, OldProtect, &OldProtect);

Relocs = (PIMAGE_BASE_RELOCATION)(Relocs->SizeOfBlock + (DWORD)Relocs);

}

// 关闭源程序的随机基址

//pOptionHead(PeBase)->DllCharacteristics &= ~IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE;

}
```

随机基址的问题：由于有了随机基址，即使我们上面修复了重定位项，程序也不能正常运行，我们可以替换原程序的重定位表，让原程序帮我们修复壳代码的重定位项，当程序装载之后，壳代码的重定位项已经修复为随机基址之后的地址了，我们就可以在壳代码中修复原程序的重定位表，壳代码修复原程序的重定位项需要使用到原程序的重定位表，因此，还需要备份一下原程序的重定位表地址。
那如何替换原程序的重定位表呢?新建一个区段，这个区段用于存放`DLL`的重定位表，然后备份原程序的重定位表后，将原程序的重定位表指向新建的区段。完成重定位表的替换。
![](upload/attach/202012/881392_G5JMY8DMGGH86CJ.png)

```C++
//备份并替换重定位表

VOID Pack::SetReloc() {

//1.备份原程序的重定位表地址，将原程序的重定位写入到通讯结构体中，将dll的重定位表替换到原程序的重定位表

//2.在壳代码中，系统会帮我们修复壳代码的重定位，待原程序解密之后，在壳代码中获取通讯结构体，获取原程序重定位表，修复原程序重定位

DWORD DllImageBase = pOptionHead(DllBase)->ImageBase;

//备份原重定位表

pShareData->dwRelocRva = pOptionHead(PeBase)->DataDirectory[5].VirtualAddress;

pShareData->dwRelocSize = pOptionHead(PeBase)->DataDirectory[5].Size;

pShareData->OldImageBase = pOptionHead(PeBase)->ImageBase;

//Dll重定位表

auto DllBaseReloc = (PIMAGE_BASE_RELOCATION)(pOptionHead(DllBase)->DataDirectory[5].VirtualAddress + DllImageBase);

DWORD DllRelocaSize = pOptionHead(DllBase)->DataDirectory[5].Size;

//新增区段

AddSection(".mysec", DllRelocaSize);

auto NewSecHed = GetSecHeadAddr(PeBase, ".mysec");

auto OldSecHed = GetSecHeadAddr(DllBase, ".text");

auto PackSecHed = GetSecHeadAddr(PeBase, ".pack");

auto NewRelocaSection = (PIMAGE_BASE_RELOCATION)(NewSecHed->PointerToRawData + PeBase);

DWORD OldSectionAddr = (DWORD)(OldSecHed->VirtualAddress + DllBase);

memcpy((DWORD*)NewRelocaSection, (DWORD*)(DllBaseReloc), DllRelocaSize);

while (NewRelocaSection->VirtualAddress){

//新的内存页起始RVA = 原RVA - 原段基址 +.pack段基址

NewRelocaSection->VirtualAddress = NewRelocaSection->VirtualAddress - (OldSectionAddr -DllBase) + PackSecHed->VirtualAddress;

NewRelocaSection = (PIMAGE_BASE_RELOCATION)(NewRelocaSection->SizeOfBlock + (DWORD)NewRelocaSection);

}

//替换原程序重定位表

pOptionHead(PeBase)->DataDirectory[5].VirtualAddress = NewSecHed->VirtualAddress;

pOptionHead(PeBase)->DataDirectory[5].Size = DllRelocaSize;

}
```

添加区段的函数：

```C++
//添加一个新区段

VOID Pack::AddSection(LPCSTR Name, DWORD Size) {

PIMAGE_DOS_HEADER pDos = pDosHead(PeBase);

PIMAGE_NT_HEADERS pNt = pNtHead(PeBase);

//获取需要添加的区段

PIMAGE_SECTION_HEADER pLastSection = (IMAGE_FIRST_SECTION(pNt)) + (pNt->FileHeader.NumberOfSections - 1);

PIMAGE_SECTION_HEADER pNewSection = pLastSection + 1;

//Name

memcpy(pNewSection->Name, Name, 7);

//VirtualSize

pNewSection->Misc.VirtualSize = AlignmentToSize(Size, pNt->OptionalHeader.SectionAlignment);

//VirtualAddress = 最后一个区段的 VirtualAddress +最后一个区段内存大小

pNewSection->VirtualAddress = pLastSection->VirtualAddress +

AlignmentToSize(pLastSection->Misc.VirtualSize, pNt->OptionalHeader.SectionAlignment);

//SizeOfRawData

pNewSection->SizeOfRawData = AlignmentToSize(Size, pNt->OptionalHeader.FileAlignment);

//PointerToRawData = 最后一个区段的 PointerToRawData + 最后一个区段文件大小

pNewSection->PointerToRawData = pLastSection->PointerToRawData +

AlignmentToSize(pLastSection->SizeOfRawData, pNt->OptionalHeader.FileAlignment);

//Characteristics

pNewSection->Characteristics = 0xE00000E0;

//NumberOfSections

pNt->FileHeader.NumberOfSections += 1;

//SizeOfImage大小

pNt->OptionalHeader.SizeOfImage += AlignmentToSize(pNewSection->Misc.VirtualSize, pNt->OptionalHeader.SectionAlignment);

//从新分配空间，将新区段添加到堆空间中

PeSize = pNewSection->SizeOfRawData + pNewSection->PointerToRawData;

PeBase = (DWORD)realloc((VOID*)PeBase, PeSize);

pNewSection = GetSecHeadAddr(PeBase, Name);

memset((DWORD*)(pNewSection->PointerToRawData + PeBase), 0, pNewSection->SizeOfRawData);

return;

}
```

## 2.10 拷贝数据

拷贝`dll`的`.text`区段到被加壳程序的`.pack`区段

```C++
//拷贝区段数据

VOID Pack::CopySecData(CHAR* DestName) {

//.text的地址

CHAR* Src = (CHAR*)(GetSecHeadAddr(DllBase, ".text")->VirtualAddress+DllBase);

//.pack的地址

CHAR* Dest = (CHAR*)(GetSecHeadAddr(PeBase, DestName)->PointerToRawData+PeBase);

memcpy(Dest, Src, GetSecHeadAddr(DllBase, ".text")->Misc.VirtualSize);

return;

}
```

## 2.11 写入磁盘

```C++
//将新文件写入磁盘

VOID Pack::WriteToFile(LPCSTR Path) {

HANDLE hFile = CreateFile(Path, GENERIC_WRITE, 0, NULL,

CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

DWORD WriteSize = 0;

DWORD Ret = WriteFile(hFile, (LPVOID)PeBase, PeSize, &WriteSize, NULL);

if (!Ret)

print("写入文件失败");

CloseHandle(hFile);

return;

}
```

# 三、壳代码

壳代码其实就是动态链接库中合并后的`.text`区段，在`2.1`中有提到过

```C++
#include "../pack/Share.h" // 壳代码与加壳器之间共用的结构体类型头文件
#include"../aplib/aplib.h" // 压缩库头文件
#include <Windows.h>

#pragma comment(lib, "../aplib/aplib.lib") // 压缩库

// 对 dll 的区段合并为.text区段，并设置区段属性为[读写执行]
#pragma comment(linker, "/merge:.data=.text")
#pragma comment(linker, "/merge:.rdata=.text")
#pragma comment(linker, "/section:.text,RWE")

// 去除所有导出函数和导出变量的名称粉碎
extern "C"{
__declspec(dllexport) SHARE ShareData; //导出变量，用于壳代码与加壳器之间的通讯
__declspec(dllexport) __declspec(naked) void start() //导出函数地址，将会作为新的OEP
{
_asm{
mov eax, fs:[0x30]
mov eax, [eax + 0x8] //获取当前进程gImageBase
mov gImageBase, eax
}

GetFunAdd(); //获取GetProcAddress
GetApi(); //获取API
CallShllFun(); //密码弹窗
UnCompression(ShareData); //解压
DecSection(); //解密代码段
```
