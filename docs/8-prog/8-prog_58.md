
# INC2l - win32 版本+源码+演示

> 原文：[`www.pediy.com/kssd/pediy12/144409.html`](https://www.pediy.com/kssd/pediy12/144409.html)

`inc2l` 郁闷了一段时间，没办法了只好自己写个，

把它放在 `masm\Bin` 下面，把 `*.inc` 拖放进去，按开始，搞定。

由于不怎么用的东西，功能也从简了。![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) |
| :--- |
| `MakeLib.rar` |

### 海阔天空 08:25:28

使用 `VC++` 的 `LIB` 工具，带 `/DEF:(.def 文件名)` `/MACHINE:IX86` (80X86 机器)，就输出符合 `VC++` 格式的 `LIB` 文件了。

`EXAMPLE`：

```
LIB /DEF:VideoDeCoder.def /MACHINE:IX86
```

感谢他提供了更,,,代码也我懒的改了，`MASM` 的也一样
