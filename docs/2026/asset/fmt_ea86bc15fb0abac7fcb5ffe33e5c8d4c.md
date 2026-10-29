
# [原创]看我攻破所谓的“安全系统”

## 前言

在前段时间，因为闲着无聊，我注意到看雪上有个“成功登陆文件系统，并取到文件”的挑战，奖金高达50块钱。如下所示：

![图片描述](upload/attach/202107/651413\_VUFAZPVE4HFV2PT.png)

一共3个系列，难度逐渐提高，题目的套路是给一个Linux虚拟机文件，你不知道root密码，让你获取到root下的文件。下面开始介绍我使用的分析方法。

![图片描述](upload/attach/202107/651413\_ZY73NFRV9SWGM5Z.png)

## 题目的考点

因为是虚拟机文件，我最开始直接在另一台Kali挂载了题目中的VMDK虚拟磁盘文件，可以读取硬盘中的内容：

该硬盘分区和传统的Linux差不多，它一共有两个分区，启动分区和存放根文件系统的分区，启动分区是没加密的，而存放着root文件夹的另一个分区被LUKS保护着。

启动分区存放着GURB，Linux内核，initramfs。

![图片描述](upload/attach/202107/651413\_GHGC7TJ36FEN3ED.png)

LUKS保护分区：

![图片描述](upload/attach/202107/651413\_RXHZNU2ABKCBPWF.png)

很明显的是，LUKS解密的逻辑肯定是放在启动分区的initramfs中。不幸的是，该文件系统被加密了：

![图片描述](upload/attach/202107/651413\_K6VF26JSS4E2739.png)

但是Linux kernel没被加密：

![图片描述](upload/attach/202107/651413\_SP3JKCC2V4M9EF8.png)

## 解密文件系统

根据本人的分析经验，加解密文件系统的算法一般是放在Linux内核中，使用IDA直接分析`populate_rootfs`。

![图片描述](upload/attach/202107/651413\_KAZPRF3DV7YF9EZ.png)

好了直接调用这个`SerpentDecrypt()`函数直接解密该文件系统，当然可以使用QEMU直接调试打断点然后DUMP出明文数据也可以，但是搭建环境需要时间，最快的方法直接使用现成的函数，`unicorn`模拟执行即可：

```python
from unicorn import *
from capstone import *
from unicorn.x86_const import *
from elftools.elf.elffile import ELFFile
from elftools.elf.segments import Segment
import ctypes
import binascii
import hexdump
import struct

filepath = 'kernel.elf'
load_base = 0xFFFFFFFF81000000
stack_base = 0
stack_size = 0x20000
var_base = stack_base + stack_size
var_size = 0x2000000
stop_stub_addr = var_base + var_size
stop_stub_size = 0x10000

emu = Uc(UC_ARCH_X86, UC_MODE_64)

def disasm(bytecode, addr):
    md = Cs(CS_ARCH_X86, UC_MODE_64)
    for asm in md.disasm(bytecode, addr):
        return '%s\t%s' % (asm.mnemonic, asm.op_str)

def align(addr, size, growl):
    UC_MEM_ALIGN = 0x1000
    to = ctypes.c_uint64(UC_MEM_ALIGN).value
    mask = ctypes.c_uint64(0xFFFFFFFFFFFFFFFF).value ^ ctypes.c_uint64(to - 1).value
    right = addr + size
    right = (right + to - 1) & mask
    addr &= mask
    size = right - addr
    if growl:
        size = (size + to - 1) & mask
    return addr, size

def hook_code(uc, address, size, user_data):
    # bytecode = emu.mem_read(address, size)
    # print(" 0x%x :%s" % (address, disasm(bytecode, address)))
    if address == stop_stub_addr:
        emu.emu_stop()

# init var
def init_emu():
    with open(filepath, 'rb') as elffile:
        elf = ELFFile(elffile)
        load_segments_addr = [0xFFFFFFFF81000000, 0xFFFFFFFF815EE000]
        load_segments = [
            x for x in elf.iter_segments()
            if x.header.p_type == 'PT_LOAD'
        ]

        for segment in load_segments:
            prot = UC_PROT_ALL
            if segment.header.p_vaddr not in load_segments_addr:
                continue

            print(
                'mem_map: addr=0x%x size=0x%x'
                % (segment.header.p_vaddr, segment.header.p_memsz)
            )
            addr, size = align(
                load_base + segment.header.p_vaddr,
                segment.header.p_memsz,
                True
            )
            addr = segment.header.p_vaddr
            print('fix_mem_map: addr=0x%x size=0x%x' % (addr, size))
            emu.mem_map(addr, size, prot)
            emu.mem_write(addr, segment.data())

    emu.mem_map(stack_base, stack_size, UC_PROT_ALL)
    emu.mem_map(var_base, var_size, UC_PROT_ALL)
    emu.mem_map(stop_stub_addr, stop_stub_size, UC_PROT_ALL)

def SerpentDecrypt(data):
    emu.mem_write(
        stack_base + stack_size - 8,
        struct.pack('Q', stop_stub_addr)
    )
    emu.mem_write(var_base, data)

    SerpentDecrypt_addr = 0xFFFFFFFF81004660

    # SerpentDecrypt
    code = emu.mem_read(SerpentDecrypt_addr, 8)
    hexdump.hexdump(code)

    emu.reg_write(UC_X86_REG_RSP, stack_base + stack_size - 8)
    emu.reg_write(UC_X86_REG_RDI, var_base)
    emu.reg_write(UC_X86_REG_RSI, len(data))

    # emu.reg_write(UC_MIPS_REG_RA, stop_stub_addr)
    # emu.reg_write(UC_MIPS_REG_SP, stack_base + stack_size)

    emu.hook_add(
        UC_HOOK_CODE,
        hook_code,
        begin=stop_stub_addr,
        end=stop_stub_addr + stop_stub_size
    )
    emu.emu_start(
        SerpentDecrypt_addr,
        SerpentDecrypt_addr + 0x1000
    )
    return emu.mem_read(var_base, len(data))

if __name__ == "__main__":
    with open('initramfs.img', 'rb') as fd:
        data = fd.read()

    print('%x' % len(data))
    init_emu()
    dec_data = SerpentDecrypt(data)

    # dec_data = SerpentDecrypt(bytes(dec_data))[0:32]
    # hexdump.hexdump(dec_data)

    with open('initramfs.gzip', 'wb+') as fd:
        fd.write(dec_data)
```

解开文件系统后，可以看到LUKS加密密钥生成的逻辑（再后面的题目中还魔改了LUKS加密引擎）。到这里可以说是基本获取到权限，我们可以修改文件系统，插入代码，反弹shell回来之类的方法获取到权限。

![图片描述](upload/attach/202107/651413\_M3RFYE8B9Z9KRGH.png)

![图片描述](upload/attach/202107/651413\_Y5MRC2JPPUKQH3S.png)

## 更快的方法

有没有更快的方法？如果我能直接读写虚拟机内存，将判断登陆的二进制代码直接强制让它进入登陆成功的流程，那我不是可以无视加密直接登陆进系统？是的，虚拟机允许修改guest机的内存，无论是跨进程读写vmx进程或者修改vmem文件的方法都能读写虚拟机实例的内存。

先调小虚拟机内存，方便定位：

![图片描述](upload/attach/202107/651413\_U9VUSTMSFNVMDRA.png)

随便找个x64的login程序，找到特征码：

![图片描述](upload/attach/202107/651413\_4PNC325XA8SGXSH.png)

挂起虚拟机，虚拟机目录下生成对应的内存文件：

![图片描述](upload/attach/202107/651413\_BPDYH2SUXA9QFX6.png)

通过特征码定位到判断点：

![图片描述](upload/attach/202107/651413\_UMZTRBJZ5VRJFFZ.png)

![图片描述](upload/attach/202107/651413\_9MNBHS5EHHCCMY7.png)

恢复快照，输入错误的密码，也可直接进入系统：

![图片描述](upload/attach/202107/651413\_F6QRVRBZUR8ZQMV.png)

成功获取到文件：

![图片描述](upload/attach/202107/651413\_TJC5WHRDYZYAC6X.png)

一共三个系列，方法都大同小异：

![图片描述](upload/attach/202107/651413\_X634K7WXMENSX93.png)

## 总结

类似的挑战还是颇具现实意义，例如一些设备，就是使用这类加密，但是设备不容易修改内存，通过纯逆向的方法也可以进行分析。下面简单总结下分析方法：

1. 纯逆向工程，分析Linux内核。内核加密就分析GRUB。解开文件系统，重新打包获取权限。
2. 直接修改内核或者GRUB文件，插入shellcode，获取权限。
3. 直接修改内存，插入shellcode，或者修改跳转，获取权限。
4. 进入单用户模式，但是有可能开发者会防止你进入单用户模式移除一些代码。

## 前言

在前段时间，因为闲着无聊，我注意到看雪上有个“成功登陆文件系统，并取到文件”的挑战，奖金高达50块钱。如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_VUFAZPVE4HFV2PT.png)

一共3个系列，难度逐渐提高，题目的套路是给一个Linux虚拟机文件，你不知道root密码，让你获取到root下的文件。下面开始介绍我使用的分析方法。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_ZY73NFRV9SWGM5Z.png)

## 题目的考点

因为是虚拟机文件，我最开始直接在另一台Kali挂载了题目中的VMDK虚拟磁盘文件，可以读取硬盘中的内容：

该硬盘分区和传统的Linux差不多，它一共有两个分区，启动分区和存放根文件系统的分区，启动分区是没加密的，而存放着root文件夹的另一个分区被LUKS保护着。

启动分区存放着GURB，Linux内核，initramfs。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_GHGC7TJ36FEN3ED.png)

LUKS保护分区：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_RXHZNU2ABKCBPWF.png)

很明显的是，LUKS解密的逻辑肯定是放在启动分区的initramfs中。不幸的是，该文件系统被加密了：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_K6VF26JSS4E2739.png)

但是Linux kernel没被加密：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_SP3JKCC2V4M9EF8.png)

## 解密文件系统

根据本人的分析经验，加解密文件系统的算法一般是放在Linux内核中，使用IDA直接分析`populate_rootfs`。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/651413_KAZPRF3DV7YF9EZ.png)

好了直接调用这个`SerpentDecrypt()`函数直接解密该文件系统，当然可以使用QEMU直接调试打断点然后DUMP出明文数据也可以，但是搭建环境需要时间，最快的方法直接使用现成的函数，`unicorn`模拟执行即可：

|  |  |
| --- | --- |
