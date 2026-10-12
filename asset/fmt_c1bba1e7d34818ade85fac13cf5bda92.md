
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
