
# 在 shellcode 中遇到疑惑的浮点指令

> 原文：[`www.pediy.com/kssd/pediy12/135162.html`](https://www.pediy.com/kssd/pediy12/135162.html)

今天看到一个样本中一段 shellcode 的解码头，从样本中拿出来后用 IDA 反汇编得到如下结果：

```
seg000:00000000                  nop
seg000:00000001                  nop
seg000:00000002                  nop
seg000:00000003                  nop
seg000:00000004                  fldz
seg000:00000006                  fnstenv byte ptr [esp-0Ch]
seg000:0000000A                  pop     esi
seg000:0000000B                  add     esi, 20h ; '  '
seg000:0000000E                  push    esi
seg000:0000000F                  pop     edi
seg000:00000010                  xor     ecx, ecx
seg000:00000012                  mov     cx, 2FFh
seg000:00000016
seg000:00000016 loc_16:                                  ; CODE XREF: seg000:00000022
seg000:00000016                  lodsw
seg000:00000018                  sub     ax, 6161h
seg000:0000001C                  shl     al, 4
seg000:0000001F                  add     al, ah
seg000:00000021                  stosb
seg000:00000022                  loop    loc_16
seg000:00000022 seg000          ends
```

【后面是编码的 shellcode】

看到里面 `fldz` 和 `fnstenv` 指令感到很疑惑，怀疑这段代码不能执行。于是放到 OD 中调试，由于 `esi` 被初始化指向了非法地址，导致运行产生异常。

这个样本先把包括解码头的整个 shellcode 复制到内存，然后用 `createthread` 把 shellocde 带起来。但是直接打开样本发现 shellcode 被解码并成功执行，最终释放一个 exe 并运行。

我的问题是 `fldz` 和 `fnstenv` 在线程中是如何对 `esi` 产生影响并完成解码工作，请各位指点。谢谢！

我认真看了下。。。。

`FLDZ`：将 0.0 装入 `st(0)`： `st(0)` <- `0.0`

`FSTENV`/`FNSTENV` :(`DST`:14 字节存贮器操作数)

`(DST)`(控制寄存器)

`(DST+2)`(状态寄存器)

`(DST+4)`(标记寄存器)

`(DST+6)`(指令指示器)

`(DST+10)`(操作数指示器)

所有错误屏蔽位置 1

感觉 `FLDZ` 和 `FNSTENV` 执行算是垃圾指令，并不影响解码的过程

`http://web.nuist.edu.cn/courses/wjyljyy/07336.htm`

引用: *最初由 **仙果** 发布* ![查看帖子](img/showthread.php?p=967882#post967882) 能否解释详细点呢？

`fnstenv` 将最后执行的一条 FPU 指令相关的协处理器的信息保存在指定的内存中，是按结构保存的

该结构偏移 12 字节处就是最后执行的浮点指令的运行时地址，本来有图的我这里粘贴不方便，更详细的你可以 google ‘`fnstenv` 重定位’
