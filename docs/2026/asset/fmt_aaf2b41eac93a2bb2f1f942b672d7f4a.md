
``` 
# import all of Sulley's functionality.

from sulley import *

# this request is for fuzzing: {GET,HEAD,POST,TRACE} /index.html HTTP/1.1

# define a new block named "HTTP BASIC".

s_initialize("HTTP BASIC")

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

    s_static("\r\n\r\n")

# close the open block, the name argument is optional here.

s_block_end("body")
```

该脚本由`sulley`组件的引入开始，接下来，初始化并一个请求，并为该请求命名为“HTTP BASIC”，该名称稍后被引用，来对`request`进行直接存取。

下一步，一个组被定义，名称为“verbs”，其可能的字符串值为`"GET"`, `"HEAD"`, `"POST"`, `"TRACE"`，一个名为`body`的`block`被定义，并使用`group`参数连接到先前定义的`group`原语。注意，`s_block_start()`总是返回`True`，这让你可以通过使用`if`语句来组织`block`的结构，让程序更具有可读性。注意，`s_block_end()`的名称参数是可选的，一系列的基本原语和数据原语被限定在`body`块中，当定义的请求被载入到一个`sulley`的会话中时，`fuzzer`将为每一个定义在组中的值，生成并传输`body`块中的所有可能值。

# Encoders

编码者是一个简单但非常有效的`block`修改器，在`block`将`readered`的内容返回并传输到物理线路之前，一个函数能被指定并附加到该`block`上来对该数据进行修改。

``` 
def trend_xor_encode(str):
    key = 0xA8534344
    ret = ""
    # pad to 4 byte boundary.
    pad = 4 - (len(str) % 4)
    if pad == 4:
        pad = 0
    str += "\x00" * pad
    while str:
        dword = struct.unpack("<L", str[:4])[0]
        str = str[4:]
        dword ^= key
        ret += struct.pack("<L", dword)
        key = dword
    return ret
```

`encoder`直带一个单一参数，即，传入被编码的数据，返回编码后的数据，通过定义一个`encoder`并附加到一个可`fuzzable`的`block`上，允许`fuzz`的开发者可以继续他的开发而不用担心这个小障碍。

# 依赖性

`Dependencies`允许你在读取一个`block`的值时应用一个依赖条件，可在块的初始化时连接一个原语来完成依赖性，该原语将被使用`dep`关键字的`block`依赖，当`sulley`读取依赖的块时，它将检查说连接的原始值，并以此指导自己的行为。只有一个依赖的值时，可以通过指定`dep_value`关键字参数，如果有多个依赖的值，依赖的值列表能被`dep_values`关键字参数指定。最后，实际的条件比较关系可以通过`dep_compare`关键字修改。考虑如下一种情景，依赖于一个整数的值，但不同的数据被期望。

``` 
s_short("opcode", full_range=True)

# opcode 10 expects an authentication sequence.
if s_block_start("auth", dep="opcode", dep_value=10):
    s_string("USER")
    s_delim(" ")
    s_string("pedram")
    s_static("\r\n")
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

# Block Helps

在使用`sulley`生成数据时，一个重要的方面就是熟悉block helper的利用，包括，大小，校验和，和重复。

## Sizers

`SPIKE`使用者将会非常熟悉`s_sizer()`或者`s_size()`块助手，辅助函数使用`block`的名称作为它的第一个参数来确定该`block`的大小，并可接受如下参数：

- `length`: (integer, default=4) 大小域的长度
- `endian`: (character, default='<') bit 位序 指定 `'<'` 为小端，`'>'`为大端
- `format`: (string, default="binary") 输出格式，`"binary"` 或 `"ascii"`，控制着整数原语读入的格式
- `inclusive`: (boolean, default=False) `sizer`是否应该计算它自身的长度？
- `signed`: (boolean, default=False) 使`sizer`成为有符号或无符号数，仅当格式为`"ascii"`时适用
- `fuzzable`: (boolean, default=False) 该原语可fuzzing性的开关
- `name`: (string, default=None) 可以通过在`request`中指定该名称来直接存储该原语的值

`sizer`是数据生成中重要的组件，允许我们表示诸如`XDR`符号，`ASN.1`等复杂的协议，但读取`sizer`的时候，`sulley`将动态计算关联块的长度，默认情况下，`sulley`将不会`fuzz` `Sizer`域，在许多情况下，可能会有这种需求，如果遇到，enable `fuzzable`标志即可。

## Checksums

和`Sizers`相似，`s_checksum()`助手使用`block`的名称作为其第一个参数来确定该`block`的checksum，如下的可选参数可被指定：

- `algorithm`: (string or function pointer, default="crc32"). 应用于目标块的checksum的算法(`crc32`, `adler32`, `md5`, `sha1`)
- `endian`: (character, default='<') bit 位序 指定 `'<'` 为小端，`'>'`为大端
- `length`: (integer, default=0) 校验和的长度，指定为`0`则自动计算
- `name`: (string, default=None) 可以通过在`request`中指定该名称来直接存储该原语的值

`algorithm`参数能被指定为`"crc32"`, `"adler32"`, `"md5"`或`"sha1"`其中之一，或者，你可以指定一个函数指针，作为你自己定制的算法。

## Repeaters

`s_repeat()`或者`s_repeater()`助手，被用于重复一个块为一个变量所代表的次数。这对于测试需要解析一个表为多个元素的溢出案例是非常有用的。

这个辅助程序需要携带三个必须参数，被重复的`block`的名称，最小重复次数和最大重复次数，此外，下面的可选参数可使用：

- `step`: (integer, default=1) 最小，最大重复次数的步长
- `fuzzable`: (boolean, default=False) 该原语可fuzzing性的开关
- `name`: (string, default=None) 可以通过在`request`中指定该名称来直接存储该原语的值

考虑下下面这个例子：

它连接了三个我们所介绍的辅助程序，我们`fuzz`的协议的一部分是一个包含字符串的表格。每一个表格中的项由两个字节的字符串类型域，两个字节的长度域，一个字符串域，最后以一个需要接受整个字符串域的crc-32校验和结尾。我们并不知道对这个有效的类型域是什么类型，所有我们使用随机数据来对其进行`fuzz`。使用suelly定义的这部分协议如下：

``` 
# table entry: [type][len][string][checksum]
if s_block_start("table entry"):
    # we don't know what the valid types are, so we'll fill this in with random data.
    s_random("\x00\x00", 2, 2)

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

这个`sulley`脚本不仅可以发现解析表项时的错误，而且可能发现在处理超长表时的错误。

## Legos

`sulley`使用`legos`来代表用户定义的组件，诸如，`e_mail`地址，主机名，以及用于微软RPC，XDR，ASN.1等协议原语，在`ASN.1`中，`/BER`字符串被描述为一个系列`[0x04][0x84][dword length][string]`，在`fuzzing`基于`ASN.1`的协议时，在每一个字符串前面都包括一个长度和类型的前缀，将变得相当难处理，作为代替，我们定义该`lege`来引用它。

``` 
s_lego("ber_string", "anonymous")
```

除了可选择的可选关键字参数外，每一个`lego`都有相似的格式，来指定一个独立的`lego`。作为一个例子，考虑到`tag` `lego`的定义，当`fuzz` xml-ish协议时是非常有用的：

``` 
class tag(blocks.block):
    def __init__(self, name, request, value, options={}):
        blocks.block.__init__(self, name, request, None, None, None, None)
        self.value = value
        self.options = options

        if not self.value:
            raise sex.error("MISSING LEGO.tag DEFAULT VALUE")

        #
        # [delim][string][delim]
        self.push(primitives.delim("<"))
        self.push(primitives.string(self.value))
        self.push(primitives.delim(">"))
```

这个`lego`例子接受一个期望的`tag`作为字符串，并且将其压缩在适当的分割符内，如此做扩展了`blocks`类，通过`self.push()`将合适的分割符和字符串添加到`block`的栈中。

这里是另外一个为了生成ANS.1/DER整数的简单`lego`，最小的共同点是代表的整数都是4字节整数，并有如下的形式，`[0x02][0x04][dword]`：`0x02`指定整数的类型，`0x04`指定整数的长度是4字节长，`doword`代表我们实际传入的值，如下是在`sulley`中的定义：

``` 
class integer(blocks.block):
    def __init__(self, name, request, value, options={}):
        blocks.block.__init__(self, name, request, None, None, None, None)
        self.value = value
        self.options = options

        if not self.value:
            raise sex.error("MISSING LEGO.ber_integer DEFAULT VALUE")

        self.push(primitives.dword(self.value, endian=">"))

    def render(self):
```
