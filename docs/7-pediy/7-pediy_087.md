
# Photo Watermark

> 原文：[`www.pediy.com/kssd/pediy05/pediy50232.htm`](https://www.pediy.com/kssd/pediy05/pediy50232.htm)

`Photo Watermark` 破解 (`SMC`)

`Photo Watermark` 是我最近从网上下载的一个给图片加水印的软件。加的壳不难，是 `UPX` modified 的壳。在 `oep` 处 `makepe` 出来居然不能用，`dump` 出来还要修复导入表，实在想不到一个比较简单的办法，最后想到了用 `SMC`。

注册很简单

```
0187:0054D6B5 E87AC3FCFF       CALL    00519A34     //注册码判断
0187:0054D6BA 85C0             TEST    EAX,EAX      //eax 是否为 1
0187:0054D6BC 7572             JNZ     0054D730
0187:0054D6BE A1001D5600       MOV     EAX,[00561D00]
0187:0054D6C3 8B00             MOV     EAX,[EAX]
0187:0054D6C5 BA6CD85400       MOV     EDX,0054D86C
0187:0054D6CA E88D73EBFF       CALL    00404A5C
0187:0054D6CF 0F85DC000000     JNZ     NEAR 0054D7B1
0187:0054D6D5 686CD85400       PUSH    DWORD 0054D86C
0187:0054D6DA 8D45DC           LEA     EAX,[EBP-24]
0187:0054D6DD 50               PUSH    EAX
0187:0054D6DE 8D55D8           LEA     EDX,[EBP-28]
```

进入 `Call` 看看去

```
:00519A34 55                   push ebp     //好办,这里修改为 mov  eax,1  ret 不就好了吗?
:00519A35 8BEC                 mov ebp, esp
:00519A37 83C4F0               add esp, FFFFFFF0
:00519A3A 53                   push ebx
:00519A3B 33C9                 xor ecx, ecx
:00519A3D 894DF4               mov dword ptr [ebp-0C], ecx
:00519A40 894DF0               mov dword ptr [ebp-10], ecx
:00519A43 8955F8               mov dword ptr [ebp-08], edx
:00519A46 8945FC               mov dword ptr [ebp-04], eax
:00519A49 8B45FC               mov eax, dword ptr [ebp-04]
```

关键是如何 `SMC`

`ok`，跟踪它的壳吧

```
0187:005AF5ED 8903             MOV     [EBX],EAX
0187:005AF5EF 83C304           ADD     EBX,BYTE +04
0187:005AF5F2 EBE1             JMP     SHORT 005AF5D5
0187:005AF5F4 FF9654281B00     CALL    NEAR [ESI+001B2854]
0187:005AF5FA 60               PUSHA
0187:005AF5FB E9BCB9FAFF       JMP     0055AFBC     //这里跳 OEP,是 UPX 常见的手段,我就在这里做文章吧,看看下面全部是 00,真好!:)
0187:005AF600 18F6             SBB     DH,DH
0187:005AF602 5A               POP     EDX
0187:005AF603 0028             ADD     [EAX],CH
0187:005AF605 F65A00           NEG     BYTE [EDX+00]
0187:005AF608 D434             AAM     34
0187:005AF60A 56               PUSH    ESI
0187:005AF60B 0000             ADD     [EAX],AL
0187:005AF60D 0000             ADD     [EAX],AL
```

就在 `trw` 里修改吧

```
0187:005AF5ED 8903             MOV     [EBX],EAX
0187:005AF5EF 83C304           ADD     EBX,BYTE +04
0187:005AF5F2 EBE1             JMP     SHORT 005AF5D5
0187:005AF5F4 FF9654281B00     CALL    NEAR [ESI+001B2854]
0187:005AF5FA 60               PUSHA
0187:005AF5FB E931000000       JMP     005AF631     //修改了.
0187:005AF600 18F6             SBB     DH,DH
0187:005AF602 5A               POP     EDX
0187:005AF603 0028             ADD     [EAX],CH
0187:005AF605 F65A00           NEG     BYTE [EDX+00]
0187:005AF608 D434             AAM     34
0187:005AF60A 56               PUSH    ESI
0187:005AF60B 0000             ADD     [EAX],AL
0187:005AF60D 0000             ADD     [EAX],AL
0187:005AF60F 0000             ADD     [EAX],AL
......
0187:005AF621 0000             ADD     [EAX],AL
0187:005AF623 0000             ADD     [EAX],AL
0187:005AF625 0000             ADD     [EAX],AL
0187:005AF627 0000             ADD     [EAX],AL
0187:005AF629 0000             ADD     [EAX],AL
0187:005AF62B 0000             ADD     [EAX],AL
0187:005AF62D 0000             ADD     [EAX],AL
0187:005AF62F 0000             ADD     [EAX],AL
0187:005AF631 C705349A5100B8010000 MOV  DWORD [00519A34],01B8   //SMC 就在这里
0187:005AF63B C705389A510000C39000 MOV  DWORD [00519A38],0090C300
0187:005AF645 E972B9FAFF       JMP     0055AFBC       //再跳 OEP
0187:005AF64A 0000             ADD     [EAX],AL
0187:005AF64C 0000             ADD     [EAX],AL
0187:005AF64E 0000             ADD     [EAX],AL
0187:005AF650 0000             ADD     [EAX],AL
```

运行修改后的程序看看，成功了。

后记：其实做 `SMC` 还要注意段是否可写，如果不可写......，当然了 `upx` 加壳的还不用考虑这个。 `4dd466d0e1e0cc123afbe504da2462ca`
