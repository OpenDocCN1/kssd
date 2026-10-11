
# Gypsee Pro 1.6

> 原文：[`www.pediy.com/kssd/pediy03/forum716.htm`](https://www.pediy.com/kssd/pediy03/forum716.htm)

软件名称：`Gypsee Pro`

整理日期：2001.3.18

最新版本：`1.6`

文件大小：`1351KB`

软件授权：免费软件

使用平台：`Win9x/Me/NT/2000`

发布公司：`Home Page`

软件简介：建立内含音乐的动画 `GIF`，该软件能使用 `MIDI/WAV` 文件给动画 `GIF` 文件插入背景音乐或音效，该软件支持 `GIF` 文件优化，还可将其转化为可执行文件。

下载：`http://www.newhua.com/Gypsee.htm`

或 `http://www.onlinedown.net/Gypsee.htm`

存放注册码的文件：`GYPSEE.KEY`

注册标志：`DS:[004F39AC]`，为 `0` 则已注册。

没仔细看注册码。

```
:004EDCA0 803DAC394F0000               cmp byte ptr [004F39AC], 00

:004EDCA7 0F849A010000                je 004EDE47

:004EDCAD 8D55D8                       lea edx, dword ptr [ebp-28]

:004EDCB0 A16C3D4F00                   mov eax, dword ptr [004F3D6C]

:004EDCB5 8B00                          mov eax, dword ptr [eax]

:004EDCB7 E890BFF4FF                   call 00439C4C

:004EDCBC 8B45D8                       mov eax, dword ptr [ebp-28]

:004EDCBF 8D55DC                       lea edx, dword ptr [ebp-24]

:004EDCC2 E871B0F1FF                   call 00408D38

:004EDCC7 8D45DC                       lea eax, dword ptr [ebp-24]

* Possible StringData Ref from Code Obj ->"\GYPSEE.KEY"

                                      |

:004EDCCA BA4CE04E00                   mov edx, 004EE04C

:004EDCCF E86463F1FF                   call 00404038

:004EDCD4 8B45DC                       mov eax, dword ptr [ebp-24]

:004EDCD7 E86CADF1FF                   call 00408A48

:004EDCDC 84C0                          test al, al

:004EDCDE 0F8425010000                 je 004EDE09

:004EDCE4 B201                          mov dl, 01

* Possible StringData Ref from Code Obj ->"l2A"

                                      |

:004EDCE6 A1ECF84000                   mov eax, dword ptr [0040F8EC]

:004EDCEB E86853F1FF                   call 00403058

:004EDCF0 8BF0                          mov esi, eax

:004EDCF2 8D55D0                       lea edx, dword ptr [ebp-30]

:004EDCF5 A16C3D4F00                   mov eax, dword ptr [004F3D6C]

:004EDCFA 8B00                          mov eax, dword ptr [eax]

:004EDCFC E84BBFF4FF                   call 00439C4C

:004EDD01 8B45D0                       mov eax, dword ptr [ebp-30]

:004EDD04 8D55D4                       lea edx, dword ptr [ebp-2C]

:004EDD07 E82CB0F1FF                   call 00408D38

:004EDD0C 8D45D4                       lea eax, dword ptr [ebp-2C]

* Possible StringData Ref from Code Obj ->"\GYPSEE.KEY"

                                      |

:004EDD0F BA4CE04E00                   mov edx, 004EE04C

:004EDD14 E81F63F1FF                   call 00404038

:004EDD19 8B55D4                       mov edx, dword ptr [ebp-2C]

:004EDD1C 8BC6                          mov eax, esi

:004EDD1E E8CD54F2FF                   call 004131F0

:004EDD23 8BC6                          mov eax, esi

:004EDD25 E88E50F2FF                   call 00412DB8

:004EDD2A 8BD0                          mov edx, eax

:004EDD2C 83EA04                       sub edx, 00000004

:004EDD2F 33C9                          xor ecx, ecx

:004EDD31 8BC6                          mov eax, esi

:004EDD33 8B38                          mov edi, dword ptr [eax]

:004EDD35 FF570C                       call [edi+0C]

:004EDD38 8D55F8                       lea edx, dword ptr [ebp-08]

:004EDD3B B904000000                   mov ecx, 00000004

:004EDD40 8BC6                          mov eax, esi

:004EDD42 8B38                          mov edi, dword ptr [eax]

:004EDD44 FF5704                       call [edi+04]

:004EDD47 FF45F8                       inc [ebp-08]

:004EDD4A 8BC6                          mov eax, esi

:004EDD4C E86750F2FF                   call 00412DB8

:004EDD51 8BD0                          mov edx, eax

:004EDD53 83EA05                       sub edx, 00000005

:004EDD56 33C9                          xor ecx, ecx

:004EDD58 8BC6                          mov eax, esi

:004EDD5A 8B38                          mov edi, dword ptr [eax]

:004EDD5C FF570C                       call [edi+0C]

:004EDD5F 66C745FE0000                 mov [ebp-02], 0000

:004EDD65 66C745FC0000                 mov [ebp-04], 0000

:004EDD6B 8D55FC                       lea edx, dword ptr [ebp-04]

:004EDD6E B901000000                   mov ecx, 00000001

:004EDD73 8BC6                          mov eax, esi

:004EDD75 8B38                          mov edi, dword ptr [eax]

:004EDD77 FF5704                       call [edi+04]

:004EDD7A 33C9                          xor ecx, ecx

:004EDD7C 33D2                          xor edx, edx

:004EDD7E 8BC6                          mov eax, esi

:004EDD80 8B38                          mov edi, dword ptr [eax]

:004EDD82 FF570C                       call [edi+0C]

:004EDD85 33C0                          xor eax, eax

:004EDD87 8945F4                       mov dword ptr [ebp-0C], eax

:004EDD8A 8BC6                          mov eax, esi

:004EDD8C E82750F2FF                   call 00412DB8

:004EDD91 6683E805                     sub ax, 0005

:004EDD95 6685C0                       test ax, ax

:004EDD98 764E                         jbe 004EDDE8

:004EDD9A 668945EE                     mov word ptr [ebp-12], ax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDDE6(C)

|

:004EDD9E 8D55FE                       lea edx, dword ptr [ebp-02]

:004EDDA1 B901000000                   mov ecx, 00000001

:004EDDA6 8BC6                          mov eax, esi

:004EDDA8 8B38                          mov edi, dword ptr [eax]

:004EDDAA FF5704                       call [edi+04]

:004EDDAD 8BC6                          mov eax, esi

:004EDDAF E8E84FF2FF                   call 00412D9C

:004EDDB4 8BD0                          mov edx, eax

:004EDDB6 4A                             dec edx

:004EDDB7 33C9                          xor ecx, ecx

:004EDDB9 8BC6                          mov eax, esi

:004EDDBB 8B38                          mov edi, dword ptr [eax]

:004EDDBD FF570C                       call [edi+0C]

:004EDDC0 0FB745FE                     movzx eax, word ptr [ebp-02]

:004EDDC4 0145F4                       add dword ptr [ebp-0C], eax

:004EDDC7 668B45FC                     mov ax, word ptr [ebp-04]

:004EDDCB 663145FE                     xor word ptr [ebp-02], ax

:004EDDCF 8D55FE                       lea edx, dword ptr [ebp-02]

:004EDDD2 B901000000                   mov ecx, 00000001

:004EDDD7 8BC6                          mov eax, esi

:004EDDD9 8B38                          mov edi, dword ptr [eax]

:004EDDDB FF5708                       call [edi+08]

:004EDDDE 66FF45FC                     inc [ebp-04]

:004EDDE2 66FF4DEE                     dec [ebp-12]

:004EDDE6 75B6                         jne 004EDD9E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDD98(C)

|

:004EDDE8 8B45F4                       mov eax, dword ptr [ebp-0C]

:004EDDEB 3B45F8                       cmp eax, dword ptr [ebp-08]

:004EDDEE 7409                         je 004EDDF9

:004EDDF0 C605AC394F0001               mov byte ptr [004F39AC], 01

:004EDDF7 EB07                         jmp 004EDE00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDDEE(C)

|

:004EDDF9 C605AC394F0000               mov byte ptr [004F39AC], 00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDDF7(U)

|

:004EDE00 8BC6                          mov eax, esi

:004EDE02 E88152F1FF                   call 00403088

:004EDE07 EB07                         jmp 004EDE10

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDCDE(C)

|

:004EDE09 C605AC394F0001               mov byte ptr [004F39AC], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004EDE07(U)

|

:004EDE10 803DAC394F0000               cmp byte ptr [004F39AC], 00

:004EDE17 741F                         je 004EDE38

:004EDE19 8D45CC                       lea eax, dword ptr [ebp-34]

* Possible StringData Ref from Code Obj ->" UNREGISTERED"
```
