
# 推箱子 1.6

> 原文：[`www.pediy.com/kssd/pediy03/forum838.htm`](https://www.pediy.com/kssd/pediy03/forum838.htm)

:00403199 EB6F             jmp 0040320A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00402CBD(C) //要注册成功，只有从 00402CBD 跳来
|

* Possible StringData Ref from Code Obj ->"恭喜你已经成功注册！"

|

:0040319B B888324000       mov eax, 00403288

:004031A0 E827310300       call 004362CC

:004031A5 8D85D0FDFFFF     lea eax, dword ptr [ebp+FFFFFDD0]

----------------------------------------------------------

:00402C96 E83D6B0600       call 004697D8

:00402C9B FF45F8           inc [ebp-08]

:00402C9E 817DF801020000   cmp dword ptr [ebp-08], 00000201

:00402CA5 75DB             jne 00402C82

:00402CA7 B8E0A14800       mov eax, 0048A1E0

:00402CAC E80B970600       call 0046C3BC

:00402CB1 E8226B0600       call 004697D8

:00402CB6 803DDCA1480000   cmp byte ptr [0048A1DC], 00//要跳，这 0048A1DC 不能为 0，所以下 bpm 48a1dc

:00402CBD 0F85D8040000     jne 0040319B

:00402CC3 B832000000       mov eax, 00000032

:00402CC8 E8ABE90500       call 00461678

:00402CCD 8945F4           mov dword ptr [ebp-0C], eax

:00402CD0 6A32             push 00000032

:00402CD2 8B45F4           mov eax, dword ptr [ebp-0C]

:00402CD5 50               push eax

----------------------------------------------------------

:00401DB3 C685CCFEFFFF00   mov byte ptr [ebp+FFFFFECC], 00

:00401DBA C605DCA1480001   mov byte ptr [0048A1DC], 01 //先置 1

:00401DC1 C745FC01000000   mov [ebp-04], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401DF8(C)
|

:00401DC8 8B45FC           mov eax, dword ptr [ebp-04]

:00401DCB 8A8405C5FEFFFF   mov al, byte ptr [ebp+eax-0000013B] //取一个字符到 al

:00401DD2 8B55FC           mov edx, dword ptr [ebp-04]

:00401DD5 3A84152AFFFFFF   cmp al, byte ptr [ebp+edx-000000D6] //比较

:00401DDC 7413             je 00401DF1 //必须跳

:00401DDE C605DCA1480000   mov byte ptr [0048A1DC], 00//清零

:00401DE5 C60546A3480000   mov byte ptr [0048A346], 00

:00401DEC E9B2020000       jmp 004020A3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401DDC(C)
|

:00401DF1 FF45FC           inc [ebp-04]

:00401DF4 837DFC07         cmp dword ptr [ebp-04], 00000007

:00401DF8 75CE             jne 00401DC8  4dd466d0e1e0cc123afbe504da2462ca
