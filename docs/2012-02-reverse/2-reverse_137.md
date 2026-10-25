
# od 中，如何确定跳转来自何处？

> 原文：[`www.pediy.com/kssd/pediy10/59965.html`](https://www.pediy.com/kssd/pediy10/59965.html)

用 `od` 调试程序时经常会遇到这样的情况，程序中断在一个函数的入口点

`od` 会把所有调用这个函数的地址列出来，但有时候调用这个函数的地方太多了

我想问的是：如何精确的确定本次执行的到底是哪个地址调用了这个函数？我现在是 `run` 跟踪确定，或者一个一个的试，但感觉都不是很方便，高手有更好的方法吗？指点一下啊，非常感谢

有个 `ring3` 的插件 `Conditional Branch Logger`, 我更喜欢 `ring0` 的 `DebugCtlMSR`, `openrce` 上有讨论.

## Hook int1

代码:

```
__declspec(naked) void NewInt01()
{
  //  - Interrupt 1 Handler -
  //
  //  offset   | contains
  //  ---------+-----------------------------
  //  esp     : EIP Context
  //  esp + 4 : CS  Context
  //  esp + 8 : EFLAGS Context

  __asm
  {
      pushad

      mov    eax, dr6
      bt    eax, 0Eh  //单步?
      jnc    __oldint01

      mov    ecx, 1D9h
      rdmsr
      or    eax, 3
      wrmsr
      
    
    __oldint01:  
      popad
      jmp    OldHandler01
  }
}
```

在 `OD` 中 `trace` 时记录的只有控制转移指令
