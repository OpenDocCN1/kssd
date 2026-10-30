
![](upload/attach/202009/779730_UF9C9A9SRPKSB4X.png)

至此为止，漏洞的完整Root Case已分析完成。

### 4. 利用思路

#### 1. 利用条件

1. 基本条件

- `attacker`需要将恶意的LNK文件发送给`target`或者`target`可以正常访问LNK

2. 触发过程

- `attacker`需要诱使`target`访问包含LNK文件的路径

#### 2. 利用过程

`attacker`发送一个LNK文件给`target`：

```
[ Attacker ] <--------------------> [ Target ]
```

#### 3. 利用界面

应用协议：

1. `HTTP`，端口：`80 TCP`

2. `HTTPS`，端口：`443 TCP`

3. `SMB/CIFS`，端口：`139/445 TCP`

4. `IMAP`，端口：`143 TCP`

5. `POP3`，端口：`110 TCP`

6. `SMTP`，端口：`25 TCP`

#### 4. 攻击向量

已知文件类型：Shell Link

已知扩展名：LNK

已知MIME类型：`application/x-ms-shortcut`

### 5. 流量分析

从常规漏洞防御角度来说，文件格式类漏洞更适合使用终端防护软件进行防护。但该漏洞的主要应用场景应该是`attacker`诱骗`target`打开LNK文件，这样就会涉及到流量传输，在一定程度上可以对恶意流量进行检测：

1. `attacker`诱骗`target`请求恶意LNK文件：

![](upload/attach/202009/779730_D62M65PBNBB6Z86.png)

2. `attacker`直接响应一个恶意LNK（注意分包）：

![](upload/attach/202009/779730_TTHE2Z8NYVU45HD.png)

![](upload/attach/202009/779730_MKRYF2638C6DF53.png)

### 6. 补丁分析

#### 1. 补丁比较结果

`StructuredQuery1::ReadPROPVARIANT()`函数的补丁对比结果：

![](upload/attach/202009/779730_MC9YSCVJCZ37B6T.png)

总体上看，函数并没有做过多修改，进入查看具体更新内容。当`case`为`12`(`VT_VARIANT`类型)时，有一处比较明显的改动。在进行24字节的缓冲区分配后，对分配的缓冲区做了赋值0操作：

![](upload/attach/202009/779730_FH8MUT4B8XD4EKU.png)

#### 2. 补丁思路

在更新前，对分配的`buffer`未做任何的初始化操作，直接进入了递归调用：

![](upload/attach/202009/779730_RR5NSNJJKPD6HC8.png)

在更新后，对分配完的`buffer`做了初始化操作（3次赋值为0，总计24字节）：

![](upload/attach/202009/779730_WUCKA97RFWSZTXD.png)

所以补丁的思路很简单，将分配的24字节的`buffer`初始化为0即可。

#### 3. 补丁验证

通过动态调试，证明了这种补丁思路：

![](upload/attach/202009/779730_4NYFZ4WTFQUE8KD.png)

进行了3次`mov`操作后，将24个字节的`buffer`成功初始化为0：

![](upload/attach/202009/779730_CD5ZWGS5K3ZAH3Z.png)

使用初始化为0后的`buffer`进入函数递归：

![](upload/attach/202009/779730_MFHM3JXG5PYP8D3.png)

继续调用`IStream_Read()`函数进行读取，此时的参数情况如下：

![](upload/attach/202009/779730_KZ2BJF87JSZ377K.png)

继续向下跟，会发现并没有在`IStream_Read()`函数中产生补丁前的那种情况，而是执行完后，跳到异常处理，函数返回，而不进行内存分配：

![](upload/attach/202009/779730_T2W79RP4WZXDHMV.png)

## 四、漏洞检测和防御

### 1. 漏洞检测

针对该漏洞，目前很难做到无损检测。

### 2. 漏洞防御

#### 1. 防御思路

流量防御：流量侧的防御可以检测恶意数据的具体内容，但需要联系数据上下文，因为该漏洞的本质是多种数据结构的组合使用导致异常。流量特征较弱，不是很建议使用流量防御。

终端防御：使用热补丁，对24个字节的`buffer`使用0进行初始化即可。

#### 2. 可能存在的风险

终端防御方法基本没有风险，但是流量侧误报风险较大。

## 五、参考文献

1. <a href="elink@873K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6s2y4Z5L8r3I4A6L8X3E0Q4x3V1j5I4y4X3y4T1y4r3y4S2x3g2)9J5k6o6V1K6x3K6W2Q4x3X3b7@1k6o6m8U0i4K6u0V1j5e0j5^5k6q4)9J5k6r3u0X3x3h3b7$3j5$3x3H3k6U0V1@1x3H3`.`.">Shell Link(.LNK) Binary File Format</a>

2. <a href="elink@09aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3I4W2k6$3q4U0P5g2)9J5c8X3y4U0x3e0b7@1x3o6R3&6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">Common Explorer Concepts, Identifying Namespace Objects</a>)

3. <a href="elink@157K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6r3!0D9k6i4m8K6i4K6u0r3k6U0p5J5x3X3t1&6k6o6N6Q4x3X3c8W2y4h3y4X3i4K6u0V1y4o6b7^5y4q4)9J5k6o6R3@1y4U0k6Q4x3X3b7^5x3$3j5$3k6X3b7&6y4r3t1K6j5$3x3`.">MS-OLEPS: TypedPropertyValue</a>

4. [MS-OLEPS: IndirectPropertyName](<mark class="encrypted">832K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3!0H3k6h3&6K6M7r3g2U0M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4y4h3k6H3M7X3!0@1L8$3y4G2L8s2y4Q4x3V1k6E0M7#2)9J5k6r3!0D9k6i4m8K6i4K6u0r3k6e0y4T1k6r3t1$3x3o6S2Q4x3X3b7@1y4K6f1$3i4K6u0V1y4r3p5H3j5#2)9J5k6r3t1J5k6X3c8Q4x3X3b7I4y4h3t1%4k6h3g2W2y4e0t1&6x3o6f1`.</mark> )

5. [Structured Storage, Propidl.h, PROPVARIANT structure](<mark class="encrypted">c74K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8R3`.`.</mark> propidl/ns-propidl-propvariant )

## 六、备注

看雪论坛内部共享，未经允许请勿擅自转载。谢谢。

[CVE-2020-0729：Windows LNK远程代码执行漏洞分析（一）](https://bbs.pediy.com/thread-262082.htm)

（接前文，活动结束了，终于有时间把这个漏洞分析写完了。友情提示，在调试过程中，尽量只开一个poc的文件夹，不然可能要递归调用漏洞函数很多次。次数一多，就容易乱了。）

### 3. 详细分析

#### 1. 漏洞触发流程

造成该漏洞的主要原因是在处理`Leaf Condition`中的`type`为`VT_VARIANT`的`PropertyVariant`结构时发生错误导致。

如果一个LNK文件有一个`ItemID List`，且其第一个`ItemID`为`Delegate Folder ItemID`，且该`Delegate Folder ItemID`d的GUID表示`CLSID_SearchFolder`，那么首先调用`Windows.Storage.Search.dll`中的`CDBFolder::BindToObject()`函数进行处理，该函数会调用`CDBFolder::GetFilterConditionForChild()`函数从`child ItemID`中获取`search filter condition`。

第一阶段，`CDBFolder::GetFilterConditionForChild()`函数在进行查找时，会在`child ItemID`中搜索`PKEY_FilterInfo` `property`，如果找到了，在序列化`property store`中的包含一个`property bag`的`VT_STREAM`就会通过调用`SHLoadFilterFromStream()`函数进行加载，该函数会创建一个`CFilterCondition`对象然后提供给`IUnknown_LoadFromStream()`函数。该操作会调用`CFIlterCondition::Load()`函数，而该函数会首先将`property bag`复制到一个内存中的`store`中，然后开始校验该`store`的结构，通过查找名称为`Name`, `Type`,`key:FMTID`,`Key:PID`以及`Condition`的字符串来确认是否`property bag`元素。

第二阶段，在确认完所有元素后，调用`LoadConditionFromStream()`函数来读取`Condition` `property bag`中的`VT_STREAM`，该元素调用`IUnknown_LoadKnownImplFromStream()`来读取`Condition` GUID并从`StructuredQuery.dll`中创建并加载相关的`condition`对象。在从`stream`中加载`condition`对象的过程中，所有嵌套的对象会按照出现的顺序进行加载，包括`Attributes`和`Conditions`。当遇到一个`Leaf Condition`时，会调用`StructuredQuery1::LeafCondition::Load()`函数来读取所有的`Attributes`，然后读取`Condition Type PKEY`和`Condition Operation`，然后调用`StructuredQuery1::ReadPROPVARIANT()`函数来读取`PropertyVariant`结构。

第三节阶段，`StructuredQuery1::ReadPROPVARIANT()`先读取2字节的`type`，然后检查`VT_ARRAY(0x2000)`是否进行了设置，这里主要是因为`StructuredQuery`不支持。然后进入到一个`switch`语句来根据`type`进行不同的处理。如果`type`设置为`VT_VARIANT(0x000c)`，就会检查完整的`type`字段来确认是否设置了`VT_VECTOR`，如果没有设置，则调用`CoTaskMemAlloc()`来分配包含另一个`PropertyVariant`结构的24字节的缓冲区。在递归调用`StructuredQuery1::ReadPROPVARIANT()`函数以读取紧随`VT_VARIANT` 的`type`字段之后的另一个`PropertyVariant`结构之前，并没有对这个缓冲区进行初始化。如果下一个`type`字段设置为`VT_CF(0x0047)`，则应该是一个包含一个指向剪贴板数据的指针的`PropertyVariant`结构，`ReadPROPVARIANT()`函数尝试将`stream`的后4个字节写入先前分配的24字节缓冲区中的8字节值所指向的位置。由于缓冲区并没有进行初始化，数据会被写入到一个未定义的位置，最终导致任意代码执行。

用一个简单的流程图来描述上面的流程：

![](https://bbs.kanxue.com/upload/attach/202009/779730_6QX66WCYQTZP862.png)

![](https://bbs.kanxue.com/upload/attach/202009/779730_7DWTU9WW7SSYKDR.png)

#### 2. 静态分析

首先是`StructuredQuery1::LeafCondition::Load()`函数。在`StructuredQuery1::LeafCondition::Load()`函数中依次读取了`Condition Type PKEY`，`Condition Operation`，然后调用`StructuredQuery1::ReadPROPVARIANT()`函数读取`PorpertyVariant`结构：

![](https://bbs.kanxue.com/upload/attach/202009/779730_FQWQZDKSK7EV4KE.png)

![](https://bbs.kanxue.com/upload/attach/202009/779730_D6UBCB429JM4YSZ.png)

进入到`StructuredQuery1::ReadPROPVARIANT()`函数内部：

![](https://bbs.kanxue.com/upload/attach/202009/779730_33R4URK8SWW8MK4.png)

这里首先读取`variant`的类型，然后进行`VT_ARRAY`检查，再然后检测`type`是否为`0x13`，而`0x13`为`VT_UI4`的`code`。如果不是，进入到后续的`switch`结构的处理流程：

![](https://bbs.kanxue.com/upload/attach/202009/779730_HG9HD2S852GFGH5.png)

`VT_CF`的`code`为71，该类型表示指向`CLIPDATA`结构的一个指针，`Propvaritant type`为`pclipdata`。`CLIPDATA`的详细结构如下（与一般的PROPVARIANT略有不同）：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `typedef struct  tagCLIPDATA {`  `/``/` `cbSize` `is` `the size of the` `buffer` `pointed to`  `/``/` `by pClipData, plus sizeof(ulClipFmt)`  `ULONG              cbSize;`  `long`               `ulClipFmt;`  `BYTE``*`              `pClipData;`  `} CLIPDATA;` |

`VT_CF`时的处理代码如下：

![](https://bbs.kanxue.com/upload/attach/202009/779730_CS6KJJSRUTGCBPK.png)

如果没有设置`VT_VECTOR`，则跳转到如下代码。因为在处理`VT_CF`时，对分配的缓冲区并没有做任何的初始化处理，然后直接将`buffer`的地址传入了`rdx`，并且向`rdx`的地址中写入了4个字节：

![](https://bbs.kanxue.com/upload/attach/202009/779730_P6VTPXPG3TAXQCB.png)

`VT_VARIANT`为一个`DWORD`类型的标识后续值的类型的标识符，仅可以与`VT_VECTOR`和`VT_BYREF`结合使用。`VT_VARIANT`时的处理代码如下：

![](https://bbs.kanxue.com/upload/attach/202009/779730_YKYTR5U2FCQC454.png)
