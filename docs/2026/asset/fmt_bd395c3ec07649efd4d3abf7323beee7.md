
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式 `cvar = avar + bvar` 中，加法运算符（`+`）将 `avar` 与 `bvar` 相加，得到它们的和 `cvar`

在 `List.of(arg0, arg1, arg2)` 中，`List` 接口的工厂方法 `of()` 接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用 `cmd` 命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `#!/usr/bin/env python2`  `import` `sys`  `import` `dis`    `def` `func(a, b):`  `c` `=` `a` `+` `b`  `return` `c` `*` `10`    `co` `=` `func.__code__`  `dis.disassemble(co)`    `def` `mytrace(frame, why, arg):`  `print` `"Trace"``, frame, why, arg`  `return` `mytrace`    `print` `"=== Trace Start ==="`  `sys.settrace(mytrace)`    `func(``3``,` `4``)` |

输出如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `$ .``/``demo.py`  `6`           `0` `LOAD_FAST` `0` `(a)`  `3` `LOAD_FAST` `1` `(b)`  `6` `BINARY_ADD`  `7` `STORE_FAST` `2` `(c)`    `7`          `10` `LOAD_FAST` `2` `(c)`  `13` `LOAD_CONST` `1` `(``10``)`  `16` `BINARY_MULTIPLY`  `17` `RETURN_VALUE`  `=``=``=` `Trace Start` `=``=``=`  `Trace <frame` `object` `at` `0x10b8cb218``> call` `None`  `Trace <frame` `object` `at` `0x10b8cb218``> line` `None`  `Trace <frame` `object` `at` `0x10b8cb218``> line` `None`  `Trace <frame` `object` `at` `0x10b8cb218``>` `return` `70`  `Trace <frame` `object` `at` `0x10b98c050``> call` `None`  `Trace <frame` `object` `at` `0x10b98c050``> call` `None` |

Python 的标准库中也提供了 [`trace` 模块](https://bbs.kanxue.com/elink@cd3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2W2@1K9r3!0F1i4K6u0W2L8%4u0Y4i4K6u0r3x3W2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6@1M7X3q4U0k6g2)9J5k6h3S2@1L8h3H3`.`) 来支持字节码跟踪，查看该模块的的源码发现实际上也是用了 `sys.settrace` 或者 `threading.settrace` 来设置跟踪回调。

不过，使用 `sys.trace` 并不是每条指令都跟踪的，只针对特定事件进行跟踪:

- `call`: 函数调用
- `return`: 函数返回
- `line`: 一行新代码
- `exception`: 异常事件

而且该代码中也做了对应的防护，使用 `trace` 启动脚本直接报错:

|  |  |

| --- | --- |

| 1 | `SystemError: A debugger has been found running` `in` `your system. Please, unload it` `from` `memory` `and` `restart.` |

Python 的 `trace` 功能可以用来实现行覆盖率以及调试器等强大的功能，只是对于我们这次的目标并不适用。

类似的回调还有 `c_profilefunc` ，不过该函数不对 `line` 事件进行触发。

## LLTRACE

Python 有一个鲜为人知的特性是可以在 Debug 编译时启用底层跟踪 `LLTRACE` (即 Low Level Trace)，这也是在查看 `ceval.c` 时发现的:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25 | `next_instr` `=` `first_instr` `+` `f``-``>f_lasti` `+` `1``;`  `stack_pointer` `=` `f``-``>f_stacktop;`  `assert``(stack_pointer !``=` `NULL);`  `f``-``>f_stacktop` `=` `NULL;` `/``*` `remains NULL unless` `yield` `suspends frame` `*``/`    `#ifdef LLTRACE`  `lltrace` `=` `PyDict_GetItemString(f``-``>f_globals,` `"__lltrace__"``) !``=` `NULL;`  `#endif`  `#if defined(Py_DEBUG) || defined(LLTRACE)`  `filename` `=` `PyString_AsString(co``-``>co_filename);`  `#endif`    `why` `=` `WHY_NOT;`  `err` `=` `0``;`  `x` `=` `Py_None;` `/``*` `Not a reference, just anything non``-``NULL` `*``/`  `w` `=` `NULL;`    `if` `(throwflag) {` `/``*` `support` `for` `generator.throw()` `*``/`  `why` `=` `WHY_EXCEPTION;`  `goto on_error;`  `}`    `for` `(;;) {`  `/``/` `循环解释执行 Python 字节码`  `}` |

Low Level Trace 一方面需要编译时启用，另一方面也需要在运行时当前栈帧定义了全局变量 `__lltrace__`。

还是实践出真知，先写个简单的测试文件:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `# test.py`  `__lltrace__` `=` `1`    `def` `add(a, b):`  `return` `a` `+` `b` `-` `42`    `a` `=` `3`  `c` `=` `add(a,` `4``)` |

使用 Debug 编译的 Python 运行结果如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `$` `/``cpython_dbg``/``bin``/``python2.``7` `test.py`  `0``:` `124``,` `0`  `push` `3`  `3``:` `124``,` `1`  `push` `4`  `6``:` `23`  `pop` `4`  `7``:` `100``,` `1`  `push` `42`  `10``:` `24`  `pop` `42`  `11``:` `83`  `pop` `-``35`  `ext_pop` `4`  `ext_pop` `3`  `ext_pop <function add at` `0x7f95944a0e28``>`  `push` `-``35`  `33``:` `90``,` `3`  `pop` `-``35`  `36``:` `100``,` `4`  `push` `None`  `39``:` `83`  `pop` `None` |

打印的数字从下面的代码而来:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `if` `(lltrace) {`  `if` `(HAS_ARG(opcode)) {`  `printf(``"%d: %d, %d\n"``,`  `f``-``>f_lasti, opcode, oparg);`  `}`  `else` `{`  `printf(``"%d: %d\n"``,`  `f``-``>f_lasti, opcode);`  `}`  `}` |

其中 `push`/`pop` 相关的输出来源是如下栈追踪相关的函数:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27 | `#ifdef LLTRACE`  `static` `int`  `prtrace(PyObject` `*``v, char` `*``str``)`  `{`  `printf(``"%s "``,` `str``);`  `if` `(PyObject_Print(v, stdout,` `0``) !``=` `0``)`  `PyErr_Clear();` `/``*` `Don't know what` `else` `to do` `*``/`  `printf(``"\n"``);`  `return` `1``;`  `}`  `#define PUSH(v)         { (void)(BASIC_PUSH(v), \`  `lltrace && prtrace(TOP(),` `"push"``)); \`  `assert``(STACK_LEVEL() <``=` `co``-``>co_stacksize); }`  `#define POP()           ((void)(lltrace && prtrace(TOP(), "pop")), \`  `BASIC_POP())`  `#define STACKADJ(n)     { (void)(BASIC_STACKADJ(n), \`  `lltrace && prtrace(TOP(),` `"stackadj"``)); \`  `assert``(STACK_LEVEL() <``=` `co``-``>co_stacksize); }`  `#define EXT_POP(STACK_POINTER) ((void)(lltrace && \`  `prtrace((STACK_POINTER)[``-``1``],` `"ext_pop"``)), \`  `*``-``-``(STACK_POINTER))`  `#else`  `#define PUSH(v)                BASIC_PUSH(v)`  `#define POP()                  BASIC_POP()`  `#define STACKADJ(n)            BASIC_STACKADJ(n)`  `#define EXT_POP(STACK_POINTER) (*--(STACK_POINTER))`  `#endif` |

上面的 `lltrace` 输出可以记录每条字节码的执行，并且会打印堆栈的变化，因此在追踪和调试字节码上非常有用。

> 更多 `LLTRACE` 相关内容见: 4b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3P5i4c8Z5L8$3&6Q4x3V1k6U0M7s2W2@1K9r3!0F1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8V1#2A6M7$3y4Q4x3V1k6e0M7r3g2U0K9h3q4D9b7Y4g2A6L8r3c8K6i4K6u0W2N6s2S2@1

## Python VMP

现在有了 `LLTRACE` 的功能，但是要实现 `ether_v2.py` 的追踪还需要解决几个问题:

1. `LLTRACE` 的启用需要在当前栈帧上定义全局变量 `__lltrace__`；
2. `LLTRACE` 输出的字节码过于简略，缺乏可读性；
3. `LLTRACE` 输出的字节码是运行的代码，也就是循环展开后(flatten)的代码，进一步影响逆向分析；

所以我使用了一个简单粗暴的方法，即直接修改 CPython 源代码。首先在判断 `lltrace` 启用的地方修改判断从 `f->f_globals` 改为递归搜索 `f->f_back->f_globals`，这样只要在我们的调用栈帧定义变量即可；对于字节码的输出，最好是可以有类似 `dis` 模块的显示效果，至于平坦化的控制流，可以根据指令 `index` 再重新进行组合。

## Dynamic Trace

在 `LLTRACE` 的基础上，我们可以比较简单地修改出一版具有可读性的 Trace 代码，以下面的源码为例:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `# test.py`  `__pztrace__` `=` `1`    `def` `validate(s):`  `if` `len``(s) !``=` `4``:`  `return` `False`  `cc` `=` `0`  `for` `i` `in` `s:`  `cc ^``=` `ord``(i)`  `if` `cc` `=``=` `0``:`  `return` `True`  `return` `False`    `s` `=` `raw_input``(``'Your input: '``)`  `if` `validate(s):`  `print` `'ok'`  `else``:`  `print` `'failed'` |

其中 `__pztrace__` 是我新定义的全局跟踪触发标记，在没有源码的前提下，运行上述字节码可实时打印字节码如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `$` `/``build``/``cpython``/``build``/``bin``/``python2.``7` `test.py`  `Your` `input``: helloworld`  `=``=``=` `pztrace test.py` `=``=``=`  `0` `LOAD_GLOBAL` `0``; push <built``-``in` `function` `len``>`  `3` `LOAD_FAST` `0``; push` `'helloworld'`  `6` `CALL_FUNCTION` `1`  `ext_pop` `'helloworld'`  `ext_pop <built``-``in` `function` `len``>`  `push` `10`  `9` `LOAD_CONST` `1``; push` `4`  `12` `COMPARE_OP` `3` `(!``=``) ; pop` `4`  `15` `POP_JUMP_IF_FALSE` `22``; pop` `True`  `18` `LOAD_GLOBAL` `1``; push` `False`  `21` `RETURN_VALUE; pop` `False`  `ext_pop` `'helloworld'`  `ext_pop <function validate at` `0x7fe13a5f4ed0``>`  `push` `False`  `36` `POP_JUMP_IF_FALSE` `47``; pop` `False`  `47` `LOAD_CONST` `4``; push` `'failed'`  `50` `LOAD_BUILD_CLASS; pop` `'failed'`  `failed`  `51` `YIELD_FROM;`  `52` `LOAD_CONST` `5``; push` `None`  `55` `RETURN_VALUE; pop` `None` |

将每条字节码后对应的栈操作以及实时数据输出，更加有利于对代码的理解。从上面的字节码输出中可以基本看出实际的操作，而且打印出来的是已经执行到的分支，通过调整输入可以触达不同的分支，如下为输入 `abab` 的跟踪流程:

|  |  |

| --- | --- |
