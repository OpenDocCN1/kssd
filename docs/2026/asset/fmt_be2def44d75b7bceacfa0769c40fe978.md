
# [旧帖][原创]PE结构中插入一段补丁程序0.00雪花

本人菜鸟，学识浅薄，若有纰漏，还望高人指正。

最新学习PE结构，觉得略懂一二，于是想写一个能在PE结构中插入一小段补丁程序的程序，其功能是当PE被加载进内存后，补丁程序先于原程序运行，后才将执行权限转交给原程序。

## 补丁位置示意

补丁在PE中的大致位置如下所示：

```
                   --------------------------------
                   |             DOS 头           |
                   --------------------------------
                   |             DOS Stub         |
                   --------------------------------   <---注意:DOS Stub 和补丁程序有部分重叠
                   |             补丁程序         |
                   --------------------------------
                   |             PE 头            |
                   --------------------------------
                   |             节表             |
                   --------------------------------
                   |              节              |
                   --------------------------------
```

在此需要特别说明的是，这里的补丁程序和`DOS Stub`有部分是重叠的，其目的是使得打了补丁后的程序能更加小。

## 约定

为了便于读者阅读，再次做一个约定:

目标文件:你想给哪个文件打补丁，哪个文件就是目标文件。

结果文件:打了补丁后的目标文件就是结果文件。

## 程序整体思路

1、获取目标文件的大小(以字节为单位)，开辟一个目标文件的缓冲区(记为`ObjectFileBuffer`)，其大小为目标文件的大小，将整个PE文件读入其中。

2、获取目标文件中PE的各种信息，比如PE头的偏移，文件中节的对齐粒度等等。

3、利用获取到的信息，计算结果文件大小。

4、开辟一个结果文件缓冲区(记为`SaveFileBuffer`)，其大小为结果文件大小。

5、将`ObjectFileBuffer`中的`DOS`头和`DOS Stub`复制到`SaveFileBuffer`的起始处。

6、将补丁程序的字节码复制到`SaveFileBuffer`中的适当位置。

7、将`ObjectFileBuffer`中的其余部分复制到`SaveFileBuffer`中的补丁位置的后面。

8、将`SaveFileBuffer`中的字节写入文件中

## 源代码

以下是我的源代码，该源代码在`Windows XP SP2 VC6.0`下编译通过。

```c
/*
*   本程序的功能:在PE空隙中插入一小段补丁程序，PE被加载后，补丁程序先于原程序运行，后将执行权转移给原程序
*   补丁在PE结构中的大致位置:
*                  --------------------------------
*                  |             DOS 头           |
*                  --------------------------------
*                  |             DOS Stub         |
*                  --------------------------------
*                  |             补丁程序         |
*                  --------------------------------
*                  |             PE 头            |
*                  --------------------------------
*                  |             节表             |
*                  --------------------------------
*                  |              节              |
*                  --------------------------------
*
*   注意:为了使程序字节变得更小，在这里补丁程序和DOS Stub部分是重合的
*        本程序在Windows XP SP2 VC6.0下编译通过
*        请在测试本程序时关闭360或者将其添加为信任,否则会给您的测试带来不便
*/

#include <windows.h>
#include <stdio.h>

char cObjectFile[]="C:\\PE\\s.exe";//目标文件
char cSaveFile[]="C:\\PE\\new.exe";//结果文件
char *szFileBuffer=NULL;
char *szOldFile=NULL;

//实用补丁程序字节码
DWORD dwPatchSize;
char* cPatch;

/*
*    请将补丁字节码粘贴在这里,后面跟随若干个90,请确保该_cPatch的缓冲区比较小,
*    如果缓冲区过大，可能会使补丁程序无法正常跳转到原程序入口,
*    或者使补丁不能正常插入到原程序中
*    建议缓冲区大小为8的倍数
*/

//待用补丁程序字节码
DWORD _dwPatchSize=192;
char* _cPatch=new char[_dwPatchSize]=//本补丁的功能是弹出一个cmd,其默认msvcrt.dll已被加载
"\x55"
"\x8b\xec"
"\x33\xff"
"\x57"
"\x83\xec\x08"
"\xc6\x45\xfe\x6d"
"\xc6\x45\xfd\x6f"
"\xc6\x45\xfc\x63"
"\xc6\x45\xfb\x2e"
"\xc6\x45\xfa\x64"
"\xc6\x45\xf9\x6e"
"\xc6\x45\xf8\x61"
"\xc6\x45\xf7\x6d"
"\xc6\x45\xf6\x6d"
"\xc6\x45\xf5\x6f"
"\xc6\x45\xf4\x63"
"\x8d\x45\xf4"
"\x50"
"\xb8\xc7\x93\xbf\x77"
"\xff\xd0"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90";

DWORD dwPatchFOA;//补丁程序FOA
DWORD ddMaxSizePatch;//可以使用的最大容量补丁程序
////////////////////////////
//PE信息
DWORD dwe_lfanew=0x3c;//常量
DWORD ddPEHeaderSize=248;//PE头的大小-常量
DWORD dwSectionHeaderFileOffset=20;//节表头到文件偏移项的偏移-常量
DWORD dwSectionHeaderSize=40;//一个节表项的大小-常量
DWORD ddHeaderAndSectionSize;//头+节表在文件中总大小:DOS头+PE头+节表
LONG e_lfanew=0;
DWORD dwDosSize=0;//DOS头和DOS Stub的总大小
DWORD ddAddressOfEntryPoint;//程序执行入口RVA
WORD wNumberOfSection;//PE中节的数量
DWORD dwSectionTableStart;//节表起始处的FOA
DWORD dwDataDirectorySize=0;//数据目录数组的总大小
DWORD ddNumberOfRvaAndSizes;//数据目录项目个数
DWORD ddSegmentOffset;//节偏移
DWORD ddSectionAlignment;//内存中节的对齐粒度
DWORD ddFileAlignment;//文件中节的对齐粒度
char* ddtextName=new char[6];//.text节的名字
DWORD ddtextSizeOfRawData;//.text节的在文件中对齐后的尺寸
DWORD ddtextFileOffset;//.text节在文件中的偏移
DWORD ddtextRVA;//.text节的RVA
DWORD ddPatchRVA;//补丁起始地址RVA
DWORD ddAddressOfEntryPointXX;//原始程序入口RVA与新入口RVA跳转时的XX
BYTE byE9=233;//长跳转指令字节码 E9 XXXXXXXX
///////////////////////////////////
//导入表
DWORD ddITRVA;//导入表起始RVA
DWORD ddITFOA;//导入表起始FOA
DWORD ddITFileaddress;//导入表在内存中的文件开始地址
DWORD ddITMemoryaddress;//导入表内存的开始地址
DWORD ddIATRVA;//导入函数地址表RVA
///////////////////////////////////
//动态连接库的数量
DWORD ddNumberOfLibrary;
//某一个动态链接库的信息
char* addLibName=new char[100];//指向的动态链接库的名字
DWORD ddValueOfBridge1;//桥1的内容，一个RVA
DWORD ddValueOfBridge2;//桥2的内容
////////////////////////////////

void GetPEInformation(char* file)
{
        HANDLE h;
        DWORD readed;
        h=CreateFile(file,GENERIC_READ | GENERIC_WRITE,0,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);
        //获得文件的大小
        DWORD dwFileSize;
        dwFileSize=GetFileSize(h,NULL);
        //读取原始文件
        szOldFile=new char[dwFileSize+1];
        ReadFile(h,szOldFile,dwFileSize,&readed,NULL);
        __asm{
                pushf
                push ebp
                mov ebp,esp
                //start
                mov eax,dwe_lfanew
                mov ecx,szOldFile
                mov ebx,[eax+ecx]
                mov e_lfanew,ebx;获得e_lfanew
                ;计算DOS头和DOS Stub的总大小
                mov dwDosSize,ebx
                ;获得原始程序入口RVA
                mov eax,ebx
                mov eax,[ecx+ebx+28h]
                mov ddAddressOfEntryPoint,eax
                ;获得内存中节的对齐粒度
                xor eax,eax
                mov eax,dwDosSize
                add eax,38h
                xor ecx,ecx
                mov ecx,szOldFile
                mov ebx,[ecx+eax]
                mov ddSectionAlignment,ebx
                ;获得文件中节的对齐粒度
                add eax,4h
                mov ebx,[ecx+eax]
                mov ddFileAlignment,ebx
                ;计算节偏移
                mov eax,ddSectionAlignment
                mov ebx,ddFileAlignment
                sub eax,ebx
                mov ddSegmentOffset,eax
                ;获得PE中节的数量
                xor eax,eax
                mov ebx,dwDosSize
                mov ecx,szOldFile
                mov ax,[ecx+ebx+6h]
                mov wNumberOfSection,ax
                ;获取数据目录数组的目录项数
                xor eax,eax
                mov eax,dwDosSize
                add eax,74h
                mov ecx,szOldFile
                mov ebx,[ecx+eax]
                mov ddNumberOfRvaAndSizes,ebx
                ;计算.text节-------------------------------------------------不通用
                ;1、在文件的偏移
                ;2、在文件中对齐后的尺寸
                ;3、该节的RVA
                xor eax,eax
                xor ebx,ebx
                xor ecx,ecx
                mov ecx,szOldFile
                add ecx,dwDosSize
                add ecx,ddPEHeaderSize
                mov bl,[ecx+1]
                sub ebx,'t'
                cmp ebx,0
                jnz allover
                mov eax,ddtextName
                mov bl,[ecx]
                mov [eax],bl    ;'.'
                mov bl,[ecx+1]
                mov [eax+1],bl  ;'t'
                mov bl,[ecx+2]
                mov [eax+2],bl  ;'e'
                mov bl,[ecx+3]
                mov [eax+3],bl  ;'x'
                mov bl,[ecx+4]
                mov [eax+4],bl  ;'t'
```

