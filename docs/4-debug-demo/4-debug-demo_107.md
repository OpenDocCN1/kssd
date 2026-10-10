

# 超级个人软件 V2.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue377.htm`](https://www.pediy.com/kssd/pediy04/kanxue377.htm)

30 秒破解 超级个人软件 V2.3--简单介绍 Regmon 工具使用方法   
  
软件名称：超级个人软件 V2.3   
软件简介：本软件是为了方便自已管理一些资料而制作的，后来想放到网上供大家使用，也方便大家管理一些零碎的东西吧。   
未注册版限制：软件只能使用到 2002 年 2 月 14 日   
下载地址：http://www.shareware.net.cn/download.asp?id={B03E7D69-5FB7-43D1-8A45-6B11E843668C}   
破解工具：Regmon   
破解人：飞鹰\[BCG\]   
E-mail：flithawk@263.net   
网址：http://flithawk.longcity.net   
  
用 Regmon 跟踪软件启动及注册发现，如下的地方：   
  
一、软件启动时：   
  
8.55695040    Super    OpenKey    HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per    SUCCESS    hKey: 0xC29ABFD0       
  
8.55698240    Super    QueryValueEx    HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per\\Modem    NOTFOUND           
                                                          ^^^^^ ^^^^^^^^   
大家注意到了吗？上面 Regmon 跟踪后告诉我们说：Modem 这个键值没有找到！   
  
二、调用软件注册对话框：   
  
12.25274720    Super    OpenKey    HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per    SUCCESS    hKey: 0xC29ABFD0       
  
12.25277600    Super    QueryValueEx    HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per\\Modem    NOTFOUND           
                                                          ^^^^^ ^^^^^^^^   
大家注意到了吗？上面 Regmon 跟踪后也告诉我们说：Modem 这个键值没有找到！   
  
这时，我就猜想 Modem 这个键值会不会就是判断软件是否已注册的标志呢？我在注册表的 OpenKey    HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per 项目中添加了一个 Modem 键值，再启动软件后看一看软件注册对话框中已经显示“感谢您对共享软件的支持！”的字样。之后我又把系统的时间往后调了一年，即 2003 年 1 月 30 日，再启动软件测试，软件完全可以使用。   
  
下面让我们来写一个该软件的注册文件如何？很简单地：   
  
REGEDIT4   
  
\[HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\System\\per\]   
"Modem"="飞鹰\[BCG\]"   
  
在 Windows 记事本 中输入完上面的语句后，保存为后缀是 .reg 的文件，这样注册文件就制作好了。   
  
至此，该软件的破解完成！这几天我都在研究破解 Language Localizator 5.04.02.048，一个功能强大的汉化 DEPHI 软件的工具，但始终没有完全破掉它的功能限制，哪位大哥能帮助看一看，该软件的下载地址是：http://www.localizator.com/download/localizator/loc5pkg.exe。我这几天被它折腾的真是够惨的，如果它有 超级个人软件 V2.3 这么温柔就好了！   
  
                      Crack by 飞鹰\[BCG\]  flithawk@263.net  2002.1.30   
                      欢迎光临汉化新世纪: http://www.hanzify.org  4dd466d0e1e0cc123afbe504da2462ca