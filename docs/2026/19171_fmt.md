
# [翻译] 利用 Hostapd-WPE 渗透企业无线网络

最近，我们致力于企业无线网络的渗透式入侵，最终目的是获取用户终端的证书，提高权限来获取管理员账号。在这个案例中我的客户是基于 WPA2 企业网络中拥有 PEAP 认证

早期时候，我们通常需要特殊版本的 Kali-- 带有 WPE (Wireless Pwnage Edition) 补丁的 freeradius 认证服务器和还有 hostapd。经历几个月后，现在的 Kali rolling 获得了 `hostapd-wpe` 作为基本包。因此你不必单独地去下载 freeradius 服务器。攻击的旧方法可以点这里。

现在开始看看当前配置，从硬件方面，我们用的是 TP-LINK TL-WN722N 无线网卡和 Kali rolling 2016.2.

第一步是先升级 Kali 的版本，通过运行下面的命令，更新好后安装 `hostapd-wpe`

![](https://bbs.kanxue.com/upload/attach/201703/740658_u51lbsziv45j6it.png)

![](https://bbs.kanxue.com/upload/attach/201703/740658_g7vs4ac9xgc3csa.png)

在 Kali 运行下将 TP-link 无线网卡插到机器上。同时用 `iwconfig` 命令检测 Kali 是否检测到接口。然后，运行 `aircrack-ng suite check kill`，当接口带有 `aircrack-ng` 套件时将会检查和结束进程

![](https://bbs.kanxue.com/upload/attach/201703/740658_rciyzn3huznjudj.png)

然后，在输入 `/etc/hostapd-wpe/hostapd-wpe.conf` 来配置 AP 性能。例如更新 AP 名字，这样会更加吸引受害者。然后利用用 `hostapd-wpe` 和分别配置好的可以捕获用户验证文件的散列表到我们的伪 AP 中的文件。

![](http://bbs.pediy.com/upload/attach/201703/740658_nz5prk3t3edw0v9.png)

一旦受害机链接上攻击者广播的 AP，用户的请求和回应都会在 `hostapd-wpe` 的屏幕上显示

![](http://bbs.pediy.com/upload/attach/201703/740658_wixd10tfomme9gu.png)

但是用以上的步骤可以捕获比较低级的 WiFi 安全意识受害者的散列表。因为当受害者链接上我们广播的 AP，他总是会被询问是否信任伪证书；"Example Server Certificate" 会引起一些人的注意。

![](https://bbs.kanxue.com/upload/attach/201703/740658_fw44b2aa9qq7f0t.png)

为了匹配真的证书去伪造证书的详细资料也许会很痛苦。因此，可以用 python 脚本来辅助我们，通过命名 `apd_launchpad` 是一个方便的方法来创建 `hostapd-wpe` 配置文件和关联的伪造证书

所以在 `hostapd-wpe` cert 路径下下载 python 脚本。

![](https://bbs.kanxue.com/upload/attach/201703/740658_u4ikdzxwx6tjulh.png)

我们可以用下列提到的最少的标志来生成匹配我们访客证书，也可以生成配置文件

![](http://bbs.pediy.com/upload/attach/201703/740658_wi8aj2jcjulws8t.png)

配置文件和应用保存在 `DEMOBANK` 文件夹。

![](https://bbs.kanxue.com/upload/attach/201703/740658_3w2bnxhb30zjsg1.png)

用 `hostapd-wpe` 运行单独生成的配置文件 (`DEMOBANK.conf`)

![](http://bbs.pediy.com/upload/attach/201703/740658_g7fpiuh2iyltq38.png)

现在受害者将会看到和真的证书相一样的证书（减小怀疑）

![](http://bbs.pediy.com/upload/attach/201703/740658_bfk876dyaectn9f.png)

运行一会儿，我们就可以获得受害者的散列表

![](http://bbs.kanxue.com/upload/attach/201703/740658_r1r51ittpc0zfrd.png)

一旦获取请求和回应，用 `asleap` 和密码字典文件破解

```
root@kali:~# zcat rockyou.txt.gz | asleap -C <Challenge> -R <Response> -W -
```

破解的证书可以用来连接真实 AP，来探索逐步加强局域网管理员技术的横向发展。

原文链接：b0dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3x3H3k6o6y4^5M7r3H3H3K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6y4Q4x3V1k6W2L8Y4c8W2M7Y4m8J5K9i4y4W2i4K6u0V1N6$3W2X3K9g2)9J5k6r3S2S2j5$3E0A6L8X3N6Q4x3X3c8%4K9i4c8Z5i4K6u0V1K9r3!0K6N6r3q4H3k6q4)9J5k6s2N6H3k6g2)9J5k6h3S2@1L8h3H3`.

本文由 看雪翻译小组 南极小虾 翻译

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
