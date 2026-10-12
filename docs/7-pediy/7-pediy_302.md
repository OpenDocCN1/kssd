
# 动手给音速启动加点动力

> 原文：[`www.pediy.com/kssd/pediy12/131339.html`](https://www.pediy.com/kssd/pediy12/131339.html)

http://www.52pojie.cn/thread-84483-1-1.html

## 成品

动手给音速启动加点动力

音速启动这玩艺我也用了好几年了、昨天上官网发现作者也很久不更新了- -

习惯了还是舍不得放弃，可是每当用它打开很多文件夹的时候，任务管理器里的 `explorer` 进程多到让人感觉蛋疼，今天我就动手来给他加点动力，让它打开文件夹的时候不再这么蛋疼。

## 原理

原理是 Hook `VB` 的 `rtcShell`，执行我们指定的处理流程。

`VB` 里的 `Shell` 函数执行的这条命令，调用参数自己猜（=w=微软自然没有那么好心）、看了几个调用以后猜测应该是 2 个参数，第一个参数指向一个 `pParam` 结构体，第二个总是 1(类似于 `SW_SHOW` 么？)

### 看一看调用

```
0012F658   00605B79  返回到 VStart_-.00605B79 来自 MSVBVM60.rtcShell
0012F65C   0012F74C
0012F660   00000001

rtcShell 入口两个变量、

0012F664   0012F7E0
0012F668   0012F864
0012F66C   00000001
0012F670   00630069  VStart_-.00630069
0012F674   0049006B  VStart_-.0049006B
0012F678   0064006E  VStart_-.0064006E
```

### 我猜测的结构体信息

```c
typedef struct
{
  DWORD dwType;
  DWORD pSelf;
  PWCHAR pCommand;
}rtcParam;
```

### 函数类型

```c
typedef HMODULE (WINAPI __pfnrtcShell)(rtcParam* pParam,DWORD unknow);
```

`Hook` 以后用 `OutputDebugString` 格式化一些参数变量、然后多调用几次、寻找规律。

**代码:**

```c
char temp[MAX_PATH];
sprintf(temp,"call from: 0x%.8X\nType:0x%X",RetAddr,pParam->pSelf);
OutputDebugString(temp);
OutputDebugStringW(pParam->pCommand);
```

得到使用 `Shell` 调用的基本上分为四个类型、一个是他启动的时候对于自己部分插件的调用，被我猜测为反身指针的那个变量是个很奇怪的值，第二种是对于文件夹快捷方式的调用，这时候那个反身指针是 0，当然还有其它的系统功能调用的时候这个参数也是 0，所以这里要进行特殊处理，仅过滤出我们想要的，第三种是浏览程序目录的调用，这个时候反身指针指向 `pParam`，最后一种实际上属于第三种调用，需要特殊处理。

### 下面是根据所找出的规律写的 rtcShell 的 Hook 处理函数

**引用:**

```c
HMODULE WINAPI My_rtcShell(DWORD RetAddr,
                           __pfnrtcShell pfnrtcShell,
                           rtcParam* pParam,
                           DWORD unknow
                           )
{
    if (pParam->pSelf==(DWORD)pParam)
    /*
    [4964] call from: 0x00605B79
    [4964] Type:0x12F74C
    [4964] Explorer /select,"D:\Program Files\Microsoft Math 3.0\MATHAPP.EXE" SelectByExplorer
    */
    {
        LPWSTR pCmd=wcschr(pParam->pCommand,'"');
        if (pCmd)
        {
            *(pCmd+wcslen(pCmd)-1)=0;
            SelectByExplorer(pCmd+1);
            return 0;
        }
    }
    else if (pParam->pSelf==0)
    /*
    [4964] call from: 0x0051C1EB
    [4964] Type:0x0
    [4964] Explorer.exe "E:\Projects\Developing"无参数调用文件夹
    ----------------------------------------------------------------------
    [4964] call from: 0x00519A80
    [4964] Type:0x0
    [4964] Explorer.exe /n,::{450D8FBA-AD25-11D0-98A8-0800361B1103}
    ----------------------------------------------------------------------
    [4964] call from: 0x00519A80
    [4964] Type:0x0
    [4964] rundll32.exe shell32.dll,Control_RunDLL
    这里要考虑另外调用的情况
    */
    {
        BOOL bExecute=FALSE;
        LPWSTR pCmd=wcschr(pParam->pCommand,'"');
        if (pCmd)
        {
            *(pCmd+wcslen(pCmd)-1)=0;
            wcscat(pCmd,L"\\*.*");
            WIN32_FIND_DATAW FindFileData;
            HANDLE hFind = FindFirstFileW(pCmd+1, &FindFileData);
            while ( hFind !=INVALID_HANDLE_VALUE&&FindNextFileW(hFind, &FindFileData))
            {
                if ( FindFileData.cFileName[0] != '.' )
                {
                    *(wcsrchr(pCmd,'\\')+1)=0;//- -这样子搞很成问题、
                    wcscat(pCmd,FindFileData.cFileName);
                    SelectByExplorer(pCmd+1);
                    bExecute=TRUE;
                    break;
                }
            }
            FindClose(hFind);
            if (!bExecute)
            {
                PWCHAR pOr=wcsrchr(pCmd,'\\');
                *pOr='"';
                *(pOr+1)=0;//还原命令行
                pfnrtcShell(pParam,unknow);
            }
            return 0;
        }
    }

//     char temp[MAX_PATH];
//     sprintf(temp,"call from: 0x%.8X\nType:0x%X",RetAddr,pParam->pSelf);
//     OutputDebugString(temp);
//     OutputDebugStringW(pParam->pCommand);

    return pfnrtcShell(pParam,unknow);//这里调用非 hook 的
}
```

## 替换的核心函数

**代码:**

```c
void SelectByExplorer(LPWSTR lpPath)
{
  LPSHELLFOLDER IpsfDeskTop;
  LPITEMIDLIST pidl;
  SHGetDesktopFolder(&IpsfDeskTop);
  IpsfDeskTop ->ParseDisplayName(NULL, NULL, lpPath, NULL, &pidl, NULL);
  IpsfDeskTop->Release();
//  if (FAILED(hres)) return FALSE;
  SHOpenFolderAndSelectItems(pidl,0,NULL,NULL);
}
```

这个函数其实还是蛮有用的，要是能直接打开文件夹而不需要选中文件那就更好了。我对文件夹调用快捷方式的处理其实很挫，直接去那个文件夹底下随便找一个文件，然后粘在地址后面，再调用 `SelectByExplorer` 函数，这样当文件夹下没有文件的时候就会失败……但是我想不到怎么用更少的办法做到比这好的效果了。目前个人用了这个增强版以后感觉不错、意外崩溃一次，原因不明。大概真的是意外吧……

以上，这就是本篇文章的所有核心内容，感谢各位观赏。其实就是抛砖引玉的作用、相信各位能做出比我更好的功能拓展:)

## 最后直接修改壳的外部、在程序启动时装载 VstartPlug.dll

```asm
call 00401000

$ "VStartPlug.dll"

call dword ptr[006792DC]

jmp 411b60
```

完整源码请参考附件

![](img/fe314c9a4dce43b3e591fb7ac24de020.jpg)

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `VStartPlug.rar`

 |
