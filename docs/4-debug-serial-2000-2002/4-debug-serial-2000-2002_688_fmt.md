
# 天网防火墙个人版 2.0(beta)

> 原文：[`www.pediy.com/kssd/pediy03/forum558.htm`](https://www.pediy.com/kssd/pediy03/forum558.htm)

大家最近是不是冬眠了呀,好像很少有人写教程了,唉!就让小弟辛苦一下吧!希望大家支持!

BTW:谁能够写一篇 `Grduw3.13`(其它版本也行)的破解教程呀?我一直没能将它破掉,`KeyFile`保护的。

目标软件:天网防火墙个人版 2.0(beta)

保护方式:序列号

破解方法:暴力破解

破解人:TAE! (初学者)

说明:此软件可以免费在其网站获得注册码,但这次为了练习一下,还是将其解掉吧,毕竟对我有百利而无一害.

先运行一下,发现启动时让你输入注册名,注册码.

按取消后,正常运行,没有功能限制.

首先,试着用 `TRW` 找出它的注册码,但由于本人功力太弱,没能破解掉.

所以就想想别的方法咯,用 `W32dasm` 反汇编它!选择 `String data references`(字串数据参考),找啊,找啊...猜我找到了什么?

\* Referenced by a CALL at Address:

|:00403CD4

|

:00405F1C 55  push ebp

:00405F1D 8BEC  mov ebp, esp

:00405F1F 83C4B4  add esp, FFFFFFB4

:00405F22 53  push ebx

:00405F23 56  push esi

:00405F24 57  push edi

:00405F25 8BD8  mov ebx, eax

:00405F27 8D75B4  lea esi, dword ptr [ebp\-4C]

:00405F2A B8580A4C00  mov eax, 004C0A58

:00405F2F E80C8B0900  call 0049EA40

:00405F34 66C746100800  mov [esi+10], 0008

:00405F3A 33D2  xor edx, edx

:00405F3C 33C9  xor ecx, ecx

:00405F3E 8955FC  mov dword ptr [ebp\-04], edx

:00405F41 BA2DFD4B00  mov edx, 004BFD2D

:00405F46 FF461C  inc [esi+1C]

:00405F49 8D45EC  lea eax, dword ptr [ebp\-14]

:00405F4C 66C746101400  mov [esi+10], 0014

:00405F52 66C746102000  mov [esi+10], 0020

:00405F58 894DF8  mov dword ptr [ebp\-08], ecx

:00405F5B FF461C  inc [esi+1C]

:00405F5E 66C746101400  mov [esi+10], 0014

:00405F64 66C746102C00  mov [esi+10], 002C

:00405F6A E8F5680B00  call 004BC864

:00405F6F FF461C  inc [esi+1C]

:00405F72 8D55E8  lea edx, dword ptr [ebp\-18]

:00405F75 8B08  mov ecx, dword ptr [eax]

:00405F77 33C0  xor eax, eax

:00405F79 51  push ecx

:00405F7A 8945E8  mov dword ptr [ebp\-18], eax

:00405F7D 52  push edx

\* Possible StringData Ref from Data Obj ->"UserName"\*\*\*\*\*\*\*\*\*

|

:00405F7E BA24FD4B00  mov edx, 004BFD24

:00405F83 FF461C  inc [esi+1C]

:00405F86 8D45F0  lea eax, dword ptr [ebp\-10]

:00405F89 E8D6680B00  call 004BC864

:00405F8E FF461C  inc [esi+1C]

\* Possible StringData Ref from Data Obj ->"Register"\*\*\*\*\*\*\*\*\*

|

:00405F91 BA1BFD4B00  mov edx, 004BFD1B

:00405F96 8B08  mov ecx, dword ptr [eax]

:00405F98 8D45F4  lea eax, dword ptr [ebp\-0C]

:00405F9B 51  push ecx

:00405F9C E8C3680B00  call 004BC864

:00405FA1 FF461C  inc [esi+1C]

:00405FA4 8B10  mov edx, dword ptr [eax]

:00405FA6 8B8300030000  mov eax, dword ptr [ebx+00000300]

:00405FAC 59  pop ecx

:00405FAF 8B38  mov edi, dword ptr [eax]

:00405FB1 FF17  call dword ptr [edi]

:00405FB4 8D55E8  lea edx, dword ptr [ebp\-18]

:00405FB7 8D45FC  lea eax, dword ptr [ebp\-04]

:00405FBA E8F4690B00  call 004BC9B0

:00405FBD FF4E1C  dec [esi+1C]

:00405FC0 8D45E8  lea eax, dword ptr [ebp\-18]

:00405FC3 BA02000000  mov edx, 00000002

:00405FC6 E8B4690B00  call 004BC980

:00405FCA FF4E1C  dec [esi+1C]

:00405FCD 8D45EC  lea eax, dword ptr [ebp\-14]

:00405FD0 BA02000000  mov edx, 00000002

:00405FD3 E8A4690B00  call 004BC980

:00405FD7 FF4E1C  dec [esi+1C]

:00405FDA 8D45F0  lea eax, dword ptr [ebp\-10]

:00405FDD BA02000000  mov edx, 00000002

:00405FE0 E894690B00  call 004BC980

:00405FE4 FF4E1C  dec [esi+1C]

:00405FE7 8D45F4  lea eax, dword ptr [ebp\-0C]

:00405FEA BA02000000  mov edx, 00000002

:00405FED E884690B00  call 004BC980

:00405FF1 66C746103800  mov [esi+10], 0038

:00405FF4 BA43FD4B00  mov edx, 004BFD43

:00405FF7 8D45DC  lea eax, dword ptr [ebp\-24]

:00405FFA E855680B00  call 004BC864

:00405FFE FF461C  inc [esi+1C]

:00406001 8D55D8  lea edx, dword ptr [ebp\-28]

:00406004 8B08  mov ecx, dword ptr [eax]

:00406006 33C0  xor eax, eax

:00406008 51  push ecx

:00406009 8945D8  mov dword ptr [ebp\-28], eax

:0040600C 52  push edx

\* Possible StringData Ref from Data Obj ->"RegisterKey"\*\*\*\*\*\*\*\*\*

|

:0040600D BA37FD4B00  mov edx, 004BFD37

:00406010 FF461C  inc [esi+1C]

:00406013 8D45E0  lea eax, dword ptr [ebp\-20]

:00406016 E836680B00  call 004BC864

:0040601A FF461C  inc [esi+1C]

\* Possible StringData Ref from Data Obj ->"Register"\*\*\*\*\*\*\*\*\*

|

:0040601D BA2EFD4B00  mov edx, 004BFD2E

:00406020 8B08  mov ecx, dword ptr [eax]

:00406022 8D45E4  lea eax, dword ptr [ebp\-1C]

:00406025 51  push ecx

:00406026 E823680B00  call 004BC864

:0040602A FF461C  inc [esi+1C]

:0040602D 8B10  mov edx, dword ptr [eax]

:0040602F 8B8300030000  mov eax, dword ptr [ebx+00000300]

:00406035 59  pop ecx

:00406036 8B38  mov edi, dword ptr [eax]

:00406038 FF17  call dword ptr [edi]

:0040603A 8D55D8  lea edx, dword ptr [ebp\-28]

:0040603D 8D45F8  lea eax, dword ptr [ebp\-08]

:00406040 E854690B00  call 004BC9B0

:00406044 FF4E1C  dec [esi+1C]

:00406047 8D45D8  lea eax, dword ptr [ebp\-28]

:0040604A BA02000000  mov edx, 00000002

:0040604D E814690B00  call 004BC980

:00406051 FF4E1C  dec [esi+1C]

:00406054 8D45DC  lea eax, dword ptr [ebp\-24]

:00406057 BA02000000  mov edx, 00000002

:0040605A E804690B00  call 004BC980

:0040605E FF4E1C  dec [esi+1C]

:00406061 8D45E0  lea eax, dword ptr [ebp\-20]

:00406064 BA02000000  mov edx, 00000002

:00406067 E8F4680B00  call 004BC980

:0040606B FF4E1C  dec [esi+1C]

:0040606E 8D45E4  lea eax, dword ptr [ebp\-1C]

:00406071 BA02000000  mov edx, 00000002

:00406074 E8E4680B00  call 004BC980

:00406078 8B4DF8  mov ecx, dword ptr [ebp\-08]

:0040607B 8B55FC  mov edx, dword ptr [ebp\-04]

:0040607E 8BC3  mov eax, ebx

:00406080 E85FFCFFFF  call 00405D08

:00406085 888305030000  mov byte ptr [ebx+00000305], al

:0040608B BA02000000  mov edx, 00000002

:00406090 8A8305030000  mov al, byte ptr [ebx+00000305]
```
:00405F1C 55  push ebp
:00405F1D 8BEC  mov ebp, esp
:00405F1F 83C4B4  add esp, FFFFFFB4
:00405F22 53  push ebx
:00405F23 56  push esi
:00405F24 57  push edi
:00405F25 8BD8  mov ebx, eax
:00405F27 8D75B4  lea esi, dword ptr [ebp\-4C]
:00405F2A B8580A4C00  mov eax, 004C0A58
:00405F2F E80C8B0900  call 0049EA40
:00405F34 66C746100800  mov [esi+10], 0008
:00405F3A 33D2  xor edx, edx
:00405F3C 33C9  xor ecx, ecx
:00405F3E 8955FC  mov dword ptr [ebp\-04], edx
:00405F41 BA2DFD4B00  mov edx, 004BFD2D
:00405F46 FF461C  inc [esi+1C]
:00405F49 8D45EC  lea eax, dword ptr [ebp\-14]
:00405F4C 66C746101400  mov [esi+10], 0014
:00405F52 66C746102000  mov [esi+10], 0020
:00405F58 894DF8  mov dword ptr [ebp\-08], ecx
:00405F5B FF461C  inc [esi+1C]
:00405F5E 66C746101400  mov [esi+10], 0014
:00405F64 66C746102C00  mov [esi+10], 002C
:00405F6A E8F5680B00  call 004BC864
:00405F6F FF461C  inc [esi+1C]
:00405F72 8D55E8  lea edx, dword ptr [ebp\-18]
:00405F75 8B08  mov ecx, dword ptr [eax]
:00405F77 33C0  xor eax, eax
:00405F79 51  push ecx
:00405F7A 8945E8  mov dword ptr [ebp\-18], eax
:00405F7D 52  push edx
* Possible StringData Ref from Data Obj ->"UserName"

:00405F7E BA24FD4B00  mov edx, 004BFD24
:00405F83 FF461C  inc [esi+1C]
:00405F86 8D45F0  lea eax, dword ptr [ebp\-10]
:00405F89 E8D6680B00  call 004BC864
:00405F8E FF461C  inc [esi+1C]
* Possible StringData Ref from Data Obj ->"Register"

:00405F91 BA1BFD4B00  mov edx, 004BFD1B
:00405F96 8B08  mov ecx, dword ptr [eax]
:00405F98 8D45F4  lea eax, dword ptr [ebp\-0C]
:00405F9B 51  push ecx
:00405F9C E8C3680B00  call 004BC864
:00405FA1 FF461C  inc [esi+1C]
:00405FA4 8B10  mov edx, dword ptr [eax]
:00405FA6 8B8300030000  mov eax, dword ptr [ebx+00000300]
:00405FAC 59  pop ecx
:00405FAF 8B38  mov edi, dword ptr [eax]
:00405FB1 FF17  call dword ptr [edi]
:00405FB4 8D55E8  lea edx, dword ptr [ebp\-18]
:00405FB7 8D45FC  lea eax, dword ptr [ebp\-04]
:00405FBA E8F4690B00  call 004BC9B0
:00405FBD FF4E1C  dec [esi+1C]
:00405FC0 8D45E8  lea eax, dword ptr [ebp\-18]
:00405FC3 BA02000000  mov edx, 00000002
:00405FC6 E8B4690B00  call 004BC980
:00405FCA FF4E1C  dec [esi+1C]
:00405FCD 8D45EC  lea eax, dword ptr [ebp\-14]
:00405FD0 BA02000000  mov edx, 00000002
:00405FD3 E8A4690B00  call 004BC980
:00405FD7 FF4E1C  dec [esi+1C]
:00405FDA 8D45F0  lea eax, dword ptr [ebp\-10]
:00405FDD BA02000000  mov edx, 00000002
:00405FE0 E894690B00  call 004BC980
:00405FE4 FF4E1C  dec [esi+1C]
:00405FE7 8D45F4  lea eax, dword ptr [ebp\-0C]
:00405FEA BA02000000  mov edx, 00000002
:00405FED E884690B00  call 004BC980
:00405FF1 66C746103800  mov [esi+10], 0038
:00405FF4 BA43FD4B00  mov edx, 004BFD43
:00405FF7 8D45DC  lea eax, dword ptr [ebp\-24]
:00405FFA E855680B00  call 004BC864
:00405FFE FF461C  inc [esi+1C]
:00406001 8D55D8  lea edx, dword ptr [ebp\-28]
:00406004 8B08  mov ecx, dword ptr [eax]
:00406006 33C0  xor eax, eax
:00406008 51  push ecx
:00406009 8945D8  mov dword ptr [ebp\-28], eax
:0040600C 52  push edx
* Possible StringData Ref from Data Obj ->"RegisterKey"

:0040600D BA37FD4B00  mov edx, 004BFD37
:00406010 FF461C  inc [esi+1C]
:00406013 8D45E0  lea eax, dword ptr [ebp\-20]
:00406016 E836680B00  call 004BC864
:0040601A FF461C  inc [esi+1C]
* Possible StringData Ref from Data Obj ->"Register"

:0040601D BA2EFD4B00  mov edx, 004BFD2E
:00406020 8B08  mov ecx, dword ptr [eax]
:00406022 8D45E4  lea eax, dword ptr [ebp\-1C]
:00406025 51  push ecx
:00406026 E823680B00  call 004BC864
:0040602A FF461C  inc [esi+1C]
:0040602D 8B10  mov edx, dword ptr [eax]
:0040602F 8B8300030000  mov eax, dword ptr [ebx+00000300]
:00406035 59  pop ecx
:00406036 8B38  mov edi, dword ptr [eax]
:00406038 FF17  call dword ptr [edi]
:0040603A 8D55D8  lea edx, dword ptr [ebp\-28]
:0040603D 8D45F8  lea eax, dword ptr [ebp\-08]
:00406040 E854690B00  call 004BC9B0
:00406044 FF4E1C  dec [esi+1C]
:00406047 8D45D8  lea eax, dword ptr [ebp\-28]
:0040604A BA02000000  mov edx, 00000002
:0040604D E814690B00  call 004BC980
:00406051 FF4E1C  dec [esi+1C]
:00406054 8D45DC  lea eax, dword ptr [ebp\-24]
:00406057 BA02000000  mov edx, 00000002
:0040605A E804690B00  call 004BC980
:0040605E FF4E1C  dec [esi+1C]
:00406061 8D45E0  lea eax, dword ptr [ebp\-20]
:00406064 BA02000000  mov edx, 00000002
:00406067 E8F4680B00  call 004BC980
:0040606B FF4E1C  dec [esi+1C]
:0040606E 8D45E4  lea eax, dword ptr [ebp\-1C]
:00406071 BA02000000  mov edx, 00000002
:00406074 E8E4680B00  call 004BC980
:00406078 8B4DF8  mov ecx, dword ptr [ebp\-08]
:0040607B 8B55FC  mov edx, dword ptr [ebp\-04]
:0040607E 8BC3  mov eax, ebx
:00406080 E85FFCFFFF  call 00405D08
:00406085 888305030000  mov byte ptr [ebx+00000305], al
:0040608B BA02000000  mov edx, 00000002
:00406090 8A8305030000  mov al, byte ptr [ebx+00000305]
```



```
:004060BA 50                          push eax
:004060BB 8D45F8                      lea eax, dword ptr [ebp-08]
:004060BE FF4E1C                      dec [esi+1C]
:004060C1 E8BA680B00                  call 004BC980
:004060C6 FF4E1C                      dec [esi+1C]
:004060C9 8D45FC                      lea eax, dword ptr [ebp-04]
:004060CC BA02000000                  mov edx, 00000002
:004060D1 E8AA680B00                  call 004BC980
:004060D6 58                          pop eax
:004060D7 8B16                        mov edx, dword ptr [esi]
:004060D9 64891500000000              mov dword ptr fs:[00000000], edx
:004060E0 5F                          pop edi
:004060E1 5E                          pop esi
:004060E2 5B                          pop ebx
:004060E3 8BE5                        mov esp, ebp
:004060E5 5D                          pop ebp
:004060E6 C3                          ret
```

喔~，看到胜利之神在向我招手了！

这分明就是文件中存放注册信息的标志字符串（可以这么叫吗？）

什么，听不懂？举个例子吧！

有的软件将注册信息放在一个文件里，通常是 `<软件名>.ini` 或 `<软件名>.dat` 中，如：`WinZip Self-Extract 2.2`。

你注册后，那么在天网防火墙的 `.ini` 文件，也就是配置文件中就应该有以下几项：

```
[register]

username=你的注册名

registerkey=您的注册码
```

想想看，所以软件每次启动的时候都会读取 `.ini` 中有没有这几项，若有就检查注册名和你的注册码是不是匹配；

若没有发现这几项，就直接判断您还没有注册，就跳出提示框啦！

所以我们可以从这里入手，向上看发现它是 `00403CD4` `Call` 过来的。

于是我来到了这里：

果然是将注册信息放在了 `SNFW.INI` 文件中！

`* Possible StringData Ref from Data Obj ->"SNFW.INI"`

```
:00403C50 BA2BFB4B00                  mov edx, 004BFB2B
:00403C55 8D45F0                      lea eax, dword ptr [ebp-10]
:00403C58 E8078C0B00                  call 004BC864
:00403C5D FF45D4                      inc [ebp-2C]
:00403C60 33C0                        xor eax, eax
:00403C62 8945EC                      mov dword ptr [ebp-14], eax
:00403C65 8D55F0                      lea edx, dword ptr [ebp-10]
:00403C68 FF45D4                      inc [ebp-2C]
:00403C6B 8D4DEC                      lea ecx, dword ptr [ebp-14]
:00403C6E 58                          pop eax
:00403C6F E8648D0B00                  call 004BC9D8
:00403C74 8D4DEC                      lea ecx, dword ptr [ebp-14]
:00403C77 8B09                        mov ecx, dword ptr [ecx]
:00403C79 B201                        mov dl, 01
```

`* Possible StringData Ref from Code Obj ->"胤 C"`

```
:00403C7B A110B14300                  mov eax, dword ptr [0043B110]
:00403C80 E83B010000                  call 00403DC0
:00403C85 898300030000                mov dword ptr [ebx+00000300], eax
:00403C8B FF4DD4                      dec [ebp-2C]
:00403C8E 8D45EC                      lea eax, dword ptr [ebp-14]
:00403C91 BA02000000                  mov edx, 00000002
:00403C96 E8E58C0B00                  call 004BC980
:00403C9B FF4DD4                      dec [ebp-2C]
:00403C9E 8D45F0                      lea eax, dword ptr [ebp-10]
:00403CA1 BA02000000                  mov edx, 00000002
:00403CA6 E8D58C0B00                  call 004BC980
:00403CAB FF4DD4                      dec [ebp-2C]
:00403CAE 8D45F4                      lea eax, dword ptr [ebp-0C]
:00403CB1 BA02000000                  mov edx, 00000002
:00403CB6 E8C58C0B00                  call 004BC980
:00403CBB FF4DD4                      dec [ebp-2C]
:00403CBE 8D45F8                      lea eax, dword ptr [ebp-08]
:00403CC1 BA02000000                  mov edx, 00000002
:00403CC6 E8B58C0B00                  call 004BC980
:00403CCB C6830503000000              mov byte ptr [ebx+00000305], 00
:00403CD2 8BC3                        mov eax, ebx
:00403CD4 E843220000                  call 00405F1C <-- 来到了这儿
:00403CD9 84C0                        test al, al   - 咦！很眼熟喔.
:00403CDB 7541                        jne 00403D1E   /
:00403CDD 33C9                        xor ecx, ecx
:00403CDF B201                        mov dl, 01
```

`* Possible StringData Ref from Data Obj ->"@F"`

```
:00403CE1 A1DC304C00                  mov eax, dword ptr [004C30DC]
:00403CE6 E8D1700000                  call 0040ADBC
:00403CEB 8BF8                        mov edi, eax
:00403CED 8BC7                        mov eax, edi
:00403CEF 8B10                        mov edx, dword ptr [eax]
:00403CF1 FF92D8000000                call dword ptr [edx+000000D8]
:00403CF7 8BF7                        mov esi, edi
:00403CF9 8975E4                      mov dword ptr [ebp-1C], esi
:00403CFC 85F6                        test esi, esi
:00403CFE 741E                        je 00403D1E
:00403D00 8B06                        mov eax, dword ptr [esi]
:00403D02 8945E8                      mov dword ptr [ebp-18], eax
:00403D05 66C745C82C00                mov [ebp-38], 002C
:00403D0B BA03000000                  mov edx, 00000003
:00403D10 8B45E4                      mov eax, dword ptr [ebp-1C]
:00403D13 8B08                        mov ecx, dword ptr [eax]
:00403D15 FF51FC                      call [ecx-04]
:00403D18 66C745C82000                mov [ebp-38], 0020
```

试着将 `:00403CDB jne 00403D1E` 改为 `:00403CDB je 00403D1E`

也就是将 `7541` 改为 `7441`

运行一下，嗯！很好，那个讨厌的注册提示框再也不会出现了。

这应该是我的第一篇破解教程，唉！我终于体会到各位大哥的辛苦了，写这东西的确耗时间。我可是用拼音输入法打的喔！

在此，感谢：`看雪`, `Icebird`, `Icebird`, `冰毒`, `DDxia`, `ErrorFree`, `tKC`, `EGis` 带我进入了破解世界。

`4dd466d0e1e0cc123afbe504da2462ca`
