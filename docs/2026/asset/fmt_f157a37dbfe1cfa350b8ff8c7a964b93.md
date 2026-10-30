
# CVE-2017-11826 漏洞分析及利用

作者：Joey@天玄安全实验室

## 前言

最近开始分析 Office 漏洞，拿到 CVE-2017-11826 的样本后发现无法在 Office2010 上成功执行，打算分析并改造该 EXP。参考了许多资料，结合自己的理解写了本文，供大家学习和参考。

## 漏洞分析

### 分析环境

```
OS: Win7 x64 SP1
Office: Ofiice 2010 x86
Image name: wwlib.dll
Timestamp: Sat Mar 27 23:37:07 2010 (4BAE2623)
CheckSum: 0127F568
ImageSize: 0127A000
File version: 14.0.4762.1000
Product version: 14.0.4762.0
```

### 静态分析

在 rtf 文档中搜索 object，发现嵌入了 3 个 ole 对象：

![image-20210727171412503](upload/attach/202108/929902_8A6ZZ794KDY5V3W.jpg)

第一个对象的 CLSID 为`D5DE8D20-5BB8-11D1-A1E3-00A0C90F2731`，在注册表搜索后发现该对象位于`C:\Windows\SysWOW64\msvbvm60.dll`，而该 dll 是没有 ASLR 的。

![image-20210727175112215](upload/attach/202108/929902_7GV6PC4VTQBSJFZ.jpg)

通过 ProcessExplorer 发现 word 打开 rtf 文档后确实加载了`msvbvm60.dll`，且该 dll 无 ASLR，说明该 ole 对象的作用是绕过 ASLR。

![image-20210727180558328](upload/attach/202108/929902_9JSYFXA2FYAZR9T.jpg)

使用`rtfobj.py -s all`提取 ole 对象：

![image-20210727165632756](upload/attach/202108/929902_RMK8FXSXMAH5AUS.jpg)

第一个对象经过上面的分析是用于绕过 ASLR 的，第二和第三个都是 .doc 文档，使用压缩软件直接打开第二个文档，文档结构如下：

```
│ [Content_Types].xml
│
├─docProps
│ app.xml
│ core.xml
│
├─word
│ │ document.xml
│ │ fontTable.xml
│ │ settings.xml
│ │ styles.xml
│ │ webSettings.xml
│ │
│ ├─activeX
│ │ │ activeX1.bin
│ │ │ activeX1.xml
│ │ │ activeX2.xml
│ │ │ ······
│ │ │ activeX40.xml
│ │ │
│ │ └─_rels
│ │ activeX1.xml.rels
│ │ activeX2.xml.rels
│ │ ······
│ │ activeX40.xml.rels
│ │
│ ├─media
│ │ image1.wmf
│ │
│ ├─theme
│ │ theme1.xml
│ │
│ └─_rels
│ document.xml.rels
│
└─_rels
.rels
```

可以看出使用了 40 个 activeX.xml 文件，文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<ax:ocx ax:classid="{00000000-0000-0000-0000-000000000001}" ax:persistence="persistStorage" r:id="rId1" xmlns:ax="http://schemas.microsoft.com/office/2006/activeX" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"/>
```

40 个 xml 文件内容一致，加载了 CLSID 为`{00000000-0000-0000-0000-000000000001}`的对象，然而系统中并没有这个对象，所以并不会加载任何对象，这么做是为了提高堆喷的效率，具体原理可查看<a href="elink@ed9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4M7X3g2&6K9r3q4@1K9r3q4U0K9$3g2J5i4K6u0W2L8X3g2@1i4K6u0r3i4K6y4r3M7q4)9K6c8o6V1I4x3b7`.`.">SPRAYING THE HEAP IN SECONDS USING ACTIVEX CONTROLS IN MICROSOFT OFFICE</a>一文。

而 40 个 activeX.xml.rels 的内容也完全一致：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
<Relationship Id="rId1" Type="http://schemas.microsoft.com/office/2006/relationships/activeXControlBinary" Target="activeX1.bin"/>
</Relationships>
```

都指向了`activeX1.bin`文件，因此会将`activeX1.bin`在内存中加载 40 次，以此达到堆喷的目的。

`activeX1.bin`文件结构如下：

```
activeX1.bin
│ -文件头
│ -数据
│ │---CB 40 94 72 EC 83 88 08 CB 40 94 72 EC 83 88 08
│ │ ······
│ │---CB 40 94 72 EC 83 88 08 CB 40 94 72 EC 83 88 08
│ │---shellcode
│ │---2B 0E 98 72 2B 0E 98 72 2B 0E 98 72 2B 0E 98 72
│ │ ······
│ │---2B 0E 98 72 2B 0E 98 72 2B 0E 98 72 2B 0E 98 72
│ │ ······
│ │---CB 40 94 72 EC 83 88 08 CB 40 94 72 EC 83 88 08
│ │ ······
```

看结构似乎是滑板指令加 shellcode，待调试验证。

第三个文档结构如下：

```
│ [Content_Types].xml
│
├─docProps
│ app.xml
│ core.xml
│
├─word
│ │ document.xml
│ │ endnotes.xml
│ │ fontTable.xml
│ │ footnotes.xml
│ │ settings.xml
│ │ styles.xml
│ │ webSettings.xml
│ │
│ ├─theme
│ │ theme1.xml
│ │
│ └─_rels
│ document.xml.rels
│
└─_rels
.rels
```

`document.xml`的内容如下：

![image-20210804144955166](upload/attach/202108/929902_C9D9YWYGVNBR8TV.jpg)

观测到`<w:font`标签内有异常字符，且标签未正常闭合，预测漏洞触发于该处。

通过静态分析了解到 RTF 文档通过内嵌 3 个 ole 对象来实现 ASLR 绕过、堆喷射和漏洞触发，ASLR 绕过是通过加载 CLSID 为`D5DE8D20-5BB8-11D1-A1E3-00A0C90F2731`的 COM 对象，将`msvbvm60.dll`加载到内存中。堆喷射利用 40 个`activeX.xml.rels`指向唯一的`activeX1.bin`文件，将`activeX1.bin`文件中的数据部分，即偏移为`0x800`后的内容加载到内存中实现堆喷射。而漏洞触发部分则利用`document.xml`中的异常字符和标签触发漏洞。

### 动态调试

使用 windbg 附加 word，打开漏洞文件：

![image-20210727162527153](upload/attach/202108/929902_89U58XUPXPQHC3Y.jpg)

可以看到异常因为`ecx+4`指向的内存无法访问导致错误。查看反汇编得知`ecx`的值来源于`eax`，此时`eax`的值为`088888ec`。再次打开漏洞文件发现`ecx`的值改变，但是`eax`的值仍为`088888ec`，说明`eax`的值为故意构造。

于是打算下断在函数`wwlib!DllGetClassObject+0x42d4 (71ed98b0)`查看`eax`是如何生成的。查看 wwlib 的基地址，算出函数的偏移为`wwlib+004da16b`。

```
0:000> lm m wwlib
start end module name
71ed0000 7314a000 wwlib (export symbols) C:\PROGRA~2\MICROS~1\Office14\wwlib.dll
0:000> ? 723aa16b-71ed0000
Evaluate expression: 5087595 = 004da16b
```

重新打开漏洞文档，`bp wwlib+004da16b`下断：

![image-20210802112402518](upload/attach/202108/929902_R4MB5FXTY3W3YYJ.jpg)

步过两次后执行到如图所示位置时，查看`eax`所在的内存：

![image-20210802112612050](upload/attach/202108/929902_5R88PQJHBA9J4SY.jpg)

发现和在文档 3 中的字符串一致，接着查看`eax+44`，对应的正是异常触发时`eax`的值`088888ec`。

![image-20210802112936829](upload/attach/202108/929902_PPTMZ3Q4WDRRXMT.jpg)

但在 xml 文件中，字符串中的异常字符的十六进制为`e8a3ace0a288`：

![image-20210802140720647](upload/attach/202108/929902_WE7GABYNDU9ENS7.jpg)

在文件中显示的格式是 Ascii，然而在内存中显示的是 Unicode，于是将文件中的字符以 utf-8 格式转换为十六进制正是`eax`的值`088888ec`：

![image-20210802142105409](upload/attach/202108/929902_9ZN3E2QMQWVASR4.jpg)

说明通过修改该字符串可以控制`eax`的值，进而控制`eip`。

在 ida 中找到崩溃函数为`sub_31A55CE6`，发现变量`v3`是宽字节字符串，位于`arg2+0x18`，变量`v4`是一个长度，位于`arg2+0x1c`：

![image-20210802153650508](upload/attach/202108/929902_ZDHPHSETABF466F.jpg)

在 windbg 设置崩溃函数起始点打印`v3`为字符串，长度为`v4`：`bp wwlib+385ce6 "du poi(poi(esp+8)+18) Lpoi(poi(esp+8)+1c); g;"`

![image-20210802161250939](upload/attach/202108/929902_DGC8M7U33H978W4.jpg)

可以看到`v3`就是 xml 文件中的标签，在解析到 idmp 标签后程序崩溃，然而并没有看到 font 标签，于是寻找到崩溃函数的父函数`sub_3170FA5A`：

![image-20210802163044663](upload/attach/202108/929902_XCZW2CGQGP54YC2.jpg)

崩溃函数`arg2`的值为`edi`，而`edi`的值为父函数的`arg2`：

![image-20210802165001758](upload/attach/202108/929902_KQFP4N86Q9MPF5A.jpg)

于是在父函数和崩溃函数同时下断，查看标签解析情况：

```
bp wwlib+3fa5a ".printf \"Parent_Func: \"; du poi(poi(esp+8)+18) Lpoi(poi(esp+8)+1c); g;"
bp wwlib+385ce6 ".printf \"Crash_Func: \"; du poi(poi(esp+8)+18) Lpoi(poi(esp+8)+1c); g;"
```

![image-20210802170632918](upload/attach/202108/929902_E8GDERYR7A3ENFD.jpg)

在父函数成功解析到 font 标签，猜测因为 font 标签未闭合而导致崩溃函数解析标签出错产生漏洞，修改了 xml 文件闭合了 font 标签：

![image-20210804145041151](upload/attach/202108/929902_E8STYDT3UZ7M686.jpg)

将修改后的 docx 文件嵌入到新建的 rtf 文件中，在 windbg 中调试后发现`eax`的值改变了，并且没有异常，证实因为 font 标签未闭合导致的漏洞。

![image-20210802180616278](upload/attach/202108/929902_28AU3DTMXHWJRXN.jpg)

继续调试发现异常触发点的`eax`和`ecx`都是来自于`esi`，而`esi`为漏洞函数的`arg1`：

![image-20210803145832128](upload/attach/202108/929902_5TA9CTWJKGC3ND6.jpg)

因此在漏洞函数打印标签以及`[[esi+17f0]]`、`[[esi+17f0]+8]`、`[[esi+17f0]+c]`和`[esi+17f0]`的值：

```
bp wwlib+385ce6 "du poi(poi(esp+8)+18) Lpoi(poi(esp+8)+1c); r $t0=poi(poi(esp+4)+17f0); dd poi($t0) L1; dd poi($t0)+8 L1; dd poi($t0)+c L1; dd $t0 L1; .printf\"\n\"; g;"
```

![image-20210803162436403](upload/attach/202108/929902_2XXYVNBBXWKTNZC.jpg)

打印出的结构就是 Taglist 结构体，具体结构参考 goabout2 的<a href="elink@c13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0S2j5X3!0#2N6o6u0Q4x3V1k6H3i4K6u0r3z5o6p5^5y4U0l9I4z5q4)9J5k6h3S2@1L8h3H3`.">office CVE-2017-11826 杂谈</a>一文。

接着调试异常触发点上的函数，发现函数功能为通过层级标签获取`TagObject Array[Index-2]`：

![image-20210803192637477](upload/attach/202108/929902_QPVR36UP88RKWQX.jpg)

继续向上追溯，发现函数`GetTagObject`也调用了`GetTagObjectByIndex`，通过分析发现该函数获取的是`TagObject Array[Index-1]`的地址：

![image-20210803193709068](upload/attach/202108/929902_GXSB8HCGCUVZCRQ.jpg)

分析到这里，漏洞产生的原因也就出来了，由于 word 每解析一个标签，`Current_Index`的值就加一，当解析到闭合标签，`Current_Index`值会减 1。由于构造了没有闭合的 font 标签，因此导致在解析 idmap 标签时比正常文件的`Current_Index`多一，导致原本应该获取 OLEObject 标签的 TagObject 变成获取了 font 的 TagObject，因此造成了标签类型混淆导致漏洞的发生。

将标签层级和 xml 文件标签对应：

![image-20210803164645125](upload/attach/202108/929902_XTKBAWPJ723BYQR.jpg)

可以证实确实因为`Current_Index`值比正常文件的多一导致的类型混淆。

在内存中查看当解析 idmap 层级为 6 时 Taglist 的内存结构：
