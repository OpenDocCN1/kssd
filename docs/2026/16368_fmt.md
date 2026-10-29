
# [原创]Android逆向之CrackMe01.apk破除反调试及取得明文密码

## 1.目的

记Android逆向学习之路。

## 2.对象

`CrackMe01.apk`（见附件）（无壳，存在反调试和密码验证逻辑）

## 3.步骤

### 3.1 概述

分析apk java层代码-->安装apk并开启IDA监听端口-->IDA以Debug模式动态调试apk-->破除反调试-->取得明文密码

### 3.2 环境搭建

工欲善其事，必先利其器

工具：`Windows10`；`IDA7.0(Window)`；`ajdx-gui`；`DDMS`（`adt-bundle-windows-x86_64_20140101`里面`tools`中的`monitor.bat`，先打开）；ARM架构模拟器（这里是`armeabi-v7a`，SDK里面的相关组件可以创建模拟器）

### 3.3 分析apk java层代码

把apk拖进`ajdx-gui`中

![图片描述](upload/attach/202007/894007_KZTASG2VX8C2Q5J.png)

![图片描述](upload/attach/202007/894007_GWKTEE7CBSV75NF.png)

可得知密码验证在so层

### 3.4 安装apk并开启IDA监听端口

使用`adb install` 电脑apk路径 安装`CrackMe01.apk`，界面如下：

![图片描述](upload/attach/202007/894007_UZFSQW8BCC7X4MN.png)

开启IDA监听端口（要事先把IDA-dbgsrv目录下的`android-server`置到模拟器中，这里是`/data/local/tmp`下，修改名称为`as`并赋给可执行权限），用`./as`开启端口监听，默认端口是`23946`，app崩溃

![图片描述](upload/attach/202007/894007_K2XNX8QNRN3JS7B.png)

猜测可能存在端口检测，更改端口为`31928`，命令：`./as -p31928`

成功过app调试端口检测，接着进行端口转发

![图片描述](upload/attach/202007/894007_6BTRCYYAEQQU3X3.png)

### 3.5 IDA以Debug模式动态调试apk

先打开`DDMS`，用于模拟器进程监听。

打开app主界面，使用`adb shell dumpsys activity top`命令获取当前界面信息

![图片描述](upload/attach/202007/894007_55WNNFQ2EC4V2JH.png)

使用`adb shell am start -D -n com.yaotong.crackme/.MainActivity`命令以Debug模式启动app指定界面

![图片描述](upload/attach/202007/894007_8JJ3W9969MM47NN.png)

然后打开IDA Debugger附加`CrackMe01.apk` app进程

![图片描述](upload/attach/202007/894007_G6UD4NQRH6XPAWN.png)

![图片描述](upload/attach/202007/894007_H98KB9A2UJ9B33U.png)

![图片描述](upload/attach/202007/894007_WU5Q4ZVD82VS8H3.png)

勾选Debug选项（加载so要挂起载入）

![图片描述](upload/attach/202007/894007_AYCV67VAN7PWN23.png)

![图片描述](upload/attach/202007/894007_SSN5MTS4Y98F724.png)

IDA Debug模式动态调试配置完成。

为什么不用直接动态调试？经发现，该apk运行时会自己附加一个子进程，IDA中直接动态调试会显示两个Pid的该进程（可以进入`/proc/进程id/status`进行信息查看，子进程的`TracePid`就是父进程的`Pid`），由于一个进程只能被附加一次，因此不能再对子进程进行附加，对父进程调试也发生了错误，而Debug模式动态调试则没有问题。

### 3.6 破除反调试

`F9`然后执行`jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=端口号`，端口号是DDMS对应进程的端口，且执行后图标变成绿色。

加载`libwolf.so`进来（进程现在挂起状态），找到这个so的导出函数（`Debugger-Module list`）

![图片描述](upload/attach/202007/894007_6QV78H5NFU2NZCV.png)

在`jni_onload`函数下断点，然后`F9`

![图片描述](upload/attach/202007/894007_9A8VD4J86XNGSD2.png)

程序断在这里

经测试函数在下图中的函数执行后会使调试中断

![图片描述](upload/attach/202007/894007_XNDQ6GP45B53SFS.png)

怀疑是反调试相关函数，尝试不让这个函数执行，找到改指令对应的内存指令数据并修改为`00`（据说是注释）

![图片描述](upload/attach/202007/894007_JJGA4Y8JVKZQYKQ.png)

变为

![图片描述](upload/attach/202007/894007_RAGJR3363BX6FHX.png)

![图片描述](upload/attach/202007/894007_ATJ3SERBJZT5C9B.png)

应用之，反调试成功破除。

### 3.7 取得明文密码

从导出函数列表中可以看到许多加解密相关，猜测是把密文解密成明文然后与输入作比较，直接在`Decrypt`函数中的`CMP`（compare）指令下断点。

按`F9`让相关so加载进来，之后回到apk随便输入然后点按钮验证以触发解密函数。

按`F9`程序在指定地方断下，密码明文很可能在某一个寄存器中，找到`R7`寄存器（指向堆）`jump`过去查看存放内容

![图片描述](upload/attach/202007/894007_5VGMEVMWTCEC8Z6.png)

红圈存放的`hello5.1`应该就是密码

![图片描述](upload/attach/202007/894007_JW3K4FDC3A2ZBAA.png)

果然

## 4.总结

点点滴滴，循序渐进。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-7-20 19:34

被`mb_gujpacap`编辑，原因：

上传的附件：

- [Crackme01.apk](javascript:void(0);)

（1.57MB，146次下载）
