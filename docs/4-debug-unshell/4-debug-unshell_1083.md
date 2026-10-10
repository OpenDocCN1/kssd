

# 对付 themida 的工具及文章

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-742.htm`](https://www.pediy.com/kssd/pediy07/pediy7-742.htm)

接着 声声慢 的继续。。。续：[`bbs.pediy.com/showthread.php...;threadid=12834`](http://bbs.pediy.com/showthread.php?s=&threadid=12834)  
  
  
写这篇文章是为了好玩,不要太认真的对待它。 前段时间有个朋友(Littleluk)问我是否有兴趣帮助他写脱 themida (xprotector 商业后继者)的壳的程序。 我同意了。 然而，我却对它有另外的一个想法.因为坦诚的来讲,我不想把我的时间浪费在无用的事情上,而 themida 就是其中的一件.对那些还不知道我在讲什么的人: themida 是一个壳,而且它是在 kernel 形式下运行的壳.这意味着:要是您要用调试器去调试 themida,它将要崩溃您的系统.很不专业,我知道.这是我把 themida 看作逆向人士的玩具的主要原因.直接的说,themida 将永远不会被大量的公司使用,没有人会用这样一个给微软的 kernel,SDT,IDT 注补丁,时常令机器重启动等等的壳来保护一个很好的软件.这意味这它不会被广泛使用,除了对它本身的脱壳乐趣,不会有太大的意义.我对使用无意义的东西没有兴趣,但既然 Littleluk 提出,我就编写出一些东西(使用一些我以前编的旧代码).因此,这篇文章是围绕着我所写的一个工具.这不是脱 themida 壳的工具(那是不可能的:Littleluk 还没有成功的逆向它).这个工具能使您电脑上已有的一些工具来防卫 themida.  
  
这里有一些警告：  
  
1 - 这篇文章没有牵扯到核心部分的操作，你不需要有驱动编程的知识，这对那些驱动的编写者来说有些荒谬(而且我会说 '因为在做逆向的人中你可以很容易得找到对核心编程一无所知的人)。  
  
2 - 在这篇文章中尼了解到的东西会十几年前的东西(就像 themida 使用的那些陷阱一样)。  
  
3 - 我从来就没碰过 themida，我能写出这个工具并不是因为我的经验，而且结果是从 Littleluk 那里的来的。  
  
4 - 我不会去解释什么是 SDT、IDT 以及其他内在相关的概念的含义。如果你不知道这些涵义的话，可以去网上找找。我也没时间去解释那些不太重要的，事实上这也是我现在放出这篇文章的原因之一。我刚好有几天的时间然后又得回去工作(现实中的工作)。   
  
5 - 我会解释我所知道的 themida，或更准确地说你需要理解我写的那个工具。我不打算逆向 themida，所以对于很多事情我并不是很关心。  
  
6 - 我不知道会不会有什么其它的 AntiMida，这完全取决于逆向工作者提供给我的信息量以及我的空余时间。  
  
7 - 这个方法在你使用 PAE 扩展的时候不起作用(我已经懒得再去添加一些代码了)。  
  
8 - 这个工具的工作方式也不是绝对的，还是有其他的方法的。  
  
这次用于测试的牺牲品不是什么其它的东西而是 themida 自身(我是指试用版)。你可以从他的官方网站上下载他(当前版本是 1.0.0.5)。  
  
AntiMida 不是一个计划过的工具，只是每次我遇到问题时而编写的解决方案。现在 AntiMida 可以让你：  
  
1 - 使用普通用户模式的应用程序去转存 themida.  
2 - 使用 imprec, winhex (察看程序的内存), 等等。  
3 - 监视文件及注册表的访问。  
  
但是一次最多做一个。第一步是转存 themida。怎么做？首先我们需要知道 themida 用于保护它自己防止被转存的方法。实际上首先想到去用的是 KeAttachProcess，我们转存了 ntoskrnl (有点痛苦) 然后看到 keattachprocess 被改成了一个 jmp 到 themida 入口。所以要使用 keattachprocess 的话，就必须先修改 ntoskrnl(我在下面贴出了代码)。下面是我使用 KeAttachProcess 写的入口：  
  
    case CODE\_READ\_MEM:  
      {  
         MemReadInfo mri;  
         ULONG\_PTR ptr, addr;  
         BYTE \*Buffer;  
         UINT x, y;  
  
         RtlCopyMemory(&mri, pInput, sizeof (MemReadInfo));  
  
         if (PsLookupProcessByProcessId(mri.PID, &ptr) != STATUS\_SUCCESS)  
            return STATUS\_INVALID\_PARAMETER;  
  
         Buffer = (BYTE \*)  ExAllocatePool(NonPagedPool, dwOutputSize);  
  
         if (Buffer == NULL)  
            return STATUS\_INVALID\_PARAMETER;  
  
         KeAttachProcess(ptr);  
  
         if (dwOutputSize <= 0x1000)  
         {  
            x = 1;  
         }  
         else  
         {  
            x = dwOutputSize / 0x1000;  
  
            if (dwOutputSize % 0x1000 != 0)  
               x++;  
         }  
  
         for (y = 0; y < x; y++)  
         {  
            addr = y \* 0x1000 + (ULONG\_PTR) mri.Address;  
  
            if (MmIsAddressValid((PVOID) addr) == FALSE)  
            {  
               ExFreePool(Buffer);  
               return STATUS\_INVALID\_PARAMETER;  
            }  
         }  
  
         RtlCopyMemory(Buffer, mri.Address, dwOutputSize);  
  
         KeDetachProcess();  
  
         RtlCopyMemory(pOutput, Buffer, dwOutputSize);  
  
         ExFreePool(Buffer);  
  
         \*pdwInfo = dwOutputSize;  
  
         break;  
      }  
  
我贴这段代码只是为了信息，因为这段代码现在已经不在 AntiMida 中了。很显然 themida 正在关联 SDT，然后在使用 sdtrestore 扫描之后我得到了一个被关联的服务的列表：  
  
ZwAllocateVirtualMemory 11 --\[未知关联于 F5938BC4\]--   
ZwCreateThread 35 --\[未知关联于 F5938CBE\]--   
ZwDebugContinue 3A --\[未知关联于 F59391A0\]--   
ZwQueryVirtualMemory B2 --\[未知关联于 F5938ACA\]--   
ZwReadVirtualMemory BA --\[未知关联于 F5938014\]--   
ZwTerminateProcess 101 --\[未知关联于 F59389D0\]--   
ZwWriteVirtualMemory 115 --\[未知关联于 F5938000\]--   
  
很好 ZwAllocateVirtualMemory, ZwQueryVirtualMemory, ZwReadVirtualMemory, ZwWriteVirtualMemory, ZwCreateThread 是有用的，因为我们需要这些函数来转存 themida。ZwDebugContinue 是用来避开调试的，虽然 themida 也对 IDT 进行了一番胡搞(所以这可能就是反调试的手段，当然可能太简单了些)。ZwTerminateProcess 不是重要的，我猜他被关联只是为了能知道被保护的进程退出了。不过如果你尝试恢复这些服务中的任何一个的话，themida 就会整垮你的系统。  
  
我想到的注意使制作一个工具室的其他已经存在的工具可以使用。所以我构建了一个列有当前运行的进程的列表，并提供选择其中一个使其抗 themida 的可能，这意味着对 themida 的免疫。一个简洁的界面还是要花上不少代码的。  
  
   
当那个小按钮被按下时，一个 dll 会被注入选中的进程的地址空间，被 themida 关联以保护它自己的函数会被重定向到这个注入的 dll 上，每当调用一个被关联的函数时，这个 dll 会调用驱动来完成操作。那么我的驱动是怎么完成这些操作的呢？稍后你就会看到。现在先看看注入过程：  
BOOL InjectModule(IN ULONG\_PTR ProcessID, IN TCHAR \*ModuleName,   
              OUT ULONG\_PTR \*BaseAddress OPTIONAL)  
{  
   HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE,  
      ProcessID);  
  
   if (hProcess == NULL)  
      return FALSE;  
  
   HANDLE hFile = CreateFile(ModuleName, GENERIC\_READ, FILE\_SHARE\_READ,  
      NULL, OPEN\_EXISTING, 0, NULL);  
  
   if (hFile == INVALID\_HANDLE\_VALUE)  
   {  
      CloseHandle(hProcess);  
      return FALSE;  
   }  
  
   UINT PE\_Size = GetFileSize(hFile, NULL);  
   BYTE \*PE\_Buffer = (BYTE \*) VirtualAlloc(NULL, PE\_Size,   
      MEM\_COMMIT, PAGE\_READWRITE);  
  
   if (PE\_Buffer == NULL)  
   {  
      CloseHandle(hProcess);  
      CloseHandle(hFile);  
      return FALSE;  
   }  
  
   DWORD BR;  
  
   if (!ReadFile(hFile, PE\_Buffer, PE\_Size, &BR, NULL))  
   {  
      VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
      CloseHandle(hProcess);  
      CloseHandle(hFile);  
      return FALSE;  
   }  
  
   CloseHandle(hFile);  
  
   BYTE \*PE\_Image;  
  
   IMAGE\_DOS\_HEADER \*ImgDosHdr;  
   IMAGE\_NT\_HEADERS \*ImgNtHdrs;  
  
   ULONG\_PTR ImgBase, Delta, Reloc\_Offset;  
   IMAGE\_BASE\_RELOCATION \*ImgBaseReloc;  
   WORD \*wData;  
   UINT i, nItems;  
   ULONG\_PTR Offset;  
   DWORD Type;  
   ULONG\_PTR \*Block, BlockOffs;  
  
   ULONG\_PTR IT\_Offset;  
   IMAGE\_IMPORT\_DESCRIPTOR \*ImgImpDescr;  
   UINT x = 0, y = 0;  
   CHAR \*DllName;  
   ULONG\_PTR \*Thunks, \*FThunks;  
   IMAGE\_IMPORT\_BY\_NAME \*ImgImpName;  
  
   \_try  
   {  
      ImgDosHdr = (IMAGE\_DOS\_HEADER \*) PE\_Buffer;  
  
      ImgNtHdrs = (IMAGE\_NT\_HEADERS \*) (ImgDosHdr->e\_lfanew +  
         (ULONG\_PTR) PE\_Buffer);  
  
      if (ImgDosHdr->e\_magic != IMAGE\_DOS\_SIGNATURE ||  
         ImgNtHdrs->Signature != IMAGE\_NT\_SIGNATURE)  
      {  
         VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
         CloseHandle(hProcess);  
         return FALSE;  
      }  
  
      ImgBase = (ULONG\_PTR) VirtualAllocEx(hProcess,   
         (PVOID) 0, //ImgNtHdrs->OptionalHeader.ImageBase,  
         ImgNtHdrs->OptionalHeader.SizeOfImage, MEM\_COMMIT,   
         PAGE\_EXECUTE\_READWRITE);  
  
      if (ImgBase == NULL)  
      {  
         VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
         CloseHandle(hProcess);  
         return FALSE;  
      }  
  
      //  
      // 文件的映像基址 == 内存映像基址吗？  
      // 不是则重定位  
      //  
  
      if (ImgNtHdrs->OptionalHeader.ImageBase != ImgBase)  
      {  
         Delta = ImgBase - ImgNtHdrs->OptionalHeader.ImageBase;  
  
         if (!ImgNtHdrs->OptionalHeader.DataDirectory  
            \[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress ||  
            !(Reloc\_Offset = RvaToOffset(ImgNtHdrs,   
            ImgNtHdrs->OptionalHeader.DataDirectory  
            \[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress)))  
         {  
            VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);  
            VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
            CloseHandle(hProcess);  
            return FALSE;  
         }  
  
         ImgBaseReloc = (IMAGE\_BASE\_RELOCATION \*)   
            (Reloc\_Offset + (ULONG\_PTR) PE\_Buffer);  
  
         do  
         {  
            if (!ImgBaseReloc->SizeOfBlock)  
               break;  
  
            nItems = (ImgBaseReloc->SizeOfBlock -   
               IMAGE\_SIZEOF\_BASE\_RELOCATION) / sizeof (WORD);  
  
            wData = (WORD \*)(IMAGE\_SIZEOF\_BASE\_RELOCATION +   
               (ULONG\_PTR) ImgBaseReloc);  
  
            for (i = 0; i < nItems; i++)  
            {  
               Offset = (\*wData & 0xFFF) + ImgBaseReloc->VirtualAddress;  
  
               Type = \*wData >> 12;  
  
               if (Type != IMAGE\_REL\_BASED\_ABSOLUTE)  
               {  
                  BlockOffs = RvaToOffset(ImgNtHdrs, Offset);  
  
                  if (BlockOffs == NULL)  
                  {  
                     VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);  
                     VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
                     CloseHandle(hProcess);  
                     return FALSE;  
                  }  
  
                  Block = (DWORD \*)(BlockOffs + (ULONG\_PTR) PE\_Buffer);  
  
                  \*Block += Delta;  
               }  
  
               wData++;  
            }   
  
            ImgBaseReloc = (PIMAGE\_BASE\_RELOCATION) wData;  
  
         } while (\*(DWORD \*) wData);  
      }  
  
      //  
      // 填充输入地址表  
      //  
  
      if (ImgNtHdrs->OptionalHeader.DataDirectory  
         \[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress)  
      {  
         IT\_Offset = RvaToOffset(ImgNtHdrs,   
            ImgNtHdrs->OptionalHeader.DataDirectory  
            \[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress);  
  
         ImgImpDescr = (IMAGE\_IMPORT\_DESCRIPTOR \*) (IT\_Offset +  
            (ULONG\_PTR) PE\_Buffer);  
  
  
         // 对每一个描述符  
         while (ImgImpDescr\[x\].FirstThunk != 0)  
         {  
            DllName = (CHAR \*) (RvaToOffset(ImgNtHdrs,   
               ImgImpDescr\[x\].Name) + (ULONG\_PTR) PE\_Buffer);  
  
            Thunks = (ULONG\_PTR \*) (RvaToOffset(ImgNtHdrs,  
               ImgImpDescr\[x\].OriginalFirstThunk != 0 ?  
               ImgImpDescr\[x\].OriginalFirstThunk :   
            ImgImpDescr\[x\].FirstThunk) + (ULONG\_PTR) PE\_Buffer);  
  
            FThunks = (ULONG\_PTR \*) (RvaToOffset(ImgNtHdrs,  
               ImgImpDescr\[x\].FirstThunk) + (ULONG\_PTR) PE\_Buffer);  
  
            y = 0;  
  
            //  
            // 模块中的每一个导入的函数  
            //  
  
            while (Thunks\[y\] != 0)  
            {  
               //  
               // 原始导入的吗？  
               //  
  
               if (Thunks\[y\] & IMAGE\_ORDINAL\_FLAG)  
               {  
                  FThunks\[y\] = (ULONG\_PTR) GetProcAddress(  
                     GetModuleHandle(DllName),  
                     (LPCSTR) (Thunks\[y\] - IMAGE\_ORDINAL\_FLAG));  
  
                  y++;  
  
                  continue;  
               }  
  
               ImgImpName = (IMAGE\_IMPORT\_BY\_NAME \*) (RvaToOffset(  
                  ImgNtHdrs, Thunks\[y\]) + (ULONG\_PTR) PE\_Buffer);  
  
               FThunks\[y\] = (ULONG\_PTR) GetProcAddress(GetModuleHandle(DllName),   
                  (LPCSTR) &ImgImpName->Name);  
  
               y++;  
            }  
  
            x++;  
         }  
  
  
      }  
   }  
   \_except (EXCEPTION\_EXECUTE\_HANDLER)  
   {  
      VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);  
      VirtualFree((LPVOID) PE\_Buffer, 0, MEM\_RELEASE);  
      CloseHandle(hProcess);  
      return FALSE;  
   }  
  
   //  
   // 创建 PE 文件的虚拟映像  
   //  
  
   PE\_Image = (BYTE \*) VirtualAlloc(NULL,   
      ImgNtHdrs->OptionalHeader.SizeOfImage,   
      MEM\_COMMIT, PAGE\_READWRITE);  
  
   if (PE\_Image == NULL)  
   {  
      VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);  
      VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
      CloseHandle(hProcess);  
      return FALSE;  
   }  
  
   ZeroMemory(PE\_Image, ImgNtHdrs->OptionalHeader.SizeOfImage);  
  
   //  
   // 复制头部  
   //  
  
   IMAGE\_SECTION\_HEADER \*Sect;  
  
   Sect = IMAGE\_FIRST\_SECTION(ImgNtHdrs);  
  
   RtlCopyMemory(PE\_Image, PE\_Buffer, Sect\[0\].PointerToRawData);  
  
   //  
   // 映射区段  
   //  
  
   for (UINT j = 0; j < ImgNtHdrs->FileHeader.NumberOfSections; j++)  
   {  
      BYTE \*Source = (BYTE \*)(Sect\[j\].PointerToRawData +   
         (ULONG\_PTR) PE\_Buffer);  
  
      BYTE \*Dest = (BYTE \*)(Sect\[j\].VirtualAddress +   
         (ULONG\_PTR) PE\_Image);  
  
      RtlCopyMemory(Dest, Source, Sect\[j\].SizeOfRawData);  
   }  
  
   BOOL bRet = WriteProcessMemory(hProcess, (LPVOID) ImgBase,   
      PE\_Image, ImgNtHdrs->OptionalHeader.SizeOfImage, &BR);  
  
   if (!bRet)  
      VirtualFreeEx(hProcess, (LPVOID) ImgBase, 0, MEM\_RELEASE);  
  
   VirtualFree(PE\_Image, 0, MEM\_RELEASE);  
   VirtualFree(PE\_Buffer, 0, MEM\_RELEASE);  
   CloseHandle(hProcess);  
  
   if (bRet == TRUE && BaseAddress != NULL)  
      \*BaseAddress = ImgBase;   
  
   return bRet;  
}  
  
为了填充输入表我使用了远程的 getprocaddress 因为 dll 的输入表只是没有定位到系统的系统的 dll 上，所以一个本地的 getprocaddress 就已经足够了。下面的关联过程：  
  
void CAntiMidaDlg::OnBnClickedAntimida()  
{  
   ProcList.GetItemText(ProcList.GetNextItem(-1, LVNI\_SELECTED), 1,   
      Buffer, sizeof (Buffer) -1);  
  
   ULONG\_PTR PID = \_tcstoul(Buffer, 0, 16);  
     
   wsprintf(Buffer, \_T("%samdll.dll"), CurDir);  
  
   ULONG\_PTR BaseAddress;  
  
   //  
   // 注入模块  
   //  
  
   if (InjectModule(PID, \_T("amdll.dll"), &BaseAddress))  
   {  
      HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, PID);  
  
      if (hProcess == NULL)  
      {  
         MessageBox(\_T("Cannot make the process AntiMida"), "AntiMida");  
         return;  
      }  
  
      //  
      // 关联 ReadProcessMemory  
      //  
  
      ULONG\_PTR Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("kernel32.dll")),   
         "ReadProcessMemory");  
  
      BYTE Instr = 0xB8;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      ULONG\_PTR Rva = GetExportRva(Buffer, "\_FakeReadProcessMemory@20");  
  
      ULONG\_PTR HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WORD Instr2 = 0xE0FF;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //  
      // 关联 VirtualQueryEx  
      //  
  
      Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("kernel32.dll")),   
         "VirtualQueryEx");  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      Rva = GetExportRva(Buffer, "\_FakeVirtualQueryEx@16");  
  
      HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //  
      // 关联 WriteProcessMemory  
      //  
  
      Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("kernel32.dll")),   
         "WriteProcessMemory");  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      Rva = GetExportRva(Buffer, "\_FakeWriteProcessMemory@20");  
  
      HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //   
      // 关联 NtAllocateVirtualMemory  
      // 从这里开始我就直接关联 ntdll  
      //  
  
      Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("ntdll.dll")),   
         "NtAllocateVirtualMemory");  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      Rva = GetExportRva(Buffer, "\_FakeNtAllocateVirtualMemory@24");  
  
      HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //   
      // 关联 NtCreateThread  
      //  
  
      Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("ntdll.dll")),   
         "NtCreateThread");  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      Rva = GetExportRva(Buffer, "\_FakeNtCreateThread@32");  
  
      HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //  
      // 关联 ZwDebugContinue  
      //  
  
      Addr = (ULONG\_PTR) GetProcAddress(  
         GetModuleHandle(\_T("ntdll.dll")),   
         "ZwDebugContinue");  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr,   
         sizeof (BYTE), NULL);  
  
      Addr++;  
  
      Rva = GetExportRva(Buffer, "\_FakeZwDebugContinue@12");  
  
      HookVA = BaseAddress + Rva;  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &HookVA,   
         sizeof (ULONG\_PTR), NULL);  
  
      Addr += sizeof (ULONG\_PTR);  
  
      WriteProcessMemory(hProcess, (LPVOID) Addr, &Instr2,   
         sizeof (WORD), NULL);  
  
      //  
      // 全都关联了  
      //  
  
      CloseHandle(hProcess);  
  
      MessageBox(\_T("The process is now AntiMida"), "AntiMida");  
   }  
   else  
   {  
      MessageBox(\_T("Cannot make the process AntiMida"), "AntiMida");  
   }  
}  
  
你是不是在那们为什么我关联了 kernel32 中起始的几个函数和 ntdll 中剩下的函数: 没什么原因。我只是在试试他能不能运行并且关联 kernel32 也没什么原因，然后我又懒得再去重写这些代码并关联了 ntdll (看见了吧，我真的是很懒的)。实际上最好的方法是直接关联 ntdll，因为他是最直接也是最快捷的(你会知道的)。GetExportRva 只是我写的一个勇于从输出表获取 RVA 的小函数： 

ULONG\_PTR GetExportRva(IN TCHAR \*FileName, IN CHAR \*FunctionName)  
{  
   HMODULE hModule = LoadLibrary(FileName);  
  
   if (hModule == NULL)  
      return 0;  
  
   ULONG\_PTR VA = (ULONG\_PTR) GetProcAddress(hModule,  
      FunctionName);  
  
   FreeLibrary(hModule);  
  
   return (ULONG\_PTR) (VA - (ULONG\_PTR) hModule);  
}  
  
Very useful isn't it? Here's the code of the dll i inject:  
  
#include <windows.h>  
#include <tchar.h>  
  
//  
// 驱动部分  
//  
  
#ifndef NTSTATUS  
typedef LONG NTSTATUS;  
#define NT\_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)  
#define STATUS\_SUCCESS      ((NTSTATUS)0x00000000L)  
#endif   
  
BOOL OpenDevice(IN LPCTSTR DriverName, HANDLE \* lphDevice);  
  
#define FILE\_DEVICE\_ANTIMIDA 0x8000  
  
#define CODE\_RESTORE\_INFO   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x800, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_READ\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x801, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_QUERY\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x802, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_WRITE\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x803, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_ALLOC\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x804, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_CREATE\_THREAD   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x805, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_DBG\_CONTINUE   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x806, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
  
//  
// ZwReadVirtualMemory 部分  
//   
  
typedef struct \_Input\_ZwReadVirtualMemory  
{  
   HANDLE  ProcessHandle;  
   PVOID  BaseAddress;  
   PVOID  Buffer;  
   ULONG  BufferLength;  
   PULONG  ReturnLength;  
} Input\_ZwReadVirtualMemory;  
  
extern "C" \_\_declspec(dllexport)   
BOOL WINAPI FakeReadProcessMemory(HANDLE hProcess,     
                          LPCVOID lpBaseAddress,  
                          LPVOID lpBuffer,   DWORD nSize,  
                          LPDWORD lpNumberOfBytesRead)  
{  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   Input\_ZwReadVirtualMemory Input;  
  
   Input.ProcessHandle = hProcess;  
   Input.BaseAddress = (PVOID) lpBaseAddress;  
   Input.Buffer = (PVOID) lpBuffer;  
   Input.BufferLength = (ULONG) nSize;  
   Input.ReturnLength = (PULONG) lpNumberOfBytesRead;  
  
   DWORD RetBytes;  
  
   if (!DeviceIoControl(hDevice, CODE\_READ\_MEM,   
      &Input, sizeof (Input\_ZwReadVirtualMemory), NULL, 0, &RetBytes, NULL))  
   {  
      CloseHandle(hDevice);  
      return FALSE;  
   }  
  
   CloseHandle(hDevice);  
  
   return TRUE;  
}  
  
//  
// ZwQueryVirtualMemory 部分  
//  
  
typedef enum \_MEMORY\_INFORMATION\_CLASS {  
   MemoryBasicInformation,  
   MemoryWorkingSetList,  
   MemorySectionName,  
   MemoryBasicVlmInformation  
} MEMORY\_INFORMATION\_CLASS;  
  
typedef struct \_Input\_ZwQueryVirtualMemory  
{  
   HANDLE ProcessHandle;  
   PVOID  BaseAddress;  
   MEMORY\_INFORMATION\_CLASS  MemoryInformationClass;  
   ULONG  MemoryInformationLength;  
   PVOID  MemoryInformation;  
   PULONG  ReturnLength;  
} Input\_ZwQueryVirtualMemory;  
  
extern "C" \_\_declspec(dllexport)   
SIZE\_T WINAPI FakeVirtualQueryEx(IN HANDLE hProcess,  
                         IN LPCVOID lpAddress,  
                         OUT PMEMORY\_BASIC\_INFORMATION lpBuffer,  
                         IN SIZE\_T dwLength)  
{  
   Input\_ZwQueryVirtualMemory Input;  
   ULONG ReturnLength;  
  
   Input.ProcessHandle = hProcess;  
   Input.BaseAddress = (PVOID) lpAddress;  
   Input.MemoryInformationClass = MemoryBasicInformation;  
   Input.MemoryInformationLength = (ULONG) dwLength;  
   Input.MemoryInformation = (PVOID) lpBuffer;  
   Input.ReturnLength = &ReturnLength;  
  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   DWORD RetBytes;  
  
   if (!DeviceIoControl(hDevice, CODE\_QUERY\_MEM,   
      &Input, sizeof (Input\_ZwQueryVirtualMemory),   
      NULL, 0, &RetBytes, NULL))  
   {  
      CloseHandle(hDevice);  
      return 0;  
   }  
  
   CloseHandle(hDevice);  
  
   return (SIZE\_T) ReturnLength;  
}  
  
//  
// ZwReadVirtualMemory 部分  
//   
  
typedef struct \_Input\_ZwWriteVirtualMemory  
{  
   HANDLE  ProcessHandle;  
   PVOID  BaseAddress;  
   PVOID  Buffer;  
   ULONG  BufferLength;  
   PULONG  ReturnLength;  
} Input\_ZwWriteVirtualMemory;  
  
extern "C" \_\_declspec(dllexport)   
BOOL WINAPI FakeWriteProcessMemory(HANDLE hProcess, LPVOID lpBaseAddress,  
                           LPVOID lpBuffer, DWORD nSize,   
                           LPDWORD lpNumberOfBytesWritten)  
{  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   //  
   // 修改内存保护  
   //  
  
   DWORD dwOldProtection;  
  
   if (!VirtualProtectEx(hProcess, lpBaseAddress, nSize,   
      PAGE\_EXECUTE\_READWRITE, &dwOldProtection))  
   {  
      CloseHandle(hDevice);  
      return FALSE;  
   }  
  
   Input\_ZwWriteVirtualMemory Input;  
  
   Input.ProcessHandle = hProcess;  
   Input.BaseAddress = (PVOID) lpBaseAddress;  
   Input.Buffer = (PVOID) lpBuffer;  
   Input.BufferLength = (ULONG) nSize;  
   Input.ReturnLength = (PULONG) lpNumberOfBytesWritten;  
  
   DWORD RetBytes;  
   BOOL bRet;  
  
   if (!DeviceIoControl(hDevice, CODE\_WRITE\_MEM,   
      &Input, sizeof (Input\_ZwWriteVirtualMemory), NULL, 0, &RetBytes, NULL))  
   {  
      bRet = FALSE;  
   }  
  
   CloseHandle(hDevice);  
  
   //  
   // 恢复内存保护  
   //  
  
   VirtualProtectEx(hProcess, lpBaseAddress, nSize,   
      dwOldProtection, &dwOldProtection);  
  
   return TRUE;  
}  
  
//  
// ZwAllocateVirtualMemory 部分  
//  
  
typedef struct \_Input\_ZwAllocateVirtualMemory  
{  
   HANDLE ProcessHandle;  
   PVOID \*BaseAddress;  
   ULONG\_PTR ZeroBits;  
   PSIZE\_T RegionSize;  
   ULONG AllocationType;  
   ULONG Protect;  
} Input\_ZwAllocateVirtualMemory;  
  
extern "C" \_\_declspec(dllexport)  
NTSTATUS NTAPI FakeNtAllocateVirtualMemory(IN HANDLE ProcessHandle,  
                                 IN OUT PVOID \*BaseAddress,  
                                 IN ULONG\_PTR ZeroBits,  
                                 IN OUT PSIZE\_T RegionSize,  
                                 IN ULONG AllocationType,  
                                 IN ULONG Protect)  
{  
   Input\_ZwAllocateVirtualMemory Input;  
  
   Input.ProcessHandle = ProcessHandle;  
   Input.BaseAddress = BaseAddress;  
   Input.ZeroBits = ZeroBits;  
   Input.RegionSize = RegionSize;  
   Input.AllocationType = AllocationType;  
   Input.Protect = Protect;  
  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   DWORD RetBytes;  
  
   if (!DeviceIoControl(hDevice, CODE\_ALLOC\_MEM,   
      &Input, sizeof (Input\_ZwAllocateVirtualMemory),   
      NULL, 0, &RetBytes, NULL))  
   {  
      CloseHandle(hDevice);  
  
      // who cares? (invalid parameter)  
      return ((NTSTATUS)0xC000000DL);   
   }  
  
   CloseHandle(hDevice);  
  
   return STATUS\_SUCCESS;  
}  
  
//  
// ZwCreateThread 部分  
//  
  
typedef struct \_Input\_ZwCreateThread  
{  
   PHANDLE  ThreadHandle;  
   ACCESS\_MASK  DesiredAccess;  
   PVOID  ObjectAttributes;  
   HANDLE  ProcessHandle;  
   PVOID  ClientId;  
   PCONTEXT  ThreadContext;  
   PVOID  UserStack;  
   BOOLEAN  CreateSuspended;  
} Input\_ZwCreateThread;  
  
extern "C" \_\_declspec(dllexport)  
NTSTATUS NTAPI FakeNtCreateThread(OUT PHANDLE  ThreadHandle,  
                          IN ACCESS\_MASK  DesiredAccess,  
                          IN PVOID  ObjectAttributes,  
                          IN HANDLE  ProcessHandle,  
                          OUT PVOID  ClientId,  
                          IN PCONTEXT  ThreadContext,  
                          IN PVOID  UserStack,  
                          IN BOOLEAN  CreateSuspended)  
{  
   Input\_ZwCreateThread Input;  
  
   Input.ThreadHandle = ThreadHandle;  
   Input.DesiredAccess = DesiredAccess;  
   Input.ObjectAttributes = ObjectAttributes;  
   Input.ProcessHandle = ProcessHandle;  
   Input.ClientId = ClientId;  
   Input.ThreadContext = ThreadContext;  
   Input.UserStack = UserStack;  
   Input.CreateSuspended = CreateSuspended;  
  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   DWORD RetBytes;  
  
   if (!DeviceIoControl(hDevice, CODE\_CREATE\_THREAD,   
      &Input, sizeof (Input\_ZwCreateThread),   
      NULL, 0, &RetBytes, NULL))  
   {  
      CloseHandle(hDevice);  
  
      // 谁关心呢？(无效的参数)  
      return ((NTSTATUS)0xC000000DL);   
   }  
  
   CloseHandle(hDevice);  
  
   return STATUS\_SUCCESS;  
}  
  
//  
// ZwDebugContinue 部分  
//  
  
typedef struct \_Input\_ZwDebugContinue  
{  
   PVOID \*A;  
   PVOID \*B;  
   PVOID \*C;  
} Input\_ZwDebugContinue;  
  
  
extern "C" \_\_declspec(dllexport)  
NTSTATUS NTAPI FakeZwDebugContinue(PVOID \*A, PVOID \*B, PVOID \*C)  
{  
   Input\_ZwDebugContinue Input;  
  
   Input.A = A;  
   Input.B = B;  
   Input.C = C;  
  
   HANDLE hDevice;  
  
   if (OpenDevice(\_T("antimida"), &hDevice) == FALSE)  
      return FALSE;  
  
   DWORD RetBytes;  
  
   if (!DeviceIoControl(hDevice, CODE\_DBG\_CONTINUE,   
      &Input, sizeof (Input\_ZwDebugContinue),   
      NULL, 0, &RetBytes, NULL))  
   {  
      CloseHandle(hDevice);  
  
      // 谁关心呢？(无效的参数)  
      return ((NTSTATUS)0xC000000DL);   
   }  
  
   CloseHandle(hDevice);  
  
   return STATUS\_SUCCESS;  
}  
  
BOOL OpenDevice(IN LPCTSTR DriverName, HANDLE \* lphDevice)  
{  
   TCHAR    completeDeviceName\[64\];  
   HANDLE   hDevice;  
  
   /\*if ( (GetVersion() & 0xFF) >= 5 ) { wsprintf(completeDeviceName, TEXT("\\\\\\\\.\\\\Global\\\\%s"), DriverName); } else {\*/  
  
      wsprintf(completeDeviceName, TEXT("\\\\\\\\.\\\\%s"), DriverName);  
   //}  
  
   hDevice = CreateFile(completeDeviceName, GENERIC\_READ | GENERIC\_WRITE, 0,  
      NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);  
  
   if (hDevice == ((HANDLE)-1))  
      return FALSE;  
  
   if (lphDevice)  
      \*lphDevice = hDevice;  
   else  
      CloseHandle(hDevice);  
  
   return TRUE;  
}  
  
BOOL APIENTRY DllMain(HANDLE hModule, DWORD  Reason, LPVOID lpReserved)  
{  
   switch (Reason)  
   {  
   case DLL\_PROCESS\_ATTACH:  
      {     
         DisableThreadLibraryCalls((HMODULE) hModule);  
         break;  
      }  
  
   case DLL\_PROCESS\_DETACH:  
      {  
         return TRUE;  
      }  
  
   default:  
      {  
         return FALSE;  
      }  
   }  
  
   return TRUE;  
}  
  
我不认为对这段代码还有什么要解释的。因为我说会在驱动中完成请求的操作，但是怎么做的？仅仅是调用 ntoskrnl 中的原始的服务。问题是 ntoskrnl 没有在他的导出表中输出 SDT 中的服务，那么是什么获得正确的地址的呢？这其实是个小小的骗局。在 ntoskrnl 中有一个 SDT  的复本。所以我写了这些代码来获取原始的服务的地址：  
  
//  
// 感谢 90210 提供这些代码  
// http://www.rootkit.com/newsread.php?newsid=176  
// 你节省了我的时间  
//  
  
#include "stdafx.h"  
  
#define RVATOVA(base,offset) ((PVOID)((DWORD)(base)+(DWORD)(offset)))  
#define ibaseDD \*(PDWORD)&ibase  
#define STATUS\_INFO\_LENGTH\_MISMATCH      ((NTSTATUS)0xC0000004L)  
#define NT\_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)  
  
typedef struct {  
   WORD    offset:12;  
   WORD    type:4;  
} IMAGE\_FIXUP\_ENTRY, \*PIMAGE\_FIXUP\_ENTRY;  
  
  
typedef LONG NTSTATUS;  
  
NTSTATUS (WINAPI \*pNtQuerySystemInformation)(      
      DWORD    SystemInformationClass,  
      PVOID    SystemInformation,  
      ULONG    SystemInformationLength,  
      PULONG    ReturnLength  
      );  
  
  
typedef struct \_SYSTEM\_MODULE\_INFORMATION {//信息类 11  
   ULONG    Reserved\[2\];  
   PVOID    Base;  
   ULONG    Size;  
   ULONG    Flags;  
   USHORT    Index;  
   USHORT    Unknown;  
   USHORT    LoadCount;  
   USHORT    ModuleNameOffset;  
   CHAR    ImageName\[256\];  
}SYSTEM\_MODULE\_INFORMATION,\*PSYSTEM\_MODULE\_INFORMATION;  
  
typedef struct {  
   DWORD    dwNumberOfModules;  
   SYSTEM\_MODULE\_INFORMATION    smi;  
} MODULES, \*PMODULES;  
  
#define    SystemModuleInformation    11  
  
DWORD GetHeaders(PBYTE ibase,  
             PIMAGE\_FILE\_HEADER \*pfh,  
             PIMAGE\_OPTIONAL\_HEADER \*poh,  
             PIMAGE\_SECTION\_HEADER \*psh)  
  
{  
   PIMAGE\_DOS\_HEADER mzhead=(PIMAGE\_DOS\_HEADER)ibase;  
  
   if    ((mzhead->e\_magic!=IMAGE\_DOS\_SIGNATURE) ||          
      (ibaseDD\[mzhead->e\_lfanew\]!=IMAGE\_NT\_SIGNATURE))  
      return FALSE;  
  
   \*pfh=(PIMAGE\_FILE\_HEADER)&ibase\[mzhead->e\_lfanew\];  
   if (((PIMAGE\_NT\_HEADERS)\*pfh)->Signature!=IMAGE\_NT\_SIGNATURE)   
      return FALSE;  
   \*pfh=(PIMAGE\_FILE\_HEADER)((PBYTE)\*pfh+sizeof(IMAGE\_NT\_SIGNATURE));  
  
   \*poh=(PIMAGE\_OPTIONAL\_HEADER)((PBYTE)\*pfh+sizeof(IMAGE\_FILE\_HEADER));  
   if ((\*poh)->Magic!=IMAGE\_NT\_OPTIONAL\_HDR32\_MAGIC)  
      return FALSE;  
  
   \*psh=(PIMAGE\_SECTION\_HEADER)((PBYTE)\*poh+sizeof(IMAGE\_OPTIONAL\_HEADER));  
   return TRUE;  
}  
  
  
DWORD FindKiServiceTable(HMODULE hModule,DWORD dwKSDT)  
{  
   PIMAGE\_FILE\_HEADER    pfh;  
   PIMAGE\_OPTIONAL\_HEADER    poh;  
   PIMAGE\_SECTION\_HEADER    psh;  
   PIMAGE\_BASE\_RELOCATION    pbr;  
   PIMAGE\_FIXUP\_ENTRY    pfe;      
  
   DWORD    dwFixups=0,i,dwPointerRva,dwPointsToRva,dwKiServiceTable;  
   BOOL    bFirstChunk;  
  
   GetHeaders((PBYTE)hModule,&pfh,&poh,&psh);  
  
   // 循环通过重定位以加速搜索  
   if ((poh->DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress) &&  
      (!((pfh->Characteristics)&IMAGE\_FILE\_RELOCS\_STRIPPED))) {  
  
         pbr=(PIMAGE\_BASE\_RELOCATION)RVATOVA(poh->DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress,hModule);  
  
         bFirstChunk=TRUE;  
         // 1st IMAGE\_BASE\_RELOCATION.VirtualAddress 在 ntoskrnl 中是 0  
         while (bFirstChunk || pbr->VirtualAddress) {  
            bFirstChunk=FALSE;  
  
            pfe=(PIMAGE\_FIXUP\_ENTRY)((DWORD)pbr+sizeof(IMAGE\_BASE\_RELOCATION));  
  
            for (i=0;i<(pbr->SizeOfBlock-sizeof(IMAGE\_BASE\_RELOCATION))>>1;i++,pfe++) {  
               if (pfe->type==IMAGE\_REL\_BASED\_HIGHLOW) {  
                  dwFixups++;  
                  dwPointerRva=pbr->VirtualAddress+pfe->offset;  
                  // DONT\_RESOLVE\_DLL\_REFERENCES 标识意味着重定位没有被修复  
                  dwPointsToRva=\*(PDWORD)((DWORD)hModule+dwPointerRva)-(DWORD)poh->ImageBase;  
  
                  // 这个重定位地址是 KeServiceDescriptorTable.Base?  
                  if (dwPointsToRva==dwKSDT) {  
                     // 检查 mov \[mem32\],imm32. 我们正在尝试找  
                     // "mov ds:\_KeServiceDescriptorTable.Base, offset \_KiServiceTable"  
                     // 从 KiInitSystem.  
                     if (\*(PWORD)((DWORD)hModule+dwPointerRva-2)==0x05c7) {  
                        // 这里应该要检查在 KiServiceTable 的重定位的  
                        // 但是不管它了  
                        dwKiServiceTable=\*(PDWORD)((DWORD)hModule+dwPointerRva+4)-poh->ImageBase;  
                        return dwKiServiceTable;  
                     }  
                  }  
  
               } else  
                  if (pfe->type!=IMAGE\_REL\_BASED\_ABSOLUTE)  
                  {  
                     // 理论上不应该会用到这里的  
                  }  
                       
            }  
            \*(PDWORD)&pbr+=pbr->SizeOfBlock;  
         }  
      }      
  
      if (!dwFixups)   
      {  
         // 应该不会到这里的 - nt, 2k, xp 的核心有重定位数据  
      }  
      return 0;  
}  
  
BOOL SDT\_GetOriginalFunctions(PRESTORE\_INFO pRestoreInfo)  
{      
   HMODULE    hKernel;  
   DWORD    dwKSDT;                // rva of KeServiceDescriptorTable  
   DWORD    dwKiServiceTable;    // rva of KiServiceTable  
   PMODULES    pModules=(PMODULES)&pModules;  
   DWORD    dwNeededSize,rc;  
   DWORD    dwKernelBase,dwServices=0;  
   PCHAR    pKernelName;  
   PDWORD    pService;  
   PIMAGE\_FILE\_HEADER    pfh;  
   PIMAGE\_OPTIONAL\_HEADER    poh;  
   PIMAGE\_SECTION\_HEADER    psh;  
  
   pNtQuerySystemInformation = (NTSTATUS (WINAPI \*)(      
      DWORD, PVOID, ULONG, PULONG)) GetProcAddress(  
      GetModuleHandle("ntdll.dll"), "NtQuerySystemInformation");  
  
   // 获取系统模块 - 在这里 ntoskrnl 是第一个  
   rc=pNtQuerySystemInformation(SystemModuleInformation,pModules,4,&dwNeededSize);  
   if (rc==STATUS\_INFO\_LENGTH\_MISMATCH) {  
      pModules = (PMODULES) GlobalAlloc(GPTR,dwNeededSize);  
      rc=pNtQuerySystemInformation(SystemModuleInformation,pModules,dwNeededSize,NULL);  
   } else {  
strange:  
      return FALSE;  
   }  
   if (!NT\_SUCCESS(rc)) goto strange;  
  
   // 映像基址  
   dwKernelBase=(DWORD)pModules->smi.Base;  
   // 文件名 - 他可能会在 boot.ini 中被重命名  
   pKernelName=pModules->smi.ModuleNameOffset+pModules->smi.ImageName;  
  
   // 映射 ntoskrnl - 希望他有重定位  
   hKernel=LoadLibraryEx(pKernelName,0,DONT\_RESOLVE\_DLL\_REFERENCES);  
   if (!hKernel) {  
      return FALSE;          
   }  
  
   GlobalFree(pModules);  
  
   // 我们自己的重定位操作在这里一无是处 - 因为我们有 GetProcAddress :)   
   if (!(dwKSDT=(DWORD)GetProcAddress(hKernel,"KeServiceDescriptorTable"))) {  
      return FALSE;  
   }  
  
   // 获取 KeServiceDescriptorTable rva  
   dwKSDT-=(DWORD)hKernel;      
   // 查找 KiServiceTable  
   if (!(dwKiServiceTable=FindKiServiceTable(hKernel,dwKSDT))) {  
      return FALSE;  
   }  
  
   // 我们来转存 KiServiceTable 的内容  
  
   // 可能会失败!!!  
   // 这里应该会获得正确的 ServiceLimit，但是这在核心模式中的价值不高  
   GetHeaders((PBYTE) hKernel,&pfh,&poh,&psh);  
  
   //  
   // 我们的代码  
   //  
  
   //  
   // 获取 ZwAllocateVirtualMemory  
   //  
  
   DWORD nServ = GrabService("ZwAllocateVirtualMemory");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwAllocateVirtualMemory =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwCreateThread   
   //  
  
   nServ = GrabService("ZwCreateThread");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwCreateThread =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwDebugContinue   
   //  
  
   nServ = GrabService("ZwDebugContinue");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwDebugContinue =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwQueryVirtualMemory   
   //  
  
   nServ = GrabService("ZwQueryVirtualMemory");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwQueryVirtualMemory =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwReadVirtualMemory   
   //  
  
   nServ = GrabService("ZwReadVirtualMemory");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwReadVirtualMemory =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwTerminateProcess   
   //  
  
   nServ = GrabService("ZwTerminateProcess");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwTerminateProcess =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   //  
   // 获取 ZwWriteVirtualMemory   
   //  
  
   nServ = GrabService("ZwWriteVirtualMemory");  
  
   pService = (PDWORD)((nServ \* sizeof (DWORD)) +   
      (DWORD) hKernel + dwKiServiceTable);  
  
   pRestoreInfo->ZwWriteVirtualMemory =   
      (\*pService-poh->ImageBase + dwKernelBase);  
  
   FreeLibrary(hKernel);  
  
} 

GrabService 函数是基于另一个小骗局来获取 SDT 项目的。事实上 SDT 的项目随着 windows 的变化而变化，所以获取他的最好的方法是从 ntdll 中读取。看看 NtReadVirtualMemory:  
  
.text:7C91E2BB                  public ZwReadVirtualMemory  
.text:7C91E2BB B8 BA 00 00 00   mov     eax, 0BAh ; 这就是 SDT 项目  
.text:7C91E2C0 BA 00 03 FE 7F   mov     edx, 7FFE0300h  
.text:7C91E2C5 FF 12            call    dword ptr \[edx\]  
.text:7C91E2C7 C2 14 00         retn    14h  
  
所以我写了这个函数来获取正确的 SDT 项目:  
  
//  
// 感谢 gareth 提供这个点子  
// http://www.rootkit.com/newsread.php?newsid=248  
//  
  
#include "stdafx.h"  
  
DWORD GrabService(IN CHAR \*FunctionName)  
{  
   DWORD Exp = (DWORD) GetProcAddress(  
      GetModuleHandle(\_T("ntdll.dll")), FunctionName);  
  
   Exp++; // mov opcode  
  
   DWORD \*ptr = (DWORD \*) Exp;  
  
   return \*ptr;  
}  
  
嗯，我在收集信息并发送给驱动(恢复信息)，我还把 KeAttachProcess 原始的代码发送了过去。我并不需要他，但是他在进行恢复时会很有用：  
  
#include "stdafx.h"  
  
ULONG\_PTR RvaToOffset(IMAGE\_NT\_HEADERS \*, ULONG\_PTR);  
BOOL GetExport(BYTE \*, ULONG\_PTR \*, WORD \*, CHAR \*);   
  
BOOL GetNtoskrnlOriginalBytes(PRESTORE\_INFO pRestoreInfo)  
{  
   TCHAR Buffer\[MAX\_PATH\];  
  
   GetSystemDirectory(Buffer, MAX\_PATH);  
  
   \_tcscat(Buffer, \_T("\\\\ntoskrnl.exe"));  
  
   HANDLE hFile = CreateFile(Buffer, GENERIC\_READ, FILE\_SHARE\_READ,  
      NULL, OPEN\_EXISTING, 0, NULL);  
  
   if (hFile == INVALID\_HANDLE\_VALUE)  
      return FALSE;  
  
   DWORD FileSize = GetFileSize(hFile, NULL);  
  
   BYTE \*ptrNtoskrnl = (BYTE \*) VirtualAlloc(NULL, FileSize,   
      MEM\_COMMIT, PAGE\_READWRITE);  
  
   if (ptrNtoskrnl == NULL)  
   {  
      CloseHandle(hFile);  
      return FALSE;  
   }  
  
   DWORD BR;  
  
   if (!ReadFile(hFile, ptrNtoskrnl, FileSize, &BR, NULL))  
   {  
      VirtualFree(ptrNtoskrnl, 0, MEM\_RELEASE);  
      CloseHandle(hFile);  
      return FALSE;  
   }  
  
   CloseHandle(hFile);  
  
   IMAGE\_DOS\_HEADER \*ImgDosHdr = (IMAGE\_DOS\_HEADER \*) ptrNtoskrnl;  
  
   IMAGE\_NT\_HEADERS \*ImgNtHdrs = (IMAGE\_NT\_HEADERS \*)  
      &ptrNtoskrnl\[ImgDosHdr->e\_lfanew\];  
  
   ULONG\_PTR EP\_Rva = 0;  
  
   if (!GetExport(ptrNtoskrnl, &EP\_Rva, NULL, "KeAttachProcess"))  
   {  
      VirtualFree(ptrNtoskrnl, 0, MEM\_RELEASE);  
      return FALSE;  
   }  
  
   BYTE \*ptr = (BYTE \*) (EP\_Rva + (ULONG\_PTR) ptrNtoskrnl);  
  
   memcpy(pRestoreInfo->KeAttachProcessPatch, ptr, 5);  
  
   VirtualFree(ptrNtoskrnl, 0, MEM\_RELEASE);  
  
   return TRUE;  
}  
  
BOOL CollectInformation(PRESTORE\_INFO pRestoreInfo)  
{  
   if (SDT\_GetOriginalFunctions(pRestoreInfo) == FALSE)  
      return FALSE;  
  
   return GetNtoskrnlOriginalBytes(pRestoreInfo);  
}  
  
函数 GetExport 是我已有的函数，很有用，从输出表中获取你请求的信息：  
  
BOOL GetExport(BYTE \*PE, ULONG\_PTR \*EP, WORD \*Ordinal, CHAR \*FuncName)   
{  
   IMAGE\_DOS\_HEADER \*ET\_DOS;  
   IMAGE\_NT\_HEADERS \*ET\_NT;  
   IMAGE\_EXPORT\_DIRECTORY \*Export;  
   ULONG\_PTR ET, \*Functions;  
   PSTR \*Names;  
   WORD \*Ordinals;  
   CHAR \*ApiName;  
  
   \_\_try  
   {  
      ET\_DOS = (IMAGE\_DOS\_HEADER \*)(ULONG\_PTR) PE;  
  
      ET\_NT = (IMAGE\_NT\_HEADERS \*)(ULONG\_PTR) &PE\[ET\_DOS->e\_lfanew\];  
  
      ET = ET\_NT->OptionalHeader.DataDirectory  
         \[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress;  
  
      if (ET == NULL)  
         return FALSE;  
  
      ET = RvaToOffset(ET\_NT, ET);  
  
      if (ET == 0) return FALSE;  
  
      Export = (IMAGE\_EXPORT\_DIRECTORY \*)(ET + (ULONG\_PTR) PE);  
  
      Functions = (ULONG\_PTR \*)(RvaToOffset(ET\_NT,   
         Export->AddressOfFunctions) + (ULONG\_PTR) PE);  
      Ordinals = (WORD \*)(RvaToOffset(ET\_NT,   
         Export->AddressOfNameOrdinals) + (ULONG\_PTR) PE);  
      Names = (PSTR \*)(RvaToOffset(ET\_NT,   
         Export->AddressOfNames) + (ULONG\_PTR) PE);  
  
      if (EP != NULL && \*EP != 0)  
      {  
         for (WORD x = 0; x < Export->NumberOfFunctions; x++)  
         {  
            if (\*EP == Functions\[x\])  
            {  
               if (Ordinal) \*Ordinal = (WORD) (x + Export->Base);  
  
               if (FuncName != NULL)  
               {  
                  for (WORD i = 0; i < Export->NumberOfNames; i++)  
                  {  
                     if (Ordinals\[i\] == x)  
                     {  
                        ApiName = (char \*) RvaToOffset(ET\_NT, (ULONG\_PTR) Names\[i\]);  
  
                        if (ApiName != NULL)  
                        {  
                           ApiName = (char \*)((ULONG\_PTR) ApiName +   
                              (ULONG\_PTR) PE);  
                           strcpy(FuncName, ApiName);  
                           break;  
                        }  
                     }  
                  }  
               }  
  
               return TRUE;  
            }     
         }  
  
         return FALSE;  
      }  
      else  
      {  
         if (FuncName == NULL || FuncName\[0\] == 0)  
         {  
            if (Ordinal == NULL || \*Ordinal == 0)  
               return FALSE;  
  
            if (\*Ordinal < Export->Base ||  
               \*Ordinal > (Export->Base + (Export->NumberOfFunctions - 1)))  
               return FALSE;  
  
            WORD FuncEntry = (WORD) (\*Ordinal - Export->Base);  
  
            if (EP) \*EP = Functions\[FuncEntry\];  
  
            if (FuncName != NULL)  
            {  
               for (DWORD i = 0; i < Export->NumberOfNames; i++)  
               {  
                  if (Ordinals\[i\] == FuncEntry)  
                  {  
                     ApiName = (char \*) RvaToOffset(ET\_NT, (ULONG\_PTR) Names\[i\]);  
  
                     if (ApiName != NULL)  
                     {  
                        ApiName = (char \*)((ULONG\_PTR) ApiName + (ULONG\_PTR) PE);  
                        strcpy(FuncName, ApiName);  
                        break;  
                     }  
                  }  
               }  
  
            }  
  
            return TRUE;  
         }  
         else  
         {  
            for (DWORD x = 0; x < Export->NumberOfFunctions; x++)  
            {  
               if (Functions\[x\] == 0)  
                  continue;  
  
               for (DWORD i = 0; i < Export->NumberOfNames; i++)  
               {  
                  if (Ordinals\[i\] == x)  
                  {  
                     ApiName = (char \*) RvaToOffset(ET\_NT, (ULONG\_PTR) Names\[i\]);  
  
                     if (ApiName != NULL)  
                     {  
                        ApiName = (char \*)((ULONG\_PTR) ApiName + (ULONG\_PTR) PE);  
  
                        if (strcmp(ApiName, FuncName) == 0)  
                        {  
                           if (Ordinal) \*Ordinal = (WORD) (x + Export->Base);  
                           if (EP) \*EP = Functions\[x\];  
                           return TRUE;  
                        }  
                     }  
                  }  
               }  
  
            }  
  
            return FALSE;  
         }  
      }  
  
   }  
  
   \_\_except (EXCEPTION\_EXECUTE\_HANDLER)  
   {  
      return FALSE;  
   }  
  
   return FALSE;  
}  
  
在收集信息后，我把它通过 DeviceIoControl 发送给了驱动并接受及处理信息：  
  
NTSTATUS ControlDispatcher(PDEVICE\_CONTEXT pDeviceContext, DWORD dwCode,  
                     BYTE \*pInput, DWORD dwInputSize,  
                     BYTE \*pOutput, DWORD dwOutputSize, DWORD \*pdwInfo)  
{  
   switch (dwCode)  
   {  
  
   case CODE\_RESTORE\_INFO:  
      {  
         RtlCopyMemory(&RestoreInfo, pInput, sizeof (RESTORE\_INFO));  
  
         pZwReadVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID, PVOID, ULONG, PULONG))  
            RestoreInfo.ZwReadVirtualMemory;  
  
         pZwQueryVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID, MEMORY\_INFORMATION\_CLASS, PVOID, ULONG,  
            PULONG)) RestoreInfo.ZwQueryVirtualMemory;  
  
         pZwWriteVirtualMemory = (NTSTATUS (\*)(HANDLE, PVOID,  
            PVOID, ULONG, PULONG))  
            RestoreInfo.ZwWriteVirtualMemory;  
  
         pZwAllocateVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID \*, ULONG\_PTR, PSIZE\_T, ULONG, ULONG))  
            RestoreInfo.ZwAllocateVirtualMemory;  
  
         pZwCreateThread = (NTSTATUS (\*)(PHANDLE, ACCESS\_MASK,  
            POBJECT\_ATTRIBUTES, HANDLE, PCLIENT\_ID,  
            PCONTEXT, PUSER\_STACK, BOOLEAN))  
            RestoreInfo.ZwCreateThread;  
  
         pZwDebugContinue = (NTSTATUS (\*)(PVOID \*A, PVOID \*B, PVOID \*C))  
            RestoreInfo.ZwDebugContinue;  
  
         RebuildNtoskrnl();  
  
         break;  
      }  
  
RebuildNtoskrnl 函数修补了 KeAttachProcess 被替换的字节:  
  
//  
// 修改 KeAttachProcess  
//  
  
VOID RebuildNtoskrnl()  
{  
   PMDL Mdl;  
   DWORD CR0Backup;  
   DWORD \*ptr;  
   BYTE \*Patch;  
  
   ptr = (DWORD \*)(2 + (DWORD) &KeAttachProcess);  
  
   ptr = (DWORD\*) \*ptr;  
  
   Patch = (BYTE \*) \*ptr;  
  
   Mdl = MmCreateMdl(0, (PVOID) Patch, 5);  
  
   if (Mdl == NULL)  
      return;  
  
   MmProbeAndLockPages(Mdl, 0, 0);  
  
   if (\*Patch == 0xE9)  
   {  
      \_\_asm  
      {  
         mov eax, cr0  
         mov CR0Backup, eax  
         and eax, 0xFFFEFFFF  
         mov cr0, eax  
      }  
  
      //  
      // 修改   
      //  
  
      memcpy(Patch, RestoreInfo.KeAttachProcessPatch, 5);  
  
      \_\_asm  
      {  
         mov eax, CR0Backup  
         mov cr0, eax  
      }  
   }  
  
   MmUnlockPages(Mdl);  
   IoFreeMdl(Mdl);  
}  
  
这就是 SDT 的全部了。现在让我们谈谈监视，就是: regmon 和 filemon. 这两个中只要有任何一个启动了，TheMida 就不会启动。这里有不少检测他们驱动的方法，恐怕太多了些。下面是他们中的一部分：  
  
1 - 查找进程。  
2 - 查找窗口(我在某些壳中也发现了这个)。  
3 - 查找注册表中的驱动。  
4 - 查找内存中的驱动。  
5 - 查找驱动的对象表。  
6 - 查找 SDT (只对 regmon)。  
  
还有一些其他的方式，但是我认我这些应该是最常用的。我尝试解决了 4 和 5 (因为我有作这些的源代码)。  
  
#include <wdm.h>  
     
typedef struct \_MODULE\_ENTRY   
{  
   LIST\_ENTRY le\_mod;  
   ULONG unknown\[4\];  
   ULONG base;  
   ULONG driver\_start;  
   ULONG unk1;  
   UNICODE\_STRING driver\_Path;  
   UNICODE\_STRING driver\_Name;  
     
   //...  
} MODULE\_ENTRY, \*PMODULE\_ENTRY;  
  
//  
// 这个结构在 valerino 的代码中丢失了  
//  
  
typedef struct \_OBJECT\_HEADER  
{  
   ULONG         PointerCount;  
   ULONG         HandleCount;  
   PVOID         Type;  
   UCHAR         NameInfoOffset;  
   UCHAR         HandleInfoOffset;  
   UCHAR         QuotaInfoOffset;  
   UCHAR         Flags;  
   PVOID         ObjectCreateInfo;  
   PVOID         SecurityDescriptor;  
   struct \_QUAD   Body;  
     
}OBJECT\_HEADER, \* POBJECT\_HEADER;  
  
// valerino 的代码  
#define NUMBER\_HASH\_BUCKETS 37  
  
typedef struct \_OBJECT\_DIRECTORY\_ENTRY {  
    struct \_OBJECT\_DIRECTORY\_ENTRY \*ChainLink;  
    PVOID Object;  
} OBJECT\_DIRECTORY\_ENTRY, \*POBJECT\_DIRECTORY\_ENTRY;  
  
typedef struct \_OBJECT\_DIRECTORY {  
    struct \_OBJECT\_DIRECTORY\_ENTRY \*HashBuckets\[ NUMBER\_HASH\_BUCKETS \];  
    struct \_OBJECT\_DIRECTORY\_ENTRY \*\*LookupBucket;  
    BOOLEAN LookupFound;  
    USHORT SymbolicLinkUsageCount;  
    struct \_DEVICE\_MAP \*DeviceMap;  
} OBJECT\_DIRECTORY, \*POBJECT\_DIRECTORY;  
  
typedef struct \_DEVICE\_MAP {  
    ULONG ReferenceCount;  
    POBJECT\_DIRECTORY DosDevicesDirectory;  
    ULONG DriveMap;  
    UCHAR DriveType\[ 32 \];  
} DEVICE\_MAP, \*PDEVICE\_MAP;  
  
typedef struct \_OBJECT\_HEADER\_NAME\_INFO {  
    POBJECT\_DIRECTORY Directory;  
    UNICODE\_STRING Name;  
    ULONG Reserved;  
} OBJECT\_HEADER\_NAME\_INFO, \*POBJECT\_HEADER\_NAME\_INFO;  
  
#define OBJECT\_TO\_OBJECT\_HEADER( o ) \\  
CONTAINING\_RECORD( (o), OBJECT\_HEADER, Body )  
  
#define OBJECT\_HEADER\_TO\_NAME\_INFO( oh ) ((POBJECT\_HEADER\_NAME\_INFO) \\  
((oh)->NameInfoOffset == 0 ? NULL : ((PCHAR)(oh) - (oh)->NameInfoOffset)))  
  
NTSTATUS ObOpenObjectByName (IN POBJECT\_ATTRIBUTES ObjectAttributes,  
    IN POBJECT\_TYPE ObjectType OPTIONAL, IN KPROCESSOR\_MODE AccessMode,  
    IN OUT PACCESS\_STATE AccessState OPTIONAL, IN ACCESS\_MASK DesiredAccess OPTIONAL,  
    IN OUT PVOID ParseContext OPTIONAL, OUT PHANDLE Handle);  
  
// 结束  
  
NTSTATUS DriverEntry(IN PDRIVER\_OBJECT  DriverObject,   
                IN PUNICODE\_STRING RegistryPath);  
  
VOID Unload(IN PDRIVER\_OBJECT DriverObject);  
  
VOID StealthInitializeLateMore(VOID);  
VOID HideModule(PDRIVER\_OBJECT DriverObject);  
  
#pragma code\_seg("INIT")  
NTSTATUS DriverEntry(IN PDRIVER\_OBJECT  DriverObject,  
                IN PUNICODE\_STRING RegistryPath)  
{  
  
   PDEVICE\_OBJECT pDeviceObject = NULL;  
  
   NTSTATUS Status = IoCreateDevice(DriverObject, 0, NULL,   
      FILE\_DEVICE\_UNKNOWN, 0, FALSE, &pDeviceObject);  
  
   if (NT\_SUCCESS(Status))   
   {  
  
      DriverObject->DriverUnload   = Unload;  
        
      HideModule(DriverObject);  
      StealthInitializeLateMore();  
   }  
  
   return Status;  
}  
#pragma code\_seg()  
  
  
VOID Unload(IN PDRIVER\_OBJECT DriverObject)  
{  
   IoDeleteDevice(DriverObject->DeviceObject);  
}  
  
// 从 PsLoadedModuleList 中隐藏模块  
  
VOID HideModule(PDRIVER\_OBJECT DriverObject)  
{  
   PMODULE\_ENTRY pCurrentModule = NULL;  
     
   // 指向 PsLoadedModuleList 的指针  
     
   pCurrentModule = ((PMODULE\_ENTRY)(ULONG\_PTR) DriverObject->DriverSection);  
     
   if (pCurrentModule == NULL)  
      return;  
     
   // 开始扫描以查找我们的驱动  
     
   while (TRUE)  
   {  
      pCurrentModule =  (PMODULE\_ENTRY) pCurrentModule->le\_mod.Flink;  
  
      if (pCurrentModule == NULL)  
         return;  
        
      if (pCurrentModule->driver\_Name.Length > 3 &&   
         pCurrentModule->driver\_Name.Buffer)  
      {  
         if (wcscmp(pCurrentModule->driver\_Name.Buffer, L"hide.sys") == 0)  
         {  
            // 断开与我们的驱动的连接+  
              
            pCurrentModule->le\_mod.Blink->Flink = pCurrentModule->le\_mod.Flink;  
            pCurrentModule->le\_mod.Flink->Blink = pCurrentModule->le\_mod.Blink;  
        
            break;  
         }  
      }  
   }  
}  
  
// 感谢 valerino 提供代码  
// rootkit 上的文章  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
// VOID HideFromObjectDirectory()  
//   
// 从对象目录中隐藏驱动  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
VOID StealthInitializeLateMore(VOID)  
{  
    OBJECT\_ATTRIBUTES ObjectAttributes;  
    UNICODE\_STRING ucName;  
    NTSTATUS Status;  
    HANDLE hDirectory = NULL;  
    POBJECT\_DIRECTORY pDirectoryObject = NULL;  
    KIRQL OldIrql;  
    POBJECT\_HEADER ObjectHeader;  
    POBJECT\_HEADER\_NAME\_INFO NameInfo;  
    POBJECT\_DIRECTORY\_ENTRY DirectoryEntry;  
    POBJECT\_DIRECTORY\_ENTRY DirectoryEntryNext;  
    POBJECT\_DIRECTORY\_ENTRY DirectoryEntryTop;  
    ULONG Bucket = 0;  
    UNICODE\_STRING ObjectName;  
    BOOLEAN found = FALSE;  
  
    // 打开对象目录中的驱动目录  
    RtlInitUnicodeString(&ucName,L"\\\\Driver");  
    InitializeObjectAttributes(&ObjectAttributes,&ucName,OBJ\_CASE\_INSENSITIVE,NULL,NULL);  
    Status = ObOpenObjectByName(&ObjectAttributes,NULL,KernelMode,NULL,  
       0x80000000,NULL,&hDirectory);  
      
    if (!NT\_SUCCESS (Status))  
        goto \_\_exit;  
      
    // 获取指针  
    Status = ObReferenceObjectByHandle(hDirectory,FILE\_ANY\_ACCESS,NULL,  
       KernelMode,&pDirectoryObject, NULL);  
    if (!NT\_SUCCESS (Status))  
        goto \_\_exit;  
      
    // 我们提高 irql 以保护这个列表不被 kernel APC 访问  
    KeRaiseIrql(APC\_LEVEL,&OldIrql);  
      
    // 历遍对象目录  
    for (Bucket=0; Bucket<NUMBER\_HASH\_BUCKETS; Bucket++)   
    {  
        // 完成了吗？  
        if (found)  
            break;  
  
        DirectoryEntry = pDirectoryObject->HashBuckets\[Bucket\];  
        if (!DirectoryEntry)  
            continue;  
          
        // 检查我们是不是在 bucket 的顶端  
        ObjectHeader = OBJECT\_TO\_OBJECT\_HEADER( DirectoryEntry->Object );  
        NameInfo = OBJECT\_HEADER\_TO\_NAME\_INFO( ObjectHeader );  
          
        if (NameInfo != NULL)   
        {  
            ObjectName = NameInfo->Name;  
              
            // 这里我们将自己的驱动的名称与对象的名称进行比较(例如: ROOTKIT)  
            // 这个函数只是我对 wcsstr 的的扩展，不要管它了 .......  
              
            if (wcscmp(ObjectName.Buffer, L"hide") == 0)  
            {  
                // 到达顶部并获取下一个指针  
                DirectoryEntryTop = pDirectoryObject->HashBuckets\[Bucket\];  
                DirectoryEntryNext = DirectoryEntryTop->ChainLink;  
                  
                // 替代顶部  
                pDirectoryObject->HashBuckets\[Bucket\] = DirectoryEntryNext;  
                DirectoryEntryTop = pDirectoryObject->HashBuckets\[Bucket\];  
                  
                // 历遍链并通过一处替换返回项目  
                while (DirectoryEntryNext)  
                {  
                    DirectoryEntryTop->ChainLink = DirectoryEntryNext->ChainLink;  
                    DirectoryEntryTop = DirectoryEntryTop->ChainLink;  
                    DirectoryEntryNext = DirectoryEntryNext->ChainLink;  
                }  
                if (DirectoryEntryTop)  
                    DirectoryEntryTop->ChainLink = NULL;  
                  
                found = TRUE;  
                  
                // 我们可以安全退出了  
                break;  
            }  
        }   
          
        // 如果我们不在 bucket 顶端的话，检查 项目->下一个区域  
        // 对每个项目我们都要检查下一个  
        DirectoryEntryNext = DirectoryEntry->ChainLink;  
  
        while (DirectoryEntryNext)   
        {  
            ObjectHeader = OBJECT\_TO\_OBJECT\_HEADER( DirectoryEntryNext->Object );  
            NameInfo = OBJECT\_HEADER\_TO\_NAME\_INFO( ObjectHeader );  
              
            if (NameInfo != NULL)   
            {  
                ObjectName = NameInfo->Name;  
                  
                if (wcscmp(ObjectName.Buffer, L"hide") == 0)  
                {  
                    // 找到我们的对象，现在我们必须脱开与他的连接，这次简单些  
                    DirectoryEntry->ChainLink = DirectoryEntryNext->ChainLink;  
                      
                    found = TRUE;  
                      
                    // 退出  
                    break;  
                }  
            }   
              
            // walk the next entry if any  
            if (DirectoryEntry)  
            {  
                DirectoryEntry = DirectoryEntry->ChainLink;  
                DirectoryEntryNext = DirectoryEntry->ChainLink;  
            }  
            else  
            {  
                DirectoryEntryNext = NULL;  
            }  
        }  
    }  
  
    // 调回 irql  
    KeLowerIrql(OldIrql);  
\_\_exit:  
    // dereference and cleanup  
    if (pDirectoryObject)  
        ObDereferenceObject(pDirectoryObject);  
    if (hDirectory)  
        ZwClose (hDirectory);  
    return;  
}  
  
当然是用真实名称替换藏在代码里的哪些是很重要的。不管怎么样，TheMida 至少查找了 PsLoadedModuleList，而且还检查了其他什么东西(我想他检查了注册表)。我没有时间来尝试，我很快就会知道的(只要隐藏一些键就行了)。然而我必须要看看他有没有检查 SDT 是否被关联了(使用 regmon 的代码来关联一个)，而且很明显他没有，他很正常的执行了。我也能执行一个小的监视器来检视注册表和我的工具中的文件(这是可能的)。当然我无法编写一个文件系统的过滤器，只像 regmon 一样关联 SDT (这更简单一些)。然我想想，现在这些是驱动代码(带有 SDT 关联代码，这些可以被扩展成一个监视器之类的东西). 

#include <ntddk.h>  
#include <common.h>  
  
WCHAR DeviceName\[\] = L"\\\\Device\\\\antimida";  
WCHAR SymLinkName\[\] = L"\\\\DosDevices\\\\antimida";  
  
UNICODE\_STRING usDeviceName;  
UNICODE\_STRING usSymbolicLinkName;  
  
typedef struct \_DEVICE\_CONTEXT  
{  
   PDRIVER\_OBJECT  pDriverObject;         
    PDEVICE\_OBJECT  pDeviceObject;  
}  
DEVICE\_CONTEXT, \*PDEVICE\_CONTEXT, \*\*PPDEVICE\_CONTEXT;  
  
PDEVICE\_OBJECT  g\_pDeviceObject  = NULL;  
PDEVICE\_CONTEXT g\_pDeviceContext = NULL;  
  
#define FILE\_DEVICE\_ANTIMIDA 0x8000  
  
#define CODE\_RESTORE\_INFO   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x800, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_READ\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x801, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_QUERY\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x802, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_WRITE\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x803, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_ALLOC\_MEM      CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x804, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_CREATE\_THREAD   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x805, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
#define CODE\_DBG\_CONTINUE   CTL\_CODE(FILE\_DEVICE\_ANTIMIDA, 0x806, \\  
   METHOD\_BUFFERED, FILE\_ANY\_ACCESS)  
  
  
NTSTATUS DriverInitialize(PDRIVER\_OBJECT  pDriverObject,  
                    PUNICODE\_STRING pusRegistryPath);  
  
NTSTATUS DriverEntry(PDRIVER\_OBJECT  pDriverObject,  
                PUNICODE\_STRING pusRegistryPath);  
  
#ifdef ALLOC\_PRAGMA  
  
#pragma alloc\_text (INIT, DriverInitialize)  
#pragma alloc\_text (INIT, DriverEntry)  
  
#endif  
  
VOID NTAPI KeAttachProcess(IN PEPROCESS);  
  
//  
// ZwReadVirtualMemory 部分  
//  
  
typedef struct \_Input\_ZwReadVirtualMemory  
{  
   HANDLE  ProcessHandle;  
   PVOID  BaseAddress;  
   PVOID  Buffer;  
   ULONG  BufferLength;  
   PULONG  ReturnLength;  
} Input\_ZwReadVirtualMemory;  
  
NTSTATUS (\*pZwReadVirtualMemory)(IN HANDLE  ProcessHandle,  
                         IN PVOID  BaseAddress,  
                         OUT PVOID  Buffer,  
                         IN ULONG  BufferLength,  
                         OUT PULONG  ReturnLength  OPTIONAL);  
  
  
KMUTEX DumpMutex;  
  
//  
// ZwQueryVirtualMemory 部分  
//  
  
typedef enum \_MEMORY\_INFORMATION\_CLASS {  
   MemoryBasicInformation,  
   MemoryWorkingSetList,  
   MemorySectionName,  
   MemoryBasicVlmInformation  
} MEMORY\_INFORMATION\_CLASS;  
  
  
NTSTATUS (\*pZwQueryVirtualMemory)(IN HANDLE  ProcessHandle,  
                          IN PVOID  BaseAddress,  
                          IN MEMORY\_INFORMATION\_CLASS  MemoryInformationClass,  
                          OUT PVOID  MemoryInformation,  
                          IN ULONG  MemoryInformationLength,  
                          OUT PULONG  ReturnLength  OPTIONAL);  
  
typedef struct \_Input\_ZwQueryVirtualMemory  
{  
   HANDLE ProcessHandle;  
   PVOID  BaseAddress;  
   MEMORY\_INFORMATION\_CLASS  MemoryInformationClass;  
   ULONG  MemoryInformationLength;  
   PVOID  MemoryInformation;  
   PULONG  ReturnLength;  
} Input\_ZwQueryVirtualMemory;  
  
//  
// ZwWriteVirtualMemory 部分  
//  
  
NTSTATUS (\*pZwWriteVirtualMemory)(IN HANDLE  ProcessHandle,  
                         IN PVOID  BaseAddress,  
                         IN PVOID  Buffer,  
                         IN ULONG  BufferLength,  
                         OUT PULONG  ReturnLength  OPTIONAL);  
  
typedef struct \_Input\_ZwWriteVirtualMemory  
{  
   HANDLE  ProcessHandle;  
   PVOID  BaseAddress;  
   PVOID  Buffer;  
   ULONG  BufferLength;  
   PULONG  ReturnLength;  
} Input\_ZwWriteVirtualMemory;  
  
//  
// ZwAllocateVirtualMemory 部分  
//  
  
NTSTATUS (\*pZwAllocateVirtualMemory)(IN HANDLE ProcessHandle,  
                            IN OUT PVOID \*BaseAddress,  
                            IN ULONG\_PTR ZeroBits,  
                            IN OUT PSIZE\_T RegionSize,  
                            IN ULONG AllocationType,  
                            IN ULONG Protect);  
  
typedef struct \_Input\_ZwAllocateVirtualMemory  
{  
   HANDLE ProcessHandle;  
   PVOID \*BaseAddress;  
   ULONG\_PTR ZeroBits;  
   PSIZE\_T RegionSize;  
   ULONG AllocationType;  
   ULONG Protect;  
} Input\_ZwAllocateVirtualMemory;  
  
//  
// ZwCreateThread 部分  
//  
  
typedef struct \_USER\_STACK {  
   PVOID  FixedStackBase;  
   PVOID  FixedStackLimit;  
   PVOID  ExpandableStackBase;  
   PVOID  ExpandableStackLimit;  
   PVOID  ExpandableStackBottom;  
} USER\_STACK, \*PUSER\_STACK;  
  
  
NTSTATUS (\*pZwCreateThread)(OUT PHANDLE  ThreadHandle,  
                     IN ACCESS\_MASK  DesiredAccess,  
                     IN POBJECT\_ATTRIBUTES  ObjectAttributes,  
                     IN HANDLE  ProcessHandle,  
                     OUT PCLIENT\_ID  ClientId,  
                     IN PCONTEXT  ThreadContext,  
                     IN PUSER\_STACK  UserStack,  
                     IN BOOLEAN  CreateSuspended);  
  
typedef struct \_Input\_ZwCreateThread  
{  
   PHANDLE  ThreadHandle;  
   ACCESS\_MASK  DesiredAccess;  
   POBJECT\_ATTRIBUTES  ObjectAttributes;  
   HANDLE  ProcessHandle;  
   PCLIENT\_ID  ClientId;  
   PCONTEXT  ThreadContext;  
   PUSER\_STACK  UserStack;  
   BOOLEAN  CreateSuspended;  
} Input\_ZwCreateThread;  
  
//  
// ZwDebugContinue 部分  
// 我已经懒得去查找正确的声明了  
//  
  
NTSTATUS (\*pZwDebugContinue)(PVOID \*A, PVOID \*B, PVOID \*C);  
  
typedef struct \_Input\_ZwDebugContinue  
{  
   PVOID \*A;  
   PVOID \*B;  
   PVOID \*C;  
} Input\_ZwDebugContinue;  
  
  
//  
// 恢复信息结构  
//  
  
typedef struct \_RESTORE\_INFO  
{  
   DWORD ZwAllocateVirtualMemory;  
   DWORD ZwCreateThread;  
   DWORD ZwDebugContinue;  
   DWORD ZwQueryVirtualMemory;  
   DWORD ZwReadVirtualMemory;  
   DWORD ZwTerminateProcess;  
   DWORD ZwWriteVirtualMemory;  
  
   BYTE KeAttachProcessPatch\[5\];  
  
} RESTORE\_INFO, \*PRESTORE\_INFO;  
  
RESTORE\_INFO RestoreInfo;  
  
//  
// SDT 关联部分  
//   
  
extern PSERVICE\_DESCRIPTOR\_TABLE KeServiceDescriptorTable;  
  
PVOID \*KeServiceTablePointers;  
PMDL KeServiceTableMdl;  
BOOLEAN \*ServiceIsHooked;  
  
PPVOID MapServiceTable(BOOLEAN \*\*);  
VOID UnmapServiceTable(PVOID);  
  
#ifdef ALPHA  
#define FUNCTION\_PTR(\_Function) (\*(PULONG) \_Function) & 0x0000FFFF  
#else  
#define FUNCTION\_PTR(\_Function) \*(PULONG)((PUCHAR) \_Function + 1)  
#endif  
  
#define HOOK\_SYSCALL(\_Function, \_Hook, \_Orig)                           \\  
   if (!ServiceIsHooked\[FUNCTION\_PTR(\_Function)\]) {                     \\  
   \_Orig = (PVOID) InterlockedExchange((PLONG)                     \\  
   &KeServiceTablePointers\[FUNCTION\_PTR(\_Function)\], (LONG) \_Hook );   \\  
   ServiceIsHooked\[ FUNCTION\_PTR(\_Function) \] = TRUE; }  
  
  
#define UNHOOK\_SYSCALL(\_Function, \_Hook, \_Orig)                        \\  
   if (ServiceIsHooked\[FUNCTION\_PTR(\_Function)\] &&                     \\  
   KeServiceTablePointers\[FUNCTION\_PTR(\_Function) \] == (PVOID) \_Hook ) {   \\  
   InterlockedExchange((PLONG) &KeServiceTablePointers\[         \\  
   FUNCTION\_PTR(\_Function)\], (LONG) \_Orig );                  \\  
   ServiceIsHooked\[FUNCTION\_PTR(\_Function)\] = FALSE; }  
  
NTSTATUS (\*RealZwCreateFile)(OUT PHANDLE  FileHandle,  
                      IN ACCESS\_MASK  DesiredAccess,  
                      IN POBJECT\_ATTRIBUTES  ObjectAttributes,  
                      OUT PIO\_STATUS\_BLOCK  IoStatusBlock,  
                      IN PLARGE\_INTEGER  AllocationSize  OPTIONAL,  
                      IN ULONG  FileAttributes,  
                      IN ULONG  ShareAccess,  
                      IN ULONG  CreateDisposition,  
                      IN ULONG  CreateOptions,  
                      IN PVOID  EaBuffer  OPTIONAL,  
                      IN ULONG  EaLength  
                      );  
  
NTSTATUS (\*RealZwOpenKey)(OUT PHANDLE, IN ACCESS\_MASK, IN POBJECT\_ATTRIBUTES);  
  
NTSTATUS HookedZwCreateFile(OUT PHANDLE  FileHandle,  
                     IN ACCESS\_MASK  DesiredAccess,  
                     IN POBJECT\_ATTRIBUTES  ObjectAttributes,  
                     OUT PIO\_STATUS\_BLOCK  IoStatusBlock,  
                     IN PLARGE\_INTEGER  AllocationSize  OPTIONAL,  
                     IN ULONG  FileAttributes,  
                     IN ULONG  ShareAccess,  
                     IN ULONG  CreateDisposition,  
                     IN ULONG  CreateOptions,  
                     IN PVOID  EaBuffer  OPTIONAL,  
                     IN ULONG  EaLength)  
{  
   DbgPrint("%ws\\n", ObjectAttributes->ObjectName->Buffer);  
  
   return RealZwCreateFile(FileHandle, DesiredAccess, ObjectAttributes,  
      IoStatusBlock, AllocationSize, FileAttributes, ShareAccess,  
      CreateDisposition, CreateOptions, EaBuffer, EaLength);  
}  
  
NTSTATUS HookedZwOpenKey(OUT PHANDLE KeyHandle, IN ACCESS\_MASK DesiredAccess,  
                   IN POBJECT\_ATTRIBUTES ObjectAttributes)  
{  
   return RealZwOpenKey(KeyHandle, DesiredAccess, ObjectAttributes);  
}  
  
PPVOID MapServiceTable(BOOLEAN \*\*ServiceIsHooked)  
{  
   PVOID Mem;  
  
   Mem = ExAllocatePoolWithTag(0,   
      KeServiceDescriptorTable->ntoskrnl.ServiceLimit,  
      0x206B6444);  
  
   if (Mem == NULL)  
      return NULL;  
  
   \*ServiceIsHooked = (BOOLEAN \*) Mem;  
  
   memset(Mem, 0, KeServiceDescriptorTable->ntoskrnl.  
      ServiceLimit);  
  
   KeServiceTableMdl = MmCreateMdl(NULL,   
      KeServiceDescriptorTable->ntoskrnl.ServiceTable,  
      (KeServiceDescriptorTable->ntoskrnl.ServiceLimit \*   
      sizeof (POINTER)));  
  
   if (KeServiceTableMdl == NULL)  
      return NULL;  
  
   MmBuildMdlForNonPagedPool(KeServiceTableMdl);  
  
   return (PPVOID) MmMapLockedPages(KeServiceTableMdl, 0);  
}  
  
VOID UnmapServiceTable(PVOID KeServiceTablePointers)  
{  
   if (KeServiceTableMdl == NULL)  
      return;  
  
   MmUnmapLockedPages(KeServiceTablePointers,  
      KeServiceTableMdl);  
  
   ExFreePool(KeServiceTableMdl);  
}  
  
//  
// 修改 KeAttachProcess  
//  
  
VOID RebuildNtoskrnl()  
{  
   PMDL Mdl;  
   DWORD CR0Backup;  
   DWORD \*ptr;  
   BYTE \*Patch;  
  
   ptr = (DWORD \*)(2 + (DWORD) &KeAttachProcess);  
  
   ptr = (DWORD\*) \*ptr;  
  
   Patch = (BYTE \*) \*ptr;  
  
   Mdl = MmCreateMdl(0, (PVOID) Patch, 5);  
  
   if (Mdl == NULL)  
      return;  
  
   MmProbeAndLockPages(Mdl, 0, 0);  
  
   if (\*Patch == 0xE9)  
   {  
      \_\_asm  
      {  
         mov eax, cr0  
         mov CR0Backup, eax  
         and eax, 0xFFFEFFFF  
         mov cr0, eax  
      }  
  
      //  
      // 修改  
      //  
  
      memcpy(Patch, RestoreInfo.KeAttachProcessPatch, 5);  
  
      \_\_asm  
      {  
         mov eax, CR0Backup  
         mov cr0, eax  
      }  
   }  
  
   MmUnlockPages(Mdl);  
   IoFreeMdl(Mdl);  
}  
  
  
NTSTATUS ControlDispatcher(PDEVICE\_CONTEXT pDeviceContext, DWORD dwCode,  
                     BYTE \*pInput, DWORD dwInputSize,  
                     BYTE \*pOutput, DWORD dwOutputSize, DWORD \*pdwInfo)  
{  
   switch (dwCode)  
   {  
  
   case CODE\_RESTORE\_INFO:  
      {  
         RtlCopyMemory(&RestoreInfo, pInput, sizeof (RESTORE\_INFO));  
  
         pZwReadVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID, PVOID, ULONG, PULONG))  
            RestoreInfo.ZwReadVirtualMemory;  
  
         pZwQueryVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID, MEMORY\_INFORMATION\_CLASS, PVOID, ULONG,  
            PULONG)) RestoreInfo.ZwQueryVirtualMemory;  
  
         pZwWriteVirtualMemory = (NTSTATUS (\*)(HANDLE, PVOID,  
            PVOID, ULONG, PULONG))  
            RestoreInfo.ZwWriteVirtualMemory;  
  
         pZwAllocateVirtualMemory = (NTSTATUS (\*)(HANDLE,  
            PVOID \*, ULONG\_PTR, PSIZE\_T, ULONG, ULONG))  
            RestoreInfo.ZwAllocateVirtualMemory;  
  
         pZwCreateThread = (NTSTATUS (\*)(PHANDLE, ACCESS\_MASK,  
            POBJECT\_ATTRIBUTES, HANDLE, PCLIENT\_ID,  
            PCONTEXT, PUSER\_STACK, BOOLEAN))  
            RestoreInfo.ZwCreateThread;  
  
         pZwDebugContinue = (NTSTATUS (\*)(PVOID \*A, PVOID \*B, PVOID \*C))  
            RestoreInfo.ZwDebugContinue;  
  
         RebuildNtoskrnl();  
  
         break;  
      }  
  
   case CODE\_READ\_MEM:  
      {  
         Input\_ZwReadVirtualMemory Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwReadVirtualMemory));  
  
         return pZwReadVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.Buffer, Input.BufferLength, Input.ReturnLength);  
      }  
  
   case CODE\_QUERY\_MEM:  
      {  
         Input\_ZwQueryVirtualMemory Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwQueryVirtualMemory));  
  
         return pZwQueryVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.MemoryInformationClass, Input.MemoryInformation,  
            Input.MemoryInformationLength, Input.ReturnLength);  
      }  
  
   case CODE\_WRITE\_MEM:  
      {  
         Input\_ZwWriteVirtualMemory Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwWriteVirtualMemory));  
  
         return pZwWriteVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.Buffer, Input.BufferLength, Input.ReturnLength);  
      }  
  
   case CODE\_ALLOC\_MEM:  
      {  
         Input\_ZwAllocateVirtualMemory Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwAllocateVirtualMemory));  
  
         return pZwAllocateVirtualMemory(Input.ProcessHandle, Input.BaseAddress,  
            Input.ZeroBits, Input.RegionSize, Input.AllocationType, Input.Protect);  
      }  
  
   case CODE\_CREATE\_THREAD:  
      {  
         Input\_ZwCreateThread Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwCreateThread));  
  
         return pZwCreateThread(Input.ThreadHandle, Input.DesiredAccess,  
            Input.ObjectAttributes, Input.ProcessHandle, Input.ClientId,   
            Input.ThreadContext, Input.UserStack, Input.CreateSuspended);  
      }  
  
   case CODE\_DBG\_CONTINUE:  
      {  
         Input\_ZwDebugContinue Input;  
  
         RtlCopyMemory(&Input, pInput, sizeof (Input\_ZwDebugContinue));  
  
         return pZwDebugContinue(Input.A, Input.B, Input.C);  
      }  
  
        
   default:  
       return STATUS\_INVALID\_PARAMETER;  
   }  
      
   return STATUS\_SUCCESS;  
}  
  
NTSTATUS DeviceDispatcher(PDEVICE\_CONTEXT pDeviceContext, PIRP pIrp)  
{  
    PIO\_STACK\_LOCATION pisl;  
    DWORD dwInfo = 0;  
    NTSTATUS ns = STATUS\_NOT\_IMPLEMENTED;  
  
    pisl = IoGetCurrentIrpStackLocation(pIrp);  
  
    switch (pisl->MajorFunction)  
   {  
     
   case IRP\_MJ\_CREATE:  
   case IRP\_MJ\_CLEANUP:  
   case IRP\_MJ\_CLOSE:  
      {  
         ns = STATUS\_SUCCESS;  
            break;  
      }  
  
   case IRP\_MJ\_DEVICE\_CONTROL:  
      {  
         MUTEX\_ACQUIRE(DumpMutex);  
  
         ns = ControlDispatcher(pDeviceContext,  
            pisl->Parameters.DeviceIoControl.IoControlCode,  
            (BYTE \*) pIrp->AssociatedIrp.SystemBuffer,  
            pisl->Parameters.DeviceIoControl.InputBufferLength,  
            (BYTE \*) pIrp->AssociatedIrp.SystemBuffer,  
            pisl->Parameters.DeviceIoControl.OutputBufferLength,  
            &dwInfo);  
  
         MUTEX\_RELEASE(DumpMutex);  
           
         break;  
      }  
   }  
  
    pIrp->IoStatus.Status = ns;  
    pIrp->IoStatus.Information = dwInfo;  
  
    IoCompleteRequest (pIrp, IO\_NO\_INCREMENT);  
  
    return ns;  
}  
  
NTSTATUS DriverDispatcher(PDEVICE\_OBJECT pDeviceObject, PIRP pIrp)  
{  
   return (pDeviceObject == g\_pDeviceObject ?   
      DeviceDispatcher(g\_pDeviceContext, pIrp)  
      : STATUS\_INVALID\_PARAMETER\_1);  
}  
  
  
VOID DriverUnload(PDRIVER\_OBJECT pDriverObject)  
{  
   UNHOOK\_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
   UNHOOK\_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
   UnmapServiceTable(KeServiceTablePointers);  
  
    IoDeleteSymbolicLink(&usSymbolicLinkName);  
    IoDeleteDevice(pDriverObject->DeviceObject);  
}  
  
NTSTATUS DriverInitialize(PDRIVER\_OBJECT pDriverObject,  
                    PUNICODE\_STRING pusRegistryPath)  
{  
    PDEVICE\_OBJECT pDeviceObject = NULL;  
    NTSTATUS ns = STATUS\_DEVICE\_CONFIGURATION\_ERROR;  
      
    RtlInitUnicodeString(&usDeviceName, DeviceName);  
    RtlInitUnicodeString(&usSymbolicLinkName, SymLinkName);  
  
    if ((ns = IoCreateDevice(pDriverObject, sizeof (DEVICE\_CONTEXT),  
      &usDeviceName, FILE\_DEVICE\_ANTIMIDA, 0, FALSE,  
      &pDeviceObject)) == STATUS\_SUCCESS)  
   {  
        if ((ns = IoCreateSymbolicLink(&usSymbolicLinkName,   
         &usDeviceName)) == STATUS\_SUCCESS)  
      {  
            g\_pDeviceObject  = pDeviceObject;  
            g\_pDeviceContext = pDeviceObject->DeviceExtension;  
  
            g\_pDeviceContext->pDriverObject = pDriverObject;  
            g\_pDeviceContext->pDeviceObject = pDeviceObject;  
  
      }  
      else  
      {  
         IoDeleteDevice(pDeviceObject);  
      }  
   }  
     
   return ns;  
}  
  
NTSTATUS DriverEntry(PDRIVER\_OBJECT pDriverObject,  
                PUNICODE\_STRING pusRegistryPath)  
{  
   PDRIVER\_DISPATCH \*ppdd;  
    NTSTATUS ns = STATUS\_DEVICE\_CONFIGURATION\_ERROR;  
  
    if ((ns = DriverInitialize(pDriverObject, pusRegistryPath)) == STATUS\_SUCCESS)  
   {  
        ppdd = pDriverObject->MajorFunction;  
  
        ppdd\[IRP\_MJ\_CREATE                  \] =  
        ppdd\[IRP\_MJ\_CREATE\_NAMED\_PIPE       \] =  
        ppdd\[IRP\_MJ\_CLOSE                   \] =  
        ppdd\[IRP\_MJ\_READ                    \] =  
        ppdd\[IRP\_MJ\_WRITE                   \] =  
        ppdd\[IRP\_MJ\_QUERY\_INFORMATION       \] =  
        ppdd\[IRP\_MJ\_SET\_INFORMATION         \] =  
        ppdd\[IRP\_MJ\_QUERY\_EA                \] =  
        ppdd\[IRP\_MJ\_SET\_EA                  \] =  
        ppdd\[IRP\_MJ\_FLUSH\_BUFFERS           \] =  
        ppdd\[IRP\_MJ\_QUERY\_VOLUME\_INFORMATION\] =  
        ppdd\[IRP\_MJ\_SET\_VOLUME\_INFORMATION  \] =  
        ppdd\[IRP\_MJ\_DIRECTORY\_CONTROL       \] =  
        ppdd\[IRP\_MJ\_FILE\_SYSTEM\_CONTROL     \] =  
        ppdd\[IRP\_MJ\_DEVICE\_CONTROL          \] =  
        ppdd\[IRP\_MJ\_INTERNAL\_DEVICE\_CONTROL \] =  
        ppdd\[IRP\_MJ\_SHUTDOWN                \] =  
        ppdd\[IRP\_MJ\_LOCK\_CONTROL            \] =  
        ppdd\[IRP\_MJ\_CLEANUP                 \] =  
        ppdd\[IRP\_MJ\_CREATE\_MAILSLOT         \] =  
        ppdd\[IRP\_MJ\_QUERY\_SECURITY          \] =  
        ppdd\[IRP\_MJ\_SET\_SECURITY            \] =  
        ppdd\[IRP\_MJ\_POWER                   \] =  
        ppdd\[IRP\_MJ\_SYSTEM\_CONTROL          \] =  
        ppdd\[IRP\_MJ\_DEVICE\_CHANGE           \] =  
        ppdd\[IRP\_MJ\_QUERY\_QUOTA             \] =  
        ppdd\[IRP\_MJ\_SET\_QUOTA               \] =  
        ppdd\[IRP\_MJ\_PNP                     \] = DriverDispatcher;  
        pDriverObject->DriverUnload           = DriverUnload;  
  
      MUTEX\_INIT(DumpMutex);  
  
      KeServiceTablePointers = MapServiceTable(&ServiceIsHooked);  
  
      HOOK\_SYSCALL(ZwCreateFile, HookedZwCreateFile, RealZwCreateFile);  
      HOOK\_SYSCALL(ZwOpenKey, HookedZwOpenKey, RealZwOpenKey);  
  
   }  
  
   return ns;  
}  
  
好了，我现在很累了。没有什么要解释的了，如果你有疑惑的话，google 在那里等着你。这项工程的继续取决于我得到的信息。我对逆向 TheMida 没有兴趣。  
  
  
  
译者：我现在同样也是累得不行了。。。好长的程序代码，不过最后还是发现有词还是不会译。郁闷。。。 

 4dd466d0e1e0cc123afbe504da2462ca