
# 2022\*CTF babyarm 内核题目详细分析

## babyarm

### 概述

一道不常见的 aarch64 kernel pwn，这题很简单就简单栈溢出，`msr` 返回用户态 orw 即可

但就是返回用户态的机制不太好了解，然后流程控制也跟常规的 x86 那种有很大区别

### 代码审计

#### `device_read` 函数

![](upload/attach/202204/933934\_9DQYYEV9PTKRVUK.png)

#### `device_write` 函数

![](upload/attach/202204/933934\_2KU5EXCEDNWUC5T.png)

存在栈溢出漏洞

审一下代码就可以知道，红色框框中的东西一定会进，`read` 用来 leak，`write` 用来 rop

其实不怎么审代码，下好断点调试也可以知道进入哪里

`vmlinux` 从镜像中拉出来地址全是乱的。。。

gadgets 可以通过调试去找，直接在 `gdb` 里面找，还好这题 rop 不复杂，不然痛苦死了

但这个方法也不好，后面发现这个 aarch64 的 `Image` 文件可以直接 ida，但是只有偏移地址，但没关系

这题没开 kaslr，直接查内核基址再加上偏移就可以了

```python
# 查程序基址 各个节
grep demo /proc/modules
grep "0x" /sys/module/demo/sections/.\*
# 查 vmlinux 符号
grep prepare\_kernel\_cred /proc/kallsyms
grep commit\_creds /proc/kallsyms
grep \_text /proc/kallsyms # vmlinux 内核基址
```

然后说说 aarch64 的一些简单的汇编指令和知识

#### 寻址格式

`#` 代表立即数

```c
[sp, #0x10] // 从 sp+0x10 的地方取值
[sp, #0x10]! // 从 sp+0x10 的地方取值，取完值后 sp+0x10
[sp], #0x10 // 从 sp 的地方取值，取完值后 sp+0x10
```

#### 寄存器

在 aarch64 汇编中寄存器是 64 位的，使用 `X[n]` 表示，低32 位以 `W[n]` 表示

在 64 位架构中有 31 个 64 位的通用寄存器

使用如下指令在 `pwndbg` 中查看

```c
pwndbg> i reg
x0 0x0 0
x1 0x0 0
x2 0x8 8
x3 0x20 32
x4 0xffffffffffffffe0 -32
x5 0x40 64
x6 0x3f 63
x7 0x0 0
x8 0xffff000002eaebe8 -281474927760408
x9 0x0 0
x10 0x0 0
x11 0x0 0
x12 0x438614 4425236
x13 0x60001000 1610616832
x14 0x1200011 18874385
x15 0xdc 220
x16 0x0 0
x17 0x0 0
x18 0xcfab 53163
x19 0xffff000002ead100 -281474927767296
x20 0xffff80000a2b3da0 -140737317749344
x21 0x1200000 18874368
x22 0x0 0
x23 0xffff80000a2b8000 -140737317732352
x24 0xffff80000a2bbeb0 -140737317716304
x25 0x0 0
x26 0xffff000002eae480 -281474927762304
x27 0xffff80000a0417e0 -140737320314912
x28 0xffff000002eade80 -281474927763840
x29 0xffff80000a2b3bc0 -140737317749824
x30 0xffff800008016950 -140737354045104
sp 0xffff80000a2b3bc0 0xffff80000a2b3bc0
pc 0xffff800008016958 0xffff800008016958
cpsr 0x5 5
fpsr 0x0 0
fpcr 0x0 0
MVFR6\_EL1\_RESERVED 0x0 0
MVFR7\_EL1\_RESERVED 0x0 0
MAIR\_EL3 0x0 0
ID\_AA64PFR1\_EL1 0x1 1
ID\_AA64PFR2\_EL1\_RESERVED 0x0 0
ID\_AA64PFR3\_EL1\_RESERVED 0x0 0
SCTLR 0x200002034f4d91d 144115326403270941
ID\_AA64ZFR0\_EL1 0x0 0
CNTKCTL 0xc6 198
DACR32\_EL2 0x0 0
ID\_AA64PFR5\_EL1\_RESERVED 0x0 0
ACTLR\_EL1 0x0 0
CPACR 0x300000 3145728
ID\_AA64PFR6\_EL1\_RESERVED 0x0 0
FPEXC32\_EL2 0x0 0
ID\_AA64PFR7\_EL1\_RESERVED 0x0 0
ID\_AA64DFR0\_EL1 0x10305106 271601926
ID\_AA64DFR1\_EL1 0x0 0
ID\_AA64DFR2\_EL1\_RESERVED 0x0 0
ID\_AA64DFR3\_EL1\_RESERVED 0x0 0
ID\_AA64AFR0\_EL1 0x0 0
ID\_AA64AFR1\_EL1 0x0 0
ID\_AA64AFR2\_EL1\_RESERVED 0x0 0
CNTFRQ\_EL0 0x3b9aca0 62500000
ID\_AA64AFR3\_EL1\_RESERVED 0x0 0
SPSR\_EL1 0x60001000 1610616832
ID\_AA64ISAR0\_EL1 0x1021111110212120 1162228943821021472
DBGBVR 0x0 0
ELR\_EL1 0x438614 4425236
ID\_AA64ISAR1\_EL1 0x11101011010 1172542918672
PMEVTYPER0\_EL0 0x0 0
DBGBCR 0x0 0
ID\_AA64ISAR2\_EL1\_RESERVED 0x0 0
PMEVTYPER1\_EL0 0x0 0
DBGWVR 0x0 0
ID\_AA64ISAR3\_EL1\_RESERVED 0x0 0
ZCR\_EL1 0x0 0
DBGWCR 0x0 0
RVBAR\_EL1 0x0 0
ID\_AA64ISAR4\_EL1\_RESERVED 0x0 0
PMEVTYPER2\_EL0 0x0 0
PMEVTYPER3\_EL0 0x0 0
MDCCSR\_EL0 0x1000 4096
ID\_AA64ISAR5\_EL1\_RESERVED 0x0 0
ID\_AA64ISAR6\_EL1\_RESERVED 0x0 0
ID\_AA64ISAR7\_EL1\_RESERVED 0x0 0
SP\_EL0 0xffff000002ead100 -281474927767296
ID\_AA64MMFR0\_EL1 0x1124 4388
DBGBVR 0x0 0
ID\_AA64MMFR1\_EL1 0x11000 69632
DBGBCR 0x0 0
ID\_AA64MMFR2\_EL1\_RESERVED 0x0 0
PMINTENSET\_EL1 0x0 0
DBGWVR 0x0 0
ID\_AA64MMFR3\_EL1\_RESERVED 0x0 0
PMINTENCLR\_EL1 0x0 0
DBGWCR 0x0 0
ID\_AA64MMFR4\_EL1\_RESERVED 0x0 0
PMCNTENSET\_EL0 0x0 0
ACTLR\_EL2 0x0 0
PMCR\_EL0 0x2000 8192
ID\_AA64MMFR5\_EL1\_RESERVED 0x0 0
PMCNTENCLR\_EL0 0x0 0
VBAR 0xffff800008010800 -140737354070016
ID\_AA64MMFR6\_EL1\_RESERVED 0x0 0
PMOVSCLR\_EL0 0x0 0
MDSCR\_EL1 0x1000 4096
ID\_AA64MMFR7\_EL1\_RESERVED 0x0 0
CNTP\_CTL\_EL0 0x0 0
PMSELR\_EL0 0x0 0
CNTP\_CVAL\_EL0 0x0 0
DBGBVR 0x0 0
DBGBCR 0x0 0
PMCEID1\_EL0 0x0 0
PMCEID0\_EL0 0x20001 131073
DBGWVR 0x0 0
PMCCNTR\_EL0 0x0 0
DBGWCR 0x0 0
L2ACTLR 0x0 0
TTBR0\_EL1 0x42eca000 1122803712
TTBR1\_EL1 0x280000418b0000 11259000168054784
CNTV\_CTL\_EL0 0x1 1
TCR\_EL1 0x400034b5503510 18014624889713936
DBGBVR 0x0 0
DBGBCR 0x0 0
ACTLR\_EL3 0x0 0
CNTV\_CVAL\_EL0 0x2aff26ff 721364735
DBGWVR 0x0 0
ZCR\_EL2 0x0 0
PMUSERENR\_EL0 0x0 0
DBGWCR 0x0 0
PMOVSSET\_EL0 0x0 0
APIAKEYLO\_EL1 0x0 0
APIAKEYHI\_EL1 0x0 0
SP\_EL1 0xffff80000a2b4000 -140737317748736
MDRAR\_EL1 0x0 0
APIBKEYLO\_EL1 0x0 0
PMCCFILTR\_EL0 0x0 0
DBGBVR 0x0 0
APIBKEYHI\_EL1 0x0 0
DBGBCR 0x0 0
CPUACTLR\_EL1 0x0 0
CPUECTLR\_EL1 0x0 0
CONTEXTIDR\_EL1 0x0 0
APDAKEYLO\_EL1 0x0 0
APDAKEYHI\_EL1 0x0 0
CNTPS\_CTL\_EL1 0x0 0
CPUMERRSR\_EL1 0x0 0
CNTPS\_CVAL\_EL1 0x0 0
L2MERRSR\_EL1 0x0 0
DBGBVR 0x0 0
APDBKEYLO\_EL1 0x0 0
MAIR\_EL1 0x40044ffff 17184391167
APDBKEYHI\_EL1 0x0 0
DBGBCR 0x0 0
TPIDR\_EL1 0xffff80000673f000 -140737380093952
AFSR0\_EL1 0x0 0
OSLSR\_EL1 0x8 8
AFSR1\_EL1 0x0 0
PAR\_EL1 0x0 0
CBAR\_EL1 0x8000000 134217728
APGAKEYLO\_EL1 0x0 0
APGAKEYHI\_EL1 0x0 0
SPSR\_IRQ 0x0 0
MDCR\_EL3 0x0 0
SPSR\_ABT 0x0 0
AMAIR0 0x0 0
FPCR 0x0 0
SPSR\_UND 0x0 0
SPSR\_FIQ 0x0 0
FPSR 0x0 0
ESR\_EL1 0x56000000 1442840576
CLIDR 0xa200023 169869347
REVIDR\_EL1 0x0 0
ID\_PFR0 0x131 305
ID\_DFR0 0x3010066 50397286
ID\_AFR0 0x0 0
ID\_MMFR0 0x10101105 269488389
CSSELR 0x0 0
LORSA\_EL1 0x0 0
ID\_MMFR1 0x40000000 1073741824
AIDR 0x0 0
TPIDR\_EL0 0x11439700 289642240
LOREA\_EL1 0x0 0
ID\_MMFR2 0x1260000 19267584
TPIDRRO\_EL0 0x0 0
LORN\_EL1 0x0 0
ID\_MMFR3 0x2102211 34611729
IFSR32\_EL2 0x0 0
LORC\_EL1 0x0 0
ID\_ISAR0 0x2101110 34607376
ID\_ISAR1 0x13112111 319889681
PMEVCNTR0\_EL0 0x0 0
ID\_ISAR2 0x21232042 555950146
PMEVCNTR1\_EL0 0x0 0
ID\_ISAR3 0x1112131 17899825
CTR\_EL0 0x8444c004 2219098116
LORID\_EL1 0x0 0
PMEVCNTR2\_EL0 0x0 0
ID\_ISAR4 0x11142 69954
PMEVCNTR3\_EL0 0x0 0
ID\_ISAR5 0x11011121 285282593
ID\_MMFR4 0x0 0
ID\_ISAR6 0x11111 69905
L2CTLR\_EL1 0x0 0
MVFR0\_EL1 0x10110222 269550114
L2ECTLR\_EL1 0x0 0
FAR\_EL1 0xffffce1148b0 281474138982576
MVFR1\_EL1 0x12111111 303108369
MVFR2\_EL1 0x43 67
MVFR3\_EL1\_RESERVED 0x0 0
MVFR4\_EL1\_RESERVED 0x0 0
MVFR5\_EL1\_RESERVED 0x0 0
```

![](upload/attach/202204/933934\_TXK7CDBSMW33UJ8.png)

`x0`~`x7`：一般是函数的参数，大于8个的会通过堆栈传参

`x0` 还用作返回值

`X9`-`X15`：调用者保存的临时寄存器

`X19`-`X29`：被调用者保存的寄存器

- 特殊用途寄存器：
  - `X8`：是间接结果寄存器，用于保存子程序返回地址
  - `X16` 和 `X17`：程序内调用临时寄存器
  - `X18`：平台寄存器
  - `X29`：帧指针寄存器（FP），类似 `rsp`
  - `X30`：链接寄存器（LR），一般存的是返回地址
  - `X31`：堆栈指针寄存器 `SP` 或零寄存器 `ZXR`

#### 关于 aarch64 架构下的开辟栈帧

x86 下一般是 `push`、`pop` 指令

而 aarch 64 下就是 `LDP`、`STP` 指令

例如：

```c
STP X29, X30, [SP,#var\_40]!
...
LDP X21, X22, [SP,#0x40+var\_20]
LDP X29, X30, [SP+0x40+var\_40],#0x40
RET
```

- `STP` 将 `x29`、`x30` 存入`[SP,#var_40]`（两个八字节），然后 `SP+var_40`
- `LDP` 从 `[SP,#0x40+var_20]` 中取出值，放入 `X21`, `X22`
- `LDP` 从 `[SP+0x40+var_40]` 中取出值（两个八字节），放入 `x29`、`x30`，然后 `sp + 0x40`

#### 关于返回用户态

使用 `gdb` 单步调试可以找到如下 gadgets，也就是利用 `msr` 指令进行寄存器的恢复

```c
// ret2\_usr\_mode
► 0xffff800008011fe4: msr sp\_el0, x23
...
...
0xffff800008012024 msr elr\_el1, x21
0xffff800008012028 msr spsr\_el1, x22
0xffff80000801202c ldp x0, x1, [sp]
0xffff800008012030 ldp x2, x3, [sp, #0x10]
0xffff800008012034 ldp x4, x5, [sp, #0x20]
0xffff800008012038 ldp x6, x7, [sp, #0x30]
0xffff80000801203c ldp x8, x9, [sp, #0x40]
0xffff800008012040 ldp x10, x11, [sp, #0x50]
0xffff800008012044 ldp x12, x13, [sp, #0x60]
0xffff800008012048 ldp x14, x15, [sp, #0x70]
0xffff80000801204c ldp x16, x17, [sp, #0x80]
```

- `sp_el0`：保存用户态的栈指针
- `elr_el1`：保存要返回的用户态 PC 指针（一般写成 `system` 即可）
- `spsr_el1`：固定值 `0x80001000`

`msr` 指令会分别将 `x21`、`x22`、`x23` 的值还原给上面三个重要寄存器

所以伪造这三个值即可返回用户态

### 利用

先改一下 `init` 文件

```bash
#!/bin/sh
mount -t devtmpfs none /dev
mount -t proc none /proc
mount -t sysfs none /sys
insmod /home/pwn/demo.ko
chown -R 1000:1000 /home/pwn
echo 1 > /proc/sys/kernel/dmesg\_restrict
echo 1 > /proc/sys/kernel/kptr\_restrict
echo 1 > /proc/sys/kernel/perf\_event\_paranoid
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
# cd /home/pwn
# setsid cttyhack setuidgid 1000 sh
# setsid cttyhack setuidgid 0 sh
setsid /bin/cttyhack setuidgid 1000 /bin/sh
# setsid /bin/cttyhack setuidgid 0 /bin/sh
umount /proc
poweroff -f
```
