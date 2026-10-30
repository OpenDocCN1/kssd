
```
// 长度为0, 无法读出数据

0:005> dd 0055fcf8+c-4 l1

0055fd00  00000000



0:005> db 0055fcf8+c

0055fd04  20 94 00 02 70 0e 00 00-00 00 00 00 00 00 00 00   ...p...........

0055fd14  00 00 00 00 00 00 00 00-24 15 37 00 80 ac 00 02  ........$.7.....

0055fd24  00 00 00 00 3f 32 fb 77-86 11 00 08 00 00 00 00  ....?2.w........

0055fd34  74 ab 00 02 00 00 00 00-00 00 00 00 00 00 00 00  t...............

0055fd44  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................

0055fd54  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................

0055fd64  00 00 00 00 00 00 00 00-00 00 00 00 35 32 fa 7c  ............52.|

0055fd74  88 11 00 00 60 af 00 02-f8 81 00 02 2b 00 00 00  ....`.......+...
```

此时构造的`BSTR`的长度域为0，数据无法正常读出。

### 步骤3：读取`COleScript`对象指针

`poc`中读取`COleScript`对象指针的代码如下，原理和上一步完全相同，此处不再过多分析：

![](https://bbs.kanxue.com/upload/attach/201806/628056_X96R3MK685BBEUH.png)

### 步骤4：覆写`vbscript!SafetyOption`

![](https://bbs.kanxue.com/upload/attach/201806/628056_YQ8G7M27XTGNENE.png)

在`IE8`中，`vbscript!SafetyOption`位于`COleScript`对象的`+0x174`处。`poc`代码中通过调用`overwrite`函数去覆写这一值，如下：

![](https://bbs.kanxue.com/upload/attach/201806/628056_9SDYF79A53YFXVF.png)

代码中伪造一个`type=0x400C`间接寻址对象，接着触发漏洞，随后将一个`Csng`（单精度浮点）对象的写入`COleScript+0x16C`开始的16个字节处，`COleScript+0x174`处正好写入`Csng`的`type`值4，从而开启上帝模式。

```
// type = 0x400c 对应间接寻址

0:005> dd 03218184+4 l4

03218188  0000400c 00000000 0036f914 00440044



// 覆盖前的SafetyOption

0:005> dd 0036f914 l1

0036f914  0000000e



// 覆盖后的SafetyOption，SafetyOption & 0x0B失效(0x024f0004 & 0x0B = 0)

0:005> dd 0036f914 l1

0036f914  024f0004
```

### 步骤5

开启上帝模式后，就可以弹出`cmd`窗口了。

![](https://bbs.kanxue.com/upload/attach/201806/628056_QXWXK2A2BTW94YN.png)

## 漏洞检测

下面通过逆向某安全软件来看一下对`CVE-2016-0189`的动态检测方案。

1. 首先`hook` `oleaut32!VariantChangeTypeEx`函数

   ![](https://bbs.kanxue.com/upload/attach/201806/628056_Y6SJ7EU6SMJ8P4T.png)

   ![](https://bbs.kanxue.com/upload/attach/201806/628056_UZKEZGCFPB5UCMJ.png)

2. 可以看到代码中对`CVE-2016-0189`的检测逻辑为：在调用`oleaut32!VariantChangeTypeEx`函数前后检查`rgsabound[0].cElements`所对应的第二维度的大小，若调用后的大小小于调用前的大小，则视为检出。

   ![](https://bbs.kanxue.com/upload/attach/201806/628056_SKJPR6XCVMRGHU3.png)

```
0:013> dt ole32!tagSAFEARRAY

    +0x000 cDims : Uint2B

    +0x002 fFeatures : Uint2B

    +0x004 cbElements : Uint4B

    +0x008 cLocks : Uint4B

    +0x00c pvData : Ptr32 Void

    +0x010 rgsabound : [1] tagSAFEARRAYBOUND



0:013> dt ole32!tagSAFEARRAYBOUND

    +0x000 cElements : Uint4B

    +0x004 lLbound : Int4B



// 调用VariantChangeTypeEx前的aw.A

0041e7a0  08800002 00000010 00000000 02ff0f58

0041e7b0  000007d1 00000000 00000002 00000000



// 调用VariantChangeTypeEx后的aw.A

0041e7a0  08800002 00000010 00000000 02ff0f58

0041e7b0  00000002 00000000 00000002 00000000



// rgsabound[0].cElements大小变化如下：

7d1(2000+1) -> 2(1+1)
```

这里有一个疑问，`MSDN`对多维数组的`rgsabound`域解释如下：

![](https://bbs.kanxue.com/upload/attach/201806/628056_HG5A57DRG6TEWD4.png)

但调试时发现`A(1, 2000)`的`rgsabound`实际使用顺序和文档描述相反，看检测逻辑里面判断的也是`rgsabound[0]->cElements`。我们以实际调试结果为主。

## 致谢

特别感谢 `Hu Jiang`，`Xu Xilin`，`Yang Kang` 在调试过程中的指导

## 参考链接

《CVE-2016-0189》 [3dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2G2M7X3W2Q4x3X3g2A6L8#2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4y4W2)9J5k6o6l9I4z5o6V1`.](https://bbs.kanxue.com/elink@289K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2G2M7X3W2Q4x3X3g2A6L8#2)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3j5%4k6W2i4K6u0V1x3U0l9I4y4W2)9J5k6o6l9I4z5o6V1`.)

《theori-io/cve-2016-0189》 [4e5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2G2M7X3W2Q4x3X3c8A6L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0k6Q4x3X3b7H3x3e0R3&6](https://bbs.kanxue.com/elink@42eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2G2M7X3W2Q4x3X3c8A6L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0k6Q4x3X3b7H3x3e0R3&6)

《Nebula漏洞利用包CVE-2016-0189漏洞利用分析》 [d26K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4@1L8$3!0D9i4K6u0r3x3e0x3I4y4K6j5$3i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@887K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4@1L8$3!0D9i4K6u0r3x3e0x3I4y4K6j5$3i4K6u0W2K9s2c8E0L8l9`.`.)

《WinDbg 漏洞分析调试（三）之 CVE-2014-6332》 [716K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0t1@1x3q4)9J5c8R3`.`.](https://bbs.kanxue.com/elink@3efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0t1@1x3q4)9J5c8R3`.`.)

《Write Once, Pwn Anywhere》 [b2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4y4Q4x3V1k6#2M7#2)9J5k6o6p5@1i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4K6i4K6u0r3N6i4y4Q4x3X3b7I4y4q4)9J5k6q4W2#2i4K6u0V1g2%4u0A6N6r3g2Q4x3X3c8a6L8X3y4W2i4K6u0V1f1s2N6F1i4K6u0V1b7h3&6&6N6$3S2W2M7X3g2Q4x3X3g2H3k6r3j5`.](https://bbs.kanxue.com/elink@2e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4y4Q4x3V1k6#2M7#2)9J5k6o6p5@1i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4K6i4K6u0r3N6i4y4Q4x3X3b7I4y4q4)9J5k6q4W2#2i4K6u0V1g2%4u0A6N6r3g2Q4x3X3c8a6L8X3y4W2i4K6u0V1f1s2N6F1i4K6u0V1b7h3&6&6N6$3S2W2M7X3g2Q4x3X3g2H3k6r3j5`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-3-26 13:08

被jq0904编辑

，原因：

上传的附件：

- [CVE-2016-0189漏洞分析.pdf](javascript:void(0);)

  （521.13kb，113次下载）
