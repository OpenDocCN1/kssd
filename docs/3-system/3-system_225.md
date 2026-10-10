

# windows XP 下的向量化异常处理

> 原文：[`www.pediy.com/kssd/pediy05/pediy50017.htm`](https://www.pediy.com/kssd/pediy05/pediy50017.htm)

windows XP 下的向量化异常处理(Vectored Exception Handling)   
                                  原作:    Matt Pietrek   
                                  翻译改写:  hume/冷雨飘心   
  
首先回顾一下(SEH)结构化异常处理,结构化异常处理用 EXCEPTION\_RETGISTRATION 结构链起来的异常处理系统,那么当异常发生时,系统会遍历这个链,首先是链最前面的,系统会问:这个异常你处理吗?如果回答 YES(通过返回 EXCEPTION\_CONTINUE\_EXECUTION)的话,那系统就把控制权交给他,然后由其处理这个异常,返回到异常处理程序想返回的任何地方.如果通过返回 EXCEPTION\_CONTINUE\_SEARCH 回答:NO,let others do that! 系统就继续查找这个链,不厌其烦地问同样的问题和采取相同的处理原则.这个链最前面的 EXCEPTION\_RETGISTRATION 是由 fs:\[0\]处的一个 dword 指针指向的.具体细节还请参阅相关资料或我以前的<<seh In Asm 研究>>.   
  
让我们来看一下 seh 的缺点,就是最后安装的 seh 处理例程总是优先得到控制权,这有时并不是最好的解决方案,但确实是 seh 的工作机制,当然 Final 型的或称 top 型的(还记得吗,也就是通过 SetUnHandledExceptionFilter 安装的)例外,因为他是不允许嵌套的.我们提到的是线程相关的也就是 per\_Thread 类型的.为什么不是好的解决方案呢,让我们设想一下,假如你用两周写了一个异常完美的 seh 处理例程,能够完美处理所有异常,并希望异常全部由你来处理,但很不幸,比如你调用了一个外部模块,而这个模块自己安装了一个 ugly 的 seh 处理例程,他的动作是只要有异常发生就简单地终止程序...hmmm...!!!这意味着什么?你的两周工作全部付诸东流!又比如你想在你的加壳程序里面加密目标程序代码段,然后发生无效指令异常的时候用你自己安装的处理句柄来解密代码段继续执行,听起来这的确是一个好主意,但遗憾的是大多数 C/C++代码都用\_try{}\_except{}块来保证其正确运行,而这些异常处理例程是在你壳注册的例程之后安装的,因而也就在链的前面,无效指令一执行,首先是 C/C++编译器本身提供的处理例程或者程序其他的异常处理例程来处理,可能简单结束程序或者....天知道!   
    在 Xp 下,Microsoft 又提供了又一种异常处理,那就是 VEH(Vectored Exception Handling),我译作向量异常处理,这个东东用如下 api 注册,类似于 SEH,也是一个链状结构,让我们来看看他的不同之处,噫,好像差不多啊:   
    WINBASEAPI PVOID WINAPI AddVectoredExceptionHandler(   
    ULONG FirstHandler,   
    PVECTORED\_EXCEPTION\_HANDLER VectoredHandler );   
  
    FirstHandler:是一个标志,可以指定是否将你的 VEH 处理例程放在 VEH 链的最前面!   
    VectoredHandler:这个东东是异常处理例程入口   
    LONG NTAPI VectoredExceptionHandler(PEXCEPTION\_POINTERS);   
    PEXCEPTION\_POINTERS 是指向 EXCEPTION\_POINTERS 的指针,具体 EXCEPTION\_POINTERS 的结构是否和 SEH 的一致?估计是一致的,我装了 Xp,没有下 DDk 呢,现在正在找!   
    正像你看到的,好像和 Final 型 SEH 处理差不多?不一样!区别如下:   
    1)首先是 AddVectoredExceptionHandler 添加的异常处理句柄可以嵌套,而不是只能指定一个   
    2)其次是 AddVectoredExceptionHandler 可以指定你的异常处理句柄是否在链的最前面,hoho,这可是我们期望的!当然如果在你后面有人调用 AddVectoredExceptionHandler 也作同样指定,那对不起,你只得在他后面了.   
  
    相同之处在于他们都是进程而不是线程相关的.   
  
    XP 仍然支持 SEH,那么问题来了 SEH 和 VEH 是什么关系,答案很简单,VEH 优先权高于 SEH,只有所有 VEH 全不处理某个异常的时候,异常处理权才会到达 SEH.只要目标程序中没有利用 VEH,你的 VEH 就是第一个得到控制者.哈哈,现在的采用 SEH 作为异常处理的普通 C/C++程序对你不会再有干扰了!你可以用 VEH 来 hook api 了,god!   
  
    另外一个问题,如果有 debugger 怎么办?控制权转向又如何呢?不幸的消息来了,异常发生后首先通知的还是 debugger,debugger 不处理才返回控制权给 VEH,\[VEH 不处理,返回给 SEH,seh 不处理,又给 debugger 一个机会,如果还不处理,才由系统处理,这是我猜测的,Matt 没说这个问题,嘿嘿\]   
  
    RemoveVectoredExceptionHandler 用来移除 VEH 处理句柄.是否需要看你的了,由于没有下 XP DDK,所以没有实际代码,大家凑合着看,等我找到并下了 DDK 再说,由于要参加专业考试,所以玩耍要暂缓了,以后再写东西翻译东西的机会不多了,要等我 9 月考完再说,本来是好好学习的(刚刚发了誓),这不在网上遛跶看了 Matt 老哥吹的我又坐不住,当一个好消息和大家共享,原文去 MSDN 查查,是 <<New Vectored Exception Handling in Windows XP>>.   
      
    BB,我要开始聊天了:D...)!@#$%  4dd466d0e1e0cc123afbe504da2462ca