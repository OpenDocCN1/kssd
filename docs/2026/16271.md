
# [翻译]基于jemalloc的Android漏洞利用技巧----CENSUS

前几天读到了CENSUS基于Android的`jemalloc`利用的一些手法，其中详细讲解了`jemalloc`的内存分配与释放机制，以及在漏洞利用中的帮助。特意翻译，希望对大家有所帮助。

翻译帖先发于安全客，传送门[d6fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3%4z5o6k6Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@c85K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3%4z5o6k6Q4x3X3g2Z5N6r3#2D9)

当然也有Word原稿，我是用MD写的然后贴到Word上的，想做笔记的可以用这个。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [Jemalloc Exploit Tricks.docx](javascript:void(0);)

（3.81MB，65次下载）
