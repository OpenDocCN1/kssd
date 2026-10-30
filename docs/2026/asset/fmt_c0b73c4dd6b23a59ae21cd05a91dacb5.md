
```c
int __fastcall detect_remoteIp_invalid(char *buf)
{
    int len; // $v0
    char *v3; // $a0
    char *v4; // $v0
    int v5; // $v1
    len = strlen(buf);
    v3 = buf;
    v4 = &buf[len];
    while ( v3 != v4 )
    {
        v5 = *v3;
        if ( (unsigned __int8)(v5 - 48) < 0xAu )
        {
            ++v3;
        }
        else
        {
            ++v3;
            if ( v5 != '.' )
            {
                uf_log_printf(
                    uf_log,
                    "ERROR (%s %s %d)invalid char: %c, need [number][.][number]!",
                    "sgi.c",
                    "detect_remoteIp_invalid",
                    273,
                    v5);
                return -1;
            }
        }
    }
    return 0;
}
```

## 参考信息

<mark class="encrypted">5bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6X3g2Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8X3y4Y4K9g2)9J5k6r3u0A6L8W2)9J5c8X3y4$3k6h3&6S2L8h3g2Q4x3X3g2U0k6$3W2Q4x3@1k6F1j5h3#2W2i4K6y4p5x3U0l9J5x3#2)9J5k6o6x3^5z5e0l9J5</mark>

## 前言

这是我收获的第一个 [CVE](<https://bbs.kanxue.com/elink@f86K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6X3g2Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8X3y4Y4K9g2)9J5k6r3u0A6L8W2)9J5c8X3y4$3k6h3&6S2L8h3g2Q4x3X3g2U0k6$3W2Q4x3@1k6F1j5h3#2W2i4K6y4p5b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6x3^5z5e0l9J5)>) 编号，在复现了 **winmt** 师傅的 `CVE-2023-34644` 后，他告诉我最新的固件虽然做了一些简单的处理，导致无法在未授权的情况下 `RCE` ，但因为没有从根源上对命令执行点做限制，所以在授权后，仍然可以进行 `RCE` 。我对最新的固件进行了分析，完整记录了授权后的 `RCE` 漏洞从分析到利用的过程。从提交漏洞到现在也有半年的时间了，并且某厂商官网也已经发布了最新的固件，现将该文章分享出来，供大家进行学习和研究。

PS:本文记录的部分内容和之前发布过的复现 `CVE-2023-34644` 文章中的部分内容有相似之处，因为对前期的 `lua` 文件分析基本一致。为了保证读任何一篇单独的文章都较为通顺和连贯，因此就保留了两篇文章中相似的部分。

## 仿真环境搭建

仿真环境搭建请参考 <https://bbs.kanxue.com/thread-277386.htm#msg_header_h2_4>

该文章详细记录了某厂商路由器的仿真过程

`qemu` 的启动脚本如下

```bash
#!/bin/bash
sudo qemu-system-mipsel \
-cpu 74Kf \
-M malta \
-kernel vmlinux-3.2.0-4-4kc-malta \
-hda debian_squeeze_mipsel_standard.qcow2 \
-append "root=/dev/sda1 console=tty0" \
-net nic \
-net tap,ifname=tap0,script=no,downscript=no \
-nographic
```

其中的 `vmlinux-3.2.0-4-4kc-malta` `debian_squeeze_mipsel_standard.qcow2` 文件从 [a1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8X3#2A6M7s2y4W2L8q4)9J5c8R3`.`.](<https://bbs.kanxue.com/elink@42cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8X3#2A6M7s2y4W2L8q4)9J5c8R3`.`.)>) 进行下载

在执行 `qemu` 启动脚本之前，执行下面的脚本，创建一个网桥

```sh
#!/bin/sh
#sudo ifconfig eth0 down                 # 首先关闭宿主机网卡接口
sudo brctl addbr br0 # 添加一座名为 br0 的网桥
sudo brctl addif br0 ens33 # 在 br0 中添加一个接口
sudo brctl stp br0 off # 如果只有一个网桥，则关闭生成树协议
sudo brctl setfd br0 1 # 设置 br0 的转发延迟
sudo brctl sethello br0 1 # 设置 br0 的 hello 时间
sudo ifconfig br0 0.0.0.0 promisc up # 启用 br0 接口
sudo ifconfig ens33 0.0.0.0 promisc up # 启用网卡接口
sudo dhclient br0 # 从 dhcp 服务器获得 br0 的 IP 地址
sudo brctl show br0 # 查看虚拟网桥列表
sudo brctl showstp br0 # 查看 br0 的各接口信息
sudo tunctl -t tap0 -u root # 创建一个 tap0 接口，只允许 root 用户访问
sudo brctl addif br0 tap0 # 在虚拟网桥中增加一个 tap0 接口
sudo ifconfig tap0 0.0.0.0 promisc up # 启用 tap0 接口
sudo brctl showstp br0
```

## 漏洞分析

### lua文件调用链分析

#### 新版本219调用链分析

在 `usr/lib/lua/luci/modules/cmd.lua` 文件中有如下代码，容易让初学者搞混，所以在此简单说明一下

```lua
local opt = {"add", "del", "update", "get", "set", "clear", 'doc'}
acConfig, devConfig, devSta, devCap = {}, {}, {}, {}

for i = 1, #opt do
    ......
    devSta[opt[i]] = function(params)
        local model = require "dev_sta"
        params.method = opt[i]
        params.cfg_cmd = "dev_sta"
        local data, back, ip, password, shell = doParams(params)
        return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
    end
    ......
end
```

首先是先定义了一个表 `opt` 里面装了字符串 `add` `del` `upload` 等字符串，然后又定义了四张空表 `acConfig` `devConfig` `devSta` `devCap` ，接下来是一个 `for` 循环来遍历 `opt` 表。

以 `devSta[opt[i]] = function(params)` 这行代码为例，假设现在 `opt[i]` 是元素 `add`，`function(params)` 这里是声明了一个匿名函数，因为函数也是一个变量，这个变量被直接存储到了 `devSta` 表中，以键值的形式存在，键就是字符串 `add` 而值就是这个函数，之后调用这个函数的话可以直接写 `devSta["add"]()`

```lua
function hello()
    local model = require "dev_sta"
    params.method = opt[i]
    params.cfg_cmd = "dev_sta"
    local data, back, ip, password, shell = doParams(params)
    return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
end
devSta["add"] = hello --假设此时遍历到了opt表中的add元素
```

为什么特别说明这里呢？因为我在开始分析的时候，我一直以为这里是匹配到对应的键值后直接去执行函数，导致在此处执行了 `doParams` `fetch` 函数（实际上通过上面的分析也知道，这里只是定义了这些函数，并没有进行调用）

下面开始正式从入口分析 `/api/cmd` 的这条链，在 `/usr/lib/lua/luci/controller/eweb/api.lua` 文件中存在 `entry({"api", "cmd"}, call("rpc_cmd"), nil)` 这行代码，意味着授权后访问 `/api/cmd` 路径时，可以调用 `rpc_cmd` 函数

```lua
function rpc_cmd()
    local jsonrpc = require "luci.utils.jsonrpc"
    local http = require "luci.http"
    local ltn12 = require "luci.ltn12"
    local _tbl = require "luci.modules.cmd"
    http.prepare_content("application/json")
    ltn12.pump.all(jsonrpc.handle(_tbl, http.source()), http.write)
end
```

通过分析 `rpc_cmd` 函数得知 `_tbl` 已经包含了 `cmd.lua` 中所有变量的定义（上文已经分析过了），主要是 `ac_config` `dev_config` `dev_sta` 这三个表包含了 `add` `del` `update` `get` `set` `clear` `doc` 这些操作，而 `devCap` 表只有 `get` ，相关代码如下

```lua
local opt = {"add", "del", "update", "get", "set", "clear", 'doc'}
acConfig, devConfig, devSta, devCap = {}, {}, {}, {}

for i = 1, #opt do
    acConfig[opt[i]] = function(params)
        local model = require "ac_config"
        params.method = opt[i]
        params.cfg_cmd = "ac_config"
        local data, back, ip, password, shell = doParams(params)
        return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
    end

    devConfig[opt[i]] = function(params)
        local model = require "dev_config"
        params.method = opt[i]
        params.cfg_cmd = "dev_config"
        local data, back, ip, password, shell = doParams(params)
        return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
    end

    devSta[opt[i]] = function(params)
        local model = require "dev_sta"
        params.method = opt[i]
        params.cfg_cmd = "dev_sta"
        local data, back, ip, password, shell = doParams(params)
        return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
    end
    if opt[i] == "get" then
        devCap[opt[i]] = function(params)
            local model = require "dev_cap"
            params.method = opt[i]
            params.cfg_cmd = "dev_cap"
            local data, back, ip, password, shell = doParams(params)
            return fetch(model.fetch, shell, params, opt[i], params.module, ip, password)
        end
    end
    if opt[i] == "doc" then
        syshell = function(params)
            local tool = require "luci.utils.tool"
            return tool.doc(params)
        end
    end
end
```

然后来看 `rpc_cmd` 函数中的这行代码 `ltn12.pump.all(jsonrpc.handle(_tbl, http.source()), http.write)`

`jsonrpc.handle` 函数的参数是 `_tbl` ，看下 `luci.utils.jsonrpc` 文件中的 `handle` 函数，发现又将参数 `tbl` 传给了 `resolve` ，同时传入的还有报文中的 `method` 字段

```lua
function handle(tbl, rawsource, ...)
    ......
    if stat then
        if type(json.method) == "string" then
            local method = resolve(tbl, json.method)
            if method then
                response = reply(json.jsonrpc, json.id, proxy(method, json.params or {}))
                ......
            end
    end
end
```
