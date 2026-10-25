
# 利用 veh 完成内存补丁功能，以躲避运行时内存检测

> 原文：[`www.pediy.com/kssd/pediy12/143432.html`](https://www.pediy.com/kssd/pediy12/143432.html)

这事缘起于 `dota`，前几天和同学在某 `vx` 平台 `dota`，被一开挂黑店虐得不行。一气之下也找来个 `war3` 的挂，想报仇雪恨。结果游戏刚启动就被平台踢出来了，看来偷懒是不行的了，网上 `google` 了一番，该平台的反外挂主要基于黑名单和运行时内存检测。

黑名单就不说了，运行时内存检测是比较致命的，因为开图必然要改程序流程，改流程免不了要打内存补丁，打了补丁又不想被发现也是有办法的，比如 `shadowwalk`，这是刚学到的新名词，详见 `http://bbs.pediy.com/showthread.php?t=58896`

该方法简述就是模块重载，让读取和执行分离开来，不过要做的工作实在太多，而且不稳定，总之我真心认为硬碰硬不是好办法，也不是长久之计。其实简单的改改流程不被发现可以用异常处理来做，很安全稳定。

先复习一下异常处理流程，在没有调试器的情况下，用户层异常的分发由 `KiUserExceptionDispatcher` 完成，其中再调用 `RtlDispatchException` 分发异常，`RtlDispatchException` 中首先 `RtlCallVectoredExceptionHandlers` 执行 `veh`，如果此异常被处理则不再继续分发，未处理则展开 `seh` 链依次 `RtlExecuteHandlerForException` 执行 `seh`，同样的，如果被处理则不再分发，否则继续，如果都没有处理则进程崩溃。这么看来只要抢在游戏本身 `seh` 之前捕获并处理异常游戏肯定是不会发现的。

简单说下怎么做，`AddVectoredExceptionHandler` 给进程装上一个向量异常处理 `handler`，然后在将 `drx` 寄存器设置为要打补丁的位置，也就是在补丁位置下一个硬件执行断点，执行到此处时抛出单步异常，异常处理捕获异常，在 `handler` 里模拟补丁功能。异常处理函数被调用时会传进来线程 `context` 指针，所以可以干的事情很多很多，这里可以发挥一下想象力。

就拿 `war3` 来举例吧，游戏就不用调了，网上找个开图工具，开图进游戏，扫扫进程钩子可以发现游戏被 `patch` 了两个地方

代码:

```
0x6f361f7c  0x01 - 0x00
0x6f3a20dd  0x75 - 0xeb
```

两个补丁都打在 `game.dll` 里，而 `game.dll` 加载位置大体是固定的。其中 `0x6f361f7c` 将 `01` 改为 `00`，实际上是将 `0x6f361f7b` 处的 `mov eax,1` 改为了 `mov eax,0,` 而 `0x6f3a20dd` 处的 `75` 改为 `eb` 是无视标志位强制跳转，搞清楚了这个可以写代码了

代码:

```
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
           )
{
  switch (ul_reason_for_call)
  {
  case DLL_PROCESS_ATTACH:
    {
      AddVectoredExceptionHandler(1, (PVECTORED_EXCEPTION_HANDLER)ExceptionHandler);
      SetHwBreakpoint();
    }
  case DLL_THREAD_ATTACH:
    {
      SetHwBreakpoint();
    }
  case DLL_THREAD_DETACH:
  case DLL_PROCESS_DETACH:
    break;
  }
  return TRUE;
}

void SetHwBreakpoint()
{
  CONTEXT ctx;
  ctx.ContextFlags = CONTEXT_ALL;
  GetThreadContext(GetCurrentThread(), &ctx);
  ctx.Dr0 = 0x6f3a20dd;
  ctx.Dr1 = 0x6f361f7b;
  ctx.Dr7 = 0x405;
  SetThreadContext(GetCurrentThread(), &ctx);
}

DWORD NTAPI ExceptionHandler(EXCEPTION_POINTERS * ExceptionInfo)
{
  if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == 0x6f3a20dd)
  {
    //直接改 eip 模拟 jmp
    ExceptionInfo->ContextRecord->Eip += 0x34;
    return EXCEPTION_CONTINUE_EXECUTION;
  }
  else if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == 0x6f361f7b)
  {
    //直接设 eax 为零模拟 mov eax,0
    ExceptionInfo->ContextRecord->Eax = 0;
    ExceptionInfo->ContextRecord->Eip += 5;
    return EXCEPTION_CONTINUE_EXECUTION;
  }
  else
  {
    //在异常 handler 里重设 drx 防止断点被意外清除
    ExceptionInfo->ContextRecord->Dr0 = 0x6f3a20dd;
    ExceptionInfo->ContextRecord->Dr1 = 0x6f361f7b;
    ExceptionInfo->ContextRecord->Dr7 = 0x405;
    return EXCEPTION_CONTINUE_SEARCH;
  }
}
```

让游戏启动时候加载这个 `dll` 就生效了，这样不费一兵一卒，不动一草一木，便完成了需要打补丁才能完成的工作，对于进程本身而言只是多了一个 `dll` 和 `drx` 寄存器被使用，别的什么都没动。唯一的缺陷是 `drx` 寄存器只有 `4` 个，也就是用这方法理论上只能最多支持 `4` 个补丁位置，其实还有其他方法，搞清楚了程序流程断点可以来回设置，还是很强大的。

上述代码只是演示性质的，所以地址硬编码了，而且仅适用于 `1.24e`，别的版本没试，本来也不是真心想作弊，就是试试新方法而已，有兴趣的自己去搞吧

要注意的就是异常 `handler` 里不能有任何可能产生异常风险的代码，否则死循环了

附件是一个改了导入表的 `storm.dll` 和异常处理 `dll`，将这两个放游戏目录下就可以玩了

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `veh.rar`  
|
