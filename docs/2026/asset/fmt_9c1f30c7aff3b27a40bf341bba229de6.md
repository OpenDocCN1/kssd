
# [原创]初识Frida--Android逆向之Java层hook (一)

---

[TOC]

---

博客同步：<a href="elink@fa3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6G2N6i4W2#2j5h3&6^5j5H3`.`.">访问</a>

## 0x00 文中用到的工具

- Frida
- `jadx-gui` 一个强大的android反编译工具
- `genymotion` 模拟器
- `Python2.7` 以及 `frida-python` 库
- `radare2` 反汇编器
- `pycharm`

## 0x01 hook示例的安装与分析

Frida官网给我们了一个ctf的示例，就以此为例子，开始学习frida在android逆向的使用。

`rps.apk` <a href="elink@b5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6K6i4K6u0r3N6%4u0A6N6r3g2Q4x3X3c8#2M7s2y4Q4x3X3b7J5x3o6p5#2i4K6u0r3M7X3q4%4i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3g2U0j5$3!0F1i4K6u0V1M7i4g2S2L8s2y4Q4x3X3c8U0N6r3k6Q4x3X3b7J5x3o6p5#2i4K6u0r3j5X3W2F1j5i4u0&6i4K6u0r3M7X3g2$3k6i4u0K6k6g2)9J5k6r3g2F1k6$3W2F1k6h3g2J5K9h3&6Y4i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6r3q4H3K9#2)9J5k6o6q4Q4x3V1k6J5M7s2y4Q4x3X3g2S2M7r3D9`.">下载地址</a>

### 安装

使用`genymotion`等类似android模拟器安装好打开，发现这是一个石头剪刀布的游戏应用，简单的玩了一下，没什么特别的，直接分析代码吧，看看到底想干什么。

![](upload/attach/201806/811277\_7KPM722PW735GBM.png)

### 源代码分析

使用`jadx-gui`反编译，发现app没有加壳和混淆，当然一来就加壳和混淆的话对我们就太不友好了，接下分析就简单了，直接看java代码。当然也可以使用`androidkiller`，`jeb`等其他强大的反编译工具。

![](upload/attach/201806/811277\_5XHE4YU4CYZ3247.png)

在`MainActivity`中找到`OnCreate()`方法，可以看到只是简单的声明了button控件以及对应的监听器。

```
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_main);
this.P = (Button) findViewById(R.id.button);
this.S = (Button) findViewById(R.id.button3);
this.r = (Button) findViewById(R.id.buttonR);
this.P.setOnClickListener(this);
this.r.setOnClickListener(this);
this.S.setOnClickListener(this);
this.flag = 0;
}
```

继续查看button的onclick方法，可以看出cpu是通过随机数组出的，其判断输赢的方法在`this.showMessageTask`中。

```
public void onClick(View v) {
if (this.flag != 1) {
this.flag = 1;
((TextView) findViewById(R.id.textView3)).setText("");
TextView tv = (TextView) findViewById(R.id.textView);
TextView tv2 = (TextView) findViewById(R.id.textView2);
this.m = 0;
this.n = new Random().nextInt(3); //随机数0,1,2
tv2.setText(new String[]{"CPU: Paper", "CPU: Rock", "CPU: Scissors"}[this.n]); //随机出石头，剪刀，布
if (v == this.P) {
tv.setText("YOU: Paper");
this.m = 0;
}
if (v == this.r) {
tv.setText("YOU: Rock");
this.m = 1;
}
if (v == this.S) {
tv.setText("YOU: Scissors");
this.m = 2;
}
this.handler.postDelayed(this.showMessageTask, 1000);//输赢判断方法
}
}
```

跟进分析`showMessageTask`，可以看到如果赢了`mainActivity.cnt`会+1，但是一旦输了`cnt`就会置0，而获取flag的要求是我们得获胜1000次，...... :(

```
private final Runnable showMessageTask = new Runnable() {
public void run() {
TextView tv3 = (TextView) MainActivity.this.findViewById(R.id.textView3);
MainActivity mainActivity;
//我方:布 CPU：石头 or 我方：石头 CUP：剪刀 ，则为赢
if (MainActivity.this.n - MainActivity.this.m == 1) {
mainActivity = MainActivity.this;
mainActivity.cnt++;
tv3.setText("WIN! +" + String.valueOf(MainActivity.this.cnt));
//反过来当然是输咯
} else if (MainActivity.this.m - MainActivity.this.n == 1) {
MainActivity.this.cnt = 0;
tv3.setText("LOSE +0");
//一样则打平
} else if (MainActivity.this.m == MainActivity.this.n) {
tv3.setText("DRAW +" + String.valueOf(MainActivity.this.cnt));
//我布 cup:剪刀
} else if (MainActivity.this.m < MainActivity.this.n) {
MainActivity.this.cnt = 0;
tv3.setText("LOSE +0");
} else {
mainActivity = MainActivity.this;
mainActivity.cnt++;
tv3.setText("WIN! +" + String.valueOf(MainActivity.this.cnt));
}
//获胜1000次则能够获取flag
if (1000 == MainActivity.this.cnt) {
tv3.setText("SECCON{" + String.valueOf((MainActivity.this.cnt + MainActivity.this.calc()) \* 107) + "}");
}
MainActivity.this.flag = 0;
}
};
```

简单分析一下获取flag需要的条件，总结有3个办法:

- 分析`calc()`方法能算出答案，但这个方法在so中，得分析汇编代码才行，当然可以尝试使用`ida pro`，F5查看C代码分析，前提是算法不难。
- 获取calc函数的返回值，从而计算答案。
- 还有一个方法就是，直接将`MainActivity.this.cnt`的值构造成1000。

接下来就用frida，使用后两种思路来解这个简单的示例。但在这之前得先了解Frida自带的Messages机制，了解frida怎么从通过一个python脚本发送和接收message消息是一个提升理解frida的好方法。

## 0x02 frida自带的Messages机制与进程交互

先来看看一个Messages的模板，这里用到的语言分别是python和javascript，他们之间的关系是python作为载体，javascript作为在android中真正执行代码。

```
import frida, sys
//hook代码，采用javascript编写
jscode = """
//javascript代码，重点
"""
//自定义回调函数
def on\_message(message, data):
if message['type'] == 'send':
print("[\*] {0}".format(message['payload']))
else:
print(message)
#重点的4行代码
process = frida.get\_usb\_device().attach('应用完整包名')
script = process.create\_script(jscode)
script.on('message', on\_message)
script.load()
sys.stdin.read()
```

当然如果是对此简单的使用，只需要编写jscode，以及填写你要hook的应用完整包名就行了，不过如果单纯只会用可能在以后会被模板限制，所以一探究竟还是很有必要。

可以在cmd中，使用python终端的`help()`函数找到frida库的源代码的绝对路径。

![](upload/attach/201806/811277\_UJMDF7WJ9UZGX2C.png)

接下来就来具体看看这几句代码做了什么事情。

```
process = frida.get\_usb\_device().attach('应用完整包名')
script = process.create\_script(jscode)
script.on('message', on\_message)
script.load()
sys.stdin.read()
```

首先使用了`frida.get_usb_device()`，返回了一个`_get_device`函数，跟进`_get_device`方法。

```
def get\_usb\_device(timeout = 0):
return \_get\_device(lambda device: device.type == 'tether', timeout)
```

在`_get_device`中，通过`get_device_manager()`实例化`DeviceManager`类，并调用该类中的`enumerate_devices()`方法。

```
def \_get\_device(predicate, timeout):
mgr = get\_device\_manager() //获取设备管理
def find\_matching\_device(): //寻找匹配设备
usb\_devices = [device for device in mgr.enumerate\_devices() if predicate(device)]
if len(usb\_devices) > 0:
return usb\_devices[0]
else:
return None
device = find\_matching\_device()
...省略
```

`get_device_manager()`代码

```
def get\_device\_manager():
global \_device\_manager
if \_device\_manager is None:
from . import core
\_device\_manager = core.DeviceManager(\_frida.DeviceManager())
return \_device\_manager
```

`DeviceManager`中`enumerate_devices()`方法，可以看到`enumerate_devices()`方法实际上是返回了一个`Device()`类的实例化对象List。

```
class DeviceManager(object):
def \_\_init\_\_(self, impl):
self.\_impl = impl
def \_\_repr\_\_(self):
return repr(self.\_impl)
//返回了一个Device()类的实例化。
def enumerate\_devices(self):
return [Device(device) for device in self.\_impl.enumerate\_devices()]
def add\_remote\_device(self, host):
return Device(self.\_impl.add\_remote\_device(host))
def remove\_remote\_device(self, host):
self.\_impl.remove\_remote\_device(host)
def get\_device(self, device\_id):
devices = self.\_impl.enumerate\_devices()
if device\_id is None:
return Device(devices[0])
for device in devices:
if device.id == device\_id:
return Device(device)
raise \_frida.InvalidArgumentError("unable to find device with id %s" % device\_id)
def on(self, signal, callback):
self.\_impl.on(signal, callback)
def off(self, signal, callback):
self.\_impl.off(signal, callback)
```

继续跟进`Device`类中的，就找到了`attach()`方法。在attach方法这是设置断点，看看传入的数据。

![](upload/attach/201806/811277\_9ARJ2KFDZA7UWQQ.png)

接下来提供的“应用完整名”是通过`self._pid_of()`函数去找到对应的进程号pid，然后将pid后通过`Session`类初始化。到此第一句代码过程就算是明白了，最终得到的是一个对应进程号pid的`Session`实例化对象process。

```
class Device(object):
def \_\_init\_\_(self, device):
self.id = device.id
self.name = device.name
self.icon = device.icon
self.type = device.type
self.\_impl = device
def \_\_repr\_\_(self):
return repr(self.\_impl)
...节省空间删除部分方法，详细内容可自行查看源码
def kill(self, target):
self.\_impl.kill(self.\_pid\_of(target))
//返回了一个Session的实例化对象
def attach(self, target):
return Session(self.\_impl.attach(self.\_pid\_of(target)))
def inject\_library\_file(self, target, path, entrypoint, data):
return self.\_impl.inject\_library\_file(self.\_pid\_of(target), path, entrypoint, data)
def inject\_library\_blob(self, target, blob, entrypoint, data):
return self.\_impl.inject\_library\_blob(self.\_pid\_of(target), blob, entrypoint, data)
def on(self, signal, callback):
self.\_impl.on(signal, callback)
def off(self, signal, callback):
self.\_impl.off(signal, callback)
def \_pid\_of(self, target):
if isinstance(target, numbers.Number):
return target
else:
return self.get\_process(target).pid
```

第二句，紧接着`process.create_script(jscode)`，可以看到它返回一个`Script`类的实例化，参数不确定。

```
def create\_script(self, \*args, \*\*kwargs):
return Script(self.\_impl.create\_script(\*args, \*\*kwargs))
```

跟进`Script`类，可以找到`on()`方法，在on方法中可以设置自定义回调函数。
