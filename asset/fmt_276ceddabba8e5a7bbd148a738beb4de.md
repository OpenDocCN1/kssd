
```cpp
// Fuck_Win32Agent.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include  <windows.h>
#include <ShellAPI.h>
#pragma  comment(lib, "Shell32.lib ")
#include  <Shlwapi.h>
#pragma  comment(lib, "Shlwapi.lib ")

#define  _DBGOUT
#include "E:\\work\\封装头文件\\dbgout.h"

#include "E:\\work\\封装头文件\\mkValue.h"


unsigned long GetRAbyRVA(HANDLE hMapView,unsigned long RVA)
  {
  IMAGE_DOS_HEADER* doshead = (IMAGE_DOS_HEADER*)hMapView;
  IMAGE_NT_HEADERS* PEhead = (IMAGE_NT_HEADERS*)((unsigned long)doshead + (unsigned long)doshead->e_lfanew);
  IMAGE_SECTION_HEADER* ISH = (IMAGE_SECTION_HEADER*)((unsigned long)PEhead + sizeof(IMAGE_NT_HEADERS));
  IMAGE_EXPORT_DIRECTORY*  lpExpF = 0;
  for(int i=0;i<PEhead->FileHeader.NumberOfSections;++i)
    {
    if(RVA>=ISH->VirtualAddress && RVA<(ISH->VirtualAddress+ISH->Misc.VirtualSize))
      {
      return( (unsigned long)hMapView + ISH->PointerToRawData + RVA - ISH->VirtualAddress );
      }
    ++ISH;
    }
  return 0;
  }

bool  dealPE(const TCHAR* ps)
  {
  HANDLE hMapView = 0;
  bool rets = false;
try
  {
  HANDLE hFile = CreateFile(ps,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);
  if(hFile == INVALID_HANDLE_VALUE)  return false;
  HANDLE hFileMapping = CreateFileMapping(hFile,0,PAGE_READWRITE,0,0,0);
  CloseHandle(hFile);
  hMapView = MapViewOfFile(hFileMapping,FILE_MAP_READ|FILE_MAP_WRITE,0,0,0);
  CloseHandle(hFileMapping);
  
  if(!hMapView)        throw runtime_error("MapViewOfFile 失败！");
  IMAGE_DOS_HEADER* doshead = (IMAGE_DOS_HEADER*)hMapView;
  if(doshead->e_magic != 'ZM')        throw runtime_error("检测 MZ 标志失败！");
  IMAGE_NT_HEADERS* PEhead = (IMAGE_NT_HEADERS*)((unsigned long)doshead + (unsigned long)doshead->e_lfanew);
  if(PEhead->Signature != 'EP')        throw runtime_error("检测 PE 标志失败！");
  
  unsigned long Entry = PEhead->OptionalHeader.AddressOfEntryPoint;
  
  Entry = GetRAbyRVA(hMapView,Entry);
  bool  befuckentry = false;
  bool  befucksection   = false;
  if(mkL(Entry)==0x303D8B64)
    {
    befuckentry = true;
    errs<<"●●●"<<errTchar((TCHAR*)ps)<<"有感染的特征！"<<endd;
    }

  IMAGE_SECTION_HEADER* ISH = (IMAGE_SECTION_HEADER*)((unsigned long)PEhead + sizeof(IMAGE_NT_HEADERS));
  ISH+=(PEhead->FileHeader.NumberOfSections - 1);
  if(mkL(ISH->Name)=='\0ct.')
    {
    befucksection = true;
    errs<<"●●●"<<errTchar((TCHAR*)ps)<<"有感染的节区！"<<endd;
    }
  if(befucksection&&befuckentry)
    {
//     cout<<"※已经确定："<<errTchar((TCHAR*)ps)<<"被 Win32.Agent 感染，是否修复(Y/N)：";
//     char oks;
//     cin>>oks;
//     if((oks!='Y') && (oks!='y'))
//       {
//       errs<<"修复已经被跳过！"<<endd;
//       throw 0;
//       }
    errs<<"正在修复"<<errTchar((TCHAR*)ps)<<"..."<<endd;
    int IP = Entry + 0x64 + 1;
    int offsets = mkL(IP);  //0x118
    IP += sizeUL + offsets + 1;
    offsets = mkL(IP);
    int  realEntry = IP + sizeUL + offsets - (int)hMapView - ISH->PointerToRawData + ISH->VirtualAddress;
    PEhead->OptionalHeader.AddressOfEntryPoint = realEntry;    //修复入口
    size_t realfilelen = ISH->PointerToRawData;
    PEhead->OptionalHeader.SizeOfCode -= ISH->Misc.VirtualSize;    //修复代码块大小
    --(PEhead->FileHeader.NumberOfSections);    //修复区段个数
    PEhead->OptionalHeader.SizeOfImage -= ISH->Misc.VirtualSize;    //修复映像大小
    UnmapViewOfFile(hMapView);
    hMapView = 0;
    HANDLE hFile = CreateFile(ps,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);
    if(hFile == INVALID_HANDLE_VALUE)  return false;
    SetFilePointer(hFile,realfilelen,0,FILE_BEGIN);
    SetEndOfFile(hFile);
    CloseHandle(hFile);
    errs<<"========修复成功！========"<<endd;
    rets = true;
    }
  }
catch(...)
  {
  ;
  }
  if(hMapView)UnmapViewOfFile(hMapView);
  return rets;
  }


enum  fileflag {flag_no,PE_EXE,RAR};

fileflag  iswant(const TCHAR* ps)
  {
  size_t len = _tcslen(ps);
  if(len<=4)return flag_no;
  if(_tcsicmp(TEXT(".exe"),&ps[len-4])==0)
    {
    return  PE_EXE;
    }
  if(_tcsicmp(TEXT(".dll"),&ps[len-4])==0)
    {
    return  PE_EXE;
    }
  if(_tcsicmp(TEXT(".sys"),&ps[len-4])==0)
    {
    return  PE_EXE;
    }
  if(_tcsicmp(TEXT(".rar"),&ps[len-4])==0)
    {
    return  RAR;
    }
  if(_tcsicmp(TEXT(".zip"),&ps[len-4])==0)
    {
    return  RAR;
    }
  return flag_no;
  }

void  domain(const TCHAR* ps);

void  doit(const TCHAR* ps)
  {
  fileflag ff = iswant(ps);
  if(ff==flag_no)return;
  if(ff==PE_EXE)
    {
    dealPE(ps);
    return;
    }
  if(ff==RAR)
    {
    wstring path = ps;
    path += TEXT("$$");
    wstring cmdline = TEXT("D:\\Program Files\\WinRAR\\WinRAR.exe x -O+ \"");
    cmdline += ps;
    cmdline += TEXT("\" *.* \"");
    cmdline  += path;
    cmdline += TEXT("\\\"");
    STARTUPINFO info;
    PROCESS_INFORMATION pi;
    ZeroMemory(&info,sizeof(info));
    if(!CreateProcess(0,(TCHAR*)cmdline.c_str(),0,0,0,0,0,0,&info,&pi))
      {
      errs<<"释放："<<errTchar(ps)<<"失败！"<<endd;
      return;
      }
    WaitForSingleObject(pi.hProcess,INFINITE);
    domain(path.c_str());
    cmdline = TEXT("D:\\Program Files\\WinRAR\\WinRAR.exe m -r -ep1 -y \"");
    cmdline  += ps;
    cmdline += TEXT("\" ");
    cmdline  += path;
    cmdline += TEXT("\\*\\\"");
    ZeroMemory(&info,sizeof(info));
    if(!CreateProcess(0,(TCHAR*)cmdline.c_str(),0,0,0,0,0,0,&info,&pi))
      {
      errs<<"更新："<<errTchar(ps)<<"失败！"<<endd;
      return;
      }
    WaitForSingleObject(pi.hProcess,INFINITE);
    errs<<"〓〓〓〓已经处理完："<<errTchar(ps)<<endd;
    }
  }


void  domain(const TCHAR* ps)
  {
  if(*ps==TEXT('.'))return;
  //PathIsDirectory
  if(PathIsDirectory(ps))
    {
    //errs<<"★★★"<<errTchar((TCHAR*)ps)<<":是一个目录，向下枚举..."<<endd;
     wstring oo = ps;
     if(*(oo.rbegin())==TEXT('\\'))
       {
       oo.erase(oo.end()-1);
       }
    oo+=TEXT("\\*.*");
    WIN32_FIND_DATA  wfd;
    HANDLE  hFind = FindFirstFile(oo.c_str(),&wfd);
    if(INVALID_HANDLE_VALUE == hFind)return;
    for(;FindNextFile(hFind,&wfd);)
      {
      if(*wfd.cFileName==TEXT('.'))continue;
      wstring fn;
      fn = ps;
      if(*(fn.rbegin())==TEXT('\\'))
        {
        fn.erase(fn.end()-1);
        }
      fn+=TEXT('\\');
      fn+=wfd.cFileName;
      domain(fn.c_str());
      }
    CloseHandle(hFind);
    return;
    }
  doit(ps);
  }


//函数：显示开始时间
void  showSYSTEMTIME(void)
  {
  SYSTEMTIME systime;
  GetLocalTime(&systime);
  errs<<"**********     ";
  errs<<setw(4)<<systime.wYear<<'-';
  errs<<setw(2)<<systime.wMonth<<'-';
  errs<<setw(2)<<systime.wDay<<"        ";
  errs<<setw(2)<<systime.wHour<<':';
  errs<<setw(2)<<systime.wMinute<<':';
  errs<<setw(2)<<systime.wSecond<<"       **********"<<endd;
  }

void  usage()
  {
  cout<<"\n Fuck_Win32Agent [路径] [PE 文件] ..."<<endl;
  cout<<"\n 如果参数为“路径”时，枚举其下所有文件。"<<endl;
  }


int _tmain(int argc, _TCHAR* argv[])
{
  unsigned long times = GetTickCount();
  //更改全局输出格式
  errs<<setfill('0');
  errs<<"***********************************************************************"<<endd;
  showSYSTEMTIME();    //输出时间
  if(argc<=1)
    {
    usage();
    return 0;
    }
  for(int i = 1;i < argc;++i)
    {
    domain(argv[i]);
    }
  unsigned long usetime = GetTickCount() - times;
  errs<<"**********   共计用时："<<setw(3)<<usetime/1000;
  errs<<'.'<<setw(3)<<usetime%1000;
  errs<<"秒           **********"<<endd;
  errs<<"***********************************************************************"<<endd;
  Beep(2300,50);
  return 0;
}
```
