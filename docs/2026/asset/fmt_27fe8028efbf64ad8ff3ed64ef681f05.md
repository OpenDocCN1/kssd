
```
:00401EEB FF1530E14300            Call dword ptr [0043E130]            <== 这一段会在点击“OK”时执行，向ini文件写RI

\* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                  |

:00401EF1 6A01                    push 00000001

:00401EF3 FF7508                  push [ebp+08]

\* Reference To: USER32.EndDialog, Ord:00B9h

                                   |

:00401EF6 FF151CE44300            Call dword ptr [0043E41C]

:00401EFC 8B7D10                  mov edi, dword ptr [ebp+10]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401E6C(C)

 |

\* Possible Reference to Dialog: DialogID\_00BF, CONTROL\_ID:048B, ""

                                  |

:00401EFF B88B040000              mov eax, 0000048B

:00401F04 663BF8                  cmp di, ax

:00401F07 7405                    je 00401F0E

:00401F09 663BFB                  cmp di, bx

:00401F0C 7552                    jne 00401F60
```

对关键`Call 004252D1`的分析请见(三)

# WinAmp V2.11的注册机制分析(二)（初学者必读）

转载本文请注明出处，并保持文章的完整性。本文不得用于任何商业场合，仅供学习和参考。

作者： Fpc （感谢直接和间接提供帮助的所有朋友）

目的： 分析注册机制

工具： SI 4.05, Wdasm 8.93, Hedit 2.0

## 程序段（2）

输入注册信息的每次输入时执行：根据`Rn`计算`Rc`，若`Irc = Rc`则激活“OK”表示可以将`RI`写入`winamp.ini`文件中

```asm
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401F07(C)

 |

 :00401F0E C1EF10                  shr edi, 10

:00401F11 6681FF0003              cmp di, 0300

:00401F16 7548                    jne 00401F60

:00401F18 8B7D08                  mov edi, dword ptr [ebp+08]

:00401F1B 8D4D80                  lea ecx, dword ptr [ebp-80]

:00401F1E 6880000000              push 00000080

:00401F23 51                      push ecx

:00401F24 50                      push eax

:00401F25 57                      push edi

:00401F26 FFD6                    call esi                <== 读Name文本框，将取得的Rn保存在 Ebp-80处

:00401F28 6A00                    push 00000000

:00401F2A 6A00                    push 00000000

:00401F2C 53                      push ebx

:00401F2D 57                      push edi

\* Reference To: USER32.GetDlgItemInt, Ord:0103h

                                  |

:00401F2E FF152CE34300            Call dword ptr [0043E32C]        <== 读Reg#文本框，返回值在 Eax中

:00401F34 8BF0                    mov esi, eax

:00401F36 8D4580                  lea eax, dword ptr [ebp-80]

:00401F39 50                      push eax

:00401F3A E892330200              call 004252D1                <== !! 关键Call，由Rn计算得出Rc，保存在Eax。具体分析请看（三）

:00401F3F 3BC6                    cmp eax, esi                <== 比较Rc与Irc

:00401F41 59                      pop ecx

:00401F42 750A                    jne 00401F4E

:00401F44 807D8000                cmp byte ptr [ebp-80], 00

:00401F48 7404                    je 00401F4E

\* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                  |

:00401F4A 6A01                    push 00000001                <== “OK”为Enable

:00401F4C EB02                    jmp 00401F50

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00401F42(C), :00401F48(C)

 |

 :00401F4E 6A00                    push 00000000                <== “OK”为Disable

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401F4C(U)

 |

\* Possible Reference to String Resource ID=00001: "You must first exit Winamp before uninstalling."

                                  |

:00401F50 6A01                    push 00000001

:00401F52 57                      push edi

\* Reference To: USER32.GetDlgItem, Ord:0102h

                                   |

:00401F53 FF1574E34300            Call dword ptr [0043E374]

:00401F59 50                      push eax

\* Reference To: USER32.EnableWindow, Ord:00B7h

                                   |

:00401F5A FF1530E34300            Call dword ptr [0043E330]        <== 修改按钮状态

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00401E43(C), :00401F0C(C), :00401F16(C)

 |

 :00401F60 33C0                    xor eax, eax

:00401F62 5B                      pop ebx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00401E36(U)

 |

 :00401F63 5F                      pop edi

:00401F64 5E                      pop esi

:00401F65 C9                      leave

:00401F66 C21000                  ret 0010
```

## `Call 435150` 比较在`RI`中是否含有`"!!!"`

入口：`Ecx`指向`"!!!"+'\0'`

出口：错误则`eax=0`，正确则`eax`指向`RI`的`!!!123456`

```asm
\* Referenced by a CALL at Addresses:

 |    有N处调用，我用了 Echo y|Format C:/u

:00435150 8B4C2408                mov ecx, dword ptr [esp+08]    <== Ecx=442158，指向"!!!"+'\0'

:00435154 57                      push edi

:00435155 53                      push ebx

:00435156 56                      push esi

:00435157 8A11                    mov dl, byte ptr [ecx]        <== Dl取得第一个'!'

:00435159 8B7C2410                mov edi, dword ptr [esp+10]    <== Edi=Ebp-200=6DC4F8，即RI首址

:0043515D 84D2                    test dl, dl

:0043515F 7469                    je 004351CA                <== Dl=0证明比较完成，跳下去准备返回（怎么可能?!）

:00435161 8A7101                  mov dh, byte ptr [ecx+01]        <== 第二个'!'到Dh

:00435164 84F6                    test dh, dh

:00435166 744F                    je 004351B7

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004351A2(C), :004351B5(U)

 |

 :00435168 8BF7                    mov esi, edi

:0043516A 8B4C2414                mov ecx, dword ptr [esp+14]    <== Ecx=442158，指向"!!!"+'\0'

:0043516E 8A07                    mov al, byte ptr [edi]

:00435170 46                      inc esi

:00435171 38D0                    cmp al, dl                <== 看Al是否为'!'

:00435173 7415                    je 0043518A

:00435175 84C0                    test al, al

:00435177 740B                    je 00435184                <== Al为空则失败

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00435182(C)

 |

 :00435179 8A06                    mov al, byte ptr [esi]

:0043517B 46                      inc esi

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043518F(C)

 |

 :0043517C 38D0                    cmp al, dl

:0043517E 740A                    je 0043518A                <== 向下跳检查RI中是否含三个'!'

:00435180 84C0                    test al, al

:00435182 75F5                    jne 00435179                <== 直到Al取到'!'；若不跳则说明在遇到"!!!"之前，RI已经结束（RI中不含!!!）,失败

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00435177(C)

 |

 :00435184 5E                      pop esi

:00435185 5B                      pop ebx

:00435186 5F                      pop edi

:00435187 33C0                    xor eax, eax

:00435189 C3                      ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00435173(C), :0043517E(C)

 |

 :0043518A 8A06                    mov al, byte ptr [esi]        <== 从RI中!之后再读一字节

:0043518C 46                      inc esi

:0043518D 38F0                    cmp al, dh

:0043518F 75EB                    jne 0043517C                <== 若不等, 则在Rn中含'!'

:00435191 8D7EFF                  lea edi, dword ptr [esi-01]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004351B3(C)

 |

 :00435194 8A6102                  mov ah, byte ptr [ecx+02]

:00435197 84E4                    test ah, ah

:00435199 7428                    je 004351C3

:0043519B 8A06                    mov al, byte ptr [esi]        <== 从Rn中取第三个'!'

:0043519D 83C602                  add esi, 00000002

:004351A0 38E0                    cmp al, ah

:004351A2 75C4                    jne 00435168

:004351A4 8A4103                  mov al, byte ptr [ecx+03]        <== Al应取到0

:004351A7 84C0                    test al, al

:004351A9 7418                    je 004351C3                <== Al为空表示正常结束

:004351AB 8A66FF                  mov ah, byte ptr [esi-01]

:004351AE 83C102                  add ecx, 00000002            <== 若正确，这里不会被执行
```
