
> 并且配合这个对象，在虚拟机对象中设定相对应的三个方法： `make_frame`创建帧， `pop_frame`和`push_frame`两个对应帧和调用栈的操作，`run_frame`开始运行当前调用栈栈顶的帧。

## function

> 文章中指出这个对象的实现和理解有些曲折。
> 这个位置也的确是在我前面分析中卡住一下的位置，关于这里面的函数调用的实现。

我们需要注意他其中的`__call__()`方法创建了新的帧，并开始运行它。

```
def __call__(self, *args, **kwargs):

    if PY2 and self.func_name in ["<setcomp>", "<dictcomp>", "<genexpr>"]:

        # D'oh! 62aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0#2k6%4y4Q4x3X3g2H3P5i4c8Z5L8$3&6Q4x3X3g2G2M7X3N6Q4x3V1k6A6M7%4y4#2k6e0p5&6y4U0p5I4 Py2 doesn't know how to

        # inspect set comprehensions, dict comprehensions, or generator

        # expressions properly.  They are always functions of one argument,

        # so just do the right thing.

        assert len(args) == 1 and not kwargs, "Surprising comprehension!"

        callargs = {".0": args[0]}

    else:

        callargs = inspect.getcallargs(self._func, *args, **kwargs)

    frame = self._vm.make_frame(

        self.func_code, callargs, self.func_globals, {}

    )

    CO_GENERATOR = 32           # flag for "this code uses yield"

    if self.func_code.co_flags & CO_GENERATOR:

        gen = Generator(frame, self._vm)

        frame.generator = gen

        retval = gen

    else:

        retval = self._vm.run_frame(frame)

    return retval
```

> 然后相对应的在虚拟机对象的位置，
> 这里主要讲下对于程序执行的相关：

- **关于如何在解析code object**

在虚拟机对象下的方法`parse_nyte_and_args()`中，进行语法分析，

在这里主要使用`dis`中的方法，显示提取出来byte code， 然后`dis.opname`得到对应的命名，

> 这里要提到一点，这个字节码就是一串数据，一般来说语句自身会占有一个byte，如果没有参数后面就是跟着下一个语句，如果有参数则会是后面两位代表其参数值或者是其参数在code object 中的index。

> 因此注释中写，每次的运行只会解析1-3byte

并可以判断其是否有参数，然后对于参数跟在其后的直接取出，如果是直接其值，则直接取出，如果是对应的index 则做出对应的处理，

并且这里注意，调用这里分析，只会分析一句字节码，返回这一句的对应的相关数据，然后运行完后再继续解析下一句。于是这还保存着分析后的位置，( 其实这个道理应该是比较容易理解的)

终归就是在这里， 将我们汇编以后的二进制数据，转化成对应的的人类可读的名称，并处理出来其参数，最后返回回去一个此语句对应操作名称，对应的参数，还有对应分析到的位置。

```
def parse_byte_and_args(self):

    """ Parse 1 - 3 bytes of bytecode into

    an instruction and optionally arguments."""

    f = self.frame

    opoffset = f.f_lasti

    byteCode = byteint(f.f_code.co_code[opoffset])

    f.f_lasti += 1

    byteName = dis.opname[byteCode]

    arg = None

    arguments = []

    if byteCode >= dis.HAVE_ARGUMENT:

        arg = f.f_code.co_code[f.f_lasti:f.f_lasti+2]

        f.f_lasti += 2

        intArg = byteint(arg[0]) + (byteint(arg[1]) << 8)

        if byteCode in dis.hasconst:

            arg = f.f_code.co_consts[intArg]

        elif byteCode in dis.hasfree:

            if intArg < len(f.f_code.co_cellvars):

                arg = f.f_code.co_cellvars[intArg]

            else:

                var_idx = intArg - len(f.f_code.co_cellvars)

                arg = f.f_code.co_freevars[var_idx]

        elif byteCode in dis.hasname:

            arg = f.f_code.co_names[intArg]

        elif byteCode in dis.hasjrel:

            arg = f.f_lasti + intArg

        elif byteCode in dis.hasjabs:

            arg = intArg

        elif byteCode in dis.haslocal:

            arg = f.f_code.co_varnames[intArg]

        else:

            arg = intArg

        arguments = [arg]



    return byteName, arguments, opoffset
```

- **关于如何执行字节码**

这一部分在虚拟机对象的`dispatch`方法中。

首先是在虚拟机对象下定义好字节码对应的方法，他们使用`byte_xx`作为前缀，然后使用`getattr()`就可以获取到对应的方法，然后去执行。

最后每次执行都会返回一个指示下一次运行状态的变量，称为`why`，

> 注意这个`why`不要和我们函数调用(帧的切换)里面的返回值混为一谈。

## stack

关于栈的操作。

关于调用栈的操作我们在前面frame部分写了。

- 然后关于数据栈的操作：

主要是`top()`返回当前栈顶，`pop`和`push` 压栈和弹出，`popn`弹出指定的最后ｎ个数据组成的一个列表。

算是实现了几个基本的对于数据栈的操作。

其后的一些基础指令和顺序结构的指令基本会由这几个构成，

- 关于块栈的操作

这个块栈主要用于循环结构，还有一些生成器。并且会存在指示下一句运行状态的变量，我们前面提到的变量`why`，

## 最后

其实这里只是对于整体做出一个了解，写的还是比较简单的，下一步准备去看看关于其中的大批`byte_xx`系列的字节码实现，

原本是准备去在后面写关于ctf中python逆向的题目，但是本菜接触的python逆向题目大部分和解释器没太大关系的样子，主要是`pyc`文件，简单题目直接得到源码，一些难的会做出混淆，不能得到源码，小一点的的直接看字节码，大一些的想办法修复`pyc`文件转为源码。觉得写在一篇有些前后不搭，后面的文章再整理吧。

然后关于python的解释器部分，cpython的，在源码的[`/cpython/python/ceval.c`](https://bbs.kanxue.com/elink@cc2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3P5i4c8Z5L8$3&6Q4x3V1k6U0M7s2W2@1K9r3!0F1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2m8&6N6r3S2G2L8W2)9J5c8X3y4W2N6X3q4D9i4K6u0W2j5H3`.`., 一个五千六百多行的ｃ代码，里面的对应运行到对应字节码的部分(我们使用`getattr()`的位置)，在其中是一个巨大的两千行的`switch`结构。

> 有很多位置...写错的或者写的模模糊糊乱七八糟的位置，师傅们直接指出来我会进一步修改的，

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-3-26 15:15 被令则编辑，原因： 修改一部分代码和图片的格式
