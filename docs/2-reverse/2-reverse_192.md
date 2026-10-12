
# 请教 cdb/windbg 下，如何使用 bp 对指定文件&指定源码行数下断

> 原文：[`www.pediy.com/kssd/pediy12/137060.html`](https://www.pediy.com/kssd/pediy12/137060.html)

请教大伙一个问题，目前我需要使用 `cdb`/`windbg` 对源文件的某一行下断点。

根据 `windbg` 帮助文档和 MSDN 的解释，`windbg` 默认开启源码调试模式，只需要 `bp` `'mybin.exe!main.cpp:23'` 就可以在 `main.cpp` 的第 23 行下段。对于 `cdb`，需要先用 `-lines` 启动参数，或者 `.lines -e` 命令开启源码调试模式，再如前那样 `bp`。但是实际操作的结果是，这两种方法都有问题，如下：

```
0:000> bp 'main.cpp:23'
0:000> bl
0 e 703a3233 0001 (0001) 0:**** 
```

这个断点是错误的，并且 `703a3233` 这个地址也是错误的，我的 `mybin.exe` 并没有加载到这块空间中。

这个问题我查了不少资料，也按照 MS 的”Debugging in Source Mode“\[1\]中的提示，一步一步操作，仍然不行。

所以在这里请教大伙，希望能得到一些提示，先谢过！

\[1\]”Debugging in Source Mode“：[`msdn.microsoft.com/en-us/libr...=vs.85%29.aspx`](http://msdn.microsoft.com/en-us/library/ff540664%28v=vs.85%29.aspx)

注：我需要*通过命令*对指定文件指定源码行进行下断。而不是用鼠标选中然后 F9。

## 附上我的操作记录

### 1. windbg

```
0:000> .sympath+ d:\\123\\123\\Debug\\
Symbol search path is: D:\\WorkSpace\\symbols\\local;srv*D:\\WorkSpace\\symbols\\local*http://msdl.microsoft.com/download/symbols;d:\\123\\123\\Debug\\
Expanded Symbol search path is: d:\\workspace\\symbols\\local;srv*d:\\workspace\\symbols\\local*http://msdl.microsoft.com/download/symbols;d:\\123\\123\\debug\\
0:000> .srcpath+ d:\\123\\123\\
Source search path is: d:\\123\\123\\
0:000> bl
0:000> bp main
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
0:000> bp 'main.cpp:23'
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
1 e 703a3233 0001 (0001) 0:**** 
0:000> bc*
0:000> bp '123!d:\\123\\123\\main.cpp:23'
0:000> bl
0 e 703a3233 0001 (0001) 0:**** 
```

### 2. cdb

```
0:000> .lines -e
Line number information will be loaded
0:000> l+s
Source options are 4:
4/s - List source code at prompt
0:000> l+t
Source options are 5:
1/t - Step/trace by source line
4/s - List source code at prompt
0:000> .srcpath+ d:\\123\\123\\
Source search path is: d:\\123\\123\\
0:000> .sympath+ d:\\123\\123\\Debug\\
Symbol search path is: D:\\WorkSpace\\symbols\\local;srv*D:\\WorkSpace\\symbols\\local*http://msdl.microsoft.com/download/symbols;d:\\123\\123\\Debug\\
Expanded Symbol search path is: d:\\workspace\\symbols\\local;srv*d:\\workspace\\symbols\\local*http://msdl.microsoft.com/download/symbols;d:\\123\\123\\debug\\
0:000> bp main
*** WARNING: Unable to verify checksum for 123.exe
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
0:000> bp 'main.cpp:23'
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
1 e 703a3233 0001 (0001) 0:**** 
0:000> bc*
0:000> bp '123!d:\\123\\123\\main.cpp:23'
0:000> bl
0 e 703a3233 0001 (0001) 0:**** 
```

难得见到 `windbg` 进行源码级别调试的。

```
0:000> bp 'main.cpp:23'
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
1 e 703a3233 0001 (0001) 0:**** 
```

`bp` 语法是正确的，结果也是正确的，你需要添加一个 `module` 名称，不需要加 `path`

`bp` `123!main.cpp:23`

最关键的一点，使用"`**`"而不是"**'**" 

引用: *最初由 **ronging**发布* ![查看帖子](img/showthread.php?p=979985#post979985) 难得见到 `windbg` 进行源码级别调试的。

```
0:000> bp 'main.cpp:23'
0:000> bl
0 e 00430820 [d:\\123\\123\\main.cpp @ 15] 0001 (0001) 0:**** 123!main
1 e 703...
```

嗨，感谢您的回复。

问题确实是因为：我使用的是单引号，而不是飘号（）。非常感谢

然后您说的其他的几个点，我解释一下：

1. 跟 `module` 关系不大，`module` 只是个可选参数。
2. 加完整路径的原因是，避免和其他模块的 `main.cpp`（如果存在的话）产生歧义。

再次感谢
