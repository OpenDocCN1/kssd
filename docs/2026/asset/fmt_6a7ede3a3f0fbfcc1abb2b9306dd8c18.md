
```py
import angr
import logging
import re
import claripy
from keystone import \*
import sys
import time
def fill\_nops(addr, size):
offset = addr - base\_addr
content[offset:offset + size] = b'\x90' \* size
def fill\_jmp(src, dest):
offset = src - base\_addr
if dest != src + 5:
content[offset] = 0xE9
content[offset + 1:offset + 5] = (dest - src - 5).to\_bytes(4, 'little', signed=True)
else:
fill\_nops(src, 5)
def get\_jx\_opcode(jx\_type):
ks = Ks(KS\_ARCH\_X86, KS\_MODE\_64)
code, count = ks.asm(f'{jx\_type} 0xFFFFFFFF')
return b''.join(map(lambda x: x.to\_bytes(1, sys.byteorder), code[0:2]))
def fill\_jx(src, dest, cmov\_type):
offset = src - base\_addr
content[offset:offset + 2] = get\_jx\_opcode(cmov\_type.replace('cmov', 'j'))
content[offset + 2:offset + 6] = (dest - src - 6).to\_bytes(4, 'little', signed=True)
class SuperBlock:
def \_\_init\_\_(self, block\_addr):
self.addr = block\_addr
self.blocks = []
self.size = 0
def insert\_block(self, block):
self.blocks.append(block)
self.size += block.size
@property
def terminator(self):
return self.blocks[-1].capstone.insns[-1]
@property
def successor(self):
terminator = self.terminator
try:
return int(terminator.op\_str, 16)
except:
return None
@property
def insns(self):
insns = []
for block in self.blocks:
insns += block.capstone.insns
return insns
def \_\_repr\_\_(self):
return "<SuperBlock %#08x->%#08x>" % (self.addr, self.addr + self.size)
def \_\_hash\_\_(self):
return hash(('superblock', self.addr))
class FunctionCFG:
def \_\_init\_\_(self, proj, func\_addr, func\_end):
self.proj = proj
self.addr = func\_addr
self.size = func\_end - func\_addr
def get\_super\_block(self, block\_addr):
super\_block = SuperBlock(block\_addr)
cur\_addr = block\_addr
while True:
block = self.proj.factory.block(cur\_addr)
if block.size == 0:
return super\_block
insns = block.capstone.insns
super\_block.insert\_block(block)
cur\_addr += block.size
if insns[-1].mnemonic.startswith('j') or insns[-1].mnemonic == 'ret'\
or insns[-1].address in call\_exit:
return super\_block
def dump\_all\_blocks(self):
curr\_addr = self.addr
blocks = []
while curr\_addr - self.addr < self.size:
super\_block = self.get\_super\_block(curr\_addr)
if super\_block.size == 0:
curr\_addr += 1
continue
blocks.append(super\_block)
curr\_addr += super\_block.size
return blocks
def anaylse\_blocks(self):
all\_blocks = self.dump\_all\_blocks()
relevant\_blocks = []
useless\_blocks = []
for block in all\_blocks:
if block.addr != self.addr and \
(block.successor is not None
or block.terminator.mnemonic == 'ret'
or block.terminator.mnemonic == 'call'
or block.addr in additional\_map.values()):
relevant\_blocks.append(block)
elif block.addr != self.addr:
useless\_blocks.append(block)
return relevant\_blocks, useless\_blocks
def skip\_call(state):
proj.hook(state.addr, angr.SIM\_PROCEDURES["stubs"]["ReturnUnconstrained"](), replace=True)
# Map node onto key and return a dict
def map\_k2n():
state = proj.factory.call\_state(addr=func\_addr)
# Skip call
state.inspect.b('call', when=angr.BP\_BEFORE, action=skip\_call)
simgr = proj.factory.simgr(state)
simgr.explore(find=dispatcher\_node)
found = simgr.found[0]
# Symbolize eax
key = claripy.BVS('key', 32)
found.regs.eax = key
simgr = proj.factory.simgr(found)
visited = []
k2n\_map = {}
while len(simgr.active):
# Filter all visited states to prevent from sticking in a loop
simgr.move(from\_stash='active', to\_stash='deadended', filter\_func=lambda state: state.addr in visited)
for active in simgr.active:
visited.append(active.addr)
if active.addr in relevant\_nodes:
key\_val = active.solver.eval(key)
k2n\_map[key\_val] = active.addr
logger.debug('[\*] Mapped node(%#x)->key(%#x) [%d/%d]'
% (active.addr, key\_val, len(k2n\_map), len(relevant\_nodes)))
simgr.step()
return k2n\_map
def fix\_branch():
# Nop all useless blocks
for block in useless\_blocks:
fill\_nops(block.addr, resize\_map[block.addr] if block.addr in resize\_map else block.size)
relevant\_blocks.append(prologue\_block)
for block in relevant\_blocks:
key\_reg, sub\_key\_reg = 'ecx', 'eax'
key, sub\_key = None, None
cmov\_mnemonic = None
for insn in block.insns:
mnemonic, op\_str = insn.mnemonic, insn.op\_str
if mnemonic == 'mov':
if re.match(rf'{key\_reg}, 0x[0-9a-f]+', op\_str):
tmp\_key = int(op\_str.replace(f'{key\_reg}, ', ''), 16)
if tmp\_key in k2n\_map:
key = tmp\_key
elif re.match(rf'{sub\_key\_reg}, 0x[0-9a-f]+', op\_str):
tmp\_sub\_key = int(op\_str.replace(f'{sub\_key\_reg}, ', ''), 16)
if tmp\_sub\_key in k2n\_map:
sub\_key = tmp\_sub\_key
elif mnemonic.startswith('cmov') and op\_str == f'{key\_reg}, {sub\_key\_reg}':
cmov\_mnemonic = mnemonic
terminator = block.terminator
if block.addr == prologue\_block.addr:
logger.debug('[\*] Progolue node(%#x) has one successor(%#x)' % (block.addr, k2n\_map[sub\_key]))
fill\_jmp(terminator.address, k2n\_map[sub\_key])
elif key is not None and sub\_key is None:
logger.debug('[\*] Node(%#x) has one successor(%#x)' % (block.addr, k2n\_map[key]))
fill\_jmp(terminator.address, k2n\_map[key])
elif key is not None and sub\_key is not None:
cur\_block = block
while cur\_block.successor is not None and cmov\_mnemonic is None:
cur\_block = cfg.get\_super\_block(cur\_block.successor)
for insn in cur\_block.insns:
mnemonic, op\_str = insn.mnemonic, insn.op\_str
if mnemonic.startswith('cmov') and op\_str == f'{key\_reg}, {sub\_key\_reg}':
cmov\_mnemonic = mnemonic
if cmov\_mnemonic is not None:
logger.debug('[\*] Node(%#x) has two successors(%#x, %#x)' % (block.addr, k2n\_map[key], k2n\_map[sub\_key]))
fill\_jx(terminator.address, k2n\_map[sub\_key], cmov\_mnemonic)
fill\_jmp(terminator.address + 6, k2n\_map[key])
else:
logger.error('[\*] Error at block(%#x)' % block.addr)
exit(0)
else:
logger.debug('[\*] Node(%#x) has no successor' % block.addr)
if \_\_name\_\_ == '\_\_main\_\_':
# Initilization
logging.getLogger('cle').setLevel(logging.ERROR)
logging.getLogger('angr').setLevel(logging.ERROR)
logger = logging.getLogger('deobfu')
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.FileHandler(filename='%s.log' % time.strftime("%Y-%m-%d-%H-%M-%S", time.localtime())))
filename = 'crackme-pls.bin.recover'
proj = angr.Project(filename)
base\_addr = proj.loader.main\_object.mapped\_base
reverse\_plt = proj.loader.main\_object.reverse\_plt
with open(filename, 'rb') as file:
content = bytearray(file.read())
# Prepare sigificant arguments
func\_addr, func\_end = 0x402B70, 0x403D81
call\_exit = [0x403D3D, 0x403D7E]
additional\_map = {0x326BF726: 0x403243, 0x3311C734: 0x4035A1, 0x703837C4: 0x4037A0}
resize\_map = {0x403230: 19, 0x40358E: 19, 0x403792: 14}
# Analyse function CFG in a very crude way
cfg = FunctionCFG(proj, func\_addr, func\_end)
# Get prologue block at func addr
prologue\_block = cfg.get\_super\_block(func\_addr)
# The successor of prologue block is dispatcher block
dispatcher\_node = prologue\_block.successor
relevant\_blocks, useless\_blocks = cfg.anaylse\_blocks()
for addr in additional\_map.values():
relevant\_blocks.append(cfg.get\_super\_block(addr))
# Convert block list to addr list
relevant\_nodes = [block.addr for block in relevant\_blocks]
# Map node onto key
k2n\_map = map\_k2n()
k2n\_map.update(additional\_map)
# Fix control flow
fix\_branch()
with open(filename + '.recover\_', 'wb') as file:
file.write(content)
```

# 0x04. 总结

这题的加密逻辑有点复杂，是一个父进程调试子进程的模式，还有其他ELF释放，所以我也没继续分析下去了，感兴趣的可以看看上面贴的WP或者题目的源代码。

至于剩下几个被混淆的函数就比较简单了，跟main函数类似，用v1版本的去混淆脚本能够通杀，所以这里不再赘述。v2版本的代码相当于是在v1的基础上缝缝补补了一些东西，很不优雅，**如果大家有更巧妙的思路欢迎交流讨论**。

新年第一帖!

年初做了道很有意思的代码混淆题`crackme_pls`，自闭了一个多星期后终于想出了相对比较好的去混淆方法，在这里给大伙分享一下分析思路。

题目附件和源代码：817K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9$3#2A6k6r3q4K6i4K6u0r3e0i4W2o6g2p5k6b7N6h3u0D9K9h3y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8Y4c8W2N6r3y4@1k6U0t1H3x3U0u0Q4x3V1k6U0M7X3q4U0K9$3#2W2i4K6g2X3M7r3I4K6

目录

- 0x00. 混淆概览

- 1. 魔改控制流平坦化

- 2. 字符串加密

- 3. 间接函数调用

- 0x01. 反混淆流程（main函数）

- 1. 混淆结构

- 2. 总体思路
