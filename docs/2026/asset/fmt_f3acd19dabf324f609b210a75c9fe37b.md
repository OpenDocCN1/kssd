
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

`vim` `/home/ubuntu/anaconda3/envs/ql1-4-4/lib/python3.9/site-packages/qiling/core.py`

```
def __init__( self, argv: Sequence[str] = None, rootfs: str = r'.', env: MutableMapping[AnyStr, AnyStr] = {}, code: bytes = None, ostype: Union[str, QL_OS] = None, archtype: Union[str, QL_ARCH] = None, verbose: QL_VERBOSE = QL_VERBOSE.DEFAULT, profile: str = None, console: bool = True, log_file=None, log_override=None, log_plain: bool = False, multithread: bool = False, filter=None, stop: QL_STOP = QL_STOP.NONE, *, endian: Optional[QL_ENDIAN] = None, thumb: bool = False, libcache: bool = False, enable_procfs: bool = True # <---------------------------- 新添加的参数，默认值为True ):
self.enable_procfs = enable_procfs # <---------------------------- 设置 enable_procfs 属性
······
```

修改 `linux.py` 中的代码新增对参数`enable_procfs`的判断：

```
def run(self):
# if not self.ql.code and not self.ql.loader.is_driver:   # 原来的代码
if self.ql.enable_procfs and not self.ql.code and not self.ql.loader.is_driver:
self.setup_procfs()
```

这样我们只需要将`ql`改成下面的代码即可：

```
ql = Qiling(path, rootfs, enable_procfs=False)
```

然后修改我们的`challenge10`测试一下，运行成功！

```
Challenge 8: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x14) = 0x14
[=]     openat(fd = 0xffffff9c, path = 0x5555555556c1, flags = 0x0, mode = 0x0) = 0x3
[=]     read(fd = 0x3, buf = 0x80000000dcc0, length = 0x3f) = 0x9
[=]     close(fd = 0x3) = 0x0
Challenge 9: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x14) = 0x14
Challenge 10: SOLVED
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x15) = 0x15
```

# Challenge 11：绕过CPUID校验

> Challenge 11: Bypass CPUID/MIDR_EL1 checks.

伪代码如下：

```
unsigned __int64 __fastcall challenge11(_BYTE *a1)
{
int v7;
// [rsp+1Ch] [rbp-34h]
int v8;
// [rsp+24h] [rbp-2Ch]
char s[4];
// [rsp+2Bh] [rbp-25h] BYREF
char v10[4];
// [rsp+2Fh] [rbp-21h] BYREF
char v11[4];
// [rsp+33h] [rbp-1Dh] BYREF
unsigned __int64 v12;
// [rsp+38h] [rbp-18h]
v12 = __readfsqword(0x28u);
_RAX = 0x40000000LL;
__asm { cpuid }
v7 = _RCX;
v8 = _RDX;
if ( __PAIR64__(_RBX, _RCX) == 0x696C6951614C676ELL && (_DWORD)_RDX == 538976354 )
*a1 = 1;
sprintf(
s,
"%c%c%c%c",
(unsigned int)_RBX,
(unsigned int)((int)_RBX >> 8),
(unsigned int)((int)_RBX >> 16),
(unsigned int)((int)_RBX >> 24));
sprintf(
v10,
"%c%c%c%c",
(unsigned int)v7,
(unsigned int)(v7 >> 8),
(unsigned int)(v7 >> 16),
(unsigned int)(v7 >> 24));
sprintf(
v11,
"%c%c%c%c",
(unsigned int)v8,
(unsigned int)(v8 >> 8),
(unsigned int)(v8 >> 16),
(unsigned int)(v8 >> 24));
return __readfsqword(0x28u) ^ v12;
}
```

关键部分汇编代码如下：

```
.text:000055555555518F                 cpuid
.text:0000555555555191                 mov     eax, edx
.text:0000555555555193                 mov     esi, ebx
.text:0000555555555195                 mov     [rbp+var_30], esi
.text:0000555555555198                 mov     [rbp+var_34], ecx
.text:000055555555519B                 mov     [rbp+var_2C], eax
.text:000055555555519E                 cmp     [rbp+var_30], 696C6951h
.text:00005555555551A5                 jnz     short loc_5555555551C0
.text:00005555555551A7                 cmp     [rbp+var_34], 614C676Eh
.text:00005555555551AE                 jnz     short loc_5555555551C0
.text:00005555555551B0                 cmp     [rbp+var_2C], 20202062h
.text:00005555555551B7                 jnz     short loc_5555555551C0
.text:00005555555551B9                 mov     rax, [rbp+var_48]
.text:00005555555551BD                 mov     byte ptr [rax], 1
```

我们需要让`*a=1`，就要让`if`的判断顺利通过，`if ( __PAIR64__(_RBX, _RCX) == 0x696C6951614C676ELL && (_DWORD)_RDX == 538976354 )` 的含义为：

1. `rbx`和`rcx`两个32位寄存器组成的64位值等于`0x696C6951614C676ELL`

2. `RDX`寄存器的低32位等于`538976354`

因此我们跳过`cpuid`指令修改寄存器即可。

## writeup 11

```
def hook_cpuid(ql, address, size):    
if ql.mem.read(address, size) == b'\x0F\xA2': # CPUID指令的机器码是 0F A2
regs = ql.arch.regs
regs.ebx = 0x696C6951
regs.ecx = 0x614C676E
regs.edx = 0x20202062
regs.rip += 2 # 跳过cpuid指令防止被cpuid篡改 000055555555518F -> 0000555555555191

def challenge11(ql):
begin, end = 0, 0
for info in ql.mem.map_info:
print(info)
if info[2] == 5 and 'qilinglab-x86_64' in info[3]:
begin, end = info[:2]
print(f"{begin} -> {end}")

ql.hook_code(hook_cpuid, begin=begin, end=end)
```

最后可以看到全部题目均已解决：

```
You solved 11/11 of the challenges
[=]     write(fd = 0x1, buf = 0x55555575a260, count = 0x23) = 0x23
[=]     exit_group(code = 0x0) = ?
```

最后完整的代码为：
