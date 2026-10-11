
:00486535 `C3`                          `ret`

---

### 进入关键 CALL：486402 `call 00485F0C`

呵呵，很是烦琐，作者自己是不嫌麻烦的。想写注册机的 `CRACKER` 可不轻松了。

* `Referenced by a CALL at Address:`
| :00486402

---

:00485F0C `55`                          `push ebp`
:00485F0D `8BEC`                        `mov ebp, esp`
:00485F0F `83C4EC`                      `add esp, FFFFFFEC`
:00485F12 `53`                          `push ebx`
:00485F13 `56`                          `push esi`
:00485F14 `57`                          `push edi`
:00485F15 `33D2`                        `xor edx, edx`
:00485F17 `8955F8`                      `mov dword ptr [ebp-08], edx`
:00485F1A `8955F4`                      `mov dword ptr [ebp-0C], edx`
:00485F1D `8945FC`                      `mov dword ptr [ebp-04], eax`
:00485F20 `8B45FC`                      `mov eax, dword ptr [ebp-04]`
:00485F23 `E8ECE9F7FF`                  `call 00404914`
:00485F28 `33C0`                        `xor eax, eax`
:00485F2A `55`                          `push ebp`
:00485F2B `6849614800`                  `push 00486149`
:00485F30 `64FF30`                      `push dword ptr fs:[eax]`
:00485F33 `648920`                      `mov dword ptr fs:[eax], esp`
:00485F36 `8B45FC`                      `mov eax, dword ptr [ebp-04]`
                                  `====>EAX=fly0`
:00485F39 `E8EEE7F7FF`                  `call 0040472C`
                                  `====>取 用户名 位数入 AL`
:00485F3E `8845F2`                      `mov byte ptr [ebp-0E], al`
                                  `====>[ebp-0E]=AL=04`
:00485F41 `E87AF0FFFF`                  `call 00484FC0`
                                  `====>取硬盘序列号 入 EAX=211C1E09`
:00485F46 `8D4DF4`                      `lea ecx, dword ptr [ebp-0C]`
:00485F49 `BA08000000`                  `mov edx, 00000008`
:00485F4E `E8112BF8FF`                  `call 00408A64`
:00485F53 `33C0`                        `xor eax, eax`
:00485F55 `8A45F2`                      `mov al, byte ptr [ebp-0E]`
:00485F58 `83C009`                      `add eax, 00000009`
                                  `====>EAX=04 + 09=0D`
:00485F5B `50`                          `push eax`
:00485F5C `8D45F8`                      `lea eax, dword ptr [ebp-08]`
:00485F5F `B901000000`                  `mov ecx, 00000001`

* `Possible StringData Ref from Code Obj ->".3"`
|

:00485F64 `8B15EC5E4800`                `mov edx, dword ptr [00485EEC]`
:00485F6A `E8A9FAF7FF`                  `call 00405A18`
:00485F6F `83C404`                      `add esp, 00000004`
:00485F72 `8A45F2`                      `mov al, byte ptr [ebp-0E]`
:00485F75 `48`                          `dec eax`
:00485F76 `84C0`                        `test al, al`
:00485F78 `721C`                        `jb 00485F96`
:00485F7A `40`                          `inc eax`
:00485F7B `B300`                        `mov bl, 00`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00485F94(C)

---

:00485F7D `33D2`                        `xor edx, edx`
:00485F7F `8AD3`                        `mov dl, bl`
:00485F81 `8B4DFC`                      `mov ecx, dword ptr [ebp-04]`
:00485F84 `8A1411`                      `mov dl, byte ptr [ecx+edx]`
:00485F87 `33C9`                        `xor ecx, ecx`
:00485F89 `8ACB`                        `mov cl, bl`
:00485F8B `8B75F8`                      `mov esi, dword ptr [ebp-08]`
:00485F8E `88140E`                      `mov byte ptr [esi+ecx], dl`
                                  `====>用户名 入 [esi+ecx]处`

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[esi+ecx]`内存中的值：

```
00C92358  66 6C 79 30 00 00 00 00 00 00 00 00 00 00 00 00  fly0............
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00485F91 `43`                          `inc ebx`
:00485F92 `FEC8`                        `dec al`
:00485F94 `75E7`                        `jne 00485F7D`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00485F78(C)

---

:00485F96 `8A5DF2`                      `mov bl, byte ptr [ebp-0E]`
:00485F99 `8A45F2`                      `mov al, byte ptr [ebp-0E]`
:00485F9C `0407`                        `add al, 07`
                                  `====>AL=04 + 07=0B`
:00485F9E `2AC3`                        `sub al, bl`
                                  `====>AL=0B - 04=07`
:00485FA0 `7233`                        `jb 00485FD5`
:00485FA2 `40`                          `inc eax`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00485FD3(C)

---

:00485FA3 `8BF3`                        `mov esi, ebx`
:00485FA5 `81E6FF000000`                `and esi, 000000FF`
:00485FAB `8BD6`                        `mov edx, esi`
:00485FAD `33C9`                        `xor ecx, ecx`
:00485FAF `8A4DF2`                      `mov cl, byte ptr [ebp-0E]`
:00485FB2 `2BD1`                        `sub edx, ecx`
:00485FB4 `8B4DF4`                      `mov ecx, dword ptr [ebp-0C]`
                                  `====>ECX=211C1E09   呵呵，我的硬盘序列号`
:00485FB7 `8A0C11`                      `mov cl, byte ptr [ecx+edx]`
                                  `====>依次取硬盘序列号的字符值`
:00485FBA `51`                          `push ecx`
:00485FBB `8B4DF8`                      `mov ecx, dword ptr [ebp-08]`
:00485FBE `8D3431`                      `lea esi, dword ptr [ecx+esi]`
:00485FC1 `59`                          `pop ecx`
:00485FC2 `880E`                        `mov byte ptr [esi], cl`
:00485FC4 `8B4DF4`                      `mov ecx, dword ptr [ebp-0C]`
:00485FC7 `8A0C11`                      `mov cl, byte ptr [ecx+edx]`
:00485FCA `888A64BD4800`                `mov byte ptr [edx+0048BD64], cl`
                                  `====>硬盘序列号 入 [edx+0048BD64]处`

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`[edx+0048BD64]`内存中的值：

```
0048BD64  32 31 31 43 31 45 30 39 00 00 00 00 00 00 00 00  211C1E09........
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:00485FD0 `43`                          `inc ebx`
:00485FD1 `FEC8`                        `dec al`
:00485FD3 `75CE`                        `jne 00485FA3`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00485FA0(C)

---

:00485FD5 `33C0`                        `xor eax, eax`
:00485FD7 `8A45F2`                      `mov al, byte ptr [ebp-0E]`
                                  `====>AL=04`
:00485FDA `8B55F8`                      `mov edx, dword ptr [ebp-08]`
                                  `====>EDX=fly0211C1E09      用户名和序列号连接起来`
:00485FDD `C644020900`                  `mov [edx+eax+09], 00`
:00485FE2 `C645F100`                    `mov [ebp-0F], 00`
:00485FE6 `8A45F2`                      `mov al, byte ptr [ebp-0E]`
:00485FE9 `0408`                        `add al, 08`
                                  `====>AL=04 + 08=0C`
:00485FEB `8845EF`                      `mov byte ptr [ebp-11], al`
                                  `====>AL=0C 入 [ebp-11]`
:00485FEE `8A45F2`                      `mov al, byte ptr [ebp-0E]`
:00485FF1 `0407`                        `add al, 07`
                                  `====>AL=04 + 07=0B`
:00485FF3 `84C0`                        `test al, al`
:00485FF5 `7215`                        `jb 0048600C`
:00485FF7 `40`                          `inc eax`
:00485FF8 `B300`                        `mov bl, 00`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :0048600A(C)

---

:00485FFA `33D2`                        `xor edx, edx`
:00485FFC `8AD3`                        `mov dl, bl`
:00485FFE `8B4DF8`                      `mov ecx, dword ptr [ebp-08]`
                                  `====>ECX=fly0211C1E09`
:00486001 `8A1411`                      `mov dl, byte ptr [ecx+edx]`
                                  `====>依次取上面字符的 HEX 值，下面累加`
:00486004 `0055F1`                      `add byte ptr [ebp-0F], dl`
                                  `====>[ebp-0F]=0+66+6C+79+32+31+31+43+31+45+30+39=31  舍去溢出`
:00486007 `43`                          `inc ebx`
:00486008 `FEC8`                        `dec al`
:0048600A `75EE`                        `jne 00485FFA`
                                  `====>循环 12 次`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00485FF5(C)

---

:0048600C `33DB`                        `xor ebx, ebx`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`
| :00486117(C)

---

:0048600E `C645F300`                    `mov [ebp-0D], 00`
