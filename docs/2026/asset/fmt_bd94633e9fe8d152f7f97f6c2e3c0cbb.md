
```python
# _*_ coding: utf-8 _*_
import pefile
import struct
from capstone.x86 import *
from capstone import *
from unicorn import *
from unicorn.x86_const import *
from binascii import *

def u32(data):
    return struct.unpack("I", data)[0]

def p32(num):
    return struct.pack("I", num)

def patch(image, image_base, address, patch_data):
    '''
    :param image: memory_mapped_image 从入口点开始处的数据
    :param image_base: 基址
    :param address: imagebase+rva VA
    :param patch_data:
    :return:
    '''
    i = 0
    for b in patch_data:
        image[address - image_base + i] = b
        i += 1

# 获取重定位表的序号 pefile.py 146行
def get_reloc_data_rva(pefile_struct):
    '''
    :param pefile_struct:
    :return: 返回所有重定位数据的RVA列表
    '''
    reloc_table_num = pefile.DIRECTORY_ENTRY['IMAGE_DIRECTORY_ENTRY_BASERELOC']
    # 获取重定位表的VirtualAddress和Size
    reloc_table = pe.OPTIONAL_HEADER.DATA_DIRECTORY[reloc_table_num]
    reloc_table_rva = reloc_table.VirtualAddress
    reloc_table_size = reloc_table.Size
    print(f'重定位表RAV：{reloc_table_rva:#x},重定位表大小：{reloc_table_size:#x}')
    # reloc_table由数个IMAGE_BASE_RELOCATION结构组成，每个结构由VirutalAddress(DWORD)、SizeOfBlock(DWORD)和TypeOffset(SizeOfBlock-8)组成
    # parse_relocations_directory返回BaseRelocationData对象列表
    relocations = pe.parse_relocations_directory(reloc_table_rva, reloc_table_size)
    # 获取所有的重定位数据RVA
    reloc_data_rva = []
    for i in relocations:
        # BaseRelocationData有两个属性，struct和entries。
        # struct是IMAGE_BASE_RELOCATION结构的VA和Size。
        # entries:    list of relocation data (RelocationData instances)
        # RelocationData: type和RVA
        # print(i.struct)
        for j in i.entries:
            reloc_data_rva.append(j.rva)
    return reloc_data_rva

def get_intruction_start_rva(memory_data, reloc_data_rva, ImageBase):
    '''
    :param memory_data:  映射到内存中的文件数据
    :param reloc_data_rva: 重定位数据的rva
    :param ImageBase: ImageBase
    :return: 指令的rva
    '''
    branch = ["JZ","JP","JO","JS","JG","JB","JA","JL","JE","JNZ","JNP","JNO","JNS","JLE","JNB","JBE","JGE","JNE","JAE"]
    b = 3
    for i in range(3):
        code = memory_data[reloc_data_rva - b: reloc_data_rva - b + 40]
        if b == 3 and code[0] != 0x66:
            b = b - 1
            continue
        b = b - 1
        try:
            ins = md.disasm(code, ImageBase + reloc_data_rva - b - 1)
            ins_1 = next(ins)
            ins_2 = next(ins)
            ins.close()
        except StopIteration:
            continue
        if (ins_1.mnemonic == 'cmp' or ins_1.mnemonic == 'test') and ins_2.mnemonic.upper() in branch and len(ins_1.operands) == 2 and ins_1.operands[0].type == X86_OP_MEM and ins_1.operands[1].type == X86_OP_IMM:
            return ins_1.address - 0x10000000
    return 0

filename = 'bf3e495f43a6b333b10ae69667304cfd2c87e9100de9d31365671c7b6b93132e'
pe = pefile.PE(filename, fast_load=True)

memory_mapped_image = bytearray(pe.get_memory_mapped_image())
ImageBase = pe.OPTIONAL_HEADER.ImageBase

print('[+] Map PE')
BASE = 0x10000000
STACK_ADDR = 0x400000
STACK_SIZE = 1024 * 1024

mu = Uc(UC_ARCH_X86, UC_MODE_32)
mu.mem_map(BASE, 1024 * 1024)
mu.mem_map(STACK_ADDR, STACK_SIZE)

r_esp = STACK_ADDR + STACK_SIZE // 2
mu.reg_write(UC_X86_REG_ESP, STACK_ADDR + STACK_SIZE // 2)

# 将文件映射到内存中
mu.mem_write(0x10000000,pe.get_memory_mapped_image())
md = Cs(CS_ARCH_X86, CS_MODE_32)
md.detail = True

instruction_3 = []
def hook_code(mu, address, size, userdata):
    print(f'>>> Tracing instruction at {address:#x}, instruction size = {size:#x}')
    r_esp = mu.reg_read(UC_X86_REG_ESP)
    count = u32(mu.mem_read(r_esp + 4, 4))
    print(f'count is {count}')

    if count == 2:
        instruction_3.append(address)
        mu.emu_stop()
    try:
        exit()
    except BaseException as e:
        print(e)
    count = count + 1
    mu.mem_write(r_esp + 4, p32(count))

mu.hook_add(UC_HOOK_CODE, hook_code)

def simulate_execute(ins_addr_rva):
    mu.mem_write(r_esp + 4, p32(0))
    mu.emu_start(ins_addr_rva + ImageBase, 0x100066E6)

reloc_data_rva = get_reloc_data_rva(pe)
ins_addr_rva_all = []
count = 0
for rva in reloc_data_rva:
    ins_addr_rva = get_intruction_start_rva(memory_mapped_image, rva, ImageBase)
    if ins_addr_rva != 0:
        ins_addr_rva_all.append(ins_addr_rva)
        simulate_execute(ins_addr_rva)
# 获取按顺序执行时第3条指令地址
code = memory_mapped_image[ins_addr_rva:ins_addr_rva + 40]
ins = md.disasm(code, ImageBase + ins_addr_rva)

ins_1 = next(ins)
ins_2 = next(ins)

try:
    ins_3 = next(ins)
    ins_3_address = ins_3.address
except:
    ins_3_address = 0
ins.close()

if instruction_3[count] == ins_3_address:
    size = ins_1.size + ins_2.size
    assembly = b'\x90' * size
    patch(memory_mapped_image, ImageBase, ImageBase + ins_addr_rva, assembly)
else:
    size = instruction_3[count] - ins_1.address
    assembly = b'\x90' * size
    patch(memory_mapped_image, ImageBase, ImageBase + ins_addr_rva, assembly)
count = count + 1

for section in pe.sections:
    print(f'{section.Name}, VirtualAddress: {section.VirtualAddress:#x}, '
          f'Size: {section.SizeOfRawData:#x}, 文件偏移: {section.PointerToRawData:#x}')
    pe.set_bytes_at_rva(section.VirtualAddress,
        bytes(memory_mapped_image[section.VirtualAddress:section.VirtualAddress + section.SizeOfRawData]))

print('[+] Save to file ' + '1.bin')
pe.write('1.bin')
```

## 4. 参考链接

[1]. 使用和海莲花相似混淆手法的攻击样本分析

c6fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9g2)9J5k6i4q4A6j5h3&6^5K9h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6a6j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3X3c8K6K9h3#2A6L8r3q4J5i4K6u0V1N6r3!0Q4x3X3c8a6j5$3g2S2L8V1I4G2N6s2g2K6i4K6u0r3

[2]. Type2_Deobfuscate.py

a00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9k6i4k6S2L8Y4k6F1i4K6u0r3b7g2m8f1x3K6u0Q4y4h3k6p5k6h3!0T1k6Y4g2K6j5$3q4@1k6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6f1P5i4m8W2x3W2)9J5c8W2y4U0M7X3W2H3N6q4)9J5c8W2c8&6M7r3f1J5i4K6g2X3c8r3g2G2j5X3k6#2M7$3y4S2N6r3g2Q4x3X3g2H3P5b7`.

[3]. pefile.py

5caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2M7X3!0U0j5i4u0J5k6i4u0S2i4K6u0r3M7r3g2X3K9h3I4W2i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4m8W2k6X3W2D9k6g2)9J5k6i4m8&6

[4]. 菜鸟读capstone与keystone源码入门

https://bbs.pediy.com/thread-258473-1.htm

[5]. Unicorn引擎教程

https://bbs.pediy.com/thread-224330.htm#msg_header_h3_7
