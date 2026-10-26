
## 前言

谈到逆向，大家第一反应就是OD, IDA, x64dbg这类的工具，由于我不是专业做逆向安全的，对于这些逆向工具的使用仅仅属于会用的程度，所以，本文，我将从一个软件开发者的正向角度在Windows平台下对某网盘的通信协议进行逆向分析。

## 工具准备

- [Proxifier](https://bbs.kanxue.com/elink@1beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3M7X3!0^5K9h3k6A6k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.)：主要用于抓包
- Charles或者Fiddler：用于分析https协议
- IDA
- OllyDbg或者x64dbg
- [Resource Hacker](https://bbs.kanxue.com/elink@f05K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4F1k6%4g2K6K9W2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2K9r3q4U0K9$3g2J5i4K6u0r3)（可选）
- [StudyPE+](https://bbs.pediy.com/thread-246459-1.htm)：主要用于修改固定基址（可选）
- [网盘客户端](https://bbs.kanxue.com/elink@643K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1K9i4c8I4K9i4g2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6l9`.`.)：使用的是3.0.1.1Windows版

## 协议分析

### 准备工作

既然我们要分析协议，首先就要抓包，主要有两类工具，其实现原理也不太一样。

> Wireshark+Chrome

首先添加`SSLKEYLOGFILE`环境变量，然后用Chrome把需要抓包的域名都访问一遍，然后Wireshark配置`Secretlog`项即可解密对应的https流量，具体实施方式不写了，有兴趣的同学以`Wireshark Chrome SSLKEYLOGFILE`这三个关键词进行搜索，教程实在是太多。

这里有两点要说明的是，首先，`eax`里是二级指针，所以需要`[[eax]]`才能定位到字符串位置，另外一点就是每次会申请新的内存空间来存放数据，所以字符串地址会变化，不过只要认准了`eax`就问题不大。

最终拼接完成的格式为：`[app_id];[exe_version];[mac];pc;[computer_name];[sys_version]`。

### sign参数分析

函数`sub_410AD0`是对字符串进行拼接，最终拼接完成的格式为：`app_id=d0beea3fe1a1423392690dfd4d3713fd&passport=lniwn&password=1a46be489100d6a089358eff29b98f7a&platform=[platform]&timestamp=1599636155000ZWtVdHB0WjBocFIxdmNzZlg3cUQ5SFhEYU5pWGsv`，拿到数据之后，由函数`sub_516850`进行签名，该函数内部调用了`sub_516730`对数据进行加密，下面截取核心代码

```c++
if ( md5_516730(v1, v2, &v12) ) // 数据加密 v1:原始串 v2:原始串长度 v12:接收缓冲区，16字节长
{
    Dst[0] = 0;
    for ( i = 0; ; ++i )
    {
        v6 = i;
        if ( i >= 0x10 )
            break;
        *(_DWORD *)Dst = 0;
        v19 = 0;
        swprintf_s(Dst, 4u, L"%02x", *(&v12 + i));
        sub_411600((int)&v9, Dst, (int)&v9);
    }
}
```

`sub_516730`对数据处理完成后，通过`v12`返回，然后以十六进制字符串形式进行存储到v9中，我们看到`v12`的长度`i`为0x10即16，以`%02x`打印的话，v9最终就是长度为32的字符串了，下面看下加密逻辑`sub_516730`：

```c++
char __cdecl md5_516730(BYTE *pbData, DWORD dwDataLen, BYTE *a3)
{
    int v4; // [esp+0h] [ebp-30h]
    DWORD pdwDataLen; // [esp+10h] [ebp-20h]
    HCRYPTPROV phProv; // [esp+14h] [ebp-1Ch]
    HCRYPTHASH phHash; // [esp+18h] [ebp-18h]
    char v8; // [esp+1Fh] [ebp-11h]
    int *v9; // [esp+20h] [ebp-10h]
    int v10; // [esp+2Ch] [ebp-4h]
    v9 = &v4;
    pdwDataLen = 16;
    v8 = 0;
    phProv = 0;
    phHash = 0;
    v10 = 0;
    if ( CryptAcquireContextW(&phProv, 0, 0, 1u, 0xF0000000)
        && CryptCreateHash(phProv, 0x8003u, 0, 0, &phHash)
        && CryptHashData(phHash, pbData, dwDataLen, 0)
        && CryptGetHashParam(phHash, 2u, a3, &pdwDataLen, 0) )
    {
        v8 = 1;
    }
    if ( phHash )
    {
        CryptDestroyHash(phHash);
        phHash = 0;
    }
    if ( phProv )
    {
        CryptReleaseContext(phProv, 0);
        phProv = 0;
    }
    return v8;
}
```

[`CryptCreateHash`](<elink@d59K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8X3y4J5P5i4m8@1i4K6u0r3L8X3k6Q4x3X3c8%4K9h3&6U0M7Y4W2H3N6q4)9J5k6r3y4J5P5i4m8@1j5%4u0W2j5i4c8W2K9r3q4K6K9l9.>)用来初始化hash表，`0x8003`表示`CALG_MD5`，所以这个函数就是对数据进行md5编码。

## 代码实现

目前，我们已经分析完客户端登录需要的所有参数，可以进行代码实现了

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import asyncio
import hashlib
import logging
import time
import aiohttp
from urllib.parse import quote

log = logging.getLogger()

app_id = 'd0beea3fe1a1423392690dfd4d3713fd'
magic_code = 'ZWtVdHB0WjBocFIxdmNzZlg3cUQ5SFhEYU5pWGsv'

def md5_string(src) -> str:
    if isinstance(src, str):
        return hashlib.md5(src.encode('utf-8')).hexdigest()
    elif isinstance(src, bytes):
        return hashlib.md5(src).hexdigest()
    else:
        raise TypeError('unsupported type {}'.format(type(src)))

def gen_platform() -> str:
    exe_version = '3.0.1.1'  # Matrix_Cloud.exe version
    mac = 'FF:FF:FF:FF:FF:FF'  # your mac address
    pc_name = 'LNIWN-PC'  # your pc name
    sys_version = 'Windows 10'
    return ';'.join((app_id, exe_version, mac, 'pc', pc_name, sys_version))

def gen_login_payload(passport, password, platform) -> str:
    password = md5_string(password)
    timestamp = str(int(time.time()) * 1000)
    raw_data = '&'.join((
        'app_id=' + app_id,
        'passport=' + passport,
        'password=' + password,
        'platform=' + platform,
        'timestamp=' + timestamp,
    ))
    raw_data += magic_code
    sign = md5_string(raw_data)
    return '&'.join((
        'app_id=' + app_id,
        'passport=' + quote(passport),
        'password=' + password,
        'platform=' + quote(platform),
        'timestamp=' + timestamp,
        'sign=' + sign,
    ))

async def login(client: aiohttp.ClientSession, passport, password):
    url = "https://pan-api.bitqiu.com/loginServer/login"
    async with client.post(url, data=gen_login_payload(passport, password, gen_platform()),
                           headers={
                               'Content-Type': 'application/x-www-form-urlencoded'
                           }) as resp:
        data = await resp.json(encoding='utf-8')
        log.debug(data)
        if data["code"] == "10200":
            return data["data"]
        else:
            log.error(data["message"])
            return

async def user_get(client: aiohttp.ClientSession, user: dict):
    url = "https://pan-api.bitqiu.com/user/get"
    async with client.post(url, data=aiohttp.FormData({
        'app_id': app_id,
        'platform': gen_platform(),
        'access_token': user['token'],
        'open_id': user['open_id'],
        'user_id': user['user_id'],
    }), headers={
        'Content-Type': 'application/x-www-form-urlencoded'
    }) as resp:
        data = await resp.json(encoding='utf-8')
        log.debug(data)
        if data["code"] == "10200":
            return data["data"]
        else:
            log.error(data["message"])
            return

async def run(loop):
    async with aiohttp.ClientSession(loop=loop, headers={
        'User-Agent': 'curl/3C48359D-C0B2-4C09-A7A7-608910AA4D62',
        'Accept': 'application/json,text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Encoding': 'gzip, deflate',
        'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
        'Connection': 'keep-alive',
        'Pragma': 'no-cache'
    }) as client:
        user = await login(client, "your passport", "your password")
        await user_get(client, user)

def main():
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(asctime)s - %(name)-12s - %(levelname)-5s - %(message)s',
        datefmt='%y-%m-%d %H:%M:%S'
    )
    loop = asyncio.new_event_loop()
    try:
        loop.run_until_complete(run(loop))
    except (asyncio.TimeoutError, asyncio.CancelledError, aiohttp.ClientError) as e:
        log.exception(e)
    finally:
        loop.run_until_complete(loop.shutdown_asyncgens())
        loop.close()

if __name__ == '__main__':
    main()
```

以上只是正常登录流程，对于初次登录的用户，或者mac地址变化的用户，会进入

```json
{'code': '10807', 'message': '更换设备登录验证'}
```

这个流程顺着上面的思路分析即可，这里就不再继续写了。

## 后记

对于安卓端，抓包看了下，参数结构与windows端是一致的，只是`sign`参数不一致，所以需要解包拿到`magic_code`，由于该网盘客户端使用了360加固，且在模拟器无法运行，而我手头又没有真机环境，就暂且放下了，后面有机会再写安卓端的吧。

> 本文首发于我的[个人博客](https://bbs.kanxue.com/elink@fb1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2j5h3!0S2i4K6u0W2L8h3g2Q4x3V1k6H3L8%4y4@1M7#2)9J5c8U0t1H3x3U0m8Q4x3V1k6Q4x3U0g2q4y4q4)9J5y4f1u0n7i4K6t1#2z5p5g2Q4x3U0g2q4y4W2)9J5y4f1q4p5i4K6t1#2b7e0y4Q4x3U0g2q4y4g2)9J5y4e0V1H3i4K6t1#2z5e0q4Q4x3U0g2q4y4g2)9J5y4f1q4r3i4K6t1#2b7U0W2Q4x3U0g2q4y4W2)9J5y4e0W2r3i4K6t1#2z5e0m8Q4x3U0g2q4y4#2)9J5y4f1u0p5i4K6t1#2z5e0q4Q4x3U0g2q4y4#2)9J5y4e0W2n7i4K6t1#2z5e0S2Q4x3U0g2q4z5q4)9J5y4f1u0r3i4K6t1#2z5f1u0Q4x3U0g2q4z5q4)9J5y4f1p5I4i4K6t1#2z5p5y4Q4x3U0g2q4z5g2)9J5y4e0R3H3i4K6t1#2z5o6k6Q4x3U0g2q4y4g2)9J5y4e0V1H3i4K6t1#2z5e0q4Q4x3U0g2q4y4g2)9J5y4e0R3^5i4K6t1#2z5o6k6Q4x3U0g2q4y4W2)9J5y4e0W2q4i4K6t1#2z5e0m8Q4x3V1j5`.)
