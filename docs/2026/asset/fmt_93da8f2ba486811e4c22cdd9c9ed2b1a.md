
# [原创]一种保护应用程序的方法 模拟Windows PE加载器，从内存资源中加载DLL

一种保护应用程序的方法-模拟Windows PE加载器，从内存资源中加载DLL

作者：老实和尚  coverlove@163.com

## 1、前言

目前很多敏感和重要的`DLL`(`Dynamic-link library`) 都没有提供静态版本供编译器进行静态连接(`.lib`文件)，即使提供了静态版本也因为兼容性问题导致无法使用，而只提供`DLL`版本，并且很多专业软件的授权部分的`API`，都是单独提供一个`DLL`来完成，而主模块通过调用`DLL`中的接口来完成授权功能。虽然这些软件一般都采用了加壳和反调试等保护，但是一旦这些功能失去作用，比如脱壳，反反调试，`HOOK API`或者干脆写一个仿真的授权`DLL`（模拟授权`DLL`的所有导出函数接口），然后仿真的`DLL`再调用授权`DLL`,这样所有的输入首先被仿真`DLL`截获再传递给授权`DLL`，而授权`DLL`的输出也首先传递给仿真`DLL`再传递给主程序，这样就可以轻易的监视二者之间的输入输出之间的关系，从而轻易的截获`DLL`中的授权信息进行修改再返回给主程序。

## 2、目前隐式调用敏感DLL中可能存在的安全隐患

以下通过两个软件的授权`DLL`来说明这种问题的严重性。如下是两个软件中授权`DLL`的部分信息，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201204/37188_vzv3mpblbgo2xp9.jpg)

（图１）

通过工具`OllyICE`可以轻易的看出`IoMonitor.exe`调用授权`DLL`(`XKeyAPI.DLL`)，这样就很容易在调用这些`API`的地方设置断点，然后判断输入输出的关系，从而达到破解的目的。

![](https://bbs.kanxue.com/upload/attach/201204/37188_u9e2mq6odt8srid.jpg)

（图２）

通过工具`OllyICE`可以轻易的看出`sfeng.DLL`中导出了很多函数，其中含义也很明显。`GetHDID`获取硬盘的`ID`，`GetCpuId`获取`cpu`的`ID`，`WinAntiDebug`反调试接口。而这些都是主程序需要调用的，比如：主程序通过`GetHDID`来获取硬盘编码，以这个硬盘`ID`的伪码来生成授权码，破解者很容易修改这些接口的输出值或者干脆写一个`sfeng.DLL`来导出跟目标`sfeng.DLL`一模一样的导出函数，而主程序却完全不知晓。只要用户有一套授权码就可以让`GetHDID`不管什么机器都返回一样的值，从而达到任何机器都可以使用同一套授权码。

![](https://bbs.kanxue.com/upload/attach/201204/37188_as4fuvhnjc0syvg.jpg)

（图３）

如上图所示，直接修改`DLL`中函数`GetHDID(RVA`地址:`0093FF3C`开始)的实现，让它直接返回固定的硬盘`ID`就可以达到一个授权到处使用的目的。其中：”`WD-Z=AM9N086529ksaiy`“为需要返回的已经授权的硬盘`ID`,我们直接返回这个值即可。把原来`0093FF3C`部分的代码用`nop`替换掉，添加`Call 008FFF60`，后面添加字符串”`WD-Z=AM9N086529ksaiy`”，`Call 008FFF60`之后，`ESP=Call`后的返回地址(`Call`指令的下一行)，也就是字符串”`WD-Z=AM9N086529ksaiy`“的首地址，然后`pop EAX`后，返回值就是字符串的首地址，通过这种简单的修改就可以达到破解的目的，说明这种隐式的调用是非常危险的。

## 3、模拟Windows PE加载器，从资源中加载DLL

本文主要介绍将`DLL`文件进行加密压缩后存放在程序的资源段，然后在程序中读取资源段数据进行解压和解密工作后，从内存中加载这个`DLL`，然后模拟`PE`加载器完成`DLL`的加载过程。本文主要以`Visual C++ 6.0`为工具进行介绍，其它开发工具实现过程与此类似。

这样作的好处也很明显，`DLL`文件存放在主程序的资源段，而且经过了加密压缩处理，破解者很难找到下断点的地方，也不能轻易修改资源`DLL`，因为只有主程序完成解压和解密工作，完成`PE`加载工作后此`DLL`才开始工作。

我们知道，要显式加载一个`DLL`，并取得其中导出的函数地址一般是通过如下步骤：

1. 用`LoadLibrary`加载`DLL`文件，获得该`DLL`的模块句柄；
2. 定义一个函数指针类型，并声明一个变量；
3. 用`GetProcAddress`取得该`DLL`中目标函数的地址，赋值给函数指针变量；
4. 调用函数指针变量。

这个方法要求`DLL`文件位于硬盘上面,而我们的`DLL`现在在内存中。现在假设我们的`DLL`已经位于内存中，比如通过脱壳、解密或者解压缩得到，能不能不把它写入硬盘文件，而直接从内存加载呢？答案是肯定的，方法就是完成跟`Windows PE`加载器同样的工作即可。

加载过程大致包括以下几个部分：

1. 调用`API`读取`DLL`资源数据拷贝到内存中
2. 调用解压和解密函数对内存中的`DLL`进行处理
3. 检查`DOS`头和`PE`头判断是否为合法的`PE`格式
4. 计算加载该`DLL`所需的虚拟地址空间大小
5. 向操作系统申请指定大小的虚拟地址空间并提交
6. 将`DLL`数据复制到所分配的虚拟内存块中，注意文件段对齐方式和内存段对齐方式
7. 对每个 `DLL`文件来说都存在一个重定位节（`.reloc)`，用于记录`DLL`文件的重定位信息，需要处理重定位信息
8. 读取`DLL`的引入表部分，加载引入表部分需要的`DLL`，并填充需要的函数入口的真实地址
9. 根据`DLL`每个节的属性设置其对应内存页的读写属性
10. 调用入口函数`DLLMain`，完成初始化工作
11. 保存`DLL`的基地址（即分配的内存块起始地址）,用于查找`DLL`的导出函数
12. 不需要`DLL`的时候，释放所分配的虚拟内存，释放所有动态申请的内存

以下部分分别介绍这几个步骤，以改造过的网上下载的`CMemLoadDLL`类为例程（原类存在几个错误的地方）

### A. 调用API读取DLL资源数据拷贝到内存中

```c
//加载资源DLL
#define strKey (char)0x15
char DLLtype[4]={'D' ^ strKey ,'l'^ strKey,'l'^ strKey,0x00};
HINSTANCE hinst=AfxGetInstanceHandle();
HRSRC hr=NULL;
HGLOBAL hg=NULL;
//对资源名称字符串进行简单的异或操作，达到不能通过外部字符串参考下断点
for(int i=0;i<sizeof(DLLtype)-1;i++)
{
        DLLtype[i]^=strKey;
}
hr=FindResource(hinst,MAKEINTRESOURCE(IDR\_DLL),TEXT(DLLtype));
if (NULL == hr) return FALSE;
//获取资源的大小
DWORD dwSize = SizeofResource(hinst, hr);
if (0 == dwSize) return FALSE;
hg=LoadResource(hinst,hr);
if (NULL == hg) return FALSE;
//锁定资源
LPVOID pBuffer =(LPSTR)LockResource(hg);
if (NULL == pBuffer) return FALSE;
FreeResource(hg);
```

### B. 调用解压和解密函数对内存总的DLL进行处理

对于上面获取的`pBuffer`可以进行解压和解密操作，算法应该跟你加入的资源采取的算法进行逆变换即可，具体算法可以自己选择，此处省略。

### C. 检查DOS头和PE头判断是否为合法的PE格式

```c
BOOL CMemLoadDLL::CheckDataValide(void* lpFileData, int DataLength)
{
        if(DataLength < sizeof(IMAGE\_DOS\_HEADER)) return FALSE;
        pDosHeader = (PIMAGE\_DOS\_HEADER)lpFileData;
        if(pDosHeader->e\_magic != IMAGE\_DOS\_SIGNATURE) return FALSE;
        if((DWORD)DataLength < (pDosHeader->e\_lfanew + sizeof(IMAGE\_NT\_HEADERS)) ) return FALSE;
        pNTHeader = (PIMAGE\_NT\_HEADERS)( (unsigned long)lpFileData + pDosHeader->e\_lfanew);
        if(pNTHeader->Signature != IMAGE\_NT\_SIGNATURE) return FALSE;
        if((pNTHeader->FileHeader.Characteristics & IMAGE\_FILE\_DLL) == 0)
        return FALSE;
        if((pNTHeader->FileHeader.Characteristics & IMAGE\_FILE\_EXECUTABLE\_IMAGE) == 0)
        return FALSE;
        if(pNTHeader->FileHeader.SizeOfOptionalHeader != sizeof(IMAGE\_OPTIONAL\_HEADER)) return FALSE;
        pSectionHeader = (PIMAGE\_SECTION\_HEADER)((int)pNTHeader + sizeof(IMAGE\_NT\_HEADERS));
        for(int i=0; i< pNTHeader->FileHeader.NumberOfSections; i++)
        {
if((pSectionHeader[i].PointerToRawData + pSectionHeader[i].SizeOfRawData) > (DWORD)DataLength)return FALSE;
        }
        return TRUE;
}
```

### D. 计算加载该DLL所需的虚拟地址空间大小

```c
int CMemLoadDLL::CalcTotalImageSize()
{
        int Size;
        if(pNTHeader == NULL)return 0;
        int nAlign = pNTHeader->OptionalHeader.SectionAlignment;
        Size = GetAlignedSize(pNTHeader->OptionalHeader.SizeOfHeaders, nAlign);
        for(int i=0; i < pNTHeader->FileHeader.NumberOfSections; ++i)
        {
                int CodeSize = pSectionHeader[i].Misc.VirtualSize ;
                int LoadSize = pSectionHeader[i].SizeOfRawData;
                int MaxSize = (LoadSize > CodeSize)?(LoadSize):(CodeSize);
                int SectionSize = GetAlignedSize(pSectionHeader[i].VirtualAddress + MaxSize, nAlign);
                if(Size < SectionSize)
                Size = SectionSize;
        }
        return Size;
}
int CMemLoadDLL::GetAlignedSize(int Origin, int Alignment)
{
        return (Origin + Alignment - 1) / Alignment * Alignment;
}
```

### E. 向操作系统申请指定大小的虚拟地址空间并提交

```c
void *pMemoryAddress=VirtualAlloc((LPVOID)NULL, ImageSize,MEM\_COMMIT|MEM\_RESERVE, PAGE\_EXECUTE\_READWRITE);
        if(pMemoryAddress == NULL)
        {
                return FALSE;
        }
```

### F. 将DLL数据复制到所分配的虚拟内存块中，注意文件段对齐方式和内存段对齐方式

```c
void CMemLoadDLL::CopyDLLDatas(void* pDest, void* pSrc)
{
        int HeaderSize = pNTHeader->OptionalHeader.SizeOfHeaders;
        int SectionSize = pNTHeader->FileHeader.NumberOfSections * sizeof(IMAGE\_SECTION\_HEADER);
        int MoveSize = HeaderSize + SectionSize;
        memmove(pDest, pSrc, MoveSize);
for(int i=0; i < pNTHeader->FileHeader.NumberOfSections; ++i)
{
        if(pSectionHeader[i].VirtualAddress == 0 || pSectionHeader[i].SizeOfRawData == 0) continue;
        void *pSectionAddress = (void *)((unsigned long)pDest + pSectionHeader[i].VirtualAddress);
        memmove((void *)pSectionAddress,
        (void *)((DWORD)pSrc + pSectionHeader[i].PointerToRawData),
        pSectionHeader[i].SizeOfRawData);
}
        pDosHeader = (PIMAGE\_DOS\_HEADER)pDest;
```
