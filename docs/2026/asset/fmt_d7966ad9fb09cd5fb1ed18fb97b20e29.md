
# 加密一 在 `sub_3B9D4` 中

```c
\_\_int64 \_\_fastcall sub\_3B9D4(\_\_int64 result)
{
unsigned \_\_int64 i; // x8
\_\_int64 v2; // x10
int v3; // w11
\_\_int64 v4; // x11
int v5; // w10
int v6; // w12
unsigned \_\_int8 v7; // w14
int v8; // [xsp+Ch] [xbp-4h]
for ( i = 0LL; i < 2; ++i )
{
v2 = 3LL;
v8 = 0;
v3 = 24;
do
{
\*((\_BYTE \*)&v8 + v2) = (\*(\_DWORD \*)(result + 4 \* i) >> v3) ^ v2;
--v2;
v3 -= 8;
}
while ( v2 >= 0 );
HIBYTE(v8) ^= 0x86u;
BYTE2(v8) -= 94;
v4 = 3LL;
BYTE1(v8) ^= 0xD3u;
LOBYTE(v8) = v8 - 28;
\*(\_DWORD \*)(result + 4 \* i) = 0;
v5 = 0;
v6 = 24;
do
{
v7 = \*((\_BYTE \*)&v8 + v4) - v6;
\*((\_BYTE \*)&v8 + v4--) = v7;
v5 += v7 << v6;
\*(\_DWORD \*)(result + 4 \* i) = v5;
v6 -= 8;
}
while ( v4 >= 0 );
}
return result;
}
```

第一个加密函数 `sub_3B9D4` 取出数据的每一位进行加密,其中出现的 `HIBYTE` , `BYTE2`, `BYTE1`,`LOBYTE` 含义如下,假设有数据 `a1=0x12345678`,则

| 符号 | 取出的值 |
| ------------ | -------- |
| `HIBYTE(a1)` | 0x12 |
| `BYTE2(a1)` | 0x34 |
| `BYTE1(a1)` | 0x56 |
| `LOBYTE(a1)` | 0x78 |

我们可以使用 `frida` 去 `hook` `sub_3B9D4` 传入的值以及返回值,观察加密前后的变化,假设我此处在小键盘输入的数字是 `999999999999`,`hex(999999999999)=0xe8d4a50fff`

```js
//hook第一个加密函数,观察数值前后变化
function hook_1_enc(){
// 获取模块
var module = Process.getModuleByName("libsec2023.so")
// 转为函数地址
var addr=module.base.add("0x3B9D4");
// 获取函数入口
var func = new NativePointer(addr.toString());
console.log('[+] hook '+func.toString())
// 函数 hook 钩子附加
Interceptor.attach(func, {
onEnter: function (args) {
console.log('before first enc');
console.log(hexdump(args[0],{
offset: 0,// 相对偏移
length: 64,//dump 的大小
header: true,
ansi: true
}));
},
onLeave: function (retval) {
console.log("after first enc")
console.log(hexdump(retval,{
offset: 0,// 相对偏移
length: 64,//dump 的大小
header: true,
ansi: true
}));
}
});
}
```

![image-20230829154207172](upload/attach/202308/963320_4DZZ4NM43D96MCG.png)

于是第一个算法如下

```python
algorithm_1 = {
(0, "enc"): lambda x: (x - 28) & 0xff,
(1, "enc"): lambda x: x ^ 0xd3,
(2, "enc"): lambda x: (x - 94) & 0xff,
(3, "enc"): lambda x: x ^ 0x86,
(0, "dec"): lambda x: (x + 28) & 0xff,
(1, "dec"): lambda x: x ^ 0xd3,
(2, "dec"): lambda x: (x + 94) & 0xff,
(3, "dec"): lambda x: x ^ 0x86
}
def enc_1(input):
input_byte = bytearray(input.to_bytes(8, 'little'))
for i in range(len(input_byte)):
index = i % 4
input_byte[i] = (algorithm_1[(index, "enc")](input_byte[i] ^ index) - 8 * index) & 0xff
return int.from_bytes(input_byte, 'little')
def dec_1(input):
input_byte = bytearray(input.to_bytes(8, 'little'))
for i in range(len(input_byte)):
index = i % 4
input_byte[i] = (algorithm_1[(index, "dec")]((input_byte[i] + 8 * index) & 0xff)) ^ index
return int.from_bytes(input_byte, 'little')
def mytest_1():
input = 999999999999 # 假设在小键盘输入999999999999
# 验证加密算法
input = enc_1(input)
assert input.to_bytes(8, 'little') == b'\xe3\xd5\x39\x39\xcc\xca\x94\x6d'
# 验证解密算法
input = dec_1(input)
assert input.to_bytes(8, 'little') == b'\xff\x0f\xa5\xd4\xe8\x00\x00\x00'
mytest_1()
```

## 在 `sub_3B9D4` 之后 `bswap32` 分析

在对输入的值进行首轮加密之后,又再次对输入值经过 `bswap32` 函数加密

![image-20230829160716908](upload/attach/202308/963320_524TB88ZRA3N8A6.png)

这个函数对应的 `arm` 汇编为

```c
REV W8, W8
```

那我们去 `arm` 手册看看 `REV` 指令的定义好了

> REV
>
> Byte-Reverse Word reverses the byte order in a 32-bit register.
>
> Operation
>
> ```c
> if ConditionPassed() then
> EncodingSpecificOperations();
> bits(32) result;
> result<31:24> = R[m]<7:0>;
> result<23:16> = R[m]<15:8>;
> result<15:8> = R[m]<23:16>;
> result<7:0> = R[m]<31:24>;
> R[d] = result;
> ```

看 `Operation` 很清楚的知道这就是反转字节,比如

```c
w8 = 0x12345678;
REV W8, W8;
w8;//w8 = 0x78563412
```

所以此处 `bswap32(v6)`,是对 `v6` 进行字节翻转,而 `v6 = HIDWORD(a1)`,故在进行第一个加密函数 `sub_3B9D4` 之后,将首先对输入的高 `32` 位进行加密处理,这在我们后续 `hook` 第二个加密函数 `sub_3A924` 之后,也可以体现出来这一点

## 加密二 `BlackObfuscator` 混淆

我们首先 `hook` 一下 `sub_3A924` 让前后分析的逻辑连贯起来

```js
//hook第2个加密函数,观察数值前后变化
var enc2_count = 0;
var input_2 = [0,0]
function hook_2_enc(){
// 获取模块
var module = Process.getModuleByName("libsec2023.so")
// 转为函数地址
var addr=module.base.add("0x3A924");
// 获取函数入口
var func = new NativePointer(addr.toString());
console.log('[+] hook '+func.toString())
// 函数 hook 钩子附加
Interceptor.attach(func, {
onEnter: function (args) {
console.log('before second enc, count:',enc2_count+1);
input_2[enc2_count] = args[3]
console.log(hexdump(args[1],{
offset: 0,// 相对偏移
length: 64,//dump 的大小
header: true,
ansi: true
}));
},
onLeave: function (retval) {
console.log('after second enc, count:',enc2_count+1);
console.log(hexdump(input_2[enc2_count],{
offset: 0,// 相对偏移
length: 64,//dump 的大小
header: true,
ansi: true
}));
enc2_count+=1;
}
}
);
}
```

在经过第一轮加密后,我们得到了密文 `cc ca 94 6d e3 d5 39 39`,而在此处,第一次调用 `sub_3A924` 的输入为 `6d 94 ca cc`,第二次调用 `sub_3A924` 的输入为 `39 39 d5 e3`,正好对应了上文提到的翻转字节处理

![image-20230830024642669](upload/attach/202308/963320_RNJ2U58ARJQ4WHR.png)

我们进入该函数后,发现如 `v11+1408LL`,`v11 + 1664LL` 等等的 `fastcall` 函数调用,一般这种形式的函数调用在安卓逆向中遇到的话,那大概率就是 `JNIEnv *`

![image-20230829154859854](upload/attach/202308/963320_62HJCJDNU5HAE8Q.png)

在此题中,我们只需要对 `v11` 按下 `Y` 切换类型,然后输入 `JNIEnv *`,就会转换成 `JNIEnv *` 的结构体函数指针如图

![image-20230830000329982](upload/attach/202308/963320_A4CD9GAH3U8AB7J.png)

在这里出现了 `jni` 函数 `GetStaticMethodID`,我们 `hook` 一下这个函数观察调用了什么方法

```js
//hook GetStaticMethodID
function hook_GetStaticMethodID() {
var symbols = Module.enumerateSymbolsSync("libart.so");
var addr_jni = null;
for (var i = 0; i < symbols.length; i++) {
var symbol = symbols[i];
if (symbol.name.indexOf("GetStaticMethodID")!=-1) {
addr_jni = symbol.address;
console.log("find ",symbol.name);
console.log("[+] hook ",addr_jni);
Interceptor.attach(addr_jni, {
onEnter: function (args) {
console.log("call GetStaticMethodID: ",symbol.name);
console.log("name: ",args[2].readCString());
console.log("sig: ",args[3].readCString());
},
onLeave: function (retval) {
//console.log("return val")
//console.log(retval);
}
});
}
}
}
```

在输出中,我们发现 `GetStaticMethodID` 调用了名为 `encrypt` 的方法

![image-20230830000805212](upload/attach/202308/963320_3JHKTBPJPXNKJZ5.png)

但是我们在 `apk` 中却并未发现该方法

![image-20230830000915933](upload/attach/202308/963320_A4V3SGQ62M99THU.png)

那么由此就可以推断出这个方法是由 `dex` 动态加载的

我们可以使用 `frida-dexdump` 来把内存中的 `dex` 给 `dump` 下来,要注意 `frida` 的端口已经被我们修改为了 `1234`,所以这里也要加上 `-H` 参数

![image-20230830001101633](upload/attach/202308/963320_QXNRED5KNWYNVXP.png)

将所有 `dex` `dump` 下来之后,接下来的步骤就是把这些 `dex` 一个一个拖进 `jadx` 里面反编译,去看看哪一个 `dex` 包含 `encrypt` 方法

![image-20230830001653193](upload/attach/202308/963320_ST9DBVVXHV93SPJ.png)

这个控制流一眼看上去就是加了混淆的,据 `FallW1nd` 师傅说是 `BlackObfuscator` 混淆,本来准备去手工分析的,但是想起前几天刚下载了 `Jeb5.1`,就想看看新工具的效果怎样

当我用 `Jeb5.1` 反编译这个 `dex` 之后,这 `BlackObfuscator` 混淆怎么就直接去除了????

好家伙这 `Jeb5.1` 竟然能自动去除控制流平坦化, `crazy!`

![image-20230830022051922](upload/attach/202308/963320_HHA6E427PGARU6E.png)

那么这第二个加密的逻辑相当的清晰,算法如下
