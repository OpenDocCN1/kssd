
# SuperCapture V4.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6245.htm`](https://www.pediy.com/kssd/pediy06/pediy6245.htm)

解决自校验 + 注册爆破——超级屏捕(SuperCapture) V4.01 专业版

下载页面： `http://www.skycn.com/soft/9511.html`

软件大小： 1108 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 图像捕捉

应用平台： `Win9x/NT/2000/XP`

加入时间： 2002-10-17 11:40:01

下载次数： 28765

推荐等级： *****

开发商： `http://www.SuperCapture.com/`

**【软件简介】**：`SuperCapture` 是一款非常强大的专业图像捕捉软件。它是中国首届共享软件大赛优秀软件。`SuperCapture` 专业版 4.0 包含了标准版的所有功能，同时它还有很多专业的功能：例如从网页捕捉所有 `Flash`；播放 `Flash`；将桌面活动图像捕捉为 `AVI` 视频文件（提供多种压缩方式）；轻松地从您的电脑上捕捉您想要的任何图标 `Icon`。例如从一个文件中，或者一个文件夹中，甚至是您机器的整个硬盘；它可以轻松捕捉全屏(包括 `DirectX`, `Direct3D` 游戏屏幕)、窗口、控件、区域、固定区域、不规则区域。轻松抓取特殊菜单、鼠标、超长屏幕、网页、网页图像(可将网页内图片一次全部抓取)；支持定时捕捉、自定义热键、缩略图方式浏览；支持 `BMP/JPEG/TIF/PNG/GIF` 等 17 种图形格式的浏览与转换。可将捕捉后的图形直接发送到 `Microsoft` `Office` 文档(如 `Word`, `Excel`, `PowerPoint`)；支持多语言。它适用于任何需要对屏幕图像处理的用户。使用 `SuperCapture` 超级屏捕能极大节省您处理屏幕图像的时间，提高工作效率。

**【软件限制】**：`NAG`、45 天试用

**【作者声明】**：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`PE-Scan`、`W32Dasm` `9.0` 白金版、`Hiew`

—————————————————————————————————

**【过程】**

`SuperCap.exe` 是 `PECompact` `1.68` - `1.84` 壳，用 `PE-Scan` 脱之。`531K`->`1.35M`。 `VC++` `6.0` 编写。

《看雪论坛精华 4》里有好几篇 超级屏捕 以前的版本的破解，虽然这个 `V4.01` 专业版已经发布很长时间了，却没看见破解笔记。`[FCG]` 的 `pLayAr` 兄曾经做过破解版，这次特别感谢 `pLayAr` 兄的指点和 `fxyang` 兄的测试！:-)

呵呵，`10-1` 到了，响应 `DarkNess0ut` 老大的号召，算是送给大家的国庆小礼吧！ :-)

—————————————————————————————————

## 一、反跟踪

程序调用 `IsDebuggerPresent()` 来检测是否有调试器存在，启动时在 `00418C1F` 调用 `IsDebuggerPresent()` 检测，注册时在 `0040DDF0`、`0040DED6`、`0040E208` 调用 `IsDebuggerPresent()` 检测。但是奇怪的是程序还使用了另种方法检测调试器，使 `Ollydbg` 调试时失去响应！

脱壳后的程序碰上屠龙刀 `TRW` 就变乖了。:-) 如果调试原程序，在原程序运行 `1` 分钟后再用 `TRW` 下断！

————————————————————————

* Referenced by a CALL at Addresses:

`:0040DDF0` , `:0040DED6` , `:0040E208` , `:00418C1F`

```asm
:0043F0C0 81EC94000000            sub     esp, 00000094
:0043F0C6 8D442400              lea     eax, dword ptr [esp]
:0043F0CA C744240094000000      mov     [esp], 00000094
:0043F0D2 50                    push    eax
* Reference To: KERNEL32.GetVersionExA, Ord:0000h
                                  |
:0043F0D3 FF159C444D00          Call    dword ptr [004D449C]
:0043F0D9 85C0                  test    eax, eax
:0043F0DB 7427                  je      0043F104
:0043F0DD 837C241001            cmp     dword ptr [esp+10], 00000001
:0043F0E2 7520                  jne     0043F104
:0043F0E4 8B442404              mov     eax, dword ptr [esp+04]
:0043F0E8 83F804                cmp     eax, 00000004
:0043F0EB 770A                  ja      0043F0F7
:0043F0ED 7515                  jne     0043F104
:0043F0EF 8B442408              mov     eax, dword ptr [esp+08]
:0043F0F3 85C0                  test    eax, eax
:0043F0F5 760D                  jbe     0043F104
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043F0EB(C)
|
* Reference To: KERNEL32.IsDebuggerPresent, Ord:0000h
                                  |
:0043F0F7 FF1530444D00          Call    dword ptr [004D4430]
                                  ====>这里调用 IsDebuggerPresent 检测！
:0043F0FD 81C494000000          add     esp, 00000094
:0043F103 C3                    ret
```

—————————————————————————————————

## 二、CRC 自校验暗桩分析

程序脱壳或 `hook.dll` 修改后有自校验，没有提示，而是 `3` 分钟后程序自动退出，这算是这个东东最大的特色吧。

呵呵，作者挺精明的，不给你明确的提示，让 `Cracker` 们慢慢找吧！:-D

为了找这个暗桩可真是费了不少功夫，下 `BPX` `CreateFilea`，程序不断中断，作者布置了迷魂阵。:-(

原来作者设置了 `3` 分钟的定时器，如果校验错误则自动退出，正确则取消定时器！自校验和注册码的验证均在主程序和同目录下的 `hook.dll` 里面进行！

————————————————————————

```asm
:0041BF35 E826820100            call    00434160
                                  ===>关键 CALL！进去看看！ :-)
                                  ===> 在这里动手去除 2 个自校验！爆破点①  :-)
:0041BF3A 8B461C                mov     eax, dword ptr [esi+1C]
:0041BF3D 6A05                  push    00000005
:0041BF3F 50                    push    eax
* Reference To: USER32.KillTimer, Ord:0000h
                                  |
:0041BF40 FF1578484D00          Call    dword ptr [004D4878]
                                  ===>取消 3 分钟 的定时器
:0041BF46 E9A4060000            jmp     0041C5EF
```

————————————————————————

进入关键 CALL：`0041BF35` `call` `00434160`

* Referenced by a CALL at Address:

`:0041BF35`

```asm
:00434160 A18CE85100            mov     eax, dword ptr [0051E88C]
:00434165 81EC00040000          sub     esp, 00000400
:0043416B 56                    push    esi
:0043416C 57                    push    edi
:0043416D 33FF                  xor     edi, edi
:0043416F 8BF1                  mov     esi, ecx
:00434171 85C0                  test    eax, eax
:00434173 7418                  je      0043418D
:00434175 E8B42B0900            call    004C6D2E
:0043417A 8B4008                mov     eax, dword ptr [eax+08]
:0043417D 50                    push    eax
:0043417E FF158CE85100          call    dword ptr [0051E88C]
                                  ===>调用 hook.dll 进行校验主程序！错误则退出！
:00434184 85C0                  test    eax, eax
:00434186 7505                  jne     0043418D
                                  ===>不跳则 OVER！
:00434188 BF01000000            mov     edi, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00434173(C), :00434186(C)
|
:0043418D E81EB20000            call    0043F3B0
                                  ===>在主程序内进行校验 hook.dll！错误则提示程序未正确安装！
                                  ===>限于篇幅，这个就没记录了，跟进去就看到啦。  :-)
:00434192 85C0                  test    eax, eax
:00434194 7408                  je      0043419E
                                  ===>跳则 OVER！
:00434196 85FF                  test    edi, edi
:00434198 0F8489000000          je      00434227
                                  ===>不跳则 OVER！
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00434194(C)
|
:0043419E 8B461C                mov     eax, dword ptr [esi+1C]
:004341A1 6A05                  push    00000005
:004341A3 50                    push    eax
* Reference To: USER32.KillTimer, Ord:0000h
                                  |
:004341A4 FF1578484D00          Call    dword ptr [004D4878]
                                  ===>呵呵，迷惑 Cracker！ :-)
:004341AA 8D4C2408              lea     ecx, dword ptr [esp+08]
:004341AE 6800010000            push    00000100
:004341B3 51                    push    ecx
:004341B4 8B8EAE080000          mov     ecx, dword ptr [esi+000008AE]
:004341BA 6854B25000            push    0050B254
:004341BF E83CA50000            call    0043E700
:004341C4 8B8EAE080000          mov     ecx, dword ptr [esi+000008AE]
:004341CA 8D942408010000        lea     edx, dword ptr [esp+00000108]
:004341D1 6800010000            push    00000100
:004341D6 52                    push    edx
:004341D7 680CB25000            push    0050B20C
:004341DC E81FA50000            call    0043E700
:004341E1 8D842408010000        lea     eax, dword ptr [esp+00000108]
:004341E8 8D4C2408              lea     ecx, dword ptr [esp+08]
:004341EC 50                    push    eax
:004341ED 51                    push    ecx
:004341EE 8D942410020000        lea     edx, dword ptr [esp+00000210]
:004341F5 68E8B15000            push    0050B1E8
:004341FA 52                    push    edx
:004341FB E8A56A0600            call    0049ACA5
:00434200 83C410                add     esp, 00000010
:00434203 8D842408020000        lea     eax, dword ptr [esp+00000208]
:0043420A 6A00                  push    00000000
:0043420C 6A10                  push    00000010
:0043420E 50                    push    eax
:0043420F E89E7E0800            call    004BC0B2
:00434214 8B4E1C                mov     ecx, dword ptr [esi+1C]
:00434217 6A00                  push    00000000
:00434219 6A00                  push    00000000
```



:0043421B 6836050000               `push 00000536`

:00434220 51                     `push ecx`

\* Reference To: `USER32.PostMessageA`, Ord:0000h

                             |

:00434221 FF155C484D00         `Call dword ptr [004D485C]`

                             ==== >BAD BOY！ 未完全安装等等提示  :-(

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00434198(C)

|

:00434227 5F                     `pop edi`

:00434228 5E                     `pop esi`

:00434229 81C400040000           `add esp, 00000400`

:0043422F C3                     `ret`

————————————————————————

进入 `0043417E call dword ptr [0051E88C]`       调用 `hook.dll` 进行校验主程序

Exported `fn()`: `SCAPI_SetHook_WinXP` - Ord:0007h

:1C001294 55                     `push ebp`

:1C001295 8BEC                   `mov ebp, esp`

:1C001297 81EC60020000           `sub esp, 00000260`

:1C00129D C745FCFFFFFFFF         `mov [ebp-04], FFFFFFFF`

:1C0012A4 C785A8FDFFFF00000000  `mov dword ptr [ebp+FFFFFDA8], 00000000`

:1C0012AE C785A4FDFFFF00000000  `mov dword ptr [ebp+FFFFFDA4], 00000000`

:1C0012B8 C785F4FEFFFF00000000  `mov dword ptr [ebp+FFFFFEF4], 00000000`

:1C0012C2 6804010000             `push 00000104`

:1C0012C7 6A00                   `push 00000000`

:1C0012C9 8D85F8FEFFFF           `lea eax, dword ptr [ebp+FFFFFEF8]`

:1C0012CF 50                     `push eax`

\* Reference To: `MSVCRT.memset`, Ord:0299h

                             |

:1C0012D0 E8E50E0000             `Call 1C0021BA`

:1C0012D5 83C40C                 `add esp, 0000000C`

:1C0012D8 6804010000             `push 00000104`

:1C0012DD 8D8DF8FEFFFF           `lea ecx, dword ptr [ebp+FFFFFEF8]`

:1C0012E3 51                     `push ecx`

:1C0012E4 8B5508                 `mov edx, dword ptr [ebp+08]`

:1C0012E7 52                     `push edx`

\* Reference To: `KERNEL32.GetModuleFileNameA`, Ord:0124h

                             |

:1C0012E8 FF150C30001C           `Call dword ptr [1C00300C]`

:1C0012EE 8D85B0FDFFFF           `lea eax, dword ptr [ebp+FFFFFDB0]`

:1C0012F4 50                     `push eax`

:1C0012F5 8D8DF8FEFFFF           `lea ecx, dword ptr [ebp+FFFFFEF8]`

:1C0012FB 51                     `push ecx`

\* Reference To: `KERNEL32.FindFirstFileA`, Ord:0094h

                             |

:1C0012FC FF151030001C           `Call dword ptr [1C003010]`

:1C001302 8945FC                 `mov dword ptr [ebp-04], eax`

:1C001305 837DFCFF               `cmp dword ptr [ebp-04], FFFFFFFF`

:1C001309 7507                   `jne 1C001312`

:1C00130B 33C0                   `xor eax, eax`

:1C00130D E924010000             `jmp 1C001436`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C001309(C)

|

:1C001312 8B95D0FDFFFF           `mov edx, dword ptr [ebp+FFFFFDD0]`

:1C001318 8995F0FEFFFF           `mov dword ptr [ebp+FFFFFEF0], edx`

\* Possible StringData Ref from Data Obj -> "rb"

                             |

:1C00131E 684444001C             `push 1C004444`

:1C001323 8D85F8FEFFFF           `lea eax, dword ptr [ebp+FFFFFEF8]`

:1C001329 50                     `push eax`

\* Reference To: `MSVCRT.fopen`, Ord:0257h

                             |

:1C00132A FF154830001C           `Call dword ptr [1C003048]`

:1C001330 83C408                 `add esp, 00000008`

:1C001333 8985A4FDFFFF           `mov dword ptr [ebp+FFFFFDA4], eax`

:1C001339 83BDA4FDFFFF00         `cmp dword ptr [ebp+FFFFFDA4], 00000000`

:1C001340 7505                   `jne 1C001347`

:1C001342 E9ED000000             `jmp 1C001434`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C001340(C)

|

:1C001347 8B8DF0FEFFFF           `mov ecx, dword ptr [ebp+FFFFFEF0]`

:1C00134D 51                     `push ecx`

\* Reference To: `MSVCRT.malloc`, Ord:0291h

                             |

:1C00134E FF154C30001C           `Call dword ptr [1C00304C]`

:1C001354 83C404                 `add esp, 00000004`

:1C001357 8985A8FDFFFF           `mov dword ptr [ebp+FFFFFDA8], eax`

:1C00135D 83BDA8FDFFFF00         `cmp dword ptr [ebp+FFFFFDA8], 00000000`

:1C001364 7505                   `jne 1C00136B`

:1C001366 E9C7000000             `jmp 1C001432`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C001364(C)

|

:1C00136B 8B95A4FDFFFF           `mov edx, dword ptr [ebp+FFFFFDA4]`

:1C001371 52                     `push edx`

:1C001372 8B85F0FEFFFF           `mov eax, dword ptr [ebp+FFFFFEF0]`

:1C001378 50                     `push eax`

:1C001379 6A01                   `push 00000001`

:1C00137B 8B8DA8FDFFFF           `mov ecx, dword ptr [ebp+FFFFFDA8]`

:1C001381 51                     `push ecx`

\* Reference To: `MSVCRT.fread`, Ord:025Dh

                             |

:1C001382 FF155030001C           `Call dword ptr [1C003050]`

:1C001388 83C410                 `add esp, 00000010`

:1C00138B 85C0                   `test eax, eax`

:1C00138D 7448                   `je 1C0013D7`

:1C00138F 8B95F0FEFFFF           `mov edx, dword ptr [ebp+FFFFFEF0]`

:1C001395 83EA04                 `sub edx, 00000004`

:1C001398 52                     `push edx`

:1C001399 8B85A8FDFFFF           `mov eax, dword ptr [ebp+FFFFFDA8]`

:1C00139F 50                     `push eax`

:1C0013A0 E88FFEFFFF             `call 1C001234`

                             ==== >进行 CRC 计算！

:1C0013A5 8985ACFDFFFF           `mov dword ptr [ebp+FFFFFDAC], eax`

:1C0013AB 8B8DA8FDFFFF           `mov ecx, dword ptr [ebp+FFFFFDA8]`

:1C0013B1 038DF0FEFFFF           `add ecx, dword ptr [ebp+FFFFFEF0]`

:1C0013B7 8B51FC                 `mov edx, dword ptr [ecx-04]`

:1C0013BA 8995A0FDFFFF           `mov dword ptr [ebp+FFFFFDA0], edx`

:1C0013C0 8B85ACFDFFFF           `mov eax, dword ptr [ebp+FFFFFDAC]`

                             ==== >这里爆破可以去除对主程序的校验   :-)

:1C0013C6 33C9                   `xor ecx, ecx`

:1C0013C8 3B85A0FDFFFF           `cmp eax, dword ptr [ebp+FFFFFDA0]`

                             ==== >比较！

:1C0013CE 0F94C1                 `sete cl`

                             ==== >根据结果设置 `CL` 值！正确则 `CL=1`

:1C0013D1 898DF4FEFFFF           `mov dword ptr [ebp+FFFFFEF4], ecx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1C00138D(C), :1C001432(U), :1C001434(U)

|

:1C0013D7 83BDA4FDFFFF00         `cmp dword ptr [ebp+FFFFFDA4], 00000000`

:1C0013DE 7410                   `je 1C0013F0`

:1C0013E0 8B95A4FDFFFF           `mov edx, dword ptr [ebp+FFFFFDA4]`

:1C0013E6 52                     `push edx`

\* Reference To: `MSVCRT.fclose`, Ord:024Ch

                             |

:1C0013E7 FF155430001C           `Call dword ptr [1C003054]`

:1C0013ED 83C404                 `add esp, 00000004`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C0013DE(C)

|

:1C0013F0 837DFCFF               `cmp dword ptr [ebp-04], FFFFFFFF`

:1C0013F4 740A                   `je 1C001400`

:1C0013F6 8B45FC                 `mov eax, dword ptr [ebp-04]`

:1C0013F9 50                     `push eax`

\* Reference To: `KERNEL32.FindClose`, Ord:0090h

                             |

:1C0013FA FF151430001C           `Call dword ptr [1C003014]`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C0013F4(C)

|

:1C001400 83BDA8FDFFFF00         `cmp dword ptr [ebp+FFFFFDA8], 00000000`

:1C001407 7410                   `je 1C001419`

:1C001409 8B8DA8FDFFFF           `mov ecx, dword ptr [ebp+FFFFFDA8]`

:1C00140F 51                     `push ecx`

\* Reference To: `MSVCRT.free`, Ord:025Eh



:1C001410 `FF155830001C` `call dword ptr [1C003058]`

:1C001416 `83C404` `add esp, 00000004`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1C001407(C)

:1C001419 `83BDF4FEFFFF00` `cmp dword ptr [ebp+FFFFFEF4], 00000000`

:1C001420 `7508` `jne 1C00142A`

:1C001422 `6A01` `push 00000001`

* Reference To: `MSVCRT.exit`, Ord:0249h

:1C001424 `FF156030001C` `call dword ptr [1C003060]`

====>这里退出！不打声招呼就走 :-(

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1C001420(C)

:1C00142A `8B85F4FEFFFF` `mov eax, dword ptr [ebp+FFFFFEF4]`

:1C001430 `EB04` `jmp 1C001436`

————————————————————————

进入 CRC 计算 CALL：`1C0013A0` `call 1C001234`

* Referenced by a CALL at Addresses:

:1C0013A0 , :1C001972 , :1C001991

:1C001234 `55` `push ebp`

:1C001235 `8BEC` `mov ebp, esp`

:1C001237 `83EC08` `sub esp, 00000008`

:1C00123A `C745F8FFFFFFFF` `mov [ebp-08], FFFFFFFF`

:1C001241 `8B4508` `mov eax, dword ptr [ebp+08]`

:1C001244 `8945FC` `mov dword ptr [ebp-04], eax`

:1C001247 `EB09` `jmp 1C001252`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1C001286(U)

:1C001249 `8B4D0C` `mov ecx, dword ptr [ebp+0C]`

:1C00124C `83E901` `sub ecx, 00000001`

:1C00124F `894D0C` `mov dword ptr [ebp+0C], ecx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1C001247(U)

:1C001252 `837D0C00` `cmp dword ptr [ebp+0C], 00000000`

:1C001256 `7E30` `jle 1C001288`

:1C001258 `8B55F8` `mov edx, dword ptr [ebp-08]`

:1C00125B `C1EA08` `shr edx, 08`

:1C00125E `8B45F8` `mov eax, dword ptr [ebp-08]`

:1C001261 `25FF000000` `and eax, 000000FF`

:1C001266 `8B4DFC` `mov ecx, dword ptr [ebp-04]`

:1C001269 `0FBE09` `movsx ecx, byte ptr [ecx]`

:1C00126C `33C1` `xor eax, ecx`

:1C00126E `25FF000000` `and eax, 000000FF`

:1C001273 `3314851C40001C` `xor edx, dword ptr [4*eax+1C00401C]`

====>`[1C00401C]`内存中是一张 CRC32 数据表！:-)

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:-) `CRC-32 Table` 共有 256 个数 :-)

```
77073096 EE0E612C 990951BA 076DC419 706AF48F E963A535 9E6495A3 0EDB8832
79DCB8A4 E0D5E91E 97D2D988 09B64C2B 7EB17CBD E7B82D07 90BF1D91 00000000
6AB020F2 F3B97148 84BE41DE 1ADAD47D 6DDDE4EB F4D4B551 83D385C7 136C9856
646BA8C0 FD62F97A 8A65C9EC 14015C4F 63066CD9 FA0F3D63 8D080DF5 3B6E20C8
4C69105E D56041E4 A2677172 3C03E4D1 4B04D447 D20D85FD A50AB56B 35B5A8FA
42B2986C DBBBC9D6 ACBCF940 32D86CE3 45DF5C75 DCD60DCF ABD13D59 26D930AC
51DE003A C8D75180 BFD06116 21B4F4B5 56B3C423 CFBA9599 B8BDA50F 2802B89E
5F058808 C60CD9B2 B10BE924 2F6F7C87 58684C11 C1611DAB B6662D3D 76DC4190
01DB7106 98D220BC EFD5102A 71B18589 06B6B51F 9FBFE4A5 E8B8D433 7807C9A2
0F00F934 9609A88E E10E9818 7F6A0DBB 086D3D2D 91646C97 E6635C01 6B6B51F4
1C6C6162 856530D8 F262004E 6C0695ED 1B01A57B 8208F4C1 F50FC457 65B0D9C6
12B7E950 8BBEB8EA FCB9887C 62DD1DDF 15DA2D49 8CD37CF3 FBD44C65 4DB26158
3AB551CE A3BC0074 D4BB30E2 4ADFA541 3DD895D7 A4D1C46D D3D6F4FB 4369E96A
346ED9FC AD678846 DA60B8D0 44042D73 33031DE5 AA0A4C5F DD0D7CC9 5005713C
270241AA BE0B1010 C90C2086 5768B525 206F85B3 B966D409 CE61E49F 5EDEF90E
29D9C998 B0D09822 C7D7A8B4 59B33D17 2EB40D81 B7BD5C3B C0BA6CAD EDB88320
9ABFB3B6 03B6E20C 74B1D29A EAD54739 9DD277AF 04DB2615 73DC1683 E3630B12
94643B84 0D6D6A3E 7A6A5AA8 E40ECF0B 9309FF9D 0A00AE27 7D079EB1 F00F9344
8708A3D2 1E01F268 6906C2FE F762575D 806567CB 196C3671 6E6B06E7 FED41B76
89D32BE0 10DA7A5A 67DD4ACC F9B9DF6F 8EBEEFF9 17B7BE43 60B08ED5 D6D6A3E8
A1D1937E 38D8C2C4 4FDFF252 D1BB67F1 A6BC5767 3FB506DD 48B2364B D80D2BDA
AF0A1B4C 36034AF6 41047A60 DF60EFC3 A867DF55 316E8EEF 4669BE79 CB61B38C
BC66831A 256FD2A0 5268E236 CC0C7795 BB0B4703 220216B9 5505262F C5BA3BBE
B2BD0B28 2BB45A92 5CB36A04 C2D7FFA7 B5D0CF31 2CD99E8B 5BDEAE1D 9B64C2B0
EC63F226 756AA39C 026D930A 9C0906A9 EB0E363F 72076785 05005713 95BF4A82
E2B87A14 7BB12BAE 0CB61B38 92D28E9B E5D5BE0D 7CDCEFB7 0BDBDF21 86D3D2D4
F1D4E242 68DDB3F8 1FDA836E 81BE16CD F6B9265B 6FB077E1 18B74777 88085AE6
FF0F6A70 66063BCA 11010B5C 8F659EFF F862AE69 616BFFD3 166CCF45 A00AE278
D70DD2EE 4E048354 3903B3C2 A7672661 D06016F7 4969474D 3E6E77DB AED16A4A
D9D65ADC 40DF0B66 37D83BF0 A9BCAE53 DEBB9EC5 47B2CF7F 30B5FFE9 BDBDF21C
CABAC28A 53B39330 24B4A3A6 BAD03605 CDD70693 54DE5729 23D967BF B3667A2E
C4614AB8 5D681B02 2A6F2B94 B40BBE37 C30C8EA1 5A05DF1B 2D02EF8D 004C3E50
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:1C00127A `8955F8` `mov dword ptr [ebp-08], edx`

:1C00127D `8B55FC` `mov edx, dword ptr [ebp-04]`

:1C001280 `83C201` `add edx, 00000001`

:1C001283 `8955FC` `mov dword ptr [ebp-04], edx`

:1C001286 `EBC1` `jmp 1C001249`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:1C001256(C)

:1C001288 `8B45F8` `mov eax, dword ptr [ebp-08]`

:1C00128B `83F0FF` `xor eax, FFFFFFFF`

:1C00128E `8BE5` `mov esp, ebp`

:1C001290 `5D` `pop ebp`

:1C001291 `C20800` `ret 0008`

—————————————————————————————————

## 三、注册码验证、爆破

这部分是 `pLayAr[FCG]` 兄的成果！:-D 谢谢老兄指点！

程序的算法保护的挺好，共 36 位注册码，每框 9 位。程序对注册码有很多要求，并且分别在几个地方验证！参照 `pLayAr[FCG]` 兄的成果在特定的条件下进行爆破！

—————————————————————————————————

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00434A93(U)

:00434A9A `85C0` `test eax, eax`

:00434A9C `0F8410010000` `je 00434BB2`

:00434AA2 `8DBDD0030000` `lea edi, dword ptr [ebp+000003D0]`

:00434AA8 `83C9FF` `or ecx, FFFFFFFF`

:00434AAB `33C0` `xor eax, eax`

:00434AAD `F2` `repnz`

:00434AAE `AE` `scasb`

:00434AAF `F7D1` `not ecx`

:00434AB1 `49` `dec ecx`

:00434AB2 `83F909` `cmp ecx, 00000009`

====>9 位？

:00434AB5 `0F85F7000000` `jne 00434BB2`

:00434ABB `8DBD14050000` `lea edi, dword ptr [ebp+00000514]`

:00434AC1 `83C9FF` `or ecx, FFFFFFFF`

:00434AC4 `F2` `repnz`

:00434AC5 `AE` `scasb`

:00434AC6 `F7D1` `not ecx`

:00434AC8 `49` `dec ecx`

:00434AC9 `83F909` `cmp ecx, 00000009`

====>9 位？

:00434ACC `0F85E0000000` `jne 00434BB2`

:00434AD2 `8DBD58060000` `lea edi, dword ptr [ebp+00000658]`

:00434AD8 `83C9FF` `or ecx, FFFFFFFF`

:00434ADB `F2` `repnz`

:00434ADC `AE` `scasb`

:00434ADD `F7D1` `not ecx`

:00434ADF `49` `dec ecx`

:00434AE0 `83F909` `cmp ecx, 00000009`

====>9 位？

:00434AE3 `0F85C9000000` `jne 00434BB2`

:00434AE9 `8DBD9C070000` `lea edi, dword ptr [ebp+0000079C]`

:00434AEF `83C9FF` `or ecx, FFFFFFFF`

:00434AF2 `F2` `repnz`

:00434AF3 `AE` `scasb`



### 内存地址 00434AF4 起始的代码段

```
:00434AF4 F7D1                not ecx   
:00434AF6 49                  dec ecx   
:00434AF7 83F909              cmp ecx, 00000009   
                              ==== >9 位？   
:00434AFA 0F85B2000000        jne 00434BB2   
:00434B00 E829220900          call 004C6D2E   
:00434B05 8B7008              mov esi, dword ptr [eax+08]   
:00434B08 A1D4E85100          mov eax, dword ptr [0051E8D4]   
:00434B0D 8D889C070000        lea ecx, dword ptr [eax+0000079C]   
:00434B13 8D9058060000        lea edx, dword ptr [eax+00000658]   
:00434B19 51                  push ecx   
:00434B1A 8D8814050000        lea ecx, dword ptr [eax+00000514]   
:00434B20 52                  push edx   
:00434B21 05D0030000          add eax, 000003D0   
:00434B26 51                  push ecx   
:00434B27 50                  push eax   
:00434B28 8D542420            lea edx, dword ptr [esp+20]   
:00434B2C 68C0635000          push 005063C0   
:00434B31 52                  push edx   
:00434B32 E86E610600          call 0049ACA5   
:00434B37 8B0D78C65100        mov ecx, dword ptr [0051C678]   
:00434B3D 8B15D4E85100        mov edx, dword ptr [0051E8D4]   
:00434B43 B05A                mov al, 5A   
:00434B45 8974244D            mov dword ptr [esp+4D], esi   
:00434B49 8844244C            mov byte ptr [esp+4C], al   
:00434B4D 894C2451            mov dword ptr [esp+51], ecx   
:00434B51 8B4A1C              mov ecx, dword ptr [edx+1C]   
:00434B54 83C418              add esp, 00000018   
:00434B57 894C243D            mov dword ptr [esp+3D], ecx   
:00434B5B 8A4C2433            mov cl, byte ptr [esp+33]   
:00434B5F 3AC8                cmp cl, al   
                              ==== >最后 1 位是 Z？下面调用 hook.dll 验证！   
:00434B61 754F                jne 00434BB2   
:00434B63 68108B5000          push 00508B10   

* Reference To: KERNEL32.LoadLibraryA, Ord:0000h   
                              |   
:00434B68 FF15B8424D00        Call dword ptr [004D42B8]   
:00434B6E 8BF0                mov esi, eax   
:00434B70 85F6                test esi, esi   
:00434B72 743E                je 00434BB2   
:00434B74 8D542410            lea edx, dword ptr [esp+10]   
:00434B78 6A24                push 00000024   
:00434B7A 52                  push edx   
:00434B7B E880960000          call 0043E200   
:00434B80 83C408              add esp, 00000008   
:00434B83 8D442410            lea eax, dword ptr [esp+10]   
:00434B87 50                  push eax   
:00434B88 68F48A5000          push 00508AF4   
:00434B8D 56                  push esi   

* Reference To: KERNEL32.GetProcAddress, Ord:0000h   
                              |   
:00434B8E FF1594434D00        Call dword ptr [004D4394]   
:00434B94 50                  push eax   

* Reference To: USER32.EnumWindows, Ord:0000h   
                              |   
:00434B95 FF1514474D00        Call dword ptr [004D4714]   
:00434B9B 56                  push esi   

* Reference To: KERNEL32.FreeLibrary, Ord:0000h   
                              |   
:00434B9C FF15B4424D00        Call dword ptr [004D42B4]   
:00434BA2 B909000000          mov ecx, 00000009   
:00434BA7 8D742410            lea esi, dword ptr [esp+10]   
:00434BAB BFA0E85100          mov edi, 0051E8A0   
:00434BB0 F3                  repz   
:00434BB1 A5                  movsd   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00434A2D(C), :00434A3A(C), :00434A44(C), :00434A52(C), :00434A60(C)   

|:00434A9C(C), :00434AB5(C), :00434ACC(C), :00434AE3(C), :00434AFA(C)   

|:00434B61(C), :00434B72(C)   

|   

:00434BB2 8B4C244C            mov ecx, dword ptr [esp+4C]   
:00434BB6 8B542448            mov edx, dword ptr [esp+48]   
:00434BBA A1D8E85100          mov eax, dword ptr [0051E8D8]   
:00434BBF 53                  push ebx   
:00434BC0 51                  push ecx   
:00434BC1 52                  push edx   
:00434BC2 50                  push eax   

* Reference To: USER32.CallNextHookEx, Ord:0000h   
                              |   
:00434BC3 FF1580474D00        Call dword ptr [004D4780]   
:00434BC9 5F                  pop edi   
:00434BCA 5E                  pop esi   
:00434BCB 5D                  pop ebp   
:00434BCC 5B                  pop ebx   
:00434BCD 83C434              add esp, 00000034   
:00434BD0 C20C00              ret 000C   
```

————————————————————————

值得注意的是程序并不是仅有上面一处如此验证！下面也是：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040D8FC(C)   

…… …… 省 略 …… ……   

:0040D965 83F909              cmp ecx, 00000009   
:0040D968 0F854F010000        jne 0040DABD   

…… …… 省 略 …… ……   

:0040DA3C B959000000          mov ecx, 00000059   

…… …… 省 略 …… ……   
```

————————————————————————

### 进入 `hook.dll` 的验证：

```
* Referenced by a CALL at Addresses:   

|:1C0013A0   , :1C001972   , :1C001991    
    
                              ==== >这个 CALL 外面还有其它复杂的检测  :- (   
:1C001234 55                  push ebp   
:1C001235 8BEC                mov ebp, esp   
:1C001237 83EC08              sub esp, 00000008   
:1C00123A C745F8FFFFFFFF      mov [ebp\-08], FFFFFFFF   
:1C001241 8B4508              mov eax, dword ptr [ebp+08]   
:1C001244 8945FC              mov dword ptr [ebp\-04], eax   
:1C001247 EB09                jmp 1C001252   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:1C001286(U)   

|   

:1C001249 8B4D0C              mov ecx, dword ptr [ebp+0C]   
:1C00124C 83E901              sub ecx, 00000001   
:1C00124F 894D0C              mov dword ptr [ebp+0C], ecx   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:1C001247(U)   

|   

:1C001252 837D0C00            cmp dword ptr [ebp+0C], 00000000   
:1C001256 7E30                jle 1C001288   
:1C001258 8B55F8              mov edx, dword ptr [ebp\-08]   
:1C00125B C1EA08              shr edx, 08   
:1C00125E 8B45F8              mov eax, dword ptr [ebp\-08]   
:1C001261 25FF000000          and eax, 000000FF   
:1C001266 8B4DFC              mov ecx, dword ptr [ebp\-04]   
:1C001269 0FBE09              movsx ecx, byte ptr [ecx]   
:1C00126C 33C1                xor eax, ecx   
:1C00126E 25FF000000          and eax, 000000FF   
:1C001273 3314851C40001C      xor edx, dword ptr [4*eax+1C00401C]   
:1C00127A 8955F8              mov dword ptr [ebp\-08], edx   
:1C00127D 8B55FC              mov edx, dword ptr [ebp\-04]   
:1C001280 83C201              add edx, 00000001   
:1C001283 8955FC              mov dword ptr [ebp\-04], edx   
:1C001286 EBC1                jmp 1C001249   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:1C001256(C)   

|   

:1C001288 8B45F8              mov eax, dword ptr [ebp\-08]   
:1C00128B 83F0FF              xor eax, FFFFFFFF   
                              ==== >爆破点 ②   
:1C00128E 8BE5                mov esp, ebp   
:1C001290 5D                  pop ebp   
:1C001291 C20800              ret 0008   
```

—————————————————————————————————

### 【爆破】：

1. `0041BF35 E826820100` `call 00434160`
   改为：`9090909090` `NOP` 掉！

没办法呀，程序很狡猾，在 `hook.dll` 内进行校验 `SuperCap.exe`，在 `SuperCap.exe` 内进行校验 `hook.dll`。

省点劲，直接在外面改了。如果有问题则只好深入校验核心修改了。

2. `1C00128B 83F0FF` `xor eax, FFFFFFFF`
   改为：`33CO90` `xor eax, eax`

程序对注册码的要求挺高呀，:-) 这里爆破后还需要配合特定的注册码才能注册成功 :-
(




BTW：据 `fxyang` 兄测试在 `WIN2003` 下程序会异常出错，这也难怪，这个东东很久没升级了，发布当时还没有 `WIN2003`。

【注册信息保存】：

同目录下的 `SuperCapture.drv` 文件。

【整理】：

由于功力低微，未能完美解决。解除自校验爆破后还须输入以下注册码才能验证成功！

希望能有大侠研究出算法。或者能够得到一个可用的 `Key`……

姓名：`fly` （Random）
单位：`[OCN][FCG]` （Random）
注册码：`AAAAAAAAA-AAAAAAAAA-AAAAAAAAA-AAAAAAHHZ`

```
          ,     \_/ 
       /| \_.-~/          \_     ,        青春都一饷  :o 
      ( /~   /             ~-.|
      `\   \_/                ~ )          忍把浮名    :shock: 
  _-~~~-.)  )\__/;;,.          _ //'    
 /'\_,   --~    ~~-  ,;;\(  (.-~~~-.        换了破解轻狂   :wink: 
 \`~ _\ ,..-- (      ,;''/    ~--   /.\
  /~~//   / \`         ) /--..\ )\
  " \`" "      "  /~'     \~~
               "  "  "~' ""
             Cracked By 巢水工作坊——fly [OCN][FCG]
                      2003-09-29   9:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
