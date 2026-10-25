
# 程序的自我修改

> 原文：[`www.pediy.com/kssd/pediy10/64033.html`](https://www.pediy.com/kssd/pediy10/64033.html)

本文目的在于向读者说明程序进行自我修改的基本方法，并希望可以起到抛砖引玉的作用。

如果读者有更好的方法或见解，欢迎来信交流 E-mail: `default_and_default@yahoo.cn`

```c
/*////////////////////////////////////////////////////////////////////////
This program will modify itself at the running time,
These  methods will be very useful in some situations,
Gook Luck!
////////////////////////////////////////////////////////////////////////*/
#include <stdio.h>
#include <windows.h>

void main() {
  TCHAR Info001[MAX_PATH] = "Welcome to Big Apple!";
  TCHAR Info002[MAX_PATH] = "Welcome to Washington!";
  char temp = (char)0x90;
  WORD temp001 = 0x9090;
  DWORD temp002 = 0x90909090;
  PVOID BaseAddressOne = NULL;
  PVOID BaseAddressTwo = NULL;

  __asm {
    mov BaseAddressOne, offset LabelOne
    mov BaseAddressTwo, offset LabelTwo
  }

  MessageBox(NULL, Info001, "Information", MB_OK | MB_ICONINFORMATION);

  // a kind of method to modify itself
  WriteProcessMemory(GetCurrentProcess(), BaseAddressTwo, &temp001, 2, NULL);
  WriteProcessMemory(GetCurrentProcess(), BaseAddressOne, &temp001, 2, NULL);

  /*
  // Another method to modify itself, this method needs to modify the code section's
  // characteristics in PE file.
  __asm {
    mov ebx, BaseAddressOne
    mov ecx, BaseAddressTwo
    mov dx, temp001
    mov [ebx], dx
    mov [ecx], dx
  }
  */

LabelTwo:
  __asm {
    jmp LabelOne
  }

  __asm {
    nop
    nop
    nop
  }

  MessageBox(NULL, Info002, "Information", MB_OK | MB_ICONINFORMATION);

LabelOne:
  __asm {
    jmp Over
  }

  MessageBox(NULL, Info002, "Information", MB_OK | MB_ICONINFORMATION);

Over:
  return;
}
```

编译这个程序，我们发现`WriteProcessMemory()`成功修改了程序自身代码，程序运行正常。

然后我们屏蔽程序中的`WriteProcessMemory()`调用，用`/* */`之中的代码完成自我修改，运行后会发现系统抛出异常`Access Violation`。这是因为`PE`中代码节的属性默认为`0x60000020`，`20`表示代码，`20000000`表示可执行，`40000000`表示可读，如果我们在此基础上加上`0x80000000`（可写），操作系统的`loader`在装载可执行文件时，便会将存放代码节数据的内存标记为可读，可写，可执行。这样就不会有异常了。

读者可使用下面的程序来修改节属性：

```c
/***************************************************************************
// The following code is used to modify characteristics of sections
#include <windows.h>
#include <stdio.h>

BOOL ModifyCharacteristicsOfSections(LPCTSTR FileName) {
  DWORD i = 0;
  HANDLE hDestinationFile = NULL;
  TCHAR DestinationPEFile[MAX_PATH];
  DWORD NumberOfBytesRead = 0;    // Number of bytes read
  DWORD NumberOfBytesWritten = 0; // Number of bytes written
  DWORD ImageNtSignature = 0;     // PE signature
  DWORD OffsetOfNewHeader = 0;
  DWORD NumberOfSections = 0;
  DWORD SizeOfSectionTable = 0;             // size of section table
  HANDLE hGlobalAllocatedMemory = NULL;     // use GlobalAlloc();
  PIMAGE_SECTION_HEADER pImageSectionHeader = NULL; // a pointer to IMAGE_SECTION_TABLE
  IMAGE_DOS_HEADER ImageDosHeader;
  IMAGE_NT_HEADERS ImageNTHeaders;
  IMAGE_FILE_HEADER ImageFileHeader;
  IMAGE_OPTIONAL_HEADER ImageOptionalHeader;
  IMAGE_SECTION_HEADER ImageSectionHeader;
  DWORD dwFileSize = 0;

  RtlZeroMemory(&ImageDosHeader, sizeof(IMAGE_DOS_HEADER));
  RtlZeroMemory(&ImageNTHeaders, sizeof(IMAGE_NT_HEADERS));
  RtlZeroMemory(&ImageFileHeader, sizeof(IMAGE_FILE_HEADER));
  RtlZeroMemory(&ImageOptionalHeader, sizeof(IMAGE_OPTIONAL_HEADER));
  RtlZeroMemory(&ImageSectionHeader, sizeof(IMAGE_SECTION_HEADER));

  strcpy(DestinationPEFile, FileName);

  hDestinationFile = CreateFile(DestinationPEFile,
    FILE_WRITE_DATA | FILE_READ_DATA,
    FILE_SHARE_WRITE, NULL, OPEN_ALWAYS, FILE_ATTRIBUTE_ARCHIVE, NULL);

  if (hDestinationFile == INVALID_HANDLE_VALUE) {
    // printf("\nCreateFile() fails! Can't open file. Please try again!\n");
    // CloseHandle(hDestinationFile);
    return TRUE;
  } else {
    dwFileSize = GetFileSize(hDestinationFile, NULL);
  }

  SetFilePointer(hDestinationFile, 0, NULL, FILE_BEGIN); // Revert the file pointer, this is very important.
  ReadFile(hDestinationFile, &ImageDosHeader,
    sizeof(IMAGE_DOS_HEADER), &NumberOfBytesRead, NULL);

  if (NumberOfBytesRead != sizeof(IMAGE_DOS_HEADER)) {
    // printf("\nReadFile() fails! Can't get IMAGE_DOS_HEADER.\n");
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  OffsetOfNewHeader = ImageDosHeader.e_lfanew; // File address of new exe header
  SetFilePointer(hDestinationFile, (LONG)OffsetOfNewHeader, NULL, FILE_BEGIN);
  ReadFile(hDestinationFile, &ImageNTHeaders,
    sizeof(IMAGE_NT_HEADERS), &NumberOfBytesRead, NULL); // Retrieve IMAGE_NT_HEADERS

  if (NumberOfBytesRead != sizeof(IMAGE_NT_HEADERS)) {
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  if (ImageNTHeaders.Signature != 0x00004550) {
    // printf("Error.\nPE signature is invalid!\n");
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  SetFilePointer(hDestinationFile, OffsetOfNewHeader + 4, NULL, FILE_BEGIN); // Set the file pointer to point to IMAGE_FILE_HEADER
  ReadFile(hDestinationFile, &ImageFileHeader,
    sizeof(IMAGE_FILE_HEADER), &NumberOfBytesRead, NULL); // Retrieve IMAGE_FILE_HEADER

  if (NumberOfBytesRead != sizeof(IMAGE_FILE_HEADER)) {
    // printf("\nReadFile() fails! Can't get IMAGE_FILE_HEADER.\n");
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  if (ImageFileHeader.NumberOfSections < 1) {
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  if (dwFileSize < (sizeof(IMAGE_DOS_HEADER) + sizeof(IMAGE_NT_HEADERS) + sizeof(IMAGE_SECTION_HEADER) * ImageFileHeader.NumberOfSections)) {
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  ReadFile(hDestinationFile, &ImageOptionalHeader,
    sizeof(IMAGE_OPTIONAL_HEADER), &NumberOfBytesRead, NULL); // Retrieve IMAGE_OPTIONAL_HEADER

  if (NumberOfBytesRead != sizeof(IMAGE_OPTIONAL_HEADER)) {
    // printf("\nReadFile() fails! Can't get IMAGE_OPTIONAL_HEADER.\n");
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  if (ImageOptionalHeader.SectionAlignment < ImageOptionalHeader.FileAlignment) {
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  NumberOfSections = ImageFileHeader.NumberOfSections; // Number of sections
  SizeOfSectionTable = sizeof(IMAGE_SECTION_HEADER) * NumberOfSections; // Get the size of Section Table
  hGlobalAllocatedMemory = GlobalAlloc(GPTR, SizeOfSectionTable);      // Allocate memory and initialize with zero

  if (hGlobalAllocatedMemory == NULL) {
    // printf("\nGlobalAlloc() failed! Please try again.\n"); // if failed, return
    CloseHandle(hDestinationFile);
    return FALSE;
  }

  pImageSectionHeader = (PIMAGE_SECTION_HEADER)hGlobalAllocatedMemory; // Convert a handle to a pointer to IMAGE_SECTION_HEADER
  for (i = 0; i < NumberOfSections; i++) // Retrieve the Section Table
  {
    ReadFile(hDestinationFile, pImageSectionHeader + i,
      sizeof(IMAGE_SECTION_HEADER), &NumberOfBytesRead, NULL);

    if (NumberOfBytesRead != sizeof(IMAGE_SECTION_HEADER)) {
      // printf("Error. Can't get IMAGE_SECTION_HEADER.\n");
      CloseHandle(hDestinationFile);
      return FALSE;
    }
  }

  for (i = 0; i < NumberOfSections; i++)
  {
    DWORD dwTempCharacteristics = 0;
    if (((*(pImageSectionHeader + i)).PointerToRawData + (*(pImageSectionHeader + i)).SizeOfRawData > dwFileSize)) {
      CloseHandle(hDestinationFile);
      return FALSE;
    }
    if (((*(pImageSectionHeader + i)).PointerToRawData % ImageOptionalHeader.FileAlignment != 0)) {
      CloseHandle(hDestinationFile);
      return FALSE;
    }
```



```c
printf("\nThe name of the section%d: ",i);  
printf("%s\n",(*(pImageSectionHeader+i)).Name);  
printf("Characteristics: %#x\n",(*(pImageSectionHeader+i)).Characteristics);  
printf("\nPlease input the new characteristics of the section.\n");  
printf("If you enter 0,the characteristics of the section will not be modified.\n");  
scanf("%x",&dwTempCharacteristics);  
if(dwTempCharacteristics!=0)  
  (*(pImageSectionHeader+i)).Characteristics=dwTempCharacteristics;  
  printf("------------------------------------------------------");  
}  
SetFilePointer(hDestinationFile,-((long)SizeOfSectionTable),NULL,FILE_CURRENT); //Set the file poiner  
WriteFile(hDestinationFile,pImageSectionHeader,SizeOfSectionTable,&NumberOfBytesWritten,NULL);  
if(NumberOfBytesWritten==SizeOfSectionTable)  
{  
  printf("\nComplete successfully!\n");  
}  
else  
{  
   printf("\nWriteFile() failed!\n");  
}  
GlobalFree(hGlobalAllocatedMemory); //Free memory  
CloseHandle(hDestinationFile);  
return TRUE;    
}  

void main(int argc,char *argv[])  
{  
  if(argc!=2)  
  {  
    printf("Error\nUsage:ModifyCharacteristicsOfSections CompleteDestinationFileName\n");  
    return;  
  }  
  if(!ModifyCharacteristicsOfSections(argv[1]))  
  {  
    printf("\nError.This usually means that this file is not a valid PE file or\n");  
    printf("that this PE file has been modified by another program,for example,shell programm.\n");  
  }  
}  

/************************************************************************
**                                                                     **
**  This is just a program example                                     **
**  It is not used in practical projects                              **
**                                                                     **
*************************************************************************/
```

以下是上面程序的输出信息：

```text
The name of the section0: .text  
Characteristics: 0x60000020  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

e0000020  
------------------------------------------------------  
The name of the section1: .rdata  
Characteristics: 0x40000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section2: .data  
Characteristics: 0xc0000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section3: .idata  
Characteristics: 0xc0000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section4: .reloc  
Characteristics: 0x42000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
Complete successfully!  
```

```text
The name of the section0: .text  
Characteristics: 0xe0000020  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section1: .rdata  
Characteristics: 0x40000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section2: .data  
Characteristics: 0xc0000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section3: .idata  
Characteristics: 0xc0000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
The name of the section4: .reloc  
Characteristics: 0x42000040  
Please input the new characteristics of the section.  
If you enter 0,the characteristics of the section will not be modified.  

0  
------------------------------------------------------  
Complete successfully!
```
