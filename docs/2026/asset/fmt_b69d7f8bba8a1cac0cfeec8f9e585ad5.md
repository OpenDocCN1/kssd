
# IDA静态分析REvil家族勒索病毒

本文献给所有像我一样刚从事安全行业的萌新，希望能对您有一定帮助。同时感谢所有在我迷茫时伸出援手的大佬们。

## 病毒家族简介

REvil又称Sodinokibi，是一种以私有软件即服务（RaaS）体系运营牟利的黑客组织。他们通过招募会员为其分发勒索软件攻击，当受害者支付赎金后该组织会将赎金与会员进行分成，因此很难定位其组织真实所在地。

## 样本基本信息

`md5`:FBF8E910F9480D64E8FF6ECF4B10EF4B

`sha1`:E6B32975ACB2CC5230DD4F6CE6F243293FD984FA

`sha256`:CEC23C13C52A39C8715EE2ED7878F8AA9452E609DF1D469EF7F0DEC55736645B

`VT`:[787K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3k6X3W2D9k6g2)9J5c8X3y4W2j5K6t1K6j5K6p5K6j5K6f1J5j5e0x3&6j5K6R3%4x3e0g2W2k6e0u0W2k6o6M7^5y4K6S2X3z5r3q4S2z5e0b7#2x3X3f1$3x3o6W2V1k6U0q4V1y4o6j5&6k6h3j5%4k6U0m8V1k6h3x3#2y4e0M7K6y4U0j5@1y4h3u0Q4x3V1k6V1k6i4c8W2j5%4c8A6L8$3^5`.](https://bbs.kanxue.com/elink@857K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3k6X3W2D9k6g2)9J5c8X3y4W2j5K6t1K6j5K6p5K6j5K6f1J5j5e0x3&6j5K6R3%4x3e0g2W2k6e0u0W2k6o6M7^5y4K6S2X3z5r3q4S2z5e0b7#2x3X3f1$3x3o6W2V1k6U0q4V1y4o6j5&6k6h3j5%4k6U0m8V1k6h3x3#2y4e0M7K6y4U0j5@1y4h3u0Q4x3V1k6V1k6i4c8W2j5%4c8A6L8$3^5`.)

![](https://bbs.kanxue.com/upload/attach/202105/784955_GU8AMPQNM69TAYK.jpg)

VT沙箱以将其标识为Sodinokibi。

![](https://bbs.kanxue.com/upload/attach/202105/784955_QBY672N62R5UDAW.jpg)

## 获取病毒静态信息

查看节区信息，可以看到存在一个自定义节`.11hix`，且各个节区熵值都较高证明程序存在大量加密数据。

![](https://bbs.kanxue.com/upload/attach/202105/784955_BGUCH38KC6T5SD6.jpg)

查看导入`dll`，识别出导入了`kernel32.dll`和`user32.dll`两个模块。证明样本对程序的导入表做了处理。

![](https://bbs.kanxue.com/upload/attach/202105/784955_UPUM9NH9MFB3UWZ.jpg)

查看识别出的API，没有什么重要内容，恶意行为被隐藏起来。需要病毒样本动态运行进行解密修复IAT。

![](https://bbs.kanxue.com/upload/attach/202105/784955_KTDDT9J59WD6D9F.jpg)

查看字符串资源，大部分为未识别的字符串，进一步验证了数据被加密处理的猜想。

![](https://bbs.kanxue.com/upload/attach/202105/784955_GM56JPQDQXAWRG4.jpg)

## IDA静态加载分析

第一次静态加载时应该尽量勾选右下角选项，这样IDA会额外加载资源数据动态导入的DLL数据以及默认不加载的静态节数据，以便我们透彻分析样本。

![](https://bbs.kanxue.com/upload/attach/202105/784955_S6JA4WAZJ9BGE55.jpg)

加载完成后，我们可以看到入口点`OEP`只有两个函数调用

![](https://bbs.kanxue.com/upload/attach/202105/784955_SBQ78B5UVWVZ8GZ.jpg)

点击进入第一个函数调用，并没有发现有趣的内容，继续点击跟进第一个函数查看内容。

![](https://bbs.kanxue.com/upload/attach/202105/784955_2MZ7KA5FYB2M5PM.jpg)

进入函数后发现了有趣的内容，病毒样本貌似在填充一个由`ESI`寄存器控制的数组。首先传入数组的4字节数据给`sub_406817`函数，然后将运算结果（`eax`）的值重新回填给数组，一共循环了160次，计算了640个字节。这与我们熟知的动态解析IAT方法很相似，加上我们之前分析病毒抹去了IAT可以合理猜测该处在做回填IAT表的操作。

![](https://bbs.kanxue.com/upload/attach/202105/784955_TQGU38UYHREG9G6.jpg)

使用`F5`插件查看如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_5A7SHFTG3DP2DVX.jpg)

点击查看该数组内容，发现是`.data`节的一个全局数组。根据之前的分析运算一共占用640字节。每4字节为一个`hash`值（经过`hash`处理的数据通常会将4字节全部占满，所以合理猜测为`hash`数据）

![](https://bbs.kanxue.com/upload/attach/202105/784955_KBN88MUGT95H2BE.jpg)

设置4字节的`hash`数组，数组元素为160，每个元素占用4字节。

![](https://bbs.kanxue.com/upload/attach/202105/784955_BHPWVZDUH7JHK8S.jpg)

处理后的数据如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_USRAHRKD25NS5D5.jpg)

最后我们还需验证下我们的猜想。程序调用动态导入函数的时候，汇编会以间接调用的方式呈现出来，如下形式。我们只需要在后续找到该形式的函数调用，且数据在该数组内即可验证成功。

```
call	dword ptr:[xxxx]
```

继续向下查找，可以看到“`call` 数组基址+数组偏移”的形式的函数调用（数组最大偏移为`0x280`）,该调用在重复调用同一个函数。验证了该静态数组为`hash`处理过的IAT表。

![](https://bbs.kanxue.com/upload/attach/202105/784955_QQ6HBR8XZSB5PZS.jpg)

同时也可以确定第一个函数整体逻辑在修复IAT表，修改变量名。

![](https://bbs.kanxue.com/upload/attach/202105/784955_GQAZH8KCGTVY4FA.jpg)

## 分析动态加载API的方法

进入`mv_dy_get_api`函数查看代码内容，可以看到IDA为我们解析的函数原型如下，虽然数长度相同但是看着很别扭（IDA最强大的就是注释功能，不要小看每条注释。小的注释积累起来可以引发质变）

![](https://bbs.kanxue.com/upload/attach/202105/784955_XNVVA87ZEZCYJSJ.jpg)

按`Y`快捷键修改函数原型如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_NTREA6D3AK8Y34E.jpg)

继续向下查看代码，将输入的`hash`进行异或运算得到转换后的`hash`，又将转换后的`hash`值高11位提取赋值给临时变量`v2`。可以看到

![](https://bbs.kanxue.com/upload/attach/202105/784955_GY49WYEJGNCJF93.jpg)

继续阅读代码，看到疑似`switch/case`的判断比较高11位数据。根据不同数据进行执行不同流程中的函数地址赋值，然后跳转到`LABEL_29`。根据病毒动态加载的经验，第一件事一般是动态获取目标模块的基地址，然后根据基地址解析对应导出函数的位置。所以我们合理假设该`switch/case`语句是在做获取`dll`基址的操作。而转换后的高11位`hash`值代表不同的`hash`值。

![](https://bbs.kanxue.com/upload/attach/202105/784955_R6YZ2AE68XURCJD.jpg)

整理后如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_T9RY7V3VJ4VPT3K.jpg)

继续向下阅读代码，看到调用了赋值的函数指针`pfn_ld_dll`，符合获取指定`dll`的基址的猜想，将临时变量改名为`imagebase`可以看到`v18`变量在获取导出符号的地址偏移。

![](https://bbs.kanxue.com/upload/attach/202105/784955_U5EFMA7XEABJ7JU.jpg)

`0x3c`偏移为`_IMAGE_DOS_HEADER`的`e_lfanew`字段，通过该字段值获取`imagebase`到`_IMAGE_NT_HEADERS`的偏移。

```
pe_nt_header_address == imagebase + e_lfanew == imagebase + (DWORD*)(imagebase + 0x3c) //imagebase + 0x3c == e_lfanew
```

![](https://bbs.kanxue.com/upload/attach/202105/784955_4VSTS7ABTTMFAZG.jpg)

可以看到`_IMAGE_NT_HEADERS`一共占用`0x18`个字节，也就是说`0x78`中有`0x18`字节偏移是为了跳过`_IMAGE_NT_HEADERS`占用的空间，为了获取`_IMAGE_OPTIONAL_HEADER`结构体偏移`0x60 == 0x78 - 0x18`的字段值。

![](https://bbs.kanxue.com/upload/attach/202105/784955_YSS9MZ2CWGZW9VG.jpg)

查看`_IMAGE_OPTIONAL_HEADER`结构体。偏移`0x60`处字段正好为数据目录表的首地址，而数据目录表第0项正好记录着导出符号表的偏移和大小。

![](https://bbs.kanxue.com/upload/attach/202105/784955_FV2WU38Q9TD27DV.jpg)

查看数据目录表表项结构体如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_MMGQE3BEFUYYR7U.jpg)

可以推出该语句为获取`VirtualAddress`偏移值

![](https://bbs.kanxue.com/upload/attach/202105/784955_WYMZ398PA8YGTVC.jpg)

查看导出表结构体

![](https://bbs.kanxue.com/upload/attach/202105/784955_7WNQJ85VRF47TE3.jpg)

整理出以下内容

![](https://bbs.kanxue.com/upload/attach/202105/784955_ZUWZPUTG5D9Y3UF.jpg)

按下`shift+F1`快捷键，切换到本地类型窗口。按快捷键`ins`，插入一个结构体。

![](https://bbs.kanxue.com/upload/attach/202105/784955_DSVY35P9X7CDTB3.jpg)

双击导入该结构体

![](https://bbs.kanxue.com/upload/attach/202105/784955_7XSTQ2Y7CHWYANQ.jpg)

将变量`va_offset`转换成结构体指针

![](https://bbs.kanxue.com/upload/attach/202105/784955_KRPUC3ACCJFFFUB.jpg)

导入结构体指针类型。

![](https://bbs.kanxue.com/upload/attach/202105/784955_U88Z39ZY4EXNHJQ.jpg)

整理得到如下结果。

![](https://bbs.kanxue.com/upload/attach/202105/784955_HZAMH2X9HQT6FTN.jpg)

查看后续代码，因为之前的注释修改，我们可以很清晰的看到函数逻辑。

![](https://bbs.kanxue.com/upload/attach/202105/784955_MM466QA7JVAMMNX.jpg)

进入`calc_fn_name_hash`函数可以看到是对导出函数字符串进行逐字节`hash`运算。

![](https://bbs.kanxue.com/upload/attach/202105/784955_6ZR2MYUAMGJYNW2.jpg)

PE解析API地址整体逻辑如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_JVW4Y3P7AA6HRBV.jpg)

接下来继续向上回溯分析加载`dll`的方法。点开加载`dll`的函数，可以发现绝大多数函数模板类似，都传入了一个固定`hash`值进行函数递归。获取`api`后将`v2`变量作为参数传递给该动态获取的API并调用执行。根据函数形式结合之前的分析，合理推测`0xCB0F8A29`为`LoadLibraryA`函数的`hash`值。而`v2`是要加载的`dll`名称。

![](https://bbs.kanxue.com/upload/attach/202105/784955_ZTWXCD3Q6GX6JFJ.jpg)

继续回溯，发现除了上述模板的调用方式，还有直接传入了静态`hash`给一个函数，这种方式值得我们额外关注。

![](https://bbs.kanxue.com/upload/attach/202105/784955_47ZDWGTU3CBAES2.jpg)

继续跟进来到以下函数中，看到有循环猜测正在做某种运算。看到循环中有个判断条件在与一个常数做差，转换成16进制为`0x41`，正好对应`Ascii`码表中的‘`A`’字符。因此可以猜测在比较字符串大小。

![](https://bbs.kanxue.com/upload/attach/202105/784955_PJBB7ATP3Y4QWDP.jpg)

我们首先修改`v5`为`chr`，接下来就会发现非常神奇的事，找寻其他跟`chr`关联的变量修改名称让整个逻辑结构一下变得清晰。

![](https://bbs.kanxue.com/upload/attach/202105/784955_RAFFHHKRSXDMRWH.jpg)

继续向上查阅代码可以看到函数`sub_404B12`，跟进函数内部可以看到熟悉的`fs:30h`。通过`fs`寄存器获取`PEB`

![](https://bbs.kanxue.com/upload/attach/202105/784955_7NBAAR24GBNMKRV.jpg)

获取`PEB`基址偏移`0xC`处的字段值，即`LDR`地址。

![](https://bbs.kanxue.com/upload/attach/202105/784955_X4GGGWTXGTT3RB9.jpg)

![](https://bbs.kanxue.com/upload/attach/202105/784955_QQKP2UFD2P3TH5T.jpg)

再通过`LDR`偏移`0x14`获取`InMemoryOrderModuleList`地址值。

![](https://bbs.kanxue.com/upload/attach/202105/784955_RTTTYX5H98E46AS.jpg)

判断内存模块加载顺序链表是否为空。

![](https://bbs.kanxue.com/upload/attach/202105/784955_59REZ9UPWWF5K32.jpg)

`InMemoryOrderModuleList`是一个双向链表指针，它链接的是`LDR_DATA_TABLE_ENTRY`结构体中的`InMemoryOrderModuleList`链表节点。

![](https://bbs.kanxue.com/upload/attach/202105/784955_TKVRFFM9N8HHEKJ.jpg)

获取`dll`的名称（`F5`插件在此处解析错误，所以使用汇编查看）

![](https://bbs.kanxue.com/upload/attach/202105/784955_K8SUJPZCEDXVUPS.jpg)

如果匹配`hash`值则返回模块基址

![](https://bbs.kanxue.com/upload/attach/202105/784955_9637V2JCH2785BK.jpg)

`list_entry+0x10`为`DllBase`模块基址

![](https://bbs.kanxue.com/upload/attach/202105/784955_QHN6WBDRFF48YER.jpg)

总体逻辑如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_D9UCN42WZBKMSY4.jpg)

## RC4算法解析

### 算法识别

接下来继续分析其他模块加载函数中对字符串的解密函数。首先可以看到解密函数传入了5个参数，第一个参数为一个静态全局数据，第2、3、4个参数全部是确定的常量值。
