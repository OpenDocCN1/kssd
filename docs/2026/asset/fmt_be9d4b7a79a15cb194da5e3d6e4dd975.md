
# 分析了一下ArmaFindprotected原理

小小分析了一下ArmaFindprotected原理

## 1. 脱壳

好像是 upack，载入出错，停在系统断点。

```
7C921230 >/$  CC            INT3
7C921231      C3            RETN
```

PEid 查看入口是 `00001018`，去 `400000+1018` 处 F2，F9 到达壳入口。

```
00401018    BE B0114000     MOV     ESI, 004011B0
0040101D    AD              LODS    DWORD PTR [ESI]
0040101E    50              PUSH    EAX                              ; Armadill.00402025
0040101F    FF76 34         PUSH    DWORD PTR [ESI+34]
00401022    EB 7C           JMP     SHORT 004010A0
```

本来多跟了几步，后来发现第一个 `push eax` 就是 OEP，`Ctrl+G eax`，F4 到 OEP。

然后用 LordPE dump，ImpRec 修复，另外修复数据目录和 `NumberOfDictionaries`，不用每次都有错误提示了。

```
->DOS 头

   e_magic:     0x5A4D

   e_cblp:      0x454B

   e_cp:        0x4E52

   e_crlc:      0x4C45

   e_cparhdr:   0x3233

   e_minalloc:  0x442E

   e_maxalloc:  0x4C4C

   e_ss:        0x0000

   e_sp:        0x4550

   e_csum:      0x0000

   e_ip:        0x014C

   e_cs:        0x0004

   e_lfarlc:    0xB0BE

   e_ovno:      0x4011

   e_res:       0xAD00FF5034767CEB

   e_oemid:     0x00E0

   e_oeminfo:   0x010F

   e_res2:      0x010B6F4C6461694C726272614179000020250000

   e_lfanew:    0x00000010

->文件头

   所需机器:              0x014C  (I386)

   区段数目:              0x0004

   时间日期戳:            0x4011B0BE  (GMT: Fri Jan 23 23:39:42 2004)

   符号表指针:            0xFF50AD00

   符号数:                0x7CEB3476

   可选文件头大小:        0x00E0

   特征:                  0x010F

                          (去除重定位表)

                          (文件可执行)

                          (去除行号)

                          (去除本地标志)

                          (需32位机器)

->可选文件头

   幻术:                         0x010B  (HDR32_MAGIC)

   连接器主版本:                 0x4C

   连接器副版本:                 0x6F  -> 76.111

   代码段大小:                   0x694C6461

   已初始化数据大小:             0x72617262

   未初始化数据大小:             0x00004179

   程序执行入口点地址:           0x00002025

   代码段基址:                   0x00000010

   数据段基址:                   0x00005000

   映像基址:                     0x00400000

   内存区段对齐单位:             0x00001000

   文件区段对齐单位:             0x00001000

   所需操作系统主版本号:         0x0004

   所需操作系统副版本号:         0x0000  -> 4.00

   自定义主版本号:               0x0000

   自定义副版本号:               0x0039  -> 0.57

   所需子系统主版本号:           0x0004

   所需子系统副版本号:           0x0000  -> 4.00

   Win32版本号值:                0x00000000

   映像大小:                     0x00017000

   文件头大小:                   0x00001000

   CRC校验和:                    0x00000000

   子系统:                       0x0002  (WINDOWS_GUI)

   DLL特征值:                    0x0000

   栈初始保留区大小:             0x00100000

   栈初始提交区大小:             0x00001000

   堆初始保留区大小:             0x00100000

   堆初始提交区大小:             0x00001000

   加载标志:                     0x00000000

   数据目录结构数组项数:         0x00000010

   数据目录 (16)                 RVA        大小

   -------------                 ---------- ----------

   导出表                        0x00000000 0x00000000

   导入表                        0x00016000 0x00000050  (".mackt")

   资源                          0x0000A000 0x000009A4  (".rsrc")

   例外表                        0x00000000 0x00000000

   安全证书                      0x00000000 0x00000000

   重定位表                      0x00000000 0x00000000

   调试                          0x00000000 0x00000000

   版权号                        0x00000000 0x00000000

   全局指针                      0x00000000 0x00000000

   TLS 表                        0x00000000 0x00000000

   加载构造表                    0x00000000 0x00000000

   绑定导入表                    0x00000000 0x00000000

   IAT                           0x00000000 0x00000000

   延迟导入表                    0x00000000 0x00000000

   COM                           0x00000000 0x00000000

   保留                          0x00000000 0x00000000
```

## 2. 分析

从 `GetOpenFileNameA` 入手，很快得到：

### 主线剧情

```
mz                        IMAGE_DOS_HEADER        <>

pe                        IMAGE_NT_HEADERS        <>

call        CreateFileA, offset filename, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, NORMAL|SEQUENTIAL_SCAN, NULL

call        ReadFile, filehandle, offset mz, 40h, offset bytesread, NULL

call        SetFilePointer, filehandle, mz.e_lfanew, NULL, FILE_BEGIN

call        ReadFile, filehandle, offset pe, 0F8h, offset bytesread, NULL

00401467    66:81BD EAF8FFF>CMP     WORD PTR [EBP-716], 5253//linker version

00401470    74 1B           JE      SHORT 0040148D

00401472    66:81BD F8FBFFF>CMP     WORD PTR [EBP-408], 5253

0040147B    74 10           JE      SHORT 0040148D

0040147D    68 20634000     PUSH    00406320                         ; ASCII "?- Not Armadillo protected",CR,LF

00401482    E8 79FBFFFF     CALL    00401000

00401487    59              POP     ECX

00401488    E9 CD040000     JMP     0040195A

call        CreateProcessA, offset filename, NULL, NULL, NULL, FALSE, DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS, NULL, NULL, offset sbox, offset pbox

call        WaitForDebugEvent, offset devent, 1000
```

### 分支剧情

#### 分支 1

```
0040188A    83BD 20FFFFFF 0>CMP     DWORD PTR [EBP-E0], 6                 ; LOAD_DLL_DEBUG_EVENT

00401891    75 2B           JNZ     SHORT 004018BE

00401893    8B45 80         MOV     EAX, [EBP-80]                         ; ker

00401896    3B85 30FFFFFF   CMP     EAX, [EBP-D0]                         ; ntd, wait kernel32

0040189C    75 20           JNZ     SHORT 004018BE

0012F3A0   004018B0  /CALL to WriteProcessMemory from dumped_.004018AE

0012F3A4   00000108  |hProcess = 00000108 (window)

0012F3A8   7C812E03  |Address = 7C812E03

0012F3AC   0012FA88  |Buffer = 0012FA88

0012F3B0   0000000E  |BytesToWrite = E (14.)

0012F3B4   00000000  \pBytesWritten = NULL

004018B0    8D45 FC         LEA     EAX, [EBP-4]

004018B3    50              PUSH    EAX

004018B4    FF75 90         PUSH    DWORD PTR [EBP-70]                    ; OpenMutexA

004018B7    E8 31F8FFFF     CALL    <save_set_CC>
```

#### 分支 2

```
004014EA    3985 20FFFFFF   CMP     [EBP-E0], EAX                         ; EXCEPTION_DEBUG_EVENT

004014F0    0F85 94030000   JNZ     0040188A

004014F6    81BD 2CFFFFFF 0>CMP     DWORD PTR [EBP-D4], 80000003          ; EXCEPTION_BREAKPOINT

00401500    0F85 B8030000   JNZ     004018BE

00401506    8B8D 38FFFFFF   MOV     ECX, [EBP-C8]                         ; context.Eip

0040150C    394D 90         CMP     [EBP-70], ECX                         ; is OpenMutexA?

0040150F    0F85 FD000000   JNZ     00401612                              ; no

00401515    53              PUSH    EBX
```

### 主线剧情

```
0012F3A0   00401526  /CALL to WriteProcessMemory from dumped_.00401524//fix OpenMutexA

0012F3A4   00000108  |hProcess = 00000108 (window)

0012F3A8   7C80EC1B  |Address = 7C80EC1B

0012F3AC   0012FAF0  |Buffer = 0012FAF0

0012F3B0   00000001  |BytesToWrite = 1

0012F3B4   00000000  \pBytesWritten = NULL

0012F3AC   00401543  /CALL to GetThreadContext from dumped_.0040153D

0012F3B0   00000114  |hThread = 00000114

0012F3B4   0012F748  \pContext = 0012F748

00401543    FF8D 0CFDFFFF   DEC     DWORD PTR [EBP-2F4]                   ; eip--

0012F3AC   0040155C  /CALL to SetThreadContext from dumped_.00401556//restore EIP

0012F3B0   00000114  |hThread = 00000114

0012F3B4   0012F748  \pContext = 0012F748

0012F3A0   00401571  /CALL to ReadProcessMemory from dumped_.0040156F//read Contex.Eip

0012F3A4   00000108  |hProcess = 00000108 (window)

0012F3A8   0012F56C  |pBaseAddress = 12F56C

0012F3AC   0012FAD8  |Buffer = 0012FAD8

0012F3B0   00000004  |BytesToRead = 4

0012F3B4   00000000  \pBytesRead = NULL
```

```
/*
in debuggy

00821C99    8D95 D4FEFFFF   LEA     EDX, [EBP-12C]
00821C9F    52              PUSH    EDX
00821CA0    6A 00           PUSH    0
00821CA2    68 01001F00     PUSH    1F0001
```
