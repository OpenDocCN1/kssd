
# 杀杀毒,灌灌水

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-2.htm`](https://www.pediy.com/kssd/pediy09/pediy09-2.htm)

> 元旦休假，同学不幸中招，招什么“威金”肆虐，被拉去救急，一番下来，把过程写下来算是一次论坛灌水吧

> 找工作忙， 好久都没有来了，借此逛下

> 从同学机子上 down 一个感染文件下来，挂在另一台机子上分析，载入后，一大段 PUSH POP，不知道是被加壳了还是程序是 shellcode(应该是壳,有谁脱过的能告诉下过程),跟来半天都没有跟出来， 没有办法只好下 API 断点，

下面是程序被断下后堆栈的内容。

```
0012FE00 00404AD7 /CALL 到 CreateFileA
0012FE04 009A13F0 |FileName = "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe"
0012FE08 80000000 |Access = GENERIC_READ
0012FE0C 00000003 |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FE10 00000000 |pSecurity = NULL
0012FE14 00000003 |Mode = OPEN_EXISTING
0012FE18 00000080 |Attributes = NORMAL
0012FE1C 00000000 \hTemplateFile = NULL
0012FE20 0041299C WinAVI.0041299C
```

被感染的文件运行时先读取自己取得文件句柄 `hFile` = `00000064`

```
/CALL 到 ReadFileA
0012FE1C 00000064 |hFile = 00000064
0012FE20 009A142C |Buffer = 009A142C
0012FE24 0000DF4A |BytesToRead = DF4A (57162.)
0012FE28 0012FF98 |pBytesRead = 0012FF98
0012FE2C 00000000 \pOverlapped = NULL
```

把病毒代码读到缓存保存

```
0012FDFC 00408C09 /CALL 到 WinExec 来自 WinAVI.00408C04
0012FE00 009A00A0 |CmdLine = "net stop ""Kingsoft AntiVirus Service"""
0012FE04 00000000 \ShowState = SW_HIDE
```

//创建一个线程用来关闭程序和停止服务

```
/CALL 到 CreateFileA
0012FDDC 009A00A0 |FileName = "C:\WINDOWS\uninstall\rundl132.exe"
0012FDE0 C0000000 |Access = GENERIC_READ|GENERIC_WRITE
0012FDE4 00000000 |ShareMode = 0
0012FDE8 00000000 |pSecurity = NULL
0012FDEC 00000002 |Mode = CREATE_ALWAYS
0012FDF0 00000080 |Attributes = NORMAL
0012FDF4 00000000 \hTemplateFile = NULL
```

//创建病毒文件 得到句柄 `hFile` = `0x00000060`

```
0012FDE8 004085E1 /CALL 到 WriteFile 来自 WinAVI.004085DC
0012FDEC 00000060 |hFile = 00000060
0012FDF0 009A142C |Buffer = 009A142C
0012FDF4 0000DF4A |nBytesToWrite = DF4A (57162.)
0012FDF8 0012FE20 |pBytesWritten = 0012FE20
0012FDFC 00000000 \pOverlapped = NULL
```

//把缓存的代码写到病毒文件中
//同时把启动项 load 进注册表

```
0012FB14 004048F6 /CALL 到 CreateFileA 来自 WinAVI.004048F1
0012FB18 009A001C |FileName = "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe.exe"
0012FB1C C0000000 |Access = GENERIC_READ|GENERIC_WRITE
0012FB20 00000000 |ShareMode = 0
0012FB24 00000000 |pSecurity = NULL
0012FB28 00000002 |Mode = CREATE_ALWAYS
0012FB2C 00000080 |Attributes = NORMAL
0012FB30 00000000 \hTemplateFile = NULL
```

创建的临时文件用来存储正常文件

```
0012FB14 004048F6 /CALL 到 CreateFileA 来自 WinAVI.004048F1
0012FB18 009A0730 |FileName = "C:\WINDOWS\Logo1_.exe"
0012FB1C C0000000 |Access = GENERIC_READ|GENERIC_WRITE
0012FB20 00000000 |ShareMode = 0
0012FB24 00000000 |pSecurity = NULL
0012FB28 00000002 |Mode = CREATE_ALWAYS
0012FB2C 00000080 |Attributes = NORMAL
0012FB30 00000000 \hTemplateFile = NULL
```

系统目录下 `X:\WINDOWS` 创建病毒文件的一个副本 `Logo1_.exe`

```
0012FB24 0040805F /CALL 到 ReadFile 来自 WinAVI.0040805A
0012FB28 00000060 |hFile = 00000060
0012FB2C 00AA0020 |Buffer = 00AA0020
0012FB30 0000DF4A |BytesToRead = DF4A (57162.)
0012FB34 0012FE20 |pBytesRead = 0012FE20
0012FB38 00000000 \pOverlapped = NULL
```

读取病毒文件到缓存

生成正常文件 `WinAVI.exe.exe`

```
0012FB14 004048F6 /CALL 到 CreateFileA 来自 WinAVI.004048F1
0012FB18 009A07C8 |FileName = "C:\DOCUME~1\ADMINI~1.604\LOCALS~1\Temp\$$aB.bat"
0012FB1C C0000000 |Access = GENERIC_READ|GENERIC_WRITE
0012FB20 00000000 |ShareMode = 0
0012FB24 00000000 |pSecurity = NULL
0012FB28 00000002 |Mode = CREATE_ALWAYS
0012FB2C 00000080 |Attributes = NORMAL
0012FB30 00000000 \hTemplateFile = NULL
```

创建一个临时文件 `$$aB.bat` 把一些信息写下
内容如下：

```
:try1
Del "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe"
if exist "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe" goto try1
ren "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe.exe" "WinAVI.exe"
if exist "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe.exe" goto try2
"D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\WinAVI.exe"
:try2
del "C:\DOCUME~1\ADMINI~1.604\LOCALS~1\Temp\$$aB.bat"
```

```
0012FB10 004082F4 /CALL 到 CreateProcessA 来自 WinAVI.004082EF
0012FB14 00000000 |ModuleFileName = NULL
0012FB18 009A07C8 |CommandLine = "C:\DOCUME~1\ADMINI~1.604\LOCALS~1\Temp\$$aB.bat"
0012FB1C 00000000 |pProcessSecurity = NULL
0012FB20 00000000 |pThreadSecurity = NULL
0012FB24 FFFFFFFF |InheritHandles = TRUE
0012FB28 00000020 |CreationFlags = NORMAL_PRIORITY_CLASS
0012FB2C 00000000 |pEnvironment = NULL
0012FB30 009A088C |CurrentDir = "D:\lgyuanworkspace\WINDOWPROGRAM\virus\test\"
0012FB34 0012FB94 |pStartupInfo = 0012FB94
0012FB38 0012FBD8 \pProcessInfo = 0012FBD8
```

创建一个进程来执行 `$$aB.bat`

```
0012FAA0 0040E438 /CALL 到 CreateProcessA 来自 WinAVI.0040E433
0012FAA4 00000000 |ModuleFileName = NULL
0012FAA8 009A0730 |CommandLine = "C:\WINDOWS\Logo1_.exe"
0012FAAC 00000000 |pProcessSecurity = NULL
0012FAB0 00000000 |pThreadSecurity = NULL
0012FAB4 FFFFFFFF |InheritHandles = TRUE
0012FAB8 00000020 |CreationFlags = NORMAL_PRIORITY_CLASS
0012FABC 00000000 |pEnvironment = NULL
0012FAC0 00000000 |CurrentDir = NULL
0012FAC4 0012FAEC |pStartupInfo = 0012FAEC
0012FAC8 0012FADC \pProcessInfo = 0012FADC
```

创建一个另外一个进程来执行 `Logo1_.exe`

`Logo1_.exe` 就开始后台操作

以下是截取被感染文件的部分代码

```
00404A26 push eax
00404A27 call 004042B4 ; jmp 到 kernel32.FindFirstFileA
00404A2C cmp eax, -1
00404A2F je short 00404A65
00404A31 push eax
00404A32 call 004042AC ; jmp 到 kernel32.FindClose
00404A37 test byte ptr [ebp-14C], 10
00404A3E jnz short 00404A65
。。。
00404AC9 push eax
00404ACA mov eax, edi 
00404ACC call 00403508
00404AD1 push eax
00404AD2 call 0040425C ; jmp 到 kernel32.CreateFileA
00404AD7 pop edi ；返回到这里
00404AD8 pop esi
00404AD9 pop ebx
00404ADA retn
。。。
00404ADB nop
00404ADC push ebx
00404ADD push esi
00404ADE mov esi, edx
00404AE0 mov ebx, eax
00404AE2 mov eax, esi
00404AE4 call 00403508
00404AE9 push eax
00404AEA mov eax, ebx
00404AEC call 00403508
00404AF1 push eax
00404AF2 call 004043AC ; jmp 到 kernel32.MoveFileA
00404AF7 cmp eax, 1
00404AFA sbb eax, eax
00404AFC inc eax
00404AFD pop esi
00404AFE pop ebx
00404AFF retn
00404B00 push ebx
00404B01 mov ebx, eax
00404B03 mov eax, ebx
00404B05 call 00403508
00404B0A push eax
00404B0B call 00404304 ; jmp 到 kernel32.GetFileAttributesA
00404B10 cmp eax, -1
00404B13 je short 00404B19
00404B15 test al, 10
00404B17 jnz short 00404B1D
00404B19 xor eax, eax
00404B1B pop ebx
00404B1C retn
；完成网络初始话
00404B5C jmp dword ptr [42034C] ; WS2_32.inet_ntoa
00404B62 mov eax, eax
00404B64 jmp dword ptr [420348] ; WS2_32.gethostbyname
00404B6A mov eax, eax
00404B6C jmp dword ptr [420344] ; WS2_32.gethostname
00404B72 mov eax, eax
00404B74 jmp dword ptr [420340] ; WS2_32.WSAStartup
00404B7A mov eax, eax
00404B7C jmp dword ptr [42033C] ; WS2_32.WSACleanup
。。。
```

下面这段是用来关闭杀毒监控程序 `RAVMON。EXE` 的

```
00407278 8B45 F>mov eax, dword ptr [ebp-8]
0040727B E8 88C>call 00403508
00407280 50 push eax ；查找杀毒软件的窗口句柄
00407281 E8 26D>call 004044AC ; jmp 到 user32.FindWindowA
00407286 6A 00 push 0
00407288 6A 00 push 0
0040728A 6A 10 push 10
0040728C 50 push eax ；发送 WM_CLOSE 消息
0040728D E8 62D>call 004044F4 ; jmp 到 user32.SendMessageA
```

。。。



```asm
00408BD7 lea edx, dword ptr [ebp-10]
00408BDA mov eax, 00408C80 ; ASCII "酥匦?匦"
00408BDF call 00404CC0
00408BE4 mov eax, dword ptr [ebp-10]
00408BE7 call 00407560
00408BEC push 0
00408BEE lea edx, dword ptr [ebp-14]
00408BF1 mov eax, 00408C94
00408BF6 call 00404CC0
00408BFB mov eax, dword ptr [ebp-14]
00408BFE call 00403508
00408C03 push eax ；用来执行命令“net stop "some services" ”
00408C04 call 0040443C ; jmp 到 kernel32.WinExec
00408C09 lea eax, dword ptr [ebp-4]
00408C0C push eax
```

...

...

下面的应该都是病毒自己本身的代码，通过 `logo1_.exe` 来执行

```asm
00408FF2 lea eax, dword ptr [ebp-4]
00408FF5 push eax
00408FF6 push 0
00408FF8 push 0
00408FFA push 0040EE9C
00408FFF push 0
00409001 push 0
00409003 call 00404274 ; jmp 到 kernel32.CreateThread
00409008 mov byte ptr [41F1D5], 1
0040900F mov eax, dword ptr [41120C]
00409014 test byte ptr [eax+1C], 1
00409018 je short 00409041
0040901A push 7530
0040901F call 0040440C ; jmp 到 kernel32.Sleep
00409024 mov byte ptr [41F1D5], 0
0040902B lea eax, dword ptr [ebp-8]
0040902E push eax
0040902F push 0
00409031 push 0
00409033 push 0040FB40
00409038 push 0
0040903A push 0
0040903C call 00404274 ; jmp 到 kernel32.CreateThread
00409041 mov byte ptr [41F1D6], 1
00409048 mov eax, dword ptr [41120C]
0040904D test byte ptr [eax+1C], 2
00409051 je 004090FD
00409057 mov byte ptr [41F1D6], 0
0040905E lea eax, dword ptr [ebp-C]
00409061 push eax
00409062 push 0
00409064 push 0
00409066 push 0040F8F0
0040906B push 0
0040906D push 0
0040906F call 00404274 ; jmp 到 kernel32.CreateThread
00409074 jmp 004090FD
00409079 mov eax, dword ptr [edi]
0040907B cmp dword ptr [eax+4], 0
0040907F je short 004090BD
00409081 push 7D0
00409086 call 0040440C ; jmp 到 kernel32.Sleep
0040908B cmp byte ptr [41F1D7], 0
00409092 je 00409130
```

...

```asm
004090CC call 004044D4 ; jmp 到 user32.PeekMessageA
004090D1 test eax, eax
004090D3 je short 004090F6
004090D5 cmp dword ptr [ebp-13C], 10
004090DC je short 00409130
004090DE lea eax, dword ptr [ebp-140]
004090E4 push eax
004090E5 call 004044FC ; jmp 到 user32.TranslateMessage
004090EA lea eax, dword ptr [ebp-140]
004090F0 push eax
004090F1 call 004044A4 ; jmp 到 user32.DispatchMessageA
004090F6 push 64
004090F8 call 0040440C ; jmp 到 kernel32.Sleep
```

...

以上是对感染文件的一些分析，而病毒文件的一些运行过程，从上面的一些代码段中大致可以看出

有网络函数，估计会通过网络进行传播，同时有文件遍历的操作，遍历文件进行感染（被感染后应该有标志），在程序的字符中发现一个 `dll` 文件，运行后病毒会释放该文件，完成对系统进程的注射（`EXPLORER.EXE`），大致猜该是如此运行。06 年病毒真是猖獗，校园局域网内中毒一片，写毒的这些人，素质咋就这低呢。

修复代码如下，只是个示范

```cpp
bool CVirusFile::AntiVirus(const char * infile)
{
    HANDLE hFile = NULL;
    DWORD fileSize = 0;
    DWORD fileSizeHigh;
    char * buffer;
    IMAGE_DOS_HEADER doshd;

    __try
    {
        hFile = CreateFile(infile,
                           GENERIC_READ,
                           FILE_SHARE_READ,
                           NULL,
                           OPEN_EXISTING,
                           FILE_ATTRIBUTE_NORMAL,
                           NULL);
        fileSize = GetFileSize(hFile, &fileSizeHigh);

        DWORD rSize;
        ReadFile(hFile, &doshd,
                 sizeof(IMAGE_DOS_HEADER),
                 &rSize, NULL);
        buffer = (char *)malloc(fileSize-doshd.e_res2[4]+1);
        SetFilePointer(hFile,
                       doshd.e_res2[4],
                       NULL, FILE_BEGIN);
        ReadFile(hFile, buffer,
                 fileSize-doshd.e_res2[4],
                 &rSize, NULL);
        CloseHandle(hFile);
        DeleteFile(infile);

        hFile = CreateFile(infile,
                           GENERIC_READ|GENERIC_WRITE,
                           0, NULL, CREATE_ALWAYS,
                           FILE_ATTRIBUTE_NORMAL, NULL);
        WriteFile(hFile, buffer,
                  fileSize-doshd.e_res2[4],
                  &rSize, NULL);

        free(buffer);
        buffer = NULL;
        CloseHandle(hFile);
        hFile = NULL;
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
        if (INVALID_HANDLE_VALUE != hFile)
            CloseHandle(hFile);
        hFile = NULL;

        if (NULL != buffer)
            free(buffer);
        buffer = NULL;

        return false;
    }

    return true;
}
```

祝大家元旦快乐,新的一年有新的发展......
