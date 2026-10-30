
小米的前端也是用的`Lua`编写的，但是其中的`Lua`文件不是源码，而是编译后的二进制文件，所以我们需要对其进行反编译。目前，对`Lua`反编译的常用工具有[unluac](https://bbs.kanxue.com/elink@251K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6t1j5h3&6K6g2$3g2K6M7$3g2D9M7#2)9J5c8Y4g2F1L8s2g2S2j5H3`.`.)和[luadec](https://bbs.kanxue.com/elink@ffbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9i4u0#2M7$3y4S2L8i4m8Q4x3V1k6D9N6h3q4V1k6h3x3`.)。但是小米对`Lua`的解释器做了魔改，就不能直接用这两个工具进行反编译了，所幸已有师傅对此做了研究，并给出了专门针对小米固件的反编译工具[unluac_miwifi](https://bbs.kanxue.com/elink@76dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5P5h3q4y4K9i4y4@1P5g2)9J5c8Y4g2F1L8s2g2S2j5#2)9#2k6X3#2A6N6$3W2X3K9b7`.`.)和[luadec_miwifi](https://bbs.kanxue.com/elink@129K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5P5h3q4y4K9i4y4@1P5g2)9J5c8X3I4#2j5h3c8W2j5#2)9#2k6X3#2A6N6$3W2X3K9b7`.`.)。至于如何对被魔改的解释器或编译器所编译出来的`Lua`字节码进行逆向，网上也有不少文章，这里不再展开。

我这里用的是`unluac_miwifi`，最终可以编译出一个`unluac.jar`，但一次只能对一个`Lua`文件进行反编译，所以我们需要写一个批量处理的简单脚本：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `import` `os` `res` `=` `os.popen(`"`find ./ -name *.lua"``).readlines()` `for` `i` `in` `range`( `0`, `len`(res)) : `path` `=` `res[i].strip(`"\n"`) `cmd` `=` `"java -jar /home/winmt/unluac_miwifi/build/unluac.jar "` `+` `path` `+` `" > "` `+` `path` `+` `".dis"` `print`(cmd) `os.system(cmd)` |

小米`AX9000`路由器固件是`AArch64el`架构的，由于网上似乎没有公开的`AArch64`的内核与文件系统，系统级仿真可参考下面这篇文章的步骤`extract`出来`vmlinuz`和`initrd.img`：[a58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2V1K9h3!0\*7k6i4u0G2i4K6u0W2j5$3!0E0i4K6u0r3j5X3!0S2M7X3c8K6i4K6u0r3M7h3g2E0N6h3q4S2M7X3y4Z5y4U0c8Q4y4h3k6T1N6h3I4D9M7$3g2&6k6g2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@371K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2V1K9h3!0*7k6i4u0G2i4K6u0W2j5$3!0E0i4K6u0r3j5X3!0S2M7X3c8K6i4K6u0r3M7h3g2E0N6h3q4S2M7X3y4Z5y4U0c8Q4y4h3k6T1N6h3I4D9M7$3g2&6k6g2)9J5k6h3S2@1L8h3H3`.)

此外，小米`AX9000`的固件中采用了`Apache Thrift`的框架，使用`C++`编写的版本，相关源码可见：[228K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7r3q4U0K9r3g2Q4x3V1k6@1K9s2u0A6k6Y4c8Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3L8r3W2T1i4K6u0r3j5%4m8H3i4K6u0r3M7%4u0U0i4K6u0r3N6r3S2J5K9h3k6@1](https://bbs.kanxue.com/elink@236K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7r3q4U0K9r3g2Q4x3V1k6@1K9s2u0A6k6Y4c8Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3L8r3W2T1i4K6u0r3j5%4m8H3i4K6u0r3M7%4u0U0i4K6u0r3N6r3S2J5K9h3k6@1) ，也可参考网络上其他资料，初步认识后对接下来的逆向分析可能会有一些帮助。

## 漏洞细节

此部分只对该漏洞调用链做大致的分析，感兴趣的师傅可继续深入分析相关细节。

在反编译的`/usr/lib/lua/luci/controller/api/xqdatacenter.lua`中，可以看到 URL `/api/xqdatacenter/request` 相关的`handler`函数是`tunnelRequest`函数，且是需要鉴权通过的。

关于鉴权，这里多说两句。首先，对于`/api/xqdatacenter`这个节点来说，设置`sysauth = "admin"`即确保只有`admin`账户可以访问这个`node`（小米路由器后台的默认账户就是`admin`），设置`sysauth_authenticator = "jsonauth"`即当`token`不存在或错误时，通过`authenticator.jsonauth`函数进行登录验证。对于具体的入口来说，定义`entry{}`内的第五个参数`flag`位为`0x01`（或`&0x01=1`）代表不需要鉴权，这里`/api/xqdatacenter/request`这个入口没有设置`flag`位，因此需要鉴权。`flag`位有多种`option`可选，通过按位与的结果确定所限制的不同权限，感兴趣的师傅可自行分析。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34 | `function L0()` `local L0, L1, L2, L3, L4, L5, L6` `L0` `=` `node` `L1` `=` `"api"` `L2` `=` `"xqdatacenter"` `L0` `=` `L0(L1, L2)` `L1` `=` `firstchild` `L1` `=` `L1()` `L0.target` `=` `L1` `L0.title` `=` `""` `L0.order` `=` `300` `L0.sysauth` `=` `"admin"` `L0.sysauth_authenticator` `=` `"jsonauth"` `L0.index` `=` `true` `...` `L1` `=` `entry` `L2` `=` `{}` `L3` `=` `"api"` `L4` `=` `"xqdatacenter"` `L5` `=` `"request"` `L2[``1``]` `=` `L3` `L2[``2``]` `=` `L4` `L2[``3``]` `=` `L5` `L3` `=` `call` `L4` `=` `"tunnelRequest"` `L3` `=` `L3(L4)` `L4` `=` `_` `L5` `=` `""` `L4` `=` `L4(L5)` `L5` `=` `301` `L1(L2, L3, L4, L5)` `...` `end` `index` `=` `L0` |

在函数`tunnelRequest`中，会对传入`payload`字段内的`JSON`数据用`binaryBase64Enc`函数进行`Base64`编码处理，然后拼接入`THRIFT_TUNNEL_TO_DATACENTER`所指代的命令中并执行。

关键在于此处用的是 **`formvalue_unsafe`函数** 获取`payload`字段内容，未过滤危险字符，而`formvalue`函数中是有用`hackCheck`过滤危险字符的。这里可能是开发者考虑到`Json`格式的数据当中可能会用到某些字符所以不能直接过滤，但也没有进一步去做针对于`Json`的危险字符过滤，给了我们可趁之机。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28 | `function L5()` `local L0, L1, L2, L3, L4, L5, L6, L7, L8` `L0` `=` `require` `L1` `=` `"xiaoqiang.util.XQCryptoUtil"` `L0` `=` `L0(L1)` `L1` `=` `L0.binaryBase64Enc` `L2` `=` `_UPVALUE0_` `L2` `=` `L2.formvalue_unsafe` `L3` `=` `"payload"` `L2, L3, L4, L5, L6, L7, L8` `=` `L2(L3)` `L1` `=` `L1(L2, L3, L4, L5, L6, L7, L8)` `L2` `=` `_UPVALUE1_` `L2` `=` `L2.THRIFT_TUNNEL_TO_DATACENTER` `L2` `=` `L2` `%` `L1` `L3` `=` `require` `L4` `=` `"luci.util"` `L3` `=` `L3(L4)` `L4` `=` `_UPVALUE0_` `L4` `=` `L4.write` `L5` `=` `L3.exec` `L6` `=` `L2` `L5` `=` `L5(L6)` `L6` `=` `nil` `L7` `=` `false` `L8` `=` `true` `L4(L5, L6, L7, L8)` `end` `tunnelRequest` `=` `L5` |

在`/usr/lib/lua/xiaoqiang/common/XQConfigs.lua`中，可以找到`THRIFT_TUNNEL_TO_DATACENTER`的相关定义：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `L0` `=` `"thrifttunnel 0 '%s'"` `THRIFT_TUNNEL_TO_DATACENTER` `=` `L0` `L0` `=` `"thrifttunnel 1 '%s'"` `THRIFT_TUNNEL_TO_SMARTHOME` `=` `L0` `L0` `=` `"thrifttunnel 2 '%s'"` `THRIFT_TUNNEL_TO_SMARTHOME_CONTROLLER` `=` `L0` `L0` `=` `"thrifttunnel 3 ''"` `THRIFT_TO_MQTT_IDENTIFY_DEVICE` `=` `L0` `L0` `=` `"thrifttunnel 4 ''"` `THRIFT_TO_MQTT_GET_SN` `=` `L0` `L0` `=` `"thrifttunnel 5 ''"` `THRIFT_TO_MQTT_GET_DEVICEID` `=` `L0` `L0` `=` `"thrifttunnel 6 '%s'"` `THRIFT_TUNNEL_TO_MIIO` `=` `L0` `L0` `=` `"thrifttunnel 7 '%s'"` `THRIFT_TUNNEL_TO_YEELINK` `=` `L0` `L0` `=` `"thrifttunnel 8 '%s'"` `THRIFT_TUNNEL_TO_CACHECENTER` `=` `L0` |

可以看到，`THRIFT_TUNNEL_TO_DATACENTER`所指代的命令为`thrifttunnel 0 '%s'`。因此，最终所执行的完整命令是`thrifttunnel 0 'base64编码的payload字段'`，即`payload`字段中被`Base64`编码后的`Json`数据会被传入`thrifttunnel`程序中，且`option`为`0`。

在`/usr/sbin/thriftunnel`二进制文件中，`*(a2 + 16)`是传入的第二个参数，即`Base64`编码后的`payload`字段内的`Json`数据，其作为第一个参数被传入`sub_1B9B0`函数中，而`sub_1B9B0`函数的第二个参数`v11`此时是空串。

![](https://bbs.kanxue.com/upload/attach/202405/949925_DS43TCNHQPJ48VY.webp)

进入`sub_1B9B0`函数后，可以发现首先将与`a1`（`Base64`编码的`payload`字段）相关的数据作为参数传入了`sub_1F1F8`函数处理，并最终将其返回结果通过`string::assign()`赋值给了`a2`（即上一级的`v11`变量）。

![](https://bbs.kanxue.com/upload/attach/202405/949925_QNK6G33C5YTVPV9.webp)

`sub_1F1F8`函数看上去是做了一些编码转换的操作，可以猜测到这里就是做了`Base64`的解码工作。我们很容易根据其中抛出的异常信息确认我们的猜测，这里的确就是将`payload`字段内的`Json`数据进行了`Base64`解码。

![](https://bbs.kanxue.com/upload/attach/202405/949925_JK5MJWGKYTS5QKJ.webp)

我们再返回到主函数，进而当`*(a2 + 8)`即传入的第一个参数`option`为`0`时，会执行到`sub_1BAE0`函数，根据上文分析，其参数`v11`就是解码后的`Json`字符串。

![](https://bbs.kanxue.com/upload/attach/202405/949925_6Q48UGREBN9QN4V.webp)

在`sub_1BAE0`函数中，创建了`socket`，结合传入的参数（上级的`v11`变量）是`Json`字符串，很容易判断出此处会将`payload`字段的`Json`数据发送给本地`127.0.0.1`的`9090`端口（这里保护了端口的安全性，没有对外开放，我们想要找到未授权口而悬着的心也终于死了）。

![](https://bbs.kanxue.com/upload/attach/202405/949925_4Y7C5GMADEY26G4.webp)

`/usr/sbin/datacenter`程序一直挂在进程中，监听着`9090`端口，故我们的数据被传到了`datacenter`程序进一步处理。

![](https://bbs.kanxue.com/upload/attach/202405/949925_6UMRQ8MR6FF5KVH.webp)

在`datacenter`的`constructAPIMappingTable()`函数里分别执行了三个类的`sConstructMappingTable()`函数。

![](https://bbs.kanxue.com/upload/attach/202405/949925_6NUA3B4KHQ76U2S.webp)

其中，都是通过`STL map`建立起了`api`编号（下文解释）和对应的处理函数`handler`间的映射关系。具体来看，有一些`api`是直接在`datacenter`中被处理的，有些是被进一步转发到了`/usr/sbin/indexservice`（`9088`端口）处理，另外一些则是被转发到了`/usr/sbin/plugincenter`（`9091`端口）中进一步处理。

我们在这里直接定位到该漏洞对应的`api`，在`datacenter::PluginApiCollection::sConstructMappingTable`中，当`api`为`629`的时候，对应的`handler`是`callPluginCenter`，其实从函数名就能看出来作用了，就是转发给`plugincenter`。

![](https://bbs.kanxue.com/upload/attach/202405/949925_HCNUEBRU5VBKTAB.webp)
