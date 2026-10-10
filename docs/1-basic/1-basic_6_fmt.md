
# 给力汇编

> 原文：[`www.pediy.com/kssd/pediy12/132387.html`](https://www.pediy.com/kssd/pediy12/132387.html)

很久以来在论坛上都是以潜水为主。都没有做什么贡献。今天也发个帖子吧！讨论下在汇编中用一些给力的宏达到一些想不到的效果。正如罗云彬所说。汇编不只是用来写少量模块的语言，汇编也是很给力的..其实如果你真心喜欢上了汇编，也是能够发现很多不可思议的地方。

注意。观看本文需要你使用 `masm32+kmdkit` 软件包。其实差不多有 `masm32` 就够了，`kmdkit` 本文说的很少。还有需要你设置好了环境变量。这个问题我不会在这里讨论。如果你下面的宏你用不了，差不多就是环境变量的问题。。

还有这个帖子是我在看雪的发帖板块现场敲的代码。估计有地方我敲错一个字符什么的在所难免，如果你发现错误。请指出。谢谢！

## 简化的汇编代码

首先以前如果你每次写汇编程序都这样：

```
.586
.model flat, stdcall
option casemap:none
Include windows.inc
....
```

太长了，为此 `masm32` 包里面早就给我们带了一个很有用的头文件，如果你是用 `masm32` 写

```
Include masm32rt.inc
.code
main proc
    Invoke crt_cprintf, cfm$("hell")
main Endp
End main
```

注意你也许看到我使用了 `crt_cprintf`，这个函数哪里来的呢？很简单。翻开你的 `masm32` `Include` 文件夹，查找 `msvcrt.inc` 头文件里面已经带了 C 语言所有的函数库了。你也许会说。用了这个函数有没有什么副作用什么的。其实微软从 2000 或者 xp 开始吧，系统已经带了这个运行库。C 语言的运行库。我还想研究下 C++的运行库。调用 C++的 `cout`，不过鉴于这个 `cout` 使用起来比较麻烦。就作罢了。其实没事的时候你可以去 windows 文件夹下面去翻翻。学汇编就应该有这个必要。看看里面有什么有价值的东西。

当然前面加个 `crt_` 觉得有点长。`aogosoft` 上面已经有人搞成普通 C 语言的用法了。我就不说了自己去找找。我不喜欢那样。我喜欢前面只加个 `cprintf` 用以和 windows 的函数做区分。

### 注释

关于注释我也说两句，其实在汇编里面是支持两种注释的，一种是以 `;` 号开头的作为注释。和 C++的 `//` 一样。

还有一种注释是

```
comment #
    This is comment
    This is comment
        #
```

两个 `#` 好中的就是注释，当然你也可以不用 `#` 用 `$` 或者其他的。自己试试就能够试出效果了。。

## 文本宏

首先一个在汇编中经常写代码需要这样。比如调用 `MessageBox`

```
Invoke MessageBox, NULL, offset SzText, offset SzCaption, NULL
```

这也是很蛋疼的一件事情，当然网上已经有很多教程说到了用 `masm32` 包里面带的 `cfm$` 宏。是的当然我还会说一些别人没有说到的东西。如果用了 `cfm$` 宏以后效果就是这样了：

```
Invoke MessageBox, NULL, cfm$("hello"), cfm$("Information"), NULL
```

简单一些了，很舒服，这个宏很多地方都有，包括 `kmdkit` 包里面带的 `$CTA0` 宏。也是差不多的效果。不过 `$CTA0` 宏比 `cfm$` 好一些。因为 `$CTA0` 在字符串重复定义的时候不会浪费空间。说了这些宏相信你应该很亟不可待的想使用下这些宏的效果了。再说一次如果你的环境是 `masm32+kmdkit` 这些宏都可以直接用了。。

## 汇编支持 Unicode

关于这个宏是我最近才搞定的问题。其实 `masm32` 包里面已经自带了 Unicode 宏支持。不过只支持英文。.我倒。这样的宏有什么用呢？`masm32` 不支持中文宏一直以来都是我的一块心病。其实 `gb2312` 转 Unicode 也是一个查表的工作。只是工作量太大。我原本计划有时间的时候就自己写一个查表的宏来支持 Unicode。不过有人已经做了这件事情。奇怪的是为什么这件事情不广为人知。难道大家都没有让汇编支持 Unicode 的想法？想不通了。好吧以后你可以这样使用 Unicode 宏了，当然支持中文的。这个宏我已经上传了。以后你可以抛弃 `cfm$` 了。就和我一样。来看看使用 Unicode 程序是什么样的，宏是人家写的不过我改了下名字。因为名字短点写起来快。

```
Include masm32rt.inc
Include Unicode.inc
;===================================================================

.data
SzText ustr ("中文中文", 0)

.code
main proc
    Invoke MessageBoxW, 0, offset SzText, ustr$("中文提示"), NULL
    ret
main Endp
End main
```

已经很给力了吧。呵呵我用的比较多的是 `ustr$`，`ustr$`和 `ustr` 一个是宏，一个是宏函数。关于他们的区别。请 google 你在使用这个宏的时候记得导入 `Unicode.inc` 头文件。就可以了。用汇编写程序舒服吗？

## 高级语法

上面说的已经有些给力了吧。估计你还不满足。我了解这点。我也是一样。以前我听了罗云彬的话一般不用宏。当然现在我的汇编程序大量使用宏。就和上面的 `ustr$` 这种一样，它并不会在代码中添加什么东西。

关于高级语法。好像汇编比其他语言少的就是 `switch..case` 和 `for` 循环。来我们搞定这两个。`switch` 宏 `masm32` 包已经给我们搞定了而且搞的比 C 语言的还给力。C 语言的只能够比较 `dword` 类型吧。汇编的还能够 `switch` 字符串。真给力，看看用法。这个是 `masm32` 带的，自己去 `masm32` 包里面找找能够找到你想要的东西。打包 `masm32` 的这个家伙已经把很多东西都给我们搞定了。里面有很大好用的函数。和宏。包括 `readline`。以前用 Java 对这个印象非常深刻。一次读一行。`masm32` 里面有。我不能够都说完，很多东西还需要自己去发掘..总之就是给力。.

```
switch uMsg
case WM_INITDIALOG
    invoke SendMessage,hWin,WM_SETICON,1, FUNC(LoadIcon,NULL,IDI_ASTERISK)
    m2m hWnd, hWin
    return 1
case WM_COMMAND
    switch wParam
    case IDOK
        fn MessageBox,hWnd,"You clicked OK","Hi",MB_OK or MB_ICONINFORMATION
    case IDCANCEL
        jmp quit_dialog
    endsw
case WM_CLOSE
    quit_dialog:
    invoke EndDialog,hWin,0
endsw
```

恩，其实这个宏很给力的，只是这个编辑器有点蛋疼。`masm32` 自带的帮助里面哟详细的说明文档。我就不嗦了。`switch$` 可以比较字符串。上面还用了其他的宏。都是 `masm32` 自带的 `fn` 啊，`rv` 啊。什么的。自己好好看看..`return` 也有

`switch` 搞定以后还有什么没有... 对 `for` 没有。我以前学 Java 喜欢动不动就 `for` 一下。不过现在不这样了。现在比较喜欢用 `while` 了。当然是在这个 `for` 宏之前。看看什么样子？

```
Include masm32rt.inc
Include for.inc
.code
;===================================================================
main proc
    local _dwTemp:dword
    ;初始化 ebx 的值为 0, 最大为 10 每次步进 1
    _for ebx, 0, 10, 1
        ;Invoke cprintf, 0, cfm$("%d"), 0, ebx
    _next
    ;初始化 _dwTemp 为 0, 最大为 10 每次步进 2
    _for _dwTemp, 0, 10, 2
    _next
    ret
main Endp
End main
```

用法注释里面有了。名字我弄成了 `_for`，附件我也上传了。`for.inc`。自己下载观察下。对了这个宏是 `objasm` 中带的。里面有很大的宏。规模大过 `masm32` 里面的宏规模。还是那句话。我挑重点的说说。然后还是需要你自己去发掘新的东西。

## 异常处理

到这里为止，宏已经帮了我们做很多的事情。不过还是有些地方让人郁闷。比如啊。汇编在异常处理的时候非得分一个函数来处理，和逻辑分开了。不给力。如果能够向 Java 那样就舒服了 `try..catch..finally..` 呵呵。虽然差点不过你看看这个好使不？

```
Include masm32rt.inc
Include seh.inc
.code
;===================================================================
main proc
        local _dwTemp:dword
        Try 1
               mov eax, 0
                mov dword ptr [eax], 0
        Catch 1
               ;在 catch 中宏已经把这些东西准备好了, 你直接可以使用
               ;ebx-->EXCEPTION_RECORD 结构
               ;esi-->CONTEXT 结构
               ;edi-->lpSEH 处
                Invoke cprintf, cfm$("Exception addr: %p"), [esi].regEip
        Invoke MessageBoxW, 0, ustr$("异常了"), ustr$("Information"), NULL
        EndTry 1
        ret
main Endp
End main
```

具体用法自己研究下吧，对了这个宏还是可以自定位的哦。因为我原来写了个项目完全是自定位的代码。罗云彬说自定位的代码只能够用在少量的核心中，其实不然。如果能够好好的利用宏。写自定位的代码和写普通代码没有什么区别，但是给你带来的好处是无法想象的..呵呵。

当然还要说一个宏，罗云彬书上有摘抄出来 `_invoke`。这个宏也很不错。至少我在网上见了很多人写的宏没有这个写的漂亮。代码让人拍案叫绝。非常不错。

好了。其实还有很多我平时经常用的宏。我也就不多说了。足够多了。如果你细心的发现。其实还有很多的宏。很好用的。呵呵。以前我自己写了不少的宏。后来发现写的都没有别人的好。而且这些工作都已经有人给我们做了。我们就直接站在巨人的肩膀上吧哈哈。

这些宏。你可以选择使用，也可以选择不使用。我想说的是真的非常不错。我使用宏的一个前提是能够给代码带来逻辑上的好处。如果使用一个宏能够简便操作，但是逻辑不清晰。不要使用。能够简化逻辑的宏才是真正的宏...呵呵。到这里吧。回头有时间再写点。.

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `macro.rar` |

