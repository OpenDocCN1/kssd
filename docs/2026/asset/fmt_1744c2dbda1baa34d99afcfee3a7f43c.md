
# [原创]rop链攻击原理与思路(x86/x64)

## rop链的基础原理

* `rop`是Return Oriented Programming（面向返回的编程）的缩写；
* 根据函数调用规则，当刚跳转到其它函数去执行时，**从被调用者的视角看：栈顶是返回地址，紧接着是自己的参数**（`X86`架构下）；
* 然后，被调用者会对栈空间进行一系列操作，保存寄存器和存储临时变量，但在即将退出时会清理自己消耗的栈空间，以使其回到自己被调用前的栈空间，**保持栈平衡**；
* 最后，被调用者**以`ret`指令结尾**，`ret`指令将栈顶地址传递到`IP`寄存器，随后**代码也就跳转到之前栈顶存放的返回地址处**；
* `rop`链即是基于以上这个简单的原理，在代码空间中寻找以`ret`结尾的代码片段或函数（代码片段称为`Rop gadgets`），组合可以实现拓展可写栈空间、写入内存、shell等功能，**依靠`ret`将代码执行权紧握在自己的手里**;
* 早期的`ret2libc`原理也是基于`ret`，如下`payload`：`libc_system`函数地址位置处于存在栈溢出函数被调用前的栈顶，溢出函数执行完后，就会跳到这个地址开始执行，下一个地址大小是`libc_system`函数执行完的返回地址，在下一个地址大小开始是`libc_system`函数的参数。

```payload = "A"*n + p32(libc_system) + p32(final_ret) + arg_addr + ...```

## 构造rop链的实例（x86）

* 漏洞特征
  **描述：** 假如存在这样一个漏洞：造成了栈溢出，可以溢出数个地址大小，至少能覆盖掉`ret`返回地址（`EIP`）和后面需要布置的一些参数空间。
  **环境：** `ubuntu 16.04 32`， 开了`ASLR`、`DEP`，关闭`CANNARY`，目标代码不基址随机化，`plt`和`got`表项中存在`write`或`print`等可以打印地址的函数，有目标机器的`libc.so`
* 利用思路：
  1. `payload1`：跳转到`write`或`print`函数（`plt`处代码），构造参数使其打印`got`表中某项地址，并在地址泄露任务完成后跳回存在溢出的函数，使其可以被再次利用；
  2. 搜寻`libc.so`中`system`函数的地址和字符串`"/bin/sh"`，使用泄露地址与其相应偏移算出`system`函数和`"/bin/sh"`的真实加载地址；
  3. `payload2`: 填充`ret`地址处`system`函数加载地址，并为其布置`"/bin/sh"`地址作为参数;
  4. `get shell`，exploits如下所示：
  ![](upload/attach/202001/704018_7PYGCK479S3XQUD.png)
* 无目标机器的`libc.so`的情况下
  利用`pwntools`里的`DynELF`泄露`system`函数地址。如下，其实就是写一个泄露一个地址大小的函数：**输入一个地址，打印这个地址中存储的值（大小也为一个地址大小），并且保证可以被无数次调用**，大部分`DynELF`泄露地址姿势都如此。
  ![](upload/attach/202001/704018_MD889FBDKE4PPJU.png)

## 构造rop链的实例（x64）

* `x64`指令在栈空间操作的变化有两个：
  - 地址变成了64位;
  - 函数传参从`x86`的栈传递变成寄存器传递，前6个参数分别使用`rdi`、`rsi`、`rdx`、`rcx`、`r8`、`r9`，多于6个的参数使用栈传递。
* 漏洞特征
  **描述：** 栈溢出，可以溢出数个地址大小的空间。
  **环境：** `ubuntu 16.04 64`， 开了`ASLR`、`DEP`，关闭`CANNARY`，目标代码不基址随机化，`plt`和`got`表项中存在`write`或`print`等可以打印地址的函数，有目标机器的`libc.so`。
* 利用思路
  1. 搜索`gadget1`：可以实现写参数到`rdi`、`rsi`、`rdx`，然后调用`write`函数泄露`libc`地址；
  2. 利用泄露`libc`的函数真实地址与`system`和`"/bin/sh"`中的偏移计算`system`和`"/bin/sh"`真实加载地址；
  3. 再利用`gadgets1`是`rdi`指向`libc`中的`"/bin/sh"`，然后跳转到`system`地址执行。
* 实例调试

### 泄露libc地址

+ `linux x64`下通用的构造参数的`gadget`，`__libc_csu_init`函数是`linux`运行库重要函数，在可执行程序执行之前执行，它里面正好有一段可以用来传递参数+跳转到任意地址执行的超级`gadget`，`__libc_csu_init`函数`gadgets`片段如下图所示：
![](upload/attach/202001/704018_ZTK9K9DZ8KSSSXS.png)
1) `gadget1`从标记1处开始，把栈顶数据依此弹出到`rbx`、`rbp`、`r12`、`r13`、`r14`、`r15d`；
2) `gadget2`从标记2处开始，会把`r13`、`r14`、`r15d`中的数据分别赋值到`rdx`、`rsi`、`edi`，其中`r15d`和`edi`分别表示`r15`和`rdi`的低32位空间，所以到此为止可以控制函数的前三个参数值；
3) 到地址`0x40060`处时，`gadget2`虽然完成了寄存器参数的布置，但还没有`ret`结束掉这个`gadget`，所以还会往下执行；
4) 3处会使用上面3条指令传递好的参数去执行某函数，函数地址是`r12+rbx*8`处存的值，由于`r12`和`rbx`也可以在`gadget1`中被控制，在此处将[`r12+rbx*8`]指向`write`函数地址就可以实现地址泄露；
5) 3处执行完地址泄露后，由于无`ret`指令不能返回我们可以控制的地址中去，再往下看到4处当`rbx`不等于`rbp`时会跳到`gadget2`处不断打印刚才泄露的地址，因此我们需构造`rbx==rbp`，这样就会再次执行`gadget1`，然后通过`ret`返回到下一次漏洞函数，等待第二个`payload`。
到目前为止我们可以实现地址泄露，将`rbx`构造为0，那么`rbp`为1时可以从`gadget1`执行进入`gadget2`，那么`r12`中存入`write`函数在`got`表中地址，可在`gadget2`的3处执行地址打印，因此构造如下`payload`：

```payload1 = 'A'*offsets + p64(0)#rbx值 + p64(1)#rbp值 + p64(got_write) + p64(8)#write函数第3个参数 + p64(got_write)#write函数第2个参数,泄露的地址 + p64(1)#write函数第1个参数 + p64(gadget2) + 'A'*56 + p64(main)```

调试该`payload`，在`0x400609`处下断点：
断下时寄存器值已经变成我们想要的，如下图所示：
![](upload/attach/202001/704018_NMA2637MSEZ8QB2.png)
`r12`中是即将要跳转的打印函数，打印函数的`fd`、`addr`、`size`也已经布置好，最终泄露的地址如下：
![](upload/attach/202001/704018_T6PB6TJ2EFKGZAW.png)

### 找到合适的"/bin/sh"

通过上面可以看到`libc`中的`"/bin/sh"`地址为`0x7f42f5e3ed57`，而我们利用的`gadget`只能控制`rdi`的低32位，放不下`"/bin/sh"`的完整地址，因此无法用`libc`中泄露的`"/bin/sh"`。
由于目标函数没有`aslr`，且地址空间的高32位为0，因此可以写入`"/bin/sh"`到目标代码的`.bss`段。

```payload2='A'*0x58 + p64(gadget1) + p64(0) + p64(1) + p64(got_read) + p64(16) + p64(bss_addr) + p64(0) + p64(gadget2) + 'A'*56 + p64(main)```

如下图，成功将`"/bin/sh"`写入目标代码的`.bss`段：
![](upload/attach/202001/704018_PBBMGJHRTA3UAHV.png)

### 利用bss段的"/bin/sh"和泄露的system函数地址，执行shell:

改写一下`payload2`后一段，使其执行完后，不跳转到`main`函数，而是直接调转到`system`函数：
如果想利用`400609 call qword ptr [r12+rbx*8]`跳转到`system`，那么需要将`system`函数写入某个内存中，并将`r12+rbx*8`指向该内存，改写一下`payload2`使其一次性写入`system`地址和`"/bin/sh"`;执行完写后，再次通过`gadget1`将存放`"/bin/sh"`的`bss`地址传递到`rdi`，存放`system_addr`的`bss`地址传递到`r12`，如下：

```payload2='A'*0x58 + p64(gadget1) + p64(0) + p64(1) + p64(got_read) + p64(16) + p64(bss_addr) + p64(0) + p64(gadget2)
+ 'A'*8 + p64(0) + p64(1) + p64(bss_addr) + p64(0) + p64(0) + p64(bss_addr+8) + p64(gadget2) + ...```

`p.send(p64(system_addr) + "/bin/sh\x00")`
写入`system`地址和`"/bin/sh"`时的断点：
![](upload/attach/202001/704018_M72UQA6Z7GAD4U6.png)
`system`地址写到了`bss_addr`，`"/bin/sh"`写到了`bss_addr+8`位置。
再此执行到`gadget2`中时：
![](upload/attach/202001/704018_XYFECWD7MHNNC4M.png)
`r12+rbx*8 == r12 == 0x601040` 指向`system`函数，而`RDI`指向字符串`"/bin/sh"`，万事俱备;往下执行即可以获得`shell`：
![](upload/attach/202001/704018_WJT9BAA2QFG54SZ.png)
