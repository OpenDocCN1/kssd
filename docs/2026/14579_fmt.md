
# [原创]逆向学习litevm篇

## 乾为天

要不要写这篇文章我是想了很久的，本来不想再写；写文章看似简单，实际上合理的组织语言写一篇即有点深度又易懂的文章是很难的；但自己许下的愿就这样半途而废有悖天地法则，所谓天圆地方，圆圆满满，做事需有头有尾，乃为规律。

我同样秉着和上篇文章一样的态度，尽量用浅显的语言让更多的人了解它。

## 地水师

想研究无线保镖中的安全特性，你就不得不弄懂`litevm`；如`avmp`、`安全对抗`、`数据收集`等，很多核心安全的实现被内置在这个`litevm`中，`sgmain6.3.80`对应`sgavmp6.3.29`中还没有`litevm`，由此可以看出`litevm`和`avmp`是独立的，两者并不存在直接关系。`avmp`的时间早于`litevm`，`litevm`是一个较新版本的安全组件。

其实义如其名`litevm`就是一个小的轻量的`vm`，下面是我给它的一个定义：

![](https://bbs.kanxue.com/upload/attach/202006/665739_ZH2PG5ZZB348ERT.jpg)

## 火天大有

说什么都没有直接拨开它更具说服力，我们先看看它的sdk中暴露的组件包结构形式：

![](https://bbs.kanxue.com/upload/attach/202006/665739_HDNRSUVPN7DETAR.jpg)

在看看sdk中接口形式：

![](https://bbs.kanxue.com/upload/attach/202006/665739_M6N8PQ84Z9WYQ7J.jpg)

看看sdk中`litevm`包装类：

![](https://bbs.kanxue.com/upload/attach/202006/665739_QKQUA84DM5UT48M.jpg)

我们在看看最接近`sgmain`的部分：

![](https://bbs.kanxue.com/upload/attach/202006/665739_UA23UNV395G73YA.jpg)

看了这么多可以反过来证实一下我上面说的话了吧，他们预期将来对外提供一系列的安全方法，这些暴露给上层调用的方法是基于`dex`的方法（至于它将来内部想通过什么方式提供，如`dex vmp`、`jni`反射、或者`elf vmp`、直接字节码，这都不重要）；不过它可不仅仅提供`dex`方式的方法，它内部是个虚拟机，它可以提供任何方式的方法，并且保镖内部调用`lvm`完全和上层没有关系，`vm`执行的是字节码，`vm`还可以`vmp`化。`sgavmp`的字节码加载、签名计算都调用了`lvm`。或许在不久的将来你将无法在无线保镖中找到`sgavmp`的踪影。

## 山天大畜

就我研究的版面目前还未能提供它上层暴露的所有功能，上层也没有显著调用`lvm`的地方，应该是目前版本还未实现那些功能。目前只提供内部调用，在无线保镖内部其他组件通过`lvm command`方式调用`lvm`相关逻辑，这个后面再说。

想研究`lvm`首先需要了解它的创建过程，我们大致看一下它的创建过程。

### 初九：有厉、利己

我们就以从上到下的次序来看它的创建过程，首先通过代理调用创建`lvm`实例：

![](https://bbs.kanxue.com/upload/attach/202006/665739_GXDAUBDBNM9W3X2.jpg)

![](https://bbs.kanxue.com/upload/attach/202006/665739_BQBHUHXX7SC2GK3.jpg)

![](https://bbs.kanxue.com/upload/attach/202006/665739_S4XJH8SAF36MSM6.jpg)

### 九二：舆说輹

在通过`doCommand`调用底层`sgmain`的`command`方法：

![](https://bbs.kanxue.com/upload/attach/202006/665739_RZBSN5FMUYB748D.jpg)

而`0x30d5`=12501，对应`native`层的`command`是1，`0x19`，1，这也恰好是创建`lvm`的地方。

不过从下层看你会发现更多，你会发现1,0x19,x是一系列和`lvm`相关的命令：

![](https://bbs.kanxue.com/upload/attach/202006/665739_YYG6CN2TX5X77VA.jpg)

### 九三：良马遂，利艰贞。日闲舆卫，利有攸往

创建过程比较复杂，会生成很多的数据结构，如外层`litevm`，实例`lvm_inst`、`lvm context`等、同时初始化这些结构，如计数、索引、实例方法等。

![](https://bbs.kanxue.com/upload/attach/202006/665739_JKKJZT3P27J7NGW.jpg)

注册`lvm`方法，这些方法包含系统方法，`jni`方法，和`sgmain`内部方法。

![](https://bbs.kanxue.com/upload/attach/202006/665739_Z2QYJVPR67HEVXV.jpg)

总之为`lvm`准备先决条件。

## 风火家人

### 初九：闲在家，悔亡

在执行`command`时，`sgmain`优先在`lvm_command`中查找`command`，找不到在去普通的`command`中查找。`lvm command`查找主要存在两个主链; 分别由`0x2261`、`0x2622`作为表头，没找到返回0, 外层返回`0x52`。找到后即跳转到指定的`command`处。

但`lvm`字节码最终的执行入口都是`command1,0x19,2`； 执行过程是调用`lvm`获取`lvm_runtime`、打开`runtime`，拷贝外部参数到`lvm`栈，进入`lvm`执行字节码，关闭`lvm_runtime`开关，得到返回结果。

![](https://bbs.kanxue.com/upload/attach/202006/665739_FRQYJ6C4RDWRHYA.jpg)

### 六四：富家，大吉

真正的程序逻辑影射的是`lvm`对应的字节码，字节码的解释执行就对应着程序逻辑的前进。`lvm`执行字节码工作大致如下：

![](https://bbs.kanxue.com/upload/attach/202006/665739_6U2QMKY4RZKF5NZ.jpg)

### 九五：王假有家，勿恤，吉

这里有一个比较有意思的地方，在执行字节码前先解码，这个解码不是解密，而是把外部的字节码转换为`lvm`内部能够解释执行的`code`，我把这种`code`称之为`lvm_vcode`；每条`vcode`都对应一个`handler`，`vcode`的`handler`可以相同，也可以不同；它还可以是`lvm_method`。`lvm`是基于栈的。

如这里列举一条字节码的解密过程（字节码好像也是32位）：

![](https://bbs.kanxue.com/upload/attach/202006/665739_WQBVWR7AGKQCYBA.jpg)

### 上九：有孚威如。终吉

每个`vcode`对应的`handler`都很短，`handler`的主要责任就是执行`vcode`，`vcode`中标明了该执行的操作，如完成类似寄存器移动呀、两个数的累加呀、调用外部函数等。`vcode`运算都是基于`lvm`栈进行的，你认为它有寄存器也可以，认为没有也可以。下面是一个短小的`handler`的图示：

![](https://bbs.kanxue.com/upload/attach/202006/665739_UVVGWWXURKWME2D.jpg)

## 坎为水

我们以第一条`vcode`的执行过程为例，字节码执行的第一步就是找到第一个`vcode`的`handler`（第一个`handler`一般都是保存栈信息）：

![](https://bbs.kanxue.com/upload/attach/202006/665739_DD6KNKF8Y3NT48M.jpg)

然后更新`vm`的`pc`：

![](https://bbs.kanxue.com/upload/attach/202006/665739_9Q3F6N7DFT7767V.jpg)

保存栈信息（第一`vcode`的`handler`）：

![](https://bbs.kanxue.com/upload/attach/202006/665739_KDK8UHHVAXUDDFB.jpg)

进入下一个`vcode`：

![](https://bbs.kanxue.com/upload/attach/202006/665739_URDGJ3WFY22NUNZ.jpg)

## 雷泽归妹

就像`jvm`虚拟机一样，`java`字节码中可以调用`java`方法（自己实现的）、`java`库方法、`native`方法，`lvm`也提供了类似的方法调用，调用`lvm`自实现方法，调用外部方法（库函数如`libc`的方法、`JNI`方法）、调用`sgmain`方法。大致如下图：

![](https://bbs.kanxue.com/upload/attach/202006/665739_S6DXJ4RX6GGPC3P.jpg)

`bridge`列表，有我单独标识的：

![](https://bbs.kanxue.com/upload/attach/202006/665739_Q7EGP43GATQJ9ZW.jpg)

如调用`lvm`方法入口：

![](https://bbs.kanxue.com/upload/attach/202006/665739_B8YDTQP2458MJGA.jpg)

根据解析`vcode`对应调用外部方法的类型调用真正的外部方法：

![](https://bbs.kanxue.com/upload/attach/202006/665739_QDTC377E4FA4XGN.jpg)

外部方法部分截图：

![](https://bbs.kanxue.com/upload/attach/202006/665739_3FW93NA3AQQS99J.jpg)![](https://bbs.kanxue.com/upload/attach/202006/665739_GGJTU93UZXN3PSY.jpg)![](https://bbs.kanxue.com/upload/attach/202006/665739_XCCJWQW53ZA3MYW.jpg)

## 火水未济

好，虽然写的有些急促和不再状态，但我感觉我应该描述清楚了，最后赠上几个数据结构(不保证完全准确性)：

`lvm_method`:

![](https://bbs.kanxue.com/upload/attach/202006/665739_WD2YA6V3B3DCUS3.jpg)

`lvm_vcode`：

![](https://bbs.kanxue.com/upload/attach/202006/665739_6TV7DEVAMSWCH7U.jpg)

`lvm_runtime`:

![](https://bbs.kanxue.com/upload/attach/202006/665739_P9GMC76C67QANZZ.jpg)

好，手工，累坏我了。

本人该项目`github`地址: [31dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6&6L8r3y4S2L8X3N6W2L8q4)9J5c8X3y4J5j5h3y4C8i4K6g2X3L8r3W2@1k6i4k6E0](https://bbs.kanxue.com/elink@889K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6&6L8r3y4S2L8X3N6W2L8q4)9J5c8X3y4J5j5h3y4C8i4K6g2X3L8r3W2@1k6i4k6E0)

**该文档仅仅用来学习交流，用来提高安全门槛，不能用来做恶意事情，否则后果自付！****转载需标明出处，否则发现必究！！！**

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
