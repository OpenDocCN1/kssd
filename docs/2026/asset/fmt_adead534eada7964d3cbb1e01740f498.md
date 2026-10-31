
那么还剩下一个问题，`puts`函数此时在符号表中的值是0，又应该怎么计算呢？

## 4.2 Lazy Binding

前面有讲到，Linux下符号动态链接默认采用的是延迟绑定的方式，也就是说，在程序运行时用到该符号的时候才会去解析它的地址（值）。

开始动态调试，启动后在`puts@plt`下断点:

![](https://bbs.kanxue.com/upload/attach/202303/882195_FKVNM5F9QRX2DMJ.png)

可以看到`PLT`表的内容和上述所说的基本吻合，首先是一个`jump`指令，此时根据上面信息可以得知`ebx`寄存器存储的是`GOT`表地址，所以这条`jump`指令就跳转到`GOT`表中。第二条`push`序号入栈，第三条`jump`指令跳转到的位置就是`PLT[0]`。

![](https://bbs.kanxue.com/upload/attach/202303/882195_TA2Z2W8RZ8AB9C2.png)

查看`jump`地址的信息：

![](https://bbs.kanxue.com/upload/attach/202303/882195_HU8TNNTN4M7BQS2.png)

跳入的是`.got.plt`且该位置存储的值是`0x56556056`即刚才的`push`指令。

继续查看`0x56556030`处的内容，`ebx`存储的仍然是`GOT`表的地址，因此此处指令仍然是上面所描述的`push GOT[1] jmp GOT[2]`。

![](https://bbs.kanxue.com/upload/attach/202303/882195_2CWUKNW9337PKHY.png)

继续调试，发现它进入的即是`_dl_runtime_resolve()`函数。

![](https://bbs.kanxue.com/upload/attach/202303/882195_Z2BGFYRJ6BEXWJY.png)

到这里有关`PLT`、`GOT`延迟绑定的整个过程已经基本梳理完毕，在后续有关更多`PLT`、`GOT`攻击利用的时候，再来详细讨论`_dl_runtime_resolve()`函数。

## 5 彩蛋: What ChatGPT Will Say About `_dl_runtime_resolve()`

好的，下面我们结合源码分析一下`_dl_runtime_resolve()`函数的实现。

在GNU C库的源码中，`_dl_runtime_resolve()`函数的定义位于`elf/dl-runtime.c`文件中。该函数的主要作用是在程序运行期间解析动态链接函数。

函数的定义如下：

```c
ElfW(Addr) _dl_runtime_resolve (ElfW(Addr) *ref)
{
  /*
  ...
  */
}
```

其中，`ElfW(Addr)`是一个宏定义，根据编译时指定的目标架构不同而不同。例如，在32位x86架构上，它被定义为`unsigned int`。

函数中的`ref`参数是一个指向函数指针的指针，它指向需要被解析的动态链接函数的地址。

函数的实现过程如下：

1. 首先，从`ref`参数指向的地址中读取一个指针，这个指针指向了一个`struct link_map`类型的结构体，该结构体记录了当前共享对象（共享库）的信息，包括共享对象的地址范围、依赖的其他共享对象等。

```c
struct link_map *l;
ElfW(Addr) result;
l = ((struct link_map *) D_PTR (ref[1], l_info[DT_PLTGOT]))->l_prev;
```

这里的`D_PTR()`宏是GNU C库中的一个宏定义，用于读取动态链接器内部数据结构中的字段。`DT_PLTGOT`表示动态链接器中`GOT`表的地址。

2. 然后，从`ref`参数指向的地址中读取一个偏移量，该偏移量是一个相对于共享对象基地址的偏移量，表示需要解析的动态链接函数在共享对象中的位置。

```c
ElfW(Addr) reloc_arg = D_PTR (ref[1], l_info[DT_RELA]->d_un.d_ptr) + sizeof (ElfW(Rela));
```

这里的`DT_RELA`表示动态链接器中重定位表的地址，通过它可以找到需要解析的动态链接函数的重定位项。

3. 接着，通过解析重定位项，获取目标函数的地址。

```c
result = _dl_fixup (l, ref[0], reloc_arg, 0);
```

其中，`_dl_fixup()`函数是用于解析重定位项的函数，它会根据重定位项中的信息，计算出目标函数的地址，并返回给调用者。

4. 最后，将目标函数的地址返回给调用者，完成动态链接函数的解析过程。

```c
return result;
```

综上所述，`_dl_runtime_resolve()`函数的实现过程中，主要涉及了动态链接器的内部数据结构和重定位表的解析。它是实现动态链接的核心函数之一，可以在程序运行期间解析动态链接函数，实现代码的灵活性和可重用性。
