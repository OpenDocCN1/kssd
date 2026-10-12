
# 被 CodeVeil 加密的 .net 程序的 dump 与修复

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-67.htm`](https://www.pediy.com/kssd/pediy09/pediy09-67.htm)
>
> 2007 新年，写此文和大家共同学习，并希望更多朋友一同研究 .net 软件安全课题。这次研究的对象是 MuvAudio，这是一个 .net 软件，应该用得是 CodeVeil 加密，本文里涉及的内容包括脱壳，手动修复，后续工作留给读者了（定位关键代码，爆破+Keygen，因为有网络验证）。
>
> 软件的下载地址是：http://www.muvaudio.com/。废话不多说，下面开始。
>
> 拿到手先运行，看一下注册验证的方式，是输入用户名和激活码。用户输入错误时，会有提示。用 Reflector 载入，显示没有 CLR 头，是个本地软件。

![`bbs.pediy.com/upload/2006/8/image/1.jpg_434.jpg`](img/93670692429d055c9461b9f08a6547e0.jpg)

用 Peid 查壳是 UPolyX，记得以前看 CodeVeil 加密的软件也是显示这个壳，这个软件也是。我在《加密与解密》第 3 版中的 .net 章节里详细描述了 CodeVeil 的特点（当然，目前还没出版），这里简单说一下。CodeVeil 用本地代码对 .net 的启动和 JIT 进行了包装，不过它最大的弱点是整体加密，整体解密，也就是说，在某一时刻内存中存在全部解密完成的 IL 代码，这便是我们 dump 的时机。因此第一步，便是 dump。

运行 MuvAudio，随后用 OllyDbg 进行 Attach，下断点在 `7906E7F4`，这是 .net framework 2.0 中 JIT 函数的地址。随后点击任一个尚未运行过的方法，这里我们点击注册码输入窗口的 `Activate` 按钮。中断后，就可以进行 dump 了。

![`bbs.pediy.com/upload/2006/8/image/2.jpg_515.jpg`](img/11292d18edb6f659de8f015f5ff0af85.jpg)

不了解为什么下中断在 `7906E7F4` 的请参考我以前的文章，也可以进行验证，验证的方法如下：注意中断时椎栈中的第四个参数：`0013E648`，显示该处数据如下，其中黄色高亮的值代表该方法的 IL 代码和大小。

![`bbs.pediy.com/upload/2006/8/image/3.jpg_528.jpg`](img/2766194622da63924c0bb399bf527530.jpg)

`11010FFC` 指向原文件的 `.text` 节，将此时的数据与原文章处的数据对比，发现数据已经改变，这便是解密过的 IL 代码。扯远了，这些问题都在以前的文章中讲过，大家自己看吧。下面准备 dump。

运行 NTCore 小组的 Explorer Suite 软件中的 Task Explorer，选择 MuvAudio 后 dump PE，保存为 `MuvAudio2_Dumped.exe`。再试一下用 `ildasm`（或者 Reflector）载入这个 PE 文件，仍然显示没有 CLR 头。当然，dump 的 PE 仍然用的原被加壳 PE 的头，那个头是本地的，因此需要修复。在《加密与解密》第 3 版中我介绍了两种半自动的方法，不过有时不成功，这里介绍全手动修复过程，在工具修复失败时，手动是最保险的。这里要求对 .net 的 PE 结构有一定了解。（OllyDbg 的任务也告一段落，可以关闭了。）

![`bbs.pediy.com/upload/2006/8/image/4.jpg_537.jpg`](img/c289a153637f74a4eb0c2dbd77c8d9a3.jpg)

首先修复的是数据目录中的项，CLR 头的具体结构就不再赘述，改为偏移为 `0x2008`，大小为 `48` 即可。

![`bbs.pediy.com/upload/2006/8/image/5.jpg_552.jpg`](img/f53edf4cecc720489bd5017dfd952af2.jpg)

下面是输入表的修正。大家知道 .net 可执行 exe 中本地引入表只引入了 `mscoree.dll` 中的 `_CorExeMain`，修复此项时可以参考别的 .net Exe 文件的数据值，主要是数据目录中的 Import Table 和 IAT 项。

此时的 IAT 为 `0x2000`，大小为 `8`，这是正确的数值，指向了 `.text` 的最初始 `8` 个字节，不过该处的数据是错误的。

![`bbs.pediy.com/upload/2006/8/image/6.jpg_563.jpg`](img/069bf64881ca312d9620b185913a815f.jpg)

应该修改为如下所示：

![`bbs.pediy.com/upload/2006/8/image/7.jpg`](img/30df9f9db3945a678aaabe5b8d462d6b.jpg)

`0019EBC0` 指向哪儿呢？应该指向 `_CorExeMain` 的前两个字节处，在 `MuvAudio2_Dumped.exe` 中来到 `0019EBC0` 处，正好指向了 `_CorExeMain` 字符串。

下面是数据目录中的 Import Table 项，`0019EB90`，大小大一点无所谓，就 `60` 吧。这里的引入表结构与 win32 下完全相同，不清楚的请查阅相关资料。

![`bbs.pediy.com/upload/2006/8/image/8.jpg_580.jpg`](img/a53adb404c679ed7b3db86ba14424dce.jpg)

![`bbs.pediy.com/upload/2006/8/image/9.jpg_588.jpg`](img/e282d36f4513ed6fb4c9f4db9b299fff.jpg)

到这里，修正完毕。再用 Reflector 或者 `ildasm` 载入一下，OK，可以正常反编译了。下面的工作大家都知道，搜索关键代码，写出 keygen 或者进行 patch。

![`bbs.pediy.com/upload/2006/8/image/10.jpg`](img/c686a0aa389b3a73153ebbdf73049804.jpg)

不过这时的程序不能正常运行，要运行很简单，用 `ildasm` 编译一下，再用 `ilasm` 编译一下既可。附件中提供刚刚 dump 和文件，修正过的文件，以及最终用 `ilasm` 重新编译后的可执行文件，大家可自行对比。

在进行任何代码修改时，不要忘了将强名称的代码删除掉。

![`bbs.pediy.com/upload/2006/8/image/11.jpg_610.jpg`](img/ad29636e3d0a9c5d2d5bbfa60e00b84c.jpg)

```
4dd466d0e1e0cc123afbe504da2462ca
```
