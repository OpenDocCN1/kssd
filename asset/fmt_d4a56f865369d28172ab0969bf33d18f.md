
# 逆向进阶复杂分支结构逆向

> 原文：[`www.pediy.com/kssd/pediy12/129711.html`](https://www.pediy.com/kssd/pediy12/129711.html)

熟知 `if-else` 的典型结构是逆向入门的必经路程，然而我们实际逆向时，遇到的情况远比这要复杂

## 一、条件分支的逆向基础典型的 if-else 结构

我们知道简单的 `if-else` 的跳转结构是这样的。

例如：

```
if(i1<x1)

    ……;

else

    ……;
```

在反汇编结果中我们会看到类似下面的结果

![](img/47fb01212131b8b76b84b0f88d738297.jpg)

两点说明：

1. `jxx AAAAAAAA` 这句由于 false 则跳转，因此还原为 if 代码时，要注意。
2. `jmp BBBBBBBB` 这句在功能上属于 if 的部分，但是如果没有 else，编译器不会填写此句。

## 二、多层嵌套 if-else

```
if(i<x1)

{…

 if(i<x2)

   {…

   if(i<x3)

    ….

   else

    ….

   }

 else

  ……

}
```

对应反汇编形态：(地址 `A`、`B`、`C`、`D` 从小到大分布)

**Debug 版**：

![](img/472c26706c92ef35159abed45d69aebe.jpg)

说明：

这里我们看到在 `Debug` 版里，多层嵌套 `if-else` 只是简单重复单层 `if-else` 的结构。另外，`jxx` 仍然是 false 则跳转，翻译为 if 时，要注意。

**Release 版**：

![](img/9cdc42b7c09db5b8adcd7143eef98aa3.jpg)

说明：

我们看到 `Release` 版对 `Debug` 改进，避免了 `jmp` 的多次运行。

## 三、复合条件的条件分支

**(1) 与 `&&`**

```
if(i1<x1 && i2<x2 && i3<x3)

  ……

else

  ……
```

反汇编形态：

![](img/2a3f6c1fb4af7bf2bc2545915114de54.jpg)

说明：

`N` 条分支全部向 `Jmp` 后跳转。

**(2) 或 `||`**

```
if(i1<x1 || i2<x2 || i3<x3)

  ……

else

  ……
```

反汇编形态：

![](img/6c070496ed0aa8f80787234d9995c0fd.jpg)

说明：

只有 `N-1` 个分支向 `jmp` 后跳转。

**(3) 混合**

```
if(i1<x1||i2<x2&&i3<x3)

  ……

else

  ……
```

反汇编形态：

![](img/c7848c4814bd10e9df9836d48339ddc7.jpg)

```
if(i1<x1&&i2<x2||i3<x3)

  ……

else

  ……
```

反汇编形态：

![](img/acd4f79c080d45d827d87f07867dbff1.jpg)

恩，实际情况是类似这样的分支代码：

```
if(i<x1)

{…

 if(i<x2)

   {…

   if(i<x3)

    ….

   else

    ….

   }

 else

  ……

}
```

比如其中的 `else` 部份，及可能编译后 `else` 部份的代码已经不在整个分支代码的地址范围内，编译器采用至少两个跳转来调用 `else` 部份的代码，也就是这样

```
if(i<x1)

{…

 if(i<x2)

   {…

   if(i<x3)

    ….

   else    ; jxx aaaaaaaa 编译后的代码并未顺序排列，而是跳转到别处运行，但是总会 jmp 回来

    ….

   }

 else

  ……

         ; 地址 bbbbbbbb

}
```

第一个 `else` 部份的代码：

```
地址 aaaaaaaa

…

jmp bbbbbbbb ;此处再跳回分支代码的最后
```

当然，实际情况中还会有更多的 `jmp`，不过总会 `jmp` 回程序正常的逻辑范围内
