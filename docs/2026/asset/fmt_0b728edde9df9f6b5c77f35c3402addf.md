
# [原创]游荡于中巴两国的魅影——响尾蛇(SideWinder) APT组织针对巴基斯坦最近的活动以及2019年该组织的活动总结

## 一、前言

Gcow安全团队追影小组于2019年11月份捕获到名为SideWinder(响尾蛇)组织针对巴基斯坦的活动，介于该组织主要针对巴基斯坦和中国以及其他东南亚国家，且其于10月份时候针对中国部分国防重要行业进行类似手法的攻击活动，为了更好了解对手的攻击手段以及加以防范，团队将以最近的样本为契机来总结该组织为期一年的攻击活动。

响尾蛇（又称SideWinder、T-APT-04）是一个背景可能来源于印度的APT组织，该组织此前已对巴基斯坦和东南亚各国发起过多次攻击，该组织以窃取政府，能源，军事，矿产等领域的机密信息为主要目的。此次的攻击事件以虚假邮件为诱饵，利用Office远程代码执行漏洞（`CVE-2017-11882`）或者通过远程模板注入技术加载远程URL上的漏洞文件。在针对于巴基斯坦的攻击中我们发现了`Lnk`文件的载荷，其主要驱动是`mshta.exe`，攻击者通过各种方式以达到伪装的目的。

在我们的样本捕获中，我们发现了该组织在这一年之间的变化，其攻击的手段越来越先进，这对我国的军事部门当然是一个不容小觑的威胁，所以我们追影小组将带领各位读者来回顾该组织的攻击手法，以及其技术的更迭。

## 二、样本分析

为了方便于各位读者的理解，笔者画了一张关于该组织攻击的流程图如下：

![](https://bbs.kanxue.com/upload/attach/202001/879321_P7JYTXE3G2S4MCZ.jpg)

在2019下半年，该组织经常使用该流程针对巴基斯坦和中国的目标进行攻击。

`ADVOCATE.docx`利用远程模板注入技术加载含有漏洞的`CVE-2017-11882`漏洞RTF文档，使用这种加载方式可以绕过防病毒网关，增加成功率。当成功加载`main.file.rtf`文件后，释放`1.a`到`Temp`目录下，触发漏洞`shellcode`执行`1.a`，`1.a`是一个混淆后的`Jscript`脚本文件，再次释放`Duser.dll`文件和`tmp`文件，并拷贝`rekeywiz.exe`到`C:\ProgramData\DnsFiles`目录下，并执行`rekeywiz.exe`文件，带起`Duser.dll`，`Duser.dll`加载`tmp`文件。

### 1. 诱饵文档

#### 1.1 样本信息

![](https://bbs.kanxue.com/upload/attach/202001/879321_D22VF9J6YFU34S6.jpg)
![](https://bbs.kanxue.com/upload/attach/202001/879321_PC4YY9TD5WBQ8A7.jpg)

#### 1.2 分析

![](https://bbs.kanxue.com/upload/attach/202001/879321_GU8EAZAHS87TTGH.jpg)

打开`ADVOCATE.docx`样本后，利用远程模板注入技术远程加载远程模板：`0ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6o6q4Q4x3X3c8T1K9h3&6Q4x3X3g2F1k6i4c8Q4x3V1k6A6L8h3q4Y4k6i4y4Q4x3V1j5J5b7U0M7I4y4@1f1&6z5q4)9J5c8W2)9J5k6o6q4Q4x3V1j5I4x3U0f1%4x3g2)9J5c8U0c8o6y4K6V1@1y4@1g2o6i4K6u0r3L8h3q4A6L8W2)9J5k6h3k6A6L8r3g2Q4x3X3g2J5N6r3j5.`

![](https://bbs.kanxue.com/upload/attach/202001/879321_MGXCDXPEXV44C2H.jpg)

成功打开后显示用于掩饰目的的文档，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_Z37A26EU2Z3Q2V8.jpg)

### 2. 漏洞文档

#### 2.1 样本信息

![](https://bbs.kanxue.com/upload/attach/202001/879321_63XGJ7HHXRN28K7.jpg)

#### 2.2 分析

当`main.file.rtf`加载成功后，会将`1.a`文件释放到当前用户`temp`文件夹下面。

`1.a`是嵌入到`rtf`文档中的`OLE Object`，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_W8DB7TXATYKTPFA.jpg)

通过分析`rtf`也可以看到：

![](https://bbs.kanxue.com/upload/attach/202001/879321_YABDUDY3H5AFQF3.jpg)

默认释放到`temp`文件夹中，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_Z7RNG76QJDTERPD.jpg)

`1.a`文件通过Shellcode加载起来。

##### Shellcode 分析

Shellcode代码如下图：

直接在`00411874`处下断点，此处为`ret`处，也就是将通过覆盖`ret`返回地址，达到任意代码执行目的，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_G7RSVBSXC49ETU4.jpg)

可以看到`esp`值已经被覆盖为`0x18f354`，这个就是Shellcode入口地址，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_R35RQJ3G37KNYEB.jpg)

也可以在`rtf`文件中找到Shellcode，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_YK423RE4TA9ZV8J.jpg)

Shellcode通过获取`RunHTMLApplication`来加载恶意JS：

![](https://bbs.kanxue.com/upload/attach/202001/879321_FD8ZHKBY57SG5RP.jpg)

对异或后的JS代码进行解密，秘钥是12：

![](https://bbs.kanxue.com/upload/attach/202001/879321_BY8R6UEYXH89RZV.jpg)

将执行JS的命令行替换其原来的命令行：

![](https://bbs.kanxue.com/upload/attach/202001/879321_557J8VMZ9RP85MB.jpg)

恶意JS如下：（读取`1.a`文件的所有内容并且用`eval`执行）

![](https://bbs.kanxue.com/upload/attach/202001/879321_9VQEAU9D6CUJV2Q.jpg)
![](https://bbs.kanxue.com/upload/attach/202001/879321_Y8F4RAB56FQN2M7.jpg)

##### 1.a 分析

###### 样本信息

![](https://bbs.kanxue.com/upload/attach/202001/879321_QTDQFKQ3JT6KC75.jpg)
![](https://bbs.kanxue.com/upload/attach/202001/879321_JE5DTQ7M2VZ3ESU.jpg)

###### 分析

通过分析，`1.a`是一个通过`DotNetToJScript`生成的`Jscript`文件，并且经过混淆，但是还原后还可以看出来，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_YTYM3AVU59JQFNX.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/879321_X3AUREH6EAQJZDD.jpg)

![](https://bbs.kanxue.com/upload/attach/202001/879321_QTSC5AD9FF2BR3A.jpg)

其主要逻辑即为将内置的C# DLL解密后内存加载其`Work()`函数，传入三个参数：第一个参数是黑DLL的数据，第二个参数是同目录下的`tmp`文件数据，第三个参数是混淆的C2地址。

通过调试解密出内置DLL文件的Base64编码：

![](https://bbs.kanxue.com/upload/attach/202001/879321_695MYNNUVRENFAP.jpg)

Base64解密后：

![](https://bbs.kanxue.com/upload/attach/202001/879321_CWR7TRGRGPVZFW6.jpg)

删除前面加载部分然后保存为DLL文件：

![](https://bbs.kanxue.com/upload/attach/202001/879321_B2N6YBE2GK5GHPR.jpg)

`StInstaller.dll`

其`Work()`函数是其核心：

![](https://bbs.kanxue.com/upload/attach/202001/879321_DSZC7ZMX6EUJJFC.jpg)

1. 检测白名单文件是否存在，若存在则拷贝到其工作目录下：

![](https://bbs.kanxue.com/upload/attach/202001/879321_TU5T9F59WPK2HCR.jpg)

2. 修改注册表添加启动项以开机启动，注册表的值为拷贝后的白名单文件路径：

![](https://bbs.kanxue.com/upload/attach/202001/879321_3JGJ3QNDM9PTU3H.jpg)

3. 释放对应的恶意DLL、`tmp`文件以及配置的`config`文件：

![](https://bbs.kanxue.com/upload/attach/202001/879321_BR596UMCR6U9XW9.jpg)

4. 启动白名单程序：

![](https://bbs.kanxue.com/upload/attach/202001/879321_KFXX2ACK7SXDEVF.jpg)

通过 API Monitor 可以直观看到释放流程，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_RZACUBP8MC98TH5.jpg)

1. 拷贝`c:\windows\syswow64\rekeywiz.exe`到`c:\ProgramData\DnsFiles\rekeywiz.exe`下面。
2. 释放`Duser.dll`文件到`C:\ProgramData\DnsFiles\Duser.dll`。
3. 释放`xxx.tmp`文件到`C:\ProgramData\DnsFiles\xxx.tmp`。
4. 使用`CreatePocess`拉起`rekeywiz.exe`。

##### Duser.dll 分析

###### 样本信息

![](https://bbs.kanxue.com/upload/attach/202001/879321_45U9YHCTK48VPCN.jpg)

###### 分析

`Rekeywiz.exe`是一个白名单文件，存在DLL劫持特性，俗称白加黑，如下图：

![](https://bbs.kanxue.com/upload/attach/202001/879321_SJC8NQ7X2JDX87F.jpg)

利用`rekeywiz.exe`带起`Duser.dll`，`Duser.dll`再将`***.tmp`文件（此处用`***`表示随机文件名）解密后，内存加载.NET，使其逃避防护软件查杀。关键代码如下图所示：

![](https://bbs.kanxue.com/upload/attach/202001/879321_R64PK45N5UGCCRQ.jpg)

选取`.tmp`文件的前32字节当作秘钥，对后续的字节进行异或解密后，使用`Assembly.Load`加载到内存执行。

解密后，发现是一个.NET后门程序，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202001/879321_VJYDVZKA5MS856K.jpg)

##### SystemApp.dll 分析

###### 样本信息

![](https://bbs.kanxue.com/upload/attach/202001/879321_A2TJ44ZX7CB452Q.jpg)

###### 分析

![](https://bbs.kanxue.com/upload/attach/202001/879321_PCSJ7E59JSGPRM7.jpg)

`start()`函数

首先加载基础设置信息，设置两个时间回调函数`GET()`函数、`POST()`函数，通过基础配置`Settings`类的属性来判断是否需要获取系统信息，写入选择文件，最后执行两个时间回调函数`GET()`、`POST()`，执行时间是5000秒。

![](https://bbs.kanxue.com/upload/attach/202001/879321_9FDY5KQPVC7UR77.jpg)

`LoadSettings()`函数

通过`Settings.settingsFilePath`来获取配置文件路径，然后通过`Decode()`函数来加载到内存，再返回一个用配置文件信息初始化的`Settings`类，否则返回默认配置。

![](https://bbs.kanxue.com/upload/attach/202001/879321_4YJ4V9QD4XN5SXH.jpg)

基础配置信息

其中可以看见默认C2地址：

`75cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3q4%4K9#2)9J5k6h3&6W2N6q4)9J5c8U0t1H3x3W2)9J5c8V1!0S2h3X3u0d9c8#2b7&6b7g2Z5$3M7X3S2x3e0g2y4q4g2#2y4G2c8Y4W2C8g2$3&6u0y4@1k6W2c8h3u0j5k6r3N6$3e0Y4k6%4h3W2m8Q4x3V1k6Q4x3X3b7I4i4K6u0r3x3e0t1#2y4K6q4Q4x3V1j5I4x3o6t1#2y4h3q4X3j5H3..`

![](https://bbs.kanxue.com/upload/attach/202001/879321_URYH45XQ72S65Y4.jpg)

`DecodeData()`函数

`Decode()`函数主要复制加解密数据文件，就是将文件的前32位当作`key`，循环异或后面的数据，来解码出源文件数据。

![](https://bbs.kanxue.com/upload/attach/202001/879321_RTTG5GM22EVRX3W.jpg)

`EnCode()`函数，也就是加密函数，和`Decode()`函数同理：

![](https://bbs.kanxue.com/upload/attach/202001/879321_UKWYFPD9WR4QJPR.jpg)

`Get()`函数

从配置信息里面的C2地址下载数据，通过`DecodeData()`函数解码后传入`Process()`执行：

![](https://bbs.kanxue.com/upload/attach/202001/879321_6UHSHHFV4AX8672.jpg)

`Process()`函数上半部分

`Process()`函数主要将传入的数据文件解析执行，先申请出一个`Loader`类型，加载传入的`data`，然后将`data`解Base64后，根据解码出来的数据的第一个`byte`来选择需要执行的功能：

![](https://bbs.kanxue.com/upload/attach/202001/879321_EUQJCVMBB4PN5FH.jpg)

`Process()`函数中间部分

函数可执行的主要功能：

1. 获取系统信息，写入`.sif`文件

![](https://bbs.kanxue.com/upload/attach/202001/879321_R9J3U7FCFY967DX.jpg)

2. 获取文件列表，写入`.flc`文件

![](https://bbs.kanxue.com/upload/attach/202001/879321_JN3KNBXXNXJR38B.jpg)

3. 获取指定文件，先复制移动到`.fls`

![](https://bbs.kanxue.com/upload/attach/202001/879321_JMBH5QWP62WFRHK.jpg)

4. 修改setting

![](https://bbs.kanxue.com/upload/attach/202001/879321_ZHQ3SCESXKRQ4YT.jpg)

5. 更新C2地址

![](https://bbs.kanxue.com/upload/attach/202001/879321_JXPNVXNTBRE9D8P.jpg)

6. 准备上传文件

![](https://bbs.kanxue.com/upload/attach/202001/879321_BQ9WHYEGBE6NQRN.jpg)

7. 加载文件执行

![](https://bbs.kanxue.com/upload/attach/202001/879321_WB4U2AJZGMUKFUD.jpg)

8. 设置文件最大尺寸

![](https://bbs.kanxue.com/upload/attach/202001/879321_8PTH67YEUBSHU7Z.jpg)

9. 下载文件

![](https://bbs.kanxue.com/upload/attach/202001/879321_6EBNQSEJWPFTQ2P.jpg)

`Case`功能列举表格：

![](https://bbs.kanxue.com/upload/attach/202001/879321_F34UPRPY4ZNQUKJ.jpg)
![](https://bbs.kanxue.com/upload/attach/202001/879321_WE9S3JQQBK8B8AP.jpg)

`Process()`函数下半部分

`Process()`函数执行出现异常就写入随机命名`.err`文件
