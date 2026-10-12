
# 机器狗病毒技术分析

> 原文：[`www.pediy.com/kssd/pediy10/64902.html`](https://www.pediy.com/kssd/pediy10/64902.html)

3 年前在看雪论坛注册了个号，但嫌 `ID` 不好听，因此一直没有发过帖。后来想换个 `ID`，却发现不能注册了。昨天来逛逛，发现能注册了，终于注册了个自己比较喜欢的 `ID`。

## 前言

5 月 6 日抓到的机器狗新变种。仅得到了 `dll`、驱动以及被感染的 `services.exe` 文件，并没有捕获释放源文件。但使用网上流传的一些源代码恢复了释放原程序的一些功能。即直接操作磁盘以绕过 `sfc` 的检验。由于没有经过动态调试以及不太熟悉内核驱动，难免会有些纰漏或错误，敬请各位大虾指正。

## 摘要

机器狗新变种使用了一些流行的技术，包含了修复 `SSDT Hook`、修复 `FSD Hook`、并对一些系统还原软件进行有针对的 `Hook`，使能达到突破还原软件保护的目的。做了那么多，最终目的还是下载大量的木马到用户的系统上。

## 一、修复 SSDT

1. 获取内核第一个模块名（`XP` 系统：`ntkrnlpa.exe`）及 `SVA`（`System Virtual Adrress`），映射一份 `ntkrnlpa.exe` 到内存中，根据导出表获取 `KeServiceDescriptorTable` 的 `RVA` 转换成 `VA`，为 `X`，根据重定位表信息获得需要重定位的地址 `Y`，比较`[Y]`是否等于 `X`，否则循环，是则比较`[Y-2]`的特征是否是 `5C7h`，是则取`[Y+4]`到 `Z`。即：
   `C705E046480040C94200mov ds:_KeServiceDescriptorTable, offset _KiServiceTable`
   
   因此，`Z` 得到的是`_KiServiceTable` 的地址。会对 `Z` 再通过重定位表验证，最后减去基地址得到 `KiServiceTable` 的 `RVA`。

2. 映射文件`\\SystemRoot\\System32\\ntdll.dll` 到内存。根据导出表定位下列函数的 `RVA`，转换成 `VA` 为 `X`，取`[X+1]`得 `Y`，`Y` 即为该 `ZwXXX` 函数的序号。
   - `ZwOpenProcess`
   - `ZwCreateThread`
   - `ZwWriteVirtualMemory`
   - `ZwRestoreKey`

3. 根据序号从内存映射文件 `ntkrnlpa.exe` 获得上列函数的 `RVA`，并转换成相对于 `ntkrnlpa.exe` 模块的地址，为 `RealAddress`。

4. 根据内存映射文件 `ntkrnlpa.exe` 获得 `KeServiceDescriptorTable` 的 `RVA` 为 `X`，`X+SImageBase` 获得 `KeServiceDescriptorTable` 在系统中的 `SVA`，根据上列函数的序号获得这些函数在系统中的 `SVA`，即 `NowAddress`。

5. 判断 `RealAddress` 和 `NowAddress` 是否相等。如果不相等，则将 `RealAddress` 和 `NowAddress` 互换，以达到修复 `SSDT` 的目的。

## 二、获取关键函数的 SVA

`dll` 和驱动的通信是通过以下注册表 `value` 进行的。

`\\Registry\\Machine\\SOFTWARE\\Microsoft\\IE4\\Data`

1. `Dll` 判断内核中是否存在这些模块，是则映射文件到内存。通过特征码找到函数偏移地址。转换成 `SVA` 后，记录入注册表。格式为序号地址，每条记录 8 个字节。
2. 驱动读取该注册表 `value`，把地址保存在全局变量中，以便以后调用。
3. 有以下的函数：
   - `classpnp.sys`：`ClassReadWrite`
   - `classpnp.sys`：`ClassDeviceControlDispatch`
   - `stpdrive.sys`：`???` //没有找到这个模块，未知驱动文件。
   - `ntfs.sys`：`NtfsFsdFileSystemControl`
   - `fastfat.sys`：`FatFsdFileSystemControl`
   - `ntoskrnl.exe`（`NTKRNLPA.exe`、`ntkrnlmp.exe`、`ntkrpamp.exe`）：`Dr_FastCallDrSave_Offset`，不是在函数首部，是在函数的中部。
   - `atapi.sys`：`IdePortDispatch`

## 三、修复 FSD 的 FsdFileSystemControl Object Hook

1. 获取`\\FileSystem\\Ntfs` 或`\\FileSystem\\FastFat` 的 `DriverObject`。
2. 判断`_DRIVER_OBJECT.IrpFileSystemControlDispatch` 的地址是否和从注册表中读到的 `NtfsFsdFileSystemControl` 或 `FatFsdFileSystemControl` 相等，如果不相等，则恢复。

## 四、修复对 DrFastCallDrSave 的 inline Hook

不经过检测，直接向从注册表获取的地址 `Dr_FastCallDrSave_Offset` 处写入 5 个字节的数据。`2B E1 C1 E9 02` 这些数据为系统正常 `Dr_FastCallDrSave_Offset` 处的数据。

重复修复 4 遍

## 五、过还原系统

1. 获取`\\device\\Harddisk0\\DR0` 的设备对象。

   使用 `IoGetDeviceObjectPointer` 获取 `DeviceObject` 失败后，会枚举 `\\device\\Harddisk0` 的 `OBJECT_DIRECTORY_ENTRY`，根据 `ObQueryNameString` 判断符号名来得到 `DR0` 的 `DeviceObject`。

   ```
   lea ecx, [ebp+pObjectDirectroy]
   push ecx
   push 0
   push 0
   mov edx, ds:IoDeviceObjectType
   mov eax, [edx]
   push eax
   push 0
   push 0
   push 40h
   lea ecx, [ebp+wszDeviceName]
   push ecx
   call ObReferenceObjectByName ;
   ```
   对内核不熟，奇怪的是为什么参数为 `IoDeviceObjectType`，而返回的是 `OBJECT_DIRECTORY_ENTRY`？请大虾指点迷津。

2. 获取还原软件的 `DriverObject`

   枚举`\\device\\Harddisk0\\DR0` 的 `AttachedDevice`，根据对应的 `DriverObject` 的 `DriverName` 来获取还原软件设备的 `DriverObject`。如果是影子系统，则会获取 `\\Driver\\Atapi` 的 `DriverObject`。
   
   - `\\Driver\\DeepFrz` 冰点还原精灵
   - `\\Driver\\snpshot` 影子系统
   - `\\Driver\\YzIdiot` 网络还原精灵
   
   如果没有找到上列还原软件，则走 3-5，如果找到了则走 6。

3. `Inline Hook` `stpdrive.sys`

   直接将从注册表中读取的 `stpdrive.sys` 的某个偏移地址，写入一个字节 `C3`，即 `Ret`。由于没有 `stpdrive.sys` 文件，猜测是 `patch` 掉 `stpdrive.sys` 的主要功能，从而使之失效。

4. 再次修复 `FSD` `Object` `Hook`

   同三（修复 `FSD` 的 `FsdFileSystemControl` `Object` `Hook`）。

5. `\\device\\Harddisk0\\DR0` 对应驱动对象的 `Driver` `Object` `Hook`
   1. 修复 `DR0` 设备对应的 `IrpDeviceControlDispatch`
      判断`_DRIVER_OBJECT.IrpDeviceControlDispatch` 是否和从注册表读出的 `ClassDeviceControlDispatch` 的 `SVA` 相等，不相等则修复。
   2. `Hook` `DR0` 设备对应的 `IrpWriteDispatch`
      判断`_DRIVER_OBJECT.IrpWriteDispatch` 是否和从注册表读取的 `ClassReadWrite` 的 `SVA` 相等，不相等则进行自己的 `Object` `Hook`。
   3. `fnIrpWriteDispatch_0`
      判断写入文件的内存是否满足条件，满足则直接调用 `SVA_ClassReadWrite` 进行写文件，不满足则调用原来的 `Old_IrpWriteDispatch`。
      
      条件算法比较苛刻，判断写入的文件头 4 个字节是否是 `4D5A9111`，即被感染的 `Services.exe`，如果满足，则直接调用 `SVA_ClassReadWrite` 进行写文件。并将 `Key` 置 1。如果 `Key` 为 1，则判断写入文件的尾部 4 个字节是否为 `4B435546`，即病毒 `dll` 文件，如果是，则直接调用 `SVA_ClassReadWrite` 进行写文件。并且将 `Object` `Hook` 恢复到之前的状态（非系统状态）。
      
      这种方法只进行对 2 个文件进行过还原的写入，写入完成后，会恢复其它还原软件的 `Object` `Hook`。

6. 还原系统的 `Object` `Hook`

   原理基本同 5，`Hook` 的对象为还原设备的 `Driver` `Object`。
   
   不同点：
   1. 首先 `DrFastCallDrSaveHooK`，同四（修复对 `DrFastCallDrSave` 的 `inline` `Hook`）。
   2. 修复 `atapi.sys` 的 `IrpInternalDeviceControlDispatch`
      根据`_DRIVER_OBJECT.IrpInternalDeviceControlDispatch` 和从注册表读取的 `IdePortDispatch` 相比较，不同则恢复。
   3. 写文件的方式不同
      调用 `IofCallDriver`，直接向前一个 `AttachedDevice` 发送 `IRP` 进行写文件操作。
   4. 恢复 `Object` `Hook`
      除了恢复 `IrpWriteDispatch` 到之前的状态，也恢复 `Atapi.sys` 的 `IrpInternalDeviceControlDispatch` 到之前的状态。

## 六、dll 释放驱动

先释放驱动到`%temp%\\data??.tmp`，然后获取一些驱动执行需要的信息，并存放在注册表 `Registry\\Machine\\SOFTWARE\\Microsoft\\IE4\\Data` 中，见二（获取关键函数的 `SVA`），以保证驱动能够正常运行。将`%system32%\\drivers\\beep.sys` 复制到 `%system32%\\dllcache\\beep.sys`，停止 `beep` 服务，将`%temp%\\data??.tmp` 复制到 `%system32%\\drivers\\beep.sys`，并启动 `beep` 服务。通过这样的方法绕过卡巴等杀软的主动防御，这种方法被广泛运用在病毒上。驱动加载后，会被 `dllcache` 目录下的自动替换。

释放加载驱动是 `dll` 中的导出函数 `HttpAddUrl` 的主要功能，和 `dllmain` 函数的部分功能。

## 七、获取 ZwXXX 函数

通过动态链接 `ntdll.dll` 方式获取下列函数，但只有 `ZwQuerySystemInformation` 被使用到。

- `RtlInitUnicodeString`
- `ZwOpenSection`
- `ZwQuerySystemInformation`
- `ZwQueryInformationFile`
- `ZwQueryInformationProcess`
- `ZwOpenFile`

## 八、获取下载病毒地址

从本 `dll` 文件偏移 `0x3E8` 处开始通过特征（`F1 F7 9C 25 67 E4 00`）定位，找到加过密的字符串，存放格式：`size+string`，将 `string` 的每一个字节和 `size` 的异或运算，解得病毒地址为：

`http://z3.us-2.net/222.txt`

## 九、线程下载执行木马

线程主要功能是下载并执行木马病毒。解析 `http://z3.us-2.net/222.txt` 的内容后，下载大量病毒并执行，下载是使用了 `WinSocket` 系列函数，执行使用 `WinExec`。

线程还会杀母体和释放并加载驱动。

## 十、查杀母体

这个病毒会从注册表信息中读取母体路径，并删除文件和注册表 `value`。

`HKEY_LOCAL_MACHINE\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\explorer\\DESKTOP\\SysFile`

`DllMain` 的功能包括了六、七、八、九、十。

## 十一、查杀驱动防火墙(drvAnti.exe)

查找到 `drvAnti.exe` 后，结束它的所有线程。

```
CreateToolhelp32Snapshot
Thread32First
do
{
    OpenThread
    TerminateThread
}while(Thread32Next)
```

有的版本的 `dll` 不会查杀驱动防火墙，会检测 `protectedc.sys`（网维大师文件），找到则结束自身进程中 `ThreadID` 为`（0，10000h）`内的线程。这样做是否是因为 `protectedc.sys` 是隐藏了模块，使得除了自己软件才能枚举到该模块，这只是猜测，没有该文件无法证实。

## 十二、注入系统进程

这个功能是导出函数 `InitHttp` 实现的，有意思的是需要传入一个需要让系统进程加载的 `dll` 名作为参数。如 `InitHttp cards.dll`，注入的代码会执行 `LoadLibraryA cards.dll`。

远线程注入以下系统进程：

- `winlogon.exe`
- `explorer.exe`

注入的代码功能很简单

```
InjectCode proc near
arg_0 = dword ptr 4

mov ecx, [esp+arg_0]
mov eax, [ecx+208h]
cmpeax, 186A0h
jb Exit0
cmpdword ptr [eax], 0
jz Exit0
push ecx
call eax

Exit0:
xor eax, eax
retn 4

InjectCode endp
```

这段代码是用来执行 `CreateRemoteThread` 传入的参数的，这有点像 `ShellCode`。而在这里使用的是 `LoadLibrary DllName`，`DllName` 为 `InitHttp` 的参数。

## 小结

以上就是机器狗 `dll` 和驱动的所有功能了，`Services.exe` 被感染的方式，是在输入表里增加一项，即病毒 `dll` 名和导出函数 `HttpAddUrl`。由于没有获取释放原程序，因此不知道具体母体程序是怎么感染 `Services.exe` 的，是怎么绕过 `sfc` 写系统文件的，或者也有一些其它技术的使用。

对于感染 `Services.exe`，猜测是将 `Services.exe` 读到内存，然后进行增加输入表感染，然后将内存数据写入文件，因为驱动特别针对 `Services.exe` 和 `dll` 病毒的写入做了大量工作。

对于绕过 `sfc`，猜测是使用机器狗老变种的方法，通过 `FSCTL_GET_RETRIEVAL_POINTERS` 的方法（详细请参看 `Libra` 翻译的文档《被占用文件操作三法》）读取文件在磁盘中的存储信息，然后算出文件在磁盘的偏移，直接写磁盘操作的。

但也有其他方法，如调用 `sfc.dll` 的 `#5` 函数即可使 `sfc` 不对该文件进行检验。

对于杀毒软件，系统文件保护以及防止恶意驱动加载将会或已经成为主题。

分析者：水中雁
日期：2008.05.09
邮箱：`tigerinhm@163.com`
主页：`http://hi.baidu.com/wildgoose`

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | 机器狗技术剖析.rar [`bbs.pediy.com/showthread.php?t=64902`](http://bbs.pediy.com/showthread.php?t=64902) |
|---|---|
