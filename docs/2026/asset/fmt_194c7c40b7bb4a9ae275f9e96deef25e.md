
```asm
│00601652  |.CALL Farsight.00405080
│00601657  |.PUSH DWORD PTR SS:[EBP-90]
│0060165D  |.LEA EAX,DWORD PTR SS:[EBP-10]
│00601660  |.MOV EDX,5                                ; EDX = 5
│00601665  |.CALL Farsight.00405218
│0060166A  |.PUSH 0
│0060166C  |.LEA EAX,DWORD PTR SS:[EBP-94]
│00601672  |.MOV EDX,DWORD PTR SS:[EBP-8]
│00601675  |.MOV DL,BYTE PTR DS:[EDX+8]               ; DL = 注册码第三个四位的第一位
│00601678  |.CALL Farsight.00405080
│0060167D  |.PUSH DWORD PTR SS:[EBP-94]
│00601683  |.LEA EAX,DWORD PTR SS:[EBP-98]
│00601689  |.MOV EDX,DWORD PTR SS:[EBP-8]
│0060168C  |.MOV DL,BYTE PTR DS:[EDX+9]               ; DL = 注册码第三个四位的第二位
│0060168F  |.CALL Farsight.00405080
│00601694  |.PUSH DWORD PTR SS:[EBP-98]
│0060169A  |.LEA EAX,DWORD PTR SS:[EBP-9C]
│006016A0  |.MOV EDX,DWORD PTR SS:[EBP-8]
│006016A3  |.MOV DL,BYTE PTR DS:[EDX+A]               ; DL = 注册码第三个四位的第三位
│006016A6  |.CALL Farsight.00405080
│006016AB  |.PUSH DWORD PTR SS:[EBP-9C]
│006016B1  |.LEA EAX,DWORD PTR SS:[EBP-A0]
│006016B7  |.MOV EDX,DWORD PTR SS:[EBP-8]
│006016BA  |.MOV DL,BYTE PTR DS:[EDX+B]               ; DL = 注册码第三个四位的第四位
│006016BD  |.CALL Farsight.00405080
│006016C2  |.PUSH DWORD PTR SS:[EBP-A0]
│006016C8  |.LEA EAX,DWORD PTR SS:[EBP-14]
│006016CB  |.MOV EDX,5                                ; EDX = 5
│006016D0  |.CALL Farsight.00405218
│006016D5  |.LEA EDX,DWORD PTR SS:[EBP-A4]
│006016DB  |.MOV EAX,DWORD PTR SS:[EBP-10]
│006016DE  |.CALL Farsight.00409634                   ; 转换到大写字母
│006016E3  |.MOV EAX,DWORD PTR SS:[EBP-A4]
│006016E9  |.PUSH EAX
│006016EA  |.LEA EDX,DWORD PTR SS:[EBP-A8]
│006016F0  |.MOV EAX,DWORD PTR SS:[EBP-14]
│006016F3  |.CALL Farsight.00409634
│006016F8  |.MOV EDX,DWORD PTR SS:[EBP-A8]
│006016FE  |.POP EAX
───────────────────────────────────────────────
│006016FF  |.CALL Farsight.004052A4                   ; 末四位比较
───────────────────────────────────────────────
│00601704  |.JNZ SHORT Farsight.0060170A              ; 一跳就死
│00601706  |.MOV BL,1
│00601708  |.JMP SHORT Farsight.0060170C
│0060170A  |>XOR EBX,EBX
│0060170C  |>XOR EAX,EAX
│0060170E  |.POP EDX
│0060170F  |.POP ECX
│00601710  |.POP ECX
│00601711  |.MOV DWORD PTR FS:[EAX],EDX
│00601714  |.PUSH Farsight.0060173E
│00601719  |>LEA EAX,DWORD PTR SS:[EBP-A8]
│0060171F  |.MOV EDX,21
│00601724  |.CALL Farsight.00404EBC
│00601729  |.LEA EAX,DWORD PTR SS:[EBP-14]
│0060172C  |.MOV EDX,5
│00601731  |.CALL Farsight.00404EBC
│00601736  \.RETN
└──────────────────────────────────────────────┘
```

这段代码比较长，但结构其实很清晰。在 F8 步进的过程中，观察 OD 的寄存器窗口，不难看出其端倪。其注册过程为：

将输入的注册名作为种子（我这里是 `"WSLVIC"`）。

第一次时，将 `"WSLVIC"` 与 `"calc"` 连接起来成为 `"WSLVICcalc"`，用它来算第一个特征序列，我这里是：

- 特征序列 1：`f8d5ca7bdde057179eb023f600477990`

第二次时，将 `"WSLVIC"` 与 `"spring"` 连接起来成为 `"WSLVICspring"`，用它来算第二个特征序列，我这里是：

- 特征序列 2：`b41c33d1503c68d53e2152bdd0ad90e6`

第三次时，将 `"WSLVIC"` 与 `"fly"` 连接起来成为 `"WSLVICfly"`，用它来算第三个特征序列，我这里是：

- 特征序列 3：`e087b63270b3147c4e2ee9290c695410`

而注册码为每个特征序列的前四位相连接的结果，如：

- 特征序列 1 的前四位：`f8d5`
- 特征序列 2 的前四位：`b41c`
- 特征序列 3 的前四位：`e087`

所以当注册名为 `"WSLVIC"` 时，注册码就是：

`RegCode = f8d5b41ce087`

当然，如果你输入的注册名不是 `"WSLVIC"`，那么 `RegCode` 就不是 `"f8d5b41ce087"` 了。用这个注册名及注册码来注册 FarSight Calculator v2.2，可以发现在关于对话框中软件已经注册给 `"WSLVIC"` 了。

这里其实有个问题，就是 FarSight Calculator v2.2 如何用注册名来生成特征序列？关于这个问题，需要提问者自己解决，因为我根本就没有去追，原因是与我写这篇破文毫无关系。“算法是千变万化的，但你总需要一个结果”，所以无论 FarSight Calculator v2.2 是如何得到的这 3 个特征序列的，注册码却总是它们前四位的和！

但如果你要写一个 KeyGen，那么追踪这 3 个特征序列的生成算法，就是必须要做的事了。有兴趣的人自己试试吧！而下面的 FarSight Calculator v3.6 的注册中依然会计算特征序列，和这里一样——将不作处理。这里先做一个预告。如果有一天，如果我“良心”发现，或许会追一下它的生成算法，然后写一个注册机，呵呵，有这么一天吗？

“如果有一天，我悄然离去，请把我埋在这春天里……”

至此，FarSight Calculator v2.2 的注册算法已基本清晰，下面就要开始 FarSight Calculator v3.6 的注册探秘了。希望大家不要烦哪，破文已经写得老长了，我也想打退堂鼓……

## FarSight Calculator v3.6 注册算法探秘

与 v2.2 版一样，注册失败对话框还是那几个字 `"Invalid Registered User or Key!"`。用同样的方法找到了地址 `60ABDC`（3 个 Push 的地方），F8 单步之后，发现与 v2.2 版几乎完全一致，这就是所谓的“血继”吧。

```asm
┌──────────────────────────────────────────────┐
│0060ABDC   .PUSH EBP
│0060ABDD   .PUSH Farsight.0060AC50
│0060ABE2   .PUSH DWORD PTR FS:[EAX]
│0060ABE5   .MOV DWORD PTR FS:[EAX],ESP
│0060ABE8   .LEA EDX,DWORD PTR SS:[EBP-4]
│0060ABEB   .MOV EAX,DWORD PTR DS:[EBX+300]
```
