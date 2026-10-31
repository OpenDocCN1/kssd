
# [分享]静态分析-android killer使用

## 0x1 介绍

`android killer`是一款图形化反编译工具，底层调用`apktool`、`dex2jar`等命令行工具，可实现一键反编译、一键回编译。

## 0x2 运行环境

由于`android killer`使用的反编译工具是`java`程序，所以需要安装`jdk`环境并配置`jdk`路径。

## 0x3 特色功能

- 编辑`smali`
- 动态插桩
- 键重签名

## 0x4 使用

1. 运行程序，点配置，检查`java`环境等。

![upload/attach/202202/859914_DN5XHC6RY5WCKW7.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_DN5XHC6RY5WCKW7.jpg)

2. 点工具，查看提供的功能：

![upload/attach/202202/859914_NDKC4EW9A8NVENA.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_NDKC4EW9A8NVENA.jpg)

3. 编码转换，搜索字符串时搜索不到，可以换编码再搜索。

![upload/attach/202202/859914_BFQNTTTGYE3GQGA.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_BFQNTTTGYE3GQGA.jpg)

4. `apk`签名，默认的有点问题，签名时将`default`改成`AndroidKiller`签名。

![upload/attach/202202/859914_2NXVJ28C6GWY3CF.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_2NXVJ28C6GWY3CF.jpg)

5. 在`Android`菜单下，可以点编译，将程序重新打包成一个`apk`。

![upload/attach/202202/859914_JTVB9Y6E89JZNTD.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_JTVB9Y6E89JZNTD.jpg)

6. 安装了模拟器可以直接点“安装”将程序安装到模拟器中，还可以点“进程”查看模拟器中正在运行的进程。

![upload/attach/202202/859914_BDXF4TNTQ7FJBAE.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_BDXF4TNTQ7FJBAE.jpg)

7. 将程序拖进来，会自动加载和分析程序，双击`apk`文件，安装到雷电模拟器，打开命令行输入`adb shell`进入系统。

```bash
dumpsys activity top //查看当前最顶层的活动界面
TASK com.example.ctf2 id=3
ACTIVITY com.example.ctf2/.MainActivity ae27780 pid=2365
Local Activity 1e03868 State:
mResumed=true mStopped=false mFinished=false
mChangingConfigurations=false
mCurrentConfig={1.0 ?mcc?mnc [zh_CN] ldltr sw617dp w1097dp h593dp 280dpi lrg long land finger qwerty/v/v -nav/h s.5}
......
```

8. `ACTIVITY com.example.ctf2/.MainActivity`，就是说这个活动界面在`com.example.ctf2`这个包下面`MainActivity`文件里，那么`MainActivity.smali`代码对应的就是模拟器中运行的`apk`窗口的活动代码。

9. `app`启动时通常会调用`onCreate`方法，`.method public onCreate(Landroid/os/Bundle;)V`，可以右键插入代码，插入`log`，每个函数前面插入一个，打印字符串来分析程序的执行流程。

```smali
.method public onCreate(Landroid/os/Bundle;)V
.locals 2
const-string v0, "you message：onCreate"
```

例如打印方法名称，判断方法执行的先后顺序。

**注意：这里我们使用了一个寄存器，`.locals`这里要加1，由2改成3**

10. 编译的时候点下面那个小三角，编译时使用`Androidkiller`签名。

```text
APK 所有编译工作全部完成!!!
生成路径：
file:C:\Users\LENOVO\Desktop\AndroidKiller\projects\1\Bin\1_killer.apk
```

11. 去`android-sdk\tools`目录找到`ddms.bat`工具，双击运行，找到我们的进程，搜索添加的字符串，可以看到输出了信息。

![upload/attach/202202/859914_XNEU68DXRHTUNDB.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_XNEU68DXRHTUNDB.jpg)

## 0x5 常见问题

1. `Androidkiller`编译失败，这是因为`apktool`版本太低，按照下面步骤添加`apktool`，切换版本即可。

![upload/attach/202202/859914_9WXTB5GMP7HFA2N.jpg](https://bbs.kanxue.com/upload/attach/202202/859914_9WXTB5GMP7HFA2N.jpg)
