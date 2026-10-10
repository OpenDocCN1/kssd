

# 手脱 UPX 加密壳及修复 IAT 表

> 原文：[`www.pediy.com/kssd/pediy10/62098.html`](https://www.pediy.com/kssd/pediy10/62098.html)

首先声明：本人是一个地地道道的菜鸟，尤其在脱壳方面更是菜得不行，本文的目的旨在探寻一种更好的方式，高手见笑了。恳请各位高手看了之后，能够给予指点，万分感激。菜鸟们来一起相互学习,交流经验吧！  
  
目标软件：[`bbs.pediy.com/showthread.php?...&highlight=UPX`](http://bbs.pediy.com/showthread.php?t=61809&highlight=UPX)  
使用工具：OllyDbg,  IATRebulid V1.02  
  
**一、PEiD 查壳：**UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo  
刚一看到这个信息，心中暗喜，UPX 的壳，以为小菜一碟，用 ESP 定律脱之，OEP 处代码如下:  

代码:

```
005D9E18    55              PUSH EBP
005D9E19    8BEC            MOV EBP,ESP
005D9E1B    83C4 F0         ADD ESP,-10
005D9E1E    53              PUSH EBX
005D9E1F    B8 18955D00     MOV EAX,WishGJ.005D9518
005D9E24    E8 FFD4E2FF     CALL WishGJ.00407328

```

1. 脱后，双击运行，Windows 居然提示不是有效的 Win32 应用程序。  
2. 于是请出 ImportREC, 输入 OEP 处地址，自动获取输入表，发现里面有大量的无效指针，进行自动跟踪失败，于是剪去无效指针，再对脱壳后的程序进行修复。双击运行，Windows 又提示”应用程序初始化错误！  
这可能是由于刚才将一些有用的指针剪掉造成的。由于无效指针太多，真不敢一个一个地去手工跟踪手复这些指针！  
  
不知高手们是如何修复这些无效指针的(也许，他们有自已的独门兵器)，望不吝赐教我们这些菜鸟，十分感谢！  
  
呵呵，菜鸟有菜鸟的方法，我用自制的土工具 IATRebulid 来轻松解决这一问题。  
  
**二、解决“应用程序初始化错误”的问题**  
    用 IATRebulid 打开经 ImportREC 修补过的程序，发现已经引入了很多 DLL，这是个好现象。  
![](img/831e70fc8591af926a285c393be98a19.jpg)  
  
1.  点击 \[导出 IAT\] 按钮，将程序中的 IAT 表导出成一个文件，作为备份。  
2.  点击 \[删除 IAT\] 按钮，删掉程序中的 IAT 表，这样就可以避免因 IAT 表错误，而造成 Windows 无法加载程序.  
3.  再双击程序图标运行，Windows 弹出了一个异常消息框。OK，对于我们来说，这是一个好消息，因为 Windows 已经可以加载程序了，只不过是程序在运行中出现了一些问题。这些问题我们都可以利用 OllyDbg 分析出原因，从而再找出相应的方法进行修补。  
  
**二、增加一个新节，用于存放修复后的 IAT 表**1.   
软件用 OD 脱壳后，其 PE 头部尺寸一般会扩大到 1000h, 但在其头部尺寸却还是保持原来的值，这在增加新节时可能会引起一些问题。点击 IATRebulid 中的 \[PE 头部\] 按钮，可查看并修改这个数据，如下图：  
![](img/f4360eceaa5f48d44c28d15c979088f1.jpg)  
将 SizeOfHead 一栏中的 00000200 改为 00001000，然后点击右边的 \[3C\] 按钮，即可这个值改为 1000h 大小。  
  
2. 增加新节, 由于这个 IAT 表比较大，所以增加一个 4000h 大小的新节，如下图所示：   
![](img/f806f705bf08c371e381c470091f0cbf.jpg)  
增加后，新节名称为：NewSec, 新节 RVA 地址为：003E7000.  
  
**三、将原 IAT 表移到新节上来：**  
点击 IATRebulid 中的 \[重建 IAT\] 按钮，然后选择刚才备份的 IAT 文件，将这个 IAT 表安排在新节的位置上：003E7000。  
好，前期工作已完成，再双击运行程序一试，还是出现异常提示框，现在开始手工修复 IAT 表。  
  
**四、寻找 API 函数信息：**OD 载入新程序，F7 单步跟踪，几步可以跟到：  
00407264  - FF25 B8325E00   JMP DWORD PTR DS:\[5E32B8\]  
这里是一个没有修复的 API 函数调用，再跟踪加壳程序到相同位置，再跟进，即可到：  
代码:

```
00DB047E    F8              CLC
00DB047F    93              XCHG EAX,EBX
00DB0480    68 AB4546E8     PUSH E84645AB
00DB0485    812C24 A50266B0 SUB DWORD PTR SS:\[ESP\],B06602A5
00DB048C    93              XCHG EAX,EBX
00DB048D    813424 2FF6604B XOR DWORD PTR SS:\[ESP\],4B60F62F
00DB0494    C3              RETN

```

这里返回后，即转入 GetModuleHandleA，很明显这里是壳对引入函数进行了加密，当程序需要调用 GetModuleHandleA 函数时，先来到外壳的代码处，进行变换后，再转入到 API 函数处。由于类似这样的函数特多，我想谁也不想一个一个地进行跟踪修复吗。  
外壳要这样处理，肯定先要集中处理 API 函数地址，我们可以在其集中处理时，把这些 API 函数一次性全部找出来，于是在 5E32B8 处下内存写入断点，但像断不下来. 可能是这个壳调用了 VirtualProtect 函数，更改了内存段的保护属性，用来抵抗 OD 的访问断点。   
bp VirtualProtect 下断，重新运行程序，OD 果然有很多次中断，注意堆栈值，其中有一次如下：，  
代码:

```
0012FF70   007DB77F  /CALL 到 VirtualProtect 来自 WishGJ.007DB779
0012FF74   005E3000  |Address = WishGJ.005E3000
0012FF78   00004000  |Size = 4000 (16384.)
0012FF7C   00000040  |NewProtect = PAGE\_EXECUTE\_READWRITE
0012FF80   007DD7B6  \\pOldProtect = WishGJ.007DD7B6

```

可以看出 5E32B8 正好落在这个设置的区域上，于是这次让程序跳过 VirtualProtect 内部的代码(同时取消 VirtualProtect 的断点)，直接返回. 在 5E32B8 处设下内存写入断点。F9 运行，程序就中断在：  
代码:

```
007DB897   .  8907          MOV DWORD PTR DS:\[EDI\],EAX  ; 这里就是往 5E32B8 地址处写入数据
007DB899   .  83C7 04       ADD EDI,4
007DB89C   .  8B85 6B260000 MOV EAX,DWORD PTR SS:\[EBP+266B\]    
007DB8A2   .  EB 01         JMP SHORT WishGJ.007DB8A5
007DB8A4   >  40            INC EAX
007DB8A5   >  8038 00       CMP BYTE PTR DS:\[EAX\],0  ; 得到下一个 API 函数名称
007DB8A8   .^ 75 FA         JNZ SHORT WishGJ.007DB8A4

```

在这里可以发现，程序在将变换过的 API 函数地址存放到指定的地址上去后，然后取下一个 API 函数名称，这时，可以在 OD 的数据窗口中跟踪 EAX 就可以发现有大量的 API 函数名称, 如下：  
代码:

```
...
003F03B6  43 72 65 61 74 65 46 69 6C 65 41 00 43 6C 6F 73  CreateFileA.Clos
003F03C6  65 48 61 6E 64 6C 65 00 00 78 32 5E 00 47 65 74  eHandle..x2^.Get
003F03D6  4B 65 79 62 6F 61 72 64 54 79 70 65 00 4C 6F 61  KeyboardType.Loa
003F03E6  64 53 74 72 69 6E 67 41 00 4D 65 73 73 61 67 65  dStringA.Message
003F03F6  42 6F 78 41 00 43 68 61 72 4E 65 78 74 41 00 00  BoxA.CharNextA..
003F0406  8C 32 5E 00 52 65 67 51 75 65 72 79 56 61 6C 75  ?^.RegQueryValu
...

```

在 OD 的数据窗口中通过上/下翻，将包含所有 API 函数名称的数据全部选中，再右键，复制到一个文件中。  
好，到这一步，API 函数名称可能已经全部找到，  
  
**五、整理 API 函数名称**  
上一步中复制到文件中的只是 API 函数的一些字节数据，可以通过 IATRebulid 将其整理成 API 函数名称，操作如下：  
1. 点击 IATRebulid 主窗口中的 \[解读字符\] 按钮，弹出下面的窗口  
2. 打开刚才复制数据的文件，将里面的数据复制到下面窗口的左边编辑框中.  
3. 点击 \[解读数据\] 按钮，在右边编辑框中就会解析出相对应的 API 函数名称。  
4. 点击 \[保存结果\]，将解析出的 API 函数字符串保存到一个文件中去。  
注意：由于这个程序的 API 函数数据太多，这种编辑框一次无法全部容纳下所有数据，需要两次这样的操作方可.(我偷懒了一下，没有用 Rich Edit)  
![](img/910b473c981415eb67e4f09bf11506ad.jpg)  
这样，就将所有 API 函数名称字符串就全部整理出来了,合并到一个文件中，下一步就到了修复引入表中的 API 函数了。  
  
**六、修补引入表中的 API 函数。**  
根据上面的第四步，可知：\[5E32B8\]是 GetModuleHandleA 函数的地址，从刚才整理出的 API 函数文件中找到 GetModuleHandleA 这个函数，共找到 3 处，说明 Kernel32.dll 有 3 组引入函数数据，  
再打开最先导出 IAT 备份文件，找到与 5E32B8 相邻的上下两个 DLL 数据，如下：  
代码:

```
\[oleaut32.dll\] 005E329C
SysFreeString
SysReAllocStringLen
SysAllocStringLen

\[ADVAPI32.dll\] 005E32C0
RegSetValueExA
RegQueryValueExA
RegQueryInfoKeyA

```

可以看出这中间能够插入的 API 函数最多只能有 5 个，而上面找到的 3 组 Kernel32.dll 函数数据中只有一组的函数数量小于 5 个，这样就可以确定数量最小的那组数据是放在这里的，如是 IAT 文件修补如下：  
代码:

```
\[oleaut32.dll\] 005E329C
SysFreeString
SysReAllocStringLen
SysAllocStringLen

\[kernel32.dll\]
TlsSetValue
TlsGetValue
LocalAlloc
GetModuleHandleA 5E32B8
\[ADVAPI32.dll\] 005E32C0
RegSetValueExA
RegQueryValueExA
RegQueryInfoKeyA

```

上面红色部分就是新增的 DLL 引入函数，其中 GetMoudleHandleA 附带了这个函数在 IAT 表中的 VA 地址(说明，一组 DLL 数据中只能带一个 VA 地址数据，可以安排在任意一个函数中，也可以安排在 DLL 名称后面), 保存修改后的 IAT 文件。   
用 IATRebulid 用这个文件重建 IAT 表，IAT 表可以还是安排在新增最后一个节的起始 VA(003E7000)上，重建后，双击运行，没有反应。看来还有引入函数没有修复。  
用 OD 中跟踪此程序，发现是下面一条语句上出现了问题：  
004013A8   $- FF25 20325E00 JMP DWORD PTR DS:\[5E3220\]  
再用 OD 跟踪加壳程序，运行到 OEP 后，再在此处设置断点，可以发现这个是在调用 kernel32.GetCommandLineA 函数。打开刚才建立的 API 函数文件，找到 GetCommandLineA 函数所在的一组数据，将这组数据全部复制到 IAT 文件中，并在 GetCommandLineA 后面附带上 VA 地址 5E3220, 如下：  
代码:

```
\[kernel32.dll\]
DeleteCriticalSection
LeaveCriticalSection
EnterCriticalSection
InitializeCriticalSection
VirtualFree
VirtualAlloc
LocalFree
LocalAlloc
GetCurrentThreadId
InterlockedDecrement
InterlockedIncrement
VirtualQuery
WideCharToMultiByte
SetCurrentDirectoryA
MultiByteToWideChar
lstrlenA
lstrcpynA
LoadLibraryExA
GetThreadLocale
GetStartupInfoA
GetProcAddress
GetModuleHandleA
GetModuleFileNameA
GetLocaleInfoA
GetLastError
GetCurrentDirectoryA
GetCommandLineA 5E3220
FreeLibrary
FindFirstFileA
FindClose
CreateDirectoryA
ExitProcess
ExitThread
CreateThread
WriteFile
UnhandledExceptionFilter
SetFilePointer
SetEndOfFile
RtlUnwind
ReadFile
RaiseException
GetStdHandle
GetFileSize
GetSystemTime
GetFileType
CreateFileA
CloseHandle

```

再对目标程序进行 IAT 重建，双击运行，还是没反映。用与上面同样的方法，很容易修补下一组 DLL 数据：  
  
这次异常点：  
00407724  - FF25 AC335E00   JMP DWORD PTR DS:\[5E33AC\]  
跟踪加壳程序，可知道这是在调用：Kernel32.GetVersionExA，于是又可以修补一组 DLL 数据，如下:  
代码:

```
\[kernel32.dll\]
lstrcpyA
lstrcmpA
\_lwrite
\_lread
\_lopen
\_llseek
\_lcreat
\_lclose
WritePrivateProfileStringA
…  
GetVersionExA 5E33AC
…
CreateDirectoryA
CopyFileA
CompareStringA
CloseHandle
AreFileApisANSI

```

再对目标程序进行 IAT 重建，双击运行，还是没反映。用与上面同样的方法，很容易修补下一组 DLL 数据：  
  
这次异常点：  
0040F308  - FF25 54395E00   JMP DWORD PTR DS:\[5E3954\]  
跟踪加壳程序，可知道这是在调用：Kernel32.Sleep 函数，于是又可以修补一组 DLL 数据，在 API 字符串文件中找到该组 DLL 数据的 API 函数，如下:  
代码:

```
\[kernel32.dll\]
Sleep 5E3954

```

这组 DLL 数据中就只有一个函数，复制到 IAT 文件中去。  
  
再对目标程序进行 IAT 重建，双击运行，OK，激动的时刻已经到来，修复成功，程序窗口弹出，已经正常运行了。  
  
如果对 IATRebulid 所使用的 IAT 文件格式还不明白的，可以看看附件：目标程序的 IAT 文件，这个文件可直接修复这个程序的脱壳后的程序。  
  
心得：  
这次利用 OD 和 IATRebulid 只需对目标程序进行几次 IAT 修复，就修复了 100 多个 API 函数，操作过程也不复杂，感常还是方便的。  
  
后记：  
关于 IATRebulid 软件的更详细的操作说明及下载，大家可以看这个贴子：  
[`bbs.pediy.com/showthread.php?t=61721`](http://bbs.pediy.com/showthread.php?t=61721)  
  
希望大家喜欢这个软件，碰到什么问题，都欢迎及时反映。

上传的附件
| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || W1\_IAT.txt  
 |