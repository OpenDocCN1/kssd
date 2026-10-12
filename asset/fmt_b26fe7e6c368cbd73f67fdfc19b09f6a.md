
### 汇编指令片段

```
:0046237D E9E6010000              jmp 00462568
```

—————————————————————————————————

进入关键 `CALL`：`:0046217B` `call dword ptr [ecx+00000728]`

…… ……省略…… ……

```
:004724A8 FFD3                  call ebx
:004724AA 50                    push eax

* Possible StringData Ref from Code Obj ->"CC:\\"
                         |
:004724AB 68A4974200             push 004297A4
:004724B0 8D45CC               lea eax, dword ptr [ebp-34]
:004724B3 50                    push eax
:004724B4 FFD3                  call ebx
:004724B6 50                    push eax
:004724B7 E8EC30FBFF            call 004255A8

* Reference To: MSVBVM60.__vbaSetSystemError, Ord:0000h
                         |
:004724BC FF1598104000           Call dword ptr [00401098]
:004724C2 8B4DC8               mov ecx, dword ptr [ebp-38]

* Reference To: MSVBVM60.__vbaStrToUnicode, Ord:0000h
                         |
:004724C5 8B1D38124000           mov ebx, dword ptr [00401238]
:004724CB 51                    push ecx
:004724CC 8D55C4               lea edx, dword ptr [ebp-3C]
:004724CF 52                    push edx
:004724D0 FFD3                  call ebx
:004724D2 50                    push eax
:004724D3 8B45DC               mov eax, dword ptr [ebp-24]
:004724D6 50                    push eax
:004724D7 57                    push edi

* Reference To: MSVBVM60.__vbaLsetFixstr, Ord:0000h
                         |
:004724D8 FF1594104000           Call dword ptr [00401094]
:004724DE 8B4DC0               mov ecx, dword ptr [ebp-40]
:004724E1 51                    push ecx
:004724E2 8D55BC               lea edx, dword ptr [ebp-44]
:004724E5 52                    push edx
:004724E6 FFD3                  call ebx
:004724E8 50                    push eax
:004724E9 8B45D8               mov eax, dword ptr [ebp-28]
:004724EC 50                    push eax
:004724ED 57                    push edi

* Reference To: MSVBVM60.__vbaLsetFixstr, Ord:0000h
                         |
:004724EE FF1594104000           Call dword ptr [00401094]
:004724F4 8D4DBC               lea ecx, dword ptr [ebp-44]
:004724F7 51                    push ecx
:004724F8 8D55C0               lea edx, dword ptr [ebp-40]
:004724FB 52                    push edx
:004724FC 8D45C4               lea eax, dword ptr [ebp-3C]
:004724FF 50                    push eax
:00472500 8D4DC8               lea ecx, dword ptr [ebp-38]
:00472503 51                    push ecx
:00472504 8D55CC               lea edx, dword ptr [ebp-34]
:00472507 52                    push edx
:00472508 6A05                  push 00000005

* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h
                         |
:0047250A FF15E4124000           Call dword ptr [004012E4]
:00472510 8B5D0C               mov ebx, dword ptr [ebp+0C]
:00472513 8B03                 mov eax, dword ptr [ebx]
                         ==== >EAX=fly-12345678-fly[OCN][FCG]-E  试炼码
:00472515 83C418               add esp, 00000018
:00472518 6A01                 push 00000001
:0047251A 6AFF                 push FFFFFFFF
:0047251C 6A01                 push 00000001
:0047251E 68D0654200             push 004265D0
:00472523 68CC754200             push 004275CC
:00472528 50                    push eax

* Reference To: MSVBVM60.rtcReplace, Ord:02C8h
                         |
:00472529 FF152C124000           Call dword ptr [0040122C]
                         ==== >去除试炼码中的-
:0047252F 8BD0                 mov edx, eax
                         ==== >EDX=fly12345678fly[OCN][FCG]E
:00472531 8D4DD4               lea ecx, dword ptr [ebp-2C]

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
                         |
:00472534 FF1578134000           Call dword ptr [00401378]
:0047253A 8B0B                 mov ecx, dword ptr [ebx]

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
                         |
:0047253C 8B1D34104000           mov ebx, dword ptr [00401034]
:00472542 51                    push ecx
                         ==== >ECX=fly-12345678-fly[OCN][FCG]-E
:00472543 FFD3                  call ebx
                         ==== >取 fly-12345678-fly[OCN][FCG]-E 的长度
:00472545 8BD0                 mov edx, eax
                         ==== >EDX=1C
:00472547 8B45D4               mov eax, dword ptr [ebp-2C]
:0047254A 50                    push eax
                         ==== >EAX=fly12345678fly[OCN][FCG]E
:0047254B 899528FFFFFF           mov dword ptr [ebp+FFFFFF28], edx
                         ==== >[ebp+FFFFFF28]=EDX=1C
:00472551 FFD3                  call ebx
                         ==== >取 fly12345678fly[OCN][FCG]E 的长度=19
:00472553 8B8D28FFFFFF           mov ecx, dword ptr [ebp+FFFFFF28]
                         ==== >ECX=1C
:00472559 8B55D4               mov edx, dword ptr [ebp-2C]
:0047255C 33DB                 xor ebx, ebx
:0047255E 3BC1                 cmp eax, ecx
                         ==== >比较 2 者长度是否相同？既检测试炼码中是否有-
:00472560 52                    push edx
:00472561 0F9DC3               setnl bl
                         ==== >设置 BL 值！有-则长度不同则 BL=0

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
                         |
:00472564 FF1534104000           Call dword ptr [00401034]
                         ==== >取 fly12345678fly[OCN][FCG]E 的长度=19
:0047256A 33C9                 xor ecx, ecx
:0047256C 83F819               cmp eax, 00000019
                         ==== >去除试炼码中的-后是否是 25 位？
:0047256F 0F9CC1               setl cl
                         ==== >设置 CL 值！是 25 位则 CL=0
:00472572 0BD9                 or ebx, ecx
:00472574 0F850C010000           jne 00472686
                         ==== >如果上面 2 个条件都符合则此处不跳！
                         ==== >若此处跳就直接 OVER 了！爆破点①！
:0047257A 8B55D4               mov edx, dword ptr [ebp-2C]
                         ==== >EDX=fly12345678fly[OCN][FCG]E
:0047257D A110804A00             mov eax, dword ptr [004A8010]
                         ==== >EAX=211C1E09          C 盘的硬盘序列号
:00472582 8D4DA4               lea ecx, dword ptr [ebp-5C]
:00472585 89955CFFFFFF           mov dword ptr [ebp+FFFFFF5C], edx
:0047258B 2DCF337B00             sub eax, 007B33CF
                         ==== >EAX=211C1E09 - 007B33CF=20A0EA3A
:00472590 51                    push ecx
:00472591 8D5594               lea edx, dword ptr [ebp-6C]
:00472594 0F8020050000           jo 00472ABA
:0047259A 52                    push edx
:0047259B C78554FFFFFF08000000   mov dword ptr [ebp+FFFFFF54], 00000008
:004725A5 8945AC               mov dword ptr [ebp-54], eax
:004725A8 C745A403000000        mov [ebp-5C], 00000003

* Reference To: MSVBVM60.rtcHexVarFromVar, Ord:023Dh
                         |
:004725AF FF15D8124000           Call dword ptr [004012D8]
:004725B5 6A01                 push 00000001
:004725B7 8D8554FFFFFF           lea eax, dword ptr [ebp+FFFFFF54]
:004725BD 50                    push eax
:004725BE 8D4D94               lea ecx, dword ptr [ebp-6C]
:004725C1 51                    push ecx
:004725C2 6A01                 push 00000001
:004725C4 8D5584               lea edx, dword ptr [ebp-7C]
```
