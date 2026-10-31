
# [原创]钉钉如何下载管理员禁止的直播回放（抓包分析）

目前钉钉的直播回放只能保存12个月，但一些重要直播网课可能是一辈子都需要回看的，此时管理员还设置了不可下载的权限。

![图片描述](upload/attach/202208/940656\_Y6BHFM2RFA8BGS2.png)

首先电脑上打开`fiddler`，点击直播回放，看个2、3秒，然后关闭，此时可以看到抓包结果，有一些`mp4`视频，看了一下，这些视频都很短，只有几十秒的片段。其他的大部分抓包结果都是`recode.taobao.com`这个奇怪的URL。

![图片描述](upload/attach/202208/940656\_ENMK74CKJQSZQ37.png)

所以暂且认为钉钉的对抗策略是：视频看到哪里，就给你下发哪里的`mp4`文件，直到你看完整个视频，才能抓取到所有的`mp4`文件……那这和录屏有啥区别……

所以呢，先入为主，钉钉有没有可能下发一个汇总文件，其中就汇总了所有的片段视频，拼接到一起，就可以得到一个完整的视频。

![图片描述](upload/attach/202208/940656\_32K4UQNA3HN2XZB.png)

果不其然，万花丛中，我终于找到了你，在这个URL里面，有一个`args_url`字段，经过URL解密，得到了一个网址链接。

![图片描述](upload/attach/202208/940656\_JSW5U33EUSBM6U3.png)

当我访问该链接时，会下载一个文件，该文件内容如下，包含了所有的片段视频链接后缀。

![图片描述](upload/attach/202208/940656\_33DSJ7CZQGEESYC.png)

那么前缀是什么呢，不用担心，在刚才的`fiddler`抓包结果里就可以看到`mp4`视频的前缀是`<mark class="encrypted">80cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6r3I4A6N6X3W2F1k6#2)9J5k6r3u0B7i4K6u0W2k6r3W2F1k6%4c8S2L8r3E0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9i4k6W2i4K6g2X3K9s2m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5p5u0Q4b7V1y4Q4c8e0k6Q4z5p5g2Q4b7e0g2Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0g2Q4z5p5c8Q4b7U0y4Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0y4Q4z5o6m8Q4z5o6t1`。</mark>`

不过这么一个一个网址拼接，最后下载完了，还需要把视频进行拼接，十分麻烦，所以干脆自己写了个`py`脚本实现这一套自动化的流程。

呀，脚本太大，上传不了，不过原理都摆在这了，大家就自己动手，丰衣足食吧（tip：拼接视频可以使用`ffmpeg.exe`来实现）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)
