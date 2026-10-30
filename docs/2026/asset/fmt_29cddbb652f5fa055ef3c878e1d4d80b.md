
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28 | `from` `cryptography.hazmat.primitives.ciphers` `import` `Cipher, algorithms, modes`  `from` `cryptography.hazmat.primitives.ciphers` `import` `Cipher`  `from` `cryptography.hazmat.primitives.ciphers.algorithms` `import` `AES`  `from` `cryptography.hazmat.primitives.ciphers.modes` `import` `ECB`  `from` `cryptography.hazmat.backends` `import` `default_backend`  `import` `binascii`    `# 转换输入的16进制字符串为字节串`  `key_hex` `=` `"241F632E5907042061014C1A3A45193B"`    `key` `=` `binascii.unhexlify(key_hex)`  `#key = b"[B@7ef20235"`  `# 创建 AES 密钥`  `if` `len``(key)` `=``=` `16``:` `# 128 bit`  `algorithm` `=` `algorithms.AES(key)`  `else``:`  `raise` `ValueError(``"Invalid key size"``)`  `# 创建解密器`  `backend` `=` `default_backend()`  `cipher` `=` `Cipher(algorithm, modes.ECB(), backend``=``backend)`  `decryptor` `=` `cipher.decryptor()`  `# 解密密文`  `#iv = "a20e10ec8ebfd6069c1f84a9cff39e33"`  `iv` `=` `"51653e25e098f5e74e0f152062ee6d5d"`  `encrypted_data` `=` `binascii.unhexlify(iv)`  `decrypted_data` `=` `decryptor.update(encrypted_data)` `+` `decryptor.finalize()`  `# 打印解密结果`  `print``(binascii.hexlify(decrypted_data))` |

运行结果：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_NBZUGEG4CYNWSXU.jpg)

对我们之前的wireshark里的数据都进行解密，然后把结果用XLS表示出来：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_24WTF8V9PKX2MMW.jpg)

在app里的`com.nokelock.blelibrary.mode.order`可以发现和"Write Request"解密出来的头4个字符有关的enum：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_T9N8FYWY88XAP4C.jpg)

添加到表格中：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_QKZZDWRWGARAK6K.jpg)

发现有一个特别令我们欣喜的东西——"OPEN_LOCK"

是否意味着我们把这个value的值通过蓝牙工具发到锁中锁就开了？

答案是否定的，这个value以及它解出来的text是会变的，这一点比照前面adb打印插桩的同样的"0501"开头的数据就知道。所以我们还需要知道怎么获得这个"变"的东西

值得注意的是，同样在这个地方有个"GET_TOKEN"字样：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_G7G8QABJPXWV6DV.jpg)

猜测变化的值和这里有关

至于有没有和"Handle Value Notification"有关的类似的enum，在`com.nokelock.blelibrary.mode.a`里找到了这个：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_SH2636V7DJU7PDF.jpg)

不难看出，这里的代码应该是app接收到蓝牙锁返回的"Handle Value Notification"的value，将它解密过后根据前4个字符去执行对应的功能。而且根据"0202"这个分支里的具体内容已经能大致猜测这里响应的内容是什么了——电池电量，因为有个（v1<=100）的判断，然后对v3这个bool变量赋值判断是否合法。

再结合之前的"0201"为"GET_BATTERY"，一切的说法都看上去合理了起来，我们甚至已经能根据前面的数据，推测响应的时候电池电量为54

但这个说法还有待验证，以及我们还不知道"v0.a()"执行了什么操作，于是需要更深入的逆向

## app逆向（深入）

### 对于说法的验证

首先为了验证前面"发送过后获取响应"的说法，逆向到了`com.nokelock.blelibrary.BLEService`：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_5VJ4CXXUVUDW2TP.jpg)

看到了熟悉的两个UUID，以及它们对应的Characteristic变量

在下面能找到分别对这两个Characteristic的操作：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_8FQRDXDFJ23R7AK.jpg)

从这里就已经能看出在蓝牙通信的时候，app往UUID36f5写入请求信息，然后蓝牙锁通过UUID36f6进行响应

### 探究v0的操作

v0一开始是在这里定义的：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_GCS3KEU7C75JYEV.jpg)

这个`com.nokelock.blelibrary.a.a`是一个抽象接口：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_YYP6UXYPVSWDEGN.jpg)

以v0.a()为例，如果我们想看它具体是怎么实现的，在JEB里点击"a"就能直接查看了：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_S47H57AC6HUC2K9.jpg)

进入红框的部分就能看到具体的实现了：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_M622ZMHF75Y5VEZ.jpg)

可以看到这里有个电池电量低于20就会打印"电量不足"的字样，和使用app时候的情况一致

猜测对于"GET_TOKEN"的响应也在这里面，但是不好直接找出TOKEN的值

于是我们去找"OPEN_LOCK"的地方，因为那里大概率会有响应的"TOKEN"

### 探究"OPEN_LOCK"写入了什么

在jadx里查找"OPEN_LOCK"这个字符串，定位到了这个地方：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_RGXJA425GFQN3KQ.jpg)

红框部分通过对"SET_B_ARRAY"的逆向，可以得到是：

(0x6,0x30,0x30,0x30,0x30,0x30,0x30)

对应了OPEN_LOCK解密的部分数据:06303030303030

其余数据能够在父类`TX_Order`里看出来：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_DKZVNDZKUFB5S8W.jpg)

蓝色部分是0501，绿色部分是刚才的那块数据，红色部分应该是Token，剩下的黄色部分使用随机数进行填充

### 响应的TOKEN

分析`com.nokelock.blelibrary.b.d.a().b()`：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_E8E7TQ53S5BD9MC.jpg)

最后返回的其实是这里的字符数组：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_PUS85HUMZ7V7MYK.jpg)

"c2 == 0"对应的是它上面的一个地方：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_CFWDASU8GJEGQT2.jpg)

结合之前的分析，就能知道GET_TOKEN的响应头应该是"0602"了 那么接下来唯一需要解决的是发送的GET_TOKEN请求长什么样子了

### 如何GET_TOKEN

同样通过查找字符串的手段，我们可以定位到这里：

![img](https://bbs.kanxue.com/upload/attach/202304/943085_75PU8TJGBUN9B8P.jpg)

和"OPEN_LOCK"不一样的地方就是它重写了a()这个方法，可以很容易分析出响应头为"06010101"

剩下的数据都是为了方便最后的加密进行的随机数的填充

那么发送未加密前是"06010101000000000000000000000000"的数据就能成功获取TOKEN的值

## 发送数据尝试开锁

通过上面有点长的分析，我们弄清楚了开锁的重要流程（关于获取电量这些无关紧要的在此不考虑）：

（1）APP发送GET_TOKEN请求，锁返回TOKEN

（2）APP根据锁返回的TOKEN，执行开锁命令

接下来我们通过Bluepy这个python的蓝牙模块尝试发包开锁

### 安装Bluepy

github网址：[IanHarvey/bluepy: Python interface to Bluetooth LE on Linux (github.com)](https://bbs.kanxue.com/elink@1d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6u0j5h3&6t1j5i4u0$3k6i4W2Q4x3V1k6T1L8s2g2W2M7s2V1`.)

安装依赖（使用python3环境）：

|  |  |

| --- | --- |

| 1 | `sudo apt-get install python3-pip libglib2.0-dev` |

安装模块：

|  |  |

| --- | --- |

| 1 | `sudo pip3 install bluepy` |

如果上面两步失败，可以尝试从源码进行安装：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `sudo apt-get install git build-essential libglib2.0-dev`  `` git clone b08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6u0j5h3&6t1j5i4u0$3k6i4W2Q4x3V1k6T1L8s2g2W2M7s2W2Q4x3X3g2Y4K9i4b7`. ``  `cd bluepy`  `python setup.py build`  `sudo python setup.py install` |

### Bluepy的基本使用

官方手册：[bluepy - a Bluetooth LE interface for Python — bluepy 0.9.11 documentation (ianharvey.github.io)](https://bbs.kanxue.com/elink@14cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2S2L8X3S2S2M7Y4k6W2P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5X3I4#2k6i4m8&6i4K6u0V1k6r3!0U0i4K6u0r3)

Bluepy这个模块采用了面向对象编程的思想，在具体操作的时候对设备、服务等等都抽象成了对象以便于操作

#### 扫描

通过Scanner生成一个对象进行扫描：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `scanner` `=` `Scanner()`  `devices` `=` `scanner.scan(timeout``=``3``)`  `print``(``"%-30s %-10s"` `%` `(``"Name"``,` `"Address"``))`  `for` `dev` `in` `devices:`  `print``(``"%-30s %-20s"` `%` `(dev.getValueText(``9``), dev.addr))` |

#### 建立连接

通过Peripheral函数建立连接

|  |  |

| --- | --- |

| 1  2 | `addr` `=` `""`  `conn` `=` `Peripheral(addr)` |

#### 获取Service

|  |  |

| --- | --- |

| 1  2  3 | `services` `=` `conn.getServices()`  `for` `svc` `in` `services:`  `print``(svc.uuid)` |

#### 获取characteristics

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `# 1. 获取所有 Characteristic`  `characteristics` `=` `conn.getCharacteristics()`  `for` `charac` `in` `characteristics:`  `print``(charac.uuid)`    `# 2. 获取特定 Service 下的 Characteristic`  `characteristics` `=` `svc.getCharacteristics()` |

#### 读写Characteristic

|  |  |

| --- | --- |

| 1  2 | `charac.read()`  `charac.write()` |

### 尝试开锁

#### 订阅Notification

经过分析，我们知道36f6这个characteristic的属性是notify

对于notify属性的characteristic，没办法直接读和直接写，需要先向UUID为"0x2902"的descriptor写入"1"进行订阅才能获取所有notify属性的characteristic的响应

另外，我们写一个类来获取响应的handle和value：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `class` `NotifyDelegate(DefaultDelegate):`  `def` `__init__(``self``,params):`  `DefaultDelegate.__init__(``self``)`    `def` `handleNotification(``self``,cHandle,data):`  `global` `TOKEN`  `print``(``"Notification from Handle: 0x"` `+` `format``(cHandle,``"02X"``))`  `TOKEN` `=` `decrypt(binascii.hexlify(data))`  `print``(TOKEN)` |

#### 获取token

通过等待响应即可获取token：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `while` `True``:`  `if` `conn.waitForNotifications(``1.0``):`  `break`  `print``(``"Wating...."``)`  `TX_CHAR.write(binascii.unhexlify(encrypt(pd)))` |

#### 发包开锁

对36f5写入发包数据即可开锁:

|  |  |

| --- | --- |

| 1  2 | `pd` `=` `b``"050106303030303030"``+``TOKEN``+``b``"000000"`  `TX_CHAR.write(binascii.unhexlify(encrypt(pd)))` |

### 完整代码

|  |  |

| --- | --- |
