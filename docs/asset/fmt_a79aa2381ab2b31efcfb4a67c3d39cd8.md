
# B-Puzzle 5.1

> 原文：`[www.pediy.com/kssd/pediy06/pediy6170.htm](https://www.pediy.com/kssd/pediy06/pediy6170.htm)`

`B-Puzzle 5.1` 完全破解

早就在以前的《论坛精华》看到大哥们介绍 `ADC` 公司的游戏，说是非常适合初学者练手。今天我就在网上下了个最新版的 `B-Puzzle`，(所谓破鼓万人擂:)不过这个拼图游戏还挺好玩的)果然比较容易就看到了明码，但我们就到此为止了吗?不，我们的目标 是——没有蛀牙!~~~~不对不对，我不是卖牙膏的，我们的目标是——注册机!

废话少说，单击 `Enter Reg Code`，输入 `NAME: RoBa CODE:87654321`，下断追踪至此:

```
:0040C803 8B0D9C374A00            mov ecx, dword ptr [004A379C]
:0040C809 8B01                    mov eax, dword ptr [ecx]
:0040C80B 8B80D8020000            mov eax, dword ptr [eax+000002D8]
:0040C811 E8AA9E0400              call 004566C0
:0040C816 66C745B40800            mov [ebp-4C], 0008
:0040C81C 8B45A0                  mov eax, dword ptr [ebp-60]
:0040C81F E860080000              call 0040D084
:0040C824 66C745B42C00            mov [ebp-4C], 002C
:0040C82A 8D45F4                  lea eax, dword ptr [ebp-0C]
:0040C82D E8D24FFFFF              call 00401804
:0040C832 50                      push eax
:0040C833 FF45C0                  inc [ebp-40]
:0040C836 8D45FC                  lea eax, dword ptr [ebp-04]
:0040C839 E84258FFFF              call 00402080
:0040C83E 8BD0                    mov edx, eax
:0040C840 8B45A0                  mov eax, dword ptr [ebp-60]
:0040C843 59                      pop ecx
:0040C844 E843060000              call 0040CE8C                      <--关键 CALL
:0040C849 8D55F4                  lea edx, dword ptr [ebp-0C]       <--下 d *(ebp-c)可看到真码
:0040C84C 8D45F8                  lea eax, dword ptr [ebp-08]       <--下 d *(ebp-c)可看到假码
:0040C84F E808B10800              call 0049795C
:0040C854 50                      push eax
:0040C855 FF4DC0                  dec [ebp-40]
:0040C858 8D45F4                  lea eax, dword ptr [ebp-0C]
:0040C85B BA02000000              mov edx, 00000002
:0040C860 E827B00800              call 0049788C
:0040C865 59                      pop ecx
:0040C866 84C9                    test cl, cl
:0040C868 0F8466010000            je 0040C9D4                         <--这里 r fl z 就会"注册成功"
:0040C86E B201                    mov dl, 01
:0040C870 A15C1E4300              mov eax, dword ptr [00431E5C]
```

跟进 `40C844` 处的 `CALL`:

```
* Referenced by a CALL at Addresses:
  :0040C844  , :0040CD4E
  |
:0040CE8C 55                      push ebp
:0040CE8D 8BEC                    mov ebp, esp
:0040CE8F 81C4ACFDFFFF            add esp, FFFFFDAC
:0040CE95 894DF8                  mov dword ptr [ebp-08], ecx
:0040CE98 8955CC                  mov dword ptr [ebp-34], edx
:0040CE9B 8945D0                  mov dword ptr [ebp-30], eax
:0040CE9E B864CF4900              mov eax, 0049CF64
:0040CEA3 E82CF60700              call 0048C4D4
:0040CEA8 FF75CC                  push [ebp-34]
:0040CEAB 8D95ACFEFFFF            lea edx, dword ptr [ebp+FFFFFEAC]
:0040CEB1 52                      push edx
:0040CEB2 E821F30700              call 0048C1D8
:0040CEB7 83C408                  add esp, 00000008
:0040CEBA 8D8DACFEFFFF            lea ecx, dword ptr [ebp+FFFFFEAC]
:0040CEC0 51                      push ecx
:0040CEC1 E842F30700              call 0048C208
:0040CEC6 59                      pop ecx
:0040CEC7 83F808                  cmp eax, 00000008                   <--将长度与 8 比较
:0040CECA 7714                    ja 0040CEE0                         <--大于就跳过下面一段

* Possible StringData Ref from Data Obj ->"        "                  <--hehe,8 个空格
                                  |
:0040CECC 681DC84900              push 0049C81D                         <-这是长度小于 8 的处理
:0040CED1 8D85ACFEFFFF            lea eax, dword ptr [ebp+FFFFFEAC]
:0040CED7 50                      push eax
:0040CED8 E81FF20700              call 0048C0FC                         <-出来后在 Name 后加了 8 个空格
:0040CEDD 83C408                  add esp, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040CECA(C)
  |

* Reference To: KERNEL32.GetTickCount, Ord:0000h
                                  |
:0040CEE0 E873B00800              Call 00497F58
:0040CEE5 8945C8                  mov dword ptr [ebp-38], eax
:0040CEE8 C745C401000000          mov [ebp-3C], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040CEF6(C)
  |

:0040CEEF FF45C4                  inc [ebp-3C]                          -\  这三句在 do what?
:0040CEF2 837DC464                cmp dword ptr [ebp-3C], 00000064      |->转圈 100 次??
:0040CEF6 7CF7                    jl 0040CEEF                           -/  你不会一直按 F10 吧:)

* Reference To: KERNEL32.GetTickCount, Ord:0000h
                                  |
:0040CEF8 E85BB00800              Call 00497F58
:0040CEFD 2B45C8                  sub eax, dword ptr [ebp-38]
:0040CF00 3DE8030000              cmp eax, 000003E8
:0040CF05 760D                    jbe 0040CF14
:0040CF07 8B15B4374A00            mov edx, dword ptr [004A37B4]
:0040CF0D 8B02                    mov eax, dword ptr [edx]
:0040CF0F E89CEE0300              call 0044BDB0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040CF05(C)
  |

:0040CF14 33D2                    xor edx, edx
:0040CF16 8955C0                  mov dword ptr [ebp-40], edx
:0040CF19 33C9                    xor ecx, ecx
:0040CF1B 894DB8                  mov dword ptr [ebp-48], ecx          <--上面一段乱烘烘的不管它

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040CF90(C)                                                           <--循环开始
  |

:0040CF1E FF353CC64900            push dword ptr [0049C63C]               <--这里一个串"PUZZLEVERO"
:0040CF24 E8DFF20700              call 0048C208                           <--eax=A 串的长度??
:0040CF29 59                      pop ecx
:0040CF2A 50                      push eax
:0040CF2B 8B45B8                  mov eax, dword ptr [ebp-48]            <--eax=[ebp-48]
:0040CF2E 5A                      pop edx                                   <--edx=A
:0040CF2F 8BCA                    mov ecx, edx                              <--ecx=A
:0040CF31 33D2                    xor edx, edx
:0040CF33 F7F1                    div ecx                                   <--eax=eax div ecx
                                                    <--edx=eax mod ecx
                                                    <--这里的计算绕来绕去每次 edx+2

* Possible StringData Ref from Data Obj ->"PUZZLEVERO"
                                  |
:0040CF35 A13CC64900              mov eax, dword ptr [0049C63C]             <--eax 为串的开始位置
:0040CF3A 8A0410                  mov al, byte ptr [eax+edx]               <--得到第 edx 个字符
:0040CF3D 8B55B8                  mov edx, dword ptr [ebp-48]
:0040CF40 328415ACFEFFFF          xor al, byte ptr [ebp+edx-00000154]       <--将串与 Name 中第 edx 个字符异或
:0040CF47 0FBEC8                  movsx ecx, al
:0040CF4A 894DB4                  mov dword ptr [ebp-4C], ecx
:0040CF4D 6A0A                    push 0000000A
:0040CF4F 8D45BC                  lea eax, dword ptr [ebp-44]
:0040CF52 50                      push eax
:0040CF53 FF75B4                  push [ebp-4C]
:0040CF56 E8BD380800              call 00490818                             <--关键的 CALL
:0040CF5B 83C40C                  add esp, 0000000C
:0040CF5E 807DBD00                cmp byte ptr [ebp-43], 00                 <--是否为 0;即只有一位
:0040CF62 7504                    jne 0040CF68
:0040CF64 B231                    mov dl, 31                                <--如果为 0 则给'1'
:0040CF66 EB03                    jmp 0040CF6B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0040CF62(C)
  |

:0040CF68 8A55BD                  mov dl, byte ptr [ebp-43]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
