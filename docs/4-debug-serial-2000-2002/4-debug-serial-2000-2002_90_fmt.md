
# PacMania v1.5

> 原文：[`www.pediy.com/kssd/pediy02/forum270.htm`](https://www.pediy.com/kssd/pediy02/forum270.htm)

软件名称：`PacMania`

版本：`v1.5`

文件大小：`2.1MB`

软件简介：一个更新了的大嘴吃豆的游戏。

注册后无成功与否的提示。

没办法只好用 `W32DASM`

在 `string data reference` 中找到 `"Registered to"`，双击找到下面的程序：

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0045275A(C)

```
|
:004527C1 8B460C          mov eax, dword ptr [esi+0C]
:004527C4 BA10000000      mov edx, 00000010
:004527C9 E8A219FCFF      call 00414170
:004527CE 803D2C68450000  cmp byte ptr [0045682C], 00
:004527D5 7430            je 00452807
:004527D7 80BBB002000002  cmp byte ptr [ebx+000002B0], 02
:004527DE 753A            jne 0045281A
:004527E0 8D45FC          lea eax, dword ptr [ebp-04]
:004527E3 8B0DD0714500    mov ecx, dword ptr [004571D0]

* Possible StringData Ref from Code Obj ->"Registered to "

                                        |
:004527E9 BA84284500      mov edx, 00452884
:004527EE E88912FBFF      call 00403A7C
:004527F3 8B45FC          mov eax, dword ptr [ebp-04]
:004527F6 50              push eax
:004527F7 33C9            xor ecx, ecx
:004527F9 BA0A000000      mov edx, 0000000A
:004527FE 8BC6            mov eax, esi
:00452800 E80326FCFF      call 00414E08
:00452805 EB13            jmp 0045281A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004527D5(C)
|
* Possible StringData Ref from Code Obj ->"Unregistered"

                                        |
:00452807 689C284500      push 0045289C
:0045280C 33C9            xor ecx, ecx
:0045280E BA0A000000      mov edx, 0000000A
:00452813 8BC6            mov eax, esi
:00452815 E8EE25FCFF      call 00414E08
```

显然在 `0045682C` 中存的是注册标志，查找 `0045682C`，将相关的跳转修改即可。

`4dd466d0e1e0cc123afbe504da2462ca`
