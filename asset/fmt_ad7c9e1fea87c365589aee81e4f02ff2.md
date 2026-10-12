
# PeCompact 脱壳内存镜像输入表修复工具

## 概述

PeCompact 完美脱壳时抓取的内存镜像文件的输入表有些问题，IID 数组中 `FirstThunk` 指向的区域(IAT)全部为 0。
这个程序用来修复输入表，修复方式很简单，就是：
将 `OriginalFirstThunk` 指向的内容(INT)全部拷贝到 `FirstThunk` 指向的区域(IAT)。

*   created by asdfslw
*   2009.11.20

## 修复程序源代码

```c
#include <windows.h>

// 全局变量
HANDLE hFile;
HANDLE hFileMap;
PBYTE  pbFile;
PIMAGE_DOS_HEADER       pDos;
PIMAGE_NT_HEADERS       pNt;
PIMAGE_SECTION_HEADER   pSec;
PIMAGE_IMPORT_DESCRIPTOR pIID;

// rva 转换为 offset
DWORD RVA_2_Offset(DWORD rva)
{
  DWORD offset;
  PIMAGE_SECTION_HEADER    pSec1 = pSec + 1;
  if (rva < pSec->VirtualAddress)
  {
    offset = rva;
    return offset;
  }
  for (int i=0; i<pNt->FileHeader.NumberOfSections-1; i++)
  {
    if (rva<pSec1->VirtualAddress)
    {
      --pSec1;
      offset = rva - pSec1->VirtualAddress + pSec1->PointerToRawData;
      return offset;
    }
    ++pSec1;
  }
  --pSec1;
  offset = rva - pSec1->VirtualAddress + pSec1->PointerToRawData;
  return offset;
}

// 自定义的拷贝函数
void __declspec(naked) my_copy (DWORD dst, DWORD src)
{
  __asm
  {
    push ebp
    mov ebp, esp
    push esi
    push edi
    mov edi, [ebp+8h]  /* dst */
    mov esi, [ebp+0ch] /* src */
_loop_begin:
    lods dword ptr [esi]
    test eax, eax
    jz _loop_end
    stos dword ptr [edi]
    jmp _loop_begin
_loop_end:
    pop edi
    pop esi
    pop ebp
    retn
  }
}

int WINAPI WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
  char FileName[MAX_PATH] = {0};
  OPENFILENAME ofn;
  ofn.lStructSize     = sizeof(OPENFILENAME);
  ofn.hwndOwner     = 0;
  ofn.lpstrFilter     = "Exe file (*.exe)\\0*.exe\\0\0";
  ofn.lpstrCustomFilter  = NULL;
  ofn.nFilterIndex   = 1;
  ofn.lpstrFile     = FileName;
  ofn.nMaxFile     = sizeof FileName;
  ofn.lpstrFileTitle   = NULL;
  ofn.nMaxFileTitle   = 0;
  ofn.lpstrInitialDir   = NULL;
  ofn.lpstrTitle     = "Pecompact 完美脱壳修复工具 by asdfslw";
  ofn.Flags       = OFN_EXPLORER | OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST | OFN_HIDEREADONLY | OFN_ENABLESIZING;
  ofn.lpstrDefExt     = NULL;

  if(0==GetOpenFileName(&ofn))
  {
    goto _error;
  }

  // 打开文件
  hFile = CreateFileA(FileName, GENERIC_READ | GENERIC_WRITE, NULL, NULL,
    OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
  if (INVALID_HANDLE_VALUE==hFile)
  {
    MessageBoxA(NULL, "打开文件错误！", "错误", MB_ICONEXCLAMATION);
    goto _error;
  }

  // 创建文件映射
  hFileMap = CreateFileMappingA(hFile, NULL, PAGE_READWRITE, NULL, NULL, NULL);
  if (NULL==hFileMap)
  {
    MessageBoxA(NULL, "创建文件映射错误！", "错误", MB_ICONEXCLAMATION);
    CloseHandle(hFile);
    goto _error;
  }

  // 将文件数据映射到进程地址空间
  pbFile = (PBYTE)MapViewOfFile(hFileMap, FILE_MAP_WRITE, 0, 0, 0);
  if (NULL==pbFile)
  {
    MessageBoxA(NULL, "将文件数据映射到进程地址空间错误！", "错误", MB_ICONEXCLAMATION);
    CloseHandle(hFileMap);
    CloseHandle(hFile);
    goto _error;
  }

  // 获取 PE 结构相关数据
  pDos = (PIMAGE_DOS_HEADER)    pbFile;
  pNt  = (PIMAGE_NT_HEADERS)    ( (PBYTE)pDos + pDos->e_lfanew );
  pSec = (PIMAGE_SECTION_HEADER) ( (PBYTE)pNt + 0xF8 );
  pIID = (PIMAGE_IMPORT_DESCRIPTOR) (
    RVA_2_Offset(pNt->OptionalHeader.DataDirectory[1].VirtualAddress)
    + (DWORD)pbFile );

  while (NULL!=pIID->Name)
  {
    my_copy(
      RVA_2_Offset(pIID->FirstThunk)+ (DWORD)pbFile,
      RVA_2_Offset(pIID->OriginalFirstThunk) + (DWORD)pbFile );
    ++pIID;
  }

  MessageBoxA(NULL, "成功", "呵呵", MB_OK);

  UnmapViewOfFile(pbFile);
  CloseHandle(hFileMap);
  CloseHandle(hFile);
  return 0;
_error:
  ExitProcess(0);
}
```

本人在脱壳方面也是一个初学者。笔记中有什么不对的地方，希望各位高手指正。谢谢![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

另外，大家可以参考看雪如下文章

[`bbs.pediy.com/showthread.php?t=10932`](http://bbs.pediy.com/showthread.php?t=10932)

这篇文章的思路与我的相同（不过写这篇笔记之前我并没有看到这篇文章，之后才搜索到的）。但遗憾的是这篇文章里面没有对脱壳后的文件的输入表进行修复，我则找出了脱壳后的文件不能运行的问题所在输入表有问题，并编写了一个修复程序（虽然修复其实很简单）。

【版权声明】本文由 asdfslw 原创于看雪软件安全论坛，转载请注明出处，并保留版权声明。

更新：添加了修复程序 `PeCompact_IAT_xiufu.rar`。

关于修复程序的说明：如果你的电脑上安装有主动防御类软件或者其他安全软件，在修复程序运行的时候可能会报警。比如说，我的电脑上安装有微点主动防御软件，修复程序会修改可执行文件，默认情况下微点对修改可执行文件的程序都会报警。此时只要点允许，并把修复程序添加到可信程序中，重新运行一次修复程序就可以了。本人保证此修复程序是没有任何危害的。

我以前写过一个 `PeCompact` 的脚本，效果应该相当于静态脱壳机：

`http://bbs.pediy.com/showthread.php?t=83885`

`msg "已经到 OEP 了。程序已 dump 后另存为 UN_+原文件名。OEP 、输入表、重定位（若有）都已修正，区段都已重建。若要优化大小，请查看输出表及资源是否在最后那个“CanBeDel”段中，若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"`

"若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"

怎么重建输入表及资源？![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

另外，想请教下 `OD` 脚本怎么编写？哪里有教程或者资料？

> 引用: *最初由 **asdfslw**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)`msg "已经到 OEP 了。程序已 dump 后另存为 UN_+原文件名。OEP 、输入表、重定位（若有）都已修正，区段都已重建。若要优化大小，请查看输出表及资源是否在最后那个“CanBeDel”段中，若在，请重建输出表及资源后再删除最后的那个“CanBeDel”段。"`
>
> &quot...

不需要重建输入表，输入表已经建好了。脱出来的程序直接就可以运行。如果要优化大小的话，对于 `DLL` 这样的有输出表的，可能要把输出表从那个“CanBeDel”段中移到别的地方去（可以用 `kanxue` 兄的那个 `pemove` 小工具来完成），资源直接用 `FixRes` dump 一份，再删掉那个“CanBeDel”段，把 dump 后的资源导入到最后。

脚本编写可以参考 `hnhuqiong` 的教程，这个帖子上有列表：

`http://bbs.pediy.com/showpost.php?p=514664&postcount=2`

另外就是看别人的脚本，然后在 `OD` 中单步跑一遍，基本上就清楚了。只要会用 `OD`，再了解一下脚本命令，应该就可以写出脚本。
