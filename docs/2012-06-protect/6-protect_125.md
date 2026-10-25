
# MSTCAD 空间网格结构设计软件

> 原文：[`www.pediy.com/kssd/pediy06/pediy6818.htm`](https://www.pediy.com/kssd/pediy06/pediy6818.htm)

- **破解目标**：MSTCAD 空间网格结构设计软件  
- **工具**：`W32DASM`，`fi3.01`，`UltraEdit8.0`，`ollydbg1.09d`  
- **目的**：狗不理，不理狗  

### 软件简介
空间网格结构近二十几年来已经得到了广泛的应用和发展，这一方面是由于这种结构具有空间受力特性、建筑造型丰富、重量轻、材料省、产品工厂化、施工安装方便、工程质量高、工期短等优点，另一方面，计算机的广泛应用和普及、计算技术的渐趋成熟、软件的不断研制和开发也为空间网格结构的应用和发展奠定了基础。

该软件可在其主页公开下载，不须带狗即可安装，但须带狗运行。其帮助中说明“设计版主要进行施工图设计，企业版包括设计版所有内容外，还针对加工制作需要，进行节点翻样、统计计算等。专业版.....”

### 开工
先用 `fi3.01` 检查没有加壳，再看安装帮助文件，发现是升级版，就是说还是原来的老狗（见我的上篇破文看雪论坛精华 5 里有）。

软件在运行时，如果没有狗，则会跳出一个需要注册的对话框“没有找到加密器。”，和原来一样，怎么还没有改变啊。那就先用 `W32dsm` 看看，反编译成功后，在串式参考查找出错的信息“没有找到加密器。”，

找到下面：

```text
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A60B5(U)
|
:004A5EE0 6AFF                    push FFFFFFFF
:004A5EE2 683B495300             push 0053493B
:004A5EE7 64A100000000           mov eax, dword ptr fs:[00000000]
:004A5EED 50                     push eax
:004A5EEE 64892500000000         mov dword ptr fs:[00000000], esp
:004A5EF5 81EC04010000          sub esp, 00000104
:004A5EFB 56                     push esi
:004A5EFC 57                     push edi
:004A5EFD 33FF                   xor edi, edi
:004A5EFF 8BF1                   mov esi, ecx
:004A5F01 57                     push edi
:004A5F02 8974240C              mov dword ptr [esp+0C], esi
:004A5F06 E885D40700            call 00523390
:004A5F0B 8D44240C              lea eax, dword ptr [esp+0C]
:004A5F0F 89BC2414010000        mov dword ptr [esp+00000114], edi
:004A5F16 50                     push eax
:004A5F17 6800010000            push 00000100
:004A5F1C C706F06F5400           mov dword ptr [esi], 00546FF0

* Reference To: KERNEL32.GetCurrentDirectoryA, Ord:00F5h
                                      |
:004A5F22 FF15A8A35300           Call dword ptr [0053A3A8]
:004A5F28 8D4C240C              lea ecx, dword ptr [esp+0C]
:004A5F2C 51                     push ecx

* Possible StringData Ref from Data Obj ->"%s"
                                      |
:004A5F2D 6824C45600            push 0056C424
:004A5F32 68E8E94C03            push 034CE9E8
:004A5F37 E8C22B0600            call 00508AFE
:004A5F3C 83C40C                add esp, 0000000C
:004A5F3F 8BCE                   mov ecx, esi
:004A5F41 E8CA080000            call 004A6810
:004A5F46 83F801                cmp eax, 00000001
:004A5F49 0F85CA000000          jne 004A6019
:004A5F4F 8986C0D59201          mov dword ptr [esi+0192D5C0], eax
:004A5F55 C705B4EA4C0302000000 mov dword ptr [034CEAB4], 00000002

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6062(C)   ==== > 成功，来到这里，前提是上面的
|
:004A5F5F 8BCE                   mov ecx, esi
:004A5F61 E86A040000            call 004A63D0
:004A5F66 B9C0C0C000            mov ecx, 00C0C0C0
:004A5F6B B880808000            mov eax, 00808080
:004A5F70 890DDC321102          mov dword ptr [021132DC], ecx
:004A5F76 890DE0321102          mov dword ptr [021132E0], ecx
:004A5F7C 890DEC321102          mov dword ptr [021132EC], ecx
:004A5F82 890DF0321102          mov dword ptr [021132F0], ecx
:004A5F88 8B8C240C010000        mov ecx, dword ptr [esp+0000010C]
:004A5F8F 893DB4321102          mov dword ptr [021132B4], edi
:004A5F95 A3B8321102             mov dword ptr [021132B8], eax
:004A5F9A A3BC321102             mov dword ptr [021132BC], eax
:004A5F9F A3C0321102             mov dword ptr [021132C0], eax
:004A5FA4 893DCC321102          mov dword ptr [021132CC], edi
:004A5FAA 893DC8321102          mov dword ptr [021132C8], edi
:004A5FB0 893DD4321102          mov dword ptr [021132D4], edi
:004A5FB6 A3D8321102             mov dword ptr [021132D8], eax
:004A5FBB 893DE4321102          mov dword ptr [021132E4], edi
:004A5FC1 A3E8321102             mov dword ptr [021132E8], eax
:004A5FC6 A3F4321102             mov dword ptr [021132F4], eax
:004A5FCB A3F8321102             mov dword ptr [021132F8], eax
:004A5FD0 A3FC321102             mov dword ptr [021132FC], eax
:004A5FD5 8BC6                   mov eax, esi
:004A5FD7 5F                     pop edi
:004A5FD8 C705A8321102400D0300   mov dword ptr [021132A8], 00030D40
:004A5FE2 C705AC3211020A000000   mov dword ptr [021132AC], 0000000A
:004A5FEC C705B032110201000000   mov dword ptr [021132B0], 00000001
:004A5FF6 C705C432110201000000   mov dword ptr [021132C4], 00000001
:004A6000 C705D032110202000000   mov dword ptr [021132D0], 00000002
:004A600A 5E                     pop esi
:004A600B 64890D00000000         mov dword ptr fs:[00000000], ecx
:004A6012 81C410010000          add esp, 00000110
:004A6018 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A5F49(C)
|
:004A6019 8BCE                   mov ecx, esi
:004A601B 89BEC0D59201          mov dword ptr [esi+0192D5C0], edi
:004A6021 E85A050000            call 004A6580   ==== > 检查狗的 call，进去看看，哈哈，花指令好多，可是有什么用？！
:004A6026 85C0                   test eax, eax
:004A6028 7512                   jne 004A603C   ==== > 检测狗成功，就跳了 004A603C
:004A602A 57                     push edi
:004A602B 57                     push edi

* Possible StringData Ref from Data Obj ->"没有找到加密器。"   ----> 就是这里啦！没变化啊。
  ;; 找到这不难，难在不能走弯路，那我么这次要好好看看了。                      |
:004A602C 68A8205700            push 005720A8
:004A6031 E87A0C0700            call 00516CB0
:004A6036 57                     push edi
:004A6037 E8943B0500            call 004F9BD0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6028(C)
|
:004A603C 8BCE                   mov ecx, esi    ==== > 检测狗成功到这，还没有完呢。
:004A603E E85D050000            call 004A65A0   ==== > 后面的 call，里面还有检测，其中还要查狗，确定是设计，企业版。进去看看吧
:004A6043 85C0                   test eax, eax
:004A6045 7512                   jne 004A6059
:004A6047 57                     push edi
:004A6048 57                     push edi

* Possible StringData Ref from Data Obj ->"非合法用户，软件无法使用。"====> 到这里就玩完！
                                      |
:004A6049 688C205700            push 0057208C
:004A604E E85D0C0700            call 00516CB0
:004A6053 57                     push edi
:004A6054 E8773B0500            call 004F9BD0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A6045(C)
|
:004A6059 8BCE                   mov ecx, esi
:004A605B E8A0090000            call 004A6A00   ==== > 检测时间。在 2003-2004 之间能用
:004A6060 85C0                   test eax, eax
:004A6062 0F85F7FEFFFF          jne 004A5F5F   ==== > 成功跳
:004A6068 57                     push edi
:004A6069 E8623B0500            call 004F9BD0
:004A606E 90                     nop
:004A606F 90                     nop
:004A6070 56                     push esi
:004A6071 8BF1                   mov esi, ecx
:004A6073 E818000000            call 004A6090
:004A6078 F644240801             test [esp+08], 01
:004A607D 7409                   je 004A6088
:004A607F 56                     push esi
:004A6080 E8E04E0600            call 0050AF65
:004A6085 83C404                add esp, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A607D(C)
|
:004A6088 8BC6                   mov eax, esi
:004A608A 5E                     pop esi
:004A608B C20400                 ret 0004
:004A608E 90                     nop
:004A608F 90                     nop

* Referenced by a CALL at Address:
|:004A6073    
|

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A60D5(U)
|
```



```text
:004A6090 E92CD70700             jmp 005237C1  

:004A6095 90                     nop    ==== >看到这些 `90` 吗，这次他们可能忘了放花指令了，留下这些空。不过，这些花指令其实也没用，因为他前面自己留下了漏洞。    
  
:004A6096 90                     nop  

:004A6097 90                     nop  

:004A6098 90                     nop  

:004A6099 90                     nop  

:004A609A 90                     nop  

:004A609B 90                     nop  

:004A609C 90                     nop  

:004A609D 90                     nop  

:004A609E 90                     nop  

:004A609F 90                     nop  

:004A60A0 E80B000000             call 004A60B0  

:004A60A5 E916000000             jmp 004A60C0  

:004A60AA 90                     nop  

:004A60AB 90                     nop  

:004A60AC 90                     nop  

:004A60AD 90                     nop  

:004A60AE 90                     nop  

:004A60AF 90                     nop  
```

* Referenced by a `CALL` at Address:

| `:004A60A0`
|

```text
:004A60B0 B9C0965700             mov ecx, 005796C0  

:004A60B5 E926FEFFFF             jmp 004A5EE0  

:004A60BA 90                     nop  

:004A60BB 90                     nop  

:004A60BC 90                     nop  

:004A60BD 90                     nop  

:004A60BE 90                     nop  

:004A60BF 90                     nop  
```

* Referenced by a (U)nconditional or (C)onditional `Jump` at Address:

| `:004A60A5` (U)
|

```text
:004A60C0 68D0604A00             push 004A60D0  

:004A60C5 E80F340500             call 004F94D9  

:004A60CA 59                     pop ecx  

:004A60CB C3                     ret  

:004A60CC 90                     nop  

:004A60CD 90                     nop  

:004A60CE 90                     nop  

:004A60CF 90                     nop  

:004A60D0 B9C0965700             mov ecx, 005796C0  

:004A60D5 E9B6FFFFFF             jmp 004A6090  

:004A60DA 90                     nop  

:004A60DB 90                     nop  

:004A60DC 90                     nop  

:004A60DD 90                     nop  

:004A60DE 90                     nop  

:004A60DF 90                     nop  

:004A60E0 6AFF                   push FFFFFFFF  

:004A60E2 686A495300             push 0053496A
```

................省略

以下是 `"非合法用户，软件无法使用。"` 的 `call`，我们看看

* Referenced by a `CALL` at Addresses:

| `:004A603E` , `:004B5AC4` 我们看到的 `call`，有 2 个，就是第 2 个再作怪，我没开始注意到，当然也是因为我不是土建专业，不会用这个软件，所以没有发现错误。另外还有 2 处，`call 004A6580`，`call 004A6810` 也是一样，都是关于狗。当然破解方法也不同了。

在此感谢发现补丁 `bug` 的同志提醒，谢谢。

|

```text
:004A65A0 6AFF                   push FFFFFFFF  

:004A65A2 68184A5300             push 00534A18  

:004A65A7 64A100000000           mov eax, dword ptr fs:[00000000]  

:004A65AD 50                     push eax  

:004A65AE 64892500000000         mov dword ptr fs:[00000000], esp  

:004A65B5 83EC10                 sub esp, 00000010  

:004A65B8 53                     push ebx  

:004A65B9 56                     push esi  

:004A65BA 33DB                   xor ebx, ebx  ==== >ebx = 0  

:004A65BC 8D44240C               lea eax, dword ptr [esp+0C]  

:004A65C0 8BF1                   mov esi, ecx  

:004A65C2 C70518EA4C032A030000   mov dword ptr [034CEA18], 0000032A  

:004A65CC 881D10EA4C03           mov byte ptr [034CEA10], bl  

:004A65D2 A30CEA4C03             mov dword ptr [034CEA0C], eax  

:004A65D7 66C7051CEA4C034D00     mov word ptr [034CEA1C], 004D  

:004A65E0 66C7051EEA4C030800     mov word ptr [034CEA1E], 0008  

:004A65E9 885C2414               mov byte ptr [esp+14], bl  

:004A65ED E87E16F6FF             call 00407C70  ==== >！就是这，进去又查狗  

:004A65F2 3BC3                   cmp eax, ebx ==== >！比较 `eax`，`ebx`，不等就跳  

:004A65F4 0F857A010000           jne 004A6774  ==== >一跳就玩完了  

:004A65FA 8B0D545E5700           mov ecx, dword ptr [00575E54]  

:004A6600 894C2408               mov dword ptr [esp+08], ecx  

:004A6604 8D54240C               lea edx, dword ptr [esp+0C]  

:004A6608 8D442408               lea eax, dword ptr [esp+08]  

:004A660C 52                     push edx
```

* Possible `StringData` Ref from `Data` Obj -> `"%s"`

|

```text
:004A660D 6824C45600             push 0056C424  

:004A6612 50                     push eax  

:004A6613 895C242C               mov dword ptr [esp+2C], ebx  

:004A6617 E8E2240600             call 00508AFE  

:004A661C 83C40C                 add esp, 0000000C  

:004A661F 8D4C2408               lea ecx, dword ptr [esp+08]  

:004A6623 53                     push ebx
```

    ==== >以下就是检测学习版，设计版，企业版的

* Possible `StringData` Ref from `Data` Obj -> `"Luo98202"`

|

```text
:004A6624 683C215700             push 0057213C  

:004A6629 E89D210600             call 005087CB  

:004A662E 85C0                   test eax, eax  

:004A6630 0F8D2D010000           jnl 004A6763  

:004A6636 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo98437"`

|

```text
:004A6637 6830215700             push 00572130  

:004A663C 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A6640 E886210600             call 005087CB  

:004A6645 85C0                   test eax, eax  

:004A6647 0F8D16010000           jnl 004A6763  

:004A664D 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo98"`

|

```text
:004A664E 6828215700             push 00572128  

:004A6653 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A6657 E86F210600             call 005087CB  

:004A665C 85C0                   test eax, eax  

:004A665E 7D17                   jge 004A6677  

:004A6660 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo01"`

|

```text
:004A6661 6820215700             push 00572120  

:004A6666 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A666A E85C210600             call 005087CB  

:004A666F 85C0                   test eax, eax  

:004A6671 0F8CEC000000           jl 004A6763
```

* Referenced by a (U)nconditional or (C)onditional `Jump` at Address:

| `:004A665E` (C)
|

```text
:004A6677 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo984"`

|

```text
:004A6678 6818215700             push 00572118  

:004A667D 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A6681 E845210600             call 005087CB  

:004A6686 85C0                   test eax, eax  

:004A6688 0F8D9A000000           jnl 004A6728  

:004A668E 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo985"`

|

```text
:004A668F 6810215700             push 00572110  

:004A6694 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A6698 E82E210600             call 005087CB  

:004A669D 85C0                   test eax, eax  

:004A669F 0F8D83000000           jnl 004A6728  

:004A66A5 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo014"`

|

```text
:004A66A6 6808215700             push 00572108  

:004A66AB 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A66AF E817210600             call 005087CB  

:004A66B4 85C0                   test eax, eax  

:004A66B6 7D70                   jge 004A6728  

:004A66B8 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo01395"`

|

```text
:004A66B9 68FC205700             push 005720FC  

:004A66BE 8D4C2410               lea ecx, dword ptr [esp+10]  

:004A66C2 E804210600             call 005087CB  

:004A66C7 85C0                   test eax, eax  

:004A66C9 7D5D                   jge 004A6728  

:004A66CB 53                     push ebx
```

* Possible `StringData` Ref from `Data` Obj -> `"Luo982"`

|



## 汇编代码分析与注释

以下是详细的汇编代码段，包含内存地址、机器码、指令及中文注释。

```
:004A66CC 68F4205700              push 005720F4

:004A66D1 8D4C2410               lea ecx, dword ptr [esp+10]

:004A66D5 E8F1200600             call 005087CB

:004A66DA 85C0                   test eax, eax

:004A66DC 7D13                   jge 004A66F1

:004A66DE 53                     push ebx

* Possible StringData Ref from Data Obj ->"Luo012"

                    |

:004A66DF 68EC205700             push 005720EC

:004A66E4 8D4C2410               lea ecx, dword ptr [esp+10]

:004A66E8 E8DE200600             call 005087CB

:004A66ED 85C0                   test eax, eax

:004A66EF 7C72                   jl 004A6763

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A66DC(C)

|       ====>跳到这里就是 MST 2003(设计版)

:004A66F1 C705B4EA4C0301000000   mov dword ptr [034CEAB4], 00000001 ====>让 dword ptr [034CEAB4]等于 2

:004A66FB 8D4C2408               lea ecx, dword ptr [esp+08]

:004A66FF 899EC0D59201           mov dword ptr [esi+0192D5C0], ebx  ====>让 dword ptr [esi+0192D5C0]等于 ebx，估计是 0

:004A6705 C7442420FFFFFFFF       mov [esp+20], FFFFFFFF

:004A670D E8EE860600             call 0050EE00

:004A6712 5E                     pop esi

:004A6713 B801000000             mov eax, 00000001

:004A6718 5B                     pop ebx

:004A6719 8B4C2410               mov ecx, dword ptr [esp+10]

:004A671D 64890D00000000         mov dword ptr fs:[00000000], ecx

:004A6724 83C41C                 add esp, 0000001C

:004A6727 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A6688(C), :004A669F(C), :004A66B6(C), :004A66C9(C)

|       ====>跳到这里就是 MST 2003(企业版)

:004A6728 C705B4EA4C0302000000   mov dword ptr [034CEAB4], 00000002      ====>让 dword ptr [034CEAB4]等于 2

:004A6732 8D4C2408               lea ecx, dword ptr [esp+08]

:004A6736 C786C0D5920101000000   mov dword ptr [esi+0192D5C0], 00000001  ====>让 dword ptr [esi+0192D5C0]等于 1

:004A6740 C7442420FFFFFFFF       mov [esp+20], FFFFFFFF

:004A6748 E8B3860600             call 0050EE00

:004A674D 5E                     pop esi

:004A674E B801000000             mov eax, 00000001

:004A6753 5B                     pop ebx

:004A6754 8B4C2410               mov ecx, dword ptr [esp+10]

:004A6758 64890D00000000         mov dword ptr fs:[00000000], ecx

:004A675F 83C41C                 add esp, 0000001C

:004A6762 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A6630(C), :004A6647(C), :004A6671(C), :004A66EF(C)

|        ====>跳到这里就是 MST 2003(学习版) 

:004A6763 8D4C2408               lea ecx, dword ptr [esp+08]  ====>什么不作，dword ptr [034CEAB4]，dword ptr [esi+0192D5C0]等于初始值，当然为 0！！！

:004A6767 C7442420FFFFFFFF       mov [esp+20], FFFFFFFF

:004A676F E88C860600             call 0050EE00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A65F4(C)

| ====>一跳到这里就玩完了

:004A6774 8B4C2418               mov ecx, dword ptr [esp+18]

:004A6778 5E                     pop esi

:004A6779 33C0                   xor eax, eax

:004A677B 5B                     pop ebx

:004A677C 64890D00000000         mov dword ptr fs:[00000000], ecx

:004A6783 83C41C                 add esp, 0000001C

:004A6786 C3                     ret

:004A6787 90                     nop

:004A6788 90                     nop

:004A6789 90                     nop

:004A678A 90                     nop

:004A678B 90                     nop

:004A678C 90                     nop

:004A678D 90                     nop

:004A678E 90                     nop

:004A678F 90                     nop

:004A6790 8BC1                   mov eax, ecx

:004A6792 8B4C2404               mov ecx, dword ptr [esp+04]

:004A6796 8B80C0D59201           mov eax, dword ptr [eax+0192D5C0]

:004A679C 8B11                   mov edx, dword ptr [ecx]

:004A679E 50                     push eax

:004A679F FF12                   call dword ptr [edx]

:004A67A1 C20400                 ret 0004

:004A67A4 90                     nop

:004A67A5 90                     nop

:004A67A6 90                     nop

:004A67A7 90                     nop

:004A67A8 90                     nop

:004A67A9 90                     nop

:004A67AA 90                     nop

:004A67AB 90                     nop

:004A67AC 90                     nop

:004A67AD 90                     nop

:004A67AE 90                     nop

:004A67AF 90                     nop

:004A67B0 56                     push esi

:004A67B1 8BF1                   mov esi, ecx

:004A67B3 E8A0970600             call 0050FF58

:004A67B8 A1B4EA4C03             mov eax, dword ptr [034CEAB4]

:004A67BD 85C0                   test eax, eax

:004A67BF 750D                   jne 004A67CE

* Possible StringData Ref from Data Obj ->"MST 2003(学习版)"   ====>！就是这，往下看！！！！！

                    |

:004A67C1 6870215700             push 00572170

:004A67C6 8D4E5C                 lea ecx, dword ptr [esi+5C]

:004A67C9 E8BB870600             call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A67BF(C)

|

:004A67CE 833DB4EA4C0301         cmp dword ptr [034CEAB4], 00000001

:004A67D5 750D                   jne 004A67E4

* Possible StringData Ref from Data Obj ->"MST 2003(设计版)"

                    |

:004A67D7 685C215700             push 0057215C

:004A67DC 8D4E5C                 lea ecx, dword ptr [esi+5C]

:004A67DF E8A5870600             call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A67D5(C)

|

:004A67E4 833DB4EA4C0302         cmp dword ptr [034CEAB4], 00000002

:004A67EB 750D                   jne 004A67FA

* Possible StringData Ref from Data Obj ->"MST 2003(企业版)"

                    |

:004A67ED 6848215700             push 00572148

:004A67F2 8D4E5C                 lea ecx, dword ptr [esi+5C]

:004A67F5 E88F870600             call 0050EF89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A67EB(C)

|

:004A67FA 6A00                   push 00000000

:004A67FC 8BCE                   mov ecx, esi

:004A67FE E806700600             call 0050D809

:004A6803 B801000000             mov eax, 00000001

:004A6808 5E                     pop esi

:004A6809 C3                     ret
```

### 时间检测逻辑

以下是检测时间的 `call`，我们看看

```
* Referenced by a CALL at Address:

|:004A605B   

|

:004A6A00 83EC08                 sub esp, 00000008

:004A6A03 8D442404               lea eax, dword ptr [esp+04]

:004A6A07 50                     push eax

:004A6A08 E8EB2B0500             call 004F95F8  ====>获得系统时间的 call

:004A6A0D 8B4C2408               mov ecx, dword ptr [esp+08]

:004A6A11 83C404                 add esp, 00000004

:004A6A14 894C2400               mov dword ptr [esp], ecx

:004A6A18 8D4C2400               lea ecx, dword ptr [esp]

:004A6A1C 6A00                   push 00000000

:004A6A1E E854280600             call 00509277

:004A6A23 6A00                   push 00000000

:004A6A25 8D4C2404               lea ecx, dword ptr [esp+04]

:004A6A29 E849280600             call 00509277

:004A6A2E 6A00                   push 00000000

:004A6A30 8D4C2404               lea ecx, dword ptr [esp+04]

:004A6A34 E83E280600             call 00509277

:004A6A39 8B4014                 mov eax, dword ptr [eax+14]

:004A6A3C 056C070000             add eax, 0000076C

:004A6A41 3DD4070000             cmp eax, 000007D4   ====>7D4 十进制为 2004

:004A6A46 7E14                   jle 004A6A5C  ====>不大于 2004 年跳下去
```

.....省略



### 汇编代码分析

```
:004A6A48 6A00                 push 00000000  
:004A6A4A 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"您的软件应该升级了.请到[www.mstcenter.com 下载](http://www.mstcenter.com 下载)!"  
                     |  
:004A6A4C 6804225700           push 00572204  
:004A6A51 E85A020700           call 00516CB0  
:004A6A56 33C0                 xor eax, eax  
:004A6A58 83C408               add esp, 00000008  
:004A6A5B C3                   ret  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004A6A46(C)  
|  
:004A6A5C 3DD3070000           cmp eax, 000007D3  ====>7D3 十进制为 2003  
:004A6A61 7E0E                 jle 004A6A71  ====>不大于 2003 年跳下去,我觉得这是作者的笔误，应该是大于跳，否则没意义了。  
:004A6A63 6A00                 push 00000000  
:004A6A65 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"您的软件应该升级了.请到[www.mstcenter.com 下载](http://www.mstcenter.com 下载)!"  
                     |  
:004A6A67 6804225700           push 00572204  
:004A6A6C E83F020700           call 00516CB0  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004A6A61(C)  
|  
:004A6A71 B801000000           mov eax, 00000001  ====>成功标志，eax＝1  
:004A6A76 83C408               add esp, 00000008  
:004A6A79 C3                   ret  
```

我们分析了以上，就明白，要获得企业版，那末就要检测狗成功并{让 `dword ptr [034CEAB4]` 等于 2，让 `dword ptr [esi+0192D5C0]` 等于 1}，系统时间在 2003-2004 之间，当然现在小于 2004 就行。

这时我们在回头看看，发现这和我的上篇破文【看雪论坛精华 5】一样啊，对，上次我是初学破解，很多不明白，现在我好像又更深一步了。

我们再来看看最前面的

```
* Possible StringData Ref from Data Obj ->"%s"  
                     |  
:004A5F2D 6824C45600           push 0056C424  
:004A5F32 68E8E94C03           push 034CE9E8  
:004A5F37 E8C22B0600           call 00508AFE  
:004A5F3C 83C40C               add esp, 0000000C  
:004A5F3F 8BCE                 mov ecx, esi  
:004A5F41 E8CA080000           call 004A6810  ====>这次我们看看这个 call 的作用  
:004A5F46 83F801               cmp eax, 00000001  
:004A5F49 0F85CA000000         jne 004A6019  ====>关键的跳  
:004A5F4F 8986C0D59201         mov dword ptr [esi+0192D5C0], eax  
:004A5F55 C705B4EA4C0302000000  mov dword ptr [034CEAB4], 00000002  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004A6062(C)  ====>成功，来到这里，前提是上面的，从 004A6062 跳来的，  
|  
:004A5F5F 8BCE                 mov ecx, esi  
:004A5F61 E86A040000           call 004A63D0  
:004A5F66 B9C0C0C000           mov ecx, 00C0C0C0  
:004A5F6B B880808000           mov eax, 00808080  
:004A5F70 890DDC321102         mov dword ptr [021132DC], ecx  
:004A5F76 890DE0321102         mov dword ptr [021132E0], ecx  
..............省  
```

`====>call 004A6810` 这次我们看看这个 `call` 的作用

```
* Referenced by a CALL at Addresses:  
|:004A5F41   , :004B5AA3   我们看到的 call，有 2 个，就是第 2 个在作怪，我开始没注意到，  
|  
:004A6810 6AFF                 push FFFFFFFF  
:004A6812 68464A5300           push 00534A46  
:004A6817 64A100000000         mov eax, dword ptr fs:[00000000]  
:004A681D 50                   push eax  
:004A681E 64892500000000       mov dword ptr fs:[00000000], esp  
:004A6825 81EC80000000         sub esp, 00000080  
:004A682B 8D442404             lea eax, dword ptr [esp+04]  
:004A682F 8D4C241C             lea ecx, dword ptr [esp+1C]  
:004A6833 50                   push eax  
:004A6834 51                   push ecx  
:004A6835 C744240C90010000     mov [esp+0C], 00000190  

* Reference To: KERNEL32.GetComputerNameA, Ord:00CEh  ====>获得你的  
                     |  
:004A683D FF1594A35300         Call dword ptr [0053A394]  
:004A6843 8B15545E5700         mov edx, dword ptr [00575E54]  
:004A6849 89542400             mov dword ptr [esp], edx  
:004A684D 8D44241C             lea eax, dword ptr [esp+1C]  
:004A6851 8D4C2400             lea ecx, dword ptr [esp] ====>ecx 就是你的计算机名  
:004A6855 50                   push eax  

* Possible StringData Ref from Data Obj ->"%s"  
                     |  
:004A6856 6824C45600           push 0056C424  
:004A685B 51                   push ecx  
:004A685C C784249400000000000000  mov dword ptr [esp+00000094], 00000000  
:004A6867 E892220600           call 00508AFE  
:004A686C 83C40C               add esp, 0000000C  
:004A686F 8D4C2400             lea ecx, dword ptr [esp]  
:004A6873 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"OEMCOMPUTER"  ====>计算机名的比较，以下都是  
                     |  
:004A6875 68F8215700           push 005721F8  
:004A687A E84C1F0600           call 005087CB  
:004A687F 85C0                 test eax, eax  
:004A6881 0F84EB000000         je 004A6972   ====>跳！？  
:004A6887 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"147"  
                     |  
:004A6889 68F4215700           push 005721F4  
:004A688E 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A6892 E8341F0600           call 005087CB  
:004A6897 85C0                 test eax, eax  
:004A6899 0F84D3000000         je 004A6972  
:004A689F 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"ANSYS"  
                     |  
:004A68A1 68EC215700           push 005721EC  
:004A68A6 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A68AA E81C1F0600           call 005087CB  
:004A68AF 85C0                 test eax, eax  
:004A68B1 0F84BB000000         je 004A6972  
:004A68B7 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"MSTCAD"  ====>计算机名的比较，以下都是  
                     |  
:004A68B9 68E4215700           push 005721E4  
:004A68BE 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A68C2 E8041F0600           call 005087CB  
:004A68C7 85C0                 test eax, eax  
:004A68C9 0F84A3000000         je 004A6972  
:004A68CF 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"MST"  
                     |  
:004A68D1 68E0215700           push 005721E0  
:004A68D6 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A68DA E8EC1E0600           call 005087CB  
:004A68DF 85C0                 test eax, eax  
:004A68E1 0F848B000000         je 004A6972  
:004A68E7 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"FAZURE"  
                     |  
:004A68E9 68D8215700           push 005721D8  
:004A68EE 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A68F2 E8D41E0600           call 005087CB  
:004A68F7 85C0                 test eax, eax  
:004A68F9 7477                 je 004A6972  
:004A68FB 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"WCZGN"  
                     |  
:004A68FD 68D0215700           push 005721D0  
:004A6902 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A6906 E8C01E0600           call 005087CB  
:004A690B 85C0                 test eax, eax  
:004A690D 7463                 je 004A6972  
:004A690F 6A00                 push 00000000  

* Possible StringData Ref from Data Obj ->"JAVAS"  
                     |  
:004A6911 68C8215700           push 005721C8  
:004A6916 8D4C2408             lea ecx, dword ptr [esp+08]  
:004A691A E8AC1E0600           call 005087CB  
:004A691F 85C0                 test eax, eax  
:004A6921 744F                 je 004A6972  

* Possible StringData Ref from Data Obj ->"LDD"  
                     |  
:004A6923 68C4215700           push 005721C4  
:004A6928 8D4C2404             lea ecx, dword ptr [esp+04]  
```



:004A692C E88C1E0600    call 005087BD

:004A6931 85C0        test eax, eax

:004A6933 743D        je 004A6972

* Possible StringData Ref from Data Obj ->"7-208"

                              |

:004A6935 68BC215700    push 005721BC

:004A693A 8D4C2404      lea ecx, dword ptr [esp+04]

:004A693E E87A1E0600    call 005087BD

:004A6943 85C0        test eax, eax

:004A6945 742B        je 004A6972  ====>如果到这里还不同，那

:004A6947 8D4C2400      lea ecx, dword ptr [esp]

:004A694B C7842488000000FFFFFFFF  mov dword ptr [esp+00000088], FFFFFFFF

:004A6956 E8A5840600    call 0050EE00

:004A695B 33C0        xor eax, eax  ====>如果到这里还不同，那 `eax` 请零，返回失败

:004A695D 8B8C2480000000  mov ecx, dword ptr [esp+00000080]

:004A6964 64890D00000000  mov dword ptr fs:[00000000], ecx

:004A696B 81C48C000000  add esp, 0000008C

:004A6971 C3        ret   ====>，返回

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004A6881(C), :004A6899(C), :004A68B1(C), :004A68C9(C), :004A68E1(C)

|:004A68F9(C), :004A690D(C), :004A6921(C), :004A6933(C), :004A6945(C)

|

:004A6972 56        push esi

:004A6973 8D4C240C      lea ecx, dword ptr [esp+0C]

:004A6977 E89D890600    call 0050F319

:004A697C 6A00        push 00000000

:004A697E 6A00        push 00000000

* Possible StringData Ref from Data Obj ->"c:\windows\help\m$.TMP.txt"

                              |

:004A6980 68A0215700    push 005721A0

:004A6985 8D4C2418      lea ecx, dword ptr [esp+18]

:004A6989 C684249800000001        mov byte ptr [esp+00000098], 01

:004A6991 E8F8890600    call 0050F38E

:004A6996 8BF0        mov esi, eax

:004A6998 85F6        test esi, esi

:004A699A 7512        jne 004A69AE

:004A699C 50        push eax

:004A699D 50        push eax

* Possible StringData Ref from Data Obj ->"c:\winnt\help\m$.TMP.txt"

                              |

:004A699E 6884215700    push 00572184

:004A69A3 8D4C2418      lea ecx, dword ptr [esp+18]

:004A69A7 E8E2890600    call 0050F38E

:004A69AC 8BF0        mov esi, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A699A(C)

|

:004A69AE 83FE01      cmp esi, 00000001

:004A69B1 7509        jne 004A69BC

:004A69B3 8D4C240C      lea ecx, dword ptr [esp+0C]

:004A69B7 E8028D0600    call 0050F6BE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A69B1(C)

|

:004A69BC 8D4C240C      lea ecx, dword ptr [esp+0C]

:004A69C0 C684248C00000000        mov byte ptr [esp+0000008C], 00

:004A69C8 E87E890600    call 0050F34B  ====>可能是关键 `call`

:004A69CD 8D4C2404      lea ecx, dword ptr [esp+04]

:004A69D1 C784248C000000FFFFFFFF  mov dword ptr [esp+0000008C], FFFFFFFF

:004A69DC E81F840600    call 0050EE00

:004A69E1 8B8C2484000000  mov ecx, dword ptr [esp+00000084]

:004A69E8 8BC6        mov eax, esi  ====>到这里，`esi` 的值给 `eax`，如果是 1，那就办了

:004A69EA 5E        pop esi

:004A69EB 64890D00000000  mov dword ptr fs:[00000000], ecx

:004A69F2 81C48C000000  add esp, 0000008C

:004A69F8 C3        ret

看来作者自己留下了一个后门，就是有以上这些名字的机器，可以不带狗运行，当然还要验证。可能是他们开发组的局域网里的机器，我猜。

我没搞明白验证过程，不过我们直接让 `eax`＝1，就行了。这次我明白为什么我们改起来这么容易了。

* Possible StringData Ref from Data Obj ->"%s"

                              |

:004A5F2D 6824C45600    push 0056C424

:004A5F32 68E8E94C03    push 034CE9E8

:004A5F37 E8C22B0600    call 00508AFE

:004A5F3C 83C40C      add esp, 0000000C

:004A5F3F 8BCE        mov ecx, esi

:004A5F41 E8CA080000    call 004A6810  ====>这个关键 `call` 的作用，可以返回 `eax`，那我们让 `eax`＝1

:004A5F46 83F801      cmp eax, 00000001 ====>注意：以前改这里为 `mov eax, 00000001`

:004A5F49 0F85CA000000        jne 004A6019  ====>注意：以前关键的跳，我们不跳，改为 `nop`

:004A5F4F 8986C0D59201        mov dword ptr [esi+0192D5C0], eax   ====>那末 `dword ptr [esi+0192D5C0]`＝1

:004A5F55 C705B4EA4C0302000000        mov dword ptr [034CEAB4], 00000002  ====>那末 `dword ptr [034CEAB4]`＝2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004A6062(C)  ====>那末成功，我们直接来到这里，连时间检测也免了，^\_^

|

:004A5F5F 8BCE        mov ecx, esi

:004A5F61 E86A040000    call 004A63D0

:004A5F66 B9C0C0C000    mov ecx, 00C0C0C0

:004A5F6B B880808000    mov eax, 00808080

:004A5F70 890DDC321102        mov dword ptr [021132DC], ecx

:004A5F76 890DE0321102        mov dword ptr [021132E0], ecx

..............省

但是这次，我们不能了，他这次改变了加密，看到那些增加的 `Call` 就知道，他现在还在程序运行中增加了检查！！

至于增加的 `call` 是怎样运作的，我们不管了，总之都是陷阱，我们跳过去就行了。

我们分析了以上，就明白，要获得企业版，那末就要检测狗成功并{让 `dword ptr [034CEAB4]`等于 2，让 `dword ptr [esi+0192D5C0]`等于 1},系统时间在 2003-2004 之间，当然现在小于 2004 就行。

这时我们在回头看看，进去 `Call` 再改！

\====>`call` `004A6810` 这次我们看看这个 `call` 的作用

* Referenced by a `CALL` at Addresses:

|:004A5F41   , :004B5AA3    注意，2 个地方调用这个 `call`！！！！！

|

:004A6810 6AFF        push FFFFFFFF

:004A6812 68464A5300    push 00534A46

:004A6817 64A100000000  mov eax, dword ptr fs:[00000000]

:004A681D 50        push eax

:004A681E 64892500000000  mov dword ptr fs:[00000000], esp

:004A6825 81EC80000000  sub esp, 00000080

:004A682B 8D442404      lea eax, dword ptr [esp+04]

:004A682F 8D4C241C      lea ecx, dword ptr [esp+1C]

:004A6833 50        push eax

:004A6834 51        push ecx

:004A6835 C744240C90010000    mov [esp+0C], 00000190

* Reference To: `KERNEL32.GetComputerNameA`, `Ord:00CEh`  ====>获得你的

                              |

:004A683D FF1594A35300    Call dword ptr [0053A394]

:004A6843 8B15545E5700    mov edx, dword ptr [00575E54]

:004A6849 89542400    mov dword ptr [esp], edx

:004A684D 8D44241C      lea eax, dword ptr [esp+1C]

:004A6851 8D4C2400    lea ecx, dword ptr [esp] ===>`ecx` 就是你的计算机名

:004A6855 50        push eax

* Possible StringData Ref from Data Obj ->"%s"

                              |

:004A6856 6824C45600    push 0056C424

:004A685B 51        push ecx

:004A685C C784249400000000000000  mov dword ptr [esp+00000094], 00000000

:004A6867 E892220600    call 00508AFE

:004A686C 83C40C      add esp, 0000000C

:004A686F 8D4C2400    lea ecx, dword ptr [esp]

:004A6873 6A00        push 00000000

* Possible StringData Ref from Data Obj ->"OEMCOMPUTER"  ====>计算机名的比较，以下都是

                              |

:004A6875 68F8215700    push 005721F8

:004A687A E84C1F0600    call 005087CB

:004A687F 85C0        test eax, eax

:004A6881 0F84EB000000        je 004A6972   ====>跳！？这个太长了，我们用下面的

:004A6887 6A00        push 00000000

看来作者自己留下了一个后门，就是有以上这些名字的机器，可以不带狗运行，当然还要验证。可能是他们开发组的局域网里的机器，我猜。

继续

* Possible StringData Ref from Data Obj ->"147"

.....

省略                              |

.....

* Possible StringData Ref from Data Obj ->"FAZURE"

                              |

:004A68E9 68D8215700    push 005721D8

:004A68EE 8D4C2408      lea ecx, dword ptr [esp+08]

:004A68F2 E8D41E0600    call 005087CB

:004A68F7 85C0        test eax, eax



`:004A68F9 7477` `je 004A6972` `====>跳！？我们用这个，` `jmp [eb77]` `比较短好改！`

`:004A68FB 6A00` `push 00000000`

`* Possible StringData Ref from Data Obj ->"WCZGN"`

`.....`

`省略`

`.....`

`最后来到这里.`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:004A69B1(C)`

`|`

`:004A69BC 8D4C240C` `lea ecx, dword ptr [esp+0C]`

`:004A69C0 C684248C00000000` `mov byte ptr [esp+0000008C], 00`

`:004A69C8 E87E890600` `call 0050F34B`

`:004A69CD 8D4C2404` `lea ecx, dword ptr [esp+04]`

`:004A69D1 C784248C000000FFFFFFFF` `mov dword ptr [esp+0000008C], FFFFFFFF`

`:004A69DC E81F840600` `call 0050EE00`

`:004A69E1 8B8C2484000000` `mov ecx, dword ptr [esp+00000084]`

`:004A69E8 8BC6` `mov eax, esi` `====>到这里，esi 的值给 eax，如果是 1，那就办了`

`:004A69EA 5E` `pop esi`

`:004A69EB 64890D00000000` `mov dword ptr fs:[00000000], ecx`

`:004A69F2 81C48C000000` `add esp, 0000008C`

`:004A69F8 C3` `ret`

`:004A69F9 90` `nop` `====>看到这里的 90 吗，我们正好用他们。`

`:004A69FA 90` `nop` `====>估计以前是花指令，这次他们忘了放！`

`:004A69FB 90` `nop`

`:004A69FC 90` `nop`

`:004A69FD 90` `nop`

`:004A69FE 90` `nop`

`:004A69FF 90` `nop`

这次我们直接改这个 `call` 的返回值 `eax`，分析得知 `eax＝1` 是我们需要的。

上次没看到有 2 个地方调用这个 `call`，失误，不过是他们这个版本增加的。

我们把 `004A69F8 C3 ret` 这句推后，插入一句 `mov eax，01`，机器码 `b801000000`，还富裕两个 `90` 呢！

改 `:004A68F9 7477` `je 004A6972`

为 `:004A68F9 eb77` `jmp 004A6972`

```
改:004A69F8 C3            ret
 :004A69F9 90             nop   ==== >看到这里的 90 吗，我们正好用他们。
 :004A69FA 90             nop   ==== >估计以前是花指令，这次他们忘了放！
 :004A69FB 90             nop
 :004A69FC 90             nop
 :004A69FD 90             nop                

为:004A69F8 B801000000    mov eax，00000001
 :004A69FD C3             ret
```

当然还有另外直接改检测狗的方法，较麻烦。好几个 `Call` 要改，就不废话了。

这次还是利用它的后门。 下次，建议作者关闭这个后门吧。虽然还是防不住。

大功告成，无限制。

总结：不要太相信狗的保护能力，在软件中保护不要太脆弱。这个软件里有很多花指令，又有何用？打狗要有耐心，恒心，要多仔细观察.

建议作者别为自己方便，也给他人方便。

`4dd466d0e1e0cc123afbe504da2462ca`
