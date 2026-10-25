
`:0041E8B3` `66833901`

`cmp word ptr [ecx], 0001`

`:0041E8B7`

`7521`                    `jne`

`0041E8DA`

`:0041E8B9` `8B5114`

   `mov edx, dword ptr [ecx+14]`

`:0041E8BC` `8B4110`

           `mov eax, dword ptr [ecx+10]`

\*

Reference To: `MSVBVM60.\_\_vbaGenerateBoundsError`, Ord:0000h

   |

`:0041E8BF` `8B3DD4104000`            `mov`

edi, dword ptr [`004010D4`]

`:0041E8C5` `8BF3`

         `mov esi, ebx`

`:0041E8C7` `2BF2`

               `sub esi, edx`

`:0041E8C9`

`3BF0`                    `cmp`

esi, eax

`:0041E8CB` `7205`

   `jb` `0041E8D2`

`:0041E8CD` `FFD7`

         `call edi`

`:0041E8CF` `8B4DD4`

           `mov ecx, dword ptr [ebp-2C]`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E8CB(C)

|

`:0041E8D2`

`89B544FFFFFF`            `mov dword ptr [ebp+FFFFFF44],`

esi

`:0041E8D8` `EB15`

   `jmp` `0041E8EF`

\* Referenced by a (U)nconditional

or (C)onditional Jump at Addresses:

|:0041E8B1(C), :0041E8B7(C)

|

\*

Reference To: `MSVBVM60.\_\_vbaGenerateBoundsError`, Ord:0000h

   |

`:0041E8DA` `FF15D4104000`            `Call`

dword ptr [`004010D4`]

`:0041E8E0` `8B4DD4`

     `mov ecx, dword ptr [ebp-2C]`

\*

Reference To: `MSVBVM60.\_\_vbaGenerateBoundsError`, Ord:0000h

   |

`:0041E8E3` `8B3DD4104000`            `mov`

edi, dword ptr [`004010D4`]

`:0041E8E9` `898544FFFFFF`

   `mov dword ptr [ebp+FFFFFF44], eax`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E8D8(U)

|

`:0041E8EF`

`85C9`                    `test`

ecx, ecx

`:0041E8F1` `7421`

   `je` `0041E914`

`:0041E8F3` `66833901`

     `cmp word ptr [ecx], 0001`

`:0041E8F7` `751B`

             `jne` `0041E914`

`:0041E8F9` `8B75D8`

                 `mov esi, dword ptr`

[ebp-28]

`:0041E8FC` `8B5114`

   `mov edx, dword ptr [ecx+14]`

`:0041E8FF` `8B4110`

           `mov eax, dword ptr [ecx+10]`

`:0041E902`

`2BF3`                    `sub`

esi, ebx

`:0041E904` `0F80F0010000`            `jo`

`0041EAFA`

`:0041E90A` `2BF2`

   `sub esi, edx`

`:0041E90C` `3BF0`

         `cmp esi, eax`

`:0041E90E` `720B`

               `jb` `0041E91B`

`:0041E910`

`FFD7`                    `call`

edi

`:0041E912` `EB04`

   `jmp` `0041E918`

\* Referenced by a (U)nconditional

or (C)onditional Jump at Addresses:

|:0041E8F1(C), :0041E8F7(C)

|

`:0041E914`

`FFD7`                    `call`

edi

`:0041E916` `8BF0`

   `mov esi, eax`

\* Referenced by a (U)nconditional

or (C)onditional Jump at Address:

|:0041E912(U)

|

`:0041E918` `8B4DD4`

               `mov ecx, dword ptr [ebp-2C]`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E90E(C)

|

`:0041E91B`

`85C9`                    `test`

ecx, ecx

`:0041E91D` `7421`

   `je` `0041E940`

`:0041E91F` `66833901`

     `cmp word ptr [ecx], 0001`

`:0041E923` `751B`

             `jne` `0041E940`

`:0041E925` `8B5114`

                 `mov edx, dword ptr`

[ecx+14]

`:0041E928` `8B4110`

   `mov eax, dword ptr [ecx+10]`

`:0041E92B` `8BFB`

             `mov edi, ebx`

`:0041E92D` `2BFA`

                   `sub edi,`

edx

`:0041E92F` `3BF8`

   `cmp edi, eax`

`:0041E931` `7209`

         `jb` `0041E93C`

\* Reference

To: `MSVBVM60.\_\_vbaGenerateBoundsError`, Ord:0000h

 |

`:0041E933` `FF15D4104000`            `Call`

dword ptr [`004010D4`]

`:0041E939` `8B4DD4`

     `mov ecx, dword ptr [ebp-2C]`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E931(C)

|

`:0041E93C`

`8BC7`                    `mov`

eax, edi

`:0041E93E` `EB05`

   `jmp` `0041E945`

\* Referenced by a (U)nconditional

or (C)onditional Jump at Addresses:

|:0041E91D(C), :0041E923(C)

|

`:0041E940`

`FFD7`                    `call`

edi

`:0041E942` `8B4DD4`

 mov ecx, dword ptr [ebp-2C]

\* Referenced by

a (U)nconditional or (C)onditional Jump at Address:

|:0041E93E(U)

|

`:0041E945`

`8B490C`                  `mov ecx,`

dword ptr [ecx+0C]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

[ecx+0C]内存中的值是上面21次循环异或运算得出的值：

`0044F168`

 57 5E 4A 0E 0C 0E 0F 1A 74 05 72 0C 06 1D 00 22  W^J.tr.."

`0044F178`

 27 3A 52 03 0A

                ':R.

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`:0041E948`

`8A1431`                  `mov dl, byte`

ptr [ecx+esi]

====>倒序取57 5E 4A 0E 0C 0E 0F 1A 74 05 72 0C 06 1D 00 22 27

3A 52 03 0A

`:0041E94B` `8BB544FFFFFF`

       `mov esi, dword ptr [ebp+FFFFFF44]`

     ====>ESI=循环次数。从0开始

`:0041E951`

`321431`                  `xor dl, byte`

ptr [ecx+esi]

====>倒序取的值依次与57 5E 4A 0E 0C 0E 0F 1A 74 05 72 0C 06 1D 00

22 27 3A 52 03 0A异或

`:0041E954` `881401`

             `mov byte ptr [ecx+eax], dl`

         ====>结果入[ecx+eax]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

[ecx+eax]内存中最后得到的值：

`0044F168`

 5D 5D 18 34 2B 2C 0F 07 72 09 00 05 74 1A 0F 0E  ]]4+,r..t

`0044F178`

 0C 0E 4A 5E 57

                .J^W

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`:0041E957`

`B801000000`              `mov eax, 00000001`

`:0041E95C`

`03C3`                    `add`

eax, ebx

`:0041E95E` `0F8096010000`            `jo`

`0041EAFA`

`:0041E964` `8BD8`

   `mov ebx, eax`

`:0041E966` `33FF`

         `xor edi, edi`

`:0041E968` `E936FFFFFF`

           `jmp` `0041E8A3`

   ====>再次循环运算21次！

\* Referenced by a

(U)nconditional or (C)onditional Jump at Address:

|:0041E8A6(C)

|

\*

Reference To: `MSVBVM60.\_\_vbaStrVarMove`, Ord:0000h

 |

`:0041E96D` `8B1D2C104000`            `mov`

ebx, dword ptr [`0040102C`]

`:0041E973` `33F6`

         `xor esi, esi`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041EA3E(U)

|

`:0041E975`

`3B75D8`                  `cmp esi,`

dword ptr [ebp-28]

`:0041E978` `0F8FC5000000`

 jg `0041EA43`

`:0041E97E` `8B4DD4`

     `mov ecx, dword ptr [ebp-2C]`

`:0041E981` `8B45D0`

             `mov eax, dword ptr [ebp-30]`

`:0041E984`

`8D55CC`                  `lea edx,`

dword ptr [ebp-34]

`:0041E987` `51`

       `push ecx`

`:0041E988` `52`

             `push edx`

`:0041E989` `89857CFFFFFF`

           `mov dword ptr [ebp+FFFFFF7C], eax`

`:0041E98F`

`C78574FFFFFF08000000`    `mov dword ptr [ebp+FFFFFF74], 00000008`

\*

Reference To: `MSVBVM60.\_\_vbaAryLock`, Ord:0000h

 |

`:0041E999` `FF15C4114000`            `Call`

dword ptr [`004011C4`]

`:0041E99F` `8B4DCC`

     `mov ecx, dword ptr [ebp-34]`

`:0041E9A2` `3BCF`

               `cmp ecx, edi`

`:0041E9A4`

`7423`                    `je` `0041E9C9`

`:0041E9A6`

`66833901`                `cmp word ptr [ecx],`

0001

`:0041E9AA` `751D`

   `jne` `0041E9C9`

`:0041E9AC` `8B5114`

       `mov edx, dword ptr [ecx+14]`

`:0041E9AF` `8B4110`

               `mov eax, dword ptr [ecx+10]`

`:0041E9B2`

`8BFE`                    `mov`

edi, esi

`:0041E9B4` `2BFA`

   `sub edi, edx`

`:0041E9B6` `3BF8`

         `cmp edi, eax`

`:0041E9B8` `7209`

               `jb` `0041E9C3`

\*

Reference To: `MSVBVM60.\_\_vbaGenerateBoundsError`, Ord:0000h

   |

`:0041E9BA` `FF15D4104000`            `Call`

dword ptr [`004010D4`]

`:0041E9C0` `8B4DCC`

     `mov ecx, dword ptr [ebp-34]`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E9B8(C)

|

`:0041E9C3`

`8BC7`                    `mov`

eax, edi

`:0041E9C5` `33FF`

   `xor edi, edi`

`:0041E9C7` `EB09`

         `jmp` `0041E9D2`

\*

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0041E9A4(C),

:0041E9AA(C)

|

\* Reference To: `MSVBVM60.\_\_vbaGenerateBoundsError`,

Ord:0000h

             |

`:0041E9C9` `FF15D4104000`

         `Call dword ptr [004010D4]`

`:0041E9CF` `8B4DCC`

                 `mov ecx, dword ptr`

[ebp-34]

             ====>ECX=[ebp-34]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

[ebp-34]内存中值：

`0044F168`

 5D 5D 18 34 2B 2C 0F 07 72 09 00 05 74 1A 0F 0E  ]]4+,r..t

`0044F178`

 0C 0E 4A 5E 57

                .J^W

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041E9C7(U)

|

`:0041E9D2`

`8B490C`                  `mov ecx,`

dword ptr [ecx+0C]

`:0041E9D5` `8D5584`

     `lea edx, dword ptr [ebp-7C]`

`:0041E9D8` `03C8`

               `add ecx, eax`

`:0041E9DA`

`8D45B8`                  `lea eax,`

dword ptr [ebp-48]

`:0041E9DD` `52`

       `push edx`

`:0041E9DE` `50`

             `push eax`

`:0041E9DF` `894D8C`

               `mov dword ptr [ebp-74],`

ecx

`:0041E9E2` `C7458411400000`          `mov [ebp-7C],`

00004011

\* Reference To: `MSVBVM60.rtcHexVarFromVar`,

Ord:023Dh

             |

`:0041E9E9` `FF1594114000`

         `Call dword ptr [00401194]`

     ====>rtcHexVarFromVar

`:0041E9EF`

`8D4DCC`                  `lea ecx,`
