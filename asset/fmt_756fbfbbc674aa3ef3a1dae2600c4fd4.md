
## 汇编代码片段

```
xor ecx, ecx
@done:
mov [edi], ecx
popad
popfd
db 0c3h               ; 返回到原来的处理程序
```

一旦有按键消息产生，上面的代码就会被调用，而初始的中断处理程序句柄（已经被改写）被压入堆栈。从 `0x60` 断口读取当前的扫描码并保存到所分配的缓冲区中。这个缓冲区可以保存 `0X3FF` 个键盘输入，如果之后再有扫描码就会覆盖前面部分。

### 对注入放火墙驱动程序的思考

当在防火墙驱动程序中注入一个内核级漏洞时，将需要考虑很多问题。我们将要示范的漏洞是由处理 DNS 反馈信息过程引起的，DNS 反馈信息是由 `SYMDNS.SYS` 处理的。如果 DNS 处理过程不能成功返回，那么就不能用 socket 来通信。在研究这个问题之前，首先必须理解多种协议层的通信机制。

下面是网络层的概要：

1. 网络驱动程序接口规范（Network Driver Interface Specification Layer）

   `NDIS` 为从物理设备到网络传输提供一个通路

   `NDIS` 驱动程序直接与网络适配器打交道。

2. 网络协议层（Network Protocol Layer）

   此处为 `TCP/IP`. (`TCPIP.SYS`)

3. 传输层驱动接口（Transport Driver Interface Layer）

   `TDI` 为网络协议、客户端协议、以及网络 API 例如 `Winsock` 提供接口。

4. 网络应用程序接口（Network API Layer）

   网络应用程序接口例如 `Winsock`，为网络应用程序提供编程接口。

所有基于主机的放火墙的限制策略都工作在内核模式，通常可以通过 `TDI` 过滤驱动程序或者 `NDIS` 挂钩过滤驱动程序。虽然我没有见过这一类的放火墙产品，但是挂钩 `AFD` 接口也是可能的。

我们所面对的问题：`SYMDNS.SYS` 必须返回到 `TDI` 过滤驱动程序 `SYMTDI.SYS`，不幸的是一旦执行我们的 shell code，通信就不会结束。这里有一些的解决方案：

#### (a) “clean” 返回

Clean 返回包括在没有出现 BSOD 的情况下从 shell code 返回，有包括能继续正常的通信，这个是很难实现的。经过攻击后的堆栈不是处于最佳状态，所以必须返回到原来堆栈桢的状态

#### (b) 卸载 TDI 或者 NDIS 的过滤驱动

卸载过滤驱动是另外一个可行的方法。我们可以很方便的调用驱动程序的卸载程序，这就相当于从 `DriverEntry` 程序调用 `DriverObject->DriverUnload`。这个驱动程序的偏移地址可以通过目标驱动程序的 `DRIVER_OBJECT` 获得。

如果 `DriverUnload` 的成员 `DRIVER_OBJECT` 为空，意味目标驱动程序的卸载程序不存在。`DRIVER_OBJECT` 可以被 `DEVICE_OBJECT` 的成员所引用，可以通过把驱动程序名作为参数传递给 `IoGetDeviceObjectPointer`，取得指向 `DEVICE_OBJECT` 的指针。

#### (c) 分离或删除驱动程序（Detach/delete the devices）

驱动可以通过调用 `IoAttachDevice` 或 `IoAttachDeviceToDeviceStack` 把自身的设备对象附加到其它设备，因而对原始设备的请求首先被传递到立即设备。我们可把 `DEVICE_OBJECT` 作为参数传递给 `IoDetachDevice` 来分离驱动程序，有可以把 `DEVICE_OBJECT` 作为参数传递给 `IoDeleteDevice` 来移除设备。
