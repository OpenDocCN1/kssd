
# [原创][分享]PE文件结构之打印重定位表

# pe文件之重定位表

## 1. 为什么出现重定位表？

### 一个软件一般由一个`exe`和多个`dll`组成，当一个`exe`运行的时候`exe`文件先根据`ImageBase`来载入内存，`dll`也是属于`pe`文件，首先是`exe`先载入内存，多个`dll`文件中的`ImageBase`也许会被其他`dll`占取，又因为编译以后地址是写死的，`dll`文件中的`ImageBase`被占住，只能占别的地方，因为地址写死，占的不是原来的地址，所以需要重定位表来标明需要修改的地址

## 2. 在可选`pe`目录中，最后16个结构体的，第6个是是重定位表

#### 注意：`VirtualAddress`是`RVA`，如在文件中使用（不是载入内存）则需要转换为`FOA`

![](upload/attach/202010/889006_WK2CBG6PGQJVK28.jpg)

![](upload/attach/202010/889006_UBZAUGPFG7PZ26R.jpg)

#### 第一个结构中的`VirtualAddress`指向的是第二个结构体的开始

#### 第二个结构中的`VirtualAddress`指向的是重定位表的开始，`SizeOfBlock`表示块的大小

## 3. 重定位表的结构

![](upload/attach/202010/889006_96SMKNRTZFGUJHA.jpg)

### ① 判断一共有几块数据

#### 最后一个结构的`virtualAddress`与`SizeOfBlock`都为0，则结束。

### ② `0011101010101010`是第一项，先判断高四位`0011`

#### 值为3 代表的是需要修改的数据

#### 值为0 代表的是用于数据对齐的数据，可以不用修改

### 当高4位是3则低12位`101010101010`是需要修改的地方的偏移地址

### 真正的`RVA` = `VIrtualAddress` + 具体项的低地址

### ③ 具体项的数量 =（`SizeOfBlock`-8）/2

### ④ 下一个起始地址 = `VirtualAddress` + `SizeOfBlock`

```c
#include "stdafx.h"
#include <stdlib.h>
#include <stdio.h>
#include <windows.h>
DWORD ReadPEFile(char* filepath,PVOID* pFileBuffer)
{
PVOID pTempFileBuffer = NULL;
DWORD Filesize = 0;
FILE* pFile = NULL;
pFile = fopen(filepath,"rb");
if(!pFile)
{
printf("文件打开失败");
return 0;
}
fseek(pFile,0,SEEK_END);
Filesize = ftell(pFile);
fseek(pFile,0,SEEK_SET);
pTempFileBuffer = malloc(Filesize);
if(!pTempFileBuffer)
{
printf("申请动态内存失败");
fclose(pFile);
return 0;
}
size_t n = fread(pTempFileBuffer,Filesize,1,pFile);
if(!n)
{
printf("文件写入文件缓冲区失败");
free(pTempFileBuffer);
fclose(pFile);
return 0;
}
*pFileBuffer = pTempFileBuffer;
pTempFileBuffer = NULL;
free(pTempFileBuffer);
fclose(pFile);
return Filesize;
}
DWORD RvaToFoa(PIMAGE_NT_HEADERS pNTHeader, DWORD dwRVA)
{
PIMAGE_SECTION_HEADER pSection = (PIMAGE_SECTION_HEADER)((DWORD)pNTHeader + sizeof(IMAGE_NT_HEADERS));
for(int i = 0; i < pNTHeader->FileHeader.NumberOfSections; i++)
{
if(dwRVA >= pSection[i].VirtualAddress && dwRVA < (pSection[i].VirtualAddress + pSection[i].SizeOfRawData))
{
return pSection[i].PointerToRawData + (dwRVA - pSection[i].VirtualAddress);
}
}
return 0;
}
void relocation(PVOID pFileBuffer)
{
BYTE secName[9] = {0};
PIMAGE_DOS_HEADER pDosHeader;
PIMAGE_NT_HEADERS pNtHeaders;
PIMAGE_BASE_RELOCATION pBaseRec;
PIMAGE_SECTION_HEADER pSectionHeader;
pDosHeader = (PIMAGE_DOS_HEADER)pFileBuffer;
pNtHeaders = (PIMAGE_NT_HEADERS)((DWORD)pFileBuffer+pDosHeader->e_lfanew);
printf("重定位表的相对虚拟地址：%x\n",pNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
printf("重定位表的大小：%x\n",pNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size);
DWORD pBaseRecOffset = RvaToFoa(pNtHeaders, pNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress); //注意这个VirtualAddress是VA
pBaseRec = (PIMAGE_BASE_RELOCATION)(pBaseRecOffset+(DWORD)pFileBuffer);
pSectionHeader = (PIMAGE_SECTION_HEADER)(pNtHeaders+1);
for(int i=0; pBaseRec->SizeOfBlock && pBaseRec->VirtualAddress; i++)
{
DWORD FOA = RvaToFoa(pNtHeaders, pBaseRec->VirtualAddress);
DWORD size = (pBaseRec->SizeOfBlock - 8 )/2; // VirtualAddress SizeOfBlock共占8字节，每个项2字节
//确定该结构所属的节
for(DWORD j=0;j<pNtHeaders->FileHeader.NumberOfSections;j++)
{
DWORD lower =RvaToFoa(pNtHeaders, pSectionHeader[j].VirtualAddress);
DWORD upper =RvaToFoa(pNtHeaders, pSectionHeader[j].VirtualAddress+pSectionHeader[j].Misc.VirtualSize);
if(FOA>=lower && FOA<=upper )
{
memcpy(secName,pSectionHeader[j].Name,8);
break;
}
}
printf("第%d块.Relocation\n VirtualAddress %x(%s)\n SizeOfBlock %x\n", i+1, pBaseRec->VirtualAddress,secName, size); //打印本页的主要信息
printf("RVA,TYPE\n");
//打印一个页中，所有重定位地址与信息
WORD * recAddr = (WORD *)((BYTE *)pBaseRec+8); //指向第一个目录项
//如果高4位是3则取后12位地址加上VirtualAddress才是真正需要修复的数据的Rva
for(j=0; j<size; j++)
{
DWORD offset = (recAddr[j] & 0x0FFF) + FOA ;
WORD type = recAddr[j] >> 12;
if(type!=0)
{
printf("%08X, %x\n",offset+pBaseRec->VirtualAddress,type);
}
}
memset(secName, 0, 9);
pBaseRec = (PIMAGE_BASE_RELOCATION )((BYTE *)pBaseRec + pBaseRec->SizeOfBlock);//移到下一页
}
}
int main(int argc, char* argv[])
{
char filepath[] = "F:\\pwn\\test\\TTTT.dll";
PVOID pFileBuffer = NULL;
ReadPEFile(filepath,&pFileBuffer);
relocation(pFileBuffer);
return 0;
}
```

## 1. 为什么出现重定位表？

### 一个软件一般由一个`exe`和多个`dll`组成，当一个`exe`运行的时候`exe`文件先根据`ImageBase`来载入内存，`dll`也是属于`pe`文件，首先是`exe`先载入内存，多个`dll`文件中的`ImageBase`也许会被其他`dll`占取，又因为编译以后地址是写死的，`dll`文件中的`ImageBase`被占住，只能占别的地方，因为地址写死，占的不是原来的地址，所以需要重定位表来标明需要修改的地址

## 2. 在可选`pe`目录中，最后16个结构体的，第6个是是重定位表

#### 注意：`VirtualAddress`是`RVA`，如在文件中使用（不是载入内存）则需要转换为`FOA`

![](https://bbs.kanxue.com/upload/attach/202010/889006_WK2CBG6PGQJVK28.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/889006_UBZAUGPFG7PZ26R.jpg)

#### 第一个结构中的`VirtualAddress`指向的是第二个结构体的开始

#### 第二个结构中的`VirtualAddress`指向的是重定位表的开始，`SizeOfBlock`表示块的大小

## 3. 重定位表的结构

![](https://bbs.kanxue.com/upload/attach/202010/889006_96SMKNRTZFGUJHA.jpg)

### ① 判断一共有几块数据

#### 最后一个结构的`virtualAddress`与`SizeOfBlock`都为0，则结束。

### ② `0011101010101010`是第一项，先判断高四位`0011`

#### 值为3 代表的是需要修改的数据

#### 值为0 代表的是用于数据对齐的数据，可以不用修改

### 当高4位是3则低12位`101010101010`是需要修改的地方的偏移地址

### 真正的`RVA` = `VIrtualAddress` + 具体项的低地址

### ③ 具体项的数量 =（`SizeOfBlock`-8）/2

### ④ 下一个起始地址 = `VirtualAddress` + `SizeOfBlock`

|  |  |

| --- | --- |
