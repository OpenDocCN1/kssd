
Except for the `EXCEPTION_DEBUG_EVENT` case above, we call `ContinueDebugEvent` with `DBG_EXCEPTION_NOT_HANDLED` flag to resume the debuggee.

除了上面讨论的`EXCEPTION_DEBUG_EVENT`情况外，我们用`DBG_EXCEPTION_NOT_HANDLED`标志调用`ContinueDebugEvent`函数来恢复debuggee的执行。

```
invoke CloseHandle,pi.hProcess
invoke CloseHandle,pi.hThread
```

When the debuggee exits, we are out of the debug loop and must close both process and thread handles of the debuggee. Closing the handles doesn't mean we are killing the process/thread. It just means we don't want to use those handles to refer to the process/thread anymore.

当debuggee退出时，我们离开调试循环并且必须关闭debuggee的进程和线程句柄。关闭句柄并不意味着我们杀死进程或是线程。它仅仅意味着我们再也不需要用这些句柄来引用进程和线程了。

___

This article come from Iczelion's asm page

风向改变翻译于2008-3-9

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
