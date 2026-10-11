
# EasyTable

> 原文：[`www.pediy.com/kssd/pediy03/forum710.htm`](https://www.pediy.com/kssd/pediy03/forum710.htm)

Delphi 控件 `EasyTable` 的去除 `NagScreen`

### 控件介绍

`BDE` 作为数据库引擎似乎太笨重了一些，所以现在出了很多替代它的工具，`EasyTable` 是其中的一个。

它有自己的特点，比如数据库可以压缩，可以数据加密，当然，功能还是不够完善，但是对于小程

序来说已经足够用了。

下载地址：

`http://www.aidaim.com/cgi-bin/download.cgi?url=http://www.aidaim.com/download/tetfred5.zip`

### 使用工具

`Win32Dasm`

`DCUExplorer`

`DeDe`

`HEdit`

### 过程介绍

首先安装好这个控件，然后打开自带的 `Demo` 进行编译。在 `Delphi` 运行时候启动 `Demo`，没有 `Nagscreen`。

关闭 `Delphi`，运行 `Demo`，`NagScreen` 出来了。

`Win32Dasm` 上场，反编译后查找 `NagScreen` 里面显示的字串：`This is....`，没有！`Easy Table Free`，

还是没有！这下有些麻烦了。使用 `DeDe` 反编译，发现有 `TAboutBox` 项，好吧，就从这里入手。

先在控件安装目录下查找字串：`"Easy Table Free"` 和 `"This is blah blah blah"`，在两个 `DCU` 文件

中找到：`AboutBox.dcu`、`EasyTable.dcu`。显然，对于程序而言，`EasyTable` 里面会调用 `AboutBox`，所以注意

力集中在 `EasyTable` 里面。使用 `DCUExplorer` 对 `EasyTable.dcu` 进行反汇编，查找第一个 `AboutScreen`，所见

如下：

```
{ 19F: u?    |75 BA              | }
    JNE  @@015B; (0x15B) { OpCode : 03 }
{ 1A1: €}?  |80 7D F7 00        | }
@@01A1 : CMP  BYTE PTR [EBP-9],$00 { OpCode : 38 }
{ 1A5: u~    |75 7E              | }
    JNE  @@0225; (0x225) { OpCode : 03 }
{ 1A7: ?.... |8B 0D(00 00 00 00  | }
    MOV  ECX,DWORD PTR [AboutScreen {0x4}] { OpCode : 87 }
{ 1AD: ?...  |A1(00 00 00 00      | }
    MOV  EAX,DWORD PTR [Application {0x62}] { OpCode : 87 }
```

有些过分乱了。虽然我看不太明白，不过可以试试看，假设 `AboutScreen` 是那个 `Nagscreen`，那么看看编译后

的 `Demo` 第一次是如何调用 `AboutScreen` 的呢？好了，使用 `Hex` 编辑器查找 `Demo` 主程序里的

`75 BA 80 7D F7 00 75 7E 8B 0D`，只能找到一处，看来不错。

然后记下此处的 `offset`，到 `Win32Dasm` 反编译出来的文件中去找，会看见如下代码：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049D2C7(C)
|
:0049D2CF 46                   inc esi
:0049D2D0 FF4DF0               dec [ebp-10]
:0049D2D3 75BA                 jne 0049D28F <---------------
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0049D287(C), :0049D2CD(U)
|
:0049D2D5 807DF700             cmp byte ptr [ebp-09], 00 <-----------------
:0049D2D9 757E                 jne 0049D359 <---------------
:0049D2DB 8B0DC0224B00        mov ecx, dword ptr [004B22C0]
:0049D2E1 A1B0214B00          mov eax, dword ptr [004B21B0]
:0049D2E6 8B00                 mov eax, dword ptr [eax]
```

往上看，代码如下：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0049D2D3(C)
|
:0049D28F 8D4DEC               lea ecx, dword ptr [ebp-14]
:0049D292 8BD6                 mov edx, esi
:0049D294 8B45F8               mov eax, dword ptr [ebp-08]
:0049D297 8B38                 mov edi, dword ptr [eax]
:0049D299 FF570C               call [edi+0C]
:0049D29C 8B55EC               mov edx, dword ptr [ebp-14]
* Possible StringData Ref from Code Obj ->"Delphi"
                               |
:0049D29F B8ECD34900           mov eax, 0049D3EC
:0049D2A4 E8A36EF6FF           call 0040414C
:0049D2A9 48                   dec eax <---------------- @@
:0049D2AA 741D                 je 0049D2C9 <---------------- @@
:0049D2AC 8D4DE8               lea ecx, dword ptr [ebp-18] <-- @@
:0049D2AF 8BD6                 mov edx, esi
:0049D2B1 8B45F8               mov eax, dword ptr [ebp-08]
:0049D2B4 8B38                 mov edi, dword ptr [eax]
:0049D2B6 FF570C               call [edi+0C]
:0049D2B9 8B55E8               mov edx, dword ptr [ebp-18]
* Possible StringData Ref from Code Obj ->"C++Builder"
                               |
:0049D2BC B8FCD34900           mov eax, 0049D3FC
:0049D2C1 E8866EF6FF           call 0040414C
:0049D2C6 48                   dec eax
:0049D2C7 7506                 jne 0049D2CF
```

到了这里就有些明白了，它似乎在寻找 `Delphi` 或者 `C++Builder` 是否已经启动，没有的话就跳

对话框。在 `Dasm` 里面 `load Demo` 程序，断点设在 `0049D2AA` 处，发现此处不跳转。

试着在 `0049D2AA` 处改为 `jmp 0049D2C9`，再运行程序，没有 `NagScreen` 了。

最后一步，将 `EasyTable.dcu` 和 `tet_pack.bpl` (安装控件的时候自己将新建包命名为 `tet_pack.dpk`)

用 `Hex` 编辑器打开，找@@处的机器码，然后将 `741D` 改为 `EB1D`。重新运行 `Delphi`，编译 `Demo` 程序，

关闭 `Delphi`，运行 `Demo` 程序，没有了 `NagScreen`。为了保险起见，由于我的硬盘是两个 `C` 盘，在

另外一个干净的 `Win98` 中试用，结果还是没有 `NagScreen` 出现，看来可以收工了。
