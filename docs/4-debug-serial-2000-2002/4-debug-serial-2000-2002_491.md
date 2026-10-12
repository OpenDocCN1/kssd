
# SyGate 4.0 build712

> 原文：[`www.pediy.com/kssd/pediy03/forum925.htm`](https://www.pediy.com/kssd/pediy03/forum925.htm)

`w32dsm` 反编译 `sygate.exe`，寻找"trial version expired"共找到 5 处。跟踪可知第二处为过期时弹出窗口的程序段。

```
\* Referenced by a CALL at Addresses:

|:004270DB  , :00428B02

:0041FD40 64A100000000             mov eax, dword ptr fs:[00000000]
:0041FD46 6AFF                     push FFFFFFFF
:0041FD48 68D1B54600              push 0046B5D1
:0041FD4D 50                       push eax
:0041FD4E A13CC94800              mov eax, dword ptr [0048C93C]
:0041FD53 64892500000000          mov dword ptr fs:[00000000], esp
:0041FD5A 81ECDC000000            sub esp, 000000DC
:0041FD60 8B8063010000            mov eax, dword ptr [eax+00000163]
:0041FD66 56                       push esi
:0041FD67 83E803                  sub eax, 00000003
:0041FD6A 8BF1                    mov esi, ecx
:0041FD6C 0F84FB010000            je 0041FF6D -------->  **关键，跳则活，不跳就死翘翘。**
:0041FD72 48                       dec eax
:0041FD73 0F84D7010000            je 0041FF50
:0041FD79 A108DF4800              mov eax, dword ptr [0048DF08]
:0041FD7E 85C0                    test eax, eax
:0041FD80 746E                    je 0041FDF0
:0041FD82 8B4E1C                  mov ecx, dword ptr [esi+1C]
:0041FD85 6A40                    push 00000040
\* Possible Reference to String Resource ID=02006: "This trial version has expired. Please visit www.sygate.com "
:0041FD87 68D6070000              push 000007D6
\* Possible Reference to String Resource ID=02005: "Trial Version Expired"
:0041FD8C 68D5070000              push 000007D5
:0041FD91 51                       push ecx
```

先改日期，使 `sygate` 过期。在 `trw` 中设断点 `bpx 41FD40`，点 `load` 运行，`f5` 运行 `sygate`，`trw` 栏下，在 `41fd67` 处吧 `al` 至为 `3`，

`f5` 运行程序发现程序不在过期。

那么，怎么让 `SyGate` 4.0 build712 自己告诉你注册码呢？只要按下面的步骤就可搞定：

1. 让 `sygate` 过期，运行时弹出过期窗口让你注册，不管它，直接 `cancel` 掉。
2. 用上面的改法运行一遍，在 `help-register license...` 中看到了什么？

以后在运行 `sygate` 也不会过期了，因为那时真的注册码呀。这个注册码可能是 `sygate` 的万能注册码，因为我在 2 台不同配置的机器下以不同的用户名都得到了相同的注册码，不过也可能是根据 `windows` 的 `cdkey` 生成的因为 2 个机器的 `win98` 都用 1 张盘装的，所以没法验证了。不过为什么会这样，我想大概是 `sygate` 的程序员偷懒造成的吧：）

附：

`serial number:520D7FA0-9AD4E63`

`registration code:2E549118`

首次运行需输 `trial key:`

`trial key:H1(或 2)XXXXXX(6 位任意数或字母)`

不止为什么，`sygate` 在我的机器上始终无法实现共享上网（未破前也一样），在别人的机器上可以，`lan` 没问题，

望高手指教。

另，拜托高手帮忙搞定 `mmtools(multimedia tools 一个 delphi VCL 控件)` 1.7 版（）

老爸还急着要用呢。多谢！

`4dd466d0e1e0cc123afbe504da2462ca`
