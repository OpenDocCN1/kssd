
```
8053e5cd 8bf8            mov     edi,eax    ;系统服务例程号
8053e5cf c1ef08          shr     edi,8        ;右移八位
8053e5d2 83e730          and     edi,30h    ; index
8053e5d5 8bcf            mov     ecx,edi    ; ecx = index
8053e5d7 03bee0000000    add     edi,dword ptr [esi+0E0h]    ;esi+0E0h = ServiceTable_ptr 代表是去找index=1还是=0的结构的项
8053e5dd 8bd8            mov     ebx,eax    ;系统服务例程号
8053e5df 25ff0f0000      and     eax,0FFFh    ;取后24位
8053e5e4 3b4708          cmp     eax,dword ptr [edi+8]    ;edi + 8的位置为MaxServiceNumber：最大的系统服务例程号
8053e5e7 0f8345fdffff    jae     nt!KiBBTUnexpectedRange (8053e332)
8053e5ed 83f910          cmp     ecx,10h    ;比较index是否为1
8053e5f0 751a            jne     nt!KiFastCallEntry+0xcc (8053e60c)
8053e5f2 8b0d18f0dfff    mov     ecx,dword ptr ds:[0FFDFF018h]
8053e5f8 33db            xor     ebx,ebx    ;ebx = 0
8053e5fa 0b99700f0000    or      ebx,dword ptr [ecx+0F70h]    ;调试的时候发现这里都是0
8053e600 740a            je      nt!KiFastCallEntry+0xcc (8053e60c)
8053e602 52              push    edx
8053e603 50              push    eax
8053e604 ff15e43f5580    call    dword ptr [nt!KeGdiFlushUserBatch (80553fe4)]
8053e60a 58              pop     eax
8053e60b 5a              pop     edx
8053e60c ff0538f6dfff    inc     dword ptr ds:[0FFDFF638h]    ;系统调用的次数+1
8053e612 8bf2            mov     esi,edx    ;三环的edx给了当前的esi
8053e614 8b5f0c          mov     ebx,dword ptr [edi+0Ch]    ;ebx指向了参数表的
8053e617 33c9            xor     ecx,ecx    ;ecx = 0
8053e619 8a0c18          mov     cl,byte ptr [eax+ebx]    ;读取参数的个数ArgumentSizeTable：提供每个例程所需要的参数大小，这个值将要用来从 caller 里复制多少个参数
8053e61c 8b3f            mov     edi,dword ptr [edi]    ;获取ServiceRoutineTable(提供真正的系统服务例程的地址)的地址
8053e61e 8b1c87          mov     ebx,dword ptr [edi+eax*4]    ;读取服务例程地址
8053e621 2be1            sub     esp,ecx    ;在当前栈上开辟空间容纳参数
8053e623 c1e902          shr     ecx,2    ;ecx/4 获取参数个数
8053e626 8bfc            mov     edi,esp    ;edi指向栈
8053e628 3b35d4995580    cmp     esi,dword ptr [nt!MmUserProbeAddress (805599d4)]    ;看是否属于用户空间
8053e62e 0f83a8010000    jae     nt!KiSystemCallExit2+0x9f (8053e7dc)
8053e634 f3a5            rep movs dword ptr es:[edi],dword ptr [esi]    ;复制参数到当前栈上
8053e636 ffd3            call    ebx    ;调用最终的服务例程
```

表示在windbg里面跟了一边 清晰多了！但是windbg太不好看了 呜呜呜！决定换回windbg preview??

画一手流程图！

![image-20210818110300919](https://bbs.kanxue.com/upload/attach/202109/873515_7MA6D3G3TQZA8SV.png)

![image-20210818111719339](https://bbs.kanxue.com/upload/attach/202109/873515_844V36HZKNDDCX3.png)

`ecx`用来指定`SYSEXIT`返回用户模式时的目标地址，当使用`INT 2E`进行系统调用的时候，由于`INT N`的指令会i东将中断发生时的`CS`和`EIP`寄存器压入栈中，当中断处理例程通过执行`iretd`返回时，`iretd`指令会让栈中保存的`cs`和`EIP`返回合适的位置，但是因为`SYSENTER`指令不会向栈中压入要返回的位置，所以`sysexit`指令需要通过其他机制知道需要返回的位置

| 发起系统调用的方式 | 入口内核例程 | 返回时的指令 | 返回时使用的内核例程 |
| --- | --- | --- | --- |
| `INT 2E` | `KiSystemService` | `iret` | `KiSystemCallExit` |
| `sysenter` | `KiFastCallEntry` | `sysexit` | `KiSystemCallExit2` |
| `syscall` | `kiFastCallEntry` | `sysret` | `KiSystemCallExit3` |

书上给了一个例子，来加深这个系统调用的一个理解，因为栈上记录了函数相互调用时的参数和返回地址等信息，栈回溯是从栈上去找到这些信息，然后显示出来的过程

![image-20210818140828233](https://bbs.kanxue.com/upload/attach/202109/873515_HR8PCYAJM58WTXU.png)

可以看到第一列是序号，第二列是每个函数的栈帧基地址，第三列是返回地址，第四列是执行的位置

### 逆向调用

之前我们知道了用户模式进入内核模式的两种方式，通过这两种形式，用户模式的代码可以调用位于内核模式的系统服务，如果我们想让内核模式的代码主动调用用户模式代码的时候，那么这种调用叫做逆向调用

逆向调用的过程：

内核代码使用`KiCallUserMode()`发起调用，接下来的执行的过程和系统返回的`KiSystemCallExit()`类似，只是进入用户模式的时候执行的是`NTDLL.dll`中的`KiUserCallbackDispatcher()`，而后该函数会调用内核希望调用的用户态函数，用户模式的工作完成后，执行返回动作的函数会执行`INT 2B`的指令，从而触发`0x2B`的异常

![image-20210818141425926](https://bbs.kanxue.com/upload/attach/202109/873515_DD8TE4TMVR6K2U9.png)

可以知道对应的异常的函数是`nt!KiCallbackReturn()`

#### 实例分析

如果要切换`.process` 一定要加上`/i` `/p` 这样`cr3`才会换回来 上下文才会过来

![image-20210818144644681](https://bbs.kanxue.com/upload/attach/202109/873515_FZB6E868EQGVSDV.png)

可以看到 栈回溯是 `nt!KeUserModeCallback() -> KiCallUserMode() -> KiUserCallbackDispatcher()`

![image-20210818150600107](https://bbs.kanxue.com/upload/attach/202109/873515_F4RZV45CCQ777QM.png)

可以看到整个过程有之前说到的`nt!KiFastCallEntry()`，通过服务号去找对应的函数，执行完后将消息组织好去调用`KeUserModeCallback()`函数，从而进行逆向调用，用户模式状态下的过程没有体现出来，但是可以看到`XyCallbackReturn()`函数用于返回内核模式，调用`int 2B` `nt!KiCallbackReturn()`返回内核模式

![image-20210818151407769](https://bbs.kanxue.com/upload/attach/202109/873515_3SSKZZHNSFSURF6.png)

### 线程

线程是进程中的生命，一个进程内有一个或者多个线程，进程创建初期或者进程退出和销毁的过程中，进程内可能没有任何线程

#### ETHREAD

NT内核使用`ETHREAD`结构来描述线程，在内核代码中，大多时候使用`ETHREAD`结构的地址来索引线程，执行`.thread`命令，就可以显示出当前线程的`ETHREAD`结构地址

![image-20210818152607482](https://bbs.kanxue.com/upload/attach/202109/873515_2NU526UDS7AZAFF.png)

查看一下`KTHREAD`的结构：

![image-20210818152646566](https://bbs.kanxue.com/upload/attach/202109/873515_X75WEDUKH2BK7GX.png)

包含了线程的各种的属性，`ETHREAD`开头的512字节是一个`KTHREAD`结构，也称作为线程控制块（`TCB`），里面主要是供内核调度线程时使用的

![image-20210818152921564](https://bbs.kanxue.com/upload/attach/202109/873515_4KRXDKFMF43WGHA.png)

`Header`字段为`Dispatcher_Header`类型，是NT内核的线程调度器的别名，代表了分发CPU时间片的意思

因为字段太多了，所以调试的时候谜一般不直接观察，使用扩展命令`!thread`，从而用这个扩展命令来更好的显示线程属性

![image-20210818153325342](https://bbs.kanxue.com/upload/attach/202109/873515_9PA4P7A2GDYGA48.png)

右上角的`RUNNING on processor 0`代表了这个线程正运行在0号CPU上，来源于`KTHREAD`结构的`state`字段

![image-20210818200954467](https://bbs.kanxue.com/upload/attach/202109/873515_EBYT565NT99J63H.png)

为枚举类型`KTHREAD_STATE`，共有9个值从而代表了线程的状态

| 状态 | 值 | 含义 |
| --- | --- | --- |
| `Initialized` | 0 | 正在创建和初始化 |
| `Ready` | 1 | 就绪，可以被分发器调度运行 |
| `Running` | 2 | 正在某个CPU运行 |
| `Standby` | 3 | 待命，每个CPU只有一个线程处于此状态，代表下一个要执行的线程 |
| `Terminated` | 4 | 结束执行 |
| `Waiting` | 5 | 等待，通常指线程调用了`sleep()`函数，取消西函数或者等待同步对象的各种函数，主动放弃执行机会 |
| `Transition` | 6 | 过渡状态，一般是因为线程已经可以运行，但是它的内核态栈被交换出了内存，一旦栈被交换回内存，便进入就绪状态 |
| `DeferredReady` | 7 | 延迟就绪，为了缩短扫描调度数据库时的加锁时间，内核就把就绪的线程先设置为此状态 |
| `GateWait` | 8 | 门等待，在等待门分发器对象时，进入此状态 |

可以看到当前的线程，在`Running`

![image-20210818154629168](https://bbs.kanxue.com/upload/attach/202109/873515_K3REZR7P2BUYMKK.png)

NT内核给每个CPU定义了一个名为处理控制块的`PRCB`的庞大结构，这个结构中有一个`DispatcherReadyListHead`的数组，包括了32个元素，代表了线程的32个优先级，每个元素是一个`LIST_ENTRY`的结构，从而起到链表头的作用，用来挂接对应优先级的就绪线程

![image-20210818163256805](https://bbs.kanxue.com/upload/attach/202109/873515_GW7R6DTTB5WRQJV.png)

各个线程状态之间的切换关系：

![image-20210818214052568](https://bbs.kanxue.com/upload/attach/202109/873515_3RF2QHB3YFCV9M6.png)

上面所说的那个链表的事情??找到了答案，我用的是xp的系统32位的，那时候的这个结构在`TEB`中的`WaitListEntry`中

![image-20210818190008784](https://bbs.kanxue.com/upload/attach/202109/873515_XX8GZ7HT3Q5PCHS.png)

如果线程处于等待状态下，那么`!thread`命令会显示出等待的原因，在`KTHREAD`结构中有一个叫做`WaitReason`的字段

![image-20210818213650162](https://bbs.kanxue.com/upload/attach/202109/873515_D3E44GHWGT6AAGU.png)

从而记录线程等待的原因，长度为1字节，是枚举的类型，名为`KWAIT_REASON`，在公开的`PDB`符号文件中，包含了这个枚举类型的定义

#### TEB

NT内核定义了线程环境快`TEB`来描述线程的用户空间的信息，包括了用户态栈，异常处理，错误码，线程局部存储等等

windbg调试应用程序的时候，可以使用`!teb`来显示当前线程的`TEB`结构的位置，我用的是本机win10 挂载的`notepad`

![image-20210818214538501](https://bbs.kanxue.com/upload/attach/202109/873515_EVZMYA2MWUUEPJ3.png)

NT内核会使用CPU的硬件机制来快速定位当前线程的`TEB`，所以内核在创建线程的时候，会分配专门的内存也来用作`TEB`，将地址记录在`KTHREAD`中，所以`TEB`的地址总是按照页对齐的

同时也可以用`dt _TEB`的命令查看`TEB`的信息

![image-20210819101024955](https://bbs.kanxue.com/upload/attach/202109/873515_FZC4SEBDV3EW33F.png)

### WOW进程

现在大多数的windows系统是64位的，运行在支持64位CPU的地方，但是为了兼容32位的应用程序，64位windows系统可以运行32位的应用程序，这样运行在64位内核上的32位进程有一个专门的名字，叫做`WoW64`（Windows 32 on Windows 64）进程

#### 架构

![image-20210819110047070](https://bbs.kanxue.com/upload/attach/202109/873515_3KSWTCZFZ8UM5AR.png)

由于32位的代码是不能与64位内核交互的，所以中间转换层为了解决这个问题设计，转接层本身为64位的模块，他给32位的应用程序营造了一个32位的环境，主要负责指针长度的转换和解决API兼容等问题

#### 工作过程

可以使用`.effmach`命令再两种代码间进行切换

`.effmach amd64`切换到64位的模式下
