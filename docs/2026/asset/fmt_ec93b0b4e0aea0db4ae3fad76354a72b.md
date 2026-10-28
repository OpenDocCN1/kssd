
也可以会对某一个 Filter Module 来更改一些 `FilterXXX` 函数，可以用在 Data Bypass Mode 小节中讲的那样。如果 `NdisSetOptionalHandlers` 在 `FilterRestart` 例程中调用，只影响本 Filter Module 的配置，其它 Filter Module 的配置并不会受到影响。

## 2.9 Filter Module发送和接收

在这一节将介绍 NDIS 6.0 Filter Driver 的发送和接收操作。Filter Driver 可以发起一个发送主请求和接收指示或过滤其它驱动的接收和发送指示。Filter Module 在一个微端口适配器（Miniport Adapter）的上层。一个 Filter Module 可以在一个驱动栈上过滤他所联结的下层适配器的所有发送请求和接收指示。绑定在这个适配器上的协议驱动的发送请求也会经过它。

Filter Driver 不直接支持老的基于 `NDIS_PACKET` 结构的发送请求和接收指示。替代老的结构现在采新的 `NET_BUFFER` 结构来进行发送请求和接收指示。也就是说，NDIS 控制由 `NET_BUFFER` 向 `NDIS_PACKET` 的转换。

注意：Filter Driver 来说它可以动态改变一个 Filter Module 的发送和接收的 `FilterXXX` 例程。

下面的小节将介绍关于 Filter Driver 如果进行发送请求和接收指示处理的内容。

### 2.9.1 Filter Driver的缓冲区管理

Filter Driver 创建缓冲区用于拷贝从其它驱动获得的数据或用于发起一个发送请求和接收指示。如果一个 Filter Driver 不能创建一个缓冲区那就不能管理缓冲区池，这样就只能传递发送请求和接收指示。Filter Driver 创建缓冲区用于发送和接收就必须创建 `NET_BUFFER_LIST` 结构池和 `NET_BUFFER` 结构池，下面是创建这些池的例程。

`NdisAllocateNetBufferListPool`
`NdisAllocateNetBufferPool`

下面是从池里分配置结构的例程。

`NdisAllocateNetBufferAndNetBufferList`
`NdisAllocateNetBufferList`
`NdisAllocateNetBuffer`

调用 `NdisAllocateNetBufferAndNetBufferList` 比调用 `NdisAllocateNetBufferList` 后再调用 `NdisAllocateNetBuffer` 更 有 高 效 。 然 而 `NdisAllocateNetBufferAndNetBufferList` 仅 在 `NET_BUFFER_LIST` 中 创 建 一 个 `NET_BUFFER` 结 构 。 要 使 用 `NdisAllocateNetBufferAndNetBufferList` 必 须 在 调 用 `NdisAllocateNetBufferListPool` 时 设 置 `AllocateNetBuffer` 为 `TRUE`。

Filter Driver 要发起一个发送请应该先确害内容和下层驱动对回填空间的要求，一个 Filter Driver 根据重启属性（在 `FilterRestart` 被调用时可以获得）来决定下层驱动回填空间的要求，一个 Filter Driver 应该在 `Restarting` 状态时决定回填和内容的需求。Filter Driver 应该为整个栈分配足够的回填和内容空间。如果需要一个 Filter Driver 可以在 `Restarting` 状态释放缓冲区池并重新分配它们。下面是释放池和释放人池中分配的结构的例程。

`NdisFreeNetBufferListPool`
`NdisFreeNetBufferPool`
`NdisFreeNetBufferList`
`NdisFreeNetBuffer`

驱动在释放 `NET_BUFFER_LIST` 之前应该先释放用 `NdisAllocateNetBuffer` 的 `NET_BUFFER`。如果用 `NET_BUFFER` 是用 `NdisAllocateNetBufferAndNetBufferList` 申请的那个直接调用 `NdisFreeNetBufferList` 释放 `NET_BUFFER_LIST`。

### 2.9.2 在Filter Driver中发送数据

Filter Driver 可以发起一个发送请求或过滤上层驱动引发的一个发送请求。当一个协议驱动调用 `NdisFSendNetBufferLists` 时，提交一个 `NET_BUFFER_LIST` 结构给在驱动栈最顶层的 Filter Module。

#### 1）一个由 Filter Driver 引发的发送请求

下面图中示例了一个 Filter Driver 引发的一个发送操作

![](https://bbs.kanxue.com/upload/attach/201402/203861_zapud7h8th0ofwk.png)

Filter Driver 调用 `NdisFSendNetBufferLists` 发一个定义为 `NET_BUFFERLIST` 结构的网络数据。Filter Driver 必须设置 `NET_BUFFER_LIST` 结构中 `SourceHandle` 的值和 `NdisFSendNetBufferLists` 参数中 `NdisFilterHandle` 的值一样。对 NDIS 驱动来说如果不是自己引发的 `NET_BUFFER_LIST` 结构数据，那么就不应该修改它其中的 `SourceHandle` 成员的值。在调用 `NdisFSendNetBufferLists` 发送前可以，利用 `NET_BUFFER_LIST_INFO` 宏来设置陪伴发送请求的信息。下层的驱动同样可以通这个宏来找到这些信息。一个 Filter Driver 一调用 `NdisSentNetBufferLists`，它就放弃了对 `NET_BUFFER_LIST` 及所有联结资源的所有权。NDIS 会控制发送请求并传递给下层驱动。

NDIS 调用 `FilterSendNetBufferListsComplete` 把发送的结构和数据返还给 Filter Driver。NDIS 可以收集多次 `NdisFSendNetBufferLists` 发送的结构和数据形成一个单链表传递给 `FilterSendNetBufferListsComplete`。除非到 NDIS 调用 `FilterSendNetBufferListsComplete`，否则一个发送请求的当前状态总是未知的。一个 Filter Driver 是不能在 NDIS 调用 `FilterSendNetBufferListsComplete` 返回结构之前对 `NET_BUFFER_LIST` 和其关联的数据做检查的。`FilterSendNetBufferListsComplete` 要完成一个发送请求完成后的任何必要的后继处理。当 NDIS 调用 `FilterSendNetBufferListsComplete` 时，Filter Driver 就重新获地对结构及结构相关资源的所有权。可以在 `FilterSendNetBufferListsComplete` 中释放所有相关的资源和准备下一个 `NdisFSendNetBufferLists` 调用。

NDIS 总是按照 Filter Driver 调用 `NdisFSendNetBufferLists` 提交的顺序传递给下层驱动，但是回返 `FilterSendNetBufferListsComplete` 的顺序则是任意的。Filter Driver 可以请求一个回环发送请求，只要把 `NdisFSendNetBufferLists` 的 `SendFlags` 设置成 `NDIS_SEND_FLAGS_CHECK_FOR_LOOPBACK` 就行了。NDIS 会引发一个包含发送数据的接收包指示。

注意：一个 Filter Driver 应该对自己引发的发送请求保持跟踪并确保在完成时不调用 `NdisFSendNetBufferComplete` 例程。

#### 2）过滤发送请求

下面图示例了过滤一个上层驱动引发的发送请求的过程

![](https://bbs.kanxue.com/upload/attach/201402/203861_ipr4r5l5u3h2sqi.png)

NDIS 调用一个 Filter Driver 的 `FilterSendNetBufferLists` 例程来过滤上层驱动的发送请求。Filter Driver 不能改变其它驱动传来的 `NET_BUFFER_LIST` 结构中的 `SourceHandle` 成员的值。它可以过滤数据并发送过滤的数据到下层驱动。对每一个提交到 `FilterSendNetBufferLists` 的 `NDIS_BUFFER_LIST`，我们可做下面的操作。

1） 可以把缓冲区通过 `NdisFSendBufferLists` 传递给下层驱动，NDIS 保证上下文空间对 Filter Driver 的有效性。过滤驱动可以在发送前修改缓冲区的内容。可以像处理自己引发的发送请求的缓冲区一样处理这个缓冲区。

2） 可以调用 `NdisFSendNetBufferListsComplete` 拒绝传递这个包

3） 排队缓冲区内容到本地的供以后处理。例如要在一定超时后处理或要接收到特定包后才处理等。注意：如果支持这种处理方式就要支持取消请求的操作。

4） 可以拷贝缓冲区并引发一个发送请求。它类似自己引发一个发送请求，但必须先调用 `NdisFSendNetBufferComplete` 返回上层驱动的缓冲区。

发送请求在驱动栈继续完成，当一个微端口驱动调用 `NdisMSendNetBufferListsComplete` 完成一个发送请求时，NDIS 会调用微端口驱动之上最近的 Filter Driver 的 `FilterSendNetBufferLists` 例程。在一个发送操作完成后，Filter Driver 可以做在 `FilterSendNetBufferLists` 中所有修改的相反操作。`FilterSendNetBufferComplete` 返回一个 `NET_BUFFER_LIST` 结构的单链表和发送请求的最终状态给上层的驱动。当最顶层的 Filter Module 的 `FilterSendNetBufferListsComplete` 被调用完成后 NDIS 会调用引发发送请求的协议驱动的 `ProtocolSendNetBufferListsComplete`。如果 Filter Driver 不提供 `FilterSendNetBufferLists` 它还是可以引发一个发送操作的，但它必须提供一个 `FilterSendNetBufferListsComplete` 并且不能在这个例程里把这个事件传递给上层驱动。一个 Filter Driver 可以传递或过滤一个上层驱动的回环请求，要传递一个回环请求，NDIS 会设置 `FilterSendNetBufferLists` 的 `SendFlags` 参数为 `NDIS_SEND_FLAGS_CHECK_FOR_LOOPBACK`，Filter Driver 在调用 `NdisFSendNetBufferLists` 时把这个标记传给它即可。在回环请求的情况下 NDIS 会指示一个包含发送数据的接收包。

通常情况下，如果一个 Filter Driver 修改的任何行为不是 NDIS 提供的标准服务，那么它应该当自己为 NDIS 提供相应的服务。例如，如果一个 Filter Driver 修改了一个硬件地址请求，就必须处理直接到这个新地址回环包。在这种情况下，因为 Filter Driver 已经更改了地址 NDIS 是不能提供一个回环服务的。还有就是如果 Filter Driver 设置了混杂模式那它就不能传递额外的数据给上层接收。

### 2.9.3 在Filter Driver中取消一个发送请求

一个 Filter Driver 可以取消一个自己引发的或上层驱动引发的发送请求

#### 1）取消 Filter Driver 的发送请求

下面图示中展示了一个 Filter Driver 的发送请求被取消的过程

![](https://bbs.kanxue.com/upload/attach/201402/203861_599zf83e8c312go.png)

过滤驱动调用 `NDIS_SET_NET_BUFFER_LIST_CANCEL_ID` 宏为每一个 `NET_BUFFER_LIST` 标记一个取消 Id。在为网络数据分配取消 ID 之前，必须先调用 `NdisGenratePartialCanceId` 获得取消 ID 的高字节。这是为了确保不是驱动不会把一个取消 ID 分配给两个驱动。驱动通常在 `DriverEntry` 调用 `NdisGenratePartialCanceld`，但是驱动可以在不同的时间多次调用它来获得多个取消 ID。要取消被标记过取消 ID 且正在传输的数据，驱动可以调用 `NdisFCancelSendNetBufferLists` 例程来完成。要获得取消 ID 可以用 `NDIS_GET_NET_BUFFER_LIST_CANCEL_ID` 宏来完成。如果一个 Filter Driver 对所有发送的 `NET_BUFFER_LIST` 标记了相同的取消 ID 那它可以用一个 `NdisFCancelSendNetBufferLists` 来取消所有的发送请求。如果把一部发送请求的 `NET_BUFFER_LIST` 标记相同的取消 ID 那么就可以调用一次 `NdisFCancelSendNetBufferLists` 来取消这部分发送请求。

在实现这个功能时 NDIS 会调用下层驱动的取消发送功能。中断正在执行的发送任务后，下层驱动会调用发送完成全程（如：`NdisMSendNetBufferListComplete`）返回指定的 `NET_BUFFER_LIST` 结构并指定返回状态为 `NDIS_STATUS_CANCELLED`，NDIS 依次调用 Filter Driver 的 `FilterSendNetBufferListsComplete` 例程。在 `FilterSendNetBufferListsComplete` 中要用 `NDIS_SET_NET_BUFFER_LIST_CANCEL_ID` 设置取消的 `NET_BUFFER_LIST` 的取消 ID 为 `NULL`，这样是为了防止这个 ID，在 `NET_BUFFER_LIST` 被再次分配时使用。

#### 2）取消上层驱动引发的发送请求

下面图例中展示了这种情况

![](https://bbs.kanxue.com/upload/attach/201402/203861_y2s8m7z2dul9454.png)

上层驱动在取消一个未完成的发送请求时也必须对这个发送请求的 `NET_BUFFER_LIST` 结构设定取消 ID。NDIS 会传递给 Filter Driver 的 `FilterCancelSendNetBufferLists` 一个取消 ID 来取消发送请求的 `NET_BUFFER_LIST` 发送。`FilterCanCelSendNetBufferLists` 下执行下列操作。

1）遍历 Filter Driver 的发送队列，用 `NDIS_GET_NET_BUFFER_LSIT_CANCEL_ID` 获得队列中 `NET_BUFFER_LIST` 的取消 ID 与 `FilterCancelSendBufferLists` 的取消 ID 比较

2）移除队列中取消 ID 和 `FilterCancelSentBufferLists` 中取消 ID 相同的元素

3）调用 `NdisFSendNetBufferListsComplete` 来完成这些 `NET_BUFFER_LIST` 并设定返回状态为 `NDIS_STATUS_CANCELLED`

4）调用 `NdisFCancelSendNetBufferLists` 传递取消发送请求给下层驱动。传递取消 ID 给下层驱动就 Filter Driver 取消自己引发的发关请求一样

### 2.9.4 在Filter Driver中接收数据

在 Filter Driver 中可以引发一个接收指示或过滤下层驱动引发的接收指示。当下层驱动微端口驱动调用 `NdisMIndicateReceiveNetBufferLists` 时，NDIS 会提交一个 `NET_BUFFER_LIST` 给在驱动栈上离微端口驱动最近的那个 Filter Module。

#### 1) Filter Driver 引发一个接收指示

下面图例中展示了这个过程

![](https://bbs.kanxue.com/upload/attach/201402/203861_fz0zhteccsfe6fi.png)

Filter Driver 调用 `NdisFIndicateReceiveNetBufferLists` 来指示发送数据。这个函数通过 `NET_BUFFER_LIST` 结构给上层驱动指示数据。Filter Driver 可以从池中分配这个结构。如果 Filter Driver 设置了 `NdisFIndicateReceiveNetBufferLists` 的状态为 `NDIS_STATUS_SUCCESS`，NDIS 通过驱动的 `FilterReturnNetBufferLists` 返回指示数据。在这种情况下 Filter Driver 失去了对 `NET_BUFFER_LIST` 的所有权直到 `FilterReturnNetBufferLists` 被调用。如果 Filter Driver 在调用
