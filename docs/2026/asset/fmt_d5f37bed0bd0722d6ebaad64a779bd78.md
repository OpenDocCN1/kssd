

# [原创]使用 LIEF 打造类似王者 X 荣耀的静态代码注入

使用 LIEF 工具写一个类似于腾讯 SO 保护的东西。本篇记录 LIEF 工具的使用和 TX 加固的原理，不会对 ELF 文件格式进行记录。

## 一、工具介绍

首先，LIEF 是一个文件解析工具，可以解析 ELF、PE、DEX 等，用途比较广泛。

GitHub 地址：<mark class="encrypted">ef0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3g2X3i4K6u0V1M7s2u0G2K9X3g2U0N6q4)9J5c8V1I4u0c8f1j5`.</mark>

我使用的是 Python 版，在使用过程中经常出现毛病，所以并不建议使用 Python 版。最好是不怕麻烦自己写一个，就可以避免很多不稳定导致的 fix 代码。

## 二、TX SO 加密介绍

首先，王者荣耀采用的是 `il2cpp`，比较关键的 SO 有 3 个：`libil2cpp.so`、`libGameCore.so`、`libtprt.so`。其中 `libtprt.so` 为保护 SO。

王者荣耀加载被保护 SO（`il2cpp` 或 `GameCore`）时，会优先加载链接库。所以，整个流程在 Unity 启动后，会优先加载 `libtprt.so`。

![图 1](upload/attach/202103/833877\_MPHRXJ9M3MTHUWJ.png)

`libtprt.so` 对自身的 `tptext` 节进行了加密，在加载过程中执行 `init_array` 进行解密。跟踪 `mprotect`，可以看到解密代码：

![](upload/attach/202103/833877\_8YQHZSZFREFEP4R.png)

![](upload/attach/202103/833877\_4HBW6DVY2GPB65X.png)

使用 LIEF 快速解密：

```python
tprt = lief.parse(name)
tptext_section = tprt.get_section(".tptext").content
print(len(tptext_section))
offset = tprt.get_section(".tptext").offset
out = b""
for i in range(len(tptext_section)):
    out += (tptext_section[i] ^ 0xb8).to_bytes(1, byteorder="little")
print(len(out))
tpp = tprt_bin[:offset] + out + tprt_bin[offset + len(tptext_section):]
with open("tpp.so", "wb") as fp:
    fp.write(tpp)
```

![](upload/attach/202103/833877\_NMTGQ79VGMJST3J.png)

`tptext` 节里的函数主要用于初始化各类保护，会被 `jni_onload` 调用。现在 `tprt` 已经被成功加载，接下来就需要解密 `libil2cpp.so`。

由于 `libGameCore.so` 小一点，分析起来方便，所以后面就不分析 `il2cpp` 了。先观察 ELF 结构，可以发现它的 LOAD 段格外多。

个人分析它比原生多了 3 个 LOAD 段。新增的第一个 LOAD 段（04）里面是 `.dynamic` 和 `.init_array`。很好理解，因为强行添加了一个依赖库和 `init_array`，所以需要生成新的 `.dynamic`。至于 `init_array`，个人猜测是添加字符串混淆之类的东西。

![](upload/attach/202103/833877\_AF77K2YW2GQDVP8.png)

新增的第二个 LOAD 段（05）是新的 `.rel.plt`。用 IDA 分析，可以明显看出有增加项，且增加项在新增的第三个 LOAD 段（06）：

![](upload/attach/202103/833877\_PSPXGF4J46FMPBX.png)

跳转到 `0x228cfcc`，可以看到这是一个 GOT 表，并且它的 PLT 表在新增的第二个 LOAD 段（05）。

![](upload/attach/202103/833877\_5EYTCBPQVUBK8KZ.png)

![](upload/attach/202103/833877\_PEWXYBNQ6YR4XCH.png)

![](upload/attach/202103/833877\_NUVYT86NZFUQD3U.png)

对添加的 LOAD 段有一定了解了，再看一下修改后的 `.dynamic` 有什么变化：

![](upload/attach/202103/833877\_RN842J8P4MWWF9X.png)

需要关注的，我已经添加了红色方框。大部分只是修改偏移，指向新的节（`JMPREL`、`INIT_ARRAY`），值得注意的是 `DT_INIT`，这个是 init 段，比 `init_array` 更早执行，更适合用来解密。

![](upload/attach/202103/833877\_94R6T8SYWS7EVTE.png)

通过取 `off_1cc` 的地址减去 `off_1cc` 的值（地址为 `base + 0x1cc`，值是自己设置的，为 `1cc`），所以算出来是当前 SO 的基址，然后调用 `sub_2289b58` 并且传入基址。

`sub_2289b58` 就是解密函数，网上对它的分析很多了，总的就是调用 `g_tprt_pfn_array`（`".text"`、`base`、`3`）对当前的 `text` 段进行解密。（`".text"` 这个字符串在新增的第三个 LOAD 段（06）中，意味着是写死的。）

![](upload/attach/202103/833877\_3R4CW5HRV9KJ8A9.png)

所以总结一下：

- MTP 对 SO 新增了 3 个 LOAD 段。第一个 LOAD 新增 `.dynamic`、`.init_array`；第二个 LOAD 段是映射新的 `.rel.plt`、`.plt`、`.text` 节；第三个 LOAD 段用来映射 `.got`、`.data` 节。
- MTP 在 program header 之后添加了一段汇编，在 init 时执行，获取当前 SO 的基址，传入解密函数，并调用 `libtprt.so` 的导出函数 `g_tprt_pfn_array` 进行 text 节解密。

## 三、代码实现

由于一次写完大概率会很乱，思路不够清晰，所以分几步写，方便理解，也好记录。

### 步骤一：添加 INIT 段，并调用任意函数

首先，生成一个 SO，很简单，有一个 `PrintLog` 函数没有调用，所以 LOG 只有一条：

![](upload/attach/202103/833877\_FHSPFTEMCYX5WUR.png)

![](upload/attach/202103/833877\_SXHRYK86K7P78FP.png)

执行以下代码：

```python
"""
添加 init 段，并且调用 ADD_FUNC_NAME 并传入一个参数（当前 SO 的基址）
"""
INIT_PROC_SIZE = 0x2c
INIT_PROC_CONTENT = [
    0xC0, 0x46, 0xFF, 0xB5, 0x83, 0xB0, 0x6D, 0x46,
    0x00, 0xA3, 0x14, 0x3B, 0x19, 0x1C, 0x1F, 0x68,
    0xC9, 0x1B, 0x29, 0x60, 0x5E, 0x68, 0x03, 0xD0,
    0x76, 0x18, 0x6E, 0x60, 0x28, 0x68, 0xB0, 0x47,
    0x03, 0xB0, 0xFF, 0xBD
]  # 腾讯代码里扣出来的嘿嘿

def add_init_proc():
    binary = lief.parse(TARGET_BIN)

    # 检测 .dynamic 节的空位是否足够，如果小于 3 个就要扩展 dynamic 节内容
    free_dynamic_entry = 0
    for entry in binary.dynamic_entries:
        if entry.tag == lief.ELF.DYNAMIC_TAGS.NULL:
            free_dynamic_entry += 1

    print("free dynamic entry num:", free_dynamic_entry)
    assert free_dynamic_entry > 3

    # 获取位于 segment header 后的 offset，用于添加 init_proc
    init_proc_offset = (
        binary.header.program_header_offset
        + binary.header.program_header_size
        * (binary.header.numberof_segments + 2)
    )
    print("init_proc offset:", hex(init_proc_offset))

    # 添加 init 入口
    if not binary.has(ELF.DYNAMIC_TAGS.INIT):
        # 先用 0 占位，直接写入偏移，LIEF 工具会有点问题
        # 如果不出问题 binary.add(ELF.DynamicEntry(ELF.DYNAMIC_TAGS.INIT, init_proc_offset + 8))
        binary.add(ELF.DynamicEntry(ELF.DYNAMIC_TAGS.INIT, 0))
    else:
        init_entry = binary.get(ELF.DYNAMIC_TAGS.INIT)
        print("[x] binary has init_proc:", init_entry)
        exit(1)

    binary.write("libnative-lib.so")

    # 手动修复 DynamicEntry 中的 value
    outbin = lief.parse("libnative-lib.so")
    out_dynamic = outbin.get_section(".dynamic")
    ADD_FUNC = outbin.get_symbol(ADD_FUNC_NAME).value

    num = 0
    for entry in outbin.dynamic_entries:
        if entry.tag == ELF.DYNAMIC_TAGS.INIT:
            break
        num += 1

    init_entry_offset = out_dynamic.offset + (num * out_dynamic.entry_size)
    print(hex(init_entry_offset))
    patch_file(
        "libnative-lib.so",
        init_entry_offset + 4,
        struct.pack("<I", init_proc_offset + 8 + 1)
    )

    # 前四位为 init_proc 的偏移，后四位为要调用的函数地址
    print("init_proc_offset:", hex(init_proc_offset))
    print("ADD_FUNC:", hex(ADD_FUNC))

    INIT_PROC_CONTENTS = (
        list(struct.pack("<I", init_proc_offset))
        + list(struct.pack("<I", ADD_FUNC))
        + INIT_PROC_CONTENT
    )
    patch_file("libnative-lib.so", init_proc_offset, INIT_PROC_CONTENTS)
```

![](upload/attach/202103/833877\_SKQ8DJ222SEACXA.png)

![](upload/attach/202103/833877\_236673NWQVB4CX3.png)

**可以看到函数 `PrintLog` 优先于 JNI 函数执行，并且传入的参数为基址。**

### 步骤二：向 SO 中添加可执行代码

[关于重定位](https://bbs.pediy.com/thread-221821.htm)

需要记住的几个总结：

- `rel.plt` -> `got` -> `plt` -> extern 表（导入函数）
- `rel.plt` -> `got` -> `text`（导出）

```cpp
// ELF.h 查看 rel.plt 的格式，r_offset 指向 got 表的地址，
// r_info 高 8 位为类型，后 24 位为
// Relocation entry, without explicit addend.
struct Elf32_Rel {
    Elf32_Addr r_offset; // Location (file byte offset, or program virtual addr)
    Elf32_Word r_info;   // Symbol table index and type of relocation to apply

    // These accessors and mutators correspond to the ELF32_R_SYM、ELF32_R_TYPE、
    // and ELF32_R_INFO macros defined in the ELF specification:
    Elf32_Word getSymbol() const { return (r_info >> 8); }
    unsigned char getType() const { return (unsigned char)(r_info & 0x0ff); }
    void setSymbol(Elf32_Word s) { setSymbolAndType(s, getType()); }
    void setType(unsigned char t) { setSymbolAndType(getSymbol(), t); }
    void setSymbolAndType(Elf32_Word s, unsigned char t) {
        r_info = (s << 8) + t;
    }
};

// Symbol table entries for ELF32.
struct Elf32_Sym {
    Elf32_Word st_name;  // Symbol name (index into string table)
    Elf32_Addr st_value; // Value or address associated with the symbol
    Elf32_Word st_size;  // Size of the symbol
    unsigned char st_info; // Symbol's type and binding attributes
    unsigned char st_other; // Must be zero; reserved
    Elf32_Half st_shndx; // Which section (header table index) it's defined in

    // These accessors and mutators correspond to the ELF32_ST_BIND、
    // ELF32_ST_TYPE、ELF32_ST_INFO macros defined in the ELF specification:
    unsigned char getBinding() const { return st_info >> 4; }
    unsigned char getType() const { return st_info & 0x0f; }
    void setBinding(unsigned char b) { setBindingAndType(b, getType()); }
    void setType(unsigned char t) { setBindingAndType(getBinding(), t); }
    void setBindingAndType(unsigned char b, unsigned char t) {
        st_info = (b << 4) + (t & 0x0f);
    }
};
```

编译一个 SO，里面的示范代码：

![](upload/attach/202103/833877\_TCMWHA54C5UN764.png)

十分简单，就是一个打印 LOG，将编译好的 SO 拖入 IDA，可以看到，虽然我只写了一个函数，实际上 SO 运行时还需要许多其他的函数。

![](upload/attach/202103/833877\_BRA8NMDWUGEPKKF.png)

执行以下代码：

