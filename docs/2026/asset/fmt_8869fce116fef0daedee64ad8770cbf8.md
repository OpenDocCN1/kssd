
目前做游戏汉化无非就是在资源文件里面去找到相关的文字，将他汉化版重新写回去，受制于原位置长度，当然还要考虑很多因素。资源文件一般情况在`level`，`assets`文件目录中，并与`global-metadata.dat`存在了一定的关联关系，所以完成此项工作还得对这个几个文件的二进制结构有一定得理解才能完成

## 目的

这篇文章提出一种更加方便的汉化思路，基于内存字节对比替换实现汉化，对于汉化人员理论可以简单到只是提供一个简单的字符串映射关系的文本即可实现游戏汉化

## 实践环境

`cache`目录下一个文本以下格式创建文本

1. 第一行为手动指定`get_text()`、`set_text()`的地址
2. 左边列原来的文字，右边是替换成的文字

![](https://bbs.kanxue.com/upload/attach/202009/868525_8B8RKCAGPV8KWA9.png)

3. 打包出来的`so`放在`lib`目录下重打包`apk`

![](https://bbs.kanxue.com/upload/attach/202009/868525_2P728EPSP59MJWJ.png)

4. 记得重打包的时候加上一句`smali`

（代码里虽然是对加载时机做了判断，但实际建议是尽早加载`inject`，先于`libil2cpp`，避免部分汉化无效）

|  |  |
| --- | --- |
| 1  2 | `const-string v1, "inject"`  `invoke-static {v1}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V` |

## 工具

用到的还是我们的还是我们熟悉的

[Inlinehook --- > 持久化hook](https://bbs.kanxue.com/elink@5f2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8i4m8W2N6%4y4Q4x3V1k6p5L8$3u0T1P5b7`.`.)

[frida ---> 动态调试](https://bbs.kanxue.com/elink@034K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2)

## 原理

1. 一切渲染在屏幕上的文字都会通过`U3D`引擎的`UnityEngine.UI.Text`命名空间下的`get_text`和`set_text`（不仅限于以上命名空间`get/set`）
2. 通过`Hook` `libil2cpp.so` 的 `il2cpp_class_get_methods` 函数，用来找到`UnityEngine.UI.Text`命名空间下的`set_text`以及`get_text`的函数地址（当然你也可以使用[`Il2CppDumper`](https://bbs.kanxue.com/elink@272K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7k6i4u0X3j5i4u0W2i4K6u0r3d9h3H3J5b7%4m8H3c8s2g2E0M7r3g2J5)去导出再找）
3. `Hook`上述找到的两个地址，判断修改函数参数或者返回值

## 实践

首先是为了找到`get_text`和`set_text`的地址，具体可以参见我的另一篇文章 [Unity游戏逆向的小工具](https://bbs.pediy.com/thread-261925.htm) 后文中用到的脚本

具体为什么是这样写的可以参考源码，这里不细说：

`Unity/Editor/Data/il2cpp/libil2cpp/vm/Class.cpp`

`SetupMethods` ——> `SetupMethodsLocked`

里面去找他的结构体即可分析得出以下脚本

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `//运行以下脚本输出日志到文本`  `frida -U -f <pkgName> -l hook.js --no-pause -o c:\temp.txt`    `function hook(){`  `var p_size` `=` `Process.pointerSize`  `var soAddr` `=` `Module.findBaseAddress("libil2cpp.so")`  `Interceptor.attach(Module.findExportByName("libil2cpp.so","il2cpp_class_get_methods"),{`  `onEnter:function(args){`  `},`  `onLeave:function(ret){`  `console.error("--------------------------------------------------------")`  `console.log(hexdump(ret,{length:16}))`  `console.log("methodPointer => \t"+ret.readPointer()` `+"\t ===> \t"+ret.readPointer().sub(soAddr))`  `console.log("invoker_method => \t"+ret.add(p_size*1).readPointer()` `+"\t ===> \t"+ret.add(p_size*1).readPointer().sub(soAddr))`  `console.log("MethodName => \t\t"+ret.add(p_size*2).readPointer().readCString())`  `var klass` `=` `ret.add(p_size*3).readPointer()`  `console.log("namespaze => \t\t"+klass.add(p_size*3).readPointer().readCString()+"."` `+klass.add(p_size*2).readPointer().readCString())`  `}`  `})`  `}` |

导出到文本里就可以快速找到实际运行时候的地址以及`ida`里面分析的地址

![](https://bbs.kanxue.com/upload/attach/202009/868525_7N6DU5J3JMBHPZT.png)

当然我们`inlinehook`自然不可能使用`js`代码，那就翻译成`c`代码

（`emmm` 其实也不是不行，咋们的`frida`也是提供了本地`hook`的能力的,只是这里咋们考虑使用`dobby`进行`hook`,还有就是`js`在这里使用不是很方便文件操作，实现通用化的汉化模式，测试学习玩玩还是可以的 所以 嗯...，具体怎么用`frida`本地`hook`，参考我的另一篇文章---> [**见文章底部**](https://bbs.kanxue.com/elink@485K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0b7J5z5e0q4W2k6e0b7J5j5K6b7I4x3R3`.`）

首先是对函数地址的计算

我们从函数的导出函数中只能找到`il2cpp_class_get_methods`，然而`il2cpp_class_get_methods`函数只有一条指令，是不能支持我们使用`inlinehook`的[（`inlinehook`工作会替换前两条指令）](https://bbs.pediy.com/thread-261851.htm)

![](https://bbs.kanxue.com/upload/attach/202009/868525_5WGR28KC4FF5E3Z.png)

所以我们需要计算一下他这条跳转指令真实的跳转位置，用跳转过去的位置作为`hook`点

![](https://bbs.kanxue.com/upload/attach/202009/868525_KK48NBJBYENFG2Y.png)

先明确他的跳转指令是如何构成的

`ea == b eb == bl e1 == bx`

地址的偏移 = （目的地址 - 当前地址 - 8 ）÷ 4

验证一下：

（`1B3AE0` - `193B64` - 8 ）÷ 4 = `7FDD`

下面用`c`来实现计算

![](https://bbs.kanxue.com/upload/attach/202009/868525_CPTWUQ2TKB3EVHU.png)

左移八位去掉操作码，右移八位还原，+ 8 再乘以 4 计算偏移

拿到正常地址我们去`hook`再做点判断即可拿到上述两个函数地址

![](https://bbs.kanxue.com/upload/attach/202009/868525_Z7FMKVK4R3YE82G.png)

我们再使用`frida`去看看看这两个函数究竟是什么参数

![](https://bbs.kanxue.com/upload/attach/202009/868525_75G536HZ5FK4SH2.png)

- 两个记得视作指针
- `64`位 视作`8+8+4+`变长字符串
- `32`位 视作`4+4+4+`变长字符串
- `32`位下前面十二位管他是什么保留就行

  （唯一用到的可能就是第9位到13为字符串长度，记得除以二,实际使用`demo`的时候这个长度好像对修改结果并没有什么影响,所以改不改都没事）

  具体是什么意思参考 ---> [`MonoString`](https://bbs.kanxue.com/elink@73eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3j5i4c8W2d9r3q4U0K9#2)9J5c8V1#2G2L8X3!0e0N6s2u0A6L8X3M7`.`)
- 然后的数据就是`UnionCode`小端存储

  （两个字节表示一个汉字，两个字节表示一个字母，兼容`utf-8`高位补零）
- 最后的八位补齐8个0（中文补齐四个0）

知道这些后我们就用`c`去申请空间，按照这样的规则去构造一块内存区域，替换原先的返回值或者参数指针即可实现内容的替换

|  |  |
| --- | --- |
