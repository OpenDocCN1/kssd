
# 菜鸟第一次 DIY, 为你的软件加载皮肤

> 原文：[`www.pediy.com/kssd/pediy11/104318.html`](https://www.pediy.com/kssd/pediy11/104318.html)

【文章标题】: 菜鸟第一次 DIY, 为你的软件加载皮肤  
【文章作者】: blueapplez  
【作者邮箱】: blueapple1987@163.com  
【软件名称】: War3 改键工具 1.02  
【下载地址】: 附件  
【操作平台】: Windows XP SP2  
【完成时间】:2010-1-2 2:58:11  
【文章链接】:[`bbs.pediy.com/showthread.php?p=737611`](http://bbs.pediy.com/showthread.php?p=737611)  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

今晚刚修改了不久之前做的一个 War3 改键工具，看了安静 a 发的一篇求助帖《如何给一个写好的软件加个皮肤！》，后来有人回复说加载一些开源的皮肤库就行了，于是我就动手 DIY 我的刚完成的小作品了！

## 详细过程

### (一) 找到一个 Appface 的一个开源库，查了一下里面的调用过程

以前我用的时候是用静态库这样用：

代码:

```
SkinStart(_T("Skin\\sap_af.urf"),WINDOW_TYPE_VC,"",GTP_LOAD_FILE,NULL,NULL);
```

然后我修改了一下动态加载他，并且把参数 和宏都替换掉。

代码:

```
  typedef BOOL   (__stdcall *APPFACE_START)(char*,int,char*,int,DWORD,char*) ;
  
  HMODULE hDll = LoadLibrary("1.dll");
  if (hDll != NULL)
  {
    APPFACE_START fun = (APPFACE_START)GetProcAddress(hDll, "SkinStart");
    if (fun != NULL)
    {
      fun(_T("1.urf"),1,"",1,NULL,NULL);
    }
  }
```

测试一下通过！

### (二) 用之前学过的 PE 文件格式的知识，为我的小软件增加一个 Section 区段

设置属性为可读可写可执行，增加 Section 段的 C++代码如下：

新增一个 Section。

首先，我们需要在 PE 头内找到 `NumberOfSections`,使之加 1。

然后，在文件末尾增加一个新的空间，假设为 200h，记住起始行到 PE 文件首部的偏移。假如这个值是 00034500h。同时将 PE 头内的 `SizeOfImage` 的值加 200h。

然后，找到 PE 头内的 Section 头部。通常在 Section 头部结束到 Section 数据部分开始间会有一些空间，找到 Section 头部的最后然后加入一个新的头部。假设最后一个 Section 头部的数据是：

Virtual offset : 34000h  
Virtual size : 8E00h  
Raw offset: 2F400h  
Raw size : 8E00h

而文件对齐和 Section 对齐的数据分别是：

Section Alignment : 1000h  
File Alignment : 200h

那么新增加的 Section 必须与最后一个 Section 的边界对齐。它的数据分别：

Virtual offset : 3D000h (因为最后一个 Section 的最后边界是 34000h + 8E00h = 3CE00h,加上 Section 对齐，则 Virtual offset 的值为 3D000h)。  
Virtual size : 200h。  
Raw offset: 00034500h。  
Raw size: 200h.  
Characteristics : E0000060 (可读、可写、可执行)。这地方最好用 windows 提供的宏定义来设定 `IMAGE_SCN_MEM_EXECUTE|IMAGE_SCN_MEM_READ|IMAGE_SCN_MEM_WRITE`

6. 最后，只需要修改一下 PE 头内的 `SizeOfCode` 和 `SizeOfInitialishedData` 两个字段，分别加上 200h。

（上面这段介绍是我抄来的，出处已经找不到了，- -！）

需要注意的问题是 修改过后的 PE 的入口地址是我们新添加上的 Section 段的 `Virtual offset + ImageBase`，也就是修改完 PE 文件是不能执行的，需要你先记下他之前的入口点，然后用 OD 加一个 `jmp` 跳回去就 ok 了

下面代码：

代码:

```
//Get the alignment Address
DWORD Alignment(DWORD dwValue, DWORD dwDuiQ)
{
while(dwValue % dwDuiQ != 0)
{
dwValue++;
}
return dwValue;
}

//Modify the PE file, include Add new Section and sth else
DWORD SetPeInfo(void *ImageOfPe, DWORD dwFileSize, DWORD dwOffset)
{
WORD NumOfSection=0;
DWORD RawOfEntryPoint=0;
DWORD AddressOfEntryPoint=0;

PIMAGE_DOS_HEADER   pDosHead=NULL;
PIMAGE_NT_HEADERS   pNtHead=NULL;
PIMAGE_SECTION_HEADER pSectionHead=NULL;
pDosHead=(PIMAGE_DOS_HEADER)ImageOfPe;
pNtHead=(PIMAGE_NT_HEADERS)((unsigned char *)pDosHead+pDosHead->e_lfanew);

//Record the old PE EnterPoint, we will modify it later.
AddressOfEntryPoint=pNtHead->OptionalHeader.AddressOfEntryPoint;
//Record the number of the section segment, we will modify it later.
NumOfSection=pNtHead->FileHeader.NumberOfSections;
//the first section's address
pSectionHead=(PIMAGE_SECTION_HEADER)((unsigned char *)pNtHead+
sizeof(pNtHead->Signature)+sizeof(IMAGE_FILE_HEADER)+
pNtHead->FileHeader.SizeOfOptionalHeader);

//Get The Last Section Pointer
PIMAGE_SECTION_HEADER pLastSection = (IMAGE_SECTION_HEADER *)((DWORD)pSectionHead + sizeof(IMAGE_SECTION_HEADER)*(NumOfSection-1));
IMAGE_SECTION_HEADER imageSectionInsert;
memset(&imageSectionInsert, 0, sizeof(IMAGE_SECTION_HEADER));
//Write sth to the new Section
//set a name for the new section
memcpy(imageSectionInsert.Name, ".blue", sizeof(".blue"));
imageSectionInsert.Misc.VirtualSize = 0x200;
imageSectionInsert.VirtualAddress = Alignment(pLastSection->VirtualAddress + pLastSection->Misc.VirtualSize, 0x1000);
imageSectionInsert.SizeOfRawData = 0x200;
imageSectionInsert.PointerToRawData = dwFileSize + dwOffset;

//Set Characteristics to read write and execute.
imageSectionInsert.Characteristics = IMAGE_SCN_MEM_EXECUTE|IMAGE_SCN_MEM_READ|IMAGE_SCN_MEM_WRITE ;
//the add 1, in fact add a sizeof IMAGE_SECTION_HEADER
memcpy(pLastSection + 1, &imageSectionInsert, sizeof(IMAGE_SECTION_HEADER));

//1.Modify the PE Entrypoint
pNtHead->OptionalHeader.AddressOfEntryPoint = imageSectionInsert.VirtualAddress;
//2.Add the number of section segment
pNtHead->FileHeader.NumberOfSections++;
//3.Add 0x200 to SizeofImage
pNtHead->OptionalHeader.SizeOfImage += 0x200;
//4.Add 0x200 to SizeOfCode
pNtHead->OptionalHeader.SizeOfCode += 0x200;
//5.Add 0x200 to SizeOfInitialishedData
pNtHead->OptionalHeader.SizeOfInitializedData += 0x200;

//return the new Entrypoint and the ImageBase.
return AddressOfEntryPoint + pNtHead->OptionalHeader.ImageBase;
}

void CMyDlg::OnOK() 
{
DWORD dwFileSize = 0;
HANDLE hFile = ::CreateFile("a.exe", GENERIC_READ|GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL | FILE_FLAG_SEQUENTIAL_SCAN, NULL);
if (hFile == INVALID_HANDLE_VALUE)
{
return ;
}
dwFileSize = GetFileSize(hFile, NULL);
if (dwFileSize == 0)
{
return;
}
char *szpPE = new char [dwFileSize];
memset (szpPE, 0, dwFileSize);
DWORD dwRead = 0;
ReadFile(hFile, szpPE, dwFileSize, &dwRead, NULL);
if (dwRead != dwFileSize)
{
return ;
}
CloseHandle(hFile);

//Set PE Info
DWORD dwOffset = Alignment(dwFileSize, 0x200) - dwFileSize;
DWORD dwEnterLastPoint = SetPeInfo(szpPE, dwFileSize, dwOffset);

//Add 200 Bytes and create a new file
hFile = ::CreateFile("b.exe", GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL | FILE_FLAG_SEQUENTIAL_SCAN, NULL);
if (hFile == INVALID_HANDLE_VALUE)
{
return ;
}
BYTE *sz200Size = new BYTE[0x200 + dwOffset];
memset(sz200Size, 0, 0x200+dwOffset);
//    Write binary codes here, but do not forget jmp to the old EntryPoint.
//     sz200Size[dwOffset] = 0xe9;
//     sz200Size[dwOffset+1] = 0xbb;
//     sz200Size[dwOffset+2] = 0x7b;
//     sz200Size[dwOffset+3] = 0xFF;
//     sz200Size[dwOffset+4] = 0xFF;

//set the file size
SetFilePointer(hFile, dwFileSize + dwOffset + 0x200, NULL, FILE_BEGIN);
//set file to the start position
SetFilePointer(hFile, NULL, NULL, FILE_BEGIN);
DWORD dwFeiWu = 0;
//write modified szpPE info to new file.
WriteFile(hFile, szpPE, dwFileSize, &dwFeiWu, NULL);
//write the 0x200 section code to new file, do not forget the offset file size.
WriteFile(hFile, sz200Size, dwOffset + 0x200, &dwFeiWu, NULL);
CloseHandle(hFile);

//release the memory we used.
delete [] sz200Size;
delete [] szpPE;
}
```

好了修改一下，在新入口点加一个 `jmp original Address`，保存，能正常运行，ok，进行下一步。

PS:这段代码是我查阅相关资料写出来的，文件处理的不好，最好用文件映射来做，对有些 PE 文件就会失效，我不知道为什么，期待有人来解释解释是为什么。

### (三) 下面开始加载我们的皮肤库了

用 OD 载入，从第二步最后加的 `jmp` 开始修改

代码:
