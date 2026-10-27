
# Vmware 嵌套 VT EPT 性能暴降解决方案

## 前言

近段时间在 Vmware 中使用 VT EPT 技术时发生性能爆降甚至鼠标不能动的现象。检查 VM exit 事件，无任何问题。设置 Vmware 核心数 16 发现比核心数 4 还卡，设置核心数 1 发现稳定运行无卡顿。觉得可能是 EPT 多次寻址问题。HyperPlatform 项目将在每个 CPU 设置不同的 EPT 目录。

## 解决方案

我将代码改为以下：

```
EptData\* g\_EptData = 0;
// Builds EPT, allocates pre-allocated entires, initializes and returns EptData
\_Use\_decl\_annotations\_ EptData\* EptInitialization() {
PAGED\_CODE()
if (g\_EptData != 0)//增加
return g\_EptData;//增加
static const auto kEptPageWalkLevel = 4ul;
// Allocate ept\_data
const auto ept\_data = static\_cast<EptData\*>(ExAllocatePoolWithTag(
NonPagedPool, sizeof(EptData), kHyperPlatformCommonPoolTag));
if (!ept\_data) {
return nullptr;
}
RtlZeroMemory(ept\_data, sizeof(EptData));
//// Allocate EptPointer
//const auto ept\_poiner = reinterpret\_cast<EptPointer\*>(ExAllocatePoolWithTag(
// NonPagedPool, PAGE\_SIZE, kHyperPlatformCommonPoolTag));
//if (!ept\_poiner) {
// ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
// return nullptr;
//}
//RtlZeroMemory(ept\_poiner, PAGE\_SIZE);
// Allocate EPT\_PML4 and initialize EptPointer
const auto ept\_pml4 = static\_cast<EptCommonEntry\*>(
ExAllocatePoolWithTag(NonPagedPool, PAGE\_SIZE, kHyperPlatformCommonPoolTag));
if (!ept\_pml4) {
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
return nullptr;
}
RtlZeroMemory(ept\_pml4, PAGE\_SIZE);
ept\_data->ept\_pointer.all = 0;
ept\_data->ept\_pointer.fields.memory\_type =
static\_cast<ULONG64>(EptpGetMemoryType(UtilPaFromVa(ept\_pml4)));
ept\_data->ept\_pointer.fields.page\_walk\_length = kEptPageWalkLevel - 1;
ept\_data->ept\_pointer.fields.pml4\_address =
UtilPfnFromPa(UtilPaFromVa(ept\_pml4));
// Initialize all EPT entries for all physical memory pages
const auto pm\_ranges = UtilGetPhysicalMemoryRanges();
for (auto run\_index = 0ul; run\_index < pm\_ranges->number\_of\_runs;
++run\_index) {
const auto run = &pm\_ranges->run[run\_index];
const auto base\_addr = run->base\_page \* PAGE\_SIZE;
for (auto page\_index = 0ull; page\_index < run->page\_count; ++page\_index) {
const auto indexed\_addr = base\_addr + page\_index \* PAGE\_SIZE;
const auto ept\_pt\_entry =
EptpConstructTables(ept\_pml4, 4, indexed\_addr, nullptr);
if (!ept\_pt\_entry) {
EptpDestructTables(ept\_pml4, 4);
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
return nullptr;
}
}
}
// Initialize an EPT entry for APIC\_BASE. It is required to allocated it now
// for some reasons, or else, system hangs.
const Ia32ApicBaseMsr apic\_msr = { UtilReadMsr64(Msr::kIa32ApicBase) };
if (!EptpConstructTables(ept\_pml4, 4, apic\_msr.fields.apic\_base \* PAGE\_SIZE,
nullptr)) {
EptpDestructTables(ept\_pml4, 4);
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
return nullptr;
}
// Allocate preallocated\_entries
const auto preallocated\_entries\_size =
sizeof(EptCommonEntry\*) \* kEptpNumberOfPreallocatedEntries;
const auto preallocated\_entries = static\_cast<EptCommonEntry\*\*>(
ExAllocatePoolWithTag(NonPagedPool, preallocated\_entries\_size,
kHyperPlatformCommonPoolTag));
if (!preallocated\_entries) {
EptpDestructTables(ept\_pml4, 4);
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
return nullptr;
}
RtlZeroMemory(preallocated\_entries, preallocated\_entries\_size);
// And fill preallocated\_entries with newly created entries
for (auto i = 0ul; i < kEptpNumberOfPreallocatedEntries; ++i) {
const auto ept\_entry = EptpAllocateEptEntry(nullptr);
if (!ept\_entry) {
EptpFreeUnusedPreAllocatedEntries(preallocated\_entries, 0);
EptpDestructTables(ept\_pml4, 4);
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
return nullptr;
}
preallocated\_entries[i] = ept\_entry;
}
// Initialization completed
ept\_data->ept\_pml4 = ept\_pml4;
ept\_data->preallocated\_entries = preallocated\_entries;
ept\_data->preallocated\_entries\_count = 0;
g\_EptData = ept\_data;//增加
return ept\_data;
}
```

测试发现性能正常，操作流畅，无卡顿，CPU 占用降低，所以问题解决。

原理是设置 CPU 核心为同一页表。Vmware 可能某种原因无法识别我们是为那个 CPU 设置的页表，但是他可以根据设置的物理地址判断是否为同一个页表结构。

别忘记卸载时：

```
// Frees all EPT stuff
\_Use\_decl\_annotations\_ void EptTermination(EptData\* ept\_data) {
if (g\_EptData == 0)//增加
return;
HYPERPLATFORM\_LOG\_DEBUG("Used pre-allocated entries = %2d / %2d",
ept\_data->preallocated\_entries\_count,
kEptpNumberOfPreallocatedEntries);
EptpFreeUnusedPreAllocatedEntries(ept\_data->preallocated\_entries,
ept\_data->preallocated\_entries\_count);
EptpDestructTables(ept\_data->ept\_pml4, 4);
ExFreePoolWithTag(ept\_data, kHyperPlatformCommonPoolTag);
g\_EptData = NULL;//增加
}
```
