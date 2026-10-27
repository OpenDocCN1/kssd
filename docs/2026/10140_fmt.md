
# [原创]vs2005对函数局部变量的堆栈检测分析

前两天，在论坛上看了泉哥的“泉行之路”和笨笨熊的“职场游记”，深受鼓舞，于是决定写下下面这篇鸟文，高手飘过，如有错误，概不负责哦![](https://bbs.kanxue.com/view/img/face/12.gif)![](https://bbs.kanxue.com/view/img/face/12.gif)，呵呵

## 引子

<本文的分析是在debug模式下，呜呜，有人怒了……>

在vc6.0下，写溢出程序很容易，在.net下由于添加了`/GS`选项，加入了`__security__cookie`，溢出就没那么简单了。对`__security__cookie`我也是最近才研究了一下，才知道：当在函数中`buff`的长度 <= 4时，编译器是不会在函数中插入`cookie`进行校验的。于是产生了下面这个天真的想法：

```
void foo()
{
  char buf[4] = {0};
  strcpy(buf,shellcode);
}
```

是不是这样`foo`函数流程就能被逆转了呢（想法很天真呐![](https://bbs.kanxue.com/view/img/face/12.gif)，嘿嘿……），于是我就进行尝试了一把，结果挂了，弹出下面这个框框：

![](https://bbs.kanxue.com/upload/attach/201012/253799_tc0x5oe3phbv487.jpg)

单步调试，才发现编译器加入了函数`void __fastcall _RTC_CheckStackVars(void *frame, _RTC_framedesc *v)`对函数的局部变量进行校验……呜呼，显然我的想法很傻很天真![](https://bbs.kanxue.com/view/img/face/50.gif)。

其实微软的编译器是这么检查堆栈变量的：在一个存在局部变量的函数里头，编译器统计函数的局部变量个数`num`，在栈上分配`192 + num * 2 * sizeof(DWORD) + num*sizeof(DWORD)`个字节的栈空间，并将其全部填充为`0xCCCCCCCC`，当函数结束后，编译器会检测每个局部变量的前4个字节和后四个字节是否为`0xCCCCCCCC`，如果不是，则说明发生数据覆盖等，跳向错误处理……

## 实例演示

```
#include <stdio.h>

void foo()
{
  char x[4] = {0};
  char y[4] = {0};

  x[3] = 'A';
  y[3] = 'A';

  printf("x = %x\n",x[3]);
  printf("x = %x\n",y[3]);
}

void main()
{
  foo();
}
```

用vs2005编译后，用ida打开，进入`foo`函数内部：

![](https://bbs.kanxue.com/upload/attach/201012/253799_bf05d654kry1kuz.jpg)

继续往下走，找到`_RTC_CheckStackVars`函数：

![](https://bbs.kanxue.com/upload/attach/201012/253799_oljutaluer9wy3f.jpg)

在进入`_RTC_CheckStackVars`函数之前我们先分析一下变量`v`到底是个什么东西……

![](https://bbs.kanxue.com/upload/attach/201012/253799_pkdr1p3p3zkdb43.jpg)

下面在进入关键函数`_RTC_CheckStackVars`分析：

![](https://bbs.kanxue.com/upload/attach/201012/253799_5va7xbj1b3kjahk.jpg)

![](https://bbs.kanxue.com/upload/attach/201012/253799_7s4mhpxc6wnu26g.jpg)

通过对编译器插入的函数`_RTC_CheckStackVars`的分析，估计大家对编译器的行为又有了一个新的了解。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（18.00kb，280次下载）
- [2.jpg](javascript:void(0);)（80.58kb，288次下载）
- [3.jpg](javascript:void(0);)（65.69kb，284次下载）
- [4.jpg](javascript:void(0);)（99.43kb，282次下载）
- [5.jpg](javascript:void(0);)（125.00kb，282次下载）
- [7.jpg](javascript:void(0);)（146.95kb，311次下载）
