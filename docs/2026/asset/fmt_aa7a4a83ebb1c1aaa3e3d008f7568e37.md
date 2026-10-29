
`(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`

Starting program: `/home/reader/booksrc/a.out` `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`

Breakpoint `1`, `main` (`argc=2`, `argv=0xbffff874`) at `auth_overflow2.c:24`

`24`              `if(check_authentication(argv[1]))` {

`(gdb) i r esp`

`esp`            `0xbffff7e0`       `0xbffff7e0`

```
(gdb) x/32xw $esp
0xbffff7e0:     0xb8000ce0      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
0xbffff820:     0x40f5f7f0      0x48e0fe81      0x00000000      0x00000000
0xbffff830:     0x00000000      0xb7ff9300      0xb7eafded      0xb8000ff4
0xbffff840:     0x00000002      0x08048350      0x00000000      0x08048371
0xbffff850:     0x08048474      0x00000002      0xbffff874      0x08048510
```

`(gdb)`

第一个断点在调用`check_authentication()`函数之前的`main()`函数中。此时堆指针寄存器`ESP`的地址是`0xbffff7e0`，栈顶如上所示。这是`main()`堆栈的所有组成部分。继续下一个断点，这个断点在`check_authentication()`函数内部，如下输出所示`ESP`变小了，是因为它为`check_authentication()`的堆栈开辟空间而提升内存的表。在找到`auth_flag`变量和`password_buffer`变量后，在堆栈中还可以看到具体数据。

代码如下:

```
(gdb) c
Continuing.
Breakpoint 2, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>) at
auth_overflow2.c:9
9               strcpy(password_buffer, password);
```

`(gdb) i r esp`

`esp`            `0xbffff7a0`       `0xbffff7a0`

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

`(gdb) p 0xbffff7e0 - 0xbffff7a0`

`$1 = 64`

```
(gdb) x/s password_buffer
0xbffff7c0:      "?o??\200????????o???G??\020\205\004\b?????\204\004\b????\020\205\004\bH???????\002"
```

```
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
```

`(gdb)`

继续分析`check_authentication()`中的第二个断点，当函数调用时执行一个压栈操作。因为堆栈的生长方向是朝着低内存地址的，因此目前的堆栈指针是比第一个断点时少64字节的`0xbffff7a0`。堆栈大小和结构很大程度上取决于函数和某些编译器优化程度。例如，这个堆栈的开始的24个字节是编译器生成的内容。局部堆栈变量`auth_flag`和`password_buffer`则在堆栈中它们各自的内存单元。`auth_flag`在`0xbffff7bc`，16字节的`password_buffer`在`0xbffff7c0`。

堆栈中不仅包括局部变量和编译器信息还包括更多的信息。`check_authentication()`堆栈的详细组成如下所示。

首先，被用来保存局部变量的内存用斜体字表示。从`auth_flag`变量开始于`0xbffff7bc`直到16字节的`password_buffer`变量结束。堆栈中接下来的一些数据是编译器写入的无用信息，附加了一些被称作存储附加指针的数据。如果程序使用`-fomit-frame-pointer`参数进行编译，就不会再堆栈中使用框架指针。`0x080484bb`就是堆栈返回地址，地址`0xbffffe9b7`是一个指向包含30个A的字符串指针。这是`check_authentication()`函数的参数。

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
```

```
(gdb) x/32xb 0xbffff9b7
0xbffff9b7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9bf:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9c7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9cf:     0x41    0x41    0x41    0x41    0x41    0x41    0x00    0x53
```

```
(gdb) x/s 0xbffff9b7
0xbffff9b7:      'A' <repeats 30 times>
```

`(gdb)`

通过了解堆栈是怎么被创建的可以定位堆栈中的返回地址。进程开始于`main()`，这是在函数调用之前的。

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
```

`(gdb)`

注意上面两条指令。此时`EAX`寄存器包含一个指向第一个命令行参数的指针。这也是`check_authentication()`函数的参数。第一条汇编指令将`EAX`的值写入到`ESP`所指的地址（栈顶）。`check_authentication()`函数参数从这个堆栈开始的。第二条指令是实际调用。这条指令将下一条指令的地址压入栈并移动执行指针寄存器`EIP`到`check_authentication()`函数开始处。压入堆栈的地址就是函数调用后的返回地址。因此下一个指令的地址是`0x080484bb`也就是函数调用的返回地址了。

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
```

`(gdb) p 0x38`

`$3 = 56`

`(gdb) p 0x38 + 4 + 4`

`$4 = 64`

程序继续执行到`check_authentication()`函数中，此时`EIP`发生改变，如上所示开始的几条指令完成保存堆栈功能。这些指令通常是函数开始时的固定指令 。前两条指令是为了存储堆栈指针，第三条指令是将`ESP`减`38`，为函数的局部变量保存56字节。返回地址和存贮框架指针已经压入栈，也就是64字节堆栈中8个字节的作用了。

当函数完成时，`leave`和`ret`指令清除堆栈并设置执行指针寄存器`EIP`来保存堆栈中的返回地址。这使得程序在函数调用后执行到`main()`函数中的下一条指令，也就是地址`0x080484bb`处。这个过程在任何程序的函数调用时都会发生。

Code View:

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
```
