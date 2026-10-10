

# 腾讯 2010 竞赛第三阶段题目思路+代码

> 原文：[`www.pediy.com/kssd/pediy11/125264.html`](https://www.pediy.com/kssd/pediy11/125264.html)

本文使用注入 DLL 设置硬件断点以及模拟执行指令的方式来记录 API 调用。流程如下：  
  
1，将 EXE 自身复制一份，修改入口点和 DLL 标志，变成 xixihaha.dll。  
2，将 xixihaha.dll 注入目标进程。  
3，DLL 会挂钩以下 5 个函数：  

代码:

```
VOID HookLdrLoadDll();  //分析新载入模块的导出表
VOID HookCsrNewThread();  //对新创建的线程下硬件断点
VOID HookKiUserExceptionDispatcher(VOID); //处理异常
VOID HookZwGetContextThread(); //防止硬件断点检测 1
VOID HookZwSetContextThread(); //防止清除硬件断点

```

4，对所有模块的导出表进行分析，将所有信息记录到全局数组，并对第一个要监视的 API 下硬件断点。  
5，硬件断点断下来后，分析 API 的返回地址，如果是 EXE 调用该 API，说明要监视的第一个 API 被调用，开始模拟执行 EXE 的指令。  
6，如果不是 EXE 调用，则对返回地址下断，执行到返回后，再对第一个 API 下断。  
7，模拟执行指令过程中，如果 EIP 不在 EXE 内，则分析此地址对应的 API 名称，获取第二个 API。  
  
PS：  
1，模拟执行指令没有加入大循环代码块的分析，否则，速度应该会再快一点  
2，通过 Hook KiUserExceptionDispatcher 来处理异常，而不是通过调试器和设置 ExceptionHandler，可以避免目标程序的大部分检测。首先分析异常是否是我们要处理的类型，如果是，直接处理然后返回（不再调用原始的异常处理），如果不是，首先将 DRX 寄存器清掉，然后再调用原始的异常处理，调用完毕后，再将 DRX 寄存器内容恢复。  
  
exile.rar