
```assembly
mov dword ptr [ebp-08], eax

:10008282 AD

  lodsd

:10008283 33058FF90010            xor eax, dword

ptr [1000F98F]    \*注册码后8位地址

:10008289 50

  push eax

:1000828A FF158CF80010            call dword ptr

[1000F88C]    \*同前面:1000806D之F(2)

:10008290 83C404

add esp, 00000004

:10008293 6652

  push dx

:10008295 6650

  push ax

:10008297 58

  pop eax        \*注册码后8位经转换后内容

:10008298 3345F8

xor eax, dword ptr [ebp-08]

:1000829B 66B90800                mov

cx, 0008

:1000829F 8BD8

  mov ebx, eax

:100082A1 53

  push ebx

:100082A2 83E30F

and ebx, 0000000F

:100082A5 83FB09

cmp ebx, 00000009

:100082A8 7602

  jbe 100082AC

:100082AA 33C0

  xor eax, eax        \*EAX为0注册失败,必须跳过。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100082A8(C)

|

:100082AC 5B

  pop ebx

:100082AD C1EB04

shr ebx, 04

:100082B0 E2EF

  loop 100082A1

:100082B2 8B7510

mov esi, dword ptr [ebp+10]

:100082B5 8906

  mov dword ptr [esi], eax

:100082B7 61

  popad

:100082B8 C9

  leave

:100082B9 C20C00

ret 000C
```

至此程序注册算法基本上已弄明白，有兴趣的朋友可写注册机了（函数`F(1)`和`F(2)`互逆）。
