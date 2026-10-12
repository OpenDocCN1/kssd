
# 3rd：告别硬编码，使用符号文件自动解析符号

> 原文：[`www.pediy.com/kssd/pediy12/140160.html`](https://www.pediy.com/kssd/pediy12/140160.html)

好吧……我承认这个东西很是有骗子的嫌疑

但是我本身就是说放的初级的文章。好东西留着以后再放，还望大家海涵~

符号文件大家应该很熟咯，我就不说什么废话了

具体看`DbgHelp`库的用法。要想解析符号，只需要做以下几步即可；

1. 首先创建一个目录`symsrv.yes`文件，据说`symsrv.dll`会检查，如果存在才允许下载，道听途说的，没有去逆一下，见谅见谅
2. 调用`SymInitialize`初始化，其中必须要传入有效进程句柄，可选传入符号文件搜索路径……如果没有指定这个路径，稍后可以调用`SymSetSearchPath`来指定
3. 欲解析符号前，使用`SymGetSymbolFile`来取得符号文件，如果搜索路径没有这个文件，那么将从微软服务器下载，然后调用`SymLoadModule64`加载它咯
4. 解析咯

注：很重要的一点，要想从微软符号文件服务器下载东西，第二步符号文件搜索路径时须得指定：

`srv\* xxxx \*http://msdl.microsoft.com/download/symbols`

大家都眼熟，就不多说了

PS:我代码里用了一个硬编码，大家见谅，嘿嘿![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")

代码很短，我直接贴上来了

代码:

```c
#include <stdio.h>
#include <Windows.h>
#include <DbgHelp.h>

#pragma comment(lib , "DbgHelp.lib")
#pragma comment(lib , "ImageHlp.lib")


PLOADED_IMAGE IMAGEAPI ImageLoad(
  __in PSTR DllName,
  __in PSTR DllPath
  );


HANDLE hProcess;
char* url = "http://msdl.microsoft.com/download/symbols";

HANDLE hIn;
HANDLE hOut;

void ChangeOutputTextColor(
  DWORD rgbColor)
{
  SetConsoleTextAttribute ( hOut , FOREGROUND_RED);
}

BOOLEAN InitSymHandler()
{
  HANDLE hfile;
  char Path[MAX_PATH]={0};
  char FileName[MAX_PATH]={0};
  char SymPath[MAX_PATH*2]={0};

  if (!GetCurrentDirectoryA( MAX_PATH ,Path))
  {
    printf ("cannot get current directory \n");
    return FALSE;
  }
  strcpy( FileName , Path);
  strcat(FileName ,"\\symsrv.yes");
  printf ("%s \n",FileName);
  hfile = CreateFileA ( FileName,
            FILE_ALL_ACCESS,
            FILE_SHARE_READ,
            NULL,
            OPEN_ALWAYS,
            FILE_ATTRIBUTE_NORMAL,
            NULL);
  if (hfile == INVALID_HANDLE_VALUE)
  {
    printf ("create or open file error: 0x%X \n",GetLastError());
    return FALSE;
  }
  CloseHandle (hfile);

  hProcess = OpenProcess( PROCESS_ALL_ACCESS,
             FALSE,
             4);
  if (hProcess == INVALID_HANDLE_VALUE)
  {
    printf ("Cannot open system process \n");
    return FALSE;
  }
  SymSetOptions ( SYMOPT_CASE_INSENSITIVE | SYMOPT_DEFERRED_LOADS | SYMOPT_UNDNAME);
  strcat(Path,"\\symbols*");
  strcpy(SymPath,"SRV*");
  strcat(SymPath,Path);
  strcat(SymPath,url);

  printf ("%s \n",SymPath);
  SymInitialize( hProcess,
        SymPath,
        FALSE);
  return TRUE;
}

BOOLEAN LoadSymModule(
  char* ImageName,
  DWORD ModuleBase)
{
  DWORD64 tmp;
  char    SymFileName[MAX_PATH]={0};
  PLOADED_IMAGE pli = ImageLoad ( ImageName , NULL);

  if (pli == NULL)
  {
    printf ("cannot get loaded module of %s \n",ImageName);
    return FALSE;
  }


  if (!SymGetSymbolFile( hProcess,
    NULL,
    pli->ModuleName,
    sfPdb,
    SymFileName,
    MAX_PATH,
    SymFileName,
    MAX_PATH))
  {
    printf ("cannot get symbol file of %s ,error: 0x%x \n",ImageName,GetLastError());
    return FALSE;
  }

  tmp =  SymLoadModule64( hProcess,
            pli->hFile,
            pli->ModuleName,
            NULL,
            (DWORD64)ModuleBase,
            pli->SizeOfImage);


  if (!tmp)
  {
    printf ("cannot load module , error : %X \n",GetLastError());
    return FALSE;
  }

  return TRUE;
}


BOOLEAN EnumSyms(
  char* ImageName,
  DWORD ModuleBase,
  PSYM_ENUMERATESYMBOLS_CALLBACK EnumRoutine,
  PVOID Context)
{
  BOOLEAN bEnum;

  if ( !LoadSymModule( ImageName , ModuleBase) )
  {
    return FALSE;
  }

  bEnum = SymEnumSymbols( hProcess,
            ModuleBase,
            NULL,
            EnumRoutine,
            Context);
  if (!bEnum)
  {
    printf ("cannot enum symbols ,error: %x \n",GetLastError());
  }

  return bEnum;
}

BOOLEAN CALLBACK EnumSymRoutine(
  PSYMBOL_INFO psi,
  ULONG     SymSize,
  PVOID     Context)
{
  printf ("%s : 0x%X \n",psi->Name,psi->Address);

  return TRUE;
}

int main(void)
{

  hIn = GetStdHandle ( STD_INPUT_HANDLE);
  hOut = GetStdHandle (STD_OUTPUT_HANDLE);

  if (!InitSymHandler ())
  {
    goto __exit;
  }
  ;
  //ChangeOutputTextColor (4);

  EnumSyms("ntkrnlpa.exe", 0x804d8000,(PSYM_ENUMERATESYMBOLS_CALLBACK)EnumSymRoutine,NULL);
__exit:
  getchar();
}
```

![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑") 这货只能判断没导出的函数吧？

俺研究了很久了不知道是俺写的有问题还是本来只能解析没导出的函数.

```c
#pragma once

#ifndef PDBDATA_H

#define PDBDATA_H

#include "KillProtectDll.h"

#include "..\\KillProtect\\struct.h"

#include <fstream>

;using namespace std;

class PDBData

{

private:

  wchar_t ch_path[MAX_PATH];

protected:

public:

  PDBData();

  ~PDBData();

  wchar_t* GetPDBPath();

  BOOL Symbol_View(char* ch_Module,DWORD64 BaseAddr,DWORD FileSize);

  BOOL WcharToAchar(wchar_t* ch_temp,char* ch_temp1);

  ULONG EnumModule(wchar_t* ch_ModuleName,BOOL b_open);

};

#endif

//------------------------------------------

#include "PDBData.h"

PDBData::PDBData()

{

  ZeroMemory(ch_path,MAX_PATH);

}

PDBData::~PDBData()

{

}

wchar_t* PDBData::GetPDBPath()

{

  ::GetModuleFileName(::GetModuleHandle(DLL_Name),ch_path,MAX_PATH);

  ::PathRemoveFileSpec(ch_path);

  ::PathAppend(ch_path,PDB_Path);

  return ch_path;

}

char* ch_Name[]={"PsTerminateProcess","PsGetNextProcess","PsGetNextProcessThread"};

_FuncCou FunCont[100];

int in_Count=0;

BOOL CALLBACK EnumSymCallBack( PSYMBOL_INFO pSymInfo,ULONG SymbolSize,PVOID UserContext )

{

  for(unsigned int i=0;i<3;i++)

  {

    if(strcmp(pSymInfo->Name,ch_Name[i])==0)

    {

      ZeroMemory(&FunCont[in_Count],sizeof(FunCont[in_Count]));

      ::memcpy(FunCont[in_Count].ch_FunName,pSymInfo->Name,strlen(pSymInfo->Name)+1);

      FunCont[in_Count].dw_Addr=pSymInfo->Address;

      in_Count++;

    }

  }

  return TRUE;

}

BOOL PDBData::Symbol_View(char* ch_Module,DWORD64 BaseAddr,DWORD FileSize)

{

  PIMAGEHLP_SYMBOL pSymbol=NULL;

  DWORD Options=::SymGetOptions();

  Options=Options|SYMOPT_DEBUG;

  ::SymSetOptions(Options);

  HANDLE hProcess=::GetCurrentProcess();

  BOOL bRet=::SymInitialize(hProcess,0,FALSE);

  if(!bRet)

  {

    return FALSE;

  }

  char SymbolPath[MAX_PATH];

  ::GetCurrentDirectoryA(MAX_PATH,SymbolPath);

  ::PathAppendA(SymbolPath,PDB_PathA);

  ::SymSetSearchPath(hProcess,SymbolPath);

  char FileName[MAX_PATH] ;

  ::GetSystemDirectoryA(FileName,MAX_PATH);

  ::PathAppendA(FileName,ch_Module);

  DWORD64 BaseOfDll=::SymLoadModule64(hProcess,NULL,FileName,NULL,BaseAddr,FileSize);

  if(BaseOfDll == 0)

  {

    return FALSE;

  }

  ::SymEnumSymbols(hProcess,BaseOfDll,0,EnumSymCallBack,0);

  ::SymUnloadModule64(hProcess,BaseOfDll);

  ::SymCleanup(hProcess);

  return TRUE;

}

BOOL PDBData::WcharToAchar( wchar_t* ch_temp,char* ch_temp1 )

{

  int in_len=::WideCharToMultiByte(CP_ACP,0,ch_temp,wcslen(ch_temp)*2+1,NULL,NULL,NULL,NULL);

  ::WideCharToMultiByte(CP_ACP,0,ch_temp,wcslen(ch_temp)*2+1,ch_temp1,in_len,NULL,NULL);

  return TRUE;

}

ULONG PDBData::EnumModule( wchar_t* ch_ModuleName,BOOL b_open=FALSE)

{

  int status;

  LPVOID lpBuffer;

  int len=0;

  PSYSTEM_MODULE_INFORMATION ModuleTion;

  NtQuerySystemInformation* _NtQuerySystemInformation;

  _NtQuerySystemInformation=(NtQuerySystemInformation*)::GetProcAddress(::GetModuleHandle(L"ntdll.dll"),"NtQuerySystemInformation");

  status=_NtQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,NULL,0,(PULONG)&len);

  lpBuffer=new BYTE[len];

  if(lpBuffer==NULL)

  {

    return 0;

  }

  _NtQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,lpBuffer,len,NULL);

  ModuleTion=(PSYSTEM_MODULE_INFORMATION)lpBuffer;

  ULONG data=0;

  char ch_modulenames[MAX_PATH];

  ZeroMemory(ch_modulenames,MAX_PATH);
```



```
WcharToAchar(ch_ModuleName,ch_modulenames);  

for(unsigned int i=0;i<ModuleTion->Count;i++)  
{  
  if(strstr(ModuleTion->Module[i].ImageName,ch_modulenames))  
  {  
    if(b_open==FALSE)  
    {  
      data=(ULONG)ModuleTion->Module[0].Base;  
    }  
    else  
    {  
      data=(ULONG)ModuleTion->Module[0].Size;  
    }  
    break;  
  }  
}  
delete [] lpBuffer;  
return data;  
}
```
