
![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_THMZ8ZY7FXNPHJR.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_H6PPYUBFZJNJQGU.png)

# 动态调试

小技巧，使用`PsExec`得到system权限。

`PsExec64.exe -i -s cmd`

以system权限启动`x64dbg`，以方便附加调试目标进程。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_NUHJMN68D9W24Y2.png)

在调试时需要隐藏调试器，否则在调试过程中会报异常。

`调试->高级->隐藏调试器（PEB）`

对`CreateProcessA`函数下断点，然后执行exp触发断点。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_GGZ2FTYM2BEXATU.png)

# 静态分析

有个`upx`壳，使用`upx -d`直接脱掉。脱掉后的程序直接运行的话，还是会报错，没有探究原因，不过`ida`可以正常分析了。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_HXQ75PHBV8KAXBA.png)

根据动态调试的结果，可以很快定位到关键代码位置。找到URL路由，这可以用来分析其他api功能。另外，发现除了exp里提到的`ping`可以导致命令执行，`nslookup`也是可以的，可以自行编写脚本测试。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_GR958H7CN3YZ53X.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_EHCUCB3QPEE925D.png)

# 渗透测试

## 编写Goby脚本

### 配置“漏洞信息”

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_3EZGKHTFJWCREK3.png)

这里的指纹信息十分关键，`Goby`在扫描的时候，会先扫描资产，这个指纹就是用来判断资产种类的，匹配上指纹之后，才会打对应的poc。指纹的好坏，直接决定了扫描速度。

`Goby`语法和fofa是一致的，这里匹配的是`"GET /"`的应答，因为`Goby`在做资产探测的时候不会探测太多URL。

```text
body="Verification failure" && body="false" && header="Cache-Control: no-cache" && header="Content-Length: 46" && header="Content-Type: application/json"
```

### 配置“扫描测试”

扫描测试有两个步骤，1.获得访问令牌`CID`；2.带令牌执行命令。

#### Test1

访问获取令牌的URL。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_S8PM5PFZYMB8CHH.png)

指纹判断，如果访问成功，则根据正则提取`CID`。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_9TYZE6QHF47N7K9.png)

可以用`python`快速测试正则：

```python
import re
a = r'''{"__code":0,"enabled":"1","verify_string":"ysDRmcQu37usMmdA60fniHTv3cJzlWHz","code":0}'''
filter = re.compile(r'''"verify_string":"(\S+?)",''')
res = filter.findall(a)
print(res)
```

#### Test2

带Cookie访问命令执行的URL，上一步设置的变量`CID`可以套三个大括号来使用，即`{{{CID}}}`。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_SGDGMCPNNX78KBZ.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_P5KVN3YWEMVGV6T.png)

### 测试效果

测试效果，发现漏洞。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_KVNHD6NVXBYVGNP.png)

测试过程可以使用`wireshark`抓包来辅助poc编写，也可以参考老的poc脚本，其目录在`goby-win-x64-1.8.293\golib\exploits\user`，或者通过poc管理导出来也是可以的。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_N6TC74J7G5R3GYN.png)

# 态势感知

## 流量监测

### 尽可能多的生成多种形式的攻击流量

考虑合理变形，尽可能多的生成多种形式的攻击流量，以便用来测试检测规则。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_D6UBDXJRW4DFVAH.png)

打poc的同时，用`wireshark`抓包，这里得到攻击流量包`sunlogin_rce_multi_payload.pcap`。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_2MQMQAX5269GWK7.png)

### 编写规则并测试

测试环境为`Kali-Linux-2021.2-vmware-amd64`，`suricata`版本为6.0.4。

根据payload编写pcre正则：

```regex
/\/check?.*cmd[\s]*?=(?:ping|nslookup).*?(?:\.\.\/|\.\.\\)/
```

正则解释：

- `[\s]*?=` 匹配任意个空白符，非贪婪匹配，匹配最近的`=`
- `(?:ping|nslookup)` 匹配`ping`或`nslookup`，`?:`表示不获取匹配结果
- `(?:\.\.\/|\.\.\\)` 匹配`../`或`..\`

测试pcre正则：

```bash
apt install pcre2-utils
pcre2test
```

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_BW2E6AE6RGRXDDW.png)

编写suricata规则：

（此规则可以应对正常攻击和部分变形，但依然存在被绕过的可能。规则写严了容易漏报，写松了容易误报，另外还应该要考虑报文分片传输、丢包的问题。）

`vim /etc/suricata/rules/test.rules`

```text
alert http any any -> any any (msg:"CNVD-2022-10270 SunloginClient RCE"; pcre:"/\/check?.*cmd[\s]*?=(?:ping|nslookup).*?(?:\.\.\/|\.\.\\)/U"; classtype:attempted-admin; sid:22022801; rev:2;)
```

`/U`里的`U`表示在标准`uri`上进行`pcre`匹配（区别于`http_raw_uri`，类似于`http_uri`，相当于URL解码后再匹配）。

`vim /etc/suricata/suricata.yaml`

```yaml
rule-files:
  # - suricata.rules
  - test.rules
```

测试`suricata`规则，5条攻击报文触发了5次告警，测试成功。

`suricata -r sunlogin_rce_multi_payload.pcap`

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_8XZVGPW4TWNM8KK.png)

流量监测相关资料

f05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4u0A6j5$3q4@1j5g2)9J5k6i4u0W2j5h3c8@1K9r3g2V1L8$3y4K6i4K6u0W2K9h3!0Q4x3V1k6W2L8W2)9J5c8Y4y4#2M7X3W2U0j5i4c8S2i4K6u0V1y4W2)9J5k6e0m8Q4x3X3f1H3i4K6u0r3M7Y4g2D9k6i4y4Q4x3V1j5`.

769K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3y4C8L8Y4y4E0i4K6u0W2K9h3!0Q4x3V1j5`.

dc6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7X3E0A6L8h3g2Q4x3V1k6S2M7X3E0A6L8h3f1`.

## 终端监测

windows下的事件监控可以用`sysmon`，Linux下则可以用`auditd`，然后借助`wazuh`来管理日志。

`Sysmon64.exe -i`

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_GXJ58E4JPSHJ4V9.png)

`事件查看器->应用程序和服务日志->Microsoft->Windows->Sysmon`

![图片描述](https://bbs.kanxue.com/upload/attach/202203/598931_FZUDWUW4T49EMTK.png)

终端监测相关资料

efcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6P5i4y4Y4k6h3g2C8i4K6u0W2j5$3&6Q4x3V1k6K6P5i4y4E0L8$3&6Q4x3V1j5`.

9e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5h3I4A6j5$3W2G2N6i4y4C8M7W2)9J5k6h3y4U0i4K6u0r3x3U0l9I4z5q4)9J5c8U0p5I4i4K6u0r3x3e0g2Q4x3V1k6Q4c8e0c8Q4b7V1c8Q4b7V1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2a6f1#2y4q4b7#2!0q4y4W2)9&6c8g2)9^5y4q4!0q4y4g2!0n7b7W2!0n7b7g2!0q4y4q4!0n7z5q4!0n7b7W2!0q4y4W2)9&6b7#2!0n7b7g2!0q4y4g2!0n7x3g2)9^5x3W2!0q4y4g2)9^5y4g2!0m8y4g2!0q4y4q4!0n7c8g2!0n7y4g2!0q4y4W2!0m8x3#2)9^5x3q4!0q4y4W2!0n7y4g2)9^5b7W2)9J5c8R3`.`.

021K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4A6L8X3N6D9k6e0N6Q4y4h3k6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4x3o6l9K6z5o6p5I4y4H3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-3-5 00:09 被 Jtian 编辑，原因：补充与纠错

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
