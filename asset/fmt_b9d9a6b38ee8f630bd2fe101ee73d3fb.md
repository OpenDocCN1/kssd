
# 简单方法重设 iphone 的 unix root password

> 原文：[`www.pediy.com/kssd/pediy10/92497.html`](https://www.pediy.com/kssd/pediy10/92497.html)

有些兄弟喜欢使用 `SSH` 进入 `iphone`，在彷似 `linux` 的空间里面玩
`iphone` 的初始 `root` 密码是 `alpine`，相信很多朋友都会在第一次 `login` 时修改了
如果忘记了密码，不用怕，不需要再 `jailbreak` 系统
方法如下：

`iphone` 的户口密码储存在 `/private/etc/master.password`
只需要在 `PC` 上安装一个 `iphone browser` 工具，就可以编辑它

1. 接上 `USB` 线，在计算机打开 `iphone` 的目录
2. 把 `master.password` 下载到计算机
3. 用编辑工具打开它。要注意的是它的分行字符 `CR LF` 跟 `windows` 的有分别，所以不要用 `notepad` 打开它。如果用 `wordpad` 打开，你是可以顺利阅读它的，但是，使用 `wordpad` 储存的 `master.password` 会变成 `windows` 的分行，放回 `iphone` 使用会登入错误，所以也是不建议使用

方法有很多，可以把这个掉到你家中的 `linux` 机上，用 `vi` 去编辑它，这是最简单的方法。如果没有 `linux` 机，你也可以使用 `win` 版的 `vim` `gvim72.zip`，使用方式跟 `vi` 相同。你也可以把这个当成一般文字文件，掉回 `iphone` 里的 `/tmp`，然后用你平日的 `ssh iphone` 去 `vi` 它。

打开档案后，你会看见像这样的内容

## User Database

```
# User Database

# This file is the authoritative user database.

nobody:*: -2:-2::0:0:Unprivileged User:/var/empty:/usr/bin/false
root:/smx7MYTQIi2M:0:0::0:0:System Administrator:/var/root:/bin/sh
mobile:/smx7MYTQIi2M:501:501::0:0:Mobile User:/var/mobile:/bin/sh
daemon:*:1:1::0:0:System Services:/var/root:/usr/bin/false
_securityd:*:64:64::0:0:securityd:/var/empty:/usr/bin/false
_mdnsresponder:*:65:65::0:0:mDNSResponder:/var/empty:/usr/bin/false
_sshd:*:75:75::0:0:sshd Privilege separation:/var/empty:/usr/bin/false
_unknown:*:99:99::0:0:Unknown User:/var/empty:/usr/bin/false
```

`iphone` 的系统很简单，它把密码 `hash` 放进了这里。你只需要把你的 `root` 那个密码 `hash` 修改回上面这样 `smx7MYTQIi2M`，那么密码便重设回 `alpine`。

修改完成，用 `iphone browser` 把档案放回 `/private/etc/` 去，可以登入 `root` 了

当然，上面的方法是介绍给大家自救用的，不要拿别人的 `iphone` 来玩

![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")
