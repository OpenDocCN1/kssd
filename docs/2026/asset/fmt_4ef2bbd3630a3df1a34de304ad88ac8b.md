
为了和其他 callout 合作完成包的检查、修改和连接重定向，在包被 pend 之前，callout 必须丢弃原始的包，即在 `classifyFn()` 返回前清除 `FWPS_CLASSIFY_OUT0` 结构中的 `FWPS_RIGHT_ACTION_WRITE` 标志位。如果 `FWPS_RIGHT_ACTION_WRITE` 位被设置，对于 `classifyFn()` 来说，意味着包可以被 pend 然后重注入或修改。此时，callout 不能 pend 这次提交，不能改变 `action` 类型。它必须等待更高 weight 的 callout 注入这个包或者修改。`FwpsPendOperation0()` 函数用于 pend 一个包。

## 3.5 包注入函数

callout 驱动可以利用 `FwpsInjectForwardAsync0()` 等函数注入 pend 或者修改一个包。

## 3.6 包修改的例子

## 3.7 流的检查

## 4 修改流数据

在 callout 驱动注入流数据之前，需要先创建一个注入句柄。关于如何修改流数据，可以看 WFP 中的 Windows Filtering Platform Stream Edit Sample 示例源码。

## 5 数据日志记录

callout 的 `classifyFn()` 可以记录的数据有 data fields、metadata fields、原始网络数据和存储在 `context` 中的相关数据。例如，如果 callout 在网络层通过 filter 日志记录接收的 ipv4 数据包中有多少被丢弃了，callout 可以在 `FWPM_LAYER_INBOUND_IPPACKET_V4_DISCARD` 层向 filter engine 添加一个 filter。`classifyFn()` 函数如下：

```
VOID NTAPI
ClassifyFn()
{
    //增加所有丢弃包的统计数量
    InterlockedIncrement(&TotalDiscardCount);

    // 检查丢弃原因字段metadata 是否存在
    if (FWPS_IS_METADATA_FIELD_PRESENT(
            inMetaValues,
            FWPS_METADATA_FIELD_DISCARD_REASON))
    {
        // 检查是不是普通的丢包
        if (inMetaValues->discardMetadata.discardModule ==
                FWPS_DISCARD_MODULE_GENERAL)
        {
            // 检查是不是因为filter丢包
            if (inMetaValues->discardMetadata.discardReason ==
                    FWPS_DISCARD_FIREWALL_POLICY)
            {
                // 增加因为filter丢包的统计数量
                InterlockedIncrement(&FilterDiscardCount);
            }
        }
    }

    // 对包的action不做改变
    classifyOut->actionType = FWP_ACTION_CONTINUE;
}
```

## 6 为一个数据流关联一个 cotext

对于支持数据流的 callout 在 filtering layer 处理数据时，callout driver 可以和流关联一个 `context`。这个 `context` 对 filter engine 是不透明的。callout 的 classify 函数可以使用这个 `context` 为指定的数据流存储状态信息，以便于 filter engine 下次调用 callout 时使用。filter engine 通过 `flowContext` 传递给 callout，如果没有关联 `context`，则参数为 0。

callout 使用函数 `FwpsFlowAssociateContext0()` 关联一个 `context`。例如：

```
// Context structure to be associated with data flows

typedef struct FLOW_CONTEXT_ {
    .
    .  // Driver-specific content
    .
} FLOW_CONTEXT, *PFLOW_CONTEXT;

#define FLOW_CONTEXT_POOL_TAG 'fcpt'

VOID NTAPI
ClassifyFn()
{
    PFLOW_CONTEXT context;
    UINT64 flowHandle;
    NTSTATUS status;
    ...

    // 在metadata中检查流的句柄
    if (FWPS_IS_METADATA_FIELD_PRESENT(
            inMetaValues,
            FWPS_METADATA_FIELD_FLOW_HANDLE))
    {
        //获取流句柄
        flowHandle = inMetaValues->flowHandle;

        // 分配一个context结构
        context =
            (PFLOW_CONTEXT)ExAllocatePoolWithTag(
                NonPagedPool,
                sizeof(FLOW_CONTEXT),
                FLOW_CONTEXT_POOL_TAG
            );

        //检查是否分配失败
        if (context == NULL) {}
        else
        {
            //初始化context结构
            ...

            // 和数据流关联context
            status = FwpsFlowAssociateContext0(
                flowHandle,
                FWPS_LAYER_INBOUND_IPPACKET_V4,
                calloutId,
                (UINT64)context
            );

            //检查关联结果
            if (status != STATUS_SUCCESS)
            {
                // Handle error
                ...
            }
        }
    }
    ...
}
```

可以通过函数 `FwpsFlowRemoveContext0()` 解除关联。

## 7 处理鉴别异步数据

wfp callout 驱动可以通过在 `classifyFn()` 函数中返回 `FWP_ACTION_PERMIT`、`FWP_ACTION_CONTINUE` 或 `FWP_ACTION_BLOCK` 授权或者拒绝一个网络行为，或者放行、丢弃一个网络包。

很多情况下，callout 驱动不能在 `classifyFn()` 函数立即返回检查结果，必须异步地等待一些相关数据的处理。比如 classifiable fields、metadata、packets 可能被转发给别的组件处理，或者应用层返回判断结果。

wfp 对不同的层的异步处理机制不同，一般的处理方式如下：

### a. 异步的ALE层鉴别

callout 驱动在 `classifyFn()` 中调用 `FwpsPendOperation0()` 异步处理。异步操作完成后调用 `FwpsCompleteOperation0()` 函数结束。

### b. 异步的packet鉴别

callout 驱动应该在 classify 函数中返回 `FWP_ACTION_BLOCK` 并设置 `FWPS_CLASSIFY_OUT_FLAG_ABSORB` 标志。网络包应该被引用或者复制。异步的操作可以是重新注入复制或者修改后的包，或者丢弃包。

### c. 异步的包含packet的ALE鉴别

包含上述两种操作。

需要考虑的特殊情况如下：

### a. ALE connect与Receive/Accept 层

当在 ALE connect 层（`FWPS_LAYER_ALE_AUTH_CONNECT_V4` 或 `FWPS_LAYER_ALE_AUTH_CONNECT_V6`）完成了一个 pended 的 classify 操作时，`FwpsCompleteOperation0()` 函数被调用。此时，ALE 的 reauthorization 的 classify 行为被触发在各自的 ALE 连接层。callout 驱动应该从这种 reauthorization 行为中返回一个检查的决定。可以通过 `FWP_CONDITION_FLAG_IS_REAUTHORIZE` 标志看是不是 reauthoriaztion 行为。

callout 驱动必须为每一个 pended 的 `ALE_AUTH_CONNECT` classify 行为维持一个唯一的状态，以便于在 `FwpsCompleteOperation0()` 触发的 reauthoriaztion 期间对每一个 classify 行为的 classify 决定都可以被查询。如果在 pended 的 `ALE_AUTH_CONNECT` classify 行为期间，reauthoriaztion 发生后包被引用或者复制时他们可以被重注入。

当在 ALE 的 Receive/Accept（`FWPS_LAYER_ALE_AUTH_RECV_ACCEPT_V4` 或 `FWPS_LAYER_ALE_AUTH_RECV_ACCEPT_V6`）完成一个 classify 操作时，`FwpsCompleteOperation0()` 被调用不会触发 ALE reauthoriaztion 操作。后面几句没看懂。 Instead a new call to `classifyFn()` is made again when the cloned packet is reinjected incoming if the modification was not significant enough to bypass the filter. Permitting the self-injected clone from the `ALE_RECV_ACCEPT` layer effectively authorizes the incoming connection. If the incoming connection is not to be allowed, discard the incoming packet after it calls `FwpsCompleteOperation0()`.

### b. ALE reauthoriaztion

当一些事件发生时，比如策略改变（例如增加删除一个 filter），检测到新的 arrival 接口，通过使用 IPsec 更新连接密钥，callout 驱动可以在 ALE connect 或者 receive/accept 层 reclassified。这种 reauthoriaztion 不能通过调用 `FwpsCompleteOperation0()` 被 pended。callout 驱动在 reauthoriaztion 时可以利用规则处理这些包。

所有进来和出去的数据包都可以被 reauthorized 在 `ALE_AUTH_CONNECT` 或者 `ALE_RECV_ACCEPT` 层。例如，一个进来的数据包可以被 reauthorized 在 `ALE_AUTH_CONNECT` 层。callout 驱动不可以假设包的方向和 connection 的方向相同。

### c. ALE_FLOW_ESTABLISHED 层

`FWPS_LAYER_ALE_FLOW_ESTABLISHED_V4` 或 `FWPS_LAYER_ALE_FLOW_ESTABLISHED_V6` 层不支持异步处理数据。

### d. INBOUND_TRANSPORT Layers

当在进来的传输层（`FWPS_LAYER_INBOUND_TRANSPORT_V4` 或 `FWPS_LAYER_INBOUND_TRANSPORT_V6`）要求 ALE classify 处理包时，callout 驱动必须不完成异步处理过程。这个可以和流的创建进行交互。当 wfp 对进来的包调用 classify 函数时，它对那些要求 ALE classify 处理的包设置了 `FWPS_METADATA_FIELD_ALE_CLASSIFY_REQUIRED` 标志。callout 驱动应该从 `INBOUND_TRANSPORT` 层对这种包放行，并且延迟处理直到 `ALE_RECV_ACCEPT` 层。

### e. stream 层

在 stream 层（`FWPS_LAYER_STREAM_V4` 或 `FWPS_LAYER_STREAM_V6`），tcp 数据段代替了 ip 或者 tcp 头提交。stream 层也是 net buffer list 可以被提交的在 classify 函数中。WFP 可以对 stream 层将调用的函数进行复制或者注入，通过 `FwpsCloneStreamData0()` 和 `FwpsStreamInjectAsync0()` 函数。

因为提交的 stream 层的数据的有序性，callout 驱动必须复制和处理数据只要流数据是 pending 的。对一个给定的数据流混合同步和异步操作对导致异常行为。

## 8 连接绑定或者重定向连接

WFP 连接重定向 callouts 重定向一个程序的连接请求以便程序连接代理服务取代原始的连接。代理服务有两种 sockets，一个用于被重定向的原始连接，一个用于新的代理发送连接。

一条 wfp 重定向记录是不透明的数据缓冲区，它被 wfp 在 `FWPM_LAYER_ALE_AUTH_CONNECT_REDIRECT_V4` 和 `FWPM_LAYER_ALE_AUTH_CONNECT_REDIRECT_V6` 层建立了对外的连接请求，从而把重定向后的连接和原始的连接关联起来。

win7 之前，callout driver 可以代理 tcp 连接的方式只有复制、丢弃、修改或者重注入传输层的包。有了连接绑定重定向的支持，callout driver 可以修改任何 tcp 的 4 元组（本地、远程 ip 地址和端口号）。因为可以绑定重定向，所以没必要在连接重定向上支持本地地址和端口修改。

重定向可以在以下重定向层完成：

- `FWPM_LAYER_ALE_BIND_REDIRECT_V4`（`FWPS_LAYER_ALE_BIND_REDIRECT_V4`）
- `FWPM_LAYER_ALE_BIND_REDIRECT_V6`（`FWPS_LAYER_ALE_BIND_REDIRECT_V6`）
- `FWPM_LAYER_ALE_CONNECT_REDIRECT_V4`（`FWPS_LAYER_ALE_CONNECT_REDIRECT_V4`）
- `FWPM_LAYER_ALE_CONNECT_REDIRECT_V6`（`FWPS_LAYER_ALE_CONNECT_REDIRECT_V6`）

重定向的效果取决于所在层，connect 层只影响连接流，bind 层影响所有连接。

可以使用 `FwpsCalloutRegister1()` 或者 `FwpsCalloutRegister2()` 而不是 the older `FwpsCalloutRegister0()` 注册重定向。

重定向不是适用于所有类型的网络流量，支持的类型有 tcp、udp、Raw UDPv4 without the header include option、raw icmp。

示例代码如下。

## 9 ale终端的生命周期管理

支持 ale 的 callout driver 可能会分配资源处理提交数据。现在讨论当相关的端点关闭时，如何配置 callout driver 释放这些资源。ale 端点的生命周期管理在 win7 后的系统都支持。

为了管理 ale 端点相关的资源，callout 必须在以下层注册：

- `FWPS_LAYER_ALE_RESOURCE_RELEASE_V4`（`FWPM_LAYER_ALE_RESOURCE_RELEASE_V4`）
- `FWPS_LAYER_ALE_RESOURCE_RELEASE_V6`（`FWPM_LAYER_ALE_RESOURCE_RELEASE_V6`）
- `FWPS_LAYER_ALE_ENDPOINT_CLOSURE_V4`（`FWPM_LAYER_ALE_ENDPOINT_CLOSURE_V4`）
- `FWPS_LAYER_ALE_ENDPOINT_CLOSURE_V6`（`FWPM_LAYER_ALE_ENDPOINT_CLOSURE_V6`）

...

## 10 使用数据包的标记

callout driv 可以标记有兴趣的包或者接受标记包的事件通知。包的标记在 win7 之后系统支持。

为了使用包标记，callout driver 必须实现 `FWPS_NET_BUFFER_LIST_NOTIFY_FN0` 和 `FWPS_NET_BUFFER_LIST_NOTIFY_FN1` 回调函数。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
