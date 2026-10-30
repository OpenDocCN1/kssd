
```lua
function rpc_auth()
  ...
  local _tbl = require "luci.modules.noauth"
  ...
  ltn12.pump.all(jsonrpc.handle(_tbl, http.source()), http.write)
end
```

进一步分析`/usr/lib/lua/luci/utils/jsonrpc.lua`中的`handle`及其相关函数，可以得知这里**通过`JSON`数据的`method`字段定位并调用`noauth.lua`中对应的函数，同时将`Json`数据的`params`字段内容作为参数传入**（由于与该漏洞原理关系不大，此处不展开分析）。

### 寻找可能存在漏洞的入口

在`noauth.lua`中，有`login`，`singleLogin`，`merge`和`checkNet`四个方法。其中，`singleLogin`函数无可控制的参数，不用看；`checkNet`函数中参数可控的字段只有`params.host`，并拼接入了命令字符串执行，但是在之前有`tool.checkIp(params.host)`对其的合法性进行了检查，无法绕过。

再来看到`login`登录验证函数，这里可控的字段乍一看比较多，比如`params.password`，`params.encry`，`params.limit`等字段。其中，对`params.password`字段用`tool.includeXxs`函数过滤了危险字符，故大概率之后会有相关的命令执行点。

```lua
function login(params)
  ...
  if params.password and tool.includeXxs(params.password) then
    tool.eweblog("INVALID DATA", "LOGIN FAILED")
    return
  end
  ...
  local checkStat = {
    password = params.password,
    username = "admin", -- params.username,
    encry = params.encry,
    limit = params.limit
  }
  local authres, reason = tool.checkPasswd(checkStat)
  ...
end
```

再来看到继续调用的`tool.checkPasswd`函数（在`/usr/lib/lua/luci/utils/tool.lua`中），其中检测了传入的`encry`和`limit`字段的真假值，并在这两个字段中写入了相应的固定字符串，`checkStat.username`又是传入的固定用户名`admin`，因此**真正可控的只有`password`字段**，并调用了`cmd.devSta.get`函数进一步处理。

```lua
function checkPasswd(checkStat)
  ...
  local _data = {
    type = checkStat.encry and "enc" or "noenc",
    password = checkStat.password,
    name = checkStat.username,
    limit = checkStat.limit and "true" or nil
  }
  local _check = cmd.devSta.get({module = "adminCheck", device = "pc", data = _data})
  ...
end
```

然而，虽然`password`字段用`includeXxs`函数（同样在`tool.lua`中）过滤了危险字符，但是**并没有过滤`\n`这个命令分隔符**。因此，若之后当真存在命令执行点的话，似乎还是有希望完成命令注入的。

```lua
function includeXxs(str)
  local ngstr = "[`&$;|]"
  return string.match(str, ngstr) ~= nil
end
```

继续往下看到`/usr/lib/lua/luci/modules/cmd.lua`，`devSta.get`对应着如下处理函数，其中`opt[i]`循环匹配到`get`方式，会通过`doParams`函数对传入的`Json`参数进行解析，将其中的`data`等字段分离出来，传入`fetch`函数做进一步处理。

```lua
devSta[opt[i]] = function(params)
  local model = require "dev_sta"
  params.method = opt[i]
  params.cfg_cmd = "dev_sta"
  local data, back, ip, password, shell = doParams(params)
  return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
```

然而，注意到`doParams`函数中对`data`字段进行提取的时候，用到了`luci.json.encode`函数。这里的`data`字段就是上述`checkPasswd`函数中传入`devSta.get`作为`Json`参数的`_data`的内容，我们的疑似注入点`password`字段就在其中。**此处的`luci.json.encode`函数会对`\n`（即`\u000a`）类字符进行转义，也就不会被解析成换行符了**，不论我们后续再如何传参，这个疑似的漏洞点已经被封堵住了。

```lua
if params.data then
  data = luci.json.encode(params.data)
  _shell = _shell .. " '" .. data .. "'"
end
```

因此，我们只能将目光聚焦于`noauth.lua`中最后一个可能的入口`merge`方法了。这个函数比较简单，**调用了`devSta.set`函数，其`Json`参数中的`data`字段就是传入的`POST`报文中`params`的内容**。

```lua
function merge(params)
  local cmd = require "luci.modules.cmd"
  return cmd.devSta.set({device = "pc", module = "networkId_merge", data = params, async = true})
end
```

这里`merge`方法的入口处没有任何过滤，不过之后是否存在字符过滤和命令执行点还需要进一步分析。

### 进一步分析参数传递过程

在`noauth.lua`的`merge`函数中，调用了`devSta.set`函数，同样是对应着`cmd.lua`中的如下位置，此时`opt[i]`循环到了`set`方式。此时，由于之前**没有任何过滤**，无需使用换行符作为命令分隔符，最简单的分号、反引号之类的即可，故`doParams`函数中的`encode`不会造成影响。

```lua
devSta[opt[i]] = function(params)
  local model = require "dev_sta"
  params.method = opt[i]
  params.cfg_cmd = "dev_sta"
  local data, back, ip, password, shell = doParams(params)
  return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
```

接着，我们可控的`data`字段将被传入`cmd.lua`的`fetch`函数中。其中，会将从第四个开始的参数（包括`data`字段），均传递到第一个参数所指向的函数中，**即`/usr/lib/lua/dev_sta.lua`中的`fetch`函数**。

```lua
local function fetch(fn, shell, params, ...)
  ...
  local _res = fn(...)
  ...
end
```

在`/usr/lib/lua/dev_sta.lua`的`fetch`函数中，这里的`cmd`是`set`方式，`module`是`networkId_merge`，而此处的`param`就是我们可控的`data`字段（即最初`POST`报文中`params`的内容）。可见，对一些字段赋予了真假值后，**最终将参数都传递给了`/usr/lib/lua/libuflua.so`中的`client_call`函数**。接下来，就是对二进制文件逆向分析并寻找是否存在命令执行点了。

```lua
function fetch(cmd, module, param, back, ip, password, force, not_change_configId, multi)
  local uf_call = require "libuflua"
  ...
  local stat = uf_call.client_call(ctype, cmd, module, param, back, ip, password, force, not_change_configId, multi)
  ...
end
```

然而，分析`libuflua.so`可以发现，`Lua`中所调用的`client_call`函数，其实**是`uf_client_call`函数**，这是在其他共享库中定义的函数。查找对比一下，不难发现这个函数定义在`/usr/lib/libunifyframe.so`中。

![](https://bbs.kanxue.com/upload/attach/202306/949925_5KFV68J7FM28BW7.png)

**在`/usr/lib/libunifyframe.so`的`uf_client_call`函数**中，先将传入的`data`等字段转为`Json`格式的数据，作为`param`字段的内容。然后将`Json`数据**通过`uf_socket_msg_write`用`socket`套接字（分析可知，此处采用的是本地通信的方式）进行数据传输**。

```c
json_object_object_add(v22, "data", v35);
LABEL_82:
...
json_object_object_add(v5, "params", v22);
v44 = (const char *)json_object_to_json_string(v5);
...
v45 = uf_socket_client_init(0LL);
...
v51 = strlen(v44);
uf_socket_msg_write(v45, v44, v51);
```

既然这里采用`uf_socket_msg_write`进行数据发送，那么肯定有某个地方**会使用`uf_socket_msg_read`进行数据接收**，再进一步处理。匹配一下，一共三个文件，很容易锁定`/usr/sbin/unifyframe-sgi.elf`文件。又发现在初始化脚本`/etc/init.d/unifyframe-sgi`中，启动了`unifyframe-sgi.elf`，即**说明`unifyframe-sgi.elf`一直挂在进程中**。因此，我们可以确定`unifyframe-sgi.elf`就是接收`libunifyframe.so`所发数据的文件（这里采用了`Ubus`总线进行进程间通信）。

![](https://bbs.kanxue.com/upload/attach/202306/949925_E8BC5BRJMZVY5FM.png)

```bash
$ cat etc/init.d/unifyframe-sgi
...
PROG=/usr/sbin/unifyframe-sgi.elf
...

if [ -f "$IPKG_INSTROOT/lib/functions/procd.sh" ]; then
...
else
...
start() {
...
echo "Starting $PROG ..."
service_start $PROG
...
}

stop() {
...
}
fi
```

接下来就是最核心的部分，对`unifyframe-sgi.elf`二进制文件进行逆向分析并寻找命令执行点了。

### 逆向分析并寻找命令执行点

> 由于篇幅限制，笔者无法对所有细节都做到详细分析，故建议读者在复现此部分内容之前，自己先逆向分析一遍，体会一下。

在`unifyframe-sgi.elf`中，**对`uf_socket_msg_read`函数交叉引用**，找到`socket`数据接收点。如下方代码段，`v6 = 0x432000`，简单计算一下，可知`v57`即为`uf_socket_msg_read`函数，其中**接收到的数据存储在`v56[1]`中**。接收到的`Json`数据**形如`{"method":"devSta.set", "params":{"module":"networkId_merge", "async":true, "data":"xxx"}}`（可结合上文，自行分析得出）**，其中`data`字段可控。

```c
v6 = 0x432000uLL;
...
v57 = *(__int64 (__fastcall **)(__int64, unsigned int **))(v6 + 1784); // uf_socket_msg_read
v58 = *v46;
*v56 = v46;
v59 = v57(v58, v56 + 1);
```

接下来就是根据调试等级向日志写入相关信息的部分，不需要管。之后，会调用`parse_content`函数，从这个名字就可以看出是对`v56`中的`Json`数据进行解析的。解析成功后，就会将处理后的`v56`作为参数传入`add_pkg_cmd2_task`函数。

```c
if ( !(*(unsigned int (__fastcall **)(unsigned int **))(v6 + 3328))(v56) ) // 0x432D00 parse_content
{
  ...
  if ( !(*(unsigned int (__fastcall **)(unsigned int **))(v6 + 1776))(v56) ) // 0x4326F0 add_pkg_cmd2_task
    ...
}
```

我们先来看`parse_content`函数，显然`method`字段不包含`cmdArr`，因此进入`else`分支，其中调用`parse_obj2_cmd`函数进行数据解析。
