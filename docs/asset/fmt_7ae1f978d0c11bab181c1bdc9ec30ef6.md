
# `call _EP_prolog` 是什么意思呢？

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-824.htm`](https://www.pediy.com/kssd/pediy08/pediy8-824.htm)

请问 `call _EP_prolog` 是什么意思呢？是在安装异常处理吗？

谢谢！

`VC` 编译出来的程序有 `bp-based frame`、`non bp-based frame` 之分，就是说函数入口处是否有显式的"push `ebp`/mov `ebp`, `esp`"或者 `enter` 指令存在。

如果两个 `push` 之后是"`call __SEH_prolog`"指令，就属于 `non bp-based frame` 情形。

据我逆向经验，`XP SP1/ntdll` 中涉及 `SEH/__try` 块的基本都是 `non bp-based frame` 情形，此时 `stack` 布局如下:

--------------------------------------------------------------------------

内存高址方向

`[EBP+0x004]` `RetAddr`

`[EBP-0x000]` `_ebp`

`[EBP-0x004]` `trylevel`

`[EBP-0x008]` `scopetable`

`[EBP-0x00C]` `handler`

`[EBP-0x010]` `prev`

`[EBP-0x014]` `PEXCEPTION_POINTERS/GetExceptionInformation()`

`[EBP-0x018]` Hold the final `ESP` after all the prologue code has executed

`[EBP-0x01C]` 第 001 个局部变量

内存低址方向

--------------------------------------------------------------------------

`hume` 指出 `2000/ntdll` 是 `bp-based frame` 情形，此时第 001 个局部变量是`[EBP-0x004]`。

====================================

以上内容载自 `http://www.opencjk.org/~scz/windows/200311231550.txt`

我们可以逆向一下`_SEH_PROLOG` 函数：

```
lkd> uf _SEH_PROLOG

ntdll!_SEH_prolog:

7c92edc2 6818ee927c       push    0x7c92ee18  
7c92edc7 64a100000000     mov     eax,fs:[00000000]  
7c92edcd 50               push    eax  
7c92edce 8b442410         mov     eax,[esp+0x10]  
7c92edd2 896c2410         mov     [esp+0x10],ebp  
7c92edd6 8d6c2410         lea     ebp,[esp+0x10]  
7c92edda 2be0             sub     esp,eax  
7c92eddc 53               push    ebx  
7c92eddd 56               push    esi  
7c92edde 57               push    edi  
7c92eddf 8b45f8           mov     eax,[ebp-0x8]  
7c92ede2 8965e8           mov     [ebp-0x18],esp  
7c92ede5 50               push    eax  
7c92ede6 8b45fc           mov     eax,[ebp-0x4]  
7c92ede9 c745fcffffffff   mov     dword ptr [ebp-0x4],0xffffffff  
7c92edf0 8945f8           mov     [ebp-0x8],eax  
7c92edf3 8d45f0           lea     eax,[ebp-0x10]  
7c92edf6 64a300000000     mov     fs:[00000000],eax  
7c92edfc c3               ret
```

而 `0x7c92ee18` 地址为 `ntdll!_except_handler3` 函数

```
lkd> u 0x7c92ee18

ntdll!_except_handler3:

7c92ee18 55               push    ebp  
7c92ee19 8bec             mov     ebp,esp  
7c92ee1b 83ec08           sub     esp,0x8  
。。。。。。。。。。。。。
```
