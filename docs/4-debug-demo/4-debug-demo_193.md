

# OICQ 聊天伴侣 2.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50123.htm`](https://www.pediy.com/kssd/pediy05/pediy50123.htm)

标 题:OICQ 聊天伴侣 2.0 版爆破 Nag 窗口手记   
发信人:laoqian   
  
软件名称：    OICQ 聊天伴侣 2.0   
平台：    Win9x,Win2000   
简述：    配合腾讯 QQ 使用，可以查 IP，查地理位置，发炸弹，贴图……   
作者：    周杰   
邮件：    zhoujiehg@263.net   
网址：    http://www.jackysoft.com/cn/qqbl   
功能：   
  
1、查看好友的 IP 地址，分析好友所在的地理位置；   
2、内部存储大量聊天贴图及聊天用语，可供复制使用；   
3、内置消息炸弹，允许多消息循环自动发送，允许设置消息发送间隔时间；   
4、如果你无意点了“发送／接收消息”上的广告，会弹出广告网页。本软件屏蔽了该广告，无论你怎么点，都不会出现广告页面   
  
    若您刚从互联网上下载得到本软件，本软件属于未注册的版本。该版本会不定期地跳出一个“提示注册”窗口。您可以通过向作者支付注册费拿到一个注册码。使用该注册码您可以使本软件从未注册版本变为已注册版本。已注册版本不会跳出让您心烦的“提示注册”窗口。这样您就可以放心使用。   
  
破解过程：   
  工具:eXeScope 6.10 汉化版,Ultra Edit 8.0   
（1）用 Fi 查出它用 Upx 压缩，则用 Upx 脱壳成功。   
（2）作者号称“OICQ 聊天伴侣 2.0 版 - 2001.12.30”改用 RSA 加密算法计算注册码，杜绝了黑客制作注册机的可能性。那我就别费劲了，想别的辙。   
    用 eXeScope 6.10 打开 qqbl.exe（当然是脱壳的），点“资源”-->“RC 数据”-->来到“TForm2”键，往下看：   
.......   
  
object Timer1: TTimer   
    Interval = 1   
    OnTimer = Timer1Timer   
    Left = 271   
    Top = 92   
  end   
  object Timer2: TTimer   
    Interval = 60000      <====这里可疑，每隔 60000 毫秒（1 分钟），跳出一个“提示注册”窗口，改为 0      ！！！   
    OnTimer = Timer2Timer   
    Left = 272   
    Top = 124   
  end   
  object Timer3: TTimer   
    Interval = 1   
    OnTimer = Timer3Timer   
    Left = 271   
    Top = 156   
  end   
  .....   
存盘退出，运行 OICQ 聊天伴侣，不再跳出让您心烦的“提示注册”窗口。收工。  4dd466d0e1e0cc123afbe504da2462ca