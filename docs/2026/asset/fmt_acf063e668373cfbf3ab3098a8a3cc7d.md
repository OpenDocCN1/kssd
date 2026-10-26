
# [原创]某圈机器人仿真程序提升得分和排名的逆向分析（一）

摘要:  通过对软件的逆向分析，使用MS IL指令手工改变程序逻辑和数据，形成软件的Patched文件，达到提升得分和排名的功能。

关键词：dnSpy,IL,软件逆向

免责声明：本文仅作为学习逆向技术分析、研究所用，一切由于其他非法用途而引起的法律纠纷与本作者无关。

某圈的机器人仿真程序是目前学生参加中小学机器人竞赛的热门软件，学生利用该仿真软件进行机器人的搭建和程序的编写在虚拟的场景中进行各种项目的比赛，取得成绩和排名从而获得相应的奖项。

从游戏辅助的角度获得启发，通过实践发现可以手工修改软件的IL指令提升比赛的得分和排名。本文是笔者对该软件的逆向分析过程，主要是一些逆向工具的使用和.NET下IL指令的编写，因为IL指令的使用介绍网上不是太多，在具体的逆向过程中，由于本人水平有限还是遇到不少问题，另外能不能在不修改原程序的情况下通过静态注入实现相同的功能，笔者没有进行相应的研究，所以对于文中的错误或问题还请高手不吝赐教。

## 一、工具的准备

1. dnSpy：开源的.Net程序逆向工具，最新版在github上有下载；
2. de4dot：一款.Net程序去壳反混淆的工具，这里用的是 2.0.3版本
3. DotNet Id : 一款.Net 查壳工具；
4. IDA: 大名鼎鼎，实在是居家旅行逆向必备之工具；
5. Net IL命令查询器 :文中用来参考IL指令的功能

前面4个工具均可以到看雪网站0dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3k6h3c8A6P5g2)9J5k6h3y4G2L8g2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5q4!0n7c8q4!0n7c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2!0m8c8W2!0n7z5g2!0q4y4q4!0n7b7g2)9^5c8f1&6W2N6l9`.`. IL命令查询器，请百度下载安装；

## 二、查壳去混淆

安装某圈的仿真平台后，找到安装目录下的`IRobotQ.exe`文件，先复制出一份，以备逆向破坏后进行还原，然后将该文件拖入`IDA`，发现是.NET编写（如图1）。

![](https://bbs.kanxue.com/upload/attach/201904/766550_9QKMNYDNUD8AP73.jpg)

图1

![](https://bbs.kanxue.com/upload/attach/202002/766550_H96JUMZB96V7T7K.jpg)

图2

在Function name窗口中发现程序应该是经过名字混淆处理了（如图2），使用`de4dot`对`IRobotQ.exe`进行去混淆，在操作前把`IRobotQ.exe`复制到一个临时文件夹下，原因貌似当前版本的`de4dot`不支持含有空格的文件路径，然后输入命令行（如图3）得到一个去混淆后的`IRobotQ-cleaned.exe`文件。从`de4dot`对文件的处理过程中可以发现原来的`IRobotQ.exe`使用了SmartAssembly进行混淆。由于本文的重点不是混淆技术，所以不再赘述，有兴趣的读者可以研究下.Net名称混淆的相关技术。

![](https://bbs.kanxue.com/upload/attach/201904/766550_9JTX3HDP3ES5NW3.jpg)

图3

## 三、修改`IRobotQ.IRQ_ResultBoard`信息

首先运行平台程序，登录后挑选相应的机器人和运行程序，在场景中启动机器人，最后出现如下对话框显示当前取得的成绩（如图4），我们的首要目标就是改变对话框中的“得分”，“基础分”，“时间奖励分”，“避让行人”，“安全会车”，“飞车”的相应数据。为什么要处理该对话框中的数据呢？应为当我们提交成绩时，平台会将当前的对话框信息以截屏的方式保存在`IRobotQ3D\DriverTemp`目录下作为成绩的证据。

![](https://bbs.kanxue.com/upload/attach/201904/766550_2X8YSSE5JZU3GQY.jpg)

图4

接下来的问题是如何找出这个”对话框”，首先想到的是获取窗口句柄用OD的下断，但马上就意识到不行，应为游戏中的界面都是“画”出来的，无法通过传统的方法下断，所以就想到使用IDA中对去混淆的文件进行关键字符串查找，使用IDA打开去混淆后的“IRobotQ-cleaned.exe”，在Text search对话框中输入”得分”显示的结果如图5：

![](https://bbs.kanxue.com/upload/attach/201904/766550_VSR2QGSNJ9244UJ.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/766550_RMDYWR4A74XDUDB.jpg)

图5

这里我们看到了一个重要的信息`IRobotQ.IRQ_ResultBoard`类，根据字面理解应该是跟图4的对话框有关，Bing go！现在就可以使用dnSpy工具进行源码级的分析了，将`IRobotQ-cleaned.exe`拖入dnSpy找到`IRobotQ.IRQ_ResultBoard`类下的`saveEvidence()`函数（如图6），Ctrl+R进行分析，逐步展开“被使用”，可以发现调用该函数的各个地方如图（7）。

![](https://bbs.kanxue.com/upload/attach/201904/766550_CT9E6R4VMRU8K3B.jpg)

图6

![](https://bbs.kanxue.com/upload/attach/201904/766550_RVTMYS6KSE9JBMF.jpg)

图7

这里出现了关键的两个函数`savectl_EventMouseButtonReleased`和`Show`，我们点击show函数，dnSpy自动反编译，对它的分析如下：

```csharp
publicstaticvoidShow()
{
   //加载对话框
    IRQ_ResultBoard.LoadScoreUISetting();
   //设置对话框可见
    IRQ_ResultBoard.SetBoardVisible(true);
   //判断列表是否已经创建，没建的建立Grid，并清空对话框中原有内容
    if (!IRQ_ResultBoard.m_CreatedList)
    {
        IRQ_ResultBoard.CreateGrid();
        IRQ_ResultBoard.ClearAllStaticTextsCaption();
    }

    string[] array;

  //如果m_ListText列表项有数据
    if (IRQ_ResultBoard.m_ListTexts.Count > 0)
    {
 //给array设置表格列名
 array = IRQ_ResultBoard.m_ListTexts[0];
    }
    else
    {
        array = newstring[IRQ_ResultBoard.m_Column];
        for (int i = 0; i < IRQ_ResultBoard.m_Column; i++)
        {
            array[i] = string.Empty;
        }
    }

   //设置提交次数
    StaticText staticText = UIB.Find("sim_ScoreListRace_LeaveNumber") asStaticText;
    if (staticText != null)
    {
        if (Class1890.enum116_0 == Enum116.const_0 && Class1868.GetGame().MessionInfo.method_0())
        {
            staticText.Caption = Class1890.class1885_0.CurrentUser.GetAvilibleSubmitCount().ToString();
        }
        else
        {
            staticText.Caption = "无限制";
        }
    }

   //设置提交按钮的有效性
    if (Class1890.enum116_0 == Enum116.const_0 && !Class1868.GetGame().IsReset)
    {
        UIB.SetVisible("sim_ScoreListRace_TiJiao", true);
    }
    else
    {
        UIB.SetVisible("sim_ScoreListRace_TiJiao", false);
    }

  //设置排名按钮
    UIB.SetVisible("sim_ScoreListRace_PaiMing", Class1890.enum116_0 == Enum116.const_0);

 //设置表格的列名
    for (int j = 2; j < IRQ_ResultBoard.m_Column; j++)
    {
        string strName = string.Format("sim_ScoreList_Def_{0}_{1}", 0, j);
        Widget widget = UIB.Find(strName);
        if (widget != null)
        {
            widget.Caption = array[j];
        }
    }

 //添加表中每行第一列数据
    for (int k = 1; k < IRQ_ResultBoard.m_Row; k++)
    {
 //在表格中添加序号
        Widget widget2 = UIB.Find("sim_ScoreList_Name" + k.ToString());
        if (widget2 != null)
        {
            widget2.Caption = IRQ_ResultBoard.m_ListTexts[k][0];
        }

   //开始添加昵称
        widget2 = UIB.Find("sim_ScoreList_Score" + k.ToString());
        if (widget2 != null)
        {
            widget2.Caption = IRQ_ResultBoard.m_ListTexts[k][1];
        }
    }

 //添加表格每行第二列开始的数据，即添加“得分”，“基础分”等数据
    for (int l = 1; l < IRQ_ResultBoard.m_Row; l++)
    {
        for (int m = 2; m < IRQ_ResultBoard.m_Column; m++)
        {
            string strName2 = string.Format("sim_ScoreList_Def_{0}_{1}", l, m);
            Widget widget3 = UIB.Find(strName2);
            if (widget3 != null)
            {
                widget3.Caption = IRQ_ResultBoard.m_ListTexts[l][m];
            }
        }
    }
}
```

从中可以发现取得的数据来自`m_listTexts`这个`List`对象，所以我们现在需要在该函数中的头部改变`m_listTexts`列表项的值，从而达到改变“得分”，“基础分”等这些列表项的值，从代码中可以得出:`m_listTexst[1][2]`存放“得分”数据，`m_listTexts[1][5]`存放”基础分”数据，`m_listTexts[1][6]`存放”时间奖励分”数据，`m_listTexts[1][7]`存放”避让行人”，`m_listTexts[1][9]`存放”安全会车”数据，`m_listTexts[1][10]`存放”飞车”数据。

接下来就是编写IL代码，打开原文件`IRobotQ.exe`（注意是原文件哦，不是去混淆的`IRobotQ-cleaned.exe`）找到`IRQ_ResultBoard`类在`show`函数上右击，选择“Edit IL Instructions”菜单或者（Ctrl+E），从000B开始添加IL指令如图8所示：

![](https://bbs.kanxue.com/upload/attach/201904/766550_XDU58CRAMEH7K53.jpg)

```il
ldsfl class [mscorlib]System.Collections.Generic.List`1<string[]> [IRobotQ.exe]IRobotQ.IRQ_ResultBoard::m_ListTexts     //加载m_listTexts字段到栈顶

ldc.i4.1                                                                                    //加载1到栈顶

callvirtinstance !0 class [mscorlib]System.Collections.Generic.List`1<string[]>::get_Item(int32)                //取得m_listTexts第1行数据并到栈顶

ldc.i4.2                                                                                    //加载2到栈顶

ldstr"333.61"                                                       //将得分数据333.61送栈顶

stelem.ref
```

.Net IL指令的特点是以堆栈为基础进行操作，程序中用到的常量、变量、参数、属性等都需要压入堆栈进行相应的使用，而对变量、属性等的赋值操作通过出栈来完成，上面的`callvirt`指令调用`list`实例的`get_Item(int32)`虚方法，在调用前首先要将`IRQ_ResultBoard`类下的`m_ListTexts`实例入栈，然后将`get_Item(int32)`的参数1通过`ldc.i4.1`指令入栈,当执行``callvirt instance !0 class[mscorlib]System.Collections.Generic.List`1<string[]>::get_Item(int32)``后栈顶只剩下`get_Item(1)`返回值；接下来通过`ldc.i4.2`入栈，指定对`m_ListTexts`进行操作的元素下标，接着`ldstr "333.61"`载入要设置的“得分”值，最后用`stelem.ref`完成对`m_listTexts[1][2]`赋值操作。图8只进行了“得分”项的设置，其他“基础分”，“时间奖励分”等的设置可以通过”复制”“粘贴”的方法来完成，最后的代码如下所示：

```il
3       000B         ldsfld        class[mscorlib]System.Collections.Generic.List`1<string[]>[IRobotQ.exe]IRobotQ.IRQ_ResultBoard::m_ListTexts

4       0010         ldc.i4.1

5       0011         callvirt      instance !0 class[mscorlib]System.Collections.Generic.List`1<string[]>::get_Item(int32)

6       0016         ldc.i4.2

7       0017         ldstr          "333.61"

8       001C         stelem.ref

9       001D         ldsfld        class[mscorlib]System.Collections.Generic.List`1<string[]>[IRobotQ.exe]IRobotQ.IRQ_ResultBoard::m_ListTexts

10    0022         ldc.i4.1

11    0023         callvirt      instance !0 class[mscorlib]System.Collections.Generic.List`1<string[]>::get_Item(int32)

12    0028         ldc.i4.3

13    0029         ldstr          "0"

14    002E         stelem.ref
```

