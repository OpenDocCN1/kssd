
# [原创] CTF2017 第 5 题分析

## 1. 确定大概流程

用 OD 启动，发现有反调试，直接尝试先屏蔽驱动相关的。

![](https://bbs.kanxue.com/upload/attach/201706/44250_5fprl7bgyw5ojpl.png)

运行起来后，断在`GetWindowTextA`，可以得到输入字符串，然后通过内存访问断点，可以定位到字符串处理和最后的比较点。

![](https://bbs.kanxue.com/upload/attach/201706/44250_rquftkf9ixpy5rn.png)

因为屏蔽了驱动，可以对照 IDA 看看这个函数流程。

这里是通过`WriteFile`传入倒序后的字符到驱动，然后通过`ReadFile`读取返回数据。

可以直接跳过驱动操作，直接跳到`401E78`，看看对返回数据的处理。

![](https://bbs.kanxue.com/upload/attach/201706/44250_q0xbvoa74u544z9.png)

先是对驱动返回的`0x10`数据转字符串，然后再进行 MD5，再转字符串，取第 3 个字符开始的 10 个字符和目标串比较。

（这里，根据传入数据和输出数据，就可以猜测是 MD5 了，可以直接验证下，都不用具体看算法了，当然里面也有比较明显的特征）

## 2. 分离驱动文件

通过断在`DeleteFileA`，或者通过找驱动加载相关的字符串引用点，得道驱动文件`vmxdrv.sys`。

IDA 分析，确定主要的处理点。

![](https://bbs.kanxue.com/upload/attach/201706/44250_v99siojojjp9up3.png)

直接看`WriteFile`的处理，关键点`sub_104B6`。

![](https://bbs.kanxue.com/upload/attach/201706/44250_mztqvakxdxzmdqs.png)

`dword_114D8`是操作码`222004`会影响的变量，交叉引用就可以看到对应设置。

这里是对传入字符串各个字符加了对应增量，如`654321`变为`766666`，然后 MD5 后返回。

## 3. 枚举

最近对`node.js`有点兴趣，尝试写了个函数跑了下。

```javascript
var fs = require('fs');

var path = require('path');

var Crypto = require('crypto');

const util = require('util');

function ctf_5() {
    var patternBuf = new Buffer("pqrstuvwxyzabcdefghijklmno0123456789");

    var strIn = "123456";

    var buf = new Buffer(7);

    var iLen = patternBuf.length;

    console.log("iLen: " + iLen);

    for (var a1 = 0; a1 < iLen; a1++) {
        for (var a2 = 0; a2 < iLen; a2++) {
            for (var a3 = 0; a3 < iLen; a3++) {
                for (var a4 = 0; a4 < iLen; a4++) {
                    for (var a5 = 0; a5 < iLen; a5++) {
                        for (var a6 = 0; a6 < iLen; a6++) {
                            var buf = new Buffer([patternBuf.readUInt8(a1) + 1, patternBuf.readUInt8(a2) + 1,
                                patternBuf.readUInt8(a3) + 2, patternBuf.readUInt8(a4) + 3,
                                patternBuf.readUInt8(a5) + 4, patternBuf.readUInt8(a6) + 5]);

                            strIn = buf.toString();

                            var md5sum = Crypto.createHash('md5');

                            md5sum.update(strIn);

                            var str = md5sum.digest('hex');

                            // 对结果串再 MD5
                            var md5sum2 = Crypto.createHash('md5');

                            md5sum2.update(str);

                            var str2 = md5sum2.digest('hex');

                            // 从第 3 个字符开始的 10 个字符，和字符串 ASCII "888aeda4ab" 比较
                            var subStr = str2.substring(2, 2 + 10);

                            if (subStr == "888aeda4ab") {
                                console.log(str2);

                                console.log("result：" + strIn);

                                return;
                            }
                        }
                    }
                }
            }
        }
    }

    console.log("done");
}

ctf_5();
```

运行结果：

`result：79;4yx`

对应原输入`6891us`，倒序就是`su1986`。

想快点，可以开多个终端分段跑。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
