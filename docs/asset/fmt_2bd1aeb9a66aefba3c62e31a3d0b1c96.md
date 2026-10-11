
# 用 Ollydbg 手脱 Armadillo 加壳的 DLL——Visual.Assist.X.V10.2.1437.0

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-772.htm`](https://www.pediy.com/kssd/pediy07/pediy7-772.htm)

## 基本信息

- **下载地址**：`http://www.wholetomato.com/downloads/VA_X_Setup1437.exe`
- **软件大小**：`3318 KB`
- **软件语言**：英文
- **软件类别**：国外软件 / 共享版 / 其它控件
- **应用平台**：`Win9x/NT/2000/XP`
- **加入时间**：`2005-12-10`
- **下载次数**：`21401`
- **开发商**：`http://www.wholetomato.com/index.html`

**软件简介**：`Visual.Assist.X` 是一款非常好的 `Visual Studio .NET 2003`、`2002` 插件，支持 `C/C++`、`C#`、`ASP`、`Visual Basic`、`Java` 和 `HTML` 等语言，也支持 `VC++6`、`VC++5`，能自动识别各种关键字、系统函数、成员变量、自动给出输入提示、自动更正大小写错误、自动标示错误等，有助于提高开发过程地自动化和开发效率。

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`WinXP`、`OllyDBG`、`PEiD`、`LordPE`、`ImportREC`、`DT_FixRes`、`PeMove`、`WinHex`

## 脱壳过程

有兄弟要求再写篇 `Armadillo` 加壳 `DLL` 的脱壳教程，所以用 `VAX` 作个例子了。

用 `VC` 的人很多喜欢这个软件，其核心 `VA_X.dll` 加了 `Armadillo` 壳。

### 一、OllyDBG 设置

用修改版 `OllyDBG`，原版会被检测到。

![](img/de7e3147de3dc76beecd27de20e61510.jpg)

如何避开这些 Anti 将在 `Armadillo V4.0-V4.4.DLL.osc` 脚本里面实现。

首先要设置 `OllyDBG` 忽略除了“内存访问异常”和“异常范围”之外的其他异常选项

![](img/b7204f60fe0314b9f4f1303e8bcd35d7.jpg)

否则直接运行下面不好演示清楚

现在我们暂停在第一个内存异常处：

```
00999781      8900                mov dword ptr ds:[eax],eax
```

可以设置 `OllyDBG` 或略所有异常选项了。

老规矩：用 `IsDebug` 插件去掉 `OllyDBG` 的调试器标志。

### 二、Magic Jump

下断：`HE GetModuleHandleA`

如果硬件断点无法中断，可以 `Ctrl+G`：`GetModuleHandleA`，在函数末尾设断

`Shift+F9`，注意看堆栈

```
000698E4   00990D97  /CALL 到 GetModuleHandleA 来自 00990D91
000698E8   009A4D68  \pModule = "kernel32.dll"
000698EC   009A5F58  ASCII "VirtualAlloc"

000698E4   00990DB4  /CALL 到 GetModuleHandleA 来自 00990DAE
000698E8   009A4D68  \pModule = "kernel32.dll"
000698EC   009A5F4C  ASCII "VirtualFree"

0006965C   00979A2D  /CALL 到 GetModuleHandleA 来自 00979A27
00069660   00069798  \pModule = "kernel32.dll"
```

当堆栈如上显示变化是，就可以删除 `GetModuleHandleA` 处硬件断点了，`Alt+F9` 返回

```
00979A27      FF15 CCF09900      call dword ptr ds:[99F0CC]      ; kernel32.GetModuleHandleA
00979A2D      8B0D 40A19A00     mov ecx,dword ptr ds:[9AA140]

//返回这里

00979A33      89040E            mov dword ptr ds:[esi+ecx],eax
00979A36      A1 40A19A00       mov eax,dword ptr ds:[9AA140]
00979A3B      393C06            cmp dword ptr ds:[esi+eax],edi
00979A3E      75 16             jnz short 00979A56
00979A40      8D85 B4FEFFFF     lea eax,dword ptr ss:[ebp-14C]
00979A46      50                push eax
00979A47      FF15 D4F09900     call dword ptr ds:[99F0D4]      ; kernel32.LoadLibraryA
00979A4D      8B0D 40A19A00     mov ecx,dword ptr ds:[9AA140]
00979A53      89040E            mov dword ptr ds:[esi+ecx],eax
00979A56      A1 40A19A00       mov eax,dword ptr ds:[9AA140]
00979A5B      393C06            cmp dword ptr ds:[esi+eax],edi
00979A5E      0F84 B0000000     je 00979B14

//Magic Jump！ 修改为：jmp 00979B14  ★

00979A64      33C9              xor ecx,ecx
00979A66      8B03              mov eax,dword ptr ds:[ebx]
00979A68      3938              cmp dword ptr ds:[eax],edi
00979A6A      74 06             je short 00979A72
00979A6C      41                inc ecx
00979A6D      83C0 0C           add eax,0C
00979A70      EB F6             jmp short 00979A68
```

### 三、OEP

下断：`BP _set_new_handler`

`Shift+F9`，中断后取消断点，`Alt+F9` 返回

```
1EFBCD3C      FF15 4061FE1E     call dword ptr ds:[1EFE6140] ; msvcrt._set_new_handler
1EFBCD42      83C4 04           add esp,4

//返回这里

1EFBCD45      837D FC 01        cmp dword ptr ss:[ebp-4],1
1EFBCD49      75 0E             jnz short 1EFBCD59
1EFBCD4B      68 A896FE1E       push 1EFE96A8
1EFBCD50      FF15 CC96FE1E     call dword ptr ds:[1EFE96CC]
1EFBCD56      83C4 04           add esp,4
1EFBCD59      8B45 FC           mov eax,dword ptr ss:[ebp-4]
1EFBCD5C      8BE5              mov esp,ebp
1EFBCD5E      5D                pop ebp
1EFBCD5F      C3                retn

//返回 1EFBCDCD

1EFBCDCD      8945 FC           mov dword ptr ss:[ebp-4],eax
1EFBCDD0      837D FC 01        cmp dword ptr ss:[ebp-4],1
1EFBCDD4      75 40             jnz short 1EFBCE16
1EFBCDD6      833D C496FE1E 00  cmp dword ptr ds:[1EFE96C4],0
1EFBCDDD      74 30             je short 1EFBCE0F
1EFBCDDF      68 E898FE1E       push 1EFE98E8
1EFBCDE4      6A 01             push 1
1EFBCDE6      8B0D 9096FE1E     mov ecx,dword ptr ds:[1EFE9690]
1EFBCDEC      51                push ecx
1EFBCDED      FF15 C496FE1E     call dword ptr ds:[1EFE96C4] ; VA_X.1EE42A48

//飞向光明之巅

1EE42A48      6A 0C             push 0C

//OEP

1EE42A4A      68 3867EE1E       push 1EEE6738
1EE42A4F      E8 D4130000       call 1EE43E28
1EE42A54      33C0              xor eax,eax
1EE42A56      40                inc eax
1EE42A57      8945 E4           mov dword ptr ss:[ebp-1C],eax
1EE42A5A      8B75 0C           mov esi,dword ptr ss:[ebp+C]
1EE42A5D      33FF              xor edi,edi
1EE42A5F      3BF7              cmp esi,edi
1EE42A61      75 0C             jnz short 1EE42A6F
```

### 四、Code Splicing

某些兄弟对如何知道程序有 `Code Splicing` 感到困惑，下面提供 2 种方法吧。

1. 一切按正常流程操作，修复后运行程序若有 `Code Splicing` 肯定会崩溃，而崩溃的地方就能看见 `Code Splicing` 地址了。
2. 根据 `Code Splicing` 段的特征来查找

目前版本 `Armadillo` 的 `Code Splicing` 区段长度为 `00020000`，可以根据此特征来判断。

`Alt+M`，察看目标程序的区段上下，可以看到 `02B70000` 段的长度为 `00020000`

| 地址 | 大小 | 物主 | 区段 | 包含 |
| :--- | :--- | :--- | :--- | :--- |
| `02B70000` | `00020000` | | | |
| `1ED00000` | `00001000` | `VA_X` | | |
| `1ED01000` | `0019D000` | `VA_X` | `.text` | |
| `1EE9E000` | `00018000` | `VA_X` | `CODE` | |
| `1EEB6000` | `00059000` | `VA_X` | `.rdata` | |
| `1EF0F000` | `00023000` | `VA_X` | `.data` | |
| `1EF32000` | `00001000` | `VA_X` | `.SHARED` | |
| `1EF33000` | `00001000` | `VA_X` | `DATA` | |
| `1EF34000` | `00042000` | `VA_X` | `BSS` | |
| `1EF76000` | `00030000` | `VA_X` | `.reloc` | |
| `1EFA6000` | `00030000` | `VA_X` | `.text1` | |
| `1EFD6000` | `00010000` | `VA_X` | `.adata` | `code` |
| `1EFE6000` | `00010000` | `VA_X` | `.data1` | `imports` |
| `1EFF6000` | `00010000` | `VA_X` | `.reloc1` | `relocations` |
| `1F136000` | `00041000` | `VA_X` | `.rsrc` | `resources` |
| `5D170000` | `00001000` | `COMCTL32` | | `PE header` |

`Alt+C`，返回代码窗口，`Ctrl+G`: `02B70000`

```
02B70000      66:87F3           xchg bx,si
02B70003      66:96             xchg ax,si
02B70005      7B 02             jpo short 02B70009
02B70007      7B 09             jpo short 02B70012
02B70009      87F9              xchg ecx,edi
02B7000B      90                nop
02B7000C      87F9              xchg ecx,edi
02B7000E      66:96             xchg ax,si
02B70010      66:87F3           xchg bx,si
02B70013      55                push ebp
02B70014      8BEC              mov ebp,esp
02B70016      6A FF             push -1
02B70018      E9 E80F191C       jmp 1ED01005

//注意，跳到目标程序空间

1ED01000      E9 FBEFE6E3       jmp 02B70000

//Code Splicing

1ED01005      68 606BEB1E       push 1EEB6B60
1ED0100A      68 78D3E31E       push 1EE3D378
1ED0100F      64:A1 00000000    mov eax,dword ptr fs:[0]
1ED01015      E9 03F0E6E3       jmp 02B7001D
```

好了，这样就找到了 `Code Splicing` 区段，用 `ArmInline` 来修复吧，注意各参数的填写

![](img/652129c2064cbdf666cc3c9a0aac338a.jpg)

可以用 `LordPE` 把此 `DLL` 抓取下来了。

### 五、输入表修复

`VA_X.dll` 没有使用 `Import Table Elimination`，输入表函数都是有序排列的，可以正常获取。

运行 `ImportREC`，由于此 `DLL` 加载后没有进行重定位，所以保留“`Use PE Header From Disk`”选项

填入 `OEP RVA=00142A48`，获取输入表后 `Cut` 掉函数中填充的无效垃圾指针。

![](img/b24f800718ad12f4ae2216b5492cb17a.jpg)

可以新增区段来修复输入表，但是这样会增加大小，所以还是放在近似原来的位置吧

用 `WinHex` 打开 `dump.exe`，搜索输入表中函数的 `DLL` 名，`Armadillo` 没有清掉输入表的函数 `DLL` 名，当找到很多 `00` 中的输入表中函数 `DLL` 名处就可以放输入表了。当然，`Size` 要算好，一般是够用的。

```
0020CF20   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
```
