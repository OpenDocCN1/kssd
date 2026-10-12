
# 雷霆战将

> 原文：[`www.pediy.com/kssd/pediy03/forum1270.htm`](https://www.pediy.com/kssd/pediy03/forum1270.htm)

刚买的《雷霆战将》就被一无耻之徒顺手牵羊。痛感交友不当之后写下此文，望后来者以儆效尤。

首先用`w32dasm`反汇编之`RogueSpear.exe`。

在查找`"cd"`、`"nocd"`、`"GetVolumeInformationA"`、`"GetLogicalDriveStringsA"`之类的子串，我们来到下面的代码：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C27D(C)
|
:0040C1EC 5F                       pop edi
:0040C1ED 5E                       pop esi
:0040C1EE 5D                       pop ebp
:0040C1EF B001                     mov al, 01
:0040C1F1 5B                       pop ebx
:0040C1F2 81C410060000            add esp, 00000610
:0040C1F8 C3                       ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C1EA(C)
|
:0040C1F9 8D94241C020000         lea edx, dword ptr [esp+0000021C]
:0040C200 52                       push edx
:0040C201 6800040000               push 00000400

* Reference To: KERNEL32.GetLogicalDriveStringsA, Ord:011Eh－－－取当前路径。
|
:0040C206 FF15A8507600            Call dword ptr [007650A8]
:0040C20C 8BD8                     mov ebx, eax
:0040C20E 85DB                     test ebx, ebx
:0040C210 750A                     jne 0040C21C－－－－－－－如果是完全安装，ok。

* Possible StringData Ref from Data Obj ->"GAME: Could not get drives installed "
->"in the system"
|
:0040C212 6884517B00              push 007B5184
:0040C217 E986000000               jmp 0040C2A2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C210(C)
|
:0040C21C 33F6                     xor esi, esi
:0040C21E 85DB                     test ebx, ebx
:0040C220 7E7B                     jle 0040C29D－－－－－－－－cd 都不装，就别想玩游戏。

* Reference To: KERNEL32.GetDriveTypeA, Ord:0104h－－－－－－－检查单前盘是否是 cd。
|
:0040C222 8B2DAC507600            mov ebp, dword ptr [007650AC]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C29B(C)
|
:0040C228 8DBC341C020000         lea edi, dword ptr [esp+esi+0000021C]
:0040C22F 57                       push edi
:0040C230 FFD5                     call ebp
:0040C232 83F805                   cmp eax, 00000005
:0040C235 754C                     jne 0040C283－－－－－－－－不是光驱，game over。
:0040C237 8D84241C010000          lea eax, dword ptr [esp+0000011C]
:0040C23E 6800010000               push 00000100
:0040C243 8D4C241C                 lea ecx, dword ptr [esp+1C]
:0040C247 50                       push eax
:0040C248 8D54241C                 lea edx, dword ptr [esp+1C]
:0040C24C 51                       push ecx
:0040C24D 8D44241C                 lea eax, dword ptr [esp+1C]
:0040C251 52                       push edx
:0040C252 50                       push eax
:0040C253 8D4C2430                 lea ecx, dword ptr [esp+30]
:0040C257 6800010000               push 00000100
:0040C25C 51                       push ecx
:0040C25D 57                       push edi

* Reference To: KERNEL32.GetVolumeInformationA, Ord:0177h－－－－－－检查卷标。
|
:0040C25E FF15B0507600            Call dword ptr [007650B0]
:0040C264 83F801                   cmp eax, 00000001
:0040C267 751A                     jne 0040C283－－－－－－不成功，让你死的难看。
:0040C269 8D54241C                 lea edx, dword ptr [esp+1C]

* Possible StringData Ref from Data Obj ->"ROGUESPR"－－－－－光盘卷标。
|
:0040C26D 6878517B00              push 007B5178
:0040C272 52                       push edx
:0040C273 E898112900               call 0069D410
:0040C278 83C408                   add esp, 00000008
:0040C27B 85C0                     test eax, eax
:0040C27D 0F8469FFFFFF             je 0040C1EC－－－－－ok，检查完毕退出 CALL。

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:0040C235(C), :0040C267(C)
|
:0040C283 803F00                     cmp byte ptr [edi], 00－－－－是否读卷标失败。
:0040C286 7410                     je 0040C298

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C296(C)
|
:0040C288 3BF3                     cmp esi, ebx－－－－－是否再读几次。
:0040C28A 7D0C                     jge 0040C298
:0040C28C 8A84341D020000          mov al, byte ptr [esp+esi+0000021D]
:0040C293 46                       inc esi
:0040C294 84C0                     test al, al
:0040C296 75F0                     jne 0040C288－－－－－循环。

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:0040C286(C), :0040C28A(C)
|
:0040C298 46                       inc esi
:0040C299 3BF3                     cmp esi, ebx
:0040C29B 7C8B                     jl 0040C228－－－－－－再给你几次机会。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C220(C)
|

* Possible StringData Ref from Data Obj ->"GAME: Could not find CD"－－－－－game over。
|
:0040C29D 6860517B00              push 007B5160

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040C217(U)
|
:0040C2A2 B914468100              mov ecx, 00814614
:0040C2A7 E878EC2800               call 0069AF24
:0040C2AC 6850174000               push 00401750
:0040C2B1 6A0A                     push 0000000A
:0040C2B3 8BC8                     mov ecx, eax
:0040C2B5 E823EB2800               call 0069ADDD
:0040C2BA 8BC8                     mov ecx, eax
:0040C2BC E86F54FFFF               call 00401730
:0040C2C1 B908468100              mov ecx, 00814608
:0040C2C6 E8C5240200               call 0042E790
:0040C2CB 5F                       pop edi
:0040C2CC 5E                       pop esi
:0040C2CD 5D                       pop ebp
:0040C2CE 32C0                     xor al, al
:0040C2D0 5B                       pop ebx
:0040C2D1 81C410060000            add esp, 00000610
:0040C2D7 C3                       ret－－－－－－－－－－－－－－退出 CALL。

:0040C2D8 90                       nop
:0040C2D9 90                       nop
:0040C2DA 90                       nop
:0040C2DB 90                       nop
:0040C2DC 90                       nop
:0040C2DD 90                       nop
:0040C2DE 90                       nop
:0040C2DF 90                       nop
```

自己看者办，要乎乎了。

`Zzzzzzzzzz` 4dd466d0e1e0cc123afbe504da2462ca
