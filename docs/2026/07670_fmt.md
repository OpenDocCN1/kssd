
# [原创]向节表空白区添加代码

菜鸟一只，最近一直在学PE，刚学的往节表空白区添加代码，就想水个贴，然后算是总结一下，大牛就别看了，太菜……

用的软件是 WinHex、OD、PEtools 和 VS2013，感染的软件是本地的记事本`notepad.exe`。这是很简单的一个感染，在打开程序之前加上弹窗。先执行我们的弹窗，之后再执行软件自身的代码。

步骤如下：

1. 获取`MessageBox`地址，构造`ShellCode`代码。
2. `E8`/`E9`计算公式。
3. 在代码区手动添加代码。
4. 修改`OEP`，指向`ShellCode`。

## 1. 获取MessageBox地址，构造ShellCode代码

用 OD 随便打开一个软件，执行`BP MessageBoxA`，去断点窗口，查看`MessageBox`的地址，我的地址是`77d507ea`。而`MessageBox`需要四个参数，在反汇编窗口可以看到它的硬编码为`6A 00 6A 00 6A 00 6A 00 E8 Address`。

## 2. E8/E9计算公式

`E8`、`E9`是硬编码的`Call`和`JMP`，也就是程序在硬盘中的存储方式。这是我写的一个获取这两个硬编码的测试代码：

```c
#include<stdio.h>

voidFunction() {
    printf("123");
}; //执行完这个函数之后会有返回到之前的位置的跳转代码字节为E9D505
0000

intmain() {
    Function(); //反汇编（只显示代码字节）E8A8FDFFFF
    return 0;
}
```

但是这个地方`E8`之后的地址并不是准确的跳转地址，含有一定的计算关系，计算公式为：

`X = 想要跳转的地址 - E8运行时的地址 + 5`；

## 3. 在代码区手动添加代码

这个时候就可以用 WinHex 在代码区的空白处添加代码。首先用 PEtools 打开`notepad`，查看第一个节表的信息，以及`SectionAlignment`和`FileAlignment`的大小（因为最后改的代码的地址必须是`ImageBuffer`时的地址，所以一定要看文件对齐跟内存对齐），找到第一个节表（也就是代码区节表）的信息，找到`VirtualSize`和`PointerTORawData`来确定代码区的尾部，看下代码区剩余的空间大小是不是可以添加我们的代码。

准备工作做完就可以开始添加代码。根据第一步构造的`ShellCode`，我们添加的代码格式为：

```
6A 00 6A 00 6A 00 6A 00 E8 __ __ __ __  E9 __ __ __ __
```

空白处是需要我们计算的地址。`E9`的原因是返回原始的`OEP`，不能影响程序本来的执行顺序。

现在开始计算地址。因为要在运行状态下达到感染的目的，所以地址必须按照运行状态计算。首先计算`E8`之后的`Address`，而这个时候`E8`的地址需要计算出程序在`ImageBuffer`中的地址：

计算方式为：`E8运行状态的地址 = E8此时地址 + ImageBase + SectionAlignment - FileAlignment`

之后根据公式 `X = 想要跳转的地址 - E8的地址 + 5`。

计算出来，在我的电脑上是`76 D4 80 8D`。

即`E8 8D 80 D4 76`。现在再计算`E9`以后的地址，同样是根据公式 `X = 想要跳转的地址 - E8的地址 + 5`。

即 `E9运行状态的地址 = E9此时地址 + ImageBase + SectionAlignment - FileAlignment`

此时`X = 原始的OEP - E9运行时的地址 + 5`，这个地方的目的是执行完之后跳转到原始`OEP`。

这是我的计算完成添加之后的样子：

![](https://bbs.kanxue.com/upload/attach/201805/774936_72GZ8UMY2MBDPRJ.png)

## 4. 修改OEP，让程序从我们改过的地方开始运行

通过 PEtools 查找可以看到是`739D`。

![](https://bbs.kanxue.com/upload/attach/201805/774936_3MD53TQB42GPVW7.png)
![](https://bbs.kanxue.com/upload/attach/201805/774936_F2VHG6SC9G7UUXZ.png)
![](https://bbs.kanxue.com/upload/attach/201805/774936_NU2JGMA4PT4RBGY.png)

然后用 WinHex 修改程序的入口点，即为第一个`6A`的地址。

`7B50 + (SectionAlignment - FileAlignment) = 8750`

在可选PE头部找到`73 9D`，改为![]()。

保存文件，大功告成。打开文件实验一下：

![](https://bbs.kanxue.com/upload/attach/201805/774936_VY6D3SR4WKCPBYB.png)
![](https://bbs.kanxue.com/upload/attach/201805/774936_AHND8AUD7Y7884E.png)

正常运行，并且会先运行我们添加的代码，点击确定之后可以功能使用正常。

嘿嘿，比较简单，算是水一贴，刚开始学，算是找个存储空间……如果有哪个地方不对，欢迎挑错~~~

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
