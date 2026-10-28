
```
   +0x01c NormalRoutine    : Ptr32 to     void

   +0x020 NormalContext    : Ptr32 to Void

   +0x024 SystemArgument1  : Ptr32 to Void

   +0x028 SystemArgument2  : Ptr32 to Void

   +0x02c ApcStateIndex    : Char

   +0x02d ApcMode          : Char

   +0x02e Inserted         : UChar

kd> dt \_kapc

ntdll!\_KAPC

   +0x000 Type             : Int2B

   +0x002 Size             : Int2B

   +0x004 Spare0           : Uint4B

   +0x008 Thread           : Ptr32 \_KTHREAD

   +0x00c ApcListEntry     : \_LIST\_ENTRY

   +0x014 KernelRoutine    : Ptr32

   +0x018 RundownRoutine   : Ptr32

   +0x01c NormalRoutine    : Ptr32

   +0x020 NormalContext    : Ptr32 Void

   +0x024 SystemArgument1  : Ptr32 Void

   +0x028 SystemArgument2  : Ptr32 Void

   +0x02c ApcStateIndex    : Char

   +0x02d ApcMode          : Char

   +0x02e Inserted         : UChar
```

```
typedef struct \_KAPC {

    CSHORT Type;

    CSHORT Size;

    ULONG Spare0;

    struct \_KTHREAD \*Thread;

    LIST\_ENTRY ApcListEntry;

    PKKERNEL\_ROUTINE KernelRoutine;

    PKRUNDOWN\_ROUTINE RundownRoutine;

    PKNORMAL\_ROUTINE NormalRoutine;

    PVOID NormalContext;

    //

    // N.B. The following two members MUST be together.

    //

    PVOID SystemArgument1;

    PVOID SystemArgument2;

    CCHAR ApcStateIndex;

    KPROCESSOR\_MODE ApcMode;

    BOOLEAN Inserted;

} KAPC, \*PKAPC, \*RESTRICTED\_POINTER PRKAPC;
```

## APC环境

一个线程在它执行的任意时刻，假设当前的IRQL是在Passive级，它可能需要临时在其他的进程上下文中执行代码，为了完成这个操作，线程调用系统功能函数`KeAttachProcess`，在从这个调用返回时，线程执行在另一个进程的地址空间。先前所有在线程自己的进程上下文中等待执行的APC,由于这时其所属进程的地址空间不是当前可用的，因此他们不能被投递执行。然而，新的插入到这个线程的APC可以执行在这个新的进程空间。甚至当线程最后从新的进程中分离时，新的插入到这个线程的APC还可以在这个线程所属的进程上下文中执行。

NT中每个线程维护了两个APC环境或者说是状态。每一个APC环境包含了用户模式的APC队列和内核模式的APC队列，一个指向当前进程对象的指针和三个控制变量，用于指出：是否有未决的内核模式APC(`KernelApcPending`),是否有常规内核模式APC在进行中(`KernelApcInProgress`)，是否有未决的用户模式的APC(`UserApcPending`). 这些APC的环境保存在线程对象的`ApcStatePointer`域中。这个域是由2个元素组成的数组。即：`+0x138 ApcStatePointer : [2] Ptr32 \_KAPC\_STATE`

```
kd> dt  \_KAPC\_STATE

   +0x000 ApcListHead      : [2] \_LIST\_ENTRY

   +0x010 Process          : Ptr32 \_KPROCESS

   +0x014 KernelApcInProgress : UChar

   +0x015 KernelApcPending : UChar

   +0x016 UserApcPending   : UChar
```

```
typedef struct \_KAPC\_STATE {

    LIST\_ENTRY ApcListHead[MaximumMode];

    struct \_KPROCESS \*Process;

    BOOLEAN KernelApcInProgress;

    BOOLEAN KernelApcPending;

    BOOLEAN UserApcPending;

} KAPC\_STATE, \*PKAPC\_STATE, \*PRKAPC\_STATE;
```

主APC环境是位于线程对象的`ApcState`域，即：

`+0x034 ApcState         : \_KAPC\_STATE`

线程中等待在当前进程上下文中执行的APC保存在`ApcState`的队列中。无论何时，NT的APC派发器(dispatcher)和其他系统元件查询一个线程未决的APC时, 他们都会检查主APC环境，如果这里有任何未决的APC,就会马上被投递，或者修改它的控制变量稍后投递。

第二个APC环境是位于线程对象的`SavedApcState`域，当线程临时挂接到其他进程时，它是用来备份主APC环境的。`+0x14c SavedApcState    : struct \_KAPC\_STATE, 5 elements, 0x18 bytes`

一个线程调用`KeAttachProcess`，在另外的进程上下文中执行后续的代码时，`ApcState`域的内容就被拷贝到`SavedApcState`域。然后`ApcState`域被清空，它的APC队列重新初始化，控制变量设置为0，当前进程域设置为新的进程。这些步骤成功的确保先前在线程所属的进程上下文地址空间中等待的APC，当线程运行在其它不同的进程上下文时，这些APC不被传送执行。随后，`ApcStatePointer`域数组内容被更新来反映新的状态，数组中第一个元素指向`SavedApcState`域，第二个元素指向`ApcState`域，表明线程所属进程上下文的APC环境位于`SavedApcState`域。线程的新的进程上下文的APC环境位于`ApcState`域。最后，当前进程上下文切换到新的进程上下文。

对于一个APC对象，决定当前APC环境的是`ApcStateIndex`域。`ApcStateIndex`域的值作为`ApcStatePointer`域数组的索引来得到目标APC环境指针。随后，目标APC环境指针用来在相应的队列中存放apc对象.

当线程从新的进程中脱离时(`KeDetachProcess`), 任何在新的进程地址空间中等待执行的未决的内核APCs被派发执行。随后`SavedApcState`域的内容被拷贝回`ApcState`域。`SavedApcState`域的内容被清空，线程的`ApcStateIndex`域被设为`OriginalApcEnvironment`，`ApcStatePointer`域更新，当前进程上下文切换到线程所属进程。

## 使用APCs

设备驱动程序使用两个主要函数来利用APC, 第一个是`KeInitializeApc`，用来初始化APC对象。这个函数接受一个驱动分配的APC对象，一个目标线程对象指针，APC环境索引（指出APC对象存放于哪个APC环境），APC的kernel,rundown和normal例程指针，APC类型（用户模式或者内核模式）和一个上下文参数。 函数声明如下：

```
NTKERNELAPI

VOID

KeInitializeApc (

    IN PRKAPC Apc,

    IN PKTHREAD Thread,

    IN KAPC\_ENVIRONMENT Environment,

    IN PKKERNEL\_ROUTINE KernelRoutine,

    IN PKRUNDOWN\_ROUTINE RundownRoutine OPTIONAL,

    IN PKNORMAL\_ROUTINE NormalRoutine OPTIONAL,

    IN KPROCESSOR\_MODE ApcMode,

    IN PVOID NormalContext

    );

typedef enum \_KAPC\_ENVIRONMENT {

    OriginalApcEnvironment,

    AttachedApcEnvironment,

    CurrentApcEnvironment

} KAPC\_ENVIRONMENT;
```

`KeInitializeApc` 首先设置APC对象的`Type`和`Size`域一个适当的值，然后检查参数`Environment`的值，如果是`CurrentApcEnvironment`，那么`ApcStateIndex`域设置为目标线程的`ApcStateIndex`域。否则，`ApcStateIndex`域设置为参数`Environment`的值。随后，函数直接用参数设置APC对象`Thread`，`RundownRoutine`，`KernelRoutine`域的值。为了正确地确定APC的类型，`KeInitializeApc` 检查参数`NormalRoutine`的值，如果是`NULL`，`ApcMode`域的值设置为`KernelMode`，`NormalContext`域设置为`NULL`。如果`NormalRoutine`的值不是`NULL`，这时候它一定指向一个有效的例程，就用相应的参数来设置`ApcMode`域和`NormalContext`域。最后，`KeInitializeApc` 设置`Inserted`域为`FALSE`.然后初始化APC对象，并没有把它存放到相应的APC队列中。

可以了解到APC对象如果缺少有效的`NormalRoutine`，就会被当作内核模式APC.尤其是它们会被认为是特殊的内核模式APC.

实际上，I/O管理器就是用这类的APC来完成异步I/O操作。相反地，APC对象定义了有效的`NormalRoutine`，并且`ApcMode`域是`KernelMode`，就会被当作常规的内核模式APCs,否则就会被当作是用户模式APCs

`NTDDK.H`中`KernelRoutine`, `RundownRoutine`, and `NormalRoutine` 的定义如下：

```
typedef

VOID

(\*PKKERNEL\_ROUTINE) (

    IN struct \_KAPC \*Apc,

    IN OUT PKNORMAL\_ROUTINE \*NormalRoutine,

    IN OUT PVOID \*NormalContext,

    IN OUT PVOID \*SystemArgument1,

    IN OUT PVOID \*SystemArgument2

    );

typedef

VOID

(\*PKRUNDOWN\_ROUTINE) (

    IN struct \_KAPC \*Apc

    );

typedef

VOID

(\*PKNORMAL\_ROUTINE) (

    IN PVOID NormalContext,

    IN PVOID SystemArgument1,

    IN PVOID SystemArgument2

    );
```

通常，无论是什么类型，每个APC对象必须要包含一个有效的`KernelRoutine`函数指针。当这个APC被NT的APC dispatcher传送执行时，这个例程首先被执行。用户模式的APC必须包含一个有效的`NormalRoutine`函数指针，这个函数必须在用户内存区域。同样的，常规内核模式APC也必须包含一个有效的`NormalRoutine`，但是它就像`KernelRoutine`一样运行在内核模式。作为可选择的，任意类型的APC都可以定义一个有效的`RundownRoutine`，这个例程必须在内核内存区域，并且仅仅当系统需要释放APC队列的内容时，才被调用。例如线程退出时，在这种情况下，`KernelRoutine`和`NormalRoutine`都不执行，只有`RundownRoutine`执行。没有这个例程的APC对象会被删除。

投递APC到一个线程的动作，仅仅是操作系统调用`KiDeliverApc`完成的。执行APC实际上就是调用APC内的例程。

一旦APC对象完成初始化后，设备驱动调用`KeInsertQueueApc`来将APC对象存放到目标线程的相应的APC队列中。这个函数接受一个由`KeInitializeApc`完成初始化的APC对象指针，两个系统参数和一个优先级增量。跟传递给`KeInitializeApc`函数的参数context 一样，这两个系统参数只是在APC的例程执行时，简单的传递给APC的例程。

```
NTKERNELAPI

BOOLEAN

KeInsertQueueApc (

    IN PRKAPC Apc,

    IN PVOID SystemArgument1,

    IN PVOID SystemArgument2,

    IN KPRIORITY Increment

    );
```

在`KeInsertQueueApc` 将APC对象存放到目标线程相应的APC队列之前，它首先检查目标线程是否是APC queueable。如果不是，函数立即返回`FALSE`.如果是，函数直接用参数设置`SystemArgument1`域和`SystemArgument2` 域，随后，函数调用`KiInsertQueueApc`来将APC对象存放到相应的APC队列。

`KiInsertQueueApc` 仅仅接受一个APC对象和一个优先级增量。这个函数首先得到线程APC队列的spinlock并且持有它，防止其他线程修改当前线程的APC结构。随后，检查APC对象的`Inserted`域。如果是`TRUE`,表明这个APC对象已经存放到APC队列中了，函数立即返回`FALSE`.如果APC对象的`Inserted`域是`FALSE`.函数通过`ApcStateIndex`域来确定目标APC环境，然后把APC对象存放到相应的APC队列中，即将APC对象中的`ApcListEntry` 域链入到APC环境的`ApcListHead`域中。链入的位置由APC的类型决定。常规的内核模式APC,用户模式APC都是存放到相应的APC队列的末端。相反的，如果队列中已经存放了一些APC对象，特殊的内核模式APC存放到队列中第一个常规内核模式APC对象的前面。如果是内核定义的一个当线程退出时使用的用户APC,它也会被放在相应的队列的前面。然后，线程的主APC环境中的`UserApcPending`域被设置为`TRUE`。这时`KiInsertQueueApc` 设置APC对象的`Inserted`域为`TRUE`，表明这个APC对象已经存放到APC队列中了。接下来，检查这个APC对象是否被排队到线程的当前进程上下文APC环境中，如果不是，函数立即返回`TRUE`。如果这是一个内核模式APC，线程主APC环境中的`KernelApcPending`域设置为`TRUE`。

WIN32 SDK文档中是这样描述APC的： 当一个APC被成功的存放到它的队列后，发出一个软中断，APC将会在线程被调度运行的下一个时间片执行。然而这不是完全正确的。这样一个软中断，仅仅是当一个内核模式的APC（无论是常规的内核模式APC还是特殊的内核模式APC）针对于调用线程时，才会发出。随后函数返回`TRUE`。

1）如果APC不是针对于调用线程，目标线程在Passive权限等级处在等待状态；

2）这是一个常规内核模式APC

3）这个线程不再临界区

4）没有其他的常规内核模式APC仍然在进行中

那么这个线程被唤醒，返回状态是`STATUS\_KERNEL\_APC`。但是等待状态没有aborted。 如果这是一个用户模式APC，`KiInsertQueueApc`检查判断目标线程是否是alertable等待状态，并且`WaitMode`域等于`UserMode`。如果是，主APC环境的`UserApcPending`域设置为`TRUE`。等待状态返回`STATUS\_USER\_APC`，最后，函数释放spinlock，返回`TRUE`，表示APC对象已经被成功放入队列。

早期作为APC管理函数的补充，设备驱动开发者可以使用未公开的系统服务`NtQueueApcThread`来直接将一个用户模式的APC投递到某个线程。

这个函数内部实际上是调用了`KeInitializeApc` 和 `KeInsertQueueApc` 来完成这个任务。

## NT的APC派发器

NT检查是否线程有未决的APC. 然后APC派发器子程序`KiDeliverApc`在这个线程上下文执行来开始将未决的APC执行。注意，这个行为中断了线程的正常执行流程，首先将控制权给APC派发器，随后当`KiDeliverApc`完成后，继续线程的执行。

例如：当一个线程被调度运行时，最后一步，上下文切换函数`SwapContext` 用来检查是否新的线程有未决的内核APC.如果是，`SwapContext`要么（1）请求一个APC级别的软中断来开始APC执行，由于新线程运行在低的IRQL（Passive级别¬）。

或者（2）返回`TRUE`，表示新的线程有未决的内核APC。

`SwapContext`的返回值仅仅是特定系统函数可用的,这些系统函数调用`SwapContext`来强制切换线程上下文到另一个线程. 然后,当这些系统函数经过一段时间再继续时,他们通常检查`SwapContext`的返回值,如果是`TRUE`,他们就会调用APC派发器来投递内核APC到当前的线程. 例如:
