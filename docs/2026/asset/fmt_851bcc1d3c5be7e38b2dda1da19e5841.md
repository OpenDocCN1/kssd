
```text
fffff880`03302760  fffff880`0336c780 srv!SrvSmbOpen2
fffff880`03302768  fffff880`03333b20 srv!SrvSmbFindFirst2
fffff880`03302770  fffff880`03369f40 srv!SrvSmbFindNext2
fffff880`03302778  fffff880`03335650 srv!SrvSmbQueryFsInformation
fffff880`03302780  fffff880`0335ed20 srv!SrvSmbSetFsInformation
fffff880`03302788  fffff880`03333670 srv!SrvSmbQueryPathInformation
fffff880`03302790  fffff880`0336ccb0 srv!SrvSmbSetPathInformation
fffff880`03302798  fffff880`03331420 srv!SrvSmbQueryFileInformation
fffff880`033027a0  fffff880`03332080 srv!SrvSmbSetFileInformation
fffff880`033027a8  fffff880`03353660 srv!SrvSmbFsctl
fffff880`033027b0  fffff880`0336cae0 srv!SrvSmbIoctl2
fffff880`033027b8  fffff880`03353660 srv!SrvSmbFsctl
fffff880`033027c0  fffff880`03353660 srv!SrvSmbFsctl
fffff880`033027c8  fffff880`0335f4f0 srv!SrvSmbCreateDirectory2
fffff880`033027d0  fffffa80`040a3060
fffff880`033027d8  fffff880`03353460 srv!SrvTransactionNotImplemented

kd> ba e1 fffffa80`040a3060
```

接着，触发第二轮攻击，等待断点触发。以下是行为分析。

## `shellcode`的行为分析

第二轮发送的包，为SMB2协议的包，其`timeout`项被作为控制`flag`。

![](https://bbs.kanxue.com/upload/attach/201705/751592_qx6i7951r365g1z.png)

该域经过计算后，它只有三种值，分别对应了“检查”、“卸载”和“执行程序”，计算公式如下：

![](https://bbs.kanxue.com/upload/attach/201705/751592_eey2uh83tp3tq6s.png)

域的解析判断部分如下：

![](https://bbs.kanxue.com/upload/attach/201705/751592_mqxzjhad6v5fmn7.png)

在经过`ping`包后，会执行`exec`包，执行的程序也包含在（拼接后并解码后的）SMB2包中。其行为为：

1. 找`ntoskrnl`的基址：

   ![](https://bbs.kanxue.com/upload/attach/201705/751592_30l6uil5ik879di.png)

2. 通过`hash`，找出关键的系统调用。

   ![](https://bbs.kanxue.com/upload/attach/201705/751592_xqqgbh7hlsojhk1.png)

3. 通过遍历`PID`，获取`EPROCESS`，再通过比对进程`EPROCESS`中进程名的`HASH`值，获取指定的进程`PID`。

   ![](https://bbs.kanxue.com/upload/attach/201705/751592_0iw2ncxutiv1la5.png)

   ![](https://bbs.kanxue.com/upload/attach/201705/751592_nu7vjo66ygp38a9.png)

   ![](https://bbs.kanxue.com/upload/attach/201705/751592_8qbif7c38cc3xgz.png)

最后，利用`APC`注入，将自带的`DLL`注入到指定系统进程中，`APC`注入看雪中已有相关文章。

![](https://bbs.kanxue.com/upload/attach/201705/751592_h6dg6a4pnrkqqfk.png)

`APC`的回调函数地址是：

![](https://bbs.kanxue.com/upload/attach/201705/751592_kcaswfjvnju9rxd.png)

也就是刚才拷贝的`APC`注入的`shellcode`，用于内存加载`DLL`，并执行其唯一的导出函数`PlayGame`。该`shellcode`及`DLL`我均已经`dump`出，大家可以从附件下载：

![](https://bbs.kanxue.com/upload/attach/201705/751592_eqrjbceegicehd7.png)

`PlayGame`运行后，会释放并执行母体，溢出攻击到此结束，但是悲剧将在新的机器上循环上演。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [EmitSample.dll](javascript:void(0);)
  - （5.05MB，165次下载）
- [shellcodeToLoadDll.bin](javascript:void(0);)
  - （6.00kb，154次下载）
