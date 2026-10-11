
# 寂寞轰炸：再次小窥 VMProtect 虚拟机

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-637.htm`](https://www.pediy.com/kssd/pediy08/pediy8-637.htm)

## VMProtect 的虚拟机

最近被寂寞轰炸，元旦快完蛋了，分析一下 VMProtect 吧。

可能不准确，但是可以作为初级读本来了解这个保护。

附件下载：`xxx.rar`

\==========================

## Abstact

\==========================

1 - 既然是 VM，就有自己的 CONTEXT，而 `VMCONTEXT` 是线程相关的（安全、可重入）

   每个未注册的线程进入 VM 会在 VM 的线程槽中用 `ThreadID` 注册一个槽

   然后用此槽号来访问 `VMSEG_DATA` 中对应的 `VMCONTEXT`

2 - 有一个主循环解码 `OPCODE`

   可以设断开发一个类似 `WKT DBGR` 的工具

3 - 类似于 `VC PCODE` 或者 `MMX` 指令

   所有数据（包括运算结果的符号）都通过堆栈传递

4 - 跟 VM 数据区交换的同一指令在 `DispatchDescriptor` 中有很多描述，比如 `vpop`

   其中某些位指出了偏移量

5 - 运算指令总是结束后 `pushf`

   这样在 VM 解码的过程中不会丢失符号

6 - 指令解释器的序列不同的文件不一样，但看起来指令的特征不会变（想象 `FLIRT`）

   同一个 `VICTIM` 可以识别出来生成一个 `MAP` 来调试！这一点很重要！！！！

7 - 指令解释器的代码基本不变，但涉及立即数操作的

   会加入随机的解码算法，但应该可以根据首尾特征来判断虚拟指令

8 - 一些指令的 `BYTE` 立即数被扩展成了 `WORD`，但只利用 `BYTE` 位

9 - VM 跟 `x86` 不对应，很多垃圾，也许我指分析到了最浅的一层

   我深刻地感到，也许还有 VM 中的 VM？

\==========================

## Example

\==========================

很晕！以我加了一个混乱 OEP 的 `notepad.exe` 为例

!!!注记符是我起的，不要太在意，可以按浮点指令理解

进入 VM 的代码样式为：

```
  push  key0
  push  VMSEG_CODE//代码段
  jmp  VMP_MAIN
```

此例一开始堆栈环境为

```
[00000000]
[EDI]
[EFlags]
[ESI]
[EAX]
[EBP]
[EDX]
[ECX]
[EBX]
[EAX]
[VMSEG_CODE]
[key0]//这里为 E95243D1
```

```
  vpop//这个指令是一个系列的，OPCODE & 3C 也就是 111100 为 1 的位表示在 VMSEG_DATA 中的偏移
  ...
  vpop//将上面的寄存器弹到 VM CONTEXT 中去
  vpush  B2C49D2C
  vadd//注意这个指令运算后 pushfd，运算结果为 9C16E0FD
  vpop//保存符号
  vpop//保存结果
  vpushw//压入 0001
  vpushv//0001 入栈 C
  vpushv//9C16E0FD 入栈 B
  vpushv//9C16E0FD 入栈 A
  vshld//shld A,B,C，保存结果&符号
  vpop//保存符号
  ...好多
```

\===============================

## List

\===============================

我用猜的方法命名了一些指令，都以 `v` 开头，

后来有点规则混乱但是没有改，涉及到跟 `VMDATA` 传输的以 `vv` 开头

如果是求值操作结尾为 `pd/w/b`，其中 `b` 可能被扩展成 `WORD` 样子 `push` 了

我也混乱了，不说了，这是一张表，详细的请看 `idb`

```
pcode_dispatch_table           dd offset vvpop

.vmp0:010174FD                      

.vmp0:01017501                     dd offset vshr
.vmp0:01017505                     dd offset loc_10174EF
.vmp0:01017509                     dd offset loc_10174AD
.vmp0:0101750D                     dd offset vvpop
.vmp0:01017511                     dd offset loc_10174D1
.vmp0:01017515                     dd offset vpopw
.vmp0:01017519                     dd offset vpushpb
.vmp0:0101751D                     dd offset vvpop
.vmp0:01017521                     dd offset vpushvesp
.vmp0:01017525                     dd offset vpopcs
.vmp0:01017529                     dd offset vpushvesp
.vmp0:0101752D                     dd offset vvpop
.vmp0:01017531                     dd offset vpushb2d
.vmp0:01017535                     dd offset loc_10174EF
.vmp0:01017539                     dd offset vpushimm32
.vmp0:0101753D                     dd offset vvpop
.vmp0:01017541                     dd offset vnaw
.vmp0:01017545                     dd offset vaddp
.vmp0:01017549                     dd offset vpoppd
.vmp0:0101754D                     dd offset vvpop
.vmp0:01017551                     dd offset vaddb
.vmp0:01017555                     dd offset vvpushpw
.vmp0:01017559                     dd offset vpopcs
.vmp0:0101755D                     dd offset vvpop
.vmp0:01017561                     dd offset vshlb
.vmp0:01017565                     dd offset loc_10178FD
.vmp0:01017569                     dd offset vpushdwptr
.vmp0:0101756D                     dd offset vvpop
.vmp0:01017571                     dd offset loc_10174AD
.vmp0:01017575                     dd offset vpushw
.vmp0:01017579                     dd offset vaddw
.vmp0:0101757D                     dd offset vvpop
.vmp0:01017581                     dd offset vshld
.vmp0:01017585                     dd offset vsetesp
.vmp0:01017589                     dd offset vnna
.vmp0:0101758D                     dd offset vvpop
.vmp0:01017591                     dd offset vshr
.vmp0:01017595                     dd offset vvpushb
.vmp0:01017599                     dd offset vaddp
.vmp0:0101759D                     dd offset vvpop
.vmp0:010175A1                     dd offset loc_1017400
.vmp0:010175A5                     dd offset vvpushpw
.vmp0:010175A9                     dd offset vpushimm32
.vmp0:010175AD                     dd offset vvpop
.vmp0:010175B1                     dd offset vpushpd
.vmp0:010175B5                     dd offset vshlb
.vmp0:010175B9                     dd offset vpushw
.vmp0:010175BD                     dd offset vvpop
.vmp0:010175C1                     dd offset vpushw2d
.vmp0:010175C5                     dd offset vshlw
.vmp0:010175C9                     dd offset loc_10174EF
.vmp0:010175CD                     dd offset vvpop
.vmp0:010175D1                     dd offset vpushsp
.vmp0:010175D5                     dd offset vpushwi
.vmp0:010175D9                     dd offset vpushpw
.vmp0:010175DD                     dd offset vvpop
.vmp0:010175E1                     dd offset vpushb2d
.vmp0:010175E5                     dd offset vnna
.vmp0:010175E9                     dd offset vpopw
.vmp0:010175ED                     dd offset vvpop
.vmp0:010175F1                     dd offset vpushvesp
.vmp0:010175F5                     dd offset vpoppd
.vmp0:010175F9                     dd offset vnaw
.vmp0:010175FD                     dd offset vvpush
.vmp0:01017601                     dd offset vvsetb
.vmp0:01017605                     dd offset vpushw2d
.vmp0:01017609                     dd offset vnnab
.vmp0:0101760D                     dd offset vvpush
.vmp0:01017611                     dd offset vhalt
.vmp0:01017615                     dd offset loc_101740C
.vmp0:01017619                     dd offset vpopcs
.vmp0:0101761D                     dd offset vvpush
.vmp0:01017621                     dd offset vpushpd
.vmp0:01017625                     dd offset vpushw2d
.vmp0:01017629                     dd offset vshlb
.vmp0:0101762D                     dd offset vvpush
.vmp0:01017631                     dd offset vshr
.vmp0:01017635                     dd offset vshl
.vmp0:01017639                     dd offset loc_10174D1
.vmp0:0101763D                     dd offset vvpush
.vmp0:01017641                     dd offset vpushpd
.vmp0:01017645                     dd offset loc_10174D1
.vmp0:01017649                     dd offset vpushwi
.vmp0:0101764D                     dd offset vvpush
.vmp0:01017651                     dd offset vpushw
.vmp0:01017655                     dd offset vpopsp
.vmp0:01017659                     dd offset vstob
.vmp0:0101765D                     dd offset vvpush
.vmp0:01017661                     dd offset vvpushb
.vmp0:01017665                     dd offset vpopcs
.vmp0:01017669                     dd offset vpushdwptr
.vmp0:0101766D                     dd offset vvpush
.vmp0:01017671                     dd offset vshlb
.vmp0:01017675                     dd offset vpushpb
.vmp0:01017679                     dd offset loc_10174EF
.vmp0:0101767D                     dd offset vvpush
.vmp0:01017681                     dd offset vpushsp
.vmp0:01017685                     dd offset vaddw
.vmp0:01017689                     dd offset vpopsp
.vmp0:0101768D                     dd offset vvpush
.vmp0:01017691                     dd offset loc_10174AD
.vmp0:01017695                     dd offset vvpushpw
.vmp0:01017699                     dd offset loc_101740C
.vmp0:0101769D                     dd offset vvpush
.vmp0:010176A1                     dd offset loc_10174AD
.vmp0:010176A5                     dd offset vpoppw
.vmp0:010176A9                     dd offset vstopb
.vmp0:010176AD                     dd offset vvpush
.vmp0:010176B1                     dd offset vd2w
.vmp0:010176B5                     dd offset loc_10174EF
.vmp0:010176B9                     dd offset vvpushb
```
