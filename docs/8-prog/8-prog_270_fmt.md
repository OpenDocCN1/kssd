
# 老瓶装新酒：0xCC+SEH 实现 HOOK 任意地址指令！

> 原文：[`www.pediy.com/kssd/pediy11/111360.html`](https://www.pediy.com/kssd/pediy11/111360.html)

看了“hook 任意函数，任意位置 【共享资源】 soucre ”一文，还以为实现了我想要的“HOOK 任意地址指令”功能，遗憾的是还是和以前的 HOOK 代码类似，都有局限性，5 个字节的跳转要求使我们总不能想 HOOK 啥地址就 HOOK 啥地址，要选一些没有相对转移的位置来实施 HOOK 大法。一个字节 `0xCC` 的软中断是我们理想的选择，不用考虑什么跳转不跳转，直接中断下来干点什么坏事才转回去正是我们经常用 `OD` 干的事情，为什么不能用它来 HOOK 呢，哈哈！搜了下，已经有前辈写过类似的利用 `SEH` 实施 `API` hook 的文章，我就不重复了，我写的不是 HOOK API 而是 HOOK 任意地址，比如给游戏开个远程线程，给它的主线程安装个“HOOK 任意地址”的 `SEH`，干点什么想干的事，那是多么惬意的事情啊，哈哈！废话少说，高手请飘过，测试代码如下：

```
EXCEPTION_DISPOSITION _cdecl cc_handler
(struct _EXCEPTION_RECORD *ExceptionRecord,
        void * EstablisherFrame,
        struct _CONTEXT *ContextRecord,
        void * DispatcherContext )
{
  if (ExceptionRecord->ExceptionCode==STATUS_BREAKPOINT)
  {
    AfxMessageBox("不用调试器，自己处理断点！");
    //ContextRecord 里想改啥改啥，EAX,EBX,ECX 啥的，甚至 EIP 都可以改，嘿嘿！
    *(LPBYTE)(ContextRecord->Eip)=0x90;//0xCC 处改成 NOP，不然就一直在这里面打转了！
    return ExceptionContinueExecution;   
  } 
  else
    return ExceptionContinueSearch;
}

void CHookAnyAddrDlg::OnButton1() 
{
  // TODO: Add your control notification handler code here
    AfxMessageBox("start");
    __asm
    { 
        // 创建 EXCEPTION_REGISTRATION 结构：
        push offset cc_handler// handler 函数的地址
        push DWORD PTR FS:[0]  // 前一个 handler 函数的地址
        mov FS:[0],ESP // 安装新的 EXECEPTION_REGISTRATION 结构
        __emit 0xcc//触发 INT3 异常
        // 移去我们的 EXECEPTION_REGISTRATION 结构
        pop DWORD PTR FS:[0]
    add esp, 4 // 将我们的 EXECEPTION_REGISTRATION 弹出堆栈
    } 
  AfxMessageBox("end");   
}
```

是不是很简单，简单得不可思议，能简单实现的事情为什么要弄复杂呢，哈哈！

开一个远程线程实现下任意地址写 `0xcc`，把 `SEH` 处理程序弄到全局范围内，一个远程“HOOK 任意地址”就实现了。

## 上传的附件

| `![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)` | `Hook Any Addr.rar` |
| :--- | :--- |
