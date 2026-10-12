
![](img/cec3f7662d28678567385a3a7f77b7de.jpg)  

这是注册的主窗口。

对照 `LIDA` 中的反汇编代码加上 `GTK` 的一些基本知识，我们可以知道程序运行过程是这样的：`main`->跳出 `killme` 窗口->注册 `destroy` 事件处理函数->在关闭 `killme` 窗口时将引发 `destroy` 事件->运行回调处理函数 `0x80489bd`->跳出 `crackme` 窗口。

我们 `CRACK` 的任务就是 `nop` 掉 `killme` 窗口，以及找出注册码。

注：`GTK` 的相关文档你可以到这来获取 `http://www.gtk.org/tutorial/`

## 第一个任务：去除 killme 窗口。

用 `lida` 反汇编 `cm2`，点击菜单 `view` -> `Functions` 找到 `main` 函数，点击进入，如下所示：

```
代码:

* * *

main():
08048970  55                                   push      ebp
08048971  89 E5                                mov      ebp, esp
08048973  81 EC 08 00 00 00                    sub      esp, 0x8
08048979  68 00 00 00 00                       push      0x0
0804897E  68 03 00 00 00                       push      0x3
08048983  E8 88 FE FF FF                       call      gtk_init (08048810) ; (near - 0x178)
08048988  90                                   nop  -\\
08048989  90                                   nop    \\
0804898A  90                                   nop     |->怎么会有 nop 指令出现？
0804898B  90                                   nop    /   我们在这加入 call Function___080489BD
0804898C  90                                   nop  -/    直接显示 crackme 窗口，跳过 killme 窗口的调用。因为在
0804898D  BE D2 A0 04 08                       mov      esi, " Kill this window (p" (0804A0D2)
08048992  E8 5C 04 00 00                       call      Function___08048DF3 (08048DF3) ; 建立 killme 窗口
08048997  68 00 00 00 00                       push      0x0
0804899C  68 BD 89 04 08                       push      Function___080489BD (80489BD)  ; 回调函数，显示 crackme 窗口
080489A1  68 28 A2 04 08                       push      "destroy" (0804A228)
080489A6  FF 35 A0 A3 04 08                    push      0804A3A0
080489AC  E8 CF FE FF FF                       call      gtk_signal_connect (08048880) ; 建立 destroy 事件回调函数 0x080489BD
080489B1  81 C4 10 00 00 00                    add      esp, 0x10
080489B7  E8 84 FE FF FF                       call      gtk_main (08048840) ; 显示 killme 窗口
080489BC  C3                                   ret

* * *

```

我们跟入 `Function___08048DF3` 函数，如下：

```
代码:

* * *

Function___08048DF3:
08048DF3  55                                   push      ebp  ; xref ( 08048992 08048C9A 08048CA6 08048DAC )
08048DF4  89 E5                                mov      ebp, esp
08048DF6  81 EC 08 00 00 00                    sub      esp, 0x8
08048DFC  68 01 00 00 00                       push      0x1
08048E01  E8 BA F9 FF FF                       call      gtk_window_new (080487C0) ; (near - 0x646)
08048E06  A3 A0 A3 04 08                       mov      0x0804A3A0, eax
08048E0B  68 2C A0 04 08                       push      "CrackMe v13 (2 linux" (0804A02C)
08048E10  FF 35 A0 A3 04 08                    push      0804A3A0
08048E16  E8 C5 F9 FF FF                       call      gtk_window_set_title (080487E0) ; (near - 0x63B)
08048E1B  56                                   push      esi
08048E1C  E8 8F F9 FF FF                       call      gtk_label_new (080487B0) ; (near - 0x671)
08048E21  50                                   push      eax
08048E22  FF 35 A0 A3 04 08                    push      0804A3A0
08048E28  E8 63 F9 FF FF                       call      gtk_container_add (08048790) ; (near - 0x69D)
08048E2D  68 64 00 00 00                       push      0x64
08048E32  68 2C 01 00 00                       push      0x12C
08048E37  FF 35 A0 A3 04 08                    push      0804A3A0
08048E3D  E8 5E F9 FF FF                       call      gtk_window_set_default_size (080487A0) ; (near - 0x6A2)
08048E42  FF 35 A0 A3 04 08                    push      0804A3A0
08048E48  E8 33 F9 FF FF                       call      gtk_widget_show_all (08048780) ; (near - 0x6CD)
08048E4D  C9                                   leave
08048E4E  C3                                   ret

* * *

```

上面这个函数建立 `killme` 窗口。

为了去除这个 `killme` 窗口。我们可以在

```
代码:

* * *

08048988  90                                   nop  -\\
08048989  90                                   nop    \\
0804898A  90                                   nop     |->怎么会有 nop 指令出现？
0804898B  90                                   nop    /   我们在这加入 call Function___080489BD
0804898C  90                                   nop  -/    直接显示 crackme 窗口，跳过 killme 窗口的调用。
0804898D  BE D2 A0 04 08                       mov      esi, " Kill this window (p" (0804A0D2)
08048992  E8 5C 04 00 00                       call      Function___08048DF3 (08048DF3)

* * *

```

上面的 `nop` 正好 5 个，我怀疑作者写 `crackme` 在这有一个对 `crackme` 函数的调用，生成程序后，他手工把这个代码 `NOP` 掉了，他是故意留出空间来做提示。因为在 `crackme` 窗口函数中有完整的窗口初始化和退出处理，如下：

```
代码:

* * *

08048C22  68 3D A2 04 08                       push      "clicked" (0804A23D)
08048C27  FF 35 AC A3 04 08                    push      0804A3AC
08048C2D  E8 4E FC FF FF                       call      gtk_signal_connect (08048880) ; (near - 0x3B2)
08048C32  81 C4 10 00 00 00                    add      esp, 0x10
08048C38  E8 03 FC FF FF                       call      gtk_main (08048840) ; (near - 0x3FD)
08048C3D  68 2C A0 04 08                       push      "CrackMe v13 (2 linux" (0804A02C)
08048C42  E8 D9 FB FF FF                       call      g_print (08048820) ; (near - 0x427)
08048C47  81 C4 04 00 00 00                    add      esp, 0x4
08048C4D  68 00 00 00 00                       push      0x0
08048C52  E8 09 FC FF FF                       call      exit (08048860) ; 当点击 crackme 窗口上的 quit 按钮后程序直接退出

Function___08048C57:
08048C57  55                                   push      ebp
08048C58  89 E5                                mov      ebp, esp
08048C5A  81 EC 08 00 00 00                    sub      esp, 0x8
08048C60  E8 17 00 00 00                       call      Function___08048C7C (08048C7C) ; (near + 0x17)
08048C65  B8 00 00 00 00                       mov      eax, 0x0
08048C6A  C9                                   leave
08048C6B  C3                                   ret

* * *

```

从上面的代码可以看到，如果先调用 `crackme` 窗口的函数，那么 `killme` 窗口就不会出现了。

好的，下面我们就改 `08048988` 处的指令为 `call    080489BD`，用计算器算一下:`80489BD-08048988-5=00000030`，得到代码为 `e8 30 00 00 00`。嘿嘿！

好的，用 `hexedit` 打开 `cm2` 程序。看看前面 `objdump -x cm2` 列出的基地址为 `0x8048000`，所以对应的 `file offset` 应该是 `0x8048988-0x8048000=0x988`，我们按 `return` 键，打开 `go to` 窗口，输入 `0x988`，定位到 `08048988` 处，改代码为 `e830000000`。改完后，“`./cm2`”运行程序，OK。出现 `crackme` 窗口了。

## 第二个任务，找出正确的注册码。

为保持本文一个苗条的身材及加大本文所包含的信息量，我就不讲怎么根据按钮名找对应的点击事件处理函数了，讲一种简单省事的方法，找注册码最省事的方法当然是动态跟踪啦。搞不好还可以看到明码哟。费话不多说，运行命令：`gdb cm2`。提示没有符号文件，不用管它。

下断点:

```
代码:

* * *

br gtk_entry_get_text 注（类似于 WIN 平台下的 getdlgitemtexta）

* * *

```

再键入命令 `" run "`运行程序。出现了界面。

输入名字和注册码，我这用的是:

名字：`ncc2008`

注册码：`78787878`

按 `crackme` 按钮，中断到 `gdb` 中。

```
代码:

* * *

Breakpoint 1, 0x40096eb0 in gtk_entry_get_text () from /usr/lib/libgtk-1.2.so.0

* * *

```

让我们看看是哪个 `CALL` 调用的，下命令"`info frame`" ，显示如下：

```
代码:

* * *

(gdb) info frame
Stack level 0, frame at 0xbfffe250:
 eip = 0x40096eb0 in gtk_entry_get_text; saved eip 0x8048cc1  ；从这 CALL 来的
 called by frame at 0xbfffe254
 Arglist at 0xbfffe248, args:
 Locals at 0xbfffe248, Previous frame's sp is 0xbfffe250
 Saved registers:
  eip at 0xbfffe24c

* * *

```

呵呵，清除全部断点，下命令“`d`”。在 `0x8048cc1` 下一个断点：

```
代码:

* * *

br *0x8048cc1

* * *

```

然后键“`C`”继续运行程序。

程序中断到我们所下的断点 `0x8048cc1` 处。让我们看看汇编代码：

下命令：`x /10i $eip`

显示如下：

```
