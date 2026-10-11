
# 轻松文本 2003 V6.13

> 原文：[`www.pediy.com/kssd/pediy06/pediy6134.htm`](https://www.pediy.com/kssd/pediy06/pediy6134.htm)

Ollydbg——轻松文本 2003 V6.13（VB）

下载页面：  [`www.skycn.com/soft/5977.html`](http://www.skycn.com/soft/5977.html)

软件大小：  2995 KB

软件语言：  简体中文

软件类别：  国产软件 / 共享版 / 文字处理

应用平台：  `Win9x/NT/2000/XP`

加入时间：  2003-06-09 16:24:39

下载次数：  28988

推荐等级：  *****

开 发 商：  [`www.ypall.com/qswb/`](http://www.ypall.com/qswb/)

【软件简介】：目前国内最好用文本编辑及阅读类软件之一，微软记事本的最佳替代品，独具非常丰富的工具，功能超强，界面友好，使用方便，绝对会给你耳目一新的感觉。主要功能：★支持对各种文本文件的打开、编辑，不受 64K 大小的限制，采用资源管理器界面，独创快捷键打开同一目录下所有文本文件和以选定文字为文件名快速保存文件功能，打开保存文件非常方便，支持历史文档记忆功能；★支持对文件的加解密，对网页和文本文件的双向转换，超强文本到网页转换功能可将普通文本迅速变成出色的网页，还可以压缩网页；★具有强大的编辑排版功能，支持多次撤消、重做和智能段落重排，可迅速删除文中所有空行、行首行尾空格、段内文字间空格和文中所有同类字符，迅速删除块内硬回车和替换空格为硬回车，以及文字上下标设置、英文字母大小写转换、全半角转换、文本左中右对齐方式和段落边界设置、行列互换、光标前后内容的快速选择、网址的选定链接和文本倒序翻转和定时自动存盘等功能；★具有英文拼写检查功能和强大的查找替换功能，支持对单词、句子、整行或整段文字的快速选定以及光标的快速定位，可快速高亮显示文中所有指定文本，中英文字数统计、字符出现次数统计，迅速查找文件位置或文本内容等功能；★具有自动制表功能，可迅速在文中插入当前日期时间、分隔线、连续个选定字符、特殊符号和项目符号、系统安装字体名称、输入法名称、大写金额、常用短语、其它文本文件、字符画、图形文字、图像文件、当前文件路径名、当前目录下所有文本名和当前盘内所有同类文件名；★具有英文朗读、`Big5` 码与 `Gb` 码相互转换、文本排序、邮件发送、网页预览、多次剪贴板、`ASCII` 码查询、音频视频文件后台循环播放、定时提醒、成语查询、日历查询、百年日历制作、批量改名、俄罗斯方块等许多非常独特的工具或功能，还可在软件中运行各种外部可执行程序；★可自动监视剪贴板，并可选择剪贴板内容粘贴到当前文件还是存成新文件，具有 `NetAnts` 那种拖放篮，可让你轻松保存文本资料；★具有像素极文字自动滚屏功能和自动翻页功能，阅读文本很轻松；★具有顶层显示功能，支持鼠标拖放打开文件、复制移动文本图形，可任意调整编辑区前背景颜色等，功能多多，是你电脑中一款不可多得的必备软件！

【软件限制】：`NAG`、功能限制

【作者声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`Ollydbg1.09`、`PEiD`、`AspackDie`、`W32Dasm` 9.0 白金版

—————————————————————————————————

【过   程】：

其实这个东东数 10 天前就做了，因为太忙，今天才把笔记整理出来，呵呵，作者也快升级了吧？

顺便看了一下同门的《英语音标大师 V1.02》，算法是一样的，就没必要写了。^O^ ^O^

`easypad.exe` 是 `ASPack` 2.12 壳，用 `AspackDie` 脱之。`169K->732K`。 `VB` 编写。

这个东东不算难，只是有些方面不好掌握。              ~Q~ ^Q^ ^v^ ^v^

序列号：`FLYN649065455613`

试炼码：`fly-12345678-fly[OCN][FCG]-E`

—————————————————————————————————

```
* Reference To: MSVBVM60.rtcInputBox, Ord:0254h

:004620D2 FF15FC104000             Call dword ptr [004010FC]

:004620D8 8BD0                    mov edx, eax

                                  ====>EDX=fly-12345678-fly[OCN][FCG]-E  试炼码

:004620DA 8D4DA8                  lea ecx, dword ptr [ebp-58]

:004620DD FFD6                    call esi

:004620DF 8BD0                    mov edx, eax

:004620E1 8B8D78FEFFFF            mov ecx, dword ptr [ebp+FFFFFE78]

* Reference To: MSVBVM60.__vbaStrCopy, Ord:0000h

                                  |

:004620E7 FF15D4124000            Call dword ptr [004012D4]

:004620ED 8D55A4                  lea edx, dword ptr [ebp-5C]

:004620F0 52                      push edx

:004620F1 8D45A8                  lea eax, dword ptr [ebp-58]

:004620F4 50                      push eax

:004620F5 8D4DAC                  lea ecx, dword ptr [ebp-54]

:004620F8 51                      push ecx

:004620F9 8D55B0                  lea edx, dword ptr [ebp-50]

:004620FC 52                      push edx

:004620FD 8D45B4                  lea eax, dword ptr [ebp-4C]

:00462100 50                      push eax

:00462101 8D4DB8                  lea ecx, dword ptr [ebp-48]

:00462104 51                      push ecx

:00462105 8D55BC                  lea edx, dword ptr [ebp-44]

:00462108 52                      push edx

:00462109 8D45C0                  lea eax, dword ptr [ebp-40]

:0046210C 50                      push eax

:0046210D 8D4DC4                  lea ecx, dword ptr [ebp-3C]

:00462110 51                      push ecx

:00462111 8D55C8                  lea edx, dword ptr [ebp-38]

:00462114 52                      push edx

:00462115 8D45CC                  lea eax, dword ptr [ebp-34]

:00462118 50                      push eax

:00462119 6A0B                    push 0000000B

* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h

                                  |

:0046211B FF15E4124000            Call dword ptr [004012E4]

:00462121 8D4D9C                  lea ecx, dword ptr [ebp-64]

:00462124 51                      push ecx

:00462125 8D55A0                  lea edx, dword ptr [ebp-60]

:00462128 52                      push edx

:00462129 6A02                    push 00000002

* Reference To: MSVBVM60.__vbaFreeObjList, Ord:0000h

                                  |

:0046212B FF1558104000            Call dword ptr [00401058]

:00462131 8D852CFFFFFF            lea eax, dword ptr [ebp+FFFFFF2C]

:00462137 50                      push eax

:00462138 8D8D3CFFFFFF            lea ecx, dword ptr [ebp+FFFFFF3C]

:0046213E 51                      push ecx

:0046213F 8D954CFFFFFF            lea edx, dword ptr [ebp+FFFFFF4C]

:00462145 52                      push edx

:00462146 8D855CFFFFFF            lea eax, dword ptr [ebp+FFFFFF5C]

:0046214C 50                      push eax

:0046214D 8D8D6CFFFFFF            lea ecx, dword ptr [ebp+FFFFFF6C]

:00462153 51                      push ecx

:00462154 8D957CFFFFFF            lea edx, dword ptr [ebp+FFFFFF7C]

:0046215A 52                      push edx

:0046215B 8D458C                  lea eax, dword ptr [ebp-74]

:0046215E 50                      push eax

:0046215F 6A07                    push 00000007

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

                                  |

:00462161 FF1544104000            Call dword ptr [00401044]

:00462167 83C45C                  add esp, 0000005C

:0046216A 8B0B                    mov ecx, dword ptr [ebx]

:0046216C 8D95C8FEFFFF            lea edx, dword ptr [ebp+FFFFFEC8]

:00462172 52                      push edx

:00462173 8B8578FEFFFF            mov eax, dword ptr [ebp+FFFFFE78]

:00462179 50                      push eax

:0046217A 53                      push ebx

:0046217B FF9128070000            call dword ptr [ecx+00000728]

                                  ====>关键 CALL！进入！

:00462181 85C0                    test eax, eax

:00462183 7D12                    jge 00462197

:00462185 6828070000              push 00000728

:0046218A 688C574200              push 0042578C

:0046218F 53                      push ebx

:00462190 50                      push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h

                                  |

:00462191 FF15A4104000            Call dword ptr [004010A4]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00462183(C)
|

:00462197 6683BDC8FEFFFF00        cmp word ptr [ebp+FFFFFEC8], 0000

:0046219F 0F84C3030000            je 00462568

                                  ====>跳则 OVER！

:004621A5 8D4D8C                  lea ecx, dword ptr [ebp-74]

:004621A8 51                      push ecx

* Reference To: MSVBVM60.rtcGetDateVar, Ord:0262h

                                  |

:004621A9 FF1524134000            Call dword ptr [00401324]

:004621AF 6A00                    push 00000000

:004621B1 8D558C                  lea edx, dword ptr [ebp-74]

:004621B4 52                      push edx

:004621B5 8D857CFFFFFF            lea eax, dword ptr [ebp+FFFFFF7C]

:004621BB 50                      push eax

* Reference To: MSVBVM60.rtcGetDayOfWeek, Ord:0228h

                                  |

:004621BC FF1588104000            Call dword ptr [00401088]

:004621C2 8D8D7CFFFFFF            lea ecx, dword ptr [ebp+FFFFFF7C]

:004621C8 51                      push ecx

* Reference To: MSVBVM60.rtcRandomNext, Ord:0251h

                                  |

:004621C9 FF15D4104000            Call dword ptr [004010D4]

:004621CF D80D9C284000            fmul dword ptr [0040289C]

:004621D5 DFE0                    fstsw ax

:004621D7 A80D                    test al, 0D

:004621D9 0F856D040000            jne 0046264C

* Reference To: MSVBVM60.__vbaFpI4, Ord:0000h

                                  |
```



```asm
:004621DF FF1560134000             Call dword ptr [00401360]
:004621E5 89437C                 mov dword ptr [ebx+7C], eax
:004621E8 8D957CFFFFFF           lea edx, dword ptr [ebp+FFFFFF7C]
:004621EE 52                     push edx
:004621EF 8D458C                 lea eax, dword ptr [ebp-74]
:004621F2 50                     push eax
:004621F3 6A02                   push 00000002
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
:004621F5 FF1544104000           Call dword ptr [00401044]
:004621FB 83C40C                 add esp, 000000C
:004621FE B904000280             mov ecx, 80020004
:00462203 898D34FFFFFF           mov dword ptr [ebp+FFFFFF34], ecx
:00462209 B80A000000             mov eax, 0000000A
:0046220E 89852CFFFFFF           mov dword ptr [ebp+FFFFFF2C], eax
:00462214 898D44FFFFFF           mov dword ptr [ebp+FFFFFF44], ecx
:0046221A 89853CFFFFFF           mov dword ptr [ebp+FFFFFF3C], eax
:00462220 898D54FFFFFF           mov dword ptr [ebp+FFFFFF54], ecx
:00462226 89854CFFFFFF           mov dword ptr [ebp+FFFFFF4C], eax
:0046222C 898D64FFFFFF           mov dword ptr [ebp+FFFFFF64], ecx
:00462232 89855CFFFFFF           mov dword ptr [ebp+FFFFFF5C], eax
:00462238 C78514FFFFFFD0654200   mov dword ptr [ebp+FFFFFF14], 004265D0
:00462242 C7850CFFFFFF08000000   mov dword ptr [ebp+FFFFFF0C], 00000008
:0046224C 8D950CFFFFFF           lea edx, dword ptr [ebp+FFFFFF0C]
:00462252 8D8D6CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF6C]
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
:00462258 FF1538134000           Call dword ptr [00401338]
* Possible StringData Ref from Code Obj ->"搹 eQnx x"
:0046225E C78524FFFFFF009A4200   mov dword ptr [ebp+FFFFFF24], 00429A00
:00462268 C7851CFFFFFF08000000   mov dword ptr [ebp+FFFFFF1C], 00000008
:00462272 8D951CFFFFFF           lea edx, dword ptr [ebp+FFFFFF1C]
:00462278 8D8D7CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF7C]
* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
:0046227E FF1538134000           Call dword ptr [00401338]
* Possible StringData Ref from Code Obj ->"a"尐`剉/ec m`淯╜剉 b 烺 (W 孾 b 鑜孮 MR鲖峇"
:00462284 6894994200             push 00429994
:00462289 8B4B7C                 mov ecx, dword ptr [ebx+7C]
:0046228C 51                     push ecx
* Reference To: MSVBVM60.__vbaStrI4, Ord:0000h
:0046228D FF1520104000           Call dword ptr [00401020]
:00462293 8BD0                   mov edx, eax
:00462295 8D4DCC                 lea ecx, dword ptr [ebp-34]
:00462298 FFD6                   call esi
:0046229A 50                     push eax
:0046229B FFD7                   call edi
:0046229D 894594                 mov dword ptr [ebp-6C], eax
:004622A0 C7458C08000000         mov [ebp-74], 00000008
:004622A7 8D952CFFFFFF           lea edx, dword ptr [ebp+FFFFFF2C]
:004622AD 52                     push edx
:004622AE 8D853CFFFFFF           lea eax, dword ptr [ebp+FFFFFF3C]
:004622B4 50                     push eax
:004622B5 8D8D4CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF4C]
:004622BB 51                     push ecx
:004622BC 8D955CFFFFFF           lea edx, dword ptr [ebp+FFFFFF5C]
:004622C2 52                     push edx
:004622C3 8D856CFFFFFF           lea eax, dword ptr [ebp+FFFFFF6C]
:004622C9 50                     push eax
:004622CA 8D8D7CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF7C]
:004622D0 51                     push ecx
:004622D1 8D558C                 lea edx, dword ptr [ebp-74]
:004622D4 52                     push edx
* Reference To: MSVBVM60.rtcInputBox, Ord:0254h
:004622D5 FF15FC104000           Call dword ptr [004010FC]
====>恭喜完成！输入确认号码！7055
:004622DB 8BD0                   mov edx, eax
====>EDX=7055
:004622DD 8D4DC8                 lea ecx, dword ptr [ebp-38]
:004622E0 FFD6                   call esi
:004622E2 50                     push eax
* Reference To: MSVBVM60.__vbaR8Str, Ord:0000h
:004622E3 FF15C0124000           Call dword ptr [004012C0]
:004622E9 DB437C                 fild dword ptr [ebx+7C]
:004622EC DD9D70FEFFFF           fstp qword ptr [ebp+FFFFFE70]
:004622F2 DC9D70FEFFFF           fcomp qword ptr [ebp+FFFFFE70]
====>比较 确认号码 是否是 7055？
:004622F8 DFE0                   fstsw ax
:004622FA F6C440                 test ah, 40
:004622FD 7407                   je 00462306
:004622FF B801000000             mov eax, 00000001
:00462304 EB02                   jmp 00462308
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004622FD(C)
:00462306 33C0                   xor eax, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00462304(U)
:00462308 F7D8                   neg eax
:0046230A 668BF0                 mov si, ax
:0046230D 8D45C8                 lea eax, dword ptr [ebp-38]
:00462310 50                     push eax
:00462311 8D4DCC                 lea ecx, dword ptr [ebp-34]
:00462314 51                     push ecx
:00462315 6A02                   push 00000002
* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h
:00462317 FF15E4124000           Call dword ptr [004012E4]
:0046231D 8D952CFFFFFF           lea edx, dword ptr [ebp+FFFFFF2C]
:00462323 52                     push edx
:00462324 8D853CFFFFFF           lea eax, dword ptr [ebp+FFFFFF3C]
:0046232A 50                     push eax
:0046232B 8D8D4CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF4C]
:00462331 51                     push ecx
:00462332 8D955CFFFFFF           lea edx, dword ptr [ebp+FFFFFF5C]
:00462338 52                     push edx
:00462339 8D856CFFFFFF           lea eax, dword ptr [ebp+FFFFFF6C]
:0046233F 50                     push eax
:00462340 8D8D7CFFFFFF           lea ecx, dword ptr [ebp+FFFFFF7C]
:00462346 51                     push ecx
:00462347 8D558C                 lea edx, dword ptr [ebp-74]
:0046234A 52                     push edx
:0046234B 6A07                   push 00000007
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
:0046234D FF1544104000           Call dword ptr [00401044]
:00462353 83C42C                 add esp, 0000002C
:00462356 6685F6                 test si, si
:00462359 0F8409020000           je 00462568
:0046235F 8B8578FEFFFF           mov eax, dword ptr [ebp+FFFFFE78]
:00462365 8B08                   mov ecx, dword ptr [eax]
:00462367 51                     push ecx
* Possible StringData Ref from Code Obj ->"rregnumber"
:00462368 6870684200             push 00426870
* Possible StringData Ref from Code Obj ->"rregist"
:0046236D 685C684200             push 0042685C
* Possible StringData Ref from Code Obj ->"eeasypad"
:00462372 68E8634200             push 004263E8
* Reference To: MSVBVM60.rtcSaveSetting, Ord:02B2h
:00462377 FF150C104000           Call dword ptr [0040100C]
====>保存注册信息！
```



### 汇编指令片段

```
:0046237D E9E6010000              jmp 00462568
```

—————————————————————————————————

进入关键 `CALL`：`:0046217B` `call dword ptr [ecx+00000728]`

…… ……省略…… ……

```
:004724A8 FFD3                  call ebx
:004724AA 50                    push eax

* Possible StringData Ref from Code Obj ->"CC:\\"
                         |
:004724AB 68A4974200             push 004297A4
:004724B0 8D45CC               lea eax, dword ptr [ebp-34]
:004724B3 50                    push eax
:004724B4 FFD3                  call ebx
:004724B6 50                    push eax
:004724B7 E8EC30FBFF            call 004255A8

* Reference To: MSVBVM60.__vbaSetSystemError, Ord:0000h
                         |
:004724BC FF1598104000           Call dword ptr [00401098]
:004724C2 8B4DC8               mov ecx, dword ptr [ebp-38]

* Reference To: MSVBVM60.__vbaStrToUnicode, Ord:0000h
                         |
:004724C5 8B1D38124000           mov ebx, dword ptr [00401238]
:004724CB 51                    push ecx
:004724CC 8D55C4               lea edx, dword ptr [ebp-3C]
:004724CF 52                    push edx
:004724D0 FFD3                  call ebx
:004724D2 50                    push eax
:004724D3 8B45DC               mov eax, dword ptr [ebp-24]
:004724D6 50                    push eax
:004724D7 57                    push edi

* Reference To: MSVBVM60.__vbaLsetFixstr, Ord:0000h
                         |
:004724D8 FF1594104000           Call dword ptr [00401094]
:004724DE 8B4DC0               mov ecx, dword ptr [ebp-40]
:004724E1 51                    push ecx
:004724E2 8D55BC               lea edx, dword ptr [ebp-44]
:004724E5 52                    push edx
:004724E6 FFD3                  call ebx
:004724E8 50                    push eax
:004724E9 8B45D8               mov eax, dword ptr [ebp-28]
:004724EC 50                    push eax
:004724ED 57                    push edi

* Reference To: MSVBVM60.__vbaLsetFixstr, Ord:0000h
                         |
:004724EE FF1594104000           Call dword ptr [00401094]
:004724F4 8D4DBC               lea ecx, dword ptr [ebp-44]
:004724F7 51                    push ecx
:004724F8 8D55C0               lea edx, dword ptr [ebp-40]
:004724FB 52                    push edx
:004724FC 8D45C4               lea eax, dword ptr [ebp-3C]
:004724FF 50                    push eax
:00472500 8D4DC8               lea ecx, dword ptr [ebp-38]
:00472503 51                    push ecx
:00472504 8D55CC               lea edx, dword ptr [ebp-34]
:00472507 52                    push edx
:00472508 6A05                  push 00000005

* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h
                         |
:0047250A FF15E4124000           Call dword ptr [004012E4]
:00472510 8B5D0C               mov ebx, dword ptr [ebp+0C]
:00472513 8B03                 mov eax, dword ptr [ebx]
                         ==== >EAX=fly-12345678-fly[OCN][FCG]-E  试炼码
:00472515 83C418               add esp, 00000018
:00472518 6A01                 push 00000001
:0047251A 6AFF                 push FFFFFFFF
:0047251C 6A01                 push 00000001
:0047251E 68D0654200             push 004265D0
:00472523 68CC754200             push 004275CC
:00472528 50                    push eax

* Reference To: MSVBVM60.rtcReplace, Ord:02C8h
                         |
:00472529 FF152C124000           Call dword ptr [0040122C]
                         ==== >去除试炼码中的-
:0047252F 8BD0                 mov edx, eax
                         ==== >EDX=fly12345678fly[OCN][FCG]E
:00472531 8D4DD4               lea ecx, dword ptr [ebp-2C]

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
                         |
:00472534 FF1578134000           Call dword ptr [00401378]
:0047253A 8B0B                 mov ecx, dword ptr [ebx]

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
                         |
:0047253C 8B1D34104000           mov ebx, dword ptr [00401034]
:00472542 51                    push ecx
                         ==== >ECX=fly-12345678-fly[OCN][FCG]-E
:00472543 FFD3                  call ebx
                         ==== >取 fly-12345678-fly[OCN][FCG]-E 的长度
:00472545 8BD0                 mov edx, eax
                         ==== >EDX=1C
:00472547 8B45D4               mov eax, dword ptr [ebp-2C]
:0047254A 50                    push eax
                         ==== >EAX=fly12345678fly[OCN][FCG]E
:0047254B 899528FFFFFF           mov dword ptr [ebp+FFFFFF28], edx
                         ==== >[ebp+FFFFFF28]=EDX=1C
:00472551 FFD3                  call ebx
                         ==== >取 fly12345678fly[OCN][FCG]E 的长度=19
:00472553 8B8D28FFFFFF           mov ecx, dword ptr [ebp+FFFFFF28]
                         ==== >ECX=1C
:00472559 8B55D4               mov edx, dword ptr [ebp-2C]
:0047255C 33DB                 xor ebx, ebx
:0047255E 3BC1                 cmp eax, ecx
                         ==== >比较 2 者长度是否相同？既检测试炼码中是否有-
:00472560 52                    push edx
:00472561 0F9DC3               setnl bl
                         ==== >设置 BL 值！有-则长度不同则 BL=0

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
                         |
:00472564 FF1534104000           Call dword ptr [00401034]
                         ==== >取 fly12345678fly[OCN][FCG]E 的长度=19
:0047256A 33C9                 xor ecx, ecx
:0047256C 83F819               cmp eax, 00000019
                         ==== >去除试炼码中的-后是否是 25 位？
:0047256F 0F9CC1               setl cl
                         ==== >设置 CL 值！是 25 位则 CL=0
:00472572 0BD9                 or ebx, ecx
:00472574 0F850C010000           jne 00472686
                         ==== >如果上面 2 个条件都符合则此处不跳！
                         ==== >若此处跳就直接 OVER 了！爆破点①！
:0047257A 8B55D4               mov edx, dword ptr [ebp-2C]
                         ==== >EDX=fly12345678fly[OCN][FCG]E
:0047257D A110804A00             mov eax, dword ptr [004A8010]
                         ==== >EAX=211C1E09          C 盘的硬盘序列号
:00472582 8D4DA4               lea ecx, dword ptr [ebp-5C]
:00472585 89955CFFFFFF           mov dword ptr [ebp+FFFFFF5C], edx
:0047258B 2DCF337B00             sub eax, 007B33CF
                         ==== >EAX=211C1E09 - 007B33CF=20A0EA3A
:00472590 51                    push ecx
:00472591 8D5594               lea edx, dword ptr [ebp-6C]
:00472594 0F8020050000           jo 00472ABA
:0047259A 52                    push edx
:0047259B C78554FFFFFF08000000   mov dword ptr [ebp+FFFFFF54], 00000008
:004725A5 8945AC               mov dword ptr [ebp-54], eax
:004725A8 C745A403000000        mov [ebp-5C], 00000003

* Reference To: MSVBVM60.rtcHexVarFromVar, Ord:023Dh
                         |
:004725AF FF15D8124000           Call dword ptr [004012D8]
:004725B5 6A01                 push 00000001
:004725B7 8D8554FFFFFF           lea eax, dword ptr [ebp+FFFFFF54]
:004725BD 50                    push eax
:004725BE 8D4D94               lea ecx, dword ptr [ebp-6C]
:004725C1 51                    push ecx
:004725C2 6A01                 push 00000001
:004725C4 8D5584               lea edx, dword ptr [ebp-7C]
```



```
:004725C7 52 
push edx

:004725C8 89BD4CFFFFFF
mov dword ptr [ebp+FFFFFF4C], edi

:004725CE C78544FFFFFF02800000
mov dword ptr [ebp+FFFFFF44], 00008002

* Reference To: MSVBVM60.__vbaInStrVar, Ord:0000h
|

:004725D8 FF1570124000
Call dword ptr [00401270]
====>比较 CALL！进入！有点特别呀 ^O^ ^O^

:004725DE 50
push eax

:004725DF 8D8544FFFFFF
lea eax, dword ptr [ebp+FFFFFF44]

:004725E5 50
push eax

* Reference To: MSVBVM60.__vbaVarTstGt, Ord:0000h
|

:004725E6 FF1504104000
Call dword ptr [00401004]

:004725EC 8D4D84
lea ecx, dword ptr [ebp-7C]

:004725EF 51
push ecx

:004725F0 8D5594
lea edx, dword ptr [ebp-6C]

:004725F3 668BD8
mov bx, ax
====>爆破点②！ ^O^ ^O^

:004725F6 52
push edx

:004725F7 8D45A4
lea eax, dword ptr [ebp-5C]

:004725FA 50
push eax

:004725FB 6A03
push 00000003

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
|

:004725FD FF1544104000
Call dword ptr [00401044]

:00472603 83C410
add esp, 00000010

:00472606 663BDF
cmp bx, di

:00472609 0F84E3000000
je 004726F2
====>跳则 OVER！

:0047260F 8B0E
mov ecx, dword ptr [esi]

:00472611 56
push esi

:00472612 C745D0FFFFFFFF
mov [ebp-30], FFFFFFFF

:00472619 FF912C060000
call dword ptr [ecx+0000062C]

:0047261F 50
push eax

:00472620 8D55B8
lea edx, dword ptr [ebp-48]

:00472623 52
push edx

* Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
|

:00472624 FF15F4104000
Call dword ptr [004010F4]

:0047262A 8D4DB4
lea ecx, dword ptr [ebp-4C]

:0047262D 51
push ecx

:0047262E 8BF0
mov esi, eax

:00472630 8B06
mov eax, dword ptr [esi]

:00472632 6A03
push 00000003

:00472634 56
push esi

:00472635 FF5040
call [eax+40]

:00472638 DBE2
fclex

:0047263A 3BC7
cmp eax, edi

:0047263C 7D0F
jge 0047264D

:0047263E 6A40
push 00000040

:00472640 68BC654200
push 004265BC

:00472645 56
push esi

:00472646 50
push eax
```

### 进入比较 CALL

`004725D8` `Call` `dword` `ptr` `[00401270]`

再进入：`7347A9CC` `Call` `MSVBVM60.__vbaInStr`

```
733A45A5  55
push ebp

733A45A6  8BEC
mov ebp,esp

733A45A8  81EC BC000000
sub esp,0BC

733A45AE  8365 EC 00
and dword ptr ss:[ebp-14],0

733A45B2  53
push ebx

733A45B3  56
push esi

733A45B4  8B75 0C
mov esi,dword ptr ss:[ebp+C]
====>ESI=20A0EA3A

733A45B7  57
push edi

733A45B8  8B7D 10
mov edi,dword ptr ss:[ebp+10]
====>EDI=fly12345678fly[OCN][FCG]E

733A45BB  8D85 44FFFFFF
lea eax,dword ptr ss:[ebp-BC]

733A45C1  897D F8
mov dword ptr ss:[ebp-8],edi

733A45C4  85FF
test edi,edi

733A45C6  8945 F4
mov dword ptr ss:[ebp-C],eax

733A45C9  8975 FC
mov dword ptr ss:[ebp-4],esi

733A45CC  0F84 09350300
je MSVBVM60.733D7ADB

733A45D2  8B47 FC
mov eax,dword ptr ds:[edi-4]

733A45D5  D1E8
shr eax,1
====>取 fly12345678fly[OCN][FCG]E 长度

733A45D7  8945 E4
mov dword ptr ss:[ebp-1C],eax
====>EAX=19

733A45DA  0F84 FB340300
je MSVBVM60.733D7ADB

733A45E0  85F6
test esi,esi

733A45E2  0F84 EB340300
je MSVBVM60.733D7AD3

733A45E8  8B46 FC
mov eax,dword ptr ds:[esi-4]

733A45EB  D1E8
shr eax,1
====>取 20A0EA3A 的长度

733A45ED  8945 E4
mov dword ptr ss:[ebp-1C],eax
====>EAX=8

733A45F0  0F84 DD340300
je MSVBVM60.733D7AD3

733A45F6  8B45 14
mov eax,dword ptr ss:[ebp+14]

733A45F9  8D58 FF
lea ebx,dword ptr ds:[eax-1]

733A45FC  85DB
test ebx,ebx

733A45FE  0F8C 33330300
jl MSVBVM60.733D7937

733A4604  81FB FFFFFF3F
cmp ebx,3FFFFFFF

733A460A  0F87 27330300
ja MSVBVM60.733D7937

733A4610  8B45 08
mov eax,dword ptr ss:[ebp+8]

733A4613  895D E8
mov dword ptr ss:[ebp-18],ebx

733A4616  85C0
test eax,eax

733A4618  0F85 20330300
jnz MSVBVM60.733D793E
====>跳下去，转变大写字母为小写字母！

733A461E  8B45 F8
mov eax,dword ptr ss:[ebp-8]
====>转变完了再跳回来！

733A4621  85C0
test eax,eax
====>EAX=fly12345678fly[ocn][fcg]e

733A4623  0F84 06340300
je MSVBVM60.733D7A2F

733A4629  8B48 FC
mov ecx,dword ptr ds:[eax-4]

733A462C  D1E9
shr ecx,1

733A462E  85F6
test esi,esi

733A4630  0F84 00340300
je MSVBVM60.733D7A36

733A4636  8B56 FC
mov edx,dword ptr ds:[esi-4]

733A4639  D1EA
shr edx,1

733A463B  8B7D E8
mov edi,dword ptr ss:[ebp-18]

733A463E  3BF9
cmp edi,ecx

733A4640  73 74
jnb short MSVBVM60.733A46B6

733A4642  85D2
test edx,edx

733A4644  0F84 F3330300
je MSVBVM60.733D7A3D

733A464A  3BD1
cmp edx,ecx

733A464C  0F87 F6330300
ja MSVBVM60.733D7A48

733A4652  8D0478
lea eax,dword ptr ds:[eax+edi*2]

733A4655  8B7D F8
mov edi,dword ptr ss:[ebp-8]

733A4658  2BCA
sub ecx,edx

733A465A  8D5C4F 02
lea ebx,dword ptr ds:[edi+ecx*2+2]

733A465E  0FB70E
movzx ecx,word ptr ds:[esi]

733A4661  894D 14
mov dword ptr ss:[ebp+14],ecx

733A4664  8D4C12 FE
lea ecx,dword ptr ds:[edx+edx-2]

733A4668  3BC3
cmp eax,ebx

733A466A  894D E4
mov dword ptr ss:[ebp-1C],ecx

733A466D  73 47
jnb short MSVBVM60.733A46B6

733A466F  8BCB
mov ecx,ebx

733A4671  2BC8
sub ecx,eax

733A4673  D1F9
sar ecx,1

733A4675  51
push ecx

733A4676  FF75 14
push dword ptr ss:[ebp+14]

733A4679  50
push eax

733A467A  E8 46000000
call MSVBVM60.733A46C5
====>循环取试炼码，比较第“1”位是否是 2？

733A467F  85C0
test eax,eax

733A4681  74 33
je short MSVBVM60.733A46B6

733A4683  8B4D E4
mov ecx,dword ptr ss:[ebp-1C]

733A4686  40
inc eax

733A4687  40
inc eax

733A4688  8D7E 02
lea edi,dword ptr ds:[esi+2]
```



## 733A468B 反汇编代码

```
733A468B     8BF0                mov esi,eax  
733A468D     33D2                xor edx,edx  
733A468F     F3:A6               repe cmps byte ptr es:[edi],byte ptr ds:[esi]  
                                  ==== > 依次比较剩下的 7 位是否是 0a0ea3a 

733A4691     75 1A               jnz short MSVBVM60.733A46AD  
733A4693     8BC8                mov ecx,eax  
733A4695     2B4D F8             sub ecx,dword ptr ss:[ebp\-8]  
733A4698     D1F9                sar ecx,1  
733A469A     837D 08 00          cmp dword ptr ss:[ebp+8],0  
733A469E     0F85 AD330300       jnz MSVBVM60.733D7A51  
733A46A4     8BC1                mov eax,ecx  
733A46A6     5F                  pop edi  
733A46A7     5E                  pop esi  
733A46A8     5B                  pop ebx  
733A46A9     C9                  leave  
733A46AA     C2 1000             retn 10 
733A46AD     3BC3                cmp eax,ebx  
733A46AF     73 05               jnb short MSVBVM60.733A46B6  
733A46B1     8B75 FC             mov esi,dword ptr ss:[ebp\-4]  
733A46B4    ^ EB B9             jmp short MSVBVM60.733A466F  
                                  ==== > 循环比较！  
                                  ==== > 其实就是循环比较试炼码中是否有 8 位是 20a0ea3a 
```

## 跳转至 733A4618

由 `733A4618` 跳到这里：

```
733D793E     83F8 01             cmp eax,1  
733D7941     75 3D               jnz short MSVBVM60.733D7980  
733D7943     E8 818DFCFF         call MSVBVM60.733A06C9  
733D7948     8945 08             mov dword ptr ss:[ebp+8],eax  
733D794B     8B45 08             mov eax,dword ptr ss:[ebp+8]  
733D794E     3B05 2C1E4A73       cmp eax,dword ptr ds:[734A1E2C]  
733D7954     74 06               je short MSVBVM60.733D795C  
733D7956     50                  push eax  
733D7957     E8 D83B0A00         call MSVBVM60.7347B534  
733D795C     8B45 F4             mov eax,dword ptr ss:[ebp\-C]  
733D795F     33F6                xor esi,esi  
733D7961     56                  push esi  
733D7962     56                  push esi  
733D7963     56                  push esi  
733D7964     C700 FEFFFFFF       mov dword ptr ds:[eax],-2  
733D796A     FF75 0C             push dword ptr ss:[ebp+C]  
733D796D     E8 CA3D0A00         call MSVBVM60.7347B73C  
                                  ==== > 将 20A0EA3A 中的大写字母转为小写字母！ 

733D7972     3BC6                cmp eax,esi  
                                  ==== > EAX=20a0ea3a 
733D7974     8945 FC             mov dword ptr ss:[ebp\-4],eax  
733D7977     75 1D               jnz short MSVBVM60.733D7996  
733D7979     6A 07               push 7  
733D797B     E8 D9D9FDFF         call MSVBVM60.733B5359  
733D7980     83F8 02             cmp eax,2  
733D7983     74 0A               je short MSVBVM60.733D798F  
733D7985     50                  push eax  
733D7986     E8 69830900         call MSVBVM60.7346FCF4  
733D798B     85C0                test eax,eax  
733D798D    ^ 75 BC             jnz short MSVBVM60.733D794B  
733D798F     6A 05               push 5  
733D7991     E8 C3D9FDFF         call MSVBVM60.733B5359  
733D7996     56                  push esi  
733D7997     8D45 F4             lea eax,dword ptr ss:[ebp\-C]  
733D799A     56                  push esi  
733D799B     50                  push eax  
733D799C     57                  push edi  
                                  ==== > EDI=fly12345678fly[OCN][FCG]E 
733D799D     E8 9A3D0A00         call MSVBVM60.7347B73C  
                                  ==== > 将 fly12345678fly[OCN][FCG]E 中的大写字母转为小写字母！ 

733D79A2     8BF0                mov esi,eax  
                                  ==== > ESI=fly12345678fly[ocn][fcg]e 
```
……省略……
```
733D7A1F     E9 FACBFCFF         jmp MSVBVM60.733A461E  
                                  ==== > 转变完了再跳上去！ 
```

## 注册源码的生成

```
:00461E75 FF15E4124000           Call dword ptr [004012E4]  
:00461E7B 83C418                add esp, 00000018  
:00461E7E 8B0D10804A00          mov ecx, dword ptr [004A8010]  
                                  ==== > ECX=211C1E09             C 盘的硬盘序列号 
:00461E84 81E957300E00          sub ecx, 000E3057  
                                  ==== > ECX=211C1E09 - 000E3057=210DEDB2 
:00461E8A 0F80C1070000          jo 00462651  
:00461E90 51                    push ecx 

* Reference To: MSVBVM60.__vbaStrI4, Ord:0000h  

                                  |  
:00461E91 FF1520104000          Call dword ptr [00401020]  
                                  ==== > 取 210DEDB2 的 10 进制值 
:00461E97 8BD0                  mov edx, eax  
                                  ==== > EDX=554560946 
:00461E99 8D4DC4                lea ecx, dword ptr [ebp\-3C]  
:00461E9C FFD6                  call esi  
:00461E9E 50                    push eax 

* Reference To: MSVBVM60.rtcStrReverse, Ord:02C9h  

                                  |  
:00461E9F FF153C124000          Call dword ptr [0040123C]  
                                  ==== > 把 554560946 倒序排列 
:00461EA5 8BD0                  mov edx, eax  
                                  ==== > EDX=649065455 
```
……省略……
```
:00461FCC 8B4344                mov eax, dword ptr [ebx+44]  
                                  ==== > EAX=FLY                  计算机用户名 
:00461FCF 50                    push eax  
:00461FD0 683C994200            push 0042993C  
                                  ==== > 0042993C=N        这个应该是作者预设的固定值 

* Reference To: MSVBVM60.__vbaStrCat, Ord:0000h  

                                  |  
:00461FD5 8B3D80104000          mov edi, dword ptr [00401080]  
:00461FDB FFD7                  call edi  
                                  ==== > 连接 FLY 和 N 
:00461FDD 8BD0                  mov edx, eax  
                                  ==== > EDX=FLYN 
:00461FDF 8D4DC0                lea ecx, dword ptr [ebp\-40]  
:00461FE2 FFD6                  call esi  
:00461FE4 50                    push eax  
:00461FE5 8B957CFEFFFF          mov edx, dword ptr [ebp+FFFFFE7C]  
                                  ==== > EDX=649065455 
:00461FEB 8D4DBC                lea ecx, dword ptr [ebp\-44]  
:00461FEE FFD6                  call esi  
:00461FF0 50                    push eax  
:00461FF1 FFD7                  call edi  
                                  ==== > 连接 FLYN 和 649065455 
:00461FF3 8BD0                  mov edx, eax  
                                  ==== > EDX=FLYN649065455  这就是程序显示的注册源码！ 
```

## 算法总结

1. 注册码要有`-`
2. 去除`-`后还需要 `25` 位数字或字母
3. 取 C 盘序列号 `211C1E09 - 007B33CF=20A0EA3A`
4. `25` 位字符中要有 `8` 位是 `20A0EA3A`，其他任意

不清楚程序是否还有其他暗桩，有朋友发现的话，麻烦指出来！

## 完美爆破

1. `00472574 0F850C010000          jne 00472686`  
   改为：`909090909090          NOP 掉！`
2. `004725F3 668BD8               mov bx, ax`  
   改为：`B301                 mov bl, 01`

## 注册信息保存

`REGEDIT4`

`[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\easypad\regist]`

`"regnumber"="fly-20A0EA3A-fly[OCN][FCG]-E"`

## 整理

序列号：`FLYN649065455613`

注册码：`fly-20A0EA3A-fly[OCN][FCG]-E`

```
          ,      \_/   
         /| \_.-~/      \\_     ,        青春都一饷
         ( /~   /          \~-.\_ |\
         `\\\\  \_/           \   ~\ )       忍把浮名
          _-~~~-.)  )\_\_/;;,.      \\_  //’
         /'\_,\   --~   \ ~~~-  ,;;\_\_(  (.-~~~-.      换了破解轻狂
        \`~ \_( ,\_..--\ (     ,;'' /   ~--   /.\_\`\\
       /~~//'   /' \`~\       ) /--..\_, )\_  \`~
       "  \`~"  "      \`"      /~'\`\\   \`\\\\~~\\ 

                  "      "   "~'  ""

                Cracked By 巢水工作坊——fly [OCN][FCG] 

                      2003-06-28  22:35 

 4dd466d0e1e0cc123afbe504da2462ca
```
