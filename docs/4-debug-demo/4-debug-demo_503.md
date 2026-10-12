
# 三角洲部队之大地勇士

> 原文：[`www.pediy.com/kssd/pediy03/forum1261.htm`](https://www.pediy.com/kssd/pediy03/forum1261.htm)

三角洲部队之大地勇士光盘版的破解

前几天到电脑市场买了这个游戏，没光盘也能玩，不过只能玩 `Multiplayer` 的 `join` 模式，不插光盘毫无提示，破解颇费了一番周折。首先查看了安装目录下的文件，发现 `DFLW.CD` 里面有光驱的盘符 `G:`，先下断 `BPX GETDRIVETYPE`，运行游戏......，怎么？没拦住？用 `W32DASM` 反汇编 `DFLW.EXE`，发现并没加壳。查看输入的函数，`KERNEL32` 里没有此函数，难怪没拦住。但看到 `GETVOLUMEINFORMATIONA`，双击它，来到：

```
\* Reference To: KERNEL32.GetVolumeInformationA, Ord:01DEh ----------------读取卷信息

  :0042E7E9 FF1544524E00            Call dword ptr [004E5244]
  :0042E7EF 6A0B                    push 0000000B
  :0042E7F1 8D8424B4000000          lea eax, dword ptr [esp+000000B4]

\* Possible StringData Ref from Data Obj ->"DELTAFORCELW" -----------------这不是光盘的卷名吗？有戏！

  :0042E7F8 682C834E00              push 004E832C
  :0042E7FD 50                      push eax
  :0042E7FE E89DA60800              call 004B8EA0
  :0042E803 83C40C                  add esp, 0000000C
  :0042E806 85C0                    test eax, eax
  :0042E808 7549                    jne 0042E853
  :0042E80A 53                      push ebx

\* Possible StringData Ref from Data Obj ->"CDFS" ------------------- 光盘的标志

  :0042E80B BE24834E00              mov esi, 004E8324
  :0042E810 8D8424B4010000          lea eax, dword ptr [esp+000001B4]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0042E839(C)
  |

  :0042E817 8A10                    mov dl, byte ptr [eax]
  :0042E819 8A1E                    mov bl, byte ptr [esi]
  :0042E81B 8ACA                    mov cl, dl
  :0042E81D 3AD3                    cmp dl, bl
  :0042E81F 751E                    jne 0042E83F
  :0042E821 84C9                    test cl, cl
  :0042E823 7416                    je 0042E83B
  :0042E825 8A5001                  mov dl, byte ptr [eax+01]
  :0042E828 8A5E01                  mov bl, byte ptr [esi+01]
  :0042E82B 8ACA                    mov cl, dl
  :0042E82D 3AD3                    cmp dl, bl
  :0042E82F 750E                    jne 0042E83F
  :0042E831 83C002                  add eax, 00000002
  :0042E834 83C602                  add esi, 00000002
  :0042E837 84C9                    test cl, cl
  :0042E839 75DC                    jne 0042E817

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0042E823(C)
  |

  :0042E83B 33C0                    xor eax, eax
  :0042E83D EB05                    jmp 0042E844

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0042E81F(C), :0042E82F(C)
  |

  :0042E83F 1BC0                    sbb eax, eax
  :0042E841 83D8FF                  sbb eax, FFFFFFFF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0042E83D(U)
  |

  :0042E844 85C0                    test eax, eax
  :0042E846 5B                      pop ebx
  :0042E847 750A                    jne 0042E853
  :0042E849 C705CCD9CD0000000000    mov dword ptr [00CDD9CC], 00000000

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0042E775(C), :0042E808(C), :0042E847(C)
  |

  :0042E853 5E                      pop esi
  :0042E854 81C4AC020000            add esp, 000002AC
  :0042E85A C3                      ret
```

试着将上述的几个跳转改一下，均未成功。苦思冥想时，觉得 `:0042E849 mov dword ptr [00CDD9CC], 00000000` 有点问题，试着改为 `mov dword ptr [00CDD9CC], 00000001`，再运行游戏，哇！成功了。原来光盘检测函数的返回值是个全局变量，在 `DS:00CDD9CC` 处，当它为 `0` 时表示无光盘，为 `1` 时表示有光盘。

用 `HIEW32t` 打开 `DFLW.EXE` 找到：

`C705CCD9CD0000000000`

改为：

`C705CCD9CD0001000000`

搞定！

大波罗

2001.8.24

转载请保持完整，谢谢
