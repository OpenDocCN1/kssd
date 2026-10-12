
### 3.3 程序代码

#### Nickname Conflict Resolution

`03/16-09:19:48.633843 172.16.198.129:6667 -> 172.16.198.130:1025`
`TCP TTL:64 TOS:0x0 ID:233 IpLen:20 DgmLen:147 DF***AP*** Seq: 0x3FCD1E87 Ack: 0x301A2 Win: 0x7D78 TcpLen: 20`
`3A 69 72 63 2E 74 69 63 6B 6C 61 62 73 2E 63 6F :irc.ticklabs.co`
`6D 20 34 33 33 20 2A 20 6D 69 6B 65 79 20 3A 4E m 433 * mikey :N`
`69 63 6B 6E 61 6D 65 20 69 73 20 61 6C 72 65 61 ickname is alrea`
`64 79 20 69 6E 20 75 73 65 2E 0D 0A 3A 69 72 63 dy in use...:irc`
`2E 74 69 63 6B 6C 61 62 73 2E 63 6F 6D 20 34 35 .ticklabs.com 45`
`31 20 2A 20 4A 4F 49 4E 20 3A 52 65 67 69 73 74 31 * JOIN :Regist`
`65 72 20 66 69 72 73 74 2E 0D 0A er first...`

`03/16-09:19:48.651467 172.16.198.130:1025 -> 172.16.198.129:6667`
`TCP TTL:128 TOS:0x0 ID:7936 IpLen:20 DgmLen:61 DF***AP*** Seq: 0x301A2 Ack: 0x3FCD1EF2 Win: 0x2128 TcpLen: 20`
`4E 49 43 4B 20 59 76 0A 4A 4F 49 4E 20 23 64 61 NICK Yv.JOIN #da`
`46 75 63 6B 0A Fuck.`

图 3-8

该木马似乎有意地确保至少多个实例之一持有“mikey”这个名字。如果持有该名字的那个木马实例从 IRC 服务器断开连接，程序的另一个实例最多会在 3 秒钟内获取这个名字。该木马连接上网的实例越多，它们当中被冠于“mikey”之名的可能性就越大。`Doug Kahler` 在他与我们联系的 e-mail 中建议，该木马的目的之一可能是为它的操作者保留昵称或者阻止有人得到它。可能该程序的作者能够通过 IRC 消息与它通信、命令该木马释放这个名字以便那个人可能获得它。

我们有一个实验针对，检验在一个潜在攻击者和该木马多个实例之间通信的性质。IRC 是用来中心控制一大群分布式攻击代理的一个极好的频道，这是该木马比如 `srvcp.exe` 经常被人猜测具有分布式拒绝服务能力的原因之一。通过在 IRC 频道上发出一个单一命令，攻击者能够与该木马多个实例通信，由该服务器决定转播消息给已连接上网的木马。IRC 通信的这个性质使得很难追溯攻击来源。此外，IRC 提供通过私人消息单独与该木马每个实例沟通的能力。

实现一对多和一对一的 IRC 消息都是使用下层 IRC 协议的“`PRIVMSG`”命令，如下面图 3-9 所示范。(为了表示清楚，我们从跟踪记录里删掉无关数据包)。开始两个数据包携带一个消息有意让所有的频道参与者见到；发送这个消息只要在 IRC 客户端提示符下输入“hi all”。我们的昵称被设置为“attacker”，我们的真实名字是“lzeltser”，而且我们从“`127.0.0.1`”被连接。可以看到该服务器单独发送这个消息给该木马的每个实例，它们藉由不同的目的地 IP 地址被识别。第三个数据包携带一个消息预定只被持有昵称“mikey”的那个木马实例见到；在 IRC 客户端提示符下输入“`/msg mikey just for you`”发送这个信息。对于这两种情况，我们都不能够通过发送 IRC 消息给该木马而引起它的任何响应，稍后对程序代码的分析揭示了，需要以适当的方式加密命令才能让该木马理解它们。

#### Relaying Messages to IRC Clients

`03/16-09:43:15.650781 172.16.198.129:6667 -> 172.16.198.130:1025`
`TCP TTL:64 TOS:0x0 ID:962 IpLen:20 DgmLen:94 DF***AP*** Seq: 0x46C1D0D2 Ack: 0x404F7 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 23 64 61 46 75 63 6B 20 3A 68 69 VMSG #daFuck :hi`
`20 61 6C 6C 0D 0A all..`

`03/16-09:43:15.651144 172.16.198.129:6667 -> 172.16.198.131:1030`
`TCP TTL:64 TOS:0x0 ID:963 IpLen:20 DgmLen:94 DF***AP*** Seq: 0x3498DF73 Ack: 0xFF37 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 23 64 61 46 75 63 6B 20 3A 68 69 VMSG #daFuck :hi`
`20 61 6C 6C 0D 0A all..`

`03/16-09:43:49.946018 172.16.198.129:6667 -> 172.16.198.131:1030`
`TCP TTL:64 TOS:0x0 ID:989 IpLen:20 DgmLen:98 DF***AP*** Seq: 0x3498DFE1 Ack: 0xFFB0 Win: 0x7D78 TcpLen: 20`
`3A 61 74 74 61 63 6B 65 72 21 6C 7A 65 6C 74 73 :attacker!lzelts`
`65 72 40 31 32 37 2E 30 2E 30 2E 31 20 50 52 49 er@127.0.0.1 PRI`
`56 4D 53 47 20 6D 69 6B 65 79 20 3A 6A 75 73 74 VMSG mikey :just`
`20 66 6F 72 20 79 6F 75 0D 0A for you..`

图 3-9

本文前面说过，该木马试图在启动后从系统目录读 `gus.ini` 文件。当我们把 `gus.ini` 文件放进系统目录内的时候，`Filemon` 显示该木马成功地打开并读文件。使用正运行在 Linux 实验室机器上的 `Snort`，我们能够观察到该木马尝试连接到 IRC 服务器上的 `TCP` 端口 `6666`，而非先前使用的 `TCP` 端口 `6667`。因为我们的服务器没有在这个端口上监听，所以这个连接尝试失败。如下面图 3-10 所示，我们观察到该木马随后尝试解析一些 IRC 服务器的主机名字。(为了表示清楚，我们从跟踪记录上删掉 `TCP` 重试数据包)。该木马继续尝试解析其他一些似乎与 IRC 有关的主机名直到我们杀死该进程。

#### Resolving IRC Server Names

`03/16-10:29:11.319877 172.16.198.131:1040 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:63756 IpLen:20 DgmLen:61`
`Len: 41`
`00 01 01 00 00 01 00 00 00 00 00 00 05 65 66 6E .............efn`
`65 74 02 63 73 03 68 75 74 02 66 69 00 00 01 00 et.cs.hut.fi....`
`01 .`

`03/16-10:29:44.423230 172.16.198.131:1042 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:1037 IpLen:20 DgmLen:63`
`Len: 43`
`00 03 01 00 00 01 00 00 00 00 00 00 05 65 66 6E .............efn`
`65 74 05 64 65 6D 6F 6E 02 63 6F 02 75 6B 00 00 et.demon.co.uk..`
`01 00 1 ...`

`03/16-10:30:15.667072 172.16.198.131:1044 -> 172.16.198.1:53`
`UDP TTL:128 TOS:0x0 ID:3341 IpLen:20 DgmLen:64`
`Len: 44`
`00 05 01 00 00 01 00 00 00 00 00 00 03 69 72 63 .............irc`
`0A 63 6F 6E 63 65 6E 74 72 69 63 03 6E 65 74 00 .concentric.net.`
`00 01 00 1 ....`

图 3-10

为了响应观察到的行为，我们在我们的服务器上配置 IRC 精灵(daemon)以监听 `TCP` 端口 `6666`。一旦该木马被重新启动，`Snort` 日志显示该程序连接到 IRC 服务器 `TCP` 端口 `6666`。然而，如下面图 3-11 示例，该木马现在使用键“`soup`”加入频道“`#mikag`”。我们还来不及使该木马可用 `gus.ini`，它就使用了一个不同的频道名字，而且没有提供频道键。(在 IRC 上这个键有时被用来限制对一个频道的存取)。事实上，该木马现在的行为与 `Doug` 报告的观察结果相符，这在本文的‘背景信息’小节中先已描述过。

#### Connecting to a Different IRC Channel

`03/16-11:07:12.816149 172.16.198.131:1032 -> 172.16.198.129:6666`
`TCP TTL:128 TOS:0x0 ID:34304 IpLen:20 DgmLen:112 DF***AP*** Seq: 0xF46A Ack: 0x28E76CC2 Win: 0x2238 TcpLen: 20`
`4E 49 43 4B 20 3A 6D 69 6B 65 79 0A 55 53 45 52 NICK :mikey.USER`
`20 49 66 4A 64 43 6E 20 49 66 4A 64 43 6E 20 49 IfJdCn IfJdCn I`
`66 4A 64 43 6E 20 3A 66 69 67 68 74 20 6D 65 2C 66JdCn :fight me,`
`20 70 75 73 73 79 0A 4A 4F 49 4E 20 23 6D 69 6B  pussy.JOIN #mik`
`61 67 20 73 6F 75 70 0A ag soup.`

图 3-11

在环球网上搜索 `srvcp.exe` 这个木马相关信息的时候，我们偶然看到 `Joe Abrams` 的一篇文章，其中他分析了该木马某个变种的几个代码段。`Joe` 指出在该木马可执行文件中嵌入的许多加密字符串而且叙述了一个解密方法，这些字符串是用一个基于异或(`XOR`)的算法加密的\[`JA1`\]。虽然在我们这个木马版本的可执行文件中没有发现该文提到的大部份字符串，但是 `Joe` 解密的字符串中有一个文本值明显地是“`gus.ini`”。此外有个解密后值为“`joeblow`”，这间接说明 `Joe` 的木马版本与本文‘背景信息’中施密特报告提到的那个相似。

为了弄清解密算法，以便可以解密在我们的 `srvcp.exe` 拷贝中嵌入的字符串，我们把这个可执行文件装入 `IDA Pro` 反汇编器。(读者若想查阅该程序的反汇编代码，可在 [`www.zeltser.com`](http://www.zeltser.com/) 上查看我们反汇编这个程序结果的 `Adobe Acrobat` 打印输出)。遵循 `Joe` 的分析，我们搜索字符串“`nhl*pwf`”，据他说这是“`gus.ini`”的加密值。这引领我们来到一个代码段，见下面图 3-12 中所示。在这个段中该程序反复地把一个好象已加密的字符串推入栈，而且调用相同的例程，这暗示“`sub_4012C6`”可能是一个解密例程。`Joe` 也描述了这个过程，尽管他的数据偏移量与我们的不相符，或许这是由于 `srvcp.exe` 可执行文件细微的版本差异。

#### Probable Calls to String Decryption Routine

```
.text:0040141D   push   offset aNhlPwf   ; "nhl*pwf"
.text:00401422   call   sub_4012C6
.text:00401427   push   offset aAhkl     ; "|ahkli"
.text:0040142C   call   sub_4012C6
.text:00401431   push   offset aWtwgr    ; "wtwgr"
.text:00401436   call   sub_4012C6
.text:0040143B   push   offset aCdkk     ; "|cdkk"
.text:00401440   call   sub_4012C6
.text:00401445   push   offset aMfqece  ; "mfqEce"
.text:0040144A   call   sub_4012C6
```

图 3-12

遵循 `Joe` 对解密过程的分析并在 `IDA Pro` 中查看它的汇编码，我们可以创建一个基于 `Perl` 的例程来模仿该木马代码的相应部分。汇编例程在 `IDA Pro` 中被标示为“`sub_4012C6`”，而且在偏移量“`text:004012C6`”处开始。下面图 3-13 给出我们的解密嵌入字符串 `Perl` 例程。该例程取得加密字符串长度，然后从加密字符串右边开始倒着迭代异或每个字符的原始值和它的位置值。为确保我们的例程实现适当地工作，其方法之一是解密在我们和 `Joe` 的可执行文件版本中都出现的“`nhl* pwf`”字符串，而且确保结果值是“`gus.ini`”。我们的 `Perl` 例程使用 “`&decryptEmbeddedString($encryptedString)`” 形式调用并且返回解密后的字符串。

#### Decrypting Embedded Strings

```perl
sub decryptEmbeddedString {
my($encrypted) = @_;
my(@encrypted) = split(//, $encrypted);
my($length) = $#encrypted;
my($plain) = "";
my($counter);
for ($counter=0; $counter<=$length; $counter++) {
$plain .= chr(ord($encrypted[$length-$counter]) ^ ($counter+1));
}
return($plain);
}
```

图 3-13

要获得嵌入字符串的明文译文，其方法之一是调用我们 `Perl` 例程处理每一个需要经过该程序“`sub_4012C6`”例程解密的字符串。此外，为了确保我们没有遗漏任何加密字符串，我们剖析了使用基于 `UNIX` 字符串程序的 `srvcp.exe`，而且使用一个 `Perl` 脚本自动尝试解密每个字符串。然后我们查看生成的字符串列表、手工挑选出看起来象是英语的那些。结合运用两种方法得到的解密字符串结果见下面图 3-14。

#### Embedded Strings Decrypted

| Encrypted Value | Decrypted Value |
| :--- | :--- |
| `nhl*pwf` | `gus.ini` |
| `\|ahkl` | `mikey` |
| `Wtwgr` | `Setpr` |
| `\|cdkk` | `Jiggy` |
| `mfqEce` | `daFuck` |
| `~h\`PmfqEce` | `daFuckWhat` |
| `v}~y{\*%mj&qldkg` | `fight me, pussy` |
| `og&teh\*\`ph` | `irc.mcs.ne` |
| `O_ATU@VDE@` | `AGGRESSIVE` |
| `5|3u1v/k-h+i)j'g%JLQH` | `ISON a b c d e f g h` |

图 3-14
