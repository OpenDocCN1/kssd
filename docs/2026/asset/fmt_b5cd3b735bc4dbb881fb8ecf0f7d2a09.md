
                  ->"\*/

"  ^^^^^^^^^^^^^^^^^^^^^^^^

            |

      页面内容大小

:10004DFA 68D4CC0710              push 1007CCD4

\* Possible StringData Ref from Data Obj ->"%s"

            |

:10004DFF 6844B10710              push 1007B144

:10004E04 57

  push edi

:10004E05 E8B13B0300              call 100389BB

:10004E0A 8B4DEC

mov ecx, dword ptr [ebp-14]

:10004E0D 51

  push ecx

\* Possible StringData Ref from Data Obj ->".pg%d{"

            |

:10004E0E 68CCCC0710              push 1007CCCC

:10004E13 57

  push edi

:10004E14 E8A23B0300              call 100389BB

\* Possible StringData Ref from Data Obj ->"absolute"

            |

:10004E19 68C0CC0710              push 1007CCC0

\* Possible StringData Ref from Data Obj ->"position:%s;"

            |

:10004E1E 68B0CC0710              push 1007CCB0

:10004E23 57

  push edi

:10004E24 E8923B0300              call 100389BB

:10004E29 8B9684110000            mov edx, dword

ptr [esi+00001184]

:10004E2F 8B8E98110000            mov ecx, dword

ptr [esi+00001198]

:10004E35 8B8680110000            mov eax, dword

ptr [esi+00001180]

:10004E3B 03D1

  add edx, ecx

:10004E3D 03C2

  add eax, edx

:10004E3F 50

  push eax

\* Possible StringData Ref from Data Obj ->"top:%dpx

            |        ^^^^------->上边

:10004E40 68A4CC0710              push 1007CCA4

:10004E45 57

  push edi

:10004E46 898680110000            mov dword ptr

[esi+00001180], eax

:10004E4C E86A3B0300              call 100389BB

:10004E51 8B8688110000            mov eax, dword

ptr [esi+00001188]

:10004E57 50

  push eax

\* Possible StringData Ref from Data Obj ->"left:%dpx;"

            |        ^^^^^^^^^---->左边

:10004E58 6898CC0710              push 1007CC98

:10004E5D 57

  push edi

:10004E5E E8583B0300              call 100389BB

:10004E63 8B8E8C110000            mov ecx, dword

ptr [esi+0000118C]

:10004E69 83C448

add esp, 00000048

:10004E6C 51

  push ecx

\* Possible StringData Ref from Data Obj ->"height:%dpx;"

            |        ^^^^^^---------->高度

:10004E6D 6888CC0710              push 1007CC88

:10004E72 57

  push edi

:10004E73 E8433B0300              call 100389BB

:10004E78 8B9690110000            mov edx, dword

ptr [esi+00001190]

:10004E7E 52

  push edx

\* Possible StringData Ref from Data Obj ->"width:%dpx;"

            |        ^^^^^------------->宽度

:10004E7F 687CCC0710              push 1007CC7C

:10004E84 57

  push edi

:10004E85 E8313B0300              call 100389BB

\* Possible StringData Ref from Data Obj ->"}

"

            |

:10004E8A 6878CC0710              push 1007CC78

:10004E8F 57

  push edi

:10004E90 E8263B0300              call 100389BB

\* Possible StringData Ref from Data Obj ->"

"

            |

:10004E95 6874CC0710              push 1007CC74

:10004E9A 57

  push edi

:10004E9B E81B3B0300              call 100389BB

\* Possible StringData Ref from Data Obj ->"/\* text positioning information "

                  ->"\*/

" ^^^^^^^^^^^^^^^^^^^^^^^^^^

            |

      准备放上内容

:10004EA0 6850CC0710              push 1007CC50

:10004EA5 57

  push edi

:10004EA6 E8103B0300              call 100389BB

:10004EAB D945BC

fld dword ptr [ebp-44]

:10004EAE D80D00580610            fmul dword ptr

[10065800]

:10004EB4 83C430

add esp, 00000030

:10004EB7 C745C800000000          mov [ebp-38], 00000000

:10004EBE E8C5430300              call 10039288

:10004EC3 D945BC

fld dword ptr [ebp-44]

:10004EC6 D80DFC570610            fmul dword ptr

[100657FC]

:10004ECC 8945B4

mov dword ptr [ebp-4C], eax

:10004ECF E8B4430300              call 10039288

:10004ED4 8945B8

mov dword ptr [ebp-48], eax

:10004ED7 C745E000000000          mov [ebp-20], 00000000

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10004F6C(C)

|

:10004EDE 8B45C8

mov eax, dword ptr [ebp-38]

:10004EE1 8B4DEC

mov ecx, dword ptr [ebp-14]

:10004EE4 50

  push eax

:10004EE5 51

  push ecx

\* Possible StringData Ref from Data Obj ->".ps%dp%d{"

            |

:10004EE6 6844CC0710              push 1007CC44

:10004EEB 57

  push edi

:10004EEC E8CA3A0300              call 100389BB

\* Possible StringData Ref from Data Obj ->"absolute"

            |

:10004EF1 68C0CC0710              push 1007CCC0

\* Possible StringData Ref from Data Obj ->"position:%s;"

            |

:10004EF6 68B0CC0710              push 1007CCB0

:10004EFB 57

  push edi

:10004EFC E8BA3A0300              call 100389BB

:10004F01 8B868C110000            mov eax, dword

ptr [esi+0000118C]

:10004F07 8B8E80110000            mov ecx, dword

ptr [esi+00001180]

:10004F0D 99

  cdq

:10004F0E 2BC2

  sub eax, edx

:10004F10 8B55E0

mov edx, dword ptr [ebp-20]

:10004F13 D1F8

  sar eax, 1

:10004F15 03C1

  add eax, ecx

:10004F17 03C2

  add eax, edx

:10004F19 50

  push eax

\* Possible StringData Ref from Data Obj ->"top:%dpx;"

            |

:10004F1A 68A4CC0710              push 1007CCA4

:10004F1F 57

  push edi

:10004F20 E8963A0300              call 100389BB

:10004F25 8B55B4

mov edx, dword ptr [ebp-4C]

:10004F28 52

  push edx

> \* Possible StringData Ref from Data Obj ->"left:%dpx;"

>

>             |

> :10004F29 6898CC0710              push 1007CC98

>

> :10004F2E 57

>   push edi

> :10004F2F E8873A0300              call 100389BB

>

> :10004F34 8B45B8

> mov eax, dword ptr [ebp-48]

> :10004F37 50

>   push eax

>

> \* Possible StringData Ref from Data Obj ->"width:%dpx;"

>

>             |

> :10004F38 687CCC0710              push 1007CC7C

>

> :10004F3D 57

>   push edi

> :10004F3E E8783A0300              call 100389BB

>

> :10004F43 83C440

> add esp, 00000040

>

> \* Possible StringData Ref from Data Obj ->"}}"

>

>             |

> :10004F46 6840CC0710              push 1007CC40

>

>

> \* Possible StringData Ref from Data Obj ->"%s

>

> "

>

>             |

> :10004F4B 6838CC0710              push 1007CC38

>

> :10004F50 57

>   push edi

> :10004F51 E8653A0300              call 100389BB

>

> :10004F56 8B45C8

> mov eax, dword ptr [ebp-38]

> :10004F59 83C40C

> add esp, 0000000C

> :10004F5C 40

>   inc eax

> :10004F5D 8945C8

> mov dword ptr [ebp-38], eax

> :10004F60 8B45E0

> mov eax, dword ptr [ebp-20]

> :10004F63 83C01E

> add eax, 0000001E

> :10004F66 83F83C

> cmp eax, 0000003C

> :10004F69 8945E0

> mov dword ptr [ebp-20], eax

> :10004F6C 0F8C6CFFFFFF            jl 10004EDE

>

>

> \* Possible StringData Ref from Data Obj ->"-->

> </STYLE>

> <BODY>

> "

>

>             |

> ^^^^^^^^^^^^^^^^^------>又是字体风格

> :10004F72 6820CC0710              push 1007CC20

>

>

> \* Possible StringData Ref from Data Obj ->"%s"

>

>             |

> :10004F77 6844B10710              push 1007B144

>

> :10004F7C 57

>   push edi

> :10004F7D E8393A0300              call 100389BB

>

> 以上内容：相信会html编写的朋友都知道是什么吧？大形式如下:

> --------------------------------------------------------------------------------

>

>

> --------------------------------------------------------------------------------

>

>

> --------------------------------------------------------------------------------

>

> 生成这样线条!注这时线条，是不可见！只是大至定线条位置

> \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

>

> \* Possible StringData Ref from Data Obj ->"<!-- text starts here -->"

>

>             |        ^^^^^^^^^^^^^^^^^^^^^^^^垃圾内容准备加载！

>

> :10004F82 6804CC0710              push 1007CC04

>

>

> \* Possible StringData Ref from Data Obj ->"%s

>

> "

>

>             |

> :10004F87 6838CC0710              push 1007CC38

>

> :10004F8C 57

>   push edi

> :10004F8D E8293A0300              call 100389BB

>

> :10004F92 8B4DEC

> mov ecx, dword ptr [ebp-14]

> :10004F95 6A00

>   push 00000000

> :10004F97 51

>   push ecx

>

> \* Possible StringData Ref from Data Obj ->"<SPAN class="ps%dp%d">"

>

>             |        ^^^^^^^^^^^^^^^^^^^^^^---------->线条出来了!:(
