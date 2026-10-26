
# 汇编代码片段

```
  push eax

:00417E0F A178ED4500              mov eax, dword ptr [0045ED78]

:00417E14 50
  push eax
* Reference To: kernel32.GetCurrentProcess, Ord:0000h
            |
:00417E15 E822CAFEFF              Call 0040483C

:00417E1A 50
  push eax
* Reference To: kernel32.WriteProcessMemory, Ord:0000h
            |
:00417E1B E82CCCFEFF              Call 00404A4C
      <-这个是将注册信息转为程序，再将其写入自身的程序中……这段程序可能是作写入文件的程序

:00417E20 85C0
  test eax, eax

:00417E22 0F84F4000000            je 00417F1C

:00417E28 817C2408BC060000        cmp dword ptr [esp+08], 000006BC

:00417E30 0F85E6000000            jne 00417F1C
* Possible Ref to Menu: MenuID_0001, Item: "einfgen...  Strg+V"
            |
:00417E36 6A20
  push 00000020

:00417E38 68B8514600              push 004651B8

:00417E3D 8D442441                lea eax, dword ptr [esp+41]

:00417E41 50
  push eax
* Reference To: kernel32.lstrcpynA, Ord:0000h
            |
:00417E42 E855CCFEFF              Call 00404A9C

:00417E47 8D442428                lea eax, dword ptr [esp+28]
* Possible Ref to Menu: MenuID_0001, Item: "Sicherung laden..."
            |
:00417E4B BA10000000              mov edx, 00000010

:00417E50 E8A3D2FEFF              call 004050F8
* Possible StringData Ref from Data Obj ->"Offset"
            |
:00417E55 BAE4E14500              mov edx, 0045E1E4

:00417E5A 8D442428                lea eax, dword ptr [esp+28]

:00417E5E E899D3FEFF              call 004051FC

:00417E63 6A00
  push 00000000

:00417E65 8D4C242C                lea ecx, dword ptr [esp+2C]

:00417E69 BA08574000              mov edx, 00405708

:00417E6E 33C0
  xor eax, eax

:00417E70 E883DBFEFF              call 004059F8

:00417E75 8BF0
  mov esi, eax

:00417E77 6A00
  push 00000000

:00417E79 8D54243D                lea edx, dword ptr [esp+3D]
* Possible Ref to Menu: MenuID_0001, Item: "einfgen...  Strg+V"
            |
:00417E7D B920000000              mov ecx, 00000020

:00417E82 8BC6
  mov eax, esi

:00417E84 E85BDCFEFF              call 00405AE4
* Possible Ref to Menu: MenuID_0001, Item: "Hex-Werte  Strg+Shift+C"
            |
:00417E89 BA2C000000              mov edx, 0000002C

:00417E8E 8BC6
  mov eax, esi

:00417E90 E8FFA6FEFF              call 00402594

:00417E95 8D442428                lea eax, dword ptr [esp+28]
* Possible Ref to Menu: MenuID_0001, Item: "Sicherung laden..."
            |
:00417E99 BA10000000              mov edx, 00000010

:00417E9E E855D2FEFF              call 004050F8

:00417EA3 8D442408                lea eax, dword ptr [esp+08]

:00417EA7 50
  push eax
* Possible Ref to Menu: MenuID_0001, Item: "einfgen...  Strg+V"
            |
:00417EA8 6A20
  push 00000020

:00417EAA 8D442441                lea eax, dword ptr [esp+41]

:00417EAE 50
  push eax

:00417EAF A178ED4500              mov eax, dword ptr [0045ED78]

:00417EB4 05A0010000              add eax, 000001A0

:00417EB9 50
  push eax
* Reference To: kernel32.GetCurrentProcess, Ord:0000h
            |
:00417EBA E87DC9FEFF              Call 0040483C

:00417EBF 50
  push eax
* Reference To: kernel32.WriteProcessMemory, Ord:0000h
            |
:00417EC0 E887CBFEFF              Call 00404A4C
      <-这段写入自身的程序可能是作写入内存的程序

:00417EC5 8D442439                lea eax, dword ptr [esp+39]

:00417EC9 BABC060000              mov edx, 000006BC

:00417ECE E825D2FEFF              call 004050F8

:00417ED3 8D442404                lea eax, dword ptr [esp+04]

:00417ED7 50
  push eax

:00417ED8 8B442404                mov eax, dword ptr [esp+04]

:00417EDC 50
  push eax

:00417EDD 8B442420                mov eax, dword ptr [esp+20]

:00417EE1 50
  push eax

:00417EE2 8B442418                mov eax, dword ptr [esp+18]

:00417EE6 50
  push eax
* Reference To: kernel32.VirtualProtect, Ord:0000h
            |
:00417EE7 E828CBFEFF              Call 00404A14

:00417EEC 8B442418                mov eax, dword ptr [esp+18]

:00417EF0 50
  push eax

:00417EF1 8B442410                mov eax, dword ptr [esp+10]

:00417EF5 50
  push eax
* Reference To: kernel32.GetCurrentProcess, Ord:0000h
            |
:00417EF6 E841C9FEFF              Call 0040483C

:00417EFB 50
  push eax
* Reference To: kernel32.FlushInstructionCache, Ord:0000h
            |
:00417EFC E81BC9FEFF              Call 0040481C

:00417F01 C605B465460001          mov byte ptr [004665B4], 01        <-给个标记表明已经更改过自身的程序

:00417F08 A178ED4500              mov eax, dword ptr [0045ED78]

:00417F0D 05BC060000              add eax, 000006BC

:00417F12 48
  dec eax

:00417F13 803800
cmp byte ptr [eax], 00

:00417F16 7504
  jne 00417F1C

:00417F18 B301
  mov bl, 01

:00417F1A EB1D
  jmp 00417F39
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00417D7F(C), :00417E22(C), :00417E30(C), :00417F16(C)
|
* Possible StringData Ref from Data Obj ->"f4"
            |
:00417F1C B8E0EB4500              mov eax, 0045EBE0

:00417F21 E8665CFFFF              call 0040DB8C

:00417F26 B8C05A4600              mov eax, 00465AC0

:00417F2B E86059FFFF              call 0040D890
      <-查找user.txt文件是否存在

:00417F30 84C0
  test al, al

:00417F32 7405
  je 00417F39

:00417F34 E8BF18FFFF              call 004097F8
      <-提示出错信息“Invalid file: user.txt”
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00417D55(U), :00417F1A(U), :00417F32(C)
|
:00417F39 8BC3
  mov eax, ebx

:00417F3B 81C4F8060000            add esp, 000006F8

:00417F41 5E
  pop esi

:00417F42 5B
  pop ebx

:00417F43 C3
  ret
```
