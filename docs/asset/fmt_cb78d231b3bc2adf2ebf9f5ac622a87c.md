
```
:00403CB6 6800F14200               push 0042F100   
:00403CBB 51                       push ecx   
:00403CBC FFD7                     call edi   
:00403CBE 6AFF                     push FFFFFFFF   
:00403CC0 8D4C2414                 lea ecx, dword ptr [esp+14]   
:00403CC4 E8BF820100               call 0041BF88   
:00403CC9 8B542414                 mov edx, dword ptr [esp+14]   
:00403CCD 8D4C2414                 lea ecx, dword ptr [esp+14]   
:00403CD1 8B42F8                   mov eax, dword ptr [edx-08]   
:00403CD4 50                       push eax   
:00403CD5 50                       push eax   
:00403CD6 E85E820100               call 0041BF39   
:00403CDB 50                       push eax   
:00403CDC 8B442420                 mov eax, dword ptr [esp+20]   
:00403CE0 6A01                       push 00000001   
:00403CE2 53                       push ebx   
* Possible StringData Ref from Data Obj ->"RegName"   
                                   |   
:00403CE3 68C8F04200               push 0042F0C8   
:00403CE8 50                       push eax   
:00403CE9 FFD7                     call edi   
:00403CEB 6AFF                     push FFFFFFFF   
:00403CED 8D4C2418                 lea ecx, dword ptr [esp+18]   
:00403CF1 E892820100               call 0041BF88   
:00403CF6 8B4C2418                 mov ecx, dword ptr [esp+18]   
:00403CFA 51                       push ecx   
* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh   
                                   |   
:00403CFB FF1500504200             Call dword ptr [00425000]   
:00403D01 6A40                     push 00000040   
==>上面的大概过程是：把已注册成功的名称和注册码写入到注册表相关的键值中   
* Possible StringData Ref from Data Obj ->"软件注册"   
                                   |   
:00403D03 68F8F24200               push 0042F2F8   
* Possible StringData Ref from Data Obj ->"注册成功"   
                                   |   
:00403D08 68ECF24200               push 0042F2EC   
:00403D0D 8BCE                     mov ecx, esi   
:00403D0F E877630100               call 0041A08B   
:00403D14 8B0D70134300             mov ecx, dword ptr [00431370]   
* Possible StringData Ref from Data Obj ->"卸载精灵"   
                                   |   
:00403D1A 689CF24200               push 0042F29C   
:00403D1F E85D750100               call 0041B281   
:00403D24 8BCE                     mov ecx, esi   
:00403D26 E8C8470100               call 004184F3   
:00403D2B 8D4C241C                 lea ecx, dword ptr [esp+1C]   
:00403D2F C644242801               mov [esp+28], 01   
:00403D34 E82D7F0100               call 0041BC66   
:00403D39 EB7F                     jmp 00403DBA   
==>上面是：软件注册成功后，软件给出的提示信息   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00403C59(C)   
|   
:00403D3B 6A10                     push 00000010   
* Possible StringData Ref from Data Obj ->"软件注册"   
                                   |   
:00403D3D 68F8F24200               push 0042F2F8   
* Possible StringData Ref from Data Obj ->"注册码错误！请重新输入"   
                                   |   
:00403D42 68D4F24200               push 0042F2D4   
:00403D47 8BCE                     mov ecx, esi   
:00403D49 E83D630100               call 0041A08B   
:00403D4E 8B542410                 mov edx, dword ptr [esp+10]   
* Reference To: USER32.SendMessageA, Ord:0214h   
                                   |   
:00403D52 8B35A4544200             mov esi, dword ptr [004254A4]   
:00403D58 8B42F8                   mov eax, dword ptr [edx-08]   
:00403D5B 50                       push eax   
:00403D5C 8B451C                   mov eax, dword ptr [ebp+1C]   
:00403D5F 53                       push ebx   
:00403D60 68B1000000               push 000000B1   
:00403D65 50                       push eax   
:00403D66 FFD6                     call esi   
:00403D68 8B4D1C                   mov ecx, dword ptr [ebp+1C]   
:00403D6B 53                       push ebx   
:00403D6C 53                       push ebx   
:00403D6D 68B7000000               push 000000B7   
:00403D72 51                       push ecx   
:00403D73 FFD6                     call esi   
:00403D75 8BCD                     mov ecx, ebp   
:00403D77 EB3C                     jmp 00403DB5   
==>上面是：软件注册失败后，软件给出的提示信息   
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00403C1B(C), :00403C35(C)   
|   
:00403D79 6A10                     push 00000010   
* Possible StringData Ref from Data Obj ->"软件注册"   
                                   |   
:00403D7B 68F8F24200               push 0042F2F8   
* Possible StringData Ref from Data Obj ->"用户名及注册码不能为空"   
                                   |   
:00403D80 68BCF24200               push 0042F2BC   
:00403D85 8BCE                     mov ecx, esi   
:00403D87 E8FF620100               call 0041A08B   
:00403D8C 8B542414                 mov edx, dword ptr [esp+14]   
* Reference To: USER32.SendMessageA, Ord:0214h   
                                   |   
:00403D90 8B35A4544200             mov esi, dword ptr [004254A4]   
:00403D96 8B42F8                   mov eax, dword ptr [edx-08]   
:00403D99 50                       push eax   
:00403D9A 8B471C                   mov eax, dword ptr [edi+1C]   
:00403D9D 53                       push ebx   
:00403D9E 68B1000000               push 000000B1   
:00403DA3 50                       push eax   
:00403DA4 FFD6                     call esi   
:00403DA6 8B4F1C                   mov ecx, dword ptr [edi+1C]   
:00403DA9 53                       push ebx   
:00403DAA 53                       push ebx   
:00403DAB 68B7000000               push 000000B7   
:00403DB0 51                       push ecx   
:00403DB1 FFD6                     call esi   
:00403DB3 8BCF                     mov ecx, edi   
==>上面是：软件注册过程中，如果你输入的用户名及注册码为空时，软件给出的提示信息   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00403D77(U)   
|   
:00403DB5 E831760100               call 0041B3EB   
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00403D39(U)   
|   
:00403DBA 8D4C2410                 lea ecx, dword ptr [esp+10]   
:00403DBE 885C2428                 mov byte ptr [esp+28], bl   
:00403DC2 E89F7E0100               call 0041BC66   
:00403DC7 8D4C2414                 lea ecx, dword ptr [esp+14]   
:00403DCB C7442428FFFFFFFF         mov [esp+28], FFFFFFFF   
:00403DD3 E88E7E0100               call 0041BC66   
:00403DD8 8B4C2420                 mov ecx, dword ptr [esp+20]   
:00403DDC 5F                       pop edi   
:00403DDD 5E                       pop esi   
:00403DDE 5D                       pop ebp   
:00403DDF 64890D00000000           mov dword ptr fs:[00000000], ecx   
:00403DE6 5B                       pop ebx   
:00403DE7 83C41C                   add esp, 0000001C   
:00403DEA C3                       ret   
```

跟踪进入 `00403930` 后：

* Referenced by a CALL at Addresses:
|`:004037D3` , `:00403C52`  
|

```
:00403930 64A100000000             mov eax, dword ptr fs:[00000000]   
:00403936 6AFF                     push FFFFFFFF   
:00403938 6838394200               push 00423938   
:0040393D 50                       push eax   
:0040393E 64892500000000           mov dword ptr fs:[00000000], esp   
:00403945 56                       push esi   
:00403946 8B442414                 mov eax, dword ptr [esp+14]   
==>EAX 中存放着你输入的注册码   
:0040394A 8B48F8                   mov ecx, dword ptr [eax-08]   
:0040394D 83F908                   cmp ecx, 00000008   
==>判断你输入的注册码位数是否为 8 位   
:00403950 7425                     je 00403977   
==>如果注册码位数为 8 位，则跳转；不跳转将注册失败   
:00403952 8D4C2414                 lea ecx, dword ptr [esp+14]   
:00403956 C744240CFFFFFFFF         mov [esp+0C], FFFFFFFF   
:0040395E E803830100               call 0041BC66   
:00403963 33C0                     xor eax, eax   
:00403965 8B4C2404                 mov ecx, dword ptr [esp+04]   
```
