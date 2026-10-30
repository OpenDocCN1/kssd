
```plain
F:\platform-tools>adb logcat -s SewellDinG
--------- beginning of main
03-27 17:18:08.895 31117 31117 D SewellDinG: 05010630303030303082E3C89616017D
03-27 17:18:08.895 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:08.895 31117 31117 D SewellDinG: 6629B62C88A7E50525E92C328AF258E6
03-27 17:18:10.114 31117 31117 D SewellDinG: 732F5CB22C06B0C2D0D17AD31D165805
03-27 17:18:10.114 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:10.114 31117 31117 D SewellDinG: 05020100E3C896010202000000000000
03-27 17:18:11.770 31117 31117 D SewellDinG: 530B1FCA1467A408A321E71F3D152127
03-27 17:18:11.771 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:11.772 31117 31117 D SewellDinG: 050D0100E3C896010202000000000000
03-27 17:18:37.864 31117 31117 D SewellDinG: 05010630303030303082E3C89601635C
03-27 17:18:37.864 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:37.864 31117 31117 D SewellDinG: 3FFA9BE0BA05D2ECC8CF74D2CDB7867C
03-27 17:18:39.263 31117 31117 D SewellDinG: 732F5CB22C06B0C2D0D17AD31D165805
03-27 17:18:39.263 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:39.263 31117 31117 D SewellDinG: 05020100E3C896010202000000000000
03-27 17:18:40.923 31117 31117 D SewellDinG: 530B1FCA1467A408A321E71F3D152127
03-27 17:18:40.923 31117 31117 D SewellDinG: 241F632E5907042061014C1A3A45193B
03-27 17:18:40.924 31117 31117 D SewellDinG: 050D0100E3C896010202000000000000
```

## 解密

在插桩打印的信息中有两个点值得我们注意：

（1）`241F632E5907042061014C1A3A45193B`这个字符串一直在重复，且一直处于一组数据的中部

（2）出现了以`050D`、`0502`、`0501`开头的后面不大一样的疑似有规律的数据

根据我们的插桩，一直处于中部的数据只可能是密钥，那么`241F632E5907042061014C1A3A45193B`就是密钥的某种形式。通过密钥创建的方法`SecretKeySpec`的参数类型可以看出，这是密钥的16进制字符串表示：

![img](upload/attach/202304/943085_9FGK8FGE3THUQVS.jpg)

对chatgpt输入如下指令即可自动生成解密脚本：

```plain
请用python实现一个密钥的16进制字符表示形式是241F632E5907042061014C1A3A45193B的ECB模式无padding的AES-128解密程序，并以16进制字符串的形式输出打印明文
```

解密脚本：

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives.ciphers import Cipher
from cryptography.hazmat.primitives.ciphers.algorithms import AES
from cryptography.hazmat.primitives.ciphers.modes import ECB
from cryptography.hazmat.backends import default_backend
import binascii
# 转换输入的16进制字符串为字节串
key_hex = "241F632E5907042061014C1A3A45193B"
key = binascii.unhexlify(key_hex)
#key = b"[B@7ef20235"
# 创建 AES 密钥
if len(key) == 16: # 128 bit
algorithm = algorithms.AES(key)
else:
raise ValueError("Invalid key size")
# 创建解密器
backend = default_backend()
cipher = Cipher(algorithm, modes.ECB(), backend=backend)
decryptor = cipher.decryptor()
# 解密密文
#iv = "a20e10ec8ebfd6069c1f84a9cff39e33"
iv = "51653e25e098f5e74e0f152062ee6d5d"
encrypted_data = binascii.unhexlify(iv)
decrypted_data = decryptor.update(encrypted_data) + decryptor.finalize()
# 打印解密结果
print(binascii.hexlify(decrypted_data))
```

运行结果：

![img](upload/attach/202304/943085_NBZUGEG4CYNWSXU.jpg)

对我们之前的wireshark里的数据都进行解密，然后把结果用XLS表示出来：

![img](upload/attach/202304/943085_24WTF8V9PKX2MMW.jpg)

在app里的`com.nokelock.blelibrary.mode.order`可以发现和`Write Request`解密出来的头4个字符有关的enum：

![img](upload/attach/202304/943085_T9N8FYWY88XAP4C.jpg)

添加到表格中：

![img](upload/attach/202304/943085_QKZZDWRWGARAK6K.jpg)

发现有一个特别令我们欣喜的东西——`OPEN_LOCK`

是否意味着我们把这个`value`的值通过蓝牙工具发到锁中锁就开了？

答案是否定的，这个`value`以及它解出来的`text`是会变的，这一点比照前面`adb`打印插桩的同样的`0501`开头的数据就知道。所以我们还需要知道怎么获得这个"变"的东西

值得注意的是，同样在这个地方有个`GET_TOKEN`字样：

![img](upload/attach/202304/943085_G7G8QABJPXWV6DV.jpg)

猜测变化的值和这里有关

至于有没有和`Handle Value Notification`有关的类似的enum，在`com.nokelock.blelibrary.mode.a`里找到了这个：

![img](upload/attach/202304/943085_SH2636V7DJU7PDF.jpg)

不难看出，这里的代码应该是app接收到蓝牙锁返回的`Handle Value Notification`的`value`，将它解密过后根据前4个字符去执行对应的功能。而且根据`0202`这个分支里的具体内容已经能大致猜测这里响应的内容是什么了——电池电量，因为有个（`v1<=100`）的判断，然后对`v3`这个bool变量赋值判断是否合法。

再结合之前的`0201`为`GET_BATTERY`，一切的说法都看上去合理了起来，我们甚至已经能根据前面的数据，推测响应的时候电池电量为54

但这个说法还有待验证，以及我们还不知道`v0.a()`执行了什么操作，于是需要更深入的逆向

## app逆向（深入）

### 对于说法的验证

首先为了验证前面"发送过后获取响应"的说法，逆向到了`com.nokelock.blelibrary.BLEService`：

![img](upload/attach/202304/943085_5VJ4CXXUVUDW2TP.jpg)

看到了熟悉的两个UUID，以及它们对应的`Characteristic`变量

在下面能找到分别对这两个`Characteristic`的操作：

![img](upload/attach/202304/943085_8FQRDXDFJ23R7AK.jpg)

从这里就已经能看出在蓝牙通信的时候，app往`UUID36f5`写入请求信息，然后蓝牙锁通过`UUID36f6`进行响应

### 探究v0的操作

`v0`一开始是在这里定义的：

![img](upload/attach/202304/943085_GCS3KEU7C75JYEV.jpg)

这个`com.nokelock.blelibrary.a.a`是一个抽象接口：

![img](upload/attach/202304/943085_YYP6UXYPVSWDEGN.jpg)

以`v0.a()`为例，如果我们想看它具体是怎么实现的，在JEB里点击`a`就能直接查看了：

![img](upload/attach/202304/943085_S47H57AC6HUC2K9.jpg)

进入红框的部分就能看到具体的实现了：

![img](upload/attach/202304/943085_M622ZMHF75Y5VEZ.jpg)

可以看到这里有个电池电量低于20就会打印`电量不足`的字样，和使用app时候的情况一致

猜测对于`GET_TOKEN`的响应也在这里面，但是不好直接找出`TOKEN`的值

于是我们去找`OPEN_LOCK`的地方，因为那里大概率会有响应的`TOKEN`

### 探究`OPEN_LOCK`写入了什么

在jadx里查找`OPEN_LOCK`这个字符串，定位到了这个地方：

![img](upload/attach/202304/943085_RGXJA425GFQN3KQ.jpg)

红框部分通过对`SET_B_ARRAY`的逆向，可以得到是：

`(0x6,0x30,0x30,0x30,0x30,0x30,0x30)`

对应了`OPEN_LOCK`解密的部分数据:`06303030303030`

其余数据能够在父类`TX_Order`里看出来：

![img](upload/attach/202304/943085_DKZVNDZKUFB5S8W.jpg)

蓝色部分是`0501`，绿色部分是刚才的那块数据，红色部分应该是`Token`，剩下的黄色部分使用随机数进行填充

### 响应的TOKEN

分析`com.nokelock.blelibrary.b.d.a().b()`：

![img](upload/attach/202304/943085_E8E7TQ53S5BD9MC.jpg)

最后返回的其实是这里的字符数组：

![img](upload/attach/202304/943085_PUS85HUMZ7V7MYK.jpg)

`c2 == 0`对应的是它上面的一个地方：

![img](upload/attach/202304/943085_CFWDASU8GJEGQT2.jpg)

结合之前的分析，就能知道`GET_TOKEN`的响应头应该是`0602`了，那么接下来唯一需要解决的是发送的`GET_TOKEN`请求长什么样子了

### 如何GET_TOKEN

同样通过查找字符串的手段，我们可以定位到这里：

![img](upload/attach/202304/943085_75PU8TJGBUN9B8P.jpg)

和`OPEN_LOCK`不一样的地方就是它重写了`a()`这个方法，可以很容易分析出响应头为`06010101`

剩下的数据都是为了方便最后的加密进行的随机数的填充

那么发送未加密前是`06010101000000000000000000000000`的数据就能成功获取`TOKEN`的值

## 发送数据尝试开锁

通过上面有点长的分析，我们弄清楚了开锁的重要流程（关于获取电量这些无关紧要的在此不考虑）：

（1）APP发送`GET_TOKEN`请求，锁返回`TOKEN`

（2）APP根据锁返回的`TOKEN`，执行开锁命令

接下来我们通过`Bluepy`这个python的蓝牙模块尝试发包开锁

### 安装Bluepy

github网址：<a href="elink@9bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6u0j5h3&6t1j5i4u0$3k6i4W2Q4x3V1k6T1L8s2g2W2M7s2V1`.">IanHarvey/bluepy: Python interface to Bluetooth LE on Linux (github.com)</a>

安装依赖（使用python3环境）：

```plain
sudo apt-get install python3-pip libglib2.0-dev
```

安装模块：

```plain
sudo pip3 install bluepy
```

如果上面两步失败，可以尝试从源码进行安装：

```plain
sudo apt-get install git build-essential libglib2.0-dev
git clone https://github.com/IanHarvey/bluepy.git
cd bluepy
python setup.py build
sudo python setup.py install
```

### `Bluepy`的基本使用

官方手册：<a href="elink@4aaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2S2L8X3S2S2M7Y4k6W2P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5X3I4#2k6i4m8&6i4K6u0V1k6r3!0U0i4K6u0r3">bluepy - a Bluetooth LE interface for Python — bluepy 0.9.11 documentation (ianharvey.github.io)</a>

`Bluepy`这个模块采用了面向对象编程的思想，在具体操作的时候对设备、服务等等都抽象成了对象以便于操作

#### 扫描

通过`Scanner`生成一个对象进行扫描：

```python
scanner = Scanner()
devices = scanner.scan(timeout=3)
print("%-30s %-10s" % ("Name", "Address"))
for dev in devices:
print("%-30s %-20s" % (dev.getValueText(9), dev.addr))
```

#### 建立连接

通过`Peripheral`函数建立连接

```python
addr = ""
conn = Peripheral(addr)
```

#### 获取`Service`

```python
services = conn.getServices()
for svc in services:
print(svc.uuid)
```

#### 获取characteristics

```python
# 1. 获取所有 Characteristic
characteristics = conn.getCharacteristics()
for charac in characteristics:
print(charac.uuid)
# 2. 获取特定 Service 下的 Characteristic
characteristics = svc.getCharacteristics()
```

#### 读写Characteristic

```python
charac.read()
charac.write()
```

### 尝试开锁

#### 订阅Notification

经过分析，我们知道`36f6`这个`characteristic`的属性是notify

对于`notify`属性的`characteristic`，没办法直接读和直接写，需要先向UUID为`0x2902`的descriptor写入`1`进行订阅才能获取所有`notify`属性的`characteristic`的响应

另外，我们写一个类来获取响应的`handle`和`value`：
