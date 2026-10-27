
# `_PyEval_EvalFrameDefault`

最终来到执行的函数`_PyEval_EvalFrameDefault`，该函数大约有 **3000** 行 C 代码，并且其中大量使用了宏来加速运算。前面说过 Python 字节码是基于栈的，这里的 Frame 就是指代某个栈帧，也就是当前执行流的上下文。栈帧中包括字节码、全局变量、本地变量等信息，如下所示:

```c
struct _frame {
PyObject_VAR_HEAD
struct _frame *f_back; /* previous frame, or NULL */
PyCodeObject *f_code; /* code segment */
PyObject *f_builtins; /* builtin symbol table (PyDictObject) */
PyObject *f_globals; /* global symbol table (PyDictObject) */
PyObject *f_locals; /* local symbol table (any mapping) */
PyObject **f_valuestack; /* points after the last local */
PyObject *f_trace; /* Trace function */
int f_stackdepth; /* Depth of value stack */
char f_trace_lines; /* Emit per-line trace events? */
char f_trace_opcodes; /* Emit per-opcode trace events? */
/* Borrowed reference to a generator, or NULL */
PyObject *f_gen;
int f_lasti; /* Last instruction if called */
/* Call PyFrame_GetLineNumber() instead of reading this field
directly. As of 2.3 f_lineno is only valid when tracing is
active (i.e. when f_trace is set). At other times we use
PyCode_Addr2Line to calculate the line from the current
bytecode index. */
int f_lineno; /* Current line number */
int f_iblock; /* index in f_blockstack */
PyFrameState f_state; /* What state the frame is in */
PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
PyObject *f_localsplus[1]; /* locals+stack, dynamically sized */
};
```

这里的`PyCodeObject`就是字节码对象，和`dis`模块中的对象类似:

```c
/* Bytecode object */
struct PyCodeObject {
PyObject_HEAD
int co_argcount; /* #arguments, except *args */
int co_posonlyargcount; /* #positional only arguments */
int co_kwonlyargcount; /* #keyword only arguments */
int co_nlocals; /* #local variables */
int co_stacksize; /* #entries needed for evaluation stack */
int co_flags; /* CO_..., see below */
int co_firstlineno; /* first source line number */
PyObject *co_code; /* instruction opcodes */
PyObject *co_consts; /* list (constants used) */
PyObject *co_names; /* list of strings (names used) */
PyObject *co_varnames; /* tuple of strings (local variable names) */
PyObject *co_freevars; /* tuple of strings (free variable names) */
PyObject *co_cellvars; /* tuple of strings (cell variable names) */
/* The rest aren't used in either hash or comparisons, except for co_name,
used in both. This is done to preserve the name and line number
for tracebacks and debuggers; otherwise, constant de-duplication
would collapse identical functions/lambdas defined on different lines.
*/
Py_ssize_t *co_cell2arg; /* Maps cell vars which are arguments. */
PyObject *co_filename; /* unicode (where it was loaded from) */
PyObject *co_name; /* unicode (name, for reference) */
PyObject *co_lnotab; /* string (encoding addr<->lineno mapping) See
Objects/lnotab_notes.txt for details. */
// ...
}
```

回到(默认的)`eval_frame`函数，抽取一些关键部分如下:

```c
#define JUMPTO(x) (next_instr = first_instr + (x) / sizeof(_Py_CODEUNIT))
PyObject* _Py_HOT_FUNCTION
_PyEval_EvalFrameDefault(PyThreadState *tstate, PyFrameObject *f, int throwflag)
{
//...
if (tstate->use_tracing) {
if (tstate->c_tracefunc != NULL) {
if (call_trace_protected(tstate->c_tracefunc,
tstate->c_traceobj,
tstate, f, PyTrace_CALL, Py_None)) {
/* Trace function raised an error */
goto exit_eval_frame;
}
}
}
// ...
first_instr = (_Py_CODEUNIT *) PyBytes_AS_STRING(co->co_code);
next_instr = first_instr;
// ...
main_loop:
for (;;) {
assert(stack_pointer >= f->f_valuestack); /* else underflow */
assert(STACK_LEVEL() <= co->co_stacksize); /* else overflow */
assert(!_PyErr_Occurred(tstate));
fast_next_opcode:
if (PyDTrace_LINE_ENABLED())
maybe_dtrace_line(f, &instr_lb, &instr_ub, &instr_prev);
/* line-by-line tracing support */
if (trace...) {
err = maybe_call_line_trace(tstate->c_tracefunc,
tstate->c_traceobj,
tstate, f,
&instr_lb, &instr_ub, &instr_prev);
}
dispatch_opcode:
// ...
switch (opcode) {
case TARGET(NOP): {
FAST_DISPATCH();
}
case TARGET(LOAD_FAST): {/*...*/}
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}
case TARGET(STORE_FAST): {/*...*/}
case TARGET(POP_TOP): {/*...*/}
// ...
case TARGET(BINARY_MULTIPLY): {
PyObject *right = POP();
PyObject *left = TOP();
PyObject *res = PyNumber_Multiply(left, right);
Py_DECREF(left);
Py_DECREF(right);
SET_TOP(res);
if (res == NULL)
goto error;
DISPATCH();
}
// ...
case TARGET(JUMP_ABSOLUTE): {
PREDICTED(JUMP_ABSOLUTE);
JUMPTO(oparg);
#if FAST_LOOPS
/* Enabling this path speeds-up all while and for-loops by bypassing
the per-loop checks for signals. By default, this should be turned-off
because it prevents detection of a control-break in tight loops like
"while 1: pass". Compile with this option turned-on when you need
the speed-up and do not need break checking inside tight loops (ones
that contain only instructions ending with FAST_DISPATCH).
*/
FAST_DISPATCH();
#else
DISPATCH();
#endif
}
// ...
case TARGET(EXTENDED_ARG): {
int oldoparg = oparg;
NEXTOPARG();
oparg |= oldoparg << 8;
goto dispatch_opcode;
}
// ...
// switch end
}
/* This should never be reached. Every opcode should end with DISPATCH()
or goto error. */
Py_UNREACHABLE();
error:
// ...
exception_unwind:
// ...
exiting:
// ...
}
}
/* pop frame */
exit_eval_frame:
// ...
return _Py_CheckFunctionResult(tstate, NULL, retval, __func__);
}
```

大部分的代码是对字节码中的 `opcode` 进行 `switch/case` 处理，上面截取了几个提到的字节码，比如 `LOAD_CONST`、`JUMP_ABSOLUTE`、`BINARY_MULTIPLY`、`EXTENDED_ARG` 等，根据代码的执行流程大概知道了 Python 解释器如何对这些字节码进行理解。

## c_tracefunc

在 `switch` 语句之前有部分代码值得注意，即关于`c_tracefunc`的处理。从代码中看出，Python实际上内置了追踪字节码的功能。我们可以使用 `sys.settrace` 来设置跟踪函数，下面是一个简单的例子:

```python
#!/usr/bin/env python2
import sys
import dis
def func(a, b):
c = a + b
return c * 10
co = func.__code__
dis.disassemble(co)
def mytrace(frame, why, arg):
print "Trace", frame, why, arg
return mytrace
print "=== Trace Start ==="
sys.settrace(mytrace)
func(3, 4)
```

输出如下:

```bash
$ ./demo.py
6 0 LOAD_FAST 0 (a)
3 LOAD_FAST 1 (b)
6 BINARY_ADD
7 STORE_FAST 2 (c)
7 10 LOAD_FAST 2 (c)
13 LOAD_CONST 1 (10)
16 BINARY_MULTIPLY
17 RETURN_VALUE
=== Trace Start ===
Trace <frame object at 0x10b8cb218> call None
Trace <frame object at 0x10b8cb218> line None
Trace <frame object at 0x10b8cb218> line None
Trace <frame object at 0x10b8cb218> return 70
Trace <frame object at 0x10b98c050> call None
Trace <frame object at 0x10b98c050> call None
```

Python 的标准库中也提供了 [trace 模块][trace] 来支持字节码跟踪，查看该模块的的源码发现实际上也是用了 `sys.settrace` 或者 `threading.settrace` 来设置跟踪回调。

不过，使用 sys.trace 并不是每条指令都跟踪的，只针对特定事件进行跟踪:

- `call`: 函数调用
- `return`: 函数返回
- `line`: 一行新代码
- `exception`: 异常事件

而且该代码中也做了对应的防护，使用 trace 启动脚本直接报错:

```sh
SystemError: A debugger has been found running in your system. Please, unload it from memory and restart.
```

Python 的 trace 功能可以用来实现行覆盖率以及调试器等强大的功能，只是对于我们这次的目标并不适用。

类似的回调还有 `c_profilefunc` ，不过该函数不对 `line` 事件进行触发。

## LLTRACE

Python 有一个鲜为人知的特性是可以在 Debug 编译时启用底层跟踪 `LLTRACE` 即 Low Level Trace，这也是在查看 `ceval.c` 时发现的:

```c
next_instr = first_instr + f->f_lasti + 1;
stack_pointer = f->f_stacktop;
assert(stack_pointer != NULL);
f->f_stacktop = NULL; /* remains NULL unless yield suspends frame */
#ifdef LLTRACE
lltrace = PyDict_GetItemString(f->f_globals, "__lltrace__") != NULL;
#endif
#if defined(Py_DEBUG) || defined(LLTRACE)
filename = PyString_AsString(co->co_filename);
#endif
why = WHY_NOT;
err = 0;
x = Py_None; /* Not a reference, just anything non-NULL */
w = NULL;
if (throwflag) { /* support for generator.throw() */
why = WHY_EXCEPTION;
goto on_error;
}
for (;;) {
// 循环解释执行 Python 字节码
}
```

Low Level Trace 一方面需要编译时启用，另一方面也需要在运行时当前栈帧定义了全局变量`__lltrace__`。

还是实践出真知，先写个简单的测试文件:
