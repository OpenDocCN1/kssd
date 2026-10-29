
`filesize equ $ - $$`

编译，

```
$ nasm -f bin -o a.out tiny.asm
$ chmod +x a.out
$ ./a.out ; echo $?
42
$ wc -c a.out
       76 a.out
```

看上去这一次我们真是走到尽头了。

除非，我们能改变这些结构。那需要再来认真地看一看，Linux真正需要这些文件中哪些域？譬如，Linux是否会检验`e_machine`域中放的是3？还是它会默认？

令人惊讶，实际上很多域都被Linux系统忽略了。

我们来看一看哪些项是必须的，哪些项是可以忽略的。开始的4个字节必须包含`magic number`，不然Linux不会处理该文件。但是另外三个字节并没有被检查，意味着我们有连续12个字节可以用来存放任何内容。`e_type`为2，表示是可执行文件；`e_machine`是3。`e_version`被忽略（目前ELF只有一个版本）。`e_entry`指向程序入口，`e_phoff`包含`program header table`的偏移，`e_phnum`包含`table`的项数。`e_flags`，目前没用；`e_ehsize`用于验证大小，但是Linux没用。`e_phentsize`用于验证`program header table`的项数，目前需要。其他内容与`section header table`相关，和我们的程序无关。

再来看看`program header table`的项。`p_type`的值为1，表明是可装载的段。`p_offset`需要指明文件的正确偏移职位，`p_vaddr`指明的是加载位置。注意，并不一定需要装在`0x08048000`;任何一个大于`0x00000000`，小于`0x80000000`并且页对齐的位置都行。`p_paddress`可忽略。`p_filesz`指明文件多少字节应该装入内存，`p_memsz`指明内存段应该是多大。`p_flags`指明内存段的权限。应该是4（可读），以及1（可执行）。`p_align`指明内存段的对齐需求，当重定位有`position-independent`代码的时候需要，所以目前我们也用不着。

有了以上基础，新的代码出炉：

```
ITS 32
                org     0x00200000
                db      0x7F, "ELF"             ; e_ident
                db      1, 1, 1, 0, 0
  _start:
                mov     bl, 42
                xor     eax, eax
                inc     eax
                int     0x80
                dw      2                       ; e_type
                dw      3                       ; e_machine
                dd      1                       ; e_version
                dd      _start                  ; e_entry
                dd      phdr - $$               ; e_phoff
  phdr:         dd      1                       ; e_shoff       ; p_type
                dd      0                       ; e_flags       ; p_offset
                dd      $$                      ; e_ehsize      ; p_vaddr
                                                ; e_phentsize
                dw      1                       ; e_phnum       ; p_paddr
                dw      0                       ; e_shentsize
                dd      filesize                ; e_shnum       ; p_filesz
                                                ; e_shstrndx
                dd      filesize                                ; p_memsz
                dd      5                                       ; p_flags
                dd      0x1000                                  ; p_align
  filesize      equ     $ - $$
```

两部分重叠了20个字节，并且重叠的相当好。`e_phnum`域需要，而`p_paddr`忽略；`e_phentsize`域和`p_vadress`相同。These are made to match up by selecting a non-standard load address for our program, with a top half equal to `0x0020`.

编译运行查大小：

```
$ nasm -f bin -o a.out tiny.asm
$ chmod +x a.out
$ ./a.out ; echo $?
42
$ wc -c a.out
       64 a.out
```

接下来的问题是，我们能不能让`program header table`完全在ELF header中的？

直接上移很明显会有问题；但是我们注意到`p_memsz`指示的是分配的内存空间，它有最小需求（`p_filesz`），但是没上限。同时，可执行的`bit`位可以被覆盖。这样，新的代码是：

```
; tiny.asm
BITS 32
                org     0x00010000
                db      0x7F, "ELF"             ; e_ident
                dd      1                                       ; p_type
                dd      0                                       ; p_offset
                dd      $$                                      ; p_vaddr
                dw      2                       ; e_type        ; p_paddr
                dw      3                       ; e_machine
                dd      _start                  ; e_version     ; p_filesz
                dd      _start                  ; e_entry       ; p_memsz
                dd      4                       ; e_phoff       ; p_flags
  _start:
                mov     bl, 42                  ; e_shoff       ; p_align
                xor     eax, eax
                inc     eax                     ; e_flags
                int     0x80
                db      0
                dw      0x34                    ; e_ehsize
                dw      0x20                    ; e_phentsize
                dw      1                       ; e_phnum
                dw      0                       ; e_shentsize
                dw      0                       ; e_shnum
                dw      0                       ; e_shstrndx
  filesize      equ     $ - $$
```

`p_filesz`发生了变化：为了保证对齐，`p_filesz`和`p_memsz`同样大小，和真实情况相比大了不少，但是无伤大雅。

大小为52字节。

目前文件的大小就只是ELF header的大小了，最后的一个小把戏是Linux会把ELF头部缺省位补0，我们可以把它们省去。

```
; tiny.asm
BITS 32
                org     0x00010000
                db      0x7F, "ELF"             ; e_ident
                dd      1                                       ; p_type
                dd      0                                       ; p_offset
                dd      $$                                      ; p_vaddr
                dw      2                       ; e_type        ; p_paddr
                dw      3                       ; e_machine
                dd      _start                  ; e_version     ; p_filesz
                dd      _start                  ; e_entry       ; p_memsz
                dd      4                       ; e_phoff       ; p_flags
  _start:
                mov     bl, 42                  ; e_shoff       ; p_align
                xor     eax, eax
                inc     eax                     ; e_flags
                int     0x80
                db      0
                dw      0x34                    ; e_ehsize
                dw      0x20                    ; e_phentsize
                db      1                       ; e_phnum
                                                ; e_shentsize
                                                ; e_shnum
                                                ; e_shstrndx
  filesize      equ     $ - $$
```

45字节！

啊，题目都错了！是45字节~~

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
