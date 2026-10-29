
```
from capstone import *

from capstone.arm import *



CODE = b"\xf1\x02\x03\x0e\x00\x00\xa0\xe3\x02\x30\xc1\xe7\x00\x00\x53\xe3"



md = Cs(CS_ARCH_ARM, CS_MODE_ARM)

for i in md.disasm(CODE, 0x1000):

    print("%x:\t%s\t%s" %(i.address, i.mnemonic, i.op_str))

```

上面这段代码的输出如下

```
1000:    mcreq    p2, #0, r0, c3, c1, #7

1004:    mov    r0, #0

1008:    strb    r3, [r1, r2]

100c:    cmp    r3, #0

```

可以看出，`Capstone`十分简单，而我们编写调试器的反汇编部分，也是如此简单。

## 实现反汇编
```
def dump_asm(self, addr, size):

    md = self._capstone #构造函数中初始化的Capstone对象 Cs(CS_ARCH_ARM, CS_MODE_ARM) 

    code = self._mu.mem_read(addr, size)

    for ins in md.disasm(code, addr):

        print("%s:\t%s\t%s" % (self.sym_handler(ins.address), ins.mnemonic, ins.op_str))

```

## dump 内存

Python的`hexdump`模块的偏移量不支持修改，所以我魔改了`hexdump`的代码实现了这个功能。
```
def dump_mem(self, addr, size):

    data = self._mu.mem_read(addr, size)

    print (advance_dump(data, addr))

```

魔改的`hexdump`
```
def advance_dump(data, base):

    PY3K = sys.version_info >= (3, 0)

    generator = hexdump.genchunks(data, 16)

    retstr = ''

    for addr, d in enumerate(generator):

        # 00000000:
        line = '%08X: ' % (base + addr * 16)

        # 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
        dumpstr = hexdump.dump(d)

        line += dumpstr[:8 * 3]

        if len(d) > 8:  # insert separator if needed
            line += ' ' + dumpstr[8 * 3:]

        # ................
        # calculate indentation, which may be different for the last line
        pad = 2

        if len(d) < 16:
            pad += 3 * (16 - len(d))

        if len(d) <= 8:
            pad += 1

        line += ' ' * pad

        for byte in d:
            # printable ASCII range 0x20 to 0x7E
            if not PY3K:
                byte = ord(byte)

            if 0x20 <= byte <= 0x7E:
                line += chr(byte)
            else:
                line += '.'

        retstr += line + '\n'

    return retstr

```

## 寄存器显示

`Unicorn`中的寄存器都是以常量来管理的，所以我们要把寄存器常量和文本映射起来。
```
REG_ARM = {arm_const.UC_ARM_REG_R0: "R0",

           arm_const.UC_ARM_REG_R1: "R1",

           arm_const.UC_ARM_REG_R2: "R2",

           arm_const.UC_ARM_REG_R3: "R3",

           arm_const.UC_ARM_REG_R4: "R4",

           arm_const.UC_ARM_REG_R5: "R5",

           arm_const.UC_ARM_REG_R6: "R6",

           arm_const.UC_ARM_REG_R7: "R7",

           arm_const.UC_ARM_REG_R8: "R8",

           arm_const.UC_ARM_REG_R9: "R9",

           arm_const.UC_ARM_REG_R10: "R10",

           arm_const.UC_ARM_REG_R11: "R11",

           arm_const.UC_ARM_REG_R12: "R12",

           arm_const.UC_ARM_REG_R13: "R13",

           arm_const.UC_ARM_REG_R14: "R14",

           arm_const.UC_ARM_REG_R15: "R15",

           arm_const.UC_ARM_REG_PC:  "PC",

           arm_const.UC_ARM_REG_SP:  "SP",

           arm_const.UC_ARM_REG_LR:  "LR"

           }



REG_TABLE = {UC_ARCH_ARM: REG_ARM}

........

self._regs = REG_TABLE[self._arch] #根据平台切换寄存器映射表，目前只有ARM



def dump_reg(self):

    result_format = ''

    count = 0

    for rid in self._regs:

        rname = self._regs[rid]

        value = self._mu.reg_read(rid)

        if count < 4:

            result_format = result_format + '  ' + rname + '=' + hex(value)

            count += 1

        else:

            count = 0

            result_format += '\n' + rname + '=' + hex(value)

    print (result_format)

##### 修改寄存器的值

def write_reg(self, reg_name, value):

    for rid in self._regs:

        rname = self._regs[rid]

        if rname == reg_name:

            self._mu.reg_write(rid, value)

            return

    print ("[Debugger Error] Reg not found:%s " % reg_name)

```

## 实现单步调试

使用`list`来记录所有断点的地址，当`UC_HOOK_CODE`的`callback`被调用的时候检查`address`是否在断点列表中，如果当前指令的执行地址没有断点，则直接返回，如果存在断点，则打印寄存器信息、最近的汇编代码、等待用户指令等操作。

单步分为两种：步入、步过。

步入：如果遇到`call`、`bl`指令会跟随进入（跟随每一条指令）

步过：遇到`call`、`bl`等指令不不进入（跟随每一条地址相邻的指令）

我这个思路可能有点牵强，因为按照地址相邻，会误判跳转指令，但是为了简单起见，我只好这么做。

下面是断点管理相关的代码。
```
def list_bpt(self):

    for idx in range(len(self._list_bpt)):

        print ("[%d] %s" % (idx, self.sym_handler(self._list_bpt[idx])))



def add_bpt(self, addr):

    self._list_bpt.append(addr)



def del_bpt(self, addr):

    self._list_bpt.remove(addr)

```

`callback`中判断断点。

`_tmp_bpt`是临时断点，用于支持步过。

`_is_step`步入标记，每条指令都停下来。
```
def _dbg_trace(mu, address, size, self): #self就是user_info传入

    self._tracks.append(address)

    if not self._is_step and self._tmp_bpt == 0:#步入 步过

        if address not in self._list_bpt:

            return

    if self._tmp_bpt != address and self._tmp_bpt != 0:

        return

```

处理调试指令
```
elif command[0] == 's' or command[0] == 'step': # 步入
    self._tmp_bpt = 0 # 清除步过标记
    self._is_step = True # 设置步入标记
    break



elif command[0] == 'n' or command[0] == 'next': # 步过
    self._tmp_bpt = address + size # 设置下一条指令地址断点
    self._is_step = False # 清除步入标记
    break



elif command[0] == 'r' or command[0] == 'run': # 运行
    self._tmp_bpt = 0
    self._is_step = False
    break

```

## 完整代码
