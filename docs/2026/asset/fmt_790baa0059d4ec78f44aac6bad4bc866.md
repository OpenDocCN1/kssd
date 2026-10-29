
# 一个Android通用`svc`跟踪以及hook方案——Frida-Seccomp

暂时只支持ARM64，ARM32的逻辑也是一样的，有兴趣的大佬可以自行更改哈。

## 效果

### 对 `openat` 进行跟踪

![](https://bbs.kanxue.com/upload/attach/202203/903162_55EERQXHHTXD4FW.jpg)

### 对 `recvfrom` 进行跟踪

![](https://bbs.kanxue.com/upload/attach/202203/903162_AE84XGKCDB8RBY6.jpg)

**在这里感谢珍惜大佬介绍的seccomp机制，推荐一波珍惜大佬的课程能学到很多有趣的骚操作。**

## 什么是seccomp

[seccomp沙箱机制介绍文章](<https://bbs.kanxue.com/elink@2b8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8G2L8r3I4#2P5q4)9J5k6h3y4U0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9&6i4K6u0r3x3U0u0Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6t1#2c8e0k6Q4x3U0g2n7x3W2)9J5y4e0V1&6i4K6t1#2c8e0N6Q4x3U0g2m8c8g2)9J5y4f1t1I4i4K6t1#2c8e0k6Q4x3U0f1&6b7#2)9J5y4f1u0m8i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4f1t1$3i4K6t1#2x3U0m8Q4x3U0k6S2L8i4m8Q4x3@1u0Q4x3U0f1J5x3o6t1H3x3e0W2n7P5i4c8W2b7#2c8r3i4K6t1#2x3U0m8h3d9g2m8Q4x3V1k6Q4x3U0y4Q4x3U0g2q4z5g2)9J5y4e0R3H3i4K6t1#2z5f1q4Q4x3U0g2q4z5q4)9J5y4f1u0r3i4K6t1#2z5o6N6Q4x3U0g2q4y4q4)9J5y4f1u0p5i4K6t1#2b7V1k6Q4x3U0g2q4y4#2)9J5y4e0V1@1i4K6t1#2b7e0S2Q4x3U0g2q4z5q4)9J5y4f1q4r3i4K6t1#2b7e0g2Q4x3U0g2q4y4g2)9J5y4f1u0m8i4K6t1#2z5e0y4Q4x3U0g2q4y4#2)9J5y4e0W2m8i4K6t1#2z5o6c8Q4x3U0g2q4y4g2)9J5y4e0R3%4i4K6t1#2b7V1c8Q4x3U0g2q4y4W2)9J5y4e0V1#2i4K6t1#2b7U0m8Q4x3U0g2q4y4g2)9J5y4f1q4q4i4K6t1#2z5f1g2Q4x3U0g2q4y4#2)9J5y4e0S2q4i4K6t1#2b7U0m8Q4x3U0g2q4y4#2)9J5y4f1p5$3i4K6t1#2z5o6q4Q4x3U0g2q4y4#2)9J5y4e0V1@1i4K6t1#2b7e0S2W2P5r3g2U0N6X3g2Q4x3U0g2q4y4#2)9J5y4f1t1K6i4K6t1#2b7V1u0Q4x3U0g2q4y4#2)9J5y4f1u0n7i4K6t1#2z5f1k6Q4x3U0g2q4z5q4)9J5y4f1t1H3i4K6t1#2z5o6y4Q4x3U0g2q4y4#2)9J5y4e0V1@1i4K6t1#2b7e0R3>)

seccomp 是 Linux 内核提供的一种应用程序沙箱机制，主要通过限制进程的系统调用来完成部分沙箱隔离功能。seccomp-bpf 是 seccomp 的一个扩展，它可以通过配置来允许应用程序调用其他的系统调用。

## 如何和frida结合

### 基本原理

这是一个bpf规则：

```c
struct sock_filter filter[] = {
    BPF_STMT(BPF_LD + BPF_W + BPF_ABS,
             (offsetof(struct seccomp_data, nr))),
    BPF_JUMP(BPF_JMP + BPF_JEQ + BPF_K, nr, 0, 1),
    BPF_STMT(BPF_RET + BPF_K, SECCOMP_RET_TRAP),
    BPF_STMT(BPF_RET + BPF_K, SECCOMP_RET_ALLOW),
};
```

seccomp的具体用法可以参考「**什么是seccomp**」中的seccomp介绍文章。当返回规则设置为「`SECCOMP_RET_TRAP`」，目标系统调用时seccomp会产生一个`SIGSYS`系统信号并软中断，这时就可以通过捕获这个`SIGSYS`信号获得`svc`调用和打印具体参数。

### 如何脚本化安装seccomp规则呢

这里使用Frida的API「`CModule`」，`CModule`提供强大的动态编译功能可以让你在JS中写C，

**frida文档中的示例**

```js
const cm = new CModule(`
#include <stdio.h>
void hello(void) {
printf("Hello World from CModule\\n");
}
`);
const hello = new NativeFunction(cm.hello, 'void', []);
hello();
```

### 如何捕获异常

使用Frida的API「`Process.setExceptionHandler`」即可捕获异常并在自己写的回调中进行数据处理。

数据处理的逻辑解释写在注释里啦。

```js
// 异常处理
Process.setExceptionHandler(function (details) {
const current_off = details.context.pc - 4;
// 判断是否是seccomp导致的异常 读取opcode 010000d4 == svc 0
if (details.message == "system error" && details.type == "system" && hex(ptr(current_off).readByteArray(4)) == "010000d4") {
// 上锁避免多线程问题
lock(syscall_thread_ptr)
// 获取x8寄存器中的调用号
const nr = details.context.x8.toString(10);
let loginfo = "\n=================="
loginfo += `\nSVC[${syscalls[nr][1]}|${nr}] ==> PC:${addrToString(current_off)} P${Process.id}-T${Process.getCurrentThreadId()}`
// 构造线程syscall调用参数
const args = Memory.alloc(7 * 8)
args.writePointer(details.context.x8)
let args_reg_arr = {}
for (let index = 0; index < 6; index++) {
eval(`args.add(8 * (index + 1)).writePointer(details.context.x${index})`)
eval(`args_reg_arr["arg${index}"] = details.context.x${index}`)
}
// 获取手动堆栈信息
loginfo += "\n" + stacktrace(ptr(current_off), details.context.fp, details.context.sp).map(addrToString).join('\n')
// 打印传参
loginfo += "\nargs = " + JSON.stringify(args_reg_arr)
// 调用线程syscall 赋值x0寄存器
details.context.x0 = call_task(syscall_thread_ptr, args, 0)
loginfo += "\nret = " + details.context.x0.toString()
// 打印信息
call_thread_log(loginfo)
// 解锁
unlock(syscall_thread_ptr)
return true;
}
return false;
})
```

## 还有什么坑

### 1.syscall调用resume

#### 问题描述

根据Frida文档介绍「`setExceptionHandler`」捕获异常后只需要让回调返回`true`就会resume原本的线程，但是其只是跳过了`svc`指令继续执行，实际上并不会执行`svc`，这时候如果不执行`syscall`轻则导致APP数据异常，重则导致APP直接崩溃。所以在异常的回调中需要手动调用了`syscall`并赋值给`x0`。

但这时候会发生个新的问题，因为在主线程开启seccomp后，主线程和其后创建出来的线程都会被seccomp规则约束，在异常处理函数直接调用`syscall`同样会被seccomp约束再次抛出异常，就形成了“死锁”了。

#### 如何解决

可以注意到上面“死锁”部分描述，那我们在主线程被约束前，提前创建一个线程，这个线程就是不被约束的，同时受到线程池启发，我们让这个`syscall`线程循环接受任务，就能完成在一个没有约束的线程里进行`syscall`调用。

### 2.堆栈回溯

#### 问题描述

直接使用Frida的API「`Thread.backtrace`」很容易导致崩溃，原因可能是seccomp规则或者「`prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0)`」导致的权限收紧和Frida实现堆栈回溯功能冲突。

#### 如何解决

手动实现堆栈回溯，原理是Arm64中每个函数都会在函数头部位置对`x29`、`x30`寄存器存入栈中，所以可以对`x29`不断读取往上回溯，最后得到完整的堆栈信息。

**实现**

```js
function stacktrace(pc, fp, sp) {
let n = 0, stack_arr = [], fp_c = fp;
stack_arr[n++] = pc;
const mem_region = call_thread_read_maps(sp);
while (n < MAX_STACK_TRACE_DEPTH) {
if (parseInt(fp_c.toString()) < parseInt(sp.toString()) || fp_c < mem_region.start || fp_c > mem_region.end) {
break
}
let next_fp = fp_c.readPointer()
let lr = fp_c.add(8).readPointer()
fp_c = next_fp
stack_arr[n++] = lr
}
return stack_arr;
}
```

### 3.「`Process.findModuleByAddress`」「`Process.enumerateModules`」类的API导致崩溃或找不到Module信息

#### 问题描述

疑似同坑2的原因

#### 如何解决

在`CModule`中手动实现了通过地址查`soinfo`信息「`base`, `size`, `soname`」等

### 4.`write`调用约束下调用Frida的API「`send`」崩溃

#### 问题描述

同坑2的原因

#### 如何解决

直接改用`syscall`线程使用「`__android_print_log`」打印信息

## 还可以实现什么

在调用`syscall`线程前后可以更改传参、返回值、地址等更改，达到HOOK的效果

## GITHUB

**求star**

[90fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8j5X3u0T1j5X3u0A6i4K6u0r3c8Y4u0A6k6r3q4Q4x3X3c8e0k6h3y4U0L8$3#2H3](<elink@80dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8j5X3u0T1j5X3u0A6i4K6u0r3c8Y4u0A6k6r3q4Q4x3X3c8e0k6h3y4U0L8$3#2H3>)

## 如何使用

```bash
pip3 install frida
python3 multi_frida_seccomp.py
```

`log`信息可以在`logcat`过滤“seccomp”查看

同时也自动保存到了「`包名_pid_时间戳`」文件夹内（支持多进程）

![图片描述](upload/attach/202203/903162_DSYWMSBGQMMVBJ8.png)
