
唯一的一个入口位置就是`run_code`方法，这个方法创建第一个帧，并运行，此后解释器将会执行里面对应的字节码，并由此，开始处理数据，处理函数调用，返回的调用栈增长和收缩，传递函数返回值，
当建立的第一个帧运行结束返回时，整个程序执行完毕。

> 整个这个对象都在文件`pyvm2.py`中

> 和自身虚拟机相关的是一个`__init__()`定义 和`run_code()`一个入口方法的定义。

### frame

这个对象是一些属性的梳理和设定，而并不会提供相对应的可运行的方法，
主要的属性包括，被编译后的code object，命名空间，保存调用自身的前一个帧，一个自身独立的数据栈和块栈，当前执行的最后一句指令，

> 并且配合这个对象，在虚拟机对象中设定相对应的三个方法：`make_frame`创建帧，`pop_frame`和`push_frame`两个对应帧和调用栈的操作，`run_frame`开始运行当前调用栈栈顶的帧。

### function

> 文章中指出这个对象的实现和理解有些曲折。
>
> 这个位置也的确是在我前面分析中卡住一下的位置，关于这里面的函数调用的实现。

我们需要注意他其中的`__call__()`方法创建了新的帧，并开始运行它。

```python
def __call__(self, *args, **kwargs):
if PY2 and self.func_name in ["<setcomp>", "<dictcomp>", "<genexpr>"]:
# D'oh! http://bugs.python.org/issue19611 Py2 doesn't know how to
# inspect set comprehensions, dict comprehensions, or generator
# expressions properly. They are always functions of one argument,
# so just do the right thing.
assert len(args) == 1 and not kwargs, "Surprising comprehension!"
callargs = {".0": args[0]}
else:
callargs = inspect.getcallargs(self._func, *args, **kwargs)
frame = self._vm.make_frame(
self.func_code, callargs, self.func_globals, {}
)
CO_GENERATOR = 32 # flag for "this code uses yield"
if self.func_code.co_flags & CO_GENERATOR:
gen = Generator(frame, self._vm)
frame.generator = gen
retval = gen
else:
retval = self._vm.run_frame(frame)
return retval
```

> 然后相对应的在虚拟机对象的位置，
>
> 这里主要讲下对于程序执行的相关：

* **关于如何在解析code object**

在虚拟机对象下的方法`parse_nyte_and_args()`中，进行语法分析，
在这里主要使用dis中的方法，显示提取出来byte code，然后`dis.opname`得到对应的命名，

> 这里要提到一点，这个字节码就是一串数据，一般来说语句自身会占有一个byte，如果没有参数后面就是跟着下一个语句，如果有参数则会是后面两位代表其参数值或者是其参数在code object 中的index。

因此注释中写，每次的运行只会解析1-3byte
并可以判断其是否有参数，然后对于参数跟在其后的直接取出，如果是直接其值，则直接取出，如果是对应的index 则做出对应的处理，
并且这里注意，调用这里分析，只会分析一句字节码，返回这一句的对应的相关数据，然后运行完后再继续解析下一句。于是这还保存着分析后的位置，( 其实这个道理应该是比较容易理解的)
终归就是在这里，将我们汇编以后的二进制数据，转化成对应的的人类可读的名称，并处理出来其参数，最后返回回去一个此语句对应操作名称，对应的参数，还有对应分析到的位置。

```python
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

* **关于如何执行字节码**

这一部分在虚拟机对象的`dispatch`方法中。
首先是在虚拟机对象下定义好字节码对应的方法，他们使用`byte_xx`作为前缀，然后使用getattr 就可以获取到对应的方法，然后去执行。
最后每次执行都会返回一个指示下一次运行状态的变量，称为`why`，

> 注意这个why不要和我们函数调用(帧的切换)里面的返回值混为一谈。

### stack

关于栈的操作。
关于调用栈的操作我们在前面frame部分写了。
* 然后关于数据栈的操作：
主要是`top()`返回当前栈顶，`pop`和`push`压栈和弹出，`popn`弹出指定的最后ｎ个数据组成的一个列表。
算是实现了几个基本的对于数据栈的操作。
其后的一些基础指令和顺序结构的指令基本会由这几个构成，
* 关于块栈的操作
这个块栈主要用于循环结构，还有一些生成器。并且会存在指示下一句运行状态的变量，我们前面提到的变量why，

# 最后

其实这里只是对于整体做出一个了解，写的还是比较简单的，下一步准备去看看关于其中的大批`byte_xx`系列的字节码实现，
原本是准备去在后面写关于ctf中python逆向的题目，但是本菜接触的python逆向题目大部分和解释器没太大关系的样子，主要是pyc文件，简单题目直接得到源码，一些难的会做出混淆，不能得到源码，小一点的的直接看字节码，大一些的想办法修复pyc文件转为源码。觉得写在一篇有些前后不搭，后面的文章再整理吧。

然后关于python的解释器部分，cpython的，在源码的`/cpython/python/ceval.c`，一个五千六百多行的c代码，里面的对应运行到对应字节码的部分(我们使用`getattr()`的位置)，在其中是一个巨大的两千行的switch结构。

> 有很多位置...写错的或者写的模模糊糊乱七八糟的位置，师傅们直接指出来我会进一步修改的，

# pyhton interpreter byterun和底层`code object`的简单了解

目录

- pyhton interpreter byterun和底层`code object`的简单了解
- python interpreter
- python interpreter
- stack 栈机器
- `byte code` 和 `code object`
- python - dis模块：
- frame 帧
- [byterun](https://bbs.kanxue.com/#byterun)
- [顺序分析](https://bbs.kanxue.com/#顺序分析)
- [main](https://bbs.kanxue.com/#main)
- [`run_python_...`](https://bbs.kanxue.com/#`run_python_...`)
- [`run_code_object`](https://bbs.kanxue.com/#`run_code_object`)
- [`VirtualMachine`](https://bbs.kanxue.com/#`virtualmachine`)
- [`run_code`](https://bbs.kanxue.com/#`run_code`)
- [`make_frame`](https://bbs.kanxue.com/#`make_frame`)
- [`run_frame`](https://bbs.kanxue.com/#`run_frame`)
- [`dispatch()`](https://bbs.kanxue.com/#`dispatch()`)
- [`calll_function`](https://bbs.kanxue.com/#`calll_function`)
- [整体分析](https://bbs.kanxue.com/#整体分析)
- [`VirtualMachine`](https://bbs.kanxue.com/#`virtualmachine`)
- [frame](https://bbs.kanxue.com/#frame)
- [function](https://bbs.kanxue.com/#function)
- [stack](https://bbs.kanxue.com/#stack)
- [最后](https://bbs.kanxue.com/#最后)

> > 首先这是对于文章
>
> 这是一篇关于python虚拟机底层的一个文章
>
> 前部分是主要核心是500line那篇文章的笔记。
>
> 后部分主要是对于`byterun`的源码分析。
>
> > 主要参考文档：
>
> [500 Lines or Less: python interpreter written in python-原文](https://bbs.kanxue.com/elink@33eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4G2M7$3q4T1L8$3!0C8i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3V1j5#2x3o6m8x3i4K6u0r3j5g2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6r3W2F1N6r3g2J5M7s2u0W2N6r3g2J5i4K6u0V1N6%4u0A6N6s2c8W2L8W2)9J5k6r3W2F1i4K6u0V1M7s2W2@1K9r3!0F1i4K6u0W2K9s2c8E0L8l9`.`.) 、[500 Lines or Less: python interpreter -- 翻译](https://bbs.kanxue.com/elink@003K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4q4A6L8X3N6&6N6h3&6Z5j5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6r3q4G2N6r3q4G2i4K6u0r3)、[实验楼也有一个相关的教程](https://bbs.kanxue.com/elink@f80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6K9r3W2&6j5h3&6D9L8%4g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4g2J5M7$3g2K6i4K6u0r3y4e0f1@1)、[一个相关的ppt](https://bbs.kanxue.com/elink@6afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8r3W2V1k6i4y4Z5j5i4u0W2i4K6u0W2L8X3g2@1i4K6u0r3j5h3E0S2M7s2c8#2M7W2)9J5c8X3u0&6N6r3g2J5N6h3&6Q4x3X3c8S2i4K6u0V1M7s2W2@1K9r3!0F1i4K6u0V1j5Y4W2@1k6h3y4G2k6r3g2Q4x3X3c8A6L8Y4c8W2M7Y4m8J5k6i4c8W2M7W2)9J5k6r3q4@1i4K6u0V1L8Y4W2U0M7s2W2@1K9r3!0F1)
>
> 这几个都是基于原文　讲的差不多的东西。
>
> > 主要的研究对象：
>
> [byterun-github](https://bbs.kanxue.com/elink@0c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6h3c8T1j5i4c8Q4x3V1k6T1P5i4c8W2M7Y4g2F1)
>
> > 另外还有一些文章，关注的位置就和我们这个byterun不同了，想进一步了解的朋友可以看下：
>
> [python运行原理－淡水](https://bbs.kanxue.com/elink@52aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5k6i4y4@1M7X3q4F1i4K6u0W2L8X3g2@1i4K6u0r3x3U0l9I4y4g2)9J5c8U0p5H3i4K6u0r3x3U0u0Q4x3V1k6Z5L8%4N6Q4x3X3c8H3P5i4c8Z5L8$3&6Q4x3X3c8U0L8$3c8W2i4K6u0V1M7Y4g2F1i4K6u0r3)、[python import 机制 -- laike9m](https://bbs.kanxue.com/elink@f31K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3W2C8k6e0W2E0i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4m8&6N6r3S2G2L8Y4S2A6j5h3&6Y4i4K6u0V1k6s2g2A6i4K6u0V1k6r3q4G2i4K6u0V1M7Y4g2Q4x3X3c8B7K9g2)9J5k6s2A6Z5K9g2)9J5k6s2S2A6j5h3&6Y4i4K6u0V1K9X3W2W2i4K6u0o6y4U0m8Q4x3V1j5`.)、[python解释器和java虚拟机](https://bbs.kanxue.com/elink@27cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3y4o6b7I4z5o6t1@1i4K6u0r3K9X3q4$3j5g2)9J5k6s2k6A6M7Y4c8#2j5h3I4Q4x3X3c8E0j5h3y4Z5K9h3&6W2i4K6u0V1N6Y4y4Q4x3X3c8H3P5i4c8Z5L8$3&6Q4x3X3c8A6L8Y4c8W2M7Y4m8J5k6i4c8W2M7W2)9J5k6s2m8S2M7X3I4S2L8X3y4W2)

# python interpreter

## python interpreter

这里是两个点：
- 什么是python interpreter：
我们在这里提到的python解释器，是指python执行代码最后的一步
- python 程序运行：
在解释器接手之前。python会执行：词法分析，语法分析， 编译。
然后将python的源代码生成 `code object`， 这里面就是python解释器可以识别的指令，然后解释器去解释(运行) 其中的指令。
另外Python的解释器是一个栈堆机器， 其底层使用的逻辑是用栈表示的，而我们普通的电脑底层是寄存器机器，使用的是数据存放在内存地址

> 这里说明:
>
> - 编译 - compile
>
