
# [原创] 基于 `frida` 框架的脱壳工具 `frida-unpack` 的使用心得

## 一、引言

第一次发帖，不周之处，还请原谅与指正。

众所周知，`frida` 是一款用于 `hook` 的优秀工具，其实就是 `python` 下的一个包。由于 `frida` 本身只是一个注入方式，所以如果要对目标对象——移动设备进行注入操作，就一定要在对应的终端上**安装相应的**用于监听注入的服务器，也就是 `frida-server`，当然，**两方的版本也要保持一致**。然后，对安卓 `hook` 来说，其对应着一款可以破壳的工具，即 `frida-unpack`。

`frida-unpack` 的详情：[b5eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7%4c8E0j5i4c8Z5i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8#2L8Y4m8S2j5$3D9`](https://bbs.kanxue.com/elink@c53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7%4c8E0j5i4c8Z5i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8#2L8Y4m8S2j5$3D9`）。

利用它，能对一些 `app` 壳进行去除，达到取到源码的目的。

## 二、使用方法

1. 搭建 `frida` 环境。由于 `frida-unpack` 是基于 `frida` 的，所以前提必须具备 `frida` 环境。电脑端：安装 `frida` 包和工具，这里使用 `python3.7`，装 `frida` 包和工具：

   ```bash
   pip install frida
   pip install frida-tools
   ```

   装好后我的是对应 `frida` 版本是 `12.8.20`。移动设备端：安装 `frida-server` 服务器，装 `frida-server` 要找到移动设备对应的架构下载，这里使用模拟器 `x86-android7`，下好后解压 `push` 到设备的 `/data/local/tmp` 下。

2. 启动 `frida-server` 服务器，到 `/data/local/tmp` 下执行 `./frida-server` 启动服务器，**注意如果出现 `warnning` 可以尝试更换安卓版本（否则可能会出现 `time out`）**。

3. 检查设备连接，执行 `adb devices` 可查看连接设备。

4. 端口转发，执行：

   ```bash
   adb forward tcp:27042 tcp:27042
   adb forward tcp:27043 tcp:27043
   ```

5. 找到对应移动设备的 `OpenMemory` 的导出名称，可以通过将移动设备下 `/system/lib/libart.so` `pull` 到电脑，然后用 `IDA` 打开到 `Export` 那里查找，形如：

   ```text
   _ZN3art7DexFile10OpenMemoryEPKhjRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPNS_6MemMapEPKNS_10OatDexFileEPS9_
   ```

6. 找到 `app` 包名，可以通过反编译等方式找到。

7. 执行 `frida-unpack` 脚本，如介绍所示，有 `js` 版本和 `python` 版本，这里使用 `python` 版本：

   ```python
   #-*- coding:utf-8 -*-
   # coding=utf-8

   import frida
   import sys

   def on_message(message, data):
       base = message['payload']['base']
       size = int(message['payload']['size'])
       print hex(base), size

       # print session
       # dex_bytes = session.read_bytes(base, size)
       # f = open("1.dex", "wb")
       # f.write(dex_bytes)
       # f.close()

       # 9.0 arm 需要拦截 _ZN3art13DexFileLoader10OpenCommonEPKhjS2_jRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPKNS_10OatDexFileEbbPS9_NS3_10unique_ptrINS_16DexFileContainerENS3_14default_deleteISH_EEEEPNS0_12VerifyResultE
       # 7.0 arm：_ZN3art7DexFile10OpenMemoryEPKhjRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPNS_6MemMapEPKNS_10OatDexFileEPS9_

   package = sys.argv[1]
   print "dex 导出目录为: /data/data/%s" % (package)
   device = frida.get_usb_device()
   pid = device.spawn(package)
   session = device.attach(pid)

   src = """
   Interceptor.attach(Module.findExportByName("libart.so", "_ZN3art13DexFileLoader10OpenCommonEPKhjS2_jRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPKNS_10OatDexFileEbbPS9_NS3_10unique_ptrINS_16DexFileContainerENS3_14default_deleteISH_EEEEPNS0_12VerifyResultE"), {
       onEnter: function (args) {
           var begin = args[1]

           console.log("magic : " + Memory.readUtf8String(begin))

           var address = parseInt(begin, 16) + 0x20
           var dex_size = Memory.readInt(ptr(address))
           console.log("dex_size :" + dex_size)

           var file = new File("/data/data/%s/" + dex_size + ".dex", "wb")
           file.write(Memory.readByteArray(begin, dex_size))
           file.flush()
           file.close()

           var send_data = {}
           send_data.base = parseInt(begin, 16)
           send_data.size = dex_size
           send(send_data)
       },

       onLeave: function (retval) {
           if (retval.toInt32() > 0) {
           }
       }
   });
   """ % (package)

   script = session.create_script(src)
   script.on("message", on_message)
   script.load()
   device.resume(pid)
   sys.stdin.read()
   ```

把里面的 `package` 和导出名称替换执行即可。

注意，**这里如果是模拟器要把 `get_usb_device()` 改为 `get_remote_device()`（否则可能会出现 `time out` 和 `device not found`）**，效果：

![图片描述](https://bbs.kanxue.com/upload/attach/202005/894007_XQ9GU3S9FUQYT8F.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202005/894007_3ZPZUK53BKQTGBM.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
