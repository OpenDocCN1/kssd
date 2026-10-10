

# 异常处理机制探索与分析 (代码在 33 楼)

> 原文：[`www.pediy.com/kssd/pediy10/59368.html`](https://www.pediy.com/kssd/pediy10/59368.html)

各位新年快乐:rose :rose   
本文成型感谢 shoooo 大牛,下文还是称为大 S  
大 S 问偶 int3 具体是怎么执行的 偶无法回答.于是认真学习了 1 下  
  
我们执行 int3 立刻会进入系统中断门  

引用: 什么是中断(interrupt)  
    "中断被定义为当一个事件发生时，改变处理器的指令序列。这样的事件可由 CPU 芯片  
内部或者外部硬件产生电信号产生"  
(摘自: "Understanding the Linux kernel," O'Reilly publishing.)Intel 参考手册上指出“同步中断”(在一个指令执行完成后，由 CPU 控制单元产生的)作为“异常”。  
异步中断（可能会在任意时刻由其他硬件产生的）才称为“中断”。中断被外部的 I/O 设备产生。  
但是异常是由编程错误或者是由反常情况（必须由内核来处理）触发的。在该文档中，  
术语“中断信号”既指异常又指中断。  
    中断分为两种类型：可屏蔽中断--它在短时间片段里可被忽略；不可屏蔽中断--它必须被立即处理。  
不可屏蔽中断是由紧急事件产生例如硬件失败。著名的 IRQS（中断请求）失败划为可屏蔽中断。  
   异常被分为不同的两类：处理器产生的异常(Faults, Traps, Aborts)和编程安排的  
异常（用汇编指令 int or int3 触发）。后一种就是我们经常说到的软中断  
  
异常列表  
  
\--------------------------------------------------------------------------+  
number  | Exception             | Exception Handler               |  
\--------------------------------------------------------------------------+  
0       | Divide Error            | divide\_error()                  |  
1       | Debug                | debug()                         |  
2       | Nonmaskable Interrupt      | nmi()                           |  
3       | Break Point            | int3()                  |  
4       | Overflow            | overflow()                      |  
5       | Boundary verification        | bounds()                        |  
6       | Invalid operation code    | invalid\_op()                    |  
7       | Device not available        | device\_not\_available()          |  
8       | Double Fault                    | double\_fault()                  |  
9       | Coprocessor segment overrun    | coprocesseur\_segment\_overrun()  |  
10      | TSS not valid            | invalid\_tss()                   |  
11      | Segment not  present        | segment\_no\_present()            |  
12      | stack exception         | stack\_segment()                 |  
13      | General Protection         | general\_protection()            |  
14     | Page Fault            | page\_fault()                    |  
15      | Reserved by Intel        | none                            |  
16      | Calcul Error with float virgul| coprocessor\_error()             |  
17      | Alignement check        | alignement\_check()              |  
18      | Machine Check            | machine\_check()                 |  
\--------------------------------------------------------------------------+  
  
3       | Break Point            | int3()                  |  
我们关心的就是这个 当异常出现时会发生什么 ?  
    当一个中断发生，当前中断的中断处理函数被执行。该处理函数不是真正的处理异常函数，  
它仅仅做个跳转，跳转到更好的处理函数。  
  
异常就像中断，不管是什么原因(“软异常”除外)所引起，一旦发生首先进入的是内核中的异常响应/处理程序的入口，这就是类似于 KiTrap0()那样的底层内核函数，只是因为引起异常的原因不同而进入不同的入口，就像对于不同的中断向量有不同的入口一样。在内核中，仍以页面异常为例，正如读者已经看到，CPU 会从 KiTrap14()进入函数 KiPageFaultHandler()。在那儿，如果所发生的并非如“缺页”或“写时复制(Copy-On-Write)”那样的“正常”异常，就要根据 CPU 在发生异常时所处的空间而分别调用 KiKernelTrapHandler()或 KiUserTrapHandler()。如果调用的是 KiKernelTrapHandler()，就会顺着 KPCR 数据结构中的“异常(处理)队列”、即 ExceptionList，依次让各个节点认领。如果被认领，就会通过 SEHLongJmp()长程跳转到当初通过\_SEH\_HANDLE{}给定的代码中  
  
对于发生于用户空间的异常，这里应该做些什么。显然，用户空间的异常不应靠内核里面的程序处理，应用软件理应为此作好了准备。前面讲过，Windows 的 SEH 机制并不是仅为内核而设计的，用户空间的程序同样可以使用类似于\_SEH\_TRY{} \_SEH\_HANDLE{} \_SEH\_END 那样的手段为应用程序提供保护。事实上，在通过 NtCreateThread()创建的线程首次被调度运行时，整个线程的执行都是作为一个 SEH 域而受到保护的  
用户空间的每个线程都有一个 ExceptionList，只不过这个队列在每个线程的 TEB 中，而不是在 KPCR 中。既然内核中的 ExceptionList 是由 KiDispatchException()加以处理的，用户空间就应该有个类似于 KiDispatchException()的函数。事实上，动态连接库 ntdll.dll 中的 KiUserExceptionDispatcher()就是用户空间 SEH 处理的总入口。  
  
KiUserExceptionDispatcher  位于 NTDLL.DLL 中，它是异常发生后执行的起点。这样说也不是百分之百的准确。例如，在 Intel 体系下，异常会使控制转到一个  ring 0 （内核模式）的处理程序。此处理程序由对应此异常的中断描述符表表项所定义。我将跳过所有的内核模式代码并假设发生异常时 CPU 直接执行 KiUserExceptionDispatcher。   
KiUserExceptionDispatcher  的关键就是对 RtlDispatchException 的调用。这个调用启动了对注册的异常处理程序的查找。如果处理程序处理了异常并继续执行，则对  RtlDispatchException 的调用不再返回。如果 RtlDispatchException 返回了，则有两种可能：要么调用了  NtContinue 使进程继续，要么就是产生了另一个异常。若是后者，异常就不能再继续了，进程必须结束。接着说   RtlDispatchExceptionCode，这就是遍历异常帧的代码。函数获得一个指向 EXCEPTION\_REGISTRATIONs  链表的指针并遍历每一个节点查找处理程序。因为堆栈可能崩溃掉，这个函数非常谨慎。在调用每个 EXCEPTION\_REGISTRATION 指定的处理程序之前，代码要保证在线程堆栈中 EXCEPTION\_REGISTRATION 是 DWORD 对齐的且前面的  EXCEPTION\_REGISTRATION 的地址高。  
  
RtlDispatchException 并不直接调用 EXCEPTION\_REGISTRATION 结构体中指定的地址，而是调用  RtlpExecuteHandlerForException 来做这个脏累活儿。根据  RtlpExecuteHandlerForException 内部发生的情况，RtlDispatchException 要么继续遍历异常帧要么产生另一个异常。这个二级异常指示异常回调函数中出现问题不能继续执行。RtlpExecuteHandlerForException 的代码和另一个函数 RtlpExecutehandlerForUnwind 紧密相关。我在前面讲 unwinding 时曾提到这个函数。这两个函数都在将控制送到 ExecuteHandler 函数之前用不同的值加载 EDX 寄存器。换种说法就是  RtlpExecuteHandlerForException 和 RtlpExecutehandlerForUnwind 是同一个  ExecuteHandler 函数的不同的前端。  
  
ExecuteHandler 就是 EXCEPTION\_REGISTRATION 的 handler 域被取出和执行的地方。也许看上去有些奇怪，对异常回调函数的调用本身也被一个结构化异常处理程序封装了起来。在这里使用 SEH 尽管有点儿怪，但认真考虑一下还是合理的。如果异常回调引起了另一个异常，操作系统需要知道此事件。根据异常是发生在初始的回调还是 unwind 中的回调，ExecuteHandler 返回  DISPOSITION\_NESTED\_ EXCEPTION 或 DISPOSITION\_COLLIDED\_UNWIND。这两个可都是“红色警戒！立即关闭！”级别的代号。  
  
发生异常时系统的处理顺序(by Jeremy Gordon):  
    1.系统首先判断异常是否应发送给目标程序的异常处理例程,如果决定应该发送,并且目标程序正在被调试,则系统  
    挂起程序并向调试器发送 EXCEPTION\_DEBUG\_EVENT 消息.呵呵,这不是正好可以用来探测调试器的存在吗?  
    2.如果你的程序没有被调试或者调试器未能处理异常,系统就会继续查找你是否安装了线程相关的异常处理例程,如果  
    你安装了线程相关的异常处理例程,系统就把异常发送给你的程序 seh 处理例程,交由其处理.  
    3.每个线程相关的异常处理例程可以处理或者不处理这个异常,如果他不处理并且安装了多个线程相关的异常处理例程,  
        可交由链起来的其他例程处理.  
    4.如果这些例程均选择不处理异常,如果程序处于被调试状态,操作系统仍会再次挂起程序通知 debugger.  
    5.如果程序未处于被调试状态或者 debugger 没有能够处理,并且你调用 SetUnhandledExceptionFilter 安装了最后异  
    常处理例程的话,系统转向对它的调用.  
    6.如果你没有安装最后异常处理例程或者他没有处理这个异常,系统会调用默认的系统处理程序,通常显示一个对话框,  
    你可以选择关闭或者最后将其附加到调试器上的调试按钮.如果没有调试器能被附加于其上或者调试器也处理不了,系统  
    就调用 ExitProcess 终结程序.  
    7.不过在终结之前,系统仍然对发生异常的线程异常处理句柄来一次展开,这是线程异常处理例程最后清理的机会.  

很长的介绍文字,希望没有看晕![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")  
简单的说,就是我们执行 int3 之后系统会让我们进入 int3 处理函数,KiUserExceptionDispatcher 就是我们的处理函数  
还记得 softworm 大侠的那个脱壳机吗?  
  
KiUserExceptionDispatcher 是一个永远永远都不会返回的过程，它将要调用 NtContinue 或者是 NtRaiseException。我们看看都发生了什么：  
\- 异常发生  
\- ntoskrnl.exe 通过 KiTrapXX 接手控制权  
\- KiTraps 实际上是 IDT 的入口(?)，并且根据不同的异常在 KiTrapXX 的入口处有两种可能的堆栈布局：  
+---------------+    +---------------+  
|     EFLAGS      |    |      EFLAGS     |  
+---------------+    +---------------+  
|       CS         |   |         CS       |  
+---------------+   +---------------+  
|       EIP        |   |        EIP     |  
+---------------+   +---------------+  
            |   Error Code   |  
            +---------------+  
  
因为一些异常并不引发错误，同时也为了从 ring0 中退出来时更容易些，不管什么异常发生了一些 KiTrapXX 都将 0 压入堆栈模仿代码，比如 KiTrap01 和 KiTrap03：  
\_KiTrap01  
0008:804D8D7C PUSH 00 <--- dummy Error Code  
0008:804D8D7E MOV WORD PTR \[ESP+02\],0000  
0008:804D8D85 PUSH EBP  
0008:804D8D86 PUSH EBX  
0008:804D8D87 PUSH ESI  
0008:804D8D88 PUSH EDI  
0008:804D8D89 PUSH FS  
\_KiTrap03  
0008:804D915B PUSH 00 <--- dummy Error Code  
0008:804D915D MOV WORD PTR \[ESP+02\],0000  
0008:804D9164 PUSH EBP  
0008:804D9165 PUSH EBX  
0008:804D9166 PUSH ESI  
0008:804D9167 PUSH EDI  
0008:804D9168 PUSH FS  
但是 KiTrap0E (内存页错误处理程序) 并没有将 0 压入堆栈因为错误代码存在了堆栈中。  
\_KiTrap0E  
0008:804DAF25 MOV WORD PTR \[ESP+02\],0000  
0008:804DAF2C PUSH EBP  
0008:804DAF2D PUSH EBX  
0008:804DAF2E PUSH ESI  
0008:804DAF2F PUSH EDI  
0008:804DAF30 PUSH FS  
0008:804DAF32 MOV EBX,00000030  
从中断中返回是由一个简单的 IRETD 指令完成的，它与 ret 指令相近，也是跳转到堆栈中所保存的 EIP。异常处理完毕之后，ring0 确定要调用 KiUserExceptionDispatcher 时它就会将 KiUserExceptionDispatcher 的地址存储在堆栈中，所以 IRETD 只是简单的返回了 KiUserExceptionDispatcher ：  
0008:804F5A0F MOV EAX,\[\_KeUserExceptionDispatcher\]  
0008:804F5A14 MOV \[EBX+68\],EAX  
:dd ebx+68  
0010:EEC21DCC 7C90EAEC 0000001B 00000246 0013FCD0 ìê |....F.......  
0010:EEC21DDC 00000023 00000000 00000000 00000000 #...............  
正如你所看到的，EIP 被 KiUserExceptionDispatcher 的地址覆盖了以及堆栈中保存的 CS，Eflags，esp 和 SS。因为我们要做的是 hook 这些指令，所以他会指向 ntdll.dll 中其他的代码，就是我们使用 yates 展示的方法所存储的那些代码。 同样，也有更好的方法应该尽量不要扫描磁盘上的 ntdll.dll，而是使用内存中已经载入的文件直接重新引导至 UserSharedData，在用户模式下被设置成了只读：  
kd> ? SharedUserData  
Evaluate expression: 2147352576 = 7ffe0000  
kd>  
但是在 ring0 它被映射到了：  
#define KI\_USER\_SHARED\_DATA 0xffdf0000  
  
Windows NT 的 IDT 中所有的异常处理程序地址都在 0x80000000 之上。0x80000000 之上的地址被 Windows NT 保留用于特权级（Ring 0）访问。尽管从图上看可能不明显，但是确实几乎所有的异常处理程序地址都在 NTOSKRNL.EXE 中，它是 Windows NT 中运行于 Ring 0 的核心组件。由于我事先已经从 NTOSKRNL 的 DBG 文件中加载了调试符号，所以 SoftICE 查找异常处理程序地址并且找到了大部分异常处理程序的名称。前 0x20 个异常被一系列名字为\_KiTrap00，\_KiTrap01 等的例程处理。“Ki”代表内核中断（Kernel Interrupt）  
还有一个应该注意的是 IDT 中的描述符特权级（Descriptor Privilege Level，DPL）域。它指定了允许调用特定软件中断的最低特权级。例如，INT 2EH 可以被从 Ring 3（最低特权级）到 Ring 0（最高特权级）中任何一级调用。同样，用于断点的 INT 3H，也可以被 Ring 3 及更高特权级的代码调用。  
  
从 0x2A 到 0x2E 的异常被 NTOSKRNL.EXE 中的其它例程处理。例如，在我 1996 年八月的文章 “Poking Around Under the Hood: A Programmer’s View of Windows NT 4.0”，我讲到了 Ring 3 级的应用程序代码传递控制权到 Ring 0 级的系统代码以完成诸如创建一个新进程之类的特殊操作的机制，那就是调用 INT 2E。 INT 2E 被系统 DLL，例如 NTDLL.DLL、USER32.DLL 和 GDI32.DLL 从 Ring 3 调用。看一下 IDT 的 0x2E 这一项，你会看到它的地址指向 NTOSKRNL 中的\_KiSystemService 函数。正是这个函数把控制权转到了相应的代码  
  
很多很复杂的东西,我们知道 win32 异常处理机制其实是很复杂,我们来看看和我们相关的  
如果我们 hook system exception handler，所有你在代码中设的 INT3 中断，都能拦下，在 NT 下，这个 handler 就是 NT.DLL 中 的 KiUserExceptionDispatcher  
  
    \_\_asm   int 3   // 普通的断点指令 这个就是 EXCEPTION\_BREAKPOINT  
    \_\_asm   int 1        EXCEPTION\_SINGLE\_STEP  
他们对应的就是的 1 号和 3 号异常处理,  
设置 int3 断点,当运行到这里  
系统异常触发,系统自动调用处理函数 KiTrap03,  
下发 KiUserExceptionDispatcher,然后枚举下方的异常处理链表  
应该是程序自己的异常处理过程,od 就在这里进行异常处理  
但是 softice 不同,是直接修改系统异常处理函数完成的  
壳经常玩的把戏之一 SEH 就是这个了,因为 SEH 异常能够改变流程,然而异常处理过程  
比较复杂,我们不能 od 跟踪,只能找准壳设置的函数下断,巧妙利用 SEH 可以极大  
的干扰调试,另外,系统调用 int2e 其实也是这个机制  
关于 xp 下得系统调用，系统调用由 2k 下得 int2e 换乘了 sysenter    
,是为了提高效率，但是你单步跟踪一下，就会发现其实他还是会转到 int2e 得入口地址去处理。  
不知谁说得 xp 就不用 int2e 了，实际 sysenter 之后再执行几句特殊得语句就转到 int2e 得地址  
入口了，完全可以用 softice 验证  
  
异常机制偶不打算继续深入下去了,如果 idt 被替换了  
我们需要使用 1 个反汇编引擎来搜索代码,寻找 jmp 掉邪恶代码  
当然更换系统异常处理分支 KiTrap03 也行,通用 unhook 手段很多  
只要本质上也是 unhook,那么就和 SSDT ,shodow sSDT 的 unhook  
没有多少本质区别  
最后祝各位新年快乐:rose :rose 

向大家拜年  
祝新年快乐![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
  
.还有.向大家道歉..  
偶的一些文章存在问题,按照国际惯例是不能给予精华的![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
kanxue 老大请取消精华  
其实每个文章都应该配有自己写的实现代码,之所以没贴出来是偶的  
代码是硬编码的![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
就是这样  
  
WPOFF();  
mov eax,0x11111111  
mov \[eax\],0x22222222  
WPON();  
  
也就是硬修改,注意上面代码有以下问题  
1.没有关中断 sti cti  
2.没有升 dpc-dispatch\_level  
3.没有 lockpages  
4.没有 spinlock  
以上不是每个都必须,但是某些必须.之所以偶不这么写  
原因很简单,为了更快的测试偶的想法有无严重问题  
但是这种代码贴出来是肯定不行的,换了机器肯定没法运行  
如果发生中断,线程切换,页被换出的任意一个那么就直接 BSOD 了  
向所有鼓励和支持偶的大牛们表示感谢,更向那些批评和指点偶的大牛表示感谢  
向被偶搞 BSOD 的大牛表示道歉  
祝福各位新年快乐  
  
那个 int3 代码,偶还原的时候是直接硬性写回去的,  
一时找不到怎么动态获取 int3 处理函数真实地址的办法,只好先写一部分,看看有无大牛批评指点  
实话承认,这些东西比较偏,所以只好写 1 部分看看有无感兴趣的大牛哦![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")   
  
还有,那个某人提到的主场问题.偶不好怎么说拉.![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")   
米办法,一些东西有点偏,所以只好各大论坛发 1 下,看看有大牛无意中能看见不.  
要是发到某地方,既米人看,也米人回,偶文章就白写了![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")   
  
偶想实现的东西是向系统中加入一个自己的异常处理  
实现 myint3 和 myint1,就是 mykitrap3 和 mykitrap1  
系统发生 int3 和 int1 的时候直接进入 mykitrap3 和 mykitrap1  
完全不经过那些被 hook 的函数,实现完整异常处理流程模拟![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")   
  
这也是偶的喜欢做法之一,对于那些邪恶的 hook,完全模拟被 hook 的过程  
从而本质上无视 hook,干净利落做掉 hook![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
  
新的一年到来了,老的保护技术应该把他留在上一年把![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
新的一年应该是 VM 的天下了,光靠 1 个邪恶驱动就敢横行霸道的时代也留在上一年把![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑") ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")   
本来想在年前完成所有代码,看来是不行的了.![](img/32b12ba0f4c0e6a8843aaa06ba62fbdd.jpg "皱眉")   
  
年后偶希望分析 1 下异常的本质过程,偶的猜测是异常是平权的  
\--------------------------------------------------------------------------+  
number  | Exception             | Exception Handler               |  
\--------------------------------------------------------------------------+  
0       | Divide Error            | divide\_error()                  |  
1       | Debug                | debug()                         |  
2       | Nonmaskable Interrupt      | nmi()                           |  
3       | Break Point            | int3()                  |  
4       | Overflow            | overflow()                      |  
上面只是一小部分  
那些反断点的 anti 一定是接管了 int3()的某些异常流程  
我们是否可以改造 od?让常规断点改用 Divide Error ?  
既然断点本质上是发生异常,程序中断下来  
我们可否把要设置断点的地方改成  
xor,eax,eax  
div eax  
这样触发的是除 0 异常,一样能够在这里中断下来,进入我们的处理过程  
但是不会落入壳的 int3 陷阱,这样的好处是很简单,不需要写驱动  
也不会改系统内核,却绕开了陷阱,不知是否可行  
希望大牛予以指点.谢谢 

这里是炉子大牛提供的一份完善的通用代码  
  
偶自己的代码太烂.不贴了![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || IDTGuard.rar  
 |