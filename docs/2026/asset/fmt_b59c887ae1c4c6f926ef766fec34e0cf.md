
# [原创]dll模块隐藏的另一种方法及实现细节

标 题：【原创】dll模块隐藏的另一种方法及实现细节

作 者：J.Boy

EMail：ajunboy@GMail.com

时 间：2009-04-08, 18:13

## 背景

关于隐藏 DLL 模块的方法网上也说得很多，如 NetRoc 大牛的《进程中dll模块的隐藏》一文中提到的抹链的方法。

自己印象比较深的还有另一种方法，也是很古老的时候就有大牛提出并实现了的，但却没有提及细节以及具体实现代码，且笔者至今也还没看到有详细介绍此种方法以及细节的文章，可能是小弟孤陋寡闻吧....

于是翻出这份早已发霉在硬盘里的代码来和大家分享一下，小弟才疏学浅，难免有错误的地方，希望大牛们能给予指正。

## 思路

说了半天废话，现在回到正题，思路很简单，就是先让 DLL 正常加载，然后 copy 一份完整的 DLL 内存映象，把 DLL free 掉，再以 DLL 正常加载的 imagebase 为基址申请与原 DLL 内存映象相同大小的内存，然后再把直接 copy 的映象备份复制回去，这样该 DLL 就在 DLL 链中消失了，并且免去了重定位的麻烦。

## 实现细节

思路说着顺畅，但是实现起来仍有一些细节需要注意：

一、在我们 `FreeLibrary` 的时候，系统会连 DLL 中用到的一些对象或资源一起释放掉。

二、同样是 `FreeLibrary` 的时候，系统可能会将 `LoadLibrary` 时一同加载的该模块用到的其他 DLL 一起卸载掉。

在 N 年前刚看到大牛们说到这个思路，自己实现时就因为不了解上面 2 个问题的存在，而盘旋在各种错误框中....- -!

其实这 2 个问题都是很好解决的：

一、在 `FreeLibrary` 时，会调用 EP 进行一些资源的卸载，我的处理方法是直接把 EP 处改为 `retn`，让它直接返回掉，而不去释放资源。

二、`FreeLibrary` 时系统会把该 DLL 所用到的其他 DLL 的 `LoadCount` 减 1，然后判断 `LoadCount` 为 0 则把该 DLL 也 Free 掉，否则无做任何操作。这里我的处理方法是在之前就用 `LoadLibrary` 来使同进程中的其他 DLL 的 `LoadCount` 增加。

## 代码实现

这样就 OK 了，下面是 Delphi 的实现代码：

```pascal
unit HideModuleUnit;

interface

uses Windows, Messages, SysUtils, Classes, TlHelp32;

type

  TVirtualAlloc = function (lpvAddress: Pointer; dwSize, flAllocationType,
    flProtect: DWORD): Pointer; stdcall;

  TVirtualProtect = function (lpAddress: Pointer; dwSize, flNewProtect: DWORD;
      var OldProtect: DWORD): BOOL; stdcall;

  TVirtualFree = function (lpAddress: Pointer;
    dwSize, dwFreeType: DWORD): BOOL; stdcall;

  TWriteProcessMemory = function (hProcess: THandle;
    const lpBaseAddress: Pointer; lpBuffer: Pointer; nSize: DWORD;
    var lpNumberOfBytesWritten: DWORD): BOOL; stdcall;

  TGetCurrentProcess = function : THandle; stdcall;

  TFreeLibrary = function (hLibModule: HMODULE): BOOL; stdcall;

  THideModuleRec = record

    pModule: pointer;

    pVirtualAlloc: TVirtualAlloc;

    pVirtualProtect: TVirtualProtect;

    pVirtualFree: TVirtualFree;

    pWriteProcessMemory: TWriteProcessMemory;

    pGetCurrentProcess: TGetCurrentProcess;

    pFreeLibrary: TFreeLibrary;

  end;

  PHideModuleRec = ^THideModuleRec;

  procedure HideModule(hModule: THandle);

implementation

procedure ExecuteHide(HM: THideModuleRec);

var

  pBakMemory: pointer;

  ImageOptionalHeader: TImageOptionalHeader;

  ImageDosHeader: TImageDosHeader;

  td: dword;

  i: Integer;

begin

  { 取得映象数据 }

  ImageDosHeader := PImageDosHeader(HM.pModule)^;

  ImageOptionalHeader := PImageOptionalHeader(Pointer(integer(HM.pModule) +

    ImageDosHeader._lfanew + SizeOf(dword) +

      SizeOf(TImageFileHeader)))^;

  { 申请内存以备份原始模块数据 }

  pBakMemory := HM.pVirtualAlloc(nil, ImageOptionalHeader.SizeOfImage,

    MEM_COMMIT or MEM_RESERVE, PAGE_EXECUTE_READWRITE);

  if pBakMemory = nil then

    exit;

  { 修改原始内存为可读写属性 }

  HM.pVirtualProtect(HM.pModule, ImageOptionalHeader.SizeOfImage,

    PAGE_EXECUTE_READWRITE, td);

  { 备份原始模块数据 }

  HM.pWriteProcessMemory(HM.pGetCurrentProcess, pBakMemory, HM.pModule,

    ImageOptionalHeader.SizeOfImage, td);

  { 修改原DllEntryPoint为retn,防止FreeLibrary时的一些卸载操作 }

  pByte(integer(HM.pModule) + ImageOptionalHeader.AddressOfEntryPoint)^ := $C3;

  { 卸载原模块,这里多次卸载防止因为LoadCount的关系一次卸载不掉 }

  //HM.pFreeLibrary(integer(HM.pModule));

  i := 0;

  repeat

    Inc(i);

  until not HM.pFreeLibrary(integer(HM.pModule)) or (i >= 30);

  { 申请dll原始加载地址空间 }

  HM.pModule := HM.pVirtualAlloc(HM.pModule, ImageOptionalHeader.SizeOfImage,

    MEM_COMMIT or MEM_RESERVE, PAGE_EXECUTE_READWRITE);

  if HM.pModule = nil then

    exit;

  { 写回原始数据 }

  HM.pWriteProcessMemory(HM.pGetCurrentProcess, HM.pModule, pBakMemory,

    ImageOptionalHeader.SizeOfImage, td);

  { 释放备份时用的内存 }

  HM.pVirtualFree(pBakMemory, 0, MEM_RELEASE);

end;

(*注意该间隔处不能添加任何代码, 且不能改变上下2个函数位置

  因为下面使用了2个函数入口来计算上一函数Size*)

procedure LockedAllModule(CurrentModuleHandle: THandle);

var

  ModuleList: THandle;

  pm: tagMODULEENTRY32;

begin

  pm.dwSize := sizeof(tagMODULEENTRY32);

  ModuleList := CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, 0);

  if not Module32First(ModuleList, pm) then

  begin

    CloseHandle(ModuleList);

    exit;

  end;

  //不处理第一个模块,因为那是主模块

  { 对每个模块LoadLibrary一次,是为了把LoadCount加1 }

  while Module32Next(ModuleList, pm) do

  begin

    if pm.hModule <> CurrentModuleHandle then

      LoadLibrary(PAnsiChar(@GetModuleName(pm.hModule)[1]));

  end;

  CloseHandle(ModuleList);

end;

procedure HideModule(hModule: THandle);

type

  TExecuteHide = procedure (HM: THideModuleRec);

var

  HM: THideModuleRec;

  pExecuteHide: pointer;

  ExecuteHideSize: integer;

  MyExecuteHide: TExecuteHide;

  td: dword;

  Module_kernel32: integer;

begin

  Module_kernel32 := GetModuleHandle('kernel32.dll');

  HM.pModule := pointer(hModule);

  HM.pVirtualAlloc := GetProcAddress(Module_kernel32, 'VirtualAlloc');

  HM.pVirtualProtect := GetProcAddress(Module_kernel32, 'VirtualProtect');

  HM.pVirtualFree := GetProcAddress(Module_kernel32, 'VirtualFree');

  HM.pWriteProcessMemory := GetProcAddress(Module_kernel32, 'WriteProcessMemory');

  HM.pGetCurrentProcess := GetProcAddress(Module_kernel32, 'GetCurrentProcess');

  HM.pFreeLibrary := GetProcAddress(Module_kernel32, 'FreeLibrary');

  ExecuteHideSize := integer(@LockedAllModule) - integer(@ExecuteHide);

  pExecuteHide := VirtualAlloc(nil, ExecuteHideSize,

    MEM_COMMIT or MEM_RESERVE, PAGE_EXECUTE_READWRITE);

  if pExecuteHide = nil then

    exit;

  { 防止系统将需要的Dll卸载掉 }

  LockedAllModule(integer(HM.pModule));

  CopyMemory(pExecuteHide, @ExecuteHide, ExecuteHideSize);

  MyExecuteHide := pExecuteHide;

  MyExecuteHide(HM);

end;

end.
```

## 遗留问题

以上代码就实现了 DLL 的隐藏，不过由于该份代码早已发霉，没有处理以下几个特殊问题：

一、多线程。

二、模块内部取模块信息时的处理。

同样的，处理方法也很简单：

一、在 Free 模块前，先把进程内当前线程以外的所有线程 Suspend，处理完之后再 resume，就 OK 了。

二、Hook `GetModuleName`、`GetModuleHandle` 等取模块信息的函数，判断到是隐藏的 DLL 的代码段在调用（判断返回地址），就给予相应的信息即可。

不过这里第 2 个问题的解决方法有失隐秘性，希望有大牛能提点下更好的办法。

第一次写文，有错误之处还请各位大大见谅并给予指正。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
