
```
PeStructOptionHeader->ImageBase=NewImagebase;

DWORD* Item=NULL;

WORD* Data=NULL;

while(PeStructReLocatHeader->VirtualAddress&&PeStructReLocatHeader->SizeOfBlock){

Data=(WORD*)((DWORD)PeStructReLocatHeader+8);

for(int i=0;i<(PeStructReLocatHeader->SizeOfBlock-8)/2;i++){

if((Data[i]&0xF000)==0x3000){

Item=(DWORD*)((DWORD)MemAddr+PeStructReLocatHeader->VirtualAddress+(Data[i]&0x0FFF));

*Item=*Item+(DWORD)NewImagebase-oldImageBase;

}

}

PeStructReLocatHeader=(PIMAGE_BASE_RELOCATION)((DWORD)PeStructReLocatHeader+PeStructReLocatHeader->SizeOfBlock);

}

return 1;

}

int RestoreIATTaber(DWORD MemBase){

PIMAGE_DOS_HEADER PeStructDosHeader=NULL;

PIMAGE_FILE_HEADER PeStructFileHeader=NULL;

PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;

PIMAGE_BASE_RELOCATION PeStructReLocatHeader=NULL;

PIMAGE_IMPORT_DESCRIPTOR PeStructImPortHeader=NULL;

PeStructDosHeader=(PIMAGE_DOS_HEADER)MemBase;

PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);

PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

if(PeStructOptionHeader->DataDirectory[1].VirtualAddress){

PeStructImPortHeader=(PIMAGE_IMPORT_DESCRIPTOR)((DWORD)MemBase+PeStructOptionHeader->DataDirectory[1].VirtualAddress);

}

else{

MessageBox(NULL,"不存在导入表",0,0);

return 0;

}

char* DLLName=NULL;

HMODULE hMoudle=NULL;

DWORD* INT=NULL;

DWORD* IAT=NULL;

char *FuntionName=NULL;

while(PeStructImPortHeader->FirstThunk&&PeStructImPortHeader->OriginalFirstThunk){

DLLName=(char*)(MemBase+PeStructImPortHeader->Name);

hMoudle=LoadLibrary(DLLName);

if(NULL==hMoudle){

MessageBox(NULL,"LoadLibrary 加载DLL失败",0,0);

return 0;

}

INT=(DWORD*)((DWORD)MemBase+PeStructImPortHeader->OriginalFirstThunk);

IAT=(DWORD*)((DWORD)MemBase+PeStructImPortHeader->FirstThunk);

while(*INT){

if((*INT)&0x80000000==0x80000000){

}

else{

FuntionName=(char*)(MemBase+*INT+2);

*IAT=(DWORD)GetProcAddress(hMoudle,FuntionName);

if(!*INT){

MessageBox(NULL,"GetProcAddress通过函数名获取函数地址出现错误",0,0);

return 0;

}

}

INT++;

IAT++;

}

PeStructImPortHeader++;

}

DWORD jmpip=(DWORD)MemBase+PeStructOptionHeader->AddressOfEntryPoint;

__asm{

jmp jmpip;

}

return 100;

}

int ImageBuffToFileBuff(){

PIMAGE_DOS_HEADER PeStructDosHeader=NULL;

PIMAGE_FILE_HEADER PeStructFileHeader=NULL;

PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;

PIMAGE_SECTION_HEADER PeStructSetionHeader=NULL;

PeStructDosHeader=(PIMAGE_DOS_HEADER)Mem2;

PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);

PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

PeStructSetionHeader=(PIMAGE_SECTION_HEADER)((DWORD)PeStructOptionHeader+PeStructFileHeader->SizeOfOptionalHeader);

DWORD VirtualAddress=0;

DWORD PointerToRawData=0;

DWORD SizeOfRawData=0;

//memcpy(Mem1,0,FileSize);

memcpy(Mem1,Mem2,PeStructOptionHeader->SizeOfHeaders); //cpy PE Head

for(int i=0;i<PeStructFileHeader->NumberOfSections;i++){

VirtualAddress=PeStructSetionHeader->VirtualAddress;

PointerToRawData=PeStructSetionHeader->PointerToRawData;

SizeOfRawData=PeStructSetionHeader->SizeOfRawData;

memcpy(Mem1+PointerToRawData,Mem2+VirtualAddress,SizeOfRawData);

PeStructSetionHeader++;

}

return 1;

}

int SaveFile(){

HANDLE hFile=CreateFile(SAVE,

GENERIC_READ|GENERIC_WRITE,

FILE_SHARE_READ|FILE_SHARE_WRITE,

NULL,

CREATE_ALWAYS, //如果文件存在则重写

FILE_ATTRIBUTE_NORMAL,

NULL);

if(NULL==hFile){

MessageBox(NULL,"打开文件失败",0,0);

return 0;

}

DWORD Size=0;

if(!WriteFile(hFile,Mem1,FileSize,&Size,NULL)){

int c=GetLastError();

MessageBox(NULL,"WriteFile写入文件失败",0,0);

return 0;

}

HeapFree(Heap,HEAP_NO_SERIALIZE,Mem1);

CloseHandle(hFile);

}

int main(int argc, char* argv[])

{

if(!ReadCurrentImageBuff()){ //读取自己的imagebuf 到mem3内存没有修复重定位

return 0;

}

if(!ReadData()){ //读取磁盘上一个文件的FIleBuf 保存在开辟Mem1空间中个

return 0;

}

GetSizeofImagSize(); //获取这个磁盘的OEP imagebase sizeof等信息

if(!MallocMemroy()){ //开辟一个Mem3准备保存 Mem1拉伸后的内容

return 0;

}

FIleBufToImageBuff(); //将FIleBuf拉伸成 ImageBuf 并保存在Mme2中 此时Mme2和Mem3都没修复重定位

#if 0

if(!RestoreIATTaber()){

return -4;

}

if(!RestoreReLcationTaber()){ //修复重定位

return 0;

}

ImageBuffToFileBuff();

if(!SaveFile()){

return -5;

}

#endif

OpenDestProcess();

printf("Hello World!\n");

return 0;

}
```

各位大佬们，我在做内存写入注入的时候，使用的方法是首先，通过自己写的PE解释器，查看进程PID，这个就是写入进程了，然后通过一个进程读取，一个磁盘exe，拉伸，修复重定位后，注入目标进程，由于还没有修复IAT表所，在把自己镜像修复重定位后注入进去(为什么要注入两个镜像呢)，因为自己的镜像里有修复IAT的函数，在创建远程线程执行 修复IAT的函数，修复IAT的函数最后有一个JMP跳转到  `IMAGEBSE+OEP` 执行真正要执行的注入的程序了，（注意一定要在目标进程中修复IAT  因为DLL必须要加载进去，否则无法执行的）

注入都是可以成功的也是可以执行的，通过了测试，但是我把一个Win32窗口程序注入控制台程序,Win32窗口程序却弹不出窗口不知道为什么？

而且吧一个控制台循环产生MessageBox的程序，注入Win32成功 会不停打印MessageBox，求教一个看雪的大佬们，以前辈们见多识广的眼光，如果能指点一二，相信对我就收获良多，下面的代码共享了自己写的没参考 注入了两次镜像傻的狠,希望也能对一些朋友产生一些帮助 ,还有程序里的开辟的指针都么有释放，时间紧

```cpp
// 内存写入注入进程.cpp : Defines the entry point for the console application.
//
// 贴入exe.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <Windows.h>

#define PID 0x9E8
#define EXEPATH "C:\Documents and Settings\Administrator\桌面\测试贴入.exe"
#define SAVE "C:\Documents and Settings\Administrator\桌面\1111.exe"
#define BASE 0x1000000

char *Mem1=NULL; //读取磁盘文件的FileBuf
char* Mem2=NULL; //将磁盘问的Filebuf拉伸成ImageBuf
char* Mem3=NULL; //自己进程的Imagebase 因为IAT表代表在自己的进程中

DWORD FileSize=0;
DWORD ImageBase=0;
DWORD SizeofImage=0;
DWORD CurrentImageBase=0;
DWORD CuurentSizeofImage=0;
DWORD OEP=0;
HANDLE Heap=0;

int SaveFile();
int RestoreIATTaber(DWORD MemBase);
int ImageBuffToFileBuff();
int RestoreReLcationTaber(DWORD MemAddr,DWORD NewImagebase,DWORD loImageBase);

int ReadCurrentImageBuff(){

    HMODULE Hmoudle=GetModuleHandle(NULL);
    if(NULL==Hmoudle){
        MessageBox(NULL,"获取当前进程模块失败",0,0);
        return 0;
    }

    PIMAGE_DOS_HEADER PeStructDosHeader=NULL;
    PIMAGE_FILE_HEADER PeStructFileHeader=NULL;
    PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;

    PeStructDosHeader=(PIMAGE_DOS_HEADER)Hmoudle;
    PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);
    PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

    CurrentImageBase=PeStructOptionHeader->ImageBase;
    CuurentSizeofImage=PeStructOptionHeader->SizeOfImage;

    Mem3=(char*)VirtualAlloc(NULL,CuurentSizeofImage,MEM_RESERVE | MEM_COMMIT,PAGE_EXECUTE_READWRITE);
    if(NULL==Mem3){
        int c=GetLastError();
        MessageBox(0,"VirtualAlloc开辟保存Current进程imagebuf失败",0,0);
        return 0;
    }
    memcpy(Mem3,Hmoudle,CuurentSizeofImage);

    return 1;
}

int ReadData(){

    HANDLE hFile=CreateFile(EXEPATH,
        GENERIC_READ|GENERIC_WRITE,
        FILE_SHARE_READ|FILE_SHARE_WRITE,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL);

    if(NULL==hFile){
        MessageBox(NULL,"打开文件失败",0,0);
        return 0;
    }

    FileSize=GetFileSize(hFile,NULL);
    if(INVALID_FILE_SIZE==FileSize){
        MessageBox(NULL,"获取函数大小失败",0,0);
        CloseHandle(hFile);
        return 0;
    }

    Heap=HeapCreate(0,FileSize,0);
    if(NULL==Heap){
        MessageBox(NULL,"创建进程私有堆失败",0,0);
        CloseHandle(hFile);
        return 0;
    }

    Mem1=(char*)HeapAlloc(Heap,HEAP_ZERO_MEMORY,FileSize);    //全都初始化为0
    if(NULL==Mem1){
        MessageBox(NULL,"HeapAlloc开辟堆内存失败",0,0);
        CloseHandle(hFile);
        return 0;
    }

    DWORD LpSize=0;
    if(!ReadFile(hFile,Mem1,FileSize,&LpSize,NULL)){
        MessageBox(NULL,"ReadFile函数失败",0,0);
        CloseHandle(hFile);
        return 0;
    }

    if(FileSize!=LpSize){
        MessageBox(0,"读取文件字节数和预期不一致",0,0);
        return 0;
    }

    CloseHandle(hFile);
    return 1;
}

DWORD GetSizeofImagSize(){

    PIMAGE_DOS_HEADER PeStructDosHeader=NULL;
    PIMAGE_FILE_HEADER PeStructFileHeader=NULL;
    PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;
```
