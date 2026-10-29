
```javascript
function find_hook_fun() {
    var fun_Name = "";
    var libart = Module.findBaseAddress('libart.so'); // 查找基地址
    var exports = Module.enumerateExportsSync("libart.so");
    for (var i = 0; i < exports.length; i++) {
        if (exports[i].name.indexOf("OpenMemory") !== -1) {
            fun_Name = exports[i].name;
            console.log("导出模块名: " + exports[i].name + "\t\t偏移地址: " + (exports[i].address - libart - 1));
            break;
        } else if (exports[i].name.indexOf("OpenCommon") !== -1) {
            fun_Name = exports[i].name;
            console.log("导出模块名: " + exports[i].name + "\t\t偏移地址: " + (exports[i].address - libart - 1));
            break;
        }
    }
    return fun_Name;
}

function DexFileVerifier(Verify) {
    var magic_03x = true;
    var magic_Hex = [0x64, 0x65, 0x78, 0x0a, 0x30, 0x33, 0x35, 0x00];
    for (var i = 0; i < 8; i++) {
        if (Memory.readU8(ptr(Verify).add(i)) !== magic_Hex[i]) {
            if (Memory.readU8(ptr(Verify).add(i)) === 0x37 || 0x38) {
                console.log('new dex');
            } else {
                magic_03x = false;
                break;
            }
        }
    }
    return magic_03x;
}

function dump_Dex(fun_Name, apk_Name) {
    if (fun_Name !== '') {
        var hook_fun = Module.findExportByName("libart.so", fun_Name);
        Interceptor.attach(hook_fun, {
            onEnter: function (args) {
                var begin = 0;
                var dex_flag = false;
                dex_flag = DexFileVerifier(args[0]);
                if (dex_flag === true) {
                    begin = args[0];
                }
                if (begin === 0) {
                    dex_flag = DexFileVerifier(args[1]);
                    if (dex_flag === true) {
                        begin = args[1];
                    }
                }
                if (dex_flag === true) {
                    console.log("magic : " + Memory.readUtf8String(begin));
                    var address = parseInt(begin, 16) + 0x20;
                    var dex_size = Memory.readInt(ptr(address));
                    console.log("dex_size :" + dex_size);
                    var dex_path = "/data/data/" + apk_Name + "/" + dex_size + ".dex";
                    var dex_file = new File(dex_path, "wb");
                    dex_file.write(Memory.readByteArray(begin, dex_size));
                    dex_file.flush();
                    dex_file.close();
                }
            },
            onLeave: function (retval) {}
        });
    } else {
        console.log("Error: no hook function.");
    }
}

var fun_Name = find_hook_fun();
var apk_Name = 'com.dodonew.online';
dump_Dex(fun_Name, apk_Name);
// frida -U -f com.dodonew.online -l dumpdex.js --no-pause
```

# 0x04 dex 解析

将脱壳后的 dex 推出：

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_V74K3GNQY7K4D8J.png)

其中第一个为加壳程序。

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_VQPPGMQMC6HMMPF.png)

第二个为 IjkMediaPlayer 和 rx 库，IjkMediaPlayer 是基于 FFmpeg 的 Android 多媒体播放器库，大佬们可自行百度了解。

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_ZPQR2QQRASES2Q5.png)

第三个为应用程序界面信息 dex。

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_WSVHHHMW75XDXKM.png)

第四个为应用程序逻辑代码。

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_9B9FPBREZ25MYSR.png)

既然是分析登录逻辑，那肯定是在第四个 dex 中分析啦！

# 0x05 协议分析

`jadx` 每次生成的参数名称会有所出入，各位在对照这份教程进行分析的时候只需把握整体步骤即可。

## 5.1 入手点定位

将第四个文件拖入 `jadx`，等待加载完成，搜索 `"Encrypt"`，结果还挺多：

![图片描述](https://bbs.kanxue.com/upload/attach/202401/945390_BGNJU2ANBK95EZH.png)

挺好定位，`com.dodonew.online.http.JsonRequest` 类中存在 `addRequestMap(Map<String, String>, int)` 方法和 `paraMap(Map<String, String>)` 方法，两方法中都有进行参数存放操作。

第一个方法 `addRequestMap` 翻译以下：添加请求的 Map，可疑，跟进去看看：

```java
public void addRequestMap(Map<String, String> map, int i) {
    String str = System.currentTimeMillis() + "";
    if (map == null) {
        map = new HashMap<>();
    }
    map.put("timeStamp", str);
    String encodeDesMap = RequestUtil.encodeDesMap(
        RequestUtil.paraMap(map, Config.BASE_APPEND, "sign"),
        this.desKey,
        this.desIV
    );
    JSONObject jSONObject = new JSONObject();
    try {
        jSONObject.put("Encrypt", encodeDesMap);
        this.mRequestBody = jSONObject + "";
    } catch (JSONException e) {
        e.printStackTrace();
    }
}
```

看这两句代码：

```java
String encodeDesMap = RequestUtil.encodeDesMap(
    RequestUtil.paraMap(map, Config.BASE_APPEND, "sign"),
    this.desKey,
    this.desIV
);

jSONObject.put("Encrypt", encodeDesMap);
```

第一句中生成的 `encodeDesMap` 就是 `Encrypt`，入口点定位无误。

## 5.2 MD5 算法分析

继续分析 `addRequestMap` 函数代码，看代码：

```java
String str = System.currentTimeMillis() + "";
map.put("timeStamp", str);
```

获取时间戳，然后将时间戳添加进 Map 中，再调用：

```java
RequestUtil.paraMap(map, Config.BASE_APPEND, "sign");
```

跟进 `RequestUtil.paraMap` 函数看看：

```java
public static String paraMap(Map<String, String> map, String str, String str2) {
    try {
        Set<String> keySet = map.keySet();
        StringBuilder sb = new StringBuilder();
        ArrayList arrayList = new ArrayList();
        for (String str3 : keySet) {
            arrayList.add(str3 + "=" + map.get(str3));
        }
        Collections.sort(arrayList);
        for (int i = 0; i < arrayList.size(); i++) {
            sb.append((String) arrayList.get(i));
            sb.append("&");
        }
        sb.append("key=" + str);
        map.put(str2, Utils.md5(sb.toString()).toUpperCase());
        String json = new GsonBuilder()
            .serializeNulls()
            .create()
            .toJson(sortMapByKey(map));
        Log.w(AppConfig.DEBUG_TAG, json + "   result");
        return json;
    } catch (Exception e) {
        e.printStackTrace();
        return "";
    }
}
```

首先将 Map 中的键提取出来存入 Set 中，再定义一个 List 集合用来存放键值信息，并进行 sort 排序。

其中有处：

```java
sb.append("key=" + str);
```

`str` 是入参参数二，向上跟一下是个固定值：

```java
public static final String BASE_APPEND = "sdlkjsdljf0j2fsjk";
```

经过一系列操作完后对值进行 MD5，MD5 得到的值就是 `sign` 的值，hook 看看哪些值需进行 MD5：

```javascript
function main() {
    Java.perform(function () {
        var Utils = Java.use("com.dodonew.online.util.Utils");
        Utils["md5"].implementation = function (string) {
            console.log('md5 is called' + ', ' + 'string: ' + string);
            var ret = this.md5(string);
            console.log('md5 ret value is ' + ret);
            return ret;
        };
    });
}

setImmediate(main);
```

hook 结果：

> md5 is called, string: equtype=ANDROID&loginImei=Androidc0b30f35fc9535b5&timeStamp=1687772161410&userPwd=12334&username=123456789&key=sdlkjsdljf0j2fsjk
>
> md5 ret value is e888bef28d91b42fc10cf91540ec057b

试着用 Python 还原一下，看看是不是标准 MD5 算法：

```python
from hashlib import md5

def get_encode_mes(mes):
    new_md5 = md5()
    new_md5.update(mes.encode(encoding='utf-8'))
    return new_md5.hexdigest()

if __name__ == '__main__':
    print(get_encode_mes('equtype=ANDROID&loginImei=Androidc0b30f35fc9535b5&timeStamp=1687772161410&userPwd=12334&username=123456789&key=sdlkjsdljf0j2fsjk'))
```

结果：`e888bef28d91b42fc10cf91540ec057b`，对照一致，标准 MD5 算法。

## 5.3 DES 加密算法分析

继续分析 `addRequestMap` 函数代码，看代码：

```java
String encodeDesMap = RequestUtil.encodeDesMap(
    RequestUtil.paraMap(map, Config.BASE_APPEND, "sign"),
    this.desKey,
    this.desIV
);
```

其中 `this.desKey`、`this.desIV`，猜测为 DES 算法，先 hook 看看数据，hook 代码：

