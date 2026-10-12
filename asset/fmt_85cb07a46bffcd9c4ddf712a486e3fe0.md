
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
