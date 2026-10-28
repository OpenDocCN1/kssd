
# [原创] x86/x64 处理器实现调试者的自我调试

大家好，我叫 cc3。前几天遇见一个搞笑者，他跟一小白炫技说，x86/x64 处理器根本无法调试`int 3`和`int 1`中断里的代码。

我友情地提示他不要误导小白……几天过去了，突然我发现我成为世人的笑柄，好吧……

大家都知道，系统调试机制主要依赖`int 1`和`int 3`两个中断实现的。很多人之所以认为无法调试——调试代码的原因，是因为调试代码里如果发生调试事件，将会再次陷入调试代码，最后形成死循环。

在这里我强调一点：抱有这种观点的人，在逻辑层面上是错的。

正确的观点是：中断调试代码中发生调试事件，如果无法处理，将会陷入死循环。

有了这一个观点，我们就可以在处理调试事件上做文章。

大家先看下成果：

![](https://bbs.kanxue.com/upload/attach/201811/686541_2SJ77HTRVGW7HW2.png)

![](https://bbs.kanxue.com/upload/attach/201811/686541_Z9EV48HTCN7XVKG.png)

其实代码实现起来很简单。很多人认为做不到，是因为其“逻辑”上出了问题，导致做不到；其实并不是真的做不到。

![](https://bbs.kanxue.com/upload/attach/201811/686541_DKFX72SR6NN3GQM.png)

```c
case DEBUG_EXCEPTION:

EventInject(TRAP_DEBUG,0);

if (qwSgin==0)

{

rsp=AsmVmRead(GUEST_RSP);

rsp-=0x30;

*(QWORD *)(rsp     )=AsmVmRead(GUEST_RIP);

*(QWORD *)(rsp+0x8 )=AsmVmRead(GUEST_CS_SELECTOR);

*(QWORD *)(rsp+0x10)=AsmVmRead(GUEST_RFLAGS);

*(QWORD *)(rsp+0x18)=AsmVmRead(GUEST_RSP);

*(QWORD *)(rsp+0x20)=AsmVmRead(GUEST_SS_SELECTOR);

*(QWORD *)(rsp+0x28)=0x0fff000100000001;

GuestRegs->rsp=rsp;

qwSgin=1;

AsmVmWrite(GUEST_RIP,qwKiBreakpointTrapAddr );

}

break;
```

几行代码就能搞定很多看似不可能的事。

最后送大家一句话，不要让狭义的观点局限了“视界”的宽度。

最后提个醒：不是`HOST`调试`GUEST`，而是`GUEST`在调试自己。`HOST`只不过是控制器。其实也可以实现`cc`断点，引发中断到`HOST`，恢复`cc`断点，传递`int 3`到`GUEST`。`cc`断点相关代码就不贴了，我实现的时候只是简单地进行区域判断，恢复`cc`占用的数据。

在这里，被调试者一直是被调试者。执行到`int 1`或`int 3`这个过程属于击穿过程，击穿了逻辑自我设定的屏障，让被调试者同时拥有了调试者身份。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2018-11-30 22:15

被 dsjflw 编辑

，原因：
