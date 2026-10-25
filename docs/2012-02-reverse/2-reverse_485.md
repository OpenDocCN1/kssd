
# SMC 破解 DLL 时基址的问题

> 原文：[`www.pediy.com/kssd/pediy10/86583.html`](https://www.pediy.com/kssd/pediy10/86583.html)

我在用 SMC 破解一个 DLL 的时候再 OD 里破解成功了，可是保存后在 OD 也可以运行，但脱离 OD 后我运行他时基址变了，内存地址当然也跟着变了，所以又打不开了，有啥解决办法吗，大家告诉下。

```
call $+5
pop ebp
sub xxxxxx
```

我没有脱 DLL 得壳，用的 SMC，不用重定位，我只想知道如何在 OD 中用指令获取基址的方法

我的 SMC 代码，谁能告诉下怎么改？

再 OD 中基址是 `910000`，用这代码可以破解。而在不同的机子上基址也不同，所以内存地址也就跟着变了怎么办呢？

```
mov     dword ptr [B091EF], FFEA3ED0
jmp     009AD268
cmp     dword ptr [AFEA4E], 4A388647
je      short 009AD0D4
jmp     00B0607A
mov     dword ptr [AFEA4E], 4EAFBF11
jmp     00B0607A
nop
```

以第一条指令 `mov dword ptr [B091EF], FFEA3ED0` 为例:

改成这样:

```
push edx //保护一下
call $+5 //机器码为 E8 00 就是 call 它下面的一条指令
pop edx //把刚 push 进去的 eip 弹出至 edx. edx 存放该指令地址.
sub edx, [你调试时上条指令的地址] //得到差值.
mov dword ptr [B091EF+edx], FFEA3ED0 //加差值即为重定位后值.
pop edx //恢复现场.
```
