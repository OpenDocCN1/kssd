
27eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4S2A6N6W2)9J5k6h3!0J5k6#2)9J5c8Y4m8V1k6W2)9J5c8U0t1I4x3o6y4Q4x3X3f1H3x3U0t1^5x3W2)9J5k6i4m8V1k6R3`.`.

ba4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2j5i4m8H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6r3N6T1i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4y4W2j5$3f1&6z5e0c8V1x3o6p5J5y4W2)9J5c8Y4N6W2j5R3`.`.

a75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8K9g2)9J5k6q4c8Z5K9h3&6C8k6i4u0Q4x3X3c8a6M7r3g2F1i4K6u0r3g2r3g2D9K9h3&6C8i4K6g2X3z5o6t1#2h3q4)9#2k6W2y4p5d9H3`.`.

817K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3g2E0L8$3!0Q4x3X3c8D9j5h3u0Q4x3V1k6G2M7r3g2F1K9r3q4&6M7%4c8S2j5$3D9`.

eb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2k6r3q4E0j5$3q4@1L8r3g2&6i4K6u0r3j5h3c8S2L8h3y4S2N6r3I4W2P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5X3I4G2j5W2)9J5c8X3p5H3y4h3t1&6k6r3y4T1j5X3j5J5x3r3f1%4x3X3q4T1x3K6q4W2y4o6p5$3j5K6t1I4x3r3u0V1x3e0y4V1y4e0j5$3k6e0W2U0z5h3y4Q4x3V1k6V1L8$3y4K6i4K6u0r3b7h3W2J5g2r3q4Y4i4K6u0W2L8h3b7`.

---

### 广播信标追踪

![image-20211021102943540](https://bbs.kanxue.com/./upload/attach/202110/634068_8WWX5K7KF4W63PH.png)

其实上还有很多智能产品采用BLE 广播的协议实现设备发现等功能,例如两家国内销量Top2的可穿戴手环产品,在运行期间会不断广播自身唯一标识,周围任何人都能收到这个信息,还有国内大部分的快传协议也有这种风险,不过需要使用者打开系统自带的快传功能并且部分品牌手机需要保持亮屏状态

危害就是在一个城市部署足够多的蓝牙嗅探节点,进行统一汇总这样可以推算出某个人一天时间之内去了那些地方场所,或者在固定场所部署嗅探节点,例如判断这个人具体的休息工作时间

真实AirTag会定期更新所广播的公钥以及唯一的标识目的就是解决上面这个跟踪的问题,只是在国内似乎并没有人关注到这个风险

---

### 蓝牙耳机实现远程监听

![image-20210924161632753](https://bbs.kanxue.com/./upload/attach/202110/634068_TNB2XEJC5N9UYP6.png)

最开始其实是在我现有的耳机上在挖RCE的洞,然后意外找到Enco Air 链接时候有个缺陷

正常耳机在开机后默认是与已配对的手机进行链接,配对模式需要长按某个按键才会进入,而这个耳机似乎是为了用户体验考虑,开盖后默认就是配对模式,任意人都能连接配对上,关键点就是没有授权的操作比如说按一下物理按键完成配对(比赛项目那个耳机也是处于开盖状态下,相同的条件),这样攻击者就能配对然后调用麦克风实现远程监听效果了

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-10-22 10:30

被mobier编辑，原因：更新素材链接

[#硬件开发](https://bbs.kanxue.com/forum-128-1-166.htm)

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)

[#无线电安全](https://bbs.kanxue.com/forum-128-1-177.htm)
