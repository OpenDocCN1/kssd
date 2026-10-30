
# [原创]从0到Reverseshell：Mikrotik SMB漏洞实战（CVE-2018–7445）

## 一.前言

最近阅读了 <a href="elink@650K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1@1K9r3!0#2i4K6u0W2j5$3!0E0i4K6u0r3N6Y4g2D9L8X3g2J5j5h3u0D9k6g2)9J5c8U0p5$3y4K6l9I4i4K6u0W2K9s2c8E0L8l9`.`.">模糊测试与漏洞利用实战：MikroTik无需认证的远程代码执行漏洞</a>，这几天动手实践了下，fuzz出“0day”的感觉真不赖，文章给出了一个新的利用方式（请勿用于非法用途）。大家有兴趣的也可以自己尝试下。

## 二.mikrotik环境搭建

磁盘文件：<mark class="encrypted">c71K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6o6u0Q4x3X3g2E0K9h3E0J5L8%4c8A6K9#2)9J5k6h3y4G2L8g2)9J5c8Y4u0G2N6i4c8W2M7X3!0K6i4K6u0r3y4W2)9J5k6e0b7H3i4K6u0W2y4g2)9J5c8X3y4Z5M7W2)9J5k6o6k6Q4x3X3f1@1x3q4)9J5k6e0g2Q4x3X3g2$3L8h3c8C8i4@1g2r3i4@1u0o6i4K6S2o6N6X3#2Q4c8e0g2Q4b7f1g2Q4z5o6W2Q4c8e0S2Q4b7e0y4Q4z5o6f1`.</mark>

![1554773965590](/upload/attach/201904/847726\_J44MU4YMEC7FJY8.png)

![1554773974492](/upload/attach/201904/847726\_CN8JYWB8X6WNFTW.png)

![1554773982646](/upload/attach/201904/847726\_CFVEEQEMVDVAKM2.png)

默认管理员帐户是`admin`，其默认密码为空。shell输入，开启`smb`自启，记下`ip`

`ip smb set enabled=yes`

`ip smb print`

`ip address print` 记录下`ip`

![1554773991790](/upload/attach/201904/847726\_B2HXGUCDMWYXFTD.png)

```
下载越狱工具https://github.com/0ki/mikrotik-tools，在 ubuntu 16 中运行./exploit_full.sh，填写前面记录的ip
root@ubuntu:~/Desktop/mikrotik-tools-master/exploit-backup# ./exploit_full.sh
。。。
We'll need the IP address of the device, user and password.
IP [192.168.88.1]: 192.168.74.140
USER [admin]:
PASS []:
We got admin@192.168.74.140 with password ''.
Is this correct? (y/N) y
。。。。
Would you like some additional utilities with your jailbreak? (y/N) y
。。。
Please be aware that telnet will stay enabled on 23/tcp!
# Enjoy your new shell via telnet using user 'devel' with admin's password.
```

之后可以登录使用`devel`+空密码（默认）登陆`telnet`可以进入`root` `lshell`

![1554774679413](/upload/attach/201904/847726\_3DFE3JEYQV22P2Z.png)

## 三.fuzz，触发漏洞

使用`Cisco-Talos`发布的`mutiny-fuzzer`进行`fuzz`，首先使用`smbclient`与`mikrotik`的`smb`交互，使用`wireshark`抓取请求包（只要第一个请求包），保存的`pcap`文件，`fuzz`，向`mikrotik`的`smb`发送请求，虽然`mutiny-fuzzer`带了一些检测模式，但是结合`wireshark`，其默认机制“如果连接不上，将停止`fuzz`“ 就已经可以达到目的。

`smb`路径`/nova/bin/smb`，日志文件路径`/rw/logs/backtrace.log`

### （一）fuzz环境搭建

漏洞挖掘工具<mark class="encrypted">8b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9i4y4U0L8#2)9J5k6q4c8S2L8r3!0K6i4K6u0r3L8i4g2@1K9h3&6&6i4K6u0V1k6Y4g2\*7P5X3g2J5i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1#2i4@1q4q4i4K6R3&6i4@1f1^5i4@1p5K6i4K6R3#2</mark>

`pip install --pre scapy;#--pre Include pre-release and development versions. By default, pip only finds stable versions.`

`git clone ` <mark class="encrypted">656K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9i4y4U0L8#2)9J5k6q4c8S2L8r3!0K6i4K6u0r3L8i4g2@1K9h3&6&6i4K6u0V1k6Y4g2\*7P5X3g2J5i4K6u0W2k6$3W2@1i4K6y4n7</mark>

`cd mutiny-fuzzer;`

`tar xfz radamsa-0.3.tar.gz;`

`cd radamsa-0.3/;`

`make;`

测试`radamsa`（用来变异数据包的）

`root@ubuntu:~/Desktop/fuzz_microtik/mutiny-fuzzer/radamsa-0.3/bin# echo testfuzz >testfuzz`

`root@ubuntu:~/Desktop/fuzz_microtik/mutiny-fuzzer/radamsa-0.3/bin# ./radamsa -n 5 testfuzz`

`tstuz`

`testfuzz`

`testfuzz`

`testfuzzsesestfufuzsestfuzsseseeeeeseteetesteesetestesetestfuzstesetesestfuz`

`testfuz`

将日志保存到硬盘

![1554776806309](/upload/attach/201904/847726\_XW8EYBBPUEABFD4.png)

![1554776814845](/upload/attach/201904/847726\_7PRDXGVDCX84YKK.png)

### （二）fuzz

交互窗口启动`smb`

![1554776990398](/upload/attach/201904/847726\_Z9AUKRXY4T8HQ3F.png)

`ubuntu` 使用`smbclient`与`mikrotik smb`交互，使用`wireshark`抓包，只保存第一个由`ubuntu`发送到`mikrotik` 的数据包，只需要保存这一个数据包，注意`wireshark`中的保存选项。

#### 交互并抓包

`root@ubuntu:~/Desktop# apt install smbclient`

`root@ubuntu:~/Desktop# smbclient -L \\\192.168.74.140`

`WARNING: The "syslog" option is deprecated`

`Enter root's password:`

`Server does not support EXTENDED_SECURITY but 'client use spnego = yes and 'client ntlmv2 auth = yes'`

`Anonymous login successful`

`Domain=[MSHOME] OS=[Unix] Server=[MikrotikSMB]`

`Sharename Type Comment`

`--------- ---- -------`

`pub Disk default share`

`Server does not support EXTENDED_SECURITY but 'client use spnego = yes and 'client ntlmv2 auth = yes'`

`Anonymous login successful`

`Domain=[MSHOME] OS=[Unix] Server=[MikrotikSMB]`

`Server Comment`

`--------- -------`

`MikroTik MikrotikSMB`

`Workgroup Master`

`--------- -------`

`MikroTik MikrotikSMB`

`==`

![1554779021342](/upload/attach/201904/847726\_E86WQH7KHYGBQKG.png)

![1554779046195](/upload/attach/201904/847726\_8NU47PM5C89HCBD.png)

![1554779053676](/upload/attach/201904/847726\_8WKSYEPPHWXNQUX.png)

![1554779098229](/upload/attach/201904/847726\_3HK73RDY5WVFES6.png)

#### 根据pcap生成变异

`root@ubuntu:~/Desktop/fuzz_microtik/mutiny-fuzzer# ./mutiny_prep.py theloginsession.pcapng`

`Processing theloginsession.pcapng...`

`Which port is the server listening on? (445/41888)`

`Default 445:`

`Message #0 - Processed 194 bytes outbound`

`Processed input file theloginsession.pcapng`

`How many times should a test case causing a crash or error be repeated?`

`Default 3: 不能连接了之后要试几次才最终停止`

`When the test case is repeated above, how many seconds should it wait between tests?`

`Default 5: 1 每次等待多久`

`Which protocol? (tcp/udp/layer3)`

`Default tcp:`

`What port should the fuzzer connect to?`

`Default 445:`

`Would you like to auto-generate a .fuzzer for each client message? (y/n)`

`Default n: y`

`Wrote .fuzzer file: theloginsession-0.fuzzer`

`All files have been written.`

![1554779857998](/upload/attach/201904/847726\_VH2VS4CSQVD3PYF.png)

#### 修改theloginsession-0.fuzzer文件

目的是触发更多的路径，这是`wireshark`解析的数据包格式，注意其中的`Requested Dialects`段，有10个`dialects`

![1554780103035](/upload/attach/201904/847726\_VEXRMZ9JWKNZGJW.png)

这是`theloginsession-0.fuzzer`文件，其实也就是`NetBIOS Session Service`的整个数据（见`wireshark`解析）

```
# Directory containing any custom exception/message/monitor processors
# This should be either an absolute path or relative to the .fuzzer file
# If set to "default", Mutiny will use any processors in the same
# folder as the .fuzzer file
processor_dir default
# Number of times to retry a test case causing a crash
failureThreshold 3
# How long to wait between retrying test cases causing a crash
failureTimeout 1
# How long for recv() to block when waiting on data from server
receiveTimeout 1.0
# Whether to perform an unfuzzed test run before fuzzing
shouldPerformTestRun 1
# Protocol (udp or tcp)
proto tcp
# Port number to connect to
port 445
# Port number to connect from
sourcePort -1
# Source IP to connect from
sourceIP 0.0.0.0
# The actual messages in the conversation
# Each contains a message to be sent to or from the server, printably-formatted
outbound fuzz '\x00\x00\x00\xbe\xffSMBr\x00\x00\x00\x00\x18C\xc8\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xfe\xff\x00\x00\x00\x00\x00\x9b\x00\x02PC NETWORK PROGRAM 1.0\x00\x02MICROSOFT NETWORKS 1.03\x00\x02MICROSOFT NETWORKS 3.0\x00\x02LANMAN1.0\x00\x02LM1.2X002\x00\x02DOS LANMAN2.1\x00\x02LANMAN2.1\x00\x02Samba\x00\x02NT LANMAN 1.0\x00\x02NT LM 0.12\x00'
```

作者最后只留下了两个`dialect`， 这里我一开始是有些疑惑的，毕竟之前没做过`fuzz`，后来想了想，多一个`dialect`永远不会增加代码覆盖，`dialect`越少，变异的空间就越大。之所以疑惑，就是之前的惯性思维：数据必须是符合规范的。

```
==
作者的：
outbound fuzz '\x00\x00\x00\xbe\xffSMBr\x00\x00\x00\x00\x18C\xc8\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xfe\xff\x00\x00\x00\x00\x00\x9b\x00\x02NT LANMAN 1.0\x00\x02NT LM 0.12\x00'
原：
outbound fuzz '\x00\x00\x00\xbe\xffSMBr\x00\x00\x00\x00\x18C
\xc8\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00
\x00\xfe\xff\x00\x00\x00\x00\x00\x9b\x00\x02PC NETWORK PROGRAM 1.0\x00\x02MICROSOFT NETWORKS 1.03\x00
\x02MICROSOFT NETWORKS 3.0\x00\x02LANMAN1.0\x00\x02LM1.2X002\x00\x02DOS LANMAN2.1\x00\x02LANMAN2.1
\x00\x02Samba\x00\x02NT LANMAN 1.0\x00\x02NT LM 0.12\x00'
==
修改theloginsession-0.fuzzer文件中的outbound保存为theloginsession-0-change.fuzzer
```

#### fuzz

`./mutiny.py -s 0.5 --logAll theloginsession-0-change.fuzzer mikrotik_ip` //每隔0.5s发送一次，保存所有信息

注意开启`wireshark` 抓取所有数据包

开始：

![1554781267149](/upload/attach/201904/847726\_XBEH4NZSEP5WBT8.png)

结束

![1554781766868](/upload/attach/201904/847726\_ZPY9E8CB79GSJTK.png)

`mikrotik`中的崩溃日志

![1554787858637](/upload/attach/201904/847726\_9MAAPYMFX9QY9UD.png)
