
# windows下程序加壳、内存加载运行的实验

<a href="elink@bbcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5i4c8S2k6s2u0A6N6X3g2J5i4K6u0r3f1p5g2e0K9r3g2D9L8l9`.`.">源码地址点击此处</a>

第一次在此版发帖，更多的是涉及内存加载执行。

该方案比较简单，只适用于测试和简单情景下的应用，测试中能够运行大多数体积和功能比较简单的32位/64位程序，兼容性和稳定性有待测试和提高。

## （一）原理

源码包含两个工程，加壳程序`PeShell`和脱壳程序`PeUnshell`。

### 加壳程序

`PeShell`工程。该工程根据命令行参数，将需要加壳的程序和文件先用`zip`压缩，后用`xor`加密，生成一段新的数据，接着在`PEUnshell.exe`程序中创建一个新的段区，名称默认为`ldata`，最后将这段数据写入在`PEUnshell.exe`文件的新的`ldata`段中。这块加密数据的内存影像（从低地址到高地址）为：

文件个数（`int`型），密钥（`char`型，16字节，用的当前时间的`md5`值），文件大小1（`int`），文件名1（`char`型，64字节），文件大小2（`int`），文件名2（`char`型，64字节），……，文件大小n（`int`），文件名n（`char`型，64字节）。

代码实现：

```c
unsigned char* Crypto::makeDataBlock(int flag, const char filename[MAX_FILE_COUNT][256], int cnt, int& dstdatasize) {
int ret = 0;
int filesize = 0;
for (int i = 0; i < cnt; i++)
{
int fz = FileHelper::getfilesize(filename[i]);
filesize += fz;
printf("file name:%s size:%d\r\n", filename[i], fz);
}
int dstbufsize = filesize + 0x1000;
unsigned char* dstblock = new unsigned char[dstbufsize];
*(int*)dstblock = flag;
// if (cnt == 1 && strstr((char*)filename[0],".exe") )
// {
// *(int*)dstblock = ONLY_ONE_EXE;
// }
// else if (cnt == 1 && strstr((char*)filename[0], ".dll"))
// {
// *(int*)dstblock = ONLY_ONE_DLL;
// }
// else if (cnt > 1 )
// {
// int flagexe = 0;
// int flagdll = 0;
// for (int i = 0;i < cnt; i ++)
// {
// if (strstr((char*)filename[i], ".dll")) {
// flagdll = 1;
// }else if (strstr(filename[i],".exe"))
// {
// flagexe = 1;
// }
// }
//
// if (flagexe && flagdll)
// {
// *(int*)dstblock = ONE_EXE_AND_ONE_DLL;
// }
// else {
// *(int*)dstblock = SOME_OTHER_FILES;
// }
// }
// else {
// return 0;
// }
unsigned char* key = dstblock + 4;
getkey(key);
*(int*)(dstblock + 4 + CRYPT_KEY_SIZE) = cnt;
unsigned char* dstbuf = dstblock + 4 + CRYPT_KEY_SIZE + 4;
int dstbuflimit = dstbufsize - 4 - CRYPT_KEY_SIZE - 4;
for (int i = 0; i < cnt; i++)
{
lstrcpyA((char*)dstbuf, filename[i]);
PathStripPathA((char*)dstbuf);
dstbuf += FILENAME_LEN;
dstbuflimit -= FILENAME_LEN;
char* lpdata = 0;
ret = FileHelper::fileReader(filename[i], &lpdata, &filesize);
if (ret > 0)
{
unsigned long cmpresssize = dstbuflimit - 4;
ret = Compress::compressdata((unsigned char*)lpdata, filesize, dstbuf + 4, &cmpresssize);
delete[] lpdata;
if (ret != 0)
{
delete dstblock;
printf("compress file:%s error:%u\r\n", filename[i], GetLastError());
return 0;
}
*(int*)(dstbuf) = cmpresssize;
dstbuf += 4;
dstbuf += cmpresssize;
dstbuflimit -= 4;
dstbuflimit -= cmpresssize;
}
else {
delete dstblock;
printf("read file:%s error\r\n", filename[i]);
return 0;
}
}
dstdatasize = dstbuf - dstblock;
CryptData(dstblock + 4 + CRYPT_KEY_SIZE, dstdatasize - 4 - CRYPT_KEY_SIZE, key, CRYPT_KEY_SIZE);
//revertkey(key);
return dstblock;
}
```

加壳命令行的写法：

```
PeShell -be sogou.exe sbiedll.dll -c params.dat -o out.exe

PeShell -bd sogou.exe sbiedll.dll -c params.dat -o out.dll

PeShell -e sogou.exe -c params.dat -o out.exe

PeShell -d sogou.exe -c params.dat -o out.dll
```

参数的解释：

`-be`：`b`代表`bind`，绑定的意思；`e`代表`executable`，可执行的。

`-bd`：`b`代表`bind`，绑定的意思；`d`代表`dll`，即动态重定位文件。

`-o`：输出文件，即加壳后的程序。

`-c`：参数。

加壳程序比较复杂的代码是添加区段。`pe`文件的区段结构如下：

```c
typedef struct _IMAGE_SECTION_HEADER {
BYTE Name[IMAGE_SIZEOF_SHORT_NAME];
union {
DWORD PhysicalAddress;
DWORD VirtualSize;
} Misc;
DWORD VirtualAddress;
DWORD SizeOfRawData;
DWORD PointerToRawData;
DWORD PointerToRelocations;
DWORD PointerToLinenumbers;
WORD NumberOfRelocations;
WORD NumberOfLinenumbers;
DWORD Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

其中，`VirtualSize`是区段大小，`VirtualAddress`是对齐后的虚拟地址，`SizeOfRawData`是文件对齐后的大小，`PointerToRawData`是区段在文件中相对于文件头的偏移地址。这里采用的思路是，一般情况下，段表结构`PIMAGE_SECTION_HEADER`中有空余的项，找到空项后，计算和重新修改该段表项即可。其中，

`VirtualAddress`字段=上一个段表的值+上一个段表内存页面对齐后的大小，

`PointerToRawData` = 上一个段表的值+上一个段表扇区对齐后的大小，

`SizeOfRawData`是段文件对齐后大小，

`VirtualSize`是段实际大小，

`Characteristics`必须是代码或者数据段，否则不会被装入内存，

`IMAGE_FILE_HEADER`中`NumberOfSections`字段加1，

`IMAGE_OPTIONAL_HEADER32`中`SizeOfImage`字段要加上该段对齐后的大小。

上述功能中使用的结构体如下：

```c
typedef struct _IMAGE_FILE_HEADER {
WORD Machine;
WORD NumberOfSections;
DWORD TimeDateStamp;
DWORD PointerToSymbolTable;
DWORD NumberOfSymbols;
WORD SizeOfOptionalHeader;
WORD Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
typedef struct _IMAGE_OPTIONAL_HEADER {
WORD Magic;
BYTE MajorLinkerVersion;
BYTE MinorLinkerVersion;
DWORD SizeOfCode;
DWORD SizeOfInitializedData;
DWORD SizeOfUninitializedData;
DWORD AddressOfEntryPoint;
DWORD BaseOfCode;
DWORD BaseOfData;
DWORD ImageBase;
DWORD SectionAlignment;
DWORD FileAlignment;
WORD MajorOperatingSystemVersion;
WORD MinorOperatingSystemVersion;
WORD MajorImageVersion;
WORD MinorImageVersion;
WORD MajorSubsystemVersion;
WORD MinorSubsystemVersion;
DWORD Win32VersionValue;
DWORD SizeOfImage;
DWORD SizeOfHeaders;
DWORD CheckSum;
WORD Subsystem;
WORD DllCharacteristics;
DWORD SizeOfStackReserve;
DWORD SizeOfStackCommit;
DWORD SizeOfHeapReserve;
DWORD SizeOfHeapCommit;
DWORD LoaderFlags;
DWORD NumberOfRvaAndSizes;
IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

至此，加壳程序工作基本完成。

### 解壳和内存加载执行

即`PEUnshell`工程。该工程支持编译为`PEUnshell.exe`和`PEUnshell.dll`，可以根据命令行参数，将程序隐藏在`exe`或者`dll`中，并且都可以正确的释放和加载运行。在程序运行时（`PEUnshell.exe`是在`WinMain`函数入口处，`PEUnshell.dll`是在`DllEntry`入口处）在自己的内存加载映像中查找`ldata`区段，然后按照加壳中定义的数据结构，将这段压缩加密内存数据解密解压后，写入当前目录，并将其中后缀名为`exe`的程序启动起来。启动方法是内存加载执行。内存加载执行主要分为三个部分，段加载，导入表填充，重定位。

其中，程序释放程序跟加壳程序压缩加密和内存布局是相反的。

内存加载执行代码如下：
