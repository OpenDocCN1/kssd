
# x64 与反调试

> 原文：[`www.pediy.com/kssd/pediy12/126516.html`](https://www.pediy.com/kssd/pediy12/126516.html)

还有个 EXE 文件，原文地址 `http://forum.exetools.com/showthread.php?t=12726`，下载不了，要邀请注册

在逆向中，反调试技巧一直是一个非常有趣的事情，由于对 X64 的硬件和 OS'es 到来，虽然有些东西已经改变。

前些日子，我碰到一个 X64 的软件，总是在某测试系统的检测调试不成功。跳水此事，并在调试器下的所有规避反调试技巧

它工作得很好。外界调试 faillure 理由证明是众所周知的代表 `stos` / `movs` 伎俩。

代码：

Example code

```
t1 equ goodboy-badboy-2

new: db 0EBh,01,81h,0EBh,t1

; cut //
lea rsi,[new]
lea rdi,[here]
mov rdx,[rdi]
mov rcx,3

here:
rep movsw

badboy:
mov r9,30h
lea r8,[DebugStatus]
lea rdx,[DbgFoundText]
xor rcx,rcx
invoke MessageBox      ; badboy!
jmp Exit

goodboy:
mov r9,40h
lea r8,[DebugStatus]
lea rdx,[DbgNotFoundText]
xor rcx,rcx
invoke MessageBox      ; goodboy!

; cut //
```

`rep stos` / `movs` 招式不需要进一步解释，因为大家都知道这一点，因为 16 位系统，然而，在 X64 位下使用被警告

为了测试，我附上一个 EXE 文件测试，单步它 `F7` 键（在 `MessageBox` 的调用 `F8` 键），它会永远检测你

不过我敢肯定，一个很小的比例，拥有最新的 X64 的 CPU 技术，会得到假的检测调试器外！
