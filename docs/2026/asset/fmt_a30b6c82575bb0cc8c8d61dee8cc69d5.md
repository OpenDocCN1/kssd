
# [原创]翻译Dos环境下的鼠标编程

![](https://bbs.kanxue.com/view/img/face/16.gif) ![](https://bbs.kanxue.com/view/img/face/16.gif) ![](https://bbs.kanxue.com/view/img/face/16.gif) ![](https://bbs.kanxue.com/view/img/face/16.gif) 要做图形学作业,!就采用了这中古老的方式,顺便翻译了篇文章.![](https://bbs.kanxue.com/view/img/face/50.gif) ![](https://bbs.kanxue.com/view/img/face/50.gif)

=========================================================================================================

# Dos环境下的鼠标编程

输入设备只有键盘（没有鼠标）的时候已经过去了，以图形界面为基础的操作系统以及应用程序的到来导至了对鼠标的迫切需求与广泛应用。鼠标是一种指向设备用来控制显示在屏幕上的指针。当我们移动鼠标的时候，指针也以同样的方向在屏幕上移动。通过使用鼠标，我们可以进行单击图标或者拖动对象等操作。软件会对这些行为做特定的响应。比如，如果你用鼠标点击一个文件，程序就会把文件的内容显示给你。对于图形界面的操作系统（比如windows）而言，鼠标已经象键盘一样成为了系统的一部分。也就是说，每个应用程序在这种系统下创建，运行，系统给与了关于鼠标的充分的支持。在这种情况下开发应用程序，会非常容易，因为每一个鼠表事件，操作系统都会感知并为你产生一个关于此事件的消息。当用户点击按钮或者用鼠标做些其他事情的时候，系统会产生一个消息发送给相应的应用程序，这个消息描述了鼠标的行为.在这种情况下，软件开发人员所需要做的只是处理这些消息。

在dos下写支持鼠标的应用程序会需要有一些技巧，稍微有点复杂。这是因为dos提供的是文字模式（字符模式），dos的开发者从来没有想过要将鼠标做为操作系统的一部分功能。所以，这里没有想windows系统那样的消息概念（在windows下，单击鼠标，会产生一个消息，发送给相关的应用程序）。但是，dos是兼容鼠标的。只要你确实有个鼠标，并且有相应的驱动程序。

在这篇教程中，我写一些关于用c语言处理一些鼠标的功能。我已经假设你已经使用过了c语言，如果你还没有，你最好先去阅读关于c基础的书，在dos下进行鼠标编程包括了调用中断。让我们开始学习之前，先熟悉一下关于中断方面的知识。

## 中断小知识：

当你电脑的处理器正在忙着工作，而你恰好按下了键盘上的按键，会发生什么呢？无论当时处理器在做什么，我们会看见，根据我们的按键，会有一个相应的字母显示在屏幕上。因为按键中断了处理器的执行，我们就说，键盘中断产生了。简明的说，中断是一个终止处理器正在执行的操作，而转去执行别的操作的信号。仅仅在新的工作完成之后，处理器才回去执行开始中断前尚未完成的任务。这种恢复之所以可能是因为中断一但发生，处理器就会通过将所有的内部寄存器压入堆栈来保存处理器状态，并在恢复的时候将堆栈中的数据弹回到原来的寄存中。现在我们来了解一些技术细节，每一个中断都有它的中断处理程序。中断处理程序是在内存中的一断程序，当中断发生时，处理器就转去执行相应的中断处理程序，中断处理程序的地址全部保存在一个叫中断向量表的地方。中断发生时，处理器从中断向量表中获取中断程序地址，并转去执行。每一个中断都有一个号码标识。`9h`是键盘的中断号，`8h`是时钟的，等等。当调用一个中断的时候，我门需要将一些简单的数字传进去，被子过程使用。为什么我们需要这些子函数？拿中断号-`10h`的处理屏幕的中断来说，它允许我们利用使用子过程（`41h`和`42h`）来进行文字模式和图形模式的切换。允许我们只用子中断`ah`来写一个字符到屏幕上，允许我们调用子中断`42h`清理屏幕，等等。我们将子中断号作为参数传递进去，以告诉系统我们想要具体使用的中断功能。

## 让我们进入“唯c”的世界吧：

我们的第一程序是检查判断鼠标驱动程序是否已经被载入。

这里再说一下。驱动是一个检查鼠标的存在，并解释从鼠标端口传来的信号的程序。

### 1）探测鼠标是否被载入.

```
#include <dos.h>

main()

{
  union REGS i,o;
  clrscr();
  i.x.ax=0;
    int86(0x33,&i,&o);
 if(o.x.ax==0)
    printf("No Mouse Available.....");
   else
        printf("Mouse Available......");
}
```

上面这个程序声明了两个类型为`REGS`的联合体。关于`REGS`的具体声明，已经包含在`dos.h`头文件中，有两个结构体(`struct WORDREGS x, struct BYTEREGS h`)这两个结构题包含一些1字节单位长度和二字节单位长度的变量，用来间接的表示寄存器。通过传递0给`AX`寄存器，并调用鼠标中断，我们能够检测出鼠标驱动程序是否已经被载入。我们使用`int86`这个函数来调用中断。`Int86()`需要参个参数，分别是中断号，和两个`REGS`类型的变量。如果不存在鼠标驱动，0会被返回到`AX`寄存器。在程序中，所有的返回值都能在`REGS`类型变量`o`中获得。

### 2．一个显示鼠标指针的程序。

我们的第一个程序仅仅报告了鼠标驱动是否存在。就算驱动存在，我们也不会发现有鼠标指针的标记。如果要看到鼠标指针，我们需要使用第一个子过程。看下面的这个程序。

```
#include <dos.h>
main()
{
  union REGS i,o;
  clrscr();
  i.x.ax=0;
    int86(0x33,&i,&o);
    if(o.x.ax==0)
  {
     printf("No Mouse Available.....");
         exit();
  }
    i.x.ax=1;
  int86(0x33,&i,&o);
}
```

上面这个程序除了最后几行，基本是一致的。我们已经把1付给了`AX`寄存器，然后调用了中断，以看见指针。因为我们处于文字模式，我们的鼠标指针是一个小方块。如果我们切换到图形模式，我们将看到一个小箭头。

### 3) 实现隐藏鼠标指针。

甚至当我们的程序结束后，鼠标指针仍然在。看下面的程序：

```
#include <dos.h>
main()
{
    union REGS i,o;
      clrscr();
    i.x.ax=0;
    int86(0x33,&i,&o);
         if(o.x.ax==0)
    {
           printf("No Mouse Available.....");
           exit();
    }
    i.x.ax=1;
    int86(0x33,&i,&o);
    gotoxy(24,23);
      printf("Press any key to hide mouse cursor...");
    getch();
    i.x.ax=2;
    int86(0x33,&i,&o);
      gotoxy(10,23);
      printf("Mouse cursor is hidden !! Press any key to terminate the program ...");
    getch();
}
```

上面这个程序使用了子过程2，并调用中断实现隐藏鼠标指针。当当我们移动黍标画直线和方筐的时候，这个功能非常有用。当写这类程序的时候，我们不想让指针擦去了我们所画的，所以我们把指针隐藏起来。

### 4．编写程序实现获取鼠标位置。

为了写一个有用的支持鼠标的程序，我们通常需要知道指针所在坐标(`x,y`);下面这个程序打印出鼠标的位置，伴随着鼠标移动。

```
#include <dos.h>
main()
{
  union REGS i,o;
  clrscr();
  i.x.ax=0;
  int86(0x33,&i,&o);
  if(o.x.ax==0)
  {
    printf("No Mouse Available...");
    exit();
  }
  i.x.ax=1;
  int86(0x33,&i,&o);
  gotoxy(25,23);
  printf("Press any key to exit...");
  while(!kbhit())
  {
   i.x.ax=3;
   int86(0x33,&i,&o);
   gotoxy(2,2);
   printf("x->co-ordinate=%d     \n y->co-ordinate=%d      ",o.x.cx,o.x.dx);
  }
  i.x.ax=2;
  int86(0x33,&i,&o);
}
```

上面这个程序有个循环，直到我们按某个键，循环才退出。在这个循环里，我们使用子过程3。子过程三返回x坐标到`cx`寄存器，y坐标到`dx`寄存器。然后一直打印x,y的坐标直到循环退出。鼠标的范围在文字模式下是640x200，在图形模式下是640x480。

### 5》打印出鼠标键点击时候的情况。

知道鼠标哪个键被按下是非常重要的，下面这个程序显示了我们点击鼠标的时候是哪个键被按下。

```
#include <dos.h>
main()
{
  union REGS i,o;
  int button;
  clrscr();
  i.x.ax=0;
  int86(0x33,&i,&o);
  if(o.x.ax==0)
  {
    printf("No mouse available....");
    exit();
  }
  i.x.ax=1;
  int86(0x33,&i,&o);
  gotoxy(24,23);
  printf("Press any key to exit....");
  while(!kbhit())
  {
    i.x.ax=3;
    int86(0x33,&i,&o);
    button=o.x.bx&7;
    gotoxy(23,11);
    switch(button)
    {
      case 1:
        printf("Left button pressed                                  ");
      break;
      case 2:
        printf("Right button pressed                                  ");
      break;
      case 4:
        printf("Middle button pressed                                  ");
      break;
      case 3:
        printf("Left and Right buttons pressed                          ");
      break;
      case 5:
        printf("Left and Middle buttons pressed                          ");
      break;
      case 6:
        printf("Right and Middle buttons pressed                          ");
      break;
      case 7:
        printf("All the three buttons pressed                          ");
      break;
      default:
        printf("No button pressed....");
    }
  }
   i.x.ax=2;
   int86(0x33,&i,&o);
}
```

上面着个程序除了在循环外有一些例外外，其他跟前面那个程序是一致的，我们仍然调用子中断3,鼠标键被按下的信息其实已经存放在了`bx`寄存器里面。全部的按键信息被存放在`bx`的前三个bit里。所以我们将`bx`与7做与操作以分离前三个bit,并把他们存放在`button`这个变量里面。第1位是1表示左键按下了，如果是0，表示没有被按下。第2位是1表示右键按下了，如果是0，表示没有被按下。如果最后通1位是1表示中间的键按下了，如果是0，表示没有被按下。

### 6.编写程序实现置鼠标指针到屏幕上所需要的位置。

一些时候我们需要实现置鼠标指针到屏幕上所需要的位置。就项我们使用`gotoxy()`将键盘光标置到指定的位置那样。下面这个程序将鼠标指针放到了屏幕上(`x=150,y=100`)这个位置。

```
#include <dos.h>
main()
{
  union REGS i,o;
  clrscr();
  i.x.ax=0;
    int86(0x33,&i,&o);
  if(o.x.ax==0)
  {
     printf("No mouse available");
     exit();
  }
  i.x.ax=1;
  int86(0x33,&i,&o);
  i.x.ax=3;
  int86(0x33,&i,&o);
     gotoxy(1,1);
  printf("Current Position:x=%d    y=%d    ",o.x.cx,o.x.dx);
  gotoxy(15,23);
  printf("Press any key to set the mouse pointer to (150,100)...");
  getch();
  i.x.ax=4;
  i.x.cx=150;
  i.x.dx=100;
  int86(0x33,&i,&o);
 gotoxy(15,23);
 printf("Cursor is set ... press a key to exit                    ");
 getch();
}
```

上面那个程序中，我们使用子过程4设置了鼠标位置。我们通过将值传递给`CX`寄存器设置x坐标，通过将值传递给`DX`寄存器设置Y坐标。

### 7切换到图形模式

下面这个程序将实现从文字模式切换到图形模式。执行程序后，你会发现鼠标已经变成了箭头。这个程序试图在图形模式下执行程序4,并且观察鼠标的边界已经增加到了640x480。

```
#include <graphics.h>
#include <dos.h>
main()
{
  int gd=DETECT,gm;
  union REGS i,o;
```
