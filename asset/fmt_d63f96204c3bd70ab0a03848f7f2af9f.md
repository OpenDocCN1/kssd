
# 连连看游戏，2 点是否可消除算法分析

> 原文：[`www.pediy.com/kssd/pediy11/124658.html`](https://www.pediy.com/kssd/pediy11/124658.html)

现在在看郁金香的 `VC` 外挂教程，其中对于连连看外挂

在此感谢这位兄弟 [`bbs.pediy.com/showthread.php?...8%8B+%E7%A8%8B`](http://bbs.pediy.com/showthread.php?t=110703&highlight=%E9%83%81%E9%87%91+%E9%87%91%E9%A6%99+vc+%E6%95%99%E7%A8%8B+%E7%A8%8B)

提供的下载地址，现在学习中……

言归正传，这个外挂的最难的就是检查 2 点是不是可以联通的算法，我们就来研究这一点

这个游戏，发现 2 点之间连接线段数需要满足 `<= 3` 这个条件

如图 1：

![](img/abfca142624243b27d5189a89e528fa0.jpg)

我们先来谈谈 3 条线段的情况，先假设，棋盘上除了这 2 点以外全部为空，这 2 点分别用 `p1`, `p2` 表示，并显示坐标，那么在连接线段数 `<= 3` 的情况下，一共有多少种路径呢？

如图 2：

![](img/6915857b4ec514b7e3b8a2cbcc7349f5.jpg)

我们来思考一下，以下是所有路径，如图 3：

![](img/d3e06057d4e4d4bdea5d90df2cffed48.jpg)

（路径用绿线表示）可以看到这不是一两种的问题，而是由很多种，其中我用深颜色的线画出了几条有代表性的路径，

仔细的看图就会发现其中有规律可寻，看图 4：

![](img/69275c0c7b34827db21e1f61a6ead511.jpg)

我标出了其中比较重要的 4 个点！~

这 4 个点分别是 `p1`, `p2` 在坐标轴上的投影，分别用 `px1`，`px2`(表示在 `x` 轴上的投影)，`py1`, `py2`(表示 `y` 轴上的投影) 如图 5：

![](img/00dd9a66ee5ef9c5af6c2148c9a897ea.jpg)

先来分析 `px1`, `px2` 可以从图 3 上清楚的看到，联通路径上的有一部分就是就是线段 `px1`，`px2`,或者是它的平行线！~

再分析 `py1`，`py2` 也是一样的道理

我们再来分析一下 `px1`，`px2`，`py1`, `py2` 的坐标情况，如果已知 `p1`（`x1`, `y1`）, `p2`（`x2`, `y2`）的坐标,那么 `px1`，`px2`，`py1`, `py2` 坐标是什么呢？

只要有几何的初步知识就可以知道 `px1`(`x1`, 0)，`px2`(`x2`, 0)，`py1`(0, `y1`), `py2`(0, `y2`)

那么分析上面的一大堆对于我们有什么用呢？

答案是肯定有用！~

看图 6：

![](img/1d0559a2ad7c5a636fdcaff3d072b32d.jpg)

图中连通的路径由 3 条线段，有 4 个点组成，只要判断线段 1，2，3，是否都连通，就可以判断 2 点是否可以连通了!~

实际上分析，每个人都可以分析的头头是道，编程最难的就是怎么样把分析好的数学模型通过编程来实现，分析是不需要考虑很多的细节问题的，而编程实现需要考虑很多的细节问题！~

那这样的话，就需要先写一个判断 2 点是否可以连通的函数了！~

好了我们下面来实现这个函数：

注：我用的是 `VC6.0`

代码:

```
bool LineIfConnect(POINT Point1, POINT Point2, byte ChessData[10][18])
{
    /* 功能   ：检查 Point1,Point2 是否连通(2 点之间连线的数值是不是为 NULL,包括这 2 点自身)
     * 
     * 参数 1  ：第 1 点
     * 参数 2  ：第 2 点
     * 对于参数的要求：Point1.x == Point2.x || Point1.y == Point2.y
     * 参数 3  ：检测 2 点所在的数组(棋盘)
     * 
     * 返回值 ：true 表示可以连通，false 表示不可以连通
     * */
    int i;
    if ((Point1.x == Point2.x) && (Point1.y != Point2.y))    // 横坐标相同，纵坐标不同
    {
        if (Point1.y < Point2.y)    // point1 在 point2 上方
        {
            for (i = Point1.y; i <= Point2.y; i++)    // for 循环检测 2 点之间的所有元素
                if (ChessData[i][Point1.x] != NULL)
                    return false;
        }
        else if (Point1.y > Point2.y)    // point1 在 point2 下方
        {
            for (i = Point2.y; i <= Point1.y; i++)    // for 循环检测 2 点之间的所有元素
                if (ChessData[i][Point1.x] != NULL)
                    return false;
        }
    }
    else if ((Point1.x != Point2.x) && (Point1.y == Point2.y))    // 横坐标不同，纵坐标相同
    {
        if (Point1.x < Point2.x)    // point1 在 point2 左边
        {
            for (i = Point1.x; i <= Point2.x; i++)
                if (ChessData[Point1.y][i] != NULL)
                    return false;
        }
        else if (Point1.x > Point2.x)    // // point1 在 point2 右边
        {
            for (i = Point2.x; i <= Point1.x; i++)    // for 循环检测 2 点之间的所有元素
                if (ChessData[i][Point1.x] != NULL)
                    return false;
        }
    }
    else if ((Point1.x == Point2.x) && (Point1.y == Point2.y))  // 2 点为同一点的情况
        return true;
    else
        return false;

    return true;
}
```

有了上面这个函数，我们就可以真正编写 `qq` 连连看中 2 点是否可以消除了的函数了，代码如下：

代码:

```
bool DecideChessIfRemove(POINT Point1, POINT Point2, byte ChessData[10][18])
{
    /* 功能   ：检查 Point1,Point2 是否可以消除
     * 
     * 参数 1  ：第 1 点
     * 参数 2  ：第 2 点
     * 参数 3  ：检测 2 点所在的数组(棋盘)
     * 
     * 返回值 ：true 表示可以消除，false 表示不可以消除
     * */
    int x1 = Point1.x;
    int y1 = Point1.y;     // Point1 == (x1, y1)
    int x2 = Point2.x;
    int y2 = Point2.y;     // Point2 == (x2, y2)
    int i, j;
    POINT p1, p2;
    byte point1 = ChessData[Point1.y][Point1.x];
    byte point2 = ChessData[Point2.y][Point2.x];
    ChessData[Point1.y][Point1.x] = ChessData[Point2.y][Point2.x] = NULL;      // 2 点在数组的位置设为 NULL
    p1.y = y1;
    p2.y = y2;
    for (i = 0; i <= 18; i++)   // 横坐标在变 下面的两个 for 循环是关键，线段 py1,py2 在平移
    {
        p1.x = i;    // p1 = (i, y1)
        p2.x = i;    // p2 = (i, y2)
        if (LineIfConnect(p1, p2, ChessData))
            if ((LineIfConnect(Point1, p1, ChessData)) && (LineIfConnect(Point2, p2, ChessData)))
            {
                ChessData[Point1.y][Point1.x] = point1;
                ChessData[Point2.y][Point2.x] = point2;
                return true;
            }
    }
    
    p1.x = x1;
    p2.x = x2;
    for (j = 0; j <=10; j++)    // 纵坐标在变 ， 线段 px1,px2 在平移
    {
        p1.y = j;   // p1 = (x1, j)
        p2.y = j;   // p2 = (x2, j)
        if (LineIfConnect(p1, p2, ChessData))
            if ((LineIfConnect(Point1, p1, ChessData)) && (LineIfConnect(Point2, p2, ChessData)))
            {
                ChessData[Point1.y][Point1.x] = point1;
                ChessData[Point2.y][Point2.x] = point2;
                return true;
            }
    }
            
    ChessData[Point1.y][Point1.x] = point1;
    ChessData[Point2.y][Point2.x] = point2;
    return false;   // 没有找到连通的路径
}
```

有了前面的一大堆分析，代码中也有注释，

好了函数的编写告一段落，实际上写程序最为艰难就是调试阶段了

来写一个程序测试一下：

代码:

```
#include <afxwin.h>         // MFC core and standard components
#include <afxext.h>         // MFC extensions
#include <afxdisp.h>        // MFC Automation classes
#include <afxdtctl.h>        // MFC support for Internet Explorer 4 Common Controls
#include <afxcmn.h>            // MFC support for Windows Common Controls


#include <stdio.h>
#include <stdlib.h>
#include "function.h"

int main()
{
    printf("开始 测试\r\n\n\n");

    
    byte ChessData[10][18];
    int i,j;
    for (j = 0; j <= 10; j++)
    {
        for (i = 0; i <= 18; i++)
        {
            ChessData[j][i] = rand()%2;
            printf("%2d  ",ChessData[j][i]);
        }
        printf("\r\n");
    }

    POINT p1;  // p1 的位置
    p1.x = 13;
    p1.y = 5;

    POINT p2; // p2 的位置
    p2.x = 14;
    p2.y = 0;

    if (DecideChessIfRemove(p1, p2, ChessData))
        printf("2 点相连\n\n");
    else
        printf("2 点不相连！~\n\n");

    for (j = 0; j <= 10; j++)
    {
        for (i = 0; i <= 18; i++)
        {
            printf("%2d  ",ChessData[j][i]);
        }
        printf("\r\n");
    }

    return 0;
}
```

程序输出的，图 7：

![](img/7a85e4c028ea5cfebeeeb2512182476a.jpg)

上面是可以联通的情况，那找一个不可以联通的情况试验一下：

代码:

```
#include <afxwin.h>         // MFC core and standard components
#include <afxext.h>         // MFC extensions
#include <afxdisp.h>        // MFC Automation classes
#include <afxdtctl.h>        // MFC support for Internet Explorer 4 Common Controls
#include <afxcmn.h>            // MFC support for Windows Common Controls


#include <stdio.h>
#include <stdlib.h>
#include "function.h"

int main()
{
    printf("开始 测试\r\n\n\n");

    
    byte ChessData[10][18];
    int i,j;
    for (j = 0; j <= 10; j++)
    {
        for (i = 0; i <= 18; i++)
        {
            ChessData[j][i] = rand()%2;
            printf("%2d  ",ChessData[j][i]);
        }
        printf("\r\n");
    }

    POINT p1;  // 这里改了
    p1.x = 7;
    p1.y = 7;

    POINT p2;  // 这里改了
    p2.x = 10;
    p2.y = 6;

    if (DecideChessIfRemove(p1, p2, ChessData))
        printf("2 点相连\n\n");
    else
        printf("2 点不相连！~\n\n");

    for (j = 0; j <= 10; j++)
    {
        for (i = 0; i <= 18; i++)
        {
            printf("%2d  ",ChessData[j][i]);
        }
        printf("\r\n");
    }

    return 0;
}
```

程序输出，图 8：

![](img/bf9b340f06efb7c1e9a01637347da9a2.jpg)

好了大功告成，说明我们的函数还是有些用的

最后感谢大家能看我一篇文章

最后请看图 9：

![](img/9bf3c49cef6da50287ede3dd298bf556.jpg)

谁能给我分析一下，为什么 2 个图标红线处不一样呢？

我没有深入的进行调试，对不起了大家，如果用我的代码写外挂，有什么 `bug`，我有免责申明啊
