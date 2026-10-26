
```python
import lief
from keystone import Ks, KS_ARCH_ARM64, KS_MODE_LITTLE_ENDIAN
from capstone import Cs, CS_ARCH_ARM64, CS_MODE_ARM

# add section (as well as segment)
arm64elf = lief.parse("./build/hello_arm64")
new_sect = lief.ELF.Section(".new", lief.ELF.SECTION_TYPES.PROGBITS)
new_sect += lief.ELF.SECTION_FLAGS.WRITE  # .EXECINSTR
new_sect.alignment = 4
new_sect.content = list(bytes('xxhooked str hello world!\n', encoding='ansi'))
new_sect = arm64elf.add(new_sect)

#9E4 00 00 00 90 ADRP  X0, #aHelloWorld@PAGE ; "hello world"
#9E8 01 00 2D 91 ADD   X1, X0, #aHelloWorld@PAGEOFF ; "hello world"
code = b"\x00\x00\x00\x90\x01\x00\x2d\x91"
offset = 0x9e4
text_sect = arm64elf.get_section(".text")
addr = text_sect.search(code) + text_sect.virtual_address

# disassembly adrp "hello sdr"
cs = Cs(CS_ARCH_ARM64, CS_MODE_ARM)
cs.detail = True
for i in cs.disasm(code, addr):
    print(i.mnemonic, i.op_str)

# change addrss to new sect
ks = Ks(KS_ARCH_ARM64, KS_MODE_LITTLE_ENDIAN)
page = new_sect.virtual_address - (new_sect.virtual_address & 0xfff)  # address page
pageoff = (new_sect.virtual_address + 2) & 0xfff
asm_str = "adrp x0, #0x{:x};add x1, x0, #0x{:x}".format(page, pageoff)
print("rebuild_asm:", asm_str)
code_rebuild, _ = ks.asm(asm_str, addr)
print("rebuild_code:", "".join([f"{i:02x}" for i in code_rebuild]))
for i in cs.disasm(bytes(code_rebuild), addr):
    print(i.mnemonic, i.op_str)

# patch file
arm64elf.patch_address(addr, code_rebuild)
arm64elf.write(r"./build/hello_arm64_rebuild")
```

运行patch脚本后，qemu测试成功！

![arm64_hookstr](https://bbs.kanxue.com/upload/attach/202108/617776_8K3TQH3R5FDHCV8.jpg)

## 0x5 后记

这篇教程虽然难度不高，本来以为很快就能写完了，但是写了好久。很多东西叙述起来很麻烦，我尽量结合实例清晰和有条理性地讲述，不知不觉中写了好多东西。目前关于主机系列的汉化教程非常少，以往很多的教程也都随着论坛的关闭等无法访问了。同样，比起`x86`的汇编的修改，`arm64 elf`之类的分析也不多。我系统地做《galgame汉化中的逆向》系列是想总结一下汉化方法，结合我自身对一些问题的思考和理解，以使汉化逆向这门技术今后能更好地传承下去。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
