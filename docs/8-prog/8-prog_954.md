
# 俄罗斯方块 Dota 版（附 VC 源码）

> 原文：[`www.pediy.com/kssd/pediy11/105988.html`](https://www.pediy.com/kssd/pediy11/105988.html)

![](img/e0cebb9e84a3e92932c1c922f2bb1015.jpg)

## 程序框架流程

程序采用`Win32 SDK`和`C++`编写。玩了这么多年的俄罗斯方块，自己也来写一个，为了不太单调，加入了一些`Dota`的元素和连连看功能呵呵，可真当自己写的时候才发现创造者当时的巧妙的构思，俄罗斯方块中每个方块都是由四个小方块组成，但每个块的形状，变换又不相同，故利用`C++`继承已经多态来实现比较恰当。

整个程序由一个背景窗口，一个玩家窗口，一个`AI`窗口和`2`个分数窗口组成。

俄罗斯方块，顾名思义，即为方块组成，将整个窗口分为若干相同的小方块，采用全局数组来存储窗口中每个块的信息，掉落的方块均为`4`个小方块，下文中所说的掉落块即为游戏中的`7`种掉落方块，小块即为整个窗口中的每个小块，定义一个`Block`类来当做基类，包含`4`个`Cpoint`结构用于存放当前掉落方块在整个窗口的位置，另定义各种虚函数，如`SwitchBlock`来变换掉落块，`MoveBlock`来移动掉落块，`FillBlock`来将当前掉落块的信息填充到屏幕，`CleanBlock`、`CleanSame`等等。

从`Block`基类派生出`7`种方块，然后重载`SwitchBlock`和`AI`功能函数实现每个掉落方块的不同功能。

通过重载`AI`函数来实现不同掉落块的`AI`功能，在`Block`中加入`m_hwnd`来存储当前掉落方块属于的窗口句柄，加入全局数组的引用来表示该掉落块属于那个窗口的数组。

为了和玩家窗口独立，采用多线程的方式，将`AI`功能窗口创建在另一个线程的消息循环中。

程序大体流程如下：

![](img/929ba4729e435e91b76d0df0df20322c.jpg)

玩家窗口通过相应键盘的上下左右按键消息来调用相应的`MoveBlock`函数来移动掉落块，在`MoveBlock`中首先通过`Check`函数来判断该块的位置是否能移动，是否有满行，是否有相同连续的图案需要清除。如需要清除则调用`Clean`，`CleanSame`函数来清除相应的块。在`Clean`，`CleanSame`中调用`Fall`函数来实现悬空块的掉落。

## 图案背景的实现

程序通过全局数组来保存整个窗口的每个小块信息。将整个窗口的背景分成若干小块，每个小块对应数组中的一个元素，数组中存放着每个小块位图的`ID`，而每个掉落块`Block`中的`Cpoint`存放的则是掉落块在数组中的索引，`m_BitmapIndex[4]`程序存放`4`个小块的相应的位图`ID`。`FillBlock`函数即实现`m_BitmapIndex[4]`中的`ID`写到全局数组中，`CleanBlock`即将掉落块对应位置的数组元素改回成相应背景图片的`ID`，`ShowBlock`函数则通过数组中的位图`ID`，显示出整个窗口的图画。

## 方块的移动实现和变换

方块的移动通过`MoveBlock`函数实现。`MoveBlock`函数首先判断当前掉落块是否可以移动，如是否已到达边界，是否已经到达底部，然后判断当前整个窗口中是否存在满行或者连续的相同图案（连连功能），若存在则清除，然后让悬空的小块下落到相应的位置，最后调用`ShowBlock`函数刷新画面。若掉落块可移动则调用`CleanBlock`函数将该掉落块当前的全局数组中的位置的位图`ID`还原成背景的位图`ID`，然后`4`个`CPoint`成员中的的索引位置往相应的方向变动，再调用`FillBlock`函数填充当前位置的数组元素，使得该位置的位图变成该方块的图案，最后再调用`ShowBlock`函数刷新整个窗口。

方块的变换通过虚函数`SwitchBlock`实现。从`Block`基类派生出`7`个掉落块的派生类，重写`SwitchBlock`虚函数。块的变换的工程和块的移动相似，不同之处在于变换块时改变`4`个`Cpoint`的方式。

## 清除满行块和连连看功能

在`MoveBlock`中，通过`Check`函数判断当前块，如果为下移，则遍历当前数组，看是否存在满行的块，即该行所有的元素都不为背景图片`ID`，若为满行块则调用`Clean`函数清除,即将数组的该行元素全部改回相应的背景图片`ID`。然后遍历整个窗口，通过链表寻找每个小块周围的小块，看是否存在`2`个或者`2`个以上的相同图案的小块，若存在则清除，清除过程和清除满行块类似。清除完成后会有悬空的小块，调用`Fall`函数来实现小块的下落，因为下落后可能还会存在满行的块或者图案相同的小块，故采用循环的方式来实现清除块的功能。

相应的代码如下：

```
do
{
  if (BFull=Clean())
    Sound(KillKeep+=BFull);
  if (BSame=CleanSame())
    Sound(KillKeep+=BSame);
}  
while (BSame||BFull);//当没有满行块和连续的相同图案块时才结束循环
```

## 小块的下落实现

在清除满行块或者连续的相同图案的块后，会有小块悬空，这些小块需要下落，`Fall`函数即在`Clean`和`CleanSame`函数后实现下落功能，`Fall`函数遍历整个数组，如果数组元素不为背景图片`ID`即为小块，然后通过定则判断该小块是否悬空，定则如下：

1. 一个小块如果左边右边下边都没有别的块，说明该小块悬空
2. 一个小块如果左边右边有别的块，但是该块下面一行全部都是空的，说明该小块所在的行悬空

相关代码如下：

```
for (int N=Length-2;N>=0;N--)
  for (int M=0;M!=Width;M++)       
    if (st_AllState[M][N]&&!st_AllState[M][N+1])  //不为空的且下面没有块才判断
    {
      BFall=false;
      Blank_LfRt=false;
      if (!M||M==Width-1)       
      {
        if (!M&&!st_AllState[M+1][N]||M==Width-1&&!st_AllState[M-1][N])
          Blank_LfRt=true;   
      }
      else if (!st_AllState[M-1][N]&&!st_AllState[M+1][N])
        Blank_LfRt=true;         
      CPoint *SP=NULL;
      m_hwnd==MainHwnd?SP=PlayerSame:SP=AISame;
      if (Blank_LfRt)           //左右为空，直接下落
        BFall=true;
      else                    //左右不空，判断是否有地基
      {
        BFoundation=false;
        for (int l=M;l>=0;l--)     //判断左地基
        {
          if (!st_AllState[l][N])     
            break;
          if (st_AllState[l][N+1])
            BFoundation=true;
        }
        for (int r=M;r<=Width-1;r++)  //判断右地基
        {
          if (!st_AllState[r][N])     
            break;
          if (st_AllState[r][N+1])
            BFoundation=true;
        }
        if (!BFoundation)
          BFall=true;
      }
      for (int index=0;index<=SameIndex;index++)
      {
        if (M==SP[index].x&&N<=SP[index].y)
          BFall=true;
      }
    }
```

如果小块悬空，则将小块下移一格，然后创建一个线程显示小块移动的动画(创建新线程是为了不影响主线程的运行速度)，下落函数采用递归的形式，直到没有悬空的小块为止。

相关代码如下：

```
if (BFall)
{
  Repeat=true;         
  st_AllState[M][N+1]=st_AllState[M][N];
  st_AllState[M][N]=0;
  FallBlinkStruct *FBall=new FallBlinkStruct();
  if (!FBall)
    MessageBox(0,"内存不足",0,0);
  FBall->hwnd=m_hwnd;
  FBall->st_back=st_BackGround;
  FBall->CurP.x=M;
  FBall->CurP.y=N;
  CreateThread(NULL,NULL,FallBlink,(void*)FBall,NULL,NULL);
}
     }
   if (Repeat)
     Fall();
     }
```

## 方块的诞生和消亡

定义一个全局的`Block*`指针`PBlock`，因为所有的掉落块都是派生自`Block`，故`PBlock`可以指向任何掉落块。在程序启动的时候使用`new Block`分配一个掉落块，构造函数中定义了该块的初始位置，属于的窗口，全局数组的引用，背景图片数组的引用，是`7`个方块中的哪一种掉落块和`4`个小块的图案都则是随机生成，程序启动时同时分配一个定时器来自动实现方块的下落。玩家可以通过向下的按键实现快速的下落。当该掉落块不能下移的时候，`MoveBlock`通过`Check`得知不可下移会使用`delete Pblock`，将该块从内存中释放掉，然后使用相同的方法`new`一个新块，并将`PBlock`指向它，就这样一直循环。

## AI 功能的实现

`AI`实现人机对战，在程序运行时候，为程序另外创建一个线程，在该线程中也创建一个窗口并且用一个全局数组来记录整个窗口的信息，同样的分配一个新的背景图片`ID`数组。`AI`的线程和玩家主线程的实现基本一样，只是`AI`线程中没有捕获键盘按键和设置定时器，而是通过`Block`中的`AI`函数算出该块的移动路径，只是把玩家通过判断后使用键盘操作来移动块的方法换成了自动计算出路径。

`AI`函数是虚函数，在从`Block`基类中派生出的`7`个子类中进行重写。`AI`函数通过遍历整个数组来计算出当前块的移动路径,具体实现如下

从上到下竖行的遍历整个数组，从而去的每一竖行中最底端的块的位置，然后通过堆排序选出最底的块的位置，根据当前下落块的各种性质（各种变换后的形态），判断该块是否可到达最底的块的位置上方，如果可以则根据计算的路径移动块到该位置，然后`delete PBlock`并且`new`一个新的下落块，如果不行则将该元素从堆中移除，重新平衡堆，再次取出最大的元素，如此往复，知道算出路径。

## 计分功能的实现

玩家和`AI`各有`5`个小窗口用于显示各自的分数。在`Clean`或`CleanSame`后会加上相应的分数，再各自的分数窗口中显示出来。

每个窗口显示一位数字，每次调用`AddScore`函数后,为了不影响主程序的执行效率，创建新一个新的线程来实现分数窗口的刷新，并为新的线程传入加的分数作为参数。为了不让加分变得僵硬，采用累加一次刷新一次，而不是一次全加后刷新，使用随机函数来确定每次加的分值。相关代码如下：

```
while (i<Add)
{
  n=rand()%100;
  if (n>Add-i)
    n=Add-i;
  i+=n;
  if (Add%10==1)
  {
    Score+=n;
    for (int n=0;n!=5;n++)
      SendMessage(hPlayerScore[n],WM_PAINT,NULL,NULL);  
  }
  else
  {
    AIScore+=n;
    for (int n=5;n!=10;n++)
      SendMessage(hPlayerScore[n],WM_PAINT,NULL,NULL);  
  }
}
```

在分数窗口的`WM_PAINT`消息中则显示当前的分数，通过对对应的当前分值进行除和求余操作得到当前的窗口显示的数字。`0-9`每个数字都是一副位图，窗口通过使用位图来填充以显示窗口，但是为了显示的数字是镂空的（背景透明），则需计算出当前数字（不含背景）的区域，然后调用`SetWindowRgn`来将窗口设置成相应的形状。相关代码如下：

```
BakClor=GetPixel(hDC,0,0);           //背景像素
WndRgn=CreateRectRgn(0,0,0,0);         //创建一个空区域
for (Y=0;Y<56;Y++)
       {
          int MX=0;
          do
          {
              int LX=MX;
              while (LX<50 && BakClor==GetPixel(hDC,LX,Y))
                LX++;
              MX=LX;
              while (LX<50 && BakClor!=GetPixel(hDC,LX,Y))
                LX++;
              HRGN TmpRgn=CreateRectRgn(MX,Y,LX,Y+1);   //确定该像素不为背景像素
              CombineRgn(WndRgn,WndRgn,TmpRgn,RGN_OR);//将该像素选入区域中
              DeleteObject(TmpRgn);
              MX=LX;
          }while (MX<50);     
       }
       SetWindowRgn(hwnd,WndRgn,TRUE);
```
