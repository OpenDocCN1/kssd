
# ACPI WMI 杂记 (对 OEM Bios导出给Windows的接口----导出OEM内部使用的WMI接口 一文补全)

关于WMI ACPI，建议先通读：1. <[[原创]BIOS知识点滴Follow Bini系列之---WMI ACPI](https://bbs.kanxue.com/elink@fecK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2X3L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1y4o6t1H3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9)>了解WMI ACPI能提供什么后，再参考MSDN 2. <[Windows Instrumentation: WMI and ACPI](https://bbs.kanxue.com/elink@380K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3K9r3q4J5k6s2N6S2M7X3g2Q4x3V1k6V1k6i4y4A6k6$3&6Q4x3V1k6V1L8U0j5I4y4o6l9J5z5q4)9J5z5s2k6Q4x3@1c8$3M7#2)9J5k6e0R3#2i4K6t1&6)>，另外，3. <[文件系统驱动编程基础篇之4——Wmi管理规范 mof文件](https://bbs.kanxue.com/elink@331K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2X3L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1y4o6t1H3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9)>也可以作为1.的补充。

需要特别说明的，1.中留了demo程序，一般读者没有环境测试(也不建议用真实机器测试，万一不开机损失挺大的)，这时倒可以参考我的文章<[解密OEM Bios导出给Windows的接口----导出OEM内部使用的WMI接口](https://bbs.pediy.com/thread-261129.htm)>，从现有BIOS中提取MOF和WMI接口用于学习。

本文应该是<[解密OEM Bios导出给Windows的接口----导出OEM内部使用的WMI接口](https://bbs.pediy.com/thread-261129.htm)>一文的原理补充和总结，主要补充一下MOF资源文件和ACPI中各个WMI接口的关系。要厘清他们的关系，得先用<[[原创]BIOS知识点滴Follow Bini系列之---WMI ACPI](https://bbs.kanxue.com/elink@2c7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2X3L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1y4o6t1H3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9)>文中使用的代码片：

Mof描述文件:

```
// Author: bini.Yi易祝兵 026K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2X3L8$3W2@1i4K6u0W2j5$3!0E0i4K6t1$3L8X3u0K6M7q4)9K6b7U0t1H3x3o6S2Q4x3X3b7H3z5g2)9J5k6o6t1@1

// File: demowmi.mof

//{39142400-C6A3-40fa-BADB-8A2652834100}

//IMPLEMENT_OLECREATE

//0x39142400, 0xc6a3, 0x40fa, 0xba, 0xdb, 0x8a, 0x26, 0x52, 0x83, 0x41, 0x00);

[WMI,
Dynamic,
Provider("WmiProv"),
Locale("MS\\0x409"),
Description("Events"),
guid("{39142400-C6A3-40fa-BADB-8A2652834100}")
]
class DemoWMIData //DemoWMIData是ClassName
{
    [key, read]
     string InstanceName;
    [read] boolean Active;

    [WmiDataId(1),
     read, write,
     Description("description")
    ] uint32 Data;
};
```

ASL（WMI）文件：

```
// Author: bini.Yi易祝兵 2008-09-24
// File: demowmi.ASL

Device(DWMI)
{
    // PNP0C14 is PNP ID assigned to WMI mapper
    Name(_HID, EISAID("PNP0C14"))
    Name(_UID, 0x0)

    Name(_WDG, Buffer(){
        // {39142400-C6A3-40fa-BADB-8A2652834100}
        0x00, 0x24, 0x14, 0x39, 0xA3, 0xC6, 0xFA, 0x40, 
        0xBA, 0xDB, 0x8A, 0x26, 0x52, 0x83, 0x41, 0x00, //GUID
        0x30, 0x30, //'00' Object ID
        0x01, //Instance Num
        0x01,        // 00 = Demo
    })

    Name(DD00, 0)

    Method(WQ00, 1)
    {
      DBGS("Demo Wmi Get Function:")
      DW2H(DD00)
        Return(DD00)
    }

    Method(WS00, 2)
    {
      DBGS("Demo Wmi Set Function:")
      DW2H(ARG1)
        Store(ARG1, DD00)
    }
}
```

### WMI ACPI调用模型

MOF描述文件是最终导出给用户调用的接口，WMI ACPI中的ASL code部分则是MOF所描述的`Class`的实现。如果读者反复阅读和对比MOF/ASL文件，会觉得他们很像动态链接库：MOF文件如同`Class`头文件，负责约定和导出调用接口。ASL如同`DLL`文件负责`DLL`函数的实现。至于`_WDG`对象，则充当了`DLL`的`EAT`（函数地址表）的中间人角色（其实还依赖于`_WDG`对象中嵌入式`MOF`文件，后面会提到），当然`EAT`表中可能会有若干表项组成，`_WDG`对象也是如此。`_WDG`中单个表项具有如下数据结构：

```
typedef struct _Mapper
{
     GUID guid;             // GUID that names data block
     union
     {
         CHAR ObjectId[2];  // 2-character ACPI ID  (Data Blocks and Methods)
         struct 
         {
             UCHAR NotificationValue;  // Byte value passed by event handler control method
             UCHAR Reserved[1];
         } NotifyId;
    }
     USHORT InstanceCount;  // Number of separate instances of data block
     USHORT Flags;          // Flags
}Mapper;
```

当所有表项结合在一起，形成一个巨大的`Mapper[N]`数组，这个数组最终构成`_WDG`对象。而`Mof`资源文件的编写者(无疑是`OEM`厂商了)，会为`_WDG`对象中每个`Mapper[i]`制作`Class`。每个`Class`都有`Mapper[i].guid`对应。

### 调用过程（纯个人臆测，不过我觉得挺有道理的！）

1. WMI的调用者(想象成`DLL`的调用者)，根据`MOF`资源文件(想象成`Class`头文件)描述的`WMI`接口，以`ClassName`的形式去调用`ASL` code的(想象成`DLL`文件)时，`ACPI.sys`会到`ACPI`命名空间中先查找`_WDG`对象。

2. 找到`_WDG`对象后，遍历其`Mapper[N]`数组，并定位到`Mapper[i].Flags==0`的数组项，该数组项指向一块`buffer`，`buffer`中包含编译后的`MOF`文件，`MSDN`称之为嵌入式`MOF`资源。

3. `ACPI.sys`从嵌入式`Mof`资源中查找匹配的`ClassName`（因为上层使用`Class`调用`WMI`）。大家不妨回忆前面`Mof`资源描述文件中含有`guid`和`ClassName`

4. 获得`ClassName`后，进一步可以获得接口名的`guid`，然后回到`_WDG`的`Mapper[N]`数组中，根据`Mapper[i].guid`查找与调用者`GUID`相匹配的数组项。

5. 匹配到`GUID`后，`ACPI.sys`会取出`Mapper[i].ObjectID`成员，成员中包含`xx`值，`ACPI.sys`将`WQ/WS`和`xx`拼接，形成`WQxx`或者`WSxx`，然后去`ACPI`命名空间调用相应的`WQxx`或者`WSxx`方法。

<[[原创]BIOS知识点滴Follow Bini系列之---WMI ACPI](https://bbs.kanxue.com/elink@96aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2X3L8$3W2@1i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1y4o6t1H3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9)>作为一个`demo`程序，`_WDG`对象的`Mapper[N]`数组只有一个数组项，没有体现上述搜索过程。所以，我换一个`N>1`的例子，嗯，那就用从<[解密OEM Bios导出给Windows的接口----导出OEM内部使用的WMI接口](https://bbs.pediy.com/elink@c9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6U0L8$3&6K6L8$3I4W2i4K6u0r3k6h3c8A6N6r3!0J5i4K6u0r3K9s2c8E0L8q4)9J5c8U0p5H3y4K6M7I4x3K6l9I4y4H3`.`.)>提取出来的`ThinkPad`的`_WDG`对象：

```
Device (WMI2)
{
    Name (_HID, EisaId ("PNP0C14") /* Windows Management Instrumentation Device */)  // _HID: Hardware ID
    Name (_UID, 0x02)  // _UID: Unique ID
    Name (_WDG, Buffer (0x64)
    {
    //==============Mapper[0]==============
            0xF1, 0x24, 0xB4, 0xFC, 0x5A, 0x07, 0x0E, 0x4E,
            0xBF, 0xC4, 0x62, 0xF3, 0xE7, 0x17, 0x71, 0xFA,
            0x41, 0x37,
            0x01,
            0x01,
     //==============Mapper[1]==============
            0xE3, 0x5E, 0xBE, 0xE2, 0xDA, 0x42, 0xDB, 0x49, 
            0x83, 0x78, 0x1F, 0x52, 0x47, 0x38, 0x82, 0x02,
            0x41, 0x38,
            0x01,
            0x02,
     //==============Mapper[2]==============
            0x9A, 0x01, 0x30, 0x74, 0xE9, 0xDC, 0x48, 0x45,
            0xBA, 0xB0, 0x9F, 0xDE, 0x09, 0x35, 0xCA, 0xFF, 
            0x41, 0x39,
            0x0A,
            0x05,
     //==============Mapper[3]==============
            0x03, 0x70, 0xF4, 0x7F, 0x6C, 0x3B, 0x5E, 0x4E,
            0xA2, 0x27, 0xE9, 0x79, 0x82, 0x4A, 0x85, 0xD1,
            0x41, 0x41,
            0x01,
            0x06,
     //==============Mapper[4]==============
            0x21, 0x12, 0x90, 0x05, 0x66, 0xD5, 0xD1, 0x11,
            0xB2, 0xF0, 0x00, 0xA0, 0xC9, 0x06, 0x29, 0x10, 
            0x42, 0x42,
            0x01,
            0x00                           
    })
```

这是`ThinkPad T460`部分`_WDG`对象，其中有5个数组项，最后是一个特殊数组项，暂不讨论。其他几个数组项对应的`WMI`接口如下：
