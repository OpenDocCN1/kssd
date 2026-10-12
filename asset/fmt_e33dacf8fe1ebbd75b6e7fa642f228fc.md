
This option tells the compiler to generate a `Stdcall` calling sequence for function calls (does not generate underscores, preserve case, called function pops the stack, and pushes parameters right to left). This is the same as declaring all subroutines and functions with the `___stdcall` keyword. Functions must pass the correct number and type of arguments.

You can use the `___cdecl`, `___pascal`, `___fastcall` keywords to specifically declare a function or subroutine using another calling convention.

`Default = C (-pc)`

![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg)

学习了～

不过还是要解决两个问题

毕竟 `BCB` 是 `OMF` 格式的

首先解决的是～函数参数传递方式（这个很好解决一般都可以搞定的～`use C`）

下面要解决的就是～`obj` 中导出函数的问题了

以前写过那个说明的～`COFF` 格式回在使用的 `API` 前面自动连接～`___imp_`

还有后面增加`@X` 的标号

这个问题要解决不然函数无法进行连接的

不过看了 `LZ` 的文章～小弟我深感自己的渺小.....

感谢 `LZ` 这么好的文章～谢谢～我要把这篇文章顶上去。。

让全坛子的人都看到～～`LZ` 原谅我的自私吧。。。
