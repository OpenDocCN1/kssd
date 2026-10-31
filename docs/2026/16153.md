
# [原创] Android 协议分析（三）——HFS 文件服务器 HTTPS 访问的实现

## 一、Stunnel 介绍

HFS 是一种轻量级的文件服务管理器，能够很好地模拟 HTTP 访问请求，但是不支持 HTTPS。查阅资料后，我发现可以使用 Stunnel + HFS 实现 HTTPS 访问。

## 二、HTTPS 访问实现

### （1）Stunnel 的安装配置

[官方网址](<mark class="encrypted">b12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6s2g2F1L8X3g2D9i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0W2K9s2c8E0L8q4!0q4c8W2!0n7b7#2)9^5z5b7</mark>)

我们下载 Stunnel：

![avatar](upload/attach/202107/905443_MKSPTQB763PX6G8.png)

Stunnel 使用参考链接：<mark class="encrypted">5d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6V1%4x3o6t1&6y4o6N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3%4y4K6p5@1y4e0t1H3</mark>

我们安装 Stunnel 过程中会涉及一些信息填写，依次填写即可。

![avatar](upload/attach/202107/905443_N8Z88KW23SSHHTK.png)

安装参考链接：<mark class="encrypted">5d4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2K9X3g2@1N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3N6r3!0H3K9h3y4Q4x3V1x3K6x3o6R3K6i4K6u0W2L8i4y4Y4x3e0l9I4y4e0V1%4y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4E0M7$3M7I4x3o6p5#2z5e0M7$3</mark>

我们这里重新配置了 `stunnel.pem`。

然后我们需要进入配置文件，配置 HTTP 信息。软件的样例文档也提供了参考。

![avatar](upload/attach/202107/905443_9ATFTH56C86RQ24.png)

![avatar](upload/attach/202107/905443_RE5AVVMKRXUYN6Y.png)

我们将分号去除即可使用，最终的配置信息如下：

![avatar](upload/attach/202107/905443_5TTQFM9GZR7WTDE.png)

然后我们 `reload configure`，重启配置即可实现。

### （2）问题解决

我们发现报错：

![avatar](upload/attach/202107/905443_FW3WT4CFC97A528.png)

显示端口被监听了，我们查询端口：

![avatar](upload/attach/202107/905443_388ANH4X2Y9KPRN.png)

发现 443 端口号被 2540 这个进程监听。我们进入任务管理器，发现是虚拟机进程，将其结束后再重启配置。

![avatar](upload/attach/202107/905443_65G3ZFP4NK5VPPV.png)

![avatar](upload/attach/202107/905443_YYSRF9G6JS29CWG.png)

发现可以成功访问。

我们还可以配置证书，参考链接：<mark class="encrypted">918K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6A6M7r3q4U0K9#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6R3#2z5o6M7I4x3K6t1</mark>

## 三、参考链接

[1] <mark class="encrypted">57bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6s2g2F1L8X3g2D9i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0W2K9s2c8E0L8l9</mark>

[2] <mark class="encrypted">461K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6V1%4x3o6t1&6y4o6N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3%4y4K6p5@1y4e0t1H3</mark>

[3] <mark class="encrypted">4ceK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2K9X3g2@1N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3N6r3!0H3K9h3y4Q4x3V1x3K6x3o6R3K6i4K6u0W2L8i4y4Y4x3e0l9I4y4e0V1%4y4W2)9J5k6h3S2@1L8h3I4Q4x3U0y4E0M7$3M7I4x3o6p5#2z5e0M7$3</mark>

[4] <mark class="encrypted">1f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6A6M7r3q4U0K9#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6R3#2z5o6M7I4x3K6t1</mark>

[[招生] 科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-7-14 17:25

被随风而行aa编辑

，原因：

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)
