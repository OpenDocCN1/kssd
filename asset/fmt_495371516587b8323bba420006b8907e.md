
# 被 Worm.Win32.Viking.ix 搞疯了，马虎写了个东东，分享一下

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-583.htm`](https://www.pediy.com/kssd/pediy09/pediy09-583.htm)

> 【文章标题】: 被已 `Worm.Win32.Viking.ix` 搞疯了，马虎写了个东东，分享一下  
> 【文章作者】: `llydd`  
> 【作者邮箱】: `llydd@163.com`  
> 【下载地址】: 自己搜索下载  
> 【使用工具】: `peid`, `vc++6`, `winhex`  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

> 【详细过程】  

> 前些天突然发现进程里出现些怪进程像 `Logo1\_.exe`, `rund1l32.exe`, `iexp1ore.exe` 之类的，一看就知道不是什么好鸟  
> 注册表启动项里都出现了那些东东，  
> 本以为把它们从系统里删除了就不会出现问题了，系统里差不多每个 `EXE` 文件都被搞的面目全非了，图标都变没了  
> 找了个以前写的测试程序用 `PEID` 查了一下发现区块名都变成 `lw0x60`, `lw0x61` 之类的，运行一下发现程序半天没反应  
> 过了一会才运行，突然发现只要运行那些被感染的文件后，被感染的程序会被变成原先被感染的文件，突然想到正常程序可能被捆绑到病毒体后才出现这种情况，一旦运行感染后的程序，病毒体会先执行，然后从被感染文件中剥离并修改注册表启动项，释放 `uninstall` 在 `c:\\winnt\\install` 目录，释放 `rund1l32.exe` 在 `c:\\winnt\\` 目录释放 `Logo1\_.exe`  
> 找到这些东东都把它们删了，清除注册表，但每个 `EXE` 都被感染了，我总不能每个 `EXE` 都运行一遍，让它们恢复正常吧  
> 装杀毒软件吧，偶的老机子跑不动，`AMD K7` 的老机机了～～～，权当练练手，写的清除工具  

```c
#include "windows.h"
#include "stdio.h"
#include "string.h"

int g_Count_File = 0;

void list(char *cPath)
{
    WIN32_FIND_DATA wfd;
    HANDLE hFind;
    char cDir[MAX_PATH];
    char cSubDir[MAX_PATH];

    ::GetCurrentDirectory(sizeof(cDir), cDir);
    ::SetCurrentDirectory(cPath);

    hFind = ::FindFirstFile("*.*", &wfd);
    if (hFind == INVALID_HANDLE_VALUE)
    {
        return;
    }

    do
    {
        // 判断是否是目录
        if (wfd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
        {
            if (wfd.cFileName[0] != '.')
            {
                // 将该目录名与当前路径合并成目录路径
                ::wsprintf(cSubDir, "%s\\%s", cPath, wfd.cFileName);
                list(cSubDir);
            }
        }
        else
        {
            HANDLE hFile = ::CreateFile(
                wfd.cFileName,
                GENERIC_READ | GENERIC_WRITE,
                FILE_SHARE_READ | FILE_SHARE_WRITE,
                NULL,
                OPEN_EXISTING,
                FILE_ATTRIBUTE_NORMAL,
                NULL);

            if (hFile == INVALID_HANDLE_VALUE)
                continue;

            HANDLE hMap = ::CreateFileMapping(
                hFile,
                NULL,
                PAGE_READWRITE,
                0,
                0,
                NULL);

            void *pAdd = ::MapViewOfFile(
                hMap,
                FILE_MAP_ALL_ACCESS,
                0,
                0,
                0);

            if (hMap == 0)
                continue;

            DWORD *pPe;
            PIMAGE_SECTION_HEADER pSec;
            WORD nSecNumber = 0;
            DWORD dNewFileSize = 0;
            DWORD *pNewFile;

            // 判断是否文件首字为'MZ'
            if (*(WORD *)pAdd == 0x5a4d)
            {
                pPe = (DWORD *)*((DWORD *)((char *)pAdd + 0x3c));
                pPe = (DWORD *)((DWORD)pAdd + (DWORD)pPe);

                // 获取文件大小,以免得到一些文件首字为'MZ'但偏移 3C 处的双字段值超出内存映像文件范围
                dNewFileSize = ::GetFileSize(hFile, &dNewFileSize);

                // 判断是否'PE\0\0'
                if ((DWORD)pPe <= ((DWORD)pAdd + dNewFileSize) && *pPe == 0x004550)
                {
                    nSecNumber = *(WORD *)((char *)pPe + 6);
                    pSec = (PIMAGE_SECTION_HEADER)((char *)pPe + 248);

                    // 定位到第一个块表，判断其区名是否为 lw0x60 若是则被感染
                    if (strcmp((const char *)(pSec->Name), "lw0x60") == 0)
                    {
                        // 定位到最后一个区块
                        for (unsigned int i = 1; i < nSecNumber; i++)
                        {
                            pSec = (PIMAGE_SECTION_HEADER)((char *)pSec + 40);
                        }

                        // 原文件大小为感染后的文件大小减去病毒体大小后的值
                        dNewFileSize = dNewFileSize - (pSec->PointerToRawData + pSec->SizeOfRawData);
                        pNewFile = (DWORD *)(pSec->SizeOfRawData + pSec->PointerToRawData + (char *)pAdd);

                        char *temp1, *temp2;
                        temp1 = (char *)pAdd;
                        temp2 = (char *)pNewFile;

                        // 将原文件数据依次移到感染后文件的头部，以还原原文件
                        for (i = 0; i < dNewFileSize; i++)
                        {
                            *(temp1++) = *(temp2++);
                        }

                        ::FlushViewOfFile(pAdd, dNewFileSize);
                        ::UnmapViewOfFile(pAdd);
                        ::CloseHandle(hMap);

                        // 修改文件大小
                        ::SetFilePointer(hFile, dNewFileSize, NULL, FILE_BEGIN);
                        ::SetEndOfFile(hFile);

                        g_Count_File++;
                        printf("%s\t 被清除\n", wfd.cFileName);
                    }
                }
            }

            ::UnmapViewOfFile(pAdd);
            ::CloseHandle(hMap);
            ::CloseHandle(hFile);
        }
    }
    while (::FindNextFile(hFind, &wfd) != 0);

    ::SetCurrentDirectory(cDir);
}

void main()
{
    char cDrive[4] = "A:\\";
    char *p = cDrive;
    UINT uDriveType;

    for (; *p <= 'Z'; (*p)++)
    {
        uDriveType = ::GetDriveType(cDrive);
        if (uDriveType == DRIVE_REMOVABLE || uDriveType == DRIVE_FIXED)
        {
            list(cDrive);
        }
    }

    printf("共%d 个文件被感染\n", g_Count_File);
    system("pause");
}
```

> 清除工具只能清除区块名为 `lw0x60` 之类的 `Win32.Viking.ix` 病毒,具体的分析小菜菜只能弄到这里，见笑了

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

> 2007 年 03 月 16 日 下午 11:59:43

> 附件在 11 楼～～～～～～～～～

> 请教楼主，我调试了一下楼主的代码，我感觉 `pPe=(DWORD*)((DWORD)pAdd+(DWORD)pPe);` 是不是应该改为：`pPe = (DWORD*)((DWORD)pAdd + *pPe);` 阿？  
> 我用楼上的代码到不到 `'PE\\0\\0'` 头。  
> 基地址＋`e_lfanew`  
> 自已感觉代码风格不是很好

> 引用:  
> `pPe=(DWORD*)((DWORD)pAdd+(DWORD)pPe);` 是不是应该改为：`pPe = (DWORD*)((DWORD)pAdd + *pPe);` 阿？

> `pPe=(DWORD*)((DWORD*)((char*)pAdd+0x3c));`  
> 执行之后 `pPe` 就存放的是该 `PE` 文件的 `IMAGE_NT_HEADERS` 结构的 `RVA`  
> `(DWORD*)((DWORD)pAdd+(DWORD)pPe)` 后便可在去判断是否 `PE` 是标志  
> 若是你 `*pPe` 的话搞不好就内存访问异常了，而且 `*pPe` 必是错误用法

> 被感染后的文件都被附加到病毒体尾部去了，只需把尾部的数据全部取出来便可 恢复原文件

`4dd466d0e1e0cc123afbe504da2462ca`
