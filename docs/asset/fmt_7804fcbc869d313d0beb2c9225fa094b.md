
# Clipmate 5.3.05 Build 268

> 原文：[`www.pediy.com/kssd/pediy04/kanxue482.htm`](https://www.pediy.com/kssd/pediy04/kanxue482.htm)

软件名称：`Clipmate 5.3.05 Build 268`

下载地址：`www.thornsoft.com`

破解工具：`ollydbg 1.06`

破解方法：快速暴破法

声明：

此文仅用于学习之用，转载请注明出处。本人对读者阅读本文之后的行为不负任何责任。

请尊重软件开发者的权利，如果你真的喜欢一个软件，请支持它！

破解过程：

1. 打开 `ollydbg`，`open clipmate`，按 `F9` 运行
2. 进入软件注册画面，输入用户名和任意注册码，这儿设为 `name：Slimboy`，`s/n：77777777777777`。
3. 按 `F12` 暂停，进入 `CPU` 窗口
4. 在右下角 `Stack` 窗处，右键单击最顶端的一行数字，选 `Follow in Dump`，进入相应的内存段。
5. 在 `Dump` 窗（即内存段）右键单击，选 `Search`，在 `ascii` 处填 `Slimboy`，然后 `search`
6. 在高亮显示的 `Slimboy` 处，右键单击选 `breakpoint－on memory access`
7. 按 `F9` 继续运行 `clipmate`，然后重复第一步
8. 你会发现 `ollydbg` 自动暂停了程序，查看 `Stack` 窗，你见到了什么？你的注册码，应该是 `G013668254526`
9. `remove all breakpoint`，然后按 `ALT＋F12` 退出，这很关键，否则你会死机的，不过即使是 `ALT＋F12` 在某些机器上也会死机

破解总结：

依靠 `ollydbg` 的强大功能，这样简单的破解不到半分钟，而且简单地甚至不知道什么叫汇编的人也能快速掌握。特别适合 `newbie` 用，不过不是所有的软件都可以这样解决，我在这儿也是提供了一种方法而已。

by `slimboy`

`find some trouble` ？`mail me`，`slimboy@kali.com.cn` `4dd466d0e1e0cc123afbe504da2462ca`
