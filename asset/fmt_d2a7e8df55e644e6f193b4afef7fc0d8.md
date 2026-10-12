
#### 00401D95 - 00401DF2

```
:00401D95 751D                      jne 00401DB4                <== 不相等则 GAME OVER，大家将此处 NOP 掉，看会发生什么有趣的事情

:00401D97 50                        push eax

:00401D98 8D8500FEFFFF              lea eax, dword ptr [ebp+FFFFFE00]

:00401D9E 50                        push eax

:00401D9F 8D8500FAFFFF              lea eax, dword ptr [ebp+FFFFFA00]

* Possible StringData Ref from Data Obj ->"----- LICENSED TO: %s (%d) -----"

                                        |

:00401DA5 6834214400                  push 00442134

:00401DAA 50                        push eax

* Reference To: USER32.wsprintfA, Ord:02ACh

                                        |

:00401DAB FF1568E34300              Call dword ptr [0043E368]    <== 打印授权信息

:00401DB1 83C410                    add esp, 00000010

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00401D4C(C), :00401D63(C), :00401D95(C)

|

:00401DB4 80BD00FAFFFF00            cmp byte ptr [ebp+FFFFFA00], 00

:00401DBB 5F                        pop edi

:00401DBC 5E                        pop esi

:00401DBD 5B                        pop ebx

:00401DBE 751C                      jne 00401DDC

:00401DC0 6A00                      push 00000000

:00401DC2 6A00                      push 00000000

* Possible Reference to String Resource ID=00018: "UNREGISTERED SHAREWARE VERSION - PLEASE REGISTER"

                                        |

:00401DC4 6A12                      push 00000012

:00401DC6 E8C5A10200                call 0042BF90

:00401DCB 50                        push eax

:00401DCC 8D8500FAFFFF              lea eax, dword ptr [ebp+FFFFFA00]

:00401DD2 50                        push eax

* Reference To: USER32.wsprintfA, Ord:02ACh

                                        |

:00401DD3 FF1568E34300              Call dword ptr [0043E368]    <== 打印未授权信息

:00401DD9 83C414                    add esp, 00000014           <== 向下很长一段与分析无关

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401DBE(C)

|

:00401DDC 8D8500FAFFFF              lea eax, dword ptr [ebp+FFFFFA00]

:00401DE2 50                        push eax

* Possible Reference to Dialog: DialogID_009C, CONTROL_ID:0483, "License"

                                        |

:00401DE3 6883040000                push 00000483

:00401DE8 FF7508                    push [ebp+08]

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch

                                        |

:00401DEB FF1534E34300              Call dword ptr [0043E334]

:00401DF1 C9                        leave

:00401DF2 C3                        ret

.........

* Possible StringData Ref from Data Obj ->"!!!"

                                        |

:00401EAC 6858214400                push 00442158

:00401EB1 50                        push eax

:00401EB2 E829330300                call 004351E0

:00401EB7 59                        pop ecx

:00401EB8 8D8580FBFFFF              lea eax, dword ptr [ebp+FFFFFB80]

:00401EBE 59                        pop ecx

:00401EBF 57                        push edi

:00401EC0 50                        push eax

:00401EC1 E8FA330300                call 004352C0

:00401EC6 59                        pop ecx

:00401EC7 8D840580FBFFFF            lea eax, dword ptr [ebp+eax-00000480]

:00401ECE 50                        push eax

:00401ECF 53                        push ebx

:00401ED0 FF7508                    push [ebp+08]

:00401ED3 FFD6                      call esi

:00401ED5 8D8580FBFFFF              lea eax, dword ptr [ebp+FFFFFB80]

* Possible StringData Ref from Data Obj ->"winamp.ini"

                                        |

:00401EDB 687C214400                push 0044217C

:00401EE0 50                        push eax

* Possible StringData Ref from Data Obj ->"RegisteredTo"

                                        |

:00401EE1 686C214400                push 0044216C

* Possible StringData Ref from Data Obj ->"WinampRegInfo"

                                        |

:00401EE6 685C214400                push 0044215C

* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:02E5h

                                        |

:00401EEB FF1530E14300              Call dword ptr [0043E130]    <== 这一段会在点击“OK”时执行，向 ini 文件写 RI

* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                        |

:00401EF1 6A01                      push 00000001

:00401EF3 FF7508                    push [ebp+08]

* Reference To: USER32.EndDialog, Ord:00B9h

                                        |

:00401EF6 FF151CE44300              Call dword ptr [0043E41C]

:00401EFC 8B7D10                    mov edi, dword ptr [ebp+10]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401E6C(C)

|

* Possible Reference to Dialog: DialogID_00BF, CONTROL_ID:048B, ""

                                        |

:00401EFF B88B040000                mov eax, 0000048B

:00401F04 663BF8                    cmp di, ax

:00401F07 7405                      je 00401F0E

:00401F09 663BFB                    cmp di, bx

:00401F0C 7552                      jne 00401F60
```

对关键 `call 004252D1` 的分析请见(三)

#### WinAmp V2.11 的注册机制分析(二)（初学者必读）

转载本文请注明出处，并保持文章的完整性。本文不得用于任何商业场合，仅供学习和参考。

**作者**：`Fpc`（感谢直接和间接提供帮助的所有朋友）

**目的**：分析注册机制

**工具**：`SI 4.05`, `Wdasm 8.93`, `Hedit 2.0`

程序段（2），输入注册信息的每次输入时执行：根据 `Rn` 计算 `Rc`，若 `Irc=Rc` 则激活“`OK`”表示可以将 `RI` 写入“`winamp.ini`”文件中

#### 00401F0E - 00401F50

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401F07(C)

|

:00401F0E C1EF10                    shr edi, 10

:00401F11 6681FF0003                cmp di, 0300

:00401F16 7548                      jne 00401F60

:00401F18 8B7D08                    mov edi, dword ptr [ebp+08]

:00401F1B 8D4D80                    lea ecx, dword ptr [ebp-80]

:00401F1E 6880000000                push 00000080

:00401F23 51                        push ecx

:00401F24 50                        push eax

:00401F25 57                        push edi

:00401F26 FFD6                      call esi                    <== 读 Name 文本框，将取得的 Rn 保存在 Ebp-80 处

:00401F28 6A00                      push 00000000

:00401F2A 6A00                      push 00000000

:00401F2C 53                        push ebx

:00401F2D 57                        push edi

* Reference To: USER32.GetDlgItemInt, Ord:0103h

                                        |

:00401F2E FF152CE34300              Call dword ptr [0043E32C]    <== 读 Reg#文本框，返回值在 Eax 中

:00401F34 8BF0                      mov esi, eax

:00401F36 8D4580                    lea eax, dword ptr [ebp-80]

:00401F39 50                        push eax

:00401F3A E892330200                call 004252D1               <== !! 关键 Call，由 Rn 计算得出 Rc，保存在 Eax。具体分析请看（三）

:00401F3F 3BC6                      cmp eax, esi                <== 比较 Rc 与 Irc

:00401F41 59                        pop ecx

:00401F42 750A                      jne 00401F4E

:00401F44 807D8000                  cmp byte ptr [ebp-80], 00

:00401F48 7404                      je 00401F4E

* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                        |

:00401F4A 6A01                      push 00000001               <== “OK”为 Enable

:00401F4C EB02                      jmp 00401F50

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00401F42(C), :00401F48(C)

|

:00401F4E 6A00                      push 00000000               <== “OK”为 Disable

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401F4C(U)

|

* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                        |

:00401F50 6A01                      push 00000001
```
