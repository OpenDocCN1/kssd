
# 调试日志与版权声明

## 调试过程记录

第二处强力爆破后，启动 `Nag` 一起搞调了，原来它们是调用的同一个语句，程序帮我们 `Kill` 掉了(自动关闭)

最后请看胜利截图。

\--------------------------------------------------------------------------------

### 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

本代码的着色效果由 `xTiNt` 自动完成

下载 `xTiNt` [`211.90.75.84/web/kanaun/download/xTiNt.rar`](http://211.90.75.84/web/kanaun/download/xTiNt.rar)

## 社区讨论引用

> 引用:
>
> * * *
>
> *最初由 `ljy3282393` 发布*
>
> **二哥：下 `bp DestroyWindow` 断点后，应该按多少次 `Ctrl+F9` ?为什么?**
>
> * * *

## 断点设置与指令分析

### 断点设置

退出断点下

```
bp DestroyWindow
```

`DestroyWindow` 用途是

破坏（即清除）指定的窗口以及它的所有子窗口（在 `vb` 里使用：用处不大。原文：it is unlikely to be of much use.）一般程序退出时调用或关闭窗口时调用。

点不看了按钮,立即中断。

### 汇编指令分析

```
77DF736E > B8 59110000 MOV EAX,1159 //这里中断。
77DF7373 8D5424 04 LEA EDX,DWORD PTR SS:[ESP+4]
77DF7377 CD 2E INT 2E //如果返回看到这样的语句，必须 Nop 掉，不然 Over
77DF7379 C2 0400 RETN 4
```

### 堆栈友好提示

```
0012F9DC 00445769 /CALL 到 DestroyWindow 来自 Crack.00445764 //这里清除断点,Ctrl+F9 返回。
0012F9E0 001104A0 \hWnd = 001104A0 ('计算机资料大全 v3.0 火力加强版',class='TMessageForm',parent=000404BC)
0012F9E4 0012FBCC 指针到下一个 SEH 记录
0012F9E8 00445789 SE 句柄
0012F9EC 0012F9F4
```

### 代码流跟踪

```
00445769 . 85C0 TEST EAX,EAX
0044576B . 75 05 JNZ SHORT Crack.00445772
0044576D . E8 A283FCFF CALL Crack.0040DB14
00445772 > 33C0 XOR EAX,EAX
00445774 . 5A POP EDX
00445775 . 59 POP ECX
00445776 . 59 POP ECX
00445777 . 64:8910 MOV DWORD PTR FS:[EAX],EDX
0044577A . 68 90574400 PUSH Crack.00445790
0044577F > 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]
00445782 . 66:8160 54 FF>AND WORD PTR DS:[EAX+54],0FDFF
00445788 . C3 RETN
```

不断 `Ctrl+F9` 返回。

### 深层 API 调用

```
0043AFCF |> \8B45 F4 MOV EAX,DWORD PTR SS:[EBP-C]
0043AFD2 |. 8B10 MOV EDX,DWORD PTR DS:[EAX]
0043AFD4 |. FF92 E8000000 CALL DWORD PTR DS:[EDX+E8] //F7 进入。
```

以到这里为主

`0043AFCF`

这个是调试发现的，你将深入到一些 `Api`，可以根据需要调试下去。

`4dd466d0e1e0cc123afbe504da2462ca`
