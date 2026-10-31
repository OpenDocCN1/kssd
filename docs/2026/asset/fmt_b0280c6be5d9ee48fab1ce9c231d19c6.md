
# [原创]还原word文档中的图片

## 发文由来

某次CTF流量分析题目触碰的坑，同时也在office恶意文档分析中遇见。

## 场景还原

### 创建测试文档

创建word文档并插入某个图片，保存文档，提前计算图片hash值，为后续比较做准备。

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_R452KGAMA3MUSRR.png)

原始图片MD5: `EC4E8052DE5D1E3F55ECE8E76D56FE34`

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_XJ4ZYTQM2B4XK7P.png)

### 还原图片：

打开测试文档，将文档另存为"**网页格式**",**注意：只能选择图片中所示格式**

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_8955UZHPU5QVSG4.png)

保存后，在保存文件路径下会生成两个文件：一个为htm格式的文件，另一个是网页格式中加载文件的文件夹，原始图片就这其中。

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_BJ4HAG249EZMBKD.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_EKFB5VA3TJUJKFR.png)

文件夹中出现了连个图片且大小不一样，计算HASH:

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_KKP8J6DWPC5XXU2.png)

通过比较，发现图片`image001`即为原始图片

### 原因分析

这是由于将图片插入word过程中，word文档会将原始图片先保存，然后根据word文档情况进行图片的自动编辑或者文档作者的编辑，造成图片大小的变化。

### 局限性

上述方法是在知道原始图片hash的情况下进行对比，存在局限性。

### 新策略

将测试文档后缀名改为`zip`保存，解压该zip文件，进入解压后的文档路径：`\word\media` 将为发现相关原始图片，每次插入的图片都将唯一一份被保存。

![图片描述](https://bbs.kanxue.com/upload/attach/202008/886568_5ABJEGETXJQB9HV.png)

### 结语

学习了一波word文档的一些机制，有兴趣的同学可以试试。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
