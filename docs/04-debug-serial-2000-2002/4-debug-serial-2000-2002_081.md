
# Net PAL 1.2C 注册码破解

> 原文：[`www.pediy.com/kssd/pediy02/forum172.htm`](https://www.pediy.com/kssd/pediy02/forum172.htm)

由于近日持续遇见拨号上网难以接入的问题，今天终于从网上拉了一个强力拨号软件 `NET PAL 1.2C`。

这个软件需要注册，否则只能使用 30 天。

`bpx hmemcpy`

`F12` `11` 下，以后我们来到：

```
:00417EB4 E8CA340000      call 0041B383  <--比较用户名输入否?
:00417EB9 85C0           test eax, eax
:00417EBB 7413           je 00417ED0
:00417ED4 E8AA340000      call 0041B383  <--比较注册码输入否?
:00417ED9 85C0           test eax, eax
:00417EDB 7413           je 00417EF0
:00417F28 E8EF0D0000      call 00418D1C  <--比较核心, F8 进入
:00418D62 83F845          cmp eax, 00000045          <--???
:00418D65 7529           jne 00418D90
:00418D67 8B4514          mov eax, dword ptr [ebp+14]
:00418D6A 40             inc eax
:00418D6B 50             push eax
:00418D6C 8D45C8          lea eax, dword ptr [ebp-38]
:00418D6F 50             push eax
* Reference To: KERNEL32.lstrcpyA, Ord:0302h
:00418D70 FF1598D24300    Call dword ptr [0043D298]
:00418D76 66C745E80100    mov [ebp-18], 0001
:00418D7C 6A1E            push 0000001E
:00418D7E FF7510          push [ebp+10]
:00418D81 6A07            push 00000007
:00418D83 FF7508          push [ebp+08]
:00418D86 E8CAF8FFFF      call 00418655
:00418D8B 8945BC          mov dword ptr [ebp-44], eax
:00418D8E EB25            jmp 00418DB5
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00418D65(C)
:00418D90 FF7514          push [ebp+14]
:00418D93 8D45C8          lea eax, dword ptr [ebp-38]
:00418D96 50             push eax
* Reference To: KERNEL32.lstrcpyA, Ord:0302h
:00418D97 FF1598D24300    Call dword ptr [0043D298]
:00418D9D 668365E800      and word ptr [ebp-18], 0000
:00418DA2 6A1E            push 0000001E
:00418DA4 FF7510          push [ebp+10]
:00418DA7 FF750C          push [ebp+0C]
:00418DAA FF7508          push [ebp+08]
:00418DAD E8A3F8FFFF      call 00418655
:00418DB2 8945BC          mov dword ptr [ebp-44], eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00418D8E(U)
:00418DB5 6A2D            push 0000002D
:00418DB7 8D45C8          lea eax, dword ptr [ebp-38]
:00418DBA 50             push eax
:00418DBB E860680000      call 0041F620
:00418DC0 59             pop ecx
:00418DC1 59             pop ecx
:00418DC2 8945EC          mov dword ptr [ebp-14], eax
:00418DC5 8365B800        and dword ptr [ebp-48], 00000000
:00418DC9 837DEC00        cmp dword ptr [ebp-14], 00000000
:00418DCD 742F            je 00418DFE
:00418DCF 8B45EC          mov eax, dword ptr [ebp-14]
:00418DD2 802000          and byte ptr [eax], 00
:00418DD5 8B45EC          mov eax, dword ptr [ebp-14]
:00418DD8 40             inc eax
:00418DD9 8945EC          mov dword ptr [ebp-14], eax
:00418DDC 6A2D            push 0000002D
:00418DDE FF75EC          push [ebp-14]
:00418DE1 E83A680000      call 0041F620
:00418DE6 59             pop ecx
:00418DE7 59             pop ecx
:00418DE8 8945B8          mov dword ptr [ebp-48], eax
:00418DEB 837DB800        cmp dword ptr [ebp-48], 00000000
:00418DEF 740D            je 00418DFE
:00418DF1 8B45B8          mov eax, dword ptr [ebp-48]
:00418DF4 802000          and byte ptr [eax], 00
:00418DF7 8B45B8          mov eax, dword ptr [ebp-48]
:00418DFA 40             inc eax
:00418DFB 8945B8          mov dword ptr [ebp-48], eax
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00418DCD(C), :00418DEF(C)
:00418DFE 6A0A            push 0000000A
:00418E00 6A00            push 00000000
:00418E02 8D45C8          lea eax, dword ptr [ebp-38]
:00418E05 50             push eax
:00418E06 E84F7A0000      call 0042085A
:00418E0B 83C40C          add esp, 0000000C
:00418E0E 8945F4          mov dword ptr [ebp-0C], eax
:00418E11 8B45F4          mov eax, dword ptr [ebp-0C]
:00418E14 3B45BC          cmp eax, dword ptr [ebp-44]  <--比较注册码
:00418E17 0F85DB010000    jne 00418FF8
```

所以：

`name`: `xiaolou[CCG]`
`serial`: `1895830`

此外，有个奇怪问题，从 `HELP` 文件中得知，注册分个人用户与团体用户两种，我猜：

```
:00418D62 83F845          cmp eax, 00000045          <--???
```

处，即第一个字母为 `E`，就是团体用户，可是这样得到的注册码输入后，程序告诉我出现一个"致命错误"，并要我上报"`BUG FILE`"。

`who know???` `4dd466d0e1e0cc123afbe504da2462ca`
