
# 金天地的 TDMD

> 原文：[`www.pediy.com/kssd/pediy04/kanxue164.htm`](https://www.pediy.com/kssd/pediy04/kanxue164.htm)

## TDMD 软件狗破解方法（带狗杀狗）

**工具：** `TRW2000`，`Hview`

这次杀狗还是用带狗杀狗方法，破解狗的常用断点：`bpio 378`，`bpx deviceiocontrol`，`bpx environmentstringa`（听说对 `HASP` 狗有效，我没有试过），今天我们用 `bpx deviceiocontrol` 这个断点来破它。

### 一般 TDMD 狗的调用形式

```assembly
:0057C054 53                 push ebx
:0057C055 56                 push esi
:0057C056 57                 push edi
:0057C057 56                 push esi
:0057C058 57                 push edi
:0057C059 52                 push edx
:0057C05A 51                 push ecx
:0057C05B 53                 push ebx
:0057C05C 50                 push eax
:0057C05D E82C020000         call 0057C28E     <===== 判断狗的 CALL
:0057C062 85C0               test eax, eax      <===== 有狗就返回 0
:0057C064 7405               je 0057C06B        <===== 有狗就跳到正常的程序,否则就退出程序
:0057C066 E989000000         jmp 0057C0F4
```

### 中断返回主程序

好了，下断点后程序会被中断在狗驱动中，从 `TRW2000` 中我们知道这个狗是金天地的 `TDMD`，中断后我们用 `F10` 和 `F12` 返回到主程序中，如下：

```assembly
:0057C7B0 50                 push eax
:0057C7B1 FF15AC815D00       call dword ptr [005D81AC]     <==== 第一次读狗,在 TRW2000 中应该是 call KERNEL32!Deviceiocontrol,这个 CALL 是判断端口上有没有活动的打印机和加密狗
:0057C7B7 8945F8             mov dword ptr [ebp-08], eax    <===== 返回到这里
:0057C7BA 837DF800          cmp dword ptr [ebp-08], 00000000
:0057C7BE 7421              je 0057C7E1                     <===== 不跳
:0057C7C0 83BD20FFFFFF00    cmp dword ptr [ebp+FFFFFF20], 00000000
:0057C7C7 7416              je 0057C7DF                     <===== 不跳
:0057C7C9 81BD20FFFFFF00300000 cmp dword ptr [ebp+FFFFFF20], 00003000
:0057C7D3 740A              je 0057C7DF                     <====== 不跳
:0057C7D5 818520FFFFFF0000500000 add dword ptr [ebp+FFFFFF20], 00500000
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:0057C7C7(C), :0057C7D3(C)`

```assembly
:0057C7DF EB0A              jmp 0057C7EB
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C7BE(C)`

```assembly
:0057C7E1 C78520FFFFFF00005200 mov dword ptr [ebp+FFFFFF20], 00520000
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C7DF(U)`

```assembly
:0057C7EB EB01              jmp 0057C7EE
:0057C7ED E8                BYTE E8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C7EB(U)`

```assembly
:0057C7EE 83BD20FFFFFF00    cmp dword ptr [ebp+FFFFFF20], 00000000    <==== 第一次判断狗
:0057C7F5 7427              je 0057C81E       <==== 没有狗就不跳,所以我们这里要把它改为 Jmps 0057C81E
:0057C7F7 EB01              jmp 0057C7FA
:0057C7F9 E8                BYTE E8
```

### 查看 0057C81E 处的代码

看一下 `0057C81E` 的代码:

```assembly
:0057C81E 8B8530FEFFFF      mov eax, dword ptr [ebp+FFFFFE30]
:0057C824 A3A0705D00        mov dword ptr [005D70A0], eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C77F(C)`

```assembly
:0057C829 EB01              jmp 0057C82C
:0057C82B E8                BYTE E8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C829(U)`

```assembly
:0057C82C E8C8F8FFFF        call 0057C0F9     <===== 计算密码的 CALL
:0057C831 0FB7D8            movzx ebx, ax
:0057C834 C1E310            shl ebx, 10
:0057C837 E8BDF8FFFF        call 0057C0F9     <===== 计算密码的 CALL
:0057C83C 0FB7C0            movzx eax, ax
:0057C83F 0BD8              or ebx, eax
:0057C841 899D38FEFFFF      mov dword ptr [ebp+FFFFFE38], ebx
:0057C847 EB01              jmp 0057C84A
:0057C849 E8                BYTE E8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0057C847(U)`

```assembly
:0057C84A A168705D00        mov eax, dword ptr [005D7068]
:0057C84F 50                push eax
:0057C850 E883FAFFFF        call 0057C2D8
:0057C855 83C404            add esp, 00000004
:0057C858 898540FEFFFF      mov dword ptr [ebp+FFFFFE40], eax
:0057C85E EB01              jmp 0057C861
```

### 第二次读狗与判断

然后就是按 `F10` 一直向下走了,直到出现第二个 `CALL KERNEL32!Deviceiocontrol` 时:

```assembly
:0057C8E1 50                push eax
:0057C8E2 FF15AC815D00      call dword ptr [005D81AC]     <===== 第二次读狗(Call KERNEL32!Deviceiocontrol)
:0057C8E8 8945F8            mov dword ptr [ebp-08], eax
:0057C8EB 8B45FC            mov eax, dword ptr [ebp-04]
:0057C8EE 50                push eax
:0057C8EF FF1558815D00      call dword ptr [005D8158]
:0057C8F5 8B8528FEFFFF      mov eax, dword ptr [ebp+FFFFFE28]
:0057C8FB 50                push eax
:0057C8FC E85DFBFFFF        call 0057C45E
:0057C901 83C404            add esp, 00000004
:0057C904 837DF800          cmp dword ptr [ebp-08], 00000000
:0057C908 7434              je 0057C93E
:0057C90A 83BD20FFFFFF00    cmp dword ptr [ebp+FFFFFF20], 00000000 <===== 第二次判断狗,如果没有狗,这里的内存地址[81FD08]是一个非 0 的数,同时这个内存地址也是加密狗的重要内存地址,所以我们要把这个地址的内容改为 0,我的改法是:and dword prt [ebp+ffffff20],00000000,刚好是一样的代码长度
:0057C911 7513              jne 0057C926
:0057C913 8B8524FFFFFF      mov eax, dword ptr [ebp+FFFFFF24]   <==== 取从狗中读出的数据
:0057C919 338538FEFFFF      xor eax, dword ptr [ebp+FFFFFE38]  <==== 与另外的一个数据异或,将结果赋给 EAX
:0057C91F A3D8705D00        mov dword ptr [005D70D8], eax      <===== 将结果赋给内存地址[005D70D8],我们先带狗运行程序,可知运行到这里的时候,EAX 的值是 5CCAB580,所以我们必须把这个内存地址的内容赋为 5CCAB580
:0057C924 EB16              jmp 0057C93C
```

### 修改方案

我的改法是:

```assembly
:0057C90A 83A520FFFFFF00    and dword ptr [ebp+FFFFFF20], 00000000
:0057C911 7513              jne 0057C926
:0057C913 8B8524FFFFFF      mov eax, dword ptr [ebp+FFFFFF24]
:0057C919 B880B5CA5C        mov eax, 5CCAB580     <===== 将 EAX 赋予 5CCAB580
:0057C91E 90                nop
:0057C91F A3D8705D00        mov dword ptr [005D70D8], eax  <====== 将 5CCAB580 赋给内存地址[005D70D8]
:0057C924 EB16              jmp 0057C93C
```

至此,`TDMD` 狗就被我们破掉了
