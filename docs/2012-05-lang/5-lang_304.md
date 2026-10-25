
# PCPro(PigChamp Pro) v3.0.23

> 原文：[`www.pediy.com/kssd/pediy04/kanxue096.htm`](https://www.pediy.com/kssd/pediy04/kanxue096.htm)

软件简介机注册机制：猪场管理软件，一个专业方面的软件。VB 6.0 程序，`CD-KEY` 为：`100-5410`，演示版有 30 天或 40 次使用的限制，注册时可以选择标准版、`PLUS` 版或咨询顾问版等，程序根据选择版本的不同产生软件的序列号，软件序列号的产生使用了 `Randomize` 函数并调用了 `GetVolumeInformationA` 这个 `API`，因此每台机器每次产生的的序列号是不同的，那么我们就不能自动产生序列号以及自动产生注册码了。序列号产生后，会根据序列号生成注册码并进行注册比较。但不管什么版本的序列号，其生成的注册码的过程是一样的，我们的目标就是分析其注册码的产生过程。

下载地址：`http://www.pigchampinc.com/products/pcprodownload.htm`

使用工具：`SmartCheck` 6.20，`IDA` Pro 4.15

注册计算过程：

首先安装 `PCPro`，运行 `SmartCheck`，配置好 `SmartCheck` 后，运行 `PCPro`，软件运行过程可以在 `SmartCheck` 看到，保存成 `SmartCheck` 文件。用 `IDA` 反编译 `PCPro.exe`，下面为反编译的结果。需要说明的是，下面的反编译结果，考虑到篇幅的关系，去掉了无关紧要的语句，因此并不完整，不要将反编译的内容照抄进行对照，应结合相应的语句标号进行对照。

在 `SmartCheck` 中，找到相应 `click` 的 `InputBox` 函数，向下找 `len` 函数，可知该函数的发生是在 `PCPro.exe` 的 `44CAAC` 处，加上基址 `400000`，则该函数在 `IDA` 中就是 `84CAAC`，如下：

```
:0084CAAC             call    __vbaLenBstr        ====> 序列号的长度
:0084CAC3             call    rtcLeftCharVar      ====> 左取序列号字符，长度不定，即不包含硬盘序列号部分
:0084CAE6             call    __vbaLenBstr        ====> 所取序列号的长度

从 84CAFB-84CC1A 利用 mid 函数将所取序列号中的连字符“-”去掉，过程略
:0084CC22             call    __vbaR8Str          ====> 转为双精度型
:0084CC27             fstp    qword ptr [ebp-38h] ====> 存到[ebp-38h]
:0084CC3A             call    __vbaLenBstr        ====> 输入的注册码的长度
:0084CC49             call    __vbaI2I4           ====> 转为整型

从 84CC57-84CD5B 利用 mid 函数分别取输入的注册码各字符，测试是否是数字，如果不是，则给出错误信息，过程略
:0084CD78             call    __vbaR8Str          ====> 输入的注册码转为双精度型
:0084CD7D             fstp    qword ptr [ebp-30h] ====> 存到[ebp-30h]
:0084CD80             fld     qword ptr [ebp-38h] ====> 调入序列号
:0084CD83             fsub    ds:dbl_407788       ====> 407788 的值为 56551
:0084CD92             fdiv    ds:dbl_401ED8       ====> 401ED8 的值为 5
:0084CDBA             fsubr   qword ptr [ebp-38h] ====> 相减
:0084CDBD             fadd    ds:dbl_407780       ====> 38341
:0084CDC3             fmul    ds:dbl_406470       ====> 乘 7
:0084CDD3             fstp    qword ptr [ebp-0C4h] ====> 存到[ebp-0C4h]
:0084CDD9             fld     qword ptr [ebp-38h] ====> 调入
:0084CDDC             fadd    ds:dbl_407778       ====> 48342
:0084CDEB             fdiv    ds:dbl_401D68       ====> 除以 2
:0084CE0A             call    __vbaFPInt
:0084CE0F             fsubr   qword ptr [ebp-0C4h] ====> 相减
:0084CE1B             call    __vbaFpR8
:0084CE20             fcomp   qword ptr [ebp-30h] ====> 进行比较
```

在 `SmartCheck` 中，`84CE1B` 中的值是可看到的，也就是该注册码，但由于该注册码太长，在 `SmartCheck` 中是用科学记数法显示的，因而不能看到全部的数字。

计算过程：设生成的要参与计算的序列号为 `sn`

```
((sn-int((sn-56551)/5)+38341)*7)-int((sn+48342)/2)
```

例如：序列号为 `84802-47223-824-9654`，则左取的字符 `sn` 为 `8480247223824`，经上面计算注册码为 `43249261164894`，测试一下，注册成功，做个注册器，选择不同版本反复注册，均注册成功。

该软件如果输入的注册码不对，会退出，但其中还有一个计算，则是输入该注册码后，不会退出，又变成 `Unregisted` 版，有兴趣的可以试试找出来，很简单的。 `4dd466d0e1e0cc123afbe504da2462ca`
