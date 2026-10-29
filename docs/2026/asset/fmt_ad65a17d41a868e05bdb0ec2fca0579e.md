
```
function hook_jni() {
Java.perform(function () {
var a = Java.use("com.a.easyjni.a");
a["a"].implementation = function (bArr) {
console.log("a.a is called: bArr=${bArr}", bArr);
var result = this["a"](bArr);
console.log("a.a result=${result}", result);
return result;
};
})
}
function main() {
hook_jni()
}
setImmediate(main)
```

输入：`abcdefgabcdefgabcdefgab` 结果：`2eXTQS8AQC7I2CntQAkO2A1/QeQx2eG=`

输入的字符串经过 `a(byte[] bArr)` 加密后，带入 so 里面经过处理，剩下逻辑在 so 里面，我们来看 so 里面的代码：

![图片描述](upload/attach/202312/961239_R6MXFT7HKJHJUP3.webp)

对于这段代码我们分析下：

这个函数的主要逻辑是对字符串进行一系列处理，包括颠倒、移位等操作，然后将结果与预定义字符串进行比较。如果最终处理后的字符串与预定义字符串相等，函数返回 `true`；否则，返回 `false`。

根据 so 的代码逻辑，我们还原她的正向和逆向算法：

```
def ncheck(v5):
if len(v5) == 32:
v12 = [0] * 32
for i in range(16):
v7 = v12[i]
v12[i] = v5[i + 16]
v8 = v5[i]
v12[i + 16] = v8
v9 = 0
while v9 < 30:
v10 = v9 < 30
v13 = v12[v9]
v12[v9] = v12[v9 + 1]
v12[v9 + 1] = v13
v9 += 2
print("".join(v12))
return "".join(v12)
else:
return 0
def reverse_ncheck(ciphertext):
if len(ciphertext) == 32:
v12 = ['\x00'] * 32
# 第一步：将逆向处理后的字符串进行逆颠倒
for i in range(16):
v12[i] = ciphertext[i + 16]
v12[i + 16] = ciphertext[i]
# 第二步：将处理后的字符串进行逆移位操作
v9 = 0
while v9 < 30:
v13 = v12[v9]
v12[v9] = v12[v9 + 1]
v12[v9 + 1] = v13
v9 += 2
# 输出逆向还原的字符串
reversed_result = "".join(v12)
return reversed_result
else:
return 0
```

解密函数 `reverse_ncheck(ciphertext)` 带入 `MbT3sQgX039i3g==AQOoMQFPskB1Bsc7` 得到：

`QAoOQMPFks1BsB7cbM3TQsXg30i9g3==`

so 函数的分析到此为止。

现在来还原下，java 层的 `a()` 方法：

```
class A:
a = ['i', '5', 'j', 'L', 'W', '7', 'S', '0', 'G', 'X', '6', 'u', 'f', '1', 'c', 'v', '3', 'n', 'y', '4', 'q', '8', 'e', 's', '2', 'Q', '+', 'b', 'd', 'k', 'Y', 'g', 'K', 'O', 'I', 'T', '/', 't', 'A', 'x', 'U', 'r', 'F', 'l', 'V', 'P', 'z', 'h', 'm', 'o', 'w', '9', 'B', 'H', 'C', 'M', 'D', 'p', 'E', 'a', 'J', 'R', 'Z', 'N']
@staticmethod
def base64_encode(bArr):
sb = []
for i in range(0, len(bArr), 3):
bArr2 = [0] * 4
b = 0
for i2 in range(3):
if i + i2 <= len(bArr) - 1:
bArr2[i2] = b | ((bArr[i + i2] & 255) >> ((i2 * 2) + 2))
b = ((((bArr[i + i2] & 255) << (((2 - i2) * 2) + 2)) & 255) >> 2)
else:
bArr2[i2] = b
b = 64
bArr2[3] = b
for i3 in range(4):
if bArr2[i3] <= 63:
sb.append(A.a[bArr2[i3]])
else:
sb.append('=')
return ''.join(sb)
# 示例用法：
obj = A()
result = obj.base64_encode(b"abcdefgabcdefgabcdefgab")
print(result)
```

那么逆向还原这个算法：

```
class Base64New:
RANGE = 255
Base64ByteToStr =['i', '5', 'j', 'L', 'W', '7', 'S', '0', 'G', 'X', '6', 'u', 'f', '1', 'c', 'v', '3', 'n', 'y', '4', 'q', '8', 'e', 's', '2', 'Q', '+', 'b', 'd', 'k', 'Y', 'g', 'K', 'O', 'I', 'T', '/', 't', 'A', 'x', 'U', 'r', 'F', 'l', 'V', 'P', 'z', 'h', 'm', 'o', 'w', '9', 'B', 'H', 'C', 'M', 'D', 'p', 'E', 'a', 'J', 'R', 'Z', 'N']
StrToBase64Byte = {char: idx for idx, char in enumerate(Base64ByteToStr)}
def Base64Encode(self, bytes):
res = []
for i in range(0, len(bytes), 3):
enBytes = [0] * 4
tmp = 0
for k in range(3):
if i + k < len(bytes):
enBytes[k] = ((bytes[i + k] & 255) >> ((k * 2) + 2)) | tmp
tmp = (((bytes[i + k] & 255) << ((2 - k) * 2 + 2)) & 255) >> 2
else:
enBytes[k] = tmp
tmp = 64
enBytes[3] = tmp
for k2 in range(4):
if enBytes[k2] <= 63:
res.append(self.Base64ByteToStr[enBytes[k2]])
else:
res.append('=')
return ''.join(res)
def Base64Decode(self, encoded_str):
decoded_bytes = bytearray()
for i in range(0, len(encoded_str), 4):
enChars = encoded_str[i:i + 4]
deBytes = [0] * 4
for k in range(4):
if enChars[k] != '=':
deBytes[k] = self.StrToBase64Byte[enChars[k]]
decoded_bytes.extend((((deBytes[0] << 2) & 255) | (deBytes[1] >> 4)).to_bytes(1, 'big'))
if enChars[2] != '=' and len(enChars) > 2:
decoded_bytes.extend((((deBytes[1] << 4) & 255) | (deBytes[2] >> 2)).to_bytes(1, 'big'))
if enChars[3] != '=' and len(enChars) > 3:
decoded_bytes.extend((((deBytes[2] << 6) & 255) | deBytes[3]).to_bytes(1, 'big'))
return decoded_bytes
# 示例用法:
data_to_encode = b'abcdefgabcdefgabcdefgab'
base64_encoder = Base64New()
encoded_data = base64_encoder.Base64Encode(data_to_encode)
print("编码结果:", encoded_data)
decoded_data = base64_encoder.Base64Decode(encoded_data)
print("解码结果:", decoded_data)
```

![图片描述](upload/attach/202312/961239_Q4M99AAB558K2V9.webp)

算法加解密验证没问题。

将 `QAoOQMPFks1BsB7cbM3TQsXg30i9g3==` 带入解密函数中，得到结果：`flag{just_ANot#er_@p3}`

### 0x05 easy_so

这题的整体逻辑并不复制，加密算法主要在 so 里面，读懂代码，并能推导出来，这个很重要。接下来我们详细的分析，先来看代码：

![图片描述](upload/attach/202312/961239_4V47R3CD3KZG4RT.webp)

java 层的代码逻辑非常清晰，就是验证输入值在 so 里面的检测，来看 so 代码：

![图片描述](upload/attach/202312/961239_GV9A4QZQ4G963Q5.webp)

1. 首先获取字符串的长度 `v4`，然后根据长度动态分配内存。这里使用了 `malloc` 函数，它会分配 `v6` 个字节的内存空间，并将其起始地址存储在 `v7` 中。
2. 将字符串 `v3` 的内容复制到动态分配的内存中。首先判断内存是否足够，如果不够则进行相应的内存清零操作。然后使用 `memcpy` 将字符串复制到动态分配的内存中。
3. 对字符串的每两个字符进行交换，即将字符串的第一个字符与第二个字符、第三个字符与第四个字符交换位置。
4. 对字符串进行了更多的字符交换操作，将第一个字符与第二个字符交换，然后每两个字符交换一次，直到字符串末尾。
5. `return strcmp(v8, "f72c5a36569418a20907b55be5bf95ad") == 0` 最后，通过 `strcmp` 函数比较经过处理的字符串 `v8` 是否与固定字符串 `"f72c5a36569418a20907b55be5bf95ad"` 相等，如果相等则返回 0，表示字符串相等。

算法还原：

```
def TestDec(a1):
v2 = 0
if len(a1) >= 2:
while v2 < len(a1) // 2:
a1[v2], a1[v2 + 16] = a1[v2 + 16], a1[v2]
v2 += 1
result = ord(a1[0]) if a1 else 0
if a1:
a1[0], a1[1] = a1[1], chr(result)
result = len(a1)
if result >= 3:
v6 = 0
while v6 + 4 <= result:
a1[v6 + 2], a1[v6 + 3] = a1[v6 + 3], a1[v6 + 2]
v6 += 2
return a1
def reverse_TestDec(encrypted_str):
a1 = list(encrypted_str)
v2 = 0
if len(a1) >= 2:
while v2 < len(a1) // 2:
a1[v2], a1[v2 + 16] = a1[v2 + 16], a1[v2]
v2 += 1
result = len(a1)
if result >= 3:
v6 = result - 4
while v6 >= 0:
a1[v6 + 2], a1[v6 + 3] = a1[v6 + 3], a1[v6 + 2]
v6 -= 2
if result > 0:
a1[0], a1[1] = a1[1], chr(ord(a1[0]))
return "".join(a1)
# Example usage:
original_str = "f72c5a36569418a20907b55be5bf95ad"
encrypted_result = TestDec(list(original_str))
print(encrypted_result)
encrypted_result = list("f72c5a36569418a20907b55be5bf95ad")
print(encrypted_result)
decrypted_str = reverse_TestDec(encrypted_result)
print("Original String:", original_str)
print("Encrypted Result:", "".join(encrypted_result))
print("Decrypted String:", decrypted_str)
```

![图片描述](upload/attach/202312/961239_N5HVYDSG7ZF57A2.webp)

`90705bb55efb59da7fc2a5636549812a` 验证成功

### 0x01 背景

这是由一系列 CTF easy 项目组成的，难度系数也是逐渐增加的。不管是简单还是复杂，获取 flag 不是目的，目的是分析和解决问题的方法和思路，不管多少道题，还是逆向其他网站，解决问题的方式是差不多，接下来我们将详细的分析每个项目的算法。

### 0x02 easy_apk

这题考察的是 base64 算法的知识点，通过 `Base64Encode()` 对输入的字符串进行加密，然后与字符串 `5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs=` 进行比较，如果相等就完成破解。这题我们用 2 种解法来获取 flag, 先来看代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202312/961239_6YYQBPQC6WRRBUZ.webp)

既然如此，我们前期不考虑验证通过的问题，先正向还原他的加密，输入一个简单的数据：例如 `abcd` 编码结果: `JHqKyv==`

有了输入值，和返回值，我们用 `python` 还原他的算法：

#### 解法1：

|  |  |
| --- | --- |
