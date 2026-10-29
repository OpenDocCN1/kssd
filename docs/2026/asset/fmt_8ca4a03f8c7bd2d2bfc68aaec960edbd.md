
# 在Windows环境中利用Responder工具窃取NTLMv2哈希

翻译自：<mark class="encrypted">a30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3y4J5k6h3c8A6j5X3I4W2K9h3&6V1K9i4y4Z5k6h3I4D9i4K6u0r3g2$3W2F1k6r3!0%4M7#2)9J5k6p5q4p5i4K6u0V1k6h3&6$3K9i4u0G2L8X3#2W2L8Y4c8Q4x3X3c8J5k6h3I4S2N6r3g2V1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2u0W2M7%4m8G2L8X3c8W2M7R3`.`.</mark>

翻译：lipss

校对：看雪翻译小组 一壶葱茜

responder 下载地址:

- <<mark class="encrypted">009K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9k6$3q4F1k6s2S2Q4x3V1k6d9k6i4y4H3L8$3&6V1k6i4t1`.</mark>>

该工具有向LLMNR和NBT-NS请求进行欺骗的能力。

```
LLMNR（Link-Local Multicast Name Resolution，链路本地多播名称解析）协议是一种基于DNS包格式的协议。它可以将主机名解析为IPv4和IPv6的IP地址。这样用户就可以直接使用主机名访问特定的主机和服务，而不用记忆对应的IP地址。该协议被广泛使用在Windows Vista/7/8/10操作系统中。
该协议的工作机制很简单。例如，计算机A和计算机B同处一个局域网中。当计算机A请求主机B时，先以广播形式发送一个包含请求的主机名的UDP包。主机B收到该UDP包后，以单播形式发送UDP的响应包给主机A。由于整个过程中，都是以UDP方式进行，主机A根本不能确认响应主机B是否为该主机名对应的主机。这就造成欺骗的可能。
```

假设我们已连接到Windows Active Directory环境，当网络中任何一台计算机尝试使用LLMNR或NBT-NS请求来解析目标计算机时，Responder假装为目标计算机。当受害者计算机尝试登录到攻击者计算机时，Responder就可以捕获受害者计算机用户的NTLMv2哈希。

文章基于例子讨论两种攻击方式。

- 1、捕获NTLMv2哈希并使用`Hashcat`密码破解工具破解。
- 2、使用Responder和`Multirelay.py`脚本执行中转NTLMv2哈希到“已禁用SMB签名”的计算机。如果中转成功，在目标计算机上将获得shell访问权限。

## 捕获NTLMv2哈希值

通过使用参数`-I`指定机器的以太网接口来运行Responder，这个例子中使用的接口是`eth0`。

```
python Responder.py -I <Interface_card_name>
```

命令是：

```
python Responder.py -I eth0
```

![](upload/attach/201901/214974\_4C99DVE695F2XKJ.png)

Responder监听模式将是这样的

![](upload/attach/201901/214974\_VG37Z4U8SQ343TM.png)

如果网络中的任何用户试图访问共享哪个机器没有IP或用户类型共享名称不正确，该机器将触发LLMNR请求到网络，Responder将通过说我是那台机器来回答该请求，请提供给我NTLMv2哈希并访问资源。

![](upload/attach/201901/214974\_56XRAMK2ZUVJKPB.png)

这里，用户“box1”（机器IP `192.168.56.101`）正在尝试访问名称为“pwned68”的共享。当机器触发LLMNR请求时，Responder响应该请求并从域“DC2”捕获用户“box1”的NTLMv2哈希值。

现在，我们需要破解此哈希以获取纯文本密码。`Hashcat`是执行最快的hashcracking的绝佳工具。它支持CPU / GPU散列破解，并支持多种哈希格式。

Hahcat官方下载网站是：

```
- <https://hashcat.net/hashcat/>
```

下载一个好的密码字典，这里有`<<mark class="encrypted">27bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4y4Z5K9$3W2D9L8r3g2J5i4K6u0W2j5$3!0Q4x3X3g2#2K9#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5k6h3q4K6M7s2R3`.</mark>> `

运行`hashcat`并等待，如果运气好的话。

![](upload/attach/201901/214974\_5REGYVG8CJH34YV.png)

获取纯文本密码后，我们可以使用此密码登录Windows域上的其他计算机，以查看我们是否可以在其他计算机上访问任何敏感信息。

## 在网络中的计算机上访问Shell

Responder还有一些其他工具允许攻击者通过转发NTLMv2 哈希来执行攻击，例如获取对网络中机器的shell访问权限。这里条件是，如果捕获哈希的用户在目标机器上具有“administrator”访问权限，则捕获的哈希可以帮助我们获得shell访问。普通域用户哈希不会帮助我们在机器上获得shell访问权限。在我给出的案例中，其中一个管理员用户试图访问一个不存在的共享，Responder投毒成功，然后`Multirelay.py`脚本使用捕获的NTLMv2哈希登录到Windows域网络的一部分机器。

设置此项，在`Responder.conf`文件中进行更改。 打开`Responder.conf`文件并将SMB和HTTP的值设置为“Off”，以便Responder不捕获哈希值，但`Multirelay.py`执行任务。

![](upload/attach/201901/214974\_VFM7BRKM7PXYZFV.png)

现在运行`RunFinger.py`脚本来识别网络中已将“SMB signing”设置为“False”的HOST计算机，因为我们只能对那些没有将“SMB signing”设置为“True”的计算机执行攻击。该脚本位于Responder的”tools“目录中，首先进入该目录。`RunFinger.py`脚本需要IP范围，它将检查活动主机是否启用了SMB签名。运行`RunFinger.py`脚本的命令是：

```
- python RunFinger.py -i IP_Range
```

Example:

```
- python RunFinger.py -i 192.168.56.100-200
```

![](upload/attach/201901/214974\_37VPQQUCWYPAVCZ.png)

脚本的输出将是这样的:

![](upload/attach/201901/214974\_VZ56HWSX578HZ3X.png)

在我的例子里，只有2台机器在线。一个是域控制器机器，第二个是“Windows 7”，它是Windows域的一部分。具有IP `192.168.56.101`的计算机未启用SMB签名。因此，我们可以尝试对这台机器执行NTLMv2哈希转发。在发现机器之后，让我们设置responder和`multirelay.py`脚本以获得未启用“SMB signing”的机器上的访问权限。运行下面提到的命令，在2个不同的终端中启动Responder和`Multirelay.py`脚本

在第一个终端中，运行responder

```
python Responder.py -I <interface_card>
```

在第二个终端中，运行`Multirelay.py`

```
python MultiRelay.py -t <target_machine_IP> -u ALL
```

![](upload/attach/201901/214974\_RUY24Z9PTWAEC44.png)

这两个脚本都是这样的：

![](upload/attach/201901/214974\_MU54AWT7QEGE25C.png)

如果Domain中的任意管理员用户尝试访问不存在的/拼写错误的共享，Responder将向被响应的机器投毒（发出伪造请求）。`Multirelay.py`将通过捕获NTLMv2哈希完成剩余的工作，并将它们关联到目标机器。成功的中转将让我们在目标机器上访问shell（在我的例子中是Windows 7）

![](upload/attach/201901/214974\_PP4KS44Q8AD9FBH.png)

一旦我们获得shell访问权限，我们也可以执行其他工作，其中一个正在加载“`Mimikatz`”，它可以执行诸如转储当前登录用户的纯文本密码/哈希或从SAM文件中获取本地用户哈希等任务。

![](upload/attach/201901/214974\_C58WEBN53YTAS5T.png)

== [[Greetz to]] == -

zero cool, code breaker ica, root_devil, google_warrior, INX_r0ot,Darkwolf indishell, Baba, Silent poison India, Magnum sniper, ethicalnoob Indishell, Local root indishell, Irfninja indishell, Reborn India, L0rd Crus4d3r, cool toad, Hackuin, Alicks, Gujjar PCP, Bikash, Dinelson Amine, Th3 D3str0yer, SKSking, rad paul, Godzila, mike waals, zoo zoo, cyber warrior, shafoon, Rehan manzoor, cyber gladiator, 7he Cre4t0r, Cyber Ace, Golden boy INDIA, Ketan Singh, Yash, Aneesh Dogra, AR AR, saad abbasi, hero, Minhal Mehdi, Raj bhai ji, Hacking queen, lovetherisk, D2 and rest of TEAM INDISHELL

--==[[Love to]]==--

My Father,Ritu Tomer Rathi,cold fire hacker, Mannu, ViKi, Ashu bhai ji, Soldier Of God, Bhuppi, Gujjar PCP, rafay baloch Mohit, Ffe, Ashish, Shardhanand, Budhaoo, Jagriti, Salty, Hacker fantastic, Jennifer Arcuri, Don(Deepika kaushik) and all lovely people around the world <3

翻译自：bbaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3y4J5k6h3c8A6j5X3I4W2K9h3&6V1K9i4y4Z5k6h3I4D9i4K6u0r3g2$3W2F1k6r3!0%4M7#2)9J5k6p5q4p5i4K6u0V1k6h3&6$3K9i4u0G2L8X3#2W2L8Y4c8Q4x3X3c8J5k6h3I4S2N6r3g2V1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2u0W2M7%4m8G2L8X3c8W2M7R3`.`.

翻译：lipss

校对：看雪翻译小组 一壶葱茜

responder 下载地址:

- [661K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9k6$3q4F1k6s2S2Q4x3V1k6d9k6i4y4H3L8$3&6V1k6i4t1`.](https://bbs.kanxue.com/elink@1b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9k6$3q4F1k6s2S2Q4x3V1k6d9k6i4y4H3L8$3&6V1k6i4t1`.)

该工具有向LLMNR和NBT-NS请求进行欺骗的能力。

```
LLMNR（Link-Local Multicast Name Resolution，链路本地多播名称解析）协议是一种基于DNS包格式的协议。它可以将主机名解析为IPv4和IPv6的IP地址。这样用户就可以直接使用主机名访问特定的主机和服务，而不用记忆对应的IP地址。该协议被广泛使用在Windows Vista/7/8/10操作系统中。
该协议的工作机制很简单。例如，计算机A和计算机B同处一个局域网中。当计算机A请求主机B时，先以广播形式发送一个包含请求的主机名的UDP包。主机B收到该UDP包后，以单播形式发送UDP的响应包给主机A。由于整个过程中，都是以UDP方式进行，主机A根本不能确认响应主机B是否为该主机名对应的主机。这就造成欺骗的可能。
```

假设我们已连接到Windows Active Directory环境，当网络中任何一台计算机尝试使用LLMNR或NBT-NS请求来解析目标计算机时，Responder假装为目标计算机。当受害者计算机尝试登录到攻击者计算机时，Responder就可以捕获受害者计算机用户的NTLMv2哈希。

文章基于例子讨论两种攻击方式。

- 1、捕获NTLMv2哈希并使用“Hashcat”密码破解工具破解。
- 2、使用Responder和`Multirelay.py`脚本执行中转NTLMv2哈希到“已禁用SMB签名”的计算机。如果中转成功，在目标计算机上将获得shell访问权限。

## 捕获NTLMv2哈希值

通过使用参数`-I`指定机器的以太网接口来运行Responder，这个例子中使用的接口是`eth0`。

```
python Responder.py -I <Interface_card_name>
```

命令是：

```
python Responder.py -I eth0
```

![](https://bbs.kanxue.com/upload/attach/201901/214974_4C99DVE695F2XKJ.png)

Responder监听模式将是这样的

![](https://bbs.kanxue.com/upload/attach/201901/214974_VG37Z4U8SQ343TM.png)

如果网络中的任何用户试图访问共享哪个机器没有IP或用户类型共享名称不正确，该机器将触发LLMNR请求到网络，Responder将通过说我是那台机器来回答该请求，请提供给我NTLMv2哈希并访问资源。

![](https://bbs.kanxue.com/upload/attach/201901/214974_56XRAMK2ZUVJKPB.png)

这里，用户“box1”（机器IP `192.168.56.101`）正在尝试访问名称为“pwned68”的共享。当机器触发LLMNR请求时，Responder响应该请求并从域“DC2”捕获用户“box1”的NTLMv2哈希值。

现在，我们需要破解此哈希以获取纯文本密码。“Hashcat”是执行最快的hashcracking的绝佳工具。它支持CPU / GPU散列破解，并支持多种哈希格式。

Hahcat官方下载网站是：

```
 - <1b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4y4Z5j5$3q4@1i4K6u0W2L8X3g2@1i4K6u0r3K9r3q4K6K9r3y4S2N6q4)9J5c8W2)9J5y4X3N6@1i4K6y4n7
```

下载一个好的密码字典，这里有`` <e70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4y4Z5K9$3W2D9L8r3g2J5i4K6u0W2j5$3!0Q4x3X3g2#2K9#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5k6h3q4K6M7s2S2Q4x3U0k6Y4N6q4)9K6b7R3`.`. ``

运行hashcat并等待，如果运气好的话。

![](https://bbs.kanxue.com/upload/attach/201901/214974_5REGYVG8CJH34YV.png)

获取纯文本密码后，我们可以使用此密码登录Windows域上的其他计算机，以查看我们是否可以在其他计算机上访问任何敏感信息。

## 在网络中的计算机上访问Shell

Responder还有一些其他工具允许攻击者通过转发NTLMv2 哈希来执行攻击，例如获取对网络中机器的shell访问权限。这里条件是，如果捕获哈希的用户在目标机器上具有“administrator”访问权限，则捕获的哈希可以帮助我们获得shell访问。普通域用户哈希不会帮助我们在机器上获得shell访问权限。在我给出的案例中，其中一个管理员用户试图访问一个不存在的共享，Responder投毒成功，然后`Multirelay.py`脚本使用捕获的NTLMv2哈希登录到Windows域网络的一部分机器。

设置此项，在“`Responder.conf`”文件中进行更改。 打开`Responder.conf`文件并将SMB和HTTP的值设置为“Off”，以便Responder不捕获哈希值，但`Multirelay.py`执行任务。

![](https://bbs.kanxue.com/upload/attach/201901/214974_VFM7BRKM7PXYZFV.png)
