
# [原创][车联网安全]使用STM32开发板实战汽车UDS诊断

## 前言

UDS（统一诊断服务）是车上很重要的一个诊断协议，但除非能接触到实际项目，否则目前没见过有开源的能进行真实 UDS 诊断的练习板，纯讲理论没啥意思

我在闲鱼买了一套 `UDS_bootloader` 的源码，目前跑通了几个 UDS 服务的功能，水一篇文章，介绍一下如何通过一块 STM32 的开发板实际练习 UDS 诊断，配套固件也会传上来，师傅们可以自己买板子烧进去练习

## 环境配置

### 硬件

`STM32F103ZE` 开发板，用来跑我们的 UDS 代码的

![](https://bbs.kanxue.com/upload/attach/202312/837755_N3W6FDTF5QV8QEF.webp)

`TJA1050` CAN 控制器接口模块，用来转换 CAN 信号的，淘宝卖的默认是没有排针的，要是自己有电烙铁可以焊上排针，方便接杜邦线

![](https://bbs.kanxue.com/upload/attach/202312/837755_K9DCGU59USNFZ36.webp)

CAN 调试仪，用来与 STM32 建立 CAN 通信的，什么品牌无所谓，我用 PCAN 习惯了

![](https://bbs.kanxue.com/upload/attach/202312/837755_YXUWR4EJKEJY87R.webp)

`Jlink` 或 `STlink`（主要是烧写固件的，最好买带这个排线的，不然自己按照引脚定义去接杜邦线去嗷）

![](https://bbs.kanxue.com/upload/attach/202312/837755_N9PC4N2EW9M5XA2.webp)

还需要一些杜邦线用来连接这几个硬件设备

![](https://bbs.kanxue.com/upload/attach/202312/837755_Q7RKPM78HU29ZYU.webp)

### 软件

软件主要是用两个，一个是 `Jflash` `TSMaster` CAN 通信的（夸一夸 TSMaster，个人觉得很好用

![](https://bbs.kanxue.com/upload/attach/202312/837755_RVTCFEUHJEYPGYA.webp)

### 硬件连接

`Jlink` 直接通过排线与 `STM32` 开发板相连即可，`STM32` 右边的 `USB` 接口是个串口可以看 `UART` 日志

`TJA1050` 的 `RX` 接 `STM32` 的 `PA11`，`TX` 接 `STM32` 的 `PA12`，`VCC` 接`STM32` 的 `5V`，`GND` 接 `STM32` 的 `GND`

`TJA1050` 的 `CANH` 接 CAN 分析仪的 `CAN_H`、`CANL` 接 CAN 分析仪的 `CAN_L`（ 这里以 `PCAN` 为例）

![](https://bbs.kanxue.com/upload/attach/202312/837755_3FUXZYS8J7DEZ58.webp)

### 固件刷写

安装好 `Jflash` 之后打开，选择新建项目

![](https://bbs.kanxue.com/upload/attach/202312/837755_U7GARCY62JHHMC5.webp)

点击三个点，在输入框输入 `STM32F103ZE` 过滤出来，选择下面那个短的，然后 ok

![](https://bbs.kanxue.com/upload/attach/202312/837755_DXZGC7XF4Q5WPUH.webp)

把两个固件都拖到右边的数据文件窗口，然后点击 `Target -> Production Programming` 烧写固件（`hex` 文件都是记录着地址信息的，直接烧录即可）

![](https://bbs.kanxue.com/upload/attach/202312/837755_C4GJPQZ79X5FYP2.webp)

然后打开串口调试工具，波特率设置为 `115200`，按下复位键看看是不是有输出了，如下输出说明正常

![](https://bbs.kanxue.com/upload/attach/202312/837755_BUFBFFHK4C2KBE9.webp)

## UDS通信

UDS 定义了一系列的服务，每个服务都有自己的 ID 即 `SID`（Service Identifier），接下来通过开发板实际进行 UDS 诊断通信体验一下，具体理论知识可以参考网上其他文章或者直接看 `14229` 的标准，文末我会上传附件

### 22 通过ID读数据

`22` 服务通过 ID 读取数据，例如读取当前会话状态的 ID 是 `F1 86`，那么可以使用 `7DF # 03 22 F1 86` 来读取当前会话，`F1 86` 后面跟的 `01` 就是当前会话状态

![](https://bbs.kanxue.com/upload/attach/202312/837755_XA49MABUB9HCUHS.webp)

在 `14229` 标准里面还有很多 ID，比如 `F1 90` 读取 VIN 码等（开发板暂未实现），以及厂商也会自定义 ID

### 10 诊断会话控制

先使用 `7DF # 03 22 F1 86` 读取当前会话

![](https://bbs.kanxue.com/upload/attach/202312/837755_2QC9GUTZ9QJHH3F.webp)

切换到扩展会话 `7DF # 02 10 03` 然后 `0x22` 读取会话确认一下 `7DF # 03 22 F1 86`，一般在扩展会话进行一些高权限的操作，比如读写数据

![](https://bbs.kanxue.com/upload/attach/202312/837755_QMNMY6C889MM6F3.webp)

切换到编程会话 `7DF # 02 10 02` 此时观察串口可以看到进入到了 `bootloader` 的代码中，一般在这个会话状态进行刷写烧录相关操作

![](https://bbs.kanxue.com/upload/attach/202312/837755_2XKUGGMXYN8BW7N.webp)

当进入非默认会话后如果不及时发送 `3E` 维持会话，过一阵就会退回默认会话

![](https://bbs.kanxue.com/upload/attach/202312/837755_NZM4GF6VDVPXXYH.webp)

### 3E 会话维持

前面 `10` 服务提到，如果不及时发送会话维持，过一阵就会退回到默认会话，会话维持的服务是 `3E`

有两种子功能，`00` 和 `80`

`7DF # 02 3E 00` 表示需要诊断服务端响应

![](https://bbs.kanxue.com/upload/attach/202312/837755_2G8QW9PS3JGWCXK.webp)

`7DF # 02 3E 80` 表示不需要诊断服务端响应，具体表现为你发送之后并不会收到回应

![](https://bbs.kanxue.com/upload/attach/202312/837755_AMWAX3WDFTP7VCN.webp)

### 27 安全访问

在 ECU 中很多数据和功能都需要通过安全访问之后才能访问或使用，安全访问的流程大致为：诊断仪发出安全访问请求，ECU 回复一个 `seed`，诊断仪根据 `seed` 计算出 `key` 返回给 ECU，ECU 检查 `key` 是否正确，若正确则通过

这时候就得注意区分一下物理寻址和功能寻址了，前面通过 `7DF` 进行功能寻址，所有 ECU 都能收到的，虽然我们的实验只有 `STM32` 这一块板子，但实际在车上肯定不是，而且可能一堆不同厂商的 ECU，那解锁安全访问的算法必然也要不同，所以 `27` 服务的时候要使用物理寻址，指定哪个 ECU

但这玩意都是代码里定义的呀，我们咋知道呢，可以使用 `CaringCaribou` 这个工具去探测嗷，比如我这里探测的结果是：`0x721`

![](https://bbs.kanxue.com/upload/attach/202312/837755_WCDVD5TZG4WNE4A.webp)

那接下来就可以请求 `seed` 了，发送 `721 # 02 27 01` 发现报错了

![](https://bbs.kanxue.com/upload/attach/202312/837755_B28ZTE3398SA5S3.webp)

![](https://bbs.kanxue.com/upload/attach/202312/837755_CSTMU6YTR8XAMAE.webp)

那么切换到扩展会话 `7DF # 02 10 03`，然后发送维持会话 `7DF # 02 3E 80`，再次请求便可得到种子，但是我们并不知道怎么从种子算出密钥呀，这时候就要反编译固件分析逻辑了，可以把 `app.hex` 拖到 `IDA` 里面以 ARM 小端格式打开，搜索字符串，我把 `SeedToKey` 字符串加在了代码里方便定位

![](https://bbs.kanxue.com/upload/attach/202312/837755_7CPH5RG7PKJTE7K.webp)

转成伪代码可以看到具体逻辑，`a1` 就是传进来的 `seed`，`v2` 就是计算出来的 `key`

![](https://bbs.kanxue.com/upload/attach/202312/837755_3Z6QW8G28FPY7M6.webp)

让 chatGPT 写个 Python 脚本计算一下

```
def factory_security_seed_to_key(seed):

    seed = int.from_bytes(seed.to_bytes(4, 'little'), 'big')  #切换大小端序

    xor = 0x4368656e  # ASCII "Chen"

    key = seed ^ xor

    print(f"Key after xor: 0x{key:08X}")

    for i in range(32):

        if key & 0x80000000:

            key = (key << 1) ^ xor

        else:

            key = key << 1



    return key & 0xFFFFFFFF # 限制为32位整数



seed_value = 0x34023105  # 用你实际的种子值替换这里的数值

result = factory_security_seed_to_key(seed_value)

print(f"Seed: 0x{seed_value:08X}, Key: 0x{result:08X}")
```

请求种子

![](https://bbs.kanxue.com/upload/attach/202312/837755_8WCXBCUCQV6VAKZ.webp)

计算

![](https://bbs.kanxue.com/upload/attach/202312/837755_THY8AR5Y9KP4B7S.webp)

返回密钥，成功通过安全访问

![](https://bbs.kanxue.com/upload/attach/202312/837755_JBTSE954WT6BSJR.webp)

如果你多次发错密钥，则会得到一个否定响应，表示尝试解锁次数已经达到了设定的上限

![](https://bbs.kanxue.com/upload/attach/202312/837755_9YN44HA9K8DH8UP.webp)

![](https://bbs.kanxue.com/upload/attach/202312/837755_BCBSPF3ZU8QFHH8.webp)

### 11 复位功能

先进扩展会话，再发送 `7DF # 02 11 01`

![](https://bbs.kanxue.com/upload/attach/202312/837755_K6BFCTD2Z3ZVZHN.webp)

可以在串口中观察到设备重启，与按下复位按键效果是一样的

![](https://bbs.kanxue.com/upload/attach/202312/837755_DAJ4NB26JSPR7RJ.webp)

## TODO

因为原来那个卖家是卖 `UDS_bootloader` 的，也就是通过 UDS 刷写升级 `app` 的，因此很多功能并没有实现，也没有实现的必要。等把目前他实现的功能梳理出来之后试试能不能自己写点，实现一个完整的 UDS 练习板

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#汽车安全](https://bbs.kanxue.com/forum-128-1-175.htm)

上传的附件：

- [hex.zip](javascript:void(0);)

  （72.36kb，78次下载）

- [ISO14229 UDS中文翻译版-542页-加目录.pdf](javascript:void(0);)

  （7.86MB，108次下载）
