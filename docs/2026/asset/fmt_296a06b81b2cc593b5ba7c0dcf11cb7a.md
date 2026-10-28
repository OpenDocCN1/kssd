
```
HTTP/1.1 200 OK
Date: Thu, 21 Mar 2013 06:32:36 GMT
Server: Tencent Login Server/2.0.0
P3P: CP="CAO PSA OUR"
Set-Cookie: pt2gguin=o0314468953; EXPIRES=Fri, 02-Jan-2020 00:00:00 GMT; PATH=/; DOMAIN=qq.com;
Set-Cookie: uin=o0314468953; PATH=/; DOMAIN=qq.com;
Set-Cookie: skey=@91lFrIZry; PATH=/; DOMAIN=qq.com;
Set-Cookie: ETK=; PATH=/; DOMAIN=ptlogin2.qq.com;
Set-Cookie: RK=ph5+gHdbUv; EXPIRES=Sun, 19-Mar-2023 06:32:36 GMT; PATH=/; DOMAIN=qq.com;
Set-Cookie: ptuserinfo=46694679; PATH=/; DOMAIN=ptlogin2.qq.com;
Pragma: no-cache
Cache-Control: no-cache; must-revalidate
Connection: Close
Content-Type: application/x-javascript; charset=utf-8

ptuiCB('0','0','c95K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4q4*7M7#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7i4A6G2L8X3g2Q4x3V1k6$3y4g2)9J5c8X3I4G2k6$3W2F1M7%4g2U0j5#2)9J5k6h3S2@1L8h3I4Q4x3@1k6H3j5i4u0S2i4K6y4p5K9i4A6G2L8X3g2Q4x3U0N6Q4x3V1y4Q4x3U0M7I4i4K6t1%4i4K6u0o6i4K6t1%4i4@1f1%4i4K6V1&6i4@1u0n7i4@1f1#2i4@1u0p5i4K6V1#2i4@1f1$3i4K6R3^5i4K6V1H3i4@1f1#2i4K6S2m8i4K6W2r3i4@1g2r3i4@1u0o6i4K6R3I4i4K6t1%4i4K6u0o6 'FiFy');
```

然后去发表说说抓包，这个就很好分析了，改下包模拟提交就可以，发文章也是一样。

不过要注意编码，说说是 `utf-8`，文章是 `gb2312`，不过也可以在 `post` 的内容中对编码做修改。

![](https://bbs.kanxue.com/upload/attach/201303/258380_912apsecik9glcz.png)

源码：[`LoginQQ.rar`](https://bbs.kanxue.com/attach-download-77477.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`login.png`](javascript:void(0);)（16.37kb，7次下载）
- [`LoginQQ.rar`](javascript:void(0);)（31.77kb，843次下载）
- [`ishimima.png`](javascript:void(0);)（40.48kb，53次下载）
- [`2222222.png`](javascript:void(0);)（266.54kb，222次下载）
- [`vercode.png`](javascript:void(0);)（10.53kb，30次下载）
