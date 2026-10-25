
# [原创]驱动蓝屏后调试DUMP文件

关于分析DUMP文件的一些想法：

在分析一个驱动蓝屏的时候，如果有驱动的PDB文件就很容易找到出错的地方，然是如果没有PDB文件，往往比较麻烦，只能给出个大题的东西，今天调试了一下DUMP文件，有点感触，先来分析一下，（`ProbeForWrite(bb,10,10);bb`为NULL指针）

双机调试的方法不说了，直接开始调试：加载驱动文件，直接蓝屏了，这个时候`WINDBG`给出了一下提示：

`FOLLOWUP_IP`:

`HelloDDK+5ae`

`f7c455ae 8be5            mov     esp,ebp`

`BUGCHECK_STR:  0x7E`

`DEFAULT_BUCKET_ID:  NULL_DEREFERENCE`

`LAST_CONTROL_TRANSFER:  from f7c455ae to 8060d589`

`STACK_TEXT`:

`f7a01c54 f7c455ae 00000000 0000000a 0000000a nt!ProbeForWrite+0x39`

`WARNING: Stack unwind information not available. Following frames may be wrong.`

`f7a01c74 f7c45504 f7a01d4c 805777f1 864d2b10 HelloDDK+0x5ae`

`f7a01c7c 805777f1 864d2b10 860cd000 00000000 HelloDDK+0x504`

`f7a01d4c 80577901 800008a8 00000001 00000000 nt!IopLoadDriver+0x66d`

`f7a01d74 80535c32 800008a8 00000000 865b4020 nt!IopLoadUnloadDriver+0x45`

`f7a01dac 805c71e0 f7a69cf4 00000000 00000000 nt!ExpWorkerThread+0x100`

`f7a01ddc 80542e12 80535b32 00000001 00000000 nt!PspSystemThreadStartup+0x34`

`00000000 00000000 00000000 00000000 00000000 nt!KiThreadStartup+0x16`

`SYMBOL_STACK_INDEX:  1`

`SYMBOL_NAME:  HelloDDK+5ae`

`FOLLOWUP_NAME:  MachineOwner`

`MODULE_NAME: HelloDDK`

`IMAGE_NAME:  HelloDDK.sys`

`DEBUG_FLR_IMAGE_TIMESTAMP:  4d60d8a6`

`STACK_COMMAND:  .cxr 0xfffffffff7a01880 ; kb`

`FAILURE_BUCKET_ID:  0x7E_HelloDDK+5ae`

`BUCKET_ID:  0x7E_HelloDDK+5ae`

这里我只给出其中的一部分代码。

其中有用的信息大家可以自己分析，今天主要想找找出错位置，看其中的一部分

`HelloDDK+5ae`

`f7c455ae 8be5            mov     esp,ebp`

主要看`HelloDDK+5ae`，也就是出错在加载地址的`5ae`偏移处。

然后从新打开虚拟机，挂上调试，这个时候先加载驱动，（先别启动驱动，也就是别让你的驱动蓝），然后再`WINDBG`中`BREAK`一下，然后下`bp          nt!IopLoadDriver+0x66a`，这个是SP3的（P2的为`bp          nt!IopLoadDriver+0x669`），然后再启动驱动，这个时候断下来了，如果所示：

![](https://bbs.kanxue.com/upload/attach/201102/335860_vyiraq0m4w7k3jw.jpg)

然后单布进入（`F11`）

得到如下图：

 ![](https://bbs.kanxue.com/upload/attach/201102/335860_f44mxw7j1zol9t7.jpg)

这个时候就能得到`HelloDDK`的基址为`f7c0d000`然后加上`5ae`得到`f7c0d5ae`

然后反汇编`f7c0d5ae`得到

![](https://bbs.kanxue.com/upload/attach/201102/335860_lbgyrufx3v6rhap.jpg)

看到没有事这个地方或者上面出错了，然后咱往上面看得到

![](https://bbs.kanxue.com/upload/attach/201102/335860_964xek83c2sgkd1.jpg)

这里就能判断出来时这个`CALL`出问题了，实际上是第三个`push 0`的问题

在分析一下

 ![](https://bbs.kanxue.com/upload/attach/201102/335860_nhqpag0ulu22sye.jpg)

这里就知道是`ProbeForWrite`出了问题了，可以用`IDA`在分析一下，然后用二进制工具改，也可以直接用`WINDBG`修改

如下`eb f7c0d5a0 90 90 90 90 90 90 90 90 90 90 90 90 90 90`

然后再反汇编看一下

![](https://bbs.kanxue.com/upload/attach/201102/335860_mpw23w1jmipufuz.jpg)

确定好后运行即可

当然，这里是最简单的东西，实际上好多`NOP`掉了也会出错，不过嘛，往往很多只需要改成`JMP`或直接`JZ`或`JNZ`就能实现你想要的东西，希望对你有帮助。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.jpg](javascript:void(0);)

  （13.48kb，676次下载）

- [2.jpg](javascript:void(0);)

  （21.71kb，673次下载）

- [3.jpg](javascript:void(0);)

  （15.53kb，673次下载）

- [4.jpg](javascript:void(0);)

  （25.56kb，675次下载）

- [5.jpg](javascript:void(0);)

  （16.63kb，674次下载）

- [6.jpg](javascript:void(0);)

  （32.55kb，673次下载）
