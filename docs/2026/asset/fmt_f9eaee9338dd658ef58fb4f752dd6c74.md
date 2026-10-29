
# [翻译]42字节可执行文件；ELF介绍；求Kx（四）

正经描述Intel-386架构ELF格式的文档在`856K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2k6Y4y4H3k6h3y4K6i4K6u0W2k6Y4u0W2k6i4y4@1j5h3&6V1j5i4u0V1M7#2)9J5k6h3!0J5k6#2)9J5c8X3g2D9k6W2)9J5c8X3g2D9k6W2)9J5k6i4m8V1k6W2)9J5k6b7`.` `. 该文档包括各个边角，因此，人们不愿意读是完全可以理解的。实际上，关于ELF，需要知道的差不多就是一下这些内容。

每个ELF文件都由ELF头部开始。ELF头部长度是52字节，描述了该文件的一些元信息。譬如，前16个字节包含了一个“标示符”，包含文件的“magic number”`7F 45 4C 46`，还有1字节的标识信息如是32bit还是64bit，little-endian还是big-endian等等。其他信息包括该ELF文件是可执行的、目标文件还是共享库；该程序的起始地址，文件内程序头表和section头表的位置。

`program header table`和`section header table`可以在文件的任意位置，但是一般而言，`program header table`则位于文件的结尾。这两个表的作用相似，都拥护标记文件的哥哥部分。不同之处在于`program header table`用于标记文件的每个部分会被加载到内存的什么位置，而`section header table`标记各个部分在文件里的位置。简而言之，编译器和链接器使用的是`section header table`，而加载器用着的是`program header table`. 因为这个原因，目标文件用不着`program header table`，而可执行文件用不着`section header table`，但是它们总是存在的！

因此，我们还能够去掉的部分就是`section header table`以及其它一些同样没用的部分了。问题是，怎么样去掉它们？

现在的标准工具里没有哪个生成的可执行文件中会没有`section header table`的。我们得自己动手。当然也并不意味着我们得自己动手写二进制文件，Nasm有一个 flat 二进制输出格式，我们只需要一个空白的ELF可执行文件的镜像，然后把文件填进去进行了。只填我们的程序。

我们来看一下ELF的规范，`/usr/include/linux/elf.h`，以及标准工具生成的可执行文件，弄明白空的ELF可执行文件的格式。但是，如果你不耐烦这么一步步的琢磨，那么看下面的一些内容：

```asm
BITS 32

                org     0x08048000

  ehdr:                                                          ; Elf32_Ehdr

                db      0x7F, "ELF", 1, 1, 1, 0         ;   e_ident

        times 8 db      0

                dw      2                                        ;   e_type

                dw      3                                        ;   e_machine

                dd      1                                         ;   e_version

                dd      _start                                 ;   e_entry

                dd      phdr - $$                            ;   e_phoff

                dd      0                                         ;   e_shoff

                dd      0                                         ;   e_flags

                dw      ehdrsize                             ;   e_ehsize

                dw      phdrsize                             ;   e_phentsize

                dw      1                                         ;   e_phnum

                dw      0                                         ;   e_shentsize

                dw      0                                         ;   e_shnum

                dw      0                                         ;   e_shstrndx

  ehdrsize      equ     $ - ehdr

  phdr:                                                            ; Elf32_Phdr

                dd      1                                           ;   p_type

                dd      0                                           ;   p_offset

                dd      $$                                         ;   p_vaddr

                dd      $$                                         ;   p_paddr

                dd      filesize                                   ;   p_filesz

                dd      filesize                                   ;   p_memsz

                dd      5                                            ;   p_flags

                dd      0x1000                                  ;   p_align

  phdrsize      equ     $ - phdr

  _start:

  ; your program here

  filesize      equ     $ - $$
```

以上的镜像包含ELF头部，指明这个文件是一个intel 386可执行文件，没有`section header table`，`program header table`仅包含一项，指明加载器应该把该文件整体地加载到内存`0x8048000`的位置，并且从`_start`处开始执行。这个镜像中，没有`.data`，没有`.bss`，只有必须的部分。

那么我们新的汇编程序如下：

```asm
; tiny.asm

                org     0x08048000

  ;

  ; (as above)

  ;

_start:

            mov bl, 42

            xor eax, eax

            inc eax

            int 0x80

filesize equ $ - $$
```

编译：

```bash
$ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
```

我们一点点自己写出的可执行文件的大小如何呢？

```bash
$ wc -c a.out
       91 a.out
```

这个文件还能够进一步缩减么？

如果大家刚才认真看了ELF的规范，可能会注意到这样几个问题：1. ELF文件的不同部分可以位于ELF文件的任意位置（ELF头部除外，必须放在文件的起始位置），不同部分之间甚至可以重叠；2）头部的一些域并没有使用；

特别地，16个字节的标识域的后部分的0都是填充来着，为以后的EFL扩展做准备。我们的代码仅有7个字节长，那我们能不能把代码放在ELF头部呢？

```asm
; tiny.asm

  BITS 32

                org     0x08048000

  ehdr:                                                 ; Elf32_Ehdr

                db      0x7F, "ELF"                     ;   e_ident

                db      1, 1, 1, 0, 0

  _start:       mov     bl, 42

                xor     eax, eax

                inc     eax

                int     0x80

                dw      2                               ;   e_type

                dw      3                               ;   e_machine

                dd      1                               ;   e_version

                dd      _start                          ;   e_entry

                dd      phdr - $$                       ;   e_phoff

                dd      0                               ;   e_shoff

                dd      0                               ;   e_flags

                dw      ehdrsize                        ;   e_ehsize

                dw      phdrsize                        ;   e_phentsize

                dw      1                               ;   e_phnum

                dw      0                               ;   e_shentsize

                dw      0                               ;   e_shnum

                dw      0                               ;   e_shstrndx

  ehdrsize      equ     $ - ehdr

  phdr:                                                 ; Elf32_Phdr

                dd      1                               ;   p_type

                dd      0                               ;   p_offset

                dd      $$                              ;   p_vaddr

                dd      $$                              ;   p_paddr

                dd      filesize                        ;   p_filesz

                dd      filesize                        ;   p_memsz

                dd      5                               ;   p_flags

                dd      0x1000                          ;   p_align

  phdrsize      equ     $ - phdr

  filesize      equ     $ - $$
```

编译：

```bash
$ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42

  $ wc -c a.out
       84 a.out
```

现在为止，我们的可执行文件只不过是一个ELF的头部和`program header table`项了，这两项是程序装载执行所必须的，看上去实在是无可缩减了！

只不过....

我们能不能想处理程序一样，把`program header table`也放在ELF头中呢？让它们重叠，有没有可能？

实际上确实是可行的，ELF头的最后的八个字节和`program headers table`的前八个字节是一模一样的。

```asm
; tiny.asm

  BITS 32

                org     0x08048000

  ehdr:

                db      0x7F, "ELF"             ; e_ident

                db      1, 1, 1, 0, 0

  _start:       mov     bl, 42

                xor     eax, eax

                inc     eax

                int     0x80

                dw      2                       ; e_type

                dw      3                       ; e_machine

                dd      1                       ; e_version

                dd      _start                  ; e_entry

                dd      phdr - $$               ; e_phoff

                dd      0                       ; e_shoff

                dd      0                       ; e_flags

                dw      ehdrsize                ; e_ehsize

                dw      phdrsize                ; e_phentsize

  phdr:         dd      1                       ; e_phnum       ; p_type

                                                ; e_shentsize

                dd      0                       ; e_shnum       ; p_offset

                                                ; e_shstrndx

  ehdrsize      equ     $ - ehdr

                dd      $$                                      ; p_vaddr

                dd      $$                                      ; p_paddr

                dd      filesize                                ; p_filesz

                dd      filesize                                ; p_memsz

                dd      5                                       ; p_flags

                dd      0x1000                                  ; p_align

  phdrsize      equ     $ - phdr
```
