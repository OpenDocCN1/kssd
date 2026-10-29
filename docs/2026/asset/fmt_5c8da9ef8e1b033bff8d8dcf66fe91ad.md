
```python
from emu_utils import *
from unicorn import *
from unicorn.arm64_const import *
# from ida_bytes import *
def trace_back_insn_with_target(insn_queue, target_reg):
for insn in insn_queue:
if (target_reg in insn.op_str):
if (insn.mnemonic == 'add'):
print(insn.mnemonic + '\t' + insn.op_str)
if (insn.mnemonic == 'ldr'):
print(insn.mnemonic + '\t' + insn.op_str)
if (insn.mnemonic == 'csel'):
print(insn.mnemonic + '\t' + insn.op_str)
def log_hook(emu, addr, size, user_data):
disasm = get_disasm(emu, addr, size)
print(hex(addr) + '\t' + disasm.mnemonic + '\t' + disasm.op_str)
def step_over_hook(emu, addr, size, none):
disasm = get_disasm(emu, addr, size)
if (disasm.mnemonic == 'bl' or disasm.mnemonic == 'blr'):
emu.reg_write(UC_ARM64_REG_PC, addr + size)
if (disasm.mnemonic == 'ret'):
print('function returned')
emu.emu_stop()
if (addr == 0x3ac68):
emu.reg_write(UC_ARM64_REG_W10, 0xEECF7326)
def normal_hook(emu, addr, size, insn_queue):
global const_value, offset_value, cond, cond_value, uncond_value
disasm = get_disasm(emu, addr, size)
reg_maps = get_reg_maps()
insn_queue.insert(0, disasm)
if (len(insn_queue) > 8):
insn_queue.pop()
if (disasm.mnemonic == 'csel'):
cond_value = emu.reg_read(reg_maps[disasm.op_str.split(', ')[1]])
uncond_value = emu.reg_read(reg_maps[disasm.op_str.split(', ')[2]])
cond = disasm.op_str.split(', ')[3]
if (disasm.mnemonic == 'cset'):
cond_value = 1
uncond_value = 0
cond = disasm.op_str.split(', ')[1]
if (disasm.mnemonic == 'ldr'):
if (len(disasm.op_str.split(', ')) == 3):
offset_value = emu.reg_read(
reg_maps[disasm.op_str.split(', ')[1].split('[')[1]])
elif (len(disasm.op_str.split(', ')) == 4):
offset_value = emu.reg_read(
reg_maps[disasm.op_str.split(', ')[1].split('[')[1]])
cond_value *= 8
if (disasm.mnemonic == 'add' and '#' not in disasm.op_str and 'w' not in disasm.op_str):
const_value = emu.reg_read(reg_maps[disasm.op_str.split(', ')[2]])
if (disasm.mnemonic == 'br'):
print('on br insn')
target_reg = disasm.op_str
trace_back_insn_with_target(insn_queue, target_reg)
print(hex(const_value), hex(offset_value),
cond, cond_value, uncond_value)
cond_addr = emu.mem_read(offset_value + cond_value, 4)
cond_addr = (int.from_bytes(
cond_addr, byteorder='little') + const_value) & 0xffffffff
uncond_addr = emu.mem_read(offset_value + uncond_value, 4)
uncond_addr = (int.from_bytes(
uncond_addr, byteorder='little') + const_value) & 0xffffffff
patch_asm = b''
patch_asm += get_asm('b' + cond + ' ' + hex(cond_addr), addr - 4)
patch_asm += get_asm('b ' + hex(uncond_addr), addr)
# patch_bytes(addr - 4, patch_asm)
emu.reg_write(UC_ARM64_REG_PC, addr + size)
def emulate_execution(filename, start_addr, hook_func, user_data):
emu = Uc(UC_ARCH_ARM64, UC_MODE_LITTLE_ENDIAN)
textSec = get_section(filename, '.text')
dataSec = get_section(filename, '.data')
textSec_entry = textSec.header['sh_addr']
textSec_size = textSec.header['sh_size']
textSec_raw = textSec.header['sh_offset']
TEXT_BASE = textSec_entry >> 12 << 12
TEXT_SIZE = (textSec_size + 0x1000) >> 12 << 12
TEXT_RBASE = textSec_raw >> 12 << 12
dataSec_entry = dataSec.header['sh_addr']
dataSec_size = dataSec.header['sh_size']
dataSec_raw = dataSec.header['sh_offset']
DATA_BASE = dataSec_entry >> 12 << 12
DATA_SIZE = (dataSec_size + 0x1000) >> 12 << 12
DATA_RBASE = dataSec_raw >> 12 << 12
VOID_1_BASE = 0x00000000
VOID_1_SIZE = TEXT_BASE
VOID_2_BASE = TEXT_BASE + TEXT_SIZE
VOID_2_SIZE = DATA_BASE - VOID_2_BASE
STACK_BASE = DATA_BASE + DATA_SIZE
STACK_SIZE = 0xFFFFFFFF - STACK_BASE >> 12 << 12
emu.mem_map(VOID_1_BASE, VOID_1_SIZE)
emu.mem_map(TEXT_BASE, TEXT_SIZE)
emu.mem_map(DATA_BASE, DATA_SIZE)
emu.mem_map(VOID_2_BASE, VOID_2_SIZE)
emu.mem_map(STACK_BASE, STACK_SIZE)
emu.mem_write(TEXT_BASE, read(filename)[TEXT_RBASE:TEXT_RBASE+TEXT_SIZE])
emu.mem_write(DATA_BASE, read(filename)[DATA_RBASE:DATA_RBASE+DATA_SIZE])
emu.reg_write(UC_ARM64_REG_FP, STACK_BASE + 0x1000)
emu.reg_write(UC_ARM64_REG_SP, STACK_BASE + STACK_SIZE // 2)
emu.hook_add(UC_HOOK_CODE, log_hook)
emu.hook_add(UC_HOOK_CODE, step_over_hook, user_data)
emu.hook_add(UC_HOOK_CODE, hook_func, user_data)
emu.emu_start(start_addr, 0x0)
if __name__ == '__main__':
filename = './libsec2023.so'
start_addr = 0x3ac68
insn_queue = []
emulate_execution(filename, start_addr, normal_hook, insn_queue)
```

### 分析加密函数

去混淆后即可分析函数，经过分析，可以知道是对`input`以4字节为一组分别进行加密。

![图片描述](upload/attach/202304/887676_JSS4VCMYUCRFR4Z.png)

其中`enc_1`逻辑较为明显，显然是可逆的

![图片描述](upload/attach/202304/887676_KQ69N4NHDPP7Z55.png)

下面这个函数实现了上面的加密算法，经过`frida`验证输入输出是正确的。

```c
void EncryptObfusedRoundI(uint32_t ipt[]) {
for (ssize_t idx = 0; idx < 2; ++idx) {
uint8_t buf[4] = {0}, tmp[4] = {0};
for (ssize_t i = 0; i < 4; i++) {
buf[i] = ((uint8_t *)&ipt[idx])[i] ^ i;
}
buf[3] ^= 0x86;
buf[2] -= 0x5E;
buf[1] ^= 0xD3;
buf[0] -= 0x1C;
for (ssize_t i = 0; i < 4; i++) {
tmp[i] = buf[i] - i * 8;
}
ipt[idx] = *(uint32_t *)&tmp;
}
}
```

而`enc_2`中调用了一个类中的方法，我们只知道这个方法叫什么，但是并不知道是哪个类下的方法，因此我们需要hook `GetStaticMethodID`这个JNI函数。

[圖片]

通过hook JNI函数找到调用解密函数的类

通过hook，我们可以知道调用的是`Sec2023.Encrypt`方法，但是`JADX`中并没有找到，不能直接分析。说明这个类是由壳动态加载的，因此我们需要找办法dump下来`dex`。

```javascript
function HookGetStaticMethodID() {
var libart = TraverseModules('single', {name: 'libart.so'});
var GetStaticMethodID = libart.base.add(0x3A87B4);
Interceptor.attach(GetStaticMethodID, {
onEnter: function(args) {
var clazz = args[1];
var name = args[2].readUtf8String();
var sig = args[3].readUtf8String();
// get clazz name
var clazz_name = Java.vm.getEnv().getClassName(clazz);
console.log(`[*] GetStaticMethodID called: ${clazz_name} ${name} ${sig}`);
}
});
}
```

### 使用`frida-dexdump`获取壳加载的`Dex` && 分析`Sec2023.Encrypt`方法

通过这个工具我们可以很轻松地在内存里找到相关的`dex`，dump下来就可以接着分析了。

![图片描述](upload/attach/202304/887676_RT2PR94AFT56STX.png)

打开分析，可以发现加了`BlackObfuscator`，而我并没有现成的解决方案可以参考和使用，那就只能推理和猜猜看了。

![图片描述](upload/attach/202304/887676_YN9H93V2MU3VEX4.png)

### 分析被`BlackObfuscator`混淆的算法

这里我用一种很简单粗暴的办法，就是把每个`case`中的真实指令直接复制出来，然后可以发现变量之间的运算有明显的逻辑关系，经过推理和结合`frida`分析验证（上面的`frida`脚本log了加密完的`key`），可以推理出正确的加密算法

```c
uint32_t EncryptObfusedRoundII(uint32_t ipt) {
uint32_t n1 = 0, n2 = 0;
uint8_t result[4] = {0}, *buf = (uint8_t *)&ipt;
uint8_t key[8] = {0x32, 0xCD, 0xFF, 0x98, 0x19, 0xB2, 0x7C, 0x9A};
n1 = __builtin_bswap32(*(uint32_t *)buf);
n2 = (n1 >> 7) | (n1 << 25);
*(uint32_t *)result = __builtin_bswap32(n2);
for (int idx = 0; idx < 8; idx++) {
result[idx] = (result[idx] ^ key[idx % 8]);
result[idx] = (result[idx] + idx);
}
return *(uint32_t *)result;
}
```

### 使用`frida`分析VM && 使用`frida`解释VM

分析完上面提到的算法之后，我们可以在此处找到一个函数的调用，它改变了`input`的值，所以应为加密算法

![图片描述](upload/attach/202304/887676_FX4MRNCV7DE4HSF.png)

对这两个类进行分析，可以知道上面那个类中的都是`opcode` `dict`的`key`，下面那个类是VM的`Handler`和一些相关的字段。

![图片描述](upload/attach/202304/887676_7QSFNBDF9MJZZ9D.png)

VM直接分析不好分析，我们需要侧面去分析。两种常用的方法：一是把指令数组和`handler`自己实现在程序外部自行分析，二是直接hook来分析。这里选择直接hook通过打log来还原VM逻辑。

首先需要解析VM类的`this`指针

```javascript
function ParseVMClazz(thiz) {
var fields = thiz.add(0x10);
var opcode = fields.add(0x00);
var input = fields.add(0x08);
var buf = fields.add(0x10);
var x = fields.add(0x18);
var cnt = fields.add(0x1C)
var maybe_idx = fields.add(0x20);
var num = fields.add(0x24);
var opdict = fields.add(0x28);
var vm_context = {
opcode: opcode.readPointer(),
input: input.readPointer(),
buf: buf.readPointer(),
x: x.readS32(),
cnt: cnt.readS32(),
maybe_idx: maybe_idx.readS32(),
num: num.readS32(),
opdict: opdict.readPointer(),
};
return vm_context;
}
```

然后通过手动hook一个个`handler`来实现VM解释器
