
# Hying's Armor v0.7x Code Injection

- **目标**：商业类的东西不便说出名字
- **工具**：`Olydbg1.1(diy版)`、`LORDPE`、`ImportREC1.6F`
- **目的**：通过注入代码，在程序运行前显示一个提示框
- **操作平台**：Windows 2003 server
- **作者**：LOVEBOOM[DFCG][FCG][US]

## 简要说明

相信会脱这个壳的人已经有很多很多了，在你想明白怎么注入代码之前，你得有一点基础知识，对这个壳比较了解，这篇文章是讲如何注入，并非脱壳分析，如果你想知道如何脱壳，可以看我以前的文章和别的网友的文章，如果那些文章你都看懂了，而你有一定的基础，相信这篇文章对你来说是非常简单的:-)。

## 详细过程

正式开始前先讲一些：这个壳经常伪装成别的壳来欺骗检测工具的检测，骗过对壳没有了解的朋友。这次我找的目标也同样伪装了，这次伪装成\*Neolite 2.0 -> Neoworx Inc.\*的了，如果你按那个壳的方法去操作肯定是没有结果的。

这个壳的反跟踪比较多，有精彩代码值得我们学习的。用`OD`跟踪比用`SICE`跟踪更为方便，异常太多了。花指令也是做的比较有个性的:-)。

用`OD`跟踪的的话，只有一个anti-debug可以防止我们跟踪。

也许你会说，直接在壳代码处就改自己的代码不就行了吗?是的，没错。我选择在程序代码解压出来之后再出现信息框是想说，代码已经解压出来了，你可以修改程序的流程了，可以暴力破解或找修改壳的流程，去除全部的anti-debug让壳和upx一样等等，只有想不到的，没有做不到的。

要在壳运行过程中添加提示信息的话，先得找入手点，找入手点当然比须对壳比较了解，然后考虑我们的写注入的东西里要用到哪些东西，这些东西又从哪里获取，是自己添加还是直接借用壳的。Patch代码又存放到哪里，是否有足够的空间。等等。

我选择的是在壳解压出程序的代码后，壳CRC检测时用到的`UnmapViewOfFile`作为入口点，我做的方式是在壳入口处就开始hook掉该进程的`UnmapViewOfFile`，让壳执行这个程序之前先显示我想显示的对话框。Hook的过程我们要修改进程空间的代码，所以我们就得先获取相关权限，然后修改内存页，让相关空间代码能够修改，这个过程我们就得知道，我这里偷了下懒，作为完整的写法应该是修改完后，还原程序代码，然后把相关权限也写回去的，我没有写还原代码的。获取和修改权限就得用到`VirtualQuery`和`VirtualProtect`，用`VirtualQuery`获取原有的权限后应该保存一个地方呀，这个地方我选择动态申请，动态申请用取了`VirtualAlloc`和`VirtualFree`这两个API。添加信息框要用到`MessageBoxA`这个Api，这个API在`USER32.DLL`中，壳开始的时候并没有加载这个dll，所以还得用到`LoadLibraryA`，载入了DLL之后，还得取api的地址呀，所以就用到了`GetProcAddress`，用`GetProcAddress`中要用到DLL的句柄，获取句柄的话就得用到`GetModuleHandleA`。还好壳里已经有`GetModuleHandleA`这个api。总结一下用到了这么些API：

- `GETMODULEHANDLEA`
- `GETPROCADDRESS`
- `LOADLIBRARYA`
- `VIRTUALALLOC`
- `VIRTUALFREE`
- `VIRTUALQUERY`
- `VIRTUALPROTECT`
- `UnmapViewOfFile`
- `MESSAGEBOXA`

要用的API已经得到，要提示信息还得到在空地上写上想提示的信息，还得把要动态获取地址的API的名字写上去，还得把那两个DLL的名字都写上去。

OK，自己想了这么些情况，然后开始动手：

用`peid`看了一下报告成伪壳，随便在段后面的空间里写的点东西，再运行，发现程序不让运行，这说明程序有检测。用`LordPE`看看只有少数几个api函数：

```
->Import Table

   1. ImageImportDescriptor:
    OriginalFirstThunk:  0x0007E079
    TimeDateStamp:       0x00000000  (GMT: Thu Jan 01 00:00:00 1970)
    ForwarderChain:      0x00000000
    Name:                0x0007E089  ("KERNEL32.dll")
    FirstThunk:          0x0007E079

    Ordinal/Hint API name
    ------------ ---------------------------------------
    0x0000       "GetProcAddress"
    0x0000       "LoadLibraryA"
    0x0000       "GetModuleHandleA"
```

看到这些输入表信息我们就能达到目标了（如果输入表为空的话，能不能达到我的目的呢?答案是肯定的。因为壳自己也要用到API，壳用什么方法获取，我们也模拟它来完成使命。）

上面分析后代码量可能比较大，所以还得找找有没有足够的空间。所以用`lordpe`看看程序的“空地”有多少，壳是加密后，通过在原程序的基础上增加一个段来保存解壳代码，所以我们看看最后一个段有没有足够的空间。最后一个段的信息：

```
item:
    Name:
    VirtualSize:           0x00014000
    VirtualAddress:        0x0007E000
    SizeOfRawData:         0x00014000
    PointerToRawData:      0x00029A00
    PointerToRelocations:  0x00000000
    PointerToLinenumbers:  0x00000000
    NumberOfRelocations:   0x0000
    NumberOfLinenumbers:   0x0000
    Characteristics:       0xC0000040
    (INITIALIZED_DATA, READ, WRITE)
```

看了下空间不是很多，所以自己修改下，把空间改大点，添加了`&H1000`的空间，这样可以放足够的代码了。修改后：

```
item:
    Name:
    VirtualSize:           0x00015000       *************
    VirtualAddress:        0x0007E000
    SizeOfRawData:         0x00015000      *************
    PointerToRawData:      0x00029A00
    PointerToRelocations:  0x00000000
    PointerToLinenumbers:  0x00000000
    NumberOfRelocations:   0x0000
    NumberOfLinenumbers:   0x0000
    Characteristics:       0xC0000040
    (INITIALIZED_DATA, READ, WRITE)
```

修改好后用`OD`载入目标，现在来获取`LoadLibraryA`、`GetModuleHandleA`、`GetProcAddress`：

```
00491DAD > /E9 00000000     JMP 00491DB2                             ; EP,伪装别的壳的入口
00491DB2   \60              PUSHAD
00491DB3    E8 14000000     CALL 00491DCC
00491DB8    5D              POP EBP
```

载入后，下命令`g GetModuleHandleA`，断下后，通过堆栈看看`GetModuleHandleA`载入地址：

```
0012FF9C   0047E117  /CALL to GetModuleHandleA from cjbo.0047E114
0012FFA0   0047E089  \pModule = "KERNEL32.dll"
……
0047E114    FF53 08         CALL DWORD PTR DS:[EBX+8]                ; 这里调用GetModuleHandleA
0047E117    8BF8            MOV EDI,EAX                              ; 返回到这里
```

计算一下得知`GetModuleHandleA`保存在`0047E081`处。因为`IAT`地址一般是连续的，这样的话，直接`DD 47e081`看看数据窗口就知道其它两个API保存的位置了：

```
0047E079 >77E12DFB  kernel32.GetProcAddress
0047E07D >77E1850D  kernel32.LoadLibraryA
0047E081 >77E12CD1  kernel32.GetModuleHandleA
```

因为关系到重定位，所以为了防止意外我用到相对地址的方式来获取实际地址。

第一步已经完成了，第二步看看`UnmapViewOfFile`在哪里调用，看它的原因是去找CRC检测数据。重来，这次要去除`ZwSetInformationThread`这个anti-ring3’debug，否则壳会脱离Debugger，让调试器不能跟踪。去除`ZwSetInformation`时要注意不能直接改成`RET 10`之类的，壳会检测的：

```
00382CB2    8A06            MOV AL,BYTE PTR DS:[ESI]
00382CB4    3C B8           CMP AL,0B8                               ; 开始检测了，判断是否为mov eax,xxxxxx的方式，如果不是则跳
00382CB6    75 04           JNZ SHORT 00382CBC
00382CB8    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00382CB9    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
00382CBA    EB 33           JMP SHORT 00382CEF
00382CBC    3C 8D           CMP AL,8D
00382CBE    75 03           JNZ SHORT 00382CC3                       ; 判断是否为LEA的方式
00382CC0    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
00382CC1    EB 2C           JMP SHORT 00382CEF
00382CC3    3C CD           CMP AL,0CD                               ; 判断是否为Int xx的入口
00382CC5    75 04           JNZ SHORT 00382CCB
00382CC7    66:A5           MOVS WORD PTR ES:[EDI],WORD PTR DS:[ESI]
00382CC9    EB 24           JMP SHORT 00382CEF
00382CCB    3C BA           CMP AL,0BA                               ; 判断是否为mov edx,address的方式
00382CCD    75 04           JNZ SHORT 00382CD3
00382CCF    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00382CD0    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
00382CD1    EB 1C           JMP SHORT 00382CEF
00382CD3    3C FF           CMP AL,0FF                               ; 判断是否为push [address]的方式
00382CD5    75 04           JNZ SHORT 00382CDB
00382CD7    66:A5           MOVS WORD PTR ES:[EDI],WORD PTR DS:[ESI]
00382CD9    EB 14           JMP SHORT 00382CEF
00382CDB    3C C2           CMP AL,0C2                               ; 判断是否为RET val的方式
00382CDD    75 0A           JNZ SHORT 00382CE9
00382CDF    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00382CE0    66:A5           MOVS WORD PTR ES:[EDI],WORD PTR DS:[ESI]
00382CE2    61              POPAD
00382CE3    33C0            XOR EAX,EAX
00382CE5    40              INC EAX
00382CE6    C2 0C00         RETN 0C
00382CE9    61              POPAD
00382CEA    33C0            XOR EAX,EAX
00382CEC    C2 0C00         RETN 0C
```

当然如果你的目标不是新版本的不会检测的。

去除这个anti-Debug后，`follow UnmapViewOfFile`，在`RET 4`处下断。运行中断后，取消断点，返回壳代码：

```
0037DAB9    FFB5 64174100   PUSH DWORD PTR SS:[EBP+411764]           ; 返回到这里
0037DABF    7C 05           JL SHORT 0037DAC6
0037DAC1    EB 05           JMP SHORT 0037DAC8
0037DAC3    0FCD            BSWAP EBP
0037DAC5    2074F9 8D       AND BYTE PTR DS:[ECX+EDI*8-73],DH
0037DAC9    859D C34000EB   TEST DWORD PTR SS:[EBP+EB0040C3],EBX
0037DACF    020F            ADD CL,BYTE PTR DS:[EDI]
0037DAD1    0F50E8          MOVMSKPS EBP,XMM0
0037DAD4    04 00           ADD AL,0
0037DAD6    0000            ADD BYTE PTR DS:[EAX],AL
0037DAD8    CD 20           INT 20
0037DADA  - E9 6883C404     JMP 04FC5E47
0037DADF    8B85 A2184100   MOV EAX,DWORD PTR SS:[EBP+4118A2]        ; CloseHandle
0037DAE5    EB 04           JMP SHORT 0037DAEB
```

没跟多久看会看到，CRC的比较了。

```
0037DC7E    3BF7            CMP ESI,EDI                              ; 到这里进行CRC检测
0037DC80    0F85 F65E0000   JNZ 00383B7C                             ; 不等则over
```

其中`ESI`是计算后的结果，`EDI`就是正确的CRC值，这里就有三种去除方式了，第一种在把crc值保存在文件中，第二种`mov esi,edi`的方式，第三种直接改成`nop`。

用`LordPE`看一下正确的CRC值正是保存在`PE header-4`处。

```
004000F0  00 00 00 00 00 00 00 00 00 00 00 00 61 95 C1 32  ............a?2
00400100  50 45 00 00 4C 01 03 00 19 5E 42 2A 00 00 00 00  PE..L  . ^B\*....
```

原程序没修改时的正确CRC值为`32c19561`，这个值是换算后的。换算方法是：

```
KEY==CRC xor Key(048F7032)
key==KEY ROR 3
```

如果你现在直接在`od`里是看不到程序解码的，因为我并没有处理`DRx`部分。这篇文章是讲代码注入的，所以我不讲那些东西。

现在我们的CRC值的信息已经找到，也知道是哪里用`UnmapViewOfFile`这个函数，现在只需在壳入口处写上自己的代码，然后在上面比较CRC值是否正确的地方，
