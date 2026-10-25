
`PUSH`是一条压栈指令，下面这一句的意思是将`EBP`的值压入栈。`main()`函数中，`EBP`为栈帧指针，用来把`EBP`之前的值备份到栈中（`main()`函数执行完毕后，在返回之前，该值会再次恢复）。同时执行，这一条`PUSH`指令后，`ESP`的值变成`12 FF78`（`0012FF7C-4`）

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_WJW6B6XY7UQRC6T.png)

同时执行这一条`PUSH`指令后，`ESP`的值变成`12 FF78`（`0012FF7C-4`）

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_FXW2SJHEUCUDRG3.png)

`MOV`是一条数据传输指令，这条语句的意思是将`ESP`的值传送到`EBP`。也就是说，从这条指令开始，`EBP`就有了与现在`ESP`相同的值，并且直到`main()`函数执行完毕，`EBP`的值始终保持不变。换言之，可以通过`EBP`安全的访问到存储在栈中的函数参数与局部变量。执行完`PUSH`与`MOV`两条指令，栈帧就生成了，即`EBP`被设置好了。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_3CSC5KD2KDZEF3M.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_RGX82TW448NNJD4.png)

在`OD`的栈窗口，右击依次选择`Address-Relative to EBP`。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_8P6JFXCBQAKDCC6.png)

现在的栈内情况被转换成相对于`EBP`的偏移后，能更直观地观察到栈内情况。如下，在栈窗口中可以看到`EBP`的位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_AQSG2NGKG6B26WX.png)

## 2、设置局部变量

开始分析源文件中的变量声明与赋值语句。下面的语句用于在栈中为局部变量(`a`,`b`)分配空间

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_N8UX2E7FD75SJDZ.png)

`SUB`是汇编中的一条减法指令，下面的语句用来将`ESP`的值减去8个字节。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_3VQ87AJZ4DUABD6.png)

执行该指令之后，`ESP`由原来的`12 FF78`变成`12 FF70`。减去这8个字节，是为局部变量（`a`和`b`）开辟空间，以便将它们保存在栈中。由于局部变量`a`与`b`都是`long`型（长整型），它们分别占据4个字节，因此需要开辟8个字节的空间。为函数变量开辟好栈空间后，在`main()`函数内部，无论`ESP`的值如何变化，变量`a`与`b`的栈空间都不会收到损坏。由于`EBP`的值在函数内部是固定不变的，所以可以以它为基准来访问函数的局部变量了。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_M44C4JTBY3J5AKJ.png)

接下来看如下代码，`dword ptr ss:[ebp-0x4]`可以理解为`c`语言中的指针

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_35H55QBNX66VQP9.png)

汇编语言与`C`语言的指针语句格式

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_UDE6TEGTVSYRE64.png)

提示：

`DWORD PTR SS:[EBP-4]`语句中，`SS`是`Stack Segment`的缩写，表示栈段。由于`Windows`中使用的段内存模型，使用时需要指出相关内存属于哪一个区段。起始，32位的`Windows OS`中，`SS`、`DS`、`ES`的值皆为0，所以采用这种方式附上区段并没有什么意义。因为`EBP`与`ESP`是指向栈的寄存器，所以添加上了`SS`寄存器。`DWORD PTR SS：`等字符串可以通过`OD`的相关选项来隐藏。

再次分析一下刚刚的两条`MOV`指令，它们的含义是把数据1与数据2分别保存在`[EBP-4]`与`[EBP-8]`中，即`[EBP-4]`代表局部变量`a`，`[EBP-8]`代表局部变量`b`。执行完两条指令后，栈内情况如下图，可以看到其对应的值分别为1，2

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_F6DMUN4Q9WZA6CN.png)

## 3、add()函数参数传递与调用

代码如下

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_NGFXEQPMCJJEGPU.png)

请看下面五行汇编代码，它描述了`add()`函数的整个过程。

地址`0040103C`处为"`CALL 40100`" 命令，用于调用`40100`处的函数，而`40100`处的函数为`add()`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_BE58VUYH488269Y.png)

函数`add()`接收`a`、`b`两个长整型参数，所以在调用`add()`之前需要把两个参数压入栈，地址`401034—40103B`之间的指令就是这个作用。这一过程中需要注意的是，参数入栈的顺利与`C`语言源码中的参数顺序相反（这种现象被称为函数参数的逆向存储）。换言之，变量`B`首先入栈，接着变量`a`再入栈。执行完`401034—40103B`的指令后，栈内情况如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_P3VRF8Y3PBHQ2ZS.png)

执行`CALL`命令进入被调用函数之前，`CPU`会把函数的返回值压入栈，用做函数执行完毕后的返回地址。从上图可以看出调用`add()`函数之后，下一条命令的地址为`401041`。函数执行完毕后就会跳转到这个地址。这个地址被称为函数的返回地址。执行`CALL`命令后的栈窗口如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_228GCB8BZFERC9Q.png)

接下来执行`add()`函数的汇编指令。指令如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_9M4Y4ZNNEGSP5X9.png)

指令先把`EBP`值（`main()`函数的基址指针）保存到栈中,再把当前的`ESP`的值存储到`EBP`当中。执行完，栈窗口中的情况如下

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_G9EZQTYD9A46PW4.png)

## 4、设置add()函数的局部变量（x,y）

源代码如下:

声明了两个长整型的局部变量（`x`，`y`），并用两个形式参数（`a`，`b`）分别为它们赋初始值。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_Y3VJRYA8UV7XZQQ.png)

执行完下面的指令后，`[EBP+8]`与`[EBP+C]`分别执行参数`a`与`b`，而`[EBP-8]`与`[EBP-4]`分别指向`add()`函数的两个局部变量`x`，`y`。栈窗口如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_XF882TZVEQZNWP2.png)

## 5、ADD运算

源码：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_J3B2QM3HU8SUB4G.png)

汇编指令：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_JR3T2MYRDUF9D6S.png)

小知识：

`EAX`是一种通用寄存器，在算术运算中存储输入输出数据，为函数提供返回值。若向`EAX`输入某个值，该值就会原封不动地返回。执行过程中，栈内状态不变。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_59X55ECGQY3VNR4.png)

## 6、删除函数add()的栈帧以及函数执行完成返回

把`EBP`值赋给`ESP`，与前文的`MOV EBP,ESP`指令对应。把存储到`EBP`的值恢复到`ESP`中

提示：

执行完`MOV ESP,EBP`之后，地址`401003`处的指令不在有效。

`POP EBP`用于恢复函数`add()`开始执行时备份到栈中的`EBP`值，

可以看到`ESP`的值为`0012FF64`，对应的值为`401041`是执行`call 401000`命令时`cpu`存储到栈中的返回地址.

`EBP`恢复为`0012FF78`，是当初`main()`函数的`EBP`值

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_M6N5CJSV64B7DA5.png)

执行下面的语句后，存储在栈中的返回地址被调用

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_9WUWRX372EQ23QT.png)

栈中地址如下：此时栈中的地址与返回到调用`add()`函数之前的状态

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_V6UNYCQYU8DQVY6.png)

## 7、从栈中删除函数add()的参数

执行如下命令，将`ESP+8`。

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_CA7RGERF8QUR6ZU.png)

`ESP`加8的原因可以从上一张图看出，地址`12FF68`与`12FF6C`处存储的是传递给函数`add()`的参数`a`与`b`。函数`add()`执行后，不需要参数`a`与`b`了，所以`ESP+8`将它们从栈中清理掉。

提示：

1、调用`add()`函数之前先使用`push`指令把参数`a,b`压入栈

2、被调函数执行完毕后，函数的调用者(`Caller`)负责清理掉存储在栈中的参数，这种方式称为`cdecl`方式;

反之，被调用者(`Caller`)负责清理保存在栈中的参数，这种方式被称为`stdcall`方式。

这些函数调用规则统称为调用约定。

## 8、调用printf()函数

汇编指令如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_RMUQP6DWCV67TAW.png)

地址`401044`处的 `EAX` 寄存器中存储着函数 `add()`的返回值,它是执行加法运算后的结果值3。地址`40104A`处的 `CALL 401067`命令中调用的是`401067`地址处的函数,它是一个 `C` 标准库函数 `printf` ,所有 `C` 标准库函数都由 `Visual C ++`编写而成(其中包含着数量庞大的函数,在此不详细介绍)。由于上面的 `printf` 函数有2个参数,大小为8个字节(32位寄存器+32位常量=64位=8字节),所以在`40104F`地址处使用 `ADD` 命令,将 `ESP` 加上8个字节,把函数的参数从栈中删除。函数 `printf()`执行完毕并通过 `ADD` 命令删除参数后。栈内情况如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_PPF4F5E25W6TC2P.png)

执行如下指令来设置返回值

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_SSGS3NW8BNGBXJ8.png)

常用于寄存器的初始化操作

执行下面两条命令来对栈帧进行删除以及函数终止。

执行后栈内地址如下图

![图片描述](https://bbs.kanxue.com/upload/attach/202202/890008_4WKUQNFMBFWCWZU.png)

执行完下面的命令，主函数执行完毕并返回，程序执行跳转到返回地址（`401250`）。该地址执行`visual C++`的启动函数区域。

```
00401056 retn
```

参考《逆向工程核心原理》

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-3-25 16:54

被顾忧编辑，原因： 上传相应文件

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

上传的附件：

- [实验文件.zip](javascript:void(0);)

  （51.07kb，10次下载）
