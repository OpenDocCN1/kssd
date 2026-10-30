
| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Value Size(x)` |
| `0x0004` | 4 | `Name Size(y)` |
| `0x0008` | 1 | `Reserved(0x00)` |
| `0x0009` | `y` | `Name (wide characters)` |
| `0x0009 + y` | `x - y - 9` | `TypePropertyValue` |

`Property bags`通常包含类似名称为`"Key:FMTID"`或`"Key:PID"`的内容，这些名称表示了解释其他元素的特定的`PKEY`。

如果`Property Format GUID`不是前面提到的`Property Bags`形式的GUID，那么每个property将会使用一个整数形式的`PID`进行标识，其结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Size(x)` |
| `0x0004` | 4 | `Id` |
| `0x0008` | 1 | `Reserved(0x00)` |
| `0x0009` | `x - 9` | `TypePropertyValue` |

`TypePropertyvalue`对应于property集中的property的类型值，property集遵循 Microsoft Object Linking and Embedding (OLE) Property Set Data Structures 中的规范(详见参考文献3)。

#### 4. `PKEY_FilterInfo`

各种`PKEY`由Windows SDK中的头文件进行定义，但是，许多文件没有记录，只能通过检查相关库的调试符号中的引用来识别。对于包含嵌入式搜索数据的LNK文件来说，`User Property View ItemID`中的第一个`PropertyStore`有一个`Property Format GUID`，其值为`{1E3EE840-BC2B-476C-8237-2ACD1A839B22}`，其结构中的`Id`值为2，对应于`PKEY_FilterInfo`。

`PKEY_FilterInfo`的`TypedPropertyValue`字段包含一个`VT_STREAM`属性。通常情况下，一个`VT_STREAM` property包含的type值为`0x0042`，2字节的padding，和一个`IndirectPropertyName`。`IndirectPropertyName`指明了一个备用流，该备用流或者包含simple property set存储中的`PropertySetStream`数据包，或者包含non-simple property set存储中的`CONTENTS`流数据(详细内容参考文献4)。该名称由宽字符的字符串`"prop"`开始，其后跟对应于`PropertySet`数据包中的property标识符的十进制字符串。

然而，因为`LNK`文件使用嵌入在`VT_STREAM`中的序列化property store，`IndirectPropertyName`仅仅检查是否以`"prop"`开头，而`value`本身会被忽略，这就导致`TypedPropertyValue`的结构变成如下形式：

| Offset | Size | Field | Description |
| --- | --- | --- | --- |
| `0x0000` | 2 | `Property Type` | `0x0042`, `VT_STREAM` |
| `0x0002` | 2 | `Padding` | `0x0` |
| `0x0004` | 4 | `IndirectPropertyName Length` | `x` |
| `0x0008` | `x` | `IndirectPropertyName` |  |
| `0x0008 + a` | 2 | `Padding` | `0x0` |
| `0x000a + x` | 4 | `Stream Data Size` | `y` |
| `0x000e + x` | `y` | `Stream Data` |  |

`Stream Data`中的数据取决于stream property属于的特定的`PKEY`。对于`PKEY_FilterInfo`来说，`Stream Data`通常包含一个嵌入的`PropertyStoreList`，其中包含了多个序列化的`PropertyStore`结构。此时，`Stream Data`的结构如下所示：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Size(x)` |
| `0x0004` | 4 | `PropertyStoreListSize(y)` |
| `0x0008` | `y` | `PropertyStoreList` |
| `0x0008 + y` | 4 | `Padding(0x0)` |
| `0x000c + y` | `var` | `Alignment padding` |

`PKEY_FilterInfo` stream 中的嵌套`PropertyStoreList`实际上是`.search-ms`文件中"conditions"标记的序列化版本。下面是一个conditions tag的样例结构：

```xml
<conditions>
  <condition type="andCondition">
    <attributes>
      <attribute attributeID="{9554087B-CEB6-45AB-99FF-50E8428E860D}" clsid="{C64B9B66-E53D-4C56-B9AE-FEDE4EE95DB1}" chs="1" sqro="585" timestamp_low="2637566744" timestamp_high="30796574">
        <condition type="andCondition">
          <attributes>
            <attribute attributeID="{9554087B-CEB6-45AB-99FF-50E8428E860D}" clsid="{C64B9B66-E53D-4C56-B9AE-FEDE4EE95DB1}" chs="0" parsedString="size:500..1000 blah" localeName="en-US" timestamp_low="2555883479" timestamp_high="30796574" />
          </attributes>
          <condition type="leafCondition" property="System.Size" operator="imp" propertyType="stringarray" value="=500; =1000" valuetype="System.StructuredQueryType.Integer" localeName="en-US">
            <attributes />
          </condition>
          <condition type="leafCondition" property="" operator="imp" propertyType="string" value="blah" localeName="en-US">
            <attributes />
          </condition>
        </condition>
      </attribute>
    </attributes>
    [......]
  </condition>
</conditions>
```

"attribute"标签的详细功能尚未公开，但是该标记包含一个对应于`CONDITION_HISTORY`的`GUID`，一个对应于`StructuredQuery`的`CConditionHistory`类的`CLSID`，这表示嵌套的condition和attribute结构代表着之前已保存的搜索查询记录。通常情况下，`attribute`标签的`chs`可以确定可选history是否存在（猜测）。当此结构序列化到property store中时，会将其放入`PKEY_FilterInfo PropertyStoreList`中，它采用带有上述`Property Format GUID`的property bag的形式。特别情况下，序列化的`Conditions`结构包含在一个使用名称为`Condition`进行标识的`VT_STREAM` property中。此时，`PropertyStore` item的结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Value Size(x)` |
| `0x0004` | 4 | `Name Size(0x14)` |
| `0x0008` | 1 | `Reserved(0x00)` |
| `0x0009` | 20 | `Name("Condition")` |
| `0x001d` | 2 | `Property Type(0x0042 VT_STREAM)` |
| `0x001f` | 2 | `Padding(0x0)` |
| `0x0021` | 4 | `Indirect Propname Length (y)` |
| `0x0025` | `y` | `Indirect Propname (prop *)` |
| `0x0025 + y` | `var` | `Condition object` |

`Condition object`通常是一个`Leaf Condition`或者`Compound Condition`对象，其中可以包含嵌套对象，通常包含一个或多个`Leaf Condition`，也可能包含其他的`Compound Condition`对象。2种condition object都具有以下结构：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Condition size (x)` |
| `0x0004` | 16 | `Condition GUID` |
| `0x0014` | 4 | `Number of Attributes` |
| `0x0018` | `var` | `Attributes` |

对于`Leaf Condition`来说，其`Condition GUID`为`{52F15C89-5A17-48E1-BBCD-46A3F89C7CC2}`；对于`Compound Condition`来说，其`Condition GUID`为`{116F8D13-101E-4FA5-84D4-FF8279381935}`。`Attributes`字段由几个`Attributes` attribute结构组成，每个对应于之前提到的`.search-ms`文件中的`attribute`标签，字段结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 16 | `AttributeID` |
| `0x0010` | 16 | `Attribute CLSID` |

`Attribute`的其他结构取决于`AttributeID`和`CLSID`。对于前面的`attributeID`为`{9554087B-CEB6-45AB-99FF-50E8428E860D}`，`CLSID`为`{C64B9B66-E53D-4C56-B9AE-FEDE4EE95DB1}`的`CONDITION_HISTORY` attribute来说，其剩余结构为一个`ConditionHistory`对象，该对象结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `chs` |
| `0x0004` | 4 | `parseStringWChars (x = value - 1)` |
| `0x0008` | `x * 2` | `parseString` |
| `0x0008 + (x * 2)` | 4 | `localeStringWChars (y = value - 1)` |
| `0x000c + (x * 2)` | `y * 2` | `localeString` |
| `0x000c + (x * 2) + (y * 2)` | 4 | `sqro` |
| `0x0010 + (x * 2) + (y * 2)` | 4 | `unknown` |
| `0x0014 + (x * 2) + (y * 2)` | 8 | `timestamp` |
| `0x001c + (x * 2) + (y * 2)` | 4 | `nested_condition (may be number of directly nested conditions)` |
| `0x0020 + (x * 2) + (y * 2)` | `var` | `Condition object( if nested_condition > 0)` |

如果`nested_condition`的值大于0，`CONDITION_HISTORY`就会有嵌套的condition object，该对象本身也可能具有具有嵌套条件的嵌套属性，依此类推。

在完全读取了所有的嵌套结构后，`Compound Condition`和`Leaf Condition`结构后续就不同了。紧接`Attributes`字段后面，`Compound Condition`的结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Unknown(通常为0x0)` |
| `0x0004` | 4 | `numFixedObjects` |
| `0x0008` | `var` | `Conditions( if numFixedObjects > 0)` |

`numFixedObjects`字段指明了后续会跟多少附加conditions（通常为`Leaf Conditions`）。

而对于`Leaf Condition`，其结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 16 | `Condition Type PKEY GUID` |
| `0x0010` | 4 | `Condition Type Property ID` |
| `0x0014` | 4 | `Condition Operation(CONDITION_OPERATION enum)` |
| `0x0018` | `var` | `PropertyVariant` |
| `var` | 4 | `valuetypeStringWChars (x = value - 1)` |
| `var` | `x * 2` | `valueTypeString` |
| `var` | 4 | `localeStringWChars (y = value - 1)` |
| `var` | `y * 2` | `localeString` |
| `var` | `var` | `TokenInformationComplete1 flag` |
| `var` | 4 | `TokenInformationComplete1 ( if flag = 0x1 or 0x2)` |
| `var` | `var` | `TokenInformationComplete2 flag` |
| `var` | 4 | `TokenInformationComplete2 ( if flag = 0x1 or 0x2)` |
| `var` | `var` | `TokenInformationComplete3` |
| `var` | 4 | `TokenInformationComplete3 ( if flag = 0x1 or 0x2)` |

`TokenInformationComplete`结构存在与否取决于前面的flag是否进行了设置，如果没有设置那么紧跟下一个flag；如果进行了设置，那么紧跟的结构如下：

| Offset | Size | Field |
| --- | --- | --- |
| `0x0000` | 4 | `Unknown` |
| `0x0004` | 4 | `Unknown` |
| `0x0008` | 4 | `numWChars (x = value - 1)` |
| `0x000c` | `x * 2` | `String` |
| `0x000c + (x * 2)` | 4 | `Value (flag = 2时才存在)` |

对于`Leaf Condition`的`PropertyVariant`字段，其结构大致与`PROPVARIANT`结构（参考文献5）相似。`PropertyVariant`结构包含2字节的type，其后为该type的data。重要的是要注意，由于参考文献5中指定的填充字节往往不存在，因此`StructuredQuery`似乎对`PROPVARIANT`结构进行了自定义实现。 还需要注意的是，值`0x1000`或`VT_VECTOR`与另一种类型结合使用，意味着将有多个指定类型的值。

简述一个具有保存的搜索的`LNK`文件的最简结构如下所示（去除无关结构）：

![](https://bbs.kanxue.com/upload/attach/202009/779730_UQT8J4PY2VX7YBC.png)

以上只是一个最简单的使用单个`Leaf Condition`结构的搜索`LNK`结构，真实情况中往往是从`Compound Condition`开始，并且包含很多嵌套结构（包括许多的`Leaf Condition`）。

### 3. 详细分析
