
![](https://bbs.kanxue.com/upload/attach/201504/142288_ljtc2hhfnmmi6vy.png)

看看，我们的两个`NOP`消失了，原始的代码又回来了（不过变成了灰色）。我们的补丁被回收了！现在回到Patch窗口：(p24)

![](https://bbs.kanxue.com/upload/attach/201504/142288_dpxtxsvuvm26sfc.png)

注意那个地址不是红色的了，State也变成了"Removed"。Olly已经禁用了我们的补丁，并且在我们每一次重启程序时都会这么做。我们想要做的就是让这个补丁永远有效，而不用每一次都激活它。

为了让我们的补丁能够长久有效，我们必须将修改的版本保存到磁盘。首先，选中补丁再按下空格键以重新启用补丁。那个`JNZ`指令就会变回到我们的`NOP`，那两个`NOP`也会以红色字体重新出现在反汇编窗口。现在，在反汇编窗口的任何地方右键，选择"Copy to executable"->"All modifications"：(p25)

![](https://bbs.kanxue.com/upload/attach/201504/142288_8tvwvrcgfy5ube7.png)

如果弹出窗口问你是否要保存所有的修改，你就选"Copy All"：(p26)

![](https://bbs.kanxue.com/upload/attach/201504/142288_n8c6u1235x1dtne.png)

当你打了多个补丁，并且想一次性全部保存的话，这么做很重要。因为有时候，你很容易就忘记你打过多个补丁。本例中，即使我们只打了一个补丁，选择所有的补丁也只会保存这一个。当然，只有在Patch窗口中被激活的补丁才会被保存。

后面，你可能想要选择"Selection"而不是"All Modifications"，但是你必须保证你在反汇编窗口所做的修改是高亮显示（通过点击或拖拽以选中所有修改的行）。如果你选中的行比修改过的行要多也行，因为Olly只会更改已经修改的行。

在点击"Copy All"以后会打开一个新窗口，里面基本上是整个进程的数据，不过我们的补丁也在里面：(p27)

![](https://bbs.kanxue.com/upload/attach/201504/142288_aqjjb1l1u1lojdk.png)

在顶部你可以看到我们的补丁。但要意识到这个只是在内存中的修订版本，还没有保存到磁盘呢。不过，如果你关了这个窗口或重启了程序，它是不会被保存的！咱们来保存好它：右键新窗口的任意位置，选择"save file"。这会将该进程的内存空间数据保存到一个文件中。一个另存为对话框会显示出来。将文件另存为`Crackme2_patched`（我通常在后门加一个`_patched`用来区分，你也可以加任何你喜欢的）：(p28)

![](https://bbs.kanxue.com/upload/attach/201504/142288_no8s9dm58u5eyaq.png)

我们现在有了一个crackme的打补丁版本。咱们来试试看。在Olly中打开这个新文件（打过补丁的）。按下`Ctrl+G`或点击`GOTO`图标，输入我们打过补丁的地址：(p29)

![](https://bbs.kanxue.com/upload/attach/201504/142288_zlfr01treavhz7i.png)

看看咱们的补丁：(p30)

![](https://bbs.kanxue.com/upload/attach/201504/142288_p9zhyns5x9esjaz.png)

yes，补丁还在那。现在运行程序，输入`info`和`viola`（译者注：info和viola是作者用了当用户名和注册码的输入）：(p31)

![](https://bbs.kanxue.com/upload/attach/201504/142288_7gezc4gas64jcog.png)

现在，我们有了我们第一个破解过并打过补丁的二进制文件:0。

## 五、作业

本章的作业很简单（只要你一直在学习汇编语言）。

思考题：你可以将`4010E9`处的“`TEST EAX, EAX`”修改成什么，来防止跳转到显示bad boy处？

要注意的是无论你将`TEST`指令修改成什么，都不能超过2字节，那是`TEST EAX, EAX`指令的长度。如果你打了一个长点的补丁，就会覆盖掉`JNZ`指令后面的指令......

ps.如果你需要提示的话，请点[这里](https://bbs.kanxue.com/elink@a4fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5k6h3I4W2k6$3g2F1k6r3!0X3M7X3q4F1k6r3!0E0i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8X3S2A6L8Y4b7$3)。不过你应该真正意义上自己试着做。那是学习的最好方法！

本文PDF文件下载（已排版）：

[第六章：第一次（真正的）破解.pdf](https://bbs.kanxue.com/attach-download-97338.htm)

本文相关附件下载地址（国外链接，不是一直好用）：

[包含`Crackme2`、`MnemonicHelp.dll`插件、`X86`指令帮助文件、英文原文文档、英文视频教程](https://bbs.kanxue.com/elink@bc6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8Z5k6h3I4W2k6$3g2F1k6r3!0X3M7X3q4F1k6r3!0E0i4K6u0W2j5$3!0E0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6@1N6i4c8K6i4K6u0r3f1U0c8F1k6r3!0E0i4K6g2X3N6s2g2@1L8%4u0A6j5h3I4Q4y4h3j5$3i4K6u0W2P5X3W2H3)

吐个槽：咱坛的编辑功能好弱，尤其是图片，极其不方便，容易丢失

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [第六章：第一次（真正的）破解.pdf](javascript:void(0);)

  （1.11MB，403次下载）

- [图片1.png](javascript:void(0);)

  （8.56kb，3次下载）

- [图片2.png](javascript:void(0);)

  （57.43kb，3次下载）

- [图片3.png](javascript:void(0);)

  （7.81kb，3次下载）

- [图片4.png](javascript:void(0);)

  （12.29kb，3次下载）

- [图片5.png](javascript:void(0);)

  （34.92kb，4次下载）

- [图片6.png](javascript:void(0);)

  （1.33kb，3次下载）

- [图片7.png](javascript:void(0);)

  （2.97kb，4次下载）

- [图片8.png](javascript:void(0);)

  （25.98kb，4次下载）

- [图片9.png](javascript:void(0);)

  （10.31kb，4次下载）

- [图片10.png](javascript:void(0);)

  （10.69kb，3次下载）

- [图片11.png](javascript:void(0);)

  （14.95kb，3次下载）

- [图片12.png](javascript:void(0);)

  （34.77kb，4次下载）

- [图片13.png](javascript:void(0);)

  （1.34kb，3次下载）

- [图片14.png](javascript:void(0);)

  （6.46kb，4次下载）

- [图片15.png](javascript:void(0);)

  （0.75kb，3次下载）

- [图片16.png](javascript:void(0);)

  （7.88kb，3次下载）

- [图片17.png](javascript:void(0);)

  （7.81kb，3次下载）

- [图片18.png](javascript:void(0);)

  （7.08kb，3次下载）

- [图片19.png](javascript:void(0);)

  （4.27kb，3次下载）

- [图片20.png](javascript:void(0);)

  （7.88kb，3次下载）

- [图片21.png](javascript:void(0);)

  （9.59kb，4次下载）

- [图片22.png](javascript:void(0);)

  （8.98kb，3次下载）

- [图片23.png](javascript:void(0);)

  （3.14kb，3次下载）

- [图片24.png](javascript:void(0);)

  （9.16kb，3次下载）

- [图片25.png](javascript:void(0);)

  （16.02kb，3次下载）

- [图片26.png](javascript:void(0);)

  （6.73kb，3次下载）

- [图片27.png](javascript:void(0);)

  （17.08kb，4次下载）

- [图片28.png](javascript:void(0);)

  （23.21kb，4次下载）

- [图片29.png](javascript:void(0);)

  （6.06kb，3次下载）

- [图片30.png](javascript:void(0);)

  （3.20kb，3次下载）

- [图片31.png](javascript:void(0);)

  （7.88kb，3次下载）

- [legend.jpg](javascript:void(0);)

  （49.94kb，3次下载）
