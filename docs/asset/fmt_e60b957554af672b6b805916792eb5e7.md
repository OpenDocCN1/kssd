
# `ObRegisterCallbacks` 返回 `0xC0000022` (拒绝访问) 解决方案

> 原文：[`www.pediy.com/kssd/pediy12/140891.html`](https://www.pediy.com/kssd/pediy12/140891.html)

最近写驱动，发现 `ObRegisterCallbacks` 这个函数总是调用不成功。在网上查了一下，发现使用这个函数对驱动有特殊要求，必须使用特殊的签名才行！否则这个函数会返回 `0xC0000022` (拒绝访问)。逆向此函数可以看到以下是有关是否加签名的判断：

```
PAGE:00000001404AA75D                 call    MmVerifyCallbackFunction  
PAGE:00000001404AA762                 cmp     eax, ebx  
PAGE:00000001404AA764                 jz      short loc_1404AA7E1  
PAGE:00000001404AA766  
PAGE:00000001404AA766 loc_1404AA766:                  ; CODE XREF: ObRegisterCallbacks+11Bj  
PAGE:00000001404AA766                 mov     rcx, [rsi+rbp+18h]  
PAGE:00000001404AA76B                 cmp     rcx, rbx  
PAGE:00000001404AA76E                 jz      short loc_1404AA779  
PAGE:00000001404AA770                 call    MmVerifyCallbackFunction  
PAGE:00000001404AA775                 cmp     eax, ebx  
PAGE:00000001404AA777                 jz      short loc_1404AA7E1  
```

开发调试时可以将

```
PAGE:00000001404AA764                 jz      short loc_1404AA7E1  
PAGE:00000001404AA76E                 jz      short loc_1404AA779  
```

这两行改为 `nop`，暂时使 `ObRegisterCallbacks` 调用成功，方便开发调试和暂时没签名的朋友。

`ObRegisterCallbacks` 这个函数是在 Vista sp1 之后的版本中添加的，可用来监控系统中对进程/线程句柄的操作，如打开进程、复制线程句柄等。

每次测试驱动都要 `patch`。。
