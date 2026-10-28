
1. 在一个结构中如果在新的 NDIS 版本中为其添加了新的成员，那它的保留（`revision`）成员会有一个新的值。例如，在 NDIS6.1 中在结构的末尾添加了一个联合体或一个字节域，那个 `Header` 成员中的保留成员会有一个新的值。
2. 一个结构被改变后，后面版本结构的长度可能会大于早期版本结构的长度，但是不能小于早期版本的长度。后面版本新添加的成员要放到结构的末尾。
3. 如果结构早期版本的信息仍然有效且完整结构仅新加一个新的版本号。在这种情况下应该对老的版本成员支持。注意：如果上述条件不满足，NDIS 会提供一个新命名的结构来替代原来的结构。
4. NDIS 驱动应该使用预定义的保留版本值。
5. `Header.Size` 的值应该和 `Header.Revision` 的值保持一致。这就是说它们应该保持是与同一版本相对应的对应值。

## 4.2.2 NDIS驱动需要支持的版本信息

NDIS 结构提供了一个 `Header` 成员来保存版本信息，且 NDIS 驱动也需要对它进行支持。NDIS 支持驱动运行在不同版本操作系统所支持的 NDIS 接口上。此处，高版本的 NDIS 驱动可以在低版本的 NDIS 上注册。例如：一个 NDIS5.X 或 6.1 的驱动可以注册在只支持 NDIS6.0 版本的操作系统上。对于 NDIS6.1 的驱动，则必须检查当前 NDIS 的版本，同时注册为驱动后对上级别的驱动做适当的支持。注意：驱动不需要对之前版本的所有结构特性进行支持。例如，一个驱动创建了 revision 2 版本的一个结构，只需要适当地支持 revision 1 版本的成员即可。

要访问结构的成员信息必须完成下列处理：

1. 在访问任何成员之前先检查 `Header.revision` 和 `Header.size`。
2. 对于早期版的结构，其 `Header.revision` 和 `Header.size` 必须匹配，同时必须根据这两个对应的 NDIS 版本对结构进行正确的访问。
3. 对于一个之后版本的结构，可以使用这个结构里的信息。对于高版本的结构总是兼容低版本的结构。
4. 一个驱动使用结构的版本信息必须和其注册的版本信息相匹配。如一个驱动注册为 6.1 的 NDIS 驱动，那么它也应该使用 NDIS6.1 的结构。
5. NDIS 驱动成功完成一个 OID 请求后，必须设置 `NDIS_OID_REQUEST` 的 `SupportedRevision` 成员。这个值要设置为请求发起者的 revision 版本。
6. 要决定请求是不是被下层驱动完成了，要检查 OID 请求返回来的 `SupportedRevision` 值。

## 4.2.3 NDIS需要的版本信息

NDIS 通过头部标识的不同版本信息来保证在不同的 NDIS 版本上使驱动兼容。在支持头版本信息上，NDIS 主要负责下面两点：

1. 被操纵结构的版本信息低于 NDIS 的版本；NDIS 检查头部信息，并基于相应的修订版对结构进行解释。
2. 如果一个驱动使用了错误的版本信息，则使调用失败并返回适当的错误码；比如在 NDIS6.1 上使用了 `XXX_REVISION_2` 这种版本信息，NDIS 就会使调用失败。

## 4.2.4 获得NDIS版本

NDIS 的版本在相同的计算机上也可能是不同的，例如我们可以通过 `RtlGetVersion` 和 `RtlVerifyVersionInfo` 来获取系统的版本信息，但我们不能依此来判断 NDIS 的版本信息。NDIS 驱动要获得 NDIS 的版本信息，可以使用 `NdisGetVersion`。

## 4.2.5 NDIS 对象版本发布到WMI

这个主题是关于 NDIS 对象版本对 WMI 的支持。WMI 的管理对象格式化文件是没有版本的，因此如果对 NDIS 结构有了新的修订，添加了多个域，就要添加到管理对象格式化文件中去。同时，应用程序在查询和设置一个版本时，也必须检查并提供相关的修订版本信息。

注：本帖由看雪论坛志愿者PEstone 重新将PDF整理排版，若和原文有出入，以原作者附件为准

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [NDIS Filter Drivers指南.pdf](javascript:void(0);)（771.24kb，695次下载）
- [1.png](javascript:void(0);)（6.31kb，14次下载）
- [2.png](javascript:void(0);)（50.63kb，10次下载）
- [3.png](javascript:void(0);)（2.10kb，7次下载）
- [4.png](javascript:void(0);)（10.53kb，7次下载）
- [5.png](javascript:void(0);)（16.47kb，6次下载）
- [6.png](javascript:void(0);)（16.03kb，6次下载）
- [7.png](javascript:void(0);)（15.20kb，9次下载）
- [8.png](javascript:void(0);)（11.57kb，6次下载）
- [9.png](javascript:void(0);)（13.10kb，5次下载）
- [10.png](javascript:void(0);)（14.35kb，6次下载）
- [11.png](javascript:void(0);)（12.17kb，5次下载）
- [12-1.png](javascript:void(0);)（7.43kb，5次下载）
- [12-2.png](javascript:void(0);)（21.73kb，5次下载）
- [13.png](javascript:void(0);)（6.10kb，5次下载）
- [14.png](javascript:void(0);)（5.33kb，5次下载）
- [15.png](javascript:void(0);)（5.56kb，5次下载）
- [16-1.png](javascript:void(0);)（36.48kb，5次下载）
- [16-2.png](javascript:void(0);)（44.87kb，5次下载）
- [17.png](javascript:void(0);)（94.27kb，5次下载）
- [18.png](javascript:void(0);)（22.08kb，5次下载）
- [19.png](javascript:void(0);)（19.98kb，5次下载）
- [20.png](javascript:void(0);)（16.57kb，5次下载）
- [21.png](javascript:void(0);)（24.20kb，5次下载）
- [22.png](javascript:void(0);)（3.10kb，5次下载）
- [23.png](javascript:void(0);)（9.76kb，5次下载）
- [24.png](javascript:void(0);)（15.08kb，5次下载）
- [25.png](javascript:void(0);)（15.56kb，5次下载）
- [26.png](javascript:void(0);)（11.64kb，5次下载）
- [27.png](javascript:void(0);)（87.57kb，12次下载）
- [28.png](javascript:void(0);)（18.63kb，5次下载）
- [29.png](javascript:void(0);)（46.68kb，5次下载）
- [30.png](javascript:void(0);)（41.75kb，5次下载）
