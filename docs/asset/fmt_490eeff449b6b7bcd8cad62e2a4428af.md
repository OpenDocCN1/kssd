
:00451752 `8BF3` `mov esi, ebx` `esi = ebx`

:00451754 `03F6` `add esi, esi` `esi * 2`

:00451756 `8BC6` `mov eax, esi` `eax = esi`

:00451758 `83C00E` `add eax, 0000000E` `eax = 14`

:0045175B `8A0407` `mov al, byte ptr [edi+eax]` 第 15 位到 `al`（`eax` 改变）

:0045175E `E881FEFFFF` `call 004515E4` 转换

:00451763 `C1E004` `shl eax, 04` `eax` 逻辑左移 4 位

:00451766 `50` `push eax` `eax` 入栈

:00451767 `83C60F` `add esi, 0000000F` `esi += 15`

:0045176A `8A0437` `mov al, byte ptr [edi+esi]` 第 16 位到 `al`（`eax` 改变）

:0045176D `E872FEFFFF` `call 004515E4` 转换

:00451772 `5A` `pop edx` 入栈的原 `eax` 值出栈到 `edx`

:00451773 `0BD0` `or edx, eax` 或运算（`al` 已经变了）

:00451775 `8D4308` `lea eax, dword ptr [ebx+08]`

:00451778 `889060864800` `mov byte ptr [eax+00488660], dl` 存 `dl` 到目标第九位

:0045177E `43` `inc ebx` `ebx++`

:0045177F `83FB04` `cmp ebx, 00000004` `ebx` 等于 4 时

:00451782 `75CE` `jne 00451752` 停止循环（4 次）（取注册码第（15，16），（17，18）（19，20）（21，22）位到目标第 9，10，11，12 位）

:00451784 `8A4712` `mov al, byte ptr [edi+12]` 注册码第 13 位到 `al`

:00451787 `E858FEFFFF` `call 004515E4` 转换(`eax` 改变）

:0045178C `8BD8` `mov ebx, eax` `ebx = eax`

:0045178E `C1E30C` `shl ebx, 0C` `ebx` 左移 12 位（16 进制 3 位）

:00451791 `8A4713` `mov al, byte ptr [edi+13]` 第 14 位到 `al`

:00451794 `E84BFEFFFF` `call 004515E4` 转换(`eax` 改变）

:00451799 `C1E008` `shl eax, 08` `eax` 左移 8 位

:0045179C `0BD8` `or ebx, eax` `ebx` 与 `eax` 或运算

:0045179E `8A4714` `mov al, byte ptr [edi+14]` 第 15 位到 `al`

:004517A1 `E83EFEFFFF` `call 004515E4` 转换（`eax` 改变）

:004517A6 `C1E004` `shl eax, 04` `eax` 左移 4 位

:004517A9 `0BD8` `or ebx, eax` `ebx` 再与 `eax` 或

:004517AB `8A4715` `mov al, byte ptr [edi+15]` 第 16 位到 `al`

:004517AE `E831FEFFFF` `call 004515E4` 转换(`eax` 改变）

:004517B3 `0BD8` `or ebx, eax` `ebx` 再与 `eax` 或

:004517B5 `8A4716` `mov al, byte ptr [edi+16]` 第 17 位到 `al`

:004517B8 `E827FEFFFF` `call 004515E4` 转换(`eax` 改变）

:004517BD `8BF0` `mov esi, eax` `esi = eax`

:004517BF `C1E61C` `shl esi, 1C` `esi` 左移 28 位

:004517C2 `8A4717` `mov al, byte ptr [edi+17]` 第 18 位到 `al`

:004517C5 `E81AFEFFFF` `call 004515E4` 转换(`eax` 改变）

:004517CA `C1E018` `shl eax, 18` `eax` 左移 24 位

:004517CD `0BF0` `or esi, eax` `esi` 与 `eax` 或

:004517CF `8A4718` `mov al, byte ptr [edi+18]` 第 19 位到 `al`

:004517D2 `E80DFEFFFF` `call 004515E4` 转换(`eax` 改变）

:004517D7 `C1E014` `shl eax, 14` `eax` 左移 20 位

:004517DA `0BF0` `or esi, eax` `esi` 再与 `eax` 或

:004517DC `8A4719` `mov al, byte ptr [edi+19]` 第 20 位

:004517DF `E800FEFFFF` `call 004515E4`

:004517E4 `C1E010` `shl eax, 10`

:004517E7 `0BF0` `or esi, eax`

:004517E9 `8A471A` `mov al, byte ptr [edi+1A]` 第 21 位

:004517EC `E8F3FDFFFF` `call 004515E4`

:004517F1 `C1E00C` `shl eax, 0C`

:004517F4 `0BF0` `or esi, eax`

:004517F6 `8A471B` `mov al, byte ptr [edi+1B]` 第 22 位

:004517F9 `E8E6FDFFFF` `call 004515E4`

:004517FE `C1E008` `shl eax, 08`

:00451801 `0BF0` `or esi, eax`

:00451803 `8A471C` `mov al, byte ptr [edi+1C]` 第 23 位

:00451806 `E8D9FDFFFF` `call 004515E4`

:0045180B `C1E004` `shl eax, 04`

:0045180E `0BF0` `or esi, eax`

:00451810 `8A471D` `mov al, byte ptr [edi+1D]` 第 24 位

:00451813 `E8CCFDFFFF` `call 004515E4` 调用（此位下面`:0045181A` 的调用要用到）

:00451818 `0BF0` `or esi, eax` `esi` 再于 `eax` 或，结果到 `esi`

:0045181A `E861FDFFFF` `call 00451580` 调用见下

:0045181F `3BD8` `cmp ebx, eax` 比较 `ebx` 与 `eax`

:00451821 `0F85F8000000` `jne 0045191F` 不等于就玩完

:00451827 `E888FDFFFF` `call 004515B4` 调用类似上一个

:0045182C `3BF0` `cmp esi, eax` 比较 `esi` 与 `eax`

:0045182E `0F85EB000000` `jne 0045191F` 不等于就玩完

:00451834 `8A4707` `mov al, byte ptr [edi+07]` 注册码第 8 位到 `al`

:00451837 `E8A8FDFFFF` `call 004515E4` 还记得这个转换吧

:0045183C `83F803` `cmp eax, 00000003` `eax` 与 3 比较

:0045183F `0F85DA000000` `jne 0045191F` 不等于就玩完

:00451845 `8A4708` `mov al, byte ptr [edi+08]`

:00451848 `E897FDFFFF` `call 004515E4`

:0045184D `83F802` `cmp eax, 00000002`

:00451850 `0F85C9000000` `jne 0045191F` 不等于就玩完

:00451856 `807F042D` `cmp byte ptr [edi+04], 2D`

:0045185A `0F85BF000000` `jne 0045191F` 不等于就玩完

:00451860 `807F0D2D` `cmp byte ptr [edi+0D], 2D`

:00451864 `0F85B5000000` `jne 0045191F` 不等于就玩完

:0045186A `8D45F0` `lea eax, dword ptr [ebp-10]` 、

:0045186D `8A17` `mov dl, byte ptr [edi]` |

:0045186F `E89C1EFBFF` `call 00403710` |

:00451874 `8B45F0` `mov eax, dword ptr [ebp-10]` |

:00451877 `8D55F4` `lea edx, dword ptr [ebp-0C]` |

:0045187A `E81546FBFF` `call 00405E94` 》一个块

:0045187F `8B45F4` `mov eax, dword ptr [ebp-0C]` |

:00451882 `BA5C194500` `mov edx, 0045195C` |

:00451887 `E8F81FFBFF` `call 00403884` |

:0045188C `0F858D000000` `jne 0045191F` ~ 还是玩完

:00451892 `8D45F0` `lea eax, dword ptr [ebp-10]`

:00451895 `8A5701` `mov dl, byte ptr [edi+01]`

:00451898 `E8731EFBFF` `call 00403710`

:0045189D `8B45F0` `mov eax, dword ptr [ebp-10]`

:004518A0 `8D55F4` `lea edx, dword ptr [ebp-0C]`

:004518A3 `E8EC45FBFF` `call 00405E94`

:004518A8 `8B45F4` `mov eax, dword ptr [ebp-0C]`

:004518AB `BA68194500` `mov edx, 00451968`

:004518B0 `E8CF1FFBFF` `call 00403884`

:004518B5 `7568` `jne 0045191F`

:004518B7 `8D45F0` `lea eax, dword ptr [ebp-10]`

:004518BA `8A5702` `mov dl, byte ptr [edi+02]`

:004518BD `E84E1EFBFF` `call 00403710`

:004518C2 `8B45F0` `mov eax, dword ptr [ebp-10]`

:004518C5 `8D55F4` `lea edx, dword ptr [ebp-0C]`

:004518C8 `E8C745FBFF` `call 00405E94`

:004518CD `8B45F4` `mov eax, dword ptr [ebp-0C]`

:004518D0 `BA74194500` `mov edx, 00451974`

:004518D5 `E8AA1FFBFF` `call 00403884`

:004518DA `7543` `jne 0045191F`
