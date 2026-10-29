
```java
unicorn.hook_add(new CodeHook() {
@Override
public void hook(Unicorn u, long address, int size, Object user) {
Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM, Capstone.CS_MODE_THUMB);
Capstone.CsInsn[] disasm = capstone.disasm(u.mem_read(address, size), address);
for (Capstone.CsInsn ins : disasm) {
System.out.println("0x"+Long.toHexString(ins.address) + ": " + ins.mnemonic + " " + ins.opStr );
}
}
}, 0, -1, null);
```

```text
0x1000: movs r0, #3
0x1002: movs r1, #2
0x1004: add r0, r1
0x1006: bl #0x1100
0x1100: add r0, r1
0x1102: movs.w r2, #0x1200
0x1106: str.w lr, [r2]
0x110a: ldr.w pc, [r2]
0x100a: svc #0
0x100c: add r0, r1
the emulate finished result is ==> 9
```

# Unidbg的底层支持-Unicorn

- [Unidbg的底层支持-Unicorn](https://bbs.kanxue.com/#unidbg的底层支持-unicorn)

  - [Unicorn简介](https://bbs.kanxue.com/#unicorn简介)

  - [Unicorn使用步骤](https://bbs.kanxue.com/#unicorn使用步骤)

    - [引入Unicorn](https://bbs.kanxue.com/#引入unicorn)

    - [使用Unicorn](https://bbs.kanxue.com/#使用unicorn)

    - [Unicorn原生Hook](https://bbs.kanxue.com/#unicorn原生hook)

      - [CodeHook](https://bbs.kanxue.com/#codehook)

      - [BlockHook](https://bbs.kanxue.com/#blockhook)

      - [ReadHook](https://bbs.kanxue.com/#readhook)

      - [WriteHook](https://bbs.kanxue.com/#writehook)

      - [MemHook](https://bbs.kanxue.com/#memhook)

      - [InterruptHook](https://bbs.kanxue.com/#interrupthook)

      - [EventMemHook](https://bbs.kanxue.com/#eventmemhook)

  - [Keystone](https://bbs.kanxue.com/#keystone)

    - [引入](https://bbs.kanxue.com/#引入)

    - [使用](https://bbs.kanxue.com/#使用)

  - [Capstone](https://bbs.kanxue.com/#capstone)

    - [引入](https://bbs.kanxue.com/#引入-1)

    - [使用](https://bbs.kanxue.com/#使用-1)

  - [总结](https://bbs.kanxue.com/#总结)

## Unicorn简介

> cf5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2L8X3W2U0L8%4u0F1i4K6u0V1k6h3&6Y4K9h3&6W2i4K6u0r3N6h3&6A6j5$3!0J5L8R3`.`.

`Unicorn`的官网简介很简单明了：`Unicorn`是一个基于Qemu的轻量级的多平台、多架构的 CPU 模拟器框架。一句话让我们明白了它是做什么的。在Unidbg对一个Elf文件进行模拟执行的时候，我们一般是在跨平台运行的，所以就底层就需要一个模拟器Backend。其中一种就是`Unicorn`，下面我们就来看看`Unicorn`的使用

## Unicorn使用步骤

- 语言: Java
- 架构: Arm

我们使用一个例子，就可以讲述`Unicorn`的使用。

### 引入Unicorn

`Unicorn`是开源的，可自行编译。凯神已经编译好了，可以直接使用

```xml
<dependency>
    <groupId>com.github.zhkl0228</groupId>
    <artifactId>unicorn</artifactId>
    <version>1.0.12</version>
</dependency>
```

### 使用Unicorn

我们先来写一段简单的Thumb汇编代码

```asm
movs r0, #3
movs r1, #2
add  r0, r1
```

这三行汇编如果被CPU执行，结束后R0寄存器的值应该为5对吧。那么我们就看一下用`Unicorn`如何来模拟执行这三条指令

`Unicorn`是不认识汇编代码的，它跟CPU一样，只认识机器码，所以我们可以去下面的网站在线转换成机器码:

> 9aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3#2U0L8$3&6$3k6i4u0@1k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.

转换成机器码后

```asm
0x0320 0x0221 0x0844
```

```java
public class UnicornTest {
    long BASE = 0x1000;

    @Test
    public void test(){
        // 先来定义我们的机器码
        byte[] code = new byte[]{
            0x03, 0x20,
            0x02, 0x21,
            0x08, 0x44
        };

        // 创建 Unicorn 对象, 它就像一个CPU, 我们可以使用它的接口来操作这个CPU
        // 参数一：架构
        // 参数二：运行模式(在ARM、Thumb中无关紧要，最终还是靠运行时判断)
        Unicorn unicorn = new Unicorn(UnicornConst.UC_ARCH_ARM, UnicornConst.UC_MODE_THUMB);

        // mem_map 进行内存映射，在使用Unicorn提供的内存时，必须先进行映射
        // 参数一：起始地址
        // 参数二：映射内存区域的大小
        // 参数三：内存操作权限
        unicorn.mem_map(BASE, 0x1000, UnicornConst.UC_PROT_WRITE | UnicornConst.UC_PROT_READ | UnicornConst.UC_PROT_EXEC);

        // mem_write 可以进行对映射出的内存进行写入操作
        // 参数一：写入的地址
        // 参数二：写入的内容
        unicorn.mem_write(BASE, code);

        // emu_start 开始执行CPU
        // 参数一：开始执行的地址(Thumb指令地址+1)
        // 参数二：结束地址(当结束地址命中时，结束执行)
        // 参数三：超时时间(ms)，当该值为0时，Unicorn将在无限时间内模拟代码，直到模拟完成
        unicorn.emu_start(BASE + 1, BASE + code.length, 0, 0);

        // 此时三条Thumb指令已经模拟完成
        // reg_read 可以读取寄存器，相应的reg_write可以写入寄存器的值
        // 参数：寄存器常量
        Long o = (Long) unicorn.reg_read(ArmConst.UC_ARM_REG_R0);

        System.out.println("the emulate finished result is ==> " + o.intValue());

    }
}
```

我们来看下执行结果, 符合我们的预期

```text
the emulate finished result is ==> 5
```

对于上面代码的补充：在`mem_map`的时候，参数一起始地址必须是1K(32位下)/4K(64位下)对齐的，否则会抛出Invalid argument (`UC_ERR_ARG`)异常，参数三的操作权限可参考下表:

```java
public static final int UC_PROT_NONE = 0;
public static final int UC_PROT_READ = 1;
public static final int UC_PROT_WRITE = 2;
public static final int UC_PROT_EXEC = 4;
public static final int UC_PROT_ALL = 7;
```

### Unicorn原生Hook

#### CodeHook

```java
// hook_add 添加一个Hook(后面的Hook参数通用，只有第一个不同)
// 参数一：Hook回调
// 参数二：Hook起始地址
// 参数三：Hook结束地址
// 参数四：自定义参数，可以在Hook的回调中拿到，也就是 Object user
unicorn.hook_add(new CodeHook() {
    /*
     * CodeHook将注册UC_HOOK_CODE类型的Hook，它将在Unicorn对起始地址跟结束地址中间每一条指令的执行时进行调用，
     * 当结束地址 < 起始地址时，则将对每一条指令执行时进行调用，相当于Trace
     */
    /*
     * 回调函数有当前Unicorn对象，所以我们可以基于此对象做关于CPU的任何事情
     */
    @Override
    public void hook(Unicorn u, long address, int size, Object user) {
        System.out.print(String.format(">>> Tracing instruction at 0x%x, instruction size = 0x%x\n", address, size));
    }
}, 0, -1, null);
```

```text
>>> Tracing instruction at 0x1000, instruction size = 0x2
>>> Tracing instruction at 0x1002, instruction size = 0x2
>>> Tracing instruction at 0x1004, instruction size = 0x2
the emulate finished result is ==> 5
```

#### BlockHook

```java
/*
 * BlockHook将注册UC_HOOK_BLOCK类型的Hook，当输入的基本块且基本块的地址(BB)在起始地址<=BB<=结束地址范围内时，
 * 将调用已注册的回调函数。当结束地址 < 起始地址时，输入任何基本块时，将调用回调
 */
/*
 * 基本块：在Unicorn中，基本块就相当于未发生跳转的所有指令为一基本块，发生跳转就是另一块
 */
unicorn.hook_add(new BlockHook() {
    @Override
    public void hook(Unicorn u, long address, int size, Object user) {
        System.out.print(String.format(">>> Tracing basic block at 0x%x, block size = 0x%x\n", address, size));
    }
}, 0, -1, null);
```

```text
>>> Tracing basic block at 0x1000, block size = 0x6
the emulate finished result is ==> 5
```

如果我们执行一段有跳转的汇编(后面我们测试也使用这段汇编代码)

```asm
0x1000: movs r0, #3
0x1002: movs r1, #2
0x1004: add r0, r1
0x1006: bl #0x1100
0x100a: add r0, r1
...
0x1100: add r0, r1
0x1102: movs.w r2, #0x1200
0x1106: str.w lr, [r2]
0x110a: ldr.w pc, [r2]
...
```

那么我们再来看BlockHook的结果

```text
>>> Tracing basic block at 0x1000, block size = 0xa
>>> Tracing basic block at 0x1100, block size = 0xe
>>> Tracing basic block at 0x100a, block size = 0x2
the emulate finished result is ==> 9
```

#### ReadHook

## 总结

上面我们介绍了`Unicorn`的详细使用，有了`Unicorn`的加持，我们就可以模拟Arm架构的指令集，从而执行So中的各种指令，当然多条指令组合成的函数都是可以的。我们可以看到`Unicorn`本身提供了丰富的API供我们使用，它本身提供的Hook，覆盖了我们需要实现的所有场景。最后介绍了`Keystone`跟`Capstone`这对兄弟，那么本次的分享就结束啦，对本块内容有兴趣的朋友可以加个VX一起学习呀: roy5ue
