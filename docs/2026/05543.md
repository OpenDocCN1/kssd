
# 第二章 Dump程序的改进

> “发展才是硬道理”
>
> ——邓小平

正如我们上面一节所做的，一个 `dump` 的雏形已经形成了，现在要做的是在原来的基础上做进一步的改进。

## 一、获取 `ImageofSize`

相信大家也感觉到了，在前面一章介绍的获取 `ImageofSize` 的办法实在很不可靠，对于一个加壳程序随意地修改它载入内存中的 PE 头，是很稀疏平常的事情，所以用读取目标进程中的 PE 头并不是一个好主意。

现在我就来介绍一下一般的办法。对于 LordPE 它的 dump 是采用什么方法呢？它采用了 `Module32Next` 来获取 dump 的进程的基本信息的。

```
BOOL WINAPI Module32First(
  HANDLE hSnapshot, //这是先前的CreateToolhelp32Snapshot函数返回的快照
  LPMODULEENTRY32 lpme //这个是指向MODULEENTRY32结构的指针
);
```

下面是 `MODULEENTRY32` 结构：

```
typedef struct tagMODULEENTRY32 {
  DWORD dwSize;
  DWORD th32ModuleID;
  DWORD th32ProcessID;
  DWORD GlblcntUsage;
  DWORD ProccntUsage;
  BYTE *modBaseAddr;
  DWORD modBaseSize;    //这个是我们要获取的关键
  HMODULE hModule;
  TCHAR szModule[MAX_PATH];
  TCHAR szExePath[MAX_PATH];
  DWORD dwFlags;
} MODULEENTRY32, *PMODULEENTRY32, *LPMODULEENTRY32;
```

下面就给出重新写的新的 `GetSizeOfImage` 的函数：

```
int GetSizeOfImage(HWND hDlg,DWORD IDProcess)
{
  //这个函数的作用是获取SizeOfImage的数值
  //当函数执行失败返回的是0
  //成功返回的是非0
  HANDLE hModuleSnap = NULL;
  MODULEENTRY32 stModE  = {0};
  stModE.dwSize = sizeof(MODULEENTRY32);
  hModuleSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE,IDProcess);  //快照，对本进程中所有的模块进行snap
  if (hModuleSnap == INVALID_HANDLE_VALUE)
  {
    MessageBox(hDlg,TEXT("The Module snapshot can't get!"),TEXT("Error!"),MB_OK | MB_ICONSTOP);
      return FALSE;    //返回0
  }
  if (!Module32First(hModuleSnap, &stModE))
  {
     MessageBox(hDlg,TEXT("The Module32First can't work!"),TEXT("Error!"),MB_OK | MB_ICONSTOP);
     CloseHandle (hModuleSnap);
     return FALSE;
  }
  CloseHandle (hModuleSnap);
  return stModE.modBaseSize;//初始化为0
}
```

## 二、对齐节表

这个问题在上一节我们已经提出了，我们的程序不完善，现在要自动地实现 `RA=RVA`、`RS=RVS` 这个功能。那么我们就使用一个函数来完成它吧！我定义了下面这个函数：

```
BOOL ModifySectionFunc(HWND hDlg,LPCTSTR Dump_Name)
{
  //此函数的将修改dump下来的exe，使其RA=RVA ，RS=RVS
  //首先是打开dump文件
  HANDLE hFile=CreateFile(Dump_Name,GENERIC_WRITE | GENERIC_READ,0,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
  if(hFile==INVALID_HANDLE_VALUE)
  {
    MessageBox(hDlg,TEXT("I can open the dump file..."),TEXT("Error!!"),MB_OK | MB_ICONWARNING);
    return FALSE;
  }
    //下面移动到节表前面
  IMAGE_DOS_HEADER myDosHeader;
  DWORD NumberOfBytesRead;
  ReadFile(hFile,(LPVOID)&myDosHeader,sizeof(IMAGE_DOS_HEADER),&NumberOfBytesRead,NULL);
  SetFilePointer(hFile,myDosHeader.e_lfanew+sizeof(DWORD),NULL,FILE_BEGIN);
    IMAGE_FILE_HEADER myNtHeader;
  ReadFile(hFile,(LPVOID)&myNtHeader,sizeof(IMAGE_FILE_HEADER),&NumberOfBytesRead,NULL);
  int nSectionCount;
  nSectionCount = myNtHeader.NumberOfSections;             // 保存Section个数
  // 过了IMAGE_NT_HEADERS结构就是IMAGE_SECTION_HEADER结构数组了，注意是结构数组，有几个Section该结构就有几个元素
  // 这里动态开辟NumberOfSections个内存来存储不同的Section信息
  IMAGE_SECTION_HEADER *pmySectionHeader = (IMAGE_SECTION_HEADER *)calloc(nSectionCount, sizeof(IMAGE_SECTION_HEADER));
  SetFilePointer(hFile,myDosHeader.e_lfanew + sizeof(IMAGE_NT_HEADERS),NULL,FILE_BEGIN);
    ReadFile(hFile,(LPVOID)pmySectionHeader,sizeof(IMAGE_SECTION_HEADER)*nSectionCount,
         &NumberOfBytesRead,NULL);
  //移动回到节表的开始，准备写入
  SetFilePointer(hFile,myDosHeader.e_lfanew + sizeof(IMAGE_NT_HEADERS),NULL,FILE_BEGIN);
  for (int i = 0; i < nSectionCount; i++, pmySectionHeader++)
  {
    //将RA=RVA ，RS=RVS
    pmySectionHeader->SizeOfRawData=pmySectionHeader->Misc.VirtualSize;
    pmySectionHeader->PointerToRawData=pmySectionHeader->VirtualAddress;
    //将修改好的数值写回
    WriteFile(hFile,(LPVOID)pmySectionHeader,sizeof(IMAGE_SECTION_HEADER),&NumberOfBytesRead,NULL);
  }
  // 恢复指针
  pmySectionHeader -=nSectionCount;
  if (pmySectionHeader != NULL)          // 释放内存
  {
    free(pmySectionHeader);
    pmySectionHeader = NULL;
  }
  // 最后不要忘记关闭文件
  CloseHandle(hFile);
  return TRUE;
}
```

这个函数是在生成了一个新的 dump 文件以后，然后打开它，找到节表，把里面的数据修改过来。

## 三、小结

这样，一个改进型的程序就搞定了。现在当我们用它 dump 下新的程序的时候得到的就是一个有图标的程序了。因为我们已经把节表修改过来了。下面一章我们将介绍 anti-dump 的技巧。

[附件：实例下载](https://www.kanxue.com/pediy7-659/dump_chap02.rar)
