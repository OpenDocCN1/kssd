
# 在 WinDbg 脚本中使用参数

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-173.htm`](https://www.pediy.com/kssd/pediy09/pediy09-173.htm)

> WinDbg `6.7.5.0` 版本运行脚本时多了一个新的命令 `$$>a<`，可以给脚本传递参数。下面是一个简单的例子，演示了参数的用法。

```
$$
$$ calc v0.0.1
$$ by 小喂 2007.06.08
$$

.if(@@c++(${/d:$arg1} && ${/d:$arg2}))
{
    .printf "\n%d + %d = %d\n", ${$arg1}, ${$arg2}, ${$arg1} + ${$arg2}
    .printf "%d - %d = %d\n", ${$arg1}, ${$arg2}, ${$arg1} - ${$arg2}
    .printf "%d * %d = %d\n", ${$arg1}, ${$arg2}, ${$arg1} * ${$arg2}
    .printf "%d / %d = %d\n", ${$arg1}, ${$arg2}, ${$arg1} / ${$arg2}
}

.else
{
    .printf "\nusage: $$>a< <path>\calc.txt arg1 arg2\n\n"
}
```

> 运行一下：

```
0:000> $$>a< d:\windbg\scripts\calc.txt @eax 4

1580724 + 4 = 1580728
1580724 - 4 = 1580720
1580724 * 4 = 6322896
1580724 / 4 = 395181
```

4dd466d0e1e0cc123afbe504da2462ca
