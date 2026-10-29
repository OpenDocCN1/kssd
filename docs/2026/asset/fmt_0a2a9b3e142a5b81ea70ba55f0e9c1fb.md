
awx指定读写执行，在elf中(`ida`查看)确实是读写执行。如果这个`section`中仅包含变量，那么在内存中放在可读写的段；如果存在代码或者代码和变量均存在，都是和代码段一样仅可读执行。那么在单独的`section`中存放蹦床代码意义也不大了，还是需要调用`mprotect`修改内存权限。不过考虑到如果放在仅可读写的段中，那么万一映射后的`rw-p`和`r-xp`超过了`64m`，不就白瞎了，所以还是以代码或者至少这个`section`中存在一个函数，保证和被`hook`的函数都在`r-xp`。

```
77c5545000-77c557c000 r-xp 00000000 103:37 533353 /data/app/com.zhuotong.myinhk-TRvqt0ReHRjQeeAnpXDnFQ==/lib/arm64/libInlineHook.so
77c558b000-77c558e000 r--p 00036000 103:37 533353 /data/app/com.zhuotong.myinhk-TRvqt0ReHRjQeeAnpXDnFQ==/lib/arm64/libInlineHook.so
77c558e000-77c558f000 rw-p 00039000 103:37 533353 /data/app/com.zhuotong.myinhk-TRvqt0ReHRjQeeAnpXDnFQ==/lib/arm64/libInlineHook.so
```

而如果直接和`.text`一起，主要是怕对这块内存修改权限引发什么异常，不确定如果代码正在执行，修改权限是否会出问题，所以最好能仅修改蹦床的区间。而且不确定是否有只能修改为读写和读执行的系统，所以可能要先修改成读写，写了之后再修改成读执行？这样会不会也有几率触发问题，但是如果每个蹦床都分配一个页的内存也不现实。。。

## 内嵌汇编，在自定义section中增加蹦床

![图片描述](upload/attach/202001/670707_GGZEWHVM2JPJH7Z.png)

这样定义一个无参无返回的函数，倒是可以使这个函数就是内嵌汇编，但是不确定如果`ollvm`混淆等是否会被拆分、加入垃圾代码等。

而且问题在于怎么自动生成一个蹦床函数。宏定义？那这个宏要在函数之外，不太容易自动化实现，包括蹦床的函数名称、`b`后面的函数名称。和上面的汇编中添加一个蹦床一样的问题，除非自己实现预处理之类的自动插入汇编代码、宏等，似乎不太现实。。。

似乎很难实现自动化，手动写代码太麻烦且和`arm`的部分接口、行为不一致。但是如果自己简单测试、不在乎这些也是可以的。

## 在自定义section中预留蹦床空间

```
unsigned int hkArry[256*2] __attribute__ ((section(".my_sec")));
```

```
//下面为伪代码，通过这样也可以实现不修改/保存寄存器、不修改hook函数，动态生成蹦床完成hook。
//优点是不用保存寄存器，缺点是因为正负64m的限制，hook函数应该和hook库是一起编译成动态库/可执行文件的。不能单独使用hook库。
//256个蹦床，实际使用可能要考虑这个hkArry内存对齐的问题(如果编译器未做内存对齐)。
//unsigned int hkArry[256 * 2] __attribute__ ((section(".my_sec")));
//void test_trampoline(){
//仅用于演示，应该先设置内存为可读写/执行，
// hkArry[0] = 0xf85f83e0;//或者memcpy, ldr x0, [sp, #-0x8];
//例如这样计算偏移，组合b指令。
// unsigned long offset = (unsigned long) my_open - ((unsigned long) hkArry + 1 * 4);
// hkArry[1] = 0x14000000;//计算偏移，生成指令，b 0;
//}
```

`256`个蹦床。动态生成蹦床代码，第一条指令固定，第二条指令计算生成。需要`hkArry`修改为读写执行。

![图片描述](upload/attach/202001/670707_FTDSUY5GXFXVNCB.png)

## 未实现

`so`的`r-xp`中应该是有未使用的多余的内存的，为了对齐、页面等，所以怎么确定多余的空间大小和位置，然后蹦床代码存放其中。

## 为什么尽量不使用x16、x17等不用保存的寄存器？

因为种种限制，所以最后采用的还是保存一个寄存器，而使用哪个寄存器呢，我是选的`lr`寄存器。为什么不用`x16`、`x17`等不用保存的寄存器？这里就涉及到一个标准和规范的问题。

理论上编译器编译的`c/c++`函数是遵守这个规范的，要么不使用`x16`、`x17`寄存器，要么只是临时中转，不会在调用其他函数之后再从`x16`、`x17`寄存器取值(因为其他函数可能改变`x16`、`x17`)，但是内嵌汇编(虽然指定不使用`x16`、`x17`寄存器，但还是被编译器使用了)或者人工写的汇编，虽然不常见，但确实存在。而最常见的是`plt`函数内都是使用`x16`、`x17`做中转。所以使用一个不一定会被保存的寄存器不如使用一个会被保存的寄存器。而因为`lr`寄存器的特殊性，一般使用的话都会先压栈保存，结束恢复(32位不一定恢复，64位是会恢复`lr`寄存器的，因为不能直接操作`pc`，多数都是恢复`lr`，再`ret`)

所以其实主要还是兼容性考虑，尽量采用一些绕弯的方式不改变任何寄存器，实在没办法的情况下或者标准`c/c++`函数、非函数任意位置`hook`的情况下才使用`x16`、`x17`寄存器。

## 解析寄存器、栈取出参数，调用hook函数

难点在于：

1、不知道参数个数、参数类型。

2、需要确定各种类型参数占几个寄存器、可变参数等

其实在源码中定义`hook`函数的时候是有函数原型的，但是运行时拿不到。忽然想到`"c++"`函数名的规则，里面包含参数、返回值类型，似乎可行，但是很多情况并不希望导出函数，而且也并不一定都是`c++`实现的。

那么如果定义一个接口，传入函数原型也不是不行，基本类型就用相应的字符或者枚举标识，其他所有的都是`void*`指针。可还是怕碰见可变参数函数，不确定参数个数，参数类型，只有接收/实现函数才知道。所以似乎不太可行。

而`libffi`似乎不太适合这个情况，也是需要明确指令参数和返回类型，还要传输参数。

目前的实现没有经过自己解析参数，只是中转，通过定义一致的函数原型，让编译器帮助我们解析参数。

### 代码实现_arm64_统一的跳板0

因为`arm64`限制条件最多，那么应该先实现`arm64`，这样才能更好的保证`api`的通用/一致性，因为`arm64`不能操作`pc`不得不这么做，`arm32`同样也可以这么做，但是如果先实现`arm32`，可能就先入为主的设计一些`arm32`可用的`api`。这算是一种架构思维吧，或者多思考一下就明白了。

```
STP X1, X0, [SP, #-0x10];//因为要使用一个寄存器，所以先保存，当然不一定是x0
LDR X0, 8; //把shellcode/hook函数地址存到x0
BR X0; //执行shellcode/hook函数
ADDR(64)
LDR X0, [SP, -0x8];//因为不能操作pc，所以跳回来的时候免不了也要使用一个寄存器
```

其实很简单，注释基本都说明了，使用了`24`字节，所以如果能确定要`hook`的函数是标准的`c/c++`函数，不会使用`x16`、`x17`去保存值的话也可以这样，使用`16`字节，降低被`hook`函数太短失败的概率。

```
LDR X17, 8;
BR x17;
ADDR(64)
```

### 代码实现_arm64_dump函数

```
.include "../../asm/base.s"
//.extern _dump_start
//.extern _dump_end
//.extern _hk_info
//.set _dump_start, r_dump_start
//.set _dump_end, r__dump_end
//.set _hk_info, r__hk_info
//.global _dump_start
//.global _dump_end
//.global _hk_info
//.hidden _dump_start
//.hidden _dump_end
//.hidden _hk_info
//可用于标准的c/c++函数、非标准函数、函数的一部分(用于读写寄存器)，前提都是字节长度足够
//非标准函数即非c/c++编译的函数，那么手写汇编可能存在并不遵守约定的情况，比如我们使用了sp寄存器，并在未使用的栈上保存寄存器
//但是可能不是满递减而是反过来满递增，或者不遵守栈平衡，往栈上写数据，但是并不改变sp寄存器。当然应该是很少见的。
.data
_dump_start: //用于读写寄存器/栈，需要自己解析参数，不能读写返回值，不能阻止原函数(被hook函数)的执行
//从行为上来我觉得更偏向dump，所以起名为dump。
sub sp, sp, #0x20; //跳板在栈上存储了x0、x1，但是未改变sp的值
mrs x0, NZCV
str x0, [sp, #0x10]; //覆盖跳板存储的x1，存储状态寄存器
str x30, [sp]; //存储x30
add x30, sp, #0x20
str x30, [sp, #0x8]; //存储真实的sp
ldr x0, [sp, #0x18]; //取出跳板存储的x0
save_x0_x29://保存寄存器x0-x29
sub sp, sp, #0xf0; //分配栈空间
stp X0, X1, [SP]; //存储x0-x29
stp X2, X3, [SP,#0x10]
stp X4, X5, [SP,#0x20]
stp X6, X7, [SP,#0x30]
stp X8, X9, [SP,#0x40]
stp X10, X11, [SP,#0x50]
stp X12, X13, [SP,#0x60]
stp X14, X15, [SP,#0x70]
stp X16, X17, [SP,#0x80]
stp X18, X19, [SP,#0x90]
stp X20, X21, [SP,#0xa0]
stp X22, X23, [SP,#0xb0]
stp X24, X25, [SP,#0xc0]
stp X26, X27, [SP,#0xd0]
stp X28, X29, [SP,#0xe0]
call_onPreCallBack://调用onPreCallBack函数，第一个参数是sp，第二个参数是STR_HK_INFO结构体指针
mov x0, sp; //x0作为第一个参数，那么操作x0=sp，即操作栈读写保存的寄存器
ldr x1, _hk_info;
ldr x3, [x1]; //onPreCallBack
bl get_lr_pc; //lr为下条指令
add lr, lr, #8; //lr为blr x3的地址
str lr, [sp, #0x108]; //lr当作pc，覆盖栈上的x0
blr x3
restore_regs://恢复寄存器
ldr x0, [sp, #0x100]; //取出状态寄存器
msr NZCV, x0
ldp X0, X1, [SP]; //恢复x0-x29寄存器
ldp X2, X3, [SP,#0x10]
ldp X4, X5, [SP,#0x20]
ldp X6, X7, [SP,#0x30]
ldp X8, X9, [SP,#0x40]
ldp X10, X11, [SP,#0x50]
ldp X12, X13, [SP,#0x60]
ldp X14, X15, [SP,#0x70]
ldp X16, X17, [SP,#0x80]
ldp X18, X19, [SP,#0x90]
ldp X20, X21, [SP,#0xa0]
ldp X22, X23, [SP,#0xb0]
ldp X24, X25, [SP,#0xc0]
ldp X26, X27, [SP,#0xd0]
ldp X28, X29, [SP,#0xe0]
add sp, sp, #0xf0
ldr x30, [sp]; //恢复x30
add sp, sp, #0x20; //恢复为真实sp
call_oriFun:
stp X1, X0, [SP, #-0x10]; //因为跳转还要占用一个寄存器，所以保存
ldr x0, _hk_info;
ldr x0, [x0, #8]; //pOriFuncAddr
br x0
get_lr_pc:
ret; //仅用于获取LR/PC
_hk_info: //结构体STR_HK_INFO
.double 0xffffffffffffffff
_dump_end:
.end
```

基本上注释已经说明了。`_hk_inf`为如下结构体的指针，`onPreCallBack`函数原型如下。

```
//hook信息
typedef struct STR_HK_INFO{
void (*onPreCallBack)(struct my_pt_regs *, struct STR_HK_INFO *pInfo); //回调函数，在执行原函数之前获取参数/寄存器的函数
void * pOriFuncAddr; //存放备份/修复后原函数的地址
void (*pre_callback)(struct my_pt_regs *, struct STR_HK_INFO *pInfo); //pre_callback,内部做保存lr的操作，之后回调onPreCallBack，不能被用户操作
void (*onCallBack)(struct my_pt_regs *, struct STR_HK_INFO *pInfo); //回调函数，执行原函数之后获取返回值/寄存器的函数
void (*aft_callback)(struct my_pt_regs *, struct STR_HK_INFO *pInfo); //aft_callback，内部做恢复lr的操作，之后回调onCallBack，不能被用户操作
void *pHkFunAddr; //hook函数，即自定义按照被hook的函数原型构造，处理参数/返回值的函数
//以上为在shellcode中通过偏移直接或间接用到的，所以如果有变动，相应的shellcode也要跟着变动
void **ppHkFunAddr; //上面pHkFunAddr函数在shellcode中的地址,已废弃;
void **hk_infoAddr; //shellcode中HK_INFO的地址
void *pBeHookAddr; //被hook的地址/函数
void *pStubShellCodeAddr; //跳过去的shellcode stub的地址
size_t shellCodeLength; //上面pStubShellCodeAddr的字节数
void ** ppOriFuncAddr; //shellcode 中存放备份/修复后原函数的地址，已废弃，待去除；改成上面pHkFunAddr函数的指针，应用于解除hook
void *pNewEntryForOriFuncAddr; //和pOriFuncAddr一致
BYTE szbyBackupOpcodes[OPCODEMAXLEN]; //原来的opcodes
int backUpLength; //备份代码的长度，arm64模式下为24
int backUpFixLengthList[BACKUP_CODE_NUM_MAX]; //保存
const char* methodName;
} HK_INFO;
#if defined(__aarch64__)
struct my_pt_regs {
__u64 uregs[31];
__u64 sp;
__u64 pstate; //有时间应该修复，pc在前，但是涉及到栈和生成shellcode都要改，先这么用吧，和系统结构体有这点不同
__u64 pc;
};
#define ARM_lr uregs[30]
#define ARM_sp sp
//#define SP (__u64*)sp
//#define SP_32(i) *(__u32*)((__u64)(regs->sp) + i*4)
//#define SP_32(i) *((__u32*)regs->sp+i)
//#define SP_32(i) *((__u64*)regs->sp+i)
#define SP(i) *((__u64*)regs->sp+i)
#elif defined(__arm__)
struct my_pt_regs {
long uregs[18];
};
#ifndef __ASSEMBLY__
#define ARM_cpsr uregs[16]
#define ARM_pc uregs[15]
#define ARM_lr uregs[14]
#define ARM_sp uregs[13]
#define ARM_ip uregs[12]
#define ARM_fp uregs[11]
#define ARM_r10 uregs[10]
#define ARM_r9 uregs[9]
#define ARM_r8 uregs[8]
#define ARM_r7 uregs[7]
#define ARM_r6 uregs[6]
#define ARM_r5 uregs[5]
#define ARM_r4 uregs[4]
#define ARM_r3 uregs[3]
#define ARM_r2 uregs[2]
#define ARM_r1 uregs[1]
#define ARM_r0 uregs[0]
#define ARM_ORIG_r0 uregs[17]
#define ARM_VFPREGS_SIZE (32 * 8 + 4)
#endif
#define SP(i) *((__u32*)regs->ARM_sp+i)
#endif
```
