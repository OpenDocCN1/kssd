
综上，我们对Android APP漏洞挖掘和Android安全测试就有了一个初步的认识，接下来我们来看这三个神器具体是怎么安装和使用，帮助我们移动安全分析人员提供便利。

## 三、工具的安装和使用

### 1.drozer

#### （1）drozer介绍

drozer是一款针对Android系统的安全测试框架，可以分成两个部分：其一是`console`，它运行在本地计算机上；其二是`server`，它是一个安装在目标Android设备上的app，当使用`console`与Android设备交互时，就是把Java代码输入到运行在实际设备上的drozer代理(`agent`)中。

根据drozer官方的描述，drozer主要是有助于Android研究人员去测试一些共享的Android漏洞，对于远程攻击，可以生成`shellcode`来帮助开发人员将drozer Agent 部署为远程管理员工具，从而最大程度利用设备。

drozer是一个全面的安全审计和攻击框架，可以进行更快的Android安全评估，通过自动化繁琐和耗时的工作，帮助减少Android安全评估所花费的时间。还可以针对真实的Android设备进行测试，drozer不需要启用USB调试或其他开发功能，还可以自动化和扩展，测试公共漏洞的暴露程度。

#### （2）安装准备

- （1）drozer官方地址：`https://labs.f-secure.com/tools/drozer/`
- （2）drozer github：`https://github.com/mwrlabs/drozer`
- （3）下载及drozer用户手册：`https://labs.mwrinfosecurity.com/tools/drozer/`

#### （3）安装

我们在这里主要列举Windows端和Linux端的安装：

**1.Windows端：**

我们在Windows安装时，首先需要配置环境：

- `jdk1.8`
- `python2.7`
- `android sdk`
- 其中`python`的版本必须为`2.7`版本，因为drozer现在只支持`python2.7`版本。

我们先从官网上下载drozer（RPM）和`agent.apk`。

![image-20210902165138465](https://bbs.kanxue.com/upload/attach/202109/905443_5VTVJ2P7GAR8GSK.png)

**服务端安装：**

我们下载drozer（RPM）后解压，然后点击`setup`安装，一路默认安装就可以了，它会自动安装到`C:\drozer`文件夹下。

![image-20210902165138465](https://bbs.kanxue.com/upload/attach/202109/905443_XUM4CR7WJ8UNNBV.png)

我们检测是否成功安装，进入`bin`目录下，在`cmd`中执行`drozer.bat`，出现下面的图就说明成功安装了。

![image-20210902165138465](https://bbs.kanxue.com/upload/attach/202109/905443_A4E9GYXNKAPKD89.png)

**客户端安装：**

我们将`agent.apk`安装到手机上：

```bash
adb install agent.apk
```

安装完成后手机启动drozer，点击右下角开启端口转发按钮。

![image-20210902165138465](https://bbs.kanxue.com/upload/attach/202109/905443_DKKTWKX8HAPD7M4.png)

然后我们在电脑的终端中输入命令转发端口：

```bash
adb forward tcp:31415 tcp:31415
```

最后我们在终端中进入drozer的安装目录下，输入命令运行：

```bash
drozer console connect
```

![image-20210902165138465](https://bbs.kanxue.com/upload/attach/202109/905443_YP58NZAX3EFVUYS.png)

此时我们就可以正常使用drozer框架，来进行我们的测试工作了。

**2.Linux端安装（Kali）**

Linux上的安装因为需要的库比较多，所以很容易出错，大家最好按照这个步骤一步步来：

```bash
# 首先，我们需要将python环境配置成python2.7.0
wget https://github.com/FSecureLABS/drozer/releases/download/2.4.4/drozer-2.4.4-py2-none-any.whl
# 下载drozer
apt-get --assume-yes install python-pip
pip2 install wheel
pip2 install pyyaml
pip2 install pyhamcrest
pip2 install protobuf
pip2 install pyopenssl
pip2 install twisted
pip2 install service_identity
pip2 install drozer-2.4.4-py2-none-any.whl
# 安装jdk8
apt-get install --assume-yes openjdk-8-jdk-headless
# 安装adb（已经有adb不需要再次安装）
apt-get --assume-yes install adb
```

下载客户端并安装：`https://labs.f-secure.com/tools/drozer/`

在客户端中打开`31415`端口，然后进行端口转发：

```bash
adb forward tcp:31415 tcp:31415
```

最后启动drozer：

```bash
drozer console connect
```

参考网址：

- `https://blog.csdn.net/LoopherBear/article/details/84030567`
- `https://github.com/FSecureLABS/drozer/issues/350`
- `https://github.com/FSecureLABS/drozer/issues/357#issuecomment-652669536`

一些drozer安装常规问题解决方案：`6b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1A6W2M7%4y4A6L8$3&6Q4y4h3k6p5K9h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5x3U0f1J5z5o6p5@1x3R3`.`.

#### （4）基本使用

drozer的一些常用指令：
