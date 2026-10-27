
# [原创]易语言程序分析笔记

> 易语言静态编译由于是静态地址，所以已经有工具可以分析了；但是非静态的因为是动态加载库，分析起来还是有点小麻烦的。这个笔记是我之前分析一个易样本时总结的，可能有些地方也不对，凑合看吧。

要点

- 易语言的入口函数特征
- 各种函数的调用特征以及识别
- 针对易语言程序的分析插件

# 1、如何识别易语言程序

## 一、查看文件信息`右键-属性-详细信息`

![](https://bbs.kanxue.com/upload/attach/202209/826671_VTBQ5H9FTURREAP.jpg)

## 二、查看资源文件`Resource Hacker或其他可查看资源文件的工具`

![](https://bbs.kanxue.com/upload/attach/202209/826671_5G3CX2HZKFCN7NZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_E8KFFMTZCCWVHZA.jpg)

## 三、库文件

易语言程序依赖于自实现的库`(.fnr)`执行，针对易语言的编译模式有`静态编译`、`非静态编译`和`独立编译`，如下图：

![](https://bbs.kanxue.com/upload/attach/202209/826671_CSD8EYAR2F5U8V2.jpg)

- 编译：编译程序时，会将所有需要用到的库文件生成到同程序的目录下。

![](https://bbs.kanxue.com/upload/attach/202209/826671_GDYVS7ZWMTMMF25.jpg)

- 非静态编译：编译程序时，将所有用到的库文件写入到程序中，与程序合并，编译出来的文件只有程序本身。

![](https://bbs.kanxue.com/upload/attach/202209/826671_76F67TBJZHH9EBG.jpg)

- 独立编译：与静态编译相同之处是编译出来后只有程序本身，但运行程序时，会将所有用到的库文件写入到临时目录中，然后加载库文件。

![](https://bbs.kanxue.com/upload/attach/202209/826671_KEXJARN8T5PS2A9.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_P2PBWNYBK4ZY7AE.jpg)

## 四、字符串特征

由于易语言是自写的框架，类似于MFC，因此如果未处理编译后的程序，程序中将存在一些固定的字符串。

![](https://bbs.kanxue.com/upload/attach/202209/826671_GD84JUEYGAYYTAJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_SJ3GRXMC96GF7ZE.jpg)

## 五、易语言标准入口

![](https://bbs.kanxue.com/upload/attach/202209/826671_7US8W75EHPAKPPS.jpg)

### ①静态编译

静态编译是直接将库文件与程序合并。

![](https://bbs.kanxue.com/upload/attach/202209/826671_8ZVS7VP8NHY3GWM.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_FU9EGNGR9AJEJJV.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_2TZ62Y6AY3XN2KC.jpg)

### ②非静态编译

该种编译模式会将库文件存放在程序中，在`WinMain`入口会获取读取自身的内容，然后释放出库文件。

![](https://bbs.kanxue.com/upload/attach/202209/826671_YPUBCY5YKKV7MTK.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_KZPDXE8PZFCT9WF.jpg)

随后加载库文件，并调用一个固定函数`GetNewSock`。

![](https://bbs.kanxue.com/upload/attach/202209/826671_YAWUUQQGVB9W5NM.jpg)

最后调用函数

![](https://bbs.kanxue.com/upload/attach/202209/826671_UE5D9Z8WPPF9XGP.jpg)

进入该函数后，会看到`call eax`，`eax`为函数入口。

![](https://bbs.kanxue.com/upload/attach/202209/826671_BJ9WA6CR5QAWRCY.jpg)![](https://bbs.kanxue.com/upload/attach/202209/826671_TZ4WZ7FX82KZAFP.jpg)

### ③黑月编译

黑月编译器是网友针对易语言程序编写的一款编译器，他直接会优化掉前期的初始化代码，包括`clr(c library runtime)`初始化，直接进入到易语言的标准入口。

![](https://bbs.kanxue.com/upload/attach/202209/826671_UGHAUYRK5TKSTU5.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_XUEDDGWFW7YY9EF.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_247G3G3ZZ2GQBZH.jpg)

# 2、易语言的函数类型及特征

易语言的函数大致分有三种，分别为窗体控件的消息函数、用户自定义函数、库函数和组件属性函数。

![](https://bbs.kanxue.com/upload/attach/202209/826671_CVPEFVE88SDJ63D.jpg)

## 一、消息函数

消息函数都是通过固定代码段调用，代码段定位流程如下：

![](https://bbs.kanxue.com/upload/attach/202209/826671_YGTM2BC5BXDYTNE.jpg)

找到如图特征

![](https://bbs.kanxue.com/upload/attach/202209/826671_V7H3FHN8RBMVFRE.jpg)

进入函数后找到最后一`call`局部变量，该位置调用的就是消息函数

![](https://bbs.kanxue.com/upload/attach/202209/826671_CNN5BNCQUNV6C9C.jpg)

将函数对应到`IDA`中可直接定位到流程。

![](https://bbs.kanxue.com/upload/attach/202209/826671_88M3YW8GU6DBRUU.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_X6ZRZWDVDUAK99U.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_HY9Y8TPXUVWTKAM.jpg)

找到 `case 2008`:

![](https://bbs.kanxue.com/upload/attach/202209/826671_UARFQXPWDNRFPMW.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_TXE6FQ3T9C3VPSP.jpg)

## 二、用户自定义函数

用户自定义的函数调用与通常的函数调用方式一致。

![](https://bbs.kanxue.com/upload/attach/202209/826671_8BGH2U97QQPTV8Z.jpg)

## 三、库函数

库函数为易语言自写支持库里的函数，类似Windows的`DLL`文件。函数原型如下。

```
E_FuncCallBack(2,参数一的值，参数一是否可忽略，参数一的类型，参数二的值，参数二是否可忽略，参数二的类型)
```

该函数是一个变参函数，参数一为**欲调用欲调用函数的参数个数**，然后其余的以**三个参数为一组**表示参数的信息。忽略参数的值为布尔值，`0`表示不可以忽略，`1`表示可忽略；其参数类型可参考的值如下（可能不完整）：

|  |  |
| --- | --- |
| 值 | 类型 |
| 0x80000004 | 文本型 |
| 0x80000101 | 文本型 |
| 0x80000000 | 通用型 |
| 0x80000002 | 逻辑型 |
| 0x80000005 | 字节集 |
| 0x80000006 | 子程序指针 |
| 0x80000301 | 数值型 |
| 0x80000601 | 数值型 |
| 0x10001 | 窗体 |

由于易语言的编译模式不同，库函数调用代码也会产生差异。接下来分别解释`静态编译`与`非静态编译`下，函数代码的差异。

- 静态编译

![](https://bbs.kanxue.com/upload/attach/202209/826671_HXF48USXG94N572.jpg)

- 非静态编译

![](https://bbs.kanxue.com/upload/attach/202209/826671_RJCYA6J93UMT4Q8.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_HFFMXPAKBG3D9VD.jpg)

非静态编译的程序，在调用库函数时不再通过静态地址的方式调用，而是通过传入`库文件序号`和`欲调用对应函数的偏移`到寄存器中，然后调用。

库文件序号指的是库文件是程序中加载的第几个文件。序号从`0`开始，易语言程序在运行初期最先加载的库文件为该库文件，记加载序号为`0`；如果再加载一个库文件的话则加载新加载的库文件序号为`1`。**对于调用`krnln.fnr`库文件的函数，易语言程序以隐式形式传递序号**，因此在信息框函数调用的反汇编代码中并未看到传入`0`序号。常用库文件的函数偏移在附录中贴出。

## 四、组件属性函数

函数分为两种，一种是对组件的属性修改，另一种是组件的自带函数。

- 属性修改

**参数一：**父窗口

**参数二：**子窗口

**参数三：**属性下标

**参数四：**未知，一般都为`-0x1`

**参数五：**值

![](https://bbs.kanxue.com/upload/attach/202209/826671_Z7ZBNQCVEG7CM79.jpg)

- 自带函数调用

与消息函数类型。参一：**参数个数**、参二：**值，也就是父窗口ID**、参三：**ID**、参四：**参数类型**

![](https://bbs.kanxue.com/upload/attach/202209/826671_TKK932QQAA245MR.jpg)

因为控件存在于`krnln.fnr`库中，因此在非静态编译中，库函数序号也是隐式传递。部分组件`ID`也在附录中列出。

# 3、易语言的基本数据类型

## 一、字节集

```
typdef struct 字节集

{

   int unkown;

   int length;

   char* bytes;

};
```

# 4、编译器：黑月

程序入口为两个函数。

![](https://bbs.kanxue.com/upload/attach/202209/826671_H4GXJKVQFC2TYXX.jpg)

进入第二个后可以看到如下代码特征。

![](https://bbs.kanxue.com/upload/attach/202209/826671_GBBEP25CY9ZVSAC.jpg)

或者搜索到如下字符串。

![](https://bbs.kanxue.com/upload/attach/202209/826671_8KMYUBQDFPDUVBX.jpg)

# 5、易语言分析插件

## 一、E-Debug（[1f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9Y4q4A6M7$3u0S2i4K6u0r3c8g2)9J5k6r3c8W2j5Y4g2Y4i4K6u0V1M7r3I4#2M7H3`.`.](https://bbs.kanxue.com/elink@43bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9Y4q4A6M7$3u0S2i4K6u0r3c8g2)9J5k6r3c8W2j5Y4g2Y4i4K6u0V1M7r3I4#2M7H3`.`.)）

只能用来分析静态编译的程序。

- OD

![](https://bbs.kanxue.com/upload/attach/202209/826671_Q7JQZ9H4VY5W3HQ.jpg)

- IDA

选择易语言反编译器

![](https://bbs.kanxue.com/upload/attach/202209/826671_7EVW8Y2MVY8HMVZ.jpg)

`IDA`会自动识别由于的函数，并在工具栏新增`易语言`。

![](https://bbs.kanxue.com/upload/attach/202209/826671_M3J6JDF5F3UDQ38.jpg)

新增的项可以自动识别`资源常量`、`窗体`、`消息函数（控件函数）`。

![](https://bbs.kanxue.com/upload/attach/202209/826671_74PP4NJKRYTJVVK.jpg)

- 特征生成

`EDebug`可以手动生成后缀为`sig`的易语言特征，打开配套工具。

![](https://bbs.kanxue.com/upload/attach/202209/826671_Q5AGRTEAUJ5XZDD.jpg)

然后将易语言的支持库拖入即可，最后将生成的`sig`文件放置到对应分析工具的插件目录下。

# 6、花指令

易语言可以设置花指令来干扰程序的逆向分析。在工具栏的`工具-系统配置`的窗口中选择`安全`，可以设置花指令等级，默认为`1`。

![](https://bbs.kanxue.com/upload/attach/202209/826671_PHUKYD32W5KEU6X.jpg)

级别不同，花指令形式也不同。

![](https://bbs.kanxue.com/upload/attach/202209/826671_ED7AXWJMTH63R8W.jpg)

但是每一级别的花指令代码都是固定的，所以可以在`IDA`中使用脚本进行批处理处理，或者使用`OD`的插件去除花指令。

![](https://bbs.kanxue.com/upload/attach/202209/826671_8CMSE6AK9YP9E5N.jpg)

输入要去除花指令的起始地址和地址段大小后，点击执行即可去除。

![](https://bbs.kanxue.com/upload/attach/202209/826671_TQR7CM4YEJJWEXD.jpg)

# 7、针对一些窗口跳转的破解

易语言载入窗口的命令为`载入(,,)`，窗体的类型为`0x10001`，因此可以通过搜索`push 0x10001`来找到该命令，或者有调用窗口函数的代码段来获取窗口`ID`。再将原始加载的窗口`ID`进行替换即可实现窗口跳转。

搜索`push 0x10001`结果。

![](https://bbs.kanxue.com/upload/attach/202209/826671_CH5EZASGNHMWUY5.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_EJ85E9HAGW7E9FW.jpg)

![](https://bbs.kanxue.com/upload/attach/202209/826671_J4EYR6AGA7BZ7SB.jpg)

断点放开后执行，直接能跳转到第二个窗口。

![](https://bbs.kanxue.com/upload/attach/202209/826671_CPXEX9G3XWS42PK.jpg)

# 8、附录

### 一、支持库函数偏移

#### 系统核心支持库——krnln
