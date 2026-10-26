
# [分享]3句代码，在星际争霸屏幕上写字(详细教程，有图，附源码)

我们来看看在游戏里怎么写字！

游戏版本：星际争霸1.16.0

工具:`OllyDBG`,`VS2005`

如下图：

![](https://bbs.kanxue.com/upload/attach/200901/65800_qvewy2gdqyvv0cf.jpg)

**找CALL思路：游戏里写完字后，它应该会在内存里留下字符的痕迹，我们就从这个思路开始找写屏的函数。**

## 第一步
先让游戏自已写几个字！运行游戏，选择局网IPX,和电脑对手。开始游戏后选择设置网速（玩星际的不用说怎么设置了吧？），完了它会出现以下字符：

![](https://bbs.kanxue.com/upload/attach/200901/65800_7f3qdg8a2uwpbem.jpg)

## 第二步
在内存里找关键字，按下图所示，找到为止。

1,按图所示，在`OD`里先从`D 00400000`开始，回车。

2,按`Ctrl+B`输入关键字，再确认，如果没找到，再从1步骤开始（找不到再从`D 00500000`,`D 00600000`）。

![](https://bbs.kanxue.com/upload/attach/200901/65800_eah39dhzfalyeg2.jpg)

## 第三步
找到后用你的大脑分析，怎么下断。

可以看得出，这是个动态分配的字符串，但是也有规律，每间隔N行就是一串。

![](https://bbs.kanxue.com/upload/attach/200901/65800_u9a5z1hw06w6aka.jpg)

## 第四步
找到规律后，我们试试在最后一次显示的字符串下面，选择一大片空白区下个"内存写入断点"，看看能不能找到是哪个程序在写字符？下完断点后，在`OD`里按`F9`运行，再切换回游戏设置网速，让游戏自已再写一遍字。

![](https://bbs.kanxue.com/upload/attach/200901/65800_63fwjscmfcrs4mt.jpg)

## 第五步
运气真好，居然断到了。

1,如果断点正确(看下图1的注释)，就按下图中所标的1,2,3步骤操作，之后，在`OD`按`F9`运行程序。

说明：按这三个步骤操作很重要，它可以帮你带到写屏函数那里去。因为找到写字符的内存了，我们就可以找到显示字符的函数了，因为游戏要读这段字符内存来进行显示。

![](https://bbs.kanxue.com/upload/attach/200901/65800_chfmzluj722w1pl.jpg)

## 第六步
离胜利不远了。

它把你带到写屏函数附近了。如下图所示，为什么要在2那里下个断再走一步？因为这还不是写屏的主函数，因为它`RETN 0C`，就算你用远程进程`Call`这个地址，程序也会出错，不会正常返回，所以我们还要往回找，看这里哪里`CALL`进来的，按下图的说法操作。

![](https://bbs.kanxue.com/upload/attach/200901/65800_h3xrcd94t0e2nt9.jpg)

## 第七步
搞定！

![](https://bbs.kanxue.com/upload/attach/200901/65800_0zuamca3ie0zmy3.jpg)

## 第八步
通过远程调用游戏的`CALL`，来实现游戏写屏：

这一步骤就不详说了，`VS2005`,我用的是全局钩子。你们爱怎么就怎么的，反正关键代码是：

```c
if(X键按下)
{
DWORD addr = 0x0048CFB0;//这个地址就是我们刚才找到的写屏主函数的起点(星际1.16版本)。
char *buf = "Hello Word!";
__asm
{
        xor eax,eax;
        mov edi,buf;
        call addr;
}
}
```

星际每个版本都可能这个地址有变化，但是步骤都差不多。下面是1.08版本的:

```c
DWORD addr = 0x0046DE10;//（星际1.08版本的）
char *buf = "Hello Word!";
__asm
{
        xor edx,edx;
        mov ecx,buf;
        call addr;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)
  （82.64kb，2555次下载）
- [2.jpg](javascript:void(0);)
  （74.14kb，2567次下载）
- [3.jpg](javascript:void(0);)
  （163.71kb，2564次下载）
- [4.jpg](javascript:void(0);)
  （175.45kb，2558次下载）
- [5.jpg](javascript:void(0);)
  （158.33kb，2558次下载）
- [6.jpg](javascript:void(0);)
  （156.98kb，2558次下载）
- [7.jpg](javascript:void(0);)
  （152.40kb，2555次下载）
- [8.jpg](javascript:void(0);)
  （153.78kb，2550次下载）
- [sc1.16.part1.rar](javascript:void(0);)
  （1.81MB，169次下载）
- [sc1.16.part2.rar](javascript:void(0);)
  （766.06kb，159次下载）
