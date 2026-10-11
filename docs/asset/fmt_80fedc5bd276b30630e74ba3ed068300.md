
# Audio compositor3.1b

> 原文：[`www.pediy.com/kssd/pediy05/pediy50061.htm`](https://www.pediy.com/kssd/pediy05/pediy50061.htm)

`Audio compositor` 注册码及注册机

工具：`TRW2000`，`Keymake`，`w32dasm`

软件介绍：`Audio compositor3.1b`，一个声音处理程序，能在 `Wav` 和 `Midi` 之间转换，具体不会用，拿来练手。

### 1. 运行与提示
1. 运行 `Audio compositor`，选 `Help-Register Audio compositor`，填写 `Name:esoft2001.51.net`（要大于 5 位），`Registration code:12345678`（要 8 位）。按 `OK`，出现提示“The name/code combination is invalid.”

### 2. 反汇编分析
2. `w32dasm` 反汇编 `AC.exe`，使用“串式数据参考”可找到上面提示的一句，双击该句会找到显示该提示的 `CALL` 语句（见下面），下面是注册相关的一段代码：

```
:0042B493 B87C1E4800               mov eax, 00481E7C
:0042B498 E833960100               call 00444AD0
:0042B49D 51                       push ecx
:0042B49E 51                       push ecx
:0042B49F 53                       push ebx
:0042B4A0 56                       push esi
:0042B4A1 8BF1                     mov esi, ecx
:0042B4A3 57                       push edi
:0042B4A4 8D4DF0                   lea ecx, dword ptr [ebp-10]
:0042B4A7 E842130300               call 0045C7EE
:0042B4AC 33FF                     xor edi, edi

* Possible Reference to Menu: MenuID_0001

                                      |
* Possible Reference to String Resource ID=00001: "Audio Compositor"

                                      |
:0042B4AE 6A01                      push 00000001
:0042B4B0 8BCE                      mov ecx, esi
:0042B4B2 897DFC                    mov dword ptr [ebp-04], edi
:0042B4B5 E898480300                call 0045FD52
:0042B4BA 8B465C                    mov eax, dword ptr [esi+5C]
:0042B4BD 8378F808                  cmp dword ptr [eax-08], 00000008
:0042B4C1 7409                      je 0042B4CC
:0042B4C3 57                       push edi
:0042B4C4 57                       push edi

* Possible StringData Ref from Data Obj ->"The registration code must be "
                                      ->"8 characters long."

                                      |
:0042B4C5 68045A4A00                push 004A5A04
:0042B4CA EB5E                       jmp 0042B52A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042B4C1(C)
|
:0042B4CC 8B4660                    mov eax, dword ptr [esi+60]
:0042B4CF 8D5E60                    lea ebx, dword ptr [esi+60]
:0042B4D2 8378F805                  cmp dword ptr [eax-08], 00000005
:0042B4D6 7D09                      jge 0042B4E1
:0042B4D8 57                       push edi
:0042B4D9 57                       push edi

* Possible StringData Ref from Data Obj ->"The registration name must be "
                                      ->"at least 5 characters long."

                                      |
:0042B4DA 68C8594A00                push 004A59C8
:0042B4DF EB49                      jmp 0042B52A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042B4D6(C)
|
* Possible StringData Ref from Data Obj ->"%s is not a valid output device"

                                      |
:0042B4E1 6834224A00                push 004A2234
:0042B4E6 8D4DEC                    lea ecx, dword ptr [ebp-14]
:0042B4E9 E8B9140300                call 0045C9A7
:0042B4EE 50                       push eax
:0042B4EF 8D45F0                    lea eax, dword ptr [ebp-10]
:0042B4F2 53                       push ebx
:0042B4F3 50                       push eax
:0042B4F4 C645FC01                mov [ebp-04], 01
:0042B4F8 E80699FDFF                call 00404E03
:0042B4FD 8065FC00                  and byte ptr [ebp-04], 00
:0042B501 83C40C                    add esp, 0000000C
:0042B504 8D4DEC                    lea ecx, dword ptr [ebp-14]
:0042B507 E82D140300                call 0045C939
:0042B50C FF75F0                    push [ebp-10]
:0042B50F FF765C                    push [esi+5C]
:0042B512 E8C9960100                call 00444BE0（关键）
:0042B517 F7D8                       neg eax
:0042B519 1BC0                       sbb eax, eax
:0042B51B 59                        pop ecx
:0042B51C F7D8                       neg eax
:0042B51E 84C0                       test al, al（用 al 做注册标志）
:0042B520 59                        pop ecx
:0042B521 740E                       je 0042B531
:0042B523 57                        push edi
:0042B524 57                        push edi

* Possible StringData Ref from Data Obj ->"The name/code combination is invalid."

                                      |
:0042B525 68A0594A00                push 004A59A0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0042B4CA(U), :0042B4DF(U)
|
:0042B52A E89EC50300                call 00467ACD（提示“The name/code combination is invalid.”）
:0042B52F EB4B                      jmp 0042B57C
```

### 3. 动态调试
3. 启动 `TRW` 并载入 `AC.exe`，`g 0042B512` 并按 `F8` 进入：

```
:00444BE0 A150E64A00               mov eax, dword ptr [004AE650]
:00444BE5 53                       push ebx
:00444BE6 55                       push ebp
:00444BE7 33ED                     xor ebp, ebp
:00444BE9 56                       push esi
:00444BEA 3BC5                     cmp eax, ebp
:00444BEC 57                       push edi
:00444BED 7541                     jne 00444C30（这里会跳走）
…………
:00444C30 6A19                      push 00000019
:00444C32 E8396E0000                call 0044BA70
:00444C37 8B74241C                  mov esi, dword ptr [esp+1C]
:00444C3B 8B7C2418                  mov edi, dword ptr [esp+18]（D ESI 看到注册码）
:00444C3F 83C404                    add esp, 00000004
:00444C42 B204                      mov dl, 04
```

### 4. 编写注册机
4. 编写注册机

使用“注册机编写器（`Keymaker`）”之“另类注册机”功能

1. 程序名称：`ac.exe`
2. 添加数据：
   - 中断地址：`42B512`
   - 中断次数：`1`
   - 第一字节：`E8`
   - 指令长度：`5`
   - 中断地址：`444C3B`
   - 中断次数：`1`
   - 第一字节：`8B`
   - 指令长度：`4`
3. 选择内存方式 `ESI`。

`esoft2001.51.net`

2002 年 4 月 5 日 `4dd466d0e1e0cc123afbe504da2462ca`
