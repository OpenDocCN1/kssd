
# readbook1.46

> 原文：[`www.pediy.com/kssd/pediy04/kanxue456.htm`](https://www.pediy.com/kssd/pediy04/kanxue456.htm)

`readbook1.46` 的 `pj`（我喜欢的看书工具）

它是用 `PETITE2.2` 加壳的，首先用 `PROC DUMP` 解壳。

再用 `WIN32DASM` 分析。

```
:00409E53 3975FC                 cmp dword ptr [ebp-04], esi
:00409E56 741D                   je 00409E75
:00409E58 8B55F0                 mov edx, dword ptr [ebp-10]
:00409E5B 8BC8                    mov ecx, eax
:00409E5D 83E17F                 and ecx, 0000007F
:00409E60 03348A                 add esi, dword ptr [edx+4*ecx]
:00409E63 40                      inc eax
:00409E64 3DFF0F0000             cmp eax, 00000FFF
:00409E69 A3FC744600             mov dword ptr [004674FC], eax
:00409E6E 72E3                   jb 00409E53
:00409E70 3975FC                 cmp dword ptr [ebp-04], esi
:00409E73 7578                   jne 00409EED

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00409E56(C)
|
* Reference To: USER32.GetMenu, Ord:0000h
                        |
:00409E75 8B35A4354500     mov esi, dword ptr [004535A4]
:00409E7B C70584A84B0001000000   mov dword ptr [004BA884], 00000001
:00409E85 FF731C             push [ebx+1C]
:00409E88 FFD6               call esi
:00409E8A 50                 push eax
:00409E8B E864B50300         call 004453F4

* Reference To: USER32.GetMenuItemCount, Ord:0000h
                        |
:00409E90 8B3D98354500     mov edi, dword ptr [00453598]
:00409E96 6A00               push 00000000

* Possible Ref to Menu: MenuID_0080, Item: "k 欒?(R)"
                        |
:00409E98 6850800000       push 00008050
:00409E9D FF7004           push [eax+04]
:00409EA0 FFD7             call edi
:00409EA2 FF731C           push [ebx+1C]
:00409EA5 FFD6             call esi
:00409EA7 50               push eax
:00409EA8 E864B50300       call 00445411
:00409EAD 6A00               push 00000000

* Possible Ref to Menu: MenuID_0080, Item: "俇鑼(?)"
                        |
:00409EAF 6851800000       push 00008051
:00409EB4 FF7004           push [eax+04]
:00409EB7 FFD7             call edi
:00409EB9 FF731C           push [ebx+1C]
:00409EBC FFD6             call esi
:00409EBE 50               push eax
:00409EBF E864B50300       call 00445428
:00409EC4 33F6               xor esi, esi
:00409EC6 56                 push esi

* Possible Ref to Menu: MenuID_0080, Item: "Q
?鑼(O)"
                        |
:00409EC7 68AF800000       push 000080AF
:00409ECC FF7004           push [eax+04]
:00409ECF FFD7             call edi
:00409ED1 393588A84B00     cmp dword ptr [004BA888], esi
:00409ED7 744D               je 00409F26
:00409ED9 56                 push esi

* Possible StringData Ref from Data Obj ->"祝贺"
                        |
:00409EDA 6878584600       push 00465878

* Possible StringData Ref from Data Obj ->"您已经成功地注册了!"
                        |
:00409EDF 6864584600       push 00465864
:00409EE4 8BCB               mov ecx, ebx
:00409EE6 E8258E0300         call 00442D10
:00409EEB EB39               jmp 00409F26
```

`167:00409e53 cmp [ebp-04],esi`　`[ebp-04]`是你输入的密码，`ESI` 是真正的注册码

`167:00409e70 cmp [ebp-04],esi`　`[ebp-04]`是你输入的密码，`ESI` 是真正的注册码

所以设中断　`bpx 409e70`

`? esi` 就是真正的注册码。如果看到的是负数时，重新设中断为 `BPX 00409e53` 再 `? ESI` 就会看到真正的注册码。另一个名字可以有许多注册码。字母的大小写程序不加以区分。如：`wangwei:1433841600`　`WANGWEI:1433841600`

`4dd466d0e1e0cc123afbe504da2462ca`
