
## 2、定位字符
所代表的是一个虚的字符，它代表一个位置，你也可以直观地认为“定位字符”所代表的是某个字符与字符间的那个微小间隙。

`^` 表示其后的字符必须位于字符串的开始处

`$` 表示其前面的字符必须位于字符串的结束处

`\b` 匹配一个单词的边界

`\B` 匹配一个非单词的边界

## 3、重复描述字符

`{n}` 匹配前面的字符`n`次

`{n,}` 匹配前面的字符`n`次或多于`n`次

`{n,m}` 匹配前面的字符`n`到`m`次

`?` 匹配前面的字符0或1次

`+` 匹配前面的字符1次或多于1次

`\*` 匹配前面的字符0次或式于0次

## 4、and or 匹配

`and` 符号 并

`or` 符号 或

例如：

`tcp and tcp.port==80`

`tcp or udp`

## 5、wireshark过滤匹配表达式实例

### 5.1、搜索按条件过滤udp的数据段payload（数字8是表示udp头部有8个字节，数据部分从第9个字节开始`udp[8:]`）

`udp[8]==14` （14是十六进制`0x14`）匹配payload第一个字节`0x14`的UDP数据包

`udp[8:2]==14:05` 可以`udp[8:2]==1405`，且只支持2个字节连续，三个以上须使用冒号：分隔表示十六进制。 (相当于 `udp[8]==14 and udp[9]==05`, `1405`是`0x1405`)

`udp[8:3]==22:00:f7` 但是不可以`udp[8:3]==2200f7`

`udp[8:4]==00:04:00:2a`，匹配payload的前4个字节`0x0004002a`

而`udp contains 7c:7c:7d:7d` 匹配payload中含有`0x7c7c7d7d`的UDP数据包，不一定是从第一字节匹配。

`udp[8:4] matches "\\x14\\x05\\x07\\x18"`

`udp[8:] matches "^\\x14\\x05\\x07\\x18\\x14"`

### 5.2、搜索按条件过滤tcp的数据段payload（数字20是表示tcp头部有20个字节，数据部分从第21个字节开始`tcp[20:]`）

`tcp[20:] matches "^GET [ -~]\*HTTP/1.1\\x0d\\x0a"`

等同`http matches "^GET [ -~]\*HTTP/1.1\\x0d\\x0a"`

`tcp[20:] matches "^GET (.\*?)HTTP/1.1\\x0d\\x0a"`

`tcp[20:] matches "^GET (.\*?)HTTP/1.1\\x0d\\x0a[\\x00-\\xff]\*Host: (.\*?)pplive(.\*?)\\x0d\\x0a"`

`tcp[20:] matches "^GET (.\*?)HTTP/1.1\\x0d\\x0a[\\x00-\\xff]\*Host: "`

`tcp[20:] matches "^POST / HTTP/1.1\\x0d\\x0a[\\x00-\\xff]\*\\x0d\\x0aConnection: Keep-Alive\\x0d\\x0a\\x0d\\x0a"`

检测SMB头的smb标记，指明smb标记从tcp头部第24byte的位置开始匹配。

`tcp[24:4] == ff:53:4d:42`

检测SMB头的smb标记，`tcp`的数据包含十六进制`ff:53:4d:42`，从`tcp`头部开始搜索此数据。

`tcp contains ff:53:4d:42`

`tcp matches "\\xff\\x53\\x4d\\x42"`

检测`tcp`含有十六进制`01:bd`，从`tcp`头部开始搜索此数据。

`tcp matches "\\x01\\xbd"`

检测MS08067的RPC请求路径

`tcp[179:13] == 00:5c:00:2e:00:2e:00:5c:00:2e:00:2e:00`

\      .     .     \     .     .

### 5.3、其他

`http.request.uri matches ".gif$"` 匹配过滤HTTP的请求URI中含有`".gif"`字符串，并且以`.gif`结尾（4个字节）的http请求数据包（`$`是正则表达式中的结尾表示符）

注意区别：`http.request.uri contains ".gif$"` 与此不同，`contains`是包含字符串`".gif$"`（5个字节）。匹配过滤HTTP的请求URI中含有`".gif$"`字符串的http请求数据包（这里`$`是字符，不是结尾符）

`eth.addr[0:3]==00:1e:4f` 搜索过滤MAC地址前3个字节是`0x001e4f`的数据包。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
