
从`nm`的帮助看有`--demangle`（即`-C`）选项，显示 C++ 改编前的函数名称。当时想以上查看导出函数的两个方法对于 C++ 函数是不是不能能够显示出来呢，而`98750.so`中的`tf_rkg_vavg`和`ragel_byq`这两个函数正好是 C++ 函数，也就是说`98750.so`正好是 C++ 函数。同时看到`nm`中还有个`-a`显示调试符号的选项，于是是试了一下如下输出：

```
root@speedlinux:/home/test# nm -q 98750.so
nm: invalid option -- 'q'
Usage: nm [option(s)] [file(s)]
List symbols in [file(s)] (a.out by default).
The options are:
  -a, --debug-syms       Display debugger-only symbols
  -A, --print-file-name  Print name of the input file before every symbol
  -B                     Same as --format=bsd
  -C, --demangle[=STYLE] Decode low-level symbol names into user-level names
                         The STYLE, if specified, can be `auto' (the default),
                         `gnu', `lucid', `arm', `hp', `edg', `gnu-v3', `java'
                         or `gnat'
      --no-demangle      Do not demangle low-level symbol names
  -D, --dynamic          Display dynamic symbols instead of normal symbols
      --defined-only     Display only defined symbols
  -e                     (ignored)
  -f, --format=FORMAT    Use the output format FORMAT.  FORMAT can be `bsd',
                         `sysv' or `posix'.  The default is `bsd'
  -g, --extern-only      Display only external symbols
  -l, --line-numbers     Use debugging information to find a filename and
                         line number for each symbol
  -n, --numeric-sort     Sort symbols numerically by address
  -o                     Same as -A
  -p, --no-sort          Do not sort the symbols
  -P, --portability      Same as --format=posix
  -r, --reverse-sort     Reverse the sense of the sort
      --plugin NAME      Load the specified plugin
  -S, --print-size       Print size of defined symbols
  -s, --print-armap      Include index for symbols from archive members
      --size-sort        Sort symbols by size
      --special-syms     Include special symbols in the output
      --synthetic        Display synthetic symbols as well
  -t, --radix=RADIX      Use RADIX for printing symbol values
      --target=BFDNAME   Specify the target object format as BFDNAME
  -u, --undefined-only   Display only undefined symbols
  -X 32_64               (ignored)
  @FILE                  Read options from FILE
  -h, --help             Display this information
  -V, --version          Display this program's version number
nm: supported targets: elf32-i386 a.out-i386-linux pei-i386 elf32-little elf32-big elf64-x86-64 elf32-x86-64 pei-x86-64 elf64-l1om elf64-k1om elf64-little elf64-big plugin srec symbolsrec verilog tekhex binary ihex trad-core
```

从以上输出看`-a`和`-C`确实可以显示出来，这两个函数名称也是从`gdb`和`IDA`里面看到并拷贝出来的，那么为什么`objdump -D`就显示不出来当时很是纳闷，到底跟 C++ 编译器函数名称修饰的改编有没有关系，详细查了`mangling`和`demangle`的资料其实根本没有任何关系，C++ 改编也会有规律可循的，而且经过 C++ 改编后是可以通过`c++filt`工具还原成原函数的，详细可以查阅`c++filt`的帮助说明，而且`nm --demangle`就是用来将符号中改编过的函数还原显示成原始函数的。

那么用`dlsym`去获取这个函数的地址返回结果为什么是空的，后来看了一下地址`tf_rkg_vavg`在`objdump -C`中的地址为`126b7`，然后我就用这个地址过滤，结果如下：

```
root@cjy-virtual-machine:~/test# nm -C 98750.so | cat -n | grep 126b7
   296    000126b7 T tf_rkg_vavg
root@cjy-virtual-machine:~/test# nm -D 98750.so | cat -n | grep 126b7
   218    000126b7 T gs_ext_init
```

已经很明显了，`gdb`和`IDA`里面显示的函数都是错的（其实不是错的，跟`IDA`选择解析符号顺序有关系），当时分析这个动态库的时候，还以为所有函数都加密了呢？用`gs_ext_init`名称调用`dlsym`正常调用并解密代码，到底`objdump -D`选项有什么神奇之处，可以导致`gdb`、`IDA`同一个选项地址显示的结果不一样呢？而且`gdb`对`gs_ext_init`下断点根本找不到符号。

`nm -C 98750.so | grep 126b7`这个输出为什么没有显示`gs_ext_init`这个函数呢，但是用`readelf -a`过滤就有效，如下所示：

```
root@speedlinux:/home/test# nm -C 98750.so | cat -n | grep 126b7
   296    000126b7 T tf_rkg_vavg
root@speedlinux:/home/test# readelf -a 98750.so | cat -n | grep 126b7
  1859    0001272c  00011c01 R_386_32          000126b7   gs_ext_init
  1860    00012760  00011c01 R_386_32          000126b7   gs_ext_init
  2160       284: 000126b7  1168 FUNC    GLOBAL DEFAULT   12 gs_ext_init
  2556       358: 000126b7  1168 FUNC    GLOBAL DEFAULT   12 tf_rkg_vavg
root@speedlinux:/home/test# readelf -a 98750.so | cat -n | grep tf_rkg_vavg
  2556       358: 000126b7  1168 FUNC    GLOBAL DEFAULT   12 tf_rkg_vavg
root@speedlinux:/home/test# readelf -a 98750.so | cat -n | grep gs_ext_init
  1859    0001272c  00011c01 R_386_32          000126b7   gs_ext_init
  1860    00012760  00011c01 R_386_32          000126b7   gs_ext_init
  2160       284: 000126b7  1168 FUNC    GLOBAL DEFAULT   12 gs_ext_init
```

```
readelf -a 98750.so | cat -n >readelf_a_98750.txt
```

通过查看`readelf -a`输出上下文，确认：

```
Line 1859:   1859    0001272c  00011c01 R_386_32          000126b7   gs_ext_init
Line 1860:   1860    00012760  00011c01 R_386_32          000126b7   gs_ext_init
```

这两个包含`gs_ext_init`的关键字出现在动态库符号的重定位节`.rel.dyn`中，节开头内容如下：

```
   102    Relocation section '.rel.dyn' at offset 0x3a70 contains 1761 entries:
   103     Offset     Info    Type            Sym.Value  Sym. Name
   104    00007781  00000008 R_386_RELATIVE
   105    000077b3  00000008 R_386_RELATIVE
```

。。。。

而

```
   2160       284: 000126b7  1168 FUNC    GLOBAL DEFAULT   12 gs_ext_init
```

出现在动态库符号节`.dynsym`中，该节的开头内容如下：

```
  1874    Symbol table '.dynsym' contains 319 entries:
  1875       Num:    Value  Size Type    Bind   Vis      Ndx Name
  1876         0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
```
