
![02](https://bbs.kanxue.com/upload/attach/202212/674195_3VAYPCUQTDB5ANY.png)

每个`asn1`类型被定义为一个结构体节点，一开始是无法准确地知道每个字段的含义。通过相关查找，确认这些节点都是在github上的开源代码[`asn1c-0.9.26`](https://bbs.kanxue.com/elink@882K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3L8r3#2Q4x3V1k6S2M7$3^5I4j5#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6$3x3q4)9J5k6e0W2Q4x3X3f1J5y4R3`.`.)中定义。于是，我们对这些`asn1`节点有了较为清晰的了解。

![01](https://bbs.kanxue.com/upload/attach/202212/674195_5YNYM7V8BSC3GDZ.png)

## 获取所有asn1节点

通过观察`asn1`结构体节点，可以发现一般情况下，`name`字段和`xml_tag`字段指向同一个字符串，并且所有的节点都在`data`段内。我们可以在`data`段找两个即相邻又相同的指针，并且该指针指向的内容同样在`data`段内。节点之间通过`elements`字段关联，利用`elements`，我们可以为这些节点建立树状关系或层级关系，甚至可以恢复出整个`asn1`定义。下图`LIF`的`asn1`定义。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79 | `LIF-CONTENT DEFINITIONS ::= BEGIN` `LIF-Content ::= [PRIVATE 228] IMPLICIT SEQUENCE` `{` `sw-specs[PRIVATE 161] IMPLICIT Software-Specs,` `clocks[PRIVATE 100] IMPLICIT Clocks,` `license-description[PRIVATE 125] IMPLICIT License-Description,` `binding[PRIVATE 136] IMPLICIT Binding-Scheme,` `cmact-options[PRIVATE 133] IMPLICIT CmAct-Options,` `mask[PRIVATE 142] IMPLICIT Mask OPTIONAL` `}` `Software-Specs ::= SEQUENCE` `{` `creator-name[PRIVATE 181] IMPLICIT UTF8String,` `creator-version[PRIVATE 165] IMPLICIT Format-Version,` `required-version[PRIVATE 122] IMPLICIT Format-Version` `}` `Format-Version ::= SEQUENCE` `{` `sfl[PRIVATE 78] IMPLICIT UInt8,` `sfh[PRIVATE 84] IMPLICIT UInt8,` `feature-flags[PRIVATE 32] IMPLICIT UInt32` `}` `UInt8 ::= [PRIVATE 31] IMPLICIT INTEGER` `UInt32 ::= [PRIVATE 34] IMPLICIT INTEGER` `Clocks ::= SEQUENCE` `{` `box-time[PRIVATE 178] IMPLICIT Seconds-Since-2000,` `certified-time[PRIVATE 179] IMPLICIT Seconds-Since-2000` `}` `Seconds-Since-2000 ::= INTEGER` `License-Description ::= SEQUENCE` `{` `license-description[PRIVATE 30] IMPLICIT OCTET STRING,` `-- BMPString, it does't work well in asn1tools, beacase asn1tools only support utf-8, but here is utf-16` `firm-code[PRIVATE 114] IMPLICIT Firm-Code` `}` `Firm-Code ::= INTEGER` `Binding-Scheme ::= SEQUENCE` `{` `cmact-id[PRIVATE 148] IMPLICIT CmAct-ID,` `telephone-id[PRIVATE 127] IMPLICIT Telephone-ID,` `binding-method[PRIVATE 128] EXPLICIT Binding-Method` `}` `CmAct-ID ::= IA5String` `Telephone-ID ::= INTEGER` `Binding-Method ::= CHOICE` `{` `smartbind[PRIVATE 129] IMPLICIT SmartBind-Parameters,` `custom-binding[PRIVATE 131] IMPLICIT CustomBinding-Parameters,` `dcbn-classic[PRIVATE 132] IMPLICIT DCBN-Parameters,` `nonebind[PRIVATE 231] IMPLICIT NULL,` `cm-server-ip[PRIVATE 232] IMPLICIT NULL,` `serial-number[PRIVATE 233] IMPLICIT NULL,` `random[PRIVATE 234] IMPLICIT NULL` `}` `SmartBind-Parameters ::= SEQUENCE` `{` `heuristic[PRIVATE 31] IMPLICIT UInt8,` `redundancy-level[PRIVATE 130] IMPLICIT Redundancy-Level` `}` `Redundancy-Level ::= INTEGER` `CustomBinding-Parameters ::= IA5String` `DCBN-Parameters ::= SEQUENCE` `{` `disk[PRIVATE 1] IMPLICIT BOOLEAN,` `cpu-type[PRIVATE 1] IMPLICIT BOOLEAN,` `board[PRIVATE 1] IMPLICIT BOOLEAN,` `network[PRIVATE 1] IMPLICIT BOOLEAN,` `tolerance[PRIVATE 2] IMPLICIT UInt8 OPTIONAL` `}` `CmAct-Options ::= SEQUENCE` `{` `allow-vm[PRIVATE 219] IMPLICIT BOOLEAN,` `allow-reimport[PRIVATE 220] IMPLICIT BOOLEAN` `}` `Mask ::= INTEGER` `END` |

## 解析asn1码流

有了这些节点和节点关系，我们就可以用它来解析码流了。接下来我们就来看看`LIF`文件里面到底有什么东西。先说结论吧，`LIF`文件包含一段`LIF-content`、两个证书和一段签名值。其中`LIF-content`包含一些授权信息和一些限制条件。两份证书和签名值用于验证`LIF`文件是否被修改。

以下是我使用`asn1tools`解析的`Terra2314.WibuCmLIF`（这个`LIF`我都不记得是从哪里弄来的，见附件）

|  |  |
| --- | --- |
