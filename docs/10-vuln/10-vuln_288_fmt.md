
# sulley 网络协议 Fuzzing 测试文档

> 原文：[`www.pediy.com/kssd/pediy12/135764.html`](https://www.pediy.com/kssd/pediy12/135764.html)

## Sully

`Sully`:

监视网络,并系统的自动维护着相应记录.

监视目标的健康状况,并有能力使用多种方法使其恢复到一个好的状态.

`Sully` 能够发现,跟踪,以及对检测到的错误进行分类.

`Sully` 能够并行执行,增加了测试的速度.

`Sully` 能够自动的确定测试案例中那一个单一系列引发的错误.

## Sully 的使用

数据表示: 是使用任何 fuzzer 的第一步.运行目标程序,将协议独立的请求和响应 ,并使用 sully 中的 `blocks` 表示.

会话 : 将开发的请求连接在一起形成会话,并附加上各种可用的监视代理(`socket`,`debugger`,etc..)然后开始 fuzzing

事后分析:检查生成的数据和监控结果,重放单一测试案例.

## 目录结构

`archived_fuzzies`: 这是个自由形态的目录,以 fuzz 目标名组织在一起,用来存储已存档的 fuzzer 和 fuzz 会话产生的数据.

`audits`: 一个活动会话所记录的 PCAP 数据,崩溃的二进制文件,代码覆盖范围和分析的图标都应该放在此目录.

一旦 fuzz 会话完成,所有的数据都要移到"`Archived_fuzzies`"目录中.

`docs`: 文档和生成的 Epydoc API references

`requests`: Sully requests 库,每个目标应该拥有它自己的能被用来存储多个请求的文件.

`___REQUESTS___.html`: 包含已存储请求的类别和个别类型列表说明,按字母顺序维护.

`http.py` : 各种各样的 web 服务 fuzzing 请求

`trend.py` : 包含一个完整的 fuzz 演练相关请求,在稍后的文档中讨论.

`sulley`: fuzzer 框架.除非你想扩展这个框架,否则不要轻易的改动这些文件.

`legos`: 用户定义的复杂原语

`ber.py` ASN.1/ BER 原语

`dcerpc.py` 微软 RPC NDR 原语

`misc.py` 诸如 E-mail 地址和主机名等各种各样的未分类原语

`xdr.py` XDR types

`pgraph`: python 图形抽象库,建立 sessions 时使用.

`utils`: 各种辅助例程

`dcerpc.py` 微软 RPC 帮助例程.比如绑定一个接口以及生成一个请求.

`misc.py` 各种未分类的例程,比如 CRC-16 以及 UUID 处理例程

`scada.py` SCADA 特有的帮助例程,包含 DNP3 块编码器

`__init__.py` : 用于创建请求时定义的各种各样的别名.

`blocks.py` : Blocks 和 block 帮助在这里定义.

`instrumentation.py` 外置仪表特征,用于并不支持 debugger 的监视目标

`pedrpc.py` 用于定义 sully 在不同的代理和主 fuzzer 之间通信的客户端和服务器类.

`primitives.py` 各种 fuzzer 原语,包括静态,随机,字符串和整数定义

`sessions.py` 创建并执行一个会话的功能函数.

`sex.py` sully 的自定义异常处理类

`unit_tests`: sully 的单元测试装置

`utils`: 各种独立的实用工具

`crashbin_explorer.py` 命令行工具，用于扫描序列化存储于 crash 二进制文件结果。

`pcap_cleaner.py` 命令行工具，用于清除没有错误关联的 PACP 目录

`network_monitor.py` PedRpc 驱动的网络监视代理

`process_monitor.py` PedRpc 驱动，基于调试器的目标监视代理

`unit_test.py` sulley 的单元测试

`vmcontrol.py` PedRpc 驱动的 VMWare 控制代理。

## 作者

`Pedram Amini`

`http://pedram.openrce.org`

`Aaron Portnoy`

`http://dvlabs.tippingpoint.com/team/aportnoy`

## 安装和需求

`network_monitor.py`:

`CORE Pcapy`,

`http://oss.coresecurity.com/projects/pcapy.html`

`CORE Impacket`

`http://oss.coresecurity.com/projects/impacket.html`

`process_monitor.py`

`PaiMei`

`http://code.google.com/p/paimei/`

`http://www.openrce.org/downloads/details/208/PaiMei`

`http://www.nologin.org/main.pl?action=codeList&`

## 数据表示

### 初始化并命名一个请求

`sully` 使用基于数据块表示的方法来生成独立的请求，然后将这些请求放在一起生成一次会话.

`s_initialize("new request")`

现在，你可以添加原语，`blocks` 或者嵌套的 `blocks` 来生成请求。每一个原语都能被单独的 rendered 或突变。rendering 一个原语

将返回其关联的 raw data 数据格式内容。突变一个原语将变化它内部的内容。rendering 和 mutating 的概念被大多数 fuzzer 开发者

抽象出来的，所以不用为它担心。然而，当 fuzzable 的值被穷尽时，任何可突变的原语都接受一个默认值.

### 静态数据 以及 随机数据

让我们开始一个最简单的原语，`s_static()`,这个原语添加一个任意长度静态不突变的值到 request 中，为了便利，sully 使用了

各种不同的别名 `s_dunno()`,`s_raw()`,`s_unknown()`都是 `s_static()`的别名.

```
# these are all equivalent:
s_static("pedram\\x00was\\x01here\\x02")
s_raw("pedram\\x00was\\x01here\\x02")
s_dunno("pedram\\x00was\\x01here\\x02")
s_unknown("pedram\\x00was\\x01here\\x02")
```

原语，`block` 等，都有一个可选的名字关键字参数。指定一个名字项可以直接使用 request 变量 `request.names["name"]` 存取，而

不需要遍历 block 结构来找到你期望的值.

`s_binary` 和 `s_static` 类似，但不相同，`s_binary` 接受多种形式表示的二进制数据，SPIKE 用户应该很熟悉它的使用，

```
# yeah, it can handle all these formats.
s_binary("0xde 0xad be ef \\xca fe 00 01 02 0xba0xdd f0 0d", name="complex")
```

大多数 sulley 原语又 fuzz 启发式驱动，因此它们突变的数目是有限的，但是 `s_random` 除外，它被利用来生成各种长度的随机数值，该原语必须携带两个

参数`min_length`,`max_length`,用来说明在每一次迭代生成随机数值得最大，最小长度值，该原语，还接受如下关键字参数.

`num_mutations`: (integer, default=25) 在恢复到默认值前的突变次数

`fuzzable`: (boolean, default=True) 开启和关闭该原语的可突变性

`name`: (string, default=None) 通过名字，sulley 可以对该原语直接存取.

`number_mutations` 关键字参数指定在被认为该原语耗尽时应该突变该原语多少次。如果要生成固定长度的随机数，指定 `min_length` 和 `max_length`

相等即可

### 整数

二进制和 anscii 协议中可能会包含各种大小的整数域，如 http 协议的内容长度域，和所有的 fuzz 框架一样，我们在这里也需要来产生这些数据：

1 byte: `s_byte()`, `s_char()`

2 bytes: `s_word()`, `s_short()`

4 bytes: `s_dword()`, `s_long()`, `s_int()`

8 bytes: `s_qword()`, `s_double()`

每一个整数类型至少要接受一个单一的参数，即默认整数值，此外，下面这些可选关键字参数可被指定.

`endian`: (character, default='<') bit 位序 指定 '<' 为小端，'>'为大端

`format`: (string, default="binary")输出格式， "binary" 或 "ascii", 控制者原语表示的整数格式， 例如，值 100 ascii 格式为 ‘100’ 而 "\\x64" 是二进制格式

`signed`: (boolean, default=False) 使得大小为有符号数，还是无符号数，仅当 format="ascii"时适用

`full_range`: (boolean, default=False) 使该原语突变所有的可能值

`fuzzable`: (boolean, default=True) 开启和关闭该原语的可突变性

`name`: (string, default=None) 可以通过在 request 中指定该名称来直接存储该原语的值

`full_range` 参数是上面所有参数中最值得关注的。考虑一下，当你 fuzz 一个 DWORD 类型的值时，一共有 4294967295 种可能值，以每秒 10 个测试案例的速度

你需要 13 年才能 fuzz 完这个原语，通过定义一些 smart 数值集来减小原有穷举空间，包括在 0 边界处，+，- 10，以及最大值除 2，4，6，8，10.16，32 等得到

的集合，而穷尽减小的空间值（141 测试案例） 仅需要 10 几秒.

### 字符串和分隔符

字符串能在协议中的使用非常广泛，邮件地址，主机名，用户名，密码等等，你不用怀疑，字符串的应用贯穿在整个 fuzzing 的过程中，`s_string` 原语

负责这些域，这个原语只需要带一个默认参数值为必选参数，下面这些附加参数也可被指定.

`size`: (integer, default=-1) 字符串的静态大小，对于动态大小，指定该值为-1

`padding`: (character, default='\\x00') 如果明确大小被指定，而产生的大小小于指定值， 使用这个值来填充直到满足条件为止.

`encoding`: (string, default="ascii") 字符串使用的编码，有效的选项包含无论在 python 中任何地方都能被接受的 s.encode,对于微软 unicode 字符串，指定 `utf_16_le`

`fuzzable`: (boolean, default=True) 开启和关闭该原语的可突变性

`name`: (string, default=None) 可以通过在 request 中指定该名称来直接存储该原语的值

字符串通常被分割符划分为几个子域，例如，空格被用作 http 协议请求的分割符，"GET /index.html HTTP/1.0"前面的/ 和 . 同样作为该请求的分割符.

但你定义协议时，请确保使用 `s_delim()`原语代替分割符，正如其它原语一样，必须指定一个默认值作为它的第一个参数，同样，和其它参数一致，

`s_delim()`同样接受可选的 `fuzzable` 和 `name` 参数，定界符的突变包括，重复，替代，排除.

作为一个完整的例子，fuzzing http 标签时使用如下系列原语：

```
# fuzzes the string: <BODY bgcolor="black">
s_delim("<")
s_string("BODY")
s_delim(" ")
s_string("bgcolor")
s_delim("=")
s_delim("\\"")
s_string("black")
s_delim("\\"")
s_delim(">")
```

### Blocks

必须精通的原语，让我们来看下，如何组织并嵌入一个块，新的块使用 `s_block_start()`定义，使用 `s_bolck_end()`关闭，每个块必须给定

一个名称，作为 `s_block_start()`的第一个参数，这个例程同样可以指定如下可选参数：

`group`: (string, default=None) 和该块关联的组的名称

`encoder`: (function pointer, default=None) 函数指针，在 reader 该 block 数据返回之前，提供一次处理该 block 数据的机会

`dep`: (string, default=None) 可选的参数，用来指定该 block 依赖的值

`dep_value`: (mixed, default=None) 如使该 block 的值 能被 readered，该值必须在 dep 域中被包含

`dep_values`: (list of mixed types, default=[]) 为了该 block 的值能被 readerd，dep 可能包含的值列表.

`dep_compare` (string, default="==") 应用于依赖的比较方法，有效的可选操作为"==", "!=" , ">", ">=", "<" and "<=".

Group ,encoding,以及依赖性是大多数 fuzz 框架没有的特性，我们将在下面论述它们.

### 组

`group` 允许你连接一个块到指定的 group 原语，和一个组关联的 block 必须为每一个组中的值循环穷尽该 block 的所有空间，例如，在表示一个有效的 opcode 列表或一些有

相同参数的行为时，组原语是非常有用的，`s_group` 定义一个组，并接受两个必须的参数，第一个参数指定组名称，第二个参数指定一个需要迭代的原始值列表.

例如，下面的请求完成 web ，服务器的 fuzz：

```
# import all of Sulley's functionality.
from sulley import *
# this request is for fuzzing: {GET,HEAD,POST,TRACE} /index.html HTTP/1.1
# define a new block named "HTTP BASIC".
s_initialize("HTTP BASIC")
```



``` 
# define a group primitive listing the various HTTP verbs we wish to fuzz.
s_group("verbs", values=["GET", "HEAD", "POST", "TRACE"])

# define a new block named "body" and associate with the above group.
if s_block_start("body", group="verbs"):

    # break the remainder of the HTTP request into individual primitives.
    s_delim(" ")
    s_delim("/")
    s_string("index.html")
    s_delim(" ")
    s_string("HTTP")
    s_delim("/")
    s_string("1")
    s_delim(".")
    s_string("1")
    # end the request with the mandatory static sequence.
    s_static("\\r\\n\\r\\n")

# close the open block, the name argument is optional here.
s_block_end("body")
```

该脚本由 sulley 组件的引入开始，接下来，初始化并一个请求，并为该请求命名为“HTTP BASIC”，该名称稍后被引用，来对 request 进行直接存取，

下一步，一个组被定义，名称为“verbs”，其可能的字符串值为"GET", "HEAD", "POST", "TRACE",一个名为 body 的 block 被定义，并使用 group 参数连接到

先前定义的 group 原语，注意，`s_block_start` 总是返回 True，这让你可以通过使用 if 语句来组织 block 的结构，让程序更具有可读性，注意，

`s_block_end()` 的名称参数是可选的，一系列的基本原语和数据原语被限定在 body 块中，当定义的请求被载入到一个 sulley 的会话中时，fuzzer 将为

每一个定义在组中的值，生成并传输 body 块中的所有可能值。

## Encoders

编码者是一个简单但非常有效的 block 修改器，在 block 将 readered 的内容返回并传输到物理线路之前，一个函数能被指定并附加到该 block 上来对该数据进行修改。

下面是一个 xor 加密器：

``` 
def trend_xor_encode(str):
    key = 0xA8534344
    ret = ""
    # pad to 4 byte boundary.
    pad = 4 - (len(str) % 4)
    if pad == 4:
        pad = 0
    str += "\\x00" * pad
    while str:
        dword  = struct.unpack("<L", str[:4])[0]
        str    = str[4:]
        dword ^= key
        ret   += struct.pack("<L", dword)
        key    = dword
    return ret
```

`encoder` 直带一个单一参数，即，传入被编码的数据，返回编码后的数据，通过定义一个编码器并附加到一个可 fuzzable 的 block 上，允许 fuzz 的开发者

可以继续他的开发而不用担心这个小障碍。

## Dependencies

`Dependencies` 允许你在读取一个 block 的值时应用一个依赖条件，可在块的初始化时连接一个原语来完成依赖性，该原语将被使用 `dep` 关键字的 block 依赖，当 sulley 读取

依赖的块时，它将检查说连接的原始值，并以此指导自己的行为。只有一个依赖的值时，可以通过指定 `dep_value` 关键字参数，如果有多个依赖的值，依赖的值列表能被

`dep_values` 关键字参数指定。最后，实际的条件比较关系可以通过 `dep_compare` 关键字修改。考虑如下一种情景，依赖于一个整数的值，但不同的数据被期望。

``` 
s_short("opcode", full_range=True)

# opcode 10 expects an authentication sequence.
if s_block_start("auth", dep="opcode", dep_value=10):
    s_string("USER")
    s_delim(" ")
    s_string("pedram")
    s_static("\\r\\n")
    s_string("PASS")
    s_delim(" ")
    s_delim("fuzzywuzzy")
s_block_end()

# opcodes 15 and 16 expect a single string hostname.
if s_block_start("hostname", dep="opcode", dep_values=[15, 16]):
    s_string("pedram.openrce.org")
s_block_end()

# the rest of the opcodes take a string prefixed with two underscores.
if s_block_start("something", dep="opcode", dep_values=[10, 15, 16], dep_compare="!="):
    s_static("__")
    s_string("some string")
s_block_end()
```

块的依赖性可以通过不同的数量和方式连接在一起，而得到强有力的组合。

## block helps

在使用 sulley 生成数据时，一个重要的方面就是熟悉 block helper 的利用，包括，大小，校验和，和重复。

### Sizers

SPIKE 使用者将会非常熟悉 `s_sizer()` 或者 `s_size()` 块助手，辅助函数使用 block 的名称作为它的第一个参数来确定该 block 的大小，并可接受如下参数：

`length`: (integer, default=4) 大小域的长度  
`endian`: (character, default='<') bit 位序 指定 '<' 为小端，'>' 为大端  
`format`: (string, default="binary") 输出格式, "binary" 或 "ascii", 控制着整数原语读入的格式  
`inclusive`: (boolean, default=False) sizer 是否应该计算它自身的长度?  
`signed`: (boolean, default=False) 使 sizer 成为有符号或无符号数，仅当格式为 ascii 时适用  
`fuzzable`: (boolean, default=False) 该原语可 fuzzing 性的开关  
`name`: (string, default=None) 可以通过在 request 中指定该名称来直接存储该原语的值

`sizer` 是数据生成中重要的组件，允许我们表示诸如 XDR 符号，ASN.1 等复杂的协议，但读取 sizer 的时候，sulley 将动态计算关联块的长度，

默认情况下，sulley 将不会 fuzz Sizer 域，在许多情况下，可能会有这种需求，如果遇到，enable fuzzable 标志即可。

### Checksums

和 Sizers 相似，`s_checksum()` 助手使用 block 的名称作为其第一个参数来确定该 block 的 checksum，如下的可选参数可被指定

`algorithm`: (string or function pointer, default="crc32"). 应用于目标块的 checksum 的算法(crc32, adler32, md5, sha1)  
`endian`: (character, default='<') bit 位序 指定 '<' 为小端，'>' 为大端  
`length`: (integer, default=0) 校验和的长度，指定为 0 则自动计算  
`name`: (string, default=None) 可以通过在 request 中指定该名称来直接存储该原语的值

`algorithm` 参数能被指定为"crc32", "adler32", "md5" or "sha1" 其中之一，或者，你可以指定一个函数指针，作为你自己定制的算法。

### Repeaters

`s_repeat()` 或者 `s_repeater()` 助手，被用于重复一个块为一个变量所代表的次数。这对于测试需要解析一个表为多个元素的溢出案例是非常有用的。

这个辅助程序需要携带三个必须参数，被重复的 block 的名称，最小重复次数和最大重复次数，此外，下面的可选参数可使用：

`step`: (integer, default=1) 最小，最大重复次数的步长  
`fuzzable`: (boolean, default=False) 该原语可 fuzzing 性的开关  
`name`: (string, default=None) 可以通过在 request 中指定该名称来直接存储该原语的值

考虑下下面这个例子：

它连接了三个我们所介绍的辅助程序，我们 fuzz 的协议的一部分是一个包含字符串的表格。

每一个表格中的项由两个字节的字符串类型域，两个字节的长度域，一个字符串域，最后以一个需要接受整个字符串域的 crc-32 校验和结尾

我们并不知道对这个有效的类型域是什么类型，所有我们使用随机数据来对其进行 fuzz，

使用 suelly 定义的这部分协议如下：

``` 
# table entry: [type][len][string][checksum]
if s_block_start("table entry"):
    # we don't know what the valid types are, so we'll fill this in with random data.
    s_random("\\x00\\x00", 2, 2)
    # next, we insert a sizer of length 2 for the string field to follow.
    s_size("string field", length=2)
    # block helpers only apply to blocks, so encapsulate the string primitive in one.
    if s_block_start("string field"):
        # the default string will simply be a short sequence of C's.
        s_string("C" * 10)
    s_block_end()
    # append the CRC-32 checksum of the string to the table entry.
    s_checksum("string field")
s_block_end()

# repeat the table entry from 100 to 1,000 reps stepping 50 elements on each iteration.
s_repeat("table entry", min_reps=100, max_reps=1000, step=50)
```

这个 sulley 脚本不仅可以发现解析表项时的错误，而且可能发现在处理超长表时的错误。

## Legos

sulley 使用 legos 来代表用户定义的组件，诸如，e_mail 地址，主机名，以及用于微软 RPC，XDR，ASN.1 等协议原语，在 ASN.1 中 /BER 字符串被描述为一个系列 `[0x04][0x84][dword length][string`，在 fuzzing 基于 ASN.1 的协议时，在每一个字符串前面都包括一个长度和类型的前缀，将变得相当难处理，作为代替，我们定义该 lege 来引用它。

``` 
s_lego("ber_string", "anonymous")
```

除了可选择的可选关键字参数外，每一个 lego 都有相似的格式，来指定一个独立的 lego。作为一个例子，考虑到 tag lego 的定义,当 fuzz xml-ish 协议时

是非常有用的：

``` 
class tag(blocks.block):
    def __init__(self, name, request, value, options={}):
        blocks.block.__init__(self, name, request, None, None, None, None)
        self.value   = value
        self.options = options
        if not self.value:
            raise sex.error("MISSING LEGO.tag DEFAULT VALUE")
        #
        # [delim][string][delim]
        self.push(primitives.delim("<"))
        self.push(primitives.string(self.value))
        self.push(primitives.delim(">"))
```

这个 lego 例子接受一个期望的 tag 作为字符串，并且将其压缩在适当的分割符内，如此做扩展了 blocks 类，通过 `self.push` 将合适的分割符和字符

串添加到 block 的栈中。

这里是另外一个为了生成 ANS.1/DER 整数的简单 lego，最小的共同点是代表的整数都是 4 字节整数，并有如下的形式，

`[0x02][0x04][dword]` 0x02 指定整数的类型，0x04 指定整数的长度是 4 字节长，doword 代表我们实际传入的值，如下是

在 sulley 中的定义：

``` 
class integer(blocks.block):
    def __init__(self, name, request, value, options={}):
        blocks.block.__init__(self, name, request, None, None, None, None)
        self.value   = value
```



### 代码片段

```python
self.options = options

if not self.value:
    raise sex.error("MISSING LEGO.ber_integer DEFAULT VALUE")

self.push(primitives.dword(self.value, endian=">"))
```

```python
def render(self):
    # let the parent do the initial render.
    blocks.block.render(self)

    self.rendered = "\\x02\\x04" + self.rendered

    return self.rendered
```

## Sessions

一旦你定义了许多请求，就是时候将它们连接在一起形成一个会话了。`sulley`比其它好的 fuzz 框架最大的优势在于它对协议模糊测试的深度。

它将各个请求连接在一起形成图，

```python
from sulley import *
s_static("helo")
s_initialize("ehlo")
s_static("ehlo")
s_initialize("mail from")
s_static("mail from")
s_initialize("rcpt to")
s_static("rcpt to")
s_initialize("data")
s_static("data")
sess = sessions.session()
sess.connect(s_get("helo"))
sess.connect(s_get("ehlo"))
sess.connect(s_get("helo"), s_get("mail from"))
sess.connect(s_get("ehlo"), s_get("mail from"))
sess.connect(s_get("mail from"), s_get("rcpt to"))
sess.connect(s_get("rcpt to"), s_get("data"))
fh = open("session_test.udg", "w+")
fh.write(sess.render_graph_udraw())
fh.close()
```

在 fuzz 测试时，`sulley`沿着图形结构，从根节点开始 fuzz 测试每一个组件，在这个例子中，它从 `helo` 请求开始 fuzzing，一旦完成，`sulley` 将 fuzz `mail from` 请求，它在每一个测试案例前添加一个有效的 `helo` 请求，接下来，`sulley` 将对 `rpct to` 进行 fuzzing，通过在每一个测试案例前添加一个有效的 `helo` 和 `mail from` 请求，这个过程一直到 `data` 请求完成后又从 `ehlo` 重新开始，这种将协议拆分为多个独立的请求，并 fuzz 了协议图的所有可能路径的能力是非常强大的。

当实例化一个 `sesssion` 时，下面的可选参数可能被使用：

`session_filename`: (`string`, `default=None`) 用于存储系列化数据的文件名 指定一个文件名允许你停止以及恢复一个 fuzzer 的执行。

`skip`: (`integer`, `default=0`). 需要跳过的测试案例数

`sleep_time`: (`float`, `default=1.0`) 在发送测试案例时睡眠的时间值

`log_level`: (`integer`, `default=2`) 设置日子级别，级别越高，得到的日志信息就越多

`proto`: (`string`, `default="tcp"`) 用于通信的协议

`timeout`: (`float`, `default=5.0`) 设置等待 `send`/`recv` 返回之前的超时时间.

另外一个要介绍的 `sulley` 的高级特性是可以在协议图结构的每一边注册一个回调函数，这允许我们在两个节点间注册一个回调函数，来实现诸如询问回答系统的功能，回调函数的原型如下：

```python
def callback(node, edge, last_recv, sock)
```

`node` 是将要被发送到 node，`edge` 是沿着当前路径到 node 的最后边，`last_recv` 包含从最后一次 `sock` 传输返回的数据，`sock` 是一个可用的 sock，回调函数在如下的情景中是非常有用的，例如，下一个包的大小由上一个包指定，如果你需要动态填入目标 IP 地址，注册一个 `callback` 从 `sock.getpeername()` 函数获取，边的回调函数可以在 `session.connect` 函数中又参数关键字 `callback` 指定。

## Target 和 Agents

下一步是定义目标，为它们添加代理并将其加入到会话中，下面这个例子初始化一个运行在 `vmware` 中的目标，并为其连接了 3 个代理：

```python
target = sessions.target("10.0.0.1", 5168)
target.netmon = pedrpc.client("10.0.0.1", 26001)
target.procmon = pedrpc.client("10.0.0.1", 26002)
target.vmcontrol = pedrpc.client("127.0.0.1", 26003)
target.procmon_options =
{
    "proc_name"      : "SpntSvc.exe",
    "stop_commands"  : ['net stop "trend serverprotect"'],
    "start_commands" : ['net start "trend serverprotect"'],
}
sess.add_target(target)
sess.fuzz()
```

实例化目标绑定在主机 `10.0.0.1` ，端口 为 `5168`. 一个网络监控代理运行在目标系统上，监听的默认端口为 `26001`，网络监控代理将记录所有的 `socket` 通信为独立的 `pcap` 包，使用测试案例号作为 PCap 文件标号。进程监控代理同样运行在目标系统上，监听默认端口 `26002`. 进程监控代理需要指定附加的进程名作为其参数，以及停止和打开该进程的命令行语句。最后，`VMware` 代理运行在本地引擎，监听 `26003`，目标被添加到 `session` 并开始 fuzz 测试，`sulley` 有能力 fuzzing 多个目标，每一个目标都要有唯一的代理，这可以将总测试空间划分在多个不同的目标上来为你节省时间。
