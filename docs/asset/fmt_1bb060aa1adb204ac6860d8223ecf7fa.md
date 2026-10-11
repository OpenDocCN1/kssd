
```
* Possible StringData Ref from Data Obj ->"REGISTERED USER"   
  
                   |   

:00453B5A BA3C3F4500               mov edx, 00453F3C   

:00453B5F E89004FBFF               call 00403FF4   

:00453B64 7504                     jne 00453B6A   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B55(C)   

|   

:00453B66 33C0                     xor eax, eax   

:00453B68 EB02                     jmp 00453B6C   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B64(C)   

|   

:00453B6A B001                     mov al, 01   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B68(U)   

|   

:00453B6C 8BD8                     mov ebx, eax   

:00453B6E 84DB                     test bl, bl   

:00453B70 740D                     je 00453B7F   

:00453B72 8B45F8                   mov eax, dword ptr [ebp-08]   

:00453B75 E86A03FBFF               call 00403EE4   

:00453B7A 83F80A                   cmp eax, 0000000A                 <-NAME 的位数大于等于 10 位   

:00453B7D 7D04                     jge 00453B83   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B70(C)   

|   

:00453B7F 33C0                     xor eax, eax   

:00453B81 EB02                     jmp 00453B85   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B7D(C)   

|   

:00453B83 B001                     mov al, 01   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B81(U)   

|   

:00453B85 8BD8                     mov ebx, eax   

:00453B87 84DB                     test bl, bl   

:00453B89 7426                     je 00453BB1   

:00453B8B 8D45EC                   lea eax, dword ptr [ebp-14]   

:00453B8E 50                       push eax   

:00453B8F B904000000               mov ecx, 00000004   

:00453B94 BA01000000               mov edx, 00000001   

:00453B99 8B45FC                   mov eax, dword ptr [ebp-04]   

:00453B9C E84705FBFF               call 004040E8   

:00453BA1 8B45EC                   mov eax, dword ptr [ebp-14]   

* Possible StringData Ref from Data Obj ->"BM1-"   
  
                   |   

:00453BA4 BA543F4500               mov edx, 00453F54   

:00453BA9 E84604FBFF               call 00403FF4                 <-C1 是否等于 BM1   

:00453BAE 0F94C3                   sete bl   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453B89(C)   

|   

:00453BB1 84DB                     test bl, bl   

:00453BB3 7456                     je 00453C0B                 <-去死   

:00453BB5 8B45F8                   mov eax, dword ptr [ebp-08]   

:00453BB8 E82703FBFF               call 00403EE4   

:00453BBD 8BD0                     mov edx, eax   

:00453BBF 8D45EC                   lea eax, dword ptr [ebp-14]   

:00453BC2 E84502FBFF               call 00403E0C   

:00453BC7 8B45EC                   mov eax, dword ptr [ebp-14]   

:00453BCA 8D55F0                   lea edx, dword ptr [ebp-10]   

:00453BCD E8B2FDFFFF               call 00453984   

:00453BD2 8D45EC                   lea eax, dword ptr [ebp-14]   

:00453BD5 50                       push eax   

:00453BD6 B902000000               mov ecx, 00000002   

:00453BDB BA05000000               mov edx, 00000005   

:00453BE0 8B45FC                   mov eax, dword ptr [ebp-04]   

:00453BE3 E80005FBFF               call 004040E8   

:00453BE8 8B45EC                   mov eax, dword ptr [ebp-14]   

:00453BEB 50                       push eax   

:00453BEC 8D4DE8                   lea ecx, dword ptr [ebp-18]   

:00453BEF 8B45F0                   mov eax, dword ptr [ebp-10]   

:00453BF2 0FB600                   movzx eax, byte ptr [eax]   

:00453BF5 BA02000000               mov edx, 00000002   

:00453BFA E8F950FBFF               call 00408CF8                 <-生成 C21 C22   

:00453BFF 8B55E8                   mov edx, dword ptr [ebp-18]   <-真的 C21 C22   BPX3   

:00453C02 58                       pop eax                 <-假的 C21 C22   

:00453C03 E8EC03FBFF               call 00403FF4   

:00453C08 0F94C3                   sete bl   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453BB3(C)   

|   

:00453C0B 84DB                     test bl, bl   

:00453C0D 0F8494000000             je 00453CA7                 <-去死   

:00453C13 8D45F0                   lea eax, dword ptr [ebp-10]   

:00453C16 8B55FC                   mov edx, dword ptr [ebp-04]   

:00453C19 E8E200FBFF               call 00403D00   

:00453C1E 8D45F0                   lea eax, dword ptr [ebp-10]   

:00453C21 B902000000               mov ecx, 00000002   

:00453C26 BA07000000               mov edx, 00000007   

:00453C2B E8F804FBFF               call 00404128   

:00453C30 33F6                     xor esi, esi   

:00453C32 8B45F0                   mov eax, dword ptr [ebp-10]   

:00453C35 E8AA02FBFF               call 00403EE4   

:00453C3A 84C0                     test al, al   

:00453C3C 7616                     jbe 00453C54   

:00453C3E B201                     mov dl, 01   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453C52(C)   

|   

:00453C40 33C9                     xor ecx, ecx   

:00453C42 8ACA                     mov cl, dl   

:00453C44 8B5DF0                   mov ebx, dword ptr [ebp-10]   

:00453C47 0FB64C0BFF               movzx ecx, byte ptr [ebx+ecx-01]   

:00453C4C 6603F1                   add si, cx   

:00453C4F 42                       inc edx   

:00453C50 FEC8                     dec al   

:00453C52 75EC                     jne 00453C40   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453C3C(C)   

|   

:00453C54 8D45EC                   lea eax, dword ptr [ebp-14]   

:00453C57 8BD6                     mov edx, esi   

:00453C59 6681E2FF00               and dx, 00FF   

:00453C5E E8A901FBFF               call 00403E0C   

:00453C63 8B45EC                   mov eax, dword ptr [ebp-14]   

:00453C66 8D55F0                   lea edx, dword ptr [ebp-10]   

:00453C69 E816FDFFFF               call 00453984   

:00453C6E 8D45EC                   lea eax, dword ptr [ebp-14]   

:00453C71 50                       push eax   

:00453C72 B902000000               mov ecx, 00000002   

:00453C77 BA07000000               mov edx, 00000007   

:00453C7C 8B45FC                   mov eax, dword ptr [ebp-04]   

:00453C7F E86404FBFF               call 004040E8   

:00453C84 8B45EC                   mov eax, dword ptr [ebp-14]   

:00453C87 50                       push eax   

:00453C88 8D4DE8                   lea ecx, dword ptr [ebp-18]   

:00453C8B 8B45F0                   mov eax, dword ptr [ebp-10]   

:00453C8E 0FB600                   movzx eax, byte ptr [eax]   

:00453C91 BA02000000               mov edx, 00000002   

:00453C96 E85D50FBFF               call 00408CF8                 <-生成 C23 C24  BPX4   

:00453C9B 8B55E8                   mov edx, dword ptr [ebp-18]   <-假的 C23 C24 ?写错了!呵呵没有错啦①   

:00453C9E 58                       pop eax                 <-假的 C23 C24   

:00453C9F E85003FBFF               call 00403FF4   

:00453CA4 0F94C3                   sete bl   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00453C0D(C)   

|   

:00453CA7 84DB                     test bl, bl   

:00453CA9 0F848B000000             je 00453D3A                 <-去死   

:00453CAF 8B55F8                   mov edx, dword ptr [ebp-08]   

* Possible StringData Ref from Data Obj ->"  "   
  
                   |   

:00453CB2 B8643F4500               mov eax, 00453F64   

:00453CB7 E81005FBFF               call 004041CC                 <-NAME 中不能有空格   

:00453CBC 85C0                     test eax, eax   

:00453CBE 7E15                     jle 00453CD5                 <-去死   

:00453CC0 8B55F8                   mov edx, dword ptr [ebp-08]   

* Possible StringData Ref from Data Obj ->"  "   
```
