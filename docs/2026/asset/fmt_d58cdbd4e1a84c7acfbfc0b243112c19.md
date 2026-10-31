
它的功能是将`%n`之前打印出来的字符个数（四字节）写入参数地址处（赋值给一个变量）。在32位程序中需要的这个地址即是32位，在64位中地址需要为64位。

```c
#include "stdafx.h"

int main(int argc, char* argv[])
{
 int a=0x0012ff74;
 int b=2;
 char x='h';
 printf("%10c%n",x,0x0012ff70);
 return 0;
}
```

![图片描述](https://bbs.kanxue.com/upload/attach/202211/967279_CSPNGTCRDG6QHC9.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-12-30 20:07 被kanxue编辑，原因：
