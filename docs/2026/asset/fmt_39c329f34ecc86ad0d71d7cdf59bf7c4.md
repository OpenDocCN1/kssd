
```
for addr in rel\_blk\_info.keys():
\_log.info('=> cleaning relevant block @ %#x' % addr)
asmcfg, nop\_addrs = rel\_blk\_info[addr]
link = fixed\_cfg[hex(addr)]
instrs = [instr for blk in asmcfg.blocks for instr in blk.lines]
last\_instr = instrs[-1]
end\_addr = last\_instr.offset + last\_instr.l
# calculate original length of block before patching
orig\_len = end\_addr - addr
# nop the jmp to pre-dispatcher
nop\_addrs.add(last\_instr.offset)
\_log.debug('nop\_addrs: ' + ', '.join([hex(addr) for addr in nop\_addrs]))
patch = patch\_gen(instrs, asmcfg.loc\_db, nop\_addrs, link)
patch = patch.ljust(orig\_len, b"\x90")
patches[addr] = patch
\_log.debug('patch generated %s\n' % encode\_hex(patch))
\_log.info(">>> NOPing Backbone (%#x - %#x) <<<" % (backbone\_start, backbone\_end))
nop\_len = backbone\_end - backbone\_start
patches[backbone\_start] = b"\x90" \* nop\_len
```

其中`patch\_gen`函数定义如下：

```
def patch\_gen(instrs, loc\_db, nop\_addrs, link):
final\_patch = b""
start\_addr = instrs[0].offset
for instr in instrs:
#omitting useless instructions
if instr.offset not in nop\_addrs:
if instr.is\_subcall():
#generate asm for fixed calls with relative addrs
patch\_addr = start\_addr + len(final\_patch)
tgt = loc\_db.get\_location\_offset(instr.args[0].loc\_key)
\_log.info("CALL %#x" % tgt)
call\_patch\_str = "CALL %s" % rel(tgt, patch\_addr)
\_log.debug("call patch : %s" % call\_patch\_str)
call\_patch = asmb(call\_patch\_str, loc\_db)
final\_patch += call\_patch
\_log.debug("call patch asmb : %s" % encode\_hex(call\_patch))
else:
#add the original bytes
final\_patch += instr.b
patch\_addr = start\_addr + len(final\_patch)
\_log.debug("jmps patch\_addr : %#x", patch\_addr)
jmp\_patches = b""
# cleaning the control flow by patching with real jmps locs
if 'cond' in link:
t\_addr = int(link['true\_next'], 16)
f\_addr = int(link['false\_next'], 16)
jcc = link['cond'].replace('CMOV', 'J')
\_log.info("%s %#x" % (jcc, t\_addr))
\_log.info("JMP %#x" % f\_addr)
patch1\_str = "%s %s" % (jcc, rel(t\_addr, patch\_addr))
jmp\_patches += asmb(patch1\_str, loc\_db)
patch\_addr += len(jmp\_patches)
patch2\_str = "JMP %s" % (rel(f\_addr, patch\_addr))
jmp\_patches += asmb(patch2\_str, loc\_db)
\_log.debug("jmp patches : %s; %s" % (patch1\_str, patch2\_str))
else:
n\_addr = int(link['next'], 16)
\_log.info("JMP %#x" % n\_addr)
patch\_str = "JMP %s" % rel(n\_addr, patch\_addr)
jmp\_patches = asmb(patch\_str, loc\_db)
\_log.debug("jmp patches : %s" % patch\_str)
\_log.debug("jmp patches asmb : %s" % encode\_hex(jmp\_patches))
final\_patch += jmp\_patches
return final\_patch
```

首先检查每个块中是否存在函数调用，如果存在，计算其修补后偏移；否则直接使用原指令：

```
if instr.is\_subcall():
#generate asm for fixed calls with relative addrs
patch\_addr = start\_addr + len(final\_patch)
tgt = loc\_db.get\_location\_offset(instr.args[0].loc\_key)
\_log.info("CALL %#x" % tgt)
call\_patch\_str = "CALL %s" % rel(tgt, patch\_addr)
\_log.debug("call patch : %s" % call\_patch\_str)
call\_patch = asmb(call\_patch\_str, loc\_db)
final\_patch += call\_patch
\_log.debug("call patch asmb : %s" % encode\_hex(call\_patch))
else:
#add the original bytes
final\_patch += instr.b
```

之后修复执行流程：

```
# cleaning the control flow by patching with real jmps locs
if 'cond' in link:
t\_addr = int(link['true\_next'], 16)
f\_addr = int(link['false\_next'], 16)
jcc = link['cond'].replace('CMOV', 'J')
\_log.info("%s %#x" % (jcc, t\_addr))
\_log.info("JMP %#x" % f\_addr)
patch1\_str = "%s %s" % (jcc, rel(t\_addr, patch\_addr))
jmp\_patches += asmb(patch1\_str, loc\_db)
patch\_addr += len(jmp\_patches)
patch2\_str = "JMP %s" % (rel(f\_addr, patch\_addr))
jmp\_patches += asmb(patch2\_str, loc\_db)
\_log.debug("jmp patches : %s; %s" % (patch1\_str, patch2\_str))
else:
n\_addr = int(link['next'], 16)
\_log.info("JMP %#x" % n\_addr)
patch\_str = "JMP %s" % rel(n\_addr, patch\_addr)
jmp\_patches = asmb(patch\_str, loc\_db)
```

如果是条件赋值，替换为条件跳转；否则直接替换为JMP指令。

至此，源码分析结束。如下图所示，可以看到其效果很不错：

![](upload/attach/202107/817966\_43TA6HT5GKE6V9J.jpg)

## 0x02 不足及改进

#### 0x02.1 不支持PE文件

其计算基址是通过访问`sh`属性：

```
section\_ep = cont.bin\_stream.bin.virt.parent.getsectionbyvad(cont.entry\_point).sh
bin\_base\_addr = section\_ep.addr - section\_ep.offset
\_log.info('bin\_base\_addr: %#x' % bin\_base\_addr)
```

![](upload/attach/202107/817966\_HV33GZ8DRVDB7ND.jpg)

故笔者首先增加一参数`baseaddr`：

```
parser.add\_argument('-b',"--baseaddr", help="file base address")
```

之后判断文件类型：

```
try:
if args.baseaddr:
\_log.info('Base Address:'+args.baseaddr)
baseaddr=int(args.baseaddr,16)
elif cont.executable.isPE():
baseaddr=0x400C00
\_log.info('Base Address:%x'%baseaddr)
except AttributeError:
section\_ep = cont.bin\_stream.bin.virt.parent.getsectionbyvad(cont.entry\_point).sh
baseaddr = section\_ep.addr - section\_ep.offset
\_log.info('Base Address:%x'%baseaddr)
```

如果是PE文件且未提供`baseaddr`参数，则直接给其赋值为`0x400C00`；若是ELF文件，则访问`sh`属性。

#### 0x02.2 获取CMOV指令

```
elif len(var\_asg) > 1:
#extracting the condition from the last 3rd line
cond\_mnem = last\_blk.lines[-3].name
\_log.debug('cond used: %s' % cond\_mnem)
var\_asg['cond'] = cond\_mnem
var\_asg['true\_next'] = hex(var\_asg['true\_next'])
var\_asg['false\_next'] = hex(var\_asg['false\_next'])
# map the conditions and possible values dictionary to the cfg info
fixed\_cfg[hex(addr)] = var\_asg
```

通常CMOV指令位于倒数第三条语句，但笔者在测试时发现部分块CMOV指令位于倒数第四条语句：

![](upload/attach/202107/817966\_EQ3RY4SE8MG8QPP.jpg)

故改进如下：

```
elif len(var\_asg) > 1:
#extracting the condition from the last 3rd line
cond\_mnem = last\_blk.lines[-3].name
\_log.debug('cond used: %s' % cond\_mnem)
if cond\_mnem=='MOV':
cond\_mnem = last\_blk.lines[-4].name
var\_asg['cond'] = cond\_mnem
var\_asg['true\_next'] = hex(var\_asg['true\_next'])
var\_asg['false\_next'] = hex(var\_asg['false\_next'])
# map the conditions and possible values dictionary to the cfg info
fixed\_cfg[hex(addr)] = var\_asg
```

#### 0x02.3 调用导入函数

```
if instr.is\_subcall():
#generate asm for fixed calls with relative addrs
patch\_addr = start\_addr + len(final\_patch)
tgt = loc\_db.get\_location\_offset(instr.args[0].loc\_key)
\_log.info("CALL %#x" % tgt)
call\_patch\_str = "CALL %s" % rel(tgt, patch\_addr)
\_log.debug("call patch : %s" % call\_patch\_str)
call\_patch = asmb(call\_patch\_str, loc\_db)
final\_patch += call\_patch
\_log.debug("call patch asmb : %s" % encode\_hex(call\_patch))
```

仅判断是否为函数调用，但并未判断调用函数地址是否为导入函数地址，如下图两种情形所示：

![](upload/attach/202107/817966\_NPSRRJG747AYHN5.jpg)

![](upload/attach/202107/817966\_ZTWDJ8EEMEYBKRT.jpg)

改进如下：

```
if instr.is\_subcall():
if isinstance(instr.args[0],ExprMem) or isinstance(instr.args[0],ExprId):
final\_patch += instr.b
else:
#generate asm for fixed calls with relative addrs
patch\_addr = start\_addr + len(final\_patch)
tgt = loc\_db.get\_location\_offset(instr.args[0].loc\_key)
\_log.info("CALL %#x" % tgt)
call\_patch\_str = "CALL %s" % rel(tgt, patch\_addr)
\_log.debug("call patch : %s" % call\_patch\_str)
call\_patch = asmb(call\_patch\_str, loc\_db)
final\_patch += call\_patch
\_log.debug("call patch asmb : %s" % encode\_hex(call\_patch))
```

#### 0x02.4 get\_phi\_vars

```
def get\_phi\_vars(ircfg):
res = []
blks = list(ircfg.blocks)
irblock = (ircfg.blocks[blks[-1]])
if irblock\_has\_phi(irblock):
for dst, sources in viewitems(irblock[0]):
phi\_vars = sources.args
parent\_blks = get\_phi\_sources\_parent\_block(
ircfg,
irblock.loc\_key,
phi\_vars
)
for var, loc in parent\_blks.items():
irblock = ircfg.get\_block(list(loc)[0])
for asg in irblock:
dst, src = asg.items()[0]
if dst == var:
res += [int(src)]
return res
```

未考虑到由局部变量存储状态变量值情形：

```
mov eax, 0B3584423h
mov ecx, 0CAE581F5h
...
test cl, 1
mov r9d, [rbp+18Ch]
mov r14d, [rbp+17Ch]
cmovnz r9d, r14d
mov [rbp+230h], r9d
jmp loc\_4044AE
```

如果是上述情形，状态变量值往往位于头节点中，故改进如下：
