
# [原创]WDM驱动之异步I/O,事件通知，自旋锁

网上介绍驱动程序的异步I/O和事件通知的教程实在太难找了，大多都是一笔带过，有的也只是给出一个基本框架，以偶的水平，打死我也写不出一个完整的代码出来![](https://bbs.kanxue.com/view/img/face/50.gif) 武安河的《windows 2000/xp WDM 设备驱动程序开发》一书中有这一部分的内容，不过是用DriverStudio以类的方式讲解的，实在难懂。最后终于在《windows WDM设备驱动程序开发指南》中发现其中第14章的DebugPrint源代码讲的就是这部分内容，我对驱动层代码进行了注释，自己写了一个简单的用户态程序，终于完成了驱动专题的WDM部分，呵呵![](https://bbs.kanxue.com/view/img/face/27.gif)

下面的理论部分为转载，上述代码见附件，困，快半夜3点了，不知能加精否，应该有苦劳的![](https://bbs.kanxue.com/view/img/face/41.gif)

## 一.基本框架（不知哪位老大翻译的）

```
HANDLE hfile = CreateFile(..., FILE_FLAG_OVERLAPPED, ...);

BYTE bBuffer[100];

OVERLAPPED o = { 0 };

o.Offset = 345;

BOOL fReadDone = ReadFile(hfile, bBuffer, 100, NULL, &o);

DWORD dwError = GetLastError();

if (!fReadDone && (dwError == ERROR_IO_PENDING))
{
    // The I/O is being performed asynchronously; wait for it to complete
    WaitForSingleObject(hfile, INFINITE);
    fReadDone = TRUE;
}

if (fReadDone)
{
    // o.Internal contains the I/O error
    // o.InternalHigh contains the number of bytes transferred
    // bBuffer contains the read data
}
else
{
    // An error occurred; see dwError
}
```

一旦线程发出一个异步I/O请求后，就可以继续执行，做其它有用的任务。最终，线程仍需要和该I/O操作的完成同步。换句话说，你的线程运行至某个地方就得停下来，直到来自设备的数据已完整地载入缓冲区，线程代码才能继续执行下去。

在Windows中，一个设备内核对象可用于线程同步，故内核对象要么处于有信号态要么处于无信号态。`ReadFile`和`WriteFile`函数在排队一个I/O请求前，即把设备内核对象设置为无信号态。当设备驱动程序完成请求后，驱动程序把设备内核对象设置为有信号态。

线程可以调用`WaitForSingleObject`或`WaitForMultipleObjects`来判断一个异步I/O请求是否已完成。下面是一个简单的例子：

这段代码发出一个异步I/O请求，随后立即等待请求完成，实际上背离了异步I/O的目的。显然，你绝不会编写类似的代码，不过这些代码却演示了一些重要的规则，我总结如下：

- 执行异步I/O的设备必须用`FILE_FLAG_OVERLAPPED`标志打开。
- `OVERLAPPED`结构的`Offset`、`OffsetHigh`和`hEvent`成员必须被初始化。在示例代码中，除了`Offset`我把它们全设置为0，`Offset`设为345使`ReadFile`从文件开始第346个字节处读数据。
- `ReadFile`的返回值保存于`fReadDone`中，该值指示了I/O请求是否被同步执行。
- 如果I/O请求不是被同步执行，我就检查是否产生一个错误或是I/O被异步执行。通过将`GetLastError`返回值和`ERROR_IO_PENDING`比较，即可对此得出结论。
- 为了等待数据，我调用`WaitForSingleObject`并传递设备内核对象的句柄。`WaitForSingleObject`会挂起调用线程直至内核对象变成有信号态。设备驱动在它完成I/O后使内核对象有信号。在`WaitForSingleObject`返回后，I/O被完成，我把`fReadDone`设为`TRUE`。
- 在读完成后，你可以检查`bBuffer`中的数据，错误码存放在`OVERLAPPED`结构的`Internal`成员里，传输字节数存放在`OVERLAPPED`结构的`InternalHigh`成员里。
- 如果确实发生了错误，`dwError`存放的错误码给出了进一步的信息。

## 二.异步设备I/O基础（by 马夫 2006-03-25 11:50:44）

与计算机执行的大多数操作相比，设备I/O是其中速度最慢和结果最难以预料的。CPU在执行算术运算甚至画屏幕时也比从一个文件或网络上读写数据要快得多。不过，使用异步设备I/O可让你更好地利用资源并创建更有效的程序。

设想线程向设备发出了一个异步I/O请求的情况：I/O请求被传递到实际进行I/O的设备驱动程序，当驱动程序在等待设备响应时，线程不是挂起等待I/O请求完成，而是继续执行其它有用的任务。

关键之处在于，设备驱动程序要处理完排队的I/O请求，并且必须通知程序已经发送或接收了数据，或产生了错误。你将在下一节，“接收I/O请求完成通知”，学习设备驱动程序如何通知你I/O完成了。现在，让我们集中考虑如何排队异步I/O请求。排队异步I/O请求是设计高性能、可伸缩的程序的精髓，也是本章所有内容的沉淀。

要异步地访问一个设备，你就必须在调用`CreateFile`打开该设备时，在`dwFlagsAndAttrs`参数中指定`FILE_FLAG_OVERLAPPED`标志。这个标志告诉系统你想要异步地访问设备。

要给设备驱动程序排队一个I/O请求，你可使用在“进行同步设备I/O”一节中已学过的`ReadFile`和`WriteFile`函数。为了阅读方便，我这里再次列出函数原型：

```
BOOL ReadFile(   HANDLE      hfile,    PVOID       pvBuffer,   DWORD       nNumBytesToRead,    PDWORD      pdwNumBytes,   OVERLAPPED* pOverlapped);

BOOL WriteFile(   HANDLE      hfile,    CONST VOID  *pvBuffer,   DWORD       nNumBytesToWrite,    PDWORD      pdwNumBytes,   OVERLAPPED* pOverlapped);
```

当这两个函数中的一个被调用时，函数会检查由`hfile`参数标识的设备是否用`FILE_FLAG_OVERLAPPED`标志打开。若指定了该标志，函数就执行异步I/O。顺便说一句，在用这两个函数进行异步I/O时，你可以（也是通常的做法）传递`NULL`给`pdwNumBytes`参数。既然你期望这些函数在I/O完成前就返回，那么检查此刻已传输的字节数就是毫无意义的。

### OVERLAPPED结构

要进行异步设备I/O，你必须通过`pOverlapped`参数传递一个已初始化的`OVERLAPPED`结构的地址。此处的“overlapped”一词意味着花费在执行I/O请求上的时间与线程花费在执行其它任务上的时间是重叠的。`OVERLAPPED`结构看起来如下面所示：

```
typedef struct _OVERLAPPED {
    DWORD  Internal;     // [out] Error code
    DWORD  InternalHigh; // [out] Number of bytes transferred
    DWORD  Offset;       // [in]  Low 32-bit file offset
    DWORD  OffsetHigh;   // [in]  High 32-bit file offset
    HANDLE hEvent;       // [in]  Event handle or data
} OVERLAPPED, *LPOVERLAPPED;
```

该结构里包含5个成员。其中的三个：`Offset`、`OffsetHigh`和`hEvent`，必须在调用`ReadFile`和`WriteFile`前被初始化。另外两个成员：`Internal`和`InternalHigh`，由设备驱动程序设置且可以在I/O操作完成后查看。下面是这些成员变量的更详细的解释：

- `Offset`和`OffsetHigh`：在访问文件时，这两个成员指示了在文件中的64-bit偏移值，也就是你想执行I/O操作的起始位置。前面提到每个文件内核对象都关联有一个文件指针，当发出一个同步I/O请求时，系统知道在文件指针标识的位置开始访问文件。在一次同步操作完成后，系统自动更新文件指针以便下一次操作可以从本次操作的结束位置继续。

  当执行异步I/O时，文件指针被系统忽略。想象一下会发生什么，如果你的代码（对同一个文件内核对象）先后执行了两次紧邻的异步`ReadFile`调用。在这种情况下，系统无法知道第二次调用`ReadFile`时要从哪里开始读。你可能并不想从第一次调用`ReadFile`读文件的起始位置执行第二次读操作，而是想从第一次调用`ReadFile`读出的最末一个字节之后开始执行第二次读操作。为了避免对同一个对象的多次异步调用发生混乱，所有的异步文件I/O请求必须在`OVERLAPPED`结构中指定起始偏移值。

  注意，`Offset`和`OffsetHigh`成员并不会被非文件设备忽略。你必须把这两个成员都初始化为0，否则I/O请求将失败，且`GetLastError`会返回`ERROR_INVALID_PARAMETER`。

- `hEvent`：接收I/O完成通知的方法有四种，其中一种使用了这个成员。当使用告警I/O通知方法时，这个成员则可以按你自己的方式使用。我知道很多开发者把C++对象的地址存放在`hEvent`中（在“使事件内核对象有信号”一节中我们将进一步讨论这个成员）。

- `Internal`：这个成员保存被执行的I/O的错误码。一旦你发出一个异步I/O请求，设备驱动就设置`Internal`为`STATUS_PENDING`，表示没有错误出现，只是操作还没有开始。事实上，定义在`WinBase.h`中的宏`HasOverlappedIoCompleted`，允许你检查I/O操作是否已经完成。如果请求仍然是未决的，它返回`FALSE`；如果I/O请求已完成，它返回`TRUE`。下面是该宏的定义：

```
#define HasOverlappedIoCompleted(pOverlapped)    ((pOverlapped)->Internal != STATUS_PENDING)
```

- `InternalHigh`：当一个异步I/O请求完成后，这个成员保存已传输的字节数。

在最初设计`OVERLAPPED`结构时，Microsoft决定不公布`Internal`和`InternalHigh`成员（这就是它们名字的来历）。随着时间过去，Microsoft意识到包含在这些成员中的信息对开发者是有用的，就把它们归档了。然而Microsoft没有改变这些成员的名称，因为操作系统的源代码经常引用它们，而Microsoft不想修改这些代码。

### 异步设备I/O告诫

在进行异步I/O时，你应该明白几个要点。第一，设备驱动程序不会以先进先出的方式处理排队的I/O。举个例子，如果线程执行下面的代码，设备驱动程序很可能先写文件然后再去读文件：

```
OVERLAPPED o1 = { 0 };
OVERLAPPED o2 = { 0 };
BYTE bBuffer[100];

ReadFile(hfile, bBuffer, 100, NULL, &o1);
WriteFile(hfile, bBuffer, 100, NULL, &o2);
```

只要有助于提高性能，设备驱动程序会特地打乱次序来执行I/O请求。比如，为了减少磁头的移动和寻道时间，文件系统驱动程序会搜查排队I/O请求的列表，寻找那些相邻于硬盘上同一物理位置的请求。

第二个要点是你要知道进行错误检查的正确方式。大多数Windows函数返回`FALSE`来表示失败或返回非零来表示成功。然而，`ReadFile`和`WriteFile`函数的行为有点不同。一个例子程序有助于说明这点。

在试图排队一个异步I/O请求时，设备驱动程序可能决定以同步方式来处理这个请求。当你读一个文件而系统要检查你需要的数据是否已在系统缓存中时，就会出现这种情况：如果数据可以（从缓存）获得，你的I/O请求不会被设备驱动排队，系统改为从缓存中复制数据到你的缓冲区，完成该I/O操作。

如果请求的I/O被同步执行完，`ReadFile`和`WriteFile`返回一个非零值。如果请求的I/O被异步执行，或在调用`ReadFile`和`WriteFile`时发生错误，都返回`FALSE`。当返回`FALSE`时，你必须调用`GetLastError`以确切地判断发生了什么。如果`GetLastError`返回`ERROR_IO_PENDING`，则I/O请求已被成功地排队并将随后完成。

如果`GetLastError`返回值不是`ERROR_IO_PENDING`，则I/O请求可能未被设备驱动排队。下面是I/O请求可能未被设备驱动排队时，`GetLastError`返回的最常见的错误码：

- `ERROR_INVALID_USER_BUFFER`或`ERROR_NOT_ENOUGH_MEMORY`：每个设备驱动维护有一个长度固定的未决的I/O请求列表（在非分页内存池内）。如果这个表满了，系统就不能排队你的请求，`ReadFile`和`WriteFile`返回`FALSE`，而`GetLastError`则报告这两个错误码之一（取决于驱动程序）。
- `ERROR_NOT_ENOUGH_QUOTA`：一些设备要求你的数据缓冲区内存页面锁定，使得在I/O未决期间，数据不会被切换出RAM。这种页面锁定的内存要求，当然适合于使用`FILE_FLAG_NO_BUFFERING`标志进行文件I/O。但是，系统制约了单个进程可以页面锁定的内存容量。如果`ReadFile`和`WriteFile`不能页面锁定你的缓冲区，它们就返回`FALSE`，而`GetLastError`报告`ERROR_NOT_ENOUGH_QUOTA`。你可以通过调用`SetProcessWorkingSetSize`来增加一个进程的（物理内存）配额。

你应该怎样处理这些错误呢？基本上，这些错误的产生都是因为有许多的未决I/O请求来不及完成，所以你得允许一些未决I/O请求执行完毕，然后再发起对`ReadFile`和`WriteFile`的调用。

第三个你得知道的要点是，用于执行异步I/O请求的数据缓冲区和`OVERLAPPED`结构，必须在I/O请求完成后才能被移走或销毁。当给一个设备驱动排队一个I/O请求时，传递给驱动的是数据缓冲区的地址和`OVERLAPPED`结构的地址。注意，被传递的只是地址，而不是实际的数据块。这样做的原因是显然的：内存来回复制是代价昂贵的并且浪费了大量的CPU时间。

当设备驱动即将处理你排队的请求时，它传递由`pvBuffer`地址引用的数据，并访问文件的偏移量成员和包含在由`pOverlapped`参数指定的`OVERLAPPED`结构中的其它成员。特别地，设备驱动还要用I/O的错误码更新`Internal`成员，用已传输的字节数更新`InternalHigh`成员。

**注意**：直至I/O请求被完成后才能移动或销毁这些缓冲区是绝对必要的，否则会导致内存混乱。同样地，你必须为每一个I/O请求分配和初始化一个独立的`OVERLAPPED`结构。

开发者对于上述“注意”之处要引起高度重视，它也是在实现一个异步I/O架构时易犯的最常见错误。这里有一个处理不当的例子：

```
VOID ReadData(HANDLE hfile)
{
    OVERLAPPED o = { 0 };
    BYTE b[100];
    ReadFile(hfile, b, 100, NULL, &o);
}
```

这段代码看起来毫无问题，对`ReadFile`的调用是正确的。唯一的问题在于函数在排队了一个异步I/O请求后就返回了。函数的返回实质上释放了源于线程堆栈的缓冲区和`OVERLAPPED`结构，而设备驱动并不知道`ReadData`返回了。设备驱动仍旧有两个指向线程堆栈的地址。当I/O完成时，设备驱动要修改在线程堆栈上的内存，任何碰巧在此时占用了该内存位置的数据将被损坏。这种错误特别难于查找因为内存改变是异步发生的。有时设备驱动可能同步执行I/O，这种情况下你不会发现错误。有时I/O可能在函数返回后完成，甚至超过一个小时后完成，那么谁能知道此刻堆栈正在被用作什么呢？

### 取消已排队的设备I/O请求

有时你可能想要在设备驱动处理完一个已排队的设备I/O请求前取消它，Windows对此提供了一些方法：

- 调用`CancelIo`取消在调用线程中排队并指定同一个句柄的所有的I/O请求：

  ```
  BOOL CancelIo(HANDLE hfile);
  ```

- 关闭设备句柄，取消所有已排队的I/O请求，无论这个请求由哪个线程排队。
- 在线程消亡时，系统会自动取消该线程发出的所有I/O请求。

你可以看出，没有方法能取消一个单独的、指定的I/O请求。

### 我的用户态程序

**写入**：可以多次写入向驱动程序字符串，采用同步方式，写入的字符串储存在驱动程序的双向链表中。

**启动读出线程**：新建一个线程循环向驱动程序发出读IRP，启动读出线程后，在上方的编辑控件中写入输入数据，在下方的列表控件中就会显示结果。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [DebugPrint.rar](javascript:void(0);)（81.31kb，183次下载）
