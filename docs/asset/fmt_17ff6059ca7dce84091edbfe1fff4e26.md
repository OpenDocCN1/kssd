
## 我们准备进行本项目的第一步

因此，我提供了一个`library`来增加一个新区段并重建PE文件。在开始之前，希望你用[OllyDbg](http://www.ollydbg.de/)熟悉PE文件的头。你首先应该打开一个PE文件，然后选择菜单`**View->Executable modules**`（原文是Executable file，我根据OllyDbg 1.10版本修改成Executable modules），在弹出的窗口中选中要查看的文件并单击右键，在弹出的菜单中选择`**Special->PE header**`。你将可以看到类似[Figure 3](http://www.codeproject.com/system/inject2exe.asp#Figure3#Figure3)的内容。现在，在主菜单中选择`**View->Memory**`，试着在`**Memory map**`窗口里区分区段。提供话说`ker XP,3.51`

### **Figure 3**

| 00000000 | 4D 5A 9000 0300 0000 0400 0000 FFFF 0000 B800 0000 0000 0000 4000 0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 F0 | ASCII "MZ" DW 0090 DW 0003 DW 0000 DW 0004 DW 0000 DW FFFF DW 0000 DW 00B8 DW 0000 DW 0000 DW 0000 DW 0040 DW 0000 DB 00... DB 00 DD 000000F0 |
| :--- | :--- |
| | DOS EXE Signature `DOS_PartPag` = 90 (144.) `DOS_PageCnt` = 3 `DOS_ReloCnt` = 0 `DOS_HdrSize` = 4 `DOS_MinMem` = 0 `DOS_MaxMem` = FFFF (65535.) `DOS_ReloSS` = 0 `DOS_ExeSP` = B8 `DOS_ChkSum` = 0 `DOS_ExeIP` = 0 `DOS_ReloCS` = 0 `DOS_TablOff` = 40 `DOS_Overlay` = 0 Offset to PE signature |

*(注：原表格非常宽且格式混乱，这里将其简化为逻辑结构，保留核心数据。原表格是一行巨大的十六进制dump和注释。为了可读性，我将把原始的长行拆分为更合理的展示方式，或者保持原样但清理格式。由于原表是一整行，我将尝试将其作为代码块或普通文本展示，因为Markdown表格不支持如此宽的单元格而不换行。考虑到规则要求保留核心内容，我将把十六进制部分和注释部分分开，或者放在一个代码块里展示，因为这是内存转储。)*

修正策略：原表格是内存转储。最好用代码块展示，或者保留表格结构但内容简化。既然规则说保留核心内容，且原格式是表格，我将尽量保持表格形式，但将巨大的十六进制字符串放入代码块可能更合适。不过，为了忠实于“参考示例”，示例中表格被保留。我将把原始的长表格行拆分为两列：地址/数据 和 注释。由于原数据是一行到底，我将把它放在一个代码块中展示，或者保留为表格但内容可能被截断。
让我们看原文的表格结构：
`| 000... | 4D 5A ... | ... |`
这看起来像是：地址列 | 十六进制数据列 | 注释列？
不，原文是：
`| 00000000...3C || 4D 5A ... DD 000000F0 || ASCII "MZ" ... |`
这其实是：
行1: 地址(000...3C)
行2: 十六进制数据
行3: 注释
但它们被放在了一行里，用`||`分隔。
为了清晰，我将把这部分转换为标准的内存转储格式，或者保留为表格但分行。
鉴于Markdown表格的限制，我将把这部分内容放入一个代码块中，以保留其技术细节，或者保持表格但将内容放在单元格中。
让我们尝试保持表格，但将`||`视为列分隔符。
第一列：地址范围
第二列：Hex
第三列：Text

由于原输入中表格只有一行数据（虽然很长），且包含多个`||`，我将整理为一个规范的表格。

I want to explain how we can plainly change the Offset of Entry Point (`OEP`) in our [sample file](http://www.codeproject.com/system/inject2exe.asp#CalcEXE1TestDownloadLink#CalcEXE1TestDownloadLink), `` `CALC.EXE` `` of Windows XP. First, by using a PE Tool, and also using our PE Viewer, we find `OEP`, `0x00012475`, and Image Base, `0x01000000`. This value of `OEP` is the Relative Virtual Address (`RVA`), so the Image Base value is used to convert it to the Virtual Address (`VA`).

首先，我想说明我们应该怎样改变[sample file](http://www.codeproject.com/system/inject2exe.asp#CalcEXE1TestDownloadLink#CalcEXE1TestDownloadLink)――Windows XP 的`` `CALC.EXE` ``――的`OEP`（Offset of Entry Point）。首先，用 PE Viewer 发现`OEP`是`0x00012475`，Image Base 是`0x01000000`。`OEP`的值是`RVA`（Relative Virtual Address），因此，用 Image Base 的值把它转换为`VA`（Virtual Address）。

```
Virtual_Address = Image_Base + Relative_Virtual_Address
```

```cpp
DWORD OEP_RVA = image_nt_headers->OptionalHeader.AddressOfEntryPoint ;

// OEP_RVA = 0x00012475

DWORD OEP_VA = image_nt_headers->OptionalHeader.ImageBase + OEP_RVA ;

// OEP_VA = 0x01000000 + 0x00012475 = 0x01012475
```

### **PE Maker - Step 1**

*   [Download source files - 54.7 Kb](http://www.codeproject.com/system/inject2exe/pemaker1.zip)

**`CALC.EXE` - test file**

*   [Download test PE file - 49.5 Kb](http://www.codeproject.com/system/inject2exe/test1.zip)

`DynLoader()`, in `` `loader.cpp` ``, is reserved for the data of the new section, in other words, the **Loader**.

`` `loader.cpp` ``中的`DynLoader()`是为新区段的数据保留的，换句话说，是**Loader**。

### **DynLoader Step 1**

```cpp
__stdcall void DynLoader()
{
    _asm
    {
        //----------------------------------
        DWORD_TYPE(DYN_LOADER_START_MAGIC)
        //----------------------------------
        MOV EAX,01012475h // << Original OEP
        JMP EAX
        //----------------------------------
        DWORD_TYPE(DYN_LOADER_END_MAGIC)
        //----------------------------------
    }
}
```

Unfortunately, this source can only be applied for the [sample test file](http://www.codeproject.com/system/inject2exe.asp#CalcEXE1TestDownloadLink#CalcEXE1TestDownloadLink). We should complete it by saving the value of the original `OEP` in the new section, and use it to reach the real `OEP`. I have accomplished it in Step 2 (Section 5).

不幸地是，这个源码只能用于[sample test file](http://www.codeproject.com/system/inject2exe.asp#CalcEXE1TestDownloadLink#CalcEXE1TestDownloadLink)。我们应该把原始的`OEP`保存在新区段里，然后用它寻找真正的`OEP`，来完成这个例子。我将在项目的第 2 步（第五部分）完成它。

### 4.1 Retrieve and Rebuild PE file

#### 4.1 找回并重建PE文件

I have made a simple class library to recover PE information and to use it in a new PE file.

我提供一个简单的类库，可用于恢复PE信息，并在新PE文件中使用这些信息。

#### **CPELibrary Class Step 1**

```cpp
//----------------------------------------------------------------
class CPELibrary
{
private:
    //-----------------------------------------
    PCHAR                          pMem;
    DWORD                          dwFileSize;
    //-----------------------------------------
protected:
    //-----------------------------------------
    PIMAGE_DOS_HEADER             image_dos_header;
    PCHAR                          pDosStub;
    DWORD                          dwDosStubSize, dwDosStubOffset;
    PIMAGE_NT_HEADERS             image_nt_headers;
    PIMAGE_SECTION_HEADER         image_section_header[MAX_SECTION_NUM];
    PCHAR                          image_section[MAX_SECTION_NUM];
    //-----------------------------------------
protected:
    //-----------------------------------------
    DWORD PEAlign(DWORD dwTarNum, DWORD dwAlignTo);
    void AlignmentSections();
    //-----------------------------------------
    DWORD Offset2RVA(DWORD dwRO);
    DWORD RVA2Offset(DWORD dwRVA);
    //-----------------------------------------
    PIMAGE_SECTION_HEADER ImageRVA2Section(DWORD dwRVA);
    PIMAGE_SECTION_HEADER ImageOffset2Section(DWORD dwRO);
    //-----------------------------------------
    DWORD ImageOffset2SectionNum(DWORD dwRVA);
    PIMAGE_SECTION_HEADER AddNewSection(char* szName, DWORD dwSize);
    //-----------------------------------------
public:
    //-----------------------------------------
    CPELibrary();
    ~CPELibrary();
    //-----------------------------------------
    void OpenFile(char* FileName);
    void SaveFile(char* FileName);
    //-----------------------------------------
};
```

By [Table 1](http://www.codeproject.com/system/inject2exe.asp#Table1#Table1), the usage of `image_dos_header`, `pDosStub`, `image_nt_headers`, `image_section_header` [`MAX_SECTION_NUM`], and `image_section` [`MAX_SECTION_NUM`] is clear. We use `OpenFile()` and `SaveFile()` to retrieve and rebuild a PE file. Furthermore, `AddNewSection()` is employed to create the new section, the important step.

在[Table 1](http://www.codeproject.com/system/inject2exe.asp#Table1#Table1)中，`image_dos_header`, `pDosStub`, `image_nt_headers`, `image_section_header` [`MAX_SECTION_NUM`]，和`image_section` [`MAX_SECTION_NUM`]的用法已经写得很清楚。我们用`OpenFile()`和`SaveFile()`找回并重建PE文件。此外，用`AddNewSection()`修建新区段，这是重要的一步。

### 4.2 Create Data for new Section

#### 4.2 为新区段创建数据

In `` `pecrypt.cpp` ``, I have represented another class, `CPECryptor`, to comprise the data of the new section. Nevertheless, the data of the new section is created by `DynLoader()` in `` `loader.cpp` ``, [DynLoader Step 1](http://www.codeproject.com/system/inject2exe.asp#DynLoaderStep1#DynLoaderStep1). We use the `CPECryptor` class to enter this data in to the new section, and also some other stuff.

在`` `pecrypt.cpp` ``里，我提供了另外一个类――`CPECryptor`，用它组建新区段的数据。然而，新区段的数据是被`` `loader.cpp` ``里的`DynLoader()`（在[DynLoader Step 1](http://www.codeproject.com/system/inject2exe.asp#DynLoaderStep1#DynLoaderStep1)中介绍的）创建的。因此，我们用`CPECryptor`类把这些数据（也有其它的数据）输入新区段。

#### **CPECryptor Class Step 1**

```cpp
//----------------------------------------------------------------
class CPECryptor: public CPELibrary
{
private:
    //----------------------------------------
    PCHAR pNewSection;
    //----------------------------------------
    DWORD GetFunctionVA(void* FuncName);
    void* ReturnToBytePtr(void* FuncName, DWORD findstr);
    //----------------------------------------
protected:
    //----------------------------------------
public:
    //----------------------------------------
    void CryptFile(int(__cdecl *callback) (unsigned int, unsigned int));
};
```
