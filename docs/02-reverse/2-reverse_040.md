
# OllyDbg 实用技巧六则

> 原文：[`www.pediy.com/kssd/pediy05/pediy50412.htm`](https://www.pediy.com/kssd/pediy05/pediy50412.htm)

dOSKEY lEE

关键词：`OllyDbg`、`OD`、技巧

## 1、让跳转路径显示出来

打开`Options\Debugging Option`。弹出`Debugging Option`对话框，选择`CPU`页，选定“`Show direction to jumps`”、“`Show jump path`”和“`Show grayed path if jump is not taken`”。如此以来在`Disassembler`窗口就会显示跳转的路径了。

## 2、让 `OD` 显示 `MFC42.DLL` 中的函数

如果程序是用`MFC`进行的动态编译，那么在`OD`中将只能显示`MFC42.DLL`中的函数为：

```
00410E40 |. E8 43000000 CALL <JMP.&MFC42.#1576>
```

`1576`是函数在`MFC42.DLL`中的序号。打开`Debug\Select import libraries`，单击弹出的对话框中“`Add`”，在弹出的打开文件对话框中选择“`MFC42.LIB`”并打开，重新载入`MFC`程序，你就可以看见函数名称变为：

```
00410E40 |. E8 43000000 CALL <JMP.&MFC42.#1576_?AfxWinMain@@Y>
```

`IDA`中分析出来了东西一样了！呵呵，以后不用等待`IDA`的“细嚼慢咽”也可以轻松搞定`MFC`程序了。其他的`DLL`类似，如果有序号，可以在`VC`的`LIB`目录中找到相关的`.LIB`文件，加到`OD`中便可。如果你没有“`MFC42.DLL`”，你可以到新浪论坛的下载区找，我已经上传到那里了。

## 3、让 `OD` 轻松躲过“ANTI-DEBUG”

很多“`ANTI-DEBUG`”的程序都是在程序开始时来检查是否安装调试器的。用这种特性我们可以轻松的用`OD`的“`Attach`”绕过检查部分。如“`X 语言`”，如果你用`TRW2K`/`S-ICE`/`OD`直接加载它的话，程序回警告你安装了调试器并结束。但是我们在“`X 语言`”开启后再运行`OD`，并用“`Attach`”系上它就可以了，轻松通过检查。而且在`OD`系上它后仍然可以用`CTRL+A`进行分析。如此一来，快哉！：）

## 4、轻松对付调用“MessageBoxA”以及类似的模态对话框的程序

很多人都认为`OD`不好拦截“`MessageBoxA`”这类`API`函数。其实我们有个很简单的办法将`API`拦截下来，并且快速找到比较地点/主算法地点。首先用`OD`加载目标程序，如果不能加载，用上面的方法“`Attach`”目标程序。然后，`F9`运行目标程序，并且有意让目标程序显示“`MessageBox`”，然后切换到`OD`中，`F12`暂停，如：

```
0041201F |> 53 PUSH EBX ; /Style
00412020 |. 57 PUSH EDI ; |Title
00412021 |. FF75 08 PUSH [ARG.1] ; |Text
00412024 |. FF75 F4 PUSH [LOCAL.3] ; |hOwner
00412027 |. FF15 A8534100 CALL DWORD PTR DS:[4153A8] ; \MessageBoxA
0041202D |. 85F6 TEST ESI, ESI ; 停在此处
0041202F |. 8BF8 MOV EDI, EAX
00412031 |. 74 05 JE SHORT 1551-CRA.00412038
```

`F8`单步一下，切换到“`MessageBox`”中，确认，被`OD`中断。我们可以看见上面的代码`41201F`处有一个“〉”，说明可以从某段代码跳转到此处，我们选择`41201F`这一行，在“`Information`”栏看见一句“`JUMP FROM 412003`”，右键单击，选择“`GO TO JUMP FROM 412003`”。回到`412003`，一般都是条件跳转，上面的内容就是比较的地方啦。：）

## 5、使用 `OD` 的 `TRACK` 功能

`OD`拥有强大的`TRACK`功能，在分析算法时十分有用。首先我们要设定`OD`的`TRACK`缓冲区大小，选择`Option\Debugging Option`，在弹出的对话框中选择`TRACK`页，“`Size of run track buffer(byte/record)`”，缓冲区大小，当然约大越好。其他的设置在我以前的`OLLYDBG.INI`中都已经设置好了。然后，开启目标程序，在`DEBUG`中选择“`Open or clear run track`”。然后我们就可以用`CTRL+F11`或`CTRL+F12`开启“`Track into`”和“`Track over`”了。当我们暂停程序的时候，可以用小键盘上的“`+`”、“`-`”、“`*`”来控制`TRACK`功能了。

- “`Track into`”和运行类似，但是记录所有指令以及寄存器变化。并且会自动进入所有的`CALL`中。
- “`Track over`”和“`Track into`”类似，但是不进入`CALL`。
- “`+`”用来显示`TRACK`缓冲区中的下一条指令。
- “`-`”用来显示`TRACK`缓冲区中的上一条指令。
- “`*`”用来发返回当前指令。

## 6、不是技巧的技巧

当你遇到花指令的时候一定会很头痛。但是如果你用`OD`进行分析的时候就会轻松得多。`OD`会自动标识出无效指令，即花指令。如果`OD`没有正确识别，你还可以用`CTRL+↑/↓`来单个字节的移动。可以很有效的识别出花指令的所在。

## 后记

本人脑袋不太好使，一时记不起所有内容，以后想起来再贴。转贴时请保持本文的完整。

`4dd466d0e1e0cc123afbe504da2462ca`
