
# 浅析.NET Framework 对 PE 文件格式的扩展

> 原文：`www.pediy.com/kssd/pediy06/pediy6690.htm`

浅析.NET Framework 对 PE 文件格式的扩展

WebCrazy(`webcrazy.yeah.net`)

Microsoft .NET Framework 出来小阵子了，我也自从其 Beta 1 以来，第一次接触。本文将从 .NET 生成的一个小 PE 文件着手，旨在理解 .NET Framework 对 PE 文件格式的扩展。这种扩展目的是让 Windows 系统识别 Common Language Runtime(CLR)。

PE 文件是 Windows 系列操作系统的可执行文件格式。本文假设您对这一文件格式有相当的理解，文中未涉及 PE 在之前的 win16 及之后的 win64 上的讨论。在 CLR 出现之前，PE 文件格式仅简单的由 PE Header 与 Native Image(相对于以下介绍的 CLR Header 与 CLR Data 部分)组成。Native Image 由各个 section 组成，如 `.text`，`.data`，`.rdate` 等等，需要指出的是 PE 文件的这些 section 名命名规则并不要求一定要以句点开头，事实上这只是 Microsoft 的对于代码段或数据段的默认说法，像 Borland 等其他编译器则相应分别命名为 `CODE`，`DATA` 等等。Native Image 含有已编译的相应处理器的机器代码。

在 CLR 出现后 PE 文件扩展出了另外一部分，即 CLR Header 与 CLR Data 组成的供 .NET Framework 运行的支撑部分。CLR Header 由 .NET Framework SDK 的 `CorHdr.h` 中的 `IMAGE_COR20_HEADER` 结构定义。从 `CorHdr.h` 或是 `IMAGE_COR20_HEADER` 的命名中 Cor 的全称 Com+ Runtime 即可隐隐约约的看到 .NET Framework 的发展过程，其与 COM+的渊源关系了。事实上 `IMAGE_COR20_HEADER` 在平台 SDK 的 `winnt.h` 中也有定义，我查阅的了随 Windows XP DDK Build 2505 发行的 `winnt.h` 中 Microsoft 在给出这个定义时的注释为 COM+ 2.0 header structure，而在 .NET Framework SDK 中即修改为 CLR 2.0 header structure 了。CLR Data 则包含 .NET metadata, IL method bodies 等等。metadata 及 IL method 是 .NET 中很关键的术语。IL 即 Microsoft Intermediate Language 的缩写。她是为了 .NET 跨平台、跨语言的特性而引入的，有其自身的指令集。.NET SDK 中的 `opcode.def` 列出的其支持的指令集。粗粗看来这些指令集与 Intel 的 X86 指令集十分的相像，也是由 Prefix 指定的的双字节进行编码。

下面的我将通过底下列出的这一段 C# Console 代码来简述 C#编译器生成的 PE 文件的执行流程及 PE 文件的 on disk 结构。代只是简单的输出 Hi，如下所示：

```csharp
public class App {  
  static public void Main(System.String[] args) {  
    System.Console.WriteLine("Hi");  
  }  
}
```

我们简单的使用 `csc /out:app.exe app.cs` 对其编译。生成的 PE 文件，与 .NET 出现前传统的编译器生成的 PE 文件一致，也含有 `IMAGE_DOS_HEADER`，我们知道这部分的作用即是早期的 DOS 在遇到 PE 文件格式时，能判定这个可执行文件不能执行于 DOS 下而存在的。`IMAGE_DOS_HEADER` 与将要谈及的一些结构在 `winnt.h` 中均有详细定义。Windows OS Loader 根据 `IMAGE_DOS_HEADER` 中的 `e_lfanew` 成员定位紧挨着的 `IMAGE_NT_HEADERS`。其定义如下：

```c
typedef struct _IMAGE_NT_HEADERS {  
   DWORD Signature;  
   IMAGE_FILE_HEADER FileHeader;  
   IMAGE_OPTIONAL_HEADER32 OptionalHeader;  
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

我们知道 `IMAGE_OPTIONAL_HEADER32` 的成员 `AddressOfEntryPoint` 是 PE 可执行文件的入口，在 .NET 中其仍为执行入口，这应该是很好理解的。对于一个 `COMIMAGE_FLAGS_ILONLY`(由 `IMAGE_COR20_HEADER` 的成员 `Flags` 指定)的 Image，如我们生成的 `App.exe`，这个入口也即间接定位至 `App.exe` 的 Import 表的 `_CorExeMain` 函数。`_CorExeMain` 对应 EXE 文件，由 `mscoree.dll` 导出。`mscoree.dll` 位于 `%WINNT%\system32` 下，是 Microsoft .NET Runtime Execution Engine，应该指出的她是一个 Native Image，负责调用 `IMAGE_COR20_HEADER` 中的 `EntryPointToken` 指定的 .NET Token。这才是真正 IL 语言的入口。

Native Image 部分的各个 Section 的定位，已经有很多文档介绍，而且 `winnt.h` 中都有详细的定义。我只简单的阐述一下：

`.text`、`.data` 等 section 定位是由 `IMAGE_OPTIONAL_HEADER32` 中的 `DataDirectory` 成员指定。`DataDirectory` 是一个 `IMAGE_DATA_DIRECTORY` 数组，个数为 `IMAGE_NUMBEROF_DIRECTORY_ENTRIES`(当前为 16)个。各个 DataDirectory 功能分别由 `IMAGE_DIRECTORY_ENTRY_****`指定，如 `EXPORT`、`IMPORT` 等等。因为 `IMAGE_DATA_DIRECTORY` 由 `VirtualAddress`(RVA)与 `Size` 组成，所以我们即可以很容易的找到这些 Section 的位置。与这些 Section 一样，CLR Header 的定位也是 DataDirectory 指定，其为 `IMAGE_DIRECTORY_ENTRY_COMHEADER`(值为 14，.NET Framework SDK V1 `CorHdr.h` 中称谓，在 DDK 2505 的 `winnt.h` 中为 `IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR`)。我们生成的 `App.exe` 有如下的格式：

```text
.
.
.

AddressOfEntryPoint: 0x000022CE (+0x10)

.
.
.

DataDirectory[0] - IMAGE_DIRECTORY_ENTRY_EXPORT
  VirtualAddress: 0x00000000 (+0x60)
  Size: 0x00000000 (+0x64)

DataDirectory[1] - IMAGE_DIRECTORY_ENTRY_IMPORT
  VirtualAddress: 0x0000227C (+0x68)
  Size: 0x0000004F (+0x6C)

.
.
.

DataDirectory[14] - IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR
  VirtualAddress: 0x00002008 (+0xD0)
  Size: 0x00000048 (+0xD4)

.
.
.
```

OK，从 `DataDirectory[14]`我们即可以很容易的定位 CLR Header。CLR Header 可以被合并到其它任何为只读属性的 Section 中。前面已经提及到 CLR Header 由 `IMAGE_COR20_HEADER` 结构定义。

```c
// CLR 2.0 header structure.
typedef struct IMAGE_COR20_HEADER
{
  // Header versioning
  ULONG cb;
  USHORT MajorRuntimeVersion;
  USHORT MinorRuntimeVersion;
  // Symbol table and startup information
  IMAGE_DATA_DIRECTORY MetaData;
  ULONG Flags;
  ULONG EntryPointToken;
  // Binding information
  IMAGE_DATA_DIRECTORY Resources;
  IMAGE_DATA_DIRECTORY StrongNameSignature;
  // Regular fixup and binding information
  IMAGE_DATA_DIRECTORY CodeManagerTable;
  IMAGE_DATA_DIRECTORY VTableFixups;
  IMAGE_DATA_DIRECTORY ExportAddressTableJumps;
  // Precompiled image info (internal use only - set to zero)
  IMAGE_DATA_DIRECTORY ManagedNativeHeader;
} IMAGE_COR20_HEADER;
```

这个结构的 `Flags` 与 `EntryPointToken` 上面已经提及。从这么多的 `IMAGE_DATA_DIRECTORY` 上看，这个定义很像 `IMAGE_OPTIONAL_HEADER32`，后者可以理解成 PE 文件头的精华，其用于定位 `.text` 等 Section，由 Windows OS Loader 执行。而前者用于定位 .NET CLR Data，如 `MetaData`、`Resources`、`StrongNameSignature` 等等。不同的是 `IMAGE_COR20_HEADER` 是由 `mscoree.dll` 中的 `_CorExeMain`(对应于 EXE 文件)负责调用(MSIL 语言需经过 JIT 编译成机器码才可执行)。

虽然 `EnrtyPointToken` 与上面的 `AddressOfEntryPoint` 均是执行入口，但却有非常大的区别。`AddressOfEntryPoint` 是一 RVA，直接指向执行地址(相对于 Image Base)，其只能指向一本地机器代码用于装载 NET Runtime(如 `mscoree.dll` 中的 `_CorExeMain`，对于 DLL 文件其可以置为 0)。而 `EntryPointToken` 只是一个 .NET TOKEN。TOKEN 是 .NET Type 的唯一识别，是一个 DWORD 值。其最高的 8bit 指明何种 TOKEN。其由 `CorHdr.h` 中的 `CorTokenType` enum 定义。如 `mdtMethodDef` 为 `0x06000000`，`mdtEvent` 为 `0x14000000` 等等，而余下的 24bit 则为此类 TOKEN 的唯一识别。`EnrtyPointToken` 只能是一 METHOD，而不能是 EVENT 等等。如 `App.Exe` 的 `EnrtyPointTokeno` 为 `0x06000001`，其对应于 `Main` Method。您可以使用 `ildasm.exe`(随 .NET Framework SDK 提供)进行验证。

`App.exe` 的 CLR Header 如下(只列出了部分非空字段)：

```text
Size: 0x00000048
MajorRuntimeVersion: 0x0002
MinorRuntimeVersion: 0x0000
MetaData
VirtualAddress: 0x0000207C
Size: 0x00000200
Flags: 0x00000001
COMIMAGE_FLAGS_ILONLY
EntryPointToken: 0x06000001
```

.NET MetaData 由 `MetaData` 成员指定。Microsoft 在 `CorHdr.h` 中给出了 ILMETHOD 的 on disk 组织结构(`IMAGE_COR_ILMETHOD`)。随 .NET Framework SDK 也提供了一个例子 `metainfo` 用于分析 Metadata。随 QuickStart 例子的 Class Browser 的 ASP.NET 范例也是 .NET Framework 很好的学习材料。Metainfo 使用常规的 COM 方法，而 Class Browser 使用 .NET Framework 的 `System.Reflection` Namespace。关于 .NET 的 SOAP，Web Services，Web Forms，XML 等等 QuickStart 真不愧为 QuickStart，.NET 看来是下阵子学习的方向啊。

最后应该说明的是对于 .NET 我真有一种很自清新的感觉，自身也是刚刚接触，本文仅是抱着学习的态度，权当自己的学习笔记，与各位进行交流。文中有误之处或是有所建议，请联系 tsu00@263.net。

4dd466d0e1e0cc123afbe504da2462ca
