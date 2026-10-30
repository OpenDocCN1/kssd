
这里不再进入 `sub_1B9B0` 函数继续讲解，它的逻辑较为简单，结合 ai 分析可以知道它的功能大致就是 `解开 base64 编码`，然后将结果存放在 `v12` 中

## 第四步

接下来是个 `switch`

显然我们传入的 `argv[1] = 0`，所以我们会进入 `case 0` 中

|  |  |
| --- | --- |
| 1  2 | `case` `0:`  `v3 = sub_1BAE0(v12[0]);` |

也就是随后会执行 `sub_1BAE0` 函数

而这里的 `v12[0]` 按照我们的分析，它应该是解码后的字符串，也就是我们用户输入的命令

在 `sub_1BAE0` 函数中，创建了 `socket`，传入的参数是 `Json` 字符串，很容易判断出此处会将 `payload` 字段的 `Json` 数据发送给本地 `127.0.0.1` 的 `9090` 端口

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_WMSEJKXQ6RAPVRB.webp)

`/usr/sbin/datacenter` 程序一直挂在进程中，监听着 `9090` 端口，故我们的数据被传到了 `datacenter` 程序进一步处理（这里笔者有一个疑问，不知道 winmt 师傅是如何发现这个 `datacenter` 程序监听 9090 端口，难不成是把 `sbin` 文件夹下所有文件都分析了一遍吗？）

从下图图示部分可以看到 `datacenter` 程序的 main 函数中创建了非阻塞服务器实例，监听 9090 端口

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_HEJPBP6C7Y3RKRW.webp)

下面进入到 `datacenter` 的 `constructAPIMappingTable()` 函数

在里面可以看到分别执行了三个不同类的 `sConstructMappingTable()` 函数

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_5UE2P5UAGRZU8EA.webp)

这三个函数都是 API 路由映射表构建函数，用于建立 API 编号 → 处理函数的映射关系，实现请求的路由分发

有一些 `api` 是直接在 `datacenter` 中被处理的，有些是被进一步转发到了 `/usr/sbin/indexservice`（`9088` 端口）处理，另外一些则是被转发到了 `/usr/sbin/plugincenter`（`9091` 端口）中进一步处理。

下面我们之间定位到漏洞所在的 api，在 `datacenter::PluginApiCollection::sConstructMappingTable` 中，当 `api` 为 `629` 的时候，对应的 `handler` 是 `callPluginCenter`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_8RH2BTJXGBFVYTK.webp)

进入 `callPluginCenter` 看一下它的功能

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `__int64` `__fastcall callPluginCenter(` `__int64` `a1,` `__int64` `a2)`  `{`  `char` `*v3;` `// x0`  `_BYTE v5[32];` `// [xsp+38h] [xbp+38h] BYREF`  `_QWORD v6[5];` `// [xsp+58h] [xbp+58h] BYREF`    `v3 = json_object_to_json_string(a1, &_stack_chk_guard, 0LL);`  `sub_B48B0(v5, v3);`  `ThriftClient::sCallPluginCenter(v6, v5);`  `std::string::_M_assign(a2, v6);`  `std::string::_M_dispose(v6);`  `std::string::_M_dispose(v5);`  `return` `v6[4] ^ _stack_chk_guard;`  `}` |

分析可知它的主要功能是接收 JSON 格式的请求数据，通过 `Thrift 协议` 转发到 `plugincenter 服务`，获取服务端响应并返回

进入其中的 `sCallPluginCenter` 函数可以看出它将数据转发给了本地的 9091 端口

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_W3Q7YFWA92585FV.webp)

下面来看 `DataCenterHandler::request` 函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29 | `__int64` `__fastcall DataCenterHandler::request(` `__int64` `a1,` `__int64` `a2,` `__int64` `a3)`  `{`  `__int64` `v5;` `// x0`  `__int64` `v6;` `// x2`  `__int64` `v7;` `// x0`  `_QWORD v9[6];` `// [xsp+58h] [xbp+58h] BYREF`  `_QWORD v10[5];` `// [xsp+88h] [xbp+88h] BYREF`    `APIMapping::APIMapping(v9);`  `APIMapping::redirectRequest(v10, v9, a3);`  `std::string::_M_assign(a2, v10);`  `std::string::_M_dispose(v10);`  `std::_Rb_tree<` `int`,std::pair<` `int` `const`, `void` `(*)(json_object *,std::string &)> ,std::_Select1st<std::pair<` `int` `const`, `void` `(*)(json_object *,std::string &)> >,std::less<` `int` `>,std::allocator<std::pair<` `int` `const`, `void` `(*)(json_object *,std::string &)> > >::_M_erase(`  `v9,`  `v9[2]);`  `google::LogMessage::LogMessage(`  `v9,`  `"src/thriftwrapper/DataCenterHandler.hpp"`, `40LL,`  `1LL,`  `0LL,`  `&google::LogMessage::SendToSyslogAndLog,`  `0LL);`  `v5 = google::LogMessage::stream(v9);`  `v7 = std::operator<<<std::char_traits<` `char` `>>(v5,` `"output is: "`, v6);`  `std::operator<<<` `char` `>(v7, a2);`  `google::LogMessage::~LogMessage(v9);`  `return` `v10[4] ^ _stack_chk_guard;`  `}` |

其中调用了 `APIMapping::APIMapping` 而这个函数里面就调用了我们之前分析的 `constructAPIMappingTable` 函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `void` `__fastcall APIMapping::APIMapping(APIMapping *` `this)`  `{`  `*(` `this` `+ 2) = 0;`  `*(` `this` `+ 2) = 0LL;`  `*(` `this` `+ 3) =` `this` `+ 8;`  `*(` `this` `+ 4) =` `this` `+ 8;`  `*(` `this` `+ 5) = 0LL;`  `constructAPIMappingTable(` `this` `);`  `}` |

调用 `APIMapping::APIMapping` 函数建立好上述的映射关系表后 `DataCenterHandler::request` 函数又调用了 `APIMapping::redirectRequest` 函数。

`APIMapping::redirectRequest` 函数是我们需要重点分析的内容，它的主要作用简而言之就是 `解析请求中的 API 编号，在 STL map 中查找对应的处理函数，然后通过函数指针动态调用`

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_PKXZBNRS35Z4EWS.webp)

前面我们分析过了，当 `api` 为 `629` 时，传入的 `payload` 字段的数据会被转发给 `plugincenter` 程序处理

所以下面我们来分析 `/usr/sbin/plugincenter` 程序

直接在 `plugincenter` 中找到 `datacenter::PluginApiMappingExtendCollection::sConstructMappingTable` 函数

同样这个函数也是通过 `map` 建立 `api` 编号和对应 `handler` 函数的映射关系

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_Z6464MDAE7DX8DK.webp)

|  |  |
| --- | --- |
| 1  2 | `v3 = 629;`  `*std::map<` `int`, `void` `(*)(json_object *,std::string &)> >::operator[](a1, &v3) = parseGetIdForVendor;` |

可以得知当 `api` 编号为 `629` 的时候，会执行 `parseGetIdForVendor` 函数

进入 `parseGetIdForVendor` 函数中

会将传入的 `Json` 数据内的 `appid` 字段作为参数传递到 `PluginApi::getIdForVendor` 函数中

![图片描述](https://bbs.kanxue.com/upload/attach/202511/1005607_9ZVAAFHH7YG5RV7.webp)

下面来看 `PluginApi::getIdForVendor` 函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52 | `__int64` `__usercall PluginApi::getIdForVendor@<X0>(` `__int64` `a1@<X1>,` `__int64` `a2@<X8>)`  `{`  `__int64` `v4;` `// x0`  `__int64` `v5;` `// x0`  `__int64` `JsonObject;` `// x19`  `__int64` `v7;` `// x0`  `char` `*v8;` `// x0`  `__int64` `v10;` `// [xsp+0h] [xbp+0h] BYREF`  `__int64` `v11;` `// [xsp+40h] [xbp+40h] BYREF`  `_BYTE v12[16];` `// [xsp+48h] [xbp+48h] BYREF`  `_QWORD v13[2];` `// [xsp+58h] [xbp+58h] BYREF`  `char` `v14;` `// [xsp+68h] [xbp+68h] BYREF`  `_BYTE v15[32];` `// [xsp+78h] [xbp+78h] BYREF`    `AppAccountManager::AppAccountManager(&v11);`  `if` `( !AppAccountManager::IsValidAppId(&v11, a1) )`  `{`  `google::LogMessage::LogMessage(`  `v12,`  `"src/api/PluginApi.cpp"`, `626LL,`  `2LL,`  `0LL,`  `&google::LogMessage::SendToSyslogAndLog,`  `0LL);`  `v4 = google::LogMessage::stream(v12);`  `std::operator<<<std::char_traits<` `char` `>>(v4,` `"invalid app id."` `);`  `google::LogMessage::~LogMessage(v12);`  `sub_8A940(&v10 + 120,` `""` `);`  `if` `( ApiBase::sCreateJsonObject(5LL) )`  `{`  `v5 = (json_object_to_json_string)();`  `std::string::operator=(v15, v5);`  `}`  `std::string::_M_dispose(v15);`  `}`  `v13[0] = &v14;`  `v13[1] = 0LL;`  `v14 = 0;`  `std::operator+<` `char` `>(` `"matool --method idForVendor --params "`, a1);`  `CommonUtils::sCallSystem(v15, v13);`  `std::string::_M_dispose(v15);`  `JsonObject = ApiBase::sCreateJsonObject(0LL);`  `v7 = json_object_new_string(v13[0]);`  `json_object_object_add(JsonObject,` `"idforvendor"`, v7);`  `v8 = json_object_to_json_string(JsonObject);`  `sub_8A940(a2, v8);`  `json_object_put(JsonObject);`  `std::string::_M_dispose(v13);`  `AppAccountManager::~AppAccountManager(&v11);`  `return` `a2;`  `}` |

分析可以得知这个函数的主要作用应该是下面三点

- 验证应用 ID 的有效性
- 通过系统命令调用外部工具获取供应商 ID
- 将结果封装为 JSON 格式返回

我们来一步步分析:

|  |  |
| --- | --- |
| 1 | `__int64` `__usercall PluginApi::getIdForVendor@<X0>(` `__int64` `a1@<X1>,` `__int64` `a2@<X8>)` |

- `a1`: 输入参数，指向包含 appid 的字符串（应用程序 ID）
- `a2`: 输出参数，用于存储返回结果的字符串引用

|  |  |
| --- | --- |
| 1 | `AppAccountManager::AppAccountManager(&v11);` |

在栈上构造一个 `AppAccountManager` 对象，用于验证应用 ID 的有效性

|  |  |
| --- | --- |
| 1  2  3 | `if` `( !AppAccountManager::IsValidAppId(&v11, a1) ) {`  `// 如果 appid 无效，执行这个代码块`  `}` |

检查传入的 appid 是否合法，如果不合法将会进入 if 中

但是如果不合法则不会进入 if 中，但是很奇怪，尽管不合法也没有退出该函数阻止后面的命令执行，甚至没有对该 appid 的内容做一个过滤，而是做了一个日志记录，创建了一个错误响应，但是就是没有 `return`，所以依旧会执行下面的代码

就在下面的代码中有这样一段代码

|  |  |
| --- | --- |
| 1  2  3  4  5 | `v13[0] = &v14;`  `v13[1] = 0LL;`  `v14 = 0;`  `std::operator+<` `char` `>(` `"matool --method idForVendor --params "`, a1);`  `CommonUtils::sCallSystem(v15, v13);` |

这句 `std::operator+<char>("matool --method idForVendor --params ", a1);` （这里我猜测实际应该是 `std::operator+<char>("matool...", a1, v13);` 而 `v13` 应该是被反编译的时候优化掉了，否则解释不了为什么会对 `v13` 产生影响）将 `v13` 变成了 `"matool --method idForVendor --params " + a1` 而 `a1` 则是用户可控的 `appid`
