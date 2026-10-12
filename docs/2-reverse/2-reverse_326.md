
# Debug Tutorial (完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-7.html`](https://www.pediy.com/kssd/pediy12/104828-7.html)

## 锁和同步对象

### 介绍

欢迎来到这个系列的第 7 篇。在这部分中，我们将学习一些关于锁和同步对象的知识。在这篇教程中，我们会稍作改变，我们将同时使用用户态调试器和内核态调试器。这样我们就能更好的了解他们。

### 什么是死锁

这是多线程中最基本的一个问题。当一个线程因为一个对象的无限期锁住，永远不可能得到一个资源，比如临界区，锁，或其他种类的同步对象，这就造成了死锁。最简单的例子就是有两个线程，两个锁，锁 `A` 用于从数据库读数据，锁 `B` 用于从数据库写数据。

线程 1 获得锁 `A`，并从数据库中读数据，线程 2 同时得到锁 `B`，并向数据库写数据。在没有释放各自的锁之前，线程 1 想得到锁 `B` 来写数据，线程 2 想得到锁 `A` 来读数据。两个线程都不可能完成，因为它们各自拥有对方的锁。这是一个简单的例子。

这并不是死锁的唯一形式。可能一个线程在等待一个事件，另一个会将这个事件置位的线程却已经退出了，这个线程也会形成死锁。

还有其他很多类型。解决这个问题的基本步骤如下。

1. 找到谁正在占有这个资源
2. 找出是否这个占有线程在等待其它资源
3. 找出当前线程拥有的资源

基本上就是找出哪些线程拥有哪些资源，哪些线程在等待哪些资源。一旦找到这些之后，文件很快就能解决。

## 用户态下可用的对象

用户态下，我们能使用哪些对象呢？在这部分中，我们将探索一下。

### 事件

事件就是一个有信号状态的对象。没人能占有一个事件，但是它却能用于同步。我们可以给事件命名，也就是说事件是全局的，许多进程都能打开它。也可以是没有名字的，这样只能在这个进程空间中看见。

事件可以是人工重置，也可以是自动重置的。自动重置就是说一旦等待线程得到信号之后，系统自动将事件置为无信号状态。如果是人工重置，线程就必须自己将其置为无信号。在内核中创建事件使用 `KeInitializeEvent`，并且事件可以是通知类型的或同步类型的，你可以在察看句柄信息的时候看到这个类型。通知类型是人工重置的，同步类型是自动重置的。如果你想得到更多关于事件的信息，可以在 `MSDN` 中查询关于 `CreateEvent` 和 `KeInitializeEvent` 的相关信息。

和其它大部分东西一样，在用户态中，事件仅仅是一个句柄，因此我们可以使用 `!handle`。

```
0:001> !handle 7e4 ff

Handle 7e4

  Type          Event

  Attributes    0

  GrantedAccess 0x1f0003:

         Delete,ReadControl,WriteDac,WriteOwner,Synch

         QueryState,ModifyState

  HandleCount   2

  PointerCount  4

  Name          <NONE>

  Object Specific Information

    Event Type Auto Reset

    Event is Set
```

你可以看到，这个事件当前是置位状态，并且是一个自动重置事件。事件作为同步对象来使用的一个真实例子就是 `DBGVIEW`，我们来看看 `OutputDebugString`。

Application

1. Acquire Mutex
2. Open Memory Mapped File
3. Wait for Buffer is Ready event
4. Write to Memory Mapped File
5. Signal Buffer Data Available event
6. Close Handles

DbgView

1. Wait for Buffer Data Available event
2. Read Buffer Data.
3. Signal Buffer is ready event
4. Goto 1

你可以看到，互斥量是用来保护内存影射文件的写入的，而这两个事件是用来同步程序和 `DBGVIEW` 对这个文件的读写的。

### 互斥量

互斥量是能通过名字使用的全局同步对象。也就是说，多个程序可以使用同一个互斥量，它是存在于内核中的。互斥量一次只允许一个线程获得。

```
0:005> !handle 2c0 ff

Handle 2c0

  Type          Mutant

  Attributes    0

  GrantedAccess 0x1f0001:

         Delete,ReadControl,WriteDac,WriteOwner,Synch

         QueryState

  HandleCount   2

  PointerCount  3

  Name          <NONE>

  Object Specific Information

    Mutex is Free

0:005> !handle 2b0 ff

Handle 2b0

  Type          Mutant

  Attributes    0

  GrantedAccess 0x120001:

         ReadControl,Synch

         QueryState

  HandleCount   17

  PointerCount  19

  Name          \\BaseNamedObjects\\ShimCacheMutex

  Object Specific Information

    Mutex is Free
```

互斥量是 `Mutant` 类型的。其中一个有名字，另外一个没有名字。

注意到，`Windows2003` 显示的句柄信息比 `Windows2000` 多，并且因为某些问题，比如死锁，请求信息的时候，不是所有的信息都会显示。因此，内核调试器或 `handle.exe` 这样使用驱动的程序可以直接从对象读取信息。你可以在 `part 5` 中得到更多关于句柄的信息。

那么，互斥量在使用的时候是什么样的呢？有两种情形，你的线程正在使用这个互斥量，或别的进程中的线程正在使用这个互斥量。

```
0:005> !handle 50 ff

Handle 50

  Type          Mutant

  Attributes    0

  GrantedAccess 0x1f0001:

         Delete,ReadControl,WriteDac,WriteOwner,Synch

         QueryState

  HandleCount   2

  PointerCount  3

  Name          <NONE>

  Object Specific Information

    Mutex is Owned
```

可以看到”`Mutex is Owned`”。进程中还有其他很多互斥量，你不知道它们是被占有还是空着的，那我们该如何知道谁在占有这个互斥量呢？

`Handle.exe` 也不能告诉我们占有者，我们需要到内核中去看。

因此，我创建了一个互斥量，并调用 `WaitForSingleObject`，然后在内核中的 `NtWaitForSingleObject` 下断点，然后跟进去。之后，我们可以看到这个句柄对应一个对象，然后调用 `KeWaitForSingleObject`。这个函数检察它是否已经被占有。

```
eax=00000001 ebx=fcc724a0 ecx=00000000 edx=00000000 esi=fcd19860 edi=fcd198cc

eip=8042d697 esp=fb72bcc0 ebp=fb72bce0 iopl=0         ov up ei ng nz na pe cy

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000         efl=00000a83

nt!KeWaitForSingleObject+0x1d5:

8042d697 ff4b04         dec   dword ptr [ebx+0x4] ds:0023:fcc724a4=00000001

...

eax=00000000 ebx=fcc724a0 ecx=00000000 edx=00000000 esi=fcd19860 edi=fcd198cc

eip=8042d6aa esp=fb72bcc0 ebp=fb72bce0 iopl=0         nv up ei ng nz ac po cy

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000         efl=00000297

nt!KeWaitForSingleObject+0x1e8:

8042d6aa 897318         mov     [ebx+0x18],esi    ds:0023:fcc724b8=00000000

...

kd> !object fcc724b8

Object: fcc724b8  Type: (fcc724a8) 

    ObjectHeader: fcc724a0 

    HandleCount: 0  PointerCount: 524290 

    Directory Object: 00000001  Name: (**** Name not accessable ****) 

kd> dd fcc724b8 l1

fcc724b8  fcd19860 

kd> !thread fcd19860 1

THREAD fcd19860  Cid 214.418  Teb: 7ffdc000  Win32Thread: 00000000 RUNNING
```

`EBX` 指向了对象头。因此 `ObjectHeader+18` 就是拥有线程。

### 信号量

和其它一样，这个也是有内核对象的。唯一不同的是，信号量可以有多于 1 个的计数，而互斥量一次只能被一个线程占有。

```
0:002> !handle 0 f semaphore

Handle 90

  Type          Semaphore

  Attributes    0

  GrantedAccess 0x1f0003:

         Delete,ReadControl,WriteDac,WriteOwner,Synch

         QueryState,ModifyState

  HandleCount   2

  PointerCount  3

  Name          <NONE>

  Object Specific Information

    Semaphore Count 15

    Semaphore Limit 16

1 handles of type Semaphore
```

从用户态调试器中，我们可以看到有多少个信号量，还有他们可以拥有的线程数量。0 就表示可用的都用完了。用户态调试器也把信号量和互斥量看作不同的对象。下面我们来看看我们在这个对象中发现什么。

```
kd> !handle 90 ff fccde020

processor number 0

PROCESS fccde020  SessionId: 0  Cid: 03cc    Peb: 7ffdf000  ParentCid: 03d8 

    DirBase: 00e7b000  ObjectTable: fccbfae8  TableSize:  36.

    Image: mspaint.exe

Handle Table at e1e62000 with 36 Entries in use

0090: Object: fcec1680  GrantedAccess: 001f0003 

Object: fcec1680  Type: (fcebd620) Semaphore 

    ObjectHeader: fcec1668 

        HandleCount: 1  PointerCount: 1 

kd> dd fcec1668 

fcec1668  00000001 00000001 fcebd620 00000000 

fcec1678  fcc75888 00000000 00050005 00000010 

fcec1688  fcec1688 fcec1688 00000010 7ffddfff 

fcec1698  00000000 00000000 03018002 644c6d4d 

fcec16a8  fcec19a8 fce73b48 00650074 0052006d 

fcec16b8  006f006f 005c0074 fc8f5000 fc90b87c 

fcec16c8  00025000 00500050 e134a588 00160016 

fcec16d8  fcec87a8 09104000 004b0002 ffffffff
```

上面是我们获得对象之前，对象头的一个 Dump。下面我们来获取这个对象，看看会发生什么。

```
kd> !handle 90 ff fccde020

processor number 0

PROCESS fccde020  SessionId: 0  Cid: 03cc    Peb: 7ffdf000  ParentCid: 03d8 

    DirBase: 00e7b000  ObjectTable: fccbfae8  TableSize:  36.

    Image: mspaint.exe

Handle Table at e1e62000 with 36 Entries in use

0090: Object: fcec1680  GrantedAccess: 001f0003 

Object: fcec1680  Type: (fcebd620) Semaphore 

    ObjectHeader: fcec1668 

        HandleCount: 1  PointerCount: 1 

kd> dd fcec1668

fcec1668  00000001 00000001 fcebd620 00000000 

fcec1678  fcc75888 00000000 00050005 0000000f 

fcec1688  fcec1688 fcec1688 00000010 7ffddfff 

fcec1698  00000000 00000000 03018002 644c6d4d 

fcec16a8  fcec19a8 fce73b48 00650074 0052006d 

fcec16b8  006f006f 005c0074 fc8f5000 fc90b87c 

fcec16c8  00025000 00500050 e134a588 00160016 

fcec16d8  fcec87a8 09104000 004b0002 ffffffff
```

可以看到，计数减少了，因此我们可以看到信号量的数量和现在可用的数量的存储地址。另外我们还是不知道哪个线程正在占有这个信号量，我们该如何找到他呢？

经过一番调试，我们发现上面的代码并不适用于信号量，线程上下文并没有保存下来。很不幸，信号亮并不会被一个或一堆线程占有，它们只是基于计数工作的。

事实上，如果你在同一个线程中调用几次 `WaitForSingleObject` 后，计数也在减少。这就意味着，编程的时候你必须注意，因为你可能不小心的就释放了信号量，即使你并没有自己去减少这个计数。

### 临界区

另外还有用户态的数据结构来表示同步对象。它们用来在同一个进程中对一个资源进行保护。由于它们是用户态的，所以调试起来很简单。

我们可以找到谁在占有它，你还可以使用 `!locks` 或 `!locks v` 来得到和临界区有关的信息。

```
0:000> !critsec ntdll!LdrpLoaderLock

CritSec ntdll!LdrpLoaderLock+0 at 77FC1774

LockCount          0

RecursionCount     1

OwningThread       9ac

EntryCount         0

ContentionCount    0

**** Locked
```

最有名的就是 `Windows` 的 loader 锁。当你加载库，创建或销毁线程的时候它会锁住。我们可以看到，这个锁现在正处于锁住状态。我们可以使用 `!critsec` 来得到这个锁的信息。来看看谁在拥有这个线程。

```
0:001> ~*

  0  Id: e28.9ac Suspend: 1 Teb: 7ffde000 Unfrozen

     Start: notepad!WinMainCRTStartup (01006ae0)

     Priority: 0  Priority class: 32

. 1  Id: e28.aa0 Suspend: 1 Teb: 7ffdd000 Unfrozen

     Start: ntdll!DbgUiRemoteBreakin (77f5f2f2)
```



Priority: 0  Priority class: 32  

这个进程中只有两个线程，找到这个线程很简单。  

```
0:001> !critsec ntdll!LdrpLoaderLock  

CritSec ntdll!LdrpLoaderLock+0 at 77FC1774  

LockCount          NOT LOCKED  

RecursionCount     0  

OwningThread       0  

EntryCount         0  

ContentionCount    0  
```

这是当临界区没有锁住时的输出。临界区相对于互斥量还有什么优势呢？因为它们是用户态的，因此它们不用进入内核，也就不用和其它进程去争抢。这让它实现起来更快，调试的时候更简单。对这个地址使用 `dd` 命令，你也可以看到这个数据结构很简单。  

  如果这个线程不存在，那么这个线程就是在释放这个锁之前就已经退出了。  

### 内核态可用的对象  

  这里我们讲述一些内核态下的对象。事件，信号量，互斥量都能在内核态下使用。事实上，用户态的程序也都是进入内核态，使用相同的函数然后创建这些对象的。另外还有其他一些内核态的对象。它们是自旋锁和 `ERESOURCE`。  

### 自旋锁  

  自旋锁是用来在多处理器系统中保护资源的一种同步对象。自旋锁和临界区的不同之处在于，第二个处理器会在这个锁上一直自旋知道获取它，而不是让其他线程去运行。  

  在单处理器系统上，自旋锁只是将 `IRQL` 增高，以使线程不能切换，让代码运行。但是你也不能使用分页内存，并且你只能进行一小部分操作。  

`hal!KfAcquireSpinLock`:  

```
80069850 33c0            xor     eax,eax  
80069852 a024f0dfff      mov     al,[ffdff024]  
80069857 c60524f0dfff02  mov     byte ptr [ffdff024],0x2  
8006985e c3              ret  
```

这是指向全局 `IRQL` 的一个地址。调用 `KeAcquireSpinLock` 会将 `IRQL` 置为 `2`。然后它保存以前的 `IRQL`，用来在 `KeReleaseSpinLock` 中恢复。  

  `IRQL` 表示操作系统当前处于的一个中断级。下面是 `NTDDK` 中的定义。  

```
#define PASSIVE\_LEVEL 0             // Passive release level  
#define LOW\_LEVEL 0               // Lowest interrupt level  
#define APC\_LEVEL 1               // APC interrupt level  
#define DISPATCH\_LEVEL 2            // Dispatcher level  
```

因此，自旋锁会把操作系统升到 `DISPATCH\_LEVEL`。你可以在 `MSDN` 中找到更多关于 `IRQL` 的信息。  

自旋锁函数在多处理器系统中会有一些不同。它们会一直自旋直到获得它。  

“`LOCK`”这个汇编指令会锁住总线，防止其他处理器读或写同一块内存区域。以 `0` 为参数使用 `BTS` 指令，就是把 `0` 放入一个 carry flag，然后把这个 `0` 位置为 `1`。  

`JB` 指令会在 carry flag 为 `1` 时(也就是说它以前是 `1`)跳转。如果 `0` 位是 `1`，它会做一个 `test`，如果 `0` 位不是 `1`，它就会跳回去然后再继续。如果 `0` 位是 `1`，就意味着它正在被占有，因此它会做一个”`pause`”。  

`hal!KfAcquireSpinLock`:  

```
80065420 8b158000feff    mov     edx,[fffe0080]  
80065426 c7058000feff41000000 mov dword ptr [fffe0080],0x41  
80065430 c1ea04          shr     edx,0x4  
80065433 0fb68280a30680  movzx   eax,byte ptr [edx+0x8006a380]  
8006543a f00fba2900      lock    bts dword ptr [ecx],0x0  
8006543f 7203            jb      hal!KfAcquireSpinLock+0x24 (80065444)  
80065441 c3              ret  
80065442 8bff            mov     edi,edi  

0: kd> u  

hal!KfAcquireSpinLock+0x24:  
80065444 f70101000000    test    dword ptr [ecx],0x1  
8006544a 74ee            jz      hal!KfAcquireSpinLock+0x1a (8006543a)  
8006544c f390            pause  
8006544e ebf4            jmp     hal!KfAcquireSpinLock+0x24 (80065444)  
```

这是自旋锁工作的本质，事实上并没有很多工作，并且大部分程序都不会用到自旋锁。一般情况下，信号量或互斥量已经足够了。如果你想使用自旋锁，可以去阅读 `MSDN`。另外还有”queued”自旋锁，可以提供更好的表现。  

### The `ERESOURCE`  

  下面，我来解释 `ERESOURCE`：  

A.  你可以读 `MSDN`，我不想重复已有的信息  

B.  如果你不了解他们，你可能不会使用它们，也不需要调试它们。这是调试文章，而不是编程文章。  

但是，我会大概的介绍以下。`ERESOURCE` 是允许你在内核中共享去独占访问的一个数据结构。共享的意思是多个线程可以获得它，独占的意思是只有一个线程可以获得它。  

  需要注意的是，`ERESOURCE` 存在于非分页内存的一个全局链表中。这就是说，如果你释放这块内存或覆盖掉这个数据结构，可能会造成崩溃。  

  在内核调试器中，你可以使用 `!locks` 来 Dump 出所有的系统中所有的锁。  

```
kd> !locks  

\*\*\*\* DUMP OF ALL RESOURCE OBJECTS \*\*\*\*  

KD: Scanning for held locks.....................................  

Resource @ 0xfceba0c0    Shared 1 owning threads  

    Threads: fcebeda3-01<\*> \*\*\* Actual Thread FCEBEDA0  

KD: Scanning for held locks.  

1814 total locks, 1 locks currently held  
```

  这里会显示哪个线程正在占有这个锁，还会显示正在这个锁上等待的所有线程。你可以使用 `!locks <address>` 得到信息，还可以设置一些标志，比如 `!locks v`。  

  我们得到的最重要的信息就是 `!locks` 可以列出占有线程和等待线程，因此调试这些会很简单。上面看到的那个锁只有一个占有者，并且没有等待者。在 `Windows XP/2003`，你还可以使用 `dt _ERESOURCE` 显示内部信息。  

### 总结  

  这里我们已经概括了用户态和内核态常用的同步对象。另外还有其他很多方法，比如 `InterlockedDecrement`，`LockFile` 和其他一些程序员自己实现的同步方法。这里我们不会讲述这些，那需要你自己去解决和调试。
