
```c
if (\_\_glibc\_unlikely (chunksize(p) != prevsize))
malloc\_printerr ("corrupted size vs. prev\_size while consolidating");
```

`unlink`时候的检测。

```c
if (\_\_builtin\_expect (FD->bk != P || BK->fd != P, 0))
malloc\_printerr (check\_action, "corrupted double-linked list", P, AV);
```

很明显全部满足！bingo！最后`free` `chunk24`构造`off-by-null`成功：

![](upload/attach/202003/813771_F85MV7XER6EFVRJ.png)

6.泄露`libc`和`getshell`。

```python
# leak
add(0x40, "a")
show(19)
libc_base = u64(ru("\x7f").ljust(8, "\x00"))-0x3b2ca0
success(hex(libc_base))
```

最后有了堆块复用就随便玩了，后续步骤就不说了。

## 最后

`2.29`下的`off-by-null`到这里就算分析完了，思路是来自于<a href="elink@b20K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2L8$3&6W2N6#2)9J5k6h3y4F1i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5I4x3U0x3K6">EX师傅的博客</a>，我只是详细地对其中的关键点解释了一下。

当然`2.29`下还有一些其他的很有意思的新思路和利用手法，这里因为我在过几天的某比赛的出题里用到了这种手法，所以我会在比赛结束后再分享给大家。

## 前言

在看雪发的第二篇帖子，回归老本行`pwn`。

在经典的`2.23`版本的`glibc`下，`off-by-null`算得上是堆利用的一种中等难度的利用技巧，简单来说，这种漏洞发生在读入数据的过程，存在多输入一个字节并且该字节固定为`\x00`的问题，利用这一个看似不可控却很有作用的字节，攻击者就可以精心构造出堆上的布局，进行任意地址的读写，构造的过程也可以说十分巧妙。

不过慢慢在各种`CTF`赛题的狂轰乱炸之下，`off-by-null`也算形成了比较固定的公式，开始被玩坏了。

想到目前已经`2020`年了，希望出题人也好，选手也好，不要再把思路局限于`2.23`了，甚至于`2.27`也算是一个相对过时的版本了，因为除了多了一个`tcache`机制，其实等于裸奔的`fastbins`，把`tcache`塞满之后，其余的区别并不大。所以在这里，我会详细解析一下在各项保护更加完善的`2.29`下，是如何进行`off-by-null`的，希望可以起到一些抛砖引玉的作用。

## 2.23

先从构造起来最简单最基本的`2.23`说起，其实也是一个自己复习知识的过程，由简入繁，才能更好地理解问题。

这里我画了一个简单的`2.23`下`off-by-null`的构造流程图，如果你对这个过程足够熟练，看着这个图就可以撸出`2.23`下`off-by-null`的`exp`了：

![](https://bbs.kanxue.com/upload/attach/202003/813771_RDMN83VJVTAFXQG.png)

这里说前向合并或者后向合并容易产生歧义，所以接下来，我会用高低地址来表示。先简述一下流程：

1. 这里我们首先先开辟出若干堆块，`chunk0`，`chunk2`，以及它们之间的若干堆块和`chunk2`高地址的堆块，
2. 让`chunk0`进入`unsortedbin`之中。在`chunk2`下面的`chunk`构造一个假的堆头部。
3. 用`chunk2`紧邻低地址的`chunk`去进行`off-by-null`，覆写掉`chunk2`的`size`字段，图里就是最低字节被覆盖为了`\x00`变成了`0x100`，这里不管是`0x100`还是`0x1ff`，都是会被覆盖成`0x100`的。同时也要覆盖掉`pre_size`字段，使其通过`prevsize`能够找到`chunk0`。
4. `free` `chunk2` 达成布局。再次分配就可以达成`chunk0`-`chunk2`之间区域的重复利用，达成`UAF`的效果。

这里主要利用到了`free`时的两个机制，一个是不满足`fastbin`的时候，会判断是否向低地值合并：

```c
/* consolidate backward */
if (!prev_inuse(p)) {
    prevsize = p->prev_size;
    size += prevsize;
    p = chunk_at_offset(p, -((long) prevsize));
    unlink(av, p, bck, fwd);
}
```

`p`就是`chunk2`，因为此时`chunk2`的`P`被`off-by-null`盖成了`0`，所以开始向低地值合并，通过`prevsize`找到了需要合并的`chunk`，这里也就是`chunk0`，然后触发第二个机制，也就是`unlink`。

`unlink`的作用就是在`bin`双向链表上把其中的一个`chunk`卸载下来，曾经稚嫩的`unlink`还是个宏，后来地位上升了，现在早已经变成函数了。部分能用得到的宏代码：

```c
#define unlink(AV, P, BK, FD) { \
    FD = P->fd; \
    BK = P->bk; \
    if (__builtin_expect (FD->bk != P || BK->fd != P, 0)) \
      malloc_printerr (check_action, "corrupted double-linked list", P, AV); \
    else { \
        FD->bk = BK; \
        BK->fd = FD; \
...
```

这里的`P`已经就是图中的`chunk0`了。有一段该死的检测：

```c
if (__builtin_expect (FD->bk != P || BK->fd != P, 0))
      malloc_printerr (check_action, "corrupted double-linked list", P, AV);
```

主要是在检测`P`是否在一个真正的双向链之中。这也正是`off-by-null`构造的巧妙之处，`chunk0`被我们提前丢进了`unsortedbin`之中，所以当对其检测时，就利用了系统已经帮我们构造好了的链表绕过了检测。 （这里的`chunk0`的`fd`和`bk`里面的值相同，都是`0x7f....`这种，这就是`unsortedbin`这条双向链的链表头，该链因为只插进了`chunk0`这一个元素，所以链表头里的`fd`和`bk`也会指向`chunk0`）

别急，接下来还有一步要处理，低地址检测之后还会检测是否要和高地址合并：

```c
if (nextchunk != av->top) {
    /* get and clear inuse bit */
    nextinuse = inuse_bit_at_offset(nextchunk, nextsize);
    /* consolidate forward */
    if (!nextinuse) {
        unlink(av, nextchunk, bck, fwd);
        size += nextsize;
    } else
        clear_inuse_bit_at_offset(nextchunk, 0);
}
//...
//...
```

这里也就是检测下一个`chunk`的下一个`chunk`的`P`位，因为当前`chunk`是否被使用是标记在比它地址更高的下一个堆块的`P`位上的。这也是图里步骤`2`要构造的东西，为了不让系统执行向高地址合并，用正常的`chunk`，或者用自己伪造的`chunk`都是可以的。

之后`chunk0`直到`chunk2`，同时连带着中间的这么一大块空间，就都被送进了`unsortedbin`，当我们再次从`unsortedbin`切割的时候，就会分配出中间的各种地址，达成堆块复用的效果了。

## 2.29

`2.27`我就不介绍了，把`tcache`填满了，其他的跟`2.23`没什么区别。

`2.29`的代码进行了更加完善的保护，在向低地值合并的时候，加了一行代码：

```c
/* consolidate backward */
if (!prev_inuse(p)) {
    prevsize = prev_size (p);
    size += prevsize;
    p = chunk_at_offset(p, -((long) prevsize));
    if (__glibc_unlikely (chunksize(p) != prevsize))
        malloc_printerr ("corrupted size vs. prev_size while consolidating");
    unlink_chunk (av, p);
}
```

检测了`chunksize(p) != prevsize`，以图为例，就是检测`chunk0`的`size`和`chunk2`的`prevsize`是否相等。仔细想想，这让我们之前的那种利用方式就变成了不可能。`chunk2`的`prevsize`是我们为了索引到`chunk0`而构造的，而`chunk0`和`chunk2`中间还存在准备被复用的`chunk`，所以这种构造方式，这个保护是绕不过去的。

`2.23`的链表构造，是系统帮助我们构造出来的。`2.29`也同样要想办法让系统帮我们构造出来。这里的手法的核心思想就是利用系统的残留数据。

首先根据上述的知识，要达成的目的就是伪造出一个`chunk0`，使其能够绕过这两个检测。

一个是向低地值合并的检测：

```c
if (__glibc_unlikely (chunksize(p) != prevsize))
    malloc_printerr ("corrupted size vs. prev_size while consolidating");
```

另一个就是`unlink`时候的检测

```c
if (__builtin_expect (FD->bk != P || BK->fd != P, 0))
    malloc_printerr (check_action, "corrupted double-linked list", P, AV);
```

接下来的操作全程关闭`aslr`，为了保证`100%`的成功率。

这里的思路是利用`largebin`残留的`fd_nextsize`和`bk_nextsize`两个指针，`smallbin`残留的`bk`指针，以及`fastbin`的`fd`指针。构造出一个如下的布局：

![](https://bbs.kanxue.com/upload/attach/202003/813771_NWAUHM53F7Q3RTD.png)

`2.29`下的`off-by-null`的题有`2019-TCTF-FINAL-babyheap`，不过这道题因为有`off-by-null`发生在`edit`功能中而不是`add`中，所以也很简单。

所以接下来，我会以`2019-BALSN-CTF-plaintext`为例，来详细介绍`off-by-null`的流程。

当然，因为原来的题目文件在我本地有点问题，所以我重新编译了一份文件，源代码和文件我会放到附件里。（我这个文件也有点问题，因为我是在`2.27`环境编译，在`2.29`环境运行的，不过关注点还是主要放在利用手法上嘛hh）

1. 首先做一些提前的布置。

```python
for i in range(7): # 0-6
    add(0x1000, "padding")

add(0x1000-0x410, "padding") # 7 

for i in range(7): # 8-14
    add(0x28, 't')

add(0xb20, "largebin") # 15
add(0x10, "padding") # 16

delete(15)
```

用到的最关键的堆块主要是`15`，`0-7`的目的都是为了填充堆空间，来调整所需堆块的偏移，这里不完全相同，根据情况自行调整。`8-14`正好`7`个`chunk`是为了对`tcache`进行操作。`16`是防止`15`和`topchunk`合并。这样布局之后，`delete 15` 会将其放入`unsortedbin`：

![](https://bbs.kanxue.com/upload/attach/202003/813771_M6UJEHKHWH5CREG.png)

可以看到此时`chunk15`堆地址的第二字节（因为是小端序）是`\x00`，这就是我们前面填充的原因，因为`off-by-null`不可避免的要在输入后补入`\x00`，堆地址如果是这个样子，那么我们覆盖第一个字节后，第二个字节被补`\x00`后就相当于不会产生变化。

而`chunk15`的`size`也要尽量的大，因为后续会对他各种切割，并且要保证切割后的大小移然超过`tcache`的限制，方便我们后续直接让他切割后的部分作为被`off-by-null`的目标而不用处理`tcache`。

2. 对`fake chunk`的`fd`和`bk`进行布局。

```python
add(0x1000, '\n') # chunk15 to largebin

add(0x28, p64(0) + p64(0x521) + p8(0x40)) # idx:17 get a chunk from largebin
```

当`malloc`的请求大于当前`unsortedbin`的`size`时，会触发`malloc`源码中的双循环机制，将`unsortedbin`里的`chunk`按`bk`顺序依次压入对应的`bin`中，然后再从`topchunk`切割出对应`size`的空间。

所以上一步的`chunk15`就被放进了`largebin`，同时此时因为它所在的`largebin`中只有它一个`chunk`，那么此时的`fd_nextsize`与`bk_nextsize`会指向其本身，关于`largebin`的规则这里就不展开说了，有些复杂和乱，如果有时间我会再详细介绍一下`largebin`规则的。

![](https://bbs.kanxue.com/upload/attach/202003/813771_DCRWKGCKUGS8PS2.png)

这里的`chunk17`的请求，因为`size`小于`largebin`里的`size`，所以会在`largebin`里切下来一小块，然后剩余部分放到`unsortedbin`里面。关于`chunk17`里所填入的值都是根据后续产生的指针不断调整出来的，这里暂时是无法观察出这几个值都是怎么来的。不过其作用是伪造出一个`fake chunk`：

![](https://bbs.kanxue.com/upload/attach/202003/813771_FP3FZUCAZMV7FKC.png)

3. 构造出`fake chunk`的`fd`，使`fd->bk = fake chunk`，满足条件

```python
add(0x28, 'a') # 18
add(0x28, 'b') # 19
add(0x28, 'c') # 20
add(0x28, 'd') # 21

# fill in tcache_entry[1](size: 0x30)
for i in range(7): # 8-14
    delete(8 + i)

delete(20)
delete(18)

# clear tcache_entry[1](size: 0x30)
for i in range(7): # 8-14
    add(0x28, '\n')

# fastbin to smallbin
add(0x400, '\n') #18

# get a chunk from smallbin , another smallbin chunk to tcache
# 20, change fake chunk's fd->bk to point to fake chunk
add(0x28,  p64(0) + p8(0x20))
```

前面的`18-21`还是从刚才进到`unsortedbin`的`chunk15`中切出来的，之后用到第`1`步准备的`7`个`tcache`填满对应的他`tcache_entry`链，之后`20`和`18`就会被放入`fastbin`中：

![](https://bbs.kanxue.com/upload/attach/202003/813771_XHWCSY2XHGSDUQY.png)

清空`tcache`防止后续被干扰，之后申请一个超过对应`fastbin`的请求，就会触发机制把当前`fastbin`里的压入到`smallbin`中，当然用不了`0x400`那么大，不过写的时候没注意，之后不想改了hh。

![](https://bbs.kanxue.com/upload/attach/202003/813771_DV9T4J748XF4W4Z.png)

注意`fastbin`取链是遍历`fd`，而`smallbin`或者其他任何双向链的`bin`取链都是遍历`bk`，当然`largebin`有所不同，所以此时申请`0x28`，就会从`smallbin`的`bk`取对应的`chunk`分配给我们，而这个`chunk`的`bk`如图所示，残留着堆指针可以被我们操作：

![](https://bbs.kanxue.com/upload/attach/202003/813771_YNM4J7MRZ43WM25.png)

`smallbin`申请之后，会有一个机制将对应`smallbin`链中余下的`chunk`放入对应的`tcache`里，图中可以看到。然后可以看到刚才取出来的`chunk`的`bk`被我们覆盖成了`fake chunk`的地址，这也就是第`2`步里为什么覆盖成`\x40`的原因。

4. 使`fake chunk`的`bk->fd = fake chunk`，满足条件。
