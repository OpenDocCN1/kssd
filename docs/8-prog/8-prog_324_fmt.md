
# 发一个自己平时用的简单 inlinehook 的类

> 原文：[`www.pediy.com/kssd/pediy12/133341.html`](https://www.pediy.com/kssd/pediy12/133341.html)

到看雪很久了，基本上每天都会泡一会儿论坛，可是却没做过什么贡献，也没什么露得出手的本事，今天想想把自己平时用的一个 `inlinehook` 的类扣出来给大家用用。

虽然代码写的很差劲很挫都是小改动，不过还好使。

因为我平时需要用的都只是截取一下，也不用修改程序的流程，就对修改程序流程的支持不大好。不过 `etc.cpp` 里有复制 `orgOpcode` 出来的代码，应该自己很好改成需要的样子的。

支持多线程，支持重入(只是简单的在重入的时候从 `hook stub` 中就跳过 `hookrouter` 函数了)。

要 Hook 了就:

`inlinehook * Global_Inlinehookptr = new inlinehook (addr, hookTargatLen, jmpOrCall, HookRouter);`

不 Hook 了 `delete Global_Inlinehookptr` 就好了。

主要的定义:

```
#define INLINE_5BYTESLAGGERCALL 0x1001

#define INLINE_5BYTESLAGGERJMP 0x1002

#define INLINE_UNPROTECTEVINMENT 0x1003

#define X86STRACKBUFFERCHANGE 0x7798FFAA

/*

HOOK ROUTER 中修改了返回地址时候必须返回这个！修改了寄存器的值时候不用返回这个，修改堆栈的值也不用。但你自己设置了返回地址，就要自己备份那些被 HOOK 掉的指令了。可以从 InlineSingleHook::RedirectedOpcodeInStub 获得 stub 中返回位置的代码。

*/

  typedef int  (__stdcall * InlineX86HookRouter) (PInlineX86StackBuffer X86StrackBuffer);

/*

在函数体里，堆栈和寄存器和 HOOK 地方的一模一样。只要不修改低处堆栈(比当返回地址还低)的值，就不会破坏堆栈和寄存器环境，也不会修改那些值。

*/
```

对了，这个是 `VS2005` 环境的工程，但我最早写的时候是在 `VC6` 下面写的，所以把代码复制到 `VC6` 里应该也可以用。

更新了一下 `etc` 里的 `X86ShallToRedirect`，之前的 `etc` 里有 bug，没有处理 `copy` 的原始代码中的跳转目标在被 `copy` 的范围内的情况，会导致跳转到 `NOP` 代码处。因为发出来了，就想测试下效果，`HOOK` 了函数开头 `100` 长度的代码时候出错了，检查后才发现这种情况。

又更新了下，碰到 `je/jne` 这种 `far` 跳转指令时，之前的会有 bugs。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `inlinehook 0.1.rar` |
|---|---|---|
