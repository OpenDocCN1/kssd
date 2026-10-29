
`1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121  122  123  124  125  126  127  128  129  130  131  132  133  134  135  136  137  138  139  140  141  142  143  144  145  146  147  148  149  150  151  152  153  154  155  156  157  158  159  160`

```python
import lief
from qiling.os.const import POINTER, UINT, STRING
from qiling import Qiling
from qiling.const import QL_VERBOSE
from qiling.const import QL_INTERCEPT
from qiling import os
from qiling.os.mapper import QlFsMappedObject
import struct
from pwn import elf, pack, unpack, u64
soData = lief.parse("libmanxi.so")
datasec = None
rodatasec = None
for sec in soData.sections:
    if sec.name == ".data":
        datasec = sec
    elif sec.name == ".rodata":
        rodatasec = sec
data_str = ""
elf_patch = elf.ELF("D:/new/frida-agent-example-main/agent/node_modules/libmanxi_copy.so")
def write_hook(ql: Qiling, access: int, address: int, size: int, value: int):
    Addr = address - base_addr
    if (Addr >= datasec.virtual_address and Addr <= datasec.virtual_address + datasec.size) or (Addr >= rodatasec.virtual_address and Addr <= rodatasec.virtual_address + rodatasec.size):
        # if value >= 32 and value <= 128:
        print(f"Write Address: {hex(address-base_addr)} Value:{hex(value)} Size:{hex(size)}")
        if size == 1:
            data = ql.pack8(value)
        elif size == 2:
            data = ql.pack16(value)
        elif size == 4:
            data = ql.pack32(value)
        elif size == 8:
            data = ql.pack64(value)
        elf_patch.write(Addr, data)
# Read Init_array
funcArray = [
    [22616, 23116],
    [28832, 29460],
    [39664, 40264],
    [42856, 43124],
    [43932, 44020],
    [46660, 46664],
    [55196, 62940],
    [63672, 63756],
    [63756, 63840],
    [63840, 63924],
    [73424, 73428],
    [73516, 73600],
    [87628, 88064],
    [122504, 122588],
    [123876, 124208],
    [124208, 124292],
    [125200, 125288],
    [127652, 128176],
    [131420, 131828],
    [135164, 135876],
    [153412, 154708],
    [164120, 167816],
    [175060, 175332],
    [183120, 183124],
    [183124, 183128],
    [186856, 186940],
    [193592, 193732],
    [215616, 216696],
    [219564, 219648],
    [232264, 232352],
    [232920, 232924],
    [233080, 233352],
    [244248, 253964],
    [255484, 256504],
    [258116, 258424],
    [259408, 262080],
    [262080, 262432],
    [264236, 264696],
    [268872, 269224],
    [269688, 270484],
    [274292, 276648],
    [283864, 284264],
    [286580, 287280],
    [288384, 288656],
    [291236, 291596],
    [292296, 293676],
    [294012, 295272],
    [295272, 295520],
    [295976, 296248],
    [299480, 299788],
    [300236, 307192],
    [307500, 307504],
    [311600, 312672],
    [313304, 313308],
    [316048, 316052],
    [324380, 325624],
    [378800, 380220],
    [382004, 382008],
    [389160, 389520],
    [390088, 390092],
    [392360, 392444],
    [407960, 407964],
    [410060, 410064],
    [414476, 415160],
    [448668, 457116],
    [464888, 464972],
    [469092, 469444],
    [471132, 471388],
    [472156, 472160],
    [472180, 472264],
    [472512, 472596],
    [472596, 472680],
    [472680, 472684],
    [472684, 472772],
    [485160, 489068],
    [491672, 493608],
    [493608, 493692],
    [506756, 506988],
    [520552, 521200],
    [523132, 523532],
    [528344, 528576],
    [533168, 539904],
    [548616, 550592],
    [568596, 569860],
    [573168, 575808],
    [579536, 579792],
    [589616, 589620],
    [610480, 610484],
    [626444, 626916],
    [636500, 636504],
    [641936, 643184],
    [644460, 644464],
    [644816, 644820],
    [42852, 42856],
]
# # Init Binary File
ql = Qiling(["D:/new/frida-agent-example-main/agent/node_modules/libmanxi.so"], r"D:/new/rootfs/arm64_android", verbose=QL_VERBOSE.DISASM)
base_addr = ql.mem.get_lib_base("libmanxi.so")
str = f"Base Address: {hex(base_addr)}"
ql.hook_mem_write(write_hook)
for func in funcArray:
    ql.run(func[0] + base_addr, func[1] + base_addr - 4)
elf_patch.save("aaa.so")
```

解密之后就可以看到很多敏感字符串，但实际上该程序没有检测Root、Fart，只在程序启动时对Frida检测，也就是说可通过Attach来进行附加

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_YJ5DZY8EAV3UT48.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_HV4XPAXGSTJEQBH.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_NEY2DCMYHVMBHYT.webp)

#### Frida分析

首先需要绕过启动时的Frida检测，奇怪的是我Hook `dlopen`和`android_dlopen_ext`都无法获取到`libmanxi.so`的加载，但是通过`Process.enumerateModules()`可以得到so，这也导致了后续难以定位到检测的位置和绕过。于是使用魔改Frida [094K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6L8r3q4J5L8$3c8Q4x3V1k6r3L8r3!0J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4x3`.](https://bbs.kanxue.com/elink@d5eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6k6L8r3q4J5L8$3c8Q4x3V1k6r3L8r3!0J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4x3`.) (可绕过检测)

简单Hook了一下程序中的SVC指令，通过`Radare2`的`/adj svc`可导出所有SVC指令的地址

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_8WYBYPJRHQ6CFV6.webp)

其中`addr`是导出的结果，这里只针对`openat`进行过滤

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `function hook_svc() {` `console.log("=====Hook SVC=====")` `var base_addr = Module.findBaseAddress("libmanxi.so");` `addr.forEach(function (svc) {` `var offset = 0x` + svc.offset.toString(16) `` `Interceptor.attach(base_addr.add(offset), {` `onEnter: function (args) {` `if (this.context.x8 == 0x38) {` `console.log("Openat:"` + ptr(this.context.x1).readCString())` `}` `},` `onLeave: function (retval) {` `}` `})` `})` `}` |

发现其创建线程不断打开`status`文件进而查询`TracerPID`字段

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_X6BBGY237C626SJ.webp)

## 开始脱壳

首先尝试Hook `LoadMethod`进而进行脱壳

|  |  |
| --- | --- |
