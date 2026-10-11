
# 化学金排 5.20

> 原文：[`www.pediy.com/kssd/pediy06/pediy6096.htm`](https://www.pediy.com/kssd/pediy06/pediy6096.htm)

化学金排 5.20(理科工具) 破解手记--注册算法 

作者：HAILDUZ\[CCG\]  

软件名称：化学金排 5.20(理科工具)  

整理日期：2003.6.3  

文件大小：3.15MB  

软件授权：共享软件  

加密方式：注册码  

使用工具：`TRW2000`; `W32Dasm 10.0`   

作者申明：纯技术交流，无任何商业目的,转贴请保持完整。  

/////////////////////////////////////////////////////////////////////////////////////////////  

开始 `TRW2000` 加载输入注册码，下 `BPX HMEMCPY`，点“注册”，程序被拦断，  

`BC *`，`PMODULE` 后停在这里  

```
:004419B0 3BC3                 cmp eax, ebx               
:004419B2 DBE2                 fclex
:004419B4 7D12                 jge 004419C8
:004419B6 68A0000000           push 000000A0
:004419BB 68FCE24000           push 0040E2FC
:004419C0 57                   push edi
:004419C1 50                   push eax
```
`F10` 跟踪 经过\*\*\*\*\*\*\*\*!!!!!!%%%%%%（好累！）到 `PART1`  

`/////////////////////////////////////PART1////////////////////////////////////////////////`  

```
:00441C6C FF1510114000         Call dword ptr [00401110]
:00441C72 8BF8                 mov edi, eax
:00441C74 8D55E4               lea edx, dword ptr [ebp-1C]
:00441C77 F7DF                 neg edi
:00441C79 1BFF                 sbb edi, edi
:00441C7B 52                   push edx
:00441C7C 47                   inc edi
:00441C7D F7DF                 neg edi
:00441C7F E86C220000           call 00443EF0    /关键 CALL 跟进 见 PART2
:00441C84 6648                 dec ax
:00441C86 8D4DE4               lea ecx, dword ptr [ebp-1C]
:00441C89 66F7D8               neg ax
:00441C8C 1BC0                 sbb eax, eax
:00441C8E 40                   inc eax
:00441C8F F7D8                 neg eax
:00441C91 0BF8                 or edi, eax
:00441C93 8D45E0               lea eax, dword ptr [ebp-20]
:00441C96 50                   push eax
:00441C97 51                   push ecx
:00441C98 6A02                 push 00000002
```
`/////////////////////////////////////PART2////////////////////////////////////////////////`  

* Referenced by a CALL at Addresses:  

`|:00429701 , :0042A94E , :00434736 , :00434A9A , :00435D1F`   
`|:00438CA3 , :0043D459 , :00441C7F , :004426D6 , :00442D79`   
`|`  

```
:00443EF0 55                   push ebp
:00443EF1 8BEC                 mov ebp, esp
:00443EF3 83EC08               sub esp, 00000008
:00443EF6 68662A4000           push 00402A66
:00443EFB 64A100000000         mov eax, dword ptr fs:[00000000]
:00443F01 50                   push eax
:00443F02 64892500000000       mov dword ptr fs:[00000000], esp
:00443F09 81ECB4000000         sub esp, 000000B4
:00443F0F 53                   push ebx
:00443F10 56                   push esi
:00443F11 57                   push edi
:00443F12 8965F8               mov dword ptr [ebp-08], esp
:00443F15 C745FC80254000       mov [ebp-04], 00402580
:00443F1C 8B4508               mov eax, dword ptr [ebp+08]
:00443F1F 33F6                 xor esi, esi
:00443F21 8975E0               mov dword ptr [ebp-20], esi
:00443F24 8975DC               mov dword ptr [ebp-24], esi
:00443F27 8B08                 mov ecx, dword ptr [eax]
:00443F29 8975D0               mov dword ptr [ebp-30], esi
:00443F2C 51                   push ecx
:00443F2D 8975CC               mov dword ptr [ebp-34], esi
:00443F30 8975C8               mov dword ptr [ebp-38], esi
:00443F33 8975C4               mov dword ptr [ebp-3C], esi
:00443F36 8975C0               mov dword ptr [ebp-40], esi
:00443F39 8975B0               mov dword ptr [ebp-50], esi
:00443F3C 8975A0               mov dword ptr [ebp-60], esi
:00443F3F 897590               mov dword ptr [ebp-70], esi
:00443F42 897580               mov dword ptr [ebp-80], esi
:00443F45 89B57CFFFFFF         mov dword ptr [ebp+FFFFFF7C], esi
:00443F4B 89B578FFFFFF         mov dword ptr [ebp+FFFFFF78], esi
:00443F51 89B564FFFFFF         mov dword ptr [ebp+FFFFFF64], esi
:00443F57 89B554FFFFFF         mov dword ptr [ebp+FFFFFF54], esi
```

* Reference To: `MSVBVM60.__vbaLenBstr`, Ord:0000h  

```
|
:00443F5D FF1528104000         Call dword ptr [00401028]
:00443F63 83F80E               cmp eax, 0000000E       /eax 返回注册码位数，判断是否为 14 位，否则失败
:00443F66 740E                 je 00443F76
:00443F68 8975D8               mov dword ptr [ebp-28], esi
:00443F6B 6861424400           push 00444261
:00443F70 9B                   wait
:00443F71 E9C8020000           jmp 0044423E
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:00443F66(C)`  

`|`  

```
:00443F76 B802000000           mov eax, 00000002
:00443F7B B901000000           mov ecx, 00000001
:00443F80 8945A0               mov dword ptr [ebp-60], eax
:00443F83 894590               mov dword ptr [ebp-70], eax
:00443F86 894580               mov dword ptr [ebp-80], eax
:00443F89 8D55A0               lea edx, dword ptr [ebp-60]
:00443F8C 894DA8               mov dword ptr [ebp-58], ecx
:00443F8F 894D88               mov dword ptr [ebp-78], ecx
:00443F92 8D4590               lea eax, dword ptr [ebp-70]
:00443F95 52                   push edx
:00443F96 8D4D80               lea ecx, dword ptr [ebp-80]
:00443F99 50                   push eax
:00443F9A 8D9554FFFFFF         lea edx, dword ptr [ebp+FFFFFF54]
:00443FA0 51                   push ecx
:00443FA1 8D8564FFFFFF         lea eax, dword ptr [ebp+FFFFFF64]
:00443FA7 52                   push edx
:00443FA8 8D4DE0               lea ecx, dword ptr [ebp-20]
:00443FAB 50                   push eax
:00443FAC 51                   push ecx
:00443FAD C745980C000000       mov [ebp-68], 0000000C
```

* Reference To: `MSVBVM60.__vbaVarForInit`, Ord:0000h  

```
|
:00443FB4 FF1590104000         Call dword ptr [00401090]
```

* Reference To: `MSVBVM60.__vbaFpI2`, Ord:0000h  

```
|
:00443FBA 8B3540124000         mov esi, dword ptr [00401240]
```

* Reference To: `MSVBVM60.__vbaStrMove`, Ord:0000h  

```
|
:00443FC0 8B3D6C124000         mov edi, dword ptr [0040126C]
```

* Reference To: `MSVBVM60.__vbaFreeVar`, Ord:0000h  

```
|
:00443FC6 8B1D20104000         mov ebx, dword ptr [00401020]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:0044405C(U)`  

`|`  

```
:00443FCC 85C0                 test eax, eax             /循环求前 12 位注册码中的数字和
:00443FCE 0F848D000000         je 00444061       
:00443FD4 8D55B0               lea edx, dword ptr [ebp-50]
:00443FD7 8D45E0               lea eax, dword ptr [ebp-20]
:00443FDA 52                   push edx
:00443FDB 50                   push eax
:00443FDC C745B801000000       mov [ebp-48], 00000001
:00443FE3 C745B002000000       mov [ebp-50], 00000002
```

* Reference To: `MSVBVM60.__vbaI4Var`, Ord:0000h  

```
|
:00443FEA FF1520124000         Call dword ptr [00401220]
:00443FF0 8B4D08               mov ecx, dword ptr [ebp+08]
:00443FF3 50                   push eax
:00443FF4 8B11                 mov edx, dword ptr [ecx]
:00443FF6 52                   push edx
```

* Reference To: `MSVBVM60.rtcMidCharBstr`, Ord:0277h  

```
|
:00443FF7 FF15E8104000         Call dword ptr [004010E8]
:00443FFD 8BD0                 mov edx, eax
:00443FFF 8D4DD0               lea ecx, dword ptr [ebp-30]
```



```
:00444002 FFD7                    call edi

:00444004 50                      push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:00444005 FF15A4124000            Call dword ptr [004012A4]

:0044400B 0FBF45DC               movsx eax, word ptr [ebp-24]

:0044400F 898544FFFFFF           mov dword ptr [ebp+FFFFFF44], eax

:00444015 DB8544FFFFFF           fild dword ptr [ebp+FFFFFF44]

:0044401B DD9D3CFFFFFF           fstp qword ptr [ebp+FFFFFF3C]

:00444021 DC853CFFFFFF           fadd qword ptr [ebp+FFFFFF3C]

:00444027 DFE0                   fstsw ax

:00444029 A80D                   test al, 0D

:0044402B 0F8547020000           jne 00444278

:00444031 FFD6                   call esi

:00444033 8D4DD0                 lea ecx, dword ptr [ebp-30]

:00444036 8945DC                 mov dword ptr [ebp-24], eax   /存注册码中数字的累加和

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:00444039 FF15A0124000           Call dword ptr [004012A0]

:0044403F 8D4DB0                 lea ecx, dword ptr [ebp-50]

:00444042 FFD3                   call ebx

:00444044 8D8D54FFFFFF           lea ecx, dword ptr [ebp+FFFFFF54]

:0044404A 8D9564FFFFFF           lea edx, dword ptr [ebp+FFFFFF64]

:00444050 51                      push ecx

:00444051 8D45E0                 lea eax, dword ptr [ebp-20]

:00444054 52                      push edx

:00444055 50                      push eax

* Reference To: MSVBVM60.__vbaVarForNext, Ord:0000h

:00444056 FF1594124000           Call dword ptr [00401294]

:0044405C E96BFFFFFF             jmp 00443FCC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00443FCE(C)

:00444061 8B0D34D04400           mov ecx, dword ptr [0044D034]  /取机器码后两位

:00444067 6A02                   push 00000002

:00444069 51                      push ecx

* Reference To: MSVBVM60.rtcRightCharBstr, Ord:026Ah

:0044406A FF1570124000           Call dword ptr [00401270]

:00444070 8BD0                   mov edx, eax

:00444072 8D4DD0                 lea ecx, dword ptr [ebp-30]

:00444075 FFD7                   call edi

:00444077 50                      push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:00444078 FF15A4124000           Call dword ptr [004012A4]

:0044407E FFD6                   call esi

:00444080 8D4DD0                 lea ecx, dword ptr [ebp-30]

:00444083 8BF0                   mov esi, eax                 /机器码后两位字符串转成数字后存入 esi，以下为生成机器码后两位变码

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:00444085 FF15A0124000           Call dword ptr [004012A0]

:0044408B 6683FE0A               cmp si, 000A             /比较数字是否大于等于 10

:0044408F 7D0A                   jge 0044409B

:00444091 6683C60A               add si, 000A             /小于 10 则该数字加 10

:00444095 0F80E2010000           jo 0044427D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044408F(C)

:0044409B 668BC6                 mov ax, si

:0044409E 66B90A00               mov cx, 000A

:004440A2 6699                   cwd

:004440A4 66F7F9                 idiv cx                  /数字除 10 并判断余数

:004440A7 6685D2                 test dx, dx

:004440AA 750A                   jne 004440B6

:004440AC 6683C609               add si, 0009             /除尽则原数字加 9 再乘 3；除不尽原数字乘 3

:004440B0 0F80C7010000           jo 0044427D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004440AA(C)

:004440B6 666BF603               imul si, 0003

:004440BA 0F80BD010000           jo 0044427D

:004440C0 660375DC               add si, word ptr [ebp-24]   /得到的机器码变码加注册码前 12 位中数字的和。

:004440C4 8D45A0                 lea eax, dword ptr [ebp-60]

:004440C7 8D4DB0                 lea ecx, dword ptr [ebp-50]

:004440CA 8D55DC                 lea edx, dword ptr [ebp-24]

:004440CD 50                      push eax

:004440CE 51                      push ecx

:004440CF 0F80A8010000           jo 0044427D

:004440D5 8975DC                 mov dword ptr [ebp-24], esi  /结果存入 [ebp-24]

:004440D8 8955A8                 mov dword ptr [ebp-58], edx

:004440DB C745A002400000         mov [ebp-60], 00004002

* Reference To: MSVBVM60.rtcVarStrFromVar, Ord:0265h

:004440E2 FF153C124000           Call dword ptr [0040123C]

:004440E8 8D55B0                 lea edx, dword ptr [ebp-50]

:004440EB 8D45D0                 lea eax, dword ptr [ebp-30]

:004440EE 52                      push edx

:004440EF 50                      push eax

* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h

:004440F0 FF15B8114000           Call dword ptr [004011B8]

:004440F6 50                      push eax

* Reference To: MSVBVM60.rtcTrimBstr, Ord:0207h

:004440F7 FF1558104000           Call dword ptr [00401058]

:004440FD 8BD0                   mov edx, eax         /前面计算结果转化成十进制数后再生成最终变码。

:004440FF 8D4DC0                 lea ecx, dword ptr [ebp-40]

:00444102 FFD7                   call edi

:00444104 8B4D08                 mov ecx, dword ptr [ebp+08]

:00444107 8B75C0                 mov esi, dword ptr [ebp-40]

:0044410A 6A02                   push 00000002

:0044410C C745C000000000         mov [ebp-40], 00000000

:00444113 8B11                   mov edx, dword ptr [ecx] /取注册码得到最后两位

:00444115 52                      push edx

* Reference To: MSVBVM60.rtcRightCharBstr, Ord:026Ah

:00444116 FF1570124000           Call dword ptr [00401270]

:0044411C 8BD0                   mov edx, eax

:0044411E 8D4DC8                 lea ecx, dword ptr [ebp-38]

:00444121 FFD7                   call edi

:00444123 50                      push eax

:00444124 6A02                   push 00000002

:00444126 8BD6                   mov edx, esi

:00444128 8D4DCC                 lea ecx, dword ptr [ebp-34]

:0044412B FFD7                   call edi

:0044412D 50                      push eax             /取出最终变码的后两位

* Reference To: MSVBVM60.rtcRightCharBstr, Ord:026Ah

:0044412E FF1570124000           Call dword ptr [00401270]

:00444134 8BD0                   mov edx, eax

:00444136 8D4DC4                 lea ecx, dword ptr [ebp-3C]

:00444139 FFD7                   call edi

:0044413B 50                      push eax

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h

:0044413C FF1510114000           Call dword ptr [00401110]  /比较变码的后两位与注册码后两位相同则成功。
```

### 算法总结

1. 注册码必为 14 位。将注册码前 12 位中数字累加。
2. 将机器码后两位数字变化得到变码，再用变码与前累加和相加，转化成十进制数，取最后两位并与注册码最后两位比较相同则成功。

`HAILDUZ` `[CCG]` 于 2003.6.3

`hailduz@hotmail.com`

`4dd466d0e1e0cc123afbe504da2462ca`
