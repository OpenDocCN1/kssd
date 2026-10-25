
# \_security\_cookie 的问题

> 原文：[`www.pediy.com/kssd/pediy12/126858.html`](https://www.pediy.com/kssd/pediy12/126858.html)

代码:

```
#include <stdlib.h>
void study1()
{
  char str[4];  //标记一
  gets(str);
  printf("你输入的内容是\n%s",str);
 }

int main(int argc, char* argv[])
{
  study1();
  system("PAUSE");
  return 0;
}
```

不明白的是，在上述"标记一"处，如果 `str` 的长度小于等于 4，release 下为什么没有生成`\_security\_cookie` 啊？如果长度大于 4 就会生成`\_security\_cookie`,我是在 `vs 2005` 下试验的，

求看雪牛牛给解答……

当应用程序启动时，程序的 `cookie`（4 字节（`dword`），无符号整型）被计算出来（伪随机数）并保存在

加载模块的 `.data` 节中,在函数的开头这个 `cookie` 被拷贝到栈中，位于 `EBP` 和返回地址的正前方（位于返

回地址和局部变量的中间）。

`[buffer][cookie][savedEBP][savedEIP]`

在函数的结尾处，程序会把这个 `cookie` 和保存在 `.data` 节中的 `cookie` 进行比较。

如果不相等，就说明进程栈被破坏，进程必须被终止。

为了尽量减少额外的代码行对性能带来的影响，只有当一个函数中包含字符串缓冲区或使用`\_alloca` 函数

在栈上分配空间的时候编译器才在栈中保存 `cookie`。另外，当缓冲区至少于 5 个字节时，在栈中也不保

存 `cookie`。

在典型的缓冲区溢出中，栈上的返回地址会被数据所覆盖，但在返回地址被覆盖之前，`cookie` 早已经被

覆盖了，因此就导致了 `exploit` 的失效（但仍然可以导致拒绝服务），因为在函数的结尾程序会发现 `cookie`

已经被破坏，接着应用程序会被结束。
