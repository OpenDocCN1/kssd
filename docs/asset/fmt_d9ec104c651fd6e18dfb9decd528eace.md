
:005342E0 FF75DC `push [ebp-24]`

> ① `[ebp-24]=7761746572`

:005342E3 68EC435300 `push 005343EC`

:005342E8 8D55D4 `lea edx, dword ptr [ebp-2C]`

:005342EB 8B45FC `mov eax, dword ptr [ebp-04]`

> EAX=`[ebp-04]=fly`

:005342EE E8FD46EDFF `call 004089F0`

> 将 `fly` 转化为大写字母

:005342F3 8B55D4 `mov edx, dword ptr [ebp-2C]`

> EDX=`FLY`

:005342F6 8D4DD8 `lea ecx, dword ptr [ebp-28]`

:005342F9 8BC3 `mov eax, ebx`

:005342FB E82C080000 `call 00534B2C`

> 对 `FLY` 进行运算得出下面的 `Y1728` 进入！

:00534300 FF75D8 `push [ebp-28]`

> ② `[ebp-28]=Y1728`

:00534303 68EC435300 `push 005343EC`

:00534308 8D55D0 `lea edx, dword ptr [ebp-30]`

* Possible StringData Ref from Code Obj -> "mark"

|

:0053430B B81C445300 `mov eax, 0053441C`

> EAX=`mark`

:00534310 E83FE7FCFF `call 00502A54`

> 取 `mark` 字符的 ASCII 码 `6D61726B`

:00534315 FF75D0 `push [ebp-30]`

> ③ `[ebp-30]=6D61726B`

:00534318 8D55C8 `lea edx, dword ptr [ebp-38]`

:0053431B 8B45F8 `mov eax, dword ptr [ebp-08]`

> EAX=`[ebp-08]=fly@263.net`

:0053431E E80947EDFF `call 00408A2C`

> 对 `fly@263.net` 进行检测，如果有大写字母则转化为小写

:00534323 8B55C8 `mov edx, dword ptr [ebp-38]`

> EDX=`[ebp-38]=fly@263.net`

:00534326 8D4DCC `lea ecx, dword ptr [ebp-34]`

:00534329 8BC3 `mov eax, ebx`

:0053432B E8FC070000 `call 00534B2C`

> 对 `fly@263.net` 进行运算得出下面的 `E7272`

> 与 `005342FB` 处的运算流程相同

:00534330 FF75CC `push [ebp-34]`

> ④ `[ebp-34]=E7272`

:00534333 8B4508 `mov eax, dword ptr [ebp+08]`

:00534336 BA06000000 `mov edx, 00000006`

:0053433B E8D4FCECFF `call 00404014`

> 将上面得出的 ①②③④ 连接成 ①-②-③④

:00534340 EB57 `jmp 00534399`

> “Pro” 版本注册码运算结束！跳走！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005342D1(C)

|

:00534342 8B450C `mov eax, dword ptr [ebp+0C]`

> 下面是第一次时运算 “Plus” 版本的注册码

* Possible StringData Ref from Code Obj -> "Plus"

|

:00534345 BA2C445300 `mov edx, 0053442C`

:0053434A E815FDECFF `call 00404064`

:0053434F 7548 `jne 00534399`

:00534351 8D55C0 `lea edx, dword ptr [ebp-40]`

:00534354 8B45FC `mov eax, dword ptr [ebp-04]`

:00534357 E89446EDFF `call 004089F0`

:0053435C 8B55C0 `mov edx, dword ptr [ebp-40]`

:0053435F 8D4DC4 `lea ecx, dword ptr [ebp-3C]`

:00534362 8BC3 `mov eax, ebx`

:00534364 E8C3070000 `call 00534B2C`

:00534369 FF75C4 `push [ebp-3C]`

:0053436C 68EC435300 `push 005343EC`

:00534371 8D55B8 `lea edx, dword ptr [ebp-48]`

:00534374 8B45F8 `mov eax, dword ptr [ebp-08]`

:00534377 E8B046EDFF `call 00408A2C`

:0053437C 8B55B8 `mov edx, dword ptr [ebp-48]`

:0053437F 8D4DBC `lea ecx, dword ptr [ebp-44]`

:00534382 8BC3 `mov eax, ebx`

:00534384 E8A3070000 `call 00534B2C`

:00534389 FF75BC `push [ebp-44]`

:0053438C 8B4508 `mov eax, dword ptr [ebp+08]`

:0053438F BA03000000 `mov edx, 00000003`

:00534394 E87BFCECFF `call 00404014`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:005342BF(U), :00534340(U), :0053434F(C)

|

:00534399 33C0 `xor eax, eax`

:0053439B 5A `pop edx`

:0053439C 59 `pop ecx`

:0053439D 59 `pop ecx`

:0053439E 648910 `mov dword ptr fs:[eax], edx`

:005343A1 68C3435300 `push 005343C3`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005343C1(U)

|

:005343A6 8D45B8 `lea eax, dword ptr [ebp-48]`

:005343A9 BA12000000 `mov edx, 00000012`

:005343AE E845F9ECFF `call 00403CF8`

:005343B3 8D450C `lea eax, dword ptr [ebp+0C]`

:005343B6 E819F9ECFF `call 00403CD4`

:005343BB C3 `ret`

---

进入 `005342FB call 00534B2C`

* Referenced by a CALL at Addresses:

|:0053427A , :005342AA , :005342FB , :0053432B , :00534364

|:00534384

|

```
:00534B2C 55             push ebp
:00534B2D 8BEC           mov ebp, esp
:00534B2F 6A00           push 00000000
:00534B31 6A00           push 00000000
:00534B33 6A00           push 00000000
:00534B35 6A00           push 00000000
:00534B37 6A00           push 00000000
:00534B39 6A00           push 00000000
:00534B3B 6A00           push 00000000
:00534B3D 53             push ebx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00534ACD(C)

|

```
:00534B3E 56             push esi
:00534B3F 57             push edi
:00534B40 894DF8         mov dword ptr [ebp-08], ecx
:00534B43 8955FC         mov dword ptr [ebp-04], edx
:00534B46 8B45FC         mov eax, dword ptr [ebp-04]
```

> EAX=`FLY`

```
:00534B49 E8BAF5ECFF     call 00404108
:00534B4E 33C0           xor eax, eax
:00534B50 55             push ebp
:00534B51 68154C5300     push 00534C15
:00534B56 64FF30         push dword ptr fs:[eax]
:00534B59 648920         mov dword ptr fs:[eax], esp
:00534B5C 33FF           xor edi, edi
:00534B5E 8D45F4         lea eax, dword ptr [ebp-0C]
:00534B61 8B55FC         mov edx, dword ptr [ebp-04]
```

> EDX=`FLY`

```
:00534B64 E803F2ECFF     call 00403D6C
:00534B69 8B45F4         mov eax, dword ptr [ebp-0C]
:00534B6C E8E3F3ECFF     call 00403F54
```

> 取 `FLY` 长度

```
:00534B71 8BF0           mov esi, eax
```

> ESI=EAX=3

```
:00534B73 85F6           test esi, esi
:00534B75 7E58           jle 00534BCF
:00534B77 BB01000000     mov ebx, 00000001
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00534BCD(C)

|

```
:00534B7C 8B45F4         mov eax, dword ptr [ebp-0C]
```

> EAX=`FLY`

```
:00534B7F 8A4418FF       mov al, byte ptr [eax+ebx-01]
```

> 依次取 `FLY` 字符的 `HEX` 值

> 1、 AL=46

> 2、 AL=4C

> 3、 AL=59

```
:00534B83 E858FFFFFF     call 00534AE0
```
