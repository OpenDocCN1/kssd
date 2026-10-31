
# 翻译(1) (6千字)

> 膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊?

>     膊    \_\_\_\_

>       \_\_      \_\_

> 膊

>     膊  /  \_/\_ \_  \_\_ \_  \_\_\_  \_\_\_\_/ /\_\_\_\_ \_/

> /          膊

>     膊  \_/ //  ' \/  ' \/ \_ \/ \_\_/ \_\_/ \_ `/ /

>           膊

>     膊 /\_\_\_/\_/\_/\_/\_/\_/\_/\\_\_\_/\_/  \\_\_/\\_,\_/\_/

>       膊

>     膊  \_\_\_\_

>           \_\_          \_\_

>   膊

>     膊  / \_\_ \\_\_\_ \_\_\_ \_\_\_\_\_\_\_ \_\_\_  \_\_\_/ /\_\_ \_\_\_\_  /

> /\_\_\_\_膊

>     膊 / /\_/ / -\_|\_-</ \_\_/ -\_) \_ \/ \_  / \_ `/ \_ \/ \_\_(\_-<膊

>

>     膊/\_\_\_\_\_/\\_\_/\_\_\_/\\_\_/\\_\_/\_//\_/\\_,\_/\\_,\_/\_//\_/\\_\_/\_\_\_/膊

>     膊

>

>           膊

>     膊      Web: http://www.ImmortalDescendants.com

>     膊

>     膊

>   Author: UmE

>   膊

>     膊

> Date: 03/13/00

> 膊

>     膊  Topic: Re-enabling functions - Cool Edit 2000  膊

>

>     膊            Level: Beginner/Intermediate

>         膊

>     膊

>

>           膊

>     膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊膊?

## 介绍

现在有许多软件是以演示版(有一些功能被禁止)提供给用户的.多数情况下,被禁止的功能按钮是灰色的.一般来说,我们能够利用`EnableMenuItem`这个API函数激活它们,但是它们被激活后不能正常工作.在这篇文章中,我将介绍如何使演示版的软件正常工作.

## 所用工具

`SoftIce 3.24` 或更高版本

## 试验软件

`Cool Edit 2000`, `cool2000.exe`, `3420160`字节

让我们开始吧!!!

## 步骤1

当你运行程序时,出现一对话框,提示你想激活那个功能.那意思就是说,你不能使用软件的所用功能,但同时也意味着这个程序包含所有被禁止功能的程序,它是在启动时,根据你的选择进行切换的.我们的工作就是使所有的功能都能使用.如果我们激活选项3和4,软件就提供`Filters`, `Noise redction`, `Amplify`, `Envelope`...但除了`Save`功能.我将教你如何在它被禁止的时候激活它,激活其他功能的过程与此相同.

首先,打开一个文件,你能看到在`File`菜单中的`Save`和`Save As`项是灰色的.我们想激活它,因此进入`SoftIce`(按`CTRL+D`)并设断点(`bpx enablemenuitem`). `EnableMenuItem`函数的结构如下:

```
BOOL EnableMenuItem (
        HMENU hMenu
// handle to menu resource
        UINT uIDEnableItem      // menu item
to enable, disable or gray
        UINT uEnable
// menu item flags
        );
```

在`ASM`中调用的格式是:

```
PUSH    uEnable
PUSH    uIDEnableItem
PUSH    hMenu
CALL    [KERNEL32!EnableMenuItem]
```

对于我们来说,重要的是找到那个激活菜单项的标志我们知道:

`uEnable=0` 菜单项被激活(the item is enabled)

好吧,现在退出`SoftIce`,点击`File`菜单,马上你又回到`SoftIce`.按`F11`键返回到:

```
:00411580 6A01
  push 00000001        <-uEnable flag
:00411582 6875010000              push 00000175
:00411587 56
  push esi
:00411588 FFD3
  call ebx        <- Call EnableMenuItem
```

这不是我们要找的,因为它传递了一个常数给函数如果按`Ctrl+D`,程序将继续执行`EnableMenuItem`功能enable或disable各菜单项在这个过程中,请注意`EBP`寄存器保存的是`uIDEnableItem`,直到你看到`EBP=047E`.这个`ID`正好是`Save As`菜单项(你可以用`W32Dasm`反汇编`cool2000.exe`并找到此`ID`,在列表开始部分的菜单信息区)当你看到`EBP=047E`时按`F11`后程序如下

```
:004128A5 668B440C1C              mov ax,
word ptr [esp+ecx+1C]
:004128AA 50
  push eax
:004128AB 55
  push ebp
:004128AC 56
  push esi
:004128AD FF15D0535600            Call dword ptr
[005653D0]    <-EnableMenuItem
```

`eax(uEnable标志）`的值取决于`ESP`和`ECX`.我们只要将"`mov ax,word ptr[esp+ecx+1c]`"改为"`mov ax,0000`"(注意在指令后面加`nop`,保证与前面指令字节数相同),这样程序将把`0`传递给`EnableMenuItem`函数.按上面的方法修改后,重新启动程序,你将看到`Save As`菜单项已经`enable`了.但是当你按`Save As`时,程序自动关闭了.到底发生了什麽事情?跟我来你会看到!!!

## 步骤2

现在我们已经激活了所有菜单项,我们希望它们能工作.正如你知道的:当点击菜单项时,程序将发出一个`WM_COMMAND`给系统,系统将根据相关的`ID`处理它.

打开`SoftIce`并输入"`hwnd`".这条命令将返回所有在桌面上已开窗口的`handle`.你可看到下列内容:

```
Window Handle    hQueue    Sz    QOwner
  Class Name    Window Procedure
```

我们感兴趣的是`window`如何处理`Cool Edit`的主窗口.我们想从`WM_COMMAND`产生的地方观察到底发生了什麽.注意下面:

```
Window Handle    hQueue    Sz    QOwner
      Class Name    Window Procedure
  0414(1)      1087    32    COOL2000
      COOL2000SS  32CF:0000051E
```

现在在`message WM_COMMAND`(it needs also the handel of the window)处设断点

```
bmsg 0414 WM_COMMAND
```

从`SoftIce`中退出并点击`Save As`菜单项......你将回到`SoftIce`.记住我们要找的是处理此`ID`号的一小段代码:我们能在`source code`中找到,因为编程者不可能修改系统`dll`使其适应他的程序!

设置下面的断点:`bpx k32thk1632prolog`(想了解为什麽?在`www.ImmortalDescendants.com`站点可找到相关资料),按`Ctrl+D`退出`SoftIce`,但马上又回来了,按`F11`键,到:

```
CALL    [KERNEL32!K32Thk1632Prolog]
CALL    [.....]
      <-This is very important!!
CALL    [KERNEL32!K32Thk1632Epilog]
```

按`F8`进入`[KERNEL32!K32Thk1632Prolog]`后面的`call`,直到你找到`cool2000`的代码,现在注意各个寄存器的值,继续跟踪直到:

```
:004C896D 8B8C24EC010000          mov ecx, dword ptr
[esp+000001EC]
:004C8974 8B9424E8010000          mov edx, dword ptr
[esp+000001E8]
:004C897B 51
  push ecx
:004C897C 52
  push edx
  <- 1
:004C897D 57
  push edi
  <- 2
:004C897E 50
  push eax
  <- 3
:004C897F FF15C4545600            Call dword ptr
[005654C4]        <- 4
```

1-将`edx`压入堆栈(`edx=111`).这是`WM_COMMAND`信息的`HEX`码
2-将`edi`压入堆栈(`edi=047e`).这是`Save As`菜单项的`ID`
3-将`eax`压入堆栈(`eax=0414`).这是`cool2000`的`window handle`
4-调用`SendMessageA`函数

这个功能将`WM_COMMAND`信息发送到系统的信息队列.下一步这个信息将被 `DefWindowProcA`函数处理.按`F8`进入`SendMessageA`函数直到你再次来到`cool2000`的代码,在此我们将找到关键点:

```
:00422900 55
  push ebp
:00422901 8BEC
  mov ebp, esp
:00422903 83E4F8
and esp, FFFFFFF8
:00422906 B854140000              mov eax,
00001454
:0042290B E890FE1100              call 005427A0
:00422910 A124AE5800              mov eax,
dword ptr [0058AE24]
:00422915 53
  push ebx
:00422916 56
  push esi
```

注意`EDI=047E`,继续跟踪你将发现程序如何使用`EDI`从此处我们的`ID`将与各种常数比较以确定我们选择的`ID`.跟踪若干行后,发现:

```
:0042C389 81FF7E040000            cmp edi, 0000047E
  <-Yeah!! Our ID!!
:0042C38F 0F8738070000            ja 0042CACD
:0042C395 0F840A080000            je 0042CBA5
      <-Jump to 0042CBA5
```

At `0042CBA5` we find:

```
:0042CBA5 6A01
  push 00000001
:0042CBA7 E8C4AB0600              call 00497770
:0042CBAC 83C404
add esp, 00000004
:0042CBAF 85C0
  test eax, eax
:0042CBB1 7511
  jne 0042CBC4
```

在`0042CBA7`的`call`用于选择被选的`ID`是处理还是跳过.如果`eax=0`,`ID`被跳过;`eax=1`,`ID`被处理.因此将`jne`改为`jmp`,`Save As`菜单项将恢复功能

希望你能喜欢此教程!

Bye!

UmE

Contact me at: `ume15@hotmail.com`
