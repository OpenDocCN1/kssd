
# [原创]记一次简单的sign分析

前因：没有别的，就是群里有老哥发的文件，初学逆向，为自己记录顺带分享给和我一样刚开始起步学习的人，作为第一次在看雪上发帖子，有点小激动！

```
请求包：
POST /api/adult/check\_guest HTTP/1.1
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
"needs\_check": "0",
"needs\_bind": "0",
"needs\_guest\_bind": 0,
"user\_is\_guest": -1
}
}
```

这里定位很简单，直接全局搜索关键词：`"a"`就可以了，因为可以看到POST数据包中只有一个字段就是`a`，但是这里需要带上双引号，来到如下

![图片描述](upload/attach/202011/900086\_7DX6GFR5SJWFBNE.png)

可以看到最关键的加密代码就是：

```
String a = m3385a((Map<String, String>) hashMap, initFromXML.getAppKey());
```

`initFromXML.getAppKey()`则正常返回一个定值

![图片描述](upload/attach/202011/900086\_3SDM7QFW3KTAZZE.png)

跟进`m3385a`函数可以发现，定义如下：

注意：这里发现`JSONObject`是处于`org.json.JSONObject`，系统自带的库中的类，这里也能作为一种hook的方法定位

```
public static String m3385a(Map<String, String> map, String str) {
if (C1074a.f2394c || TivicloudController.f2343a) {
map.put("testing", "1");
}
try {
JSONObject jSONObject = new JSONObject(map);
Debug.m3346d("params : " + jSONObject.toString());
String encode = URLEncoder.encode(jSONObject.toString(), "UTF-8");
String encryptString = EncryptUtil.encryptString(encode + str);
JSONObject jSONObject2 = new JSONObject();
try {
jSONObject2.put("sign", encryptString);
jSONObject2.put(C0882di.C0883a.DATA, encode);
} catch (JSONException e) {
Debug.m3354w((Exception) e);
}
return new String(Base64.encode(EncryptUtil.nativeAES(jSONObject2.toString()), 0), "UTF-8");
} catch (UnsupportedEncodingException e2) {
Debug.m3354w((Exception) e2);
return null;
}
}
```

对`JSONObejct`的hook效果如下，虽然是可以进行hook，但是我这里会发现程序会崩溃，可能代码有瑕疵，自己猜想就是因为第二个参数为`java.lang.String`类型的，但是你接收的时候如果要包含`java.lang.String`类型就必须指明`java.lang.Object`，然后再进行强转`java.cast()`，但是在接收的时候`java.lang.Object`不一定总是`java.lang.String`，可能是一个`List`或者`Map`等，这时候强转为`java.lang.String`就会造成程序结束

![图片描述](upload/attach/202011/900086\_UVVX4GNPKVFMCHH.png)

这里可以直接对`m3385a`这个函数进行观察，流程就是将传入的数据先进行一个url编码，然后拼接`initFromXML.getAppKey()`作为参数，调用`EncryptUtil.encryptString`

![图片描述](upload/attach/202011/900086\_UWQMS4ZZAXQZYT8.png)

继续跟到`encryptString`中去，代码如下，那么也就需要进`libgavesec.so`中的`nativeEncrypt`函数进行分析

![图片描述](upload/attach/202011/900086\_ZXC55BGUAXHQGXJ.png)

这里遇到了一个问题，比如下面的两个函数和注入代码，一个函数会调用native中的函数，但是如果我hook `encryptString`这个函数，返回值写的是调用native层的`nativeEncrypt`函数，这样的写法就会导致程序结束，具体原因不知道

```
public static String encryptString(String str) {
return nativeEncrypt(str);
}
private static native String nativeEncrypt(String str);
Java.perform(function () {
var EncryptUtil = Java.use("com.tivicloud.utils.EncryptUtil");
EncryptUtil.encryptString.implementation = function (a) {
send("EncryptUtil.encryptString args[0]: " + a);
var result = this.nativeEncrypt(a);
return result;
}
});
```

因为上面那样写就导致程序结束，所以这里就直接hook `nativeEncrypt`，发现这样子就不会导致程序结束了

```
setImmediate(function(){
Java.perform(function () {
var EncryptUtil = Java.use("com.tivicloud.utils.EncryptUtil");
EncryptUtil.nativeEncrypt.implementation = function (a) {
send("EncryptUtil.nativeEncrypt args[0]: " + a);
var result = this.nativeEncrypt(a);
return result;
}
});
});
```

这里可以对`EncryptUtil.encryptString(encode + str)`，java层进行一次hook来获取对应的参数先

```
[*] EncryptUtil.encryptString args[0]: {"mobile\_operator":"","app\_versioncode":"4","app\_version":"2.1.0.22277","connection\_type":"WIFI","os\_version":"8.1.0","version\_code":"4","version\_name":"2.1.0.22277","os\_lang":"zh","sdk\_version":"3.1.4","package\_name":"com.lm.lm","imei":"359906070277673","os\_name":"Android","lang":"zh","udid":"10e1ef4509a2340eb0276bb99d186506","nudid":"155c2b70-fc97-4005-abc7-6d6c4329ed23","app\_id":"10054","channel\_id":"10006","tdid":"39fb282a761c17e80c069332fc6a2ffa9"}fff18b83431fa3a83b9de80c1e413bde
```

那么的`initFromXML.getAppKey`值就为：

```
fff18b83431fa3a83b9de80c1e413bde
```

然后接着继续跟native层进行分析，将`libgavesec.so`拖入到IDA中，并且找到对应的函数，该函数是静态注册的，所以特征很明显，如果是动态注册的话 可以通过`jni\_onload`中一些偏移变量来进行寻找对应的静态注册的函数

![图片描述](upload/attach/202011/900086\_XVH5P8KW4CHK59C.png)

主要进行了`encrypt`函数的调用

![图片描述](upload/attach/202011/900086\_GD635KE4E8P4GA9.png)

接着就是来到加密函数`encrypt`中进行分析

![图片描述](upload/attach/202011/900086\_UACC974HY4HSKSD.png)

先是进行一次`sha1`加密

![图片描述](upload/attach/202011/900086\_MAKEKKXXNEUV992.png)

所以这里要hook两个地方

`encrypt`：Base + 0x1C8C ，获取第一个参数

`SHA1::Result`：Base + 0x2AA4，获取该函数调用完之后的第二个参数的结果

hook的结果如下显示:

![图片描述](upload/attach/202011/900086\_BP3TF2TBPUBE57R.png)

接着打印了`sha1`加密过后的数据`v12`变量和最终的Sign值（这里也就是`nativeEncrypt`的返回值），但是你会发现结果是不一样的

![图片描述](upload/attach/202011/900086\_GAWRFV4JVXUA3MS.png)

接着继续往下面看，会发现存在循环的操作，所以`sha1`加密的结果应该被二次修改了，这里有两个大循环的操作

![图片描述](upload/attach/202011/900086\_U3A58JRYHJVKBZF.png)

首先第一段循环：

```
SHA1::Result((SHA1 *)&v13, (unsigned int *)v12);// sha1加密的结果
v4 = 0LL; // long long
v5 = 7;
do // 进行循环操作
{
v6 = *(DWORD *)&v12[v4]; // 0X0C
v7 = &dest_cstr_1[v5];
while ( 1 )
{
v8 = v6 & 0xF; // 0X0C
v6 >>= 4; // 0X00
*v7-- = hexDigits[v8]; //
if ( !(v5 & 7) )
break;
--v5;
}
*(DWORD *)&v12[v4] = v6; // 赋值操作
v4 += 4LL;
v5 += 15;
}
while ( v4 != 20 );
```

你会发现`hexDigits`是`data`段的一段数据：

![图片描述](upload/attach/202011/900086\_4GYRWYTC3WAUHFD.png)

最后分析，其实就是将其原封不动的转换为16进制的字符，然后一起拼接起来，分析注释如下

![图片描述](upload/attach/202011/900086\_ZJVQ5V9EYW847G4.png)

然后就是第二段循环了，循环次数也可以看出来会对每个16进制字符进行处理，循环次数为40次

![图片描述](upload/attach/202011/900086\_5XT6N3883K6UBEX.png)

hook了`char2hexInt`处理过后的数据会发现，`char2hexInt`这个函数才是二次处理的函数

![图片描述](upload/attach/202011/900086\_FZA9W2GWW8KV6PD.png)

```
do
{
v10 = char2hexInt(dest_cstr_1[v9]);
dest_cstr_1[v9] = hexDigits[(signed int)((unsigned __int64)char2hexInt(a211034f8af4e6b[v9]) ^ v10)]; //异或的值为v10
++v9;
}
```

还需要`char2hexInt`的参数来观察，该地址为0x1B34，HOOK结果如下

![图片描述](upload/attach/202011/900086\_8HP6Z8T66RZSZSH.png)

其实就是两个返回值进行异或处理，比如0x2^0x2就为0，最后还会通过`hexDigits`数组转成对应的16进制在内存中保存为0x30，因为字符 '0' 对应的ASCII 十六进制就是 0x30 是相等的！

![图片描述](upload/attach/202011/900086\_FK26ZQACDG8UB34.png)

最终的`sign`值分析的注释：

![图片描述](upload/attach/202011/900086\_N9FAWXTQYDDJ56K.png)

`sign`这里也分析完了，还有个整体加密的分析，发现整体会进行一次`Base64.encode`，但是`nativeAES`可以跟进去观察

![图片描述](upload/attach/202011/900086\_D48HNNS62F9MQBQ.png)

发现该函数依旧是native层的函数，所以这里继续去看分析

![图片描述](upload/attach/202011/900086\_2FSY8PESGQWJY7Z.png)

来到如下进行分析，看到名字就知道是`AES`算法加密了，这里的话用`findcrypt`插件通过特征码也可以进行识别

![图片描述](upload/attach/202011/900086\_E2YQXZ49JRHM67U.png)

![图片描述](upload/attach/202011/900086\_623GFWA8Z5GPERU.png)

然后接着就是先对传入的数据进行字符串复制

![图片描述](upload/attach/202011/900086\_M83QN8RQY57Y6NR.png)

然后就是`AES`对象和密钥的初始化，这里的`AES`需要hook，来确认`v19`的值，还有该函数最终的返回值`dest_array_bytes`的地址来进行打印

![图片描述](upload/attach/202011/900086\_NF5T624FU6MA3W9.png)

`AES::AES`：0x3274，`Cipher`：0x397C，然后hook到的数据如下，一个是`AES`密钥，最终要进行`AES`字符串加密的字符串

![图片描述](upload/attach/202011/900086\_2QX4E8GHBBXJ96S.png)

到这里就分析结束了

最终hook的代码：
