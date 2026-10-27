
```c
#include "stdafx.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int\* OpenFile()
{
FILE\* PointToFile = NULL;
int FileSize = 0;
int\* StrBuffer = NULL;
int Num = 0;
//打开文件
if ((PointToFile = fopen("C:\\notepad.exe","rb")) == NULL) {
printf("打开文件失败!\n");
exit(1);
}
//获取文件大小
fseek(PointToFile,0,2);
FileSize = ftell(PointToFile);
//重定位指针
fseek(PointToFile,0,0);
//buffer指向申请的堆
StrBuffer = (int\*)(malloc(FileSize));
if (!StrBuffer)
{
printf("堆空间分配失败!\n");
free(StrBuffer);
return 0;
}
//读取文件内容
Num = fread(StrBuffer,FileSize,1,PointToFile);
if (!Num)
{
printf("读取文件内容失败!\n");
free(StrBuffer);
return 0;
}
//关闭文件
fclose(PointToFile);
//将缓冲区内的文件内容的地址返回到调用函数的地方
return StrBuffer;
}
int\* FileSizes = OpenFile();
int PrintfNtHeaders()
{
//文件指针
unsigned int\* PointBuffer = (unsigned int\*)FileSizes;
unsigned short\* pBuffer = (unsigned short\*)PointBuffer;
unsigned char\* pcBuffer = (unsigned char\*)PointBuffer;
//判断MZ和PE的标志
unsigned short Cmp1 = 0x5A4D;
unsigned int Cmp2 = 0x00004550;
//判断文件是否读取成功
if(!PointBuffer)
{
printf("文件读取失败！\n");
free(PointBuffer);
return 0;
}
//判断是否为MZ标志
if (\*pBuffer != Cmp1)
{
printf("不是有效MZ标志！\n");
printf("%X\n",\*pBuffer);
free(PointBuffer);
return 0;
}
printf("\*\*\*\*\*\*\*\*\*打印DOS头\*\*\*\*\*\*\*\*\*\n");
printf("e\_magic:\t\t\t%X\n",\*(pBuffer));
printf("e\_ifanew:\t\t\t%08X\n\n\n",\*(PointBuffer+15));
//判断是否为PE标志
if (\*(PointBuffer+56) != Cmp2)
{
printf("不是有效的PE标志！\n");
printf("%X\n",\*(PointBuffer+56));
free(PointBuffer);
return 0;
}
printf("\*\*\*\*\*\*\*\*\*打印标准PE文件头\*\*\*\*\*\*\*\*\*\n");
printf("PE标志:\t\t\t\t%X\n",\*(PointBuffer+56));
printf("Machine:\t\t\t%04X\n",\*(pBuffer+114));
printf("NumberOfSection:\t\t%04X\n",\*(pBuffer+115));
printf("TimeDateStamp:\t\t\t%08X\n",\*(PointBuffer+58));
printf("PointerToSymbolTable:\t\t%08X\n",\*(PointBuffer+59));
printf("NumberOfSymbols:\t\t%08X\n",\*(PointBuffer+60));
printf("SizeOfOptionalHeader:\t\t%04X\n",\*(pBuffer+122));
printf("Chrarcteristics:\t\t%04X\n\n\n",\*(pBuffer+123));
printf("\*\*\*\*\*\*\*\*\*打印标准可选PE头\*\*\*\*\*\*\*\*\*\n");
printf("Magic:\t\t\t\t%04X\n", \*(pBuffer+124));
printf("MajorLinkerVersion:\t\t%02X\n", \*(pcBuffer+250));
printf("MinorLinkerVersion:\t\t%02X\n", \*(pcBuffer+251));
printf("SizeOfCode:\t\t\t%08X\n", \*(PointBuffer+63));
printf("SizeOfInitializedData:\t\t%08X\n", \*(PointBuffer+64));
printf("SizeOfUninitializedData:\t%08X\n", \*(PointBuffer+65));
printf("AddressOfEntryPoint:\t\t%08X\n", \*(PointBuffer+66));
printf("BaseOfCode:\t\t\t%08X\n", \*(PointBuffer+67));
printf("BaseOfData:\t\t\t%08X\n", \*(PointBuffer+68));
printf("ImageBase:\t\t\t%08X\n", \*(PointBuffer+69));
printf("SectionAlignment:\t\t%08X\n", \*(PointBuffer+70));
printf("FileAlignment:\t\t\t%08X\n", \*(PointBuffer+71));
printf("MajorOperatingSystemVersion:\t%04X\n", \*(pBuffer+144));
printf("MinorOperatingSystemVersion:\t%04X\n", \*(pBuffer+145));
printf("MajorImageVersion:\t\t%04X\n", \*(pBuffer+146));
printf("MinorImageVersion:\t\t%04X\n", \*(pBuffer+147));
printf("MajorSubsystemVersion:\t\t%04X\n", \*(pBuffer+148));
printf("MinorSubsystemVersion:\t\t%04X\n", \*(pBuffer+149));
printf("Win32VersionValue:\t\t%08X\n", \*(PointBuffer+75));
printf("SizeOfImage:\t\t\t%08X\n", \*(PointBuffer+76));
printf("SizeOfHeaders:\t\t\t%08X\n", \*(PointBuffer+77));
printf("CheckSum:\t\t\t%08X\n", \*(PointBuffer+78));
printf("Subsystem:\t\t\t%04X\n", \*(pBuffer+158));
printf("DllCharacteristics:\t\t%04X\n", \*(pBuffer+159));
printf("SizeOfStackReserve:\t\t%08X\n", \*(PointBuffer+80));
printf("SizeOfStackCommit:\t\t%08X\n", \*(PointBuffer+81));
printf("SizeOfHeapReserve:\t\t%08X\n", \*(PointBuffer+82));
printf("SizeOfHeapCommit:\t\t%08X\n", \*(PointBuffer+83));
printf("LoaderFlags:\t\t\t%08X\n", \*(PointBuffer+84));
printf("NumberOfRvaAndSizes:\t\t%08X\n", \*(PointBuffer+85));
free(PointBuffer);
return 0;
}
int main()
{
PrintfNtHeaders();
OpenFile();
return 0;
}
```

## 第二种写法
