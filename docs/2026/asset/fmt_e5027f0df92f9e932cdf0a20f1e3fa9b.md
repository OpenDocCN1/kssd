
`NdisFIndicateReceiveNetBufferLists` 时设置 `ReceiveFlags` 为 `NDIS_RECEIVE_FLAGS_RESOURCES`，在函数返回后 Filter Driver 会立即恢复对 `NET_BUFFER_LIST` 的所有权，这时 Filter Driver 必须立即处理这个 `NET_BUFFER_LIST` 的返回，因为 NDIS 在这种表情下是不会调用 `FilterReturnNetBufferLists` 返回 `NET_BUFFER_LIST` 结构的。

注意：一个 Filter Driver 应该跟踪自己引发的接收指示，确保它在 `FilterReturnNetBufferLists` 中不调用 `NdisFReturnNetBufferLists`。

## 2) 过滤接收指示

下面图例中展示了这个过程

![](https://bbs.kanxue.com/upload/attach/201402/203861_u59mqi7dcv9rkix.png)

NDIS 调用 `FilterReceiveNetBufferLists` 来处理从下层驱动而来的接收指示。当下层驱动调用接收指示例程指示一个网络数据或回环数据时，NDIS 就会调用 Filter Driver 的 `FilterReceiveNetBufferLists` 例程。

如果 `FilterReceiveNetBufferLists` 的 `ReciverFlags` 没有设置 `NDIS_RECEIVE_FLAGS_RESOURCES` 这个标志，那么 Filter Driver 可以保持这个 `NET_BUFFER_LIST` 的所有权一直到 `FilterReturnNetBufferLists` 调用完成。如果设置了，Filter Driver 将不能持有它的所有权，同时说明下层的驱动运行在低资源模式下，Filter Driver 应该以最快的速度处理这个接收指示。

注意：这个标记设置了后，Filter Driver 必须保持 `NET_BUFFER_LIST` 的原有集合设置和状态。例如：在这种情况下可以在 `FilterReceiveNetBufferLists` 返回前对 `NET_BUFFER_LIST` 进行处理，但是必须对它的原始状态进行保存，处理完后必须恢复。

Filter Driver 可以在指示数据给上层驱动之前对这个数据进行过滤。可以对每一个提交到 `FilterReceiveNetBufferLists` 的数据做下列操作。

1. 可以传递这个指示给上层驱动，并且可以在传递之前修改缓冲区中的内容，NDIS 保证上下文空间的有效性。注意：如果 NDIS 设置了 `FilterReceiveNetBufferLists` 参数 `ReceiveFlags` 的 `NDIS_RECEIVE_FLAGS_RESOURCES` 标记，Filter Driver 要传递这个指示给上层驱动，并且只有恢复了对数据的所有权后才能从 `FilterReturnNetBufferLists` 返回。

2. 拒绝数据。如果 `FilterReceiveNetBufferLists` 的 `ReceiveFlags` 没有设置 `NDIS_RECEIVE_FLAGS_RESOURCES` 标记，Filter Drivre 调用 `disFReturnNetBufferList` 返回这个缓冲区数据；如果设置了，则可以什么都不做直接返回来达到拒绝的上的。

3. 排除在本地数据结构中以后再处理。如果 `FilterReceiveNetBufferLists` 的 `ReceiveFlags` 设置 `NDIS_RECEIVE_FLAGS_RESOURCES` 标记，需要拷贝一份 `NET_BUFFER_LIST` 后再返回（在这种情况下是不能直接使用传来的 `NET_BUFFER_LIST` 的，它必须原样返回给下层驱动）。

Filter Driver 调用 `NdisFIndicateNetBufferLists` 传递接收指示给驱动栈上的上层驱动，类似于 Filter Driver 直接引发的接收指示。如果上层驱动保留了对缓冲区（`NET_BUFFER_LIST`）的所有权，NDIS 会调用 Filter Driver 的 `FilterReturnNetBufferLists` 例程。在 `FilterReturnNetBufferLists` 中应该撤消在接收路径上（如在 `FilterReciveNetBufferLists` 中做的一些处理）的操作。当最底层的 Filter Module 完成对缓冲区（`NET_BUFFER_LIST`）的处理后，NDIS 把缓冲区返回给微端口驱动。如果 `FilterReceiveNetBufferLists` 的 `ReceiveFlags` 没有设置 `NDIS_RECEIVE_FLAGS_RESOURCES` 标记，Filter Drivre 调用 `NdisFReturnNetBufferList` 返回这个缓冲区数据；如果设置了，`FilterReceiveNetBufferLists` 直接返回时就把缓冲区返还给了下层微端口驱动。

## 2.10 Filter Module OID请求

NDIS 为每一个适配器定义了对象 ID 值（OID），它包括设备特征，配置设置和统计信息。更多的信息可查看 NDIS OIDs。Filter Driver 可以查询设置下层驱动的参数据或过滤上层驱动的 OID 请求。NDIS 为 6.1 以后的版本提供直接 OID 请求接口（Direct OID Request Interface）。直接 OID 请求方法支持频繁经常性的 OID 查询设置请求。例如：IPSec Offload Version 2（IPSecV2）接口提供的 `OID_TCP_TASK_IPSEC_OFFLOAD_V2_ADD_SA` OID 直接 OID 请求，直接 OID 请求在 NDIS 中提可选的。

下面小节介绍关 OID 请求的信息。

### 2.10.1 在 Filter Driver 中过滤 OID 请求

Filter Driver 可以处理上层驱动引发的 OID 请求，NDIS 调用 Filter Driver 的 `FilterOidRequest` 例程来处理 OID 请求，Filter Driver 需要调用 `NdisFOidRequest` 例程来转发请求给下层驱动。

NDIS 调用 `FilterCancelOidRequest` 来取消一个 OID 请求，当 NDIS 调用 `FilterCancelOidRequest` 时，Filter Driver 应该尽可能快的调用 `NdisFCancelOidRequest`（原文是 `NdisFOidRequest`，我理解这里是一外笔误应该为 `NdisFCancelOidRequest`）。

下图示例了一个 OID 请求

![](https://bbs.kanxue.com/upload/attach/201402/203861_uld9r99nay116hh.png)

Filter Driver 可以从 `FilterOidRequest` 同步和异步完成一个 OID 请求，分别返回 `NDIS_STATS_SUCCESS` 和 `NDIS_STATUS_PENDING` 即可。`FilterOidRequest` 可以用同步的直接完成一个 OID 请求并返回一个错误状态。

如果 Filter Driver 成功控制（处理）了一个 OID 请求，那它必须设置 OID 请求传来 `NDIS_OID_REQUEST` 结构的 `SupportRevision` 成员。这个成员是通知 OID 的发起者所支持的驱动修订版本。更多信息可以参看 Specifying NDIS Version Information。

如果 `FilterOidRequest` 返回 `NDIS_STATUS_PENDING`，就必须在 OID 请求完成后调用 `NdisFOidRequestComplete` 来通知上层驱动求请求完成。在这种情况下，请求的结果通过 `NdisFOidRequestComplete` 的 `OidRequest` 参数返回给上层驱动，并通过 `Status` 参数返回请求完成的最终状态。

如果 `FilterOidRequest` 返回 `NDIS_STATUS_SUCCESS`，通过 `FIlterOidRequest` 的 `OidRequest` 参数返回一个查询结果能上层。这时不调用 `NdisFOidRequestComplete` 例程。

要转发 OID 请求到下层驱动，Filter Driver 必须调用 `NdisFOidRequest`。如果一个 OID 请求不能被转发到下层驱动，应该当立即返回。要完成一个请求且不转发，可以直接返回 `NDIS_STATUS_SUCCESS` 或其它错误状态，或返回 `NDIS_STATUS_PENDING` 后调用 `NdisFOidRequestComplete`。

转发 OID 请求类似于自己引发 OID 请求，可见下节介绍。

当下层驱动完成这个转发的请求时，如果有必要，Filter Driver 可以修改这个请求的结果并递给上层驱动。

一个 Filter Driver 可以从上层驱动接收 OID 请求在 `Restarting`，`Running`，`Pausing` 和 `Paused` 状态时。

注意：像微端口驱动和 Filter Driver 在同一时间只可以接收到一个 OID 请求。NDIS 串行化 OID 请求发送给 Filter Module，在一个 OID 请求没有完成之前是不会收到下一个 OID 请求的。

下面这个例子是在 Filter Driver 中修改一个 OID 请求：

Filter Driver 会给数据包添加一个头的情况。在这种情况下，Filter Driver 在 `OID_GEN_MAXIMUM_FRAME_SIZE` 从下层驱动返回结果后，返回的结果中减去要插入的头的长度。因为在发送数据时 Filter Driver 要插入一个头，而在接收数据时要移除一个头。

### 2.10.2 在 Filter Driver 发起 OID 请求

Filter Driver 可以调用 `NdisFOidRequest` 引发一个 OID 查询和设置请求给下层驱动。下列图示了这个过程。

![](https://bbs.kanxue.com/upload/attach/201402/203861_2bjj1mgusdqaz66.png)

当 Filter Driver 调用 `NdisFOidRequest` 时，NDIS 会调用下层驱动的请求功能。列多信息可以能微端口驱动程序是怎么处理 OID 请求的。

如果 `NdisFOidRequest` 是同步完成的 OID 请求，它会返回 `NDIS_STATUS_SUCCESS`；如果是异步完成，它会返回 `NDIS_STATUS_PENDING`。

要确定哪些信息被下层驱动成功处理了，就要检查返回结果的 `SupportRevision` 成员的值。更多信息可以参看 Specifying NDIS Version Information。

如果 `NdisFOidRequest` 返回 `NDIS_STATUS_PENDING`，NDIS 在 OID 请求完成后调用 `FilterOidRequestComplete` 来通知求请求完成。在这种情况下，请求的结果通过 `NdisFOidRequestComplete` 的 `OidRequest` 参数返回给上层驱动，并通过 `Status` 参数返回请求完成的最终状态。

如果 `NdisFOidRequest` 返回 `NDIS_STATUS_SUCCESS`，通过 `NdisFOidRequest` 的 `OidRequest` 参数返回一个查询结果能上层。这时不调用 `FilterOidRequestComplete` 例程。

一个 Filter Driver 可以调用 `NdisFOidRequest` 引发 OID 请求在 `Restarting`，`Running`，`Pausing` 和 `Paused` 状态。

注意：Filter Driver 必须跟踪这个请求，确保不在 `FilterOidRequestComplete` 中调用 `NdisFOidRequestComplete`（因为请求是自己引发的不能传到上层）。

### 2.10.3 Filter Module 的直接 OID 请求（Direct OID Request）

要支持直接 OID 请求，要在 Filter Driver 的 `NDIS_FILTER_DRIVER_CHARACTERISTICS` 中提供相应的 `FilterXXX` 接口的入口，且 NDIS 的版本要支持 `NdisFDirectXXX` 例程。

直接 OID 请求的接口类似于标准的 OID 请求接口，例如：`NdisFDirectOidRequest` 和 `FilterDirectOidRequest` 例程类似于 `NdisFOidRequest` 和 `FilterOidRequest` 例程。

注意：只有 NDIS 6.1 指定一些 OID 在才可以使用直接 OID 请求，6.1 版本之前的和 6.1 中的一些 OID 是不支持直接 OID 请求的。哪些 OID 支持直接 OID 请求可以参看 OID 接口页。例如看看 `OID_TCP_TASK_IPSEC_OFFLOAD_V2_ADD_SA`。

直接 OID 请求不是串行的，它不像标准的 OID 请求那样被 NDIS 串行化，一个直接 OID 发生时可能存在另一个直接 OID 或标准 OID 请求也发生了。此外，直接 OID 请求例程工作在小于等等 `DISPATCH_LEVEL` 的中断级别上。

要支持直接 OID 请求就要使用文档中规定的标准备接口，下面列出直接 OID 接口与标准 OID 请求接口的对应。

![](https://bbs.kanxue.com/upload/attach/201402/203861_3pir1b7115b2rp5.png)

![](https://bbs.kanxue.com/upload/attach/201402/203861_1dkb9u13zufwifg.png)

## 2.11 Filter Module PnP 事件通知

Filter Driver 可以接收到所有下层微端口驱动要接收的 PnP 事件通知，此外还可以接收到上层协议驱动要接收的网络事件通知道。它可以控制驱动指定的通知事件。

下图展示了一个 Filter Driver 接收 PnP 通知的情形

![](https://bbs.kanxue.com/upload/attach/201402/203861_wea8jn6murxehaq.png)

Filter Driver 提供 `FilterPnpEventNotify` 来接收 NDIS 传递的 PnP 和电源管理事件，它类似于微端口的 `MiniportDevicePnPEventNotify` 全程。

Filter Driver 要下层驱动转发收到的事件，转发事件要用到 `NdisFDevicePnPEventNotify` 例程。

下面图例是 Filter Driver 接收网络通知事件的过程

![](https://bbs.kanxue.com/upload/attach/201402/203861_u8e7lyqneuqcvfo.png)

Filter Driver 提供了 `FilterNetPnpEvent` 例程来处理网络 PnP 和电源管理事件通知。它类似于协议驱动的 `ProtocolNetPnPEvent`。

Filter Driver 需要转发网络 PnP 和电源管理事件给上层驱动。转发这些事件是通 `NdisFNetPnpEvent` 来完成的。

Filter Driver 应该处理驱动栈上的更改（当 NDIS 改变一个正在运行的栈时会先发送相关的 PnP 和电源管理事件，根据实际情况 Filter Driver 也要在自己的程序中处理相应的事件）。更多关于更改运行中的驱动栈的信息可以参看 DDK 中 Modifying a Running Driver Stack。

如查有必要可以处理这些事件，因为在一个 PnP 和电源管理事件通知后 NDIS 才可以执行暂停操作。关于暂停操作在 DDK 中的 Pausing a Driver Stack 里有详细的介绍。

## 2.12 Filter Module 状态指示

Filter Driver 可以提供 `FilterStatus` 例程，当下层驱动报告状态的时候 NDIS 会调用它。此外，Filter Driver 还可以自己引发一个状态指示。

下图示例一个状态指示

![](https://bbs.kanxue.com/upload/attach/201402/203861_50nnlpjaq7ja202.png)

当下层驱动调用一个状态指示例程时（`NdisMIndicateStatusEx` 或 `NdisFIndicateStats`），NDIS 会调用 Filter Driver 的 `FilterStatus` 例程。更多的关于如何从微端口驱动指示状态的信息可以查看 DDK 中的 Adapter Status Indications。

Filter Driver 在 `FilterStatus` 中调用 `NdisFIndicateStatus` 传递一个状态指示给上层驱动。此外，还可以过滤状态指示（不用调用 `NdisFIndicateStatus`）或在调用 `NdisFIndicateStatus` 之前修改状态信息。

Filter Driver 要自己引发一个状态报告，可以在 NDIS 未调用 `FilterStatus` 的情况下调用 `NdisFIndicateStatus`。

在这种情况下，Filter Driver 要设置 `SourceHandle` 成员为 `FilteAttech` 参数提供的
