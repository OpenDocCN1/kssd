
# VCDCUT 4.03

> 原文：[`www.pediy.com/kssd/pediy03/forum1205.htm`](https://www.pediy.com/kssd/pediy03/forum1205.htm)

对 `VCDCUT 4.03` 的分析破解过程

**保护**：注册码和别的未知手段

**下载**：`http://www.seller-club.com/~vcdcut/vcd403.zip`

**功能**：
1. 提供播放器，可以播放 `MPEG`、`VCD` 和其它媒体文件（诸如 `MPG`、`DAT`、`AVI`、`MOV`、`WAV`）。
2. `VCDCutter` 可以从 `MPG` 或 `VCD` 碟片截取媒体画面和 `MPG` 片段，所截 `MPG` 片段，可用系统流（`MPG`），视频流（`M1V`）或音频流（`MP3`）格式存储。
3. 可将截取的影片片段连接成大的 `MPG` 片段，也可以将大的 `MPG` 文件切割成多个小的等长的 `MPG` 片段。
4. 可以将 `MPG` 系统流分割成视频流和音频流（`MPG`→`MP3`/`M1V`）。并支持其逆操作，即把 `MPG` 视频流和音频流打包为 `MPG` 系统流（`MP3`＋`M1V`→`MPG`）。
5. 提供文件格式转换器：`AVI`→`MPG`、`DAT`→`MPG`/`M1V`/`MP3`。
6. 支持播放时截取画面，可以多种格式（`BMP`、`JPG`）存储。

下载了这个文件后，安装运行。提示为非注册版，然后输入注册码并跟踪，找到了一个注册码 `be0034cc-0d849337`。注册，提示注册成功，再运行也没有提示未注册版。但是实际截取并合并时，提示是未注册版，且有做多 `2` 段 `VCD` 和每段最多 `7` 秒的限制。这样就有了下面的分析过程。

```
* Referenced by a CALL at Addresses:
:0042637E , :004350FD

:00430540 A13CC14900        mov eax, dword ptr [0049C13C]  '从这里开始处理把多段 VCD 合并成一个文件的过程
:00430545 81EC04020000      sub esp, 00000204
:0043054B 55                push ebp
:0043054C 33ED              xor ebp, ebp
:0043054E 56                push esi
:0043054F 3BC5              cmp eax, ebp
:00430551 57                push edi
:00430552 0F84EB030000      je 00430943
:00430558 A1E4764800        mov eax, dword ptr [004876E4]
:0043055D 6A10              push 00000010
.
.
.
.
.
.

:0043058F E81CFEFFFF        call 004303B0
:00430594 4E                dec esi
:00430595 75F8              jne 0043058F
:00430597 8B0D3CC14900      mov ecx, dword ptr [0049C13C]   ‘从这里开始的几行是计算 VCD 片断数是否超过 2，超过则设为 2
:0043059D 83F902            cmp ecx, 00000002
:004305A0 7E0B              jle 004305AD
:004305A2 B902000000        mov ecx, 00000002
:004305A7 890D3CC14900      mov dword ptr [0049C13C], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004305A0(C)

:004305AD 3BCD              cmp ecx, ebp
:004305AF 896C240C          mov dword ptr [esp+0C], ebp
:004305B3 7E26              jle 004305DB
:004305B5 8BF1              mov esi, ecx
:004305B7 B8EC854800        mov eax, 004885EC
:004305BC 8974240C          mov dword ptr [esp+0C], esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004305D9(C)

:004305C0 8B48FC            mov ecx, dword ptr [eax-04]     ’这里计算每一段 VCD 片断是否超过 7 秒，超过的话，置成 7 秒代表的值 FA000
:004305C3 8B10              mov edx, dword ptr [eax]
:004305C5 81C100A00F00      add ecx, 000FA000
:004305CB 3BCA              cmp ecx, edx
:004305CD 7C02              jl 004305D1
:004305CF 8BCA              mov ecx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004305CD(C)

:004305D1 8908              mov dword ptr [eax], ecx
:004305D3 052C010000        add eax, 0000012C
:004305D8 4E                dec esi
:004305D9 75E5              jne 004305C0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004305B3(C)

* Possible StringData Ref from Data Obj ->"Demo version can only cut 7 sec "     '提示未注册版只能截取 2 段且每段不得超过 7 秒
                                      ->"for "
                                      |
:004305DB 8B1584AC4400      mov edx, dword ptr [0044AC84]
:004305E1 53                push ebx
:004305E2 8D442414          lea eax, dword ptr [esp+14]
:004305E6 52                push edx
:004305E7 50                push eax

* Reference To: MSVCRT.sprintf, Ord:02B2h
                                      |
:004305E8 E819AE0000        Call 0043B406
:004305ED 8B154C6B4800      mov edx, dword ptr [00486B4C]
:004305F3 83C408            add esp, 00000008
:004305F6 8D4C2414         lea ecx, dword ptr [esp+14]
:004305FA 55                push ebp

* Possible StringData Ref from Data Obj ->"Warning"
                                      |
:004305FB 68787C4400        push 00447C78
:00430600 51                push ecx
:00430601 52                push edx

* Reference To: USER32.MessageBoxA, Ord:01BEh
                                      |
:00430602 FF1514C34300      Call dword ptr [0043C314] '调用 Messagebox,提示未注册版只能......
:00430608 8B3D3CC14900      mov edi, dword ptr [0049C13C]
:0043060E C705DC184A0001000000  mov dword ptr [004A18DC], 00000001
:00430618 BB39020000        mov ebx, 00000239

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0043064C(C)

:0043061D 3BFD              cmp edi, ebp
:0043061F 896C2410          mov dword ptr [esp+10], ebp
:00430623 7E26              jle 0043064B
:00430625 B8EC854800        mov eax, 004885EC
:0043062A 8BF7              mov esi, edi
:0043062C 897C2410          mov dword ptr [esp+10], edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00430649(C)

:00430630 8B48FC            mov ecx, dword ptr [eax-04]     ’这里再一次计算每一段 VCD 片断是否超过 7 秒,超过的话，置成 7 秒代表的值 FA000
:00430633 8B10              mov edx, dword ptr [eax]
:00430635 81C100A00F00      add ecx, 000FA000
:0043063B 3BCA              cmp ecx, edx
:0043063D 7C02              jl 00430641
:0043063F 8BCA              mov ecx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0043063D(C)

:00430641 8908              mov dword ptr [eax], ecx
:00430643 052C010000        add eax, 0000012C
:00430648 4E                dec esi
:00430649 75E5              jne 00430630

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00430623(C)

:0043064B 4B                dec ebx
:0043064C 75CF              jne 0043061D
:0043064E A1CCC14900        mov eax, dword ptr [0049C1CC]
:00430653 5B                pop ebx
:00430654 3BC5              cmp eax, ebp
:00430656 0F840C010000      je 00430768                  '这里如果不跳走的话，到 cs:430767 这一段会把选择的几个片段分别做成文件
:0043065C 3BFD              cmp edi, ebp
:0043065E 896C240C          mov dword ptr [esp+0C], ebp
:00430662 0F8E18020000      jle 00430880

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00430727(C)

:00430668 E8B358FEFF        call 00415F20
:0043066D 8B44240C          mov eax, dword ptr [esp+0C]
:00430671 8D942410010000    lea edx, dword ptr [esp+00000110]
:00430678 8D0440            lea eax, dword ptr [eax+2*eax]
:0043067B 8D0480            lea eax, dword ptr [eax+4*eax]
:0043067E 8D0480            lea eax, dword ptr [eax+4*eax]
:00430681 8D0C8508864800    lea ecx, dword ptr [4*eax+00488608]
:00430688 51                push ecx
:00430689 52                push edx
:0043068A E8C1020000        call 00430950      '这里打开原始文件
:0043068F 8DBC2418010000    lea edi, dword ptr [esp+00000118]
:00430696 83C9FF            or ecx, FFFFFFFF
:00430699 33C0              xor eax, eax
:0043069B 83C408            add esp, 00000008
:0043069E F2                repnz
:0043069F AE                scasb
:004306A0 F7D1              not ecx
:004306A2 49                dec ecx
:004306A3 0F84E7000000      je 00430790
:004306A9 8D842410010000    lea eax, dword ptr [esp+00000110]
```



\* Possible StringData Ref from Data Obj ->`"wb"`   
|   
`:004306B0 6888F04300  push 0043F088`   
`:004306B5 50  push eax`   
\* Reference To: `MSVCRT.fopen`, Ord:`0257h`   
|   
`:004306B6 E851AD0000  Call 0043B40C  '创建要生成的文件`   
`:004306BB 8BF0  mov esi, eax`   
`:004306BD 83C408  add esp, 00000008`   
`:004306C0 3BF5  cmp esi, ebp`   
`:004306C2 746E  je 00430732  '创建文件失败，跳到出错的处理`   
`:004306C4 BF0C000000  mov edi, 0000000C`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:004306CF(C)`   
|   
`:004306C9 E8E2FCFFFF  call 004303B0`   
`:004306CE 4F  dec edi`   
`:004306CF 75F8  jne 004306C9`   
`:004306D1 8B44240C  mov eax, dword ptr [esp+0C]`   
`:004306D5 8B0D3CC14900  mov ecx, dword ptr [0049C13C]`   
`:004306DB 8D54240C  lea edx, dword ptr [esp+0C]`   
`:004306DF 51  push ecx`   
`:004306E0 8D0440  lea eax, dword ptr [eax+2*eax]`   
`:004306E3 52  push edx`   
`:004306E4 8D0480  lea eax, dword ptr [eax+4*eax]`   
`:004306E7 8D0480  lea eax, dword ptr [eax+4*eax]`   
`:004306EA C1E002  shl eax, 02`   
`:004306ED 8B88EC854800  mov ecx, dword ptr [eax+004885EC]`   
`:004306F3 8B90E8854800  mov edx, dword ptr [eax+004885E8]`   
`:004306F9 51  push ecx  '这里是处理后的代表秒数的数值，未注册时是 FA000`   
`:004306FA 8D8008864800  lea eax, dword ptr [eax+00488608]`   
`:00430700 52  push edx`   
`:00430701 50  push eax`   
`:00430702 56  push esi`   
`:00430703 E8584BFEFF  call 00415260  '这里是把一段 VCD 片段截取过来，放到新文件里面。`   
`:00430708 E83358FEFF  call 00415F40`   
`:0043070D 56  push esi`   
\* Reference To: `MSVCRT.fclose`, Ord:`024Ch`   
|   
`:0043070E E8EDAC0000  Call 0043B400`   
`:00430713 8B442428  mov eax, dword ptr [esp+28]`   
`:00430717 8B0D3CC14900  mov ecx, dword ptr [0049C13C]`   
`:0043071D 83C41C  add esp, 0000001C`   
`:00430720 40  inc eax`   
`:00430721 3BC1  cmp eax, ecx  '这里判断是否所有的片断数都已经处理完`   
`:00430723 8944240C  mov dword ptr [esp+0C], eax`   
`:00430727 0F8C3BFFFFFF  jl 00430668  '没有处理完，继续处理`   
`:0043072D E939010000  jmp 0043086B`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:004306C2(C)`   
|   
`:00430732 8D8C2410010000  lea ecx, dword ptr [esp+00000110]  '打开文件失败的处理`   
`:00430739 8D542410  lea edx, dword ptr [esp+10]`   
`:0043073D 51  push ecx`   
\* Possible StringData Ref from Data Obj ->`"Can't create file: %s"`   
|   
`:0043073E 68B4BF4400  push 0044BFB4`   
`:00430743 52  push edx`   
`:00430744 892DDC184A00  mov dword ptr [004A18DC], ebp`   
\* Reference To: `MSVCRT.sprintf`, Ord:`02B2h`   
|   
`:0043074A E8B7AC0000  Call 0043B406`   
`:0043074F 55  push ebp`   
`:00430750 8D442420  lea eax, dword ptr [esp+20]`   
`:00430754 55  push ebp`   
`:00430755 50  push eax`   
`:00430756 E8F550FFFF  call 00425850`   
`:0043075B 83C418  add esp, 00000018`   
`:0043075E 5F  pop edi`   
`:0043075F 5E  pop esi`   
`:00430760 5D  pop ebp`   
`:00430761 81C404020000  add esp, 00000204`   
`:00430767 C3  ret`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:00430656(C)`   
|   
`:00430768 8D8C2410010000  lea ecx, dword ptr [esp+00000110] '从这里到这个 call 结束，是把选定的 VCD 片段合并成一个文件`   
`'并做一些善后工作`   
`:0043076F 6808864800  push 00488608`   
`:00430774 51  push ecx`   
`:00430775 E8D6010000  call 00430950  '这里打开原始文件`   
`:0043077A 8DBC2418010000  lea edi, dword ptr [esp+00000118]`   
`:00430781 83C9FF  or ecx, FFFFFFFF`   
`:00430784 33C0  xor eax, eax`   
`:00430786 83C408  add esp, 00000008`   
`:00430789 F2  repnz`   
`:0043078A AE  scasb`   
`:0043078B F7D1  not ecx`   
`:0043078D 49  dec ecx`   
`:0043078E 7510  jne 004307A0`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:004306A3(C)`   
|   
`:00430790 5F  pop edi`   
`:00430791 892DDC184A00  mov dword ptr [004A18DC], ebp`   
`:00430797 5E  pop esi`   
`:00430798 5D  pop ebp`   
`:00430799 81C404020000  add esp, 00000204`   
`:0043079F C3  ret`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:0043078E(C)`   
|   
`:004307A0 E87B57FEFF  call 00415F20`   
`:004307A5 8D942410010000  lea edx, dword ptr [esp+00000110]`   
\* Possible StringData Ref from Data Obj ->`"wb"`   
|   
`:004307AC 6888F04300  push 0043F088`   
`:004307B1 52  push edx`   
\* Reference To: `MSVCRT.fopen`, Ord:`0257h`   
|   
`:004307B2 E855AC0000  Call 0043B40C  '这里是创建一个新的文件`   
`:004307B7 8BF0  mov esi, eax`   
`:004307B9 83C408  add esp, 00000008`   
`:004307BC 3BF5  cmp esi, ebp`   
`:004307BE 7536  jne 004307F6`   
`:004307C0 8D842410010000  lea eax, dword ptr [esp+00000110] '创建文件失败的话，显示出错信息`   
`:004307C7 8D4C2410  lea ecx, dword ptr [esp+10]`   
`:004307CB 50  push eax`   
\* Possible StringData Ref from Data Obj ->`"Can't create file: %s"`   
|   
`:004307CC 68B4BF4400  push 0044BFB4`   
`:004307D1 51  push ecx`   
`:004307D2 892DDC184A00  mov dword ptr [004A18DC], ebp`   
\* Reference To: `MSVCRT.sprintf`, Ord:`02B2h`   
|   
`:004307D8 E829AC0000  Call 0043B406`   
`:004307DD 55  push ebp`   
`:004307DE 8D542420  lea edx, dword ptr [esp+20]`   
`:004307E2 55  push ebp`   
`:004307E3 52  push edx`   
`:004307E4 E86750FFFF  call 00425850`   
`:004307E9 83C418  add esp, 00000018`   
`:004307EC 5F  pop edi`   
`:004307ED 5E  pop esi`   
`:004307EE 5D  pop ebp`   
`:004307EF 81C404020000  add esp, 00000204`   
`:004307F5 C3  ret`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:004307BE(C)`   
|   
`:004307F6 A13CC14900  mov eax, dword ptr [0049C13C]  '[49c13c]放的是片段数`   
`:004307FB 896C240C  mov dword ptr [esp+0C], ebp`   
`:004307FF 3BC5  cmp eax, ebp`   
`:00430801 7E5A  jle 0043085D`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:0043085B(C)`   
|   
`:00430803 BF0C000000  mov edi, 0000000C`   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|`:0043080E(C)`   
|   
`:00430808 E8A3FBFFFF  call 004303B0`   
`:0043080D 4F  dec edi`   



:0043080E `75F8` `jne 00430808`

:00430810 `8B44240C` `mov eax, dword ptr [esp+0C]`

:00430814 `8B0D3CC14900` `mov ecx, dword ptr [0049C13C]`

:0043081A `8D54240C` `lea edx, dword ptr [esp+0C]`

:0043081E `51` `push ecx`

:0043081F `8D0440` `lea eax, dword ptr [eax+2*eax]`

:00430822 `52` `push edx`

:00430823 `8D0480` `lea eax, dword ptr [eax+4*eax]`

:00430826 `8D0480` `lea eax, dword ptr [eax+4*eax]`

:00430829 `C1E002` `shl eax, 02`

:0043082C `8B88EC854800` `mov ecx, dword ptr [eax+004885EC]`

:00430832 `8B90E8854800` `mov edx, dword ptr [eax+004885E8]`

:00430838 `51` `push ecx` `'这里是处理后的代表秒数的数值，未注册时是 FA000`

:00430839 `8D8008864800` `lea eax, dword ptr [eax+00488608]`

:0043083F `52` `push edx`

:00430840 `50` `push eax`

:00430841 `56` `push esi`

:00430842 `E8194AFEFF` `call 00415260` `'这里是把一段 VCD 片段截取过来，放到新文件里面。`

:00430847 `8B442424` `mov eax, dword ptr [esp+24]`

:0043084B `8B0D3CC14900` `mov ecx, dword ptr [0049C13C]`

:00430851 `83C418` `add esp, 00000018`

:00430854 `40` `inc eax`

:00430855 `3BC1` `cmp eax, ecx` `'这里判断是否所有的片断数都已经处理完`

:00430857 `8944240C` `mov dword ptr [esp+0C], eax`

:0043085B `7CA6` `jl 00430803` `'没有处理完，继续处理`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`
`|:00430801(C)`

:0043085D `E8DE56FEFF` `call 00415F40`

:00430862 `56` `push esi`

.

.

.

.

`'上面的内容搞定后，以为万事大吉，但是去截取时，发现还是只能截取 7 秒，又进入 415260 里面，发现了下面的“暗桩”`

`* Referenced by a CALL at Address:`
`|:00415325`

:004153C0 `81EC04010000` `sub esp, 00000104`

:004153C6 `53` `push ebx`

:004153C7 `8B9C2414010000` `mov ebx, dword ptr [esp+00000114]`

:004153CE `55` `push ebp`

:004153CF `56` `push esi`

:004153D0 `57` `push edi`

:004153D1 `8BBC2424010000` `mov edi, dword ptr [esp+00000124]`

:004153D8 `8DB300A00F00` `lea esi, dword ptr [ebx+000FA000]` `’这里 ESI 一般被赋值 FA000，代表 7 秒`

:004153DE `3BFE` `cmp edi, esi` `'在这里判断 EDI 的值是否大于 FA000(7 秒)`

:004153E0 `7C02` `jl 004153E4` `'小于则跳走`

:004153E2 `8BFE` `mov edi, esi` `'否则，把 ESI（FA000）赋给 EDI，`
`'在这里，我曾经把 EDI 的值给得比较大，ESI 保持 FA000，实际还是截取 7 秒`
`'所有推测 ESI 里面放的应该实际要截取的时间换算出来的值。`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`
`|:004153E0(C)`

:004153E4 `8B8C2428010000` `mov ecx, dword ptr [esp+00000128]`

:004153EB `8B84242C010000` `mov eax, dword ptr [esp+0000012C]`

:004153F2 `50` `push eax`

:004153F3 `8D442418` `lea eax, dword ptr [esp+18]`

:004153F7 `8B11` `mov edx, dword ptr [ecx]`

:004153F9 `42` `inc edx`

:004153FA `52` `push edx`

综上所述，用下面的方法改之:

```
cs:4305a7 909090909090
cs:5305cd 9090
cs:430602 909090909090
cs:43063d 9090
cs:4153e0 90908BF7
```

注册码在`cs:41ffb2`处，`d eax`即可看到。另外，它是去到`\\windows\\system\\cdplayer.dat`找注册码的。注册成功后，它会把注册码加密然后放到`\\windows\\system\\cdplayer.dat`里面,没有注册成功时也有这个文件，不过内容不对。

`javaj901` 做于 `2001,8,6`

`转载请保持完整`

`4dd466d0e1e0cc123afbe504da2462ca`
