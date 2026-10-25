
```
def __init__(self, arch, mode):
    # verify version compatibility with the core before doing anything
    (major, minor, _combined) = ks_version()
    if major != KS_API_MAJOR or minor != KS_API_MINOR:#先判断下当前版本和核心api是否匹配
        self._ksh = None
        # our binding version is different from the core's API version
        raise KsError(KS_ERR_VERSION)

    self._arch, self._mode = arch, mode
    self._ksh = c_void_p()
    status = _ks.ks_open(arch, mode, byref(self._ksh))#根据参数arch和mode通过c函数ks_open打开ks_engine，获得_ksh的引用
    if status != KS_ERR_OK:
        self._ksh = None
        raise KsError(status)

    if arch == KS_ARCH_X86:
        # Intel syntax is default for X86
        self._syntax = KS_OPT_SYNTAX_INTEL
    else:
        self._syntax = None
```

很明显，在`__init__`函数里面先通过c函数`ks_open`打开ks引擎获得一个引用，才能调用它内部的`asm`函数，在`__del__`里还会调用`ks_close`关闭

这几个函数的原型在这里

```
# setup all the function prototype
def _setup_prototype(lib, fname, restype, *argtypes):
    getattr(lib, fname).restype = restype
    getattr(lib, fname).argtypes = argtypes

kserr = c_int
ks_engine = c_void_p
ks_hook_h = c_size_t

_setup_prototype(_ks, "ks_version", c_uint, POINTER(c_int), POINTER(c_int))
_setup_prototype(_ks, "ks_arch_supported", c_bool, c_int)
_setup_prototype(_ks, "ks_open", kserr, c_uint, c_uint, POINTER(ks_engine))
_setup_prototype(_ks, "ks_close", kserr, ks_engine)
_setup_prototype(_ks, "ks_strerror", c_char_p, kserr)
_setup_prototype(_ks, "ks_errno", kserr, ks_engine)
_setup_prototype(_ks, "ks_option", kserr, ks_engine, c_int, c_void_p)
_setup_prototype(_ks, "ks_asm", c_int, ks_engine, c_char_p, c_uint64, POINTER(POINTER(c_ubyte)), POINTER(c_size_t), POINTER(c_size_t))
_setup_prototype(_ks, "ks_free", None, POINTER(c_ubyte))
```

简单食用一下

```
from keystone import *

ks = Ks(KS_ARCH_ARM, KS_MODE_ARM)
code=b"sub r1, r2, r5"
encoding, count =ks.asm(code)
print("%s = [ " % code, end='')
for i in encoding:
    print("%02x " % i, end='')
print("]")
```

可以看到汇编语句转换成了16进制

`b'sub r1, r2, r5' = [ 05 10 42 e0 ]`

总结一下，capstone 的 `disasm` 把文件的输入转换成 `CsInsn` 实例，通过 `CsInsn` 实例 可以获得汇编代码的 `address` 地址，`mnemonic` 操作码，`op_str` 操作数 等重要信息,用于分析

就是 `[ 05 10 42 e0 ]` → `sub r1, r2, r5`

keystone 的 `asm` 把汇编代码转换成输出的16进制，用于patch打包

就是 `'sub r1, r2, r5'` → `[ 05 10 42 e0 ]`

ps：原来总共就封装了俩c函数，怪不得大佬懒得写教程，我这个彩笔居然还看了半天，记录下笔记，免得下次再忘了。。。。。。

参考：

https://bbs.pediy.com/thread-205590.htm

693K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5#2y4K6f1K6

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-3-31 16:59 被挤蹭菌衣编辑，原因：

上传的附件：

- [`__init__.py`](javascript:void(0);)（36.88kb，18次下载）
- [`keystone .py`](javascript:void(0);)（8.00kb，19次下载）
- [`cs.py`](javascript:void(0);)（0.82kb，19次下载）
- [`ks.py`](javascript:void(0);)（0.20kb，18次下载）
