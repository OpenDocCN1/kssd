
# [原创]Android 自动化测试 Monkey、Monkeyrunner

软件测试是软件产品开发过程中的重要环节，对于保重软件质量具有重要的意思。手动测试和自动化测试是软件测试的两个基本手段。本文主要介Android – 自动化测试

Android平台上提供了一些自动化测试工具：`Monkey`、`Monkeyrunner`。

## Monkey、Monkeyrunner 概述

### 一、Monkey 概述

`monkey`可以直接在`XP/WIN7`上运行，只要`XP/WIN7`上能用`adb`，即可使用。

`Monkey`测试是Android平台自动化测试的一种手段，通过`Monkey`程序模拟用户触摸屏幕、滑动`Trackball`、按键等操作来对设备上的程序进行压力测试，检测程序多久的时间会发生异常,`Monkey`测试是一种为了测试软件的稳定性、健壮性的快速有效的方法。

`Monkey`程序由Android系统自带，使用Java语言写成，在Android文件系统中的存放路径是：`/system/framework/monkey.jar`。

`Monkey.jar`程序是由一个名为"`monkey`"的`Shell`脚本来启动执行，`shell`脚本在Android文件系统中的存放路径是：`/system/bin/monkey`；这样就可以通过在`shell`窗口中执行: `adb shell monkey {+命令参数}`来进行`Monkey`测试了。

### 二、Monkeyrunner 概述

1、`monkeyrunner`工具提供了一个`API`，使用此`API`写出的程序可以在Android代码之外控制Android设备和模拟器。通过`monkeyrunner`，您可以写出一个`Python`程序去安装一个Android应用程序或测试包，运行它，向它发送模拟击键，截取它的用户界面图片，并将截图存储于工作站上。

`monkeyrunner`工具的主要设计目的是用于测试功能/框架水平上的应用程序和设备，或用于运行单元测试套件，但您当然也可以将其用于其它目的

`Monkeyrunner`文件在`SDK`目录的`tools`子目录下，运行时，如果没有指定环境变量，需到`tools`目录下方可执行

`Monkeyrunner`支持自己编写插件,控制事件,随时截图,简而言之,任何你在模拟器/设备中能干的事情,`MonkeyRunner`都能干,而且还可以记录和回放!!!

2、`monkeyrunner`工具为Android测试提供了以下特性：

多设备控制：`monkeyrunner API`可以跨多个设备或模拟器实施测试套件。您可以在同一时间接上所有的设备或一次启动全部模拟器（或统统一起），依据程序依次连接到每一个，然后运行一个或多个测试。您也可以用程序启动一个配置好的模拟器，运行一个或多个测试，然后关闭模拟器。

功能测试： `monkeyrunner`可以为一个应用自动贯彻一次功能测试。您提供按键或触摸事件的输入数值，然后观察输出结果的截屏。

回归测试：`monkeyrunner`可以运行某个应用，并将其结果截屏与既定已知正确的结果截屏相比较，以此测试应用的稳定性。

可扩展的自动化：由于`monkeyrunner`是一个`API`工具包，您可以基于`Python`模块和程序开发一整套系统，以此来控制Android设备。除了使用`monkeyrunner API`之外，您还可以使用标准的`Python os`和`subpress`模块来调用如`adb`这样的Android工具。

### 三、Monkey和Monkeyrunner工具区别

`Monkey`：`Monkey`工具直接运行在设备或模拟器的`adb shell`中，生成用户或系统的伪随机事件流。

`monkeyrunner`：`monkeyrunner`工具则是在工作站上通过`API`定义的特定命令和事件控制设备或模拟器。

## Monkey 的基本使用方法

1、可以通过开发机器上的命令行或脚本来启动`Monkey`。

由于`Monkey`运行在模拟器/设备环境中，所以必须用其环境中的`shell`来进行启动。可以通过在每条命令前加上`adb shell`来达到目的，也可以进入`Shell`后直接输入`Monkey`命令。基本语法如下：

```
adb shell monkey [options]
```
或

```
adb shell
#monkey
```

如果不指定`options`，`Monkey`将以无反馈模式启动，并把事件任意发送到安装在目标环境中的全部包。下面是一个更为典型的命令行示例，它启动指定的应用程序，把调试等级加了1，并向其发送500个伪随机事件：

```
adb shell monkey -p your.package.name –v 500
```

2、`Monkey` 命令可简单分为如下四类：

基本配置选项: 如设置尝试的事件数量,反馈信息的级别。

运行约束选项: 如设置只对单独的一个包进行测试。

事件类型和频率: 如设置触摸屏事件的发生频率。

调试选项，如设置忽略应用程序异常，继续向系统发送事件，直到计数完成

3、`Monkey` 命令解析：

![](https://bbs.kanxue.com/upload/attach/201406/635615_tvljyvmsdmcmd6s.jpg)

## Monkey runner 的基本使用方法：

### 1、Monkeyrunner 环境搭建

![](https://bbs.kanxue.com/upload/attach/201406/635615_zyy5wdex8no3kde.jpg)

### 2、下载安装Python

下载网址：`349K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3P5i4c8Z5L8$3&6Q4x3X3g2G2M7X3N6Q4x3V1j5`.

### 3、环境变量配置

`;C:\Python27`（安装目录复制到`Path`的后面,注前面的“;”不要忘记哦）环境变量路径以`Win7`为例：计算机 -> 属性 -> 高级系统设置 -> 环境变量 -> 系统变量 -> 找到`Path`

### 4、`monkeyrunner` API于`com.android.monkeyrunner`包中包含三个模块：

`MonkeyRunner` ：一个为`monkeyrunner`程序提供工具方法的类。这个类提供了用于连接`monkeyrunner`至设备或模拟器的方法。它还提供了用于创建一个`monkeyrunner`程序的用户界面以及显示内置帮助的方法。

`waitForConnection (float timeout, string deviceId)`

`sleep (float seconds)`

`help (string format)`

`MonkeyImage` ：表示一个截图对象。这个类提供了截图、将位图转换成各种格式、比较两个`MonkeyImage`对象以及写图像到文件的方法。

`MonkeyImage`创建对象的方法如下：

`newimage =MonkeyDevice.takeSnapshot()`

`writeToFile (string path, string format)`

`MonkeyDevice` ：表示一个设备或模拟器。这个类提供了安装和卸载程序包、启动一个活动以及发送键盘或触摸事件到应用程序的方法。您也可以用这个类来运行测试包。

`MonkeyDevice`的常用方法：

a.`Touch (x坐标， y坐标，动作)`

可以通过开启`Pointer location`在手机上直接查看坐标点.

b.`Press('按键值', 动作)`

动作包括：`DOWN, UP, DOWN_AND_UP`。大部分时间用`DOWN_AND_UP`来模拟按键、松手的过程。比如要进行一个返回键操作：

```
device.press('KEYCODE_BACK',MonkeyDevice.DOWN_AND_UP)
```

c.`Drag((x, y)，(x, y)，持续时间，轨迹点)`

按键值为`Andriod`键值表中有的值`drag((x, y)，(x, y)，持续时间，轨迹点)`

第一组是起始坐标，第二组终点坐标。这里的坐标值要通过`hierarchyviewer`工具来获取，轨迹点类似采样

d.启动`Activity`：`startActivity('string')`

在`python`程序中，您将以`Python`模块的形式使用这些类。`monkeyrunner`工具不会自动导入这些模块。您必须使用类似如下的`from`语句：

```
from com.android.monkeyrunner import MonkeyRunner,
MonkeyDevice, MonkeyImage
```

您可以在一个`from`语句中导入超过一个模块，其间以逗号分隔。

### 5、运行`monkeyrunner`

`monkeyrunner`命令的语法为：

```
monkeyrunner -plugin <程序文件名> <程序选项>
```

1、启动一个Android系统的模拟器，即为命令中`device`

2、 将`cmd`命令定位到`android SDK`的路径下的`tools`目录

3、输入命令"`monkeyrunner`"，回车，进入`Shell`命令交互方式

导入`monkeyrunner`所要使用的模块： A.直接在`shell`命令中输入（然后回车）：

```
from com.android.monkeyrunner import MonkeyRunner,
MonkeyDevice,MonkeyImage
```

B.输入`device = MonkeyRunner.waitForConnection()`，然后回车

### 6、一个简单的`monkeyrunner`程序实例

以下为一个简单的`monkeyrunner`程序，它将会连接到一个设备，创建一个`MonkeyDevice`对象。使用`MonkeyDevice`对象，程序将安装一个Android应用包，运行其中一个活动，并向其发送按键事件。程序接下来会将结果截图，创建一个`MonkeyImage`对象，并使用这个对象截图将保存至`.png`文件。

```
# 导入此程序所需的monkeyrunner模块
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice MonkeyImage

# 连接当前设备，返回一个MonkeyDevice对象
device = MonkeyRunner.waitForConnection()

# 安装Android包，注意，此方法返回的返回值为boolean，由此您可以判断安装过程是否正常
device.installPackage('myproject/bin/MyApplication.apk')

# 运行此应用中的一个活动device.startActivity(component='com.example.android.myapplication.MainActivity')

# 按下菜单按键
device.press('KEYCODE_MENU','DOWN_AND_UP')

# 截取屏幕截图
result = device.takeSnapshot（）

# 将截图保存至文件
result.writeToFile('myproject/shot1.png','png')

#时延(秒)：
MonkeyRunner.sleep(3)

#滑动屏幕(Python脚本)：
for i in range(1,70):device.drag(start,end,0.1,10)

#触击屏幕：
device.touch(94,34,'DOWN_AND_UP')

# 运行此应用中的一个活动device.startActivity(component='com.example.android.myapplication.MainActivity')

# 按下菜单按键
device.press('KEYCODE_MENU','DOWN_AND_UP')

# 截取屏幕截图
result = device.takeSnapshot（）

# 将截图保存至文件
result.writeToFile('myproject/shot1.png','png')
```

### 7、`Monkeyrunner`自动化测试的记录和回放

`MonkeyRunner`工具中提供了记录与回放的功能，可以通过`monkey_recorder.py`和`monkey_playback.py`两个`python`文件来实现。此两个文件存放在：`android`源代码 -> `monkeyrunner`源代码 -> `scripts`文件夹。

1）记录

a、启动模拟器或通过`USB`连接手机

b、打开命令行提示符，并且定位到`Tools`文件下输入：

```
monkeyrunner monkey_recorder.py
```

按下回车键

如下图：

![](https://bbs.kanxue.com/upload/attach/201406/635615_dfuab7p89g6k2sd.jpg)

录制界面菜单介绍：

![](https://bbs.kanxue.com/upload/attach/201406/635615_p8zeqkwsvrvh471.jpg)

注意：导出的脚本要与`monkey_recorder.py`和`monkey_playback.py` 在同一目录下，即`Tools`文件夹下。

![](https://bbs.kanxue.com/upload/attach/201406/635615_g3q8aa8p0cugkdb.jpg)

2）回放

脚本回放需要使用另外一个脚本 `monkeyplayback.py`

执行步骤：

a、启动模拟器或通过`USB`连接手机

b、打开命令行提示符，并且定位到`Tools`文件下输入：

```
monkeyrunner monkeyplayback.py  “sound”
```

按下回车键

附录：脚本![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

1、`monkeyrecoder.py`

```
＃Usage: monkeyrunner monkeyrecorder.py
＃monkeyrecorder.py
from com.android.monkeyrunner import MonkeyRunner
from com.android.monkeyrunner.recorder import MonkeyRecorder
device = MonkeyRunner.waitForConnection()
MonkeyRecorder.start(device)
```

2、`monkeyplayback.py`

```
import sys
from com.android.monkeyrunner import MonkeyRunner
CMD_MAP = {
    "TOUCH":lambda dev,arg:dev.touch(**arg),
    "DRAG":lambda dev,arg:dev.drag(**arg),
    "PRESS":lambda dev,arg:dev.press(**arg),
    "TYPE":lambda dev,arg:dev.type(**arg),
    "WAIT":lambda dev,arg:MonkeyRunner.sleep(**arg)
    }
# Process a single file for the specified device.
def process_file(fp,device):
    for line in fp:
        (cmd,rest) = line.split("|")
        try:
            rest = eval(rest)
        except:
            print"unable to parse options"
            continue
        if cmd not in CMD_MAP:
            print"unknown command:" + cmd
            continue
        CMD_MAP[cmd](device,rest)
def main():
    file = sys.argv[1]
    fp = open(file,"r")
    device = MonkeyRunner.waitForConnection()
    process_file(fp,device)
    fp.close();
if __name__=="__main__":
    main()
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Monkeycmd.jpg](javascript:void(0);)

  （34.40kb，8次下载）

- [Snap8.jpg](javascript:void(0);)

  （18.48kb，8次下载）

- [1.jpg](javascript:void(0);)

  （26.01kb，9次下载）

- [2.jpg](javascript:void(0);)

  （29.10kb，9次下载）

- [3.jpg](javascript:void(0);)

  （13.80kb，7次下载）
