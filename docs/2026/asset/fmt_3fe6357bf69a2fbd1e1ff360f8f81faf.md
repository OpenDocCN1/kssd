
![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_RE44T59KXB62VKB.png)

解密完成后弹出`shellcode`的首地址到`esi`，并将`LoadLibraryA`的`API`地址作为参数传入压栈，最后调用`call`指令跳入`shellcode`区域执行。干扰分析人员的静态分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_EGJFAF8B4CMG93N.png)

使用以下命令将`shellcode`从内存中转存出来。

|  |  |
| --- | --- |
| 1 | `.writemem c:\shellcode.dump` `0x3d0000` `L0x668` `//将数据写到c:\shellcode.dump文件中，从0x3d0000字节开始写，一共写0x688个字节` |

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_BK2WH9RUGFM3R9Q.png)

查看`dump`出来的`shellcode`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_DNF5XKZFQ7B9PUQ.png)

比较数据一致。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_Y4GKKDQRSAV2897.png)

可以将`shellcode`拖入`ida`观察病毒接下来的恶意逻辑。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_MC2GU75JYP52RYD.png)

根据`PE`数据结构获取各个字段数据在内存中的地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_Q2EG7V8G4B3KHGQ.png)

接下来申请一段虚拟内存，内存大小为`0x688`字节。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_3J54DWXTWMMJ7N2.png)

将第一段`shellcode`拷贝到申请的内存空间。 ![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_ZPQGJVQVNYKDSSW.png)

调用`ret`指令弹出新的地址到`eip`执行第二段恶意逻辑。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_H69UMKK2X45TVHY.png)

第一段`shellcode`整体逻辑如下，用来获取指定`API`并执行第二段`shellcode`逻辑。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_BMGRPHT9HG5ACMJ.png)

计算基址。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_CWHJ9FJ4YFNA7HB.png)

申请新的内存空间用于存放被加密的数据。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_6PJUGKY5F972GRR.png)

从全局数据段拷贝第三段恶意代码到申请的虚拟内存。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_7KGAQPNMCSSA9K6.png)

拷贝到虚拟内存后循环加减异或解密出`PE`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_UC6G3VHU2JHVZJ9.png)

修改自身保护属性，目的是为了将当前进程替换成勒索病毒`locky`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_QTH6VWEY8ETPRE3.png)

将解密后的`PE`拷贝到当前进程的模块地址进行`PE`替换。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_J8ABZNNQ7KXCFCC.png)

循环替换`PE`各个区段。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_BAKZV8AKCHYWFXD.png)

最后将`PE`装载修复替换后通过`jmp esi`跳转到替换的`PE`入口点，我们可以通过`$exentry`伪寄存器查看入口点`VA`与`ESI`一致。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_ZDNFXDB88AZDHFP.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_QWRAG7K2295QJ34.png)

查看`dump`下来的勒索程序，发现为`VC++6.0`编写。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_9UEQEMFQGT5YT6V.png)

`ida`查看`locky`勒索软件的`WinMain`线程，可以看到有两个函数。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_NJSJF6B5KEUTXFA.png)

考虑到病毒本体一般会有检测沙箱和调试器等动作。首先设置一个条件断点检测是否有反调试操作。

|  |  |
| --- | --- |
| 1 2 | `bp $exentry` `"ba r 1 $peb+0x02-0x1000 \"ub\";g"` `//当程序执行到入口点对peb所在地址的BeingDebugged字段设置硬件访问断点并打印出读取该字段的汇编代码 （64位系统用windbg获取wow64进程的peb会额外产生0x1000的地址偏差，所以要减去0x1000）` |

设置好条件断点让程序`run`起来，发现打印以下指令并且调试器中断则证明存在调试器检测代码。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_XSDSKQNPJTM8KGS.png)

利用`ida`定位到打印指令的上一条指令地址，即调试器检测。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_VFK69C2SEAH7GDB.png)

我们查看以下`peb`结构体的`BeingDebugged`字段。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_C835VTN2SPX3Q5D.png)

我们可以在`windbg`入口点设置断点，让程序一运行到入口点就立即清理`BeingDebugged`字段达到反反调试的目的。

|  |  |
| --- | --- |
| 1 | `bp $exentry` `"eb $peb-0x1000+0x02 0x0"` |

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_4WUSXQKRT7MCKXZ.png)

如果我们希望在调试的时候不漏掉病毒关键的细节，可以利用`windbg`的条件断点对病毒行为进行过滤，实现`API`行为监控的效果。

|  |  |
| --- | --- |
| 1 2 3 4 | `//创建文件行为过滤，一般文件创建依赖于CreateFileA/W API函数。但是该函数不止有创建文件行为，所以需要通过API的第5个参数进行过滤减少打开文件误报` `//0n3代表打开文件行为属性，过滤成功后打印第二个参数指向的字符串，不支持%s格式化字符串只能通过%mu进行传递` `bp kernelbase!CreateFileW` `".if(poi(esp+0x14) != 0n3){.printf \"Creating File: %mu\",poi(esp+0x4);.echo};g"` `bp kernelbase!CreateFileA` `".if(poi(esp+0x14) != 0n3){.printf \"Creating File: %mu\",poi(esp+0x4);.echo};g"` |

`CreateFileA`函数原型。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_HC5BYMBQFMN2YHZ.png)

第5个参数属性。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_EC4SMVVWJ6EEXTH.png)

设置好条件断点后让程序运行起来，可以发现控制台导出了加密的文件信息。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_S28NZ9HTR5DX64E.png)

进入勒索病毒入口点后发现有大量的垃圾指令，干扰分析人员分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_82DX9EGFEM4X28B.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_3N5TTNKTJQUPUTS.png)

为了方便演示`windbg`的条件断点功能，我这里就偷懒不细致分析了。直接把下述断点设置好让`windbg`自动运行打印出其他关键信息。

|  |  |
| --- | --- |
