
以下是关于注册码验证逻辑的反汇编代码分析：

```
:004FD130 50                           push eax
:004FD131 6828004200                   push 00420028

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h

:004FD136 FF15F8104000                 Call dword ptr [004010F8]
:004FD13C 8BF0                         mov esi, eax
:004FD13E 8D4DD8                      lea ecx, dword ptr [ebp-28]
:004FD141 F7DE                         neg esi
:004FD143 1BF6                         sbb esi, esi
:004FD145 F7DE                         neg esi
:004FD147 F7DE                         neg esi

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:004FD149 FF1524124000                 Call dword ptr [00401224]
:004FD14F 8D4DD0                      lea ecx, dword ptr [ebp-30]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

:004FD152 FF1528124000                 Call dword ptr [00401228]
:004FD158 6685F6                       test si, si
:004FD15B 0F84F3040000                 je 004FD654
:004FD161 8B0F                         mov ecx, dword ptr [edi]
:004FD163 57                           push edi
:004FD164 FF9104030000                 call dword ptr [ecx+00000304]
:004FD16A 8D55D0                      lea edx, dword ptr [ebp-30]
:004FD16D 50                           push eax
:004FD16E 52                           push edx
:004FD16F FFD3                         call ebx
:004FD171 8BF0                         mov esi, eax
:004FD173 8D4DD8                      lea ecx, dword ptr [ebp-28]
:004FD176 51                           push ecx
:004FD177 56                           push esi
:004FD178 8B06                         mov eax, dword ptr [esi]
:004FD17A FF90A0000000                 call dword ptr [eax+000000A0]
:004FD180 85C0                         test eax, eax
:004FD182 DBE2                         fclex
:004FD184 7D12                         jge 004FD198
:004FD186 68A0000000                   push 000000A0
:004FD18B 6828084200                   push 00420828
:004FD190 56                           push esi
:004FD191 50                           push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h

:004FD192 FF156C104000                 Call dword ptr [0040106C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD184(C)

:004FD198 8B45D8                       mov eax, dword ptr [ebp-28]
                                     ==== >EAX=13572468          试炼码

:004FD19B 8D55B8                      lea edx, dword ptr [ebp-48]
:004FD19E 8945C0                       mov dword ptr [ebp-40], eax
:004FD1A1 8D45A8                      lea eax, dword ptr [ebp-58]
:004FD1A4 52                           push edx
:004FD1A5 50                           push eax
:004FD1A6 C745D800000000               mov [ebp-28], 00000000
:004FD1AD C745B808000000               mov [ebp-48], 00000008

* Reference To: MSVBVM60.rtcTrimVar, Ord:0208h

:004FD1B4 FF15BC104000                 Call dword ptr [004010BC]
:004FD1BA 8D4DA8                      lea ecx, dword ptr [ebp-58]
:004FD1BD 8D55D4                      lea edx, dword ptr [ebp-2C]
:004FD1C0 51                           push ecx
:004FD1C1 52                           push edx

* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h

:004FD1C2 FF157C114000                 Call dword ptr [0040117C]
:004FD1C8 50                           push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:004FD1C9 FF152C124000                 Call dword ptr [0040122C]
:004FD1CF 8B45E0                       mov eax, dword ptr [ebp-20]
                                     ==== >EAX=50896874          信息码处理的结果

:004FD1D2 DD9D0CFEFFFF                 fstp qword ptr [ebp+FFFFFE0C]
:004FD1D8 50                           push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h

:004FD1D9 FF152C124000                 Call dword ptr [0040122C]

* Reference To: MSVBVM60.__vbaFpI4, Ord:0000h

:004FD1DF FF15F8114000                 Call dword ptr [004011F8]
                                     ==== >将 50896874 转化成 16 进制值 03089FEA（H）

:004FD1E5 99                           cdq
:004FD1E6 B9E9471700                   mov ecx, 001747E9
                                     ==== >ECX=001747E9

:004FD1EB F7F9                         idiv ecx
                                     ==== >EDX=03089FEA % 001747E9=00085AE1

:004FD1ED 6BD20B                       imul edx, 0000000B
                                     ==== >EDX=00085AE1 * B=005BE7AB（H）=6023083（D）

:004FD1F0 0F8018060000                 jo 004FD80E
:004FD1F6 81C20F786F00                 add edx, 006F780F
                                     ==== >EDX=005BE7AB + 006F780F=00CB5FBA（H）=13328314（D）

:004FD1FC 0F800C060000                 jo 004FD80E
:004FD202 8995D0FDFFFF                 mov dword ptr [ebp+FFFFFDD0], edx
:004FD208 DB85D0FDFFFF                 fild dword ptr [ebp+FFFFFDD0]
:004FD20E DD9DC8FDFFFF                 fstp qword ptr [ebp+FFFFFDC8]
:004FD214 DD850CFEFFFF                 fld qword ptr [ebp+FFFFFE0C]

* Reference To: MSVBVM60.__vbaFpR8, Ord:0000h

:004FD21A FF15D0104000                 Call dword ptr [004010D0]
:004FD220 DC9DC8FDFFFF                 fcomp qword ptr [ebp+FFFFFDC8]
                                     ==== >真假码比较！
                                     ==== >ST=13572468.000000000000
                                     ==== >SS[0074E81C]=13328314.00000000

:004FD226 DFE0                         fstsw ax
:004FD228 F6C440                       test ah, 40
:004FD22B 7407                         je 004FD234
                                     ==== >跳则 OVER！

:004FD22D B801000000                   mov eax, 00000001
                                     ==== >置 1 则 OK！

:004FD232 EB02                         jmp 004FD236

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD22B(C)

:004FD234 33C0                         xor eax, eax
                                     ==== >清 0 则 OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004FD232(U)

:004FD236 F7D8                         neg eax
:004FD238 8D4DD4                      lea ecx, dword ptr [ebp-2C]
:004FD23B 668BF0                       mov si, ax

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

:004FD23E FF1524124000                 Call dword ptr [00401224]
:004FD244 8D4DD0                      lea ecx, dword ptr [ebp-30]

* Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h

:004FD247 FF1528124000                 Call dword ptr [00401228]

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

:004FD24D 8B1D34104000                 mov ebx, dword ptr [00401034]
:004FD253 8D55A8                      lea edx, dword ptr [ebp-58]
:004FD256 8D45B8                      lea eax, dword ptr [ebp-48]
:004FD259 52                           push edx
:004FD25A 50                           push eax
:004FD25B 6A02                         push 00000002
:004FD25D FFD3                         call ebx
:004FD25F 83C40C                       add esp, 0000000C
:004FD262 6685F6                       test si, si
:004FD265 0F8455030000                 je 004FD5C0
                                     ==== >跳则 OVER！

:004FD26B 8D7734                      lea esi, dword ptr [edi+34]
                                     ==== >下面保存注册信息！

* Possible StringData Ref from Code Obj ->"sselect * from Regsn"

:004FD26E BAF0134200                   mov edx, 004213F0
:004FD273 8BCE                         mov ecx, esi

…… ……省 略…… ……

* Possible StringData Ref from Code Obj ->"SSn"

:004FD389 B91C144200                   mov ecx, 0042141C
```
