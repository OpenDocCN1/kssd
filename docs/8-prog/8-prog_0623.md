
# 汇编写驱动字符串的一些问题

> 原文：[`www.pediy.com/kssd/pediy12/131303.html`](https://www.pediy.com/kssd/pediy12/131303.html)

刚开始学汇编写驱动时，最开始遇到的问题就是关于字符串的，因此和大家分享一下经验。

## 1. 如何用汇编实现 KdPrint

用 C 写驱动时，估计都比较喜欢用自定义的宏函数 `KdPrint` 来替代 `DbgPrint`。用 C 的宏定义来很简单，但是用汇编的宏指令就要麻烦一点。

汇编版的 `KdPrint`：

```
KdPrint macro Format,ParamList:VARARG
   ifdef Release
      EXITM
   endif
   FOR Param,<ParamList>
         push Param
   ENDM
   push Format
   call DbgPrint
endm
```

这个宏函数其实可以改进一下，由于 `DbgPrint` 是 C 调用函数，我比较懒，不喜欢自己平衡堆栈，因此我有做了这样的改进。

```
KdPrintSafe macro Format,ParamList:VARARG
LOCAL Count
   ifdef Release
      EXITM
   endif
   Count=0
   FOR Param,<ParamList>
         push Param
         Count=Count+1
   ENDM
   push Format
   call DbgPrint
   Count=(Count+1)*4
   add  esp,Count
endm
```

## 2. 汇编如何使用宽字符的问题

`win32` 汇编中，我们比较习惯使用 `ASCII` 码，然而在驱动中避免不了使用 `Unicode`。

关于宽字符的问题，直接使用 `masm32/macro/ucmacro.asm` 里的两个宏，就可以了。但是驱动中，还有使用 `UNICODE_STRING` 的问题。

（以下引自 KdmTut）

`UNICODE_STRING` 结构和里面的字符串数据，如果你不喜欢使用宏，那么可以用下面的常规方法：

```
usz dw 'U', 'n', 'i', 'c', 'o', 'd', 'e', ' ', 's', 't', 'r', 'i', 'n', 'g', 0
us  UNICODE_STRING {sizeof usz - 2, sizeof usz, offset usz}
```

但是我可不喜欢这么麻烦，所以我写了下面这些宏：`COUNTED_UNICODE_STRING`、`$COUNTED_UNICODE_STRING`、`CCOUNTED_UNICODE_STRING` 和 `$CCOUNTED_UNICODE_STRING`（定义在 `masm32\Macros\Strings.mac` 中）。

有一点我要对 KdmTut 进行补充，类似 `usz dw 'U', 'n', 'i', 'c', 'o', 'd', 'e', ' ', 's', 't', 'r', 'i', 'n', 'g', 0` 这样的形式，还是有用的，因为这些宏还不支持中文。

至于这些宏的用法，请看我的下一篇文章 [`bbs.pediy.com/showthread.php?t=131332`](http://bbs.pediy.com/showthread.php?t=131332)。
