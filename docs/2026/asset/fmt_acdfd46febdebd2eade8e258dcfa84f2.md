
利用这个解密需要利用 double free、uaf等漏洞泄露 chunk 地址。

### 暴力破解

上面提到，如果链表头和`fd`指针的地址不在内存的同一页就可能会导致解密失败，需要用暴力破解的方式解密。

```python
def decrypt(enc):
    bits = [52, 40, 28, 16, 4]
    key = 0
    for i in range(5):
        p = ((enc ^ key) >> bits[i]) << bits[i]
        key = p >> 12
    return hex(p)

p = 0x000055555555a2a0
enc = p ^ (0x000055555555a110 >> 12)
print(hex(enc))
print(decrypt(enc))
#0x55500000f7fa
#0x55555555a2a0
```

设置不同页的`pos`的解密程序：

```python
def decrypt(enc):
    bits = [52, 40, 28, 16, 4]
    key = 0
    for i in range(5):
        p = (enc ^ key) >> bits[i] << bits[i]
        if (i == 3) or (i == 4):
            key = p >> 12
            key += 0x10
        else:
            key = p >> 12
    return p

ptr = 0x000055555555a2a0
pos = ptr+7*4096
print(hex(pos))
enc = ptr ^ (pos >> 12)
plain = decrypt(enc)
print(hex(plain))
```

在还原到第四组偏移（18bit）的时候，原`pos`该三位是x6x，`ptr`这三位是x5x，所以需要加`0x10`来还原。从这一组之后，每一组都要做相同操作，因为新的`key`会覆盖修改过的`key`。

所以暴力破解其他位数的时候，每一组偏移都要尝试加减 1-F的数值测试。

# how2heap深入浅出学习堆利用（三）

## poison\_null\_byte\_glibc2.34

### 原理

伪造一个`fake_chunk`，并修改物理相邻的下一个chunk的`prev_inuse`位为0，引起合并与`unlink`，最后再申请的时候就会从这一块合并的chunk中切割，overlap这一块chunk。

注意`unlink`的时候的检查就好，具体可以看上面unsafe unlink这篇文章。就是下面这两个东西：

1. chunk size是否等于 next chunk 的 `prev_size`

2. `FD->bk == P && BK->fd == P`。

   伪造 chunk 的时候要修改好这两个指针和 `prev_size`，`prev_inuse` 位。

有的文章说不能在有 tcache bin下运行，但其实这不是问题的主要因素。因为这只是一类技术的名称，把 `size` 扩大到 large bin size，就可以了。

### POC

how2heap源码：
