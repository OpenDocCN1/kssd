
```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"encoding/hex"
	"fmt"
)

func main() {
	data := []byte("abcdef0123456789")
	data_len := len(data)
	enc_text := make([]byte, data_len)
	iv := []byte("BGrtHIdEKMvPYAEt")
	key, _ := hex.DecodeString("a7914b682110165ddb0ba7cf4eacbefc1cc21f56f341b82a890acaed2cc34a75")
	newCipher, err := aes.NewCipher(key)
	if err != nil {
		fmt.Println(err)
		return
	}
	cfbEncrypter := cipher.NewCFBEncrypter(newCipher, iv)
	cfbEncrypter.XORKeyStream(enc_text, data)
	fmt.Println(hex.EncodeToString(enc_text)) // 14994c88ae684f590c095b2614ad8b72
}
```

### 密文映射

```javascript
while (v22 < v41) // 循环对密文进行映射，也就是说对密文进行二次处理
{
	v25 = *(v44 + v22);
	if (v23 >= 2 * v41)
		runtime_panicIndex();
	*(v21 + v23) = a0123456789abcd_0[v25 >> 4]; // a0123456789abcd_0 => "0123456789abcdef" 字符串映射
	v26 = a0123456789abcd_0[v25 & 0xF];
	if (v23 + 1 >= 2 * v41)
		runtime_panicIndex();
	*(v21 + v23 + 1) = v26;
	++v22;
	v23 += 2;
}
------------------------------------------------------------------------------------
var str = "0123456789abcdef"
for (let i = 0; i < cipher.len; i++) {
	var tmp = cipher[i]
	cipher[i << 1] = str[tmp >> 4]
	cipher[i << 1 ^ 1] = str[tmp & 0xf]
	// cipher[i * 2 ^ 1] = str[tmp & f] // 等同上一句代码
}
```

### Golang 明文测试 `abcdef0123456789`

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"encoding/hex"
	"fmt"
)

func main() {
	data := []byte("abcdef0123456789")
	dataLen := len(data)
	ciphertextBytes := make([]byte, dataLen)
	iv := []byte("eoyEeRxfiXqDImeQ")
	key, _ := hex.DecodeString("a7914b682110165ddb0ba7cf4eacbefc1cc21f56f341b82a890acaed2cc34a75")
	newCipher, err := aes.NewCipher(key)
	if err != nil {
		fmt.Println(err)
		return
	}
	cfbEncrypter := cipher.NewCFBEncrypter(newCipher, iv)
	cfbEncrypter.XORKeyStream(ciphertextBytes, data)
	ciphertextStr := hex.EncodeToString(ciphertextBytes)
	fmt.Println("AES_CFB 密文:", ciphertextStr)
	newCiphertextBytes := make([]byte, len(ciphertextBytes)*2)
	mapStr := "0123456789abcdef"
	for i := 0; i < len(iv); i++ {
		tmp := iv[i]
		newCiphertextBytes[i<<1] = mapStr[tmp>>4]
		newCiphertextBytes[i<<1^1] = mapStr[tmp&0xf]
	}
	fmt.Println("最终密文结果:", string(newCiphertextBytes)+ciphertextStr)
	fmt.Println("最终密文结果:", hex.EncodeToString(iv)+ciphertextStr)
	// AES_CFB 密文: 9368e37e4c81bf11ec0b3c14e5d16f45
	// 最终密文结果: 656f79456552786669587144496d65519368e37e4c81bf11ec0b3c14e5d16f45
	// 最终密文结果: 656f79456552786669587144496d65519368e37e4c81bf11ec0b3c14e5d16f45
	// hook 输出结果
	// onEnter main_oldEncrypt arg0:0xc arg1:0x86990280 arg2:0x10 arg3:0x10 arg4:0xcfc046f0 arg5:0x86990280 arg6:0x10 arg7:0x10 arg8:0x8698e8d4 arg9:0x20 arg10:0x2c arg11:0x8698e8f4
	// onEnter main_oldEncrypt arg1:abcdef0123456789'a'
	// onEnter main_oldEncrypt arg8:4c7e?2fab4f4>1>2>5d0ee4bb61d7b03mIZUjjghGd
	// onEnter main_oldEncrypt arg11:mIZUjjghGd
	// onEnter main_oldEncrypt arg0:0x0 arg1:0x868023dc arg2:0x86918000 arg3:0x57 arg4:0xcfc046f0 arg5:0x86990280 arg6:0x10 arg7:0x10 arg8:0x8698e8d4 arg9:0x20 arg10:0x2c arg11:0x8698e8f4
	// onEnter main_oldEncrypt arg1:
	// onEnter main_oldEncrypt arg8:4c7e?2fab4f4>1>2>5d0ee4bb61d7b03mIZUjjghGd
	// onEnter main_oldEncrypt arg11:mIZUjjghGd
	// key3: fc9e9a75e633ee4d317b08520b6c3ba9
	// evpBytesToKey.init.IV: eoyEeRxfiXqDImeQ
	// evpBytesToKey.init.Key [A7 91 4B 68 21 10 16 5D DB 0B A7 CF 4E AC BE FC 1C C2 1F 56 F3 41 B8 2A 89 0A CA ED 2C C3 4A 75]
	// cfb_XORKeyStream 密文 [93 68 E3 7E 4C 81 BF 11 EC 0B 3C 14 E5 D1 6F 45]
	// onLeave main_oldEncrypt 656f79456552786669587144496d65519368e37e4c81bf11ec0b3c14e5d16f45
	// ------------------------------------------------------------
	// onLeave main_oldEncrypt 656f79456552786669587144496d65519368e37e4c81bf11ec0b3c14e5d16f45
	// ------------------------------------------------------------
	// encrypt(abcdef0123456789, "BwcnBzRjN2U/MmZhYjRmND4xPjI+NWQwZWU0YmI2MWQ3YjAzKw8cEywsIS4BIg==") :=> 656f79456552786669587144496d65519368e37e4c81bf11ec0b3c14e5d16f45
}
```

### Golang 明文测试 JSON 数据
