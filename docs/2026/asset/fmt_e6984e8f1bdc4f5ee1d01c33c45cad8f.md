
进入`/path/to/xxx`目录，找到`xxx.json`。

`public` `class` `MainActivity extends AppCompatActivity {`
`@Override` 
`protected void onCreate(Bundle savedInstanceState) {`
`super.onCreate(savedInstanceState);`
`setContentView(R.layout.activity_main);`
`Intent intent = new Intent();`
`intent.setAction("com.example.down");`
`intent.putExtra("url","d8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8R3...");`
`intent.putExtra("filename","baidu");`
`sendBroadcast(intent);`
`}`
`}`

**效果显示：**

测试样例：
![image-20211026135228985](https://bbs.kanxue.com/upload/attach/202110/905443_7U75RR65J9CQ53T.png)

攻击样例：
![image-20211026135354598](https://bbs.kanxue.com/upload/attach/202110/905443_M69EJHUBWXPXRRM.png)

我们可以很明显发现，我们可以通过进程B发送广播到进程A，从而调用进程A的下载功能，开始下载

### （2）猎豹清理大师内存清理权限泄漏漏洞

这个案例是一个很早之前的漏洞，现在参考意义不大，不过大家可以借助其学习其原理即可

**漏洞描述：**

`Android`应用程序猎豹清理大师存在自定义的权限`android.permission.RESTART_PACKAGES`，该权限主要用来结束进程来达到清理内存的目的，当其他没有申请此权限的`app`向目标程序发送相应的`intent`时，便可以结束后台运行的部分`app`进程

**漏洞实现：**

![image-20211026140524902](https://bbs.kanxue.com/upload/attach/202110/905443_W6UJZHZDHE58BUD.png)
![image-20211026140633874](https://bbs.kanxue.com/upload/attach/202110/905443_DH7G67RJ8ANNPVQ.png)
![image-20211026140609720](https://bbs.kanxue.com/upload/attach/202110/905443_QPFH293V9SQQJ9Z.png)

这就是很典型的我们利用个人定义的权限，来攻击目标程序，达到权限泄漏漏洞的目标

### 3.漏洞防护

#### （1）组件设置不可导出

我们从上面的两个案例可以发现，`Android`的权限泄漏漏洞往往和四大组件相互关联，我们对一些重要组件设置不可导出，就可以达到一定的保护效果

#### （2）设置权限级别

我们可以对自定义的权限一定要设置相应的保护级别，从上文我们可以得出：

`android:protectionLevel="normal"`，不需要动态申请
`android:protectionLevel="dangerous"`，需要动态申请
`android:protectionLevel="signature|signatureOrSystem"`，需要两个程序签名相同才能访问，这样就可以很好的避免权限泄漏的风险

## 四、Android默认设置介绍

在上文中已经向大家详细介绍了`Android`中的权限安全漏洞的情况，接下来我们进一步了解`Android`中的默认设置以及默认设置引起的安全问题，`Android`中的默认设置主要分为：`AndroidManifest.xml`配置文件和`WebView`的默认设置

![image-20211026144059511](https://bbs.kanxue.com/upload/attach/202110/905443_Z8H8HDY5NRK5RCN.png)

#### （1）AndroidManifest.xml默认设置

这里我总结了`AndroidManifest.xml`的基本结构图

![image-20211026144059511](https://bbs.kanxue.com/upload/attach/202110/905443_XVVS4CMA78AYKVV.png)

这里我们主要关注`allowBackup`、`Debuggable`、组件默认导出

##### <1> allowBackup

`Android API Level 8`及其以上`Android`系统提供了应用程序的备份和恢复功能，这是由`AndroidManifest.xml`文件中的`allowBackup`属性决定，`allowBackup`标志为`true`时，用户即可通过`adb backup`和`adb restore`来进行对应用数据的备份和恢复

`Android`属性`allowBackup`安全风险源于`adb backup`容许任何一个人能够打开`USB`调试开关，从`Android`手机中复制应用数据到外设，一旦数据被备份之后，所有应用数据都可被用户读取。`adb restore`容许用户指定一个恢复的数据来源（即备份的应用数据）来恢复应用程序数据的创建

**安全影响：**

`通讯录应用，一旦应用程序支持备份和恢复功能，攻击者即可通过 adb backup 和 adb restore 进行恢复新安装的同一个应用来查看聊天记录等信息`
`支付金融类应用，攻击者可通过此来进行恶意支付、盗取存款等`

**allowBackup安全风险详情：**

`1.allowBackup`风险位置：`AndroidMannifest.xml`文件 `android:allowBackup`属性
`2.allowBackup`风险触发前提条件：未将`AndroidMannifest.xml`文件中的`android:allowBackup`属性值设为`false`
`3.allowBackup`风险原理：当`allowBackup`标志值为`true`时，即可通过`adb backup`和`adb restore`来备份和恢复应用程序数据

**数据备份：**

```
命令：adb backup [-system|nosystem] -all [-apk|-noapk] [-shared|-noshared] -f <档案名称>[需要备份的应用包名]
```

`[-system|nosystem]`
这个指令告诉`adb`在备份时是否要连同系统一起备份，若没有打开的话，默认是`-system`表示会一起备份系统，若连系统一起备份，在还原时候会覆盖系统档案，这对已经升级的手机是非常不好的，`-nosystem`是建议要打上去的指令

`-all`
这个指令除非要备份单一`App`不然是一定要打上去的，表示是否备份全部的`App`，若有加上`-nosystem`，就只会备份你目前已经安装上去的`App`而不会连系统`App`一起备份

`[-apk|-noapk]`
默认是`-noapk`，这个参数的意思是，是否连安装的`apk`一起备份，若为`-noapk`，则只会备份`apk`的资料档

`[-shared|-noshared]`
默认是`-noshared`，表示是否连手机存储空间或是`SD`卡的档案一起备份

`-f`
用这个来选择备份文件存储在哪里，例如`-f /backup/applock.ab`将会使文件存储在根磁盘（`Windows`的`C`盘等等）下一个名为`backup`的文件夹里，并且备份文件名为`applock.ab`

例如我们直接备份
```
adb backup -apk -shared -system -all -f applock.ab
```
就会在电脑默认路径下产生`applock.ab`文件，大小与系统有关

**查看数据：**

一般我们使用`abe`工具来解析`ab`文件，工具参考网址：[abe工具](https://bbs.kanxue.com/elink@641K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6h3I4W2L8X3E0G2N6W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8T1j5h3y4C8N6i4m8Q4x3X3c8W2P5s2c8J5j5h3y4@1L8%4u0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.` )

解压命令：
```
java -jar ade.jar unpack 1.ab 1.tar
```

**恢复数据：**
```
adb restore applock.ab
```

##### <2> debuggable

`android:debuggable`属性是指定应用程序是否能够被调试，即时是以用户模式运行在设备上的时候，如果设置为 `true`，则能够被调试，否则不能调试，默认值是`false`

逆向工作人员在对一个`apk`程序进行逆向时，第一步往往就是要对`debuggable`属性进行绕过，否则就不能进行正常的调试，因此如果属性本身设为`true`，更容易导致该应用存在动态调试，泄露数据信息的风险

![image-20211026164320202](https://bbs.kanxue.com/upload/attach/202110/905443_Y2PA28AWXWQBUQY.png)

##### <3> exported

前几篇帖子，我们讲述四大组件过程中，经常会涉及到`exported`导出导致的漏洞，`exported`组件导出，会导致导出的组件被第三方`APP`任意调用，导致敏感信息，并可能受到绕过认证、恶意代码注入等风险，由于前面我们已经讲得很详细了，这里就不再做过多描述

#### （2）WebView默认设置

`WebView`是`android`中用来展示网页的重要控件，`Android`中经常会使用`WebView`来实现`Web`页面的展示，在`Activity`中启动自己的浏览器或者简单的展示一些在线内容等，而`WebView`上的漏洞问题也各种各样，过去几年内`WebView`中被披露的重大漏洞包括了任意代码执行漏洞、跨域、密码明文保存等，后面我们将`WebView`作为一个专题讲述上面存在的漏洞问题，这里我们只关注`WebView`中默认设置的安全问题

后面我们主要讲述`WebView`上几方面的默认设置漏洞

![image-20211026165646971](https://bbs.kanxue.com/upload/attach/202110/905443_U4EWKHYPP5BTR2W.png)

## 五、Android默认设置漏洞原理分析和复现

### 1. allowBackup

#### （1）备份安全设置信息泄露漏洞

上文我们已经详细的给大家讲述了`allowBackup`设置`true`的安全性，接下来我拿一个案例带大家深入了解：

##### <1> 漏洞复现

样本：`sieve.apk`

**信息设置：**

首先，我们将样本安装到手机上，并打开主界面

![image-20211026194532102](https://bbs.kanxue.com/upload/attach/202110/905443_E87D7X264BUETV4.png)
![image-20211026194611643](https://bbs.kanxue.com/upload/attach/202110/905443_J9PHD5K2CWDGJC6.png)

进入主程序我们发现，提示我们注册密码，而且需要至少`16`位，这里我们输入密码`zxcvbnm0123456789`

进入后，又要我们输入`PIN`码，我们输入`1234`

![image-20211026195015970](https://bbs.kanxue.com/upload/attach/202110/905443_WZ9AJS36S937GV8.png)

然后我们发现需要再次输入密码才能进入，我们再次输入密码

![image-20211026195112662](https://bbs.kanxue.com/upload/attach/202110/905443_2AXNMD6EW3276RK.png)

发现右上角可以添加账户密码信息，我们添加，密码设置`123456`

![image-20211026195256415](https://bbs.kanxue.com/upload/attach/202110/905443_ZE3RQ8Q6UGF2C5R.png)
![image-20211026195314419](https://bbs.kanxue.com/upload/attach/202110/905443_KSZSD8DSRGJVNK5.png)

这样我们就在`apk`中就存入了一些信息，接下来我们来看如何备份和恢复这些信息呢

**数据备份：**

我们先用`jadx-gui`打开目标程序，查看`allowbackup=true`

![image-20211026195645086](https://bbs.kanxue.com/upload/attach/202110/905443_2GUXW4F9SSJUSAV.png)

这里发现是可以备份的，所以我们输入备份命令
```
adb backup -f allowBackup.ab -noapk com.mwr.example.sieve
```

![image-20211026195847860](https://bbs.kanxue.com/upload/attach/202110/905443_JXFBT8Q2RSJA3S8.png)

我们的手机就弹出备份请求，如果你要加密就输入加密密码，这里我们直接备份

![image-20211026195945086](https://bbs.kanxue.com/upload/attach/202110/905443_KTCVNM6MUJYWZD8.png)

备份完成后，我们在当前目录下就可以发现我们备份的文件`allowBackup.ab`

**数据解析：**

我们需要使用`abe`工具对备份的文件解压，下载路径：[`abe工具`](https://bbs.kanxue.com/elink@1daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6h3I4W2L8X3E0G2N6W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8T1j5h3y4C8N6i4m8Q4x3X3c8W2P5s2c8J5j5h3y4@1L8%4u0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.` )

我们下载后，需要安装`jdk`环境，然后使用命令解压：
```
java -jar abe.jar unpack allowBackup.ab 1.tar
```

![image-20211026200752183](https://bbs.kanxue.com/upload/attach/202110/905443_TD2JS49JR8GVDQ8.png)
![image-20211026200815420](https://bbs.kanxue.com/upload/attach/202110/905443_XT4JKRKT72S24YV.png)

解压
![image-20211026200904172](https://bbs.kanxue.com/upload/attach/202110/905443_Y7XTNU62EQ6KGVJ.png)
![image-20211026200925823](https://bbs.kanxue.com/upload/attach/202110/905443_FTVRXC7MHYNYJWF.png)

我们就得到了`database.db`数据库，然后我们使用`DB Browser for SQLite`打开，下载路径：[DB Browser](https://bbs.kanxue.com/elink@edeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7h3I4A6N6r3g2T1M7X3!0%4M7$3g2J5i4K6u0W2L8%4u0Y4i4K6u0r3)

![image-20211026201414775](https://bbs.kanxue.com/upload/attach/202110/905443_8JZPXGN8638SRUN.png)
