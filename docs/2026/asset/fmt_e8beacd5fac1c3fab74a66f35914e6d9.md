
```cpp
int \_\_usercall main\_oldEncrypt@<R0>(
int a1,
int data,
int data\_len,
int data\_cap,
int a5,
int data\_,
int data\_len\_,
int data\_cap\_,
int key1,
int key1\_len,
int key1\_cap,
int key2)
{
// 省略一些参数命名
while ( (unsigned int)&a1 <= \*(\_DWORD \*)(v12 + 8) )
runtime\_morestack\_noctxt();
for ( i = 0; data\_ > (int)i; ++i ) // 对key做第一次处理
{
for ( j = 0; key1 > j; ++j )
{
if ( data\_ <= i )
runtime\_panicIndex();
\*(\_BYTE \*)(a5 + i) ^= \*(\_BYTE \*)(data\_cap\_ + j);
}
}
v44 = main\_swapByteLocation(a5, data\_, data\_len\_, data\_cap\_, key1, key1\_len);// 对key做第二次处理
((void (\*)(void))main\_opensslRandomPseudoBytes)();// 生成指定长度的随机IV
v41 = v29;
v45 = v28;
v42 = v30;
v37 = main\_evpBytesToKey(v44, v38, v39, 32, 16);// 对key做第三次处理
v31 = crypto\_aes\_NewCipher(v37, v44, v38); // 初始化aes
if ( v37 )
{
main\_logger(v26, (int)"new cipher error", 16);
key1\_cap = 0;
result = 0;
key2 = 0;
}
else
{
runtime\_makeslice((int)&uint8, data\_len, data\_len);// slice 扩容 | 申请内存空间
v19 = \*(void (\*\*)(void))(crypto\_cipher\_newCFB(v31, v34, v45, v41, v42, 0) + 16);// AES\_CFB 初始化
v35 = data;
v19(); // ⭐开始加密 偏移CC0D8 crypto\_cipher\_\_cfb\_XORKeyStream | XORKeyStream(&密文, 明文)
v40 = v41 + data\_len;
if ( v42 < v41 + data\_len )
v20 = runtime\_growslice((int)&uint8, v45, v41, v42, v41 + data\_len);
else
v20 = v45;
v43 = v20;
runtime\_memmove(); // 连接初始化向量跟密文
v32 = runtime\_makeslice((int)&uint8, 2 \* v40, 2 \* v40);// 声明 slick 类似这样 make([]int,5,10)，两倍密文长度
v21 = 0;
v22 = 0;
while ( v21 < v40 ) // 循环对密文进行映射，也就是说对密文进行二次处理，得到一个新的密文
{
v24 = \*(\_BYTE \*)(v43 + v21);
if ( v22 >= 2 \* v40 )
runtime\_panicIndex();
\*(\_BYTE \*)(v32 + v22) = a0123456789abcd\_0[v24 >> 4];// a0123456789abcd\_0 => "0123456789abcdef" 字符串映射
v25 = a0123456789abcd\_0[v24 & 0xF];
if ( v22 + 1 >= 2 \* v40 )
runtime\_panicIndex();
\*(\_BYTE \*)(v32 + v22 + 1) = v25;
++v21;
v22 += 2;
}
runtime\_slicebytetostring(v27, 0, v32, 2 \* v40, v32, v35); // 密文转 hex 字符串,是我们最终结果
result = v33;
key1\_cap = v33;
key2 = v36;
}
return result;
}
```

### key & IV

- 这里是 for 循环得到一个 `key3`，在调用函数 `main_swapByteLocation` 得到 `key4`，因为这里的值是固定的，只要 `pwd` 与前面对 `pwd` 的算法没有改变，这里就可以直接拿最后计算的结果即可

```cpp
for ( i = 0; data\_ > i; ++i )
{
for ( j = 0; key1 > j; ++j )
{
if ( data\_ <= i )
runtime\_panicIndex();
\*(a5 + i) ^= \*(data\_cap\_ + j);
}
}
v53 = main\_swapByteLocation(a5, data\_, data\_len\_, data\_cap\_, key1, key1\_len);
main\_opensslRandomPseudoBytes(v27, 0x10u, v30, v32, v34); // 生成指定长度的随机IV
...
main\_evpBytesToKey(v29, v53, v47, v48, 32, 16); // 这里还对 key 做了一次处理
// EvpBytesToKey 函数是 Go 语言中的一个加密函数，用于将一组字节数组转换为一个密钥和初始化向量（IV）。该函数通常用于将一个密码字符串转换为一个加密密钥。
// 该函数的实现基于 OpenSSL 中的 EVP\_BytesToKey 函数，它使用一个称为 EVP 的加密算法来生成密钥和 IV。该算法使用一个称为“密钥派生函数”的过程，将输入的字节数组转换为一个加密密钥和一个随机的初始化向量。
// 在 Go 中，EvpBytesToKey 函数的签名如下：
// func EvpBytesToKey(password []byte, keyLen, ivLen int) ([]byte, []byte)
// 其中，password 表示输入的密码字节数组，keyLen 表示生成的密钥长度，ivLen 表示生成的初始化向量长度。该函数返回两个字节数组，分别表示生成的密钥和初始化向量。
// 需要注意的是，EvpBytesToKey 函数并不是一个标准的加密函数，它只是一个基于 OpenSSL 的实现。因此，它可能不适用于所有的加密场景，需要谨慎使用。
```

```go
// 将原始的密码和盐值转换成密钥和初始向量
// The EVP\_BytesToKey function is a key derivation function that uses a
// hash function to derive a key and initialization vector (IV) from a
// password and salt. It is used in the OpenSSL library to derive keys for
// symmetric encryption algorithms.
func EvpBytesToKey(password, salt []byte, keyLen, ivLen int, md func() hash.Hash) (key, iv []byte) {
// The key and IV are derived from the password and salt using a
// hash function. The OpenSSL library uses the MD5 hash function by default,
// but other hash functions can be used by passing a different hash.Hash
// implementation to the function.
var (
keyBuf = make([]byte, keyLen)
ivBuf = make([]byte, ivLen)
prev []byte
)
for i := 0; i < keyLen+ivLen; {
h := md()
if len(prev) > 0 {
h.Write(prev)
}
h.Write(password)
h.Write(salt)
sum := h.Sum(nil)
copy(sum, keyBuf[i:])
i += len(sum)
if i < keyLen+ivLen {
prev = sum
}
}
copy(key, keyBuf)
copy(iv, ivBuf)
return keyBuf, ivBuf
}
// 该函数的实现过程如下：
// 1.创建一个长度为 keyLen+ivLen 的缓冲区 keyBuf，并创建一个长度为 ivLen 的缓冲区 ivBuf，用于存储密钥和初始向量。
// 2.创建一个空的前一个哈希值 prev，用于存储前一个哈希值的结果。
// 3.循环执行以下步骤，直到生成了足够的密钥和初始向量：
// 3.1创建一个新的哈希函数实例 h。
// 3.2如果 prev 不为空，则将 prev 添加到哈希函数中。
// 3.3将密码和盐值添加到哈希函数中。
// 3.4计算哈希函数的结果，并将其复制到 keyBuf 的相应位置。
// 3.5如果生成的密钥和初始向量长度还不够，则将结果存储到 prev 中，以便下次循环使用。
// 4.将 keyBuf 和 ivBuf 复制到 key 和 iv 中，并返回 key 和 iv。
// 总体来说，EvpBytesToKey 函数的实现过程比较简单，主要是通过哈希函数对密码和盐值进行迭代计算，最终生成密钥和初始向量。
```

- 关于 Key 的处理

```jsx
function hook\_main\_swapByteLocation() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0x10170C), {
onLeave: function (retval) {
console.warn(`key3: ${retval.readCString()}`); // fc9e9a75e633ee4d317b08520b6c3ba9
}
});
}
function hook\_main\_evpBytesToKey() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0x100C38), {
onEnter: function (args) {
console.error(`evpBytesToKey.init.IV: ${args[1].readCString()}`); // IV: iGSkJXWcEwNYZnvt 这个是随机生成的
}, onLeave: function (retval) {
console.error(`evpBytesToKey.init.Key ${ab2Hex(retval.readByteArray(32))}`); // 最终 key
}
});
}
// 日志
key3: fc9e9a75e633ee4d317b08520b6c3ba9
evpBytesToKey.init.IV: tOVJKhoOojtjnqxB
evpBytesToKey.init.Key [A7 91 4B 68 21 10 16 5D DB 0B A7 CF 4E AC BE FC 1C C2 1F 56 F3 41 B8 2A 89 0A CA ED 2C C3 4A 75]
```

- 关于 IV 的处理
  - 生成随机 IV `main_opensslRandomPseudoBytes` 生成指定长度的伪随机字节序列
  - `main_evpBytesToKey` 初始化向量 IV

### `crypto_cipher_newCFB`

- 还原符号就是爽！！！，这里可以直接看到算法名以及使用的模式

```jsx
// 这里有些参数没反编译出来，要么手动识别识别一下（进入函数在返回，F5），要么看汇编代码
v19 = \*(crypto\_cipher\_newCFB(v35, v36, v46, v42, v43, 0) + 16); // AES\_CFB 初始化，返回一个 Stream 使用它进行加密
v37 = a2;
v19(); // ⭐开始加密 偏移CC0D8 crypto\_cipher\_\_cfb\_XORKeyStream | XORKeyStream(&密文, 明文)
// Go 中原型
// stream := cipher.NewCFBEncrypter(block, iv)
// stream.XORKeyStream(ciphertext[aes.BlockSize:], plaintext)
```

```nasm
.text:000FF744 45 33 FF EB BL crypto\_cipher\_newCFB ; 这里调用完方法后返回
.text:000FF744
.text:000FF748 1C 00 9D E5 LDR R0, [SP,#0x58+var\_3C]
.text:000FF74C 10 00 90 E5 LDR R0, [R0,#0x10] ; 可以看到这里对 R0 做了一些处理，下面就是一些参数的准备
.text:000FF750 20 10 9D E5 LDR R1, [SP,#0x58+var\_38]
.text:000FF754 04 10 8D E5 STR R1, [SP,#0x58+var\_54]
.text:000FF758 50 10 9D E5 LDR R1, [SP,#0x58+var\_8]
.text:000FF75C 08 10 8D E5 STR R1, [SP,#0x58+var\_50]
.text:000FF760 60 20 9D E5 LDR R2, [SP,#0x58+data\_len]
.text:000FF764 0C 20 8D E5 STR R2, [SP,#0x58+var\_4C]
.text:000FF768 10 20 8D E5 STR R2, [SP,#0x58+var\_48]
.text:000FF76C 5C 30 9D E5 LDR R3, [SP,#0x58+data]
.text:000FF770 14 30 8D E5 STR R3, [SP,#0x58+var\_44]
.text:000FF774 18 20 8D E5 STR R2, [SP,#0x58+var\_40]
.text:000FF778 64 30 9D E5 LDR R3, [SP,#0x58+data\_cap]
.text:000FF77C 1C 30 8D E5 STR R3, [SP,#0x58+var\_3C]
.text:000FF780 30 FF 2F E1 BLX R0 ; 这里是 v19() 调用函数，所以 hook 这里得到 R0 地址减去基址就是我们的偏移
```

```jsx
function hook\_0xFF780() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0xFF780), {
onEnter(args) {
console.log(`call 加密函数地址偏移：${ptr(this.context.r0 - base)}`); // 容易崩溃，hook 一次崩溃一次，但是能拿到我们要的值
}
});
}
// call 加密函数地址偏移：0xcc0d8
```

![Untitled](upload/attach/202303/918604_WUK7UBHPCKNJDMZ.png)

### `cfb_XORKeyStream`

```jsx
// stream.XORKeyStream(ciphertext[aes.BlockSize:], plaintext) 这是 Go 的原型
// 根据汇编分析这里其实传了四个参数，第一个参数结束后才是我们的密文
function hook\_crypto\_cipher\_\_cfb\_XORKeyStream() {
let base = Module.findBaseAddress("libsojm.so");
Interceptor.attach(base.add(0xCC0D8), {
onEnter: function (args) {
this.arg1 = args[1] // 密文
this.arg2 = args[2] // 密文长度
}, onLeave: function (retval) {
console.error(`cfb\_XORKeyStream 密文 ${ab2Hex(this.arg1.readByteArray(this.arg2.toInt32()))}`);
}
});
}
// evpBytesToKey.init.IV: BGrtHIdEKMvPYAEt
// evpBytesToKey.init.Key [A7 91 4B 68 21 10 16 5D DB 0B A7 CF 4E AC BE FC 1C C2 1F 56 F3 41 B8 2A 89 0A CA ED 2C C3 4A 75]
// cfb\_XORKeyStream 密文 [14 99 4C 88 AE 68 4F 59 0C 09 5B 26 14 AD 8B 72]
```

写了一个 Go 的 `AES_CFB` 进行验证，验证结果是一致的，这里还剩下最后一步
