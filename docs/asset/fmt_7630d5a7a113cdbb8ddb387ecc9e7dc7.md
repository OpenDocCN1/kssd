
```
:00527544 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:0052754A BA10795200           mov edx, 00527910
:0052754F E8B0A3F0FF           call 00431904
:00527554 8BC6                 mov eax, esi
:00527556 E88D0BF3FF           call 004580E8
:0052755B 8BC6                 mov eax, esi
:0052755D E80ABAEDFF           call 00402F6C
:00527562 BB01000000           mov ebx, 00000001
:00527567 E910030000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0052753F(C)
|

* Possible StringData Ref from Code Obj ->"Register_2"
                              |
:0052756C BA4C795200           mov edx, 0052794C
:00527571 8BC6                 mov eax, esi
:00527573 E89414F3FF           call 00458A0C  <--读取注册码第二部分
:00527578 84C0                 test al, al
:0052757A 752B                 jne 005275A7  <--有注册码第二部分的话则跳转
:0052757C 8B45FC               mov eax, dword ptr [ebp-04]
:0052757F 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:00527585 BA10795200           mov edx, 00527910
:0052758A E875A3F0FF           call 00431904
:0052758F 8BC6                 mov eax, esi
:00527591 E8520BF3FF           call 004580E8
:00527596 8BC6                 mov eax, esi
:00527598 E8CFB9EDFF           call 00402F6C
:0052759D BB01000000           mov ebx, 00000001
:005275A2 E9D5020000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0052757A(C)
|
:005275A7 8D8D94FEFFFF         lea ecx, dword ptr [ebp+FFFFFE94]
:005275AD BA3F000000           mov edx, 0000003F

* Possible StringData Ref from Code Obj ->"*.*"
                              |
:005275B2 B860795200           mov eax, 00527960
:005275B7 E8741CEEFF           call 00409230
:005275BC 8B85A0FEFFFF         mov eax, dword ptr [ebp+FFFFFEA0]

* Possible StringData Ref from Code Obj ->"OctoDll.dll"
                              |
:005275C2 BA6C795200           mov edx, 0052796C
:005275C7 E87CCAEDFF           call 00404048  <--校验文件 OctoDll.dll 是否有改变
:005275CC 7537                 jne 00527605
:005275CE 81BD98FEFFFF00460000 cmp dword ptr [ebp+FFFFFE98], 00004600
:005275D8 742B                 je 00527605
:005275DA 8B45FC               mov eax, dword ptr [ebp-04]
:005275DD 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:005275E3 BA10795200           mov edx, 00527910
:005275E8 E817A3F0FF           call 00431904
:005275ED 8BC6                 mov eax, esi
:005275EF E8F40AF3FF           call 004580E8
:005275F4 8BC6                 mov eax, esi
:005275F6 E871B9EDFF           call 00402F6C
:005275FB BB01000000           mov ebx, 00000001
:00527600 E977020000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:005275CC(C), :005275D8(C)
|
:00527605 81BD98FEFFFF00680200 cmp dword ptr [ebp+FFFFFE98], 00026800
:0052760F 752B                 jne 0052763C
:00527611 8B45FC               mov eax, dword ptr [ebp-04]
:00527614 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:0052761A BA10795200           mov edx, 00527910
:0052761F E8E0A2F0FF           call 00431904
:00527624 8BC6                 mov eax, esi
:00527626 E8BD0AF3FF           call 004580E8
:0052762B 8BC6                 mov eax, esi
:0052762D E83AB9EDFF           call 00402F6C
:00527632 BB01000000           mov ebx, 00000001
:00527637 E940020000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0052760F(C)
|
:0052763C 8B95A0FEFFFF         mov edx, dword ptr [ebp+FFFFFEA0]

* Possible StringData Ref from Code Obj ->"cr-wom"
                              |
:00527642 B880795200           mov eax, 00527980
:00527647 E8D8CBEDFF           call 00404224  <--检查当前目录下的所有文件的文件名中是否包含有"cr-wom"字眼
:0052764C 85C0                 test eax, eax
:0052764E 742B                 je 0052767B
:00527650 8B45FC               mov eax, dword ptr [ebp-04]
:00527653 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:00527659 BA10795200           mov edx, 00527910
:0052765E E8A1A2F0FF           call 00431904
:00527663 8BC6                 mov eax, esi
:00527665 E87E0AF3FF           call 004580E8
:0052766A 8BC6                 mov eax, esi
:0052766C E8FBB8EDFF           call 00402F6C
:00527671 BB01000000           mov ebx, 00000001
:00527676 E901020000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0052764E(C)
|

* Possible StringData Ref from Code Obj ->"Windowsyhds.exe"
                              |
:0052767B B890795200           mov eax, 00527990
:00527680 E8FB1AEEFF           call 00409180  <--检查当前目录下的所有文件的文件名中是否有"Windowsyhds.exe"
:00527685 84C0                 test al, al
:00527687 751C                 jne 005276A5

* Possible StringData Ref from Code Obj ->"fwd.txt"
                              |
:00527689 B8A8795200           mov eax, 005279A8
:0052768E E8ED1AEEFF           call 00409180  <--检查当前目录下的所有文件的文件名中是否有"fwd.txt"
:00527693 84C0                 test al, al
:00527695 750E                 jne 005276A5

* Possible StringData Ref from Code Obj ->"wom29a_k.exe"
                              |
:00527697 B8B8795200           mov eax, 005279B8
:0052769C E8DF1AEEFF           call 00409180  <--检查当前目录下的所有文件的文件名中是否有"wom29a_k.exe"
:005276A1 84C0                 test al, al
:005276A3 742B                 je 005276D0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00527687(C), :00527695(C)
|
:005276A5 8B45FC               mov eax, dword ptr [ebp-04]
:005276A8 8B8064050000         mov eax, dword ptr [eax+00000564]

* Possible StringData Ref from Code Obj ->"Windows 优化大师 V4.2 (未注册)"
                              |
:005276AE BA10795200           mov edx, 00527910
:005276B3 E84CA2F0FF           call 00431904
:005276B8 8BC6                 mov eax, esi
:005276BA E8290AF3FF           call 004580E8
:005276BF 8BC6                 mov eax, esi
:005276C1 E8A6B8EDFF           call 00402F6C
:005276C6 BB01000000           mov ebx, 00000001
:005276CB E9AC010000           jmp 0052787C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:005276A3(C), :005277FB(C)
|
:005276D0 8D8594FEFFFF         lea eax, dword ptr [ebp+FFFFFE94]
:005276D6 E8A51BEEFF           call 00409280
:005276DB 8BD8                 mov ebx, eax
```
