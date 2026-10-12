
# source insight 分析 WRK 源码

> 原文：[`www.pediy.com/kssd/pediy11/115667.html`](https://www.pediy.com/kssd/pediy11/115667.html)

`source insight` 是一个出色的程序编辑器和代码浏览器，利用好它会给编写和阅读源代码带来极大的方便。想要配置好 `source insight` 可不是一件轻松的事，我比较喜欢深色背景有利于保护眼睛，但同时也必须更改代码颜色来匹配背景色，`source insight` 的很多窗口、选项都可以指定字体、颜色，实在让人犯晕！最麻烦的是源码窗口的配置了，这里简单提一下，`source insight` 的最终配色是由 `Style Properties` 决定的（右键菜单-->`Style Properties`）

![](img/442001cffd43043f1cf286bb688b8afd.jpg)

左边一栏是风格名，选中的是当前光标下的风格名，这些风格之间是有继承关系的，可以看到有很多=号，这代表继承了父风格（`Parent Style`），`pick` 则是让你单独设置，最右边还可以导入、导出、添加风格等。所有风格最终由 `Default Text` 继承而来的，`Font Name` 是灰的，它由 `Document Options`（`Alt+T`）中的当前文档类型 `Screen Fonts` 决定的（所以不同的文件类型，如 C，Python，Java 它们的字体可以不同的）。`Background` 是由 `Options-->Preferences-->Colors` 中的 `Window Background` 决定的，其实 `Default Text` 的背景就是源码窗口的背景。`Foreground` 也一样，`pick` 了它等于设置了 `Options-->Preferences-->Colors` 中的 `Default Text`。我也晕了，汗……

正是这些风格让 `source insight` 五彩缤纷：

![](img/84c0295f7b40b6da55c9cf8c1945910b.jpg)

还要提一句，`source insight` 中还有一个 `Draft View`（`Alt+F12`），在这个模式中会忽略 `Style Properties` 中字体部分设置，统一改用 `Draft View`（见左边栏）中设置的字体。我制作了一个背景为深黑色，字体为 `Fixdays` 的配置文件（包含其它设置）让大家下载：[`download.csdn.net/source/2480193`](http://download.csdn.net/source/2480193)

罗嗦了半天，`WRK` 还没说呢，关于如何使用请看[中文官网](http://www.sourceinsight.com.cn/tech3.htm)的文章 。`source insight` 分析完 `WRK` 后大家肯定会发现连 `IRP`、`DEVICE_OBJECT` 这样的结构都没法解析，其实罪魁祸首就是那些函数宏，比如\_`IRP` 前面的 `DECLSPEC_ALIGN`（），如果用我前面的配置文件调过颜色的话一眼就可以看出，`source insight` 把它当成了 `Declare Function` 了，这些宏大多起修饰作用并无实质影响，但一样的宏在 MS 的代码中数不胜数（如非 `vc6` 的 `crt`，头文件，连 `printf` 都解析不来）。可以看看这个帖子[`topic.csdn.net/u/20100613/16/...33f9ffeb8.html`](http://topic.csdn.net/u/20100613/16/b32fc799-c7f2-4648-9d91-cc933f9ffeb8.html) ，我一开始以为是 BUG，但 `source insight` 提供了解决方案，那就是 `Token Macro`。`Token Macro` 就是 `source insight` 的预处理器，它可以像 C 预处理器那样展开宏，正是为了对抗那些混淆 `source insight` 的宏而设计的。`Token Macro` 是一个以`.tom` 结尾的文件，对 C/C++来说是 `C.tom`，只要把它放在 `source insight` 创建工程的目录下，就可以立马起作用（不用重新 `parse` 一遍，不过有些时候还是 `parse` 一遍好），它会跟程序目录下的全局 `C.tom` 结合去展开宏。`Token Macro` 编写非常简单，跟`#define` 语法差不多（但没它强大）。大家看看我针对 `WRK` 做的 `C.tom` 就明白了，由于大多数函数宏只起修饰作用不会影响代码的解析，所以我把大多数宏做成了空宏。

下载：[`download.csdn.net/source/2480426`](http://download.csdn.net/source/2480426)

另外条件编译也是一个可能让 `source insight` 分析出错的地方，默认它会分析所有的条件分支。但碰到这样的情形就糟糕了：

代码:

```
   1. void MyFunc   
   2. #ifdef XYZ   
   3.     (int param1, int param2)   
   4. #else   
   5.     (long param1, long param2)   
   6. #endif   
   7. {   
   8.     …   
   9. }   
  10. void DoThing(  
  11.     int param1,   
  12. #ifdef ABC   
  13.     int param2)   
  14. #else   
  15.     int param2, param3)   
  16. #endif   
```

很不幸，`WRK` 中也有这样的问题，这个可以通过添加 `Condition` 的方法解决。

我在 `Global Conditons` 中添加了;

\* `MIDL_PASS`    0

\* `__midl`    0

`Project-Specific conditions` 添加了：

\* `_AMD64_`    0

\* `_MSC_VER`    1300

\* `_NDIS_`    1

\* `_NTDDK_`    1

\* `_NTIFS_`    1

\* `_WIN64`    0

\* `_X86_`    1

这样不符合条件的分支代码会不进行解析，用灰色表示。

另外，`source insight` 还有一个小 BUG，用 `typedef` 定义函数指针类型时，当函数指针返回类型（也用 `typedef` 定义）和函数指针类型定义在同一行时，函数指针返回类型无法解析。如：

代码:

```
typedef    
NTSTATUS (*WMIENTRY)(  
……)；//NTSTATUS 无法解析 
 
typedef NTSTATUS   
(*WMIENTRY)(  
……)；//NTSTATUS 可以解析
```

还有 `typedef` 定义函数指针类型时，函数的参数也都是不染色！这个问题似乎没有什么解决方法，用 `Custom Parsing` 也无济于事。

还有一种情况会让 `source insight` 解析出错，比如 `IopfCompleteRequest`、`NtQueryEaFile` 函数，在函数名和`{`之间有一段多行宏，这样 `source insight` 就解析的有问题了。这应该是个 BUG，不知道大家有什么解决办法，我迫不得已只好挪动一下宏的位置！

现在几乎所有的问题都解决了，代码中都染上颜色了吧，呵呵……

![](img/2ff799c4640b8aa857121d812734f137.jpg)

![](img/e0ca6e1261f2fddc6ce357413609f5e4.jpg)

![](img/9f8bf5c63f625a00b068d7efb1856d2d.jpg)

![](img/00a492e2c670cd3634596ee00596615e.jpg)

同时发表于：[`blog.csdn.net/better0332/arch...3/5689193.aspx`](http://blog.csdn.net/better0332/archive/2010/06/23/5689193.aspx)

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `SI.rar`

 |
