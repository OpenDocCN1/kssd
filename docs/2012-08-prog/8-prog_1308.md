
# 一份没有完成的优盘过滤程序（实现了优盘的只读控制，截获数据是出现问题）

> 原文：[`www.pediy.com/kssd/pediy10/58562.html`](https://www.pediy.com/kssd/pediy10/58562.html)

【文章标题】: 【分享】一份没有完成的优盘过滤程序（实现了优盘的只读控制，截获数据是出现问题）

【文章作者】: `vbcs`

【作者邮箱】: `baoliangster@gmail.com`

【作者主页】: [`http://hi.baidu.com/vbcs003`](http://hi.baidu.com/vbcs003)

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

```c
/*********************************
  相关资料均来自网络。
  代码也绝非完全原创是在网上提供的相应的思路的前提下，查找到相关的资料，大部分是在参考别人的基础上写出
*********************************/
```

以前在网上看到过关于 U 盘的只读操作的控制，网上可以找到一些相关的资料，这两天就不在网上找到的资料汇总了一些形成了今天的这份代码的大部分，虽然可以实现了 U 盘的只读控制，但是当我实现了这个功能的时候，我想是不是可以利用这个只读控制来做一个 U 盘的过滤程序呢，理论上是完全没有问题的。所谓过滤也就是得到他的读写数据。再对这些数据进行处理。

那首要的问题就是得到读写的数据：经过查找资料发现 U 盘的读写就是利用了 `SCSI` 的 `READ`, `WRITE` 命令，这两个命令对应的 `SCSI.H` 文件中的定义是 `SCSIOP_READ`, `SCSIOP_WRITE` 当然这个命令格式有分为 6，10 两种，这两我就步细分了。

关于 `SCSI` 命令的细节格式可以去看查阅 `SCSI` 的规范（我所付的附件中包含）

```c
/*********************************
  READ COMMAND 的命令格式
*********************************/

    Bit 7 6 5 4 3 2 1 0
  Byte
  0 Operation code (08h)
  1 Logical unit number Reserved SILI Fixed
  2 (MSB)
  3 Transfer length
  4 (LSB)
  5 Control
```

走了样了，大家还是看文档吧

命令的第一个字节表示命令操作码，这里就是 `0x08`（其他的可以打开 `scsi.h` 文件来看）
第一位的前三位表示逻辑单元数，后面的几位我们现不管，
第 2，3，4 个字节表示传递的字节数。
最后一个字节我们也不关心。

**当前遇到的问题：**

所以从上面可以看出在这个命令中我们可以知道我们读取了多少个字节，但是读取的字节是放在那里的呢？我们又怎么来得到它呢？？？这就是我一直没有解决的问题，或许我分析的不正确，走了弯路了，如果那位大虾知道的话希望给指点一下，憋了好几天了。

### 我的程序思路

好了上面说了一下我的思路，和遇到的问提，下面我把我的程序的大致结构给大家介绍一下：

因为我们的程序要实时的监控是不是又 U 盘的到来，所以呢我们要处理 `PNP` 历程，和 `ADDDEVICE` 历程，用来动态的得到新来的设备。

在 `AddDevice` 历程中我们要注意的就是一定要设置 `DEVICEOBJECT` 的标志。

`PNP` 历程只对我们关心的几个历程做了处理，其余的就直接返回 `STATUS_SUCCESS` 就好了。

对于我们不关心的历程都放到 `USBFilter` 历程里来处理。

另一个就是由于 U 盘用的是 `SCSI` 命令所以用 `IRP_MJ_READ`, `IRP_MJ_WRITE` 是得不到的要处理 `IRP_MJ_SCSI` 历程来对优盘控制。

#### 示例代码：

```c
if (IrpStack->MajorFunction==IRP_MJ_INTERNAL_DEVICE_CONTROL)
  {
    DbgPrint(CURINFO "IRP_MJ_INTERNAL_DEVICE_CONTROL\n");
    CurSrb=IrpStack->Parameters.Scsi.Srb;
    //cdb = (PCDB)CurSrb->Cdb;
    opCode=CurSrb->Cdb[0];
    if (opCode==SCSIOP_WRITE || opCode==SCSIOP_WRITE6 )
    {
      DbgPrint(CURINFO "opCode==SCSIOP_WRITE\n");
      Len1=(0x00FF0000 & CurSrb->Cdb[2])>>16;
      Len2=(0x0000FF00 & CurSrb->Cdb[3])>>8;
      Len3=(0x000000FF & CurSrb->Cdb[4]);
      DbgPrint(CURINFO "Write Len 0x%x 0x%x 0x%x\n",Len1,Len2,Len3);
      /*****打开此段代码容易引起系统的重启。问题没有解决
      RtlCopyMemory(IoInfo,(UCHAR*)CurSrb->DataBuffer,Len3);
      MyDriverWriteFile(IoInfo,
                Len3,
                pReadFile);*/
    }
    if(opCode==SCSIOP_READ6 || opCode==SCSIOP_READ )
    {
      DbgPrint(CURINFO "opCode==SCSIOP_READ\n");
      Len1=(0x00FF0000 & CurSrb->Cdb[2])>>16;
      Len2=(0x0000FF00 & CurSrb->Cdb[3])>>8;
      Len3=(0x000000FF & CurSrb->Cdb[4]);
      DbgPrint(CURINFO "Read Len 0x%x 0x%x 0x%x\n",Len1,Len2,Len3);
      /*****打开此段代码容易引起系统的重启。问题没有解决
      RtlCopyMemory(IoInfo,(UCHAR*)CurSrb->DataBuffer,Len3);
      MyDriverWriteFile(IoInfo,
                Len3,
                pWriteFile);
    */
    }
```

最后一个问题就是由于开始我是打算吧截获的数据写到一个文件中去所以程序里面涉及到了文件的操作，就是 `ZwCreateFile`,`ZwReadFile`,`ZwWriteFile` 这几个函数，调用的过程是在程序的入口想创建出我们想要的文件来：

#### 示例代码：

```c
/*********************************
  创建读写文件
*********************************/
  RtlZeroMemory(IoInfo,512*0x80);
  ntStatus=CreateLogFile(NULL,&pWriteFile,WriteFilePath);
  if(!NT_SUCCESS(ntStatus))
  {
    DbgPrint("Create WriteFile is falied with error code 0x%08x\n",ntStatus);
    return ntStatus;
  }
  ntStatus=CreateLogFile(NULL,&pReadFile,ReadFilePath);
  if(!NT_SUCCESS(ntStatus))
  {
    DbgPrint("Create ReadFile is falied with error code 0x%08x\n",ntStatus);
    return ntStatus;
  }
```

在以后的过程中我们对我们需要的地方对文件进行读写操作。

这里大家要注意一个问题就是 `IRQL` 的问题，一位文件的操作是在 `passive_level` 这个中断请求级别上的，造我们进行文件操作的时候一定要注意但前的 `IRQL`,可以用 `KeGetCurrentIrql()`来判断一下。因为我是在 `IRP` 的处理的过程中进行文件操作的，所以不能在那里直接对文件进行操作，我用了工作队列的方法来解决（`workeritem`),相应的还有一种办法就是利用系统线程（我就知道这两种），因为我一前些键盘鼠标过滤的时候用的是 `workeritem` 所以这次我就直接吧代码拿过来用来了 我比较懒。

还有一个比较丢人的文件我的 `INF` 文件没写出来，来是有问题我接直接写了一个注册表文件，导入就行了，然后吧 `*.sys` 文件拷贝到 `driver` 目录下。（这两还有个问题就是，好像不得驱动位置放的有问题吧，重启机子之前要把添加的注册表象删除掉（`LowerFilters` 这一项）不然系统会一直重启（我这里是这样 ））

#### 注册表文件：

```ini
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E967-E325-11CE-BFC1-08002BE10318}]
"LowerFilters"="usbfilter"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\usbstorfilter]
"Type"=dword:00000001
"Start"=dword:00000003
"ErrorControl"=dword:00000001
"DisplayName"="usbfilter"
```

好了我就介绍到这里，相关的代码和资料我到一起发上来了。

--------------------------------------------------------------------------------

## 经验总结

其实问题的关键还是在 `SCSI` 命令的处理还有就是对 `PNP` 的处理。

当然队还没有解决的问题希望高手给与指点！！！

谢谢。

--------------------------------------------------------------------------------

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 01 月 19 日 8:51:13

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `usbfilter.rar` |
| --- | --- |
