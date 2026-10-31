
# [原创]linux pwn防守篇题目投稿(`malloc_consolidate`+`unlink`+rop)

## 考点

`malloc_consolidate`+`unlink`+rop

## 运行环境

1. amd64（作者本地测试为kali，远程测试环境为ubuntu16.04），无`libc`要求，已经上传远程测试`libc`
2. 开启aslr

## 编译方式

`gcc -no-pie main.c -o main ; strip main`

## 题目说明

1. 题目可以给出`bin`与`libc`供下载，也可以仅给出`bin`文件，通过查找`libc-db`的方式找到对应`libc`版本号（作为一只pwn狗，建议给出`libc`，省去大家查`libc`的浪费的生命。。QAQ都是血泪史）

## 设计思路

1. 首先给用户输入姓名，此时以`malloc`一个chunk的方式存储用户的输入
2. 给了4个功能，`create`、`delete`、`edit`、`show`，其中`show`功能无效

![](https://bbs.kanxue.com/upload/attach/201705/723671_lapzslhow5wcxfd.png)

3. `create`函数可以申请一个小于4096字节的chunk，并往里面写入数据，然后置`flag`位为1，同时用一个全局变量`number`来记录已申请的chunk个数，`number`不得大于4.
4. `delete`函数可以`free`一个指针并置`flag`位为0，但是不检查是否已经`free`这个指针
5. `edit`检查`flag`位，只能修改已经`flag`为1的chunk
6. 数据结构如下

![](https://bbs.kanxue.com/upload/attach/201705/723671_rjqfd98wwz1sc3a.png)

## 漏洞点

1. uaf，在dele一个指针后没有清零，可以再次`free`这个freed的指针
2. 漏洞什么的。。应该没了吧。。

## 漏洞利用

1. 主要利用`fastbin`的`malloc_consolidate`这个函数来造成`unlink`，后面再布置栈构造`ropchain`即可。
2. 在申请large bin的时候，会将freed `fastbin`的`inuse`位清零，同时进行合并，将合并后的堆块放入`unsortbins`中。然后遍历`unsortbins`，按照大小分别放入`smallbins`和`largebins`中。

![](https://bbs.kanxue.com/upload/attach/201705/723671_4j2zxli7cys6csk.png)

![](https://bbs.kanxue.com/upload/attach/201705/723671_bgpq2jp3ds86xw8.png)

3. 这时利用uaf，`free`一个`fastbin`，也就是我们刚才申请的0x30大小的块，将它链入`fastbins`的单向链表中。因为当我们从`fastbins`中分配不会置后一个chunk的`inuse`位为1，但是由于之前的`malloc_consolidate`已经使得该`fastbin`的后一个chunk的`inuse`位为0，所以造成一个矛盾，是的我们能够`unlink`

![](https://bbs.kanxue.com/upload/attach/201705/723671_yk0iafvv91sc9i9.png)

4. 分配一个0x20大小的chunk，置后一个chunk的`presize`位为我们刚才分配的`fastbin`的大小，为`unlink`做准备，0x20会从之前合并的`smallbins`中切割，然后该`smallbins`移到`unsortbins`中，并成为`last_remainder`。

![](https://bbs.kanxue.com/upload/attach/201705/723671_q5edgm5usv393q8.png)

5. 修改0x30的`fastbin`，并填充`unlink`的payload，再在之后分配一个`smallbin`大小的chunk，使得`unsortbin`中的chunk移到`smallbins`（因为`unsortbins`有一个`check`，会检查第一个chunk的`bk`是否指向`unsortbin`的头），最后`delete`触发`unlink`。

![](https://bbs.kanxue.com/upload/attach/201705/723671_jen7lmnbjq3eva4.png)
![](https://bbs.kanxue.com/upload/attach/201705/723671_1xbi21gpykk5fu1.png)

6. 接下来的事情就简单了，修改`fflush`的`got`表为`add rsp 8; ret`的gadget，然后用rop泄露libc基址，最后修改`free_got`为`system`即可。

![](https://bbs.kanxue.com/upload/attach/201705/723671_061h1mw05fepgrp.png)

7. `free`一个事先写好`/bin/sh\0`的chunk拿shell即可。

![](https://bbs.kanxue.com/upload/attach/201705/723671_a3p48bzadp3txdn.png)

**PS:**

因为可以`malloc`任意大小，还有uaf，漏洞略大，但已经尽力避免未预料解法了。。选中的话希望不要被师傅们秒掉。。

**PPS：**

好像国内的CTF还没见过考`malloc_consolidate`的。。也可能我比赛打的少了XD

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [`main`](javascript:void(0);)（10.15kb，10次下载）
- [`main.c`](javascript:void(0);)（2.24kb，16次下载）
- [`exp.py`](javascript:void(0);)（2.05kb，15次下载）
- [`flag`](javascript:void(0);)（0.02kb，6次下载）
