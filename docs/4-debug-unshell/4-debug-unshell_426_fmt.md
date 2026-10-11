
# Ahaview2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50015.htm`](https://www.pediy.com/kssd/pediy05/pediy50015.htm)

ASPROtect 1.22 加壳的 ahaview2.0 脱壳

crack by fwnl

软件简介：AhaView 是一个功能单纯的看图软件，具有类似资源管理的树状目录分类，可以显示目录内所有图片的缩图，也可用全屏幕模式查看单张图片。

软件下载：`http://gwbn.onlinedown.net/down/ahaview.exe`

由于这个程序是用 Aspr1.22(FI 249 查的)加壳，所以 caspr 不能脱此壳，于是就只有手动来脱了。

## 一.找入口点

由于是 delphi 程序，所以利用快速寻找 aspr 入口的方法:

执行 ahaview2.0，用 `prodump` 选 `dump(full)` 脱壳,存为 `dump.exe`。接着用 UltraEdit32 打开 `dump.exe`，ALT+F3，填 `runtime`,选中查找 ASCⅡ,执行搜索，搜到后,向上查找离 `runtime` 最近的机器码为 `55 8B EC` 的地方就是程序的 oep，而在 AhaView.exe 里这个位置在 `offset:000E3978` 处，用 `peditor` 的 `flc` 功能将它转换为 virtual address 就是 `004e3978`----oep。

## 二.用 trw2000 初步脱壳

下面要在入口处脱壳，打开 SuperBPM，点 `erase`，用 `trw` 载入 `Any2Icon.exe`，下 `g 4e3978`，接着下 `pedump` 脱出程序。

## 三.修复 import table

打开原加壳程序,在 `Import REConstructor v1.4.2+` 的 `Attach to an Active Process` 窗口中选取 `ang2icon.exe` 的进程,然后在下方的 `oep` 处填入 `000e3978`,点 `IAI AutoSearch`,再点 `Get Imports` 然后点 `Show Invalid`,在 `Imported Functions Found` 窗口里的无效地址上点鼠标右键，选 `Trace Leve11(disasm)`,再点 `show invaids`，发现还有几个没有修复，再次在那几个没有修复的地址上点鼠标右键。

选中 `Plugin Tracer(Asprotect 1.2X Emul)`,再点 `show invaids` 应发现所有的 dll 显示 `valid:Yes` 了。然后点 `Fix Dump`,选中你用 `trw2000 pedump` 出的文件修复。

这样修复后的程序还是不能运行.于是我们要看程序在那里出错，然后再用原版的对比看。

```
0167:004E3978  PUSH    EBP                          //入口
0167:004E3979  MOV     EBP,ESP
0167:004E397B  ADD     ESP,BYTE -10
0167:004E397E  XOR     EAX,EAX
0167:004E3980  MOV     [EBP-10],EAX
0167:004E3983  MOV     EAX,004E3578
0167:004E3988  CALL    004065E4
0167:004E398D  XOR     EAX,EAX
0167:004E398F  PUSH    EBP
0167:004E3990  PUSH    DWORD 004E39F3
0167:004E3995  PUSH    DWORD [FS:EAX]
0167:004E3998  MOV     [FS:EAX],ESP
0167:004E399B  MOV     EAX,[004E89B0]
0167:004E39A0  MOV     EAX,[EAX]
0167:004E39A2  CALL    00451EBC
0167:004E39A7  MOV     EAX,[004E89B0]
0167:004E39AC  MOV     EAX,[EAX]
0167:004E39AE  MOV     EDX,004E3A08
0167:004E39B3  CALL    00451AC0
0167:004E39B8  LEA     EAX,[EBP-10]
0167:004E39BB  CALL    00453458
0167:004E39C0  MOV     EDX,[EBP-10]
0167:004E39C3  MOV     EAX,[004E89B0]
0167:004E39C8  MOV     EAX,[EAX]
0167:004E39CA  ADD     EAX,BYTE +40
0167:004E39CD  MOV     ECX,004E3A18
0167:004E39D2  CALL    00404014
0167:004E39D7  CALL    NEAR [004E8620] 这个 call 指向 cec8fb，F8 进去全是?号
0167:004E39DD  XOR     EAX,EAX
```

用原未脱壳的程序进入 `0167:004E39D7  CALL    NEAR [004E8620]` 发现到了下面这里:

```
0167:00CEC8FB  NOP
0167:00CEC8FC  CMP     DWORD [00CF35A8],BYTE +00
0167:00CEC903  JZ      00CEC90b
0167:00CEC905  CALL    NEAR[00CF35A8] //f8 进入
```

就来到了下面这里

```
0167:004E34F3  RET
0167:004E34F4  PUSH    EBX             //CALL    NEAR [004E8620]真正指向
0167:004E34F5  XOR     EBX,EBX
0167:004E34F7  CALL    00402A10
0167:004E34FC  TEST    EAX,EAX
```

于是知道脱出的程序少了 CXXXXX 到 CXXXXXX 这段。

我们首先 `d cec8fb`,然后 Alt+Page Up ,当看到数据框内如下显示

可知道这段数据从 `CE0100` 开始.

```
0030:00CE0FE0 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ??
0030:00CE0FF0 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ?? <---空地址了
0030:00CE1000 0A 06 73 74 72 69 6E 67-00 00 00 00 00 00 00 00 ..string........
0030:00CE1010 00 00 00 00 00 00 00 00-3C 10 C9 00 00 00 00 00 ........<.?....
0030:00CE1020 6D 11 C9 00 04 00 00 00-00 00 00 00 B0 29 C9 00 m.?........??
0030:00CE1030 98 28 C9 00 C0 28 C9 00-04 29 C9 00 11 00 0B 00 ????.)?....
0030:00CE1040 10 29 C9 00 04 46 72 65-65 13 00 1C 29 C9 00 0C .)?.Free...)?.
```

然后 Alt+Page Down ,当看到数据框内如下显示

可知道这段数据从 `cF8000` 结束.

```
0030:00CF7FD0 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0030:00CF7FE0 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0030:00CF7FF0 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0030:00CF8000 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ?? <---空地址了
0030:00CF8010 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ??
0030:00CF8020 ?? ?? ?? ?? ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ??
```

于是我们就在 `trw2000` 中 `w CE1000 CF8000 new.bin`.来保存出 `new.bin`

用 `winhex` 打开 `dump\_.exe` 和 `NEW.bin` 把 `NEW.bin` 的内容粘贴到 `dump\_.exe` 的最后，然后再用 `peditor` 载入 `dump\_.exe`,点 `sections`,然后右键点 `add a section` 添加一个新段名字为 `new`.然后选中 `new` 这段 section,右键点 `edit ection` 编辑它,由于 `CF8000-CE1000=17000`,所以在 Virtuel Siae 填入 `00017000`,Virtuel Offsize 填 `008E1000` ,因为(`cE1000-400000=8E1000`), Raw 填 `00017000`,然后点 `apply changes` 来保存修改。最后再点 `rebuilder`,去掉所有的勾，只保留 `make PE Header win nt/2k compatible` 这一项.然后点 `do` 这样就重建好的 Pe 头了.

呵呵，这样程序就可以运行了，好像没什么错误,脱壳后的东西不能用下面的这个 key 注册,这个东西的注册版我还是脱不了呀，各位大哥帮忙看看呀.谢谢了.

这个东西的注册 key: `0Klum1it1rCgBqNbtZwcvn8VzbIiGRUiN8sWOI9OP7GNX/SrjmJU0gvxCz ?? JRlA8eYj5ZTz+D/KPZI2lLKEd1oNV28TnfdgZUCOaNpgKtp/JVIQeaZ5Gs ?? 3gIHiOV6RjghDLecqi6PM5ECbzfl3sN5qO0AvxLrtOvB5lbzpP+UKlqA=`

fwnl

2002.3.24

长沙 4dd466d0e1e0cc123afbe504da2462ca
