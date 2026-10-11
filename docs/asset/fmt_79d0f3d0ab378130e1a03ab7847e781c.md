
# TRW2000 1.11

> 原文：[`www.pediy.com/kssd/pediy01/forum013.htm`](https://www.pediy.com/kssd/pediy01/forum013.htm)

我用 `W32DASM` 反汇编 `TRW2000 1.11.SYS` 未见到如作者 `czs` 所述的

```
* Possible StringData Ref from Code Obj ->"TRW2000 expired!"
                  |
:0003777C 685A770300             push 0003775A

* Possible StringData Ref from Code Obj ->"STOP"
                  |
:00037781 686E770300             push 0003776E

:00037786 E88F0DFFFF             call 0002851A

:0003778B B001                   mov al, 01

:0003778D C3                     ret
```

我的 `W32DASM` 是 8.93 完全英文版，不知道为何？

根据破解 `TRW2000 1.06` 的基础，我在 `TRW.SYS` 中用 `SEARCH D007` 找到:

```
:00037805 817C2404D0070000       cmp dword ptr [esp+04], 000007D0 <--1

:0003780D 7515                   jne 00037824

:0003780F 837C240802             cmp dword ptr [esp+08], 00000002 <--2

:00037814 7413                   je 00037829

:00037816 837C240803             cmp dword ptr [esp+08], 00000003 <--3

:0003781B 740C                   je 00037829

:0003781D 837C240804             cmp dword ptr [esp+08], 00000004 <--4

:00037822 7405                   je 00037829
```

很容易的，

1. 说明 `TRW2000 1.11` 只能用在 2000 年中.
2. 2、3、4 说明 `TRW2000 1.11` 只能在 2、3、4 月中使用

修改

1. `7515` 为 `9090`
2. `837C240802` `7413` 为 `837C240813` `7213`

ok!

sunwk  
e-mail: `sunwk@sohu.com`

HELP：谁能提供注册版的 `TRW2000` 给我，E-MAIL 联系.

* * *

答复帖子：

标 题:`W32DASM` (57 字)  

发信人:`冰毒`  

时 间:`2000-3-3 18:24:20`  

详细信息:

有可能你用的是打过 `VB` 补丁的版本，据说会有 `SDR` 不能显示的毛病. 

`4dd466d0e1e0cc123afbe504da2462ca`
