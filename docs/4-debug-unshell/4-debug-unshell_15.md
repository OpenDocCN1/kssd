
# 加壳软件简介

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-835.htm`](https://www.pediy.com/kssd/pediy08/pediy8-835.htm)

为了保护自己的软件不轻易被他人“借鉴”，有必要对软件进行一些加密保护，而这方面目前己有成熟的专业加密软件可选择。但不要太依赖壳的保护，大多数壳是可以被攻破的，还是在自身保护上下些功夫。加密软件比较多，但在强度与兼容性上做的好的并不多，这里向大家介绍几款常见的。目前流行的一些壳可以参考这里：[`www.pediy.com/tools/packers.htm`](http://www.pediy.com/tools/packers.htm)

现在壳的发展一个趋势就是虚拟机保护，利用虚拟机保护后，能大大提高强度，因此建议尽可能使用此类技术保护软件。如 `Themida`、`WinLicense`、`EXECryptor` 等壳带有虚拟机保护功能，因此得用好其 `SDK`。

### 1. `ASProtect` 加密壳

`ASProtect` 是一款应用面最广的加密壳，其兼容性和稳定性很好，许多商业软件采用这款壳加密。开发者是俄国人 `Alexey Solodovnikov`，官方站点 `www.aspack.com`。

![](img/45167482c91309630172e5cb73608d29.jpg)

`ASProtect SKE` 系列己采用了部分虚拟机技术，主要是在 `Protect Original EntryPoint` 与 `SDK` 上。保护过程中建议大量里使用 `SDK`，`SDK` 使用请参考其帮助文档，在使用时注意 `SDK` 不要嵌套，并且同一组标签用在同一个子程序段里。`ASProtect` 使用相当的简单，打开被保护的 `EXE/DLL` 文件后，选上保护的选项。再单击菜单 `Modes`，单击 `Add Mode` 按钮，将 `Is this Mode Avtive` 选上，最后，单击 `Protection` 标签，对软件进行保护即可。`ASProtect` 加壳过程中也可外挂用户自己写的 `DLL` 文件，方法是在上图中的 `External Options` 选项加上目标 `DLL` 即可。这样，用户可以在 `DLL` 加入自己的反跟踪代码，以提高软件的反跟踪能力。

**强度评介**：由于 `ASProtect` 名气太大，研究它的人很多，因此很容易被脱壳，不推荐使用。

### 2. `Armadillo` 加密壳

`Armadillo` 也称穿山甲，是一款应用面较广的壳。可以运用各种手段来保护你的软件，同时也可以为软件加上种种限制，包括时间、次数，启动画面等等！很多商用软件采用其加壳。`Armadillo` 对外发行时有 `Public`，`Custom` 两个版本。`Public` 是公开演示的版本，`Custom` 是注册用户拿到的版本。只有 `Custom` 才有完整的功能，`Public` 版有功能限制，没什么强度，不建议采用。

![](img/4115655570834a4d030ab7c86f4e256c.jpg)

**强度评介**：`Armadillo` 中比较强大的保护选项是 `Nanomites` 保护（即 `CC` 保护），用的好能提高强度，其他选项没什么强度。

### 3. `EXECryptor` 加密壳

`EXECryptor` 也是一款猛壳，可能由于兼容性等原因，采用其保护的商业软件不是太多。这款壳的特点是 `Anti-Debug` 做的比较隐蔽，另外就是采用了虚拟机保护一些关键代码。

**强度评介**：用好 `EXECryptor` 虚拟机保护功能，将关键敏感代码用虚拟机保护起来，能提高强度。`EXECryptor` 壳能脱的人很多，但对付其虚拟机代码的人不多。

![](img/8f4a02dae3bf04efed40997a9b22e473.jpg)

### 4. `Themida` 加密壳

`Themida` 是 `Oreans` 的一款商业壳，官方链接：[www.oreans.com](http://www.oreans.com/)。`Themida 1.1` 以前版本带驱动，稳定性有些影响。`Themida` 最大特点就是其虚拟机保护技术，因此在程序中擅用 `SDK`，将关键的代码让 `Themida` 用虚拟机保护起来。`Themida` 最大的缺点就是生成的软件有些大。`WinLicense` 这款壳和 `Themida` 是同一公司的一个系列产品，`WinLicense` 主要多了一个协议，可以设定使用时间，运行次数等功能，两者核心保护是一样的。

**强度评介**：用好其虚拟机保护功能，将关键敏感代码用虚拟机保护起来，能提高强度。

![](img/11295aa45ea18f18b885ab0f14113d9a.jpg)

### 5. `VMProtect`

`VMProtect` 是一款纯虚拟机保护软件，官方链接：`www.VMProtect.ru`。它是当前最强的虚拟机保护软件，经 `VMProtect` 处理过的代码，至今还没有人公开宣称能还原。

但也有缺点，就是会影响程序速度，因此在一些对速度要求很高的场合就不适合用了。`VMProtect 1.22.3` 之前是免费版，可以支持 `EXE`，`DLL` 等文件。更高版本需要购买，其支持驱动的保护。现在流行的做法，先用 `VMProtect` 将你的核心代码处理一下，再选用一款兼容性好的壳保护。

**1. 关键代码自己定位**

`VMProtect` 并没有提供使用说明，必须告诉 `VMProtect` 你要加密的代码具体地址，这对使用者有一定的要求，至少要懂一些跟踪技术，可以用调试器，如 `OllyDbg` 跟踪到程序需要保护的地址，然后添加地址到 `VMProtect`。

在这以一个记事本程序为例来演示一下使用方法。

运行 `VMProtect` 后，打开 `NOTEPAD.EXE` 文件。单击 `Dump` 标签，输入要加密的起始地址，光标来到要加密代码起始地址后，点击菜单“project/new procedure”,会出现一个新的项目，如下图。

![](img/7a6c526da276780aaae1def218d65522.jpg)

需要处理其他地址时，请依次操作。

注意事项：

1. 用 `VMProtect` 处理，请多测试，如果不稳定，请调整被保护代码的范围。
2. `VMProtect` 对双线程支持不是太好，请同一次仅处理一个线程内的代码

**2. 用 SDK 标记代码**

`VMProtect v1.2` 以上支持 `SDK` 了，可以编程时插入一个标记，然后在加密时，`VMProtect` 会认出这些标记，并在有标记的地方进行保护。在程序源码中，用这对标签将一些核心代码包含，编译成 `EXE` 文件。然后用 `VMProtect` 打开 `EXE`，单击“Project”菜单下的“New procedure”或者单击工具栏中的“New procedure”按钮，在弹出的添加地址窗口中会自动将 `SDK` 定义代码的地址填上。然后在 `VMProtect` 的“Options”窗口中设置相应的选项，最后单击工具栏中的“Compilation (F9)”按钮，便可对目标软件进行保护。经 `VMProtect` 处理过的软件，可以继续用 `Asprotect`、`Themida` 等加壳软件进一步保护。

`VMProtect` 是当前最强的虚拟机保护软件，经过 `VMProtect` 处理的软件基本是没法分析原程序思路的，关键是用好，一定要将程序关键代码进行处理。另外，经虚拟机处理代码效率会降低，因此一些对效率要求比较高的代码就不要用 `VMProtect` 进行处理。

`Delphi` 中的标记模式

引用:

```
asm 
db $EB,$10,'VMProtect begin',0      //标记开始处.
end;
//想保护的程序代码
asm 
db $EB,$0E,'VMProtect end',0        //标记结束处.
end;
```

`VC` 的 `VMProtect` 的宏：

引用:

```
#define  VMBEGIN 
__asm   //标记开始处.
{   
  _emit 0xEB
  _emit 0x10   
  _emit 0x56   
  _emit 0x4D   
  _emit 0x50 
  _emit 0x72 
  _emit 0x6F 
  _emit 0x74 
  _emit 0x65 
  _emit 0x63 
  _emit 0x74 
  _emit 0x20 
  _emit 0x62 
  _emit 0x65 
  _emit 0x67 
  _emit 0x69 
  _emit 0x6E 
  _emit 0x00
}
//想保护的程序代码
#define  VMEND
__asm  //标记结束处.
{   
  _emit 0xEB
  _emit 0x0E   
  _emit 0x56    
  _emit 0x4D    
  _emit 0x50 
  _emit 0x72 
  _emit 0x6F 
  _emit 0x74 
  _emit 0x65 
  _emit 0x63 
  _emit 0x74 
  _emit 0x20 
  _emit 0x65 
  _emit 0x6E 
  _emit 0x64 
  _emit 0x00
}
```

### 6. 小结

充分利用现成加密软件，程序员可以将更多的精力放到程序设计编写中去。为了提高强度，在使用加壳软件过程中尽可能多利用 `SDK`。

4dd466d0e1e0cc123afbe504da2462ca
