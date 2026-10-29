
# [原创][翻译]严加保护SSH-正确的方法

## 备注

原文地址：<mark class="encrypted">789K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2P5Y4y4W2j5#2)9J5k6i4g2C8i4K6u0r3L8r3!0U0K9$3W2F1k6#2)9J5k6r3c8G2N6$3&6Q4x3X3c8K6M7$3S2Q4x3X3c8@1K9r3g2Q4x3X3c8J5K9h3N6Z5N6q4)9J5k6s2N6S2P5g2)9J5c8R3`.`.</mark>

原文标题：LOCKING DOWN SSH - THE RIGHT WAY

原文信息:安全技能的一些[许多]问题

作者：Andy Gill

一个严加保护VPS或类似的小指南，以确保你的SSH连接是尽可能安全的。现在这已经不是什么新鲜事了，而是我最近学到的一些技巧和窍门，使我能够更好地保护我的服务器和主机。

我将重点介绍几个基础知识，让我们开始:

* 启用基于密钥的身份验证并生成安全密钥！
* 启用多因素身份验证（MFA）
* 启用限速和防火墙
* 掩盖你的设置
* 通过Telegram和Slack发送登录通知。

# 基于密钥的身份验证

基于密钥的SSH认证是今天在云中建立新机器时的事实标准，我学到的一个关键的东西是创建强大的密钥和加固/严加保护你的配置，可以在很大程度上帮助提供一个更安全的感觉，把一个主机放在互联网上。以安全的方式创建一个SSH密钥是非常简单的，但是在我们继续做一些之前，首先检查你是否拥有:

```
for key in ~/.ssh/\*; do ssh-keygen -l -f "${key}"; done | uniq
```

![图片描述](upload/attach/202108/767964\_35UYQM9Z5QWS3JD.png)

为了让那些想知道的人快速分解上述命令，它主要是通过

`~/.ssh/`
内的不同密钥，并显示指定公钥文件的指纹。如果你想了解该命令的全部内容，请查看这个解释的链接。

如果这返回以下任何一种算法，可能值得升级或重新生成新的钥匙，这并不是说你需要销毁你的其他钥匙，而是考虑一些更强大的东西。

* `DSA`: 它是不安全的，并且从OpenSSH第7版开始不再支持，因此请按照下面的步骤升级到一个更强大的密钥。
* `RSA`:这取决于密钥的大小。如果它有3072或4096位的长度，那么没有问题，但是如果它是其他的，建议升级到更强的密钥。
* `ECDSA`：这取决于你的机器在生成随机数方面的能力如何。
* `Ed25519`: 这是目前最值得推荐的公钥算法!

为了生成ED25519密钥，可以使用以下命令:

```
ssh-keygen -t ed25519 -a 112 -f ~/.ssh/id\_ed25519 -C "Insert a Comment Here"
```

该命令分解如下:

* `-t`: 指定要创建的密钥类型，在本例中是Ed25519，但是默认情况下，我相信`ssh-keygen`会在标准Ubuntu和Debian上生成一个RSA密钥。
* `-a`: 这是密钥衍生函数（KDF）的轮数。数字越大，密码验证的速度就越慢，一旦私钥被盗，就会增加对暴力破解密码的阻力。我选择了112作为随机数，但它可以是任何数字。
* `-f`: 指定生成的密钥文件的文件名。要在ssh中默认使用它，它必须位于`.ssh`在您的主目录中，否则可以在ssh进入服务器时使用`-i`标志来指定:`ssh -i /path/to/id\_ed25519 user@host`
* `-C`: 指定注释的选项。这纯粹是信息，可以是任何东西。但是它通常填充了生成密钥的“在此插入注释”。当您的密钥被添加到主机的`authorized\_keys`文件时，这将是注释。

生成密钥后，下一步是确保SSH配置启用了基于SSH密钥的身份验证，禁用了基于密码的身份验证。为此，编辑SSH配置文件(是的，我知道vim vs nano...想用什么编辑器就用什么，我更喜欢是nano)。

```
sudo nano /etc/ssh/sshd\_config
```

我们希望在配置中取消注释两个选项:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

默认情况下，Ubuntu服务器的`PubkeyAuthentication`设置为 "是"，但是为了更加确定，启用它并禁用密码认证是最好的选择。此配置中的其他设置也可以改变，以增加日志的粗略程度，但我们在本节中不打算这样做。

最后，一旦设置被更改，我们需要通过添加我们的公钥将我们的密钥添加到`authorized\_keys`文件中:

```
cat ~/.ssh/id\_ed25519.pub >> ~/.ssh/authorized\_keys
```

这将把我们的公钥回显到授权密钥文件中，使我们能够进行SSH。请确保在退出会话之前测试您可以SSH到您的机器中，因为您可能需要处理设置，并且不想将自己锁在外面！

一旦您满意地完成了所有配置，并且密钥都在正确的位置，请重新启动SSH服务:

```
sudo systemctl reload sshd.service
```

接下来，一旦我们配置好了密钥，就该用MFA、限速和一些不确定因素来进一步严加保护访问。一个快速的测试显示，我们被提示输入密码，然后被认证（你会注意到认证提示我输入密码，然后是第二个验证码，这就是MFA的作用 ）

![图片描述](upload/attach/202108/767964\_2F33EFC6NT3KMS5.png)

## 强化授权钥匙

感谢弗洛里安·罗斯指出，授权密钥可以通过命令限制和外壳限制进一步强化，因此，如果您有特定用户，您希望限制对命令和转发等细节的访问。截图来自Florian's tweet:

![图片描述](upload/attach/202108/767964\_BVCGY5CUQ86KGUG.png)

# 多因素身份验证

要在SSH上启用MFA，可以配置几个选项，我选择使用google authenticator，但也有其他选项。这方面的设置相当简单，需要进行一些调整。

首先，更新并升级操作系统至最新版本（假设是基于debian/ubuntu）:

```
sudo apt update
sudo apt upgrade -y
```

一旦系统全部更新，接下来就是安装Google Authenticator PAM模块，这个模块可以做很多事情，但是我要关注的主要安装是用于SSH身份验证的MFA:

```
apt install libpam-google-authenticator
```

安装PAM应用程序后，您将需要使用PAM附带的助手应用程序来生成基于时间的一次性密码(TOTP){不是我最初认为的pops之首！}要添加第二个因子的用户的密钥。该密钥是基于每个用户生成的，因此不是系统范围的。

这意味着每个想要使用TOTP身份验证应用程序的用户都需要登录并运行助手应用程序来获取他们自己的密钥；你不能只运行一次就为所有人启用它。为此，只需运行:

```
google-authenticator
```

运行该命令后，您将被询问几个问题。第一个问题是身份验证令牌是否应该基于时间。我们希望选择y:

```
Do you want authentication tokens to be time-based (y/n) y
```

一旦我们选择是，该应用程序将生成一个二维码，可以在您选择的认证应用程序中扫描，我使用谷歌认证器，因为我在其他网站和服务器上也有它。该PAM允许基于时间或基于顺序的令牌。使用基于序列的标记意味着代码从某个点开始，然后在每次使用后递增代码。使用基于时间的令牌意味着代码在经过一定时间(通常是60秒)后随机变化。我们将坚持基于时间的，因为这是像谷歌认证器这样的应用程序所期望的，所以回答y表示是。

![图片描述](upload/attach/202108/767964\_G376QC77FGDVKWG.png)

一旦你扫描二维码，你的应用程序将生成一个TOTP，你可以输入到命令行界面，它会提示你保存紧急代码，像其他多功能事务机一样，如果你的动态口令应用程序不能作为手动覆盖，请暂时保存这些代码。

剩余的问题告知PAM如何运行。我们将一个一个地检查它们。

```
Do you want me to update your "/home/zephr/.google\_authenticator" file? (y/n)
```

这会将键和选项写入`.google\_authenticator`文件。如果选择“否”，程序将退出，并且不保存任何内容，这又会导致验证器应用程序不起作用。因此，我们要为此选择“yes”！

```
By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n
```

对该问题选择“是”可在移动的四分钟窗口内启用多达8个有效代码。通过回答“否”，您可以在90秒的滚动窗口内将其限制为3个有效代码。除非您发现90秒窗口有问题，否则回答“no”是更安全的选择。

```
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

限速意味着远程攻击者在被阻止之前只能尝试一定数量的猜测。如果您以前没有将限速直接配置到SSH中，那么现在正是配置它的最佳时机，我们将在下一节更深入地讨论配置UFW。

完成此设置后，如果要备份密钥，可以将`~/.google authenticator`文件复制到受信任的位置。从这里开始，您可以将其部署到其他系统上，或在备份后重新部署，这确实意味着在多个系统之间共享相同的TOTP代码，这不如单独的代码安全！

完成配置google authenticator的步骤后，下一步是设置SSH配置以允许authenticator正常工作：

```
sudo nano /etc/pam.d/sshd
```

在文件的底部添加以下一行

```
# Standard Un\*x password updating.
@include common-password
auth required pam\_google\_authenticator.so nullok
```

您的配置可能如下所示:

![图片描述](upload/attach/202108/767964\_UHP3NS7TJZZFKDK.png)

最后一行末尾的`nullok`字告诉PAM，这种身份验证方法是可选的。这允许没有TOTP誓言令牌的用户仍然使用他们的SSH密钥登录。一旦所有用户都有了一个宣誓-TOTP令牌，您就可以从这一行中删除`nullok`，以强制执行MFA。

接下来，我们将配置SSH来支持这种身份验证。打开SSH配置文件进行编辑。

```
sudo nano /etc/ssh/sshd\_config
```

查找`ChallengeResponseAuthentication`，并将其值设置为`yes`。我们还将通过添加身份验证方法和UsePAM来使SSH了解MFA:

```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication yes
UsePAM yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

此外，查找行`@ include-common-auth`，并通过添加`#`字符作为该行的第一个字符来注释掉它。这告诉PAM不要提示输入密码。

保存配置并重新启动SSH服务。

```
sudo systemctl reload sshd.service
```

然后再次测试您是否能够进行身份验证，现在应该会提示您输入SSH密钥密码和验证码:

![图片描述](upload/attach/202108/767964\_WDZ9ZTPRXY3BTHA.png)

一旦设置了验证，您就完成了保护服务器的第二阶段，通过在两个通道(您的计算机+您的电话)上拥有两个因素(SSH密钥+ MFA令牌)，您使得攻击者很难通过SSH强行进入您的主机，并大大提高了您的主机的安全性。

# 限速和防火墙

现在，我们已经完成了一些身份验证强化，接下来是将身份验证尝试限制在我们的服务器上，这也可以通过多种方式完成，第一种可能是最直接的，并使用简单防火墙(UFW)。

这是迄今为止讨论的所有设置中最简单的，因为它可以通过两个命令完成(如果您没有安装它):

```
sudo apt install ufw
## ufw limit ssh various usage ##
ufw limit ssh comment 'Rate limit for openssh server'
### if sshd is running on tcp port 2022 add ####
ufw limit 2022/tcp comment 'SSH port rate limit'sudo apt install ufw
## ufw limit ssh various usage ##
ufw limit ssh comment 'Rate limit for openssh server'
### if sshd is running on tcp port 2022 add ####
ufw limit 2022/tcp comment 'SSH port rate limit'
```

上述规则对于防范暴力登录攻击非常有用。当使用限制规则时，ufw通常会允许连接，但如果一个IP地址试图在三十秒内发起六个或更多连接，ufw将拒绝连接。ufw支持按规则记录日志。默认情况下，当数据包与规则匹配时，不会执行日志记录。

指定`log`将记录所有与规则匹配的新连接，而`log-all`将记录所有与规则匹配的数据包。例如，要允许并记录所有新的ssh连接，请使用:

```
ufw limit log ssh comment 'Rate limit for SSH'
```

如果您想更进一步，使用UFW将允许您将服务锁定到特定的ip地址:

```
sudo ufw allow from 1.2.3.4 to any port 22
```

这将只允许`1.2.3.4` SSH到端口22，当您对主机进行端口扫描时，该端口将显示为关闭或已过滤。

# 设置FAIL2BAN

单就UFW而言，设置非常好，因为它在引擎盖下使用了IP表，但是如果进一步检测和响应，我们可以设置Fail2Ban。Fail2Ban本质上是主动寻找潜在的密码身份验证滥用迹象，以过滤掉IP地址，并定期更新系统防火墙，以在一定时间内暂停这些IP地址。

要设置fail2ban，您只需安装并启用它:

```
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.{conf,local}
sudo nano /etc/fail2ban/jail.local
```

编辑以下几行来调整设置:

```
bantime = 1d
```

也可以通过赋予负值来永久禁止。

另一个非常重要的变量是`findtime`。它定义了连续登录尝试之间允许的持续时间。如果多次登录尝试是在`findtime`定义的时间内进行的，则会在IP上设置禁令。

```
findtime = 10m
```
