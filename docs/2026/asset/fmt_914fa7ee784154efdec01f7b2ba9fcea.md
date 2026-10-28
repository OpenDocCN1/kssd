
![](https://bbs.kanxue.com/upload/attach/202309/830009_8STTNKJGUU27EXY.png)

5. 然后可以设置程序的图标和安装程序输出路径，之后全部点击下一步即可：

![](https://bbs.kanxue.com/upload/attach/202309/830009_4QS3HPGNYSFY9YH.png)

6. 然后就可以在输出路径看到生成的安装程序：

![](https://bbs.kanxue.com/upload/attach/202309/830009_DW2V3R2UBWZS9TB.png)

7. 点击运行就是熟悉的程序安装界面了，按需进行选择后即可使用，同时需要以管理员身份运行：

![](https://bbs.kanxue.com/upload/attach/202309/830009_W36A4RBHRX4PZST.png)

> 安装程序也可以在[GitHub](https://bbs.kanxue.com/elink@828K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9r3q4F1k6$3A6A6N6$3g2A6x3e0t1J5x3g2)9J5c8X3u0D9L8$3N6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3N6h3&6D9L8$3y4C8k6X3W2D9k6b7`.`.)中找到，目前只在`win10`和`win11`进行了测试。

## 总结

本文讲解了如何实现一个解除文件占用的小程序，不过还存在很多不完善的地方：

- 注册表添加项无法自定义，同时未提供删除注册表的操作
- 不是列出所有占用项让用户选择进行解锁
- 只测试了`win10`和`win11`环境下的运行
- 未实现批量解除文件占用的功能
- ...

不过相信各位参考本文的思路，实现以上的功能也是轻而易举，欢迎一起交流讨论~~~

所以，我还是选择使用火绒。

## 参考文献

在实现这个小工具的过程中，踩了很多坑，非常感谢以下文章所提供的解决思路：

- [用 Windows Native API 枚举所有句柄及查找文件句柄对应文件名的方法](https://bbs.kanxue.com/elink@316K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3f1r3S2A6L8r3W2H3i4K6u0V1g2r3g2D9L8q4)9J5k6q4c8J5N6i4c8Z5i4K6u0r3M7q4)9J5c8U0j5&6x3K6t1&6y4U0k6Q4x3X3g2Z5N6r3#2D9)
- [C++解除文件占用](https://bbs.kanxue.com/elink@5a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8S2M7$3N6C8i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5J5y4K6x3$3x3K6l9J5x3b7`.`.)
- [C++程序环境变量添加以及右键菜单](https://bbs.kanxue.com/elink@d19K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4Z5K6y4o6f1@1x3K6j5K6x3K6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4y4o6M7#2y4K6j5K6x3R3`.`.)
- [跨进程关闭句柄的方式](https://bbs.kanxue.com/thread-219110.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-9-18 11:07 被庄周の蝴蝶编辑，原因：格式调整

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
