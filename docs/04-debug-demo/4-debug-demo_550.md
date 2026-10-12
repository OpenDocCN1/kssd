
# WinImp 1.11

> 原文：[`www.pediy.com/kssd/pediy04/kanxue210.htm`](https://www.pediy.com/kssd/pediy04/kanxue210.htm)

## 解除 WinImp 的自校验

点睛工作室--梁利锋

### 声明

个人可以自由转载本文，不过应保持原文的完整性，并通知我；商业转载先请和我联系。

本文没有任何明确或不明确地提示说本文完全正确，阅读和使用本文的内容是您自己的选择，本人不负任何责任。

如果您发现本文有错漏的地方，请您给我指出；如果有什么不理解的，请您给我提出。

意见、建议和提出的问题最好写在我的主页`http://llf.126.com`的留言版上。

### 前言

最近知道，类似`WinImp`这样的软件有自校验的功能，如果软件被修改任何一个字节，都会出现出错提示，从而不能进入软件界面。这好像也是一个遥远的回忆了，很久以前的软件使用这种方法来检测病毒，而且`pctools`自带的查毒软件就有一种功能，可以给任何软件加上一个自检测的外壳。不过，随着`Windows`的流行，以前的病毒作者好像不习惯这种变化，像以前这种修改可执行文件方式的`Windows`病毒几乎绝迹，所以目前使用自校验的程序也比较少，但是毕竟还有。另外，因为这并非为了加密，而只是检测文件安全性的手段，所以破解很简单，我在这里介绍一下，希望那些并不会汇编语言的同志们也可以看懂并且自行修改。

（除这一类自校验以外，我以前常用的是`Arj`，只要键入“`Arj i`”，`Arj`就进行自校验，如果自校验出错的话，一般总是中毒了。）

### 解除

我使用`WinImp 1.11`做例子。首先修改`winimp32.exe`，使之出现错误提示，我们见到一个消息框（`MessageBox`），其中的提示为：

```
The WinImp executable file has been modified during or after extraction. It may have been damaged by a disk or download error, or your computer may have a virus. Make sure your virus scanner is the latest version. You may need to download WinImp again.
```

现在，用`W32dasm`打开`winimp32.exe`，然后保存，结果得到一个`winimp32.alf`文件，关闭`W32dasm`，我们现在对`winimp32.alf`文件进行分析。用一个文本编辑器打开`winimp32.alf`，可以看到开头的几行是这样的：

```
Disassembly of File: winimp32.exe

Code Offset = 00000400, Code Size = 0003BE00

Data Offset = 0003DE00, Data Size = 00012000

Number of Objects = 0006 (dec), Imagebase = 00400000h

  Object01: AUTO     RVA: 00001000 Offset: 00000400 Size: 0003BE00 Flags: 60000020
```

在上面可见到，蓝色的部分说明此程序的基地址为`00400000h`，而绿色的部分说明它的代码的实际地址和虚拟地址之间有`00001000h - 00000400h = 00000C00h`的差别，所以总的可执行文件中偏移量和程序代码中的偏移量之差是`00400000h + 00000C00h = 00400C00h`。

现在，我们查找在消息框中出现的文本，因为不是杀毒软件，所以出现`virus`的概率很低，查找`virus`，发现和以上消息框同样的字符串，其前后的代码如下：

```
:00430D45 E856A6FFFF                 call 0042B3A0
:00430D4A 3B051CD24400               cmp eax, dword ptr [0044D21C]
:00430D50 741C                       je 00430D6E
:00430D52 6A30                       push 00000030

* Possible StringData Ref from Data Obj ->"WinImp"
                                      |
:00430D54 6830D24400                 push 0044D230

* Possible StringData Ref from Data Obj ->"The WinImp executable file has "
                                      ->"been modified during or after "
                                      ->"extraction. It may have been damaged "
                                      ->"by a disk or download error, or "
                                      ->"your computer may have a virus. "
                                      ->"Make sure your virus scanner is "
                                      ->"the latest version. You may need "
                                      ->"to download WinImp again."
                                      |
:00430D59 68C0AE4400                 push 0044AEC0
:00430D5E 6A00                       push 00000000

* Reference To: USER32.MessageBoxA, Ord:0048h
                                      |
:00430D60 2EFF150CD84300             Call dword ptr cs:[0043D80C]
:00430D67 B8FFFFFFFF                 mov eax, FFFFFFFF
:00430D6C EB65                       jmp 00430DD3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00430D50(C)
|
:00430D6E A140384500                mov eax, dword ptr [00453840]
:00430D73 31C9                      xor ecx, ecx
```

其中绿色的部分说明这一次调用（`Call`）的是`MessageBoxA`，正和我们的预测相同。`MessageBoxA`有四个参数，所以上面也有四个`push`语句，不理会它。再向上，蓝色的部分是比较语句（`cmp`），而红色的部分是条件跳转语句（在这里是`je`），那么这次跳转究竟跳到哪里了呢？就是最后紫色的`00430D6E`的位置，正好跳过这个`MessageBoxA`，而且在`00430D6E`之上的一句是跳转（`jmp`）语句，更说明红色的部分就是我们要修改的部分。

记下红色部分的偏移量`00430D50`，然后和总的偏移量差别进行运算，`00430D50 - 00400C00h = 00030150h`，说明此代码在可执行文件的`00030150h`偏移处。用`UEdit`打开`winimp32.exe`，跳到`0x30150`处，发现这里就是代码“`je 00430D6E`”（`741C`）所在的地方，`je`就是`74`，我们把它改成跳转`jmp`，就是`eb`，所以就是把此处的“`74`”改成“`EB`”，存盘，运行，成功！

如果不习惯计算偏移量，可以使用查找的方法，在“`741C`”上面是“`3B051CD24400`”，就查找“`3B051CD24400`”，发现只有一处，后面是“`741C`”，把“`74`”改成“`EB`”，存盘，运行，成功！

这种方法只是简单的跳过检测，如果可以查到重新计算的数值，把它写入可执行文件的话，就仍然保留了原程序的病毒检测功能，应该是比较完美的，不过这样的话，一定要使用`Trw2000`，要求就高了一些，如果有兴趣的，可以一试！

### 另类的完全破解

伟告诉我，用以上的方法破解，有一个问题，就是`WinImp`只能打开`*.imp`文件了（也可以打开`*.rar`），对于原来支持的各种格式，比如`zip`现在都不支持了，这实在是不好，所以我又对它进行了分析，结果发现它进行了多次的自校验，如果只是简单的像上面那样修改的话，就可能出现各种问题，但是因为它调用校验的次数太多（大约一百余次），所以真的全部修改的话会非常繁琐，想必非人力所能及，最后我终于想出一种完全破解的方法。

对于这种方法，因为需要太多的汇编语言的知识，所以我就不详细讲了，只简单介绍一下原理。

`WinImp`的自校验在开始的时候先打开自己（`winimp32.exe`），并放进内存，然后的校验都是对于这一块内存进行的，我的方法是让他打开一个名为`winimp32.ex`的文件，把此文件放进内存进行校验，这个`winimp32.ex`文件就是原始的`winimp32.exe`文件的副本，所以校验一定会通过。

具体方法就是，首先把原始的`winimp32.exe`文件在同一目录复制一份名为`winimp32.ex`，然后修改`winimp32.exe`。修改方法是，查找“`68040100008D85F8FEFFFF50FF35101245002EFF1508D64300 BA000000808D85F8FEFFFFE8E348000089C183F8FF750AB8FBFFFFFF`”，把它改成“`8D85F8FEFFFF506A7F50FF35101245002EFF1508D6430003442400 48C6000058BA00000080E8E248000089C183F8FF75096AFB5890`”即可。

不过这种方法有一种限制，就是目标文件的全路径名不能长于`127`个字节，虽然大多数情况成立，但是毕竟是一种限制，如果用这种方法做汉化的时候，请一定在说明里注明此问题。

下面附上对于上面修改的字节流的汇编代码，如有兴趣，不妨读一读，如果能把上面所说的`127`字节的限制取消的话，请一定通知我。

#### s

```
6804010000                 push 00000104
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
50                           push eax
FF3510124500               push dword ptr [00451210]
2EFF1508D64300             Call dword ptr cs:[0043D608]
BA00000080                 mov edx, 80000000
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
E8E3480000                 call 004355B1
89C1                       mov ecx, eax
83F8FF                     cmp eax, FFFFFFFF
750A                       jne 00430CDF
B8FBFFFFFF                 mov eax, FFFFFFFB
E9F4000000                 jmp 00430DD3
```

#### c

```
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
50                           push eax
6A7F                       push 7F
50                           push eax
FF3510124500               push dword ptr [00451210]
2EFF1508D64300             Call dword ptr cs:[0043D608]
03442400                   add eax,dword ptr[esp+0]
48                           dec eax
C60000                       mov byte ptr [eax], 00
58                           pop eax
BA00000080                 mov edx, 80000000
E8E2480000                 call 004355B1
89C1                       mov ecx, eax
83F8FF                     cmp eax, FFFFFFFF
7509                       jne 00430CDF
6AFB                       push FB
58                           pop eax
90                           nop
E9F4000000                 jmp 00430DD3
```

上面，`s`部分表示原文件中的汇编代码，而`c`部分表示修改后的汇编代码。

最后说一下，上一节介绍的方法对于其它进行自校验的程序仍然很可能是可以的，下面这种方法只是用来对付像`WinImp`这种非常顽固的程序的。:)

### 解除 127 字节限制

要解除上面所说的`127`字节的限制，主要的困难在于没有空间，所以我把更下面的一段代码也加入，从那里得到了两个字节，其汇编代码如下：

#### s

```
6804010000                 push 00000104
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
50                           push eax
FF3510124500               push dword ptr [00451210]
2EFF1508D64300             Call dword ptr cs:[0043D608]
BA00000080                 mov edx, 80000000
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
E8E3480000                 call 004355B1
89C1                       mov ecx, eax
83F8FF                     cmp eax, FFFFFFFF
750A                       jne 00430CDF
B8FBFFFFFF                 mov eax, FFFFFFFB
E9F4000000                 jmp 00430DD3
A114D24400                 mov eax, dword ptr [0044D214]
E8A7590000                 call 00436690
A340384500                 mov dword ptr [00453840], eax
85C0                       test eax, eax
750A                       jne 00430CFC
B8FEFFFFFF                 mov eax, FFFFFFFE
E9D7000000                 jmp 00430DD3
```

#### c

```
8D85F8FEFFFF               lea eax, dword ptr [ebp+FFFFFEF8]
50                           push eax
6804010000                 push 00000104
50                           push eax
FF3510124500               push dword ptr [00451210]
2EFF1508D64300             Call dword ptr cs:[0043D608]
03442400                   add eax,dword ptr[esp+0]
```



```asm
48                   dec eax
C60000               mov byte ptr [eax], 00
58                   pop eax
BA00000080           mov edx, 80000000
E8DF480000           call 004355B1
89C1                 mov ecx, eax
83F8FF               cmp eax, FFFFFFFF
7508                 jne 00430CDF
6AFB                 push FB
58                   pop eax
E9F2000000           jmp 00430DD3
A114D24400           mov eax, dword ptr [0044D214]
E8A5590000           call 00436690
A340384500           mov dword ptr [00453840], eax
85C0                 test eax, eax
7508                 jne 00430CFC
6AFE                 push FE
58                   pop eax
E9D7000000           jmp 00430DD3
```

其字节代码也在其中，我就不再写了，不过因为改起来非常麻烦，所以我提供了破解自校验后的程序的下载，想汉化的话用这个破解版就可以了。

`-- 完(2000.07.16) --`

`---------------------------------------------------------------`

不要爱我因为我爱上网络

不要恨我因为我不值得 `4dd466d0e1e0cc123afbe504da2462ca`
