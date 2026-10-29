
因为`thumb16`中能直接操作的寄存器是`r0-r7`；`r8-r12`、`r14`寄存器不能被`str`、`ldr`等，`ldr pc`寄存器也不行，但是可以`ldr`到`r0-r7`，再`mov`到`sp`寄存器。而`thumb`模式下的跳板0也是`LDR PC, [PC, #0]`，是32位的`thumb32/thumb-2`指令，所以理论上`armv5`和以下的`cpu`应该是不支持指令的。

因为`shellcode`中汇编指令超出了`thumb16`的寄存器范围，使用`thumb32/thumb-2`指令，汇编器不支持：

```
Error: unexpected character `w' in type specifier

Error: bad instruction `str.w r14,[sp,#8]'
```

所以不用在`Android.mk`中指定`LOCAL_ARM_MODE := arm`，因为这样整个`module`都是`arm`指令、4字节，浪费了空间，所以默认就行。奇怪的是编译出的文件包含`arm`和`thumb32`，混合的。没有细看是什么情况，编译`c/c++`使用的汇编器不一致吗？

呃，后来看到:经过`Google`工程师的提醒，对于`ARM GCC`的汇编器，在汇编文件最上面加入`.syntax unified`之后，`Thumb-2 T3 encoding`汇编也能正常使用了，比如：

```
.syntax unified

.text

.align 4

.globl helloThumb



.thumb

.thumb_func

helloThumb:



    add.w    r0, r0, r1, lsl #2

    bx        lr
```

所以在汇编文件中加入

```
.syntax unified

.force_thumb
```

可以把操作`r0-r7`寄存器之外的指令写成，`ldr.w`、`str.w`、`add.w`等，也可以不修改，编译器会自动转成`thumb32/thumb-2`指令。

编译后就是`thumb32/thumb-2`指令了，

![图片描述](https://bbs.kanxue.com/upload/attach/202001/670707_SPSNPYEQS29PDUJ.png)

因为这个`shellcode`变成`thumb`了，所以构建跳板`shellcode`时(`LDR PC, [PC, #0]`/`hook thumb`函数或者`LDR PC, [PC, #-4]`/`hook arm`函数，指令下面的地址要加1)。

所以`hook arm`函数也可以构建`thumb`的`shellcode`，同样的`thumb`函数也可以构建`arm`的`shellcode`。但是为了方便、不考虑四字节对齐，干脆还是都用`arm`的`shellcode`吧，只要不指定为`thumb`(默认可能就是)、不指定`.syntax unified`就始终编译出来的就是`arm`的`shellcode`。毕竟没有那么严重的强迫症，也占不了多少空间。

`LDR PC, [PC, #0]`/`hook thumb`函数或者`LDR PC, [PC, #-4]`/`hook arm`函数，理论上在`armv5`及以上就可以，所以无论是`arm`到`thumb`还是`thumb`到`thumb`，`LDR`到`PC`的地址+1就可以了。同样到`arm`就保证是非奇数的地址即可。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/670707_ASK9AWASU9HR94D.png)

至于这个`shellcode`编译到`.text`还是`.data`都可以。不同的就是`.data`中是数据，`.text`中为函数。

### 跳板0统一使用ldr pc

只要`cpu`是`armv5`以上就行，因为目前很少`armv5`及以下的`cpu`了，所以不考虑支持问题。

`ldr pc, [pc, #0/#4]`,可被编译成`arm`或者`thumb32/thumb-2`指令。这里可能有人会有些误解，比如我们要`hook`的一个`so`是`armeabi`的，那么`thumb`函数只支持`thumb16`，所以是没有`ldr pc`这样的指令的，那么对于这样的函数还能使用`ldr pc`覆盖吗？其实是可以的，因为这条指令能不能被正确解析执行是取决于`cpu`，只要`cpu`支持`thumb32/thumb-2`指令(`armv5`以上的`cpu`)即可。`thumb32/thumb-2`指令更像是`thumb16`的一个超集(16位的指令应该都是相同的，32位的指令会不同)，所以不影响后面的`thumb16`的指令的解析执行。

所以这里我们不是对当前函数的`thumb`做兼容性适配而是针对`cpu`的，而如果要适配`armv5`以下，就多绕一下好了。

### 代码实现*arm/thumb*统一的跳板0

`arm`:

```
    LDR PC, [PC, #-4];流水线，pc为addr下面的指令，所以-4

    addr:0x12345678
```

`thumb32/thumb-2`:

```
    LDR PC, [PC, #0];流水线，pc为addr指令

    addr:0x12345678
```

都是占用8条指令，如果`thumb`函数地址不是4字节对齐的话就加`nop`或者`bx #-2`占用10字节。

### 代码实现_arm_dump函数

```
.global _dump_start

.global _dump_end

.global _hk_info

.global _oriFuc



.hidden _dump_start

.hidden _dump_end

.hidden _hk_info

.hidden _oriFuc



//可用于标准的c/c++函数、非标准函数、函数的一部分(用于读写寄存器)，前提都是字节长度足够

//非标准函数即非c/c++编译的函数，那么手写汇编可能存在并不遵守约定的情况，比如我们使用了sp寄存器，并在未使用的栈上保存寄存器

//但是可能不是满递减而是反过来满递增，或者不遵守栈平衡，往栈上写数据，但是并不改变sp寄存器。当然应该是很少见的。



.data



_dump_start:                    //用于读写寄存器/栈，需要自己解析参数，不能读写返回值，不能阻止原函数(被hook函数)的执行

                                //从行为上来我觉得更偏向dump，所以起名为dump。

    push    {r0-r4}                     //r0=r0,中转用 r1=sp(push之前的sp), r2=r14/lr, r3=pc, r4=cpsr

    mrs     r0, cpsr

    str     r0, [sp, #0x10]             //r4的位置存放cpsr

    str     r14, [sp, #8]               //r2的位置存放lr

    add     r14, sp, #0x14

    str     r14, [sp, #4]               //r1的位置存放真实sp

    pop     {r0}                        //恢复r0

    push    {r0-r12}                    //保存r-r12，之后是r13-r16/cpsr

    mov     r0, sp

    ldr     r1, _hk_info;

    ldr     r3, [r1];                   //onPreCallBack

    str     pc, [sp, #0x3c];            //存储pc,arm模式，所以pc+8,指向ldr     r0, [sp, #0x40]

    blx     r3

    ldr     r0, [sp, #0x40]             //cpsr

    msr     cpsr, r0

    ldmfd   sp!, {r0-r12}               //恢复r0-r12

    ldr     r14, [sp, #4]               //恢复r14/lr

    ldr     sp, [r13]                   //恢复sp(push之前的sp)

    ldr     pc, _oriFuc



_oriFuc:                                //备份/修复的原方法

.word 0x12345678



_hk_info:                               //结构体STR_HK_INFO

.word 0x12345678



_dump_end:



.end
```

这里和`arm64`比有点不同的是多了一个标号/4个字节存放`_oriFuc`/`原函数`的地址，这么做的原因是比较无奈的，因为`_hk_info`取到的是`HK_INFO`,还需要再计算一次才能得到`pOriFuncAddr`，这样就需要占用一个寄存器去计算，`pc`寄存器是不能这么用的，所以还是在`shellcode`中存储吧。当然其实还可以让`_hk_info`存放的是`pOriFuncAddr`这个变量的地址，那么可以一次取到`pc`寄存器中，但是其他`shellcode`都要变且不如原来的直观/简单了，同样的方式还可以把`pOriFuncAddr`放在结构体头部，然后`_hk_info`存放`pOriFuncAddr`变量的地址/`HK_INFO`结构体的地址，也是可以的，但是也是要变动太多。而且其实只有这一个`shellcode`需要多定义一个`_oriFuc`，其他是不需要的，所以折衷后就这样了。

`demo`和`arm64`一致，`api`也是一致的。`arm`和`thumb`共用这一个`shellcode`，注意的一点就是如果被`hook`的函数是`thumb`，那么`_oriFuc`存放的地址+1即可。

### 代码实现_arm_dump_with_ret函数

```
.global r_dump_start

.global r_dump_end

.global r_hk_info



.hidden r_dump_start

.hidden r_dump_end

.hidden r_hk_info





.data



r_dump_start:                    //用于读写寄存器/栈，需要自己解析参数，不能读写返回值，不能阻止原函数(被hook函数)的执行

                                //从行为上来我觉得更偏向dump，所以起名为dump。

    push    {r0-r4}                     //r0=r0,中转用 r1=sp(push之前的sp), r2=r14/lr, r3=pc, r4=cpsr

    mrs     r0, cpsr

    str     r0, [sp, #0x10]             //r4的位置存放cpsr

    str     r14, [sp, #8]               //r2的位置存放lr

    add     r14, sp, #0x14

    str     r14, [sp, #4]               //r1的位置存放真实sp

    pop     {r0}                        //恢复r0

    push    {r0-r12}                    //保存r-r12，之后是r13-r16/cpsr

    mov     r0, sp

    ldr     r1, r_hk_info;

    ldr     r3, [r1, #8];               //pre_callback

    str     pc, [sp, #0x3c];            //存储pc

    blx     r3



to_call_oriFun:

    ldr     r0, [sp, #0x40]             //cpsr

    msr     cpsr, r0

    ldmfd   sp!, {r0-r12}               //恢复r0-r12

    ldr     r14, [sp, #4]               //恢复r14/lr

    ldr     sp, [r13]                   //恢复sp(push之前的sp)

    ldr     lr, r_hk_info

    ldr     lr, [lr, #4];               //pOriFuncAddr

    blx     lr;



to_aft_callback:

    push    {r0-r4}                     //r0=r0,中转用 r1=sp(push之前的sp), r2=r14/lr, r3=pc, r4=cpsr

    mrs     r0, cpsr

    str     r0, [sp, #0x10]             //r4的位置存放cpsr

    str     r14, [sp, #8]               //r2的位置存放lr

    add     r14, sp, #0x14

    str     r14, [sp, #4]               //r1的位置存放真实sp

    pop     {r0}                        //恢复r0

    push    {r0-r12}                    //保存r-r12，之后是r13-r16/cpsr

    mov     r0, sp

    ldr     r1, r_hk_info;

    ldr     r3, [r1, #0x10];            //aft_callback

    str     pc, [sp, #0x3c];            //存储pc

    blx     r3



to_popreg:

    ldr     r0, [sp, #0x40]             //cpsr

    msr     cpsr, r0

    ldmfd   sp!, {r0-r12}               //恢复r0-r12

    ldr     r14, [sp, #4]               //恢复r14/lr

    ldr     sp, [r13]                   //恢复sp(push之前的sp)

    mov     pc, lr;



r_hk_info:                               //结构体STR_HK_INFO

.word 0x12345678



r_dump_end:



.end
```

`demo`和`arm64`一致，这里不用多定义一个`_oriFuc`，因为保存了`lr`，不是必须使用`pc`寄存器了。也是让`lr`寄存器做了很多事情。

### 代码实现_arm_dump_just_ret函数

```
.global j_dump_start

.global j_dump_end

.global j_hk_info



.hidden j_dump_start

.hidden j_dump_end

.hidden j_hk_info





.data



j_dump_start:



    push    {r0-r4}                     //r0=r0,中转用 r1=sp(push之前的sp), r2=r14/lr, r3=pc, r4=cpsr

    mrs     r0, cpsr

    str     r0, [sp, #0x10]             //r4的位置存放cpsr

    str     r14, [sp, #8]               //r2的位置存放lr

    add     r14, sp, #0x14

    str     r14, [sp, #4]               //r1的位置存放真实sp

    pop     {r0}                        //恢复r0

    push    {r0-r12}                    //保存r-r12，之后是r13-r16/cpsr

    mov     r0, sp

    ldr     r1, j_hk_info;

    ldr     r3, [r1];                   //onPreCallBack

    str     pc, [sp, #0x3c];            //存储pc

    blx     r3

    ldr     r0, [sp, #0x40]             //cpsr

    msr     cpsr, r0

    ldmfd   sp!, {r0-r12}               //恢复r0-r12

    ldr     r14, [sp, #4]               //恢复r14/lr

    ldr     sp, [r13]                   //恢复sp(push之前的sp)

    mov     pc, lr





j_hk_info:                               //结构体STR_HK_INFO

.word 0x12345678



j_dump_end:



.end
```

`demo`和`arm64`一致。

### 代码实现_arm_replace函数
