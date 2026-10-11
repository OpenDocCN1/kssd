
# IDA 之“不该出现的分割线”解决方法

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-305.htm`](https://www.pediy.com/kssd/pediy09/pediy09-305.htm)

> `IDA` 对浮点运算库函数没有做特别的处理。如果你 `Dasm` 类似:

```
int foo()

{

.......

    sqrt(X);

    sin(X);

    cos(X);

.......

}
```

> 你会发现 `foo` 无情的被 `sqrt` `sin` `cos` 分成几段。在这几个函数后面出现了分割线。

> 因为这些函数。在遇到异常的时候直接 `jmp exit` 了。所以 `IDA` 把这些函数归为 `does not return` 类型。因为是递归分析。这些函数所以后面代码没有和当前的函数建立关系。

> 解决办法就是把上述函数的 `does not return` 类型去掉。

`4dd466d0e1e0cc123afbe504da2462ca`
