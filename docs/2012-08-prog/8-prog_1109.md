
# WINDOWS VISTA/WIN7 内核完整性校验解析 PE 蓝屏 BUG

> 原文：[`www.pediy.com/kssd/pediy10/91096.html`](https://www.pediy.com/kssd/pediy10/91096.html)

在 `VISTA` 和 `WINDOWS 7` (`ALL SP`, 全补丁) 的内核代码完整性校验组件中，存在一处解析 `PE` 的 `BUG`，可引发系统蓝屏。不过从现在看来，此问题似乎需要 `LoadDriver` 权限才能触发，因此不能算安全性问题，属于稳定性问题。

在 `VISTA` 以后的操作系统，系统使用 `MmLoadSystemImage` 加载驱动之前，会调用 `MmCheckSystemImage` 函数来检查镜像正确性，在 `VISTA` 及以后的操作系统中，`MmCheckSystemImage` 发生了一个有意思的变化。

原本 `MmCheckSystemImage`（`vista` 以前的系统上），会使用 `SEC_IMAGE` 作为 `AllocationAttributes` 来调用 `ZwCreateSection` 为驱动文件创建 `Section`，但是 `VISTA` 以后的系统上，该参数被换成了一个未公开的值：`0x100000`（注意，`SEC_IMAGE` 是 `0x1000000`，6 个 `0`）。

在 `ZwCreateSection` 中，系统会检查如果调用线程的上个模式不是 `KernelMode`，则不允许使用这个未公开的 `AllocationAttributes`：

```
NtCreateSection:

............无关部分........

//取当前线程上个模式

  if (KeGetPreviousMode() != KernelMode)
  {

//如果是用户模式，如果 Attributes 有 0x2000000 或 0x10000 的话，则返回无效参数
    v13 = 0;
    if ( !(Attributes & 0x2100000) )
    {
        .....无关处理.... 

    }
    return STATUS_INVALID_PARAMETER_6;
  }

```

然后在 `NtCreateSection->MmCreateSection`（这个函数在 `VISTA` 开始发生了巨大的变化）中，进行一些判断后（包括这个特殊的 `Attributes`），系统会调用 `MiValidateImageHeader` 进行镜像检查，此时系统会将这个镜像通过 `MiMapImageInSystemCache` 函数 `map` 到系统缓存中，`然后将按镜像的页数 * PAGE_SIZE 大小的，分配分页内存，并将 PE 数据 COPY 到内存中`

接着系统会调用 `SeValidateImageHeader` 函数，这个函数只是简单地为 `_g_CiCallbacks` 中存放的函数准备函数，便调用 `_g_CiCallbacks` 存放的函数。`_g_CiCallbacks` 这个全局变量中存放着系统初始化时（`SeInitSystem->SepInitializationPhase1->SepInitializeCodeIntegrity`）存入的 `ci.dll` 的 `CiValidateImageHeader` 函数。

`CiValidateImageHeader` 首先会对镜像进行一些检查工作，然后开始调用 `CipValidateFileHash` 函数，`CipValidateFileHash` 函数对文件做一些解析工作后，开始调用 `CipImageGetImageHash`，此函数会分析 `PE` 的每一个节，并对其节内数据调用 `SHA` 签名算法函数 `A_SHAUpdate`。

注意前面加粗的文字，由于是按整页数来分配和 `COPY` 数据的，因此如果某一个节的数据长度（`SizeOfRawData`）超出了页对齐的范围（`MiMapImageInSystemCache` 似乎并不将这个数据算成一个新的节），那么 `A_SHAUpdate` 中的数据 `COPY` 函数将触及到未分配内存，从而引发 `BSOD`。

这里提供一个简单的例子。`bsodxx.rar`

这个 `PE` 文件的最后一个节的 `SizeOfRawData` 是 `0x1004`，使用任意一个加载工具加载此文件，系统将立即 `BSOD`，`BSOD` 时的 `Stack` 类似：

```
kd> kc

nt!MmAccessFault
nt!KiTrap0E
nt!memcpy
CI!A_SHAUpdate
CI!CipImageGetImageHash
CI!CipValidateFileHash
CI!CiValidateImageHeader
nt!SeValidateImageHeader
nt!MiValidateImageHeader
nt!MmCreateSection
nt!NtCreateSection
nt!KiFastCallEntry
nt!ZwCreateSection
nt!MmCheckSystemImage
nt!MiCreateSectionForDriver
nt!MiObtainSectionForDriver
nt!MmLoadSystemImage
nt!IopLoadDriver
nt!IopLoadUnloadDriver
```
