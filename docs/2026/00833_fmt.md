
# [旧帖][原创]OllyDBG下测试Intel IA-32(x86)EFLAG的4个主要标志位0.00雪花

## OllyDBG下测试Intel IA-32(x86)EFLAG的4个主要标志位

### 一，首先是C标志位，即进位标志
当无符号数超过它的最大表示范围（`x86` 架构中通常是32位）时，该位设置为1，并且运算结果会从最大值变为最小值，下面以`OllyDBG`演示：

![](https://bbs.kanxue.com/upload/attach/201502/533967_00a8bc33f812fb9a.jpg)

![](https://bbs.kanxue.com/upload/attach/201502/533967_74b899c976876369.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
