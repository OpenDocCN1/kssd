
# VMware 配置 SoftICE

> 原文：[`www.pediy.com/kssd/pediy12/127513.html`](https://www.pediy.com/kssd/pediy12/127513.html)

给还不能正常使用 SoftICE 的新手看的，高手就可以跳过了。

按照网上的教程在 VMware 里面安装完 `driverstudio3.2` 后，`net start ntice` 启动不了，设置 `sc config ntice start= auto` 就可以启动了。

可是按下 `Ctrl+D` 键盘鼠标都没反应了，跟死机了差不多。

后来试了一下，先打开 `cmd`，然后按 `Alt+Enter` 全屏，然后 `Ctrl+D` 终于可以见到 SoftICE 了，可是这样很不方便。

然后在 `.vmx` 文件后面加上：

```
svga.maxFullscreenRefreshTick = 5
vmmouse.present = "FALSE"
```

结果还是不行（这两句还是要的），上网查了很多资料都没有解决，大概知道了跟显卡有关，换显卡是不现实的。

所以在仔细看了看 VM 的配置选项，反正只有一个选项，VM 按下图配置即可：

![](img/12da9352fd2f0795767e921590f104f3.jpg)

或者也可以修改 `.vmx` 文件，在最后面加上：

```
svga.autodetect = "FALSE"
svga.maxWidth = "1280"
svga.maxHeight = "800"
svga.vramSize = "134217728"
```

再点 `Ctrl+D` 就出来了：

![](img/8536fd9a7973b81794067bfef3cd7f8a.jpg)

最主要是 `svga.autodetect = "FALSE"` 这一句，其它的像分辨率尽量高吧，太低了会有黑边。

简单归纳一下 SoftICE 的使用吧。

## 准备工作

打开 `symbol Loader`，在 `Exports` 选项里面添加 dll 文件。

为什么要这样呢，比如你要在 `AppendMenuA` 下断点，`AppendMenuA` 是在 `user32.dll` 里面的，那么你要在 `Exports` 选项里面添加 `user32.dll`，然后 SoftICE 会加载 `user32.dll` 的所有函数，否则 `bpx AppendMenuA` 会提示找不到的。又如果要在 MFC 的函数下断点，一般要加载 `mfcxx.dll`（xx 是版本，一般是 `mfc42.dll`）。

下面开始了，可以参考这个例子 `http://www.pediy.com/tutorial/chap3/Chap3-4.htm`。

## 使用步骤

1. 打开要调试的程序，`Ctrl+D` 调出 SoftICE，如果 SoftICE 太小可以把字体改大一点 `set font 3`。

2. 然后当然是下断点了，有 IO 断点等等一些命令，按 `F1` 可以看帮助。

   一般还是用 `bpx`，这个跟 `OD` 是一样的，大家应该知道，比如 `bpx GetDlgItemText`。

   - `bl` 可以查看断点列表，就是 `break list`，显示的断点有编号。
   - `bd` 可以禁用断点，就是 `break disable`，后面接编号，比如 `bd 3`（禁用编号为 3 的断点），`bd *`（禁用所有断点）。
   - `be` 可以启用断点，就是 `break enable`，用法跟 `bd` 一样。
   - `bc` 可以清除断点，就是 `break clear`，用法也跟 `bd` 一样。
   - `bpe` 编辑断点，就是 `break point edit`，用法也跟 `bd` 一样。

   这个 `bpx` 跟 `OD` 最大的区别就是这个是断在系统模块里面的，所以要按 `F12` 或 `F11` 返回程序，而且不管被调试的程序做了什么样的变形系统模块总是没变的，调用 API 的时候总是能断下来的，所以可以带壳调试。

3. 然后是常用控制方法。

   下好断点后只要程序调用了被断的 API 这时 SoftICE 就会自己跳出来：

   - `F8` 是单步步入，相当于 `OD` 的 `F7`。
   - `F10` 是单步步过，相当于 `OD` 的 `F8`。
   - `F12` 是从 call 中返回。
   - `F7` 是运行到光标处，相当于 `OD` 的 `F4`。

   那么 `OD` 里面还有个 `F9` 是运行程序，SoftICE 里面是什么呢？没有！

   SoftICE 的界面出来的时候除了 SoftICE 以外所有程序都暂停运行，连系统时钟都不动了。再次按下 `Ctrl+D` 或者 `F5` 隐藏 SoftICE 的时候其它程序又可以运行了，所以要运行程序的时候按下 `Ctrl+D` 就可以了，遇到断点 SoftICE 自己会弹出来。

   如果只是想看一下被调试程序的界面，不想让它运行，可以按 `F4` 隐藏 SoftICE 的界面，再次按 `F4` 又出来了。

4. 其它命令。

   - `D` 显示内存地址内容，就是 data 的首字母缩写，比如 `d eax`（显示 `eax` 的地址的内容），`d 12fe23`（显示 `0x0012fe23` 处的内容）。
   - `E` 修改内存单元，就是 edit 的首字母缩写，跟 `D` 用法一样。
   - `R` 修改寄存器的值，就是 register 的首字母缩写，比如 `recx`（修改 `ecx` 的值），`r fl`（修改标志位）。

   如果玩过 DOS 汇编的话这几个命令应该很熟，`cmd` 里面的 `debug` 就是这样的。

   修改寄存器其实用鼠标操作也很方便，直接用鼠标点要编辑的寄存器然后编辑就可以了。

   比如我们要修改一个 `jz` 的跳转，可以修改 0 标志位，直接点击标志位 `Z`，亮了 `zf` 就是 1，再点一下 `zf` 就是 0。

   - `?` 计算表达式的值，相当于 `OD` 的 `calc`。

   有了这些基本上可以使用 SoftICE 了，其它的命令可以自己去查，SoftICE 我也是第一次用。
