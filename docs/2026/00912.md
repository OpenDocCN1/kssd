
# [分享]IDA之“不该出现的分割线”解决方法

IDA对浮点运算库函数没有做特别的处理。如果你`Dasm`类似:

```c
int foo()
{
    sqrt(X);
    sin(X);
    cos(X);
}
```

你会发现`foo`无情的被`sqrt` `sin` `cos`分成几段。在这几个函数后面出现了分割线。

因为这些函数。在遇到异常的时候直接`jmp exit`了。所以IDA把这些函数归为does not return类型。因为是递归分析。这些函数所以后面代码没有和当前的函数建立关系。

解决办法就是把上述函数的does not return类型去掉。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
