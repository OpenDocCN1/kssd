
# [原创]CSDN免积分下载方法与原理小析（可制作面积分下载工具）

看到论坛上有人问csdn免积分下载器，实现原理，我看到的有两种。

1. 注册新用户，送积分方式，用机器人自动注册用户名，例如，鬼哥免积分下载器。送积分后下载，这样对csdn资源是一种浪费。

2. 利用csdn 接口未验证的bug进行下载。

例如：1. 首先到下载区搜索你想要的资源 记下它的id

例如：`ca5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8Y4A6J5k6U0p5K6z5e0p5K6z5q4)9J5c8U0f1J5z5e0x3@1z5e0y4Q4x3U0k6F1j5Y4y4H3i4K6y4n7y4e0t1&6x3K6b7&6x3#2!0q4y4q4!0n7z5q4!0n7b7h3W2V1`

2. 请求以下地址：`8f8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3M7X3g2K6N6q4)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8X3N6W2N6s2y4G2N6i4u0U0k6h3W2F1k6X3!0Q4x3V1j5#2x3U0V1^5x3e0R3#2i4K6t1$3L8X3u0K6M7q4)9K6b7U0f1J5z5e0x3@1z5e0y4Q4c8e0c8Q4b7U0S2Q4b7V1q4Q4c8e0g2Q4b7V1g2Q4z5e0N6Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0N6Q4z5f1q4Q4z5o6c8A6k6q4)9J5k6g2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4@1f1#2i4@1u0q4i4K6V1%4i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1^5i4@1u0r3i4K6W2q4i4@1f1$3i4K6S2q4i4@1p5#2i4K6t1$3L8X3u0K6M7q4)9K6b7R3`.

```
{"id":"5293493","title":"1500\u4e2a\u524d\u7aef\u5f00\u53d1\u5e38\u7528JavaScript\u7279\u6548","tag":["javascript","\u7279\u6548","\u5b9e\u4f8b","\u524d\u7aef"],"categoryid":"16007","sourcescore":"0","username":"zrf139138","sourcesize":"3489911","pubdate":"1366735880","updatetime":"2013-04-29 10:50:08","description":"1500\u4e2a\u524d\u7aef\u5f00\u53d1\u5e38\u7528JavaScript\u7279\u6548,\u5b9e\u4f8b\u8d85\u591a\uff0c\u7279\u6548\u5b9e\u7528\uff0c\u6b22\u8fce\u4e0b\u8f7d\u5b66\u4e60","isrecommended":"11","status":"2","filetype":"rar","originfile":"1500\u4e2a\u524d\u7aef\u5f00\u53d1\u5e38\u7528JavaScript\u7279\u6548.rar","fileaddr":"\/20130424\/295cc513f88b58de697799868e6a2904.rar","otherlink":"","fileaddr_prefix":"\/down10","md5":"cd58958bcefca6347db66817cd991a78","isrepeat":"0","download_num":"2499","sourcetype":"1","comment_num":"113","fav_num":"57","score_sum":"988","score_num":"231","operator":"zy3905324","operate_time":"1366768216","link":"\/detail\/zrf139138\/5293493","url":"http:\/\/dldx.csdn.net\/fd.php?i=676152934932432&"}
```

分析后：`a3bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8D9k6s2S2Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3k6X3c8Q4x3X3g2H3K9s2m8Q4x3@1k6A6i4K6y4p5y4U0M7$3x3e0f1J5z5e0x3@1z5e0x3J5y4o6x3J5i4K6t1$3i4K6t1$3L8X3u0K6M7q4)9K6b7W2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4g2)9&6c8q4)9^5x3q4!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4#2!0m8b7#2!0m8b7#2!0q4y4q4!0n7z5q4)9^5z5g2!0q4y4#2!0m8y4#2)9^5c8q4!0q4y4W2)9&6y4W2!0n7z5g2!0q4y4W2!0n7x3#2)9&6y4g2!0q4z5q4!0n7c8W2)9&6z5q4!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4#2!0m8x3q4)9&6y4q4!0q4y4#2!0m8z5g2!0n7y4W2!0q4y4W2)9&6b7g2)9^5x3W2!0q4y4W2)9&6y4#2!0n7y4W2!0q4y4q4!0n7z5q4)9^5c8q4!0q4y4g2)9^5y4g2!0m8b7#2!0q4y4g2!0n7b7#2)9^5x3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4q4!0n7b7#2)9&6b7g2!0q4y4q4!0n7b7W2)9^5b7W2!0q4y4#2!0n7b7W2)9^5c8q4!0q4x3#2)9^5x3q4)9^5x3R3`.

![](https://bbs.kanxue.com/upload/attach/201304/378384_5bni1zo74pwwx2t.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [2.jpg](javascript:void(0);)

  （26.16kb，432次下载）
