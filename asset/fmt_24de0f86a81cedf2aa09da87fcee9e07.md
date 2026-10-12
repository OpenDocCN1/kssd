
```c
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <commdlg.h>
#include "1.h"
PIMAGE_FILE_HEADER   pFileHeadera = NULL;
PIMAGE_OPTIONAL_HEADER   pOptionHeader = NULL;
BOOL   IsPeFile(LPVOID   ImageBase)    //判断是否是 PE 文件结构
{
  PIMAGE_DOS_HEADER   pDosHeader = NULL;  //定义 IMAGE_DOS_HEADER 结构变量
  PIMAGE_NT_HEADERS   pNtHeader  = NULL;  //定义 IMAGE_NT_HEADERS 结构变量
  if(!ImageBase)    //判断映像基址，映像基址由下面的 MapViewOfFile 函数传入进来
    return FALSE;
  pDosHeader = (PIMAGE_DOS_HEADER) ImageBase; //转换 ImageBase 为 PIMAGE_DOS_HEADER 结构变量类型
  if(pDosHeader->e_magic != IMAGE_DOS_SIGNATURE)  //指向 IMAGE_DOS_HEADER 结构变量中的 e_magic 成员，e_magic 是 0x5A4D,名为 IMAGE_DOS_SIGNATURE
    return FALSE;
  pNtHeader = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader+pDosHeader->e_lfanew); //从 IMAGE_DOS_HEADER 结构变量中找到 e_lfanew 成员里面的起始偏移量和加上映像基址得到 PE 文件头的指针
  if(pNtHeader->Signature != IMAGE_NT_SIGNATURE ) //指向 IMAGE_NT_HEADERS 结构变量中的 Signature 成员,Signature 是 0x4550,名为 IMAGE_NT_SIGNATURE
    return  FALSE;
  return   TRUE;
}
PIMAGE_NT_HEADERS   GetNtHeader(LPVOID   ImageBase) //PE 头由 3 部分组成，IMAGE_NT_HEADERS，IMAGE_FILE_HEADER 和 IMAGE_OPTIONAL_HEADER 组成。
{
  PIMAGE_DOS_HEADER   pDosHeader = NULL;
  PIMAGE_NT_HEADERS   pNtHeader  = NULL;

  if(!IsPeFile(ImageBase))
    return  NULL;
  pDosHeader = (PIMAGE_DOS_HEADER)ImageBase;                             //上面有注释，自己观看
  pNtHeader = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader+pDosHeader->e_lfanew);     //上面有注释，自己观看
  return   pNtHeader;
}

PIMAGE_FILE_HEADER   WINAPI   GetFileHeader(LPVOID   Imagebase)
{
  PIMAGE_FILE_HEADER   pFileHeader;   //PIMAGE_FILE_HEADER 类型同下面注释的 IMAGE_FILE_HEADER 结构类型
  PIMAGE_NT_HEADERS   pNtHeader = NULL;
  pNtHeader = GetNtHeader(Imagebase);
  if(!pNtHeader)
    return  NULL;
  pFileHeader = & pNtHeader->FileHeader; 
  return  pFileHeader;
}
//参见这个结构
// typedef struct _IMAGE_NT_HEADERS {
    // DWORD Signature;
    // IMAGE_FILE_HEADER FileHeader;
    // IMAGE_OPTIONAL_HEADER32 OptionalHeader;
// } IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;这两个 FileHeader 和 OptionalHeader 成员都被定义了类型，所以返回的类型为就是这个定义的类型
PIMAGE_OPTIONAL_HEADER   GetOptionalHeader(LPVOID   ImageBase)
{
  PIMAGE_OPTIONAL_HEADER   pOptionHeader = NULL;  // PIMAGE_OPTIONAL_HEADER32 类型同上面注释中的类型，这里名字不一样是因为有一个 typedef，typedef PIMAGE_OPTIONAL_HEADER32   PIMAGE_OPTIONAL_HEADER;
  PIMAGE_NT_HEADERS   pNtHeader = NULL;
  pNtHeader = GetNtHeader(ImageBase);
  if(!pNtHeader)
    return  NULL;
  pOptionHeader = & pNtHeader->OptionalHeader;
  return  pOptionHeader;
}
void OpenFileA(HWND hwnd)
{   HANDLE hFile;
  HANDLE hMapping;
  LPVOID ImageBase;
  char         szFileName[MAX_PATH] = {0};
    TCHAR       Buff[100];
  OPENFILENAME ofn;
    ZeroMemory(&ofn, sizeof(OPENFILENAME));                        
  ofn.lStructSize  = sizeof(OPENFILENAME);
  ofn.lpstrFile   = szFileName;
  ofn.nMaxFile     = MAX_PATH;
  ofn.lpstrFilter  = "Exe Files(\*.exe)\\0\*.exe\\0All Files(\*.\*)\\0\*.\*\\0\\0";
  ofn.nFilterIndex = 1;
  if( !GetOpenFileName(&ofn) )
  { 
    Sleep(1000);
  }
  wsprintf(Buff,"%s",ofn.lpstrFile);
  SetDlgItemText(hwnd,IDE_Edit,Buff);
    hFile=CreateFile(szFileName,GENERIC_READ,FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);//创建文件
  if (!hFile){;}
    hMapping=CreateFileMapping(hFile,NULL,PAGE_READONLY,0,0,NULL); //创建文件映像
  if(!hMapping)
  {               
    CloseHandle(hFile);
  }
  ImageBase=MapViewOfFile(hMapping,FILE_MAP_READ,0,0,0); //映射文件基址
    if(!ImageBase)
  {               
    CloseHandle(hMapping);
    CloseHandle(hFile);
  }
  if(!IsPeFile(ImageBase))
  {
    MessageBox(hwnd,"不是 PE 文件!","提示信息",MB_ICONERROR);
    return ;
  }
  else
  pFileHeadera = GetFileHeader(ImageBase);
  pOptionHeader = GetOptionalHeader(ImageBase);
  if(!(pFileHeadera&&pOptionHeader))
  {
      MessageBox(hwnd,"获取文件头失败","PEINFO",MB_ICONERROR);
    return ;
  }
  wsprintf(Buff,"%08lX",pFileHeadera->Machine);  //IMAGE_FILE_HEADER 结构内容读取，下面一样，具体观看 http://bbs.pediy.com/showthread.php?t=100823 
  SetDlgItemText(hwnd,IDE_Machine,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->NumberOfSections);
  SetDlgItemText(hwnd,IDE_NumberOfSections,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->TimeDateStamp);
  SetDlgItemText(hwnd,IDE_TimeDateStamp,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->PointerToSymbolTable);
  SetDlgItemText(hwnd,IDE_PointerToSymbolTable,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->NumberOfSymbols);
  SetDlgItemText(hwnd,IDE_NumberOfSymbols,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->SizeOfOptionalHeader);
  SetDlgItemText(hwnd,IDE_SizeOfOptionalHeader,Buff);
  wsprintf(Buff,"%08lX",pFileHeadera->Characteristics);
  SetDlgItemText(hwnd,IDE_Characteristics,Buff);
}
BOOL CALLBACK DialogProc(HWND hwndDlg, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch(uMsg)
    {
        case WM_INITDIALOG:
            return TRUE;

        case WM_CLOSE:
            DestroyWindow(hwndDlg);
            return TRUE;
        case WM_COMMAND:
            switch(LOWORD(wParam))
            {
                case 4003:
                    OpenFileA(hwndDlg);
                    return FALSE;
        case 4005:
          return FALSE;


            }
    }

    return FALSE;
}
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd)
{
  WNDCLASSEX wcx;
    wcx.cbSize = sizeof(wcx);
  wcx.hInstance = hInstance;
    wcx.hIcon = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_ICON));
  wcx.hIconSm = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_ICON));
    // The user interface is a modal dialog box
    return DialogBox(hInstance, MAKEINTRESOURCE(1001), NULL, DialogProc);
}
```

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `PE 之二.zip`  
