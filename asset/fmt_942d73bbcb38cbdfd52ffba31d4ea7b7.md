
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
