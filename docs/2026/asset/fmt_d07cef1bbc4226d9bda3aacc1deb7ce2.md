
--->对应的就是注册码的第5位

`3+1=4` --->对应`"HACKERYouMUSTDie"`的第4位--->`"K"`

--->对应的就是注册码的第10位

`1+1=2` --->对应`"HACKERYouMUSTDie"`的第2位--->`"A"`

--->对应的就是注册码的第11位

`0+1=1` --->对应`"HACKERYouMUSTDie"`的第1位--->`"H"`

--->对应的就是注册码的第12位

`A+1=B` --->对应`"HACKERYouMUSTDie"`的第11位--->`"U"`

--->对应的就是注册码的第14位

`C+1=D` --->对应`"HACKERYouMUSTDie"`的第13位--->`"T"`

--->对应的就是注册码的第15位

推出注册码前34位就是`"1K3RE6789KAH3UTC789RK2R45HCCi01234"`

再根据`00470BF0`和`00470BFB`行，得出最后两位`oU`

```
:00471013 8D45F8                   lea eax, dword ptr [ebp-08]
:00471016 BA10000000               mov edx, 00000010
:0047101B E82436F9FF               call 00404644
:00471020 BE01000000               mov esi, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047103C(C)
|
:00471025 8D45F8                   lea eax, dword ptr [ebp-08]
:00471028 E8E334F9FF               call 00404510
* Possible StringData Ref from Code Obj ->"HACKERYouMUSTDie"
           |
:0047102D BA04114700               mov edx, 00471104
:00471032 8A12                     mov dl, byte ptr [edx]
:00471034 885430FF                 mov byte ptr [eax+esi-01], dl <===晕，这个循环无非就是给出16个H
:00471038 46                       inc esi
:00471039 83FE11                   cmp esi, 00000011  <===共循环16次
:0047103C 75E7                     jne 00471025    <===从这里向上跳构成循环结构
:0047103E 8D45F8                   lea eax, dword ptr [ebp-08]  <==="HHHHHHHHHHHHHHHH"
:00471041 E8CA34F9FF               call 00404510
:00471046 8A531F                   mov dl, byte ptr [ebx+1F]  <===取第32位2
:00471049 88500D                   mov byte ptr [eax+0D], dl  <===放入H串的第14位"HHHHHHHHHHHHH2HH"
:0047104C 8D45F8                   lea eax, dword ptr [ebp-08]
:0047104F E8BC34F9FF               call 00404510
:00471054 8A5320                   mov dl, byte ptr [ebx+20]  <===取第33位3
:00471057 88500E                   mov byte ptr [eax+0E], dl  <===放入H串的第15位"HHHHHHHHHHHHH23H"
:0047105A 8D45F8                   lea eax, dword ptr [ebp-08]
:0047105D E8AE34F9FF               call 00404510
:00471062 8A5321                   mov dl, byte ptr [ebx+21]  <===取第34位4
:00471065 88500F                   mov byte ptr [eax+0F], dl  <===放入H串的第16位"HHHHHHHHHHHHH234"
:00471068 8B45F8                   mov eax, dword ptr [ebp-08]
:0047106B E87CFAFFFF               call 00470AEC  <===又是这个特殊处理的CALL，这次处理的是"HHHHHHHHHHHHH234"
:00471070 8945E8                   mov dword ptr [ebp-18], eax <===得出值3 为0
  ******经过仔细观察这里的EAX的值就是后三位字符在"HACKERYouMUSTDie"中的位置减1，例如：后三位为Die，那么这里的EAX=DEF，如果是ACK，那么EAX=123(也就是说EAX对应的只是"HHHHHHHHHHHHH234"的后8位)
:00471073 8955EC                   mov dword ptr [ebp-14], edx <===得出值4 为0 ，这里EDX始终为0(对应"HHHHHHHHHHHHH234"中的前8位，因为都是H，所以EDX必定等于0)
:00471076 33DB                     xor ebx, ebx
:00471078 8B45F0                   mov eax, dword ptr [ebp-10] <===取出值1 为0，要想正确注册码则[ebp-10]=0x2535022E，向上看0047100D行
:0047107B 8B55F4                   mov edx, dword ptr [ebp-0C] <===取出值2 为0，[ebp-0C]就应该为0x354310AC，再向上看00471010行
:0047107E 2B45E8                   sub eax, dword ptr [ebp-18] <===由于注册码的第32、33、34为234，所以[ebp-18]=0，那么EAX=0x2535022E，再向上看00471078
:00471081 1B55EC                   sbb edx, dword ptr [ebp-14] <===由于[ebp-14]始终为0，所以EDX应该为0x354310AC,向上看0047107B行
:00471084 8945F0                   mov dword ptr [ebp-10], eax <===算出的结果又放入特殊位1，则EAX应该为0x2535022E
:00471087 8955F4                   mov dword ptr [ebp-0C], edx <===算出的结果又放入特殊位2，则EDX应该为0x354310AC
:0047108A FF35D4674700             push dword ptr [004767D4]   <===内定值？ 为0
:00471090 FF35D0674700             push dword ptr [004767D0]   <===内定值？ 为0x31EC0A2
:00471096 6A00                     push 00000000
:00471098 6A01                     push 00000001
:0047109A FF35DC674700             push dword ptr [004767DC]   <===内定值？ 为0
:004710A0 FF35D8674700             push dword ptr [004767D8]   <===内定值？ 为0x12C05687
:004710A6 E8B5F8FFFF               call 00470960  <===进行处理后，得出不同的EAX和EDX值
:004710AB 8945E0                   mov dword ptr [ebp-20], eax <===得出值5  为031EC0A2   (因为是内定值，所以这里也是固定的)
:004710AE 8955E4                   mov dword ptr [ebp-1C], edx <===得出值6  为0          (因为是内定值，所以这里也是固定的)
:004710B1 8B45F0                   mov eax, dword ptr [ebp-10]
:004710B4 8B55F4                   mov edx, dword ptr [ebp-0C]
:004710B7 3345E0                   xor eax, dword ptr [ebp-20]  <===[ebp-20]=0x31EC0A2(定值)，推出EAX为0x2535022E,向上看00471084行
:004710BA 3355E4                   xor edx, dword ptr [ebp-1C]  <===[ebp-1C]因为是定值为0，所以EDX应该为0x354310AC，向上看471087行
:004710BD 3B15E4674700             cmp edx, dword ptr [004767E4] <===这里也必须相等，而[004767E4]却为0x354310AC，所以EDX也应该为0x354310AC，向上看
:004710C3 7506                     jne 004710CB
:004710C5 3B05E0674700             cmp eax, dword ptr [004767E0] <===要想成功，则这里必须两等(两处都必须相等),[004767E0]为0x262BC28C，那么EAX也应该为0x262BC28C，向上看004710B7行
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004710C3(C)
|
:004710CB 7502                     jne 004710CF <===这里不能跳了
:004710CD B301                     mov bl, 01   <===世上最高兴的事莫于经过此处
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004710CB(C)
|
:004710CF 33C0                     xor eax, eax
:004710D1 5A                       pop edx
:004710D2 59                       pop ecx
:004710D3 59                       pop ecx
:004710D4 648910                   mov dword ptr fs:[eax], edx
:004710D7 68F1104700               push 004710F1
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004710EF(U)
|
:004710DC 8D45F8                   lea eax, dword ptr [ebp-08]
:004710DF BA02000000               mov edx, 00000002
:004710E4 E8332FF9FF               call 0040401C
:004710E9 C3                       ret
:004710EA E90D29F9FF               jmp 004039FC
:004710EF EBEB                     jmp 004710DC
:004710F1 8BC3                     mov eax, ebx  <===最后一处关键的赋值，向上看EBX的值
:004710F3 5E                       pop esi
:004710F4 5B                       pop ebx
:004710F5 8BE5                     mov esp, ebp
:004710F7 5D                       pop ebp
:004710F8 C3                       ret
-----------所有的迷都在这里面只是F8跟进了:-)------------------------------
:00470AEC 53                       push ebx
:00470AED 56                       push esi
:00470AEE 57                       push edi
:00470AEF 55                       push ebp
:00470AF0 83C4F8                   add esp, FFFFFFF8
:00470AF3 8BF8                     mov edi, eax
:00470AF5 BDE07B4700               mov ebp, 00477BE0
:00470AFA C7042400000000           mov dword ptr [esp], 00000000
:00470B01 C744240400000000         mov [esp+04], 00000000
:00470B09 BE10000000               mov esi, 00000010
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470B53(C)       **********外部循环**********
|
:00470B0E BB01000000               mov ebx, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470B4E(C)    **********内部循环**********
|
:00470B13 8A4437FF                 mov al, byte ptr [edi+esi-01]  <===依次反向取2450124560136789的字符
* Possible StringData Ref from Code Obj ->"HACKERYouMUSTDie"
           |
:00470B17 BA6C0B4700               mov edx, 00470B6C
:00470B1C 3A441AFF                 cmp al, byte ptr [edx+ebx-01]  <===依次取HACKERYouMUSTDie的字符
:00470B20 7528                     jne 00470B4A   <===如果不相等就跳出不做任何处理
:00470B22 8BC3                     mov eax, ebx
:00470B24 48                       dec eax
:00470B25 99                       cdq
:00470B26 52                       push edx
:00470B27 50                       push eax
:00470B28 B811000000               mov eax, 00000011
:00470B2D 2BC6                     sub eax, esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470AE0(C)
|
:00470B2F 8B54C5FC                 mov edx, dword ptr [ebp+8*eax-04] <===有个列表每隔8位一个数字<对应下面码表的高4位>
:00470B33 8B44C5F8                 mov eax, dword ptr [ebp+8*eax-08] <===每隔8位一个数字<对应下面码表的低4位>
---------------码表(这是一个极有意思的码表)（共16行）---------------------------
00477BE0  01 00 00 00 00 00 00 00  .......
00477BE8  10 00 00 00 00 00 00 00  .......
00477BF0  00 01 00 00 00 00 00 00  .......
00477BF8  00 10 00 00 00 00 00 00  .......
00477C00  00 00 01 00 00 00 00 00  .......
00477C08  00 00 10 00 00 00 00 00  .......
00477C10  00 00 00 01 00 00 00 00  .......
00477C18  00 00 00 10 00 00 00 00  .......
00477C20  00 00 00 00 01 00 00 00  .......
00477C28  00 00 00 00 10 00 00 00  .......
```
