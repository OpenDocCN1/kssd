
# [原创]元宵节献礼，用类的思想处理PE结构附源码

最近在论坛里接了个业务，客户看了我的ID后说，我发的都是求助帖，对我的能力很质疑，我自己一看才发现，真是除了几年前发了个破解的帖子蒙混过关骗得一个邀请码之外，一直在做伸手党![](https://bbs.kanxue.com/view/img/face/50.gif)，于是翻出了之前给赏金论坛做过的一套《壳的编写》的教程，因为做的时候答应人家不能再出售，所以就不发教程了，只是把里面的源码拿来用下。![](https://bbs.kanxue.com/view/img/face/1.gif)

一直以来我都是以开发为主，类的思想在我脑海里根深蒂固，几年前研究PE结构的时候发现，PE结构完全可以用类的思想处理，于是写了几个处理的代码，并用这个思想写了个简单的加壳程序，可以给`notepad.exe`加壳，仅供参考，现把代码贴上献丑。

---------------------------------第二次编辑---------------------------------------------------------

其实主要想法就是减少重复劳动，做到代码最大限度的重用，把功能模块化，比如得到NT头，这个功能很多数据都是通过这里开始的，这样就不用每次都重复这个代码，直接调用类中的这个代码就可以了，特别是利用C++的集成功能，可以在继承的子类中使用这部分代码，而对于子类这个代码是不可见的，功能可以用类区分开，代码非常容易管理。下面贴出个别核心代码供大家参考。

```c
//基本的PE处理 可以得到dos头 NT头 节 等信息
class LPEFile
{
public:
  BOOL LoadPEFile(BYTE* lpBuffer,DWORD dwSize);          //载入PE文件
  LPVOID RvaToPtr(DWORD dwRVA);                          //计算偏移
  LPVOID GetDirectoryEntryToData(USHORT DirectoryEntry);
  PIMAGE_SECTION_HEADER GetSection(int nIndex);           //得到节的信息
  BOOL IsPEFile();
  PIMAGE_SECTION_HEADER GetFirstSectionHeader();         //得到第一个节
  int GetSectionCount();                                 //得到节的个数
  PIMAGE_DATA_DIRECTORY GetDataDirectory(int nIndex);     //得到数据目录
  PIMAGE_DATA_DIRECTORY GetDataDirectory();               //得到数据目录
  PIMAGE_OPTIONAL_HEADER GetNtOptionalHeader();
  PIMAGE_FILE_HEADER GetNtFileHeader();
  PIMAGE_NT_HEADERS GetNtHeader();
  PIMAGE_DOS_HEADER GetDosHeader();
  DWORD GetImageSize();
  BYTE* GetImage();
  bool Free();
  BOOL LoadPEFile(char* lpFilename);
  BOOL SetInfo(MAP_FILE_STRUCT *pMapFileMsg);
  MAP_FILE_STRUCT* GetInfo();
  void operator=(LPEFile& lPeFile);
  LPEFile();
  virtual ~LPEFile();

private:
  //用于标识文件的打开状态//用于标识文件的打开状态
    BYTE bLoadStact;
    //用于存储打开的文件句柄，文件数据，文件大小

protected:
    MAP_FILE_STRUCT *pstMapFile;
};
```

```c
//用于将文件载入内存
BOOL LPEFile::LoadPEFile(char *lpFilename)
{
	HANDLE hFile;
	HANDLE hMapping;
	LPVOID ImageBase;
    pstMapFile=new MAP_FILE_STRUCT;
	bLoadStact=LFILE_FROM_FILE;

	hFile=CreateFile(lpFilename,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,NULL,OPEN_ALWAYS,
		FILE_ATTRIBUTE_NORMAL,NULL);
	if (hFile==NULL)
	{
		return FALSE;
	}
	pstMapFile->dwFileSize=GetFileSize(hFile,NULL);
	hMapping=CreateFileMapping(hFile,NULL,PAGE_READWRITE,0,pstMapFile->dwFileSize,NULL);
	if (hMapping==NULL)
	{
		CloseHandle(hFile);
		return FALSE;
	}

	ImageBase=MapViewOfFile(hMapping,FILE_MAP_ALL_ACCESS,0,0,pstMapFile->dwFileSize);
	if (ImageBase==NULL)
	{
		CloseHandle(hFile);
		CloseHandle(hMapping);
		return FALSE;
	}
	pstMapFile->hFile=hFile;
	pstMapFile->hMapping=hMapping;
	pstMapFile->ImageBase=ImageBase;
   return TRUE;
}
```

```c
//得到文件的dos头
PIMAGE_DOS_HEADER LPEFile::GetDosHeader()
{
   return (PIMAGE_DOS_HEADER)GetImage();
}
```

```c
//得到文件的NT头，这里就不需要从得到Dos开始了，直接调用编写好的类函数
PIMAGE_NT_HEADERS LPEFile::GetNtHeader()
{
    PIMAGE_DOS_HEADER pDH=GetDosHeader();
	if (pDH!=NULL)
	{
		return (PIMAGE_NT_HEADERS32)((DWORD)GetImage()+pDH->e_lfanew);
	}
	return NULL;
}
```

```c
//得到文件头
PIMAGE_FILE_HEADER LPEFile::GetNtFileHeader()
{
   PIMAGE_NT_HEADERS pNth=GetNtHeader();
   if (pNth!=NULL)
   {
	   return (PIMAGE_FILE_HEADER)&pNth->FileHeader;
   }
   return NULL;
}
```

```c
//得到可选头
PIMAGE_OPTIONAL_HEADER LPEFile::GetNtOptionalHeader()
{
	PIMAGE_NT_HEADERS pNth=GetNtHeader();
	if (pNth!=NULL)
	{
		return (PIMAGE_OPTIONAL_HEADER32)&pNth->OptionalHeader;
	}
   return NULL;
}
```

```c
//得到数据目录
PIMAGE_DATA_DIRECTORY LPEFile::GetDataDirectory()
{
   PIMAGE_OPTIONAL_HEADER pOptionalHeader=GetNtOptionalHeader();
   if (pOptionalHeader!=NULL)
   {
	   return (PIMAGE_DATA_DIRECTORY) pOptionalHeader->DataDirectory;
   }
   return NULL;
}
```

```c
PIMAGE_DATA_DIRECTORY LPEFile::GetDataDirectory(int nIndex)
{
    if (nIndex>=0&&nIndex<16)
    {
        PIMAGE_DATA_DIRECTORY pDataDir=GetDataDirectory();
		if (pDataDir!=NULL)
		{
				return &pDataDir[nIndex];
		}
    }
	return NULL;
}
```

有了基本的PE处理之后，就可以在类上做继承，扩展新功能，功能上独立，代码简洁。
