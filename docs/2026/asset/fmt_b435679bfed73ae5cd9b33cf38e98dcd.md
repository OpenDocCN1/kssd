
```python
from queue import \*
import ida\_bytes
from idc import \*
import idc
from keystone import \*
from capstone import \*
asmer = Ks(KS\_ARCH\_X86, KS\_MODE\_32)
disasmer = Cs(CS\_ARCH\_X86, CS\_MODE\_32)
def disasm(machine\_code, addr=0):
l = ""
for i in disasmer.disasm(machine\_code, addr):
l += "{:8s} {};\n".format(i.mnemonic, i.op\_str)
return l.strip('\n')
def asm(asm\_code, addr=0):
l = b''
for i in asmer.asm(asm\_code, addr)[0]:
l += bytes([i])
return l
def print\_asm(ea):
print(disasm(idc.get\_bytes(ea, idc.get\_item\_size(ea)), ea))
class RelocDSU:
def \_\_init\_\_(self):
self.reloc = {}
def get(self, ea):
if ea not in self.reloc:
if idc.print\_insn\_mnem(ea) == 'jmp' and idc.get\_operand\_type(ea, 0) != idc.o\_reg:
jmp\_ea = idc.get\_operand\_value(ea, 0)
if idc.get\_segm\_name(jmp\_ea) == '.got.plt':
self.reloc[ea] = ea
return self.reloc[ea], False
self.reloc[ea], need\_handle = self.get(idc.get\_operand\_value(ea, 0))
return self.reloc[ea], need\_handle
else:
self.reloc[ea] = ea
if self.reloc[ea] != ea: self.reloc[ea] = self.get(self.reloc[ea])[0]
return self.reloc[ea], idc.get\_segm\_name(self.reloc[ea]) == '.text'
def merge(self, ea, reloc\_ea):
self.reloc[self.get(ea)[0]] = self.get(reloc\_ea)[0]
reloc = RelocDSU()
class Block:
def \_\_init\_\_(self, start\_ea, end\_ea, imm, reg, call\_target):
self.start\_ea = start\_ea
self.end\_ea = end\_ea
self.imm = imm
self.reg = reg
self.call\_target = call\_target
def mov\_code(ea, new\_code\_ea):
return asm(disasm(idc.get\_bytes(ea, idc.get\_item\_size(ea)), ea), new\_code\_ea)
def get\_real\_code(block, new\_code\_ea):
ea = block.call\_target
while True:
if idc.print\_insn\_mnem(ea) == 'cmp':
reg = idc.print\_operand(ea, 0)
imm = idc.get\_operand\_value(ea, 1)
if reg == block.reg and imm == block.imm:
ea += idc.get\_item\_size(ea)
break
ea += idc.get\_item\_size(ea)
# 在 cmp 判断找到对应位置后会依次执行 jnz，popa，popf 三条指令
assert idc.print\_insn\_mnem(ea) == 'jnz'
ea += idc.get\_item\_size(ea)
assert idc.print\_insn\_mnem(ea) == 'popa'
ea += idc.get\_item\_size(ea)
assert idc.print\_insn\_mnem(ea) == 'popf'
ea += idc.get\_item\_size(ea)
if idc.print\_insn\_mnem(ea) == 'pushf': # 第一种特殊情况，实际是 ret 指令。
return True, asm('ret')
new\_code = b''
while True:
if idc.print\_insn\_mnem(ea) == 'jmp': # 第二种特殊情况，跳转过去可能还会有几条实际功能指令。
jmp\_ea = idc.get\_operand\_value(ea, 0)
if idc.print\_insn\_mnem(jmp\_ea) == 'pushf':
break
ea = jmp\_ea
else:
code = mov\_code(ea, new\_code\_ea)
new\_code += code
new\_code\_ea += len(code)
ea += get\_item\_size(ea)
return False, new\_code
def get\_block(start\_ea):
global imm, reg, call\_target
mnem\_list = ['pushf', 'pusha', 'mov', 'call', 'pop']
ea = start\_ea
for i in range(5):
mnem = idc.print\_insn\_mnem(ea)
assert mnem == mnem\_list[i]
if mnem == 'mov':
imm = idc.get\_operand\_value(ea, 1)
reg = idc.print\_operand(ea, 0)
elif mnem == 'call':
call\_target = idc.get\_operand\_value(ea, 0)
ea += idc.get\_item\_size(ea)
return Block(start\_ea, ea, imm, reg, call\_target)
def handle\_one\_branch(branch\_address, new\_code\_ea):
new\_code = b''
ea = branch\_address
while True:
try:
block = get\_block(ea)
is\_ret, real\_code = get\_real\_code(block, new\_code\_ea)
reloc.merge(ea, new\_code\_ea)
ea = block.end\_ea
new\_code\_ea += len(real\_code)
new\_code += real\_code
if is\_ret: break
except:
get\_eip\_func = {0x900: 'ebx', 0x435c: 'eax'}
if idc.print\_insn\_mnem(ea) == 'call' and get\_operand\_value(ea, 0) in get\_eip\_func:
reloc.merge(ea, new\_code\_ea)
real\_code = asm('mov %s, 0x%x' % (get\_eip\_func[get\_operand\_value(ea, 0)], ea + 5), new\_code\_ea)
else:
if idc.print\_insn\_mnem(ea) == 'jmp' and idc.get\_operand\_type(ea, 0) != idc.o\_reg:
reloc.merge(new\_code\_ea, ea)
else:
reloc.merge(ea, new\_code\_ea)
real\_code = mov\_code(ea, new\_code\_ea)
new\_code += real\_code
if real\_code == asm('ret'): break
new\_code\_ea += len(real\_code)
if idc.print\_insn\_mnem(ea) == 'jmp' and idc.get\_operand\_type(ea, 0) != idc.o\_reg: # jmp reg is a swtich
jmp\_ea = idc.get\_operand\_value(ea, 0)
if reloc.get(jmp\_ea)[1] == False: break # 跳回之前的代码说明是个循环
ea = reloc.get(jmp\_ea)[0]
else:
ea += get\_item\_size(ea)
return new\_code
def solve():
entry\_point = 0x48F4
new\_code\_start = 0x96150
new\_code\_ea = new\_code\_start
jmp\_table = (0x892ac, 0x8c000) # [0x8c000 + (eax>>2) - 0x2d54] + 0x8c000
for \_ in range(0x10000): idc.del\_items(new\_code\_ea + \_)
ida\_bytes.patch\_bytes(new\_code\_ea, 0x10000 \* b'\x90')
func\_queue = Queue()
func\_queue.put(entry\_point)
while not func\_queue.empty():
func\_address = func\_queue.get()
if reloc.get(func\_address)[1] == False: continue
reloc.merge(func\_address, new\_code\_ea)
branch\_queue = Queue()
branch\_queue.put(func\_address)
if func\_address == 0x4148: # 特判 0x4148 地址处的函数，读取跳转表。
assert new\_code\_ea == 0x963d0
for eax in range(0x20):
jmp\_target = (ida\_bytes.get\_dword(jmp\_table[0] + eax \* 4) + jmp\_table[1]) & 0xFFFFFFFF
new\_jmp\_target, need\_handle = reloc.get(jmp\_target)
if need\_handle: branch\_queue.put(jmp\_target)
while not branch\_queue.empty():
branch\_address = branch\_queue.get()
new\_code = handle\_one\_branch(branch\_address, new\_code\_ea)
ida\_bytes.patch\_bytes(new\_code\_ea, new\_code)
# 当前 branch 去完混淆之后需要遍历代码找到 call 和 jmp 指令从而找到其他的 function 和 branch 。
ea = new\_code\_ea
while ea < new\_code\_ea + len(new\_code):
idc.create\_insn(ea)
if idc.print\_insn\_mnem(ea) == 'call':
call\_target, need\_handle = reloc.get(get\_operand\_value(ea, 0))
if need\_handle: func\_queue.put(call\_target)
elif idc.print\_insn\_mnem(ea)[0] == 'j' and idc.get\_operand\_type(ea, 0) != idc.o\_reg:
jcc\_target, need\_handle = reloc.get(get\_operand\_value(ea, 0))
if need\_handle == True:
branch\_queue.put(jcc\_target)
ea += get\_item\_size(ea)
new\_code\_ea += len(new\_code)
ea = new\_code\_start
while ea < new\_code\_ea:
idc.create\_insn(ea)
mnem = idc.print\_insn\_mnem(ea)
if mnem == 'call':
call\_target, need\_handle = reloc.get(get\_operand\_value(ea, 0))
assert need\_handle == False
ida\_bytes.patch\_bytes(ea, asm('call 0x%x' % (call\_target), ea))
elif mnem[0] == 'j' and idc.get\_operand\_type(ea, 0) != idc.o\_reg:
jcc\_target, need\_handle = reloc.get(get\_operand\_value(ea, 0))
assert need\_handle == False
ida\_bytes.patch\_bytes(ea, asm('%s 0x%x' % (mnem, jcc\_target), ea).ljust(idc.get\_item\_size(ea), b'\x90'))
elif mnem == 'pushf':
ida\_bytes.patch\_bytes(ea, b'\x90' \* 9)
ea += 9
continue
ea += idc.get\_item\_size(ea)
new\_jmp\_table = (0xA6000 - 0x2D54, 0xA6000)
# 移动并修复跳转表
for eax in range(0x20):
jmp\_target = (ida\_bytes.get\_dword(jmp\_table[0] + eax \* 4) + jmp\_table[1]) & 0xFFFFFFFF
new\_jmp\_target, need\_handle = reloc.get(jmp\_target)
assert need\_handle == False
ida\_bytes.patch\_dword(new\_jmp\_table[0] + eax \* 4, (new\_jmp\_target - new\_jmp\_table[1]) & 0xFFFFFFFF)
need\_patch\_addr = 0x963D7
ida\_bytes.patch\_bytes(need\_patch\_addr, asm('call 0x900;add ebx, 0x%x' % (new\_jmp\_table[1] - (need\_patch\_addr + 5)), need\_patch\_addr)) # 修复指令
ida\_bytes.patch\_bytes(new\_jmp\_table[1] - 0x2d7a, ida\_bytes.get\_bytes(jmp\_table[1] - 0x2d7a, 0x26)) # 复制字符串到正确位置
for \_ in range(0x10000): idc.del\_items(new\_code\_ea + \_)
idc.jumpto(new\_code\_start)
ida\_funcs.add\_func(new\_code\_start)
print("finish")
solve()
```

# 例题：SUSCTF2022 tttree

<a href="elink@70fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8U0L8$3c8W2i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5@1y4e0x3J5x3K6V1$3x3q4)9J5c8X3q4@1N6r3q4U0K9r3#2W2L8Y4c8Q4x3V1k6Q4x3X3c8Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7X3g2$3i4K6u0r3x3U0l9J5x3W2y4g2f1@1y4f1c8W2)9#2k6Y4c8@1N6s2u0W2k6b7`.`.">附件下载链接</a>

首先将 `0x140010074` ，`0x140017EFA` ，`140018C67` 起始处的数据转换为汇编。

观察汇编，发现很多代码块之间相互跳转，因此先按照 `retn` 划分代码块。通过对代码块的观察，发现这些代码块按照 `call $+5;pop rax`（即 `E8 00 00 00 00 58` ） 的出现次数可以分为三种：

- 出现 0 次：

![](upload/attach/202308/959167_DTZz55QVJKNZKGB.png)

本质上是 `其它操作` + `retn` 。

- 出现 1 次：

![](upload/attach/202308/959167_C868DHYH7T47DSR.png)
