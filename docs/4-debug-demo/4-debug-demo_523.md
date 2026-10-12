
# Any to Icon 2.1b

> 原文：`[www.pediy.com/kssd/pediy06/pediy6953.htm](https://www.pediy.com/kssd/pediy06/pediy6953.htm)`

原版下载：`http://www.aha-soft.com/download.htm`

这是 fly 前天提到的一个话题“脱壳后仍然有问题...”。今天看了后觉得作者在“暗桩”和“版权信息”上的手法值得大伙看看，就写了以下文字，不当之处还望各位大侠指点。

## 一、脱壳

因为不是 `aspr` 新版的壳，用 `od` 可以非常方便地在伪 `oep` 处从堆栈中找回 `stolen code`，如下：

```
PUSH EBP
MOV EBP,ESP
ADD ESP,-0C
MOV EAX,4D4198
```

在 `od` 中于 `4d4520` 处开始键入以上代码后 `dump`，再修复 `iat` 就算脱壳了。（这一节还算顺利）

## 二、修复

程序经上面手续后，界面出来了，就是不能正常开工——载入文件点击 `next` 就出错！看看吧，......，

呵呵，运气不错哦，在出错代码不远看到：

```
cmp ebx,0c3500
jle xxxxxx
```

（看看这里的 `ebx` 值）在此处下断后重新加载执行，顺着上次的错误路线，停在断点时，`ebx=13c00`。`13c00` 是一个什么数？

一种预感让我用鼠标右键点上了 `dumped\_.exe`，哈哈，果然在预料中，那是 `dumped\_.exe` 的 `size` 值啊！

作者在这里检查文件是否被 `unpack` 啊，好吧，简单地把上面的 `jie` 改成 `jmp` 试试，ok!

程序现在可以顺利地把一个图片搞成 `icon` 文件了，是不是很简单啊！（真这样也就没有这篇破文了）

但是...，这个但是的出现真把人搞得...，麻烦！

这个“但是”是：如果过程中你点击 `<back` 或者 `repeat` 之类的按钮，程序会进入没有尽头的递归，结果是可想的了！！！

哪里出毛病了？估计还是那个 `size` 值吧？那么程序在哪里获得这个值的，考察 `api`：

1. `GetFileSize`
2. `SetFilePointer`

辛辛苦苦看了半天，却没有发现什么！看来作者是有备而来啊（这才有意思嘛）。好吧，让咱再试试...，

哈哈，终于还是发现与 `FindFirstFileA` 这个函数相关，就是这句了：

```
0040880F CALL <JMP.&kernel32.FindFirstFileA>
```

咋办？总不能简单处理吧！没法只好加入一段代码来应付了，在程序内找一“空地”（我选定 `53b180`）键入：

```
0053B180 POP EAX
0053B181 XCHG DWORD PTR SS:[ESP+4],EAX
0053B185 MOV DWORD PTR DS:[53B160],EAX
0053B18A XCHG DWORD PTR SS:[ESP],EAX
0053B18D MOV DWORD PTR DS:[53B168],EAX
0053B192 PUSH EAX
0053B193 CALL DWORD PTR DS:[<&kernel32.FindFirstFileA>]
0053B199 PUSH EAX
0053B19A PUSH ESI
0053B19B PUSH EDI
0053B19C PUSH EBX
0053B19D PUSH ECX
0053B19E PUSH EDX
0053B19F CALL DWORD PTR DS:[<&kernel32.GetCommandLineA>]
0053B1A5 MOV ESI,EAX
0053B1A7 MOV EDI,DWORD PTR DS:[53B168]
0053B1AD MOV ECX,-1
0053B1B2 MOV AL,0
0053B1B4 REPNE SCAS BYTE PTR ES:[EDI]
0053B1B6 NOT ECX
0053B1B8 SUB EDI,ECX
0053B1BA DEC ECX
0053B1BB INC ESI
0053B1BC NOP
0053B1BD REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
0053B1BF JNZ SHORT dumped\_.0053B1CD
0053B1C1 MOV EAX,DWORD PTR DS:[53B160]
0053B1C6 MO DWORD PTR DS:[EAX+20],7F000 ;把 size 值改成原来的
0053B1CD POP EDX
0053B1CE POP ECX
0053B1CF POP EBX
0053B1D0 POP EDI
0053B1D1 POP ESI
0053B1D2 POP EAX
0053B1D3 RETN
```

以上代码的作用是 1. 完成 `FindFirstFileA` 的任务；2. 检查 `Find` 的文件是否为程序本身，若是，则把 `size` 值改过来，呵呵。

ok，把 `0040880F` 处的 `CALL <JMP.&kernel32.FindFirstFileA>` 改成 `call 0053B180` 再试试，all right！！！（啦啦啦...）

## 三、修改“注册信息”

既然都这样了，就再玩玩吧。每次启动那个未注册的信息的确让人想去改改。

`4dd466d0e1e0cc123afbe504da2462ca`
