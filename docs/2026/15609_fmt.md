
# [原创]某无名视频App协议逆向

## 0x00 说明

随便找了一个视频app进行了逆向分析，分享一下协议破解过程，大佬勿喷 (#^.^#)

## 0x01 逻辑分析

### 1、apk分析

![](https://bbs.kanxue.com/upload/attach/202001/590753_HYBA7B64UCHV3RF.jpg)

apk进行了加壳保护，砸壳如下

![](https://bbs.kanxue.com/upload/attach/202001/590753_PDK3HE4QB29G6S8.jpg)

app逻辑基本上都在这个`source-5871984.dex`中，重点看一下这个dex

![](https://bbs.kanxue.com/upload/attach/202001/590753_XHTY6MDWAGPMF6F.jpg)

### 2、请求抓包分析

用户登录

request

![](https://bbs.kanxue.com/upload/attach/202001/590753_A4FHT22GH6AET4J.jpg)

response

![](https://bbs.kanxue.com/upload/attach/202001/590753_DVJYHE3S9T74Z3M.jpg)

请求和响应进行了加密，通过对dex进行关键字搜索找到加解密函数

![](https://bbs.kanxue.com/upload/attach/202001/590753_J5BWQR2JR83PXU7.jpg)

解密函数如下

```
private static SecretKeySpec m68do(String str) {

    byte[] bArr;

    if (str == null) {
        str = "";
    }
    StringBuffer stringBuffer = new StringBuffer(16);
    stringBuffer.append(str);
    while (stringBuffer.length() < 16) {
        stringBuffer.append("0");
    }
    if (stringBuffer.length() > 16) {
        stringBuffer.setLength(16);
    }
    try {
        bArr = stringBuffer.toString().getBytes("UTF-8");
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
        bArr = null;
    }
    return new SecretKeySpec(bArr, "AES");
}

private static byte[] c(byte[] bArr, String str, String str2) {
    try {
        SecretKeySpec secretKeySpec = m68do(str);
        IvParameterSpec dp = dp(str2);
        Cipher instance = Cipher.getInstance("AES/CBC/PKCS5Padding");
        instance.init(2, secretKeySpec, dp);
        return instance.doFinal(bArr);
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}

private static IvParameterSpec dp(String str) {
    byte[] bArr;
    if (str == null) {
        str = "";
    }
    StringBuffer stringBuffer = new StringBuffer(16);
    stringBuffer.append(str);
    while (stringBuffer.length() < 16) {
        stringBuffer.append("0");
    }
    if (stringBuffer.length() > 16) {
        stringBuffer.setLength(16);
    }
    try {
        bArr = stringBuffer.toString().getBytes("UTF-8");
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
        bArr = null;
    }
    return new IvParameterSpec(bArr);
}

public static String p(String str, String str2, String str3) {
    byte[] bArr;
    String str4;
    try {
        bArr = hexString2Bytes(str);
    } catch (Exception e) {
        e.printStackTrace();
        bArr = null;
    }
    byte[] c2 = c(bArr, str2, str3);
    if (c2 == null) {
        return null;
    }
    try {
        str4 = new String(c2, "UTF-8");
    } catch (UnsupportedEncodingException e2) {
        e2.printStackTrace();
        str4 = null;
    }
    return str4;
}

public static byte[] hexString2Bytes(String str) {
    if (isSpace(str)) {
        return null;
    }
    int length = str.length();
    if (length % 2 != 0) {
        StringBuilder sb = new StringBuilder();
        sb.append("0");
        sb.append(str);
        str = sb.toString();
        length++;
    }
    char[] charArray = str.toUpperCase().toCharArray();
    byte[] bArr = new byte[(length >> 1)];
    for (int i = 0; i < length; i += 2) {
        bArr[i >> 1] = (byte) ((b(charArray[i]) << 4) | b(charArray[i + 1]));
    }
    return bArr;
}

private static int b(char c2) {
    if (c2 >= '0' && c2 <= '9') {
        return c2 - '0';
    }
    if (c2 >= 'A' && c2 <= 'F') {
        return (c2 - 'A') + 10;
    }
    throw new IllegalArgumentException();
}

private static boolean isSpace(String str) {
    if (str == null) {
        return true;
    }
    int length = str.length();
    for (int i = 0; i < length; i++) {
        if (!Character.isWhitespace(str.charAt(i))) {
            return false;
        }
    }
    return true;
}
```

`keyFromJNI`

![](https://bbs.kanxue.com/upload/attach/202001/590753_5NQDHVY5WCBNJCT.jpg)

`viFromJNI`

![](https://bbs.kanxue.com/upload/attach/202001/590753_X7DWMJ74RZ59EMX.jpg)

使用方法：

```
p(asString, keyFromJNI, viFromJNI)
```

密文

```
2593194C061134517AAD27E8DBE5421F795A486AF409F823715A36810114EF9056078D9CC978A00C93C7AD6C885506B8AAFE23AE69279F7FAD794437FBBF35BD7876D4C9255D0FF5790B1A54359BC7573F5BDFC430AC20063955557028CFF18A
```

解密

```
{"token_id":"2652308","token":"834b2815a8739569f1b38a25027c337b","serverCode":"20190705-17"}
```

并进行了`preferences`存储

![](https://bbs.kanxue.com/upload/attach/202001/590753_H2NU3NAYACVHQPG.jpg)

后续所有请求都使用了`token`、`token_id`并进行请求加密

例如列表请求

request

![](https://bbs.kanxue.com/upload/attach/202001/590753_6VZ39A9WNXUASCS.jpg)

```
request_key=234A4D14B2693D1ABB7AFD62E132CEDC -- > {"pid":"1"}
```

response

![](https://bbs.kanxue.com/upload/attach/202001/HFHHUS4GUH27CMA.jpg)

解密如下

![](https://bbs.kanxue.com/upload/attach/202001/590753_ZJE34FFVVDRGP7Q.jpg)

ps：后续的视频的详情使用了`vod_id`进行请求，然后进行播放，很简单这里就不进一步展示了。

例如检查更新，哦哦 明文的，略过。

## 0x03 总结

1、这个视频app进行的防护比较典型 加壳+协议加密

2、用户信息是以android客户端进行的token生成 用户登录->信息注册->请求加密->响应解密->播放视频

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
