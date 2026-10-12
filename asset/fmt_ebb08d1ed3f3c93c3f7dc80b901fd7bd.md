
* Reference To: `MSVBVM50.__vbaGenerateBoundsError`, Ord: `0000h`

              |

:164706B8 `FF15B8624716`         `Call dword ptr [164762B8]`

:164706BE `8B4DB4`               `mov ecx, dword ptr [ebp-4C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:164706B6(C)
|
:164706C1 `8D049D00000000`       `lea eax, dword ptr [4*ebx+00000000]`

:164706C8 `EB09`                 `jmp 164706D3`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:164706A0(C), :164706A6(C)
|
* Reference To: `MSVBVM50.__vbaGenerateBoundsError`, Ord: `0000h`

              |
:164706CA `FF15B8624716`         `Call dword ptr [164762B8]`

:164706D0 `8B4DB4`               `mov ecx, dword ptr [ebp-4C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:164706C8(U)
|
:164706D3 `8B5D9C`               `mov ebx, dword ptr [ebp-64]`

:164706D6 `C7459C00000000`       `mov [ebp-64], 00000000`

:164706DD `8B490C`               `mov ecx, dword ptr [ecx+0C]`

* Possible StringData Ref from Code Obj -> `"5A827999"`

              |
:164706E0 `68C8514616`           `push 164651C8`

:164706E5 `8B1401`               `mov edx, dword ptr [ecx+eax]`

              ==== > `EDX=31383730`

:164706E8 `52`                   `push edx`

:164706E9 `E892E6FFFF`           `call 1646ED80`

              ==== > `31383730 + 5A827999=8BBAB0C9`

:164706EE `8BD0`                 `mov edx, eax`

              ==== > `EDX=8BBAB0C9`

:164706F0 `8D4DAC`               `lea ecx, dword ptr [ebp-54]`

:164706F3 `FFD6`                 `call esi`

:164706F5 `50`                   `push eax`

:164706F6 `8B45B8`               `mov eax, dword ptr [ebp-48]`

              ==== > `EDX=c3d2e1f0`

:164706F9 `50`                   `push eax`

:164706FA `E881E6FFFF`           `call 1646ED80`

              ==== > `8BBAB0C9 + c3d2e1f0=4F8D92B9`

:164706FF `8BD0`                 `mov edx, eax`

              ==== > `EDX=4F8D92B9`

:16470701 `8D4DA8`               `lea ecx, dword ptr [ebp-58]`

:16470704 `FFD6`                 `call esi`

:16470706 `8B4DC4`               `mov ecx, dword ptr [ebp-3C]`

              ==== > `ECX=98BADCFE`

:16470709 `50`                   `push eax`

:1647070A `51`                   `push ecx`

:1647070B `E870E6FFFF`           `call 1646ED80`

              ==== > `4F8D92B9 + 98BADCFE=E8486FB7`

:16470710 `8BD0`                 `mov edx, eax`

              ==== > `EDX=E8486FB7`

:16470712 `8D4DA0`               `lea ecx, dword ptr [ebp-60]`

:16470715 `FFD6`                 `call esi`

:16470717 `50`                   `push eax`

:16470718 `8BD3`                 `mov edx, ebx`

              ==== > `EDX=E8A4602C`

:1647071A `8D4DA4`               `lea ecx, dword ptr [ebp-5C]`

:1647071D `FFD6`                 `call esi`

:1647071F `50`                   `push eax`

:16470720 `E85BE6FFFF`           `call 1646ED80`

              ==== > `E8486FB7 + E8A4602C=D0ECCFE3`

:16470725 `8BD0`                 `mov edx, eax`

              ==== > `EBX=D0ECCFE3`

:16470727 `8D4DC4`               `lea ecx, dword ptr [ebp-3C]`

:1647072A `FFD6`                 `call esi`

:1647072C `8D559C`               `lea edx, dword ptr [ebp-64]`

:1647072F `8D45A0`               `lea eax, dword ptr [ebp-60]`

:16470732 `52`                   `push edx`

:16470733 `8D4DA4`               `lea ecx, dword ptr [ebp-5C]`

:16470736 `50`                   `push eax`

:16470737 `8D55A8`               `lea edx, dword ptr [ebp-58]`

:1647073A `51`                   `push ecx`

:1647073B `8D45AC`               `lea eax, dword ptr [ebp-54]`

:1647073E `52`                   `push edx`

:1647073F `50`                   `push eax`

:16470740 `6A05`                 `push 00000005`

* Reference To: `MSVBVM50.__vbaFreeStrList`, Ord: `0000h`

              |
:16470742 `FF154C634716`         `Call dword ptr [1647634C]`

:16470748 `8B55BC`               `mov edx, dword ptr [ebp-44]`

:1647074B `83C418`               `add esp, 00000018`

:1647074E `8D4DB8`               `lea ecx, dword ptr [ebp-48]`

:16470751 `FFD7`                 `call edi`

:16470753 `8B55C0`               `mov edx, dword ptr [ebp-40]`

:16470756 `8D4DBC`               `lea ecx, dword ptr [ebp-44]`

:16470759 `FFD7`                 `call edi`

:1647075B `8B4DC8`               `mov ecx, dword ptr [ebp-38]`

:1647075E `6A1E`                 `push 0000001E`

:16470760 `51`                   `push ecx`

:16470761 `E84AECFFFF`           `call 1646F3B0`

:16470766 `8BD0`                 `mov edx, eax`

:16470768 `8D4DC0`               `lea ecx, dword ptr [ebp-40]`

:1647076B `FFD6`                 `call esi`

:1647076D `8B55D0`               `mov edx, dword ptr [ebp-30]`

:16470770 `8D4DC8`               `lea ecx, dword ptr [ebp-38]`

:16470773 `FFD7`                 `call edi`

:16470775 `8B55C4`               `mov edx, dword ptr [ebp-3C]`

:16470778 `8D4DD0`               `lea ecx, dword ptr [ebp-30]`

:1647077B `FFD7`                 `call edi`

:1647077D `B801000000`           `mov eax, 00000001`

:16470782 `660345D8`             `add ax, word ptr [ebp-28]`

:16470786 `0F80D1090000`         `jo 1647115D`

:1647078C `8945D8`               `mov dword ptr [ebp-28], eax`

:1647078F `E96AFEFFFF`           `jmp 164705FE`

              ==== > 晕倒 再循环 20 次

…… ……省略…… …… 实在是太多的循环了！…… 晕 ……

第一次加密算法 `CALL` 最后得出：`566C71F6575BD4B395FE34B037F4E04B4DD6D78B`

第二次加密算法 `CALL` 最后得出：`F968F0BBA09FA0E740C482592F641FBD33BAC0FF`

—————————————————————————————————

程序启动注册窗口时比较是否已经注册

* Reference To: `MSVBVM60.__vbaLateIdCallLd`, Ord: `0000h`

              |
:0042EC8A `FF1520114000`         `Call dword ptr [00401120]`

:0042EC90 `83C410`               `add esp, 00000010`

:0042EC93 `50`                   `push eax`

* Reference To: `MSVBVM60.__vbaBoolVar`, Ord: `0000h`

              |
:0042EC94 `FF15BC104000`         `Call dword ptr [004010BC]`

:0042EC9A `8D4DCC`               `lea ecx, dword ptr [ebp-34]`

:0042EC9D `668945A0`             `mov word ptr [ebp-60], ax`

* Reference To: `MSVBVM60.__vbaFreeObj`, Ord: `0000h`

              |
:0042ECA1 `FF1544124000`         `Call dword ptr [00401244]`

:0042ECA7 `8D4DB0`               `lea ecx, dword ptr [ebp-50]`

* Reference To: `MSVBVM60.__vbaFreeVar`, Ord: `0000h`

              |
:0042ECAA `FF1528104000`         `Call dword ptr [00401028]`

:0042ECB0 `66837DA000`           `cmp word ptr [ebp-60], 0000`

:0042ECB5 `8B0E`                 `mov ecx, dword ptr [esi]`

:0042ECB7 `56`                   `push esi`

:0042ECB8 `7433`                 `je 0042ECED`

              ==== > 爆破点！

:0042ECBA `FF91FC020000`         `call dword ptr [ecx+000002FC]`

:0042ECC0 `8D55CC`               `lea edx, dword ptr [ebp-34]`

:0042ECC3 `50`                   `push eax`

:0042ECC4 `52`                   `push edx`

:0042ECC5 `FFD3`                 `call ebx`

:0042ECC7 `8BF0`                 `mov esi, eax`

:0042ECC9 `6A00`                 `push 00000000`

:0042ECCB `56`                   `push esi`

:0042ECCC `8B06`                 `mov eax, dword ptr [esi]`

:0042ECCE `FF909C000000`         `call dword ptr [eax+0000009C]`

:0042ECD4 `85C0`                 `test eax, eax`

:0042ECD6 `DBE2`                 `fclex`

:0042ECD8 `0F8DF4010000`         `jnl 0042EED2`

:0042ECDE `689C000000`           `push 0000009C`
