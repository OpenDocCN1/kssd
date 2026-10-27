
```
VOID PatchKeFlushCurrentTb(PVOID lpNewNtoskrnlAddress, PVOID lpNtoskrnlAddress)

{

  /*

  mov     edi, offset KeFlushCurrentTb

  mov     esi, offset byte_477D57

  mov     ecx, XXX

  rep movsb

  */

  IMAGE_DOS_HEADER *lpDosHeader = (IMAGE_DOS_HEADER*)lpNewNtoskrnlAddress;

  IMAGE_NT_HEADERS *lpNtHeader = (IMAGE_NT_HEADERS *)((PCHAR)lpDosHeader + lpDosHeader->e_lfanew);

  IMAGE_BASE_RELOCATION *lpRelocateTable = (IMAGE_BASE_RELOCATION*)((PCHAR)lpDosHeader + lpNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);



  while (lpRelocateTable->SizeOfBlock)

  {

    ULONG NumberOfItems = (lpRelocateTable->SizeOfBlock - sizeof(IMAGE_BASE_RELOCATION)) / sizeof(USHORT);

    USHORT  *lpItem = (USHORT*)((PCHAR)lpRelocateTable + sizeof(IMAGE_BASE_RELOCATION));

    ULONG i;



    for (i = 0; i < NumberOfItems - 1; i++)

    {

      if ((lpItem[i] >> 12) == IMAGE_REL_BASED_HIGHLOW && (lpItem[i + 1] >> 12) == IMAGE_REL_BASED_HIGHLOW)

      {

        ULONG *lpFixAddress1 = (ULONG*)((PCHAR)lpDosHeader + lpRelocateTable->VirtualAddress + (lpItem[i] & 0x0FFF));

        ULONG *lpFixAddress2 = (ULONG*)((PCHAR)lpDosHeader + lpRelocateTable->VirtualAddress + (lpItem[i + 1] & 0x0FFF));



        if ((ULONG)lpFixAddress2 - (ULONG)lpFixAddress1 == 5)

        {

          if (*((PUCHAR)lpFixAddress1 - 1) == 0xBF && *((PUCHAR)lpFixAddress2 - 1) == 0xBE)

          {

            PUCHAR lpCheckBytes = (PUCHAR)lpFixAddress2 + sizeof(PVOID);



            if (lpCheckBytes[0] == 0xB9 && lpCheckBytes[5] == 0xF3 && lpCheckBytes[6] == 0xA4)

            {

              PUCHAR lpPatchAddress = (PUCHAR)*lpFixAddress1;



              lpPatchAddress[0] = 0xE9;

              *(ULONG*)&lpPatchAddress[1] = (ULONG)lpNtoskrnlAddress - (ULONG)lpNewNtoskrnlAddress - 5;

              return;

            }

          }

        }

      }

    }

    lpRelocateTable = (IMAGE_BASE_RELOCATION *)((PCHAR)lpRelocateTable + lpRelocateTable->SizeOfBlock);

  }

  return;

}
```

好了，没事儿了，新的NT内核没问题了，全部搞定，如果仍发现自己解决不了的问题可以联系我，最后的`DriverEntry`代码：

```
NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING  RegistryPath)

{

  PVOID lpHookKiFastCallEntryAddress;

  UCHAR HookCode[7];



  DbgPrint("Driver Load.\n");

  InitializePsLoadedModuleList(DriverObject);

  g_lpNtoskrnlAddress = KeGetModuleHandle(PsLoadedModuleList, "ntoskrnl.exe");

  g_lpNewNtoskrnlAddress = ReloadNtModule(PsLoadedModuleList);

  g_KeServiceTable = (PVOID*)BuildKeServiceTable(g_lpNewNtoskrnlAddress, g_lpNtoskrnlAddress);

  KeFixReloc2(g_lpNewNtoskrnlAddress, g_lpNtoskrnlAddress);

  PatchKeFlushCurrentTb(g_lpNewNtoskrnlAddress, g_lpNtoskrnlAddress);

  KeInsertPsLoadedModuleList(PsLoadedModuleList, g_lpNewNtoskrnlAddress, g_lpNtoskrnlAddress);

  lpHookKiFastCallEntryAddress = FindHookKiFastCallEntryAddress(GetKiFastCallEntryAddress());

  HookCode[0] = 0xE8;

  *(ULONG*)&HookCode[1] = (ULONG_PTR)_MyKiFastCallEntryFrame - (ULONG_PTR)lpHookKiFastCallEntryAddress - 5;

  HookCode[5] = 0x90;

  HookCode[6] = 0x90;

  RtlCopyMemoryEx(lpHookKiFastCallEntryAddress, HookCode, sizeof(HookCode));



  DriverObject->DriverUnload = DriverUnload;

  return STATUS_SUCCESS;

}
```

这一部分告一段落，接下来继续重载`win32k`，搞定SHADOW SSDT。

最后，为啥PCHunter不支持Wwindows 8.1 update 1呢？现在的最新版1.32运行不到一分钟就蓝屏，原因是触发了PatchGuard，望修复，没有ARK用真心难受。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [00.jpg](javascript:void(0);)（14.54kb，14次下载）
- [01.jpg](javascript:void(0);)（25.44kb，7次下载）
- [4.重载内核之蓝屏与崩溃处理.doc](javascript:void(0);)（73.50kb，287次下载）
