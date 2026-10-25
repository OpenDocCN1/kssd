
# [原创]使用x64dbg+VS2015 Spy++去除WinRAR5.40(64位)广告弹框

## 1 学习目标

WinRAR5.40(64位)的弹框广告去除，由于我的系统为x64版本，所以安装了WinRAR（x64）版本。

OD无法调试64位的程序，可以让我熟悉x64dbg进行调试的界面。

其次是这玩意儿真是太蛋疼了，无休止弹广告。

[![](https://bbs.kanxue.com/upload/attach/201707/214974_6iyev1vwbdki6nr.png)](https://bbs.kanxue.com/elink@f53K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6U0k6A6P5h3g2$3x3i4k6%4j5X3c8C8K9e0k6F1M7W2)9J5k6i4m8F1k6H3`.`.)

## 2 破解思路

1）偷梁换柱

修改汇编函数段首为返回值（本次逆向破解采用的方法）

2）NOP掉整个函数内容

## 3 涉及知识

x64dbg工具快捷键与OD无异

`F9`：运行

`bp CreateWindowExW`：在x64dbg底部输入这行命令，对使用`CreateWindowExW`函数的位置断点。

`CreateWindowExW`：该函数创建一个层叠式窗口、弹出式窗口或子窗口。

参数：

```
HWND CreateWindowEx(

    DWORD DdwExStyle,           //窗口的扩展风格

    LPCTSTR lpClassName,        //指向注册类名的指针

    LPCTSTR lpWindowName,       //指向窗口名称的指针

    DWORD dwStyle,              //窗口风格

    int x,                      //窗口的水平位置

    int y,                      //窗口的垂直位置

    int nWidth,                 //窗口的宽度

    int nHeight,                //窗口的高度

    HWND hWndParent,            //父窗口的句柄

    HMENU hMenu,                //菜单的句柄或是子窗口的标识符

    HINSTANCE hInstance,        //应用程序实例的句柄

    LPVOID lpParam              //指向窗口的创建数据)

;
```

## 4 实现流程

【软件名称】：WinRar

【软件版本】：5.4

【外壳保护】：无

【操作系统】：Windows 10

既然是弹出窗口，首先要知道弹窗窗口的窗口类名，我使用的是VS2015里自带的工具Spy++ x64。

![](https://bbs.kanxue.com/upload/attach/201707/214974_hlvdmg7casi2f5p.png)

图1 调出Spy++ x64

[![](https://bbs.kanxue.com/upload/attach/201707/214974_aniy2xuk3ya760w.png)](https://bbs.kanxue.com/elink@bc6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6X3q4F1K9i4V1J5P5s2g2C8x3%4W2S2y4K6j5H3N6#2)9J5k6i4m8F1k6H3`.`.)

图2 使用Spy++64查看WinRAR弹出的窗口类名为RarReminder

通过上诉步骤得到WinRAR的类名为`RarReminder`后，使用`x64dbg`工具载入`WinRAR.exe`。在命令的地方使用断点命令【`bp CreateWindowExW`】，在`CreateWindowEx`函数断下断点。`F9`运行到各个断点时观察广告窗口弹出的状态变化。

![](https://bbs.kanxue.com/upload/attach/201707/214974_0g5o09hestlach7.png)

图3 使用断点命令【`bp CreateWindowExW`】

`F9`运行到出现`RarReminder`字样的地方，`x64dbg`这款工具还具备查看断点触发的次数的功能，通过【断点】选项卡看到断点共触发了30次才到这里。

![](https://bbs.kanxue.com/upload/attach/201707/214974_fpibvtbanpqtlrx.png)

图4 断点触发的次数

在堆栈窗口在`call`指令的地方按回车键返回到用户层函数。

[![](https://bbs.kanxue.com/upload/attach/201707/214974_939tifo8ttzm6z0.png)](https://bbs.kanxue.com/elink@e33K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6U0V1K6z5i4c8A6k6X3)9^5N6s2c8*7L8e0k6*7x3q4)9J5k6i4m8F1k6H3`.`.)

图5 堆栈窗口信息

返回到`00007FF6780AD4E8`这个地址处，向上看会看到“ed1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1i4K6u0W2N6$3W2F1M7X3q4J5i4K6u0W2j5$3!0E0i4K6u0W2j5$3&6Q4x3V1k6K6K9r3!0%4i4K6g2X3y4o6m8Q4x3X3g2Z5N6r3#2D9i4K6y4r3e0q4)9K6c8o6N6Q4x3U0k6S2L8i4m8Q4x3@1u0T1L8q4)9K6c8o6N6Q4x3U0k6S2L8i4m8Q4x3@1u0$3i4K6y4p5y4e0b7H3i4K6t1$3j5h3#2H3i4K6y4n7j5g2)9K6c8o6j5@1i4K6t1$3j5h3#2H3i4K6y4n7M7%4u0U0i4K6y4p5N6%4u0J5i4@1f1J5i4K6R3H3i4K6W2p5i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1#2i4@1u0q4i4K6R3^5i4@1f1$3i4K6V1^5i4K6S2q4i4@1f1$3i4K6V1^5i4@1u0q4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4@1t1&6i4@1u0r3i4@1f1#2i4K6V1I4i4K6S2m8i4@1f1#2i4K6W2o6i4@1t1H3i4@1f1#2i4K6W2p5i4K6R3H3。

汇编函数的代码如下：
