
# [原创]Asprotect V2.X 的脱壳与修复的总结及练习

Asprotect V2.X 的脱壳与修复的总结及练习

作    者: blackeyes

关于Asprotect V2.X 的脱壳与修复, loveboom的《ASPROTECT 2.x 脱壳系列》已经非常的全面与经典.

本人在此只是依葫芦画瓢, 并将有些地方再详细的解释一下, 给菜鸟们脱Asprotect V2.X时进行参考, 高手就不要看了.

先在理论上研讨一下, 由于编译器的不一样, C 与 Delphi 所编译的汇编结果有差别, Asprotect 加壳时处理的也不一样.

先假设有一程序:

OEP: `00401000`

IAT: `00407000 - 00407FFF`

在`00401100 CALL DLL1.API1`

在`00401180 CALL DLL1.API2`

Ospr 加壳后, 好多的API CALL 被改成 `CALL 12000000`

然后开始研究.

## 1. C/C++ Program

### 1.1) 未被加壳的程序

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    FF15 00704000   CALL DWORD PTR DS:[00407000]              ; DLL1.API1
00401106    ...
00401180    FF15 04704000   CALL DWORD PTR DS:[00407004]              ; DLL1.API2
00401186    ...

\*\*\*\*\*\*\*\*\* IAT 可能是这样的 \*\*\*\*\*\*\*\*\*\*\*\*

00407000    DD  7C571000    // DLL1.API1
00407004    DD  7C572000    // DLL1.API2
...
004070FC    DD  00000000
00407100    DD  7D00XXXX    // DLL2.API1
...
004071FC    DD  00000000
...
00407F00    DD  7F00XXXX    // DLLn.API1
...
00407FFC    DD  00000000
```

### 1.2) 被 Asprotect 加壳后的程序在OEP

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    E8   FBEEBF11   CALL 12000000                             ; 壳
00401105    ??
00401106    ...
00401180    E8   7BEEBF11   CALL 12000000                             ; 壳
00401185    ??
00401186    ...

\*\*\*\*\*\*\*\*\*\*\*\*\*IAT\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

00407000    DD  ????????    // 被加密的 DLL1.API1 信息
00407004    DD  7C572000    // 未加密的 DLL1.API2
```

ASPR 将许多的API CALL 都改成了统一的 `CALL 12000000`

即改 `CALL DWORD PTR DS:[xxxxx]` 成 `CALL 12000000`

### 1.3) 加壳后的程序运行过API CALL 后

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    E8   00EFBF12   CALL 13000005                             ; 壳
00401105    ??
00401106    ...                                                       ; 假设运行到这
00401180    E8   7BEEBF11   CALL 12000000                             ; 壳
00401185    ??
00401186    ...
```

即 每经过一个`CALL12000000` 后, 这个 `CALL12000000` 就会被改成 `CALL13000005` 或者 其它的 `CALL 14000005`, 反正每个API 还不一样, 分别对应一段壳的CODE, 等于相对应的API CODE

### 1.4) 脱壳与修复需要解决以下几个问题

A.) 找到每一个 `CALL 12000000` 所在的地址, 例如`00401100`, `00401180`, ...

B.) 找到每一个 `CALL 12000000` 实际应该CALL到的 API 地址, 例如 `7C571000`, `7C572000`, ...

C.) 如果IAT的地址未加密, 修复 `CALL 12000000` 成 `CALL DWORD PTR DS:[0040xxxx]`

例如 `00401180    E8   7BEEBF11   CALL 12000000` 这句, 根据 API 地址 `7C572000` 可在 IAT 中找到, 位于`00407004`

修复成 `00401180    FF15 04704000   CALL DWORD PTR DS:[00407004]`

注意: `CALL 12000000` 是 5 个字节, 而 `CALL DWORD PTR DS:[xxxxxxxx]` 是 6 个字节

D.) 如果IAT的地址被加密, 可在IAT强行加上一项, 再修复 `CALL 12000000` 成 `CALL DWORD PTR DS:[0040xxxx]`

例如 `00401100    E8   FBEEBF11   CALL 12000000` 这句, API 地址 `7C571000` 在 IAT 中找不到,

可在IAT最后加上一项, 最后的IAT可能会变成这样:

```
...
00407F00    DD  7D00XXXX    // DLLn.API1
...
00407FFC    DD  00000000    // 原来的 IAT 到这结束
00408000    DD  7C571000        // 新加的 IAT 项
00408004    DD  7C573000        // 新加的 IAT 项
...
00408080    DD  00000000        // 每个DLL的IAT必须以 00000000 结束, 不能与其它DLL 的IAT连成一片
00408084    DD  7D00xxxx        // 新加的 IAT 项, 另外的DLL的
...
........    DD  00000000    // 修复的 IAT 到这结束
```

再把`00401180` 修复成 `00401100    FF15 04704000   CALL DWORD PTR DS:[00408004]`

注意: 这里已经与未加壳的程序有一点不一样了, 即 API 地址在 IAT 的位置不一样, 但功能是一样的,

### 1.5) 在到达OEP之前, 壳把程序中的API CALL 改成 CALL 12000000 的地方的CODE 是这样的:

```
012C5F7A    45              INC EBP                                  ; EBP 就是 CALL 12000000 处的地址, Pactch1
012C5F7B    8945 00         MOV DWORD PTR SS:[EBP],EAX
012C5F7E    6A 0A           PUSH 0A
012C5F80    E8 8FCEFEFF     CALL 012B2E14
012C5F85    8BC8            MOV ECX,EAX
012C5F87    038B E4000000   ADD ECX,DWORD PTR DS:[EBX+E4]
012C5F8D    8BD6            MOV EDX,ESI
012C5F8F    8BC3            MOV EAX,EBX
012C5F91    E8 9EE5FFFF     CALL 012C4534
012C5F96    FF0C24          DEC DWORD PTR SS:[ESP]
012C5F99    03B3 E4000000   ADD ESI,DWORD PTR DS:[EBX+E4]
012C5F9F    833C24 00       CMP DWORD PTR SS:[ESP],0
012C5FA3  ^ 0F87 55FEFFFF   JA 012C5DFE                              ; 循环
012C5FA9    53              PUSH EBX                                 ; 到这, 所有的 API CALL 改 CALL 12000000 都完成了
```

所以 在 `012C5F7A` 处, 要先到我们的修复 `CODE1`, 把 `EBP` 保存下来, 然后再回来

```
012C5F7A    E8 xxxxxxxx     JMP 1540100                              ;
012C5F7F    90              NOP
```

到 `012C5FA9` 后, 把 `012C5F7A` 处的CODE 还原, 再运行到 OEP

找这段CODE, 可在壳解压缩自身后, SEARCH

```
inc ebp
move [ebp], eax
push 0A
```

### 1.6) 到达OEP之后, 如果跟踪 CALL 12000000 到壳中, 壳会根据CALL是从哪儿来的 解码 API 地址,

API 地址 出现并且不会被壳检测的地方的CODE是这样的:

```
012A2544         85C0                         TEST EAX,EAX
012A2546         74 0A                        JE SHORT 012A2552
012A2548         FF15 18902C01                CALL DWORD PTR DS:[12C9018]    // 到这儿时, EDX=API 地址, Patch2
012A254E         09C0                         OR EAX,EAX
012A2550         74 01                        JE SHORT 012A2553
012A2552         C3                           RETN
```

然后壳把`CALL 12000000` 改成 相应的`CALL 13000005`

最后壳再跳到新改成的 `CALL 13000005` 处继续执行

```
015100A1         9D                           POPFD
015100A2         5C                           POP ESP
015100A3         FF6424 FC                    JMP DWORD PTR SS:[ESP-4]      // 这儿跳到00401100 处的 CALL 13000005
015100A7         F2:                          PREFIX REPNE:                 ; Superfluous prefix
015100A8         EB 01                        JMP SHORT 015100AB
015100AA       - E9 94CA2E01                  JMP 027FCB43
015100AF         C3                           RETN
```

在 OEP 处修复时, 就是用修复 `CODE2` 模拟的运行 每一处的 `CALL 12000000`, 取得 每一处的对应的API 地址,

然后在 `JMP DWORD PTR SS:[ESP-4]` 处截住, 这时当然还不能让它跑去执行真正的API CODE

根据`CALL 12000000` 所在的地址与得到的 API 地址就可修复 IAT

然后再模拟运行下一个 `CALL 12000000`

所以 在 `012A2548` 处, 要先到我们的修复 `CODE3`, 把 `EDX` 保存下来, 然后再回来

```
012A2548         FF15 xxxxxxxx                CALL DWORD PTR DS:[01540050]

// 01540050 中是CODE3 的起始地址 015401F0
```

在 `015100A3` 处, 不能让它自由的跳, 要让它回到 修复 `CODE2` 去 修复 IAT

loveboom 说这儿的 CODE 不能改, 壳会检测出来, 我没试过. 反正可在这儿设硬件断点, 再靠script 强行跳到 `CODE2` 中.

只是每次运行时 `JMP DWORD PTR SS:[ESP-4]` 这一句所在的地址还都不一样, 真的挺烦人的

找这段CODE, 可 在OD 中 用 Ctrl+T 设 Run Trace 的暂停条件 command is `POPFD`, 再 Ctrl+F11,

找到后去掉暂停条件,并设硬件断点, 否则 Script 运行会出错的.

## 2. 再讨论 Delph 程序

### 2.1) 未被加壳的程序

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    E8 FB4E0000     CALL 00406000                             ; DLL1.API1
00401105    ...
00401180    E8 7F4E0000     CALL 00406004                             ; DLL1.API2
00401185    ...
00406000    FF25 00704000   JMP DWORD PTR DS:[00407000]               ; DLL1.API1
00406006    FF25 04704000   JMP DWORD PTR DS:[00407004]               ; DLL1.API2

\*\*\*\*\*\*\*\*\*\*\*\*\*IAT\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

IAT 与 C/C++ 程序一样

### 2.2) 被 Asprotect 加壳后的程序在OEP

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    E8 FB4E0000     CALL 00406000                             ; DLL1.API1
00401105    ...
00401180    E8 7F4E0000     CALL 00406004                             ; DLL1.API2
00401185    ...
00406000    E8 FB9FBF11     CALL 12000000                             ; 壳
00406005    ??
00406006    E8 7B9FBF11     CALL 12000000                             ; 壳
0040600B    ??
```

注意: ASPR 修改的地方与C/C++程序是不一样的, 它是改 `JMP DWORD PTR DS:[xxxxx]` 成 `CALL 12000000`

### 2.3) 加壳后的程序运行过API CALL 后

```
00401000    55              PUSH EBP                                  ; 程序 OEP
00401001    8BEC            MOV EBP, ESP
...
00401100    E8 FB4E0000     CALL 00406000                             ; DLL1.API1
00401105    ...
```
