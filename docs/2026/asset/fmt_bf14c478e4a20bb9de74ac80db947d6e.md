
最后，还有`maxretry`。它定义了`findtime`中允许的失败登录尝试的确切次数。如果`findtime`内失败的授权尝试次数超过`maxretry`值，则IP将被禁止重新登录。默认值为5。

```
maxretry = 5
```

Fail2ban还允许您对自己选择的IP地址和IP范围授予豁免权。上面讨论的这些条件不会应用于这些IP，本质上是让您创建某种允许列表。

要将一个ip添加到此允许列表中，请修改`ignoreip`行并键入IP地址以免除，理想情况下，您希望您的家庭IP在这里，或者您经常访问主机的机器，以避免您将自己锁在外面！

```
ignoreip = 127.0.0.1/8 ::1 222.222.222.222 192.168.55.0/24
```

设置好您想要的所有选项后，只需启动服务，然后检查状态：

```
systemctl start fail2ban
systemctl status fail2ban
```

![图片描述](upload/attach/202108/767964_WCFBESKM89JFG2G.png)

# 掩盖你的设置

现在启用了密钥、MFA和限速，我们可以通过安全性进一步保护我们的设置。人们最常见的改变是将SSH端口从端口22移开，以降低暴力攻击和自动扫描攻击，结合良好的防火墙，可以使端口扫描看不到主机（然而，我不会在这篇文章中讨论这个问题）。

## 修改SSH端口

修改SSH端口就像在`/etc/ssh/sshd_config`中：

```
sudo nano /etc/ssh/sshd_config
```

只需取消对端口的注释，并将其更改为您想要的任何端口，然后保存文件并重新启动服务：

![图片描述](upload/attach/202108/767964_V368GM28JUQ4TJB.png)

全部保存后，您可以使用新端口`ssh`到您的服务器：

```
ssh -p 65532 zephr@zsec.uk -v
```

## SINGLE包授权

SPA是我以前从未听说过的东西，直到Tim问我是否在这篇文章中报道了它：

SPA类似于端口碰撞技术，它本质上是一种通过在一组预先指定的关闭端口上生成连接尝试来外部打开防火墙上的端口的方法。这不是什么新鲜事，但这是另一种模糊你的设置的方法，以保护它免受窥探。

端口碰撞技术是一种较早的第一代技术，它使用TCP和UDP数据包报头中的端口字段来传递信息。通常，这些协议用于封装应用层数据，然而端口碰撞技术通过使用端口号本身作为传输数据的字段，将信息编码成数据包序列发送到各个端口。

端口碰撞技术场景的一个例子是端口碰撞技术服务器配置一个数据包过滤器来阻止对服务（如SSH）的所有访问，直到端口碰撞技术客户端发送了特定的端口碰撞技术序列。例如，服务器可能要求客户端按顺序向以下端口发送TCP SYN数据包：

- 23400
- 1001
- 2003
- 65501

如果服务器监控这个knock序列，包过滤重新配置允许从发送它的IP地址SSH连接。通过利用包过滤提供的连接跟踪机制，在超时后由敲门服务器创建的初始规则被删除后，SSH会话可以保持建立。这通过模糊性增加了额外的安全性，这与SPA略有不同，SPA被认为是新的，它保留了端口碰撞技术的所有好处，但修复了旧技术带来的限制。

为了设置这一点，我们将安装`fwknop`，它代表防火墙敲门操作符。要安装和设置`fwknop`，我们需要在各自的主机上安装客户端和服务器：

```
Server side:
sudo apt install fwknop-server
Client Side:
sudo apt install fwknop-client
```

首先，我们需要设置端口以自动删除 SSH，我使用这个<a href="elink@6eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9X3!0W2K9$3W2J5i4K6u0r3j5K6M7J5x3h3f1@1x3X3q4U0x3r3p5I4y4U0c8W2k6o6y4W2k6o6g2X3j5X3j5#2k6X3p5%4x3o6W2V1x3U0b7`.">指南</a>完整设置了 fwknop。所以下面的步骤就像从指南中复制的一样：

配置 UFW 以允许 UDP 端口 62201 作为默认值，这可以在配置文件中更改：

```
ufw allow 62201/udp comment 'SPA Setup for Port Knocking'
```

如下配置`iptables`：

```
iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -j DROP
iptables -I INPUT 1 -i eth0 -p tcp -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

然后，在安装fwknop-client之后，我们将首先在客户端上设置密钥：

使用`fwknop -K keys.txt`为您的 SPA 生成密钥和`HMAC_key`

```
fwknop -K keys.tx
cat keys.txt
OUT:
KEY_BASE64: example
HMAC_KEY_BASE64: example
```

编辑`~/.fwknoprc`包括您的服务器和上面生成的密钥的详细信息：

```
[bastion]
SPA_SERVER <Target IP>
ACCESS tcp/22
KEY_BASE64 <BASE64>
HMAC_KEY_BASE64 <BASE64>
USE_HMAC Y
```

接下来，我们需要将相同的密钥数据复制到服务器`/etc/fwknop/access.conf`上：

```
#### fwknopd access.conf stanzas ###
SOURCE ANY
KEY_BASE64: example
HMAC_KEY_BASE64: example
```

将所有数据复制到客户端和服务器后，通过运行以下命令检查服务器的状态：

```
systemctl start fwknop-server
systemctl status fwknop-server
```

服务运行后，您应该能够从主机运行以下命令以打开端口：

```
fwknop -n bastion -R --verbose
```

一切都很顺利，看起来应该有点像下面的截图，初始`nmap`显示端口关闭，然后运行命令打开端口：

![图片描述](upload/attach/202108/767964_TH9HMY9QS3YV58V.png)

执行bastion命令：

![图片描述](upload/attach/202108/767964_8J9GKVU7N9RQBH8.png)

命令后执行`nmap`扫描：

![图片描述](upload/attach/202108/767964_TYDHFWZ5CBAFA4S.png)

我不会假装理解它是如何工作的，但正如我们所看到的，神奇！打开端口并能够在以下位置进行`ssh`：

![图片描述](upload/attach/202108/767964_MXT893AKJ2KJ4EZ.png)

为了简化操作，我们还可以在`.bashrc`或`.zshrc`或您使用的任何shell中为命令添加别名，以便于记忆：

```
alias fwknock="fwknop -n bastion -R --verbose"
```

Tada! SPA所有的设置和我们的端口是隐藏的，如果你想进一步加强设置，你可以考虑改变SSH端口和端口的knocking端口。

# 登录通知

最后但并非最不重要的是，我们已经经历了强化设置的过程，但是如果我们没有适当的监控来告诉我们何时有人进行身份验证，那么强化设置有什么好处呢？进入登录通知的美丽，这是我前阵子在设置PoshC2和钴击的通知后设置的东西，SSH是合乎逻辑的下一步。

我将解释如何在两个不同的平台上做到这一点，这在任何支持webhooks的平台上都是切实可行的，因为这是对您的配置文件和一些web请求的一些调整。

## TELEGRAM机器人

Telegram的bot API非常适合做各种各样的事情，最简单的方法就是先构建你的bot，你可以通过向BotFather发送一条消息来实现：

```
/newbot
```

这将指示机器人创建一个新的机器人，它将提示您输入一个必须以机器人结尾的名称，一旦您为它提供了一个名称，它将为您创建一个新的机器人标识和一个应用编程接口令牌。

![图片描述](upload/attach/202108/767964_6MFK7DFFKG7RGBG.png)

下一步是设置一个组，在其中添加您自己、您刚刚创建的bot和IDbot。

IDBot将为您提供`CHATID`。您可以通过向`channel/getgroupid`发送消息来实现这一点。它将返回频道的组ID，只需在数字前加一个连字符，这就是`CHATID:-123456789`作为示例。抓取这个加上http API密钥，并将它们添加到以下脚本中。

```
# Login Notifications
CHATID=CHANGEME
BOTKEY=CHANGEME
# get hostname
HOSTNM=$( hostname )
# get external IP address
IP=$( curl -s http://whatismyip.akamai.com/ )
# find IP address of person last logged in
LOGININFO=$( last -1 -i | head -n 1)
# parse into nice format
LOGININFO1=$( python3 -c "login='$LOGININFO'.split(' '); del login[1]; del login[1]; print(''.join([x.strip(' ') + ' \n' for x in login]));" )
# send information to telegram notification bot
curl -X POST -H 'Content-Type: application/json' -d "{\"chat_id\": \"$CHATID\", \"text\": \"Log in to: $HOSTNM\n$IP\nfrom: $LOGININFO1\", \"disable_notification\": false}" https://api.telegram.org/bot$BOTKEY/sendMessage --silent > /dev/null
```

然后可以将它捆绑到bash脚本中并从您的概要文件中调用，或者直接写入概要文件：

ZSH

```
nano /etc/zsh/zprofile
```

BASH

```
nano /etc/profile
```

保存后，每次通过SSH连接到主机并生成shell时，您将在设置的组中收到一个警报：

![图片描述](upload/attach/202108/767964_VHNWX3HT94GC95Z.png)

## SLACK机器人

在设置slack时，设置webhook的过程记录在slack的站点上。按照以下步骤设置webhook，然后您应该有一个如下所示的URL：

```
https://hooks.slack.com/services/VALUE1/VALUE2
```

在slack文档的帮助下，我编写了一个类似于上面bash脚本的python脚本，但是它没有发送到Telegram，而是发送到Slack。

```
from slack_sdk.webhook import WebhookClient
import os
url = " https://hooks.slack.com/services/VALUE1/VALUE2"
webhook = WebhookClient(url)
HOSTNM = os.popen('hostname').read()
IP = os.popen('curl -s http://whatismyip.akamai.com/').read()
LOGININFO=os.popen('last -1 -i | head -n 1 | cut -d " " -f 1').read()
response = webhook.send(text='Log in to: {}\n from {}\n by\n User: {} \n'.format(HOSTNM,IP,LOGININFO))
assert response.status_code == 200
assert response.body == "ok"
```

根据slack文档，您需要安装松弛软件开发工具包：

```
pip install slack_sdk
```

你想怎么保存都行`.py`并从您的配置文件中引用它，类似于telegram脚本：

```
python3 /etc/zsh/down.py
```

![图片描述](upload/attach/202108/767964_44RZY4Q2CURSMRC.png)

最后，当您向您的服务器进行身份验证时，您将会收到一个推送通知，通知到任何平台：

![图片描述](upload/attach/202108/767964_QRGVNDDXQXPPZX4.png)

好了，一切准备就绪！

快速回顾一下，我们已经成功设置：

- 安全密钥
- MFA
- 防火墙
- 掩盖你的设置
- 登录通知

# 备注：

原文地址：df7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2P5Y4y4W2j5#2)9J5k6i4g2C8i4K6u0r3L8r3!0U0K9$3W2F1k6#2)9J5k6r3c8G2N6$3&6Q4x3X3c8K6M7$3S2Q4x3X3c8@1K9r3g2Q4x3X3c8J5K9h3N6Z5N6q4)9J5k6s2N6S2P5g2)9J5c8R3`.`.

原文标题：LOCKING DOWN SSH - THE RIGHT WAY

原文信息：安全技能的一些[许多]问题

作者：Andy Gill

---

一个严加保护VPS或类似的小指南，以确保你的SSH连接是尽可能安全的。现在这已经不是什么新鲜事了，而是我最近学到的一些技巧和窍门，使我能够更好地保护我的服务器和主机。

我将重点介绍几个基础知识，让我们开始：

- 启用基于密钥的身份验证并生成安全密钥！

- 启用多因素身份验证（MFA）

- 启用限速和防火墙

- 掩盖你的设置

- 通过Telegram和Slack发送登录通知。

# 基于密钥的身份验证

基于密钥的SSH认证是今天在云中建立新机器时的事实标准，我学到的一个关键的东西是创建强大的密钥和加固/严加保护你的配置，可以在很大程度上帮助提供一个更安全的感觉，把一个主机放在互联网上。以安全的方式创建一个SSH密钥是非常简单的，但是在我们继续做一些之前，首先检查你是否拥有：

```
for key in ~/.ssh/*; do ssh-keygen -l -f "${key}"; done | uniq
```

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_35UYQM9Z5QWS3JD.png)

为了让那些想知道的人快速分解上述命令，它主要是通过`~/.ssh/`内的不同密钥，并显示指定公钥文件的指纹。如果你想了解该命令的全部内容，请查看这个解释 [shell](https://bbs.kanxue.com/elink@fc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2P5s2m8D9j5h3W2F1M7$3S2W2L8r3I4Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9j5h3W2F1i4K6y4r3j5$3#2V1i4K6y4p5k6X3!0J5i4K6u0n7K9$3g2&6i4K6u0n7K9h3&6Q4x3V1u0Q4x3U0f1%4c8g2)9J5y4e0u0r3i4K6u0W2M7%4y4Z5i4K6t1#2x3V1k6A6k6q4)9#2k6W2)9J5b7g2)9J5y4e0y4n7i4K6u0n7k6r3!0Q4x3V1u0K6M7$3S2Q4x3X3c8C8k6i4W2Y4k6h3&6Q4x3V1u0Q4x3X3c8D9i4K6u0n7i4K6u0V1k6W2)9J5b7W2)9J5y4e0t1J5i4K6t1#2x3U0c8Q4x3U0f1%4b7X3E0W2P5g2)9J5y4e0N6p5i4K6t1#2x3U0u0Q4x3U0f1K6b7W2)9J5b7X3c8G2L8X3g2Q4x3V1u0Q4x3U0f1%4b7#2)9J5b7Y4g2F1K9i4p5`.) 的链接。

如果这返回以下任何一种算法，可能值得升级或重新生成新的钥匙，这并不是说你需要销毁你的其他钥匙，而是考虑一些更强大的东西。

- DSA：它是不安全的，并且从OpenSSH第7版开始不再支持，因此请按照下面的步骤升级到一个更强大的密钥。

- RSA：这取决于密钥的大小。如果它有3072或4096位的长度，那么没有问题，但是如果它是其他的，建议升级到更强的密钥。

- ECDSA：这取决于你的机器在生成随机数方面的能力如何。

- Ed25519：这是目前最值得推荐的公钥算法！

为了生成ED25519密钥，可以使用以下命令：

```
ssh-keygen -t ed25519 -a 112 -f ~/.ssh/id_ed25519 -C "Insert a Comment Here"
```

该命令分解如下：

- `-t`：指定要创建的密钥类型，在本例中是`Ed25519`，但是默认情况下，我相信`ssh-keygen`会在标准Ubuntu和Debian上生成一个`RSA`密钥。
