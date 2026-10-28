
## 内核部分

```c++
//R0的
extern "C" //声明API 直接调用
NTSTATUS NTAPI ZwQuerySystemInformation(
__in SYSTEM_INFORMATION_CLASS SystemInformationClass,
__out_bcount_opt(SystemInformationLength) PVOID SystemInformation,
__in ULONG SystemInformationLength,
__out_opt PULONG ReturnLength
);
NTSTATUS EnumModuleEx(PMODULE_INFO ModuleBuffer, bool CountOnly, PULONG ModuleCount)
{
NTSTATUS Status;
PRTL_PROCESS_MODULES Modules;
PRTL_PROCESS_MODULE_INFORMATION ModuleInfo;
PVOID Buffer;
ULONG BufferSize = 4096;
ULONG ReturnLength;
ULONG i;
ULONG Count = 0;
KdPrint(("[test] EnumModuleEx: CountOnly=%d\n", CountOnly));
retry:
Buffer = ExAllocatePoolWithTag(NonPagedPool, BufferSize, 'MDLE');
if (!Buffer) {
return STATUS_NO_MEMORY;
}
Status = ZwQuerySystemInformation(SystemModuleInformation,
Buffer,
BufferSize,
&ReturnLength
);
if (Status == STATUS_INFO_LENGTH_MISMATCH) {
ExFreePool(Buffer);
BufferSize = ReturnLength;
goto retry;
}
if (!NT_SUCCESS(Status)) {
ExFreePool(Buffer);
return Status;
}
Modules = (PRTL_PROCESS_MODULES)Buffer;
Count = Modules->NumberOfModules;
if (CountOnly) {
*ModuleCount = Count;
ExFreePool(Buffer);
return STATUS_SUCCESS;
}
// 填充模块信息
for (i = 0, ModuleInfo = &(Modules->Modules[0]);
i < Modules->NumberOfModules && ModuleBuffer;
i++, ModuleInfo++) {
__try {
// 清空结构体
RtlZeroMemory(&ModuleBuffer[i], sizeof(MODULE_INFO));
// 提取模块名称（从完整路径中提取文件名）
CHAR* fileName = (CHAR*)ModuleInfo->FullPathName + ModuleInfo->OffsetToFileName;
RtlCopyMemory(ModuleBuffer[i].Name, fileName,
min(strlen(fileName), sizeof(ModuleBuffer[i].Name) - 1));
// 完整路径
RtlCopyMemory(ModuleBuffer[i].FullPath, ModuleInfo->FullPathName,
min(strlen((CHAR*)ModuleInfo->FullPathName), sizeof(ModuleBuffer[i].FullPath) - 1));
// 基地址和大小
ModuleBuffer[i].ImageBase = ModuleInfo->ImageBase;
ModuleBuffer[i].ImageSize = ModuleInfo->ImageSize;
ModuleBuffer[i].LoadOrderIndex = ModuleInfo->LoadOrderIndex;
ModuleBuffer[i].LoadCount = ModuleInfo->LoadCount;
KdPrint(("[test] Module[%d]: Name=%s, Base=%p, Size=0x%X\n",
i, ModuleBuffer[i].Name, ModuleBuffer[i].ImageBase, ModuleBuffer[i].ImageSize));
}
__except (EXCEPTION_EXECUTE_HANDLER) {
KdPrint(("[test] EnumModuleEx: Exception processing module %d\n", i));
continue;
}
}
*ModuleCount = Count;
ExFreePool(Buffer);
return STATUS_SUCCESS;
}
//DispatchDeviceControl中
case CTL_ENUM_MODULE_COUNT:
{
ULONG moduleCount = 0;
status = EnumModuleEx(NULL, TRUE, &moduleCount);
if (NT_SUCCESS(status)) {
*(PULONG)Irp->AssociatedIrp.SystemBuffer = moduleCount;
info = sizeof(ULONG);
KdPrint(("[test] CTL_ENUM_MODULE_COUNT: %d 个模块\n", moduleCount));
}
}
break;
case CTL_ENUM_MODULE:
{
ULONG moduleCount = 0;
status = EnumModuleEx((PMODULE_INFO)Irp->AssociatedIrp.SystemBuffer,
FALSE, &moduleCount);
if (NT_SUCCESS(status)) {
info = moduleCount * sizeof(MODULE_INFO);
KdPrint(("[test] CTL_ENUM_MODULE: 返回 %d 个模块信息\n", moduleCount));
}
}
```
## 用户层部分

```c++
//R3的
// 获取内核模块数量
DWORD ArkR3::ModuleGetCount()
{
DWORD dwBytes = 0;
DWORD dwEntryNum = 0;
DeviceIoControl(m_hDriver, CTL_ENUM_MODULE_COUNT, NULL, NULL, &dwEntryNum, sizeof(DWORD), &dwBytes, NULL);
return dwEntryNum;
}
//获取内核模块信息
std::vector<MODULE_INFO> ArkR3::ModuleGetVec(DWORD moduleCount)
{
DWORD dwRetBytes;
DWORD dwBufferSize = sizeof(MODULE_INFO) * moduleCount;
PMODULE_INFO pEntryInfo = (PMODULE_INFO)malloc(dwBufferSize);
BOOL bResult = DeviceIoControl(m_hDriver, CTL_ENUM_MODULE, NULL, NULL, pEntryInfo, dwBufferSize, &dwRetBytes, NULL);
MoudleVec_.clear();
DWORD Count = 0;
if (bResult) {
Count = dwRetBytes / sizeof(MODULE_INFO);
for (DWORD i = 0; i < Count; i++) {
MODULE_INFO mInfo = pEntryInfo[i];
MoudleVec_.emplace_back(mInfo);
Log("ModuleGetVec 模块[%d]: 名称=%s, 基地址=%p, 大小=0x%X, 路径=%s\n",
i, mInfo.Name, mInfo.ImageBase, mInfo.ImageSize, mInfo.FullPath);
}
}
free(pEntryInfo);
return MoudleVec_;
}
```

## XP下的相关结构体

![img](upload/attach/202507/1003898_M5Z62WGCEUKFNA9.png)

![img](upload/attach/202507/1003898_WRJQ4CRRX22MKKF.png)

![img](upload/attach/202507/1003898_SVRPYF3RP9HWQ6M.png)

![img](upload/attach/202507/1003898_222Z7UXVXQUXAKM.png)
