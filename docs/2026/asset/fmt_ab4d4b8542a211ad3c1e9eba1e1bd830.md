
同时在github上找到类似的三维转换二维的代码03aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8M7o6M7%4y4o6u0Q4x3V1k6b7g2f1u0s2f1r3q4@1j5$3S2W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1j5J5y4K6j5J5z5r3t1K6k6X3j5#2x3$3f1H3y4K6j5$3k6e0M7H3z5o6c8S2z5o6u0T1x3U0V1K6j5X3t1#2z5r3g2W2y4X3x3@1x3U0k6T1i4K6u0r3c8r3q4W2L8h3!0F1i4K6u0r3K9X3&6A6i4K6u0r3f1$3g2J5N6X3g2J5i4K6u0r3f1%4c8J5N6h3y4@1M7@1y4G2L8h3#2G2L8W2)9J5k6h3R3`.

![image-20220425034640328](https://bbs.kanxue.com/upload/attach/202204/862028_GUDPXX8VRXJUFMC.png)

不过这里计算并没有fov，其他倒是差不多

![image-20220425034732985](https://bbs.kanxue.com/upload/attach/202204/862028_22XUNUUWH4BKDHP.png)

#### 将计算后的屏幕坐标输出到`/sdcard/1A.txt`文件中

![image-20220425004009517](https://bbs.kanxue.com/upload/attach/202204/862028_RW78TN5U5TPZ8JH.png)

### 读取数据方式

远程读写的操作都集中在这个`getprocessvalue`函数（自命名)中

![image-20220425114453625](https://bbs.kanxue.com/upload/attach/202204/862028_BSANSXRM6A6E4AS.png)

根据不同情况选择用`syscall`或者`process_vm_readv`来进行远程读写

# 总结

第一次参与这个比赛，还是挺多收获的，感谢主办方和出题人了，也希望看wp的师傅们也能有所收获2333。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#NDK分析](https://bbs.kanxue.com/forum-161-1-119.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
