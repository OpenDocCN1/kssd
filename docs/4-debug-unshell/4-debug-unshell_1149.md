
# vfp&exe 加密程序

> 原文：[`www.pediy.com/kssd/pediy03/forum1233.htm`](https://www.pediy.com/kssd/pediy03/forum1233.htm)

`vfp&exe` 加密程序破解实录

`vfp&exe.exe v1.8.5` 演示版

用 `vfp6` 编译一应用程序 `myvfp.exe`

运行 `vfp&exe` 对 `myvfp.exe` 进行加密

运行加密后的 `myvfp.exe`

出现输入密码窗口(如加密时选择了密码项)和一个演示版的提示窗口

去除 `vfp&exe` 加的壳：

用 `trw2000` 调入加密后的 `myvfp.exe`

下断点 `bpx 401760`(`vfp6` 编译的文件入口处)

`f5` 几次

出现输入密码窗口时输入密码

当出现演示版提示窗口时按确定，再按 `f5` 键

A！记下 `trw2000` 横线上的当前文件名

`suspend` 回到 `windows`

运行 `ollydbg`

选 `file` 菜单下的 `attach` 子菜单

在弹出的窗口中选 A！处记下的文件名后击 `attach` 钮

右击左上方窗口

选 `go to --> address` 输入 `401760` 击 `ok` 钮

右击弹出的窗口

选 `copy to exxcutable file`

右击弹出的窗口

选 `backup-->save data to file`

选一个存放文件的路径和文件名(`unmyvfp.exe`)

击打开钮

运行 `prodump.exe`

选 菜单编辑 `pe` 文件

将文件 `unmyvfp.exe` 的入口地址由 `48e8` 改为 `1760` 后存盘退出

(或者

运行 `winhex` 改文件入口偏移

调入 `unmyvfp.exe`

取地址 `3CH` 处的值，为 `B8H`，计算 `B8H+28H=E0H`

改 `E0H` 处的 `E848H` 为 `6017`------程序入口处)

运行 `winhex`

寻找 `00FEF2`

删除第二次找到的地方的 `FF` 处开始到文件结束的所有数据后存盘退出

用 `unfoxall` 对 `unmyvfp.exe` 就可进行反编译了！！！

此法完全用于学习，不得用于非法用途！

好了搞定这个东西耗费了我两天的时间,眼睛受不了了.....

lpyxt 写于 2001 年 8 月 16 日 17 时 20 分

`email:liupeiyuxt@km169.net`

首先，我们来看看 `vfp&exe` 对 `VFP` 文件的加密：

1、将 `VFP` 的 `EXE` 文件看做一个普通的 `PE` 文件看待，在加密后程序运行后自身解密，然后 `SHELL` 这个文件，通常保存在 `WINDOWS` 一些特殊的目录里面，如垃圾桶、`INF` 等。

2、为防止别人拷贝这个文件，在运行这个文件的时候，对文件的存取做了限制；

3、为防止通过 `FILEMON` 查看到自身还原的程序藏身之处后将文件拷贝，

会自动终止 `WINDOWS` 自带“查找”；

4、加有反 `unvfp&exe` 的代码“☆ 文件型...终结者”、`UNVFP&EXE`...当发现前者自动弹出输入密码的窗口后停滞不前；发现后者则以“文件不能锁定”自动退出。

5、针对 `APP` 文件的结构作了一些小动作，将文件列表的标识符放到错误的位置，`UNFOXALL` 不能正确识别；

6、随机加入垃圾信息，使得文件在文件列表的偏移发生变化，`UNFOXALL` 不能正确识别；

7、加入一个 `APP` 文件，使主 `APP` 文件入口及调用产生偏移，`UNFOXALL` 不能正确识别。

`LPYXT` 朋友的方法只是做到了两样：

A、利用 `OLLYDBG` 找到自身还原的程序；

B、去掉使主 `APP` 文件入口及调用产生偏移的 `APP` 信息。

这种方法的局限性很大，从理论上使得对付很多文件都不可行。

其实想找到自身还原的程序并不需要 `TRW2000`、`OLLYDBG` 等，您只要

用一个 `FILEMON` 记下加密程序自身还原后的存放位置，狠心关机，把它拷贝出来就是了。

`LPYXT` 朋友，你不妨用您的方法试一下：

`http://www.javasoft.com.cn`

上面的 `Foxmis`。

\== 欢迎光临品琳居 ==

永远的朋友：`Aming`   `4dd466d0e1e0cc123afbe504da2462ca`
