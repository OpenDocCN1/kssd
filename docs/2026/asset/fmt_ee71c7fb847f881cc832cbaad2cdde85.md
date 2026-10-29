
```jsx
function hook\_main\_parsePassphrase\_ret() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0xFFD9C), {
onEnter(args) {
console.log(`call 0xFFD9C ${JSON.stringify(this.context)}`);
console.log(`call 0xFFD9C ${findRangeByAddress(this.context.r0)}`);
console.log(`call 0xFFD9C ${findRangeByAddress(this.context.r1)}`);
}
});
}
```
// BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==
// ..'.4c7e?2fab4f4>1>2>5d0ee4bb61d7b03+...,,!.." 这是直接解码的字符串
// 得到了两个字符串（其实这里就是对我们传入的 PWD 解码对后半部分进行了一些类的运算
8692ece4 6d 49 5a 55 6a 6a 67 68 47 64 00 00 00 00 00 00 mIZUjjghGd......
8692ecc4 34 63 37 65 3f 32 66 61 62 34 66 34 3e 31 3e 32 4c7e?2fab4f4>1>2
8692ecd4 3e 35 64 30 65 65 34 62 62 36 31 64 37 62 30 33 >5d0ee4bb61d7b03
// 关于这里为什么是从栈上取的，是因为它的调用约定并不是 fastcall，分析汇编可知都是栈传递参数跟接受返回值
.text:000FFD64 5B 0D 00 EB BL main\_logger
.text:000FFD64
.text:000FFD68 90 00 9D E5 LDR R0, [SP,#0x7C+arg\_14]
.text:000FFD6C 04 00 8D E5 STR R0, [SP,#0x7C+var\_78] ; int
.text:000FFD70 94 00 9D E5 LDR R0, [SP,#0x7C+data]
.text:000FFD74 08 00 8D E5 STR R0, [SP,#0x7C+var\_74] ; int
.text:000FFD78 98 00 9D E5 LDR R0, [SP,#0x7C+data\_len]
.text:000FFD7C 0C 00 8D E5 STR R0, [SP,#0x7C+var\_70] ; int
.text:000FFD80 1C 05 00 EB BL main\_parsePassphrase
.text:000FFD80
.text:000FFD84 14 00 9D E5 LDR R0, [SP,#0x7C+var\_68]
.text:000FFD88 20 10 9D E5 LDR R1, [SP,#0x7C+var\_5C]
.text:000FFD8C 18 20 9D E5 LDR R2, [SP,#0x7C+var\_64]
.text:000FFD90 1C 30 9D E5 LDR R3, [SP,#0x7C+var\_60]
.text:000FFD94 24 40 9D E5 LDR R4, [SP,#0x7C+var\_58]
.text:000FFD98 28 50 9D E5 LDR R5, [SP,#0x7C+var\_54]
.text:000FFD9C 2C 60 9D E5 LDR R6, [SP,#0x7C+var\_50]
.text:000FFDA0 10 70 DD E5 LDRB R7, [SP,#0x7C+var\_6C]
.text:000FFDA4 11 80 DD E5 LDRB R8, [SP,#0x7C+var\_6C+1]
.text:000FFDA8 00 00 56 E3 CMP R6, #0
.text:000FFDAC 32 01 00 1A BNE loc\_10027C

### main\_md5File

- 这里就是传入了一个文件路径，得到了一个文件的 MD5 值

```cpp
function hook\_main\_md5File() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0xFF94C), {
onEnter: function (args) {
this.arg1 = args[1]
if (this.arg1 == 0x3b) {
console.log(`onEnter main\_md5File arg0:${args[0]} arg1:${args[1]} arg2:${args[2]} arg3:${args[3]} arg4:${args[4]}`);
console.log(`onEnter main\_md5File arg0:${args[0].readCString(0x3b)}`); // 包路径 /data/app/tv.iytqy.cvhaca-gmGCryB\_O0HztLC7rqkhRQ==/base.apk
// console.log(`onEnter main\_md5File arg1:${findRangeByAddress(args[1])}`);// arg1 参数0 长度
// console.log(`onEnter main\_md5File arg2:${findRangeByAddress(args[2])}`); // 地址内容为0
// console.log(`onEnter main\_md5File arg3:${findRangeByAddress(args[3])}`); // 地址内容为0
// console.log(`onEnter main\_md5File arg4:${findRangeByAddress(args[4])}`); // 不知道这个是啥
}
}, onLeave: function (retval) {
if (this.arg1 == 0x3b) {
console.log(`onLeave main\_md5File ${retval.readCString(32)}`); // 678c7f3bf3584a2079295d8834928146
}
}
});
}
```

```cpp
onEnter main\_md5File arg0:0x86844940 arg1:0x3b arg2:0x304f5f42 arg3:0x4c747a48 arg4:0xe6f73709
onEnter main\_md5File arg0:/data/app/tv.iytqy.cvhaca-gmGCryB\_O0HztLC7rqkhRQ==/base.apk
onLeave main\_md5File 678c7f3bf3584a2079295d8834928146
```

- 写个了 Python 代码进行验证，发现结果是对的

```python
import hashlib
filename = '50ash\_5.7.1\_230305\_3.apk'
hasher = hashlib.md5()
with open(filename, 'rb') as f:
buf = f.read()
hasher.update(buf)
md5hash = hasher.hexdigest()
print(md5hash)
# [Running] python -u "c:\Users\Administrator\Desktop\50度灰\getMD5File.py"
# 678c7f3bf3584a2079295d8834928146
```

### encoding\_json\_Marshal

- 这里就是把我们之前的字符串再次序列化

```cpp
v68 = encoding\_json\_Marshal((int)&map\_string\_interface\_, \*(\_DWORD \*)v95); // 转json数据
v18 = v68;
main\_oldEncrypt(v28, v19, v18, v17, v93, v80, v81, v92, v87, v88, v81, 0); // 开始加密
// 所以这里我们直接 hook main\_oldEncrypt 就可以得到加密前的明文了
```

### main\_oldEncrypt

- hook 代码

```cpp
function hook\_main\_oldEncrypt() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0xFF5EC), {
onEnter: function (args) {
console.log(`onEnter main\_oldEncrypt arg0:${args[0]} arg1:${args[1]} arg2:${args[2]} arg3:${args[3]} arg4:${args[4]} arg5:${args[5]} arg6:${args[6]} arg7:${args[7]} arg8:${args[8]} arg9:${args[9]} arg10:${args[10]} arg11:${args[11]}`);
console.warn(`onEnter main\_oldEncrypt arg1:${args[1].readCString()}`);
// console.log(`onEnter main\_oldEncrypt arg4:${findRangeByAddress(args[4])}`);
console.log(`onEnter main\_oldEncrypt arg8:${args[8].readCString()}`); // key1
console.log(`onEnter main\_oldEncrypt arg11:${args[11].readCString()}`); // key2
}, onLeave: function (retval) {
// console.log(`onLeave main\_oldEncrypt ${findRangeByAddress(retval)}`);
console.log(`onLeave main\_oldEncrypt ${retval.readCString()}`);
console.error("------------------------------------------------------------");
}
});
}
```

```cpp
[Pixel::XXXXX]-> call()
onEnter main\_oldEncrypt arg0:0xc arg1:0x869d0580 arg2:0x288 arg3:0x2c0 arg4:0xc96436f0 arg5:0x869d0580 arg6:0x288 arg7:0x2c0 arg8:0x868a4c34 arg9:0x20 arg10:0x2c arg11:0x868a4c54
onEnter main\_oldEncrypt arg1:{"\_\_package\_hash\_\_":"678c7f3bf3584a2079295d8834928146","\_\_package\_name\_\_":"tv.iytqy.cvhaca","app\_status":"9A5C6BDC62AD1CFE45A6578F84E858F9CA1A4F76:2","bundle\_id":"tv.iytqy.cvhaca","new\_player":"fx","system\_app\_type":"local","system\_build\_aff":"","system\_build\_id":"a1000","system\_iid":"923868eb5f543afa55f1f33cfac37d35","system\_oauth\_id":"02274477773a9ac8a2cf3605400ece4b","system\_oauth\_type":"android","system\_token":"023FAC3AFE2285DC98E50CA4C638E0845277DC8CB62CCCC088DA1E3CB679180C97F2688C0DBE197F03950D84900C2A6B050334216C4CC604021DCC624543AFDDEF8D19363E43F9C0B8EEC79BBF269E7DE77B0304FF1034FED885ACA4C74DAC768D26D9FADD","system\_version":"5.7.1"}
onEnter main\_oldEncrypt arg8:4c7e?2fab4f4>1>2>5d0ee4bb61d7b03mIZUjjghGd
onEnter main\_oldEncrypt arg11:mIZUjjghGd
onLeave main\_oldEncrypt 436a4b4d6961565043615063737077526c2e5dd70001b006397ce672c6fe1948f74ac72436bccb766585953419b0a7d4643fd7b98319a6696558f3e129bd7865383a4311256cdc126a00d49126379d5cc3f5a1a0fcd8a6fb6f6293d0ad6e628451cd26efecfaa5021099af5b428da2081533953dcf9a1e3a6f46a560471e6cef6e591b4937c36cd5ccb0de0364279eecf0970e101286faa0d25476e49a2e8b5a24f5d4d2e2c1afb67785f96511f48290864e6fdac548a325f9b30b0c7a99dd2c909518ccac45d9efa2c9973c5f0de468d094e6573afc44d2ffe73d6d79ee76d45fd8fcdf7d45c8a47fca7e85769239da6bfd9e35eaaa59bf3a6bb7741258a42bca7272e611bb8ddd93e807d126c3898f0f1aa9f37af485c140b706fe7a3f0d684eb3132ecd208cf52338a617095e7bb2b2b3eb749c7f7f01d8bfc7d386a2799ae6f015a87ba1519109aa92c4ec2e9d15650a46d239616ec515605d56b7ef3a954f2edbbb85f8c86e2f19c19e6422cc7dcf2277ebd968cf9751f5548f5ae77387985e0dfb7262787b13416138cb5c5952d4d3305a7688910984c75adc0a7f85347917bef0d909ad5a024f2275a5609e2d813f4c43d9bd00b4e82479725b78b197032bf78a163db5f1cbbbd1d09df698cbcb0024646d58d848dd08ad6168413842aa3925a5dd782c6b7d4d58d38a24d3dfe17e8a5e7be5dbeb0ade63d730e10015fd421d1f78f8744322f38f2f2764584dd4f6b36fc3cd0ccd22a9aaaacd92174882fccabc91712364d2afbe83bb6c2ad3d5cafcfd2f09e6e94d79d9dea18c9f7a599640177893b37bdd3e6ed388339d9c1ef9f5d9c197cff02555488bbce4b8ac7366b623b9dadcf3813a6451290094773076b616bbf9f7079c5c39c1ed3c3e211648f862d47b34d644d4c0ae05e0d5fb73d8b046151d2677
// 分析
// onLeave 返回值就是我们最终加密的密文
// arg1 是我们加密前的明文
// 传入了两个key 4c7e?2fab4f4>1>2>5d0ee4bb61d7b03mIZUjjghGd 与 mIZUjjghGd
// 其实这里应该是 4c7e?2fab4f4>1>2>5d0ee4bb61d7b03 与 mIZUjjghGd，他们挨在一起的，所以是一起打印的
```

## **main\_oldEncrypt** 详细分析
