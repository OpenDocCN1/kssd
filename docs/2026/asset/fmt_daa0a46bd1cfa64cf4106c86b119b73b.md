
参考 [2b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3W2@1k6i4u0G2L8h3q4F1i4K6u0r3f1r3I4S2P5f1W2F1N6r3g2Y4M7X3W2@1P5f1k6A6P5l9`.`.](https://bbs.kanxue.com/elink@b5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3W2@1k6i4u0G2L8h3q4F1i4K6u0r3f1r3I4S2P5f1W2F1N6r3g2Y4M7X3W2@1P5f1k6A6P5l9`.`.) 大佬写的插件进行学习，对大佬拙劣的模仿。

### 准备证书

将证书文件移动`/data/local/tmp`下，并提供读权限；

![](https://bbs.kanxue.com/upload/attach/202312/790193_EPB42WX6BR5J439.webp)

### 注入目标进程

代码入口在`cpp/main.cpp`

![](https://bbs.kanxue.com/upload/attach/202312/790193_MHDSDN8BFB8C2DQ.webp)

![](https://bbs.kanxue.com/upload/attach/202312/790193_ZF3B37YXJG8BAAK.webp)

### 替换证书链

![](https://bbs.kanxue.com/upload/attach/202312/790193_DT54ADWWRPJ6M4Z.webp)

`CustomKeyStoreSpi.java`

![](https://bbs.kanxue.com/upload/attach/202312/790193_UMXS3Y4BQ4MYPU5.webp)

替换后效果

## 结语

我认为设备异常判定，即使各大厂商去自研解决方案，也应该去学习了解系统层面提供的安全检测手段 和 官方在用的判定方案。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2023-12-8 10:41 被pareto编辑 ，原因： 调整图片

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)
