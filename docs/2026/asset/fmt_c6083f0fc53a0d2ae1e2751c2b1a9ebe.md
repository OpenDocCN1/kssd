
```c
/\* 一下是获取SSDT表函数索引 \*/
#define PE\_ERROR\_VALUE (ULONG)-1
NTSTATUS InitReadFile(PWCHAR Path);/\* 初始 将Path读入内存中 \*/
int GetExportSsdtIndex(const char\* ExportName);/\* 获取导出表中 函数索引 \*/
ULONG GetExportOffset(const unsigned char\* FileData, ULONG FileSize, const char\* ExportName);/\* 获取导出表中 函数地址 \*/
ULONG RvaToOffset(PIMAGE\_NT\_HEADERS pnth, ULONG Rva, ULONG FileSize);/\* 将Rva转为文件偏移 \*/
void\* RtlAllocateMemory(bool InZeroMemory, SIZE\_T InSize);/\* 分配内存 \*/
void RtlFreeMemory(void\* InPointer);/\* 释放内存 \*/
/\* 初始 将Path读入内存中 \*/
NTSTATUS InitReadFile(PWCHAR Path)
{
UNICODE\_STRING FileName;
OBJECT\_ATTRIBUTES ObjectAttributes;
RtlInitUnicodeString(&FileName, Path);/\* L"\\SystemRoot\\system32\\ntdll.dll" \*/
InitializeObjectAttributes(&ObjectAttributes, &FileName,
OBJ\_CASE\_INSENSITIVE | OBJ\_KERNEL\_HANDLE,
NULL, NULL);
if (KeGetCurrentIrql() != PASSIVE\_LEVEL)
{
#ifdef \_DEBUG
DbgPrint("[TITANHIDE] KeGetCurrentIrql != PASSIVE\_LEVEL!\n");
#endif
return STATUS\_UNSUCCESSFUL;
}
HANDLE FileHandle;
IO\_STATUS\_BLOCK IoStatusBlock;
NTSTATUS NtStatus = ZwCreateFile(&FileHandle,
GENERIC\_READ,
&ObjectAttributes,
&IoStatusBlock, NULL,
FILE\_ATTRIBUTE\_NORMAL,
FILE\_SHARE\_READ,
FILE\_OPEN,
FILE\_SYNCHRONOUS\_IO\_NONALERT,
NULL, 0);
if (NT\_SUCCESS(NtStatus))
{
FILE\_STANDARD\_INFORMATION StandardInformation = { 0 };
NtStatus = ZwQueryInformationFile(FileHandle, &IoStatusBlock, &StandardInformation, sizeof(FILE\_STANDARD\_INFORMATION), FileStandardInformation);
if (NT\_SUCCESS(NtStatus))
{
g\_FileSize = StandardInformation.EndOfFile.LowPart;
kprintf("[TITANHIDE] FileSize of ntdll.dll is %08X!\r\n", StandardInformation.EndOfFile.LowPart);
g\_FileData = (unsigned char\*)RtlAllocateMemory(true, g\_FileSize);
LARGE\_INTEGER ByteOffset;
ByteOffset.LowPart = ByteOffset.HighPart = 0;
NtStatus = ZwReadFile(FileHandle,
NULL, NULL, NULL,
&IoStatusBlock,
g\_FileData,
g\_FileSize,
&ByteOffset, NULL);
if (!NT\_SUCCESS(NtStatus))
{
RtlFreeMemory(g\_FileData);
kprintf("[TITANHIDE] ZwReadFile failed with status %08X...\r\n", NtStatus);
}
}
else
kprintf("[TITANHIDE] ZwQueryInformationFile failed with status %08X...\r\n", NtStatus);
ZwClose(FileHandle);
}
else
kprintf("[TITANHIDE] ZwCreateFile failed with status %08X...\r\n", NtStatus);
return NtStatus;
}
/\* 获取导出表中 SSDT表 ShadowSSDT表 函数索引 \*/
int GetExportSsdtIndex(const char\* ExportName)
{
ULONG\_PTR ExportOffset = GetExportOffset(g\_FileData, g\_FileSize, ExportName);
if (ExportOffset == PE\_ERROR\_VALUE)
return -1;
int SsdtOffset = -1;
unsigned char\* ExportData = g\_FileData + ExportOffset;
for (int i = 0; i < 32 && ExportOffset + i < g\_FileSize; i++) {
if (ExportData[i] == 0xC2 || ExportData[i] == 0xC3) //RET
break;
if (ExportData[i] == 0xB8) { //mov eax,X
SsdtOffset = \*(int\*)(ExportData + i + 1);
break;
}
}
if (SsdtOffset == -1) {
kprintf("[TITANHIDE] SSDT Offset for %s not found...\r\n", ExportName);
}
if (SsdtOffset >= 0x1000) {/\* ShadowSSDT表 函数索引减0x1000使用 \*/
SsdtOffset -= 0x1000;
}
return SsdtOffset;
}
/\* 获取导出表中 函数地址 \*/
ULONG GetExportOffset(const unsigned char\* FileData, ULONG FileSize, const char\* ExportName)
{
//Verify DOS Header
PIMAGE\_DOS\_HEADER pdh = (PIMAGE\_DOS\_HEADER)FileData;
if (pdh->e\_magic != IMAGE\_DOS\_SIGNATURE)
{
kprintf("[TITANHIDE] Invalid IMAGE\_DOS\_SIGNATURE!\r\n");
return PE\_ERROR\_VALUE;
}
//Verify PE Header
PIMAGE\_NT\_HEADERS pnth = (PIMAGE\_NT\_HEADERS)(FileData + pdh->e\_lfanew);
if (pnth->Signature != IMAGE\_NT\_SIGNATURE)
{
kprintf("[TITANHIDE] Invalid IMAGE\_NT\_SIGNATURE!\r\n");
return PE\_ERROR\_VALUE;
}
//Verify Export Directory
PIMAGE\_DATA\_DIRECTORY pdd = NULL;
if (pnth->OptionalHeader.Magic == IMAGE\_NT\_OPTIONAL\_HDR64\_MAGIC)
pdd = ((PIMAGE\_NT\_HEADERS64)pnth)->OptionalHeader.DataDirectory;
else
pdd = ((PIMAGE\_NT\_HEADERS32)pnth)->OptionalHeader.DataDirectory;
ULONG ExportDirRva = pdd[IMAGE\_DIRECTORY\_ENTRY\_EXPORT].VirtualAddress;
ULONG ExportDirSize = pdd[IMAGE\_DIRECTORY\_ENTRY\_EXPORT].Size;
ULONG ExportDirOffset = RvaToOffset(pnth, ExportDirRva, FileSize);
if (ExportDirOffset == PE\_ERROR\_VALUE)
{
kprintf("[TITANHIDE] Invalid Export Directory!\r\n");
return PE\_ERROR\_VALUE;
}
//Read Export Directory
PIMAGE\_EXPORT\_DIRECTORY ExportDir = (PIMAGE\_EXPORT\_DIRECTORY)(FileData + ExportDirOffset);
ULONG NumberOfNames = ExportDir->NumberOfNames;
ULONG AddressOfFunctionsOffset = RvaToOffset(pnth, ExportDir->AddressOfFunctions, FileSize);
ULONG AddressOfNameOrdinalsOffset = RvaToOffset(pnth, ExportDir->AddressOfNameOrdinals, FileSize);
ULONG AddressOfNamesOffset = RvaToOffset(pnth, ExportDir->AddressOfNames, FileSize);
if (AddressOfFunctionsOffset == PE\_ERROR\_VALUE ||
AddressOfNameOrdinalsOffset == PE\_ERROR\_VALUE ||
AddressOfNamesOffset == PE\_ERROR\_VALUE)
{
kprintf("[TITANHIDE] Invalid Export Directory Contents!\r\n");
return PE\_ERROR\_VALUE;
}
ULONG\* AddressOfFunctions = (ULONG\*)(FileData + AddressOfFunctionsOffset);
USHORT\* AddressOfNameOrdinals = (USHORT\*)(FileData + AddressOfNameOrdinalsOffset);
ULONG\* AddressOfNames = (ULONG\*)(FileData + AddressOfNamesOffset);
//Find Export
ULONG ExportOffset = PE\_ERROR\_VALUE;
for (ULONG i = 0; i < NumberOfNames; i++)
{
ULONG CurrentNameOffset = RvaToOffset(pnth, AddressOfNames[i], FileSize);
if (CurrentNameOffset == PE\_ERROR\_VALUE)
continue;
const char\* CurrentName = (const char\*)(FileData + CurrentNameOffset);
ULONG CurrentFunctionRva = AddressOfFunctions[AddressOfNameOrdinals[i]];
if (CurrentFunctionRva >= ExportDirRva && CurrentFunctionRva < ExportDirRva + ExportDirSize)
continue; //we ignore forwarded exports
if (!strcmp(CurrentName, ExportName)) //compare the export name to the requested export
{
ExportOffset = RvaToOffset(pnth, CurrentFunctionRva, FileSize);
break;
}
}
if (ExportOffset == PE\_ERROR\_VALUE)
{
kprintf("[TITANHIDE] Export %s not found in export table!\r\n", ExportName);
}
return ExportOffset;
}
/\* 将Rva转为文件偏移 \*/
ULONG RvaToOffset(PIMAGE\_NT\_HEADERS pnth, ULONG Rva, ULONG FileSize)
{
PIMAGE\_SECTION\_HEADER psh = IMAGE\_FIRST\_SECTION(pnth);
USHORT NumberOfSections = pnth->FileHeader.NumberOfSections;
for (int i = 0; i < NumberOfSections; i++)
{
if (psh->VirtualAddress <= Rva)
{
if ((psh->VirtualAddress + psh->Misc.VirtualSize) > Rva)
{
Rva -= psh->VirtualAddress;
Rva += psh->PointerToRawData;
return Rva < FileSize ? Rva : PE\_ERROR\_VALUE;
}
}
psh++;
}
return PE\_ERROR\_VALUE;
}
/\* 分配内存 \*/
void\* RtlAllocateMemory(bool InZeroMemory, SIZE\_T InSize) {
void\* Result = ExAllocatePoolWithTag(NonPagedPool, InSize, 'HIDE');
if (InZeroMemory && (Result != NULL))
RtlZeroMemory(Result, InSize);
return Result;
}
/\* 释放内存 \*/
void RtlFreeMemory(void\* InPointer) {
ExFreePool(InPointer);
}
```

### 获取GUI相关的函数地址，还需附加GUI进程

只有在有 GUI 的线程当中，`win32k.sys` 的内存才可以被访问。
调用`KeAttachProcess`将当前线程附加到GUI进程的地址空间
`KeAttachProcess`需要用到被附加进程的`PEPROCESS`

#### 获取进程 `PEPROCESS`

目前最好用的是，遍历 PID 对比进程名
其他方式在高版本可能不再兼容
这个直接贴代码

```c
/\* 通过进程名 获取进程EPROCESS \*/
NTSTATUS LookupProcessByName(IN PCHAR pcProcessName, OUT PEPROCESS\* pEprocess) {
PEPROCESS pCurrentEprocess = NULL;
ULONG uPid = 4;
PCSZ lpcProcessName = NULL;
ULONG uLength = 0; /\* 进程名长度 \*/
if (!ARGUMENT\_PRESENT(pcProcessName) || !ARGUMENT\_PRESENT(pEprocess)) {
return STATUS\_INVALID\_PARAMETER;
}
if (KeGetCurrentIrql() > PASSIVE\_LEVEL) {
return STATUS\_UNSUCCESSFUL;
}
uLength = strlen(pcProcessName);
while (uPid < 0x186a0) {
if (NT\_SUCCESS(PsLookupProcessByProcessId((HANDLE)uPid, &pCurrentEprocess)))
{
lpcProcessName = (PCSZ)PsGetProcessImageFileName(pCurrentEprocess);
ObDereferenceObject(pCurrentEprocess);
if (\_strnicmp(pcProcessName, lpcProcessName, uLength) == 0) {
\*pEprocess = pCurrentEprocess;
/\*DbgBreakPoint();\*/
return STATUS\_SUCCESS;
}
}
uPid += 4;
}
return STATUS\_FLT\_INSTANCE\_NOT\_FOUND;
}
```

### 获取函数地址

用法示例
