
```python
#coding:utf8
import hashlib
import datetime

def md5(test_str: str, key=None):
    print('MD5加密前为 ：' + test_str)
    if key is None:
        m = hashlib.md5()
    else:
        m = hashlib.md5(key.encode('utf-8'))
    m.update(test_str.encode())
    res = m.hexdigest()
    print('MD5加密后为 ：' + res)
    return res

def gethkey():
    str1 = "1666285158"
    str1 = "1666259837"
    str2 = "/bbs/app/api/user/permission/"
    #str2 = "/account/get_ads_info/"
    if str2[-1] == "/":
        str2 = str2[:-1]
    zuhe_str = str2 + "/bfhdkud_time=" + str1
    #print(zuhe_str)
    ret = md5(zuhe_str).replace("a", "app").replace("0", "app")
    hkey = md5(ret)[0:10]
    print(hkey)

if __name__ == '__main__':
    gethkey()

'''
MD5加密前为 ：/bbs/app/api/user/permission/bfhdkud_time=1666259837
MD5加密后为 ：5e3fa112b55e9b8a087a5a1196f4645f
MD5加密前为 ：5e3fapp112b55e9b8appapp87app5app1196f4645f
MD5加密后为 ：0cf9be6f884cbf25ffd97e9fa61778d4
0cf9be6f88
'''
```

对比下请求报文，发现 `key` 值是一样的。

```shell
GET /bbs/app/api/user/permission?heybox_id=???&imei=???&os_type=Android&os_version=8.1.0&version=1.3.92&_time=1666259837&hkey=0cf9be6f88 HTTP/1.1
Referer: http://api.maxjia.com/
User-Agent: Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36 ApiMaxJia/1.0
Cookie: ???
Host: api.xiaoheihe.cn
Connection: Keep-Alive
Accept-Encoding: gzip
#####################################################
GET /account/get_ads_info/?heybox_id=???&imei=???&os_type=Android&os_version=8.1.0&version=1.3.92&_time=1666285158&hkey=d630c904c2 HTTP/1.1
Referer: http://api.maxjia.com/
User-Agent: Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36 ApiMaxJia/1.0
Cookie: ????
Host: api.xiaoheihe.cn
Connection: Keep-Alive
Accept-Encoding: gzip
```

### 参考

[小黑盒逆向分析笔记（二） - Chr\_小屋 (chrxw.com)](<elink@857K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3S2J5P5s2N6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7X3y4Z5K9i4k6W2M7#2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3z5q4)9J5c8U0l9#2i4K6u0r3x3e0x3@1y4#2)9J5k6h3S2@1L8h3H3.>)

## 前言

业余时间写的，纯兴趣更新! 做个记录。

## 测试环境

雷电模拟器 Android 7.1.2/pixexl2 Android 8.1

### 抓包测试

首先在apk中进行登录账户的时候进行抓包，获取登录凭证。

请求报文如下

```http
GET /store/has_unfinished_game/?heybox_id=???&imei=???&os_type=Android&os_version=7.1.2&version=1.1.50&_time=1666165262&hkey=5656c95245635812219460113c5a14eb HTTP/1.1
Referer: http://api.maxjia.com/
User-Agent: Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36 ApiMaxJia/1.0
Cookie: pkey=???
Host: api.xiaoheihe.cn
Connection: Keep-Alive
Accept-Encoding: gzip
```

在 `GET` 请求中发现有传递以下参数

| 参数 | 说明 |
| --- | --- |
| `heybox_id` | 固定且唯一（对于每一个用户来说） |
| `imei` | 移动通信国际识别码（每个移动设备具有唯一性） |
| `os_type` | 系统类型 |
| `os_version` | 系统版本 |
| `version` | apk 的版本 |
| `_time` | 时间 |
| `hkey` | 时间 key |

以及 cookie 中的 `pkey`：

`Cookie: pkey=???` 固定且唯一（对于每一个用户登录后）。

以上参数，我们需要知道 `hkey` 是怎么来的。

因为是请求报文，所以算法肯定在 apk 内部实现的，要么在应用层，要么在 `so` 层。

### 定位算法(v1.1.50)

将 apk 拖入到 GDA 中进行字符串搜索 `hkey`。

![](https://bbs.kanxue.com/upload/attach/202210/847319_4MU8GUG8JZCDEKC.png)

发现在 `intercept` 方法中存在这个字符串，双击进入。
