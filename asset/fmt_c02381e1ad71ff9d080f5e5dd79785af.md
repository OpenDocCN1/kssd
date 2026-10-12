
# 宏汇编语言控制台程序的“宏” 解析

> 原文：[`www.pediy.com/kssd/pediy10/66818.html`](https://www.pediy.com/kssd/pediy10/66818.html)

作 者: hjjdebug

时 间: 2008-06-18, 20:07

虽然我们不提倡用宏，宏本质上只不过是一种替换而已。

但宏汇编语言控制台程序中却大量使用了宏。经研究后，发现某些宏用起来确实方便。

这里推荐 4 个，`chr$`、`str$`、`print`、`input`

当然，象那些简单的宏例如 `exit` 宏，就随你意愿可用可不用了。

下面是控制台程序中常用的宏：

## 1. 控制台程序 `print` 宏的完全解析

例子程序可以参考宏汇编的 `tutorial\console` 下的程序，我们重点分析一句代码：

`print` `chr$("Hey, this actually works.",13,10)`

先看 `chr$` 宏

```asm
chr$ MACRO any_text:VARARG
    LOCAL txtname
    .data
      txtname db any_text,0
    .code
    EXITM <OFFSET txtname>
ENDM
```
`chr$` 是带一个参数的宏。

1. 在数据区中定义一个标号 `txtname`，这个标号是局部的，意思是说，仅在该宏中有意义。
   然后把后续的参数定义到数据区中，尾巴补 0.
2. 宏结束，返回一个标号地址。

再看 `print` 宏

```asm
print MACRO arg1:REQ,varname:VARARG   ;; display zero terminated string
    invoke StdOut,reparg(arg1)
  IFNB <varname>
    invoke StdOut,chr$(varname)
  ENDIF
ENDM
```
1. `print` 宏似乎带 2 个参数的宏？一个 `arg1`, 一个 `varname`, `varname` 可以为空。当为空时，只替代第一句。

`invoke` `StdOut`, `reparg(arg1)`

再看 `reparg(arg1)` 宏

```asm
; -----------------------------------------------------------
; This macro replaces quoted text with a DATA section OFFSET
; and returns it in ADDR "name" format. It is used by other
; macros that handle optional quoted text as a parameter.
; -----------------------------------------------------------
reparg MACRO arg
  LOCAL nustr
    quot SUBSTR <arg>,1,1
  IFIDN quot,<">     ;; if 1st char = "
    .data
      nustr db arg,0        ;; write arg to .DATA section
    .code
    EXITM <ADDR nustr>      ;; append name to ADDR operator
  ELSE
    EXITM <arg>             ;; else return arg
  ENDIF
ENDM
```
1. 判断参数第一个字符是否为", 如是， 定义为数据区，末尾补 0，返回地址。否则，直接退出。返回原 `arg`.
   可见，`args` 经 `replace` 宏替代后，可以不用 `chr$` 宏

; ---------------------------------------------------------------------------

分析完后，可以做个新实验。

改原句为如下，应也可工作

`print "Hey, this actually works.",13,10`

虽然打印结果没有变化，但实际宏的控制过程却是改变了的。

实际宏的工作过程是，将 `print` 后所跟参数看成两个参数， `"Hey, this actually works."` 为第一参数，`replace` 参数，
第二部分 `13,10` ... 为可变参数，`print` 的第二参数， 用 `chr$` 宏修饰之。所以编译的结果形成两条语句。

```asm
invoke StdOut,reparg(arg1)          .data
                ??0019 db "Hey, this actually works.",0
                .code
                invoke StdOut,ADDR ??0019
invoke StdOut,chr$(varname)       .data
                ??001A db 13,10,0
                .code
                invoke StdOut,OFFSET ??001A
```
也一样能满足要求。

如此看来，用 `chr$` 修饰字符串，有更高的效率。

`print chr$("Hey, this actually works.",13,10)`  `.data`
`??0019 db "Hey, this actually works.",13,10,0`
`invoke StdOut,OFFSET ??0019`

; ---------------------------------------------------------------------------

说了一大堆，到底得出点什么结论呢？两点最重要

1. `chr$` 是个宏，用来定义 0 字终结字符串。
2. `print` 是个宏，用来执行 `StdOut` 函数。

另外，关于 `exit` 宏：如下定义

```asm
; --------------------------------------------------------
; exit macro with an optional return value for ExitProcess
; --------------------------------------------------------
exit MACRO optional_return_value
  IFNDEF optional_return_value
    invoke ExitProcess, 0
  ELSE
    invoke ExitProcess,optional_return_value
  ENDIF
ENDM
```
也就是说，根据 `exit` 宏是否带参数，将被翻译成不同的 `ExitProcess` 函数。

`input` 宏有如下的解释和定义

```asm
comment * -------------------------------------
    use the "input" macro as follows,
    If you want a prompt use this version
    mov lpstring, input("Type text here : ")
    If you don't need a prompt use the following
    mov lpstring, input()
    NOTE : The "lpstring" is a preallocated
           DWORD variable that is either LOCAL
           or declared in the .DATA or .DATA?
           section. Any legal name is OK.
    LIMITATION : MASM uses < > internally in its
    macros so if you wish to use these symbols
    in a prompt, you must use the ascii value
    and not use the symbol literally.
    EXAMPLE mov var, input("Enter number here ",62," ")
    ------------------------------------------- *
  input MACRO prompt:VARARG
      LOCAL txt
      LOCAL buffer
    IFNB <prompt>
      .data
        txt db prompt, 0
        buffer db 128 dup (0)
        align 4
      .code
      invoke StdOut,ADDR txt
      invoke StdIn,ADDR buffer,LENGTHOF buffer
      invoke StripLF,ADDR buffer
      mov eax, offset buffer
      EXITM <eax>
    ELSE
      .data
        buffer db 128 dup (0)
        align 4
      .code
      invoke StdIn,ADDR buffer,LENGTHOF buffer
      invoke StripLF,ADDR buffer
      mov eax, offset buffer
      EXITM <eax>
    ENDIF
  ENDM
```
; ---------------------------------------------------------------------------

有了 `print` 宏解释的基础，`input` 宏就好理解了。我这里简单翻译一下，为意译非逐字译。

`input` 宏有两种用法，根据是否有提示信息而划分.

一种带提示，例如：

`mov lpstring, input("Type text here : ")`

一种不带提示：例如：

`mov lpstring, input()`

其中 `lpstring` 是预先分配好的变量，可以为局部变量或存在于 `data`, `data?`段中。它是一个

只占 4 字节的指针。而那个 `input` 为我们提供的缓冲区，是 128 字节。定义在数据区中，如果

`promp` 不为空，`promp` 字符串也会被定义在数据区中。

```asm
; ----------------------------------------------------------
;  str$ macros that takes a DWORD parameter and
; returns the address of the buffer that holds the result.
; The return format is for use within the INVOKE syntax.
; ----------------------------------------------------------
  str$ MACRO DDvalue
    LOCAL rvstring
    .data
      rvstring db 20 dup (0)
      align 4
    .code
    invoke dwtoa,DDvalue,ADDR rvstring
    EXITM <ADDR rvstring>
  ENDM
```
`str$` 宏以 `DWORD` 值为参数，定义一个缓冲区 20bytes 接受转换后的字符串。

然后调用 `dwtoa` 函数将数值转换为字符串。注意，字符串是 10 进制的。

`str$` 宏把数值转变成字符串。在数据区定义了 20 bytes 空间，隐含调用 `dwtoa` 或 `dw2a` 函数。

`chr$` 宏在数据区定义了相关数据。

这两个宏比较重要，可以常用，再加上 `print` 宏， `input` 宏，

`print` 宏，隐含调用了 `StdOut` 向屏幕输出字符串。字符串定义在堆中。

`input` 宏，隐含调用了 `StdIn`, 并在堆中定义了 128 个 bytes 来接受键盘的输入。

这四个宏需要掌握。

其它宏视情况了，理解了可以用，不理解就不用，尽量少用。

宏与函数的显著差别是什么？

宏是编译时的替代，函数是运行时的连接。这是实质差别。

调用宏是不需要 `invoke` 修饰的，而调用函数需要用 `invoke` 修饰（供查错用）。这是书写差别。

宏不仅在控制台下常用，在 windows 程序中也可以常用。不过 `print` 和 `input` 宏就不好用了，但 `chr$` 和 `str$` 还一样能用。举一个简单例子吧。

```asm
.386
.model flat,stdcall
option casemap:none
include windows.inc
include user32.inc
include kernel32.inc
include masm32.inc
include c:\masm32\macros\macros.asm
includelib masm32.lib
includelib kernel32.lib
includelib user32.lib
.code
start:
  mov eax,100
  invoke MessageBox,NULL,str$(eax), chr$("hello"), MB_OK
  invoke ExitProcess,NULL
  end start
```
如果不用宏完成这个功能，既要在数据区定义字符串，又要定义数据转换接受区空间，又要调用整数到 ascii 的转换函数。而这个程序，在一个 `messagebox` 中全部搞定。
