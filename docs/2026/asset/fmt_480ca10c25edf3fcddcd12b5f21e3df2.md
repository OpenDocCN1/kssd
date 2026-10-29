
```
caikelun@debian:~$ arm-linux-androideabi-readelf -d ./libtest.so 



Dynamic section at offset 0x2e48 contains 30 entries:

  Tag        Type                         Name/Value

 0x00000003 (PLTGOT)                     0x3f7c

 0x00000002 (PLTRELSZ)                   240 (bytes)

 0x00000017 (JMPREL)                     0xcb8

 0x00000014 (PLTREL)                     REL

 0x00000011 (REL)                        0xc78

 0x00000012 (RELSZ)                      64 (bytes)

 0x00000013 (RELENT)                     8 (bytes)

 0x6ffffffa (RELCOUNT)                   3

 0x00000006 (SYMTAB)                     0x1f0

 0x0000000b (SYMENT)                     16 (bytes)

 0x00000005 (STRTAB)                     0x590

 0x0000000a (STRSZ)                      1201 (bytes)

 0x00000004 (HASH)                       0xa44

 0x00000001 (NEEDED)                     Shared library: [libc.so]

 0x00000001 (NEEDED)                     Shared library: [libm.so]

 0x00000001 (NEEDED)                     Shared library: [libstdc++.so]

 0x00000001 (NEEDED)                     Shared library: [libdl.so]

 0x0000000e (SONAME)                     Library soname: [libtest.so]

 0x0000001a (FINI_ARRAY)                 0x3e3c

 0x0000001c (FINI_ARRAYSZ)               8 (bytes)

 0x00000019 (INIT_ARRAY)                 0x3e44

 0x0000001b (INIT_ARRAYSZ)               4 (bytes)

 0x0000001e (FLAGS)                      BIND_NOW

 0x6ffffffb (FLAGS_1)                    Flags: NOW

 0x6ffffff0 (VERSYM)                     0xbc8

 0x6ffffffc (VERDEF)                     0xc3c

 0x6ffffffd (VERDEFNUM)                  1

 0x6ffffffe (VERNEED)                    0xc58

 0x6fffffff (VERNEEDNUM)                 1

 0x00000000 (NULL)                       0x0

```

## 动态链接器（linker）

安卓中的动态链接器程序是 `linker`。源码在 [这里](https://bbs.kanxue.com/elink@da9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6T1K9h3!0F1K9h3y4Q4x3V1k6Q4x3V1u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6D9K9h3&6C8k6i4u0Q4x3V1j5`.)。

动态链接（比如执行 `dlopen`）的大致步骤是：

1. 检查已加载的 ELF 列表。（如果 `libtest.so` 已经加载，就不再重复加载了，仅把 `libtest.so` 的引用计数加一，然后直接返回。）

2. 从 `libtest.so` 的 `.dynamic` section 中读取 `libtest.so` 的外部依赖的 ELF 列表，从此列表中剔除已加载的 ELF，最后得到本次需要加载的 ELF 完整列表（包括 `libtest.so` 自身）。

3. 逐个加载列表中的 ELF。加载步骤：

   - 用 `mmap` 预留一块足够大的内存，用于后续映射 ELF。（`MAP_PRIVATE` 方式）

   - 读 ELF 的 PHT，用 `mmap` 把所有类型为 `PT_LOAD` 的 segment 依次映射到内存中。

   - 从 `.dynamic` segment 中读取各信息项，主要是各个 section 的虚拟内存相对地址，然后计算并保存各个 section 的虚拟内存绝对地址。

   - 执行重定位操作（`relocate`），这是最关键的一步。重定位信息可能存在于下面的一个或多个 secion 中：`.rel.plt`, `.rela.plt`, `.rel.dyn`, `.rela.dyn`, `.rel.android`, `.rela.android`。动态链接器需要逐个处理这些 `.relxxx` section 中的重定位诉求。根据已加载的 ELF 的信息，动态链接器查找所需符号的地址（比如 `libtest.so` 的符号 `malloc`），找到后，将地址值填入 `.relxxx` 中指明的**目标地址**中，这些“**目标地址**”一般存在于`.got` 或 `.data` 中。

   - ELF 的引用计数加一。

4. 逐个调用列表中 ELF 的构造函数（`constructor`），这些构造函数的地址是之前从 `.dynamic` segment 中读取到的（类型为 `DT_INIT` 和 `DT_INIT_ARRAY`）。各 ELF 的构造函数是按照依赖关系逐层调用的，先调用被依赖 ELF 的构造函数，最后调用 `libtest.so` 自己的构造函数。（ELF 也可以定义自己的析构函数（`destructor`），在 ELF 被 unload 的时候会被自动调用）

等一下！我们似乎发现了什么！再看一遍重定位操作（`relocate`）的部分。难道我们只要从这些 `.relxxx` 中获取到“**目标地址**”，然后在“**目标地址**”中重新填上一个新的函数地址，这样就完成 hook 了吗？也许吧。

## 追踪

静态分析验证一下还是很容易的。以 `armeabi-v7a` 架构的 `libtest.so` 为例。先看一下 `say_hello` 函数对应的汇编代码吧。

```
caikelun@debian:~/$ arm-linux-androideabi-readelf -s ./libtest.so



Symbol table '.dynsym' contains 58 entries:

   Num:    Value  Size Type    Bind   Vis      Ndx Name

     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND 

     1: 00000000     0 FUNC    GLOBAL DEFAULT  UND __cxa_finalize@LIBC (2)

     2: 00000000     0 FUNC    GLOBAL DEFAULT  UND snprintf@LIBC (2)

     3: 00000000     0 FUNC    GLOBAL DEFAULT  UND malloc@LIBC (2)

     4: 00000000     0 FUNC    GLOBAL DEFAULT  UND __cxa_atexit@LIBC (2)

     5: 00000000     0 FUNC    GLOBAL DEFAULT  UND printf@LIBC (2)

     6: 00000f61    60 FUNC    GLOBAL DEFAULT   12 say_hello

...............

...............

```

找到了！`say_hello` 在地址 `f61`，对应的汇编指令体积为 `60`（10 进制）字节。用 `objdump` 查看 `say_hello` 的反汇编输出。

```
caikelun@debian:~$ arm-linux-androideabi-objdump -D ./libtest.so

...............

...............

00000f60 <say_hello@@Base>:

     f60:   b5b0        push    {r4, r5, r7, lr}

     f62:   af02        add r7, sp, #8

     f64:   f44f 6080   mov.w   r0, #1024   ; 0x400

     f68:   f7ff ef34   blx dd4 <malloc@plt>

     f6c:   4604        mov r4, r0

     f6e:   b16c        cbz r4, f8c <say_hello@@Base+0x2c>

     f70:   a507        add r5, pc, #28 ; (adr r5, f90 <say_hello@@Base+0x30>)

     f72:   a308        add r3, pc, #32 ; (adr r3, f94 <say_hello@@Base+0x34>)

     f74:   4620        mov r0, r4

     f76:   f44f 6180   mov.w   r1, #1024   ; 0x400

     f7a:   462a        mov r2, r5

     f7c:   f7ff ef30   blx de0 <snprintf@plt>

     f80:   4628        mov r0, r5

     f82:   4621        mov r1, r4

     f84:   e8bd 40b0   ldmia.w sp!, {r4, r5, r7, lr}

     f88:   f001 ba96   b.w 24b8 <_Unwind_GetTextRelBase@@Base+0x8>

     f8c:   bdb0        pop {r4, r5, r7, pc}

     f8e:   bf00        nop

     f90:   7325        strb    r5, [r4, #12]

     f92:   0000        movs    r0, r0

     f94:   6568        str r0, [r5, #84]   ; 0x54

     f96:   6c6c        ldr r4, [r5, #68]   ; 0x44

     f98:   0a6f        lsrs    r7, r5, #9

     f9a:   0000        movs    r0, r0

...............

...............

```

对 `malloc` 函数的调用对应于指令 `blx dd4`。跳转到了地址 `dd4`。看看这个地址里有什么吧：

```
caikelun@debian:~$ arm-linux-androideabi-objdump -D ./libtest.so

...............

...............

00000dd4 <malloc@plt>:

 dd4:   e28fc600    add ip, pc, #0, 12

 dd8:   e28cca03    add ip, ip, #12288  ; 0x3000

 ddc:   e5bcf1b4    ldr pc, [ip, #436]! ; 0x1b4

...............

...............

```

果然，跳转到了 `.plt` 中，经过了几次地址计算，最后跳转到了地址 `3f90` 中的值指向的地址处，`3f90` 是个函数指针。

稍微解释一下：因为 arm 处理器使用 3 级流水线，所以第一条指令取到的 `pc` 的值是当前执行的指令地址 + `8`。

于是：`dd4` + `8` + `3000` + `1b4` = `3f90`。

地址 `3f90` 在哪里呢：

```
caikelun@debian:~$ arm-linux-androideabi-objdump -D ./libtest.so

...............

...............

00003f60 <.got>:

    ...

    3f70:   00002604    andeq   r2, r0, r4, lsl #12

    3f74:   00002504    andeq   r2, r0, r4, lsl #10

    ...

    3f88:   00000da8    andeq   r0, r0, r8, lsr #27

    3f8c:   00000da8    andeq   r0, r0, r8, lsr #27

    3f90:   00000da8    andeq   r0, r0, r8, lsr #27

...............

...............

```

果然，在 `.got` 里。

顺便再看一下 `.rel.plt`：

```
caikelun@debian:~$ arm-linux-androideabi-readelf -r ./libtest.so



Relocation section '.rel.plt' at offset 0xcb8 contains 30 entries:

 Offset     Info    Type            Sym.Value  Sym. Name

00003f88  00000416 R_ARM_JUMP_SLOT   00000000   __cxa_atexit@LIBC

00003f8c  00000116 R_ARM_JUMP_SLOT   00000000   __cxa_finalize@LIBC

00003f90  00000316 R_ARM_JUMP_SLOT   00000000   malloc@LIBC

...............

...............

```

`malloc` 的地址居然正好存放在 `3f90` 里，这绝对不是巧合啊！还等什么，赶紧改代码吧。我们的 `main.c` 应该改成这样：

```
#include <test.h>



void *my_malloc(size_t size)

{

    printf("%zu bytes memory are allocated by libtest.so\n", size);

    return malloc(size);

}



int main()

{

    void **p = (void **)0x3f90;

    *p = (void *)my_malloc; // do hook



    say_hello();

    return 0;

}

```

编译运行一下：

```
caikelun@debian:~$ adb push ./main /data/local/tmp

caikelun@debian:~$ adb shell "chmod +x /data/local/tmp/main"

caikelun@debian:~$ adb shell "export LD_LIBRARY_PATH=/data/local/tmp; /data/local/tmp/main"

Segmentation fault

caikelun@debian:~$

```

思路是正确的。但之所以还是失败了，是因为这段代码存在下面的 3 个问题：

1. `3f90` 是个相对内存地址，需要把它换算成绝对地址。

2. `3f90` 对应的绝对地址很可能没有写入权限，直接对这个地址赋值会引起段错误。

3. 新的函数地址即使赋值成功了，`my_malloc` 也不会被执行，因为处理器有指令缓存（`instruction cache`）。

我们需要解决这些问题。

## 内存

### 基地址

在进程的内存空间中，各种 ELF 的加载地址是随机的，只有在运行时才能拿到加载地址，也就是**基地址**。我们需要知道 ELF 的基地址，才能将相对地址换算成绝对地址。
