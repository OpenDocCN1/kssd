
# `if_changed`作用概括

以下是`$(if_changed)`的使用形式，及其参数的含义：

![](https://bbs.kanxue.com/upload/attach/202304/815036_RNWC3TAYP47MSEN.png)

## 2. 编译外部模块

### 2.1. 涉及`Makefile`内容

![](https://bbs.kanxue.com/upload/attach/202304/815036_872EXH4TNCB52PW.png)

### 2.2. 概要流程

![](https://bbs.kanxue.com/upload/attach/202304/815036_NGMS2KEQ9AMYQG9.png)

### 2.3. 详细流程

![](https://bbs.kanxue.com/upload/attach/202304/815036_J6DJEZBDS9Q2HNY.png)

## 3. make menuconfig

### 3.1. 涉及`Makefile`内容

![](https://bbs.kanxue.com/upload/attach/202304/815036_UWHNP98VVKN4RX7.png)

### 3.2. 概要流程

![](https://bbs.kanxue.com/upload/attach/202304/815036_QTZR72GCE7SFNGM.png)

### 3.3. 详细流程

![](https://bbs.kanxue.com/upload/attach/202304/815036_36USFJFWGTXSJ8P.png)

## 4. Make [all/\_all/modules]

### 4.1. 涉及`Makefile`内容

`make`命令行指定`all`/`\_all`/`modules`目标，或者不指定目标时，是为了生成`vmlinux`文件，而`vmlinux`目标间接依赖`prepare`目标，且`prepare`目标编译规则展开内容比较多，所以以下分开介绍：

- `vmlinux`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_RV62PHBFB2KWA3G.png)

- `prepare`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_V4GHPKJ4FUW6BTU.png)

### 4.2. 概要流程

- `vmlinux`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_E6KN3TCH22YHKV7.png)

- `prepare`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_HEBEBJTTRZYS5UT.png)

### 4.3. 详细流程

- `vmlinux`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_TT8EKFGUX8NA5Z7.png)

- `vmlinuz`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_2AXMN3T66Y57FMG.png)

- `prepare`目标

  ![](https://bbs.kanxue.com/upload/attach/202304/815036_3QH7DB89NUSQH4G.png)

## 5. 参考

linux Kbuild详解系列：[7bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3$3x3U0j5@1x3o6x3@1x3H3`.`.](https://bbs.kanxue.com/elink@6aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3$3x3U0j5@1x3o6x3@1x3H3`.`.)

跟我一起写makefile：[bf8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6Z5K9i4c8W2k6X3W2K6K9o6f1J5x3q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6z5e0j5^5y4U0l9&6](https://bbs.kanxue.com/elink@074K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6Z5K9i4c8W2k6X3W2K6K9o6f1J5x3q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6z5e0j5^5y4U0l9&6)

---

*附件：文中部分图片，根据"执行流程.txt"内容截取，其余已经打包到"图.zip"文件*。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)

[#工具脚本](https://bbs.kanxue.com/forum-41-1-137.htm)

上传的附件：

- [图.zip](javascript:void(0);)

  （5.74MB，60次下载）

- [执行流程.txt](javascript:void(0);)

  （63.03kb，57次下载）
