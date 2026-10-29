
```asm
.global replace_start
.global replace_end
.global p_hk_info

.hidden replace_start
.hidden replace_end
.hidden p_hk_info

.data

// 这种方式尽量用于标准的 C/C++ 函数，因为通过 hook 函数再调用原函数，只能保证参数寄存器和 lr 寄存器是一致的，其他寄存器可能被修改。

replace_start:                      // 如果只是替换/跳到 hook 函数，其实是不用保存寄存器的，只是重新写比较麻烦，所以在之前的基础上

    push    {r0-r4}                  // r0=r0，中转用；r1=sp（push 之前的 sp），r2=r14/lr，r3=pc，r4=cpsr
    mrs     r0, cpsr
    str     r0, [sp, #0x10]          // r4 的位置存放 cpsr
    str     r14, [sp, #8]            // r2 的位置存放 lr
    add     r14, sp, #0x14
    str     r14, [sp, #4]            // r1 的位置存放真实 sp
    pop     {r0}                     // 恢复 r0
    push    {r0-r12}                 // 保存 r0-r12，之后是 r13-r16/cpsr
    mov     r0, sp
    ldr     r1, p_hk_info
    ldr     r3, [r1, #8]              // pre_callback，保存 lr
    str     pc, [sp, #0x3c]           // 存储 pc
    blx     r3

to_call_oriFun:

    ldr     r0, [sp, #0x40]           // cpsr
    msr     cpsr, r0
    ldmfd   sp!, {r0-r12}             // 恢复 r0-r12
    ldr     r14, [sp, #4]             // 恢复 r14/lr
    ldr     sp, [r13]                 // 恢复 sp（push 之前的 sp）
    ldr     lr, p_hk_info
    ldr     lr, [lr, #0x14]            // pHkFunAddr
    blx     lr

to_aft_callback:

    push    {r0-r4}                  // r0=r0，中转用；r1=sp（push 之前的 sp），r2=r14/lr，r3=pc，r4=cpsr
    mrs     r0, cpsr
    str     r0, [sp, #0x10]           // r4 的位置存放 cpsr
    str     r14, [sp, #8]             // r2 的位置存放 lr
    add     r14, sp, #0x14
    str     r14, [sp, #4]             // r1 的位置存放真实 sp
    pop     {r0}                      // 恢复 r0
    push    {r0-r12}                  // 保存 r0-r12，之后是 r13-r16/cpsr
    mov     r0, sp
    ldr     r1, p_hk_info
    ldr     r3, [r1, #0x10]            // aft_callback
    str     pc, [sp, #0x3c]            // 存储 pc
    blx     r3

to_popreg:

    ldr     r0, [sp, #0x40]            // cpsr
    msr     cpsr, r0
    ldmfd   sp!, {r0-r12}              // 恢复 r0-r12
    ldr     r14, [sp, #4]              // 恢复 r14/lr
    ldr     sp, [r13]                  // 恢复 sp（push 之前的 sp）
    mov     pc, lr

p_hk_info:                            // 结构体 STR_HK_INFO
.word 0x12345678

replace_end:

.end
```

`demo` 和 `arm64` 一致。

### 细节的优化

框架已经搭好了且已经可以运行了，接下来就是一些细节的优化。

#### 内存对齐

`shellcode`、备份/修复的原函数不能使用 `malloc` 等非内存对齐的函数，原因在下面的 `Android_Inline_Hook` 中较严重的一些 bug 有提到。

但是每个 `shellcode` 都占用一页内存也太浪费了。理想情况是一页用完或者放不下一个 `shellcode` 再申请，但是考虑到解除 `hook` 释放 `shellcode` 等问题，这么做实现起来很复杂，要监听整个流程，暂时没精力写这么完美。

所以就把 `shellcode` 和备份/修复的原函数放在同一页内存，当然还是用不满，但是至少节约了一些。这样解除 `hook` 直接释放这页内存都可以。

当然还可以进一步把 `HK_INFO` 这个结构体也放在这一页内，但是不确定是否确实有些系统只允许内存权限为 `rw` 或者 `rp`，暂时没有把 `HK_INFO` 混在一起。

#### 刷新指令缓存

```c
void test_args_11(int a0, int a1, int a2, int a3, int a4, int a5, int a6, int a7, int a8, int a9, int a10, int a11) {
    LE("a0=%d, a1=%d, a2=%d, a3=%d, a4=%d, a5=%d,"
       " a6=%d, a7=%d, a8=%d, a9=%d, a10=%d. a11=%d",
       a0, a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11);
}

void nullCallBack(my_pt_regs *regs, HK_INFO *pInfo) {
}

void test_args_for_cache() {
    // 测试多参数传递情况
    dump((void *)(test_args_11), nullCallBack, /*onCallBack*/NULL);
    // dumpRet((void *) (test_args_11), onPreCallBack, "test_args_11");
    test_args_11(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11);
    HK_INFO *pInfo = isHooked((void*)test_args_11);
    // unHook(pInfo);
}

// 拷贝回原指令后不刷新缓存。机器不同、CPU 性能、缓存大小不同等，这个阈值需要自己测试。
// 性能好的机器，循环 100 次基本都可以 100% 复现，差的机器 200 次也只有几次复现。
void test_cache() {
    test_args_for_cache();
    LE("test_args_11=%p", test_args_11);

    for (int i = 0; i < 188; ++i) {
        if (i == 178) {
            HK_INFO *pInfo = isHooked((void *) test_args_11);
            unHook(pInfo);
        }

        test_args_11(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11);
        LE("i=%d", i);
        // usleep(1);
    }

    LE("end test cache");
}
```

为了测试不刷新缓存的情况，使用了循环，因为实测发现想触发缓存不刷新 `crash` 概率太低了。我能想到的就是循环执行一段指令，触发 `jit`，提高指令被缓存的概率。然后修改指令后不刷新缓存，触发 `crash`。

关于如何刷新缓存，在下面的 `Android_Inline_Hook` 中较严重的一些 bug 有提到。

#### 既是入参也是出参的函数

只是确认下，因为代码写的没问题，所以应该是不会有问题的。

```c
int (*ori__system_property_get)(const char *name, char *value);

// 注意操作参数之前最好都检查参数是否为 null。如果原函数也是 crash 处理还行，如果不是就改变了函数逻辑、进程中断等。
// 所以细节都要注意，参数和返回值也有对应关系。
int my__system_property_get(const char *name, char *value) {
    LE("hk: __system_property_get(%s, %p)", name ? name : "is null", value);

    ori__system_property_get =
        (int (*)(const char *, char *))(getOriFunByHkFun(
            (void *)(my__system_property_get)));

    if (!ori__system_property_get) {
        ori__system_property_get = __system_property_get;
    }

    int ret = ori__system_property_get(name, value);

    LE("ori: __system_property_get(%s, %s)=%d",
       name ? name : "is null",
       ret > 0 ? value : "is null",
       ret);

    if (name && !strncmp("ro.serialno", name, strlen("ro.serialno"))) {
        strcpy(value, "12345678");
        return (int)(strlen("12345678"));
    }

    return ret;
}

// 测试既是参数也做返回值的函数
void test__system_property_get() {
    const RetInfo info = dump_replace(
        (void *)__system_property_get,
        (void *)(my__system_property_get),
        NULL,
        NULL,
        "__system_property_get");

    if (info.status != success) {
        LE("hook __system_property_get error=%d", info.status);
    }

    char sn[256];
    int ret = __system_property_get("ro.serialno", sn);
    LE("serialno=%s", ret > 0 ? sn : "is null");

    // 2019-12-23 17:10:14.810 22213-22213/? E/zhuo: hk: __system_property_get(ro.serialno, 0x7fd55a2088)
    // 2019-12-23 17:10:14.810 22213-22213/? E/zhuo: ori: __system_property_get(ro.serialno, 8acd10de)=8
    // 2019-12-23 17:10:14.810 22213-22213/? E/zhuo: lr aft_callback
    // 2019-12-23 17:10:14.810 22213-22213/? E/zhuo: serialno=12345678
}
```

测试不存在 `hookzz` 的问题。

### todo

基本上上面已经是大部分的框架和部分细节，剩下的对用户 API 的具体实现都是很灵活的，只要保持和 API 函数原型和定义行为一致即可（已实现）。

跳板、`shellcode`、API 都实现了，还剩下一个对备份的原函数进行修复。还没有重新开始写，暂时用的 `Android_Inline_Hook` 修复 `pc` 相关的指令。

在写 `arm` 的 `vmp`，到时抽取出指令解析和生成的代码就好了，加一些简单的语义分析。

线程暂停的问题，抽时间加上，因为一般自己常用的场景是 `fork` 一个进程还未执行应用代码的时候，基本上不会出现多线程导致的 `crach`，不过还是应该提供接口，毕竟有些场景需要，但是怕是也不能百分百解决线程问题，还需要用户自己判断函数行为。

可变参数的传递，如果不修改只是调用原函数倒是有思路了，但是如果是对明确的参数进行修改了，还是要抽时间写解析构造参数的部分。

### 已知 bug

#### 1. arm64 指令修复

```asm
.text:0000000000000EF8 dlopen                                  ; DATA XREF: LOAD:0000000000000508↑o
.text:0000000000000EF8
.text:0000000000000EF8 var_s0          =  0
.text:0000000000000EF8
.text:0000000000000EF8 ; __unwind {
.text:0000000000000EF8                 STP             X29, X30, [SP,#-0x10+var_s0]!
.text:0000000000000EFC                 MOV             X29, SP
.text:0000000000000F00                 MOV             X2, X30
.text:0000000000000F04                 BL              .__loader_dlopen
.text:0000000000000F08                 LDP             X29, X30, [SP+var_s0],#0x10
.text:0000000000000F0C                 RET
.text:0000000000000F0C ; } // starts at EF8
.text:0000000000000F0C ; End of function dlopen
```

例如其中的 `BL` 指令并未修复，这个其实还容易修复，可以替换成 `LDR LR, 8`（应该放到最后）；`BLR LR`。因为既然是 `BL` 指令，那么 `LR` 寄存器肯定会被修改，那就说明 `LR` 寄存器已经保存了，所以我们使用 `LR` 寄存器也是没有关系的。

如果是 `B` 指令就麻烦了，因为不能操作 `PC`，那么就要至少使用一个寄存器且还不能是 `LR`，只能使用 `X16`、`X17`。

后面看了下 `Android_Inline_Hook` 写了 `B` 指令的修复（是错的），`BL` 指令未实现。后来自己写了 `BL` 指令的实现，只是暂时用，不准备在这个基础上再写了，缺少的待修复指令还有不少，且已实现的也有一些错误，待重构。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/670707_MPBB6EEC9YCY2N2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202001/670707_6TNDZYAC7HWG8D4.png)

修复之后，Android 9.0 arm64 的 `dlopen` 可以 hook 了。

#### 2. arm64 从备份/修复的原方法跳回跳板 0 之后执行

采用的方式是在备份/修复的原方法后面加入：

```asm
STP X1, X0, [SP, #-0x10]

LDR X0, 8

BR X0

ADDR(64)

LDR X0, [SP, -0x8]
```

和跳板 0 是一样的，但是跳板 0 是一个函数开头的概率很大，所以一般都是会栈平衡的，`SP` 指向栈顶，暂时存放寄存器不会被覆盖。

而放在后面就不太确定能保证 `SP` 指向栈顶了，所以可能要考虑解析前几条指令是否有操作 `SP`，或者尽可能地把寄存器储存在 `SP` 减去较大的值，或者使用 `X16`、`X17` 寄存器。

当然这个可能概率较小，可以先不管。

#### 3. 无法重复 hook 一个函数

当然这个是说在其他框架或者本框架的另一个版本（或者无法维护保存 `hook` 的容器）。如果是在本框架内，是可以重复 `hook` 同一个函数的，比如解除 `hook` 再 `hook`，或者再扩展成链表的形式，依次调用 `hook`/`dump` 函数。

其实就是指令修复不完整，`arm` 的应该是 `ldr pc` 没有正确修复。这个事情排在后面，不是特别重要。
