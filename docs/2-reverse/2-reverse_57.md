
# 请教 OD 中：`ecx` 指向 "abcdefghigklmn" 这个字符串的条件断点如何下？

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-820.htm`](https://www.pediy.com/kssd/pediy08/pediy8-820.htm)

请教 OD 中：`ecx` 指向 "abcdefghigklmn" 这个字符串的条件断点如何下？

引用:  *最初由 Spring.W 发布*

**我想断：`CreateFileA` 条件是文件名字为 "abcdefghigklmn"，我在 `CreateFileA` 上 `Shift-F4` 或 `Shift-F2` 设定条件，都不起作用。我用的不对？能否详细解说一下？谢谢！ **

不好意思，这几天太困，睡了一觉。如果是 `CreateFileA`，你可以在命令行中这样下条件断点：

```
bp CreateFileA,[[STRING [esp+4]]]=="abcdefghigklmn"
```

若按 `OllyDBG` 中的帮助文档上说的，应该是

```
bp CreateFileA,[STRING [esp+4]]=="abcdefghigklmn"
```

或

```
bp CreateFileA,[[esp+4]]=="abcdefghigklmn"
```

但我测试过如果按 `OD` 帮助文档中所说的下条件断点的话，断不下来。原因尚不清楚，等有空再来测试看看。

昨天主要是赶着去医院看孩子，具体为什么这么设断没说明。今天白天回来睡觉前再来解释一下，对 `Spring.W` 兄来说应该是没必要，这里主要是想让新手了解一下：

首先我们看一下 `CreateFileA` 在 `MSDN` 中的解释：

```c
HANDLE CreateFile(
  LPCTSTR lpFileName,               // pointer to name of the file
  DWORD dwDesiredAccess,            // access  (read-write) mode
  DWORD dwShareMode,                // share mode
  LPSECURITY_ATTRIBUTES lpSecurityAttributes,
                                   // pointer to security attributes
  DWORD dwCreationDisposition,      // how to create
  DWORD dwFlagsAndAttributes,       // file attributes
  HANDLE hTemplateFile              // handle to file with attributes to
                                   // copy
);
```

从上面我们可以看出第一个参数就是文件名指针，也就是说这个参数中存放的就是文件名称的地址。在 32 位程序中调用这个函数时这个参数的堆栈地址就应该是 `esp+4`（4×8=32），同理，下一个参数就应该是 `esp+8`。其它类推，返回值是 `esp+0`。我们现在要判断文件名，这里的文件名就是 `esp+4` 地址所指向的地址中的内容。取地址中的内容在 `OD` 中用双方括号来操作，如取 `esp+4` 中的内容就该写成这样：`[esp+4]`。现在我们取的 `[esp+4]` 中的内容还是个地址，所以要得到文件名则还要再取这个地址中的内容，就该这样：`[[esp+4]]`。而那个 `STRING` 前缀在 `OD` 中的解释是以零作为结尾的 `ASCII` 字符串。所以我们下条件断点时这样写：

```
bp CreateFileA,[STRING [esp+4]]=="abcdefghigklmn"
```

但却发现断不下来，写成这样：

```
bp CreateFileA,[[STRING [esp+4]]]=="abcdefghigklmn"
```

才能断下来，这里就应该是三层的地址了。为什么这样目前尚不清楚，等我以后有空写第六篇条件断点时我会再详细解释一下。

`OD` 在处理这些中括号的时候
好像是有些问题
在搞那些指针指来指去的时候
我就发现我的条件断点没用
后来有一招
可以先把你要用的条件表达式写成一个 `Watch`
在那个 `Watch` 里可以检测一下对不对
不对就再加个 `[]` 试试
记得有一回我就加了一对 `[]` 才搞定
方法比较粗俗
但还算好用
加上用条件断的地方不多
也就没再细研究
写上来和大家分享

```
bp CreateFileA,[[STRING [esp+4]]]=="abcdefghigklmn"
```

若按 `OllyDBG` 中的帮助文档上说的，应该是

```
bp CreateFileA,[STRING [esp+4]]=="abcdefghigklmn"
```

或

```
bp CreateFileA,[[esp+4]]=="abcdefghigklmn"
```

爽啊!
