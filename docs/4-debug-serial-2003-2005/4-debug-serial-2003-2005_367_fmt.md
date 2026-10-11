
# 拱猪大战 V2.3XP

> 原文：[`www.pediy.com/kssd/pediy06/pediy6138.htm`](https://www.pediy.com/kssd/pediy06/pediy6138.htm)

拱猪大战 V2.3XP 算法破解手记

机器码：`928257269`

注册名：`leexoyo`（不参与运算）

假码：`98765432109`

真码：`41589464321`

工具：`TRW20001.23` 注册版、`PE-SCAN3.31`、`W32Dasm8.93` 黄金版，`PEiD`

`PEiD` 侦壳，`ASPack` 2.12 -> `Alexey Solodovnikov` 的壳，`PE-scan` 脱之，`hmemcpy`，`bd` `*`，`pmodule` 来到下面，在 `442758` 改跳转后提示重启验证，

```
0187:00442684 8D4DFC           LEA      ECX,[EBP-04]
0187:00442687 51               PUSH     ECX
0187:00442688 BA163F5700       MOV      EDX,00573F16
0187:0044268D 8D45F8           LEA      EAX,[EBP-08]
0187:00442690 E833C90F00       CALL     0053EFC8
0187:00442695 FF471C           INC      DWORD [EDI+1C]
0187:00442698 8D55F8           LEA      EDX,[EBP-08]
0187:0044269B 58               POP      EAX
0187:0044269C E84FCB0F00       CALL     0053F1F0
0187:004426A1 50               PUSH     EAX
0187:004426A2 FF4F1C           DEC      DWORD [EDI+1C]
0187:004426A5 8D45F8           LEA      EAX,[EBP-08]
0187:004426A8 BA02000000       MOV      EDX,02
0187:004426AD E86ECA0F00       CALL     0053F120
0187:004426B2 FF4F1C           DEC      DWORD [EDI+1C]
0187:004426B5 8D45FC           LEA      EAX,[EBP-04]
0187:004426B8 BA02000000       MOV      EDX,02
0187:004426BD E85ECA0F00       CALL     0053F120
0187:004426C2 59               POP      ECX
0187:004426C3 84C9             TEST     CL,CL
0187:004426C5 7427             JZ       004426EE     //下 r fl z 提示请输入注册名
0187:004426C7 6A40             PUSH     BYTE +40
0187:004426C9 682D3F5700       PUSH     DWORD 00573F2D
0187:004426CE 68173F5700       PUSH     DWORD 00573F17
0187:004426D3 8BC3             MOV      EAX,EBX
0187:004426D5 E81E090900       CALL     004D2FF8
0187:004426DA 50               PUSH     EAX
0187:004426DB E8AA4F1200       CALL     `USER32!MessageBoxA`
0187:004426E0 8B17             MOV      EDX,[EDI]
0187:004426E2 64891500000000   MOV      [FS:00],EDX
0187:004426E9 E9B4020000       JMP      004429A2
0187:004426EE 66C747101400     MOV      WORD [EDI+10],14     //★★★
0187:004426F4 33C9             XOR      ECX,ECX
0187:004426F6 894DF4           MOV      [EBP-0C],ECX
0187:004426F9 8D55F4           LEA      EDX,[EBP-0C]
0187:004426FC FF471C           INC      DWORD [EDI+1C]
0187:004426FF 8B830C030000     MOV      EAX,[EBX+030C]
0187:00442705 E8CAA00800       CALL     004CC7D4
0187:0044270A 8D4DF4           LEA      ECX,[EBP-0C]
0187:0044270D 33D2             XOR      EDX,EDX
0187:0044270F 8B01             MOV      EAX,[ECX]
0187:00442711 50               PUSH     EAX
0187:00442712 8955F0           MOV      [EBP-10],EDX
0187:00442715 FF471C           INC      DWORD [EDI+1C]
0187:00442718 8D55F0           LEA      EDX,[EBP-10]
0187:0044271B 8B8314030000     MOV      EAX,[EBX+0314]
0187:00442721 E8AEA00800       CALL     004CC7D4
0187:00442726 8D4DF0           LEA      ECX,[EBP-10]
0187:00442729 8B01             MOV      EAX,[ECX]    //eax=假码
0187:0044272B 50               PUSH     EAX
0187:0044272C E83BF3FBFF       CALL     00401A6C     //算法 call，循环太多，看晕过去了。
0187:00442731 83C408           ADD      ESP,BYTE +08
0187:00442734 BA02000000       MOV      EDX,02
0187:00442739 50               PUSH     EAX
0187:0044273A 8D45F0           LEA      EAX,[EBP-10]
0187:0044273D FF4F1C           DEC      DWORD [EDI+1C]
0187:00442740 E8DBC90F00       CALL     0053F120
0187:00442745 FF4F1C           DEC      DWORD [EDI+1C]
0187:00442748 8D45F4           LEA      EAX,[EBP-0C]
0187:0044274B BA02000000       MOV      EDX,02
0187:00442750 E8CBC90F00       CALL     0053F120
0187:00442755 59               POP      ECX
0187:00442756 84C9             TEST     CL,CL    //cl 为 0 吗？
0187:00442758 0F8422020000     JZ       NEAR 00442980    //为 0 则跳，下 r fl z 使它不跳，提示注册成功，请重启验证。
0187:0044275E B201             MOV      DL,01
0187:00442760 A170805000       MOV      EAX,[00508070]

**********************************************************************
```

跟进 `44272C E83BF3FBFF` `CALL` `00401A6C` 这个 call：

```
:00401A6C 55                      push ebp
:00401A6D 8BEC                    mov ebp, esp
:00401A6F 81C4BCFEFFFF            add esp, FFFFFEBC
:00401A75 53                      push ebx
:00401A76 56                      push esi
:00401A77 57                      push edi
:00401A78 8D7DCC                  lea edi, dword ptr [ebp-34]
:00401A7B B8FC835600              mov eax, 005683FC
:00401A80 E8C7EA1100              call 0052054C
:00401A85 C7471C02000000          mov [edi+1C], 00000002
:00401A8C 8D550C                  lea edx, dword ptr [ebp+0C]
:00401A8F 8D450C                  lea eax, dword ptr [ebp+0C]
:00401A92 E869D51300              call 0053F000
:00401A97 FF471C                  inc [edi+1C]
:00401A9A 8D5508                  lea edx, dword ptr [ebp+08]
:00401A9D 66C747100800            mov [edi+10], 0008
:00401AA3 8D4508                  lea eax, dword ptr [ebp+08]
:00401AA6 E855D51300              call 0053F000
:00401AAB FF471C                  inc [edi+1C]
:00401AAE 57                      push edi
:00401AAF 8DBDBCFEFFFF            lea edi, dword ptr [ebp+FFFFFEBC]
:00401AB5 BE44815600              mov esi, 00568144
:00401ABA B90C000000              mov ecx, 0000000C
:00401ABF F3                      repz
:00401AC0 A5                      movsd
:00401AC1 837D0800                cmp dword ptr [ebp+08], 00000000
:00401AC5 5F                      pop edi
:00401AC6 7408                    je 00401AD0
:00401AC8 8B4508                  mov eax, dword ptr [ebp+08]     //eax=假码
:00401ACB 8B50FC                  mov edx, dword ptr [eax-04]     //edx=假码的位数
:00401ACE EB02                    jmp 00401AD2    //Go!!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AC6(C)
|
:00401AD0 33D2                    xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401ACE(U)
|
:00401AD2 83FA0B                 cmp edx, 0000000B     //注册码是 11 位吗？
:00401AD5 7432                   je 00401B09     //不是？over，是则跳
:00401AD7 33C0                   xor eax, eax
:00401AD9 BA02000000             mov edx, 00000002
:00401ADE 50                     push eax
:00401ADF 8D4508                 lea eax, dword ptr [ebp+08]
:00401AE2 FF4F1C                 dec [edi+1C]
:00401AE5 E836D61300             call 0053F120
:00401AEA FF4F1C                 dec [edi+1C]
:00401AED 8D450C                 lea eax, dword ptr [ebp+0C]
:00401AF0 BA02000000             mov edx, 00000002
:00401AF5 E826D61300             call 0053F120
:00401AFA 58                     pop eax
:00401AFB 8B17                   mov edx, dword ptr [edi]
:00401AFD 64891500000000         mov dword ptr fs:[00000000], edx
:00401B04 E9B0020000             jmp 00401DB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AD5(C)
|
:00401B09 BE01000000             mov esi, 00000001     //esi 置 1
:00401B0E 8D8548FFFFFF           lea eax, dword ptr [ebp+FFFFFF48]
:00401B14 8945C8                 mov dword ptr [ebp-38], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401BDF(C)
|
:00401B17 66C747101400           mov [edi+10], 0014
:00401B1D 33D2                   xor edx, edx
:00401B1F 8D4DFC                 lea ecx, dword ptr [ebp-04]
:00401B22 8955FC                 mov dword ptr [ebp-04], edx
```



```
:00401B25 51                     push ecx

:00401B26 FF471C                 inc [edi+1C]

:00401B29 B901000000             mov ecx, 00000001

:00401B2E 8BD6                   mov edx, esi

:00401B30 8D4508                 lea eax, dword ptr [ebp+08]

:00401B33 E860D71300             call 0053F298

:00401B38 837DFC00               cmp dword ptr [ebp-04], 00000000

:00401B3C 7405                   je 00401B43

:00401B3E 8B45FC                 mov eax, dword ptr [ebp-04]

:00401B41 EB05                   jmp 00401B48

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B3C(C)
|

:00401B43 B878815600             mov eax, 00568178

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B41(U)
|

:00401B48 8A18                   mov bl, byte ptr [eax]

:00401B4A FF4F1C                 dec [edi+1C]

:00401B4D 8D45FC                 lea eax, dword ptr [ebp-04]

:00401B50 BA02000000             mov edx, 00000002

:00401B55 E8C6D51300             call 0053F120

:00401B5A 0FBEC3                 movsx eax, bl   //依次取输入假码的 hex 值送 eax

:00401B5D 83F830                 cmp eax, 00000030

:00401B60 7C05                   jl 00401B67

:00401B62 83F839                 cmp eax, 00000039

:00401B65 7E32                   jle 00401B99   //上面两个比较是看输入的注册码是否是数字，是则继续，有一个不是则 over

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B60(C)
|

:00401B67 33C0                   xor eax, eax

:00401B69 BA02000000             mov edx, 00000002

:00401B6E 50                     push eax

:00401B6F 8D4508                 lea eax, dword ptr [ebp+08]

:00401B72 FF4F1C                 dec [edi+1C]

:00401B75 E8A6D51300             call 0053F120

:00401B7A FF4F1C                 dec [edi+1C]

:00401B7D 8D450C                 lea eax, dword ptr [ebp+0C]

:00401B80 BA02000000             mov edx, 00000002

:00401B85 E896D51300             call 0053F120

:00401B8A 58                     pop eax

:00401B8B 8B17                   mov edx, dword ptr [edi]

:00401B8D 64891500000000         mov dword ptr fs:[00000000], edx

:00401B94 E920020000             jmp 00401DB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401B65(C)
|

:00401B99 66C747102000           mov [edi+10], 0020

:00401B9F 33C9                   xor ecx, ecx

:00401BA1 8D45F8                 lea eax, dword ptr [ebp-08]

:00401BA4 894DF8                 mov dword ptr [ebp-08], ecx

:00401BA7 50                     push eax

:00401BA8 FF471C                 inc [edi+1C]

:00401BAB 8D4508                 lea eax, dword ptr [ebp+08]

:00401BAE B901000000             mov ecx, 00000001

:00401BB3 8BD6                   mov edx, esi

:00401BB5 E8DED61300             call 0053F298

:00401BBA 8D45F8                 lea eax, dword ptr [ebp-08]

:00401BBD E8BAD71300             call 0053F37C

:00401BC2 8B55C8                 mov edx, dword ptr [ebp-38]

:00401BC5 8902                   mov dword ptr [edx], eax

:00401BC7 FF4F1C                 dec [edi+1C]

:00401BCA 8D45F8                 lea eax, dword ptr [ebp-08]

:00401BCD BA02000000             mov edx, 00000002

:00401BD2 E849D51300             call 0053F120

:00401BD7 46                     inc esi

:00401BD8 8345C804               add dword ptr [ebp-38], 00000004

:00401BDC 83FE0B                 cmp esi, 0000000B

:00401BDF 0F8E32FFFFFF           jle 00401B17

:00401BE5 BE01000000             mov esi, 00000001

:00401BEA 8D85F0FEFFFF           lea eax, dword ptr [ebp+FFFFFEF0]

:00401BF0 8945C4                 mov dword ptr [ebp-3C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401CBB(C)
|

:00401BF3 66C747102C00           mov [edi+10], 002C

:00401BF9 33D2                   xor edx, edx

:00401BFB 8D4DF4                 lea ecx, dword ptr [ebp-0C]

:00401BFE 8955F4                 mov dword ptr [ebp-0C], edx

:00401C01 51                     push ecx

:00401C02 FF471C                 inc [edi+1C]

:00401C05 B901000000             mov ecx, 00000001

:00401C0A 8BD6                   mov edx, esi

:00401C0C 8D450C                 lea eax, dword ptr [ebp+0C]

:00401C0F E884D61300             call 0053F298

:00401C14 837DF400               cmp dword ptr [ebp-0C], 00000000

:00401C18 7405                   je 00401C1F

:00401C1A 8B45F4                 mov eax, dword ptr [ebp-0C]

:00401C1D EB05                   jmp 00401C24

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C18(C)
|

:00401C1F B879815600             mov eax, 00568179

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C1D(U)
|

:00401C24 8A18                   mov bl, byte ptr [eax]

:00401C26 FF4F1C                 dec [edi+1C]

:00401C29 8D45F4                 lea eax, dword ptr [ebp-0C]

:00401C2C BA02000000             mov edx, 00000002

:00401C31 E8EAD41300             call 0053F120

:00401C36 0FBEC3                 movsx eax, bl

:00401C39 83F830                 cmp eax, 00000030

:00401C3C 7C05                   jl 00401C43

:00401C3E 83F839                 cmp eax, 00000039

:00401C41 7E32                   jle 00401C75

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C3C(C)
|

:00401C43 33C0                   xor eax, eax

:00401C45 BA02000000             mov edx, 00000002

:00401C4A 50                     push eax

:00401C4B 8D4508                 lea eax, dword ptr [ebp+08]

:00401C4E FF4F1C                 dec [edi+1C]

:00401C51 E8CAD41300             call 0053F120

:00401C56 FF4F1C                 dec [edi+1C]

:00401C59 8D450C                 lea eax, dword ptr [ebp+0C]

:00401C5C BA02000000             mov edx, 00000002

:00401C61 E8BAD41300             call 0053F120

:00401C66 58                     pop eax

:00401C67 8B17                   mov edx, dword ptr [edi]

:00401C69 64891500000000         mov dword ptr fs:[00000000], edx

:00401C70 E944010000             jmp 00401DB9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401C41(C)
|

:00401C75 66C747103800           mov [edi+10], 0038

:00401C7B 33C9                   xor ecx, ecx

:00401C7D 8D45F0                 lea eax, dword ptr [ebp-10]

:00401C80 894DF0                 mov dword ptr [ebp-10], ecx

:00401C83 50                     push eax

:00401C84 FF471C                 inc [edi+1C]

:00401C87 8D450C                 lea eax, dword ptr [ebp+0C]

:00401C8A B901000000             mov ecx, 00000001

:00401C8F 8BD6                   mov edx, esi

:00401C91 E802D61300             call 0053F298

:00401C96 8D45F0                 lea eax, dword ptr [ebp-10]

:00401C99 E8DED61300             call 0053F37C

:00401C9E 8B55C4                 mov edx, dword ptr [ebp-3C]

:00401CA1 8902                   mov dword ptr [edx], eax

:00401CA3 FF4F1C                 dec [edi+1C]

:00401CA6 8D45F0                 lea eax, dword ptr [ebp-10]

:00401CA9 BA02000000             mov edx, 00000002

:00401CAE E86DD41300             call 0053F120

:00401CB3 46                     inc esi

:00401CB4 8345C404               add dword ptr [ebp-3C], 00000004

:00401CB8 83FE09                 cmp esi, 00000009

:00401CBB 0F8E32FFFFFF           jle 00401BF3

:00401CC1 33F6                   xor esi, esi
```



:00401CC3 8D95BCFEFFFF              `lea edx, dword ptr [ebp+FFFFFEBC]`

:00401CC9 8D8548FFFFFF              `lea eax, dword ptr [ebp+FFFFFF48]`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401CE4(C) |

:00401CCF 8B1A                      `mov ebx, dword ptr [edx]`

:00401CD1 8B08                      `mov ecx, dword ptr [eax]`

:00401CD3 83C004                    `add eax, 00000004`

:00401CD6 83C204                    `add edx, 00000004`

:00401CD9 46                        `inc esi`

:00401CDA 898C9D14FFFFFF           `mov dword ptr [ebp+4*ebx-000000EC], ecx`

:00401CE1 83FE0A                    `cmp esi, 0000000A`

:00401CE4 7EE9                      `jle 00401CCF`

:00401CE6 BE01000000                `mov esi, 00000001`

:00401CEB 8D55A0                    `lea edx, dword ptr [ebp-60]`

:00401CEE 8D8518FFFFFF              `lea eax, dword ptr [ebp+FFFFFF18]`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401D02(C)                       //上面都是一些循环，不知道什么用，没看明白。

:00401CF4 8B08                      `mov ecx, dword ptr [eax]`   //ECX=   (假码：98765432109)

1、取假码第 7 位(3)
2、取假码第 6 位(4)
3、取假码第 8 位(2)
4、取假码第 1 位(9)
5、取假码第 9 位(1)
6、取假码第 3 位(7)
7、取假码第 10 位(0)
8、取假码第 4 位(6)
9、取假码第 5 位(5)

没有假码第 2 位和第 11 位，大概是任意数字吧，往下看。

:00401CF6 890A                      `mov dword ptr [edx], ecx`    //ecx=342917065

:00401CF8 46                        `inc esi`

:00401CF9 83C204                    `add edx, 00000004`

:00401CFC 83C004                    `add eax, 00000004`

:00401CFF 83FE09                    `cmp esi, 00000009`

:00401D02 7EF0                      `jle 00401CF4`

:00401D04 BB04000000                `mov ebx, 00000004`

:00401D09 BE01000000                `mov esi, 00000001`

:00401D0E 3BDE                      `cmp ebx, esi`

:00401D10 7C2B                      `jl 00401D3D`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401D3B(C) |

:00401D12 8D8574FFFFFF              `lea eax, dword ptr [ebp+FFFFFF74]`

:00401D18 50                        `push eax`

:00401D19 8D559C                    `lea edx, dword ptr [ebp-64]`

:00401D1C 52                        `push edx`

:00401D1D E8FAFCFFFF                `call 00401A1C`

:00401D22 83C408                    `add esp, 00000008`

:00401D25 8D4D9C                    `lea ecx, dword ptr [ebp-64]`

:00401D28 51                        `push ecx`

:00401D29 8D8574FFFFFF              `lea eax, dword ptr [ebp+FFFFFF74]`

:00401D2F 50                        `push eax`

:00401D30 E8E7FCFFFF                `call 00401A1C`

:00401D35 83C408                    `add esp, 00000008`

:00401D38 46                        `inc esi`

:00401D39 3BDE                      `cmp ebx, esi`    //ebx=4

:00401D3B 7DD5                      `jge 00401D12`    //这里循环了 4 次，有两个相同的 CALL，所以 342917065(假码的第一次变形)被变形了 8 次，具体跟进 00401A1C 这个 call 看看 -:)

将 342917065 变形后为：586638915

将 586638915 变形后为：508885365

将 508885365 变形后为：055353215

将 055353215 变形后为：550587665

将 550587665 变形后为：141962515

将 141962515 变形后为：295633965

将 295633965 变形后为：027885815

将 027885815 变形后为：284353265

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401D10(C) |

:00401D3D BE01000000                `mov esi, 00000001`

:00401D42 8D95F0FEFFFF              `lea edx, dword ptr [ebp+FFFFFEF0]`

:00401D48 8D45A0                    `lea eax, dword ptr [ebp-60]`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401D8A(C) |

:00401D4B 8B08                      `mov ecx, dword ptr [eax]`    //依次取最终变形后的值 284353265

:00401D4D 3B0A                      `cmp ecx, dword ptr [edx]`    //和机器码 928257269 依次比较

:00401D4F 742F                      `je 00401D80`     //有一个不等则 over，所以这里必须跳

:00401D51 33C0                      `xor eax, eax`

:00401D53 BA02000000                `mov edx, 00000002`

:00401D58 50                        `push eax`

:00401D59 8D4508                    `lea eax, dword ptr [ebp+08]`

:00401D5C FF4F1C                    `dec [edi+1C]`

:00401D5F E8BCD31300                `call 0053F120`

:00401D64 FF4F1C                    `dec [edi+1C]`

:00401D67 8D450C                    `lea eax, dword ptr [ebp+0C]`

:00401D6A BA02000000                `mov edx, 00000002`

:00401D6F E8ACD31300                `call 0053F120`

:00401D74 58                        `pop eax`

:00401D75 8B17                      `mov edx, dword ptr [edi]`

:00401D77 64891500000000            `mov dword ptr fs:[00000000], edx`

:00401D7E EB39                      `jmp 00401DB9`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401D4F(C) |

:00401D80 46                        `inc esi`     //计数器 esi 加 1

:00401D81 83C204                    `add edx, 00000004`

:00401D84 83C004                    `add eax, 00000004`

:00401D87 83FE09                    `cmp esi, 00000009`   //比较 9 次

:00401D8A 7EBF                      `jle 00401D4B`

:00401D8C B001                      `mov al, 01`    //全部正确，则 al 置 1

:00401D8E BA02000000                `mov edx, 00000002`

:00401D93 50                        `push eax`

:00401D94 8D4508                    `lea eax, dword ptr [ebp+08]`

:00401D97 FF4F1C                    `dec [edi+1C]`

:00401D9A E881D31300                `call 0053F120`

:00401D9F FF4F1C                    `dec [edi+1C]`

:00401DA2 8D450C                    `lea eax, dword ptr [ebp+0C]`

:00401DA5 BA02000000                `mov edx, 00000002`

:00401DAA E871D31300                `call 0053F120`

:00401DAF 58                        `pop eax`

:00401DB0 8B17                      `mov edx, dword ptr [edi]`

:00401DB2 64891500000000            `mov dword ptr fs:[00000000], edx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :00401B04(U), :00401B94(U), :00401C70(U), :00401D7E(U) |

:00401DB9 5F                        `pop edi`

:00401DBA 5E                        `pop esi`

:00401DBB 5B                        `pop ebx`

:00401DBC 8BE5                      `mov esp, ebp`

:00401DBE 5D                        `pop ebp`

:00401DBF C3                        `ret`

---------------------------------------------------------------------

跟进 401D1D E8FAFCFFFF `call 00401A1C` 这个 call，以假码第一次变形后的值 342917065 为例：

:00401A1C 55                        `push ebp`

:00401A1D 8BEC                      `mov ebp, esp`

:00401A1F 53                        `push ebx`

:00401A20 8B450C                    `mov eax, dword ptr [ebp+0C]`

:00401A23 8B5508                    `mov edx, dword ptr [ebp+08]`

:00401A26 8B4A24                    `mov ecx, dword ptr [edx+24]`

:00401A29 894824                    `mov dword ptr [eax+24], ecx`

:00401A2C 8B4A20                    `mov ecx, dword ptr [edx+20]`

:00401A2F 3B4A24                    `cmp ecx, dword ptr [edx+24]`  //第 8 位与第 9 位相比

:00401A32 7D04                      `jge 00401A38`    //大于等于则跳

:00401A34 8342200A                  `add dword ptr [edx+20], 0000000A`   //如果小于，这里第 8 位就加上 A

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401A32(C) |

:00401A38 8B4A20                    `mov ecx, dword ptr [edx+20]`   //ecx=第 8 位的值“6”

:00401A3B 2B4A24                    `sub ecx, dword ptr [edx+24]`    //如果第 8 位与第 9 位相等，则 ECX=0；如果不等，则 ECX=第 8 位-第 9 位=1

:00401A3E 894820                    `mov dword ptr [eax+20], ecx`   //将 ECX 的值送入[eax+20]

:00401A41 B907000000                `mov ecx, 00000007`   //ECX 为计数器，初始值为 7

:00401A46 83C020                    `add eax, 00000020`

:00401A49 83C21C                    `add edx, 0000001C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401A66(C) |

:00401A4C 8B1A                      `mov ebx, dword ptr [edx]`    //ebx=

1、第 7 位：0
2、第 6 位：7
3、第 5 位：1
4、第 4 位：9
5、第 3 位：2



6、第 2 位：4
7、第 1 位：3

```
:00401A4E 3B18                cmp ebx, dword ptr [eax]   //[eax]=
```

1、上次第 8 位-第 9 位的值：1
2、下面计算的 `ebx` 的结果：9
3、下面计算的 `ebx` 的结果：8
4、下面计算的 `ebx` 的结果：3
5、下面计算的 `ebx` 的结果：6
6、下面计算的 `ebx` 的结果：6
7、下面计算的 `ebx` 的结果：8

```
:00401A50 7D03                jge 00401A55    //大于等于则跳
:00401A52 83020A              add dword ptr [edx], 0000000A   //否则 ebx 加 A
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :00401A50(C)
|

```
:00401A55 8B1A                mov ebx, dword ptr [edx]    //ecx=
```

1、第 7 位的值 `0+A`
2、第 6 位的值 `7+A`
3、第 5 位的值 `1+A`
4、第 4 位的值 `9`
5、第 3 位的值 `2+A`
6、第 2 位的值 `4+A`
7、第 1 位的值 `3+A`

```
:00401A57 2B18                sub ebx, dword ptr [eax]   //ebx=
```

1、`A-1=9`
2、`11-9=8`
3、`B-8=3`
4、`9-3=6`
5、`C-6=6`
6、`E-6=8`
7、`D-8=5`

```
:00401A59 8958FC              mov dword ptr [eax-04], ebx
:00401A5C 49                  dec ecx    //计数器 ecx 减 1
:00401A5D 83C0FC              add eax, FFFFFFFC
:00401A60 83C2FC              add edx, FFFFFFFC
:00401A63 83F901              cmp ecx, 00000001
:00401A66 7DE4                jge 00401A4C     //ecx 大于等于 1 结束循环，否则跳上去继续
:00401A68 5B                  pop ebx
:00401A69 5D                  pop ebp
:00401A6A C3                  ret
```

//此循环功能将得到的 9 位数字，从尾部开始依次取值，例如取到第 N 位值，减去新值的第 N+1 的值(如果 N 是最后一位，则是减 0，就是说最后一位始终不变，如果不够减，则第 N 位值就加上 10 再去减)，得到的数就是新值的第 N 位值。

//这里 342917065 经过这段处理后，得到 586638915

```
****************************************************************
算法总结：

㈠将注册码 98765432109，按(7->1、6->2、8->3、1->4、9->5、3->6、10->7、4->8、5->9)这样的规律得到 342917065

㈡将 342917065 经过 8 次的处理后的值 284353265 与机器码 928257269 比较

㈢反推：新值第 N 位=旧第 N 位值+旧第 N+1 位值(如果有进位，就只取个位数，最后一位保持不变)，机器码经过 8 次反推，再按步骤 1 规律反归位，在得到的数字的第 2 位和最后一位插上任意数字就得到了正确的注册码。

㈣我的机器码是：928257269 ，反推：             

1 推---100729859
2 推---107917349
3 推---176080739
4 推---836887029
5 推---194657219
6 推---030129309
7 推---331312399
8 推---644435289

再按照步骤㈠复位，得到我的注册码：4*58946432*(\*为任意数字)

****************************************************************
```

正确注册后，注册信息保留在以下键值，删除即为未注册版。

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\Hearts\Register]

"user"="leexoyo"
"pass"="41589464321"

                                                             李逍遥[cschina]
                                                              2003.06.30

 4dd466d0e1e0cc123afbe504da2462ca
```
