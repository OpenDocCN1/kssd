
# 利用 EXE 中的数据与代码

> 原文：[`www.pediy.com/kssd/pediy11/113450.html`](https://www.pediy.com/kssd/pediy11/113450.html)

某日，逆向一 `EXE`（这里命名它为 `HH`），发现内部使用了 `Rijndael-256`，以及某个不知名的算法。

从网上找到 N 个相关 `AES` 算法的实现，结果密码表是一样了，但运行结果却怎么都不一样。![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

由于实在对 `AES` 头晕，再加上还有某个不知名的算法（内部流程复杂），所以考虑需要利用这个 `EXE` 的加解密实现。

可是 `EXE` 不是 `DLL`，没有重定位表。不能像 `DLL` 一样 `Load` 一下，直接 `CALL` 就能用。

怎么办？把 `EXE` 当 `DLL` 使？修正重定位信息？网上找了找，基本是这个说法，但说法的后面，也明确指出：依然不足，修正重定位信息是件苦力活，还容易出错。![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")

而且，N 个地方需要重定位，这一路做下来，不吐血也脑溢血了。![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

考虑考虑怎么取巧吧。

思路：

首先，自己写的利用程序（这里命名它为 `CC`）是可以设置有重定位信息的，所以可以让出 `EXE` 的默认加载位置。

把 `HH` 放在 `400000` 位置，嘿嘿，重定位不就省了？

说得容易，实现一下看看？指定 `CC` 加载到别处，设置重定位，`MapViewOfFileEx` 把 `HH` 加载到 `0x400000`。

报错？不行？不行！

什么错？目标位置已被使用，无法加载？！

`OD` 看看，果然，让出来的茅坑让系统给用了。![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")

再想：

由于茅坑让出来了，系统就认为是无主的，爱怎么用，用多少，咱管不到。

就算这个茅坑系统没有使用，万一 `HH` 屁股比较大，需要连坐 N 个坑，那就保不齐系统把后面坑占走，害 `HH` 屁股装不下了。。。

如何通知系统留着坑哩？

试着手工打造一个占坑 `PE`。试来试去，不尽人意。

又想：

最好的办法是让 `HH` 自己占。

可是 `HH` 自己占坑就得运行，我们只想使用数据与代码，不想它运行起来。

于是需要改造 `HH` 入口。

由于 `HH` 入口承接着主线程，不能让它结束。正好，把主线程转移给 `CC`。

用 `LoadLibrary` 加载 `CC`。

`LoadLibrary` 加载 `PE` 文件，找到 `PE` 入口，并把入口视作 `DLLMain`，以 `DLL_PROCESS_ATTACH` 为参数调用之。

所以不管你是 `EXE` 还是 `DLL`，入口总能得到运行。

如此，主线程将被交付 `CC`。![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")

试试呗。

实现 `CC`，内部使用 `HH` 的绝对地址。当然，这样 `CC` 程序是不能独立运行的。

修改 `HH` 入口，指定加载 `CC`。

不成功？`CC` 无法加载。![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

其实是小问题，用 `LordPE` 把 `CC` 的 `Characteristics` 加上 `DLL` 标志。

再来，成功了！！！成功利用了 `HH` 内部数据与代码！！！![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

好，再分析一下细节：注意到可能 `CC` 与 `HH` 的子系统不同，所以需要统一子系统。

另外，如果 `HH` 加过壳，这个方法就不能直接使用了。虽然俺不大会脱，但总有牛人会脱，脱了壳再应用，一样一样的。

给出实现 DEMO：

代码:

```cpp
// Replacer.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"

#include <windows.h>
#include <iostream>
using namespace std;
#include "netline.h"

void usage()
{
 cout<<endl;
 cout<<"Usage: Replacer Host Client [Client_argv]\\n"<<endl;
 cout<<"Options:"<<endl;
 cout<<"    Host\t 将占用原有的原始空间"<<endl;
 cout<<"    Client\t 将作为 DLL 被加载，需要有重定位信息"<<endl;
 cout<<"\t\t 主线程将被交付到入口"<<endl;
 cout<<"\t\t 注意导入库可能无法正确识别(环境设置)"<<endl;
 cout<<"    Client_argv\t 将被用于 Client 启动参数\\n"<<endl;
}


unsigned long GetRAbyRVA(HANDLE hMapView,unsigned long RVA)
{
 IMAGE_DOS_HEADER* doshead = (IMAGE_DOS_HEADER*)hMapView;
 IMAGE_NT_HEADERS* PEhead = (IMAGE_NT_HEADERS*)((unsigned long)doshead + (unsigned long)doshead->e_lfanew);
 IMAGE_SECTION_HEADER* ISH = (IMAGE_SECTION_HEADER*)((unsigned long)PEhead + sizeof(IMAGE_NT_HEADERS));
 IMAGE_EXPORT_DIRECTORY*  lpExpF = 0;
 for(int i=0;i<PEhead->FileHeader.NumberOfSections;++i)
   {
   if(RVA>=ISH->VirtualAddress && RVA<=(ISH->VirtualAddress+ISH->Misc.VirtualSize))
     {
     return( (unsigned long)hMapView + ISH->PointerToRawData + RVA - ISH->VirtualAddress );
     }
   ++ISH;
   }
 return 0;
}

const TCHAR* hostEXE = TEXT("host.exe");
const TCHAR* clientEXE = TEXT("client.exe");

int _tmain(int argc, _TCHAR* argv[])
{
  HANDLE hMapView = 0;
try
  {
  if(argc<3)    throw runtime_error("");
  cout<<"配置 Client 中...";
  if(!CopyFile(argv[2],clientEXE,false))    throw runtime_error("复制 Client 文件失败！");
  HANDLE hFile = CreateFile(clientEXE,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);
  if(hFile == INVALID_HANDLE_VALUE)        throw runtime_error("打开 Client 文件失败！");
  HANDLE hFileMapping = CreateFileMapping(hFile,0,PAGE_READWRITE,0,0,0);
  CloseHandle(hFile);
  if(!hFileMapping)        throw runtime_error("CreateFileMapping 失败！");
  hMapView = MapViewOfFile(hFileMapping,FILE_MAP_READ|FILE_MAP_WRITE,0,0,0);
  CloseHandle(hFileMapping);
  if(!hMapView)        throw runtime_error("MapViewOfFile 失败！");
  IMAGE_DOS_HEADER* doshead = (IMAGE_DOS_HEADER*)hMapView;
  if(doshead->e_magic != 'ZM')        throw runtime_error("检测 MZ 标志失败！");
  IMAGE_NT_HEADERS* PEhead = (IMAGE_NT_HEADERS*)((unsigned long)doshead + (unsigned long)doshead->e_lfanew);
  if(PEhead->Signature != 'EP')        throw runtime_error("检测 PE 标志失败！");
  WORD subsystem = PEhead->OptionalHeader.Subsystem;    //得到子系统
  IMAGE_DATA_DIRECTORY* IDD = PEhead->OptionalHeader.DataDirectory;
  IDD+=5;    //指向重定位表
  if(IDD->Size==0)    throw runtime_error("Client 没有重定位表，无法继续！");
  if((IMAGE_FILE_DLL & PEhead->FileHeader.Characteristics)!=IMAGE_FILE_DLL)
    {
    cout<<"Client 没有 DLL 标志，加载修改成 DLL...";
    PEhead->FileHeader.Characteristics = PEhead->FileHeader.Characteristics | IMAGE_FILE_DLL;
    }
  UnmapViewOfFile(hMapView);
  hMapView = 0;
  cout<<"完成！"<<endl;
  cout<<"配置 Host 中...";
  if(!CopyFile(argv[1],hostEXE,false))    throw runtime_error("复制 Host 文件失败！");
  hFile = CreateFile(hostEXE,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0);
  if(hFile == INVALID_HANDLE_VALUE)        throw runtime_error("打开 Host 文件失败！");
  hFileMapping = CreateFileMapping(hFile,0,PAGE_READWRITE,0,0,0);
  CloseHandle(hFile);
  if(!hFileMapping)        throw runtime_error("CreateFileMapping 失败！");
  hMapView = MapViewOfFile(hFileMapping,FILE_MAP_READ|FILE_MAP_WRITE,0,0,0);
  CloseHandle(hFileMapping);
  if(!hMapView)        throw runtime_error("MapViewOfFile 失败！");
  doshead = (IMAGE_DOS_HEADER*)hMapView;
  if(doshead->e_magic != 'ZM')        throw runtime_error("检测 MZ 标志失败！");
  PEhead = (IMAGE_NT_HEADERS*)((unsigned long)doshead + (unsigned long)doshead->e_lfanew);
  if(PEhead->Signature != 'EP')        throw runtime_error("检测 PE 标志失败！");
  if(subsystem!=PEhead->OptionalHeader.Subsystem)
    {
    cout<<"Host 与 Client 的子系统不同，修改匹配之...";
    PEhead->OptionalHeader.Subsystem = subsystem;
    }
  cout<<"\\n 重做 Host 入口...";
  unsigned long Entry = PEhead->OptionalHeader.AddressOfEntryPoint;
  Entry = GetRAbyRVA(hMapView,Entry);
  int ClientName_len = _tcslen(clientEXE)+sizeof(TCHAR);
  ClientName_len*=2;
  HMODULE hMod = GetModuleHandle(TEXT("Kernel32.dll"));
  unsigned long LodLib = (unsigned long)GetProcAddress(hMod,"LoadLibraryW");
  netline nline;
  nline.net(false);
  nline<<(unsigned char)0xB8<<LodLib;  //mov eax, LoadLibrary
  nline<<(unsigned char)0xE8;    //call ...
  nline<<ClientName_len;
  nline.setstr((unsigned char*)clientEXE,ClientName_len);
  nline.setstr((unsigned char*)"\\xFF\\xD0\\xC2\\x10\\x00",5);  //call   eax        retn 10
  nline.getstr((unsigned char*)Entry,(int)nline);
  cout<<"搞定！"<<endl;
  UnmapViewOfFile(hMapView);
  hMapView = 0;
  }
catch (runtime_error &err)
  {
  DeleteFile(clientEXE);
  DeleteFile(hostEXE);
  if(hMapView)UnmapViewOfFile(hMapView);
  cout<<err.what()<<endl;
  usage();
  }
catch (...)
  {
  cout<<"未知异常！"<<endl;
  usage();
  }
  return 0;
}
```
