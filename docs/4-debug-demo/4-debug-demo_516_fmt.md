
# BCL Magellan Plugin Demo 4.2

> 原文：`www.pediy.com/kssd/pediy04/kanxue415.htm`

解密者: `PeterChen`

组织: `Free Cracking Group of China=[FCG]`

对象: `BCL Magellan Plugin Demo 4.2`

功能: `PDF to HTML Publishing Tool`

保护: 超级大水印与自制广告词。

破解工具: `win32asm9.0`

它是 `Acrobat 5.x` 正版加强功能插件，老外很早就有破解文件下载。

就是没有看到，老外最近也太水气了吧？这个非一般破解，应该拿出来交流一下才对!

没有办法，明天就要上班。就试一下自己动手，写一篇教程出来祝贺 `Last Day`!

很久没有动手，分析它流程用一天时间。_

最明显提示:

```
--------------------------------------------------------------------------------
This page is not extracted in demo version.
--------------------------------------------------------------------------------
BCL
http://www.bcl-computers.com/
--------------------------------------------------------------------------------
```

最后完整搞定，就是听前恩师话耐心分析汇编流程内容:

* Possible Reference to String Resource ID=00049: "Processing Page %d of %d..."

```
:10004C4F 6A31        push 00000031
:10004C51 8D4DF0      lea ecx, dword ptr [ebp-10]
:10004C54 E874640500  call 1005B0CD
:10004C59 8B45EC      mov eax, dword ptr [ebp-14]
:10004C5C 8B8E5C110000 mov ecx, dword ptr [esi+0000115C]
:10004C62 8B55F0      mov edx, dword ptr [ebp-10]
:10004C65 40          inc eax
:10004C66 41          inc ecx
:10004C67 51          push ecx
:10004C68 50          push eax
:10004C69 8D45E4      lea eax, dword ptr [ebp-1C]
:10004C6C 52          push edx
:10004C6D 50          push eax
:10004C6E E8A4140500  call 10056117
:10004C73 8B17        mov edx, dword ptr [edi]
:10004C75 83C410      add esp, 00000010
:10004C78 8D45E4      lea eax, dword ptr [ebp-1C]
:10004C7B 8BCF        mov ecx, edi
:10004C7D 50          push eax
:10004C7E FF521C      call [edx+1C]
:10004C81 51          push ecx
:10004C82 8D55E4      lea edx, dword ptr [ebp-1C]
:10004C85 8BCC        mov ecx, esp
:10004C87 8965AC      mov dword ptr [ebp-54], esp
:10004C8A 52          push edx
:10004C8B E8F25D0500  call 1005AA82
:10004C90 8B07        mov eax, dword ptr [edi]
:10004C92 8BCF        mov ecx, edi
:10004C94 FF5038      call [eax+38]
:10004C97 8A4514      mov al, byte ptr [ebp+14]
:10004C9A 84C0        test al, al
:10004C9C 742B        je 10004CC9
:10004C9E 8B8E5C110000 mov ecx, dword ptr [esi+0000115C]
:10004CA4 8B55EC      mov edx, dword ptr [ebp-14]
:10004CA7 41          inc ecx
:10004CA8 8D4201      lea eax, dword ptr [edx+01]
:10004CAB 51          push ecx
:10004CAC 50          push eax
:10004CAD 8D45F0      lea eax, dword ptr [ebp-10]
```

* Possible StringData Ref from Data Obj ->"Page %d of %d - Magellan"

```
:10004CB0 6824CD0710  push 1007CD24
:10004CB5 50          push eax
:10004CB6 E85C140500  call 10056117
:10004CBB 8B17        mov edx, dword ptr [edi]
:10004CBD 83C410      add esp, 00000010
:10004CC0 8D45F0      lea eax, dword ptr [ebp-10]
:10004CC3 8BCF        mov ecx, edi
:10004CC5 50          push eax
:10004CC6 FF5210      call [edx+10]
```

以上内容是整理 `pdf to html` 进程！

`----------------------------------------------------------------`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10004C3A(C), :10004C9C(C)|

```
:10004CC9 D98654110000 fld dword ptr [esi+00001154]
:10004CCF D80D04580610 fmul dword ptr [10065804]
:10004CD5 8B4DCC      mov ecx, dword ptr [ebp-34]
:10004CD8 8B45EC      mov eax, dword ptr [ebp-14]
```

`-------------------------------------------------------------`

```
:10004CDB 3BC1        cmp eax, ecx--->@ffset 4cdh 抢着必死信心，试改之!^\_^
:10004CDB 3BC0        cmp eax, eax--->Yeach！万岁，You Win!
```

`-------------------------------------------------------------`

```
:10004CDD D95DBC      fstp dword ptr [ebp-44]--->这好像是浮点命令。
:10004CE0 0F84A9050000 je 1000528F---->☆ 所以不修改此处而>>>
:10004CE6 8A8678110000 mov al, byte ptr [esi+00001178]
:10004CEC 84C0        test al, al
:10004CEE 0F8466050000 je 1000525A
:10004CF4 8B8644110000 mov eax, dword ptr [esi+00001144]
:10004CFA 83F804      cmp eax, 00000004
:10004CFD 0F8457050000 je 1000525A
:10004D03 83F801      cmp eax, 00000001
:10004D06 752E        jne 10004D36
:10004D08 8D9640100000 lea edx, dword ptr [esi+00001040]
:10004D0E 8D86400E0000 lea eax, dword ptr [esi+00000E40]
:10004D14 52          push edx
:10004D15 8D8E28080000 lea ecx, dword ptr [esi+00000828]
:10004D1B 50          push eax
:10004D1C 51          push ecx
:10004D1D 8D9568FEFFFF lea edx, dword ptr [ebp+FFFFFE68]
```

* Possible StringData Ref from Data Obj ->"%s%s%s"

```
:10004D23 6870C00710  push 1007C070
:10004D28 52          push edx
:10004D29 E80D3D0300  call 10038A3B
:10004D2E 83C414      add esp, 00000014
:10004D31 E996000000  jmp 10004DCC
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10004D06(C)|

```
:10004D36 83F802      cmp eax, 00000002
:10004D39 752B        jne 10004D66
:10004D3B 8D8640100000 lea eax, dword ptr [esi+00001040]
:10004D41 8D8E400E0000 lea ecx, dword ptr [esi+00000E40]
:10004D47 50          push eax
:10004D48 8D962C090000 lea edx, dword ptr [esi+0000092C]
:10004D4E 51          push ecx
:10004D4F 52          push edx
:10004D50 8D8568FEFFFF lea eax, dword ptr [ebp+FFFFFE68]
```

* Possible StringData Ref from Data Obj ->"%s%s%s"

```
:10004D56 6870C00710  push 1007C070
:10004D5B 50          push eax
:10004D5C E8DA3C0300  call 10038A3B
:10004D61 83C414      add esp, 00000014
:10004D64 EB66        jmp 10004DCC
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10004D39(C)|

```
:10004D66 83F803      cmp eax, 00000003
:10004D69 7561        jne 10004DCC
:10004D6B 8DBE40100000 lea edi, dword ptr [esi+00001040]
:10004D71 8D9E400E0000 lea ebx, dword ptr [esi+00000E40]
:10004D77 57          push edi
:10004D78 8D8E28080000 lea ecx, dword ptr [esi+00000828]
:10004D7E 53          push ebx
:10004D7F 51          push ecx
:10004D80 8D9568FEFFFF lea edx, dword ptr [ebp+FFFFFE68]
```

* Possible StringData Ref from Data Obj ->"%s%s%s"

```
:10004D86 6870C00710  push 1007C070
:10004D8B 52          push edx
```



```
:10004D8C E8AA3C0300              call 10038A3B   

:10004D91 57                      push edi   

:10004D92 8D862C090000           lea eax, dword ptr [esi+0000092C]   

:10004D98 53                      push ebx   

:10004D99 50                      push eax   

:10004D9A 8D8D20F5FFFF           lea ecx, dword ptr [ebp+FFFFF520]   

* Possible StringData Ref from Data Obj ->"%s%s%s"   

                                  |   

:10004DA0 6870C00710             push 1007C070   

:10004DA5 51                      push ecx   

:10004DA6 E8903C0300             call 10038A3B   

:10004DAB 8D9520F5FFFF           lea edx, dword ptr [ebp+FFFFF520]   

* Possible StringData Ref from Data Obj ->"aa"   

                                  |   

:10004DB1 6820CD0710             push 1007CD20   

:10004DB6 52                      push edx   

:10004DB7 E86C3C0300             call 10038A28   

:10004DBC 8BD8                    mov ebx, eax   

:10004DBE 83C430                  add esp, 00000030   

:10004DC1 85DB                    test ebx, ebx   

:10004DC3 895DC4                  mov dword ptr [ebp-3C], ebx   

:10004DC6 0F8443070000           je 1000550F   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

:10004D31(U), :10004D64(U), :10004D69(C)   

|   

:10004DCC 8D8568FEFFFF           lea eax, dword ptr [ebp+FFFFFE68]   

* Possible StringData Ref from Data Obj ->"aa"   

                                  |   

:10004DD2 6820CD0710             push 1007CD20   

:10004DD7 50                      push eax   

:10004DD8 E84B3C0300             call 10038A28   

:10004DDD 8BF8                    mov edi, eax   

:10004DDF 83C408                  add esp, 00000008   

:10004DE2 85FF                    test edi, edi   

:10004DE4 0F8425070000           je 1000550F   

* Possible StringData Ref from Data Obj ->"</BODY>

<STYLE TYPE="text/css">"

                                  |       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   

:10004DEA 68FCCC0710             push 1007CCFC 这里就是字体风格   

* Possible StringData Ref from Data Obj ->"%s

"

                                  |   

:10004DEF 68F8CC0710             push 1007CCF8   

:10004DF4 57                      push edi   

:10004DF5 E8C13B0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"<!-- /* page size information "

                                  ->"*/

"  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

                                  |               页面内容大小   

:10004DFA 68D4CC0710             push 1007CCD4   

* Possible StringData Ref from Data Obj ->"%s"

                                  |   

:10004DFF 6844B10710             push 1007B144   

:10004E04 57                      push edi   

:10004E05 E8B13B0300             call 100389BB   

:10004E0A 8B4DEC                  mov ecx, dword ptr [ebp-14]   

:10004E0D 51                      push ecx   

* Possible StringData Ref from Data Obj ->".pg%d{"

                                  |   

:10004E0E 68CCCC0710             push 1007CCCC   

:10004E13 57                      push edi   

:10004E14 E8A23B0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"absolute"

                                  |   

:10004E19 68C0CC0710             push 1007CCC0   

* Possible StringData Ref from Data Obj ->"position:%s;"

                                  |   

:10004E1E 68B0CC0710             push 1007CCB0   

:10004E23 57                      push edi   

:10004E24 E8923B0300             call 100389BB   

:10004E29 8B9684110000           mov edx, dword ptr [esi+00001184]   

:10004E2F 8B8E98110000           mov ecx, dword ptr [esi+00001198]   

:10004E35 8B8680110000           mov eax, dword ptr [esi+00001180]   

:10004E3B 03D1                    add edx, ecx   

:10004E3D 03C2                    add eax, edx   

:10004E3F 50                      push eax   

* Possible StringData Ref from Data Obj ->"top:%dpx

                                  |       ^^^^------->上边   

:10004E40 68A4CC0710             push 1007CCA4   

:10004E45 57                      push edi   

:10004E46 898680110000           mov dword ptr [esi+00001180], eax   

:10004E4C E86A3B0300             call 100389BB   

:10004E51 8B8688110000           mov eax, dword ptr [esi+00001188]   

:10004E57 50                      push eax   

* Possible StringData Ref from Data Obj ->"left:%dpx;"

                                  |       ^^^^^^^^^---->左边   

:10004E58 6898CC0710             push 1007CC98   

:10004E5D 57                      push edi   

:10004E5E E8583B0300             call 100389BB   

:10004E63 8B8E8C110000           mov ecx, dword ptr [esi+0000118C]   

:10004E69 83C448                  add esp, 00000048   

:10004E6C 51                      push ecx   

* Possible StringData Ref from Data Obj ->"height:%dpx;"

                                  |       ^^^^^^---------->高度   

:10004E6D 6888CC0710             push 1007CC88   

:10004E72 57                      push edi   

:10004E73 E8433B0300             call 100389BB   

:10004E78 8B9690110000           mov edx, dword ptr [esi+00001190]   

:10004E7E 52                      push edx   

* Possible StringData Ref from Data Obj ->"width:%dpx;"

                                  |       ^^^^^------------->宽度   

:10004E7F 687CCC0710             push 1007CC7C   

:10004E84 57                      push edi   

:10004E85 E8313B0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"}

"

                                  |   

:10004E8A 6878CC0710             push 1007CC78   

:10004E8F 57                      push edi   

:10004E90 E8263B0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"

"

                                  |   

:10004E95 6874CC0710             push 1007CC74   

:10004E9A 57                      push edi   

:10004E9B E81B3B0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"/* text positioning information "

                                  ->"*/

" ^^^^^^^^^^^^^^^^^^^^^^^^^^

                                  |               准备放上内容   

:10004EA0 6850CC0710             push 1007CC50   

:10004EA5 57                      push edi   

:10004EA6 E8103B0300             call 100389BB   

:10004EAB D945BC                 fld dword ptr [ebp-44]   

:10004EAE D80D00580610           fmul dword ptr [10065800]   

:10004EB4 83C430                  add esp, 00000030   

:10004EB7 C745C800000000         mov [ebp-38], 00000000   

:10004EBE E8C5430300             call 10039288   

:10004EC3 D945BC                 fld dword ptr [ebp-44]   

:10004EC6 D80DFC570610           fmul dword ptr [100657FC]   

:10004ECC 8945B4                 mov dword ptr [ebp-4C], eax   

:10004ECF E8B4430300             call 10039288   

:10004ED4 8945B8                 mov dword ptr [ebp-48], eax   

:10004ED7 C745E000000000         mov [ebp-20], 00000000   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:10004F6C(C)

|

:10004EDE 8B45C8                 mov eax, dword ptr [ebp-38]   

:10004EE1 8B4DEC                 mov ecx, dword ptr [ebp-14]   

:10004EE4 50                      push eax   

:10004EE5 51                      push ecx   

* Possible StringData Ref from Data Obj ->".ps%dp%d{"

                                  |   

:10004EE6 6844CC0710             push 1007CC44   

:10004EEB 57                      push edi   

:10004EEC E8CA3A0300             call 100389BB   

* Possible StringData Ref from Data Obj ->"absolute"

                                  |   

:10004EF1 68C0CC0710             push 1007CCC0   

* Possible StringData Ref from Data Obj ->"position:%s;"

                                  |   
```



```
:10004EF6 68B0CC0710                 push 1007CCB0

:10004EFB 57                         push edi

:10004EFC E8BA3A0300                 call 100389BB

:10004F01 8B868C110000               mov eax, dword ptr [esi+0000118C]

:10004F07 8B8E80110000               mov ecx, dword ptr [esi+00001180]

:10004F0D 99                         cdq

:10004F0E 2BC2                       sub eax, edx

:10004F10 8B55E0                     mov edx, dword ptr [ebp-20]

:10004F13 D1F8                       sar eax, 1

:10004F15 03C1                       add eax, ecx

:10004F17 03C2                       add eax, edx

:10004F19 50                         push eax

* Possible StringData Ref from Data Obj ->"top:%dpx;"

                                   |

:10004F1A 68A4CC0710                 push 1007CCA4

:10004F1F 57                         push edi

:10004F20 E8963A0300                 call 100389BB

:10004F25 8B55B4                     mov edx, dword ptr [ebp-4C]

:10004F28 52                         push edx

* Possible StringData Ref from Data Obj ->"left:%dpx;"

                                   |

:10004F29 6898CC0710                 push 1007CC98

:10004F2E 57                         push edi

:10004F2F E8873A0300                 call 100389BB

:10004F34 8B45B8                     mov eax, dword ptr [ebp-48]

:10004F37 50                         push eax

* Possible StringData Ref from Data Obj ->"width:%dpx;"

                                   |

:10004F38 687CCC0710                 push 1007CC7C

:10004F3D 57                         push edi

:10004F3E E8783A0300                 call 100389BB

:10004F43 83C440                     add esp, 00000040

* Possible StringData Ref from Data Obj ->"}}

                                   |

:10004F46 6840CC0710                 push 1007CC40

* Possible StringData Ref from Data Obj ->"%s"

                                   |

:10004F4B 6838CC0710                 push 1007CC38

:10004F50 57                         push edi

:10004F51 E8653A0300                 call 100389BB

:10004F56 8B45C8                     mov eax, dword ptr [ebp-38]

:10004F59 83C40C                     add esp, 0000000C

:10004F5C 40                         inc eax

:10004F5D 8945C8                     mov dword ptr [ebp-38], eax

:10004F60 8B45E0                     mov eax, dword ptr [ebp-20]

:10004F63 83C01E                     add eax, 0000001E

:10004F66 83F83C                     cmp eax, 0000003C

:10004F69 8945E0                     mov dword ptr [ebp-20], eax

:10004F6C 0F8C6CFFFFFF               jl 10004EDE

* Possible StringData Ref from Data Obj ->"-->

</STYLE>

<BODY>

"

                                   |       ^^^^^^^^^^^^^^^^^------>又是字体风格

:10004F72 6820CC0710                 push 1007CC20

* Possible StringData Ref from Data Obj ->"%s"

                                   |

:10004F77 6844B10710                 push 1007B144

:10004F7C 57                         push edi

:10004F7D E8393A0300                 call 100389BB

以上内容：相信会 html 编写的朋友都知道是什么吧？大形式如下:

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------

生成这样线条!注这时线条，是不可见！只是大至定线条位置

______________________________________________________________________________

* Possible StringData Ref from Data Obj ->"<!-- text starts here -->"

                                   |       ^^^^^^^^^^^^^^^^^^^^^^^^垃圾内容准备加载！

:10004F82 6804CC0710                 push 1007CC04

* Possible StringData Ref from Data Obj ->"%s"

                                   |

:10004F87 6838CC0710                 push 1007CC38

:10004F8C 57                         push edi

:10004F8D E8293A0300                 call 100389BB

:10004F92 8B4DEC                     mov ecx, dword ptr [ebp-14]

:10004F95 6A00                       push 00000000

:10004F97 51                         push ecx

* Possible StringData Ref from Data Obj ->"<SPAN class="ps%dp%d">"

                                   |       ^^^^^^^^^^^^^^^^^^^^^^---------->线条出来了!:(

:10004F98 68ECCB0710                 push 1007CBEC

:10004F9D 57                         push edi

:10004F9E E8183A0300                 call 100389BB

* Possible StringData Ref from Data Obj ->"This "

                                           ->"page is not extracted in demo "

                                           ->"version."

                                   |       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^垃圾内容。

:10004FA3 6898CB0710                 push 1007CB98

:10004FA8 57                         push edi

:10004FA9 E80D3A0300                 call 100389BB

* Possible StringData Ref from Data Obj ->"</SPAN>"

                                   |       ^^^^^^^^^-------->结束语

:10004FAE 6890CB0710                 push 1007CB90

* Possible StringData Ref from Data Obj ->"%s"

                                   |

:10004FB3 68F8CC0710                 push 1007CCF8

:10004FB8 57                         push edi

:10004FB9 E8FD390300                 call 100389BB

:10004FBE 8B55EC                     mov edx, dword ptr [ebp-14]

* Possible Reference to String Resource ID=00001: "All"

                                   |

:10004FC1 6A01                       push 00000001

:10004FC3 52                         push edx

* Possible StringData Ref from Data Obj ->"<SPAN class="ps%dp%d">"

                                   |       ^^^^^^^^^^^^^^^^^^^^^^-------->第二条线出来

:10004FC4 68ECCB0710                 push 1007CBEC

:10004FC9 57                         push edi

:10004FCA E8EC390300                 call 100389BB

:10004FCF 83C44C                     add esp, 0000004C

* Possible StringData Ref from Data Obj ->"http://www.bcl-"

                                           ->"computers.com/"

                                   |       ^^^^^^^^^^^^^^^^^^^^^又句垃圾话。

:10004FD2 684CCB0710                 push 1007CB4C

:10004FD7 57                         push edi

:10004FD8 E8DE390300                 call 100389BB

* Possible StringData Ref from Data Obj ->"</SPAN>"

                                   |       ^^^^^^^----------->结束语

:10004FDD 6890CB0710                 push 1007CB90

* Possible StringData Ref from Data Obj ->"%s"

                                   |

:10004FE2 68F8CC0710                 push 1007CCF8

:10004FE7 57                         push edi

:10004FE8 E8CE390300                 call 100389BB

:10004FED 8B45EC                     mov eax, dword ptr [ebp-14]

:10004FF0 8B8E5C110000               mov ecx, dword ptr [esi+0000115C]

:10004FF6 83C414                     add esp, 00000014

:10004FF9 3BC1                       cmp eax, ecx

:10004FFB 750E                       jne 1000500B

* Possible StringData Ref from Data Obj ->"</BODY>

</HTML>

"

                                   |       ^^^^^^^^^^^^^^^^^^------->最后结束语

:10004FFD 6838CB0710                 push 1007CB38

:10005002 57                         push edi

:10005003 E8B3390300                 call 100389BB

:10005008 83C408                     add esp, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10004FFB(C)
|

:1000500B 57                         push edi
```



```asm
:1000500C E82D390300          call 1003893E
:10005011 8B8644110000       mov eax, dword ptr [esi+00001144]
:10005017 83C404             add esp, 00000004
:1000501A 83F803             cmp eax, 00000003
:1000501D 0F85F9010000       jne 1000521C

* Possible StringData Ref from Data Obj -> "<STYLE TYPE="text/css">"
                                  |       ^^^^^^^^^^^^^^^^^^^^^^^---->又是字体风格

:10005023 6820CB0710         push 1007CB20

* Possible StringData Ref from Data Obj -> "%s\n"
                                  |

:10005028 68F8CC0710         push 1007CCF8
:1000502D 53                 push ebx
:1000502E E888390300         call 100389BB

* Possible StringData Ref from Data Obj -> "<!-- /* page size information "
                                     -> "*/\n"
                                  |       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^----->同上

:10005033 68D4CC0710         push 1007CCD4

* Possible StringData Ref from Data Obj -> "%s"
                                  |

:10005038 6844B10710         push 1007B144
:1000503D 53                 push ebx
:1000503E E878390300         call 100389BB
:10005043 8B4DEC             mov ecx, dword ptr [ebp-14]
:10005046 51                 push ecx

* Possible StringData Ref from Data Obj -> ".pg%d{"
                                  |

:10005047 68CCCC0710         push 1007CCCC
:1000504C 53                 push ebx
:1000504D E869390300         call 100389BB

* Possible StringData Ref from Data Obj -> "absolute"
                                  |

:10005052 68C0CC0710         push 1007CCC0

* Possible StringData Ref from Data Obj -> "position:%s;"
                                  |

:10005057 68B0CC0710         push 1007CCB0
:1000505C 53                 push ebx
:1000505D E859390300         call 100389BB
:10005062 8B9684110000       mov edx, dword ptr [esi+00001184]
:10005068 8B8698110000       mov eax, dword ptr [esi+00001198]
:1000506E 8BBE80110000       mov edi, dword ptr [esi+00001180]
:10005074 03D0               add edx, eax
:10005076 03FA               add edi, edx
:10005078 8BC7               mov eax, edi
:1000507A 89BE80110000       mov dword ptr [esi+00001180], edi
:10005080 50                 push eax

* Possible StringData Ref from Data Obj -> "top:%dpx;"
                                  |

:10005081 68A4CC0710         push 1007CCA4
:10005086 53                 push ebx
:10005087 E82F390300         call 100389BB
:1000508C 8B8688110000       mov eax, dword ptr [esi+00001188]
:10005092 50                 push eax

* Possible StringData Ref from Data Obj -> "left:%dpx;"
                                  |

:10005093 6898CC0710         push 1007CC98
:10005098 53                 push ebx
:10005099 E81D390300         call 100389BB
:1000509E 8B8E8C110000       mov ecx, dword ptr [esi+0000118C]
:100050A4 83C448             add esp, 00000048
:100050A7 51                 push ecx

* Possible StringData Ref from Data Obj -> "height:%dpx;"
                                  |

:100050A8 6888CC0710         push 1007CC88
:100050AD 53                 push ebx
:100050AE E808390300         call 100389BB
:100050B3 8B9690110000       mov edx, dword ptr [esi+00001190]
:100050B9 52                 push edx

* Possible StringData Ref from Data Obj -> "width:%dpx;"
                                  |

:100050BA 687CCC0710         push 1007CC7C
:100050BF 53                 push ebx
:100050C0 E8F6380300         call 100389BB

* Possible StringData Ref from Data Obj -> "}\n"
                                  |

:100050C5 6878CC0710         push 1007CC78
:100050CA 53                 push ebx
:100050CB E8EB380300         call 100389BB

* Possible StringData Ref from Data Obj -> "\n"
                                  |

:100050D0 6874CC0710         push 1007CC74
:100050D5 53                 push ebx
:100050D6 E8E0380300         call 100389BB

* Possible StringData Ref from Data Obj -> "/* text positioning information "
                                     -> "*/\n"
^^^^^^^^^^^^^^^^^^^^^^^^^^^---->不用多说
                                  |

:100050DB 6850CC0710         push 1007CC50
:100050E0 53                 push ebx
:100050E1 E8D5380300         call 100389BB
:100050E6 83C430             add esp, 00000030
:100050E9 C745E000000000     mov [ebp-20], 00000000
:100050F0 BFD4FEFFFF         mov edi, FFFFFED4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000517A(C)
|

:100050F5 8B45E0             mov eax, dword ptr [ebp-20]
:100050F8 8B4DEC             mov ecx, dword ptr [ebp-14]
:100050FB 50                 push eax
:100050FC 51                 push ecx

* Possible StringData Ref from Data Obj -> ".ps%dp%d{"
                                  |

:100050FD 6844CC0710         push 1007CC44
:10005102 53                 push ebx
:10005103 E8B3380300         call 100389BB

* Possible StringData Ref from Data Obj -> "absolute"
                                  |

:10005108 68C0CC0710         push 1007CCC0

* Possible StringData Ref from Data Obj -> "position:%s;"
                                  |

:1000510D 68B0CC0710         push 1007CCB0
:10005112 53                 push ebx
:10005113 E8A3380300         call 100389BB
:10005118 8B9694110000       mov edx, dword ptr [esi+00001194]
:1000511E 8B8680110000       mov eax, dword ptr [esi+00001180]
:10005124 03D0               add edx, eax
:10005126 03D7               add edx, edi
:10005128 52                 push edx

* Possible StringData Ref from Data Obj -> "top:%dpx;"
                                  |

:10005129 68A4CC0710         push 1007CCA4
:1000512E 53                 push ebx
:1000512F E887380300         call 100389BB
:10005134 68C8000000         push 000000C8

* Possible StringData Ref from Data Obj -> "left:%dpx;"
                                  |

:10005139 6898CC0710         push 1007CC98
:1000513E 53                 push ebx
:1000513F E877380300         call 100389BB
:10005144 6890010000         push 00000190

* Possible StringData Ref from Data Obj -> "width:%dpx;"
                                  |

:10005149 687CCC0710         push 1007CC7C
:1000514E 53                 push ebx
:1000514F E867380300         call 100389BB
:10005154 83C440             add esp, 00000040

* Possible StringData Ref from Data Obj -> "}}"
                                  |

:10005157 6840CC0710         push 1007CC40

* Possible StringData Ref from Data Obj -> "%s\n\n"
                                  |

:1000515C 6838CC0710         push 1007CC38
:10005161 53                 push ebx
```



```assembly
:10005162 E854380300                call 100389BB
:10005167 8B55E0                   mov edx, dword ptr [ebp-20]
:1000516A 83C40C                   add esp, 0000000C
:1000516D 83C71E                   add edi, 0000001E
:10005170 42                       inc edx
:10005171 81FF10FFFFFF             cmp edi, FFFFFF10
:10005177 8955E0                   mov dword ptr [ebp-20], edx
:1000517A 0F8C75FFFFFF             jl 100050F5

* Possible StringData Ref from Data Obj ->"-->
</STYLE>
<BODY>
"
|
:10005180 6820CC0710               push 1007CC20

* Possible StringData Ref from Data Obj ->"%s"
|
:10005185 6844B10710               push 1007B144
:1000518A 53                       push ebx
:1000518B E82B380300               call 100389BB

以后就内容，就已经产生:
-----------------------------------------------------浅黑色
This page is not extracted in demo version. ----->红色
--------------------------------------------------------------------------------
BCL --->2 号大蓝字体
http://www.bcl-computers.com/--->蓝色的链接
--------------------------------------------------------------------------------
__________________________________________________________________________________
* Possible StringData Ref from Data Obj ->"<!-- text starts here -->"
|
:10005190 6804CC0710               push 1007CC04

* Possible StringData Ref from Data Obj ->"%s

"
|
:10005195 6838CC0710               push 1007CC38
:1000519A 53                       push ebx
:1000519B E81B380300               call 100389BB
:100051A0 8B7DEC                   mov edi, dword ptr [ebp-14]
:100051A3 6A00                     push 00000000
:100051A5 57                       push edi

* Possible StringData Ref from Data Obj ->"<SPAN class="ps%dp%d">"
|
:100051A6 68ECCB0710               push 1007CBEC
:100051AB 53                       push ebx
:100051AC E80A380300               call 100389BB

* Possible StringData Ref from Data Obj ->"This "
                                      ->"page is not extracted in demo "
                                      ->"version."
|
:100051B1 6898CB0710               push 1007CB98
:100051B6 53                       push ebx
:100051B7 E8FF370300               call 100389BB

* Possible StringData Ref from Data Obj ->"</SPAN>"
|
:100051BC 6890CB0710               push 1007CB90

* Possible StringData Ref from Data Obj ->"%s
"
|
:100051C1 68F8CC0710               push 1007CCF8
:100051C6 53                       push ebx
:100051C7 E8EF370300               call 100389BB

* Possible Reference to String Resource ID=00001: "All"
|
:100051CC 6A01                     push 00000001
:100051CE 57                       push edi

* Possible StringData Ref from Data Obj ->"<SPAN class="ps%dp%d">"
|
:100051CF 68ECCB0710               push 1007CBEC
:100051D4 53                       push ebx
:100051D5 E8E1370300               call 100389BB
:100051DA 83C44C                   add esp, 0000004C

* Possible StringData Ref from Data Obj ->"http://www.bcl-"
                                      ->"computers.com/"
|
:100051DD 684CCB0710               push 1007CB4C
:100051E2 53                       push ebx
:100051E3 E8D3370300               call 100389BB

* Possible StringData Ref from Data Obj ->"</SPAN>"
|
:100051E8 6890CB0710               push 1007CB90

* Possible StringData Ref from Data Obj ->"%s
"
|
:100051ED 68F8CC0710               push 1007CCF8
:100051F2 53                       push ebx
:100051F3 E8C3370300               call 100389BB
:100051F8 8B865C110000             mov eax, dword ptr [esi+0000115C]
:100051FE 83C414                   add esp, 00000014
:10005201 3BF8                     cmp edi, eax
:10005203 750E                     jne 10005213

* Possible StringData Ref from Data Obj ->"</BODY>

</HTML>
"
|
:10005205 6838CB0710               push 1007CB38
:1000520A 53                       push ebx
:1000520B E8AB370300               call 100389BB
:10005210 83C408                   add esp, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10005203(C)
|
:10005213 53                       push ebx
:10005214 E825370300               call 1003893E
:10005219 83C404                   add esp, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000501D(C)
|
:1000521C 8B7D10                   mov edi, dword ptr [ebp+10]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:10005277(U), :1000528D(U), :10005301(U), :1000534C(U), :10005384(U)
|
:1000521F 8B45DC                   mov eax, dword ptr [ebp-24]
:10005222 85C0                     test eax, eax
:10005224 0F855F010000             jne 10005389
:1000522A 85FF                     test edi, edi
:1000522C 0F84EB010000             je 1000541D
:10005232 8B17                     mov edx, dword ptr [edi]
:10005234 8BCF                     mov ecx, edi
:10005236 FF5250                   call [edx+50]
:10005239 84C0                     test al, al
:1000523B 0F85C9010000             jne 1000540A
:10005241 8B07                     mov eax, dword ptr [edi]
:10005243 8BCF                     mov ecx, edi
:10005245 8945E0                   mov dword ptr [ebp-20], eax
:10005248 FF5044                   call [eax+44]
:1000524B 48                       dec eax
:1000524C 8BCF                     mov ecx, edi
:1000524E 50                       push eax
:1000524F 8B45E0                   mov eax, dword ptr [ebp-20]
:10005252 FF5040                   call [eax+40]
:10005255 E9B0010000               jmp 1000540A

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:10004CEE(C), :10004CFD(C)
|
:1000525A 83BE4411000004           cmp dword ptr [esi+00001144], 00000004
:10005261 7416                     je 10005279
:10005263 8B45EC                   mov eax, dword ptr [ebp-14]
:10005266 50                       push eax
:10005267 56                       push esi
:10005268 E883040000               call 100056F0
:1000526D 83C408                   add esp, 00000008
:10005270 C745DC00000000           mov [ebp-24], 00000000
:10005277 EBA6                     jmp 1000521F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10005261(C)
|
:10005279 8B4DEC                   mov ecx, dword ptr [ebp-14]
:1000527C 51                       push ecx
:1000527D 56                       push esi
:1000527E 8D4DD4                   lea ecx, dword ptr [ebp-2C]
:10005281 E83AD7FFFF               call 100029C0
:10005286 C745DC00000000           mov [ebp-24], 00000000
```



> :1000528D EB90                     `jmp` 1000521F

> \----------------------------------------------------------------------------

> 作者也太小心眼了，又要玩 `Game`!一不小心就 `game over`!^\_^

> \----------------------------------------------------------------------------

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10004CE0(C)---->返回看一下！★

> |

> :1000528F A1286A0810                `mov eax, dword ptr [10086A28]`
> :10005294 8D956CFFFFFF             `lea edx, dword ptr [ebp+FFFFFF6C]`
> :1000529A 68B0D10010               `push 1000D1B0`
> :1000529F 52                       `push edx`
> :100052A0 FF5008                   `call [eax+08]`
> :100052A3 8D8D6CFFFFFF             `lea ecx, dword ptr [ebp+FFFFFF6C]`
> :100052A9 68D8360710               `push 100736D8`
> :100052AE 8B55FC                   `mov edx, dword ptr [ebp-04]`
> :100052B1 52                       `push edx`
> :100052B2 68AE830310               `push 100383AE`

>
> \* Possible Reference to String Resource ID=00003: "Selected Pages"
>
>                               |               ^^^^^^^^^^^^^^^^------->搜索页面

> :100052B7 6A03                     `push 00000003` 到这里才是正常生成 `html`,所以向上看。
> :100052B9 51                       `push ecx`
> :100052BA E845390300               `call 10038C04`
> :100052BF 8B7D10                   `mov edi, dword ptr [ebp+10]`
> :100052C2 83C41C                   `add esp, 0000001C`
> :100052C5 85C0                     `test eax, eax`
> :100052C7 0F8584000000             `jne 10005351`
> :100052CD 8B750C                   `mov esi, dword ptr [ebp+0C]`
> :100052D0 83BE4411000004           `cmp dword ptr [esi+00001144], 00000004`
> :100052D7 742D                     `je 10005306`
> :100052D9 8B45EC                   `mov eax, dword ptr [ebp-14]`
> :100052DC 8B4D08                   `mov ecx, dword ptr [ebp+08]`
> :100052DF 56                       `push esi`
> :100052E0 50                       `push eax`
> :100052E1 51                       `push ecx`
> :100052E2 E8D9420000               `call 100095C0`
> :100052E7 8945DC                   `mov dword ptr [ebp-24], eax`
> :100052EA A1286A0810               `mov eax, dword ptr [10086A28]`
> :100052EF 83C40C                   `add esp, 0000000C`
> :100052F2 FF500C                   `call [eax+0C]`
> :100052F5 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :100052F8 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :100052FB 83C002                   `add eax, 00000002`
> :100052FE 8945CC                   `mov dword ptr [ebp-34], eax`
> :10005301 E919FFFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:100052D7(C)
>
> |

> :10005306 8D4DD4                   `lea ecx, dword ptr [ebp-2C]`
> :10005309 E8B2D5FFFF               `call 100028C0`
> :1000530E 8B4D08                   `mov ecx, dword ptr [ebp+08]`
> :10005311 50                       `push eax`
> :10005312 8B45EC                   `mov eax, dword ptr [ebp-14]`
> :10005315 8D9524F6FFFF             `lea edx, dword ptr [ebp+FFFFF624]`
> :1000531B 52                       `push edx`
> :1000531C 8B1518690810             `mov edx, dword ptr [10086918]`
> :10005322 50                       `push eax`
> :10005323 51                       `push ecx`
> :10005324 FF5210                   `call [edx+10]`
> :10005327 83C410                   `add esp, 00000010`
> :1000532A 8945DC                   `mov dword ptr [ebp-24], eax`
> :1000532D 85C0                     `test eax, eax`
> :1000532F 7407                     `je 10005338`

>
> \* Possible Reference to String Resource ID=00013: "BCL Magellan Online Guide"
>
>                               |

> :10005331 C745DC0D000000           `mov [ebp-24], 0000000D`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000532F(C)
>
> |

> :10005338 A1286A0810               `mov eax, dword ptr [10086A28]`
> :1000533D FF500C                   `call [eax+0C]`
> :10005340 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :10005343 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :10005346 83C002                   `add eax, 00000002`
> :10005349 8945CC                   `mov dword ptr [ebp-34], eax`
> :1000534C E9CEFEFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:100052C7(C)
>
> |

> :10005351 8B0D286A0810             `mov ecx, dword ptr [10086A28]`
> :10005357 FF510C                   `call [ecx+0C]`
> :1000535A 8B750C                   `mov esi, dword ptr [ebp+0C]`

>
> \* Possible Reference to String Resource ID=00013: "BCL Magellan Online Guide"
>
>                               |

> :1000535D C745DC0D000000           `mov [ebp-24], 0000000D`
> :10005364 85FF                     `test edi, edi`
> :10005366 8B965C110000             `mov edx, dword ptr [esi+0000115C]`
> :1000536C 8955EC                   `mov dword ptr [ebp-14], edx`
> :1000536F 7407                     `je 10005378`
> :10005371 8B07                     `mov eax, dword ptr [edi]`
> :10005373 8BCF                     `mov ecx, edi`
> :10005375 FF504C                   `call [eax+4C]`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000536F(C)
>
> |

> :10005378 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :1000537B 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :1000537E 83C002                   `add eax, 00000002`
> :10005381 8945CC                   `mov dword ptr [ebp-34], eax`
> :10005384 E996FEFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:10005224(C)
>
> |

> :10005389 85FF                     `test edi, edi`
> :1000538B 7439                     `je 100053C6`
> :1000538D 8B07                     `mov eax, dword ptr [edi]`
> :1000538F 8BCF                     `mov ecx, edi`
> :10005391 8945E0                   `mov dword ptr [ebp-20], eax`
> :10005394 FF5044                   `call [eax+44]`
> :10005397 8B55E0                   `mov edx, dword ptr [ebp-20]`
> :1000539A 48                       `dec eax`
> :1000539B 50                       `push eax`
> :1000539C 8BCF                     `mov ecx, edi`
> :1000539E FF5240                   `call [edx+40]`
> :100053A1 51                       `push ecx`
> :100053A2 8B4DDC                   `mov ecx, dword ptr [ebp-24]`
> :100053A5 8BC4                     `mov eax, esp`
> :100053A7 8965B0                   `mov dword ptr [ebp-50], esp`
> :100053AA 51                       `push ecx`
> :100053AB 50                       `push eax`
> :100053AC E8FF0E0000               `call 100062B0`
> :100053B1 8B865C110000             `mov eax, dword ptr [esi+0000115C]`
> :100053B7 8B4DEC                   `mov ecx, dword ptr [ebp-14]`
> :100053BA 8B17                     `mov edx, dword ptr [edi]`
> :100053BC 83C408                   `add esp, 00000008`
> :100053BF 50                       `push eax`
> :100053C0 51                       `push ecx`
> :100053C1 8BCF                     `mov ecx, edi`
> :100053C3 FF5248                   `call [edx+48]`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000538B(C)
>
> |

> :100053C6 8B451C                   `mov eax, dword ptr [ebp+1C]`
> :100053C9 85C0                     `test eax, eax`
> :100053CB 743D                     `je 1000540A`
> :100053CD 8A45EB                   `mov al, byte ptr [ebp-15]`
> :100053D0 84C0                     `test al, al`
> :100053D2 751C                     `jne 100053F0`



```asm
:100053D4 8B451C            mov eax, dword ptr [ebp+1C]
:100053D7 8D9618040000      lea edx, dword ptr [esi+00000418]
:100053DD 52                push edx

* Possible StringData Ref from Data Obj -> "FILE: %s

"

                                                    |
:100053DE 6814CB0710        push 1007CB14
:100053E3 50                push eax
:100053E4 E8D2350300        call 100389BB
:100053E9 83C40C            add esp, 0000000C
:100053EC C645EB01          mov [ebp-15], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100053D2(C)
|
:100053F0 8B55EC            mov edx, dword ptr [ebp-14]
:100053F3 8B4DF0            mov ecx, dword ptr [ebp-10]
:100053F6 8B451C            mov eax, dword ptr [ebp+1C]
:100053F9 42                inc edx
:100053FA 51                push ecx
:100053FB 52                push edx

* Possible StringData Ref from Data Obj -> "PAGE: %d
%s
"

                                                    |
:100053FC 6804CB0710        push 1007CB04
:10005401 50                push eax
:10005402 E8B4350300        call 100389BB
:10005407 83C410            add esp, 00000010

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:1000523B(C), :10005255(U), :100053CB(C)
|
:1000540A 85FF              test edi, edi
:1000540C 740F              je 1000541D
:1000540E 8B45D0            mov eax, dword ptr [ebp-30]
:10005411 8B17              mov edx, dword ptr [edi]
:10005413 40                inc eax
:10005414 8BCF              mov ecx, edi
:10005416 50                push eax
:10005417 8945D0            mov dword ptr [ebp-30], eax
:1000541A FF5228            call [edx+28]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:1000522C(C), :1000540C(C)
|
:1000541D 8B45EC            mov eax, dword ptr [ebp-14]
:10005420 8B8E5C110000      mov ecx, dword ptr [esi+0000115C]
:10005426 40                inc eax
:10005427 3BC1              cmp eax, ecx
:10005429 8945EC            mov dword ptr [ebp-14], eax
:1000542C 0F8EDCF7FFFF      jle 10004C0E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10004C02(C)
|
:10005432 8D4DD4            lea ecx, dword ptr [ebp-2C]
:10005435 C645FC01          mov [ebp-04], 01
:10005439 E872D4FFFF        call 100028B0
:1000543E 8D4DE4            lea ecx, dword ptr [ebp-1C]
:10005441 C645FC00          mov [ebp-04], 00
:10005445 E8C3580500        call 1005AD0D
:1000544A 8D4DF0            lea ecx, dword ptr [ebp-10]
:1000544D C745FCFFFFFFFF    mov [ebp-04], FFFFFFFF
:10005454 E8B4580500        call 1005AD0D
:10005459 B001              mov al, 01
:1000545B 8B4DF4            mov ecx, dword ptr [ebp-0C]
:1000545E 64890D00000000    mov dword ptr fs:[00000000], ecx
:10005465 5F                pop edi
:10005466 5E                pop esi
:10005467 5B                pop ebx
:10005468 8BE5              mov esp, ebp
:1000546A 5D                pop ebp
:1000546B C3                ret
------------------------------------------------------------------------------------------------
```

难怪 RU，叫我遇到不常破解一定要分析流程。运行的是，这里都有易懂 `html` 命令。

最后总结几种破解思路：

1. **掷石问路法**：`This page is not extracted in demo version.`

   直接找此内容，不过 `win32asm` 在搜索只能看前顾后。

   `This or This page` [前]

   `demo version` [后]

   找到之后要向上分析。

2. **功能分析法**：它功能是 `pdf to html`

   所以着重放在 `html` 中！

   在 `win32asm` 字串窗找到：

   方法一：

   ```
   "<!-- text starts here -->
   "-->此处到正常状态!
   :1000B304 682CD50710             push 1007D52C
   "<!-- text starts here -->"--->关键就是这里了!
   :10004F82 6804CC0710             push 1007CC04-->从程序运行角度，就要在这里开始向上分析
   :10005190 6804CC0710             push 1007CC04
   ```

   方法二：

   ```
   "<HTML>
   <HEAD><TITLE>Blank Page</TITLE></HEAD>
   "
   :10005875 682CCF0710             push 1007CF2C--同理，向上看
   :10005926 682CCF0710             push 1007CF2C
   ```

3. **逻辑分析法**：从逻辑推角度，它应该在处理进程就有一致命判断。

   处理 [Processing]

   ```
   String Resource ID=00049: "Processing Page %d of %d..."
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   ```

   跳到好几处，不是好方法。再找一个比较好的：

   ```
   "Page %d of %d - Magellan"
                ^^^^^^^^^---->咦！标志这么明白？不管了有前无后打死就算!:)
   ```

   ```asm
   * Possible StringData Ref from Data Obj -> "Page %d of %d - Magellan"
                                                    |
   :10004CB0 6824CD0710        push 1007CD24
   :10004CB5 50                push eax
   ```

   只有一个好地方！

   上面就是 `Processing Page %d of %d...` 下面不久就是 `This page is not extracted in demo version.`

   最好关键是：`:10004CE0 0F84A9050000             je 1000528F`

   是否跳？而且这值，就要看 `:10004CDD D95DBC               fstp dword ptr [ebp-44]`

   这个值又取决于：`:10004CDB 3BC1                 cmp eax, ecx`

   所以试一下把修改为：`cmp eax,eax`

   果然成功。

   最后方法：就是关羽单枪负会，直接了当！

   很久没有写教程，打手软！谁掷东西？什么来的？不好了，嗅鸡蛋！当场嗅晕！!@#$% *_*

   师傅绝招，闪啊！^_^

   2002-2-18 晚上 11:30 分钟 `4dd466d0e1e0cc123afbe504da2462ca`
