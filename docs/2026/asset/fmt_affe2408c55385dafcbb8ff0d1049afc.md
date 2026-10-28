
# [原创][求助]内存写入注入

各位大佬们，我在做内存写入注入的时候，使用的方法是首先，通过自己写的PE解释器，查看进程`PID`，这个就是写入进程了，然后通过一个进程读取，一个磁盘`exe`，拉伸，修复重定位后，注入目标进程，由于还没有修复`IAT`表所，在把自己镜像修复重定位后注入进去(为什么要注入两个镜像呢)，因为自己的镜像里有修复`IAT`的函数，在创建远程线程执行 修复`IAT`的函数，修复`IAT`的函数最后有一个`JMP`跳转到 `IMAGEBSE+OEP` 执行真正要执行的注入的程序了，（注意一定要在目标进程中修复`IAT` 因为`DLL`必须要加载进去，否则无法执行的）

注入都是可以成功的也是可以执行的，通过了测试，但是我把一个`Win32`窗口程序注入控制台程序,`Win32`窗口程序却弹不出窗口不知道为什么？

而且吧一个控制台循环产生`MessageBox`的程序，注入`Win32`成功 会不停打印`MessageBox`，求教一个看雪的大佬们，以前辈们见多识广的眼光，如果能指点一二，相信对我就收获良多，下面的代码共享了自己写的没参考 注入了两次镜像傻的狠,希望也能对一些朋友产生一些帮助 ,还有程序里的开辟的指针都么有释放，时间紧

## 内存写入注入进程.cpp

```cpp
// 内存写入注入进程.cpp : Defines the entry point for the console application.

//

// 贴入exe.cpp : Defines the entry point for the console application.

//

#include "stdafx.h"

#include <Windows.h>

#define PID 0x9E8

#define EXEPATH "C:\\Documents and Settings\\Administrator\\桌面\\测试贴入.exe"

#define SAVE "C:\\Documents and Settings\\Administrator\\桌面\\1111.exe"

#define BASE 0x1000000

char* Mem1=NULL; //读取磁盘文件的FileBuf

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

Mem1=(char*)HeapAlloc(Heap,HEAP_ZERO_MEMORY,FileSize); //全都初始化为0

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

PeStructDosHeader=(PIMAGE_DOS_HEADER)Mem1;

PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);

PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

OEP=PeStructOptionHeader->AddressOfEntryPoint;

ImageBase=PeStructOptionHeader->ImageBase;

SizeofImage=PeStructOptionHeader->SizeOfImage;

return 1;

}

DWORD MallocMemroy(){

Mem2=(char*)VirtualAlloc((char*)BASE,SizeofImage,MEM_RESERVE | MEM_COMMIT,PAGE_EXECUTE_READWRITE);

if(NULL==Mem2){

int c=GetLastError();

MessageBox(0,"VirtualAlloc开辟内存失败",0,0);

return 0;

}

return 1;

}

int FIleBufToImageBuff(){

PIMAGE_DOS_HEADER PeStructDosHeader=NULL;

PIMAGE_FILE_HEADER PeStructFileHeader=NULL;

PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;

PIMAGE_SECTION_HEADER PeStructSetionHeader=NULL;

PeStructDosHeader=(PIMAGE_DOS_HEADER)Mem1;

PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);

PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

PeStructSetionHeader=(PIMAGE_SECTION_HEADER)((DWORD)PeStructOptionHeader+PeStructFileHeader->SizeOfOptionalHeader);

DWORD VirtualAddress=0;

DWORD PointerToRawData=0;

DWORD SizeOfRawData=0;

memcpy(Mem2,Mem1,PeStructOptionHeader->SizeOfHeaders); //cpy PE Head

for(int i=0;i<PeStructFileHeader->NumberOfSections;i++){

VirtualAddress=PeStructSetionHeader->VirtualAddress;

PointerToRawData=PeStructSetionHeader->PointerToRawData;

SizeOfRawData=PeStructSetionHeader->SizeOfRawData;

memcpy(Mem2+VirtualAddress,Mem1+PointerToRawData,SizeOfRawData);

PeStructSetionHeader++;

}

return 1;

}

int InjectMem2(HANDLE hProcess){ //Mem2 注入磁盘文件

char* Dest2=(char*)VirtualAllocEx(hProcess,NULL,SizeofImage,MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

if(NULL==Dest2){

MessageBox(NULL,"目标进程开辟第一块空间失败",0,0);

return 0;

}

if(!RestoreReLcationTaber((DWORD)Mem2,(DWORD)Dest2,ImageBase)){

return 0;

}

DWORD length=0;

if(!WriteProcessMemory(hProcess,Dest2,Mem2,SizeofImage,&length)){

int c=GetLastError();

MessageBox(NULL,"向目标进程中写入镜像失败",0,0);

return 0;

}

if(ImageBase==length){

MessageBox(NULL,"向目标进程中写入字节数不对",0,0);

return 0;

}

memset(Mem2,0,SizeofImage);

if(!ReadProcessMemory(hProcess,Dest2,Mem2,SizeofImage,&length)){

MessageBox(NULL,"读取目表进程内容失败",0,0);

return 0;

}

if(ImageBase==length){

MessageBox(NULL,"从目标进程中读取字节数不对",0,0);

return 0;

}

ImageBuffToFileBuff();

if(!SaveFile()){

return -5;

}

return (DWORD)Dest2;

}

int InjectMem3(HANDLE hProcess,DWORD PosBase){ //注入字节 Mem3是自己

char* Dest1=(char*)VirtualAllocEx(hProcess,NULL,CuurentSizeofImage,MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

if(NULL==Dest1){

MessageBox(NULL,"目标进程开辟第一块空间失败",0,0);

return 0;

}

if(!RestoreReLcationTaber((DWORD)Mem3,(DWORD)Dest1,CurrentImageBase)){

return 0;

}

DWORD length=0;

if(!WriteProcessMemory(hProcess,Dest1,Mem3,CuurentSizeofImage,&length)){

int c=GetLastError();

MessageBox(NULL,"向目标进程中写入镜像失败",0,0);

return 0;

}

if(ImageBase==length){

MessageBox(NULL,"向目标进程中写入字节数不对",0,0);

return 0;

}

typedef int(*ADDR)(DWORD);

DWORD Rva=(DWORD)RestoreIATTaber-CurrentImageBase;

ADDR addr=(int (__cdecl *)(unsigned long))(Rva+Dest1);

HANDLE hThread=CreateRemoteThread(hProcess,0,0,(LPTHREAD_START_ROUTINE)addr,(void*)PosBase,0,0);

if(NULL==hThread){

MessageBox(NULL,"创建远程线程失败",0,0);

return 0;

}

DWORD Code=0;

WaitForSingleObject(hThread,-1); //永久等待直到线程返回

GetExitCodeThread(hThread,&Code);

}

int OpenDestProcess(){

HANDLE hProcess=OpenProcess( PROCESS_ALL_ACCESS,FALSE,PID);

if(NULL==hProcess){

MessageBox(NULL,"打开进程失败",0,0);

return 0;

}

DWORD Pos=0;

if(!(Pos=InjectMem2(hProcess))){ //先注入磁盘镜像

return 0;

}

if(!(InjectMem3(hProcess,Pos))){ //在注入自己的镜像

return 0;

}

}

int RestoreReLcationTaber(DWORD MemAddr,DWORD NewImagebase,DWORD oldImageBase){

PIMAGE_DOS_HEADER PeStructDosHeader=NULL;

PIMAGE_FILE_HEADER PeStructFileHeader=NULL;

PIMAGE_OPTIONAL_HEADER PeStructOptionHeader=NULL;

PIMAGE_BASE_RELOCATION PeStructReLocatHeader=NULL;

PeStructDosHeader=(PIMAGE_DOS_HEADER)MemAddr; //Mem2获取Mem3的其实地址

PeStructFileHeader=(PIMAGE_FILE_HEADER)((DWORD)PeStructDosHeader+PeStructDosHeader->e_lfanew+4);

PeStructOptionHeader=(PIMAGE_OPTIONAL_HEADER)((DWORD)PeStructFileHeader+sizeof(IMAGE_FILE_HEADER));

if(PeStructOptionHeader->DataDirectory[5].VirtualAddress){

PeStructReLocatHeader=(PIMAGE_BASE_RELOCATION)((DWORD)MemAddr+PeStructOptionHeader->DataDirectory[5].VirtualAddress);

}

else{

MessageBox(NULL,"不存在重定位表",0,0);

return 0;

}
```
