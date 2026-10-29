
# ①普通模式调试

## 案例：以“注册机”APP为例

●打开雷电模拟器安装“注册机”APP，安装完成后打开“注册机”程序。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_9FSDUAZCHSP8CQ8.png)

●点击左下角的“windows”图标,输入`cmd`命令,打开命令窗口。

输入“`adb shell ps`”命令，查看当前设备的进程信息,通过浏览进程信息,找到“注册机”程序的进程`pid`,并且记录下来。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_D27QHAWQQZKJ2QS.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_ZEP4FRNBGHXPNSK.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_4CSAUVNCBXUXWS5.png)

●再次点击左下角的“windows”图标，输入`cmd`命令，打开新的命令窗口。输入“`adb forward tcp:8700 jdwp:2524`”命令，进行端口转发，其中的“`tcp`”是之前配置调试环境时指定的端口号，“`jdwp`”这里指的是我们要调试的程序的进程`pid`，在此之前已经记录下来。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_7M88D4NWARU5R29.png)

●在菜单中找到“`debug`”调试按钮，点击该按钮开始调试，出现下面的“socket”信息，这就说明附加成功了

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_MVTW26FGDBVX6XH.png)

附加上了之后呢我们去进行下断，找到`MainActivity.smali`文件里面的`checkSN`方法，在方法前面的行号处点击即可下断。

有小伙伴可能有点懵逼了……为什么要在这里下段了？

因为`jeb`调试时，已经逆向分析过这个APP程序了。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_F66R3ESKNTQVT7S.png)

●我们会发现，没有立即断下来，这是因为程序还没有调用该方法 ，接下来，我们去操作程序，让他调用该方法，在这里随便输入“用户名”，但注册码必须是16位的，这点要记住，否则，会影响到后面的调试分析。输入相应的信息后了直接点击 “注册”按钮，发现程序被断下来了，出现了参数寄存器的值。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_BWNK34H8Z8XVJ2B.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_Q4M7X3WJJUA9K6H.png)

●此刻呢，我们会使用到一些按钮和快捷键进行后续操作：

`F9`：跳到下一个断点

`F8`：单步步过

`F7`：单步步入

`Alt+shift+F7`:`强制进入`

`Shift+F8`:`单步步出`

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_5SESCZD2YUCXN33.png)

●我们可以一直按`f8`或者可以在这个位置下个断，按`f9`程序就会在这个断点断下来。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_4ZXZJSUFCJ26PN4.png)

●此刻我们发现，正确的注册码已经出现，复制注册码，然后停掉`as`，去模拟器里面粘贴注册码，点击注册发现注册成功了，说明我们已经获取到正确注册码。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_7Z2NQPGUZPZBYEV.png) ![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_PMNJQQW6FYQVDJ6.png)

# ② 以调试模式启动调试

●在这里呢，设置断点和普通模式是一样的。

首先我们先打开模拟器，然后再`cmd`里面输入“`adb shell am start -D -n 应用程序包名/应用程序入口界面`”命令回车，我们可以看到模拟器弹出了一个弹窗，等待被调试。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_548CW59DG87U3HF.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_JDPTH974D27S354.png)

●此时，我们需要打开`cmd`命令窗口，进行端口转发，输入“`adb forward tcp:8700 jdwp:2700`”回车转发端口。

●转发成功后，点击调试按钮开始调试。

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_Z9Z63TYS4FVFMKD.png)

●同样去操作程序，点击注册，马上就断下来

![图片描述](https://bbs.kanxue.com/upload/attach/202010/905378_R8WCX8TFDQ7MGK7.png)

调试成功就算是完成了环境搭建了。

以上就是`AndroidStudio+smalidea`插件调试的过程、步骤以及需要注意的事项，小伙伴们看完是不是觉得很简单呢？快动手试试吧。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
