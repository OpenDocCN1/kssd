
# [原创]Relocate、PLT、GOT And Lazy Binding

本篇文章将会尝试解释重定位，PLT，GOT以及延迟绑定，并且通过一个简单的例子动态调试追踪延迟绑定的过程。

## 1 Relocate

> Relocation is the process of connecting symbolic references with symbolic definitions. Relocatable files must have information that describes how to modify their section contents, thus allowing executable and shared object files to hold the right information for a process's program image. Relocation entries are these data.

即：
- **重定位** 是将符号定义和符号引用进行连接的过程。
- **重定位文件** 需要提供一些描述如何修改section内容的相关信息，从而保证可执行文件和共享目标文件能够在程序镜像中存储正确的信息。

以32位系统为例（为了便于理解，本文后续内容均会以32位系统场景描述），其关键的数据结构如下：

```c
// 隐式加法重定位，重定位处的汇编一般会是 e8 fc ff ff ff
typedef struct {
Elf32_Addr r_offset;
uint32_t r_info;
} Elf32_Rel;
// 显式加法重定位，重定位时需要计算的加数存储在 r_addend
typedef struct {
Elf32_Addr r_offset;
uint32_t r_info;
int32_t r_addend;
} Elf32_Rela;
```

- `r_offset`指向了需要重定位的位置。
- `r_info`存储了需要重定位符号表的索引和重定位类型。
- `r_addend` 用于计算存储在可定位字段中的值。

在程序加载时，会通过自己的`.rel` section，告诉连接器需要重定位的位置，在后续会详细的展示一个32位程序重定位的过程。

## 2 PLT &GOT

像上面那种在程序加载时，通过`.rel` section，让编译器基于重定位信息计算出调用函数在程序中的实际位置的加载方式，一般被称为**静态链接**，如果程序使用了外部的库函数时，整个库函数都会被直接编译到程序中。

可以思考一下它的缺点，以及对应的改正方法：
- 在一段只输出`hello world`的程序中，采用静态链接，需要将整个`glibc`链接到程序中，如果500个程序都需要使用`glibc`中的函数，那么`glibc`就会被封装进500个程序中。
- 可不可以将多个程序都会使用的库单独剥离出来，同时在源程序和库之间建立某种联系，确保源程序在执行的时候可以调用到库函数？

**动态链接**技术的提出就是为了解决这个问题，在程序运行时，将共享库和程序本身进行链接，同时，内存里的程序可以共享同一个库文件，这样既节省了硬盘存储空间，同样节省了内存空间。

静态链接与动态链接主要区别如下图所示。（本图参照《CTF竞赛权威指南》所画）

![](upload/attach/202303/882195_Z3JZAF5BFFF8SHX.png)

为了做到动态编译，首先需要生成**位置无关代码（Posistion-Independent Code,PIC）**,通过`PIC`一个共享库可以被多个进程共享。

同时想要完成动态链接在源程序中还需要有：
1. 一个用来存储外部函数地址的数据段
2. 一段用来加载外部函数的代码

因为数据段和代码段之间的距离是一个运行时常量，他们之间的偏移是固定的，于是这里就有了全局偏移表（`GOT`，Global Offset Table），它位于数据段的开始，用于保存全局变量以及库函数（外部函数）的引用，每一条8个字节，在程序加载时会完成重定位，并填入符号的绝对地址。`GOT`一般被拆成了两个section，不需要延迟绑定，用于存储全局变量，加载到内存中只需要被读取的`.got`，以及为了存储库函数需要延迟绑定写入的`.got.plt`。

而同时为了完成延迟绑定还需要将外部函数的值在运行时写入`.got.plt`，因此又引入了过程链接表（`PLT`,Procedure Liknage Table）。`PLT`是由代码片段组成，用于将地址无关函数转移到绝对地址。每一个被调用的库函数，都会映射到一组`PLT` 和 `GOT`。如下图所示：

![](upload/attach/202303/882195_J9GTGB575JAWDQZ.png)

## 3 Lazying Binding

**Lazy Binding**，即延迟绑定，指的是只有当函数被调用的时候才进行函数绑定，这种方式加快的程序的启动速度。

为了完成延迟绑定的过程，`PLT`和`GOT`需要配合完成一些事情。

假设程序中存在一个`puts`函数被调用，在`PLT`中的表项为`puts@plt`，在`GOT`中表项为`puts@got.plt`。为了完成延迟绑定，在第一次执行的时候会完成以下事情：
1. 源程序中`call puts@plt`.
2. `PLT`中对应表项存储的内容是`jmp puts@GOT`，因此会跳进到`puts@got.plt`。
3. 在绑定之前，`puts@got.plt`位置存储还是`puts@plt`表项的第二指令的地址，因此会跳到`PLT`执行第二条指令`push n`,这里的`n`指的是`puts`函数在`GOT`中的位置，这里的入栈操作是为了找到`puts`函数的符号名，以及`puts`函数在`GOT`表项中所占的位置，便于后续将函数的实际地址写入`GOT`表。
4. 执行`PLT`表项第三条指令`jmp PLT[0]`,`PLT[0]`首先将`GOT[1]`入栈，然后`jmp GOT[2]`.
5. `GOT[2]`保存的是`_dl_runtime_resolve()`函数的入口地址，因此这里实际上就是调用`_dl_runtime_resolve()`函数，完成符号解析和重定位工作，并将`puts()`函数的真实地址写入`puts@got.plt`。然后将控制权交给`puts()`函数。

在后续调用，就会直接到`GOT`表项取得`puts`函数的地址。

整理一下，`PLT`和`GOT`中存在着通用的指令和后续的指令如下所示：

```
PLT[0]: push GOT[1]
jmp GOT[2]
PLT[1]: __libc_start_main()
PLT[2]: jmp GOT[4]
push 0
jmp PLT[0]
...
GOT[0]: .dynmic 地址
GOT[1]: relor
GOT[2]: _dl_runtime_resolve()
GOT[3]: sys startup # 针对不同环境此处内容可能不同
GOT[4]: PLT[2]第二条指令地址
```

整个过程草图如下所示

![](upload/attach/202303/882195_ZRF9YY9S4QSSW3X.png)

## 4 Quiz

环境：

```
gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0
Linux null 5.15.0-67-generic #74-Ubuntu SMP Wed Feb 22 14:14:39 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

源代码：

```c
#include <stdio.h>
int print_hello() {
printf("hello PLT and GOT\n");
}
int main() {
print_hello();
return 0;
}
```

编译：

```bash
gcc main.c -o test -save-temps -m32 -g -Wl,-z,lazy
```

- `-save-temps` 会保存所有的中间输出结果。
- `-m32` 表示输出的是32程序。
- `-Wl,-z,lazy`强制开启延迟绑定。
- `-g` 方便调试。

通过`objdump`查看目标文件汇编代码：

```
objdump -M intel -d main.o
```

`print_hello`函数如下：

```
00000000 <print_hello>:
0: 53 push ebx
1: 83 ec 14 sub esp,0x14
4: e8 fc ff ff ff call 5 <print_hello+0x5>
9: 81 c3 02 00 00 00 add ebx,0x2
f: 8d 83 00 00 00 00 lea eax,[ebx+0x0]
15: 50 push eax
16: e8 fc ff ff ff call 17 <print_hello+0x17>
1b: 83 c4 18 add esp,0x18
1e: 5b pop ebx
1f: c3 ret
```

`main` 函数如下：

```
00000024 <main>:
20: 55 push ebp
21: 89 e5 mov ebp,esp
23: 83 e4 f0 and esp,0xfffffff0
26: e8 fc ff ff ff call 27 <main+0x7>
2b: b8 00 00 00 00 mov eax,0x0
30: c9 leave
31: c3 ret
```

通过`objdump`查看最终汇编代码：

```
objdump -M intel -d test
```

`print_hello`函数如下：

```
0000119d <print_hello>:
119d: 53 push ebx
119e: 83 ec 14 sub esp,0x14
11a1: e8 fa fe ff ff call 10a0 <__x86.get_pc_thunk.bx>
11a6: 81 c3 32 2e 00 00 add ebx,0x2e32
11ac: 8d 83 30 e0 ff ff lea eax,[ebx-0x1fd0]
11b2: 50 push eax
11b3: e8 98 fe ff ff call 1050 <puts@plt>
11b8: 83 c4 18 add esp,0x18
11bb: 5b pop ebx
11bc: c3 ret
```

`main`函数如下：

```
000011bd <main>:
11bd: 55 push ebp
11be: 89 e5 mov ebp,esp
11c0: 83 e4 f0 and esp,0xfffffff0
11c3: e8 d5 ff ff ff call 119d <print_hello>
11c8: b8 00 00 00 00 mov eax,0x0
11cd: c9 leave
11ce: c3 ret
```

### 4.1 Relocate

对比目标代码和最终代码的反汇编代码，可以看到，在`print_hello`函数中，调用`call`函数时对应内容是有经过重定向的，如下图所示。

![](upload/attach/202303/882195_VBG7WKXMT8KMR6E.png)

在目标代码中：

```
4: e8 fc ff ff ff call 5 <print_hello+0x5>
16: e8 fc ff ff ff call 17 <print_hello+0x17>
```

![](upload/attach/202303/882195_3WUDNCW7P2J2FJN.png)

查看对应重定位表，可以看到：

offset 为$5$ 的符号是 `__x86.get_pc_thunk.bx`, offset为 $17$的符号是`puts`。

而在最终代码中：

```
11a1: e8 fa fe ff ff call 10a0 <__x86.get_pc_thunk.bx>
11b3: e8 98 fe ff ff call 1050 <puts@plt>
```

查看编译之后程序的符号：

```
readelf -s test
20: 000010a0 4 FUNC GLOBAL HIDDEN 14 __x86.get_pc_thunk.bx
27: 00000000 0 FUNC GLOBAL DEFAULT UND puts@GLIBC_2.0
```

对于 `__x86.get_pc_thunk.bx`, 采用的是`S + A - P`的方式计算偏移:
- S 是符号的值
- A 是重定位条目中的加数，在显示加数中，会使用`ElfN_REL`中的值，而隐式的则是在`call`指令后面。
- P 是要进行重定位的存储单位的地址。

即：`__x86.get_pc_thunk.bx`: $0x10a0 + 0xffff fffc -0x11a2 = 0xffff fefa$

即重定位之后的调用：

![](upload/attach/202303/882195_CYB WQ53FUUE9BEJ.png)

`main`函数中的`print_hello`函数调用同理，不再赘述。

那么还剩下一个问题，`puts`函数此时在符号表中的值是0，又应该怎么计算呢？

### 4.2 Lazy Binding

前面有讲到，Linux下符号动态链接默认采用的是延迟绑定的方式，也就是说，在程序运行时用到改符号的时候才会去解析它的地址（值）。

开始动态调试，启动后在`puts@plt`下断点:

![](upload/attach/202303/882195_FKVNM5F9QRX2DMJ.png)

可以看到`PLT`表的内容和上述所说的基本吻合，首先是一个`jump`指令，此时根据上面信息可以得知`ebx`寄存器存储的是`GOT`表地址，所以这条`jump`指令就跳转到`GOT`表中。第二条`push` 序号入栈，第三条`jump` 指令调转到的位置就是`PLT[0]`。

![](upload/attach/202303/882195_TA2Z2W8RZ8AB9C2.png)

查看`jump`地址的信息：

![](upload/attach/202303/882195_HU8TNNTN4M7BQS2.png)

跳入的是`.got.plt`且表现值存储的值是`0x56556056`即刚才的`push`指令。

继续查看`0x56556030`处的内容，`ebx`存储的仍然是`GOT`表的地址，因此此处指令仍然是上面所描述的`push GOT[1] jmp GOT[2]`。

![](upload/attach/202303/882195_2CWUKNW9337PKHY.png)

继续调试，发现他进入的即是`_dl_runtime_resolve()`函数。

![](upload/attach/202303/882195_Z2BGFYRJ6BEXWJY.png)

到这里有关`PLT`，`GOT`延迟绑定的整个过程已经基本梳理完毕，在后续有关更多`PLT`，`GOT`攻击利用的时候，再来详细讨论`_dl_runtime_resolve()`函数。

## 5 彩蛋: What ChatGPT Will Say About `_dl_runtime_resolve()`

好的，下面我们结合源码分析一下`_dl_runtime_resolve()`函数的实现。

在GNU C库的源码中，`_dl_runtime_resolve()`函数的定义位于`elf/dl-runtime.c`文件中。该函数的主要作用是在程序运行期间解析动态链接函数。

函数的定义如下：

```c
ElfW(Addr) _dl_runtime_resolve (ElfW(Addr) *ref)
{
/* ... */
}
```

其中，`ElfW(Addr)`是一个宏定义，根据编译时指定的目标架构不同而不同。例如，在32位x86架构上，它被定义为`unsigned int`。

函数中的`ref`参数是一个指向函数指针的指针，它指向需要被解析的动态链接函数的地址。

函数的实现过程如下：
1. 首先，从`ref`参数指向的地址中读取一个指针，这个指针指向了一个`struct link_map`类型的结构体，该结构体记录了当前共享对象（共享库）的信息，包括共享对象的地址范围、依赖的其他共享对象等。

2.

```c
struct link_map *l;
ElfW(Addr) result;
l = ((struct link_map *) D_PTR (ref[1], l_info[DT_PLTGOT]))->l_prev;
```

这里的`D_PTR()`宏是GNU C库中的一个宏定义，用于读取动态链接器内部数据结构中的字段。`DT_PLTGOT`表示动态链接器中`GOT`表的地址。

3. 然后，从`ref`参数指向的地址中读取一个偏移量，该偏移量是一个相对于共享对象基地址的偏移量，表示需要解析的动态链接函数在共享对象中的位置。

```c
ElfW(Addr) reloc_arg = D_PTR (ref[1], l_info[DT_RELA]->d_un.d_ptr)
+ sizeof (ElfW(Rela));
```

这里的`DT_RELA`表示动态链接器中重定位表的地址，通过它可以找到需要解析的动态链接函数的重定位项。

4. 接着，通过解析重定位项，获取目标函数的地址。

```c
result = _dl_fixup (l, ref[0], reloc_arg, 0);
```

其中，`_dl_fixup()`函数是用于解析重定位项的函数，它会根据重定位项中的信息，计算出目标函数的地址，并返回给调用者。

5. 最后，将目标函数的地址返回给调用者，完成动态链接函数的解析过程。

```c
return result;
```
