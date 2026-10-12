
# 对付 themida 的工具及文章

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-742.htm`](https://www.pediy.com/kssd/pediy07/pediy7-742.htm)

接着 声声慢 的继续。。。续：[`bbs.pediy.com/showthread.php...;threadid=12834`](http://bbs.pediy.com/showthread.php?s=&threadid=12834)

写这篇文章是为了好玩,不要太认真的对待它。 前段时间有个朋友(Littleluk)问我是否有兴趣帮助他写脱 themida (xprotector 商业后继者)的壳的程序。 我同意了。 然而，我却对它有另外的一个想法.因为坦诚的来讲,我不想把我的时间浪费在无用的事情上,而 themida 就是其中的一件.对那些还不知道我在讲什么的人: themida 是一个壳,而且它是在 kernel 形式下运行的壳.这意味着:要是您要用调试器去调试 themida,它将要崩溃您的系统.很不专业,我知道.这是我把 themida 看作逆向人士的玩具的主要原因.直接的说,themida 将永远不会被大量的公司使用,没有人会用这样一个给微软的 kernel,SDT,IDT 注补丁,时常令机器重启动等等的壳来保护一个很好的软件.这意味这它不会被广泛使用,除了对它本身的脱壳乐趣,不会有太大的意义.我对使用无意义的东西没有兴趣,但既然 Littleluk 提出,我就编写出一些东西(使用一些我以前编的旧代码).因此,这篇文章是围绕着我所写的一个工具.这不是脱 themida 壳的工具(那是不可能的:Littleluk 还没有成功的逆向它).这个工具能使您电脑上已有的一些工具来防卫 themida.

这里有一些警告：

1. 这篇文章没有牵扯到核心部分的操作，你不需要有驱动编程的知识，这对那些驱动的编写者来说有些荒谬(而且我会说 '因为在做逆向的人中你可以很容易得找到对核心编程一无所知的人)。

2. 在这篇文章中尼了解到的东西会十几年前的东西(就像 themida 使用的那些陷阱一样)。

3. 我从来就没碰过 themida，我能写出这个工具并不是因为我的经验，而且结果是从 Littleluk 那里的来的。

4. 我不会去解释什么是 SDT、IDT 以及其他内在相关的概念的含义。如果你不知道这些涵义的话，可以去网上找找。我也没时间去解释那些不太重要的，事实上这也是我现在放出这篇文章的原因之一。我刚好有几天的时间然后又得回去工作(现实中的工作)。

5. 我会解释我所知道的 themida，或更准确地说你需要理解我写的那个工具。我不打算逆向 themida，所以对于很多事情我并不是很关心。

6. 我不知道会不会有什么其它的 AntiMida，这完全取决于逆向工作者提供给我的信息量以及我的空余时间。

7. 这个方法在你使用 PAE 扩展的时候不起作用(我已经懒得再去添加一些代码了)。

8. 这个工具的工作方式也不是绝对的，还是有其他的方法的。

这次用于测试的牺牲品不是什么其它的东西而是 themida 自身(我是指试用版)。你可以从他的官方网站上下载他(当前版本是 1.0.0.5)。

AntiMida 不是一个计划过的工具，只是每次我遇到问题时而编写的解决方案。现在 AntiMida 可以让你：

1. 使用普通用户模式的应用程序去转存 themida.

2. 使用 imprec, winhex (察看程序的内存), 等等。

3. 监视文件及注册表的访问。

但是一次最多做一个。第一步是转存 themida。怎么做？首先我们需要知道 themida 用于保护它自己防止被转存的方法。实际上首先想到去用的是 KeAttachProcess，我们转存了 ntoskrnl (有点痛苦) 然后看到 keattachprocess 被改成了一个 jmp 到 themida 入口。所以要使用 keattachprocess 的话，就必须先修改 ntoskrnl(我在下面贴出了代码)。下面是我使用 KeAttachProcess 写的入口：

```
    case CODE\_READ\_MEM:
      {
         MemReadInfo mri;
         ULONG\_PTR ptr, addr;
         BYTE \*Buffer;
         UINT x, y;
         RtlCopyMemory(&mri, pInput, sizeof (MemReadInfo));
         if (PsLookupProcessByProcessId(mri.PID, &ptr) != STATUS\_SUCCESS)
            return STATUS\_INVALID\_PARAMETER;
         Buffer = (BYTE \*)  ExAllocatePool(NonPagedPool, dwOutputSize);
         if (Buffer == NULL)
            return STATUS\_INVALID\_PARAMETER;
         KeAttachProcess(ptr);
         if (dwOutputSize <= 0x1000)
         {
            x = 1;
         }
         else
         {
            x = dwOutputSize / 0x1000;
            if (dwOutputSize % 0x1000 != 0)
               x++;
         }
         for (y = 0; y < x; y++)
         {
            addr = y \* 0x1000 + (ULONG\_PTR) mri.Address;
            if (MmIsAddressValid((PVOID) addr) == FALSE)
            {
               ExFreePool(Buffer);
               return STATUS\_INVALID\_PARAMETER;
            }
         }
         RtlCopyMemory(Buffer, mri.Address, dwOutputSize);
         KeDetachProcess();
         RtlCopyMemory(pOutput, Buffer, dwOutputSize);
         ExFreePool(Buffer);
         \*pdwInfo = dwOutputSize;
         break;
      }
```

我贴这段代码只是为了信息，因为这段代码现在已经不在 AntiMida 中了。很显然 themida 正在关联 SDT，然后在使用 sdtrestore 扫描之后我得到了一个被关联的服务的列表：

ZwAllocateVirtualMemory 11 --[未知关联于 F5938BC4]--

ZwCreateThread 35 --[未知关联于 F5938CBE]--

ZwDebugContinue 3A --[未知关联于 F59391A0]--

ZwQueryVirtualMemory B2 --[未知关联于 F5938ACA]--

ZwReadVirtualMemory BA --[未知关联于 F5938014]--

ZwTerminateProcess 101 --[未知关联于 F59389D0]--

ZwWriteVirtualMemory 115 --[未知关联于 F5938000]--

很好 ZwAllocateVirtualMemory, ZwQueryVirtualMemory, ZwReadVirtualMemory, ZwWriteVirtualMemory, ZwCreateThread 是有用的，因为我们需要这些函数来转存 themida。ZwDebugContinue 是用来避开调试的，虽然 themida 也对 IDT 进行了一番胡搞(所以这可能就是反调试的手段，当然可能太简单了些)。ZwTerminateProcess 不是重要的，我猜他被关联只是为了能知道被保护的进程退出了。不过如果你尝试恢复这些服务中的任何一个的话，themida 就会整垮你的系统。

我想到的注意使制作一个工具室的其他已经存在的工具可以使用。所以我构建了一个列有当前运行的进程的列表，并提供选择其中一个使其抗 themida 的可能，这意味着对 themida 的免疫。一个简洁的界面还是要花上不少代码的。

当那个小按钮被按下时，一个 dll 会被注入选中的进程的地址空间，被 themida 关联以保护它自己的函数会被重定向到这个注入的 dll 上，每当调用一个被关联的函数时，这个 dll 会调用驱动来完成操作。那么我的驱动是怎么完成这些操作的呢？稍后你就会看到。现在先看看注入过程：

```
BOOL InjectModule(IN ULONG\_PTR ProcessID, IN TCHAR \*ModuleName,
            OUT ULONG\_PTR \*BaseAddress OPTIONAL)
{
   HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE,
      ProcessID);
   if (hProcess == NULL)
      return FALSE;
   HANDLE hFile = CreateFile(ModuleName, GENERIC\_READ, FILE\_SHARE\_READ,
      NULL, OPEN\_EXISTING, 0, NULL);
   if (hFile == INVALID\_HANDLE\_VALUE)
   {
      CloseHandle(hProcess);
      return FALSE;
   }
   UINT PE\_Size = GetFileSize(hFile, NULL);
   BYTE \*PE\_Buffer = (BYTE \*) VirtualAlloc(NULL, PE\_Size,
      MEM\_COMMIT, PAGE\_READWRITE);
   if (PE\_Buffer == NULL)
   {
      CloseHandle(hProcess);
      CloseHandle(hFile);
      return FALSE;
   }
   DWORD BR;
   if (!ReadFile(hFile, PE\_Buffer, PE\_Size, &BR, NULL))
   {
      VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);
      CloseHandle(hProcess);
      CloseHandle(hFile);
      return FALSE;
   }
   CloseHandle(hFile);
   BYTE \*PE\_Image;
   IMAGE\_DOS\_HEADER \*ImgDosHdr;
   IMAGE\_NT\_HEADERS \*ImgNtHdrs;
   ULONG\_PTR ImgBase, Delta, Reloc\_Offset;
   IMAGE\_BASE\_RELOCATION \*ImgBaseReloc;
   WORD \*wData;
   UINT i, nItems;
   ULONG\_PTR Offset;
   DWORD Type;
   ULONG\_PTR \*Block, BlockOffs;
   ULONG\_PTR IT\_Offset;
   IMAGE\_IMPORT\_DESCRIPTOR \*ImgImpDescr;
   UINT x = 0, y = 0;
   CHAR \*DllName;
   ULONG\_PTR \*Thunks, \*FThunks;
   IMAGE\_IMPORT\_BY\_NAME \*ImgImpName;
   \_try
   {
      ImgDosHdr = (IMAGE\_DOS\_HEADER \*) PE\_Buffer;
      ImgNtHdrs = (IMAGE\_NT\_HEADERS \*) (ImgDosHdr->e\_lfanew +
         (ULONG\_PTR) PE\_Buffer);
      if (ImgDosHdr->e\_magic != IMAGE\_DOS\_SIGNATURE ||
         ImgNtHdrs->Signature != IMAGE\_NT\_SIGNATURE)
      {
         VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);
         CloseHandle(hProcess);
         return FALSE;
      }
      ImgBase = (ULONG\_PTR) VirtualAllocEx(hProcess,
         (PVOID) 0, //ImgNtHdrs->OptionalHeader.ImageBase,
         ImgNtHdrs->OptionalHeader.SizeOfImage, MEM\_COMMIT,
         PAGE\_EXECUTE\_READWRITE);
      if (ImgBase == NULL)
      {
         VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);
         CloseHandle(hProcess);
         return FALSE;
      }
      //
      // 文件的映像基址 == 内存映像基址吗？
      // 不是则重定位
      //
      if (ImgNtHdrs->OptionalHeader.ImageBase != ImgBase)
      {
         Delta = ImgBase - ImgNtHdrs->OptionalHeader.ImageBase;
         if (!ImgNtHdrs->OptionalHeader.DataDirectory
            \[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress ||
            !(Reloc\_Offset = RvaToOffset(ImgNtHdrs,
            ImgNtHdrs->OptionalHeader.DataDirectory
            \[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress)))
         {
            VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);
            VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);
            CloseHandle(hProcess);
            return FALSE;
         }
         ImgBaseReloc = (IMAGE\_BASE\_RELOCATION \*)
            (Reloc\_Offset + (ULONG\_PTR) PE\_Buffer);
         do
         {
            if (!ImgBaseReloc->SizeOfBlock)
               break;
            nItems = (ImgBaseReloc->SizeOfBlock -
               IMAGE\_SIZEOF\_BASE\_RELOCATION) / sizeof (WORD);
            wData = (WORD \*)(IMAGE\_SIZEOF\_BASE\_RELOCATION +
               (ULONG\_PTR) ImgBaseReloc);
            for (i = 0; i < nItems; i++)
            {
               Offset = (\*wData & 0xFFF) + ImgBaseReloc->VirtualAddress;
               Type = \*wData >> 12;
```



```
if (Type != IMAGE_REL_BASED_ABSOLUTE)
{
   BlockOffs = RvaToOffset(ImgNtHdrs, Offset);
   if (BlockOffs == NULL)
   {
      VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM_RELEASE);
      VirtualFree(PE_Buffer, 0, MEM_RELEASE);
      CloseHandle(hProcess);
      return FALSE;
   }
   Block = (DWORD *)(BlockOffs + (ULONG_PTR) PE_Buffer);
   *Block += Delta;
}
wData++;
}
ImgBaseReloc = (PIMAGE_BASE_RELOCATION) wData;
} while (*(DWORD *) wData);
}

//
// 填充输入地址表
//
if (ImgNtHdrs->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress)
{
   IT_Offset = RvaToOffset(ImgNtHdrs,
      ImgNtHdrs->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);
   ImgImpDescr = (IMAGE_IMPORT_DESCRIPTOR *) (IT_Offset +
      (ULONG_PTR) PE_Buffer);
   // 对每一个描述符
   while (ImgImpDescr[x].FirstThunk != 0)
   {
      DllName = (CHAR *) (RvaToOffset(ImgNtHdrs,
         ImgImpDescr[x].Name) + (ULONG_PTR) PE_Buffer);
      Thunks = (ULONG_PTR *) (RvaToOffset(ImgNtHdrs,
         ImgImpDescr[x].OriginalFirstThunk != 0 ?
         ImgImpDescr[x].OriginalFirstThunk :
      ImgImpDescr[x].FirstThunk) + (ULONG_PTR) PE_Buffer);
      FThunks = (ULONG_PTR *) (RvaToOffset(ImgNtHdrs,
         ImgImpDescr[x].FirstThunk) + (ULONG_PTR) PE_Buffer);
      y = 0;
      //
      // 模块中的每一个导入的函数
      //
      while (Thunks[y] != 0)
      {
         //
         // 原始导入的吗？
         //
         if (Thunks[y] & IMAGE_ORDINAL_FLAG)
         {
            FThunks[y] = (ULONG_PTR) GetProcAddress(
               GetModuleHandle(DllName),
               (LPCSTR) (Thunks[y] - IMAGE_ORDINAL_FLAG));
            y++;
            continue;
         }
         ImgImpName = (IMAGE_IMPORT_BY_NAME *) (RvaToOffset(
            ImgNtHdrs, Thunks[y]) + (ULONG_PTR) PE_Buffer);
         FThunks[y] = (ULONG_PTR) GetProcAddress(GetModuleHandle(DllName),
            (LPCSTR) &ImgImpName->Name);
         y++;
      }
      x++;
   }
}

}_except (EXCEPTION_EXECUTE_HANDLER)
{
   VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM_RELEASE);
   VirtualFree((LPVOID) PE_Buffer, 0, MEM_RELEASE);
   CloseHandle(hProcess);
   return FALSE;
}

//
// 创建 PE 文件的虚拟映像
//
PE_Image = (BYTE *) VirtualAlloc(NULL,
   ImgNtHdrs->OptionalHeader.SizeOfImage,
   MEM_COMMIT, PAGE_READWRITE);
if (PE_Image == NULL)
{
   VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM_RELEASE);
   VirtualFree(PE_Buffer, 0, MEM_RELEASE);
   CloseHandle(hProcess);
   return FALSE;
}

ZeroMemory(PE_Image, ImgNtHdrs->OptionalHeader.SizeOfImage);

//
// 复制头部
//
IMAGE_SECTION_HEADER *Sect;
Sect = IMAGE_FIRST_SECTION(ImgNtHdrs);
RtlCopyMemory(PE_Image, PE_Buffer, Sect[0].PointerToRawData);

//
// 映射区段
//
for (UINT j = 0; j < ImgNtHdrs->FileHeader.NumberOfSections; j++)
{
   BYTE *Source = (BYTE *)(Sect[j].PointerToRawData +
      (ULONG_PTR) PE_Buffer);
   BYTE *Dest = (BYTE *)(Sect[j].VirtualAddress +
      (ULONG_PTR) PE_Image);
   RtlCopyMemory(Dest, Source, Sect[j].SizeOfRawData);
}

BOOL bRet = WriteProcessMemory(hProcess, (LPVOID) ImgBase,
   PE_Image, ImgNtHdrs->OptionalHeader.SizeOfImage, &BR);

if (!bRet)
   VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM_RELEASE);

VirtualFree(PE_Image, 0, MEM_RELEASE);
VirtualFree(PE_Buffer, 0, MEM_RELEASE);
CloseHandle(hProcess);

if (bRet == TRUE && BaseAddress != NULL)
   *BaseAddress = ImgBase;

return bRet;
}
```

为了填充输入表我使用了远程的`GetProcAddress`因为`dll`的输入表只是没有定位到系统的系统的`dll`上，所以一个本地的`GetProcAddress`就已经足够了。下面的关联过程：

```
void CAntiMidaDlg::OnBnClickedAntimida()
{
   ProcList.GetItemText(ProcList.GetNextItem(-1, LVNI_SELECTED), 1,
      Buffer, sizeof(Buffer) -1);
   ULONG_PTR PID = _tcstoul(Buffer, 0, 16);
   wsprintf(Buffer, _T("%samdll.dll"), CurDir);
   ULONG_PTR BaseAddress;
   //
   // 注入模块
   //
   if (InjectModule(PID, _T("amdll.dll"), &BaseAddress))
   {
      HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, PID);
      if (hProcess == NULL)
      {
         MessageBox(_T("Cannot make the process AntiMida"), "AntiMida");
         return;
      }
      //
      // 关联 ReadProcessMemory
      //
      ULONG_PTR Addr = (ULONG_PTR) GetProcAddress(
         GetModuleHandle(_T("kernel32.dll")),
         "ReadProcessMemory");
      BYTE Instr = 0xB8;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,
         sizeof(BYTE), NULL);
      Addr++;
      ULONG_PTR Rva = GetExportRva(Buffer, "_FakeReadProcessMemory@20");
      ULONG_PTR HookVA = BaseAddress + Rva;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,
         sizeof(ULONG_PTR), NULL);
      Addr += sizeof(ULONG_PTR);
      WORD Instr2 = 0xE0FF;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,
         sizeof(WORD), NULL);

      //
      // 关联 VirtualQueryEx
      //
      Addr = (ULONG_PTR) GetProcAddress(
         GetModuleHandle(_T("kernel32.dll")),
         "VirtualQueryEx");
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,
         sizeof(BYTE), NULL);
      Addr++;
      Rva = GetExportRva(Buffer, "_FakeVirtualQueryEx@16");
      HookVA = BaseAddress + Rva;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,
         sizeof(ULONG_PTR), NULL);
      Addr += sizeof(ULONG_PTR);
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,
         sizeof(WORD), NULL);

      //
      // 关联 WriteProcessMemory
      //
      Addr = (ULONG_PTR) GetProcAddress(
         GetModuleHandle(_T("kernel32.dll")),
         "WriteProcessMemory");
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,
         sizeof(BYTE), NULL);
      Addr++;
      Rva = GetExportRva(Buffer, "_FakeWriteProcessMemory@20");
      HookVA = BaseAddress + Rva;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,
         sizeof(ULONG_PTR), NULL);
      Addr += sizeof(ULONG_PTR);
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,
         sizeof(WORD), NULL);

      //
      // 关联 NtAllocateVirtualMemory
      // 从这里开始我就直接关联 ntdll
      //
      Addr = (ULONG_PTR) GetProcAddress(
         GetModuleHandle(_T("ntdll.dll")),
         "NtAllocateVirtualMemory");
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,
         sizeof(BYTE), NULL);
      Addr++;
      Rva = GetExportRva(Buffer, "_FakeNtAllocateVirtualMemory@24");
      HookVA = BaseAddress + Rva;
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,
         sizeof(ULONG_PTR), NULL);
      Addr += sizeof(ULONG_PTR);
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,
         sizeof(WORD), NULL);

      //
      // 关联 NtCreateThread
      //
```



```cpp
   Addr = (ULONG_PTR) GetProcAddress(
      GetModuleHandle(_T("ntdll.dll")), 
      "NtCreateThread"); 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr, 
      sizeof (BYTE), NULL); 
   Addr++; 
   Rva = GetExportRva(Buffer, "_FakeNtCreateThread@32"); 
   HookVA = BaseAddress + Rva; 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA, 
      sizeof (ULONG_PTR), NULL); 
   Addr += sizeof (ULONG_PTR); 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2, 
      sizeof (WORD), NULL); 
   // 
   // 关联 ZwDebugContinue 
   // 
   Addr = (ULONG_PTR) GetProcAddress( 
      GetModuleHandle(_T("ntdll.dll")), 
      "ZwDebugContinue"); 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr, 
      sizeof (BYTE), NULL); 
   Addr++; 
   Rva = GetExportRva(Buffer, "_FakeZwDebugContinue@12"); 
   HookVA = BaseAddress + Rva; 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA, 
      sizeof (ULONG_PTR), NULL); 
   Addr += sizeof (ULONG_PTR); 
   WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2, 
      sizeof (WORD), NULL); 
   // 
   // 全都关联了 
   // 
   CloseHandle(hProcess); 
   MessageBox(_T("The process is now AntiMida"), "AntiMida"); 
   } 
   else 
   { 
      MessageBox(_T("Cannot make the process AntiMida"), "AntiMida"); 
   } 
} 
```

你是不是在那们为什么我关联了 `kernel32` 中起始的几个函数和 `ntdll` 中剩下的函数: 没什么原因。我只是在试试他能不能运行并且关联 `kernel32` 也没什么原因，然后我又懒得再去重写这些代码并关联了 `ntdll` (看见了吧，我真的是很懒的)。实际上最好的方法是直接关联 `ntdll`，因为他是最直接也是最快捷的(你会知道的)。`GetExportRva` 只是我写的一个勇于从输出表获取 `RVA` 的小函数：

```cpp
ULONG_PTR GetExportRva(IN TCHAR *FileName, IN CHAR *FunctionName) 
{ 
   HMODULE hModule = LoadLibrary(FileName); 
   if (hModule == NULL) 
      return 0; 
   ULONG_PTR VA = (ULONG_PTR) GetProcAddress(hModule, 
      FunctionName); 
   FreeLibrary(hModule); 
   return (ULONG_PTR) (VA - (ULONG_PTR) hModule); 
} 
```

Very useful isn't it? Here's the code of the dll i inject:

```cpp
#include <windows.h> 

#include <tchar.h> 

// 
// 驱动部分 
// 

#ifndef NTSTATUS 
typedef LONG NTSTATUS; 
#define NT_SUCCESS(Status) ((NTSTATUS)(Status) >= 0) 
#define STATUS_SUCCESS     ((NTSTATUS)0x00000000L) 
#endif 

BOOL OpenDevice(IN LPCTSTR DriverName, HANDLE * lphDevice); 

#define FILE_DEVICE_ANTIMIDA 0x8000 

#define CODE_RESTORE_INFO   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x800, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_READ_MEM     CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x801, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_QUERY_MEM     CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x802, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_WRITE_MEM     CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x803, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_ALLOC_MEM     CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x804, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_CREATE_THREAD   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x805, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

#define CODE_DBG_CONTINUE   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x806, \\ 
   METHOD_BUFFERED, FILE_ANY_ACCESS) 

// 
// ZwReadVirtualMemory 部分 
//  

typedef struct _Input_ZwReadVirtualMemory 
{ 
   HANDLE  ProcessHandle; 
   PVOID  BaseAddress; 
   PVOID  Buffer; 
   ULONG  BufferLength; 
   PULONG  ReturnLength; 
} Input_ZwReadVirtualMemory; 

extern "C" __declspec(dllexport) 

BOOL WINAPI FakeReadProcessMemory(HANDLE hProcess,   
                        LPCVOID lpBaseAddress, 
                        LPVOID lpBuffer,   DWORD nSize, 
                        LPDWORD lpNumberOfBytesRead) 
{ 
   HANDLE hDevice; 
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE) 
      return FALSE; 
   Input_ZwReadVirtualMemory Input; 
   Input.ProcessHandle = hProcess; 
   Input.BaseAddress = (PVOID) lpBaseAddress; 
   Input.Buffer = (PVOID) lpBuffer; 
   Input.BufferLength = (ULONG) nSize; 
   Input.ReturnLength = (PULONG) lpNumberOfBytesRead; 
   DWORD RetBytes; 
   if (!DeviceIoControl(hDevice, CODE_READ_MEM, 
      &Input, sizeof (Input_ZwReadVirtualMemory), NULL, 0, &RetBytes, NULL)) 
   { 
      CloseHandle(hDevice); 
      return FALSE; 
   } 
   CloseHandle(hDevice); 
   return TRUE; 
} 

// 
// ZwQueryVirtualMemory 部分 
//  

typedef enum _MEMORY_INFORMATION_CLASS { 
   MemoryBasicInformation, 
   MemoryWorkingSetList, 
   MemorySectionName, 
   MemoryBasicVlmInformation 
} MEMORY_INFORMATION_CLASS; 

typedef struct _Input_ZwQueryVirtualMemory 
{ 
   HANDLE ProcessHandle; 
   PVOID  BaseAddress; 
   MEMORY_INFORMATION_CLASS  MemoryInformationClass; 
   ULONG  MemoryInformationLength; 
   PVOID  MemoryInformation; 
   PULONG  ReturnLength; 
} Input_ZwQueryVirtualMemory; 

extern "C" __declspec(dllexport) 

SIZE_T WINAPI FakeVirtualQueryEx(IN HANDLE hProcess, 
                        IN LPCVOID lpAddress, 
                        OUT PMEMORY_BASIC_INFORMATION lpBuffer, 
                        IN SIZE_T dwLength) 
{ 
   Input_ZwQueryVirtualMemory Input; 
   ULONG ReturnLength; 
   Input.ProcessHandle = hProcess; 
   Input.BaseAddress = (PVOID) lpAddress; 
   Input.MemoryInformationClass = MemoryBasicInformation; 
   Input.MemoryInformationLength = (ULONG) dwLength; 
   Input.MemoryInformation = (PVOID) lpBuffer; 
   Input.ReturnLength = &ReturnLength; 
   HANDLE hDevice; 
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE) 
      return FALSE; 
   DWORD RetBytes; 
   if (!DeviceIoControl(hDevice, CODE_QUERY_MEM, 
      &Input, sizeof (Input_ZwQueryVirtualMemory), 
      NULL, 0, &RetBytes, NULL)) 
   { 
      CloseHandle(hDevice); 
      return 0; 
   } 
   CloseHandle(hDevice); 
   return (SIZE_T) ReturnLength; 
} 

// 
// ZwReadVirtualMemory 部分 
//  

typedef struct _Input_ZwWriteVirtualMemory 
{ 
   HANDLE  ProcessHandle; 
   PVOID  BaseAddress; 
   PVOID  Buffer; 
   ULONG  BufferLength; 
   PULONG  ReturnLength; 
} Input_ZwWriteVirtualMemory; 

extern "C" __declspec(dllexport) 

BOOL WINAPI FakeWriteProcessMemory(HANDLE hProcess, LPVOID lpBaseAddress, 
                        LPVOID lpBuffer, DWORD nSize, 
                        LPDWORD lpNumberOfBytesWritten) 
{ 
   HANDLE hDevice; 
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE) 
      return FALSE; 
   // 
   // 修改内存保护 
   // 
   DWORD dwOldProtection; 
   if (!VirtualProtectEx(hProcess, lpBaseAddress, nSize, 
      PAGE_EXECUTE_READWRITE, &dwOldProtection)) 
   { 
      CloseHandle(hDevice); 
      return FALSE; 
   } 
   Input_ZwWriteVirtualMemory Input; 
   Input.ProcessHandle = hProcess; 
   Input.BaseAddress = (PVOID) lpBaseAddress; 
   Input.Buffer = (PVOID) lpBuffer; 
   Input.BufferLength = (ULONG) nSize; 
   Input.ReturnLength = (PULONG) lpNumberOfBytesWritten; 
   DWORD RetBytes; 
   BOOL bRet; 
   if (!DeviceIoControl(hDevice, CODE_WRITE_MEM, 
      &Input, sizeof (Input_ZwWriteVirtualMemory), NULL, 0, &RetBytes, NULL)) 
   { 
      bRet = FALSE; 
   } 
   CloseHandle(hDevice); 
   // 
   // 恢复内存保护 
   // 
   VirtualProtectEx(hProcess, lpBaseAddress, nSize, 
      dwOldProtection, &dwOldProtection); 
   return TRUE; 
} 

// 
// ZwAllocateVirtualMemory 部分 
//  

typedef struct _Input_ZwAllocateVirtualMemory 
{ 
   HANDLE ProcessHandle; 
   PVOID *BaseAddress; 
   ULONG_PTR ZeroBits; 
   PSIZE_T RegionSize; 
   ULONG AllocationType; 
   ULONG Protect; 
} Input_ZwAllocateVirtualMemory; 

extern "C" __declspec(dllexport) 

NTSTATUS NTAPI FakeNtAllocateVirtualMemory(IN HANDLE ProcessHandle, 



IN OUT PVOID *BaseAddress,
IN ULONG_PTR ZeroBits,
IN OUT PSIZE_T RegionSize,
IN ULONG AllocationType,
IN ULONG Protect)
{
   Input_ZwAllocateVirtualMemory Input;
   Input.ProcessHandle = ProcessHandle;
   Input.BaseAddress = BaseAddress;
   Input.ZeroBits = ZeroBits;
   Input.RegionSize = RegionSize;
   Input.AllocationType = AllocationType;
   Input.Protect = Protect;

   HANDLE hDevice;
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE)
      return FALSE;

   DWORD RetBytes;
   if (!DeviceIoControl(hDevice, CODE_ALLOC_MEM,
      &Input, sizeof (Input_ZwAllocateVirtualMemory),
      NULL, 0, &RetBytes, NULL))
   {
      CloseHandle(hDevice);
      // who cares? (invalid parameter)
      return ((NTSTATUS)0xC000000DL);
   }

   CloseHandle(hDevice);
   return STATUS_SUCCESS;
}

// ZwCreateThread 部分

typedef struct _Input_ZwCreateThread
{
   PHANDLE  ThreadHandle;
   ACCESS_MASK  DesiredAccess;
   PVOID  ObjectAttributes;
   HANDLE  ProcessHandle;
   PVOID  ClientId;
   PCONTEXT  ThreadContext;
   PVOID  UserStack;
   BOOLEAN  CreateSuspended;
} Input_ZwCreateThread;

extern "C" __declspec(dllexport)
NTSTATUS NTAPI FakeNtCreateThread(OUT PHANDLE  ThreadHandle,
           IN ACCESS_MASK  DesiredAccess,
           IN PVOID  ObjectAttributes,
           IN HANDLE  ProcessHandle,
           OUT PVOID  ClientId,
           IN PCONTEXT  ThreadContext,
           IN PVOID  UserStack,
           IN BOOLEAN  CreateSuspended)
{
   Input_ZwCreateThread Input;
   Input.ThreadHandle = ThreadHandle;
   Input.DesiredAccess = DesiredAccess;
   Input.ObjectAttributes = ObjectAttributes;
   Input.ProcessHandle = ProcessHandle;
   Input.ClientId = ClientId;
   Input.ThreadContext = ThreadContext;
   Input.UserStack = UserStack;
   Input.CreateSuspended = CreateSuspended;

   HANDLE hDevice;
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE)
      return FALSE;

   DWORD RetBytes;
   if (!DeviceIoControl(hDevice, CODE_CREATE_THREAD,
      &Input, sizeof (Input_ZwCreateThread),
      NULL, 0, &RetBytes, NULL))
   {
      CloseHandle(hDevice);
      // 谁关心呢？(无效的参数)
      return ((NTSTATUS)0xC000000DL);
   }

   CloseHandle(hDevice);
   return STATUS_SUCCESS;
}

// ZwDebugContinue 部分

typedef struct _Input_ZwDebugContinue
{
   PVOID *A;
   PVOID *B;
   PVOID *C;
} Input_ZwDebugContinue;

extern "C" __declspec(dllexport)
NTSTATUS NTAPI FakeZwDebugContinue(PVOID *A, PVOID *B, PVOID *C)
{
   Input_ZwDebugContinue Input;
   Input.A = A;
   Input.B = B;
   Input.C = C;

   HANDLE hDevice;
   if (OpenDevice(_T("antimida"), &hDevice) == FALSE)
      return FALSE;

   DWORD RetBytes;
   if (!DeviceIoControl(hDevice, CODE_DBG_CONTINUE,
      &Input, sizeof (Input_ZwDebugContinue),
      NULL, 0, &RetBytes, NULL))
   {
      CloseHandle(hDevice);
      // 谁关心呢？(无效的参数)
      return ((NTSTATUS)0xC000000DL);
   }

   CloseHandle(hDevice);
   return STATUS_SUCCESS;
}

BOOL OpenDevice(IN LPCTSTR DriverName, HANDLE * lphDevice)
{
   TCHAR    completeDeviceName[64];
   HANDLE   hDevice;
   /*if  ( (GetVersion() & 0xFF) >= 5 ) { wsprintf(completeDeviceName, TEXT("\\\\.\Global\%s"), DriverName); } else {*/
      wsprintf(completeDeviceName, TEXT("\\\\.\\%s"), DriverName);
   //}
   hDevice = CreateFile(completeDeviceName, GENERIC_READ | GENERIC_WRITE, 0,
      NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
   if (hDevice == ((HANDLE)-1))
      return FALSE;

   if (lphDevice)
      *lphDevice = hDevice;
   else
      CloseHandle(hDevice);

   return TRUE;
}

BOOL APIENTRY DllMain(HANDLE hModule, DWORD  Reason, LPVOID lpReserved)
{
   switch (Reason)
   {
   case DLL_PROCESS_ATTACH:
      {
         DisableThreadLibraryCalls((HMODULE) hModule);
         break;
      }
   case DLL_PROCESS_DETACH:
      {
         return TRUE;
      }
   default:
      {
         return FALSE;
      }
   }

   return TRUE;
}

我不认为对这段代码还有什么要解释的。因为我说会在驱动中完成请求的操作，但是怎么做的？仅仅是调用 `ntoskrnl` 中的原始的服务。问题是 `ntoskrnl` 没有在他的导出表中输出 `SDT` 中的服务，那么是什么获得正确的地址的呢？这其实是个小小的骗局。在 `ntoskrnl` 中有一个 `SDT` 的复本。所以我写了这些代码来获取原始的服务的地址：

```
// 感谢 90210 提供这些代码
// http://www.rootkit.com/newsread.php?newsid=176
// 你节省了我的时间

#include "stdafx.h"

#define RVATOVA(base,offset) ((PVOID)((DWORD)(base)+(DWORD)(offset)))

#define ibaseDD *(PDWORD)&ibase

#define STATUS_INFO_LENGTH_MISMATCH     ((NTSTATUS)0xC0000004L)

#define NT_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)

typedef struct {
   WORD    offset:12;
   WORD    type:4;
} IMAGE_FIXUP_ENTRY, *PIMAGE_FIXUP_ENTRY;

typedef LONG NTSTATUS;

NTSTATUS (WINAPI *pNtQuerySystemInformation)(
      DWORD    SystemInformationClass,
      PVOID    SystemInformation,
      ULONG    SystemInformationLength,
      PULONG    ReturnLength
      );

typedef struct _SYSTEM_MODULE_INFORMATION {//信息类 11
   ULONG    Reserved[2];
   PVOID    Base;
   ULONG    Size;
   ULONG    Flags;
   USHORT    Index;
   USHORT    Unknown;
   USHORT    LoadCount;
   USHORT    ModuleNameOffset;
   CHAR    ImageName[256];
}SYSTEM_MODULE_INFORMATION,*PSYSTEM_MODULE_INFORMATION;

typedef struct {
   DWORD    dwNumberOfModules;
   SYSTEM_MODULE_INFORMATION    smi;
} MODULES, *PMODULES;

#define    SystemModuleInformation    11

DWORD GetHeaders(PBYTE ibase,
      PIMAGE_FILE_HEADER *pfh,
      PIMAGE_OPTIONAL_HEADER *poh,
      PIMAGE_SECTION_HEADER *psh)
{
   PIMAGE_DOS_HEADER mzhead=(PIMAGE_DOS_HEADER)ibase;
   if   ((mzhead->e_magic!=IMAGE_DOS_SIGNATURE) ||
      (ibaseDD[mzhead->e_lfanew]!=IMAGE_NT_SIGNATURE))
      return FALSE;

   *pfh=(PIMAGE_FILE_HEADER)&ibase[mzhead->e_lfanew];
   if (((PIMAGE_NT_HEADERS)*pfh)->Signature!=IMAGE_NT_SIGNATURE)
      return FALSE;

   *pfh=(PIMAGE_FILE_HEADER)((PBYTE)*pfh+sizeof(IMAGE_NT_SIGNATURE));
   *poh=(PIMAGE_OPTIONAL_HEADER)((PBYTE)*pfh+sizeof(IMAGE_FILE_HEADER));
   if ((*poh)->Magic!=IMAGE_NT_OPTIONAL_HDR32_MAGIC)
      return FALSE;

   *psh=(PIMAGE_SECTION_HEADER)((PBYTE)*poh+sizeof(IMAGE_OPTIONAL_HEADER));
   return TRUE;
}

DWORD FindKiServiceTable(HMODULE hModule,DWORD dwKSDT)
{
   PIMAGE_FILE_HEADER    pfh;
   PIMAGE_OPTIONAL_HEADER    poh;
   PIMAGE_SECTION_HEADER    psh;
   PIMAGE_BASE_RELOCATION    pbr;
   PIMAGE_FIXUP_ENTRY    pfe;
   DWORD    dwFixups=0,i,dwPointerRva,dwPointsToRva,dwKiServiceTable;
   BOOL    bFirstChunk;
   GetHeaders((PBYTE)hModule,&pfh,&poh,&psh);

   // 循环通过重定位以加速搜索
   if ((poh->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress) &&
      (!((pfh->Characteristics)&IMAGE_FILE_RELOCS_STRIPPED))) {
         pbr=(PIMAGE_BASE_RELOCATION)RVATOVA(poh->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress,hModule);
         bFirstChunk=TRUE;
         // 1st IMAGE_BASE_RELOCATION.VirtualAddress 在 ntoskrnl 中是 0
         while (bFirstChunk || pbr->VirtualAddress) {
            bFirstChunk=FALSE;
            pfe=(PIMAGE_FIXUP_ENTRY)((DWORD)pbr+sizeof(IMAGE_BASE_RELOCATION));
```



```c
for (i=0;i<(pbr->SizeOfBlock-sizeof(IMAGE_BASE_RELOCATION))>>1;i++,pfe++) {
   if (pfe->type==IMAGE_REL_BASED_HIGHLOW) {
      dwFixups++;
      dwPointerRva=pbr->VirtualAddress+pfe->offset;
      // DONT_RESOLVE_DLL_REFERENCES 标识意味着重定位没有被修复
      dwPointsToRva=*(PDWORD)((DWORD)hModule+dwPointerRva)-(DWORD)poh->ImageBase;
      // 这个重定位地址是 KeServiceDescriptorTable.Base?
      if (dwPointsToRva==dwKSDT) {
         // 检查 mov [mem32],imm32. 我们正在尝试找
         // "mov ds:_KeServiceDescriptorTable.Base, offset _KiServiceTable"
         // 从 KiInitSystem.
         if (*(PWORD)((DWORD)hModule+dwPointerRva-2)==0x05c7) {
            // 这里应该要检查在 KiServiceTable 的重定位的
            // 但是不管它了
            dwKiServiceTable=*(PDWORD)((DWORD)hModule+dwPointerRva+4)-poh->ImageBase;
            return dwKiServiceTable;
         }
      }
   } else
      if (pfe->type!=IMAGE_REL_BASED_ABSOLUTE)
      {
         // 理论上不应该会用到这里的
      }
}
*(PDWORD)&pbr+=pbr->SizeOfBlock;
}
}
if (!dwFixups) {
   // 应该不会到这里的 - nt, 2k, xp 的核心有重定位数据
}
return 0;
}

BOOL SDT_GetOriginalFunctions(PRESTORE_INFO pRestoreInfo) {
   HMODULE    hKernel;
   DWORD    dwKSDT;               // rva of KeServiceDescriptorTable
   DWORD    dwKiServiceTable;     // rva of KiServiceTable
   PMODULES    pModules=(PMODULES)&pModules;
   DWORD    dwNeededSize,rc;
   DWORD    dwKernelBase,dwServices=0;
   PCHAR    pKernelName;
   PDWORD    pService;
   PIMAGE_FILE_HEADER    pfh;
   PIMAGE_OPTIONAL_HEADER    poh;
   PIMAGE_SECTION_HEADER    psh;
   pNtQuerySystemInformation = (NTSTATUS (WINAPI *)(
      DWORD, PVOID, ULONG, PULONG)) GetProcAddress(
      GetModuleHandle("ntdll.dll"), "NtQuerySystemInformation");

   // 获取系统模块 - 在这里 ntoskrnl 是第一个
   rc=pNtQuerySystemInformation(SystemModuleInformation,pModules,4,&dwNeededSize);

   if (rc==STATUS_INFO_LENGTH_MISMATCH) {
      pModules = (PMODULES) GlobalAlloc(GPTR,dwNeededSize);
      rc=pNtQuerySystemInformation(SystemModuleInformation,pModules,dwNeededSize,NULL);
   } else {
strange:
      return FALSE;
   }

   if (!NT_SUCCESS(rc)) goto strange;

   // 映像基址
   dwKernelBase=(DWORD)pModules->smi.Base;

   // 文件名 - 他可能会在 boot.ini 中被重命名
   pKernelName=pModules->smi.ModuleNameOffset+pModules->smi.ImageName;

   // 映射 ntoskrnl - 希望他有重定位
   hKernel=LoadLibraryEx(pKernelName,0,DONT_RESOLVE_DLL_REFERENCES);

   if (!hKernel) {
      return FALSE;

   }

   GlobalFree(pModules);

   // 我们自己的重定位操作在这里一无是处 - 因为我们有 GetProcAddress :)

   if (!(dwKSDT=(DWORD)GetProcAddress(hKernel,"KeServiceDescriptorTable"))) {
      return FALSE;
   }

   // 获取 KeServiceDescriptorTable rva
   dwKSDT-=(DWORD)hKernel;

   // 查找 KiServiceTable
   if (!(dwKiServiceTable=FindKiServiceTable(hKernel,dwKSDT))) {
      return FALSE;
   }

   // 我们来转存 KiServiceTable 的内容
   // 可能会失败!!!
   // 这里应该会获得正确的 ServiceLimit，但是这在核心模式中的价值不高
   GetHeaders((PBYTE) hKernel,&pfh,&poh,&psh);

   //
   // 我们的代码
   //
   //
   // 获取 ZwAllocateVirtualMemory
   //
   DWORD nServ = GrabService("ZwAllocateVirtualMemory");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwAllocateVirtualMemory =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwCreateThread
   //
   nServ = GrabService("ZwCreateThread");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwCreateThread =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwDebugContinue
   //
   nServ = GrabService("ZwDebugContinue");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwDebugContinue =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwQueryVirtualMemory
   //
   nServ = GrabService("ZwQueryVirtualMemory");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwQueryVirtualMemory =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwReadVirtualMemory
   //
   nServ = GrabService("ZwReadVirtualMemory");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwReadVirtualMemory =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwTerminateProcess
   //
   nServ = GrabService("ZwTerminateProcess");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwTerminateProcess =
      (*pService-poh->ImageBase + dwKernelBase);

   //
   // 获取 ZwWriteVirtualMemory
   //
   nServ = GrabService("ZwWriteVirtualMemory");
   pService = (PDWORD)((nServ * sizeof (DWORD)) +
      (DWORD) hKernel + dwKiServiceTable);

   pRestoreInfo->ZwWriteVirtualMemory =
      (*pService-poh->ImageBase + dwKernelBase);

   FreeLibrary(hKernel);
}
```

`GrabService` 函数是基于另一个小骗局来获取 `SDT` 项目的。事实上 `SDT` 的项目随着 `windows` 的变化而变化，所以获取他的最好的方法是从 `ntdll` 中读取。看看 `NtReadVirtualMemory`:

```assembly
.text:7C91E2BB           public ZwReadVirtualMemory
.text:7C91E2BB B8 BA 00 00 00   mov     eax, 0BAh ; 这就是 SDT 项目
.text:7C91E2C0 BA 00 03 FE 7F   mov     edx, 7FFE0300h
.text:7C91E2C5 FF 12            call    dword ptr [edx]
.text:7C91E2C7 C2 14 00         retn    14h
```

所以我写了这个函数来获取正确的 `SDT` 项目:

```c
//
// 感谢 gareth 提供这个点子
// http://www.rootkit.com/newsread.php?newsid=248
//
#include "stdafx.h"

DWORD GrabService(IN CHAR *FunctionName)
{
   DWORD Exp = (DWORD) GetProcAddress(
      GetModuleHandle(_T("ntdll.dll")), FunctionName);

   Exp++; // mov opcode
   DWORD *ptr = (DWORD *) Exp;

   return *ptr;
}
```

嗯，我在收集信息并发送给驱动(恢复信息)，我还把 `KeAttachProcess` 原始的代码发送了过去。我并不需要他，但是他在进行恢复时会很有用：

```c
#include "stdafx.h"

ULONG_PTR RvaToOffset(IMAGE_NT_HEADERS *, ULONG_PTR);

BOOL GetExport(BYTE *, ULONG_PTR *, WORD *, CHAR *);

BOOL GetNtoskrnlOriginalBytes(PRESTORE_INFO pRestoreInfo)
{
   TCHAR Buffer[MAX_PATH];
   GetSystemDirectory(Buffer, MAX_PATH);
   _tcscat(Buffer, _T("\\\\ntoskrnl.exe"));

   HANDLE hFile = CreateFile(Buffer, GENERIC_READ, FILE_SHARE_READ,
      NULL, OPEN_EXISTING, 0, NULL);

   if (hFile == INVALID_HANDLE_VALUE)
      return FALSE;

   DWORD FileSize = GetFileSize(hFile, NULL);

   BYTE *ptrNtoskrnl = (BYTE *) VirtualAlloc(NULL, FileSize,
      MEM_COMMIT, PAGE_READWRITE);

   if (ptrNtoskrnl == NULL)
   {
      CloseHandle(hFile);
      return FALSE;
   }

   DWORD BR;
   if (!ReadFile(hFile, ptrNtoskrnl, FileSize, &BR, NULL))
   {
      VirtualFree(ptrNtoskrnl, 0, MEM_RELEASE);
      CloseHandle(hFile);
      return FALSE;
   }

   CloseHandle(hFile);

   IMAGE_DOS_HEADER *ImgDosHdr = (IMAGE_DOS_HEADER *) ptrNtoskrnl;
   IMAGE_NT_HEADERS *ImgNtHdrs = (IMAGE_NT_HEADERS *)
      &ptrNtoskrnl[ImgDosHdr->e_lfanew];

   ULONG_PTR EP_Rva = 0;

   if (!GetExport(ptrNtoskrnl, &EP_Rva, NULL, "KeAttachProcess"))
   {
      VirtualFree(ptrNtoskrnl, 0, MEM_RELEASE);
      return FALSE;
   }

   BYTE *ptr = (BYTE *) (EP_Rva + (ULONG_PTR) ptrNtoskrnl);
```



```c
memcpy(pRestoreInfo->KeAttachProcessPatch, ptr, 5);
VirtualFree(ptrNtoskrnl, 0, MEM_RELEASE);
return TRUE;
}

BOOL CollectInformation(PRESTORE_INFO pRestoreInfo)
{
   if (SDT_GetOriginalFunctions(pRestoreInfo) == FALSE)
      return FALSE;
   return GetNtoskrnlOriginalBytes(pRestoreInfo);
}
```

函数 `GetExport` 是我已有的函数，很有用，从输出表中获取你请求的信息：

```c
BOOL GetExport(BYTE *PE, ULONG_PTR *EP, WORD *Ordinal, CHAR *FuncName)
{
   IMAGE_DOS_HEADER *ET_DOS;
   IMAGE_NT_HEADERS *ET_NT;
   IMAGE_EXPORT_DIRECTORY *Export;
   ULONG_PTR ET, *Functions;
   PSTR *Names;
   WORD *Ordinals;
   CHAR *ApiName;
   __try
   {
      ET_DOS = (IMAGE_DOS_HEADER *)(ULONG_PTR) PE;
      ET_NT = (IMAGE_NT_HEADERS *)(ULONG_PTR) &PE[ET_DOS->e_lfanew];
      ET = ET_NT->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;
      if (ET == NULL)
         return FALSE;
      ET = RvaToOffset(ET_NT, ET);
      if (ET == 0) return FALSE;
      Export = (IMAGE_EXPORT_DIRECTORY *)(ET + (ULONG_PTR) PE);
      Functions = (ULONG_PTR *)(RvaToOffset(ET_NT,
         Export->AddressOfFunctions) + (ULONG_PTR) PE);
      Ordinals = (WORD *)(RvaToOffset(ET_NT,
         Export->AddressOfNameOrdinals) + (ULONG_PTR) PE);
      Names = (PSTR *)(RvaToOffset(ET_NT,
         Export->AddressOfNames) + (ULONG_PTR) PE);
      if (EP != NULL && *EP != 0)
      {
         for (WORD x = 0; x < Export->NumberOfFunctions; x++)
         {
            if (*EP == Functions[x])
            {
               if (Ordinal) *Ordinal = (WORD) (x + Export->Base);
               if (FuncName != NULL)
               {
                  for (WORD i = 0; i < Export->NumberOfNames; i++)
                  {
                     if (Ordinals[i] == x)
                     {
                        ApiName = (char *) RvaToOffset(ET_NT, (ULONG_PTR) Names[i]);
                        if (ApiName != NULL)
                        {
                           ApiName = (char *)((ULONG_PTR) ApiName + (ULONG_PTR) PE);
                           strcpy(FuncName, ApiName);
                           break;
                        }
                     }
                  }
               }
               return TRUE;
            }
         }
         return FALSE;
      }
      else
      {
         if (FuncName == NULL || FuncName[0] == 0)
         {
            if (Ordinal == NULL || *Ordinal == 0)
               return FALSE;
            if (*Ordinal < Export->Base ||
               *Ordinal > (Export->Base + (Export->NumberOfFunctions - 1)))
               return FALSE;
            WORD FuncEntry = (WORD) (*Ordinal - Export->Base);
            if (EP) *EP = Functions[FuncEntry];
            if (FuncName != NULL)
            {
               for (DWORD i = 0; i < Export->NumberOfNames; i++)
               {
                  if (Ordinals[i] == FuncEntry)
                  {
                     ApiName = (char *) RvaToOffset(ET_NT, (ULONG_PTR) Names[i]);
                     if (ApiName != NULL)
                     {
                        ApiName = (char *)((ULONG_PTR) ApiName + (ULONG_PTR) PE);
                        strcpy(FuncName, ApiName);
                        break;
                     }
                  }
               }
            }
            return TRUE;
         }
         else
         {
            for (DWORD x = 0; x < Export->NumberOfFunctions; x++)
            {
               if (Functions[x] == 0)
                  continue;
               for (DWORD i = 0; i < Export->NumberOfNames; i++)
               {
                  if (Ordinals[i] == x)
                  {
                     ApiName = (char *) RvaToOffset(ET_NT, (ULONG_PTR) Names[i]);
                     if (ApiName != NULL)
                     {
                        ApiName = (char *)((ULONG_PTR) ApiName + (ULONG_PTR) PE);
                        if (strcmp(ApiName, FuncName) == 0)
                        {
                           if (Ordinal) *Ordinal = (WORD) (x + Export->Base);
                           if (EP) *EP = Functions[x];
                           return TRUE;
                        }
                     }
                  }
               }
            }
            return FALSE;
         }
      }
   }
   __except (EXCEPTION_EXECUTE_HANDLER)
   {
      return FALSE;
   }
   return FALSE;
}
```

在收集信息后，我把它通过 `DeviceIoControl` 发送给了驱动并接受及处理信息：

```c
NTSTATUS ControlDispatcher(PDEVICE_CONTEXT pDeviceContext, DWORD dwCode,
                           BYTE *pInput, DWORD dwInputSize,
                           BYTE *pOutput, DWORD dwOutputSize, DWORD *pdwInfo)
{
   switch (dwCode)
   {
   case CODE_RESTORE_INFO:
      {
         RtlCopyMemory(&RestoreInfo, pInput, sizeof (RESTORE_INFO));
         pZwReadVirtualMemory = (NTSTATUS (*)(HANDLE,
            PVOID, PVOID, ULONG, PULONG))
            RestoreInfo.ZwReadVirtualMemory;
         pZwQueryVirtualMemory = (NTSTATUS (*)(HANDLE,
            PVOID, MEMORY_INFORMATION_CLASS, PVOID, ULONG,
            PULONG)) RestoreInfo.ZwQueryVirtualMemory;
         pZwWriteVirtualMemory = (NTSTATUS (*)(HANDLE, PVOID,
            PVOID, ULONG, PULONG))
            RestoreInfo.ZwWriteVirtualMemory;
         pZwAllocateVirtualMemory = (NTSTATUS (*)(HANDLE,
            PVOID *, ULONG_PTR, PSIZE_T, ULONG, ULONG))
            RestoreInfo.ZwAllocateVirtualMemory;
         pZwCreateThread = (NTSTATUS (*)(PHANDLE, ACCESS_MASK,
            POBJECT_ATTRIBUTES, HANDLE, PCLIENT_ID,
            PCONTEXT, PUSER_STACK, BOOLEAN))
            RestoreInfo.ZwCreateThread;
         pZwDebugContinue = (NTSTATUS (*)(PVOID *A, PVOID *B, PVOID *C))
            RestoreInfo.ZwDebugContinue;
         RebuildNtoskrnl();
         break;
      }
}
```

`RebuildNtoskrnl` 函数修补了 `KeAttachProcess` 被替换的字节：

```c
//
// 修改 KeAttachProcess
//
VOID RebuildNtoskrnl()
{
   PMDL Mdl;
   DWORD CR0Backup;
   DWORD *ptr;
   BYTE *Patch;
   ptr = (DWORD *)(2 + (DWORD) &KeAttachProcess);
   ptr = (DWORD*) *ptr;
   Patch = (BYTE *) *ptr;
   Mdl = MmCreateMdl(0, (PVOID) Patch, 5);
   if (Mdl == NULL)
      return;
   MmProbeAndLockPages(Mdl, 0, 0);
   if (*Patch == 0xE9)
   {
      __asm
      {
         mov eax, cr0
         mov CR0Backup, eax
         and eax, 0xFFFEFFFF
         mov cr0, eax
      }
      //
      // 修改
      //
      memcpy(Patch, RestoreInfo.KeAttachProcessPatch, 5);
      __asm
      {
         mov eax, CR0Backup
         mov cr0, eax
      }
   }
   MmUnlockPages(Mdl);
   IoFreeMdl(Mdl);
}
```

这就是 `SDT` 的全部了。现在让我们谈谈监视，就是：`regmon` 和 `filemon`。这两个中只要有任何一个启动了，`TheMida` 就不会启动。这里有不少检测他们驱动的方法，恐怕太多了些。下面是他们中的一部分：

1. 查找进程。
2. 查找窗口（我在某些壳中也发现了这个）。
3. 查找注册表中的驱动。
4. 查找内存中的驱动。
5. 查找驱动的对象表。
6. 查找 `SDT`（只对 `regmon`）。

还有一些其他的方式，但是我认为这些应该是最常用的。我尝试解决了 4 和 5（因为我有作这些的源代码）。

```c
#include <wdm.h>

typedef struct _MODULE_ENTRY
{
   LIST_ENTRY le_mod;
   ULONG unknown[4];
   ULONG base;
   ULONG driver_start;
   ULONG unk1;
   UNICODE_STRING driver_Path;
   UNICODE_STRING driver_Name;
   //...
} MODULE_ENTRY, *PMODULE_ENTRY;

//
// 这个结构在 valerino 的代码中丢失了
//

typedef struct _OBJECT_HEADER
{
   ULONG          PointerCount;
   ULONG          HandleCount;
   PVOID          Type;
}
```



```c
UCHAR NameInfoOffset;
UCHAR HandleInfoOffset;
UCHAR QuotaInfoOffset;
UCHAR Flags;
PVOID ObjectCreateInfo;
PVOID SecurityDescriptor;
struct _QUAD Body;
}OBJECT_HEADER, * POBJECT_HEADER;

// valerino 的代码

#define NUMBER_HASH_BUCKETS 37

typedef struct _OBJECT_DIRECTORY_ENTRY {
    struct _OBJECT_DIRECTORY_ENTRY *ChainLink;
    PVOID Object;
} OBJECT_DIRECTORY_ENTRY, *POBJECT_DIRECTORY_ENTRY;

typedef struct _OBJECT_DIRECTORY {
    struct _OBJECT_DIRECTORY_ENTRY *HashBuckets[ NUMBER_HASH_BUCKETS ];
    struct _OBJECT_DIRECTORY_ENTRY **LookupBucket;
    BOOLEAN LookupFound;
    USHORT SymbolicLinkUsageCount;
    struct _DEVICE_MAP *DeviceMap;
} OBJECT_DIRECTORY, *POBJECT_DIRECTORY;

typedef struct _DEVICE_MAP {
    ULONG ReferenceCount;
    POBJECT_DIRECTORY DosDevicesDirectory;
    ULONG DriveMap;
    UCHAR DriveType[ 32 ];
} DEVICE_MAP, *PDEVICE_MAP;

typedef struct _OBJECT_HEADER_NAME_INFO {
    POBJECT_DIRECTORY Directory;
    UNICODE_STRING Name;
    ULONG Reserved;
} OBJECT_HEADER_NAME_INFO, *POBJECT_HEADER_NAME_INFO;

#define OBJECT_TO_OBJECT_HEADER( o ) \
CONTAINING_RECORD( (o), OBJECT_HEADER, Body )

#define OBJECT_HEADER_TO_NAME_INFO( oh ) ((POBJECT_HEADER_NAME_INFO) \
((oh)->NameInfoOffset == 0 ? NULL : ((PCHAR)(oh) - (oh)->NameInfoOffset)))

NTSTATUS ObOpenObjectByName (IN POBJECT_ATTRIBUTES ObjectAttributes,
    IN POBJECT_TYPE ObjectType OPTIONAL, IN KPROCESSOR_MODE AccessMode,
    IN OUT PACCESS_STATE AccessState OPTIONAL, IN ACCESS_MASK DesiredAccess OPTIONAL,
    IN OUT PVOID ParseContext OPTIONAL, OUT PHANDLE Handle);

// 结束

NTSTATUS DriverEntry(IN PDRIVER_OBJECT  DriverObject,
                    IN PUNICODE_STRING RegistryPath);

VOID Unload(IN PDRIVER_OBJECT DriverObject);

VOID StealthInitializeLateMore(VOID);

VOID HideModule(PDRIVER_OBJECT DriverObject);

#pragma code_seg("INIT")

NTSTATUS DriverEntry(IN PDRIVER_OBJECT  DriverObject,
                    IN PUNICODE_STRING RegistryPath)
{
   PDEVICE_OBJECT pDeviceObject = NULL;
   NTSTATUS Status = IoCreateDevice(DriverObject, 0, NULL,
      FILE_DEVICE_UNKNOWN, 0, FALSE, &pDeviceObject);
   if (NT_SUCCESS(Status))
   {
      DriverObject->DriverUnload   = Unload;
      HideModule(DriverObject);
      StealthInitializeLateMore();
   }
   return Status;
}

#pragma code_seg()

VOID Unload(IN PDRIVER_OBJECT DriverObject)
{
   IoDeleteDevice(DriverObject->DeviceObject);
}

// 从 PsLoadedModuleList 中隐藏模块

VOID HideModule(PDRIVER_OBJECT DriverObject)
{
   PMODULE_ENTRY pCurrentModule = NULL;
   // 指向 PsLoadedModuleList 的指针
   pCurrentModule = ((PMODULE_ENTRY)(ULONG_PTR) DriverObject->DriverSection);
   if (pCurrentModule == NULL)
      return;
   // 开始扫描以查找我们的驱动
   while (TRUE)
   {
      pCurrentModule =  (PMODULE_ENTRY) pCurrentModule->le_mod.Flink;
      if (pCurrentModule == NULL)
         return;
      if (pCurrentModule->driver_Name.Length > 3 &&
         pCurrentModule->driver_Name.Buffer)
      {
         if (wcscmp(pCurrentModule->driver_Name.Buffer, L"hide.sys") == 0)
         {
            // 断开与我们的驱动的连接+
            pCurrentModule->le_mod.Blink->Flink = pCurrentModule->le_mod.Flink;
            pCurrentModule->le_mod.Flink->Blink = pCurrentModule->le_mod.Blink;
            break;
         }
      }
   }
}

// 感谢 valerino 提供代码
// rootkit 上的文章
//****************************************************************************
// VOID HideFromObjectDirectory()
//
// 从对象目录中隐藏驱动
//****************************************************************************

VOID StealthInitializeLateMore(VOID)
{
    OBJECT_ATTRIBUTES ObjectAttributes;
    UNICODE_STRING ucName;
    NTSTATUS Status;
    HANDLE hDirectory = NULL;
    POBJECT_DIRECTORY pDirectoryObject = NULL;
    KIRQL OldIrql;
    POBJECT_HEADER ObjectHeader;
    POBJECT_HEADER_NAME_INFO NameInfo;
    POBJECT_DIRECTORY_ENTRY DirectoryEntry;
    POBJECT_DIRECTORY_ENTRY DirectoryEntryNext;
    POBJECT_DIRECTORY_ENTRY DirectoryEntryTop;
    ULONG Bucket = 0;
    UNICODE_STRING ObjectName;
    BOOLEAN found = FALSE;
    // 打开对象目录中的驱动目录
    RtlInitUnicodeString(&ucName,L"\\Driver");
    InitializeObjectAttributes(&ObjectAttributes,&ucName,OBJ_CASE_INSENSITIVE,NULL,NULL);
    Status = ObOpenObjectByName(&ObjectAttributes,NULL,KernelMode,NULL,
       0x80000000,NULL,&hDirectory);
    if (!NT_SUCCESS (Status))
        goto __exit;
    // 获取指针
    Status = ObReferenceObjectByHandle(hDirectory,FILE_ANY_ACCESS,NULL,
       KernelMode,&pDirectoryObject, NULL);
    if (!NT_SUCCESS (Status))
        goto __exit;
    // 我们提高 irql 以保护这个列表不被 kernel APC 访问
    KeRaiseIrql(APC_LEVEL,&OldIrql);
    // 历遍对象目录
    for (Bucket=0; Bucket<NUMBER_HASH_BUCKETS; Bucket++)
    {
        // 完成了吗？
        if (found)
            break;
        DirectoryEntry = pDirectoryObject->HashBuckets[Bucket];
        if (!DirectoryEntry)
            continue;
        // 检查我们是不是在 bucket 的顶端
        ObjectHeader = OBJECT_TO_OBJECT_HEADER( DirectoryEntry->Object );
        NameInfo = OBJECT_HEADER_TO_NAME_INFO( ObjectHeader );
        if (NameInfo != NULL)
        {
            ObjectName = NameInfo->Name;
            // 这里我们将自己的驱动的名称与对象的名称进行比较(例如: ROOTKIT)
            // 这个函数只是我对 wcsstr 的的扩展，不要管它了 .......
            if (wcscmp(ObjectName.Buffer, L"hide") == 0)
            {
                // 到达顶部并获取下一个指针
                DirectoryEntryTop = pDirectoryObject->HashBuckets[Bucket];
                DirectoryEntryNext = DirectoryEntryTop->ChainLink;
                // 替代顶部
                pDirectoryObject->HashBuckets[Bucket] = DirectoryEntryNext;
                DirectoryEntryTop = pDirectoryObject->HashBuckets[Bucket];
                // 历遍链并通过一处替换返回项目
                while (DirectoryEntryNext)
                {
                    DirectoryEntryTop->ChainLink = DirectoryEntryNext->ChainLink;
                    DirectoryEntryTop = DirectoryEntryTop->ChainLink;
                    DirectoryEntryNext = DirectoryEntryNext->ChainLink;
                }
                if (DirectoryEntryTop)
                    DirectoryEntryTop->ChainLink = NULL;
                found = TRUE;
                // 我们可以安全退出了
                break;
            }
        }
        // 如果我们不在 bucket 顶端的话，检查 项目->下一个区域
        // 对每个项目我们都要检查下一个
        DirectoryEntryNext = DirectoryEntry->ChainLink;
        while (DirectoryEntryNext)
        {
            ObjectHeader = OBJECT_TO_OBJECT_HEADER( DirectoryEntryNext->Object );
            NameInfo = OBJECT_HEADER_TO_NAME_INFO( ObjectHeader );
            if (NameInfo != NULL)
            {
                ObjectName = NameInfo->Name;
                if (wcscmp(ObjectName.Buffer, L"hide") == 0)
                {
                    // 找到我们的对象，现在我们必须脱开与他的连接，这次简单些
                    DirectoryEntry->ChainLink = DirectoryEntryNext->ChainLink;
                    found = TRUE;
                    // 退出
                    break;
                }
            }
            // walk the next entry if any
            if (DirectoryEntry)
            {
                DirectoryEntry = DirectoryEntry->ChainLink;
```



```c
        DirectoryEntryNext = DirectoryEntry->ChainLink;

    }

    else

    {

        DirectoryEntryNext = NULL;

    }

}

}

// 调回 irql

KeLowerIrql(OldIrql);

__exit:

    // dereference and cleanup

    if (pDirectoryObject)

        ObDereferenceObject(pDirectoryObject);

    if (hDirectory)

        ZwClose (hDirectory);

    return;

}
```

当然是用真实名称替换藏在代码里的哪些是很重要的。不管怎么样，`TheMida` 至少查找了 `PsLoadedModuleList`，而且还检查了其他什么东西(我想他检查了注册表)。我没有时间来尝试，我很快就会知道的(只要隐藏一些键就行了)。然而我必须要看看他有没有检查 `SDT` 是否被关联了(使用 `regmon` 的代码来关联一个)，而且很明显他没有，他很正常的执行了。我也能执行一个小的监视器来检视注册表和我的工具中的文件(这是可能的)。当然我无法编写一个文件系统的过滤器，只像 `regmon` 一样关联 `SDT` (这更简单一些)。然我想想，现在这些是驱动代码(带有 `SDT` 关联代码，这些可以被扩展成一个监视器之类的东西).

```c
#include <ntddk.h>

#include <common.h>

WCHAR DeviceName[] = L"\\Device\\antimida";

WCHAR SymLinkName[] = L"\\DosDevices\\antimida";

UNICODE_STRING usDeviceName;

UNICODE_STRING usSymbolicLinkName;

typedef struct _DEVICE_CONTEXT

{

   PDRIVER_OBJECT  pDriverObject;

    PDEVICE_OBJECT  pDeviceObject;

}

DEVICE_CONTEXT, *PDEVICE_CONTEXT, **PPDEVICE_CONTEXT;

PDEVICE_OBJECT  g_pDeviceObject  = NULL;

PDEVICE_CONTEXT g_pDeviceContext = NULL;

#define FILE_DEVICE_ANTIMIDA 0x8000

#define CODE_RESTORE_INFO   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x800, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_READ_MEM      CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x801, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_QUERY_MEM      CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x802, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_WRITE_MEM      CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x803, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_ALLOC_MEM      CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x804, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_CREATE_THREAD   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x805, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

#define CODE_DBG_CONTINUE   CTL_CODE(FILE_DEVICE_ANTIMIDA, 0x806, \
   METHOD_BUFFERED, FILE_ANY_ACCESS)

NTSTATUS DriverInitialize(PDRIVER_OBJECT  pDriverObject,
                PUNICODE_STRING pusRegistryPath);

NTSTATUS DriverEntry(PDRIVER_OBJECT  pDriverObject,
            PUNICODE_STRING pusRegistryPath);

#ifdef ALLOC_PRAGMA

#pragma alloc_text (INIT, DriverInitialize)

#pragma alloc_text (INIT, DriverEntry)

#endif

VOID NTAPI KeAttachProcess(IN PEPROCESS);

//

// ZwReadVirtualMemory 部分

//

typedef struct _Input_ZwReadVirtualMemory

{

   HANDLE  ProcessHandle;

   PVOID  BaseAddress;

   PVOID  Buffer;

   ULONG  BufferLength;

   PULONG  ReturnLength;

} Input_ZwReadVirtualMemory;

NTSTATUS (*pZwReadVirtualMemory)(IN HANDLE  ProcessHandle,
             IN PVOID  BaseAddress,
             OUT PVOID  Buffer,
             IN ULONG  BufferLength,
             OUT PULONG  ReturnLength  OPTIONAL);

KMUTEX DumpMutex;

//

// ZwQueryVirtualMemory 部分

//

typedef enum _MEMORY_INFORMATION_CLASS {

   MemoryBasicInformation,

   MemoryWorkingSetList,

   MemorySectionName,

   MemoryBasicVlmInformation

} MEMORY_INFORMATION_CLASS;

NTSTATUS (*pZwQueryVirtualMemory)(IN HANDLE  ProcessHandle,
              IN PVOID  BaseAddress,
              IN MEMORY_INFORMATION_CLASS  MemoryInformationClass,
              OUT PVOID  MemoryInformation,
              IN ULONG  MemoryInformationLength,
              OUT PULONG  ReturnLength  OPTIONAL);

typedef struct _Input_ZwQueryVirtualMemory

{

   HANDLE ProcessHandle;

   PVOID  BaseAddress;

   MEMORY_INFORMATION_CLASS  MemoryInformationClass;

   ULONG  MemoryInformationLength;

   PVOID  MemoryInformation;

   PULONG  ReturnLength;

} Input_ZwQueryVirtualMemory;

//

// ZwWriteVirtualMemory 部分

//

NTSTATUS (*pZwWriteVirtualMemory)(IN HANDLE  ProcessHandle,
             IN PVOID  BaseAddress,
             IN PVOID  Buffer,
             IN ULONG  BufferLength,
             OUT PULONG  ReturnLength  OPTIONAL);

typedef struct _Input_ZwWriteVirtualMemory

{

   HANDLE  ProcessHandle;

   PVOID  BaseAddress;

   PVOID  Buffer;

   ULONG  BufferLength;

   PULONG  ReturnLength;

} Input_ZwWriteVirtualMemory;

//

// ZwAllocateVirtualMemory 部分

//

NTSTATUS (*pZwAllocateVirtualMemory)(IN HANDLE ProcessHandle,
                IN OUT PVOID *BaseAddress,
                IN ULONG_PTR ZeroBits,
                IN OUT PSIZE_T RegionSize,
                IN ULONG AllocationType,
                IN ULONG Protect);

typedef struct _Input_ZwAllocateVirtualMemory

{

   HANDLE ProcessHandle;

   PVOID *BaseAddress;

   ULONG_PTR ZeroBits;

   PSIZE_T RegionSize;

   ULONG AllocationType;

   ULONG Protect;

} Input_ZwAllocateVirtualMemory;

//

// ZwCreateThread 部分

//

typedef struct _USER_STACK {

   PVOID  FixedStackBase;

   PVOID  FixedStackLimit;

   PVOID  ExpandableStackBase;

   PVOID  ExpandableStackLimit;

   PVOID  ExpandableStackBottom;

} USER_STACK, *PUSER_STACK;

NTSTATUS (*pZwCreateThread)(OUT PHANDLE  ThreadHandle,
         IN ACCESS_MASK  DesiredAccess,
         IN POBJECT_ATTRIBUTES  ObjectAttributes,
         IN HANDLE  ProcessHandle,
         OUT PCLIENT_ID  ClientId,
         IN PCONTEXT  ThreadContext,
         IN PUSER_STACK  UserStack,
         IN BOOLEAN  CreateSuspended);

typedef struct _Input_ZwCreateThread

{

   PHANDLE  ThreadHandle;

   ACCESS_MASK  DesiredAccess;

   POBJECT_ATTRIBUTES  ObjectAttributes;

   HANDLE  ProcessHandle;

   PCLIENT_ID  ClientId;

   PCONTEXT  ThreadContext;

   PUSER_STACK  UserStack;

   BOOLEAN  CreateSuspended;

} Input_ZwCreateThread;

//

// ZwDebugContinue 部分

// 我已经懒得去查找正确的声明了

//

NTSTATUS (*pZwDebugContinue)(PVOID *A, PVOID *B, PVOID *C);

typedef struct _Input_ZwDebugContinue

{

   PVOID *A;

   PVOID *B;

   PVOID *C;

} Input_ZwDebugContinue;

//

// 恢复信息结构

//

typedef struct _RESTORE_INFO

{

   DWORD ZwAllocateVirtualMemory;

   DWORD ZwCreateThread;

   DWORD ZwDebugContinue;

   DWORD ZwQueryVirtualMemory;

   DWORD ZwReadVirtualMemory;

   DWORD ZwTerminateProcess;

   DWORD ZwWriteVirtualMemory;

   BYTE KeAttachProcessPatch[5];

} RESTORE_INFO, *PRESTORE_INFO;

RESTORE_INFO RestoreInfo;

//

// SDT 关联部分

//

extern PSERVICE_DESCRIPTOR_TABLE KeServiceDescriptorTable;

PVOID *KeServiceTablePointers;

PMDL KeServiceTableMdl;

BOOLEAN *ServiceIsHooked;

PPVOID MapServiceTable(BOOLEAN **);

VOID UnmapServiceTable(PVOID);

#ifdef ALPHA

#define FUNCTION_PTR(_Function) (*(PULONG) _Function) & 0x0000FFFF

#else

#define FUNCTION_PTR(_Function) *(PULONG)((PUCHAR) _Function + 1)

#endif

#define HOOK_SYSCALL(_Function, _Hook, _Orig)                        \
   if (!ServiceIsHooked[FUNCTION_PTR(_Function)]) {                 \
   _Orig = (PVOID) InterlockedExchange((PLONG)                      \
   &KeServiceTablePointers[FUNCTION_PTR(_Function)], (LONG) _Hook ); \
   ServiceIsHooked[ FUNCTION_PTR(_Function) ] = TRUE; }

#define UNHOOK_SYSCALL(_Function, _Hook, _Orig)                      \
   if (ServiceIsHooked[FUNCTION_PTR(_Function)] &&                   \
   KeServiceTablePointers[FUNCTION_PTR(_Function) ] == (PVOID) _Hook ) { \
```



InterlockedExchange((PLONG) &KeServiceTablePointers[  
FUNCTION_PTR(_Function)], (LONG) _Orig );  
ServiceIsHooked[FUNCTION_PTR(_Function)] = FALSE; }

```c
NTSTATUS (*RealZwCreateFile)(OUT PHANDLE FileHandle,  
             IN ACCESS_MASK DesiredAccess,  
             IN POBJECT_ATTRIBUTES ObjectAttributes,  
             OUT PIO_STATUS_BLOCK IoStatusBlock,  
             IN PLARGE_INTEGER AllocationSize OPTIONAL,  
             IN ULONG FileAttributes,  
             IN ULONG ShareAccess,  
             IN ULONG CreateDisposition,  
             IN ULONG CreateOptions,  
             IN PVOID EaBuffer OPTIONAL,  
             IN ULONG EaLength  
             );  

NTSTATUS (*RealZwOpenKey)(OUT PHANDLE, IN ACCESS_MASK, IN POBJECT_ATTRIBUTES);  

NTSTATUS HookedZwCreateFile(OUT PHANDLE FileHandle,  
             IN ACCESS_MASK DesiredAccess,  
             IN POBJECT_ATTRIBUTES ObjectAttributes,  
             OUT PIO_STATUS_BLOCK IoStatusBlock,  
             IN PLARGE_INTEGER AllocationSize OPTIONAL,  
             IN ULONG FileAttributes,  
             IN ULONG ShareAccess,  
             IN ULONG CreateDisposition,  
             IN ULONG CreateOptions,  
             IN PVOID EaBuffer OPTIONAL,  
             IN ULONG EaLength)  
{  
   DbgPrint("%ws\n", ObjectAttributes->ObjectName->Buffer);  
   return RealZwCreateFile(FileHandle, DesiredAccess, ObjectAttributes,  
      IoStatusBlock, AllocationSize, FileAttributes, ShareAccess,  
      CreateDisposition, CreateOptions, EaBuffer, EaLength);  
}  

NTSTATUS HookedZwOpenKey(OUT PHANDLE KeyHandle, IN ACCESS_MASK DesiredAccess,  
             IN POBJECT_ATTRIBUTES ObjectAttributes)  
{  
   return RealZwOpenKey(KeyHandle, DesiredAccess, ObjectAttributes);  
}  

PPVOID MapServiceTable(BOOLEAN **ServiceIsHooked)  
{  
   PVOID Mem;  
   Mem = ExAllocatePoolWithTag(0,  
      KeServiceDescriptorTable->ntoskrnl.ServiceLimit,  
      0x206B6444);  
   if (Mem == NULL)  
      return NULL;  
   *ServiceIsHooked = (BOOLEAN *) Mem;  
   memset(Mem, 0, KeServiceDescriptorTable->ntoskrnl.  
      ServiceLimit);  
   KeServiceTableMdl = MmCreateMdl(NULL,  
      KeServiceDescriptorTable->ntoskrnl.ServiceTable,  
      (KeServiceDescriptorTable->ntoskrnl.ServiceLimit *  
      sizeof (POINTER)));  
   if (KeServiceTableMdl == NULL)  
      return NULL;  
   MmBuildMdlForNonPagedPool(KeServiceTableMdl);  
   return (PPVOID) MmMapLockedPages(KeServiceTableMdl, 0);  
}  

VOID UnmapServiceTable(PVOID KeServiceTablePointers)  
{  
   if (KeServiceTableMdl == NULL)  
      return;  
   MmUnmapLockedPages(KeServiceTablePointers,  
      KeServiceTableMdl);  
   ExFreePool(KeServiceTableMdl);  
}  

//  
// 修改 KeAttachProcess  
//  
VOID RebuildNtoskrnl()  
{  
   PMDL Mdl;  
   DWORD CR0Backup;  
   DWORD *ptr;  
   BYTE *Patch;  
   ptr = (DWORD *)(2 + (DWORD) &KeAttachProcess);  
   ptr = (DWORD*) *ptr;  
   Patch = (BYTE *) *ptr;  
   Mdl = MmCreateMdl(0, (PVOID) Patch, 5);  
   if (Mdl == NULL)  
      return;  
   MmProbeAndLockPages(Mdl, 0, 0);  
   if (*Patch == 0xE9)  
   {  
      __asm  
      {  
         mov eax, cr0  
         mov CR0Backup, eax  
         and eax, 0xFFFEFFFF  
         mov cr0, eax  
      }  
      //  
      // 修改  
      //  
      memcpy(Patch, RestoreInfo.KeAttachProcessPatch, 5);  
      __asm  
      {  
         mov eax, CR0Backup  
         mov cr0, eax  
      }  
   }  
   MmUnlockPages(Mdl);  
   IoFreeMdl(Mdl);  
}  

NTSTATUS ControlDispatcher(PDEVICE_CONTEXT pDeviceContext, DWORD dwCode,  
             BYTE *pInput, DWORD dwInputSize,  
             BYTE *pOutput, DWORD dwOutputSize, DWORD *pdwInfo)  
{  
   switch (dwCode)  
   {  
   case CODE_RESTORE_INFO:  
      {  
         RtlCopyMemory(&RestoreInfo, pInput, sizeof (RESTORE_INFO));  
         pZwReadVirtualMemory = (NTSTATUS (*)(HANDLE,  
            PVOID, PVOID, ULONG, PULONG))  
            RestoreInfo.ZwReadVirtualMemory;  
         pZwQueryVirtualMemory = (NTSTATUS (*)(HANDLE,  
            PVOID, MEMORY_INFORMATION_CLASS, PVOID, ULONG,  
            PULONG) RestoreInfo.ZwQueryVirtualMemory;  
         pZwWriteVirtualMemory = (NTSTATUS (*)(HANDLE, PVOID,  
            PVOID, ULONG, PULONG))  
            RestoreInfo.ZwWriteVirtualMemory;  
         pZwAllocateVirtualMemory = (NTSTATUS (*)(HANDLE,  
            PVOID *, ULONG_PTR, PSIZE_T, ULONG, ULONG))  
            RestoreInfo.ZwAllocateVirtualMemory;  
         pZwCreateThread = (NTSTATUS (*)(PHANDLE, ACCESS_MASK,  
            POBJECT_ATTRIBUTES, HANDLE, PCLIENT_ID,  
            PCONTEXT, PUSER_STACK, BOOLEAN))  
            RestoreInfo.ZwCreateThread;  
         pZwDebugContinue = (NTSTATUS (*)(PVOID *A, PVOID *B, PVOID *C))  
            RestoreInfo.ZwDebugContinue;  
         RebuildNtoskrnl();  
         break;  
      }  
   case CODE_READ_MEM:  
      {  
         Input_ZwReadVirtualMemory Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwReadVirtualMemory));  
         return pZwReadVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.Buffer, Input.BufferLength, Input.ReturnLength);  
      }  
   case CODE_QUERY_MEM:  
      {  
         Input_ZwQueryVirtualMemory Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwQueryVirtualMemory));  
         return pZwQueryVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.MemoryInformationClass, Input.MemoryInformation,  
            Input.MemoryInformationLength, Input.ReturnLength);  
      }  
   case CODE_WRITE_MEM:  
      {  
         Input_ZwWriteVirtualMemory Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwWriteVirtualMemory));  
         return pZwWriteVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.Buffer, Input.BufferLength, Input.ReturnLength);  
      }  
   case CODE_ALLOC_MEM:  
      {  
         Input_ZwAllocateVirtualMemory Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwAllocateVirtualMemory));  
         return pZwAllocateVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.ZeroBits, Input.RegionSize, Input.AllocationType, Input.Protect);  
      }  
   case CODE_CREATE_THREAD:  
      {  
         Input_ZwCreateThread Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwCreateThread));  
         return pZwCreateThread(Input.ThreadHandle, Input.DesiredAccess,  
            Input.ObjectAttributes, Input.ProcessHandle, Input.ClientId,   
            Input.ThreadContext, Input.UserStack, Input.CreateSuspended);  
      }  
   case CODE_DBG_CONTINUE:  
      {  
         Input_ZwDebugContinue Input;  
         RtlCopyMemory(&Input, pInput, sizeof (Input_ZwDebugContinue));  
         return pZwDebugContinue(Input.A, Input.B, Input.C);  
      }  
   default:  
      return STATUS_INVALID_PARAMETER;  
   }  
   return STATUS_SUCCESS;  
}  

NTSTATUS DeviceDispatcher(PDEVICE_CONTEXT pDeviceContext, PIRP pIrp)  
{  
    PIO_STACK_LOCATION pisl;  
    DWORD dwInfo = 0;  
    NTSTATUS ns = STATUS_NOT_IMPLEMENTED;  
    pisl = IoGetCurrentIrpStackLocation(pIrp);  
    switch (pisl->MajorFunction)  
   {  
   case IRP_MJ_CREATE:  
   case IRP_MJ_CLEANUP:  
   case IRP_MJ_CLOSE:  
      {  
         ns = STATUS_SUCCESS;  
            break;  
      }  
   case IRP_MJ_DEVICE_CONTROL:  
      {  
         MUTEX_ACQUIRE(DumpMutex);  
```



```c
    ns = ControlDispatcher(pDeviceContext,  
                          pisl->Parameters.DeviceIoControl.IoControlCode,  
                          (BYTE *) pIrp->AssociatedIrp.SystemBuffer,  
                          pisl->Parameters.DeviceIoControl.InputBufferLength,  
                          (BYTE *) pIrp->AssociatedIrp.SystemBuffer,  
                          pisl->Parameters.DeviceIoControl.OutputBufferLength,  
                          &dwInfo);  
    MUTEX_RELEASE(DumpMutex);  
    break;  
   }  
  }  
  pIrp->IoStatus.Status = ns;  
  pIrp->IoStatus.Information = dwInfo;  
  IoCompleteRequest (pIrp, IO_NO_INCREMENT);  
  return ns;  
}  

NTSTATUS DriverDispatcher(PDEVICE_OBJECT pDeviceObject, PIRP pIrp)  
{  
   return (pDeviceObject == g_pDeviceObject ?  
      DeviceDispatcher(g_pDeviceContext, pIrp)  
      : STATUS_INVALID_PARAMETER_1);  
}  

VOID DriverUnload(PDRIVER_OBJECT pDriverObject)  
{  
   UNHOOK_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
   UNHOOK_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
   UnmapServiceTable(KeServiceTablePointers);  
    IoDeleteSymbolicLink(&usSymbolicLinkName);  
    IoDeleteDevice(pDriverObject->DeviceObject);  
}  

NTSTATUS DriverInitialize(PDRIVER_OBJECT pDriverObject,  
                PUNICODE_STRING pusRegistryPath)  
{  
    PDEVICE_OBJECT pDeviceObject = NULL;  
    NTSTATUS ns = STATUS_DEVICE_CONFIGURATION_ERROR;  
    RtlInitUnicodeString(&usDeviceName, DeviceName);  
    RtlInitUnicodeString(&usSymbolicLinkName, SymLinkName);  
    if ((ns = IoCreateDevice(pDriverObject, sizeof (DEVICE_CONTEXT),  
      &usDeviceName, FILE_DEVICE_ANTIMIDA, 0, FALSE,  
      &pDeviceObject)) == STATUS_SUCCESS)  
   {  
        if ((ns = IoCreateSymbolicLink(&usSymbolicLinkName,  
         &usDeviceName)) == STATUS_SUCCESS)  
      {  
            g_pDeviceObject  = pDeviceObject;  
            g_pDeviceContext = pDeviceObject->DeviceExtension;  
            g_pDeviceContext->pDriverObject = pDriverObject;  
            g_pDeviceContext->pDeviceObject = pDeviceObject;  
      }  
      else  
      {  
         IoDeleteDevice(pDeviceObject);  
      }  
   }  
   return ns;  
}  

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject,  
            PUNICODE_STRING pusRegistryPath)  
{  
   PDRIVER_DISPATCH *ppdd;  
    NTSTATUS ns = STATUS_DEVICE_CONFIGURATION_ERROR;  
    if ((ns = DriverInitialize(pDriverObject, pusRegistryPath)) == STATUS_SUCCESS)  
   {  
        ppdd = pDriverObject->MajorFunction;  
        ppdd[IRP_MJ_CREATE               ] =  
        ppdd[IRP_MJ_CREATE_NAMED_PIPE    ] =  
        ppdd[IRP_MJ_CLOSE                ] =  
        ppdd[IRP_MJ_READ                 ] =  
        ppdd[IRP_MJ_WRITE                ] =  
        ppdd[IRP_MJ_QUERY_INFORMATION    ] =  
        ppdd[IRP_MJ_SET_INFORMATION      ] =  
        ppdd[IRP_MJ_QUERY_EA             ] =  
        ppdd[IRP_MJ_SET_EA               ] =  
        ppdd[IRP_MJ_FLUSH_BUFFERS        ] =  
        ppdd[IRP_MJ_QUERY_VOLUME_INFORMATION] =  
        ppdd[IRP_MJ_SET_VOLUME_INFORMATION ] =  
        ppdd[IRP_MJ_DIRECTORY_CONTROL    ] =  
        ppdd[IRP_MJ_FILE_SYSTEM_CONTROL  ] =  
        ppdd[IRP_MJ_DEVICE_CONTROL       ] =  
        ppdd[IRP_MJ_INTERNAL_DEVICE_CONTROL ] =  
        ppdd[IRP_MJ_SHUTDOWN             ] =  
        ppdd[IRP_MJ_LOCK_CONTROL         ] =  
        ppdd[IRP_MJ_CLEANUP             ] =  
        ppdd[IRP_MJ_CREATE_MAILSLOT     ] =  
        ppdd[IRP_MJ_QUERY_SECURITY      ] =  
        ppdd[IRP_MJ_SET_SECURITY        ] =  
        ppdd[IRP_MJ_POWER               ] =  
        ppdd[IRP_MJ_SYSTEM_CONTROL      ] =  
        ppdd[IRP_MJ_DEVICE_CHANGE       ] =  
        ppdd[IRP_MJ_QUERY_QUOTA         ] =  
        ppdd[IRP_MJ_SET_QUOTA           ] =  
        ppdd[IRP_MJ_PNP                 ] = DriverDispatcher;  
        pDriverObject->DriverUnload      = DriverUnload;  
      MUTEX_INIT(DumpMutex);  
      KeServiceTablePointers = MapServiceTable(&ServiceIsHooked);  
      HOOK_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
      HOOK_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
   }  
   return ns;  
}
```

好了，我现在很累了。没有什么要解释的了，如果你有疑惑的话，google 在那里等着你。这项工程的继续取决于我得到的信息。我对逆向 `TheMida` 没有兴趣。

译者：我现在同样也是累得不行了。。。好长的程序代码，不过最后还是发现有词还是不会译。郁闷。。。

`4dd466d0e1e0cc123afbe504da2462ca`
