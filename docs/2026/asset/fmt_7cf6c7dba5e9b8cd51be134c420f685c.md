
| 1  2 | `$ cd data``/``data``/``your_app``/``aupk`  `$ ls` `-``l` |

![](https://bbs.kanxue.com/upload/attach/202103/887837_JPGH7VVX3FQ2EWD.png)

其中，`.dex`为整体dump下来的Dex文件，`class.json`记录了Dex文件里所有的类名，前缀数字代表Dex文件的大小。

可以用`findstr`命令来查找某一个类的类名在哪个文件中,如：

|  |  |

| --- | --- |

| 1 | `$ findstr` `/``m` `"SplashActivity"` `*``class``.json` |

![](https://bbs.kanxue.com/upload/attach/202103/887837_UNN6RZB83WR9MQ2.png)

可以看到，"SplashActivity"类在8273364大小的Dex文件中，那么，我们可以通过以下命令，来写入配置文件

以开始对`8273364_ExecuteSwitchImpl_class.json`里所有的类的所有方法进行主动调用

|  |  |

| --- | --- |

| 1 | `$ echo` `"com.klcxkj.zqxy 8273364_ExecuteSwitchImpl_class.json"``>data``/``local``/``tmp``/``aupk.config` |

或者对所有的Dex里的类的所有方法进行主动调用：

|  |  |

| --- | --- |

| 1 | `$ echo` `"com.klcxkj.zqxy"``>data``/``local``/``tmp``/``aupk.config` |

主动调用过程中打印的log:

![](https://bbs.kanxue.com/upload/attach/202103/887837_2DQAFDRR4MC6RP7.png)

有的壳hook了Log函数，导致Log.i()打印不出来消息，但jni层的LOG和Log.e()依然有效，当打印出`Aupk run over`时，代表整个主动调用过程结束，可以在`data/data/you_app/aupk`下看到一些以`method.json`结尾的文件，这些文件包含了Dex文件的函数CodeItem信息，用于后面对Dex文件的修复。

并非等整个主动调用过程结束才会生成`method.json`文件，而是每完成对一个`class.json`文件的解析和调用，就会立即生成对应的`method.json`，所以，利用主动调用的这段时间，你可以先修复已经完成了主动调用的Dex文件，或者去泡杯咖啡。

将脱下来的文件拷贝到电脑:

|  |  |

| --- | --- |

| 1 | `$ adb pull data``/``data``/``your_app``/``aupk` |

开始修复Dex，回填CodeItem：

|  |  |

| --- | --- |

| 1 | `$ dp fix` `-``d` `8273364_ExecuteSwitchImpl``.dex` `-``j` `8273364_ExecuteSwitchImpl_method``.json [``-``-``nolog]` |

等待片刻，即可完成修复：

![](https://bbs.kanxue.com/upload/attach/202103/887837_394BKD4PXFK3CW6.png)

带patched后缀的就是修复后的Dex文件

![](https://bbs.kanxue.com/upload/attach/202103/887837_JJ3XAF8VX4B84QD.png)

反编译看看效果：

![](https://bbs.kanxue.com/upload/attach/202103/887837_SCWER6B4C6EGNNA.png)

### 四.对抗手段

以下为某些场景下的对抗手段（纯YY,并未遇到过），以及个人的一些应对思路

1. 文件检测

![](https://bbs.kanxue.com/upload/attach/202103/887837_DGE2HYMHDDKRVHV.png)

应对方法：修改配置文件和中间文件的名字，路径。

1. 方法，类检测

![](https://bbs.kanxue.com/upload/attach/202104/887837_4M8CQWNDRB8PAVJ.png)

应对方法：更改类名，方法名

2. 插入垃圾类

![](https://bbs.kanxue.com/upload/attach/202103/887837_HQJMQ2KNYJKM9QH.png)

AUPK使用`loadClass`加载类，并不会触发类的`<clinit>`函数,也就不会触发静态代码块里的检测。但在调用静态函数时，如果类未初始化，art会调用`EnsureInitialized`去初始化这个类，此时会调用类的初始化函数，触发静态代码块，其过程如下所示：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `void EnterInterpreterFromInvoke(Thread``*` `self``, ArtMethod``*` `method,` `Object``*` `receiver,`  `uint32_t``*` `args, JValue``*` `result)`  `{`  `...`  `/``/` `Do this after populating the shadow frame` `in` `case EnsureInitialized causes a GC.`  `if` `(method``-``>IsStatic() && UNLIKELY(!method``-``>GetDeclaringClass()``-``>IsInitialized()))`  `{`  `ClassLinker``*` `class_linker` `=` `Runtime::Current()``-``>GetClassLinker();`  `StackHandleScope<``1``> hs(``self``);`  `Handle<mirror::Class> h_class(hs.NewHandle(method``-``>GetDeclaringClass()));`  `if` `(UNLIKELY(!class_linker``-``>EnsureInitialized(``self``, h_class, true, true)))`  `{`  `CHECK(``self``-``>IsExceptionPending());`  `self``-``>PopShadowFrame();`  `return``;`  `}`  `}`  `...`  `}` |

应对方法：

1. 壳为了保证效率，通常在恢复函数指令后，就不会再对其进行抽取了。所以，可以等app运行到一定阶段，确保感兴趣的地方已经被执行过，就可以采用dex整体dump的方式，来dump Dex文件，此时虽然部分函数依然是抽取状态，但关注的部分函数已经解密。

在AUPK中，你可以删除掉所有的`*.dex`文件，然后重新操作一下app,即可完成对Dex文件的重新dump。

|  |  |

| --- | --- |

| 1  2 | `$ cd data``/``data``/``your_app``/``aupk`  `$ rm` `*``.dex` |

2. `frida`提供了一个api:`enumerateLoadedClasses`，该方法可以获取到所有已加载的类，然后通过反射去获取类中的所有函数，并调用Aupk中的`fakeInvoke`，进行主动调用。这样获得的类虽然不全，但包含了app正常使用中所用到的类，且避开了插入的垃圾类。该方法是在类的粒度进行处理，理论上讲，比方法1修复得更完整。伪代码如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `function test() {`  `Java.perform(function () {`  `Java.enumerateLoadedClasses({`  `onMatch: function (className) {`  `var klass` `=` `loadClass(className);`  `var methods` `=` `klass.getMethods();`  `fakeInvoke(methods);`  `}, onComplete: function () {`    `}`  `})`  `});`  `}` |

3. 在`fakeInvoke`前，将当前方法所属的类名写入文件`temp.txt`，当最后一次执行`fakeInvoke`时，清空`temp.txt`.在发起主动调用前，先检查`temp.txt`，将其中`temp.txt`中的项追加到`detectClass.txt`，并跳过所有属于`detectClass.txt`中的类的函数。当壳检测到脱壳机，终止app时，`temp.txt`中的内容为非空。如此反复，到最后主动调用成功时，`detectClass.txt`中则记录着所有检测点所属的类。

### 五.写在最后

从学习脱壳到现在，不知不觉，竟已两月有余，期间遇到过许许多多，一言难尽的问题。对于这类事物，我认为仅仅靠一两篇文章，读者依然很难去复现它。为了能让像我一样的新手去更好的学习和参考，我将同hanbingle大佬一样，将其开源。

AUPK:2fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3k6f1A6c8i4K6u0r3b7g2g2b7d9H3`.`.

DexPatcher:f8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3k6f1A6c8i4K6u0r3c8r3g2^5f1r3q4@1j5$3S2W2M7R3`.`.

参考文献：

[1] hanbingle.[FART：ART环境下基于主动调用的自动化脱壳方案](https://bbs.pediy.com/thread-252630.htm)

[2] Youlor.[Youpk: 又一款基于ART的主动调用的脱壳机](https://bbs.pediy.com/thread-259854.htm)

[3] 邓凡平.深入理解Android：Java虚拟机ART[M].机械工业出版社.2019.04

[4] [FART](https://bbs.kanxue.com/elink@b9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3&6T1K9h3&6Y4L8r3g2F1k6%4W2#2k6g2)9J5c8V1k6m8f1W2b7`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-3-29 15:25

被FeJQ编辑

，原因： 标题多了个前缀

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
