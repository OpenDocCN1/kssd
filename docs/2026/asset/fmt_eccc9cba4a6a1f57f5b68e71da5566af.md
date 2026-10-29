
**练习软件地址：**<a href="elink@d01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6K9i4u0B7i4K6u0W2L8r3q4F1P5X3!0#2K9g2)9J5k6h3y4G2L8g2)9J5c8X3W2W2K9r3f1@1K9i4c8&6z5e0u0Z5"><mark class="encrypted">4d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6K9i4u0B7i4K6u0W2L8r3q4F1P5X3!0#2K9g2)9J5k6h3y4G2L8g2)9J5c8X3W2W2K9r3f1@1K9i4c8&6z5e0u0Z5</mark></a>

**推荐下篇文章地址：**[超级详细的Frida Hook和Xposed Hook再搞Crackme](https://bbs.pediy.com/thread-263872.htm)

# [超级详细]实战分析一个Crackme的过程

## 一、写在前面

自学这个也有几个星期了，今天就总结一下近期学习的成果，实战分析一个Crackme，并写下了这篇超级详细的过程，**从软件环境的配置到软件详细的使用再到最后的逆向分析出结果**，为了文章的贴图方便和文章美观展示，用了两台电脑+MUMU模拟器+Google6.0.1版真机，相互切换着截图附在帖子里，也主要是因为模拟器动态调试so加载好像有问题。**我是新入门的小白一个，如果文章中出现错误，希望大佬们能及时斧正，以免对其他萌新产生误导。**（此文章只发布在**吾爱和看雪**）

## 二、所用工具

`IDA Pro V7.0版`、`jadx 1.2版`、`Android Kill 1.3.1版`、`Google真机 6.0.1版`、`MUMU模拟器 2.4.2版`

## 三、jadx分析AliCrackme

先使用MUMU模拟器打开软件随便输入密码，提示 **验证码校验失败**

![](https://bbs.kanxue.com/upload/attach/202011/909733_VEMDV7VPSQSHR3V.png)

将软件拖入**jadx**去分析一下软件代码的执行流程，直接使用软件的搜索功能，发现只有一条数据并进去看一下。

![](https://bbs.kanxue.com/upload/attach/202011/909733_G244EYNNPE4MMY4.png)

分析一下软件中代码执行密码校验的流程，试图分析出软件的真实的密码

![](https://bbs.kanxue.com/upload/attach/202011/909733_ZTTWT8UGMS3BR65.png)

经过上图的一些分析步骤之后，我发现软件的真实的密码在`crackne.so`里面，接下来使用jadx的**全部保存**（ctrl+s）并选择保存的位置将so文件提取出来，如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_WR4P558ZSDD82CK.png)

## 四、IDA静态/动态/反反/再动态调试

### 1.静态分析

接着打开 IDA 点击 NEW 打开`libcrackme.so`文件，中途出现弹窗一致 **ok** 即可！

![](https://bbs.kanxue.com/upload/attach/202011/909733_C6HMBYRM6UUC7YC.png)

直接看到了这里，这个与上面反编译分析的方法名一致，只是前面加入了一些软件包的特征，如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_9S6YMCA7S7EM8FE.png)

汇编不是太方便看，直接使用左边的功能将其转换为伪C代码或直接使用快捷键 F5

![](https://bbs.kanxue.com/upload/attach/202011/909733_JSTFRYNZW9SG6TA.png)

转换为伪C代码之后，分析一下代码的执行，点击`v3`通过`JNIEnv *`还原类似`(*(\_DWORD *)v3 + 676))`格式的指令，重命名以及注释的方式记录分析，如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_P4WRCDQ3KWJPMNY.png)

下面详细的分析一下代码的逻辑，判断出用户输入的密码和软件真实的密码，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_ACFJNHB2TWBQ9UE.png)

下面使用`Rename lvar`或快捷键 N 的方式直接将分析出来的直接重命名一下，方便以后容易查看，最终如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_BX97CUYUZBU7KNH.png)

### 2.动态调试

接下来就是通过动态调试分析这个so文件的密码了，在动态调试之前我需要配置一些环境，步骤如下：

![](https://bbs.kanxue.com/upload/attach/202011/909733_GBAZN3W3XWEMQD6.png)

```bash
# 连接MUMU模拟器
adb connect 127.0.0.1:7555 //7555为mumu模拟器端口，其它模拟器百度一下
# 将文件push 进手机的指定目录下
adb push android_x86_server /data/local/tmp/
# 进入手机端命令
adb shell
# 切换获取手机的root权限
su
# 查找push的文件是否在手机中
cd /data/local/tmp/
# 查看路径下的文件以及权限
ls -l
# 拥有root权限更改文件的权限为777
chmod 777 android_x86_server
# 在手机中启动运行该文件
./android_x86_server
# windows运行 端口转发到PC
adb forward tcp:23946 tcp:23946
```

![](https://bbs.kanxue.com/upload/attach/202011/909733_UWP8P6DTVJ6ZU7H.png)

再开启一个命令行做一下端口的转发，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_GMDHC3KB2WB4MPK.png)

全部执行完毕之后，可以看到正在监听这个端口，这时候我们打开 IDA 并且点击GO然后按照下图选择所使用的

![](https://bbs.kanxue.com/upload/attach/202011/909733_9HPC25A9QQZ4BUR.png)

并输入`127.0.0.1`后面的端口是默认的，上面的命令也在监听这个端口。

![](https://bbs.kanxue.com/upload/attach/202011/909733_FMXFWSDV89SSBKK.png)

找到要动态调试的软件的包名，直接选中，点击 ok 打开即可！

![](https://bbs.kanxue.com/upload/attach/202011/909733_H8DAHAKS4H72FDH.png)

打开之后，直接使用下图的方式，或者快捷键 Ctrl+s 找到到要调试的`libcrackme.so`文件带x权限的文件。

![](https://bbs.kanxue.com/upload/attach/202011/909733_7Q86FZPU869GYT3.png)

在所有的so文件中直接使用快捷键Ctrl+f搜索，并找到 Start地址先执行且有 X(可执行权限) 的选中点击OK进入，

**模拟器的是下图显示的so：**

![](https://bbs.kanxue.com/upload/attach/202011/909733_SC3QA9GNHFSWB29.png)

这里使用模拟器的话，so文件加载会有一些问题，**在这里我改用了Google真机6.0.1版本进行动态调试**！

**真机的是下图显示的so:**

![](https://bbs.kanxue.com/upload/attach/202011/909733_T5FEQQVRUP3YPSZ.png)

打开so文件之后我们要定位到要调试函数的一个内存的**绝对地址=so文件的基地址+函数的偏移量**

![](https://bbs.kanxue.com/upload/attach/202011/909733_5EMKYAFD6GAQUMQ.png)

经计算可知，函数的绝对地址就是`F49591A8`在IDA中使用快捷键G跳转到地址的位置，也就是要调试的函数位置

![](https://bbs.kanxue.com/upload/attach/202011/909733_55TDWKHXP4NKVFR.png)

找到函数名之后，鼠标右键选择`Add breakpoint`或者直接使用快捷键F2打断点，然后让程序恢复执行，使用快捷键F8往下走。

![](https://bbs.kanxue.com/upload/attach/202011/909733_YZRJ2NQWHS29MY7.png)

点击 F8 给so文件的函数打断点调试，直接就退出或出现错误弹窗等等！如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_KJXTZ32SYBFQYXJ.png)

证明软件有反调试的机制，先看一下软件是通过何种方式反调试的，又是如何检测被调试的呢？

**检测是否被调试：**利用Linux系统`ptrace`来实现，当应用被调试时应用内存里的`TracerPid`字段就不为0，只要是不为0的时候，就会直接的退出程序，达到反调试的目的。

接下来进入设备查看一下`ptrace`字段：

```bash
# 进入设备
adb shell
# 获取Root权限
su
# 获得APP的进程ID
ps | grep 软件的包名
# 查看进程的信息及TracerPid值
cat /proc/进程ID/status
```

![](https://bbs.kanxue.com/upload/attach/202011/909733_NVSVRGR8VR345FH.png)

知道了软件有反调试之后，接下来就需要对其**反反调试**了。

### 3.反反调试

下面为方便发帖截图说明，我又换回了模拟器来做调试(模拟器可以调就用模拟器，不能调的当然就要换真机啦）

**如何反反调试：**程序的so文件在加载阶段会先执行`JNI_OnLoad`，之后就不再执行，在程序的so文件加载阶段才能给`JNI_OnLoad`打断点调试即可！

```bash
# 调试APP挂在加载界面
adb shell am start -D -n 包名/.类名
adb shell am start -D -n com.yaotong.crackme/.MainActivity
```

用调试模式启动APP，APP此时会挂在启动界面，现在并没有开始加载so文件。

![](https://bbs.kanxue.com/upload/attach/202011/909733_B4DCKAEKSACKSH7.png)

重新关闭打开 IDA 并调试上面挂载的程序，进入之后点击`Debugger->Debugger options...`选项如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_R4A9P8GJPKSTWEW.png)

勾选这两项Suspend on thread start/exit Suspend on library load/unload，之后ok，**之后点击下图左上角的运行符号**，让它运行起来。

![](https://bbs.kanxue.com/upload/attach/202011/909733_2A242VW5CWYH66M.png)

然后让模拟器的APP也运行起来，用过下面的这些命令运行APP,即可

```bash
# 进入设备
adb shell
# 获取Root权限
su
# 获得APP的进程ID
ps | grep com.yaotong.crackme
# 监听进程的ID
adb forward tcp:8700 jdwp:4495
# 4495为上面获得的APP进程ID
# 运行进程
jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8700
```

获取APP的进程的ID以及再开启一个命令行，直接做端口的监听转发，并直接的运行进程，如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_UGNBKNRG876N9ES.png)

**注意：**卡在**运行进程**这一步的不出现上图的**正在初始化jdb...**，应该是你漏掉了一个步骤，我就是因为截图过之后忘记了在IDA动态调试的时候，**忘记点击了一下IDA左上角的运行符号**，而导致的一致不出来**正在初始化jdb...**的情况，出现初始化后，APP处于这个没有控件的界面，此时的APP并没有完全的运行起来,只是挂在这里

![](https://bbs.kanxue.com/upload/attach/202011/909733_5W8N4E8TE8Q9P5K.png)

然后我们回到 IDA 的动态调试找到要调试的`crackme.so`文件，按Ctrl+s之后按Ctrl+f搜索crackme，**如果没有就点击左上角的运行**运行一步继续操作，直到可以在所有的so文件中搜索到`crackme.so`文件，并且找到可执行的X权限的文件。（这里模拟器又出现了没有X可执行的情况，直接换Google6.0.1真机）**模拟器调试**如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_A9CY5UNERXXJV2J.png)

**真机动态调试前一定要**修改APP AndroidMenifest.xml文件，在`<application`里给APP加上可调试权限，`android:debuggable="true"`，重新打包APP，签名，安装，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_3FT4G7529MAFWHM.png)

接下来点击 IDA 左上角的运行，按照上面说的，知道找到`crackme.so`中有X(可执行的)，进入，如下图：

![](https://bbs.kanxue.com/upload/attach/202011/909733_24MPT3JE5RJE8P2.png)

然后我们要进入`JNI_ONLoad`,接下来算**绝对地址=执行so文件的基地址+函数的偏移量**最终如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_NUFFDG6BMYJ966W.png)

最终的计算结果如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_WRFZCVMH2WV8PDK.png)

跳转到`JNI_ONLoad`里面去了，先结合静态分析看一下这个文件流程图的大致逻辑，然后在动态调试里打断点调试，并点击IDA左上角的运行，恢复程序的运行如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_YU9UK57TJ8Z9KB5.png)

`BLX R7`的位置跳了出去，很可疑的位置需要重点注意，尝试修补so文件的这个位置，看一下，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202011/909733_34J3QHQ6Y64UGQY.png)

然后IDA静态调试分析找到上面的`BLX R7`位置，点击一下，可以看Hex View-1视图中的`3F FF 2F E1`

![](https://bbs.kanxue.com/upload/attach/202011/909733_JMYDWV3P626H4ZG.png)

把`3F FF 2F E1`修改为`00 00 00 00`即可，下面直接使用**两种方式**修改这个：

**第一种：**

![](https://bbs.kanxue.com/upload/attach/202011/909733_KNN924J9F2S92VV.png)

![](https://bbs.kanxue.com/upload/attach/202011/909733_HRA7HPJBM2TXQGH.png)

**第二种：**

![](https://bbs.kanxue.com/upload/attach/202011/909733_9F8KK7C3TCNJVEC.png)

![](https://bbs.kanxue.com/upload/attach/202011/909733_ZGY8S6D7FU7ENV8.png)

修改完毕之后，找到这个文件所在的位置，与之前Android kill反编译之后的里面的so文件做替换，然后就回编译签名安装即可

![](https://bbs.kanxue.com/upload/attach/202011/909733_XWE9RC9X7VT3QNW.png)

![](https://bbs.kanxue.com/upload/attach/202011/909733_TA6989UUC3M7898.png)
