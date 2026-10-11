
`4A 4A 56 50 43 4E 58 41 4B 4C`分别所对应的字符是：`JJVPCNXAKL`

```
:0047068B E902FEFFFF          jmp 00470492
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004705F7(C)`

|

```
:00470690 C745FC23000000     mov [ebp-04], 00000023

:00470697 E96AFCFFFF          jmp 00470306
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00470321(C)`

|

```
:0047069C C745FC24000000     mov [ebp-04], 00000024

:004706A3 B909000000         mov ecx, 00000009
```

* Reference To: `MSVBVM50.__vbaUI1I2`, Ord: `0000h`

```
:004706A8 FF1558A44700       Call dword ptr [0047A458]

:004706AE 888510FFFFFF       mov byte ptr [ebp+FFFFFF10], al

:004706B4 C68514FFFFFF01     mov byte ptr [ebp+FFFFFF14], 01

:004706BB 33C9               xor ecx, ecx
```

* Reference To: `MSVBVM50.__vbaUI1I2`, Ord: `0000h`

```
:004706BD FF1558A44700       Call dword ptr [0047A458]

:004706C3 8845DC             mov byte ptr [ebp-24], al

:004706C6 EB12               jmp 004706DA
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0047088E(U)`

|

```
:004706C8 8A55DC             mov dl, byte ptr [ebp-24]

:004706CB 029514FFFFFF       add dl, byte ptr [ebp+FFFFFF14]

:004706D1 0F829B020000       jb 00470972

:004706D7 8855DC             mov byte ptr [ebp-24], dl
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004706C6(U)`

|

```
:004706DA 8A45DC             mov al, byte ptr [ebp-24]

:004706DD 3A8510FFFFFF       cmp al, byte ptr [ebp+FFFFFF10]

:004706E3 0F87AA010000       ja 00470893

:004706E9 C745FC25000000     mov [ebp-04], 00000025

:004706F0 8B4DDC             mov ecx, dword ptr [ebp-24]

:004706F3 81E1FF000000       and ecx, 000000FF

:004706F9 898D3CFFFFFF       mov dword ptr [ebp+FFFFFF3C], ecx

:004706FF 83BD3CFFFFFF0B     cmp dword ptr [ebp+FFFFFF3C], 0000000B

:00470706 730C               jnb 00470714

:00470708 C785C0FEFFFF00000000 mov dword ptr [ebp+FFFFFEC0], 00000000

:00470712 EB0C               jmp 00470720
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00470706(C)`

|

* Reference To: `MSVBVM50.__vbaGenerateBoundsError`, Ord: `0000h`

```
:00470714 FF1500A44700       Call dword ptr [0047A400]

:0047071A 8985C0FEFFFF       mov dword ptr [ebp+FFFFFEC0], eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00470712(U)`

|

```
:00470720 8B559C             mov edx, dword ptr [ebp-64]
                     =====>EDX=JJVPCNXAKL

:00470723 03953CFFFFFF       add edx, dword ptr [ebp+FFFFFF3C]

:00470729 33C0               xor eax, eax

:0047072B 8A02               mov al, byte ptr [edx]

:0047072D 50                 push eax

:0047072E 8D8D78FFFFFF       lea ecx, dword ptr [ebp+FFFFFF78]

:00470734 51                 push ecx
```

* Reference To: `MSVBVM50.rtcVarBstrFromAnsi`, Ord: `0260h`

```
:00470735 FF158CA44700       Call dword ptr [0047A48C]

:0047073B 8D9578FFFFFF       lea edx, dword ptr [ebp+FFFFFF78]

:00470741 52                 push edx

:00470742 8D8568FFFFFF       lea eax, dword ptr [ebp+FFFFFF68]

:00470748 50                 push eax
```

* Reference To: `MSVBVM50.rtcUpperCaseVar`, Ord: `0210h`

```
:00470749 FF15FCA34700       Call dword ptr [0047A3FC]

:0047074F 8D8D68FFFFFF       lea ecx, dword ptr [ebp+FFFFFF68]

:00470755 51                 push ecx
```

* Reference To: `MSVBVM50.__vbaStrVarMove`, Ord: `0000h`

```
:00470756 FF1514A34700       Call dword ptr [0047A314]

:0047075C 50                 push eax

:0047075D 8D55A8             lea edx, dword ptr [ebp-58]

:00470760 52                 push edx

:00470761 6A01               push 00000001
```

* Reference To: `MSVBVM50.__vbaLsetFixstrFree`, Ord: `0000h`

```
:00470763 FF15B0A44700       Call dword ptr [0047A4B0]

:00470769 8D8568FFFFFF       lea eax, dword ptr [ebp+FFFFFF68]

:0047076F 50                 push eax

:00470770 8D8D78FFFFFF       lea ecx, dword ptr [ebp+FFFFFF78]

:00470776 51                 push ecx

:00470777 6A02               push 00000002
```

* Reference To: `MSVBVM50.__vbaFreeVarList`, Ord: `0000h`

```
:00470779 FF151CA34700       Call dword ptr [0047A31C]

:0047077F 83C40C             add esp, 0000000C

:00470782 C745FC26000000     mov [ebp-04], 00000026

:00470789 8D55A8             lea edx, dword ptr [ebp-58]

:0047078C 52                 push edx

:0047078D 6A01               push 00000001
```

* Reference To: `MSVBVM50.__vbaStrFixstr`, Ord: `0000h`

```
:0047078F FF15B8A34700       Call dword ptr [0047A3B8]

:00470795 8BD0               mov edx, eax

:00470797 8D4D88             lea ecx, dword ptr [ebp-78]
```

* Reference To: `MSVBVM50.__vbaStrMove`, Ord: `0000h`

```
:0047079A FF1558A54700       Call dword ptr [0047A558]

:004707A0 50                 push eax

:004707A1 68B0D44000         push 0040D4B0
```

* Reference To: `MSVBVM50.__vbaStrCmp`, Ord: `0000h`

```
:004707A6 FF150CA44700       Call dword ptr [0047A40C]
             =====>对上面得出的 JJVPCNXAKL 检测，如有 O（4Fh）则替换成 9

:004707AC F7D8               neg eax

:004707AE 1BC0               sbb eax, eax

:004707B0 40                 inc eax

:004707B1 F7D8               neg eax

:004707B3 6689853CFFFFFF     mov word ptr [ebp+FFFFFF3C], ax

:004707BA 8D4D88             lea ecx, dword ptr [ebp-78]
```

* Reference To: `MSVBVM50.__vbaFreeStr`, Ord: `0000h`

```
:004707BD FF1588A54700       Call dword ptr [0047A588]

:004707C3 0FBF853CFFFFFF     movsx eax, word ptr [ebp+FFFFFF3C]

:004707CA 85C0               test eax, eax

:004707CC 7418               je 004707E6
             =====>不跳就进行替换工作。

:004707CE C745FC27000000     mov [ebp-04], 00000027

:004707D5 68A0D44000         push 0040D4A0

:004707DA 8D4DA8             lea ecx, dword ptr [ebp-58]

:004707DD 51                 push ecx

:004707DE 6A01               push 00000001
```

* Reference To: `MSVBVM50.__vbaLsetFixstr`, Ord: `0000h`

```
:004707E0 FF1560A34700       Call dword ptr [0047A360]
             =====>这里把 O 替换成 9
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:004707CC(C)`

|

```
:004707E6 C745FC29000000     mov [ebp-04], 00000029

:004707ED 8D55A8             lea edx, dword ptr [ebp-58]

:004707F0 52                 push edx

:004707F1 6A01               push 00000001
```

* Reference To: `MSVBVM50.__vbaStrFixstr`, Ord: `0000h`

```
:004707F3 FF15B8A34700       Call dword ptr [0047A3B8]

:004707F9 8BD0               mov edx, eax

:004707FB 8D4D88             lea ecx, dword ptr [ebp-78]
```

* Reference To: `MSVBVM50.__vbaStrMove`, Ord: `0000h`

```
:004707FE FF1558A54700       Call dword ptr [0047A558]

:00470804 50                 push eax

:00470805 68A8D44000         push 0040D4A8
```
