
# 【安卓逆向】CTF实战分析

## 既然这篇文章提到了CTF 我就先来科普一下  What is CTF???

### 赛事介绍:
CTF是一种流行的信息安全竞赛形式，其英文名可直译为“夺得Flag”，也可意译为“夺旗赛”。其大致流程是，参赛团队之间通过进行攻防对抗、程序分析等形式，率先从主办方给出的比赛环境中得到一串具有一定格式的字符串或其他内容，并将其提交给主办方，从而夺得分数。为了方便称呼，我们把这样的内容称之为“Flag”。

### 赛事起源:
CTF起源于1996年`DEFCON`全球黑客大会，以代替之前黑客们通过互相发起真实攻击进行技术比拼的方式。发展至今，已经成为全球范围网络安全圈流行的竞赛形式。

### 竞赛模式:
CTF竞赛模式具体分为以下三类：

#### 一、解题模式（Jeopardy）
在解题模式CTF赛制中，参赛队伍可以通过互联网或者现场网络参与，这种模式的CTF竞赛与ACM编程竞赛、信息学奥赛比较类似，以解决网络安全技术挑战题目的分值和时间来排名，通常用于在线选拔赛。题目主要包含逆向、漏洞挖掘与利用、Web渗透、密码、取证、隐写、安全编程等类别。

#### 二、攻防模式（Attack-Defense）
在攻防模式CTF赛制中，参赛队伍在网络空间互相进行攻击和防守，挖掘网络服务漏洞并攻击对手服务来得分，修补自身服务漏洞进行防御来避免丢分。攻防模式CTF赛制可以实时通过得分反映出比赛情况，最终也以得分直接分出胜负，是一种竞争激烈，具有很强观赏性和高度透明性的网络安全赛制。在这种赛制中，不仅仅是比参赛队员的智力和技术，也比体力（因为比赛一般都会持续48小时及以上），同时也比团队之间的分工配合与合作。

#### 三、混合模式（Mix）
结合了解题模式与攻防模式的CTF赛制，比如参赛队伍通过解题可以获取一些初始分数，然后通过攻防对抗进行得分增减的零和游戏，最终以得分高低分出胜负。采用混合模式CTF赛制的典型代表如`iCTF`国际CTF竞赛。

### CTF的江湖:
我们一般习惯把ctf分成四类：
1. 题目来源是渗透实战或安全研究的paper
   • 国内：`BCTF(blue lotus)`、`0CTF(0ops)`、`HITCON(hitcon\217)`
   • 国际：`DEFCON`、`CODEGATE`、`PlaidCTF`、`Boston Key Party CTF`
2. 题目来源是各类有趣的漏洞，举办者一般是赛棍
   • 国内：`HCTF(HDUISA)`、`CCTF(0xFA)`、`XDCTF(XDSEC)`、`SCTF(三叶草)`、`ALICTF(alibaba)`、`360`
3. 举办者一般是赛棍，但往往并不擅长于举办比赛
   • 国内：`SSCTF(四叶草)`、`RCTF(ROIS)`、`华山杯`、`ISG`、`信息安全国赛`、`XCUNA(高校网安联赛)`
4. 举办者一般是技术从业者，并不懂CTF
   • 国内：`WHCTF`、`新XDCTF`、`首都安全日`等...

## 那么科普完毕 我们接下来就实战一波

我这里就拿国内一家安全公司出的一个入门级app的题目做分析

要求如下:
1. 有壳就脱壳
2. 拿到此题的Flag

![](https://bbs.kanxue.com/upload/attach/202001/53M2F6PCKAATYCM.jpg)

首先拿到一个app用工具查一下 这里提示没有壳,那么我们安装一下看一下软件页面![](https://bbs.kanxue.com/upload/attach/202001/6P8M6Q9E4HXTW99.jpg)

一打开就是如此 两个编辑框 然后点击登陆就跳转到另一个页面提示”Waiting fot you!”

既然如此 打开`jadx-gui`工具反编译一波 从java层入手

首先找到他的”`AndroidManifest.xml`” 里面有个”`activity`”标签里面就是他的活动页面的信息

![](https://bbs.kanxue.com/upload/attach/202001/GZVHMA5XZ33EKFS.jpg)

可以看到这里有3个活动页面信息

`<action android:name="android.intent.action.MAIN"/>`

`<category android:name="android.intent.category.LAUNCHER"/>`

有这两个属性的值上面的活动页面就是我们的入口页面(一开打软件所显示的页面信息)

既然如此我们之前看到过有个登陆的按钮在一启动的活动下

那么我们到此活动页面看一下按钮下的逻辑事件(`"com.tencent.testvuln.MainActivity"`)

![](https://bbs.kanxue.com/upload/attach/202001/MAYQ7GHNPZG9XTS.jpg)

来到这个类里面找到按钮点击事件 也就是`onClick`事件下

![](https://bbs.kanxue.com/upload/attach/202001/B76HKWSCBD2SS9X.jpg)

首先一个`switch`语句获取`view.id` 进入第一个分支也就是 `case R.id.button1`

然后接着判断我们输入的内容是不是为空 或者说输入的长度是否为0 如果条件成立就进行一个弹窗提示我们输入的内容”不能为空”

然后定义两个`string`类型的变量接收我们输入的内容

`Obj`是第一个编辑框的内容 `obj2`是第二个编辑框的内容

然后`startActivity`跳转到另一个活动页面也就是”`SecondActivity`”并将我们输入的内容也传递过来 然后在`onCreate`初始化函数进行判断

![](https://bbs.kanxue.com/upload/attach/202001/9DU5NF43GS4GUKU.jpg)

这一块儿的逻辑就很清晰了

```
if (Encryto.doRawData(this, stringExtra + stringExtra2).equals("VEIzd/V2UPYNdn/bxH3Xig=="))
```

所有的比较逻辑都是在这里了

调用一个`Encryto`类下的`doRawData`将`this` 以及`stringExtra + stringExtra2`).equals("VEIzd/V2UPYNdn/bxH3Xig=="`传递进去

我们现在跟踪到`doRawData`方法

![](https://bbs.kanxue.com/upload/attach/202001/CMX287TZYURH2A2.jpg)

发现`System.loadLibrary("JNIEncrypt");` 加载了这样一个`so`库 此方法也是由`native`关键词所修饰的

那么打开此app的`lib`文件夹发现也只有一个`so`文件库

![](https://bbs.kanxue.com/upload/attach/202001/BD27EZ6P827DHXY.jpg)

拿出我们的`ida`工具反编译一波先

+Q裙8 2 3 9 3 8 3 8 9

![](https://bbs.kanxue.com/upload/attach/202001/22XGUJ7MGX9YDY5.jpg)

发现并没有`java_` 静态注册的方法那么我们也只能找到`JNI_OnLoad`动态注册的位置

![](https://bbs.kanxue.com/upload/attach/202001/FWC3UGNXEQMBXFG.jpg)

来到了

![](https://bbs.kanxue.com/upload/attach/202001/T579URP4R76VXC4.jpg)

`RegisterNatives`就是注册的函数 他的第三个参数就是我们要找到的方法体 第四个参数就是他注册的个数

![](https://bbs.kanxue.com/upload/attach/202001/W9NPSD4RBN4X5PU.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/JKUHW5FGT8C9F5U.jpg)

然后我们一步步跟踪到了这里 发现采用了一个`AES/ECB/PKCS5Padding`的加密方式

Key勒就摆在面前的”`thisisatestkey==`”

然后我们来解密一波

将`VEIzd/V2UPYNdn/bxH3Xig==` 解密得出”`aimagetencent`”

![](https://bbs.kanxue.com/upload/attach/202001/M4C2SDYH8AHGTAJ.jpg)

第三个页面这里会解密一个`9YuQ2dk8CSaCe7DTAmaqAA==`值

![](https://bbs.kanxue.com/upload/attach/202001/VKW5UHR82RYEU9T.jpg)

将`9YuQ2dk8CSaCe7DTAmaqAA==` 解密得出”`Cas3_0f_A_CAK3`”

![](https://bbs.kanxue.com/upload/attach/202001/FC8YXZSHKC69AEP.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-2-4 16:06

被kanxue编辑，原因：
