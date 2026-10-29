
```
class Script(object):
def __init__(self, impl):
self.exports = ScriptExports(self)
self._impl = impl
self._on_message_callbacks = []
self._log_handler = self._on_log
self._pending = {}
self._next_request_id = 1
self._cond = threading.Condition()
impl.on('destroyed', self._on_destroyed)
impl.on('message', self._on_message)
...节省空间删除部分类方法，详细内容可自行查看源码
def load(self):
self._impl.load()
//设置自定义回调函数
def on(self, signal, callback):
if signal == 'message':
self._on_message_callbacks.append(callback)
else:
self._impl.on(signal, callback)
在IDE中可以看到_on_message_callbacks中存放的on_message函数地址。
```

![](upload/attach/201806/811277_MB93UHFRH7DCQQA.png)

接下来调用`load()`方法，在服务端就启动javascript脚本了，至于在`frida-server`服务端怎么执行的，可逆向研究一下`frida-server`，它才是真正的核心。

## 0x03 Javascript代码构造与执行

现在就来使用frida实现刚刚试想的方法。

### 方法一：获取`calc()`返回值

第一种思路就是直接获取`calc()`的返回值，从native函数定义上知道它的返回值是`int`类型，当然直接获取`calc()`函数的返回值是解出问题最简单的方法。

```
public native int calc();
```

那怎么获取`calc()`函数的返回值呢，这个函数在`MainActivity`类中，直接引用该类下的`calc()`方法，不就ok了吗，原理是这样，下面就来构造一下Javascript代码。

```
//Java.Perform 开始执行JavaScript脚本。
Java.perform(function () {
//定义变量MainActivity，Java.use指定要使用的类
var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');
//hook该类下的onCreate方法，重新实现它
MainActivity.onCreate.implementation = function () {
send("Hook Start...");
//调用calc()方法，获取返回值
var returnValue = this.calc();
send("Return:"+returnValue);
var result = (1000+returnValue)*107;
//解出答案
send("Flag:"+"SECCON{"+result.toString()+"}");
}
});
```

JavaScript代码就是这样，如果不是很理解，学习一下JavaScript基础即可，下面看看完整的python脚本。

```
import frida, sys
def on_message(message, data):
if message['type'] == 'send':
print("[*] {0}".format(message['payload']))
else:
print(message)
jscode = """
Java.perform(function () {
var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');
MainActivity.onCreate.implementation = function () {
send("Hook Start...");
var returnValue = this.calc();
send("Return:"+returnValue);
var result = (1000+returnValue)*107;
send("Flag:"+"SECCON{"+result.toString()+"}");
}
});
"""
process = frida.get_usb_device().attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(jscode)
script.on('message', on_message)
script.load()
sys.stdin.read()
```

接下来运行一下，看看能否成功。

步骤如下：

1. 启动模拟器，使用`adb push`将对应架构的`frida-server`文件push到模拟器中`/data/local/tmp`目录下。

2. `adb shell`进入`/data/local/tmp`目录，启动`frida-server`。

3. 开启端口转发

`adb forward tcp:27043 tcp:27043`

`adb forward tcp:27042 tcp:27042`

5. 启动应用后，在命令行等执行python脚本。

因为hook的是应用的`onCreate()`方法，执行python脚本的前提是应用首先启动，这样才能attach到该应用，所以还得返回模拟器桌面重新启动应用，这样它才会执行hook的`onCreate()`方法，结果如下。

![](upload/attach/201806/811277_BDMGS8KD2X3UK4K.png)

### 方法二：修改`cnt`的值为1000

第二种思路也比较简单，我们需要修改`cnt`的值，但如果直接修改`cnt`的初始值为1000的话，在游戏中可能存在不确定因素，比如输了会置0，赢了`cnt`值就变成1001了，所以还得控制一下输赢，而输赢的条件是电脑出什么，所以最终hook的方法就在`onClick()`中。

从`onClick()`中可以知道，控制输赢的在于修改`this.n`和`this.m`的值，再来看看源代码。

```
public void onClick(View v) {
if (this.flag != 1) {
this.flag = 1;
((TextView) findViewById(R.id.textView3)).setText("");
TextView tv = (TextView) findViewById(R.id.textView);
TextView tv2 = (TextView) findViewById(R.id.textView2);
this.m = 0;
//控制电脑出拳
this.n = new Random().nextInt(3);
tv2.setText(new String[]{"CPU: Paper", "CPU: Rock", "CPU: Scissors"}[this.n]);
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
this.handler.postDelayed(this.showMessageTask, 1000);
}
```

来看JavaScript代码怎么写吧

```
Java.perform(function () {
var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');
//hook onClick方法，此处要注意的是onClick方法是传递了一个View参数v
MainActivity.onClick.implementation = function (v) {
send("Hook Start...");
//调用onClick,模拟点击事件
this.onClick(v);
//修改参数
this.n.value = 0;
this.m.value = 2;
this.cnt.value = 999;
send("Success!")
}
});
```

完整python代码

```
import frida, sys
def on_message(message, data):
if message['type'] == 'send':
print("[*] {0}".format(message['payload']))
else:
print(message)
jscode = """
Java.perform(function () {
var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');
MainActivity.onClick.implementation = function (v) {
send("Hook Start...");
this.onClick(v);
this.n.value = 0;
this.m.value = 2;
this.cnt.value = 999;
send("Success!")
}
});
"""
process = frida.get_usb_device().attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(jscode)
script.on('message', on_message)
script.load()
sys.stdin.read()
```

执行python脚本，任意点击按钮，答案就出来了。

![](upload/attach/201806/811277_78VAGC5UDFCNVCW.png)

当然，如果so中的`calc()`函数算法不难的前提，直接使用`ida pro`或者`radare2`分析汇编代码也是可以的。这里给出用`radare2`反汇编出来的代码。可以看到，`calc()`函数就单纯的返回了`int`值7。

![](upload/attach/201806/811277_B63YRPHGR9UDBGZ.png)

## 0x04 总结

- **一般分析流程**

1.反编译apk，分析代码寻找hook点。

2.编写js代码，调用类的方法或者替换。

3.在python中执行即可。

下面一篇会更详细介绍frida的使用。

---

目录

- 0x00 文中用到的工具

- 0x01 hook示例的安装与分析

- [安装](https://bbs.kanxue.com/#安装)

- [源代码分析](https://bbs.kanxue.com/#源代码分析)

- 0x02 frida自带的Messages机制与进程交互

- 0x03 Javascript代码构造与执行

- [方法一：获取`calc()`返回值](https://bbs.kanxue.com/#方法一：获取calc()返回值)

- [方法二：修改`cnt`的值为1000](https://bbs.kanxue.com/#方法二：修改cnt的值为1000)

- 0x04 总结

---

博客同步：[访问](https://bbs.kanxue.com/elink@803K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6G2N6i4W2#2j5h3&6^5j5H3`.`.)

## 0x00 文中用到的工具

- Frida

- jadx-gui 一个强大的android反编译工具

- genymotion模拟器

- Python2.7以及frida-python库

- radare2 反汇编器

- pycharm

## 0x01 hook示例的安装与分析

Frida官网给我们了一个ctf的示例，就以此为例子，开始学习frida在android逆向的使用。

`rps.apk` [下载地址](https://bbs.kanxue.com/elink@7ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6K6i4K6u0r3N6%4u0A6N6r3g2Q4x3X3c8#2M7s2y4Q4x3X3b7J5x3o6p5#2i4K6u0r3M7X3q4%4i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3g2U0j5$3!0F1i4K6u0V1M7i4g2S2L8s2y4Q4x3X3c8U0N6r3k6Q4x3X3b7J5x3o6p5#2i4K6u0r3j5X3W2F1j5i4u0&6i4K6u0r3M7X3g2$3k6i4u0K6k6g2)9J5k6r3g2F1k6$3W2F1k6h3g2J5K9h3&6Y4i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6r3q4H3K9#2)9J5k6o6q4Q4x3V1k6J5M7s2y4Q4x3X3g2S2M7r3D9`.)

### 安装

使用genymotion等类似android模拟器安装好打开，发现这是一个石头剪刀布的游戏应用，简单的玩了一下，没什么特别的，直接分析代码吧，看看到底想干什么。

![](https://bbs.kanxue.com/upload/attach/201806/811277_7KPM722PW735GBM.png)

### 源代码分析

使用jadx-gui反编译，发现app没有加壳和混淆，当然一来就加壳和混淆的话对我们就太不友好了，接下分析就简单了，直接看java代码。当然也可以使用androidkiller，jeb等其他强大的反编译工具。

![](https://bbs.kanxue.com/upload/attach/201806/811277_5XHE4YU4CYZ3247.png)

在`MainActivity`中找到`OnCreate()`方法，可以看到只是简单的声明了button控件以及对应的监听器。

```
  protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        this.P = (Button) findViewById(R.id.button);

        this.S = (Button) findViewById(R.id.button3);

        this.r = (Button) findViewById(R.id.buttonR);

        this.P.setOnClickListener(this);

        this.r.setOnClickListener(this);

        this.S.setOnClickListener(this);

        this.flag = 0;

    }
```

继续查看button的`onclick`方法，可以看出cpu是通过随机数组出的，其判断输赢的方法在`this.showMessageTask`中。
