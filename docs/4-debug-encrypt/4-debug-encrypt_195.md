
# PerformanceTest 3.4

> 原文：[`www.pediy.com/kssd/pediy03/forum892.htm`](https://www.pediy.com/kssd/pediy03/forum892.htm)

软件名称：`PassMark PerformanceTest 3.4`

破解目的：30 天试用期，程序启动时跳出注册窗

破解工具：`TRW2000`、`W32Dsm89`、`UltraEdit`

首先用 `TRW2000` 载入程序，停留在程序入口处，然后开始不停的按 `F10` 单步跟踪……

```
//****** Program Entry Point ******

.
.
.

* Reference To: KERNEL32.GetModuleHandleA, Ord:0126h
                |
:00416B34 FF1520314200        Call dword ptr [00423120]
:00416B3A 50                  push eax
:00416B3B E8302AFFFF          call 00409570  <---- 这里跳出注册窗，F8 跟进
:00416B40 8945A0              mov dword ptr [ebp-60], eax
:00416B43 50                  push eax
:00416B44 E8D9F9FFFF          call 00416522
:00416B49 8B45EC              mov eax, dword ptr [ebp-14]
:00416B4C 8B08                mov ecx, dword ptr [eax]
:00416B4E 8B09                mov ecx, dword ptr [ecx]
:00416B50 894D98              mov dword ptr [ebp-68], ecx
:00416B53 50                  push eax
:00416B54 51                  push ecx
:00416B55 E802570000          call 0041C25C
:00416B5A 59                  pop ecx
:00416B5B 59                  pop ecx
:00416B5C C3                  ret
```

重新载入，按 `F8` 跟进 `call 00409570`，继续按 `F10` 单步跟踪……

```
.
.
.

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409973(C)
|
:0040997D 8B0DC0144400       mov ecx, dword ptr [004414C0]
:00409983 8B742414           mov esi, dword ptr [esp+14]
:00409987 33C0               xor eax, eax
:00409989 83F910             cmp ecx, 00000010
:0040998C 0F9DC0             setnl al
:0040998F 3BF3               cmp esi, ebx
:00409991 A388144400         mov dword ptr [00441488], eax
:00409996 7410               je 004099A8
:00409998 8BCE               mov ecx, esi
:0040999A E861550000         call 0040EF00
:0040999F 56                 push esi
:004099A0 E883B80000         call 00415228
:004099A5 83C404             add esp, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409996(C)
|
:004099A8 E8A3480000         call 0040E250  <---- 这里跳出注册窗，F8 跟进
:004099AD 85C0               test eax, eax
:004099AF 750C               jne 004099BD
:004099B1 E85A200000         call 0040BA10
:004099B6 53                 push ebx
```

重新载入，按 `F8` 跟进 `call 0040E250`，按 `F10` 单步跟踪……

```
* Possible StringData Ref from Data Obj ->"2 旒"
                |
:0040E250 C705F81F440040B34200   mov dword ptr [00441FF8], 0042B340
:0040E25A C705F41F440000000000   mov dword ptr [00441FF4], 00000000
:0040E264 E8C7000000             call 0040E330  <---- 这里跳出注册窗。跟踪到这里的时候，我已经
                                                        没什么耐心了，而且我在下面发现了一些重要
                                                        的提示……
:0040E269 83F801                 cmp eax, 00000001
:0040E26C 7501                   jne 0040E26F
:0040E26E C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E26C(C)
|
:0040E26F 83F803                 cmp eax, 00000003
:0040E272 0F84A7000000           je 0040E31F
:0040E278 833DF41F44000F         cmp dword ptr [00441FF4], 0000000F
:0040E27F 7523                   jne 0040E2A4
:0040E281 6A30                   push 00000030

* Reference To: USER32.MessageBeep, Ord:01BDh
                |
:0040E283 FF15F8324200           Call dword ptr [004232F8]
:0040E289 A1C41A4400             mov eax, dword ptr [00441AC4]
:0040E28E 6A10                   push 00000010

* Possible StringData Ref from Data Obj ->"Date / Time Error"
                |
:0040E290 6804B44200             push 0042B404

* Possible StringData Ref from Data Obj ->"The date and time on this machine "  <--程序记录了使用
                                       ->"is earlier than when
the program "        时间，所以修改
                                       ->"was previously run. In order to "        系统时间以延长
                                       ->"enforce the
unregistered version's "    使用期是没用的
                                       ->"eval period, this is not be allowed"
                |
:0040E295 6858B34200             push 0042B358
:0040E29A 50                     push eax

* Reference To: USER32.MessageBoxA, Ord:01BEh
                |
:0040E29B FF1584324200           Call dword ptr [00423284]
:0040E2A1 33C0                   xor eax, eax
:0040E2A3 C3                     ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E27F(C)
|
:0040E2A4 53                     push ebx
:0040E2A5 56                     push esi
:0040E2A6 57                     push edi

* Possible StringData Ref from Data Obj ->"Dqqnqctqhmfrs`qsto-Bgdbjvqhsdodqlhrrhnmh"
                                       ->"r`u`hk`akdhmsgddwdbts`akdchqdbsnqx+ Bgdbj"
                                       ->"sgdjdx-c`sehkddwhrsrhmsgddwdbts`akdch"
                                       ->"qdbsnqx- Sqxsntmhmrs`kk%qdhmrs`kksgdrnes"
                                       ->"v`qd+heoqnakdlrodqrhrs Bnms`bs9vvv-o`rrl`q"
                                       ->"j-bnlenqrtoonqs Rs`qstoDqqnqmtladq"
                |                     ↑ 上面这些怪码是不是就是提示过期的信息，难怪
                                                        我查找不到。
:0040E2A7 68E8AF4200             push 0042AFE8
:0040E2AC E85FFFFFFF             call 0040E210

* Possible StringData Ref from Data Obj ->"Dqqnq"
                |
:0040E2B1 68ACB14200             push 0042B1AC
:0040E2B6 8BF0                   mov esi, eax
:0040E2B8 E853FFFFFF             call 0040E210
:0040E2BD 8BD8                   mov ebx, eax
:0040E2BF 8BFE                   mov edi, esi
:0040E2C1 83C9FF                 or ecx, FFFFFFFF
:0040E2C4 33C0                   xor eax, eax
:0040E2C6 F2                     repnz
:0040E2C7 AE                     scasb
:0040E2C8 F7D1                   not ecx
:0040E2CA 83C113                 add ecx, 00000013
:0040E2CD 51                     push ecx
:0040E2CE E81B6E0000             call 004150EE
:0040E2D3 83C40C                 add esp, 0000000C
:0040E2D6 8BF8                   mov edi, eax
:0040E2D8 6A30                   push 00000030

* Reference To: USER32.MessageBeep, Ord:01BDh
                |
:0040E2DA FF15F8324200           Call dword ptr [004232F8]
:0040E2E0 8B0DF41F4400           mov ecx, dword ptr [00441FF4]
:0040E2E6 51                     push ecx
:0040E2E7 56                     push esi
:0040E2E8 684CB34200             push 0042B34C
:0040E2ED 57                     push edi
:0040E2EE E84D6B0000             call 00414E40
:0040E2F3 8B15C41A4400           mov edx, dword ptr [00441AC4]
:0040E2F9 83C410                 add esp, 00000010
:0040E2FC 6A10                   push 00000010
:0040E2FE 53                     push ebx
:0040E2FF 57                     push edi
:0040E300 52                     push edx

* Reference To: USER32.MessageBoxA, Ord:01BEh
                |
:0040E301 FF1584324200           Call dword ptr [00423284]
:0040E307 56                     push esi
:0040E308 E8F86C0000             call 00415005
```



```
:0040E30D 53   push ebx
:0040E30E E8F26C0000 call 00415005
:0040E313 57   push edi
:0040E314 E8EC6C0000 call 00415005
:0040E319 83C40C add esp, 0000000C
:0040E31C 5F   pop edi
:0040E31D 5E   pop esi
:0040E31E 5B   pop ebx
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0040E272(C)`

```
:0040E31F 33C0 xor eax, eax
:0040E321 C3   ret
```

用 `UltraEdit` 查找 `:0040E264` 处的代码：

查找 `E8 C7 00 00 00 83 F8 01 75 01`

改为 `90 90 90 90 90 90 90 90 90 90`

`fishs`

`http://fishs.126.com`

### 温柔一点，好吗？——暴破 `pt` 的另一种方法

作者：`kingtall`

目标：`Performance Test v3.4 build 1000`，以下简称 `pt`。

工具：`w32dasm8.93`, `trw1.22`

运行 `pt`，看 `about` 框，有 `'Evaluation version'` 字样。

用 `w32dasm` 反汇编 `pt.ext`，找 `Evaluation version`，找到如下：

```
* Referenced by a CALL at Addresses:
|:004096D2 , :0040990D
|
:00408680 A148194400 mov eax, dword ptr [00441948]；若不为 0 就显示 Evaluation version
:00408685 85C0 test eax, eax
:00408687 7527 jne 004086B0
:00408689 6840704300 push 00437040
* Possible StringData Ref from Data Obj ->"V3.4"
|
:0040868E 687C614200 push 0042617C
* Possible StringData Ref from Data Obj ->"PerformanceTest"
|
:00408693 68CC684200 push 004268CC
* Possible StringData Ref from Data Obj ->"%s %s"
|
:00408698 68D8944200 push 004294D8
:0040869D 6830184400 push 00441830
:004086A2 E8D9C30000 call 00414A80
:004086A7 83C414 add esp, 00000014
:004086AA B830184400 mov eax, 00441830
:004086AF C3   ret
```

Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00408687(C)`

Possible StringData Ref from Data Obj -> `"Evaluation version"`

```
:004086B0 68C4944200 push 004294C4
```

现在只要找到程序中把 `[441948]` 置 `0` 的地方。我是这样做的，在 `w32dasm` 的 `Disassembler` 菜单中选 `Save Disassemble text file and create project file`，保存结果为 `pt.alf`。然后用写字板打开 `pt.alf`，找 `481944`，找到下面这一段：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040E3FC(C)
|
:0040E40D 8BFE mov edi, esi
:0040E40F 83C9FF or ecx, FFFFFFFF
:0040E412 33C0 xor eax, eax
:0040E414 F2   repnz
:0040E415 AE   scasb
:0040E416 F7D1 not ecx
:0040E418 49   dec ecx
:0040E419 83F903 cmp ecx, 00000003
:0040E41C 7244 jb 0040E462   ；把这一句改为 eb0f jmp 40e42d
:0040E41E 837B74FF cmp dword ptr [ebx+74], FFFFFFFF
:0040E422 753E jne 0040E462
:0040E424 80BB8000000037 cmp byte ptr [ebx+00000080], 37
:0040E42B 7535 jne 0040E462
:0040E42D C7054819440000000000 mov dword ptr [00441948], 0 ；在这里把[441948]置 0。
```

还有一点，程序启动时会先出现一个小窗口，等好长时间才能进入主窗口，好烦哪！

运行 `TRW`，`load` `pt.exe`，下 `bpx sleep`，找到下面（用 `w32dasm` 找 `sleep` 也一样）：

```
0167:00409547 68DC050000 PUSH DWORD 05DC ；改为 6800000000 push dword 0
0167:0040954C FF1528314200 CALL `KERNEL32!Sleep`
```

`4dd466d0e1e0cc123afbe504da2462ca`
