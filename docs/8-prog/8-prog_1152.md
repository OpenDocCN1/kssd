
# 【原创+源码+图已修复】利用 DbgHelp 取的 NT 内核符号信息示例

> 原文：[`www.pediy.com/kssd/pediy11/117432.html`](https://www.pediy.com/kssd/pediy11/117432.html)

## 利用 DbgHelp 查询符号名称及 Type 信息的示例

NT 内核大量的信息需要公有符号文件来解释，而在动态分析，动态 HOOK 中也常常需要知道一些非导出的私有符号的地址或者某些结构元素的偏移，但是这些偏移在各个 NT 版本中是不同的。

如果用硬编码的方式显然无法做到通用性，而对于不同 NT 版本，甚至是同一版本的不同 PACK 统统硬编码也会很麻烦，而且无法应用于新的 PACK。一般的解决办法是动态根据某些特征码在运行时查找某一地址，而利用微软的符号文件又有一种兼容性不错的方法。

为此在 `NtDriverFrame` 框架上增加了对 DbgHelp 库的支持，从而比较容易实现了对内核私有信息的查询的一个简单工具的雏形 `NtInfoGuy`，理论上兼容所有 NT 平台，目前测试的平台有：`Win2k(sp4)`, `Winxp(sp3)`, `Win2k3(sp2)`, `Vista`, `Windows7`。

该工具的重点不在于内核各种表的取得，这个已有太多的方法。而是在于使用 DbgHelp 的另一种实现方式。

`NtInfoGuy` 不准备做成 XXX 安全工具，只是一个简单的内核信息的查看器，为了方便一些程序员了解内核而已。

### `NtInfoGuy` 已实现的功能

（点击图片看大图）

![](img/afe7d49e4bdbae019aae4f56bf1c778d.jpg)  
![](img/272f3a27c483cd0184dfbcf0b58a287c.jpg)  

1. 显示系统 SSDT 表，SSDT Shadow 表并且尝试寻找可能的服务表项钩子，红色标示出了可能的钩子;  
   ![](img/e6293ca7189781d77919b147d0d6bc17.jpg)
2. 显示系统 GDT 表，并且显示各个表项的属性;  
   ![](img/918a7493af0e01fb523bad49fb9f4fed.jpg)
3. 显示系统 IDT 表，并且显示各个门的属性;  
   ![](img/2461fb9c9ecabd0a3d74f70783f706b6.jpg)
4. 显示当前系统加载模块的信息，并且可识别出 windows 可信任模块;  
   ![](img/1d806b46f03eaab64bffc9238c0586e1.jpg)  
   红字表示不受信任的模块或在磁盘上未找到的模块。
5. 直接从内核中获取系统加载模块的信息，在 DbgView 中显示;  
   ![](img/94b59cd4ce90a8257db2e6d87bec6c4c.jpg)
6. 显示系统各个主要部件内核变量的值.  
   ![](img/7e3189d0109915d6d6c2ab01046a459b.jpg)

### 准备添加的功能

视精力情况以下是准备添加的功能：

1. 准备再添加 Inline Hook 的识别，以及将 Hook 还原的功能;
2. 将内核地址对应到一个区域中，比如一个驱动，换页池，非换页池等;
3. 显示指定位置内核代码的反汇编;
4. 增加 GUI，可能用 SDK，也可能用 VB, C#等等语言来写;
5. 更加全面的内核变量的显示，目前只是显示了内存管理器的内核变量。

### 程序说明

1. 首先熊猫以人格担保代码里无任何木马，病毒, RootKit 等无聊东东;
2. 我写的代码是 `NtInfoGuy.exe` 和 `NtInfoGuy.dll` 加起来不到 60KB，其他 2 个 Dll 是微软官方的调试以及符号服务库，在运行时是要使用的。如果你的系统中安装了新版的 WinDbg，另外这两个 Dll 可以使用 WinDbg 目录中的新版本。
3. 程序需要加载驱动程序进入内核取得信息;
4. 程序需要自动连接到微软官方符号网站下载内核的符号文件，否则某些内核符号不能获得。和 WinDbg 下载符号文件是同样的道理。
5. 该程序可能有 BUG、漏洞，可能会导致系统崩溃，请在非关键系统上运行。该程序带来的一切损失和熊猫无关哦。
6. 关于该程序的更多信息请观赏 : `http://blog.csdn.net/mydo/archive/2010/0/17/5742188.aspx`
7. 程序第一次运行时因为要下载 NT 符号文件可能比较慢，一旦符号下载完成, 以后的运行都会很快. 符号文件下载的位置就在程序当前路径的 `syms` 文件夹中.

### 代码简单说明

```cpp
//打印出指定结构的元素名称及偏移
//利用 DbgHelp 的 SymGetTypeInfo 函数先取得结构 ID，然后取
//结构元素个数，最后取得元素名称和偏移
bool HySymPrintStructElements(DWORD64 ModBase, const char *StructName)
{
    bool bRet = false;
    PSYMBOL_INFO pSI = malloc(sizeof(SYMBOL_INFO)+MAX_SYM_NAME);
    if(!pSI) goto QUIT;
    pSI->SizeOfStruct = sizeof(SYMBOL_INFO);
    pSI->MaxNameLen = MAX_SYM_NAME;
    pSI->ModBase = ModBase;
    
    if(!HySymGetTypeInfo(ModBase,StructName,pSI))
    {
        PRINT("[%s]err : Get Struct Info Failed!\n", __func__);
        goto QUIT;
    }
    else
    {
        PRINT("[%s]msg Struct TypeIndex is %d\n", __func__,
            pSI->TypeIndex);
    }
    
    UINT ElementCount;
    PVOID pData = &ElementCount;
    if(!SymGetTypeInfo(GetCurrentProcess(),
        ModBase,pSI->TypeIndex,TI_GET_CHILDRENCOUNT,pData))
    {
        PRINT("[%s]err : Get Type Element Count Failed!\n",
            __func__);
        goto QUIT;
    }
    PRINT("[%s]msg : Element Count is %d\n", __func__,
        ElementCount);
    TI_FINDCHILDREN_PARAMS *pCP = malloc(sizeof(ULONG)*(2+ElementCount));
    if(!pCP) goto QUIT;
    
    memset(pCP,'\0',sizeof(ULONG)*(2+ElementCount));
    pCP->Count = ElementCount;
    if(!SymGetTypeInfo(GetCurrentProcess(),ModBase,
        pSI->TypeIndex,TI_FINDCHILDREN,pCP))
    {
        PRINT("[%s]err : Get Type Element Failed!\n", __func__);
        goto QUIT;
    }
    
    WCHAR *pNameW = NULL;
    for(int i = 0;i < ElementCount;++i)
    {
        PRINT("[%02d] TypeIndex is %d\n",i,pCP->ChildId[i]);
        if(SymGetTypeInfo(GetCurrentProcess(),ModBase,
            pCP->ChildId[i],TI_GET_SYMNAME,&pNameW))
        {
            wprintf(L"Name is %s\n",pNameW);
            LocalFree(pNameW);
        }
        else
            PrintErr();
    }
    
    bRet = true;
QUIT:
    free(pCP);
    free(pSI);
    return bRet;
}

//返回 PE 文件中指定段的文件偏移地址。
//因为 Win32k 所支持的 SSDTSdw 表在 Win32k.sys 文件中的偏移地址
//就在其数据段的文件偏移 0 处。(2k,xp,2k3,win7)
//注意在某些版本的 xp(sp2)下有问题，不排除第三方程序干扰的原因。
static PVOID GetPESectionOffset(FILE *pf, const char *SectionName)
{     
    PVOID RetDO = NULL;
    size_t SectionCount = 0;
    PVOID pNTHeader;
    IMAGE_NT_HEADERS ImgNTHeaders;
    
    if(!SectionName) return RetDO;
    //定位 IMAGE_NT_HEADERS 结构的位置信息
    fseek(pf,offsetof(IMAGE_DOS_HEADER,e_lfanew),SEEK_SET);
    //读取 IMAGE_NT_HEADERS 的位置
    fread(&pNTHeader,sizeof(long),1,pf);
    //将文件指针定位到 IMAGE_NT_HEADERS 的位置
    fseek(pf,(long)pNTHeader,SEEK_SET);
    //读取 IMAGE_NT_HEADERS 结构
    fread(&ImgNTHeaders,sizeof(IMAGE_NT_HEADERS),1,pf);
    //取得 SECTION 数目
    SectionCount = ImgNTHeaders.FileHeader.NumberOfSections;
    PRINT("[%s]msg : SectionCount is %d\n", __func__,SectionCount);
    
    //不需要显式 fseek 到 SECTION TABLE，前面 fread 已经移动到正确的偏移了。
    //fseek(pf,(long)sizeof(IMAGE_NT_HEADERS),SEEK_CUR);
    IMAGE_SECTION_HEADER ImgSecHeaders[SectionCount];
    
    //读取 SECTION 数组
    fread(ImgSecHeaders,sizeof(ImgSecHeaders),1,pf);
    for(size_t i = 0;i<SectionCount;++i)
    {
        if(strstr(ImgSecHeaders[i].Name,SectionName))
        {
            RetDO = (PVOID)ImgSecHeaders[i].PointerToRawData;
            break;
        }
    }
    return RetDO;
}

//从 Win32k.sys 中获得原始 SSDTSdw 表中的项目。
static bool GetSSDTSdwOrg(PVOID buf, size_t size)
{
    bool bRet = false;
    if(!buf || !size) goto QUIT;
    FILE *pf = fopen(g_Win32kFilePath,"rb");
    if(!pf) goto QUIT;
    
    //获取 Win32k.sys 中 data 段的文件偏移,SSDTSdw 就放在该偏移。
    PVOID SSDTSdwFileOffset = GetPESectionOffset(pf,".data");
    if(!SSDTSdwFileOffset) goto QUIT;
    
    fseek(pf,(long)SSDTSdwFileOffset,SEEK_SET);
    fread(buf,size,1,pf);
    bRet = true;
QUIT:
    if(pf) fclose(pf);
    return bRet;
}
```

其他代码请参考 `NtInfoGuy` 的源代码。

### 下载链接

`NtInfoGuy` 下载地址 : [`hopy.bokee.com/inc/NtInfoGuy.7z`](http://hopy.bokee.com/inc/NtInfoGuy.7z)  

`NtInfoGuy` 源代码下载 : 见附件  

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `NtInfoGuySRC.zip`  
|
