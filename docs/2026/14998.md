
# [原创]AntiDebug：实战讲解

Anti Debug主要是通过各种函数去确定当前进程是否处于被调试的状态。

## 一.IDA里面静态分析so文件

1.将文件拖入`jdax-gui`中，进行静态分析，会发现`OnCreate`里面没多少内容，并且上面加载了so库，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_BKNHUUBE4RMSH93.png)

2.将SO文件拖入，找到`JNI_OnLoad`，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_W8ARQQFNXYFTAAX.png)

3.按F5查看伪代码，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_D7V78F47UZZFAPT.png)

4.除了动态注册这个参数外，还有两个参数的传递，其中一个是简单的`if`判断，它使用的是或运算符，只要其中有一个成立，就会成功执行，返回`return-1`，相反则返回65540，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_A2NDGTN6WFDWU25.png)

5.进入第一个`anti_time()`，分析逻辑，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_3XY2STNQRVPMQXK.png)

6.先获取线程ip给`v0`，前面两个`gettimeofday()`的函数，传入两个值进行操作，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_AFBTMB95EXWFEAU.png)

7.查看`gettimeofday()`引用，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_6E827GWVQU2EADJ.png)

8.点击进行查看，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_H85TGPHJYAZNWDB.png)

9.进来后，查看关键字眼，是和时间相关的，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_76E6MUEZ3KNPMZP.png)

10.`v3`和`tv`分别是两个不同的时间点，分别获取两个时间，`lv_sec`是秒数，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_4253WB323UMV5YJ.png)

11.获取两个间隔做差值放到`v1`寄存器，然后继续`if`判断，小于零则给上一层的`if`返回0，反之，直接`kill()`，返回1，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_P7YU57UST5JHZAM.png)

12.查看并分析`if`的简单判断逻辑，下面有一个`for`循环，里面又是各种嵌套，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_HM3422GAJV9YS3G.png)

13.`for`循环结束后，返回上一层的布尔值，并不是在`for`的外层，如果`for`结束后，执行`++v4`操作，会返回默认值`false`（布尔值的默认值是`false`），如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_FKSH2AAF66BNQJA.png)

14.关键点就在于`for`循环里面的分析，查看`for`循环，发现有一个`break`，在C语言中，`for`循环里面的`break`就是跳出`for`循环的意思，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_T8C9MW3D6QHXTQG.png)

15.判断成立后会直接`break`，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_A5427KWDNFPS2Z6.png)

在汇编里面将它进行修改，直接`break`，就可以直接给上一层返回一个`false`。

16.分析一下，如果`v4`大于等于`v6`，`v4`上面初始值为0，而`v6`是一个获取得到的返回值，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_8H8J2SPEKKNECCG.png)

17.这个位置显示的是什么？如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_M3G67NRNTZFPUX2.png)

18.查看`v3`，有`v2`赋值，`v2`是一个方法的返回值，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_AX4MH8YRMV6786M.png)

19.回到`break`的判断，`v4`固定值0，`v6`是一个返回值+44，然后整体取地址，这个肯定是大于零的（因为地址不可能是负数），也就是说，这个`if`判断永远小于零，`break`永远不会执行。

**那么，修改方法有以下几种：**

（1）直接把`if`判断`nop`掉，也就是进来`for`循环就直接`break`；

（2）把判断的大于等于改成小于，也可以执行到`break`；

（3）直接改动`break`指令的位置，比如放到`for`循环外面。

20.继续分析逻辑，这里面有几个linux编程涉及的函数，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_EZ4WC4MM9827FRN.png)

`pthread_self()`：获取自己的线程id；

`pipe()`管道：实现进程通信；

`pthread_create()`：创建线程，它有四个参数，查看它的第三个参数，这个是自己的定义的程序，做一些其他操作。

21.同理，我们分析的这个也是在第三个参数，做了一些获取线程的操作，也就是这个函数，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_TWMB9SB8PESYRZF.png)

22.继续分析这个函数，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_7UUJMBKQVD6FQPV.png)

23.点击进入，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_F44TVVZXAFSNAGP.png)

24.它的作用就是进出函数时，堆栈是否平衡和数据是否被修改，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_F6UBE5BKBQUVBHV.png)

25.`v0`是获取当前进程id，那么这个打印的结果就是当前的进程，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_HQGC8ACWU4MJHV3.png)

26.调用`fork()`函数，父进程调用`fork()`函数可以创建子进程，那么这个返回值就是子进程，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_SNUH4QTSM4M73YE.png)

27.然后一个`if`判断，如果创建成功，把值最终传给了`childpid`这个变量，紧接着`else`，如果没成功，就关闭，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_ZPC76D4RTJVP2N7.png)

28.`else`下面还有其他的操作，就是静态so层分析，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_MBN9DWCN5VDHAFE.png)

## 二： IDA动态调试

1.在`jni_onload()`处下断点，然后运行到此处，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_9RNJ7Z3V67AQCBS.png)

2.这几个函数就是有反调试函数的地方，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_Y5MXPQN8HVWVV7C.png)

3.直接把这三个函数`nop`掉，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_YAGQGUETGXJJXCW.png)

4.然后F4直接原型到后面的指令，就完美的过掉了反调试，如图7.31所示。

![](https://bbs.kanxue.com/upload/attach/202009/880306_J3WDFHA3QXC8XMR.png)

## 小结：

本次分享，是先从java层简单分析到so层，然后静态分析了so里面反调试函数的逻辑，最后通过动态调试演示了如何过掉反调试。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
