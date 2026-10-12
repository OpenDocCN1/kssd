
# 对 Hook 内核 ntoskrnl'sZwQuerySystemInformation 隐藏任务管理器进程名 的彻底完善

> 原文：[`www.pediy.com/kssd/pediy11/106922.html`](https://www.pediy.com/kssd/pediy11/106922.html)

前段时间总结了枚举系统进程的 N 种方法，属于科普类读物。链接如下：

[`bbs.pediy.com/showthread.php?p=758997#post758997`](http://bbs.pediy.com/showthread.php?p=758997#post758997)

这几天想针对上面的枚举方法，隐藏进程。首先实现的就是 `SSDT` `Hook` `ZwQuerySystemInfor` 实现进程隐藏，这已经是一项很老的技术了。

`qiweixue` 在他的文章里面写的很清楚:

`http://bbs.pediy.com/showthread.php?t=36742&highlight=%E9%9A%90%E8%97%8F+%E8%97%8F%E8%BF%9B+%E8%BF%9B%E7%A8%8B`

这篇文章有点小错误，在 `sislcb` 的【原创】对 `Hook` 内核 `ntoskrnl'sZwQuerySystemInformation` 隐藏任务管理器进程名 的一点完善 文章中进程了完善，链接如下：

[`bbs.pediy.com/showthread.php?t=56830`](http://bbs.pediy.com/showthread.php?t=56830)

但我在实现的时候，发现 `sislcb` 的这篇文章也存在一下小失误。主要是：隐藏进程在信息开头时，未处理好。他的处理代码如下：

```
if(prev) 
    { 
      //要删除的信息在中间,则把指针指向下一个节点
      if(curr->NextEntryDelta) 
        prev->NextEntryDelta += curr->NextEntryDelta; 
      else 
        prev->NextEntryDelta = 0; //要删除的信息在末尾，则直接把指针指向 0
    } 
    else 
    { 
      if(curr->NextEntryDelta)      //这个地方处理错误
        (char *)SystemInformation += curr->NextEntryDelta; //要删除的信息在开头
      else 
        SystemInformation = NULL; 
    } 
```

虽然不是什么大错误，也不影响作者想隐藏自己的进程，因为一般信息开头是 `Idle` 进程。但为了让程序更加健全，小弟将该处的错误以及更正方式，贴出来。以便大家对这个技术有个全面的掌握。

错误的原因是，`SystemInformation` 是个指针变量，在内部改变它的值并不影响外部，我们在函数内部只能改变它指向的内容。呵呵，很基本的 C 知识吧。但却常常被人忽略。所以作者上面的改变方式，对 `SystemInformation` 值没任何影响。具体方法应该是：

```
ULONG len = sizeof( SystemProcessInformation );
ULONG uOffet = ((PSystemProcessInformation)SystemInformation)->NextEntryDelta;
pCur = (PSystemProcessInformation)((ULONG)pCur + pCur->NextEntryDelta);
memcpy( SystemInformation, pCur, len );
((PSystemProcessInformation)SystemInformation)->NextEntryDelta += uOffet;
```

我实现的代码见附件 `HookZwQuerySystemInformation.rar`
