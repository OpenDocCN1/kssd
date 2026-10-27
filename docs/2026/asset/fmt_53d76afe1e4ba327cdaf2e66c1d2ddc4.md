
```
...

005B27B5  |.  94     |vJmp_005BED37    if (v258 == 0) goto VMPCrack.005BB940

...

005B7F74  |.  94     |vJmp_005BED37    if (m37 == 0CC) goto <VMPCrack.发现调试器>

005A9103  |.  F7     |vPopReg4 vR3

005A90D8  |.  5A 00  |vPushImmSx1 0    ARG5 DWORD v262 = 0

005A90D6  |.  12 04  |vPushImmSx1 4    ARG4 DWORD v263 = 4

005A90D4  |.  25     |vPushReg4 vR14   ARG3 DWORD v264 = v20

005A90D3  |.  2F 1E  |vPushImmSx1 1E   ARG2 DWORD v265 = 1E		//ProcessDebugObjectHandle

005A90D1  |.  2F FF  |vPushImmSx1 0FF  ARG1 DWORD v266 = 0FFFFFFFF

005A90CE  |.  3A 05  |vCall 5          v267 = Call(v260)		//ZwQueryInformationProcess

...

005A907A  |.  06     |vJmp_005BED37    if (v267 == 0) goto <VMPCrack.发现调试器>
```

调用`ZwQueryInformationProcess`，参数是`ProcessDebugObjectHandle`，检查是否存在调试对象。然后是一个联机指令

```
005BB8F9  |. /37  |vRet  online 5AD874; VMPCrack.005AD874; POP SS
```

一共两条指令

```
005AD874       17              POP SS
005AD875       9C              PUSHFD
```

这个指令有对堆栈的引用和改变，填上提示`/*stackRef(0,2);stackModify(0,4,EFL)*/`，回去看代码

```
005BB940  |> \1F  |vPopReg4 vR13

005BB90D  |.  34  |vPushSs         EXIT WORD v269 = Ss				//要弹出的ss

...

005BB8F9  |.  37  |vRet            online 5AD874; VMPCrack.005AD874; POP SS

...

005B6992  |.  06  |vJmp_005BED37   if (Trap(HiWord(EFL) : LoWord(EFL))) goto <VMPCrack.发现调试器>
```

弹出ss后把标志压栈，检测标志中是否有`TF`，有的话就转到发现调试器。查了一下资料说`POP SS`执行后会屏蔽中断直到下一条指令执行，估计是用来反跟踪的，如果正在跟踪，执行`PUSHFD`时由于中断被屏蔽不会发生异常，保存的标志中会有`TF`。然后又是联机指令

```
005B9271  |. /2E  |vRet  online 5AD263; VMPCrack.005AD263; PREFIX REP:
```

指令是

```
005AD263       F3:             PREFIX REP:
005AD264       9C              PUSHFD
```

带前缀的`PUSHFD`？改变了堆栈和`esp`，添加提示`/*esp(-4);stackModify(-4,4,EFL)*/`，回去看代码

```
...

005B9271  |.  2E  |vRet            online 5AD263; VMPCrack.005AD263; PREFIX REP:

...

005A9F84  |.  06  |vJmp_005BED37   if (Trap(EFL)) goto <VMPCrack.发现调试器>
```

又是检查标志中是否有`TF`，原理不知道，估计还是反跟踪。然后是

```
004E6481  |.  1B  |vPopReg4 vR12

004E6450  |.  0D  |vPushReg4 vR8   EXIT DWORD v28 = v20

004E6449  |.  48  |vAdd4           EXIT DWORD v28 = 5B5996

004E6446  |.  4E  |vReadMemFs4     v28 = 0FFFFF7FC

004E6443  |.  32  |vWriteMemFs4    DWORD FS:[0] = v184; v184 = 0FFFFF7FC

004E6442  |.  F5  |vPushReg4 vR2   EXIT DWORD v28 = v286

...

004E6424  |.  9D  |vRet            online 5B7421; VMPCrack.005B7421; INT3

005ABC5B  |>  0F  |vPopReg4 vR9

005ABC27  |.  6E  |vWriteMemSs1    BYTE SS:[8 + v20] = v294; BYTE v294 = 1

...

005ABC0D  |.  94  |vJmp_005BED37   goto <VMPCrack.发现调试器>
```

设置异常处理后执行一个联机指令`INT3`，然后直接转到发现调试器了，`INT3`会产生异常，正常的流程应该走异常处理。分析异常处理看看，从`5B5996`开始分析虚拟程序

```
005B46AD  /$  F7  vPopReg4 vR3

005B4690  |.  2B  vReadMemSs4     DWORD s0 = Stack(vESP + 40, ESP + 0C, 4)	//第三个参数，context

005B4683  |.  8C  vReadMemDs4     DWORD m0 = DWORD DS:[s0]			//ContextFlags

005B467E  |.  C7  vNand4          DWORD v0 = AndFlag(m0, 10010)

005B4666  |.  2B  vReadMemSs4     DWORD m1 = DWORD SS:[Je(v0) + 2C]

005B4640  |.  09  vPushReg4 vR7   DWORD v1 = v0

005B4635  |.  06  vJmp_005BED37   if (m0 & 10010 == 0) goto VMPCrack.005B77D4	//10010=CONTEXT_DEBUG_REGISTERS

005B702C  |.  17  vPopReg4 vR11

005B6FEF  |.  67  vReadMemDs4     DWORD m2 = DWORD DS:[18 + s0]			//dr7

005B6FEA  |.  C7  vNand4          DWORD v2 = AndFlag(m2, 0FF)

005B6FDB  |.  2B  vReadMemSs4     DWORD m3 = DWORD SS:[Jnz(v2) + 2C]

005B6FB4  |.  0D  vPushReg4 vR8   DWORD v1 = v2

005B6FA9  |.  06  vJmp_005BED37   if (m2 & 0FF == 0) goto VMPCrack.005B77D4	//检查dr7的低BYTE是否为0

005B783E  |.  27  vPopReg4 vR15

005B780C  |.  8C  vReadMemDs4     DWORD m4 = DWORD DS:[s0 + 0B4]		//ebp

005B7805  |.  B8  vWriteMemDs1    BYTE DS:[m4 + 8] = v3; BYTE v3 = 1		//不为0时[ebp+8]设为1

005B77D5  |.  29  vPushReg4 vR15

005B77D4  |>  13  vPopReg4 vR10

005B77AE  |.  C0  vWriteMemDs4    DWORD DS:[0B8 + s0] = v4; DWORD v4 = 5AD25E	//eip设为5AD25E

...

005B778C  \.  2E  vRet            return Stack(34, 4)
```

先检查`context`中的调试寄存器是否有效，然后检查`dr7`的低`BYTE`是否为0，不为0说明设了硬件断点，`[ebp+8]`设为1，估计以后会检查这个值。把`eip`设成了`5AD25E`，异常处理结束后到这里执行，在`4E6424`添加提示`/*online(5B7421,5AD25E)*/`，告诉插件这个联机指令返回到`5AD25E`，重新反汇编，后面的内容出来了。

```
005AA083  |>  1B  ||vPopReg4 vR12

005AA06B  |.  78  ||vReadMemFs4     v297 = 0FFFFF7FC

005AA06A  |.  20  ||vPopVEsp        vESP = 0FFFFF7FC

005AA068  |.  18  ||vWriteMemFs4    DWORD FS:[0] = v184; v184 = 0FFFFF7FC

005AA053  |.  4A  ||vReadMemSs1     BYTE m42 = BYTE SS:[8 + v20]

...

005A9FF1  |.  94  ||vJmp_005BED37   if (m42 == 1) goto <VMPCrack.发现调试器>

005B644D  |.  07  ||vPopReg4 vR7

005B641D  |.  67  ||vReadMemDs4     DWORD m44 = DWORD DS:[108B3030]

...

005B6397  |.  06  ||vJmp_005BED37   if (GetBytes(m44 - 0E9CD32D5, 0, 1) == 1) goto <VMPCrack.发现调试器>
```

检查`BYTE SS:[8 + v20]`是否为1，看看执行`INT3`的那个`vRet`前`ebp`就是`v20`，刚才的异常处理中如果发现了硬件断点把这里设为1。第二个检查没看出来是什么，地址是`108B3030`，这其实是堆栈里的空间，程序开始的时候加密后保存到了`5BFE27`

```
005B1E15  |.  C0  vWriteMemDs4  DWORD DS:[5BFE27] = v10; EXIT DWORD v10 = 0EF74C7DC
```

就是初始数据中标记为初始化检查的那个，刚才取到的`api`地址都加密后保存在这里。后面有联机指令退出了虚拟机，重新返回时读出了`5BFE27`然后解密，但是程序运行后才会设置这个地址，在分析时是0，所以解密后变成了`108B3030`。这里检查其中的一个数据解密后低`BYTE`是否为1，不记得那里写过这个地址了。然后是

```
005AC35C  |.  1B     ||vPopReg4 vR12

005AC321  |.  67     ||vReadMemDs4      DWORD m46 = DWORD DS:[108B3038]

005AC311  |.  00     ||vNand4           DWORD v298 = m46 - 636A03F3

005AC2EC  |.  CE     ||vReadMemSs4      DWORD m47 = DWORD SS:[Je(AndFlag(v298, v298)) + 0FFFFF800]

005AC2CB  |.  29     ||vPushReg4 vR15   DWORD v299 = m46 - 636A03F3

005AC2B9  |.  06     ||vJmp_005BED37    if (v298 == 0) goto VMPCrack.005AC22F

005B6750  |.  27     ||vPopReg4 vR15

005B6718  |.  4B     ||vReadMemDs1      BYTE m48 = BYTE DS:[v299]

005B66F9  |.  6F     ||vAdd4            DWORD v300 = SubFlag(m48, 0CC)

...

005B66B5  |.  94     ||vJmp_005BED37    if (m48 == 0CC) goto <VMPCrack.发现调试器>

005B778B  |.  EB     ||vPopReg4 vR0

005B7747  |.  2B     ||vReadMemSs4      DWORD m50 = DWORD SS:[Je(v300) + 0FFFFF800]

005B7715  |.  06     ||vJmp_005BED37    if (Je(v300)) goto VMPCrack.005AC22F

005AC294  |.  FF     ||vPopReg4 vR5

005AC26A  |.  5A 00  ||vPushImmSx1 0    ARG4 DWORD v301 = 0

005AC268  |.  5A 00  ||vPushImmSx1 0    ARG3 DWORD v302 = 0

005AC266  |.  12 11  ||vPushImmSx1 11   ARG2 DWORD v303 = 11		//ThreadHideFromDebugger

005AC264  |.  12 FE  ||vPushImmSx1 0FE  ARG1 DWORD v304 = 0FFFFFFFE

005AC261  |.  5E 04  ||vCall 4          v305 = Call(v299)		//ZwSetInformationThread

005AC230  |.  01     ||vPushReg4 vR5

005AC22F  |>  F7     ||vPopReg4 vR3
```

`ZwSetInformationThread`，还是先检查地址是否为0和是否设了断点，但是这里多了一个`if (Je(v300))`，因为标志隔了一个条件转移，没显示出来，`v300`是`SubFlag(m48, 0CC)`，这也是检查断点，检查了两次，第二次发现断点后会跳过`ZwSetInformationThread`，不知道什么意思。然后用`ThreadHideFromDebugger`作为参数调用`ZwSetInformationThread`，使调试器收不到调试事件。然后是
