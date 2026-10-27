
# [原创]驱动开发之NT框架与R3和R0的驱动通讯深入解析-文字解说

## PAE 物理扩展地址
`PAE(物理扩展地址)`:他能解决Windows的应用程序在32位4GB的情况下能突破4GB！

他能将32位的地址总线扩展到36位地址总线！

## 分发函数
像`DispatchCreate`、`DispatchRead`、`DispatchWrithe`是分发函数

每一个API函数都对应一个分发函数、与之与名字对应！

## IRP
`IRP`:应用层的函数的一些数据、命令、参数等通过封装了一层`IRP`传送到内核层！

驱动框架目前分为：`NT`框架\`WDM`框架\应用框架 三大类 重点在于`NT`框架

当`FileObject`为0的时候触发`IRP_MJ_CLOSE`

当`Handle`为0的时候触发`IRP_MJ_CLEANUP`

`FileObject`:这个是内核中的概念，例如我们打开磁盘中的一个文件系统就会为这个文件创建一个文件控制块`FCD`然后这个文件控制块会给我们生成若干个`FileObject`文件对象，

其实打开一个文件就会对`FileObject`的应用进行了加一操作！

`FileObject`是跨进程的！

`Handle`是本进程才有效的！

当`FileObject`和`Handle`都为0的时候则都会进程触发

为了达到`R0`与`R3`进行通讯索引必须初始化一个设备名称和一个符号连接名！

驱动层为了响应应用层的一些请求

创建设备对象的目的就是为了接受`R3`层的`IRP`数据

符号连接名主要是为设备对象而创建的、创建了符号连接在应用层才能看到驱动

红框表示这个名子是固定的不能随意变动的，而后面的可以随意变动，上面`ntmodeldrv`可以和下面的`ntmodeldrv`名字相同，为了防止混淆所以取名字可以取不一样！

所以需要保持三一致，编译好的驱动名、符号连接名、设备名

开始创建设备名，创建设备名是根据驱动内核对象进行创建，`pDerverObject`。

参数一：即为驱动对象、

参数二：设备扩展，可以存放我们自己定义好的一些数据，可以是私有！用不到可以指定为0

参数三：创建的设备名称

参数四：设备的类型，可以自己指定

参数五：属性

参数六：表示我们这个设备对象创建完毕之后是否已独立堆栈的形式打开，如果`FALSE`是不独立堆栈。如果为`TRUE`，说明这个设备创建完毕在`R3`只能被一个进程进行打开，别的进程就打不开了！目的为了设备的安全，如果以安全考虑则可以将这个参数设置为`TRUE`

参数七：指向新的一个设备对象，这里是加了取地址符，是以设备指针的形式返回一个新的设备对象。仅仅改变了这个参数【这里注意：传递进去的是驱动对象参数一，返回是设备对象】

创建完新的对象之后给这个新的对象指定一个新的标志

这个标志意味着的创建好驱动之后与`R3`和`R0`之间的读/写的通行方式！

驱动通讯：就是`R3`把数据传给`R0`、`R0`把数据传给`R3`

目前一共有三种通讯方式：

1. `buffered io`
2. `direct io`
3. `neither io`

//以`buffered io(IO`管理器拷贝的方式)设置一个驱动的通讯方式

`pDeviceObject->Flags = DO_BUFFERED_IO`;

//这个设备对象是以驱动对象创建出来的新的设备对象

//`buffered io`的通讯方式：

//1.`IO`管理器会在内核中申请一块缓存区、`IO`管理器会将`R3`传递过来的数据拷贝过去

//2.然后这个内核缓存区由驱动进行接收

//3.驱动处理完成之后直接修改内核缓存、再由`IO`管理器输出到`OUT_BUFFER`中

`MDL`结构：这个结构可以将`R3`的虚拟内存数据映射到物理内存空间！然后把这块内存锁住

`R0`直接访问`R3`的虚拟内存空间有两点要求：

1. 需要保证`R0`层驱动和`R3`保持在同一个进程上下文。原因是因为`R3`不是共享内存空间，而`R0`是内存共享空间。

2. 需要通过这两个函数进行处理，这个是一个校验，判断这个地址是否处在三环的虚拟地址

3. 这种方式非常危险必须通过这两个函数进行校验

这个标志告诉驱动在初始化的时候不要发送一些`IO`命令过来让我进行处理。

清除：

```
 //设备的名称
UNICODE_STRING uDeviceName = { 0 };
 //符号连接名
UNICODE_STRING uLinkName = { 0 };
 //Nt返回的状态码
NTSTATUS ntStatus = 0;
 //设备对象
PDEVICE_OBJECT pDeviceObject = NULL;
ULONG i = 0;
 //装载驱动
DbgPrint("Driver load begin!");//打印驱动正在加载
 //初始化驱动名
RtlInitUnicodeString(&uDeviceName, DEVICE_NAME);
 //初始化符号连接名
RtlInitUnicodeString(&uLinkName,LINK_NAME);
 //1.为了达到R0与R3进行通讯索引必须初始化一个设备名称和一个符号连接名！
 //2.驱动层为了响应应用层的一些请求
 //3.创建设备对象的目的就是为了接受R3层的IRP数据
 //4.符号连接名主要是为设备对象而创建的、创建了符号连接在应用层才能看到驱动
 //5.只有驱动内部含有符号连接名应用层才能以文件的形式打开这个驱动
//////////////////////////////////////////////////////////////////////////
 //创建驱动对象
 //设备对象与驱动对象之间的关系、二者互指
 //开始创建设备名，创建设备名是根据驱动内核对象进行创建，pDerverObject。
ntStatus = IoCreateDevice(
pDriverObject,
0, &uDeviceName, FILE_DEVICE_UNKNOWN, 0, FALSE, &pDeviceObject
);
 //参数一：即为驱动对象
 //参数二：设备扩展，可以存放我们自己定义好的一些数据，可以是私有！用不到可以指定为0
 //参数三：创建的设备名称、正好是刚刚定义的宏 DEVICE_NAME L"\\device\\ntmodeldrv"
 //参数四：设备的类型，可以自己指定
 //参数五：属性
 //参数六：表示我们这个设备对象创建完毕之后是否已独立堆栈的形式打开，
 //注意点一：如果FALSE是不独立堆栈（与三环共享一个堆栈）。如果为TRUE，说明这个设备创建完毕在R3只能被一个进程进行打开，别的进程就打不开了！
 //注意点二：目的为了设备的安全，如果以安全考虑则可以将这个参数设置为TRUE！
 //参数七：指向新的一个设备对象，这里是加了取地址符，是以设备指针的形式返回一个新的设备对象。仅仅改变了这个参数【这里注意：传递进去的是驱动对象参数一，返回是设备对象】
 //判断是否创建成功
if (!NT_SUCCESS(ntStatus))
{
 //输出状态码
DbgPrint("IoCreateDevice failed:%x", ntStatus);
 //返回状态码
return ntStatus;
}
//////////////////////////////////////////////////////////////////////////
 //创建完对象之后给他指定一个R3与R0通讯的一个标志
 //驱动通讯：就是R3把数据传给R0、R0把数据传给R3
 //目前一共有三种通讯方式：
 // 1. buffered io
 // 2. direct io
 // 3. neither io
 //以buffered io(IO管理器拷贝的方式)设置一个驱动的通讯方式
pDeviceObject->Flags = DO_BUFFERED_IO;
 //这个设备对象是以驱动对象创建出来的新的设备对象
 //buffered io的通讯方式：
 //1.IO管理器会在内核中申请一块缓存区、IO管理器会将R3传递过来的数据拷贝过去
 //2.然后这个内核缓存区由驱动进行接收
 //3.驱动处理完成之后直接修改内核缓存、再由IO管理器输出到OUT_BUFFER中
//////////////////////////////////////////////////////////////////////////
 //创建符号连接对象，创建完之后应用层才能看到这个驱动程序，才能打开这个驱动文件、后才能让设备对象接受R3传递过来的IRP
ntStatus = IoCreateSymbolicLink(&uLinkName, &uDeviceName);
if (!NT_SUCCESS(ntStatus))
{
 //创建失败要删除设备对象释放资源
IoDeleteDevice(pDeviceObject);
 //输出错误码
DbgPrint("IoCreateSymbolicLink Failed:%x\n", ntStatus);
 //返回状态码
return ntStatus;
}
```

```
//////////////////////////////////////////////////////////////////////////
//创建完对象之后给他指定一个R3与R0通讯的一个标志
//驱动通讯：就是R3把数据传给R0、R0把数据传给R3
//目前一共有三种通讯方式：
// 1. buffered io
// 2. direct io
// 3. neither io
//以buffered io(IO管理器拷贝的方式)设置一个驱动的通讯方式
pDeviceObject->Flags = DO_BUFFERED_IO;
//这个设备对象是以驱动对象创建出来的新的设备对象
//buffered io的通讯方式：
//1.IO管理器会在内核中申请一块缓存区、IO管理器会将R3传递过来的数据拷贝过去
//2.然后这个内核缓存区由驱动进行接收
//3.驱动处理完成之后直接修改内核缓存、再由IO管理器输出到OUT_BUFFER中
//////////////////////////////////////////////////////////////////////////
//创建符号连接对象，创建完之后应用层才能看到这个驱动程序，才能打开这个驱动文件、后才能让设备对象接受R3传递过来的IRP
ntStatus = IoCreateSymbolicLink(&uLinkName, &uDeviceName);
if (!NT_SUCCESS(ntStatus))
{
 //创建失败要删除设备对象释放资源
IoDeleteDevice(pDeviceObject);
 //输出错误码
DbgPrint("IoCreateSymbolicLink Failed:%x\n", ntStatus);
 //返回状态码
return ntStatus;
}
//给驱动对象的每一个分发函数初始化一个通用的分发函数
for (i = 0; i < IRP_MJ_MAXIMUM_FUNCTION; i++)
{
pDriverObject->MajorFunction[i] = DispatchCommon;
}
//这个for循环就是为我们驱动中所有分发函数中做一个初始化，做一个通用的分发函数！
//通用的分发函数：什么也没有做，就直接传入一个IRP，将IRP设置成一个STATUS_SUCCESS 然后直接返回一个成功！
//实际上没有任何意义，其实提供一种通用的处理。相当于把一个局部变量初始化成0或者是NULL
//分发函数一共有0x2b+1个
//////////////////////////////////////////////////////////////////////////
//这个是拦截应用层的文件创建操作
pDriverObject->MajorFunction[IRP_MJ_CREATE] = DispatchCreate;
//这个是拦截读操作
pDriverObject->MajorFunction[IRP_MJ_READ] = DispatchRead;
//拦截写操作
pDriverObject->MajorFunction[IRP_MJ_WRITE] = DispatchWrite;
//这个一般做设备控制
pDriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = DispatchIoctrl;
//拦截文件的关闭
pDriverObject->MajorFunction[IRP_MJ_CLEANUP] = DispatchClean;
//拦截文件句柄的关闭操作
pDriverObject->MajorFunction[IRP_MJ_CLOSE] = DispatchClose;
//这里每一个分发函数都是来处理应用层发送下来的IRP，宏分别对应每一个API
pDriverObject->DriverUnload = DriverUnload;
DbgPrint("Driver Load Ok!\n");
```

```
/////////////////////////////////////////////////////
//函数说明：这个是分发函数的定义、分发函数的参数和返回值都是一样的！
//参数一：PDEVICE_OBJECT pObject 这个指的是设备对象，创建设备对象主要用来接收R3的IRP数据
//参数二：PIRP pIrp 应用层传递过来的IRP
// 这个IRP指针就是应用层传递过来的数据被IO管理器组织好的一个数据，
// 这个IRP负责发给设备对象。这个设备对象接受之后会调用这个分发函数来处理他！
//返回值:返回值是STATUS_SUCCESS、在内核层返回STATUS_SUCCESS也就是0就成功，在应用层返回0则失败！和内核层是倒过来！
//备注：
//////////////////////////////////////////////////////
NTSTATUS DispatchCommon(PDEVICE_OBJECT pObject,PIRP pIrp)
{
pIrp->IoStatus.Status = STATUS_SUCCESS;
//这里表示IO的成功状态，如果是STATUS_SUCCESS则为成功、向应用层的调用者发送的成功标志
pIrp->IoStatus.Information = 0;
//这里表示额外的信息，比如传递过来的字节数，可能在其他地方表示其他的含义！
IoCompleteRequest(pIrp, IO_NO_INCREMENT);
//这行代码表示IRP传入到我们驱动处理完成之后就把他结束掉！
//要点一：当我们接受到IRP我们把他处理之后然后把他结束掉这是一种处理方式
//要点二：当我们接受到IRP可以对它进行拦截或者是更改继续往下发送，传入给下一个驱动！
return STATUS_SUCCESS;
//最后给IO管理器返回一个成功标识
}
```

总而言之、三环函数->`NTDLL`->封装`IRP`->由驱动进行接收处理->最后返回给`R3`

## 读函数讲解

```
NTSTATUS DispatchRead(PDEVICE_OBJECT pObject,PIRP pIrp)
{
PVOID pReadBuffer = NULL;
ULONG uReadLength = 0;
PIO_STACK_LOCATION pStack = NULL;
ULONG uMin = 0;
ULONG uHelloStr = 0;
//分析ReadFile函数
//参数一：文件句柄
//参数二：文件缓存区
//参数三：缓存区的长度
//参数四：实际读取的长度 与pIrp->IoStatus.Information对应;
//参数五：是否异步
//拿到字符串Hello Word的长度 + 1指定是结尾符
uHelloStr = (wcslen(L"hello world") + 1) * sizeof(WCHAR);
//第一步：拿到内核的缓存地址和长度
//从头部拿到地址
//如果是Buffer IO 则驱动这个指针指向IO分配的一块内核中的缓存区。
//这块缓存区的IO管理器在内核中分配了一块空间，然后将应用层的缓存区拷贝过去，然后IRP的SystemBuffer指向这块内核空间
pReadBuffer = pIrp->AssociatedIrp.SystemBuffer;
//////////////////////////////////////////////////////////////////////////
//从栈上拿到缓存的长度
//IRP是一个结构体，其中IRP分为N个栈(6-8个栈)。通过这个函数可以拿到IRP当前栈的位置！
pStack = IoGetCurrentIrpStackLocation(pIrp);
//////////////////////////////////////////////////////////////////////////
//Parameters这是一个联合体
//这里如果当前是读操作，是Parameters是Read、如果是写则为Write
uReadLength = pStack->Parameters.Read.Length;
//////////////////////////////////////////////////////////////////////////
//第二步：读，写等操作
uMin = uReadLength > uHelloStr ? uHelloStr : uReadLength;
//内存拷贝
RtlMoveMemory(pReadBuffer, L"Hello world", uMin);
//要点注意一：为了考虑拷贝溢出的问题，这里选择应用层传递过来的缓存区大小和实际大小，需要判断两个数据的大小(uReadLenth和uHelloStr)，谁小传谁！
//要点注意二：如果pReadBuffer 里面的字节数小于uHelloStr则uMin需要减2，剩下的两个字节就存放结尾符号
//第三步：完成IRP操作
pIrp->IoStatus.Status = STATUS_SUCCESS;
//这里用Infomation返回实际操作的字节数
pIrp->IoStatus.Information = uMin;
//结束掉IRP
IoCompleteRequest(pIrp, IO_NO_INCREMENT);
//给IO管理器返回一个成功
return STATUS_SUCCESS;
}
```
