
# TDI HOOK 监控 tcp 连接源码+文档

> 原文：[`www.pediy.com/kssd/pediy12/138620.html`](https://www.pediy.com/kssd/pediy12/138620.html)

直接改的 `tdifw`，精简掉乱七八糟的代码，一共 500 来行搞定，写了一个简单的文档。

相信大家都可以看懂，呵呵。

编译：`VC6` + `DDK2000`（`D:\\NTDDK`）

测试：加载驱动，执行一个 `tcp` 连接，`debugview` 会有输出。如果没有输出，有可能是机器上的杀毒软件防火墙之类干扰造成。

## 流程简介

### 一、`hook_tcpip` 函数

首先获取`\\Driver\\Tcpip`驱动设备的`MajorFunction`分发函数并替换成`TdiHookDeviceDispatch`。

### 二、在 `TdiHookDeviceDispatch` 中处理具体的 `IRP`

主要有 `IRP_MJ_CREATE->tdi_create`，

在此可以监控创建的 `TdiTransportAddress` 以及 `connection object`。

如果是创建 `tdi` 地址，设置 `irp` 完成函数 `tdi_create_addrobj_complete`，通过 `tdi_dispatch_complete` 函数处理。

`tdi_dispatch_complete` 处理流程如下：

创建一个自定义的 `TDI_SKIP_CTX` 上下文，里面记录了新的 `irp` 完成函数的参数，并设置当前 `irp` 完成函数为 `tdi_skip_complete`。

在 `tdi_skip_complete` 中：

```
// call new completion 
status = ctx->new_cr(ctx->new_devobj, Irp, ctx->new_context);
```

在 `tdi_create_addrobj_complete` 中参数 `Context` 是 `tdi_create` 时，通过 `TdiBuildInternalDeviceControlIrp` 创建的一个查询 `query_irp`。

该 `IRP` 在 `tdi_create_addrobj_complete2` 时完成，此时已经获取到创建地址信息。由于具体的任务需要，我只需要获取 `tcp` 连接的 `src_port` 以及 `dst_port` 信息。

我们通过 `add_object_hash_pool` 将该地址信息添加到 `obj_hash` 表中，查询 `key` 为该地址 `obj`。

```
add_object_hash_pool(g_hash_pool, (unsigned long)ctx->fileobj, obj_item);
```

接下来会创建 `tdi` 连接 `obj`，类似地址 `obj`，无需特殊处理。

然后是 `TDI_ASSOCIATE_ADDRESS ->tdi_associate_address IRP`，将开始创建的 `local` 地址和连接 `obj` 进行绑定。我们在 `hash` 表中，通过查询地址 `obj`，获取先前记录的 `local` 地址信息，复制到新的 `obj_item` 中，该 `obj` 的 `key` 为连接 `obj` 地址。

最后是 `TDI_CONNECT ->tdi_connect 处理`，直接查询 `hash` 表即可获取该 `conn obj` 对应的 `local` 地址信息，并在调试输出中显示出来。

## 总体的流程如下

`Tdi_create->创建查询 irp->`

`tdi_create_addrobj_complete->tdi_create_addrobj_complete2`

`->建立 addr obj 到 local addr 映射`

`TDI_ASSOCIATE_ADDRESS ->tdi_associate_address`

`->建立 conn obj 到 local addr 映射，通过 addr obj 为 key 查询 hash obj_item`

`TDI_CONNECT ->tdi_connect`

`->通过 conn obj 为 key 查询 hash obj_item 获取地址信息并输出`

## 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `tdi_hook_demo.rar`  
|
