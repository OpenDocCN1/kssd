
# Tasm 编译器+虚拟机

> 原文：[`www.pediy.com/kssd/pediy11/127181.html`](https://www.pediy.com/kssd/pediy11/127181.html)

Tasm 是一个类 `asm` 的编译器+虚拟机，把源代码编译成 `bytecode` 并丢入 `VM` 中执行。

它的前身是 `OllyMachine`，因此，基本语法雷同于 `OllyMachine`，`OM` 的帮助文件请见：

http://www.luocong.com/om

【补充说明】

一、写这个编译器的初衷是为了改善 `OllyMachine` 难用的几个特性（例如不能自定义变量名，寻址方式不方便等），但是 `OllyDbg` 2.0 迟迟不发布正式版，3 年又 3 年了，究竟要等到什么时候啊？不想把这个工程烂在自己手里，我也先丢个 `alpha` 版吧。

二、取消了 `OllyMachine` 的那几十个难记的寄存器，取而代之的是可以自定义变量名（总共可以自定义 255 个，应该够用了吧？）

变量名无需声明，可直接使用，命名规则同 `C` 的语法。例如：

```
mov i, 1
```

注意：变量的初始值是 0，例如：

```
mov i, j
```

那么 `i` 和 `j` 的值都为 0，因为 `j` 的初始值是 0，然后赋值给 `i`。

三、寻址方式作了改进，可以支持：

```
mov dword ptr [0x78787878], 0x12345678
mov word ptr [0x78787878], 0x1234
```

等等。

四、函数的返回值储存在保留的变量 `rtx` 中（`RTX: ReTurn X` 的意思）。

例如：

```
invoke malloc, 1000
mov buffer, rtx
```

五、框架基本搭好了，剩下的是补充 `API` 的体力活，但我现在没时间补完了。

为了测试用，我仅仅写了以下的 `API`，使用的语法请见 `MSDN`：

- `printf`
- `malloc`
- `free`
- `memset`
- `fopen`
- `fclose`
- `fread`
- `fwrite`
- `fgetc`
- `fputc`
- `fseek`
- `ftell`
- `feof`
- `rewind`
- `PathFileExists`
- `DeleteFile`
- `RegOpenKeyEx`
- `RegCloseKey`

\- THE END -

罗聪

2010-12-27

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || Tasm\_20101227.rar
|
