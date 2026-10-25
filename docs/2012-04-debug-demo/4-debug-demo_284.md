
# udioconverterV31B

> 原文：[`www.pediy.com/kssd/pediy03/forum1287.htm`](https://www.pediy.com/kssd/pediy03/forum1287.htm)

说明: 本文仅供研究用; 这个软件很不错, 希望有经济实力的同志还是支持一下软件作者吧! ^\_^

## 1. 去掉版权提示窗口

`* Reference To: audconv.CreateLicenseManager, Ord:0019h`

```assembly
:00408317 FF1520044200    Call dword ptr [00420420]
:0040831D 83C41C          add esp, 0000001C
:00408320 85C0            test eax, eax
:00408322 0F8C12010000    jl 0040843A
:00408328 8B4DFC          mov ecx, dword ptr [ebp-04]
:0040832B 8B01            mov eax, dword ptr [ecx]
:0040832D FF501C          call [eax+1C]             ;<--此 call 执行后,eax=0
:00408330 85C0            test eax, eax             ;  程序将跳到 408386
:00408332 7452            je 00408386
:00408334 8B4DFC          mov ecx, dword ptr [ebp-04]
:00408337 33FF            xor edi, edi
:00408339 57              push edi
:0040833A 8B01            mov eax, dword ptr [ecx]
:0040833C FF10            call dword ptr [eax]
:0040833E 3D05400080      cmp eax, 80004005
:00408343 740A            je 0040834F
:00408345 3BC7            cmp eax, edi
:00408347 0F85ED000000    jne 0040843A
:0040834D EB39            jmp 00408388

.
.
.

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408332(C)
|

:00408386 33FF            xor edi, edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040834D(U)
|

:00408388 57              push edi
* Reference To: audconv.ShowEULA, Ord:002Dh
|

:00408389 FF1524044200    Call dword ptr [00420424]
:0040838F 85C0            test eax, eax
:00408391 59              pop ecx
:00408392 0F8CA2000000    jl 0040843A
:00408398 8B4DFC          mov ecx, dword ptr [ebp-04]
:0040839B 57              push edi
:0040839C 8B01            mov eax, dword ptr [ecx]
:0040839E FF5014          call [eax+14]             ;<--呼叫协议窗口
:004083A1 85C0            test eax, eax             ;<--按 Agree 按钮,eax=0;
                                                    ;  按 Cancel 按钮,eax=1
:004083A3 0F8591000000    jne 0040843A              ;<--eax=1,则跳到 40843a
                                                    ;  一跳到 40843a,就 game over 了
:004083A9 833DE462420001  cmp dword ptr [004262E4], 00000001
:004083B0 7E62            jle 00408414              ;<--从此处,跳到 408414
                                                    ;  408414 是主程序开始的地方

.
.
.

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004083B0(C), :004083DC(C)
|

:00408414 FF7514          push [ebp+14]
:00408417 8D8DB8FEFFFF    lea ecx, dword ptr [ebp+FFFFFEB8]
:0040841D FF7510          push [ebp+10]
:00408420 E894170000      call 00409BB9
:00408425 8945F8          mov dword ptr [ebp-08], eax
```

经过动态跟踪, 发现: 从 `00408388` 到 `00408414`, `ecx` 和 `edx` 的值有变化 (`edx=0`), 而 `408417` 的指令将 `ecx` 的值覆盖.

所以, 只要在 `00408388` 处, 改指令为:

```assembly
    mov edx,0
    jmp 408414
```

即可.

因此, 在 `audconv.exe` 文件中的 `00008388` 处, 找到 `57 FF 15 24 04 42 00 85 C0 59`,

    ^^ ^^ ^^ ^^ ^^ ^^ ^^ ^^ ^^ ^^

将其改成 `BA 00 00 00 00 E9 82 00 00 00`

## 2. 去掉版权提示窗口后, 如果过期, 关闭时还有提示窗口出现, 下面将去掉该提示窗口

`* Reference To: audconv.CreateLicenseManager, Ord:0019h`

```assembly
:0040C029 FF1520044200    Call dword ptr [00420420]
:0040C02F 85C0            test eax, eax
:0040C031 59              pop ecx
:0040C032 7C13            jl 0040C047              ;<--若在此处跳转,则过期提示窗口不会出现
:0040C034 8B4DFC          mov ecx, dword ptr [ebp-04]
:0040C037 FF7604          push [esi+04]
:0040C03A 8B01            mov eax, dword ptr [ecx]
:0040C03C FF5018          call [eax+18]            ;<--出现过期提示窗口
:0040C03F 8B4DFC          mov ecx, dword ptr [ebp-04]
:0040C042 8B01            mov eax, dword ptr [ecx]
:0040C044 FF5020          call [eax+20]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C032(C)
|

:0040C047 8B4514          mov eax, dword ptr [ebp+14]
:0040C04A 5E              pop esi
:0040C04B 832000          and dword ptr [eax], 00000000
:0040C04E 33C0            xor eax, eax
:0040C050 C9              leave
:0040C051 C21000          ret 0010
```

所以, 在 `audconv.exe` 文件的 `0000C032` 处, 将 `7C` 改为 `7D`.

## 3. 在转换文件时, 还弹出评估版提示窗口, 程序用的是 `audconv.dll` 中的代码.

代码如下:

`* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:10004485(U)`

```assembly
:10004489 8B4DFC          mov ecx, dword ptr [ebp-04]
:1000448C 8B01            mov eax, dword ptr [ecx]
:1000448E FF5010          call [eax+10]
:10004491 85C0            test eax, eax
:10004493 0F8540030000    jne 100047D9
:10004499 8B45E0          mov eax, dword ptr [ebp-20]

* Possible Reference to String Resource ID=00010: "MPEG Layer-1 Audio Files (.mp1)"
|

:1000449C 6A0A            push 0000000A
:1000449E 33D2            xor edx, edx
:100044A0 59              pop ecx
:100044A1 F7F1            div ecx
:100044A3 85D2            test edx, edx
:100044A5 751B            jne 100044C2
:100044A7 8B4DFC          mov ecx, dword ptr [ebp-04]
:100044AA 6A40            push 00000040           ;<--由这里跳到 100044c2,
                                                    ;  评估版提示窗口就不会出现

* Possible StringData Ref from Data Obj ->"Audio Converter"
|

:100044AC 6800530810      push 10085300

* Possible StringData Ref from Data Obj ->"This is an evaluation version "
                                ->"of Audio Converter."
|

:100044B1 681C5E0810      push 10085E1C
:100044B6 8B01            mov eax, dword ptr [ecx]
:100044B8 FF5024          call [eax+24]
:100044BB 50              push eax

* Reference To: USER32.MessageBoxA, Ord:01C3h
|

:100044BC FF15E0430710    Call dword ptr [100743E0]   ;<--call 评估版提示窗口

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100044A5(C)
|

:100044C2 8B47FC          mov eax, dword ptr [edi-04]
```

因此, 将 `audconv.dll` 中的从 `44aa` 开始的代码改为 `e9 13 00 00 00 90 90`, 即可.

另: `metadata` 窗口的限制没有解除, 特别是 `comments` 的限制, 还望高手指教!

## 4. 去掉用鼠标右键菜单转换文件时的版权提示窗口

用鼠标右键菜单转换文件时, 调用的是 `acshext.dll` (每次重启后, 程序运行时的 image base 都有所改变, 而 `w32dsm` 中的 image base 为 `10000000`, 所以下断点时要注意.)

`* Reference To: audconv.CreateLicenseManager, Ord:0019h`

```assembly
:1000586A FF15BCF20010    Call dword ptr [1000F2BC]
:10005870 85C0            test eax, eax
:10005872 59              pop ecx
:10005873 7C4B            jl 100058C0
:10005875 8B4DEC          mov ecx, dword ptr [ebp-14]
:10005878 8B01            mov eax, dword ptr [ecx]
:1000587A FF501C          call [eax+1C]
:1000587D 85C0            test eax, eax
:1000587F 7418            je 10005899             ;<--跳转到 10005899

........

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000587F(C)
|
```



### 修改代码逻辑

:10005899 8B4DEC  mov ecx, dword ptr [ebp-14]   

:1000589C FF7308  push [ebx+08]   

:1000589F 8B01  mov eax, dword ptr [ecx]   

:100058A1 FF5014  call [eax+14] ; <--呼叫协议窗口   

:100058A4 85C0  test eax, eax ; <--按 `Agree` 按钮, `eax=0`;   

; 按 `Cancel` 按钮, `eax=1`   

:100058A6 740A  je 100058B2 ; <--`eax=0`, 则跳到 100058B2,   

; 否则, 就 `game over`   

所以, 将 `1000587F` 改为 `je 100058B2`, 即   

:1000587F 7418 => 7431   

### 5. 去掉用鼠标右键菜单转换文件时的过期提示窗口

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

:10005BB2(C), :10005BC1(C)   

:10005C0B 8B4DEC  mov ecx, dword ptr [ebp-14]   

:10005C0E 3BCE  cmp ecx, esi   

:10005C10 7410  je 10005C22   

:10005C12 8B01  mov eax, dword ptr [ecx]   

:10005C14 FF7308  push [ebx+08]   

:10005C17 FF5018  call [eax+18] ; <--调出过期提示窗口   

所以, 将 `10005C10` 的 `je 10005C22` 改为 `jmps 10005C22`, 即   

:10005C10 7410 => eb10 

### 6. 去掉 Metadata 窗口的 comments 和 User defined URL link 标签中 AudConv 自动添加的内容

在 `audconv.dll` 中,   

:10014232 E851E2FFFF  call 10012488   

:10014237 8B4DFC  mov ecx, dword ptr [ebp-04]   

:1001423A 83C40C  add esp, 0000000C   

:1001423D 85C9  test ecx, ecx   

:1001423F 742B  je 1001426C ; <--跳转   

:10014241 8B01  mov eax, dword ptr [ecx]   

* Possible StringData Ref from Data Obj -> "Audio Converter Home Page"   

:10014243 68F48B0810  push 10088BF4   

:10014248 FF5030  call [eax+30]   

:1001424B 8B4DFC  mov ecx, dword ptr [ebp-04]   

* Possible StringData Ref from Data Obj -> "http://www.audconv.com"   

:1001424E 68C48B0810  push 10088BC4   

.........   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

:1001423F(C)   

:1001426C 8D45F8  lea eax, dword ptr [ebp-08]   

:1001426F 50  push eax   

:10014270 6870470710  push 10074770   

:10014275 6A03  push 00000003   

* Reference To: `audconv.MetadataCreateFrame`   

:10014277 E80CE2FFFF  call 10012488   

:1001427C 8B4DF8  mov ecx, dword ptr [ebp-08]   

:1001427F 83C40C  add esp, 0000000C   

:10014282 85C9  test ecx, ecx   

:10014284 742B  je 100142B1 ; <--跳转   

:10014286 8B01  mov eax, dword ptr [ecx]   

* Possible StringData Ref from Data Obj -> "Audio Converter"   

:10014288 68A48B0810  push 10088BA4   

:1001428D FF5030  call [eax+30]   

:10014290 8B4DF8  mov ecx, dword ptr [ebp-08]   

* Possible StringData Ref from Data Obj -> "This audio file has been encoded "   

-> "by an evaluation version of Audio "   

-> "Converter for evaluation purposes "   

-> "only."   

:10014293 68188A0810  push 10088A18   

所以, 在 `audconv.dll` 中,   

:1001423F 742B => EB2B   

:10014284 742B => EB2B 

### 7. 解除 Comments 窗口的限制和恢复 User defined URL link 窗口的功能

代码在 `Audconv.dll` 中   

#### 1> 解除 Comments 窗口的限制

这里是用 `DialogBoxParamA` 来显示每个 tag 的对话框的   

:10013757 6A00  push 00000000   

* Possible StringData Ref from Code Obj -> "VWh"   

:10013759 68A2970010  push 100097A2   

:1001375E FF742410  push [esp+10]   

:10013762 68D2000000  push 000000D2 ; <--第一个 tag(Comments)的资源号   

:10013767 50  push eax   

* Reference To: `USER32.DialogBoxParamA`, Ord:0095h   

:10013768 FF157C430710  Call dword ptr [1007437C]   

要激活窗口, 就要跟进 10013768 的 `call USER32.DialogBoxParamA`. 你会发现有下面一段代码：   

* Reference To: `USER32.SetWindowTextA`, Ord:0263h   

:1000FB3D FF1590430710  Call dword ptr [10074390]   

* Reference To: `USER32.SendDlgItemMessageA`, Ord:0213h   

:1000FB43 8B2D84440710  mov ebp, dword ptr [10074484]   

:1000FB49 53  push ebx   

:1000FB4A 6A01  push 00000001   

:1000FB4C 68CF000000  push 000000CF   

:1000FB51 68F0030000  push 000003F0 ; <--Description 下面的 Edit 的 ID 号   

:1000FB56 FF36  push dword ptr [esi]   

:1000FB58 FFD5  call ebp   

* Reference To: `USER32.GetDlgItem`, Ord:0105h   

:1000FB5A 8B1DB4430710  mov ebx, dword ptr [100743B4]   

:1000FB60 680B040000  push 0000040B ; <--Language 下面的 ComboBox 的 ID 号   

:1000FB65 FF36  push dword ptr [esi]   

:1000FB67 FFD3  call ebx   

:1000FB69 6A00  push 00000000 ; <--改为 01 才能激活 ComboBox   

:1000FB6B 50  push eax   

* Reference To: `USER32.EnableWindow`, Ord:00B9h   

:1000FB6C FF152C440710  Call dword ptr [1007442C]   

:1000FB72 6A00  push 00000000   

:1000FB74 6A01  push 00000001   

:1000FB76 68CF000000  push 000000CF   

:1000FB7B 68E9030000  push 000003E9 ; <--Text 下面的 Edit 的 ID 号   

:1000FB80 FF36  push dword ptr [esi]   

:1000FB82 FFD5  call ebp   

:1000FB84 680D040000  push 0000040D ; <--右下角的 Static 的 ID 号   

:1000FB89 FF36  push dword ptr [esi]   

:1000FB8B FFD3  call ebx   

:1000FB8D 6A05  push 00000005   

:1000FB8F 50  push eax   

* Reference To: `USER32.ShowWindow`, Ord:026Fh   

:1000FB90 FF1590440710  Call dword ptr [10074490]   

:1000FB96 68ED030000  push 000003ED ; <--未知控件 ID 号   

:1000FB9B FF36  push dword ptr [esi]   

:1000FB9D FFD3  call ebx   

:1000FB9F 6A00  push 00000000   

:1000FBA1 50  push eax   

* Reference To: `USER32.EnableWindow`, Ord:00B9h   

:1000FBA2 FF152C440710  Call dword ptr [1007442C]   

:1000FBA8 8B8740080000  mov eax, dword ptr [edi+00000840]   

:1000FBAE 8DAF3C080000  lea ebp, dword ptr [edi+0000083C]   

:1000FBB4 85C0  test eax, eax   

:1000FBB6 7505  jne 1000FBBD   

:1000FBB8 B820500710  mov eax, 10075020   

所以, 将 `Audconv.dll` 中的   

:1000FB4C 68CF000000 => 68FF000000 (激活 Description 下面的 Edit)   

:1000FB69 6A00 => 6A01 (激活 Language 下面的 ComboBox)   

:1000FB76 68CF000000 => 68FF000000 (激活 Text 下面的 Edit)   

:1000FB84 680D040000 => EB10909090 (除去右下角的 Static)   

(:1000FB9F 6A00 => 6A01)   

#### 2> 恢复 User defined URL link 窗口的功能

在 `Audconv.dll` 的 10013DBD 比较 `EAX` 是否为 4A, 是, 则弹出 "This frame can only be edited in the licensed version." 对话框.   

:10013DB8 E813D00000  call 10020DD0   

:10013DBD 83F84A  cmp eax, 0000004A   

:10013DC0 7430  je 10013DF2 ; <--一跳就没戏了   



:10013DC2 `8D4604`               `lea eax, dword ptr [esi+04]`

....

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10013DC0(C)`

\* Possible StringData Ref from Data Obj -> `"Audio Converter"`

`:10013DF2` `6A40`             `push 00000040`

\* Possible StringData Ref from Data Obj -> `"This frame can only be edited "` -> `"in the licensed version."`

`:10013DF4` `6800530810`       `push 10085300`

`:10013DF9` `68E0890810`       `push 100889E0`

`:10013DFE` `FF742414`         `push [esp+14]`

\* Reference To: `USER32.MessageBoxA`, `Ord:01C3h`

`:10013E02` `FF15E0430710`     `Call dword ptr [100743E0]`

所以，将 `Audconv.dll` 中的

`:10013DC0` `7430`   => `9090`

呵呵...打完收工啰! 应该是没问题了.

另：在 `Metadata` 窗口中点击 `Save` 按钮（特别是连续几次"打开 `Metadata` 窗口、`Save`"的时候），有时候会清空所有 `Tag`! 不知道你们有没有碰到这种情况. `4dd466d0e1e0cc123afbe504da2462ca`
