
```
'use strict';

/**

 * 此脚本在以下环境测试通过

 * android os: 7.1.2 32bit  (64位可能要改OpenMemory的签名)

 * legu: libshella-2.8.so

 * 360:libjiagu.so

 */

Interceptor.attach(Module.findExportByName("libart.so", "_ZN3art7DexFile10OpenMemoryEPKhjRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPNS_6MemMapEPKNS_10OatDexFileEPS9_"), {

    onEnter: function (args) {



        //dex起始位置

        var begin = args[1]

        //打印magic

        console.log("magic : " + Memory.readUtf8String(begin))

        //dex fileSize 地址

        var address = parseInt(begin,16) + 0x20

        //dex 大小

        var dex_size = Memory.readInt(ptr(address))



        console.log("dex_size :" + dex_size)

        //dump dex 到/data/data/pkg/目录下

        var file = new File("/data/data/xxx.xxx.xxx/" + dex_size + ".dex", "wb")

        file.write(Memory.readByteArray(begin, dex_size))

        file.flush()

        file.close()

    },

    onLeave: function (retval) {

        if (retval.toInt32() > 0) {

            /* do something */

        }

    }

});

```

## 3.2 `frida-unpack` 使用报错及解决方案

### 3.2.1 使用

`firda-unpack` 的使用方法在项目的 `README.md` 中，其中查看 `OpenMemory` 的导出名称，我们在 `/system/lib` 中找到 `libart.so` 后，还可以拖进 IDA 然后在 Exports 窗口搜索到后点击查看。

![](https://bbs.kanxue.com/upload/attach/202004/802108_MMQXRCNCNA4HQZU.png)

![](https://bbs.kanxue.com/upload/attach/202004/802108_4CDMVXFX6QY7Y4M.png)

虽然怎么操作在项目中的 `README.md` 写得十分简单易懂，如果上手就能直接脱壳最好，但是我在使用的时候还是出现了错误。

这里详细说下我的碰到的报错和解决方案。

### 3.2.2 `KeyError` 报错

首先我直接运行出现的错误是 `KeyError: 'payload'`，查看错误的代码行。

![](https://bbs.kanxue.com/upload/attach/202004/802108_MFTUH8BRATEZW36.png)

![](https://bbs.kanxue.com/upload/attach/202004/802108_7ZG4HSUZZDT22VC.png)

报错提示没有 `'payload'` 这个 key，我们加上一句 `print(mesaage)` 把 `message` 值打印出来，发现 `message` 是一个字典，`{'type': 'error', 'description': "TypeError: cannot read property 'readU8' of null", 'stack': "TypeError: cannot read property 'readU8' of null\n at [anon] (../../../frida-gum/bindings/gumjs/duktape.c:56648)\n at frida/runtime/core.js:386\n at /script1.js:29", 'fileName': '/_frida.js', 'lineNumber': 1480, 'columnNumber': 1}`，里面确实没有 `'payload'` 这个 key，并且可以看到类型是 `'error'`。

我们重新写一个 `on_message` 即可。

```
def on_message(message, data):

    if message['type'] == 'send':

        base = message['payload']['base']

        size = int(message['payload']['size'])

        print(hex(base), size)

    elif message['type'] == 'error':

        for i in message:

            if i == "type":

                print("[*] %s" % "error:")

                continue

            if type(message[i]) is str:

                print("[*] %s" %

                      i + ":\n    {0}".format(message[i].replace('\t', '    ')))

            else:

                print("[*] %s" %

                      i + ":\n    {0}".format(message[i]))

    else:

        print(message)

```

### 3.2.3 `readU8` 报错

运行发现这个报错的问题解决了，但是又出现了新的问题。

![](https://bbs.kanxue.com/upload/attach/202004/802108_AJA8VNZR5HNR3JZ.png)

不清楚是为什么，而且报错行数是 js 代码的最后一行，对报错进行搜索在 <https://bbs.pediy.com/thread-250815.htm> 这个帖子里面看到一个大佬说

用 `Module.getExportByName` 替换 `Module.findExportByName` 就会得到具体的报错原因了，尝试在 js 代码中替换发现果然有更具体的报错原因了，感谢大佬。

![](https://bbs.kanxue.com/upload/attach/202004/802108_Y367KKWEWA45PKZ.png)

可以看到错误是因为在 `libart.so` 中不能通过 `OpenMemory` 的导出函数名找到它，这个十分奇怪，我们再写个 frida 脚本把内存中 `liart.so` 的导出函数名和地址都打印出来看看有没有 `OpenMemory` 的。

```
import frida

import sys



jscode = """



  var exports = Module.enumerateExportsSync("libart.so");

    for(var i=0;i<exports.length;i++){

        send("name:"+exports[i].name+"  address:"+exports[i].address);

     }



"""



def on_message(message, data):

    if message['type'] == 'send':

        print("[*] {0}".format(message['payload']))



process = frida.get_usb_device().attach("com.cz.babySister")  

script = process.create_script(jscode)  

script.on('message', on_message)  

script.load()  

sys.stdin.read()
```

我们发现是有的，这就十分让人迷惑了，只能想想别的办法了解决。

![](https://bbs.kanxue.com/upload/attach/202004/802108_6ZPKX6RSBHER525.png)

### 3.2.4 修改代码

我们可以看到是由 `OpenMemory` 函数的地址的，因此想到尝试下直接 hook 这个地址会怎么样，需要先使用 `new NativePointer` 转换一下，试了是可行的，十分感动。

这里直接放出修改后的 `OpenMemory.js` 代码。

```
src = """

var exports = Module.enumerateExportsSync("libart.so");

    for(var i=0;i<exports.length;i++){

        if(exports[i].name == "_ZN3art7DexFile10OpenMemoryEPKhjRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPNS_6MemMapEPKNS_10OatDexFileEPS9_"){

            var openMemory = new NativePointer(exports[i].address);

            }

     }



Interceptor.attach(openMemory, {

    onEnter: function (args) {



        var begin = args[1]



        console.log("magic : " + Memory.readUtf8String(begin))



        var address = parseInt(begin,16) + 0x20



        var dex_size = Memory.readInt(ptr(address))



        console.log("dex_size :" + dex_size)



        var file = new File("/data/data/%s/" + dex_size + ".dex", "wb")

        file.write(Memory.readByteArray(begin, dex_size))

        file.flush()

        file.close()



        var send_data = {}

        send_data.base = parseInt(begin,16)

        send_data.size = dex_size

        send(send_data)

    },

    onLeave: function (retval) {

        if (retval.toInt32() > 0) {

        }

    }

});

"""%(package)
```

## 3.3 顺利脱壳

我们重新运行，非常激动地发现 `frida-unpack` 运行成功了。

![](https://bbs.kanxue.com/upload/attach/202004/802108_2KSQ9K2JFTCQANH.png)

去对应的文件目录查看，可以看到被脱掉后的两个 dex 文件。

![](https://bbs.kanxue.com/upload/attach/202004/802108_5F5FGWJGHSFHGKU.png)

都拖进 JEB 中看下，发现 APP 的逻辑代码都在 `5619552.dex` 中了，至此第一步脱壳完成。

# 4. Hook 进行破解

有了反编译的代码后相信各位破解积分和 vip 充值是手到擒来了，不然都不好意思在看雪吱声了，这里只简单说下我一般的处理思路和方法，如果不行就要具体情况具体分析了。

我通常都会先在有使用到用户数据的 activity 中查看下代码，看有没有导入 `UserInfo` 这种类，然后如果有的话直接找到这个类，积分、vip、到期时间等等这些一般都会有相应的 `getXxx()` 和 `setXxx()` 方法，然后就可以随意 hook 了。

## 4.1 积分

![](https://bbs.kanxue.com/upload/attach/202004/802108_N24GWG795G3MGG4.png)

直接上相应的 js 代码，逻辑就是 hook 住 `UserInfo` 类中 `getJifen()` 方法，然后随便设置返回积分。

```
var userinfo = Java.use("com.cz.babySister.javabean.UserInfo");

userinfo.getJifen.implementation  = function(){    

    return "100000";

}
```

![](https://bbs.kanxue.com/upload/attach/202004/802108_X6WSDCJCSYBRAY6.png)

值得一提的是，hook 后在测试过程中发现本地修改的数据会上传到服务器，因为发现换一个模拟器登录账号积分还是那么多，这个在后面的模拟抓包中会具体看下怎么回事。

## 4.2 vip

破解 vip 时候发现尝试 hook `getIsvip()` 和 `setIsvip()` 没有效果，猜测应该是内购之后向服务器进行请求成为 vip 用户，然后服务器返回自己账号 vip 用户数据，我们现在换下思路破解 vip，就是经常看到的破解内购。

![](https://bbs.kanxue.com/upload/attach/202004/802108_44WJDQ2BZTJ8BKS.png)

破解内购应该也都是看雪用户的起手水平了，这个分析过程也不多说了，直接上 js 代码。

```
var pay = Java.use("com.cz.babySister.alipay.q");pay.b.implementation = function(){    return "9000"}
```

逻辑就是把支付时候下面的代码的支付失败流程变成支付成功流程，然后就会向服务器发送购买类型的请求。

![](https://bbs.kanxue.com/upload/attach/202004/802108_R6G9XNXC28VYTWA.png)

这样 hook 之后购买积分也是可以的。

![](https://bbs.kanxue.com/upload/attach/202004/802108_HNFN4245WAS2FCV.png)

## 4.3 `android_id`

`android_id` 是在设备首次启动时，系统随机生成的一个64位的数字，并把这个数字以16进制字符串的形式保存下来，这个16进制的字符串就是 `android_id`，当设备被 wipe 后该值会被重置。

但是为我们什么要 hook 这个呢，是因为因为我测试了两个账号都被封了。。没错，就是被封了，而且被封后我发现重新注册的号就不能看那些不可描述的视频和直播了，但是换一个模拟器登录新注册的账号就又可以看了。

因为使用的模拟器配置了 frida 和 Charles 环境及安装了别的分析工具，不想再换个模拟器重新配置了，就找了下为什么不能看了。在写出模拟抓包代码打印出注册账号的请求后，发现了原因，注册请求上传的一个参数是 `memi1`，找到对应 Java 代码赋值处溯源了下发现是获取的是 `android_id`，当账号被封后带有这个用户注册所用安卓机的 `android_id` 的请求都不会被处理，这也就是为什么重新注册账号后不能再看不可描述的东西了。

![](https://bbs.kanxue.com/upload/attach/202004/802108_38V4TQUNGYWRUWU.png)

这个时候 hook 了一下返回 `android_id` 字符串的函数，随便改一下，然后这个模拟器就能重新用了。

直接上 js 代码，要提一下的就是，`Secure` 类是 `android.provider.Settings` 的一个内部类，我们要 hook 的 `getString()` 方法在 `Secure` 类中，hook 类时候写成 `Java.use("android.provider.Settings$Secure"` 就不会报错。

```
var sec = Java.use("android.provider.Settings$Secure");

sec.getString.implementation = function(arg1,arg2){

    return "5c80b60fc1f73307";

}
```

运行之后，模拟器登录后就又可以重新看了。

# 5 模拟抓包

## 5.1 Charles 抓不到包的原因

在开始 APP 分析时候我们提到使用 Charles 抓不到包，设置了 SSL 代理也没有用，当我们反编译分析脱壳后的 dex 找到构造请求的地方时，发现了真相，和 SSL 没有关系，是因 `openConnection(Proxy.NO_PROXY)`
