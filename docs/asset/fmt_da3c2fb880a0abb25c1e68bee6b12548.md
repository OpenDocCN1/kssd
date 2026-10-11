
# 谈谈 VFP 程序的解密

> 原文：[`www.pediy.com/kssd/pediy05/pediy50514.htm`](https://www.pediy.com/kssd/pediy05/pediy50514.htm)

用了几个晚上的时间，将我破解 VFP 程序的心得用抓图的方式总结了一下，希望对爱好 VFP 程序的朋友能有所帮助，其中的错误也请大家指正。

Visual FoxPro 作为一种面向对象的数据库语言，有其简单易用的特点。虽然经常受到各种各样的批评，微软发布 6.0 版后，也曾声明版本不再更新，但现在看来，VFP 程序还是有一定的生命力。微软自 6.0 版以后开发了 7.0 版，目前 8.0beta 版在微软的官方网站已有下载。尤其在我国，早期 DOS 时代 `DBaseIII` 曾被广泛运用，VFP 出现后，源程序稍加改动就能变成可在 Window 平台运行的 VFP 版本，可以节省大量的资源，减少人员的二次培训。因此在我国 VFP 程序的用量还是较大的，专门开发此类程序的公司也不在少数。

由于 VFP 程序的编译是一种伪编译过程，经过一定的处理，可以得到完整的源代码。自从 `ReFox`、`UnFoxAll` 等反编译软件出现后，为保护作者自己的利益，软件的加密保护就显得十分重要。相应的，VFP 程序的加密也越来越严。但由于 VFP 固有的特点—即解释执行，VFP 的解释器必须要得到可识别的代码，所以 VFP 程序的破解相对其他语言编译的程序来说，还是比较简单的。

## 一、VFP EXE 程序的特点

VFP 标准的编译 EXE 文件是如下一种格式：

**可执行部分 + APP 文件 + 结构说明**

其中可执行部分是标准的 PE 文件，APP 部分就是 VFP 编译的应用程序代码，最后的结构为 14 个字节，用来描述 APP 文件，其中重要的是最后 4 个字节，反映的是 APP 文件的大小。

头部的可执行部分只是简单地加载 VFP 的解释器（解释器名称为 `vfp6r.dll`、`vfp7r.dll`、`vfp8r.dll` 分别对应 6.0、7.0、8.0 版本）执行 APP 文件。现在的加密软件大多利用这个特点，自己构造一个 PE 头，利用各种壳的技术保护 APP 代码。

因此我们可以这样认为，对于 VFP 的可执行文件，除 APP 文件以外的可执行部分，都可认为是壳。

## 二、APP 文件的结构

### 1. APP 文件结构

APP 文件是 VFP 生成的应用程序或 Active Document。

其结构为：

| 字节偏移 | 说明 |
| :--- | :--- |
| `0-1` | 文件标识 `&HF2FE` 为 VFP 的 APP |
| `2` | 加密标识 bit 0 为 1, 未加密 |
| | bit 0 为 0, 加密, 此时 bit 4(0 或 1) 区分为两类 |
| `3-4` | 编译版本 `&H0220` - VFP 6.0/7.0 |
| | `&H021F` - VFP 5.0 |
| `5-6` | 文件数目 |
| `7-8` | 主文件位置(从 0 开始) |
| `9-12` | 文件名列表地址 |
| `13-16` | 文件说明结构列表地址 |
| `17-20` | 文件名列表长度 |
| `21-36` | 加密类型密码表 |
| `37-38` | 解密 key |
| `39-40` | 文件校验和 |

**文件名列表地址**
`[目录名]`, `00`, 文件名, `00`, 文件名, `00`, ...

**文件说明结构列表地址**
每个文件占用 25 个 byte, 总长度为文件数 * 25

| 字节偏移 | 说明 |
| :--- | :--- |
| `0` | 文件类型 0 为代码 其他数据 |
| `1-4` | 文件开始地址 |
| `5-8` | 文件结束地址 |
| `9-12` | 文件名相对文件名列表的首地址 |
| 其他 | 不详 |

## 三、VFP 的加密特点及脱壳

VFP 的加密基本上可以分为两种—文件型加密和内存型加密，这两种加密的区分主要在 APP 文件的存放位置，存放在某个文件中就为文件型，存放在内存中就叫内存型。但对解密者来说，关心的是算法。以上的分类加不加以区别问题不大。

![](img/c62783a11d44a0db49501509642c7c49.jpg)

### 1. 文件头简单替换型

这种加密方式是比较早期的一种加密方式，能够阻止反编译软件，但 APP 代码在文件中以明码出现，如 xx 王 5.05B(`www.cxsw168.com`)。

**脱壳方法：**

打开 HEX，查找 `FEF2` 找到后与 APP 文件的结构对比一下，发现没有问题，把从 `FEF2` 开始到文件尾的数据存成一个文件，取名 `XXX.app`，脱壳完成。

### 2. 文件隐藏型

这类加密在 EXE 文件中已经没有 APP 的明码了，APP 文件是由主程序运行过程中动态生成的，以文件形式存在硬盘的某处。如 xx 通用计划管理系统 6.02(`www.sx168.com`)。

用 `TRW2000` 加载程序，下断 `bpx createfilea`。

`createfilea` 的定义为：

```c
HANDLE CreateFile(
  LPCTSTR lpFileName,         // pointer to name of the file
  DWORD dwDesiredAccess,      // access (read-write) mode
  DWORD dwShareMode,          // share mode
  LPSECURITY_ATTRIBUTES lpSecurityAttributes,    // pointer to security attributes
  DWORD dwCreationDisposition,  // how to create
  DWORD dwFlagsAndAttributes,  // file attributes
  HANDLE hTemplateFile         // handle to file with attributes to copy
);
```

每次中断后观察打开的文件名，经过两次 `foxpro.int` 下面一个就是脱壳后的文件名了。

![](img/45379ddbdb23c2a42d908beb9f66bd65.jpg)

文件放在 `c:\windows\history\…\…exe`。

因为 `c:\windows\history\` 是 Windows 的系统目录，下面的文件不可显示，程序运行结束后又会删除这个文件，所以必须在程序没有结束前得到这个文件。

**方法一：**
用 `suspend` 将程序挂起，用 `procdump kill` 现有进程，回 DOS 利用 `xcopy /s` 将所有文件拷出，处理。

**方法二：**
利用 `copyfilea` 函数。

```c
BOOL CopyFile(
  LPCTSTR lpExistingFileName,
                       // pointer to name of an existing file
  LPCTSTR lpNewFileName,  // pointer to filename to copy to
  BOOL bFailIfExists     // flag for operation if file exists
);
```

在上图的位置 `r eax 0` `r ebx 63e630` `r edx 63e660`。

将 `63e660` 地址的内容改成目的文件如 `d:\ab\s.exe`。

然后编一段代码：

```assembly
a eip
  push eax
  push edx
  push ebx
  call copyfilea
```

最后一句不能直接输入，要用 `exescope` 找到 `import` 中 `copyfilea` 的位置，计算相对地址用 `ff 15 xx xx xx xx` 填入。

单步运行完 `copyfilea` 后，隐藏的文件就被拷出来了。

拷出的文件是 EXE 文件，用 1 的方法就能得到 APP 文件，但这个 APP 文件可运行，用 `unfoxall` 反编译会出错，后面会讲到 APP 文件的标准化处理。

### 3. 改变解释器动态解码

这种加密方式采用了狗的方式，用 `trw2000` 加载跟踪时，在文件开始运行的代码部分会看到大量的花指令，再将主文件加密的同时，它把解释器动态连接库也进行了加壳，运行时用壳中的代码将 APP 文件解码后执行。因 `vfp6r.dll` 被改动，无法放到系统目录下，因此文件安装目录下能看到 `vfp6r.dll`，用 `exescope` 文件观察此文件，能看到 `gtide` 段，这就是这种文件的特征。

例如：xx 建筑工程预结算系统 6.6 版(`fjjlsoft.yeah.net`)。

**处理方法：**
采用 2 的方法下断 `createfilea`，得到调用文件后，下断 `readfile`。

`readfile` 定义如下：

```c
BOOL ReadFile(
  HANDLE hFile,                  // handle of file to read
  LPVOID lpBuffer,               // pointer to buffer that receives data
  DWORD nNumberOfBytesToRead,    // number of bytes to read
  LPDWORD lpNumberOfBytesRead,   // pointer to number of bytes read
  LPOVERLAPPED lpOverlapped      // pointer to structure for data
);
```

第 3 次读 200byte 时中断，在读出的代码处下断，`bpm xxxxxxxx`，观察程序如何解码。

![](img/c4f0d8af806e5989dc392994600554f3.jpg)

**解码部分的指令**

```assembly
0167:0C2FA505  INC     DWORD [EBP-04]        已处理字节数
0167:0C2FA508  MOV     EAX,[EBP+0C]          要处理字节数
0167:0C2FA50B  CMP     [EBP-04],EAX
0167:0C2FA50E  JNC     0C2FA53A
0167:0C2FA510  MOV     EAX,[EBP-04]
0167:0C2FA513  MOV     ECX,[EBP+08]          要处理数据的首地址
0167:0C2FA516  XOR     EDX,EDX
0167:0C2FA518  MOV     DL,[ECX+EAX]          要处理的数据送 DL
0167:0C2FA51B  MOV     AL,[EDX+0C3352E2]    0C3352E2—0C3354E2 512 个字节为密钥
0167:0C2FA521  MOV     [EBP-08],AL
0167:0C2FA524  XOR     EAX,EAX
0167:0C2FA526  MOV     AL,[EBP-08]
0167:0C2FA529  MOV     AL,[EAX+0C3353E2]
0167:0C2FA52F  MOV     ECX,[EBP-04]
0167:0C2FA532  MOV     EDX,[EBP+08]
0167:0C2FA535  MOV     [EDX+ECX],AL
            JMP     0C2FA505
```

`W 0C3352E2 0C3354E2 key.bin`

记下读出的内容，用 `hex workshop` 在主文件中查找，将到文件尾的内容写入另一个文件 `code.bin`。

用 VB 的语言，上述的解密算法为：

```vb
Dim keyfiledata(0 To 511) As Byte
Dim codefiledata() As Byte
Dim middata() As Byte
Dim keyfilelen As Long

Open "Key.bin" For Binary As #1
   Get #1, 1, keyfiledata()
Close #1

keyfilelen = FileLen("code.bin")

Redim keyfiledata(0 To keyfilelen-1) as byte
Redim middata(0 To keyfilelen-1) as byte

Open "code.bin" For Binary As #2
Get #2, i, codefiledata
Close #2

For I=0 To keyfilelen-1
Middata(i) = keyfiledata(keyfiledata(codefiledata (i)) + 256)
Next i

Open "code.app" For Binary As #2
   Put #2, 1, middata()
Close #2

MsgBox "finish"
```

得到 APP 文件。

### 4. 改变解释器的加密方式

这种文件的加密方式的 APP 文件是以加密 APP 的方式出现，加密者将 `vfp6r.dll` 的解密部分代码和文件头标志加以改变，变成自己的动态连接库，这个库文件同样放在文件的安装目录下，但有的程序采用了一定的手段让用户看不到这个文件。

例如：xx 工程造价软件 6.6 版(`www.ztys.com`)。

这个软件安装后，改变注册表 `HKEY_LOCAL_MACHINE\software\Microsoft\windows\currentversion\explorer\advanced\folder\hidden\showall\checkedvalue`。

将键值由 0 改为 1，这样隐含系统文件你就看不到了，我认为这种方法是不可取的，文件加密的同时不应干扰用户其他的功能，利用这种方法有点缺德之嫌。

就以这个程序为例，首先比较一下正规的 `vfp6r.dll` 和程序用的 `vfp6r.dll` 有何不同。

```assembly
0167:0C02102E  MOV     ECX,02
0167:0C021033  MOV     EDI,0C2F24C8            [edi]为 c4c5 正常为 fef2
0167:0C021038  MOV     ESI,EBX
0167:0C02103A  XOR     EDX,EDX
0167:0C02103C  REPE CMPSB
0167:0C02103E  JNZ     NEAR 0C0EF608          不等则跳,与其它值比较
0167:0C021044  MOV     AL,[EBX+02]
0167:0C021047  NOT     EAX
0167:0C021049  TEST    AL,01
0167:0C02104B  JNZ     0C021096              是加密的则跳
0167:0C02104D  MOV     ESI,01
0167:0C021052  MOV     BYTE [0C30D2FC],00
0167:0C021059  MOVZX   CX,[EBX+04]
0167:0C02105E  MOVZX   AX,[EBX+03]
0167:0C021063  SHL     ECX,08
0167:0C021066  ADD     EAX,ECX
0167:0C021068  MOV     ECX,[0C2F24C4]
0167:0C02106E  AND     EAX,FFFF
0167:0C021073  CMP     EAX,ECX               比较文件的版本
0167:0C021075  JNZ     NEAR 0C0EF5E8
0167:0C02107B  MOV     ECX,EBX
0167:0C02107D  CALL    0C0210C6
0167:0C021082  CMP     AX,[EBX+27]
0167:0C021086  JNZ     NEAR 0C0EF5FB
0167:0C02108C  MOV     EAX,ESI
0167:0C02108E  POP     EDI
0167:0C02108F  POP     ESI
0167:0C021090  POP     EBP
0167:0C021091  POP     EBX
0167:0C021092  POP     ECX
0167:0C021093  RET     0C
0167:0C021096  MOV     ECX,EBX
0167:0C021098  CALL    0C0A863D              解密部分 0
0167:0C02109D  MOV     ECX,EBX
0167:0C02109F  CALL    0C0A8727              解密部分 1
```
