
```text
        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

        EBP 0013F980

        0013F980   8021D2F0  !

        0013F984   0013F9C0  .

        0013F988   00000246  F..

        0013F98C   00000020   ...

        0013F990   000359F4  Y.

        0013F994   0013F9CC  .

        0013F998   00400000  ..@.  ; OFFSET NOTEPAD

        0013F99C   00000000  ....

        0013F9A0   004253CD  SB.   ; RETURN from NOTEPAD.004255DB to NOTEPAD.004253CD

        0013F9A4   000359F4  Y.

        0013F9A8   00400000  ..@.  ; 该带着走的数据都要在EBPSTACK里面带着走，到这里还没有完毕的。

        0013F9AC   0043651A  eC.  ；还有其他的数据要放入，8021D2F0要隐藏一下

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_PUSHdw_IMMEDIATEdw

        0013F97C   7FDE2D10  -

        0013F980   8021D2F0  !

        0013F984   0013F9C0

VM_ADDdw_EBPSTACK

        0013F97C   00000247  G..

        0013F980   00000000  ....

        0013F984   0013F9C0

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F980   00000000  ....

        0013F984   0013F9C0

VM_PUSHdw_EDISTACKdw

VM_PUSHdw_EDISTACKdw

        0013F978   0043651A  eC.

        0013F97C   00000000  ....

        0013F980   00000000  ....

        0013F984   0013F9C0  .

        0013F988   00000246   F..

        0013F98C   00000020    ...

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

        0013F998   00400000   ..@.  ; OFFSET NOTEPAD

        0013F99C   00000000   ....

        0013F9A0   004253CD   SB.   ; RETURN from NOTEPAD.004255DB to NOTEPAD.004253CD

        0013F9A4   000359F4   Y.

        0013F9A8   00400000   ..@.  ; OFFSET NOTEPAD.B

        0013F9AC   0043651A   eC.

        0013F9B0   7C92118A   |    ; RETURN to ntdll.7C92118A

VM_JMP                                        ；带着14个数据，VM终于跳转，除了栈顶0043651A放入ESI，其他13个数据要重新保存

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F980   00000000  ....

        0013F984   0013F9C0  .

VM_PUSHdw_IMMEDIATEdw

        0013F97C   8021D2F0   !

        0013F980   00000000   ....

        0013F984   0013F9C0   .

VM_ADDdw_EBPSTACK

        0013F97C   00000286   ..

        0013F980   8021D2F0   !

        0013F984   0013F9C0   .        ；重新恢复出来

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw                ；到这里停一下，搞个小运算，原EDX=000359F4 XOR 4DFD2FC2

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

        0013F998   00400000   ..@.  ; OFFSET NOTEPAD.B

        0013F99C   00000000   ....

        0013F9A0   004253CD   SB.   ; RETURN from NOTEPAD.004255DB to NOTEPAD.004253CD

        0013F9A4   000359F4   Y.

        0013F9A8   00400000   ..@.  ; OFFSET NOTEPAD.B

        0013F9AC   0043651A   eC.

        0013F9B0   7C92118A   |    ; RETURN to ntdll.7C92118A

VM_PUSHdw_EBP

        0013F98C   0013F990   .

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

VM_COPYdw_EBPSTACK

        0013F98C   000359F4   Y.

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

VM_PUSHdw_EBP

        0013F98C   0013F990   .

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

VM_COPYdw_EBPSTACK

        0013F98C   000359F4   Y.

        0013F990   000359F4   Y.

        0013F994   0013F9CC   .

VM_NANDdw

        0013F98C   00000282   ..

        0013F990   FFFCA60B

        0013F994   0013F9CC   .

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F990   FFFCA60B

        0013F994   0013F9CC   .

VM_PUSHdw_IMMEDIATEdw

        0013F98C   B202D03D   =

        0013F990   FFFCA60B

        0013F994   0013F9CC   .

VM_NANDdw

        0013F98C   00000206   ..

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_PUSHdw_IMMEDIATEdw

        0013F98C   4DFD2FC2   /M

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_PUSHdw_EDISTACKdw

        0013F988   000359F4   Y.

        0013F98C   4DFD2FC2   /M

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_NANDdw

        0013F988   00000286   ..

        0013F98C   B2008009   ..

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F98C   B2008009   ..

        0013F990   000109C0   ..

        0013F994   0013F9CC   .

VM_NANDdw

        0013F98C   00000206   ..

        0013F990   4DFE7636   6vM

        0013F994   0013F9CC   .

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        EBP 0013F9B0

        0013F9B0   7C92118A   |    ; RETURN to ntdll.7C92118A
```

每一次`VM_JMP`跳转，都要带着14个数据转，而其中呢VM还要搞上一点暗码转移。以后看到是`VM_JMP`跳转的状况，就是看着`EBP`指针，哗哗哗的让它执行，完毕了才停下来。中间的操作完全可以无视，我也是为了完整的表达才把它粘贴出了代码，实际看的时候，不用管。整个过程：要带着走的数据移到`EBPSTACK`-->`VM_JMP`跳转-->重新把数据保存到`EDISTACK`。关于其中`000359F4 XOR 4DFD2FC2`的过程，请参考下一节`2.2.NAND`。

由于其他的组合都和`NAND`或标志位检测+跳转相关，放在下两节中。这一节中的3个组合熟悉后，已经可以无视掉一部分VM的操作。

## 2.2.NAND（与非门）

本文的两节重头戏来了，`NAND`（与非门）与`EFLAGS`标志位检测+跳转，理解完了这两节后，对于VM就可以无视了，一切伪指令在你眼里都是正常的指令。跟踪VMP就和跟踪普通程序一样，想看API获取就看API获取，想看看程序的anti方式就看anti方式。一切都回到了正常，你可以看穿VM（虚拟机）这个吓人的外衣。

### 2.2.1.NAND起源

`NAND`（与非门）和`NOR`（或非门）来源于`de Morgan's Laws`（德·摩根定律），运用于逻辑、数字电路等方面，本节专注于它与`and or xor not`之间的联系。

德·摩根定律是属于逻辑学的定律。 德·摩根定律(或称德·摩根定理)是形式逻辑中有关否定所描述的系统方式中的逻辑运算符对偶对的一系列法则。由此引出的关系也就被称为“德·摩根二重性”。

奥古斯都·德·摩根首先发现了在命题逻辑中存在着下面这些关系：

非(P 且 Q)＝(非 P)或(非 Q)

非(P 或 Q)＝(非 P)且(非 Q)

德·摩根的发现影响了乔治·布尔从事的逻辑问题代数解法的研究，这巩固了德·摩根作为该规律的发现者的地位，尽管亚里士多德也曾注意到类似现象、且这也为古希腊与中世纪的逻辑学家熟知(引自Bocheński《形式逻辑历史》)。（引自维基百科，关键字：德·摩根定律）

我们再来看它在数学逻辑中的表示：

![](https://bbs.kanxue.com/upload/attach/201010/365193_pih8doz1v8mn630.jpg)

（引自：MathWorld，关键字：de Morgan's Laws）

由于不是用我们熟悉的计算机方式来表达，上面的两段解说比较抽象，请看2.2.2.

### 2.2.2.NAND与逻辑运算

在加壳记事本中使用的是`NAND`，下面部分将专注于`NAND`。对于`NOR`，理论都是一样的，只是不用`NAND`来实现。

```text
NAND（A,B)：

        NOT(A)

        NOT(B)

        AND(A,B)
```

这就是`NAND`的操作方式。`NAND`的价值在于：使用`NAND`可以实现`NOT` `AND` `OR` `XOR`这4个逻辑运算。

```text
NOT(A):

        NAND(A,A)

AND(A,B):

        NAND(NAND(A,A),NAND(B,B))

OR（A,B):

        NAND(NAND(A,B),NAND(A,B))

XOR(A,B):

        NAND(NAND(NAND(A,A),NAND(B,B)),NAND(A,B))
```

### 2.2.3.VMP伪指令执行过程

`NOT(4DBE4AD8）：`

```text
        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_PUSHdw_EBP

        0013F9A8   0013F9AC  .

        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_COPYdw_EBPSTACK

        0013F9A8   4DBE4AD8  JM

        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_NANDdw

        0013F9A8   00000286  ..

        0013F9AC   B241B527  'A

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A        ；**NAND(A,A)**

VM_MOVdw_EDISTACKdw_EBPSTACKdw

        0013F9AC   B241B527  'A

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A
```

`NOT(4DBE4AD8)=B241B527`

`AND(4DBE4AD8,4DFD2FC2)：`

```text
        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_PUSHdw_EBP

        0013F9A8   0013F9AC  .

        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_COPYdw_EBPSTACK

        0013F9A8   4DBE4AD8  JM

        0013F9AC   4DBE4AD8  JM

        0013F9B0   7C92118A  |    ; RETURN to ntdll.7C92118A

VM_NANDdw

        0013F9A8   00000286  ..

        0013F9AC   B241B527  'A
```
