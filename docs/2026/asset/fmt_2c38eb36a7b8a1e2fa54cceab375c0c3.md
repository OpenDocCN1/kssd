
```bash
sudo nano /etc/ssh/sshd_config
```

只需取消对端口的注释，并将其更改为您想要的任何端口，然后保存文件并重新启动服务:

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_V368GM28JUQ4TJB.png)

全部保存后，您可以使用新端口ssh到您的服务器:

```bash
ssh -p 65532 zephr@zsec.uk -v
```

# SINGLE包授权

SPA是我以前从未听说过的东西，直到Tim问我是否在这篇文章中报道了它:

SPA类似于端口碰撞技术，它本质上是一种通过在一组预先指定的关闭端口上生成连接尝试来外部打开防火墙上的端口的方法。这不是什么新鲜事，但这是另一种模糊你的设置的方法，以保护它免受窥探。

端口碰撞技术是一种较早的第一代技术，它使用TCP和UDP数据包报头中的端口字段来传递信息。通常，这些协议用于封装应用层数据，然而端口碰撞技术通过使用端口号本身作为传输数据的字段，将信息编码成数据包序列发送到各个端口。

端口碰撞技术场景的一个例子是端口碰撞技术服务器配置一个数据包过滤器来阻止对服务(如SSH)的所有访问，直到端口碰撞技术客户端发送了特定的端口碰撞技术序列。例如，服务器可能要求客户端按顺序向以下端口发送TCP SYN数据包:

- 23400
- 1001
- 2003
- 65501

如果服务器监控这个knock序列，包过滤重新配置允许从发送它的IP地址SSH连接。通过利用包过滤提供的连接跟踪机制，在超时后由敲门服务器创建的初始规则被删除后，SSH会话可以保持建立。这通过模糊性增加了额外的安全性，这与SPA略有不同，SPA被认为是新的，它保留了端口碰撞技术的所有好处，但修复了旧技术带来的限制。

为了设置这一点，我们将安装fwknop，它代表防火墙敲门操作符。要安装和设置fwknop，我们需要在各自的主机上安装客户端和服务器:

```bash
# Server side:
sudo apt install fwknop-server

# Client Side:
sudo apt install fwknop-client
```

首先，我们需要设置端口以自动删除 SSH，我使用这个[指南](https://bbs.kanxue.com/elink@bb6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9X3!0W2K9$3W2J5i4K6u0r3j5K6M7J5x3h3f1@1x3X3q4U0x3r3p5I4y4U0c8W2k6o6y4W2k6o6g2X3j5X3j5#2k6X3p5%4x3o6W2V1x3U0b7`.)完整设置了 fwknop。所以下面的步骤就像从指南中复制的一样：

配置 UFW 以允许 UDP 端口 62201 作为默认值，这可以在配置文件中更改：

```bash
ufw allow 62201/udp comment 'SPA Setup for Port Knocking'
```

如下配置iptables:

```bash
iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -j DROP
iptables -I INPUT 1 -i eth0 -p tcp -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

然后，在安装fwknop-client之后，我们将首先在客户端上设置密钥:

使用`fwknop -K keys.txt`为您的 SPA 生成密钥和`HMAC_key`:

```bash
fwknop -K keys.tx
cat keys.txt

OUT:
KEY_BASE64: example
HMAC_KEY_BASE64: example
```

编辑`~/.fwknoprc`包括您的服务器和上面生成的密钥的详细信息:

```ini
[bastion]
SPA_SERVER          <Target IP>
ACCESS              tcp/22
KEY_BASE64          <BASE64>
HMAC_KEY_BASE64     <BASE64>
USE_HMAC            Y
```

接下来，我们需要将相同的密钥数据复制到服务器`/etc/fwknop/access.conf`上:

```ini
#### fwknopd access.conf stanzas ###

SOURCE ANY
KEY_BASE64: example
HMAC_KEY_BASE64: example
```

将所有数据复制到客户端和服务器后，通过运行以下命令检查服务器的状态:

```bash
systemctl start fwknop-server
systemctl status fwknop-server
```

服务运行后，您应该能够从主机运行以下命令以打开端口：

```bash
fwknop -n bastion -R --verbose
```

一切都很顺利，看起来应该有点像下面的截图，初始nmap显示端口关闭，然后运行命令打开端口:

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_TH9HMY9QS3YV58V.png)

执行bastion命令：

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_8J9GKVU7N9RQBH8.png)

命令后执行nmap扫描:

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_TYDHFWZ5CBAFA4S.png)

我不会假装理解它是如何工作的，但正如我们所看到的，神奇！打开端口并能够在以下位置进行ssh：

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_MXT893AKJ2KJ4EZ.png)

为了简化操作，我们还可以在`.bashrc`或`.zshrc`或您使用的任何shell中为命令添加别名，以便于记忆：

```bash
alias fwknock="fwknop -n bastion -R --verbose"
```

Tada! SPA所有的设置和我们的端口是隐藏的，如果你想进一步加强设置，你可以考虑改变SSH端口和端口的knocking端口。

# 登录通知

最后但并非最不重要的是，我们已经经历了强化设置的过程，但是如果我们没有适当的监控来告诉我们何时有人进行身份验证，那么强化设置有什么好处呢？进入登录通知的美丽，这是我前阵子在设置PoshC2和钴击的通知后设置的东西，SSH是合乎逻辑的下一步。

我将解释如何在两个不同的平台上做到这一点，这在任何支持webhooks的平台上都是切实可行的，因为这是对您的配置文件和一些web请求的一些调整。

## TELEGRAM机器人

Telegram的bot API非常适合做各种各样的事情，最简单的方法就是先构建你的bot，你可以通过向BotFather发送一条消息来实现:

```
/newbot
```

这将指示机器人创建一个新的机器人，它将提示您输入一个必须以机器人结尾的名称，一旦您为它提供了一个名称，它将为您创建一个新的机器人标识和一个应用编程接口令牌。

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_6MFK7DFFKG7RGBG.png)

下一步是设置一个组，在其中添加您自己、您刚刚创建的bot和IDbot。

IDBot将为您提供CHATID。您可以通过向channel/getgroupid发送消息来实现这一点。它将返回频道的组ID，只需在数字前加一个连字符，这就是CHATID:-123456789作为示例。抓取这个加上http API密钥，并将它们添加到以下脚本中。

```bash
# Login Notifications
CHATID=CHANGEME
BOTKEY=CHANGEME

# get hostname
HOSTNM=$( hostname )

# get external IP address
IP=$( curl -s http://whatismyip.akamai.com/ )

# find IP address of person last logged in
LOGININFO=$( last -1 -i | head -n 1 )

# parse into nice format
LOGININFO1=$( python3 -c "login='$LOGININFO'.split('   '); del login[1]; del login[1]; print(''.join([x.strip(' ') + '   \n' for x in login]));" )

# send information to telegram notification bot
curl -X POST -H 'Content-Type: application/json' -d "{\"chat_id\": \"$CHATID\", \"text\": \"Log in to: $HOSTNM\n$IP\nfrom: $LOGININFO1\", \"disable_notification\": false}" https://api.telegram.org/bot$BOTKEY/sendMessage --silent > /dev/null
```

然后可以将它捆绑到bash脚本中并从您的概要文件中调用，或者直接写入概要文件:

ZSH

```bash
nano /etc/zsh/zprofile
```

BASH

```bash
nano /etc/profile
```

保存后，每次通过SSH连接到主机并生成shell时，您将在设置的组中收到一个警报：

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_VHNWX3HT94GC95Z.png)

## SLACK机器人

在设置slack时，设置webhook的过程记录在slack的站点上。按照以下步骤设置webhook，然后您应该有一个如下所示的URL：

```
https://hooks.slack.com/services/VALUE1/VALUE2
```

在slack文档的帮助下，我编写了一个类似于上面bash脚本的python脚本，但是它没有发送到Telegram，而是发送到Slack。

```python
from slack_sdk.webhook import WebhookClient
import os

url = " a33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5L8$3!0C8M7#2)9J5k6i4y4D9j5h3y4C8i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2J5N6X3W2U0k6i4y4Q4x3V1k6h3b7f1I4g2c8e0q4Q4x3V1k6h3b7f1I4g2c8e0t1."
webhook = WebhookClient(url)
HOSTNM = os.popen('hostname').read()
IP = os.popen('curl -s 363K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6Z5j5i4c8A6M7$3#2&6K9i4m8Q4x3X3g2S2K9$3q4E0j5h3W2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3U0M7.').read()
LOGININFO=os.popen('last -1 -i | head -n 1 | cut -d " " -f 1').read()

response = webhook.send(text='Log in to: {}\n from {}\n by\n User: {} \n'.format(HOSTNM,IP,LOGININFO))
assert response.status_code == 200
assert response.body == "ok"
```

根据slack文档，您需要安装松弛软件开发工具包:

```bash
pip install slack_sdk
```

你想怎么保存都行`.py`并从您的配置文件中引用它，类似于telegram脚本:

```bash
python3 /etc/zsh/down.py
```

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_44RZY4Q2CURSMRC.png)

最后，当您向您的服务器进行身份验证时，您将会收到一个推送通知，通知到任何平台:

![图片描述](https://bbs.kanxue.com/upload/attach/202108/767964_QRGVNDDXQXPPZX4.png)

好了，一切准备就绪！

快速回顾一下，我们已经成功设置:

- 安全密钥
- MFA
- 防火墙
- 掩盖你的设置
- 登录通知

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2021-8-27 09:25
被梦幻的彼岸编辑
，原因：
