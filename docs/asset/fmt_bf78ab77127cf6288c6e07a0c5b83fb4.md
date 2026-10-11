
# PE 文件学习之 编写 PE 文件分析工具

> 原文：[`www.pediy.com/kssd/pediy10/78872.html`](https://www.pediy.com/kssd/pediy10/78872.html)

## 题记

大家好，本来是打算写`C++`逆向的内容呢；但由于时间关系，而写表达式与运算符其实更多的是写编码解码机制和汇编语言；所以不知道怎么来定位；但后期我会慢慢的整理出来，说实话，加密解密三这书写的很棒，以前在看加密解密二的时候就对`PE`文件格式研究了好一阵；直到如今慢慢的才基本理解；从编程的角度来看，`PE`文件格式只不是一堆互相嵌套的数据结构；然而从系统的角度来看，这部分就比较伤神了，比如，逻辑地址到物理地址转换，文件加载的过程，动态函数的链接，`RAV`等内容的理解，也是对刚接触的新手来说，是有些障碍；

但如果你相信你自己能，你也许就真的能；如果一开始你就觉得你不能做，那么你永远都做不了；小菜我在写这个程序的时候，就遇到不少问题，但由于没有放弃，现在就能发文章了；

## 关于这篇文章

有关`PE`文件格式的详解，请大家参照加密解密三，书上很详细，由于写`PE`分析器，主要的在于文件打开和文件映射这一部分代码的实现，而`PE`文件的那一部分，加密解密三上有很详细的代码；

所以希望大家在看玩此代码后，能继续把，输入表，输出表那一部分写出来；因为那才是`PE`文件的核心和重点；好了，废话不说；

## 关于这个程序

由于是第一次写这程序，鉴于本人的无工作经验，仅为学习练手用，有不是之处，还希望大家指出，声明，`PE`文件的相关代码，参考了加密解密三书上的；所以，前面几步实现了的，后面应该就不难了；

你可以在此程序的基础上，再做扩充，比如：`RVA`转换，输入，输出表的显示，等等；相信，不用多久，打造属于你自己的山寨`PE`查看器就不成问题了；![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")（山寨这一词，首次在看雪见了，好喜欢哦，想起了小说花木兰，嘿嘿）；

## 代码

```
#include<windows.h>

#include "resource.h"

BOOL CALLBACK DlgProc(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam);

//Pe 文件处理函数声明
BOOL IsPeFile(LPVOID ImageBase);
PIMAGE_NT_HEADERS  GetNtHeader(LPVOID  ImageBase);
PIMAGE_FILE_HEADER  WINAPI  GetFileHeader(LPVOID  Imagebase);
PIMAGE_OPTIONAL_HEADER  GetOptionalHeader(LPVOID  ImageBase);

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowcmd)
{
    DialogBox(hInstance, MAKEINTRESOURCE(IDD_DIALOG), NULL, DlgProc);

    return 0;
}

BOOL CALLBACK DlgProc(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{

  OPENFILENAME  FileName = {0,0,0}, *lpFileName = &FileName;
  HANDLE       hFile, hFileMap; 
  TCHAR       szPe[]="\PE File(*.exe)\0*.exe;*.dll;*.scr;*.fon;*.drv;\0\*.All File(*.*) \0*.*\0\0";
  TCHAR       szFileName[256]={""};

  LPVOID       lpMemory;

  TCHAR       Buff[16];
  PIMAGE_FILE_HEADER  pFileHeader = NULL;
  PIMAGE_OPTIONAL_HEADER  pOptionHeader = NULL;

  switch(message)
  {
    case  WM_INITDIALOG:
        break;
    case  WM_CLOSE:

        EndDialog(hDlg, NULL);
        break;

    case  WM_COMMAND:
          switch(LOWORD(wParam))
       {
          case  IDM_OPEN:
        FileName.hInstance = (HINSTANCE)hDlg;
        FileName.hwndOwner = hDlg;
        FileName.lStructSize = sizeof(OPENFILENAME);
        FileName.lpstrFilter = szPe;
        FileName.lpstrFile = szFileName;
        FileName.Flags = OFN_FILEMUSTEXIST||OFN_PATHMUSTEXIST;
        FileName.nMaxFile = sizeof(szFileName);


        if(!GetOpenFileName(lpFileName))
        {
          MessageBox(hDlg, "GetOpenFileName 调用失败", "ERROR", NULL);
          break;
        }

        SetDlgItemText(hDlg, IDC_FILENAME, szFileName);

        hFile = CreateFile(FileName.lpstrFile,            // open  pe file 
                    GENERIC_READ,           // open for reading 
                    FILE_SHARE_READ|| FILE_SHARE_WRITE,           // share for reading 
                    NULL,                   // no security 
                    OPEN_EXISTING,           // existing file only 
                    FILE_ATTRIBUTE_NORMAL,    // normal file 
                    NULL);               // no attr. template 

        if (hFile == INVALID_HANDLE_VALUE) 
        { 
           MessageBox(hDlg, "Could not open file.", "ERROR", MB_ICONERROR); 
           break;// process error 
        } 

        if(GetFileSize(hFile, NULL)!=0)
        {
          hFileMap = CreateFileMapping(hFile, NULL, PAGE_READONLY, 0, 0, NULL);
          if(hFileMap!=0)
          {
          lpMemory = MapViewOfFile(hFileMap, FILE_MAP_READ, NULL, NULL, NULL);
          }
        }
        
        if(IsPeFile(lpMemory))
        {

          pFileHeader = GetFileHeader(lpMemory);
          pOptionHeader = GetOptionalHeader(lpMemory);
          if(!(pFileHeader&&pOptionHeader))
          {
            MessageBox(hDlg, "获取文件头指针失败", "PEINFO", MB_ICONERROR);
            break;
          }
          else
          {
            wsprintf(Buff, "%04lX", pFileHeader->Machine);
            SetDlgItemText(hDlg, IDC_MACHINE, Buff);

            wsprintf(Buff, "%04lX", pFileHeader->NumberOfSections);
            SetDlgItemText(hDlg, IDC_NUMSECTION, Buff);

            wsprintf(Buff, "%04lX", pOptionHeader->Magic);
            SetDlgItemText(hDlg, IDC_MAGIC, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->AddressOfEntryPoint);
            SetDlgItemText(hDlg, IDC_ENTERPOINT, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[0].VirtualAddress);
            SetDlgItemText(hDlg, IDC_EDIT_RVA_EXPORT, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[0].Size);
            SetDlgItemText(hDlg, IDC_EDIT_SIZE_EXPORT, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[1].VirtualAddress);
            SetDlgItemText(hDlg, IDC_EDIT_RVA_IMPORT, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[1].Size);
            SetDlgItemText(hDlg, IDC_EDIT_SIZE_IMPORT, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[2].VirtualAddress);
            SetDlgItemText(hDlg, IDC_EDIT_RVA_RES, Buff);

            wsprintf(Buff, "%08lX", pOptionHeader->DataDirectory[2].Size);
            SetDlgItemText(hDlg, IDC_EDIT_SIZE_RES, Buff);

          }

        }
        else
        {
          MessageBox(hDlg, "你选择的不是 PE 文件", "error", MB_ICONERROR);
          UnmapViewOfFile(lpMemory);
          CloseHandle(hFileMap);
          CloseHandle(hFile);
        }
        UnmapViewOfFile(lpMemory);
        CloseHandle(hFileMap);
        CloseHandle(hFile);

        break;

      }

  }
    return FALSE;
}

BOOL  IsPeFile(LPVOID  ImageBase)   //判断是否是 PE 文件结构
{
  PIMAGE_DOS_HEADER  pDosHeader = NULL;
  PIMAGE_NT_HEADERS  pNtHeader  = NULL;

  if(!ImageBase)
    return FALSE;
  pDosHeader = (PIMAGE_DOS_HEADER) ImageBase;
  if(pDosHeader->e_magic != IMAGE_DOS_SIGNATURE)
    return FALSE;
  pNtHeader = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader+pDosHeader->e_lfanew);
  if(pNtHeader->Signature != IMAGE_NT_SIGNATURE )
    return  FALSE;
  return    TRUE;
}

//FileHeader 内容的读取

PIMAGE_NT_HEADERS  GetNtHeader(LPVOID  ImageBase) //获取 NT 结构指针
{
  PIMAGE_DOS_HEADER  pDosHeader = NULL;
  PIMAGE_NT_HEADERS  pNtHeader  = NULL;

  if(!IsPeFile(ImageBase))
    return  NULL;
  pDosHeader = (PIMAGE_DOS_HEADER)ImageBase;
  pNtHeader = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader+pDosHeader->e_lfanew);
  return    pNtHeader;
}

PIMAGE_FILE_HEADER  WINAPI  GetFileHeader(LPVOID  Imagebase)
{
  PIMAGE_FILE_HEADER  pFileHeader;
  PIMAGE_NT_HEADERS  pNtHeader = NULL;
  pNtHeader = GetNtHeader(Imagebase);
  if(!pNtHeader)
    return  NULL;
  pFileHeader = & pNtHeader->FileHeader; 
  return  pFileHeader;
}

PIMAGE_OPTIONAL_HEADER  GetOptionalHeader(LPVOID  ImageBase)
{
  PIMAGE_OPTIONAL_HEADER  pOptionHeader = NULL;
  PIMAGE_NT_HEADERS  pNtHeader = NULL;
  pNtHeader = GetNtHeader(ImageBase);
  if(!pNtHeader)
    return  NULL;
  pOptionHeader = & pNtHeader->OptionalHeader;
  return  pOptionHeader;
}

```

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `PEINFO_CODE.rar`  
|
