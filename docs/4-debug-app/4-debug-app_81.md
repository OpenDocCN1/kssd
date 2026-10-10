

# FoxMail 3.11

> 原文：[`www.pediy.com/kssd/pediy03/forum505.htm`](https://www.pediy.com/kssd/pediy03/forum505.htm)

/\*\*\*此文仅供研究，如移作他用，作者概不负责\*\*\*/   
  
如果你想编写一个获取对方信箱密码的程序，而对方用的又是 FoxMail，这篇文章也许对你有用！   
（其实这是一篇很无聊的文章，那为什么要写？我也不知道，可能我太无聊了）   
  
程序需要做的步骤是：   
1、找到 FoxMail 的安装目录；   
2、在其下查找指定的帐户目录；   
3、然后获得该帐户的信箱地址   
4、获取信箱密码的密文；   
5、最后再将其转换为明文   
  
  
在正式开始以前，先假设你在 FoxMail 中有且仅有一个帐户：abcde，它的 EMail 地址是 abcde@163.net，这个 EMail 信箱的   
密码是 12345，并且你的 FoxMail 的安装目录是 D:\\FOXMAIL。   
  
  
1、查找 FoxMail 所在路径   
  
如果对方是用 FoxMail 来收信的，那么它的默认邮件程序应该就是 FoxMail，则它的 FoxMail 所在地址就记录在   
HKEY\_LOCAL\_MACHINE\\Software\\CLASSES\\mailto\\shell\\open\\command 中，当然，你也可以在 HKEY\_LOCAL\_MACHINE\\Software\\Aerofox\\FoxMail   
下的“Executable”中获得   
  
  
2、查找帐户路径：通过 accounts.cfg（帐户配置文件）   
  
这只是为了写程序的方便，否则你可以用资源管理器手工查找该路径。   
在这个目录下你会发现这个文件：accounts.cfg，用 HEdit 编辑这个文件，   
在偏移 0x848(2120)处的值是 05 00 00 00，其实它是一个值是 5 的 DWord(双字),5 的含义就是你的帐户名(abcde)由 5 个字节组成，跳过 0x848   
处的四个字节，再跳过 abcde 这五个字节，你又会看到：15 00 00 00，不用多说，这也是一个双字，它的值为 0x15(21)，其后跟着一串   
长度为 0x15(21)的路径名：D:\\FOXMAIL\\mail\\abcde。如果你建立了更多的帐户，查找其路径的方法以此类推，并且每两个帐户间的间隔是   
0x1c(28)个字节。   
  
  
3、获取信箱地址：通过 Account.stg   
  
前面我们已经找到了 abcde 的帐户目录，在该目录下有一个称为 Account.stg 的文件，该文件十分重要，它记录着用户在帐户->属性菜单中   
的所有内容，当然也包括了我们感兴趣的密码的密文。它的格式如下：   
0x00 到 0x7FF：乱七八糟的东西   
0x800 开始：“属性”中的东西   
  
以下是从 0x800 开始的内容：   
AutoCheck=0   
AutoCheckMinutes=15   
AutoMarkRead=1   
AutoPack=1   
AutoPackSize=20   
AutoQP=1   
AutoReadSeconds=5   
CharSet=US\_ASCII   
CheckOnSend=0   
DUNEntry=   
ESMTPAccount=   
ESMTPPassword=   
HangupDUN=0   
IsESMTP=0   
LabelRow=1   
LabelsCaptions=Label1;;Label2;;Label3;;Label4;;Label5;;Label6;;Label7   
LabelsColors=255;;32768;;16711680;;8388608;;65535;;128;;32896   
LeaveInServer=0   
MailAddress=abcde@163.net   
MailListFont=   
MessageFont=   
MovetoSentbox=1   
NetType=0   
Organization=   
OrigReply=1   
PlaySoundOnNewMsg=0   
POP3Account=abcde   
POP3Host=pop.163.net   
POP3Password=F9AAB899ACA6   
POP3Port=110   
PrintFont=   
ReplyAddr=   
SameWithPop=1   
SMTPHost=smtp.163.net   
SMTPPort=25   
SoundFileOnNew=   
Subject8bit=0   
UseAccNetSet=0   
UseExistDUN=1   
UseMIME=1   
UserName=abcde   
UseWeb=0   
WordWrap=1   
  
我只挑重要的讲，最重要的当然是 MailAddress 和 POP3Password 啦，哇噻，是 POP3Password 耶，这不是 pop3 的密码吗？当然不是啦，密码以   
明文存放………FoxMail 不会这么笨的，它是经过了加密的密文（稍后我会讲的），其它的我就不多讲了，像 POP3Host 是 pop3 的服务器地   
址；POP3Port 是 pop3 的服务器端口；SMTPHost 是发信服务器，而 SMTPPort 是该服务器的端口…………等等，就此一笔带过（讲多了又要被   
扁了），其实上面的并不是属性中所有的内容，如果你细心的话就会发现属性中的“模板”并没有包含在内，对了，再接下去的文件内容   
就是模板的内容，因为这实在是无关痛痒，所以不多讲了。   
  
  
5、将取得的密文转换为明文（即密文解码公式）   
  
终于到了最刺激的部分了，这可是 FoxMail 的安全核心哦！   
在解码之前，请你先记住一个单词：~draGon~（注意要区分大小写）。   
记住了吗？好，现在解码开始：   
  
第一步：   
取得上面这个单词的 ASCII 码：7E 64 72 61 47 6F 6E 7E，记为 a1,a2,a3,a4,a5,a6,a7,a8   
  
第二步：   
将其相加，得 0x357，除以 255，余数是 0x5A， 记为 c0   
  
第三步：   
取密码的密文：F9AAB899ACA6 将它们两两分开，得到：F9 AA B8 99 AC A6，分别记为 b1,b2,b3,b4,b5,b6   
密文的长度为 12，12/2－1＝5，所以密码的长度为 5；   
  
第四步：   
c0=0X5A   
C1=C0⁰XF9=c0^b0=0xA3   
C2=0XAA   
C3=0XB8   
C4=0X99   
C5=0XAC   
  
其中^为异或；0x 代表此数为 16 进制数；   
d1=b2^a1=0xD4   
d2=b3^a2=0xDC   
d3=b4^a3=0xEB   
d4=b5^a4=0xCD   
d5=b6^a5=0xE1   
  
第五步：   
判断 di 是否大于 ci，小于或等于，则 di=di+0xFF   
  
第六步（最后一步）：   
Ei=di-ci，其中 Ei 就是将密文解码后的明文，本例中的密码是：d4h-a3h=31h;dch-aah=32h;ebh-b8h=33h;cdh-99h=34h;e1h-ach=35h   
即‘12345’。   
  
好了，终于写完了，如果还有问题别来问我，这篇文章已经写得很详细了！   
  
有空的话去看看我的网站吧（虽然还没有完工），网址是 HTTP://TSCoffee.MyEtang.Com   
  
                祝各位   
                    元旦快乐   
  
  
                                ---------- 碳烧咖啡   
                                      2001.1.2 晚  4dd466d0e1e0cc123afbe504da2462ca