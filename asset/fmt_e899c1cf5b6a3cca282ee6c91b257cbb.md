
* Reference To: `KERNEL32.GlobalLock`, Ord: `0000h`

|

`:004027A6` `E8850F0000`      `Call 00403730`

`:004027AB` `83C4F4`          `add esp, FFFFFFF4`

`:004027AE` `56`              `push esi`

`:004027AF` `50`              `push eax`

* Reference To: `CRTDLL.strcpy`, Ord: `0000h`

|

`:004027B0` `E8230E0000`      `Call 004035D8`

`:004027B5` `83C420`          `add esp, 00000020`

`:004027B8` `83C4F4`          `add esp, FFFFFFF4`

`:004027BB` `53`              `push ebx`

* Reference To: `KERNEL32.GlobalUnlock`, Ord: `0000h`

|

`:004027BC` `E8770F0000`      `Call 00403738`

`:004027C1` `FF3580C04000`    `push dword ptr [0040C080]`

* Reference To: `USER32.OpenClipboard`, Ord: `0000h`

|

`:004027C7` `E8040F0000`      `Call 004036D0`

`:004027CC` `83C4FC`          `add esp, FFFFFFFC`

* Reference To: `USER32.EmptyClipboard`, Ord: `0000h`

|

`:004027CF` `E8040F0000`      `Call 004036D8`

`:004027D4` `83C4F8`          `add esp, FFFFFFF8`

`:004027D7` `53`              `push ebx`

`:004027D8` `6A01`            `push 00000001`

* Reference To: `USER32.SetClipboardData`, Ord: `0000h`

|

`:004027DA` `E8010F0000`      `Call 004036E0`

`:004027DF` `83C4F8`          `add esp, FFFFFFF8`

* Reference To: `USER32.CloseClipboard`, Ord: `0000h`

|

`:004027E2` `E8010F0000`      `Call 004036E8`

`:004027E7` `83C420`          `add esp, 00000020`

`:004027EA` `833DACBD400000`  `cmp dword ptr [0040BDAC], 00000000` <--是否为注册版?

`:004027F1` `740D`            `je 00402800`     <--否!!!!!!!!!!

`:004027F3` `83C4F8`          `add esp, FFFFFFF8`

`:004027F6` `56`              `push esi`

`:004027F7` `8B550C`          `mov edx, dword ptr [ebp+0C]`

`:004027FA` `52`              `push edx`

`:004027FB` `EB60`            `jmp 0040285D`

`:004027FD` `8D7600`          `lea esi, dword ptr [esi+00]`

* Referenced by a `(U)nconditional` or `(C)onditional Jump at Address:`

`:004027F1(C)`

|

`:00402800` `89F7`            `mov edi, esi`

`:00402802` `B000`            `mov al, 00`

`:00402804` `FC`              `cld`

`:00402805` `B9FFFFFFFF`      `mov ecx, FFFFFFFF`

`:0040280A` `F2`              `repnz`

`:0040280B` `AE`              `scasb`

`:0040280C` `F7D1`            `not ecx`

`:0040280E` `8D41FF`          `lea eax, dword ptr [ecx-01]`

`:00402811` `83F803`          `cmp eax, 00000003`     <--密码个数是否大于 3 个
                 `cmp eax, 00000033` <-----改为你想要得个数

`:00402814` `770A`            `ja 00402820`     <--大于 3 个不予显示

`:00402816` `83C4F8`          `add esp, FFFFFFF8`

`:00402819` `56`              `push esi`

`:0040281A` `8B550C`          `mov edx, dword ptr [ebp+0C]`

`:0040281D` `52`              `push edx`

`:0040281E` `EB3D`            `jmp 0040285D`

* Referenced by a `(U)nconditional` or `(C)onditional Jump at Address:`

`:00402814(C)`

|

`:00402820` `83C4F4`          `add esp, FFFFFFF4`

`:00402823` `FF3580C04000`    `push dword ptr [0040C080]`

* Reference To: `USER32.OpenClipboard`, Ord: `0000h`

|

`:00402829` `E8A20E0000`      `Call 004036D0`

`:0040282E` `83C4FC`          `add esp, FFFFFFFC`

* Reference To: `USER32.EmptyClipboard`, Ord: `0000h`

|

`:00402831` `E8A20E0000`      `Call 004036D8`

* Reference To: `USER32.CloseClipboard`, Ord: `0000h`

|

`:00402836` `E8AD0E0000`      `Call 004036E8`

`:0040283B` `83C4F8`          `add esp, FFFFFFF8`

* Possible StringData Ref from Data Obj -> `"unregistered version of A2PR can't "`

                 -> `"show the passwords with length "`

                 -> `"more than 3 symbols"`

|

`:0040283E` `68B0234000`      `push 004023B0`

`:00402843` `8B4D0C`          `mov ecx, dword ptr [ebp+0C]`

`:00402846` `51`              `push ecx`

* Reference To: `CRTDLL.strcat`, Ord: `0000h`

|

`:00402847` `E83C0D0000`      `Call 00403588`

`:0040284C` `83C420`          `add esp, 00000020`

`:0040284F` `EB14`            `jmp 00402865`

* Referenced by a `(U)nconditional` or `(C)onditional Jump at Address:`

`:00402780(C)`

|

`:00402851` `83C4F8`          `add esp, FFFFFFF8`

* Possible StringData Ref from Data Obj -> `"this file not passworded"`

|

`:00402854` `6806244000`      `push 00402406`

`:00402859` `8B450C`          `mov eax, dword ptr [ebp+0C]`

`:0040285C` `50`              `push eax` `4dd466d0e1e0cc123afbe504da2462ca`
