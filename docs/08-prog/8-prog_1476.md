
# OllyDBG 分析报告系列(7)---文件加载调试原理

> 原文：[`www.pediy.com/kssd/pediy10/65755.html`](https://www.pediy.com/kssd/pediy10/65755.html)

这是本系列文档的第七篇，因本人水平有限，不足和疏漏之处请各位批评指正，谢谢。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

文件加载调试原理主要是使用 `CreateProcess` 函数，以调试方式加载文件进程，在加载的过程中对被加载程序的有效性进行检验，然后通过查询 `PE` 头结构，得到其 `OEP`，并通过单步方式在 `OEP` 处下 `Int3` 断点，这样 `OD` 打开的程序就停在 `OEP`，进而可以让调试人员在入口处开始调试程序。

在文件加载时，调用了 `OpenExeFile` 这个函数，得到打开文件的相关信息，并判断其是否为有效的 `PE` 文件；然后再根据得到的信息通过上述方法停在入口。由于比较仓促，只分析了前一部分函数的代码，其主要流程如下：

1.  判断打开的是否为快捷方式，若为快捷方式则通过相关函数找到对应的可执行程序；

2.  调用 `SearchPath` 查找是否有该文件，然后调用文件操作相关的库函数，打开文件，进行相关的判断；

3.  关于其是否为有效的 `PE` 文件的判断有几个方面：

    a)  是否有 `MZ` 头；

    b)  是否有 `PE` 头；

    c)  `NT` 头中的 `Machine` 字段是否为 `14Ch`；

    d)  `NT` 头中的 `Characteristics` 是否为可执行；

    e)  `NT` 头中的 `SizeOfOptionalHeader` 是否为 `0E0h`。

    等等，最后还根据判断的标识做了相应的处理；

4.  设置加载文件为 `DLL` 标识；

5.  检查是否还有活动的调试线程，将其暂停，并将那些线程有关的调试信息全部清除，检查是否清除成功，若失败则返回 `-1` 并退出；

6.  判断打开的是否为 `Dll` 文件，如果不是则跳过检查文件 `LoadDll.exe` 是否存在；

7.  调用函数 `0040F40C` 判断与 `OD` 同一目录下是否存在 `LoadDll.exe`，若不存在，则从 `OD` 可执行程序中解压资源 `RES_LOADDLL`，生成 `LoadDll.exe` 程序，若生成失败则报错，返回 `-1` 并退出；

8.  通过全文件路径名，得到其所在的文件夹，检查该调试程序的相关配置文件 `common.arg`，若存在则加载处理；

9.  处理 `Ini` 文件

10. 检查加载的是 `DLL` 还是 `EXE`，根据加载的不同生成不同的命令行参数：

    a)  如果加载的是 `DLL`，组合出命令行：`"\\Path\\LoadDll.exe" \\Path\\XXX.dll` 传递给下面的 `CreateProcess` 函数；

    b)  如果加载的是 `EXE`，将 `EXE` 文件名传给下面的 `CreateProcess` 函数；

    c)  ……还有 `EXE` 情况未知；

11. 使用 `CreateProcessA` 创建调试进程，其中，`CreationFlags = DEBUG_ONLY_THIS_PROCESS | CREATE_NEW_CONSOLE | NORMAL_PRIORITY_CLASS | CREATE_DEFAULT_ERROR_MODE`，检查是否打开成功，若失败则返回 `-1` 并退出；

12. 将 `CreateProcess` 中获得的 `PROCESS_INFORMATION` 结构体的值赋值给全局变量；

13. 设置 `OD` 的标题等界面的显示内容；

14. 读写配置文件，主要是操作一些最近打开的文件之类的配置值，然后退出。

这里为 `OpenExeFile` 函数的汇编代码，除了上述的流程，下面会对一些关键的地方做注释：

这部分检查打开的是否为快捷方式，如果不是快捷方式就跳过查找对应的可执行文件的路径部分：

```
0047731C >/$  55           push    ebp

0047731D  |.  8BEC         mov    ebp, esp

0047731F  |.  81C4 04F0FFF>add      esp, -0FFC

00477325  |.  50           push    eax

00477326  |.  81C4 D4F2FFF>add      esp, -0D2C

0047732C  |.  53           push    ebx

0047732D  |.  56           push    esi

0047732E  |.  57           push    edi

0047732F  |.  8B5D 08      mov     ebx, dword ptr [ebp+8]

00477332  |.  BE EC164C00  mov     esi, 004C16EC             ;ASCII "Table of DLL handles"

00477337  |.  33C0         xor     eax, eax

00477339  |.  8945 F8     mov     dword ptr [ebp-8], eax

0047733C  |.  8D95 D8FAFFF>lea      edx, dword ptr [ebp-528]

00477342  |.  52           push    edx

00477343  |.  6A 00        push    0

00477345  |.  6A 00        push    0

00477347  |.  6A 00        push    0

00477349  |.  53           push    ebx

0047734A  |.  E8 6DDE0200  call    004A51BC          ;__fnsplit

0047734F  |.  83C4 14      add     esp, 14

00477352  |.  8D8E 0107000>lea      ecx, dword ptr [esi+701]

00477358  |.  51           push    ecx                       ; /Arg2

00477359  |.  8D85 D8FAFFF>lea      eax, dword ptr [ebp-528]           ; |

0047735F  |.  50           push    eax                       ; |Arg1

00477360  |.  E8 57C50200  call    004A38BC                  ; \\ _stricmp

00477365  |.  83C4 08      add     esp, 8

00477368  |.  85C0         test    eax, eax  ;判断是否为快捷方式

0047736A  |.  0F85 1501000>jnz      00477485  ;不是则跳过下面的查找部分
```

该部分通过加载 `COM` 库中的函数来查找快捷方式对应的可执行文件路径：

```
00477370  |.  6A 00        push    0             ;pvReserved

00477372  |.  E8 A3820300  call    <jmp.&OLE32.CoInitialize>

00477377  |.  C685 D8FCFFF>mov      byte ptr [ebp-328], 0

0047737E  |.  8D55 E8      lea     edx, dword ptr [ebp-18]

00477381  |.  52           push    edx           ;ppv

00477382  |.  68 20A84C00  push    004CA820         ; riid

00477387  |.  6A 01        push    1             ; dwClsContext

00477389  |.  6A 00        push    0             ; pUnkOuter

0047738B  |.  68 30A84C00  push    004CA830         ; rclsid

00477390  |.  E8 7F820300  call    <jmp.&OLE32.CoCreateInstance>

00477395  |.  85C0         test    eax, eax

00477397  |.  0F8C BC00000>jl       00477459

0047739D  |.  8D4D E4      lea     ecx, dword ptr [ebp-1C]

004773A0  |.  51           push    ecx

004773A1  |.  68 40A84C00  push    004CA840

004773A6  |.  8B45 E8      mov     eax, dword ptr [ebp-18]

004773A9  |.  50           push    eax

004773AA  |.  8B10         mov     edx, dword ptr [eax]

004773AC  |.  FF12         call    dword ptr [edx]

004773AE  |.  85C0         test    eax, eax

004773B0  |.  0F8C 9A00000>jl       00477450

004773B6  |.  68 04010000  push    104                 ; /WideBufSize = 104 (260.)

004773BB  |.  8D8D 48E5FFF>lea      ecx, dword ptr [ebp-1AB8]      ; |

004773C1  |.  51           push    ecx                 ; |WideCharBuf

004773C2  |.  6A FF        push    -1                  ; |StringSize = FFFFFFFF (-1.)

004773C4  |.  53           push    ebx                 ; |StringToMap

004773C5  |.  6A 01        push    1                 ; |Options = MB_PRECOMPOSED

004773C7  |.  6A 00        push    0                   ; |CodePage = CP_ACP

004773C9  |.  E8 887D0300  call    <jmp.&KERNEL32.MultiByteToWid>; \\MultiByteToWideChar

004773CE  |.  6A 00        push    0

004773D0  |.  8D85 48E5FFF>lea      eax, dword ptr [ebp-1AB8]

004773D6  |.  50           push    eax

004773D7  |.  8B55 E4      mov     edx, dword ptr [ebp-1C]

004773DA  |.  52           push    edx

004773DB  |.  8B0A         mov     ecx, dword ptr [edx]

004773DD  |.  FF51 14      call    dword ptr [ecx+14]

004773E0  |.  85C0         test    eax, eax

004773E2  |.  7C 63        jl      short 00477447

004773E4  |.  6A 00        push    0

004773E6  |.  A1 7C3B4D00  mov     eax, dword ptr [4D3B7C]

004773EB  |.  50           push    eax

004773EC  |.  8B55 E8      mov     edx, dword ptr [ebp-18]

004773EF  |.  52           push    edx

004773F0  |.  8B0A         mov     ecx, dword ptr [edx]

004773F2  |.  FF51 4C      call    dword ptr [ecx+4C]

004773F5  |.  85C0         test    eax, eax

004773F7  |.  7C 4E        jl      short 00477447

004773F9  |.  8D85 D4E2FFF>lea      eax, dword ptr [ebp-1D2C]

004773FF  |.  6A 00        push    0

00477401  |.  50           push    eax

00477402  |.  8D95 D8FCFFF>lea      edx, dword ptr [ebp-328]

00477408  |.  68 04010000  push    104

0047740D  |.  52           push    edx

0047740E  |.  8B4D E8      mov     ecx, dword ptr [ebp-18]

00477411  |.  51           push    ecx

00477412  |.  8B01         mov     eax, dword ptr [ecx]

00477414  |.  FF50 0C      call    dword ptr [eax+C]

00477417  |.  85C0         test    eax, eax

00477419  |.  7D 07        jge     short 00477422

0047741B  |.  C685 D8FCFFF>mov      byte ptr [ebp-328], 0

00477422  |>  68 00100000  push    1000

00477427  |.  68 885D4D00  push    004D5D88

0047742C  |.  8B55 E8      mov     edx, dword ptr [ebp-18]

0047742F  |.  52           push    edx

00477430  |.  8B0A         mov     ecx, dword ptr [edx]

00477432  |.  FF51 28      call    dword ptr [ecx+28]

00477435  |.  85C0         test    eax, eax

00477437  |.  7C 07        jl      short 00477440

00477439  |.  33C0         xor     eax, eax

0047743B  |.  8945 0C      mov     dword ptr [ebp+C], eax

0047743E  |.  EB 07        jmp     short 00477447

00477440  |>  C605 885D4D0>mov      byte ptr [4D5D88], 0

00477447  |>  8B55 E4      mov     edx, dword ptr [ebp-1C]

0047744A  |.  52           push    edx

0047744B  |.  8B0A         mov     ecx, dword ptr [edx]

0047744D  |.  FF51 08      call    dword ptr [ecx+8]

00477450  |>  8B45 E8      mov     eax, dword ptr [ebp-18]

00477453  |.  50           push    eax

00477454  |.  8B10         mov     edx, dword ptr [eax]

00477456  |.  FF52 08      call    dword ptr [edx+8]
```



00477459  |>  E8 C2810300  call    <jmp.&OLE32.CoUninitialize>

检查是否完全释放 com 库，若释放失败则报错、返回-1 并退出：

```
0047745E  |.  80BD D8FCFFF>cmp     byte ptr [ebp-328], 0

00477465  |.  75 18        jnz     short 0047747F

00477467  |.  53           push    ebx                   ; /Arg2

00477468  |.  8D8E 0607000>lea     ecx, dword ptr [esi+706]     ; |

0047746E  |.  51           push    ecx                   ; |Arg1

0047746F  |.  E8 A8CBFDFF  call    _Error                  ; \_Error

00477474  |.  83C4 08      add     esp, 8

00477477  |.  83C8 FF      or      eax, FFFFFFFF    ;返回-1

0047747A  |.  E9 08060000  jmp     00477A87      ;跳转到结束处
```

得到路径后用 `SearchPathA` 查找路径是否正确，路径不正确则报错、返回-1 并退出：

```
0047747F  |>  8D9D D8FCFFF>lea     ebx, dword ptr [ebp-328]

00477485  |>  8D45 F0      lea     eax, dword ptr [ebp-10]

00477488  |.  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]

0047748E  |.  50           push    eax                   ; /ppFilePart

0047748F  |.  52           push    edx                   ; |ReturnBuffer

00477490  |.  8D8E 1704000>lea     ecx, dword ptr [esi+417]           ; |

00477496  |.  68 04010000  push    104                   ; |BufSize = 104 (260.)

0047749B  |.  51           push    ecx                   ; |Extension

0047749C  |.  53           push    ebx                   ; |FileName

0047749D  |.  6A 00        push    0                     ; |Path = NULL

0047749F  |.  E8 DC7C0300  call    <jmp.&KERNEL32.SearchPathA>      ; \SearchPathA

004774A4  |.  85C0         test    eax, eax

004774A6  |.  75 18        jnz     short 004774C0

004774A8  |.  53           push    ebx                   ; /Arg2

004774A9  |.  8D86 3F07000>lea     eax, dword ptr [esi+73F]           ; |

004774AF  |.  50           push    eax                   ; |Arg1

004774B0  |.  E8 67CBFDFF  call    _Error                  ; \_Error

004774B5  |.  83C4 08      add     esp, 8

004774B8  |.  83C8 FF      or      eax, FFFFFFFF  ;返回-1

004774BB  |.  E9 C7050000  jmp     00477A87    ;t 跳转到结束处
```

调用 C 库函数打开可执行文件，并检查打开是否成功，根据检查结果做相应处理：

```
004774C0  |>  8D96 5A07000>lea     edx, dword ptr [esi+75A]

004774C6  |.  52           push    edx                   ; /Arg2

004774C7  |.  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]           ; |

004774CD  |.  51           push    ecx                   ; |Arg1

004774CE  |.  E8 A1DE0200  call    004A5374                 ; \ _fopen

004774D3  |.  83C4 08      add     esp, 8

004774D6  |.  8BF8         mov     edi, eax

004774D8  |.  85FF         test    edi, edi      ;检查是否打开成功

004774DA  |.  75 07        jnz     short 004774E3  ;打开成功则使 ebx = 0

004774DC  |.  BB 01000000  mov     ebx, 1      ;打开失败则使 ebx = 1

004774E1  |.  EB 02        jmp     short 004774E5

004774E3  |>  33DB         xor     ebx, ebx
```

根据上面的打开情况做处理，如果打开失败则跳过读取 `IMAGE_DOS_HEADER` 部分代码：

```
004774E5  |>  85DB         test    ebx, ebx    ;检查打开文件标识

004774E7  |.  75 1E        jnz     short 00477507  ;如果失败则跳过读取部分代码
```

检查读取 `IMAGE_DOS_HEADER` 是否成功，如果失败则使 `ebx` = 1：

```
004774E9  |.  57           push    edi                   ; /Arg4

004774EA  |.  6A 40        push    40                    ; |Arg3 = 00000040

004774EC  |.  6A 01        push    1                     ; |Arg2 = 00000001

004774EE  |.  8D85 58FAFFF>lea     eax, dword ptr [ebp-5A8]          ; |文件头指针

004774F4  |.  50           push    eax                   ; |Arg1

004774F5  |.  E8 D6E00200  call    004A55D0                 ; \ _fread

004774FA  |.  83C4 10      add     esp, 10

004774FD  |.  83F8 40      cmp     eax, 40    ;检查是否读取了 40h 的 IMAGE_DOS_HEADER

00477500  |.  74 05        je      short 00477507

00477502  |.  BB 01000000  mov     ebx, 1    ;读取失败 ebx = 1
```

将文件头指针放入寄存器，继续检查读取标志 `ebx`，读取失败则跳过检查 DOS 头的正确性：

```
00477507  |>  8D85 58FAFFF>lea     eax, dword ptr [ebp-5A8]

0047750D  |.  85DB         test    ebx, ebx  ;检查读取标志 ebx

0047750F  |.  75 0C        jnz     short 0047751D  ;读取失败则跳过检查 DOS 头的正确性
```

检查 `IMAGE_DOS_HEADER` 的正确性，并做相应的标识：

```
00477511  |.  66:8138 4D5A cmp     word ptr [eax], 5A4M  ;检查 IMAGE_DOS_HEADER 的正确性

00477516  |.  74 05        je      short 0047751D

00477518  |.  BB 02000000  mov     ebx, 2    ;若 IMAGE_DOS_HEADER 不正确，设置 ebx = 2
```

检查读取标识 `ebx`，若读取失败则跳过文件移位：

```
0047751D  |>  85DB         test    ebx, ebx

0047751F  |.  75 18        jnz     short 00477539
```

偏移文件指针到 DOS 头的 3C 处，偏移完后一样做检查，并做相应标识：

```
00477521  |.  6A 00        push    0                     ; /Arg3 = 00000000

00477523  |.  8B40 3C      mov     eax, dword ptr [eax+3C]          ; |

00477526  |.  50           push    eax                   ; |Arg2

00477527  |.  57           push    edi                   ; |Arg1

00477528  |.  E8 33E10200  call    004A5660               ; \_fseek

0047752D  |.  83C4 0C      add     esp, 0C

00477530  |.  85C0         test    eax, eax

00477532  |.  74 05        je      short 00477539

00477534  |.  BB 01000000  mov     ebx, 1    ;偏移失败 ebx = 1
```

对标识做相应检查，标识非 0 则跳过对文件的处理：

```
00477539  |>  85DB         test    ebx, ebx

0047753B  |.  75 1E        jnz     short 0047755B
```

读取 `IMAGE_NT_HEADERS` 的 `Signature` 和 `IMAGE_FILE_HEADER`，并检查是否成功，做相应标识：

```
0047753D  |.  57           push    edi                   ; /Arg4

0047753E  |.  6A 18        push    18                    ; |Arg3 = 00000018

00477540  |.  6A 01        push    1                     ; |Arg2 = 00000001

00477542  |.  8D85 58FAFFF>lea     eax, dword ptr [ebp-5A8]          ; |

00477548  |.  50           push    eax                   ; |Arg1

00477549  |.  E8 82E00200  call    004A55D0               ; \_fread

0047754E  |.  83C4 10      add     esp, 10

00477551  |.  83F8 18      cmp     eax, 18

00477554  |.  74 05        je      short 0047755B

00477556  |.  BB 01000000  mov     ebx, 1      ;失败则 ebx = 1
```

对标识做相应检查，标识非 0 则跳过对文件的处理：

```
0047755B  |>  85DB         test    ebx, ebx

0047755D  |.  75 11        jnz     short 00477570
```

检查 NT 头的正确性，并做相应的标识：

```
0047755F  |.  81BD 58FAFFF>cmp     dword ptr [ebp-5A8], 4550 ;NT 头的 Signature 是否为 4550h

00477569  |.  74 05        je      short 00477570

0047756B  |.  BB 02000000  mov     ebx, 2      ;不是则 ebx = 2

00477570  |>  8D85 5CFAFFF>lea     eax, dword ptr [ebp-5A4]
```

对标识做相应检查，标识非 0 则跳过对文件的处理：

```
00477576  |.  85DB         test    ebx, ebx

00477578  |.  8945 EC      mov     dword ptr [ebp-14], eax

0047757B  |.  75 1D        jnz     short 0047759A
```

检查 NT 头中的几个参数是否正确，并做相应的标识：

```
0047757D  |.  8B55 EC      mov     edx, dword ptr [ebp-14]

00477580  |.  66:813A 4C01 cmp     word ptr [edx], 14C  ;判断是否为 14Ch - Intel 365 - CPU

00477585  |.  74 05        je      short 0047758C

00477587  |.  BB 02000000  mov     ebx, 2

0047758C  |>  8B45 EC      mov     eax, dword ptr [ebp-14]

0047758F  |.  F640 12 02   test    byte ptr [eax+12], 2  ;NtHeader.Characteristics 是否可执行

00477593  |.  75 05        jnz     short 0047759A

00477595  |.  BB 02000000  mov     ebx, 2

0047759A  |>  33C0         xor     eax, eax
```

对标识做相应检查，标识非 0 则跳过对文件的处理：

```
0047759C  |.  85DB         test    ebx, ebx

0047759E  |.  8945 FC      mov     dword ptr [ebp-4], eax

004775A1  |.  75 54        jnz     short 004775F7
```

检查 NT 头中 `NtHeader.SizeOfOptionalHeader` 是否等于 `E0h`，即检查有无特殊的数据目录，如有则跳过读取数据目录部分代码：

```
004775A3  |.  8B55 EC      mov     edx, dword ptr [ebp-14]

004775A6  |.  66:817A 10 E>cmp     word ptr [edx+10], 0E0

004775AC  |.  75 49        jnz     short 004775F7
```

读取数据目录，并检查结果：

```
004775AE  |.  57           push    edi                   ; /Arg4

004775AF  |.  68 E0000000  push    0E0                   ; |Arg3 = 000000E0
```



### 检查 NT 头中相关信息，并做相应的标识：

```
004775CC  .  66:83BD 68E4>cmp     word ptr [ebp-1B98], 3
004775D4  .  75 07        jnz     short 004775DD
004775D6  .  C745 FC 1000>mov     dword ptr [ebp-4], 10
004775DD  |>  81BD 44E4FFF>cmp     dword ptr [ebp-1BBC], 1000
004775E7  .  73 0E        jnb     short 004775F7
004775E9  .  833D D8364D0>cmp     dword ptr [4D36D8], 2  ;检查是否 WinNT 平台
004775F0  .  74 05        je      short 004775F7
004775F2  .  BB 03000000  mov     ebx, 3
```

### 最后关闭文件：

```
004775F7  |>  85FF         test    edi, edi
004775F9  .  74 07        je      short 00477602
004775FB  .  57           push    edi
004775FC  .  E8 D7D80200  call    _fclose
00477601  .  59           pop     ecx
```

### 根据标识 `ebx` 对文件做相应的处理

#### `ebx = 1` 时，弹出错误信息，返回 `-1` 并退出：

```
00477602  |>  83FB 01      cmp     ebx, 1              ;  Switch (cases 1..3)
00477605  .  75 1E        jnz     short 00477625
00477607  .  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]    ;  Case 1 of switch 00477602
0047760D  .  50           push    eax                 ; /Arg2
0047760E  .  8D96 5D07000>lea     edx, dword ptr [esi+75D]    ; |
00477614  .  52           push    edx                 ; |Arg1
00477615  .  E8 02CAFDFF  call    _Error              ; \\_Error
0047761A  .  83C4 08      add     esp, 8
0047761D  .  83C8 FF      or      eax, FFFFFFFF       ;返回-1
00477620  .  E9 62040000  jmp     00477A87            ;跳转到结束处
```

#### `ebx = 2` 时，弹出选择对话框，根据选择做相应的处理：

```
00477625  |>  83FB 02      cmp     ebx, 2
00477628  .  75 4C        jnz     short 00477676
0047762A  .  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]    ;  Case 2 of switch 00477602
00477630  .  52           push    edx                 ; /Arg3
00477631  .  8D8E 7E07000>lea     ecx, dword ptr [esi+77E]    ; |
00477637  .  51           push    ecx                 ; |Arg2
00477638  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]    ; |
0047763E  .  50           push    eax                 ; |Arg1
0047763F  .  E8 E8F50200  call    004A6C2C            ; \\_sprintf
00477644  .  83C4 0C      add     esp, 0C
00477647  .  8D96 CD07000>lea     edx, dword ptr [esi+7CD]
0047764D  .  8D8D 54F8FFF>lea     ecx, dword ptr [ebp-7AC]
00477653  .  A1 7C3B4D00  mov     eax, dword ptr [4D3B7C]
00477658  .  68 24210000  push    2124                ; /Style
0047765D  .  52           push    edx                 ; |Title
0047765E  .  51           push    ecx                 ; |Text
0047765F  .  50           push    eax                 ; |hOwner
00477660  .  E8 B17E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
00477665  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
00477668  .  0F84 B400000>je      00477722
0047766E  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
00477671  .  E9 11040000  jmp     00477A87  ;跳转到结束处
```

#### `ebx = 3` 时，弹出选择对话框，根据选择做相应的处理：

```
00477676  |>  83FB 03      cmp     ebx, 3
00477679  .  75 48        jnz     short 004776C3
0047767B  .  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]    ;  Case 3 of switch 00477602
00477681  .  52           push    edx                 ; /Arg3
00477682  .  8D8E E107000>lea     ecx, dword ptr [esi+7E1]    ; |
00477688  .  51           push    ecx                 ; |Arg2
00477689  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]    ; |
0047768F  .  50           push    eax                 ; |Arg1
00477690  .  E8 97F50200  call    004A6C2C            ; \\_sprintf
00477695  .  83C4 0C      add     esp, 0C
00477698  .  8D96 2E08000>lea     edx, dword ptr [esi+82E]
0047769E  .  8D8D 54F8FFF>lea     ecx, dword ptr [ebp-7AC]
004776A4  .  A1 7C3B4D00  mov     eax, dword ptr [4D3B7C]
004776A9  .  68 24210000  push    2124                ; /Style
004776AE  .  52           push    edx                 ; |Title
004776AF  .  51           push    ecx                 ; |Text
004776B0  .  50           push    eax                 ; |hOwner
004776B1  .  E8 607E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
004776B6  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
004776B9  .  74 67        je      short 00477722
004776BB  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
004776BE  .  E9 C4030000  jmp     00477A87  ;跳转到结束处
```

### 默认处理处：弹出选择对话框，根据选择做相应的处理

```
004776C3  |>  8B55 EC      mov     edx, dword ptr [ebp-14]     ;  Default
004776C6  .  F642 13 20    test    byte ptr [edx+13], 20
004776CA  .  74 56        je      short 00477722
004776CC  .  837D 0C FF    cmp     dword ptr [ebp+C], -1
004776D0  .  74 49        je      short 0047771B
004776D2  .  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]
004776D8  .  51           push    ecx                 ; /Arg3
004776D9  .  8D86 4108000>lea     eax, dword ptr [esi+841]    ; |
004776DF  .  50           push    eax                 ; |Arg2
004776E0  .  8D95 54F8FFF>lea     edx, dword ptr [ebp-7AC]    ; |
004776E6  .  52           push    edx                 ; |Arg1
004776E7  .  E8 40F50200  call    004A6C2C            ; \\_sprintf
004776EC  .  83C4 0C      add     esp, 0C
004776EF  .  8D8E 9F08000>lea     ecx, dword ptr [esi+89F]
004776F5  .  8D85 54F8FFF>lea     eax, dword ptr [ebp-7AC]
004776FB  .  8B15 7C3B4D0>mov     edx, dword ptr [4D3B7C]
00477701  .  68 24200000  push    2024                ; /Style
00477706  .  51           push    ecx                 ; |Title
00477707  .  50           push    eax                 ; |Text
00477708  .  52           push    edx                 ; |hOwner
00477709  .  E8 087E0300  call    <jmp.&USER32.MessageBoxA>     ; \\MessageBoxA
0047770E  .  83F8 06      cmp     eax, 6    ;检查选择是否为 IDYES
00477711  .  74 08        je      short 0047771B
00477713  .  83C8 FF      or      eax, FFFFFFFF  ;选择为 IDNO 时，返回-1
00477716  .  E9 6C030000  jmp     00477A87  ;跳转到结束处
```

### 设置加载文件为 DLL 标识：

```
0047771B  |>  C745 F8 0100>mov     dword ptr [ebp-8], 1
```

### 检查是否还有进程被加载调试，将其关闭，并检查返回值，失败则返回 `-1` 并退出：

```
00477722  |>  6A 01        push    1               ; /Arg1 = 00000001
00477724  .  E8 7BE1FFFF  call    004758A4         ; \\Ollydbg.004758A4
00477729  .  59           pop     ecx
0047772A  .  85C0         test    eax, eax
0047772C  .  74 08        je      short 00477736
0047772E  .  83C8 FF      or      eax, FFFFFFFF
00477731  .  E9 51030000  jmp     00477A87  ;跳转到结束处
```

### 清除原调试程序中的所有相关信息，检查返回值，成功则跳过错误处理：

```
00477736  |>  E8 D1DCFFFF  call    0047540C
0047773B  .  85C0         test    eax, eax
0047773D  .  74 15        je      short 00477754
```

### 报错、返回 `-1`，并退出：

```
0047773F  .  8D96 B308000>lea     edx, dword ptr [esi+8B3]
00477745  .  52           push    edx                 ; /Arg1
00477746  .  E8 D1C8FDFF  call    _Error              ; \\_Error
0047774B  .  59           pop     ecx
0047774C  .  83C8 FF      or      eax, FFFFFFFF
0047774F  .  E9 33030000  jmp     00477A87  ;跳转到结束处
```

### 判断打开的是否为 Dll 文件，如果不是则跳过检查文件是否存在：



`00477754  |>  8B55 F8      mov     edx, dword ptr [ebp-8]  ;ebp-8 标识是否为 dll 文件`  

`00477757  |.  33C9         xor     ecx, ecx`  

`00477759  |.  8915 A06E4D0>mov     dword ptr [4D6EA0], edx  ;设置全局变量 4D6EA0，保存是否为 dll 文件，1 为 dll，0 为非 dll`  

`0047775F  |.  890D A46E4D0>mov     dword ptr [4D6EA4], ecx`  

`00477765  |.  833D A06E4D0>cmp     dword ptr [4D6EA0], 0  ;检查是否为 DLL 文件`  

`0047776C  |.  74 1E        je      short 0047778C  ;如果不是 DLL 文件则跳过检查 LoadDll.exe 是否存在`  

若不存在则报错，返回-1 并退出：  

`0047776E  |.  E8 997CF9FF  call    0040F40C  ; 该函数判断与 OD 同一目录下是否存在 LoadDll.exe，若不存在，则从 OD 可执行程序中解压资源 RES_LOADDLL，生成 LoadDll.exe 程序`  

`00477773  |.  85C0         test    eax, eax`  

`00477775  |.  7D 15        jge     short 0047778C`  

`00477777  |.  8D86 D408000>lea     eax, dword ptr [esi+8D4]`  

`0047777D  |.  50           push    eax                     ; /Arg1`  

`0047777E  |.  E8 99C8FDFF  call    _Error                    ; \\_Error`  

`00477783  |.  59           pop     ecx`  

`00477784  |.  83C8 FF      or      eax, FFFFFFFF  ;返回-1`  

`00477787  |.  E9 FB020000  jmp     00477A87    ;跳转到结束处`  

通过全文件路径名，得到其所在的文件夹，检查该调试程序的相关配置文件：  

`0047778C  |>  33C0         xor     eax, eax`  

`0047778E  |.  56           push    esi`  

`0047778F  |.  8DBD E0FEFFF>lea     edi, dword ptr [ebp-120]`  

`00477795  |.  83C9 FF      or      ecx, FFFFFFFF`  

`00477798  |.  F2:AE        repne   scas byte ptr es:[edi]`  

`0047779A  |.  F7D1         not     ecx`  

`0047779C  |.  2BF9         sub     edi, ecx`  

`0047779E  |.  BE 805B4D00  mov     esi, 004D5B80                   ;  ASCII "D:\\CrackMe.exe"`  

`004777A3  |.  87F7         xchg    edi, esi`  

`004777A5  |.  8BD1         mov     edx, ecx`  

`004777A7  |.  8BC7         mov     eax, edi`  

`004777A9  |.  C1E9 02      shr     ecx, 2`  

`004777AC  |.  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]`  

`004777B2  |.  F3:A5        rep     movs dword ptr es:[edi], dwor>`  

`004777B4  |.  8BCA         mov     ecx, edx`  

`004777B6  |.  83E1 03      and     ecx, 3`  

`004777B9  |.  F3:A4        rep     movs byte ptr es:[edi], byte >`  

`004777BB  |.  5E           pop     esi`  

`004777BC  |.  6A 00        push    0`  

`004777BE  |.  68 7C5A4D00  push    004D5A7C                    ;  ASCII "CrackMe"`  

`004777C3  |.  6A 00        push    0`  

`004777C5  |.  6A 00        push    0`  

`004777C7  |.  50           push    eax`  

`004777C8  |.  E8 EFD90200  call    004A51BC`  

`004777CD  |.  83C4 14      add     esp, 14`  

`004777D0  |.  6A 00        push    0                   ; /Arg2 = 00000000`  

`004777D2  |.  68 7C5A4D00  push    004D5A7C       ; |Arg1 = 004D5A7C ASCII "CrackMe"`  

`004777D7  |.  E8 941BF9FF  call    00409370          ; \\判断与 OD 可执行程序同目录下是否存在 common.arg 文件，若有就加载处理`  

`004777DC  |.  83C4 08      add     esp, 8`  

`004777DF  |.  E8 743AFFFF  call    0046B258    ; 读取 OD 的配置文件，获取导入库等信息`  

`004777E4  |.  803D 845C4D0>cmp     byte ptr [4D5C84], 0`  

`004777EB  |.  74 2C        je      short 00477819`  

`004777ED  |.  33C0         xor     eax, eax`  

`004777EF  |.  56           push    esi`  

`004777F0  |.  83C9 FF      or      ecx, FFFFFFFF`  

`004777F3  |.  BF 845C4D00  mov     edi, 004D5C84`  

`004777F8  |.  F2:AE        repne   scas byte ptr es:[edi]`  

`004777FA  |.  F7D1         not     ecx`  

`004777FC  |.  2BF9         sub     edi, ecx`  

`004777FE  |.  8DB5 DCFDFFF>lea     esi, dword ptr [ebp-224]`  

`00477804  |.  87F7         xchg    edi, esi`  

`00477806  |.  8BD1         mov     edx, ecx`  

`00477808  |.  8BC7         mov     eax, edi`  

`0047780A  |.  C1E9 02      shr     ecx, 2`  

`0047780D  |.  F3:A5        rep     movs dword ptr es:[edi], dwor>`  

`0047780F  |.  8BCA         mov     ecx, edx`  

`00477811  |.  83E1 03      and     ecx, 3`  

`00477814  |.  F3:A4        rep     movs byte ptr es:[edi], byte >`  

`00477816  |.  5E           pop     esi`  

`00477817  |.  EB 3C        jmp     short 00477855`  

将全路径名分解成驱动器名、路径名、文件名、后缀名：  

`00477819  |>  6A 00        push    0`  

`0047781B  |.  6A 00        push    0`  

`0047781D  |.  8D85 D8FBFFF>lea     eax, dword ptr [ebp-428]`  

`00477823  |.  50           push    eax`  

`00477824  |.  8D55 F5      lea     edx, dword ptr [ebp-B]`  

`00477827  |.  52           push    edx`  

`00477828  |.  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]`  

`0047782E  |.  51           push    ecx`  

`0047782F  |.  E8 88D90200  call    004A51BC    ; _fnsplit`  

`00477834  |.  83C4 14      add     esp, 14`  

将拆分出来的文件名重新组合：  

`00477837  |.  8D85 D8FBFFF>lea     eax, dword ptr [ebp-428]`  

`0047783D  |.  8D55 F5      lea     edx, dword ptr [ebp-B]`  

`00477840  |.  8D8D DCFDFFF>lea     ecx, dword ptr [ebp-224]`  

`00477846  |.  6A 00        push    0                   ;  / ext = 00000000`  

`00477848  |.  6A 00        push    0                   ;  | name = 00000000`  

`0047784A  |.  50           push    eax                 ;  | dir`  

`0047784B  |.  52           push    edx                 ;  | drive`  

`0047784C  |.  51           push    ecx                 ;  | path`  

`0047784D  |.  E8 46D90200  call    004A5198               ;  \\ _fnmerge`  

`00477852  |.  83C4 14      add     esp, 14`  

以下是 `STARTUPINFO` 类型的变量赋值：  

`00477855  |>  C785 04E5FFF>mov     dword ptr [ebp-1AFC], 44`  

`0047785F  |.  33C0         xor     eax, eax`  

`00477861  |.  33D2         xor     edx, edx`  

`00477863  |.  8985 08E5FFF>mov     dword ptr [ebp-1AF8], eax`  

`00477869  |.  33C9         xor     ecx, ecx`  

`0047786B  |.  8995 0CE5FFF>mov     dword ptr [ebp-1AF4], edx`  

`00477871  |.  898D 10E5FFF>mov     dword ptr [ebp-1AF0], ecx`  

`00477877  |.  C785 30E5FFF>mov     dword ptr [ebp-1AD0], 81`  

`00477881  |.  66:C785 34E5>mov     word ptr [ebp-1ACC], 0A`  

`0047788A  |.  66:C785 36E5>mov     word ptr [ebp-1ACA], 0`  

`00477893  |.  33C0         xor     eax, eax`  

`00477895  |.  8985 38E5FFF>mov     dword ptr [ebp-1AC8], eax`  

`0047789B  |.  837D 0C 01   cmp     dword ptr [ebp+C], 1`  

`0047789F  |.  74 09        je      short 004778AA`  

检查加载的是 DLL 还是 EXE，根据加载的不同生成不同的命令行参数：  

`004778A1  |.  833D A06E4D0>cmp     dword ptr [4D6EA0], 0`  

`004778A8  |.  74 07        je      short 004778B1`  

`004778AA  |>  C605 885D4D0>mov     byte ptr [4D5D88], 0`  

`004778B1  |>  833D A06E4D0>cmp     dword ptr [4D6EA0], 0`  

`004778B8  |.  74 24        je      short 004778DE`  

如果加载的是 DLL，组合出命令行：`"\Path\LoadDll.exe" \Path\XXX.dll` 传递给下面的 `CreateProcess` 函数：  

`004778BA  |.  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]`  

`004778C0  |.  8D8E 5809000>lea     ecx, dword ptr [esi+958]`  

`004778C6  |.  52           push    edx                 ; /Arg4`  

`004778C7  |.  68 68384D00  push    004D3868            ; |Arg3`  

`004778CC  |.  51           push    ecx                 ; |Arg2`  

`004778CD  |.  8D85 50E7FFF>lea     eax, dword ptr [ebp-18B0]     ;  |`  

`004778D3  |.  50           push    eax                 ; |Arg1`  

`004778D4  |.  E8 53F30200  call    004A6C2C          ; \\_sprintf`  

`004778D9  |.  83C4 10      add     esp, 10`  

`004778DC  |.  EB 4A        jmp     short 00477928`  

如果加载的是 EXE，将 EXE 文件名传给下面的 `CreateProcess` 函数：  

`004778DE  |>  803D 885D4D0>cmp     byte ptr [4D5D88], 0`  

`004778E5  |.  74 24        je      short 0047790B`  

`004778E7  |.  68 885D4D00  push    004D5D88          ; /Arg4 = 004D5D88`  

`004778EC  |.  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]     ;  |`  

`004778F2  |.  52           push    edx                 ; |Arg3`  

`004778F3  |.  8D8E 6C09000>lea     ecx, dword ptr [esi+96C]     ;  |`  

`004778F9  |.  51           push    ecx                 ; |Arg2`  

`004778FA  |.  8D85 50E7FFF>lea     eax, dword ptr [ebp-18B0]    ;  |`  

`00477900  |.  50           push    eax                 ; |Arg1`  

`00477901  |.  E8 26F30200  call    004A6C2C          ; \\_sprintf`  

`00477906  |.  83C4 10      add     esp, 10`  



00477909  |.  EB 1D        jmp     short 00477928  

0047790B  |>  8D95 E0FEFFF>lea     edx, dword ptr [ebp-120]  

00477911  |.  52          push    edx                       ; /Arg3  

00477912  |.  8D8E C403000>lea     ecx, dword ptr [esi+3C4]       ; |  

00477918  |.  51          push    ecx                       ; |Arg2  

00477919  |.  8D85 50E7FFF>lea     eax, dword ptr [ebp-18B0]      ; |  

0047791F  |.  50          push    eax                       ; |Arg1  

00477920  |.  E8 07F30200  call    004A6C2C                  ; \__sprintf  

00477925  |.  83C4 0C     add     esp, 0C  

调用 `CreateProcessA` 这个函数打开调试线程，其中，`CreationFlags` = `DEBUG_ONLY_THIS_PROCESS` | `CREATE_NEW_CONSOLE` | `NORMAL_PRIORITY_CLASS` | `CREATE_DEFAULT_ERROR_MODE`，检查是否打开成功，若失败则返回 `-1` 并退出：  

00477928  |>  8D95 14E4FFF>lea     edx, dword ptr [ebp-1BEC]  

0047792E  |.  8D8D 04E5FFF>lea     ecx, dword ptr [ebp-1AFC]  

00477934  |.  52          push    edx                       ; /pProcessInfo  

00477935  |.  51          push    ecx                       ; |pStartupInfo  

00477936  |.  8D85 DCFDFFF>lea     eax, dword ptr [ebp-224]       ; |  

0047793C  |.  8D8D 50E7FFF>lea     ecx, dword ptr [ebp-18B0]      ; |  

00477942  |.  50          push    eax                       ; |CurrentDir  

00477943  |.  6A 00       push    0                         ; |pEnvironment = NULL  

00477945  |.  8B55 FC     mov     edx, dword ptr [ebp-4]      ; |  

00477948  |.  81CA 2200000>or      edx, 4000022              ; |  

0047794E  |.  52          push    edx                       ; |CreationFlags  

0047794F  |.  6A 00       push    0                         ; |InheritHandles = FALSE  

00477951  |.  6A 00       push    0                         ; |pThreadSecurity = NULL  

00477953  |.  6A 00       push    0                         ; |pProcessSecurity = NULL  

00477955  |.  51          push    ecx                       ; |CommandLine  

00477956  |.  6A 00       push    0                         ; |ModuleFileName = NULL  

00477958  |.  E8 73760300  call    <jmp.&KERNEL32.CreateProcessA>; \CreateProcessA  

0047795D  |.  85C0        test    eax, eax  ;检查返回值  

0047795F  |.  75 1E       jnz     short 0047797F  ;成功则跳过错误处理  

打开线程的错误处理部分，返回 `-1` 并退出：  

00477961  |.  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]  

00477967  |.  50          push    eax                       ; /Arg2  

00477968  |.  8D96 7409000>lea     edx, dword ptr [esi+974]     ; |  

0047796E  |.  52          push    edx                       ; |Arg1  

0047796F  |.  E8 A8C6FDFF  call    _Error                  ; \__Error  

00477974  |.  83C4 08     add     esp, 8  

00477977  |.  83C8 FF     or      eax, FFFFFFFF  ;返回-1  

0047797A  |.  E9 08010000  jmp     00477A87    ;跳转结束处  

将 `CreateProcess` 中获得的 `PROCESS_INFORMATION` 的四个值赋值给全局变量：  

0047797F  |>  8B95 14E4FFF>mov     edx, dword ptr [ebp-1BEC]  

00477985  |.  8915 605A4D0>mov     dword ptr [4D5A60], edx  

0047798B  |.  8B8D 18E4FFF>mov     ecx, dword ptr [ebp-1BE8]  

00477991  |.  890D 645A4D0>mov     dword ptr [4D5A64], ecx  

00477997  |.  C705 685A4D0>mov     dword ptr [4D5A68], 6  

004779A1  |.  C705 6C5A4D0>mov     dword ptr [4D5A6C], 6  

004779AB  |.  8B85 1CE4FFF>mov     eax, dword ptr [ebp-1BE4]  

004779B1  |.  A3 705A4D00  mov     dword ptr [4D5A70], eax  

004779B6  |.  8B95 20E4FFF>mov     edx, dword ptr [ebp-1BE0]  

004779BC  |.  8915 745A4D0>mov     dword ptr [4D5A74], edx  

004779C2  |.  8B4D F0     mov     ecx, dword ptr [ebp-10]  

设置 OD 的标题显示内容：  

004779C5  |.  51          push    ecx                       ; /Arg4  

004779C6  |.  8D86 A503000>lea     eax, dword ptr [esi+3A5]       ; |  

004779CC  |.  50          push    eax                       ; |Arg3  

004779CD  |.  8D96 8E09000>lea     edx, dword ptr [esi+98E]       ; |  

004779D3  |.  52          push    edx                       ; |Arg2  

004779D4  |.  8D8D E0FEFFF>lea     ecx, dword ptr [ebp-120]       ; |  

004779DA  |.  51          push    ecx                       ; |Arg1  

004779DB  |.  E8 4CF20200  call    004A6C2C                  ; \__sprintf  

004779E0  |.  83C4 10     add     esp, 10  

004779E3  |.  8D85 E0FEFFF>lea     eax, dword ptr [ebp-120]  

004779E9  |.  50          push    eax                       ; /Text  

004779EA  |.  8B15 7C3B4D0>mov     edx, dword ptr [4D3B7C]       ; |  

004779F0  |.  52          push    edx                       ; |hWnd  

004779F1  |.  E8 C87B0300  call    <jmp.&USER32.SetWindowTextA>  ; \SetWindowTextA  

004779F6  |.  C705 88574D0>mov     dword ptr [4D5788], 1  

下面是一些设置界面部分的代码：  

00477A00  |.  33C9        xor     ecx, ecx  

00477A02  |.  890D 94574D0>mov     dword ptr [4D5794], ecx  

00477A08  |.  8D46 14     lea     eax, dword ptr [esi+14]  

00477A0B  |.  50          push    eax                       ; /Arg3  

00477A0C  |.  6A 00       push    0                         ; |Arg2 = 00000000  

00477A0E  |.  6A 00       push    0                         ; |Arg1 = 00000000  

00477A10  |.  E8 F72BFEFF  call    _Addtolist              ; \__Addtolist  

00477A15  |.  83C4 0C     add     esp, 0C  

00477A18  |.  837D FC 00  cmp     dword ptr [ebp-4], 0  

00477A1C  |.  74 18       je      short 00477A36  

00477A1E  |.  68 805B4D00  push    004D5B80      ; /Arg3 = 004D5B80 ASCII "D:\CrackMe.exe"  

00477A23  |.  8D96 9609000>lea     edx, dword ptr [esi+996]     ; |  

00477A29  |.  52          push    edx                       ; |Arg2  

00477A2A  |.  6A 00       push    0                         ; |Arg1 = 00000000  

00477A2C  |.  E8 FF9BFBFF  call    _Message                 ; \__Message  

00477A31  |.  83C4 0C     add     esp, 0C  

00477A34  |.  EB 16       jmp     short 00477A4C  

00477A36  |>  68 805B4D00  push    004D5B80      ; /Arg3 = 004D5B80 ASCII "D:\CrackMe.exe"  

00477A3B  |.  8D8E A809000>lea     ecx, dword ptr [esi+9A8]     ; |  

00477A41  |.  51          push    ecx                       ; |Arg2  

00477A42  |.  6A 00       push    0                         ; |Arg1 = 00000000  

00477A44  |.  E8 E79BFBFF  call    _Message                 ; \__Message  

00477A49  |.  83C4 0C     add     esp, 0C  

00477A4C  |>  803D 885D4D0>cmp     byte ptr [4D5D88], 0  

00477A53  |.  74 16       je      short 00477A6B  

00477A55  |.  68 885D4D00  push    004D5D88                   ; /Arg3 = 004D5D88  

00477A5A  |.  8D86 B209000>lea     eax, dword ptr [esi+9B2]     ; |  

00477A60  |.  50          push    eax                       ; |Arg2  

00477A61  |.  6A 00       push    0                         ; |Arg1 = 00000000  

00477A63  |.  E8 C89BFBFF  call    _Message                 ; \__Message  

00477A68  |.  83C4 0C     add     esp, 0C  

读写配置文件，主要是操作一些最近打开的文件之类的配置值：  

00477A6B  |>  68 885D4D00  push    004D5D88                   ; /Arg2 = 004D5D88  

00477A70  |.  68 805B4D00  push    004D5B80      ; |Arg1 = 004D5B80 ASCII "D:\CrackMe.exe"  

00477A75  |.  E8 8AF3FFFF  call    00476E04                  ; \Ollydbg.00476E04  

00477A7A  |.  83C4 08     add     esp, 8  

00477A7D  |.  6A 03       push    3                         ; /Arg1 = 00000003  

00477A7F  |.  E8 F49EFBFF  call    00431978                  ; \Ollydbg.00431978  

00477A84  |.  59          pop     ecx  

返回 `0`，恢复现场，并退出：  

00477A85  |.  33C0        xor     eax, eax  

00477A87  |>  5F          pop     edi  

00477A88  |.  5E          pop     esi  

00477A89  |.  5B          pop     ebx  

00477A8A  |.  8BE5        mov     esp, ebp  

00477A8C  |.  5D          pop     ebp  

00477A8D  \.  C3          retn  

OD 打开被调试程序之后，进入调试循环体，在调试循环体中通过读取文件的 `OEP`，在单步的方式下对 `OEP` 下 `Int3` 断点，断下程序后，恢复 `OEP` 处 `Int3` 断点为初始指令码。



这个流程可以在本系列前几个文档中得到验证，这里就不再加以说明了。

武汉科锐学员：`driverox`

2008-5-31
