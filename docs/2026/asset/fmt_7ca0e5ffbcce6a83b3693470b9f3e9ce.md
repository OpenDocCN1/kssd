
# 6 noleek

> My code had a couple of pesky format string vulnerabilities that kept getting exploited...I'm sure it'll fix itself if I just compile with RELRO and take away output...
>
> 题目说明很清楚，存在格式化字符串漏洞，但是使用 RELRO 进行编译并删除了输出。
>
> 注意做题环境，这道题的环境是`debian@sha256:98......`，我一开始用ubuntu22做题，调试的时候执行到`fprintf`过不去，会崩掉。

Dockerfile如下:

```dockerfile
FROM pwn.red/jail

COPY --from=debian@sha256:98d3b4b0cee264301eb1354e0b549323af2d0633e1c43375d0b25c01826b6790 / /srv
COPY noleek /srv/app/run
COPY flag.txt /srv/app/flag.txt
RUN chmod 755 /srv/app/run
```

## 6.1 运行分析

有源码，先看看源码。如下所示，两次输入，然后输出不再是给stdout，而是给了`/dev/null`

```c
#include <stdio.h>
#include <stdlib.h>
#define LEEK 32
void cleanup(int a, int b, int c) {}
int main(void) {
setbuf(stdout, NULL);
FILE * leeks = fopen("/dev/null", "w");
if (leeks == NULL) {
puts("wtf");
return 1;
}
printf("leek? ");
char inp[LEEK];
fgets(inp, LEEK, stdin);
// 往长度为32的栈buffer输入数据
fprintf(leeks, inp);
// 把输入输出到/dev/null，格式化字符串漏洞
printf("more leek? ");
fgets(inp, LEEK, stdin);
// 再次输入
fprintf(leeks, inp);
// 格式化字符串漏洞
printf("noleek.\n");
cleanup(0, 0, 0);
return 0;
}
```

`fprintf`函数原型：

```c
//发送格式化输出到流 stream 中
int fprintf(FILE *stream, const char *format, ...)
 // 正常使用
fprintf(fp, "%s %s %s %d", "We", "are", "in", 2014);
```

关闭Canary，RELRO是Full RELRO。

```
root@429d186baf6b:/chal# ./noleek
leek? aaa
more leek? aaa
noleek.
root@429d186baf6b:/chal#
root@429d186baf6b:/chal# checksec noleek
[*] '/chal/noleek'
Arch:     amd64-64-little
RELRO:    Full RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      PIE enabled
```

这道题没有输出，也就不能泄露地址了。利用思路参考上一题，不过区别是本题利用`%*c`来修改栈里存储的某个指针的低4字节，使其指向存储`main_ret`的地方。

### 格式化字符串中`*`的作用

星号在 `printf` 格式化字符串中的作用是用于指定动态宽度或精度。在输出字符串时，星号可以通过传递另一参数 `int` 类型的方式动态定义输出的宽度或精度。测试格式化字符串中 `*` 的作用：

```c
#include<stdio.h>
void main(){
printf("%*c", 5, '*'); // 输出一个星号字符 '*', 并指定宽度为 5
printf("\n");
printf("%.*f", 2, 3.14159);
printf("\n");
}
```

- 第一个输出结果：在宽度为 5 的位置上输出星号字符
- 第二个输出结果：星号指定了精度为 2，输出结果保留两位小数

```
root@429d186baf6b:/chal# ./a.out
*
3.14
```

## 6.2 调试

`gdb`调试，输入`AAAAAAAA.%p.%p.%p.%p`。

- 存储`main_ret`的地址是`0x7ffc6faf3498`
- `rsi`，也就是`fprintf`第二个参数`inp`的地址是`0x7ffc6faf3460`
- `fprintf`第三个参数`vararg`的值和`rsi`一致，也是`0x7ffc6faf3460`

![image-20230716021111567](https://bbs.kanxue.com/upload/attach/202402/830671_96RBSYYKVNR9WA9.jpg)

和上一题一样，在栈里找一个存储了栈地址的地方，算出其偏移
