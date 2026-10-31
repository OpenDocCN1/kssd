
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16

```
var wasm_mode = 1;
if(wasm_mode){
  prefix = "../../";
  path = "test/mjsunit/wasm/wasm-module-builder.js";
  d8.file.execute(prefix + path);
}
builder.addFunction("poc", makeSig([], [kWasmI32,kWasmI32,kWasmI32]))
  .exportFunc()
  .addBody([]);
const instance = builder.instantiate();
let {poc} = instance.exports;
poc();
```

其实是可以的

![](https://bbs.kanxue.com/upload/attach/202511/971428_STWM67X7TDVSZWM.png)

当前的栈回溯（部分

|   |   |
| --- | --- |
