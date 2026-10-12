
# 从 CHM 的编译说起

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-160.htm`](https://www.pediy.com/kssd/pediy08/pediy8-160.htm)

从 CHM 的编译说起

by `nbw[NE365]`

注：本文纯属虚构，如有侵权，纯属巧合

51 回家没事。想看一下 CHM 电子书是怎么做的。国内好多做电子书的小软件。比如耶书制作、`EasyCHM` 等等。原理都是一样的。

这里我们拿 `EasyCHM` 看看怎么生成 CHM 文件的。

事先声明一下，市面上的 CHM 编译和反编译都做得很龌龊。如果你想看具体的算法之类，以后再研究吧。

`EasyCHM` 制作 CHM 文件的过程：

1. 首先生成 `hhc.exe`；
2. 再生成三个配置文件：
   - `FASM.HHP`
   - `INDEX.HHK`
   - `TOC.HHC`
3. 然后调用 `hhc.exe`：
   ```
   CommandLine = ""C:\Program Files\EasyCHM\hhc.exe" "F:\DisCHM\FASM\FASM.HHP""
   0051E443     call EasyCHM.00407308            ; jmp to KERNEL32.CreateProcessA
   ```

好了。电子书就制作好了。

命令行运行一下 `hhc.exe`，显示如下信息：

```
Usage:   hhc <filename>
    where <filename> = an HTML Help project file
Example: hhc myfile.hhp
```

按照提示正常编译一个 CHM 工程，输入上面的 `CommandLine`，可以编译出来一个正常的 CHM 文件，同时 `hhc` 会有以下编译提示：

```
Microsoft HTML Help Compiler 4.74.8702
Compiling f:\DisCHM\FASM\FASM.CHM
00.htm
01.htm
02.htm
03.htm
TOC.HHC
INDEX.HHK
C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\EasyCHM.html
Compile time: 0 minutes, 1 second
11     Topics
16     Local links
13     Internet links
0      Graphics
Created f:\DisCHM\FASM\FASM.CHM, 47,420 bytes
Compression decreased file by 261,481 bytes.
```

到了这里也就很明显了，他用的是 MS 出的这个工具来生成 CHM 文件。

`hhc.exe` 调用了 `HHA.dll` 文件的导出函数 `#319` 来进行编译，如下：

```
004011A0   |> \57         push edi                ;para4
004011A1   |.  68 0D104000 push 0040100D         ;para3
004011A6   |.  68 00104000 push 00401000         ;para2
004011AB   |.  FF33      push dword ptr ds:[ebx] ;para1
004011AD   |>   E8 22720000 call <jmp.&HHA.#319>
```

看一下堆栈变化，该函数共 4 个参数。

`para1`：
ASCII `"Compiling f:\DisCHM\FASM\FASM.CHM"`

`para4` 为 `0`，也不具体深究了。其中 `para2` 和 `para3` 是 `2` 个函数指针，指向的是 `2` 个外部函数，具体如下：

```
//该函数用来输出编译信息
00401000       FF7424 04      push dword ptr ss:[esp+4]    //这里是需要输出的信息，比如 ASCII "Microsoft HTML Help Compiler 4.74.8702"
00401004       E8 17020000    call 复件_hhc.00401220
00401009       59           pop ecx
0040100A       C2 0400        retn 4

//该函数很简单，估计这里 eax = 1 是一个编译选项。
0040100D       6A 01          push 1
0040100F       58           pop eax
00401010       C2 0400        retn 4
```

这 `2` 个参数很特殊。

由于编译时候需要实时把编译信息输出，所以由外部调用函数 `hhc.exe` 提供一个信息输出函数供 `HHA.dll` 可以及时把需要输出的编译信息显示出来。也就是 `para2`。

但 `para3` 仅仅为了提供一个参数，也采用此方法就有些不可思议了。

反编译了其他调用这个函数的地方，也没发现有别的意义。与其说作者有其他考虑，还不如说就是编写时候的败笔。因为程序中还残留不少作者用于测试时候的代码：

```
00401080   |.  68 C8B04000    push 复件_hhc.0040B0C8                          ;  ASCII "Cannot open hhctest.hhp"
00401085   |.  E8 96010000    call 复件_hhc.00401220
```

`HHA.#319` 生成最后的 CHM 文件，需要提一下的是这里调用了系统自带的 `itss.dll` 来完成最后的工作。

回头看一下 MS 的这个编译 CHM 文件的过程，总体还是比较混乱的。

导出函数命名混乱，如 `#319` 等。模块化也不强，来回调用了数个 `dll`，也感觉不到有什么太大意义。传递外部函数给第三方使用，貌似不符合如今的软件工程。有不少残余代码发布时候未曾去掉。

但回头考虑一下这些 `xx` 年前采用 `VC 4.x` 编写的代码，令人感受到的不是作者水平以及 MS 管理得如何，而是前辈们创业发展过程中的辛酸。

有时间说一下反编译。相对来说反编译要比这个要麻烦些了。

看一下这个软件的介绍：

- 主要功能：
  - 全自动的目录及文件导入（可以包括子目录）；
  - 支持导入任意的文件类型；
  - 自动生成 CHM 的目录列表并自动生成所有目录项；
  - 为 CHM 的目录列表自动添加多级编号；
  - 批量更换 CHM 目录各项的图标；
  - 支持批量查找替换多级目录各项的标题文字内容；
  - 允许用户指定从文本文件的第 N 行自动截取标题；
  - 易用的目录编辑器；
  - 丰富实用的 CHM 制作选项帮助用户制作更加个性化的专业 CHM 电子书或 CHM 帮助文件。
  - 内嵌 CHM 反编译工具。
  - 轻松制作上下文相关的帮助文件

可以说最前面 `xx` 条功能主要都是 MS 的 `hhc.exe` 做的，作者花很大功夫做的“CHM 反编译工具”被低调放在了最后，为什么这么不尊重自己的劳动呢？因为大家要挣钱吃饭，要养家糊口，就要把用户最需要的功能放在前面。

`4dd466d0e1e0cc123afbe504da2462ca`
