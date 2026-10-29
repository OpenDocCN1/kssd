
```
function hook_test() {
Java.perform(function () {
var EncryptUtil = Java.use("com.tivicloud.utils.EncryptUtil");
EncryptUtil.nativeEncrypt.implementation = function (a) {
console.log("=============================")
// console.log("EncryptUtil.nativeEncrypt args[0] 被加密的字符串: ", a);
var result = this.nativeEncrypt(a);
console.log("Sign的结果： ", result);
return result;
}
var b64 = Java.use("android.util.Base64");
var str = Java.use("java.lang.String");
EncryptUtil.nativeAES.implementation = function (a) {
console.log("=============================")
// console.log("EncryptUtil.nativeEncrypt args[0] 被加密的字符串: ", a);
var result = this.nativeAES(a);
console.log("base64数据", str.$new(b64.encode(result, 0)));
return result;
}
var libgavesec = Module.findBaseAddress("libgavesec.so");
// encrypt
Interceptor.attach(libgavesec.add(0x1C8C), {
onEnter: function (args) {
console.log("encrypt args[0] 被加密的字符串: ", Memory.readCString(args[0]));
this.args1 = args[1];
},
onLeave: function (retVal) {
console.log("encrypt args[1] 处理过后的数据： ", hexdump(this.args1, {
offset: 0,
length: 32,
header: true,
ansi: false
}));
}
})
// SHA1::Result
Interceptor.attach(libgavesec.add(0x2AA4), {
onEnter: function (args) {
// console.log("SHA1::Result args[1] 加密前的数据", hexdump(args[1], {
// offset: 0,
// length: 64,
// header: true,
// ansi: false
// }));
this.args1 = args[1];
},
onLeave: function (retVal) {
console.log("SHA1::Result args[1] 加密后的数据", hexdump(this.args1, {
offset: 0,
length: 32,
header: true,
ansi: false
}));
}
})
// char2hexInt
Interceptor.attach(libgavesec.add(0x1B34), {
onEnter: function (args) {
// console.log("char2hexInt 参数：", args[0]);
this.args1 = args[1];
},
onLeave: function (retVal) {
// console.log("char2hexInt 返回值:", retVal);
}
})
// AES::AES
Interceptor.attach(libgavesec.add(0x3274), {
onEnter: function (args) {
console.log("AES密钥：", hexdump(args[1], {
offset: 0,
length: 32,
header: true,
ansi: false
}));
},
onLeave: function (retVal) {
}
})
// AES::Cipher
Interceptor.attach(libgavesec.add(0x397C), {
onEnter: function (args) {
console.log("AES::Cipher args[1]：", Memory.readCString(args[1]));
this.args1 = args[1];
},
onLeave: function (retVal) {
}
});
});
};
setImmediate(function () {
Java.perform(function () {
hook_test();
});
});
```

### 简单的加密代码的实现

```
from Crypto.Cipher import AES
from urllib import parse
from binascii import b2a_hex, a2b_hex, b2a_base64
import hashlib
"""
aes加密算法
ECB模式
"""
class Aes128_(object):
    def __init__(self):
        self.key = b"14ca829f017c0357"
        self.mode = AES.MODE_ECB

    def add_to_16(self, text):
        if len(text.encode('utf-8')) % 16:
            add = 16 - len(text.encode('utf-8')) % 16
        else:
            add = 0
        text = text + ("\0"*add) # 明文 + \00填充
        return text.encode('utf-8')

    def encrypt(self, text):
        text = self.add_to_16(text)
        cryptos = AES.new(self.key, self.mode)
        cipher_text = cryptos.encrypt(text)
        # return b2a_hex(cipher_text)
        return b2a_base64(cipher_text)

    def decrypto(self, text):
        cryptor = AES.new(self.key, self.mode)
        plain_text = cryptor.decrypt(a2b_hex(text))
        return bytes.decode(plain_text).rstrip('\0')

def getSign(data):
    sign = ''
    a211034f8af4e6b = '211034f8af4e6b9546c19ae13ed099553319b6c3'
    for i in range(40):
        print(hex(int(data[i], 16) ^ int(a211034f8af4e6b[i], 16)))
        sign += str(hex(int(data[i], 16) ^ int(a211034f8af4e6b[i], 16)))[2:3]
    return sign

if __name__ == "__main__":
    data = bytes(parse.quote('{"mobile_operator":"","app_versioncode":"4","app_version":"2.1.0.22277","connection_type":"WIFI","os_version":"8.1.0","source":"SDK","user_id":"16073535","os_lang":"zh","sdk_version":"3.1.4","imei":"359906070277673","os_name":"Android","login_token":"9f6037c931db9a9cfcbb991966fad614","lang":"zh","udid":"10e1ef4509a2340eb0276bb99d186506","nudid":"155c2b70-fc97-4005-abc7-6d6c4329ed23","app_id":"10054","channel_id":"10006","tdid":"39fb282a761c17e80c069332fc6a2ffa9"}fff18b83431fa3a83b9de80c1e413bde'), encoding="utf-8")
    sha1 = hashlib.sha1(data)
    sha1_data = sha1.hexdigest()
    sign = getSign(sha1_data)
    aes_data = str(Aes128_().encrypt('{"sign":"'+sign+'","data": "' + parse.quote('{"mobile_operator":"","app_versioncode":"4","app_version":"2.1.0.22277","connection_type":"WIFI","os_version":"8.1.0","source":"SDK","user_id":"16073535","os_lang":"zh","sdk_version":"3.1.4","imei":"359906070277673","os_name":"Android","login_token":"9f6037c931db9a9cfcbb991966fad614","lang":"zh","udid":"10e1ef4509a2340eb0276bb99d186506","nudid":"155c2b70-fc97-4005-abc7-6d6c4329ed23","app_id":"10054","channel_id":"10006","tdid":"39fb282a761c17e80c069332fc6a2ffa9"}"}')), encoding='utf8').replace("\n", "")
    print(aes_data)
```

### 结果对比

hook：

![图片描述](upload/attach/202011/900086_JX674ANCFKZ9NFU.png)

python：

![图片描述](upload/attach/202011/900086_H9TRX9S38F4Q2SX.png)

初学逆向，写的不好请见谅！

前因：没有别的，就是群里有老哥发的文件，初学逆向，为自己记录顺带分享给和我一样刚开始起步学习的人，作为第一次在看雪上发帖子，有点小激动！

```
请求包：
POST /api/adult/check_guest HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: Dalvik/2.1.0 (Linux; U; Android 8.1.0; Pixel Build/OPM1.171019.011)
Host: service.kuyangsh.cn
Connection: Keep-Alive
Accept-Encoding: gzip
Content-Length: 1108

a=6ih0KN8TFL5%2FQT%2FN3JY63ZovsWQyeIxHjBLHp1GGwjUNYVaGJLC%2FYZenRFKbeqsgIoI4rD1atROr%0Ajkl1p7eobNMARMel19oiGkl5hRD72vOn9zyNbERMe8Cj3b24Ru4wc2mWbbnamKVKPepkaa2mqpJl%0Akp4%2Fa5udSz0UbDR6cTwLCRWeKb60H%2Fir4vzZv1OfwQF%2FXJQsuBmxH2F6wp9CJkk9WYchx4LQU%2FS3%0AjQfpQY2iZWwsmHAiyZGVsfZIgXTvhJygpT8vH268Py5JspYZoho0RRrx4BjUfs5boif6rEMpd5PC%0AiZTLhIPovbPpoZQJC7d%2BWPFtwjPT7Ljyzgz5QxtctnZqa4qMMkfFwAIeA7lj2wJZeZBD%2F%2BoU5R45%0AMEFK6OMrMXB1M%2BC4dBt7Rd384SYhe%2BAEp0gKNHGkrpxWImFcPAaalajyijs6V4Wl4res9CWuEE5W%0Ayj5ehtmPc6uZGuo5ns6THfDwnho8BiFOnH0QoqJEeyVdTsCzOiMwfBJJldB7qbsTfUlLbSnpq4tf%0AurPbVMVKQbk4ui1XgDH5v%2FptUHirNK0IHT%2BBms8wQ%2BSX3BcMLKFiWI0OBAjUydqcJpIi0sPSWpfh%0A2k1nmMlvPnljfc7P12iF8nFHoFWRYQPVie46K%2Bhd4%2FttkyrZ4Gy3WM6zWdmnED3h6CCgZ4rEe0DB%0AN5Dj8lJJbsOAE%2FxWcYDguyj8WkUET3yLB%2FBZQx%2BsOn9otWzjwROdhfi6V7OObXZ5XoGUIDffKaFu%0ADnvTinsAh%2FvjcSDVq%2BHyD%2FzUeRceMf6uQruBhHRzikSb2Oz0Zfxld7rqmWYZ8aIBe1DMRJuXecB%2F%0AAsBu4VVuVVfQf4hlCpVNmKnX6huuMkHtCptCLaD0pkmuY7X7OEfCsudtFIco%2F7gXaQ5aXgfCs7GJ%0AzAMxfpCRm4vnF0kc8nQ4OWlexOV5t65k%2B4eDt8wY91%2BIFHcq%2BIwZPR3e41oeKriHlbsPdocNOkeg%0AeyUw%2FXlAY97IpZA%3D%0A
返回包：
{
"ret": 0,
"msg": "没有错误",
"data": {
"needs_check": "0",
"needs_bind": "0",
"needs_guest_bind": 0,
"user_is_guest": -1
}
}
```

这里定位很简单，直接全局搜索关键词：`"a"` 就可以了，因为可以看到POST数据包中只有一个字段就是`a`，但是这里需要带上双引号，来到如下

![图片描述](https://bbs.kanxue.com/upload/attach/202011/900086_7DX6GFR5SJWFBNE.png)

可以看到最关键的加密代码就是：

```
String a = m3385a((Map<String, String>) hashMap, initFromXML.getAppKey());
```

`initFromXML.getAppKey()`则正常返回一个定值

![图片描述](https://bbs.kanxue.com/upload/attach/202011/900086_3SDM7QFW3KTAZZE.png)

跟进`m3385a()`函数可以发现，定义如下：

注意：这里发现`JSONObject`是处于`org.json.JSONObject`，系统自带的库中的类，这里也能作为一种hook的方法定位
