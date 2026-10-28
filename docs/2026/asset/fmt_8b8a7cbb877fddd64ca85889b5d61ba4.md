
利用浏览器的搜索功能搜索想要的功能API（不知道有哪些接口就靠猜，比如我想要一个修改函数名的接口我尝试了很多简单词组组合最后找到了`idc.set_name`），如果某一模块中搜索不到，点击手册左上角的所有模块再次搜索。

![](https://bbs.kanxue.com/upload/attach/202105/784955_9MG3P4SKHHNBVCD.jpg)

还要善用IDAPython API过度表：[df4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3K9h3c8S2y4K6c8Q4y4h3k6A6k6r3q4H3P5i4c8Z5L8$3&6Q4y4h3k6F1L8#2)9#2k6X3u0U0y4U0V1#2i4K6g2X3M7r3!0J5N6r3W2F1k6#2)9#2k6X3N6#2K9h3c8W2i4K6u0W2M7$3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@bb6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3K9h3c8S2y4K6c8Q4y4h3k6A6k6r3q4H3P5i4c8Z5L8$3&6Q4y4h3k6F1L8#2)9#2k6X3u0U0y4U0V1#2i4K6g2X3M7r3!0J5N6r3W2F1k6#2)9#2k6X3N6#2K9h3c8W2i4K6u0W2M7$3S2@1L8h3H3`.)

使用方法仍然是浏览器搜索匹配不同版本的接口

![](https://bbs.kanxue.com/upload/attach/202105/784955_5FQAHS55CERBYUR.jpg)

# 病毒行为分析

## 动态修复IAT

可以看到完整的修复IAT后的逻辑，首先动态获取API后再加载`ole32`模块获取创建COM组件所需接口。

![](https://bbs.kanxue.com/upload/attach/202105/784955_EU2SFE4URHDQFDD.jpg)

## 创建全局互斥体变量

退回到函数主逻辑，可以看到病毒首先修复了IAT表保证程序正常运行

![](https://bbs.kanxue.com/upload/attach/202105/784955_K98WBMERUKC36WS.jpg)

设置错误模式获取错误信息并创建名为`Global\01EB1FCA-9835-27F4-DB93-6F722EB23FB4`的全局互斥体保证进程唯一性。

![](https://bbs.kanxue.com/upload/attach/202105/784955_4AJKR9JR6S4XDPY.jpg)

如果返回系统错误码显示已存在互斥体则返回标志`1`。

![](https://bbs.kanxue.com/upload/attach/202105/784955_PUMH4UNHC48QRK9.jpg)

使用`M`快捷键找到宏定义对常量进行替换

![](https://bbs.kanxue.com/upload/attach/202105/784955_NDYSPTAXA6D46B3.jpg)

替换后的样式如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_DDTHW7HXCPZCVTP.jpg)

再次来到主函数，可以看到病毒有两个逻辑函数，暂且分别命名为`mv_logic_x`。

![](https://bbs.kanxue.com/upload/attach/202105/784955_CFVC2PFPRGEV9KM.jpg)

## 提权运行

来到主逻辑1中可以看到以下几个解析的函数与字符串信息。

![](https://bbs.kanxue.com/upload/attach/202105/784955_ETSG5CTJFSRHT2R.jpg)

### 获取当前进程版本号

进入第一个未解析函数，用于F5插件解析错误通过汇编查看内容。可以看到该函数获取PEB地址后分别读取了偏移`0xA4`和`0xA8`处地址的低一个字节数据。

![](https://bbs.kanxue.com/upload/attach/202105/784955_58RZQ4F4Q3CB632.jpg)

在PEB中，`0xA4`和`0xA8`分别是当前操作系统的主版本号和次版本号

![](https://bbs.kanxue.com/upload/attach/202105/784955_6XMQDZWHDNQVKZ8.jpg)

调整如下，可以看出该函数用于获取当前系统版本号。

![](https://bbs.kanxue.com/upload/attach/202105/784955_T75QJYF39BM6Z4J.jpg)

返回逻辑可以看出病毒在Vista版本以上的操作系统上运行时会执行

![](https://bbs.kanxue.com/upload/attach/202105/784955_EHAXBHKVWDDMASB.jpg)

### 判断用户权限

进入`sub_404842`函数可以看到该函数主要获取当前进程令牌信息。

![](https://bbs.kanxue.com/upload/attach/202105/784955_THMR9NE3K7UXSWP.jpg)

返回的令牌类型是一个`enum`类型

![](https://bbs.kanxue.com/upload/attach/202105/784955_H8JVWABU5EQ56GJ.jpg)

根据微软描述一共可以返回如下三个类型。

![](https://bbs.kanxue.com/upload/attach/202105/784955_MVN6CPEY7CHZQXA.jpg)

所以以下代码为判断是否有管理员权限，如果没有则执行以下提权代码。

![](https://bbs.kanxue.com/upload/attach/202105/784955_WRKB2G2548Y4DS5.jpg)

在判断当前用户进程处于受限权限用户组下时，病毒打开令牌获取用户SID和属性的组合值，总逻辑如下。

![](https://bbs.kanxue.com/upload/attach/202105/784955_XWP4973BWCSQURH.jpg)

首先通过`GetTokenInformation`函数获取`_SID_AND_ATTRIBUTES`结构体。该结构体包含了`Sid`和`Sid`的属性。

![](https://bbs.kanxue.com/upload/attach/202105/784955_PXJYZS5PCXBW8BZ.jpg)

可以看到微软官方形容SID代表一个用户组，而SID的属性用来表示当前用户组启动、禁用以及强制托管的权限属性。SID同时也标志如何使用这些属性。

![](https://bbs.kanxue.com/upload/attach/202105/784955_CF4A2JRWP3XF5BV.jpg)

病毒接下来获取SID中的偏移值，获取SID中Subauthority的第一个四字节值判断权限。

![](https://bbs.kanxue.com/upload/attach/202105/784955_QEDCGHZPM7U6K3P.jpg)

首先我们来观察下SID结构体，该结构体前8字节是用于记录SID的一些信息，后20字节分为5部分，每部分占用4字节表示域账户或组使用的唯一ID也称为SIDs。SIDs中前4个四字节元素用来表示域ID（我理解为网络域名之类的）。最后一个字节表示RID，也就是表示用户组或用户。

![](https://bbs.kanxue.com/upload/attach/202105/784955_WKQUTBKCR46WG96.jpg)

根据Identifier Authority字段与后续的域ID组合可以代表出不同的含义。微软将特定的组合称为"Well-know SIDs"。以下是对该类型的介绍

![](https://bbs.kanxue.com/upload/attach/202105/784955_EVZXM3RDKQRHT22.jpg)

返回函数调用位置使用IDA插件查看内容。可以看到他在比较SIDs的第一个4字节内容。

![](https://bbs.kanxue.com/upload/attach/202105/784955_2D9AHPSGENNNN2A.jpg)

我们通过计算器换算成10进制，并查询"Well-know SIDs"表，发现为高强制级别的含义，所以我们了解了该`if`判断仍然是在判断权限。我们可以看到该类型在Vista和Win server 2008中添加，这也印证了病毒为什么要判断系统版本。

![](https://bbs.kanxue.com/upload/attach/202105/784955_TSKMPKC5CWW4H8T.jpg)

### 获取病毒路径字符串

接下来病毒会通过动态申请堆内存用于存储当前病毒路径字符串。

![](https://bbs.kanxue.com/upload/attach/202105/784955_UE3W5DN7Y2NXT89.jpg)

#### 堆申请函数浅析

以下为病毒申请堆函数的简单分析

![](https://bbs.kanxue.com/upload/attach/202105/784955_Z6SYMF42HUUBKK2.jpg)

### 循环提权

可以看到`sub_404DC9`函数将某个值赋给了变量`v4`，这里我们可以不用着急分析该函数，因为`v4`与后续的函数调用有直接关系，可以考虑先分析明显行为反向猜测推导内容。继续向下分析可以看到病毒调用了`ShellExecuteExW`，并且将`v7`作为了一个参数。

![](https://bbs.kanxue.com/upload/attach/202105/784955_K3VZEZVW9FJYH9Y.jpg)

MSDN官方定义结构体为`SHELLEXECUTEINFOW`

![](https://bbs.kanxue.com/upload/attach/202105/784955_87BX8V6W88SE52R.jpg)

将`v7`变量类型修改为`SHELLEXECUTEINFOW`，可以看到病毒实际上在通过`ShellExecuteExW`循环调用`runas`命令提权当前进程。

![](https://bbs.kanxue.com/upload/attach/202105/784955_23KAX7K7QQQAVBS.jpg)

### 获取参数函数分析

整理得到函数获取参数字符串逻辑如下。

![](https://bbs.kanxue.com/upload/attach/202105/784955_JYT6KJKVZ7JBNMB.jpg)

### 提权函数整体逻辑

分析完成前述所有代码后整理提权函数逻辑如下：首先通过令牌分别进行系统信息、令牌首先信息以及令牌权级三个方面进行判断，如果进程处于高度受限的环境下则通过`ShellExecuteExW`调用`runas`进程循环提升当前进程权限，否则就陷入提权界面循环中。（如果受害者因为厌烦而点击提权确认则病毒开始加密逻辑）

![](https://bbs.kanxue.com/upload/attach/202105/784955_TG9XUMK5ERM3YUS.jpg)

#### runas简介

`Runas`是一个命令行工具，它会帮助进程提升为管理员权限运行。

![](https://bbs.kanxue.com/upload/attach/202105/784955_J9CAVSP3C2FQRGY.jpg)

## 加密逻辑

接下来我们进入加密逻辑进行静态分析

### 线程执行状态设置

进入加密逻辑函数首先看到病毒进程设置了线程状态为`ES_CONTINUOUS + ES_SYSTEM_REQUIRED`

![](https://bbs.kanxue.com/upload/attach/202105/784955_XZ342NW4N7VXFZX.jpg)

第一个属性代表设置一个状态值，该值在当前病毒逻辑函数中一直有效，直到下一次调用`SetThreadExecutionState(ES_CONTINUOUS )`函数时该值会被清除。第二个属性通过设置系统计时器强制系统处于工作状态。

![](https://bbs.kanxue.com/upload/attach/202105/784955_XDHZDJKKSA6UMUV.jpg)

### 模拟用户登录

接下来病毒会调用函数，使病毒进程模拟管理员账户登录执行病毒逻辑。

![](https://bbs.kanxue.com/upload/attach/202105/784955_NK2W9KNQETW6H54.jpg)

#### 获取explorer进程ID

首先进入第一个未解析的函数，进入函数可以看到函数内部又一次调用了一个函数，参数为1个常量，要获取的目标进程字符串名和一个函数指针。

![](https://bbs.kanxue.com/upload/attach/202105/784955_8DMFBH5SXHWKWEV.jpg)

继续跟进，病毒创建了一个进程快照通过循环对当前内存中进程信息块进行遍历，并将获取到的环境进程块作为参函数和进程名一起传入函数指针，调用函数指针指向的地址。

![](https://bbs.kanxue.com/upload/attach/202105/784955_6MBG26KTE2GDNFU.jpg)

再看函数指针指向的函数地址，它将传入的进程字符串名和目标进程字符串进行比较，如果与目标进程字符串命相等则获取进程信息块的进程ID，并将它传入字符串指针向后偏移4字节的地址存储，所以我们可以判断出函数传入的参数实际上为病毒作者设计的一个结构体，拥有目标进程字符串指针和进程ID两个元素。

![](https://bbs.kanxue.com/upload/attach/202105/784955_D9ABEWTJNQ4AVC7.jpg)

定义结构体

![](https://bbs.kanxue.com/upload/attach/202105/784955_M69WW6G7JYW9GRH.jpg)

修改后参数如下

![](https://bbs.kanxue.com/upload/attach/202105/784955_2F32XEUH9NSC26T.jpg)

#### 获取的进程权限

使用Vistual Studio查看宏对应的值计算出`0xF01FF`代表所有令牌权限

![](https://bbs.kanxue.com/upload/attach/202105/784955_FEPUEDS5TU5QVRU.jpg)

进入该函数我们可以看到以下逻辑：病毒首先获取当前进程句柄，根据当前进程的"Well-know SIDs"值判断是否具有系统的完整权限。如果具有完整权限则尝试获取`explorer`进程的所有权限令牌利用令牌模拟管理员账户登录。

![](https://bbs.kanxue.com/upload/attach/202105/784955_PWHD78VXSKQJ36J.jpg)

### 勒索配置文件解析

#### 配置文件分析

进入下一个函数，病毒首先解密出配置文件到内存中。

![](https://bbs.kanxue.com/upload/attach/202105/784955_XH6VSAG7VCJ8DW2.jpg)

我们可以进入配置文件内查看解密方式，由于前面分析注释的功效这里代码逻辑很清晰。病毒首先通过`hash`判断配置文件完整性，然后调用`rc4`解密算法对加密配置文件进行解密。

![](https://bbs.kanxue.com/upload/attach/202105/784955_R3XG3WAH8HJM37X.jpg)

我们查看病毒为加密配置文件生成的`hash`算法，内容如下，他只是用于校验我们静态分析的时候了解即可不需要对该算法详细解析。

![](https://bbs.kanxue.com/upload/attach/202105/784955_UB4KJPBZQS6GW9C.jpg)

接下来我们查看传入`rc4`算法中的常量数据，解密密钥为`'98C5SGrGnn11u9qBcQs31J2rxeq9d4oA'`。被加密的配置文件大小为`0x7AD7`，可以精准的通过IDA导出功能导出数据。

![](https://bbs.kanxue.com/upload/attach/202105/784955_7798VBCQYGTDVGP.jpg)

#### 配置文件解密

在知晓密钥、密钥长度、加密数据、加密数据长度和加密算法后我们可以用IDA对数据进行导出。（该功能在新版本IDA7.5被强化）选中要导出的数据类型（这里是自定义的字节数组类型，因为长度已知）选择`Edit->Export data`

![](https://bbs.kanxue.com/upload/attach/202105/784955_7QUVE29ETVHP6GC.jpg)

根据喜欢的形式导出，我这里选择二进制形式导出就不做二次解密了，选择直接用"网络厨师"（`cyberchef`）进行解密

![](https://bbs.kanxue.com/upload/attach/202105/784955_KP3YBWB88G7Y458.jpg)

将导出二进制数据上传网络厨师后输入密钥解密出配置文件

![](https://bbs.kanxue.com/upload/attach/202105/784955_BPSHNRNKM3KVS6J.jpg)

但是目前我们看到的内容都是连在一行，没有自动换行阅读起来十分难看。我们使用网络厨师进行符号替换来修改显示形式，如下所示。
