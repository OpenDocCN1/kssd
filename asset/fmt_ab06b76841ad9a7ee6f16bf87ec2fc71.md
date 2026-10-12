
:004063FB 8D45D0 `lea eax, dword ptr [ebp-30]`

:004063FE 50 `push eax`

:004063FF E89BBFFFFF `call 0040239F`

:00406404 8D45D0 `lea eax, dword ptr [ebp-30]`

\* Possible StringData Ref from Data Obj -> "\\"

:00406407 C70424A4E24000 `mov dword ptr [esp], 0040E2A4`

:0040640E 50 `push eax`

:0040640F 8D45C8 `lea eax, dword ptr [ebp-38]`

:00406412 50 `push eax`

:00406413 C645FC07 `mov [ebp-04], 07`

\* Reference To: MFC42.Ordinal:039C, Ord:039Ch

:00406417 E8BA1F0000 `Call 004083D6`

:0040641C 8D4DE8 `lea ecx, dword ptr [ebp-18]`

:0040641F C645FC08 `mov [ebp-04], 08`

:00406423 51 `push ecx`

:00406424 50 `push eax`

:00406425 8D45CC `lea eax, dword ptr [ebp-34]`

:00406428 50 `push eax`

\* Reference To: MFC42.Ordinal:039A, Ord:039Ah

:00406429 E8EA1F0000 `Call 00408418`

:0040642E 50 `push eax`

:0040642F 8D4DE8 `lea ecx, dword ptr [ebp-18]`

:00406432 C645FC09 `mov [ebp-04], 09`

\* Reference To: MFC42.Ordinal:035A, Ord:035Ah

:00406436 E80B1F0000 `Call 00408346`

:0040643B 8D4DCC `lea ecx, dword ptr [ebp-34]`

:0040643E C645FC08 `mov [ebp-04], 08`

\* Reference To: MFC42.Ordinal:0320, Ord:0320h

:00406442 E86F1E0000 `Call 004082B6`

:00406447 8D4DC8 `lea ecx, dword ptr [ebp-38]`

:0040644A C645FC07 `mov [ebp-04], 07`

\* Reference To: MFC42.Ordinal:0320, Ord:0320h

:0040644E E8631E0000 `Call 004082B6`

:00406453 8D45F0 `lea eax, dword ptr [ebp-10]`

:00406456 8D4DD8 `lea ecx, dword ptr [ebp-28]`

:00406459 50 `push eax`

\* Reference To: MFC42.Ordinal:0217, Ord:0217h

:0040645A E8B11E0000 `Call 00408310`

:0040645F 6A00 `push 00000000`

:00406461 C645FC0A `mov [ebp-04], 0A`

\* Reference To: MSVCRT.time, Ord:02D0h

:00406465 FF15CCA34000 `Call dword ptr [0040A3CC]`

:0040646B 50 `push eax`

\* Reference To: MSVCRT.srand, Ord:02B4h

:0040646C FF15D0A34000 `Call dword ptr [0040A3D0]`

:00406472 59 `pop ecx`

:00406473 59 `pop ecx`

\* Reference To: MSVCRT.rand, Ord:02A6h

:00406474 FF15D4A34000 `Call dword ptr [0040A3D4]`

:0040647A 8D4DD4 `lea ecx, dword ptr [ebp-2C]`

:0040647D 8BF8 `mov edi, eax`

\* Reference To: MFC42.Ordinal:021C, Ord:021Ch

:0040647F E8501E0000 `Call 004082D4`

:00406484 57 `push edi`

:00406485 8D45D4 `lea eax, dword ptr [ebp-2C]`

\* Possible StringData Ref from Data Obj -> "%d"

:00406488 68ACE64000 `push 0040E6AC`

:0040648D 50 `push eax`

:0040648E C645FC0B `mov [ebp-04], 0B`

\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

:00406492 E89D1E0000 `Call 00408334`

:00406497 FF75D8 `push [ebp-28]`

:0040649A FFD6 `call esi`

:0040649C 8B4DD4 `mov ecx, dword ptr [ebp-2C]`

:0040649F 33C7 `xor eax, edi`

:004064A1 50 `push eax`

:004064A2 57 `push edi`

:004064A3 8B49F8 `mov ecx, dword ptr [ecx-08]`

:004064A6 8D45D8 `lea eax, dword ptr [ebp-28]`

:004064A9 51 `push ecx`

\* Possible StringData Ref from Data Obj -> "%d%d%d"

:004064AA 68A4E64000 `push 0040E6A4`

:004064AF 50 `push eax`

\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h

:004064B0 E87F1E0000 `Call 00408334`

:004064B5 83C424 `add esp, 00000024`

\* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:02E5h

:004064B8 8B3554A04000 `mov esi, dword ptr [0040A054]`

\* Possible StringData Ref from Data Obj -> "REGINFO"

:004064BE BFC0E24000 `mov edi, 0040E2C0`

:004064C3 FF75E8 `push [ebp-18]`

:004064C6 FF75EC `push [ebp-14]`

\* Possible StringData Ref from Data Obj -> "USERNAME"

:004064C9 68B4E24000 `push 0040E2B4`

:004064CE 57 `push edi`

:004064CF FFD6 `call esi`

:004064D1 FF75E8 `push [ebp-18]`

:004064D4 FF75D8 `push [ebp-28]`

\* Possible StringData Ref from Data Obj -> "PASSWORD"

:004064D7 68A8E24000 `push 0040E2A8`

:004064DC 57 `push edi`

:004064DD FFD6 `call esi`

:004064DF 8B4DE0 `mov ecx, dword ptr [ebp-20]`

:004064E2 6830100000 `push 00001030`

\* Possible StringData Ref from Data Obj -> "注册信息"

:004064E7 6898E64000 `push 0040E698`

\* Possible StringData Ref from Data Obj -> "您成功注册！"

:004064EC 6888E64000 `push 0040E688`

\* Reference To: MFC42.Ordinal:1080, Ord:1080h

:004064F1 E8201E0000 `Call 00408316`

:004064F6 8B4DE0 `mov ecx, dword ptr [ebp-20]`

### 注册码整理

1. 将由用户名 `weifeng` 生成的字符串 `d2VpZmVuZw==` 的每位 ASCII 相加得 `439H`
2. 将假码的后两位 `78` 化成 16 进制 `4E`
3. `439H` XOR `4EH` = `477H` = `1143D` 为前几位数
4. 所以注册码为 `114378`，可以是其它

### 后记

这软件是我刚学破解时在看雪论坛上发贴求助过的，想不到今天找到了它的注册码，其实真的要谢 FLY，他前破过该软件作者的一个软件叫“邮件精灵 V2.0”，可我用他在邮件精灵 V2.0 里找到的注册码注册不成功，所以就有了这篇文章了，呵呵~~原来这软件还在用户名上做手脚。

而算法 `90%` 以上是相同的

`4dd466d0e1e0cc123afbe504da2462ca`
