
# 社区游戏伴侣 V1.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6016.htm`](https://www.pediy.com/kssd/pediy06/pediy6016.htm)

- 软件名称： 社区游戏伴侣 V1.0
- 软件大小： 213 KB
- 应用平台： Win9x
- 软件类别： 游戏记牌器
- 软件介绍：联众，边锋游戏牌类记牌器，注册后全部功能可用。
- 破解工具： `ollydbg 1.08`， `W32DASM10`， `UltraEdit8.0`， `AspackDie`， `fi2.5`
- 破解方法： 注册算法

朋友玩联众游戏，说有记牌器《****伴侣 1.1》要注册才能玩够级，让我看看。他的注册方法有些独特，注册表验证，而且必须连接联众后才验证，分不同地方验证，而且必须用游戏 `ID`，这就是说你只能用一个用户 `ID` 玩游戏。上网也没查到有注册机。自己动手吧。

先脱壳，`aspack2.12`，用 `AspackDie` 好脱。

用 `W32DASM` 反汇编，查找可疑字符串等。再用 `ollydbg 1.08` 调试，断点就好设了。

--------------------------------------------------------------------------------------

```assembly
0045D16C  /. 55             PUSH EBP
0045D16D  |. 8BEC           MOV EBP,ESP
0045D16F  |. B9 0C000000    MOV ECX,0C              ; ecx＝0x0C
0045D174  |> 6A 00           /PUSH 0                  ; 初始化
0045D176  |. 6A 00          |PUSH 0
0045D178  |. 49            |DEC ECX
0045D179  |. 75 F9          \\JNZ SHORT UNPACKED.0045D174
0045D17B  |. 53             PUSH EBX                  ; ebx=011ca3f8,不知道什么用
0045D17C  |. 56             PUSH ESI
0045D17D  |. 8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
0045D180  |. 33C0           XOR EAX,EAX
0045D182  |. 55             PUSH EBP
0045D183  |. 68 A1D34500    PUSH UNPACKED.0045D3A1
0045D188  |. 64:FF30        PUSH DWORD PTR FS:[EAX]
0045D18B  |. 64:8920        MOV DWORD PTR FS:[EAX],ESP
0045D18E  |. 8D55 C8        LEA EDX,DWORD PTR SS:[EBP-38]
0045D191  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0045D194  |. 8B80 08030000  MOV EAX,DWORD PTR DS:[EAX+308]
0045D19A  |. E8 5996FDFF    CALL UNPACKED.004367F8    ; 取假注册码：12345678
0045D19F  |. 8B45 C8        MOV EAX,DWORD PTR SS:[EBP-38]    ; 给 eax 地址处存放假注册码 12345678
0045D1A2  |. 33D2           XOR EDX,EDX
0045D1A4  |. E8 07B9FAFF    CALL UNPACKED.00408AB0
0045D1A9  |. 8BC8           MOV ECX,EAX              ; eax 的值给 ecx
0045D1AB  |. 81F9 80969800  CMP ECX,989680          ; 0x989680＝10000000，ecx 大于就跳。看寄存器窗口 ecx 双击看到 12345678
0045D1B1  |. 7D 0F          JGE SHORT UNPACKED.0045D1C2
0045D1B3  |. B8 B8D34500    MOV EAX,UNPACKED.0045D3B8
0045D1B8  |. E8 3F31FDFF    CALL UNPACKED.004302FC
0045D1BD  |. E9 92010000    JMP UNPACKED.0045D354
0045D1C2  |> 8D45 C4        LEA EAX,DWORD PTR SS:[EBP-3C]
0045D1C5  |. 50             PUSH EAX
0045D1C6  |. 8D55 C0        LEA EDX,DWORD PTR SS:[EBP-40]
0045D1C9  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0045D1CC  |. 8B80 08030000  MOV EAX,DWORD PTR DS:[EAX+308]
0045D1D2  |. E8 2196FDFF    CALL UNPACKED.004367F8
0045D1D7  |. 8B45 C0        MOV EAX,DWORD PTR SS:[EBP-40]
0045D1DA  |. B9 04000000    MOV ECX,4
0045D1DF  |. 33D2           XOR EDX,EDX
0045D1E1  |. E8 B277FAFF    CALL UNPACKED.00404998    ; 取假注册码的前 4 位，1234
0045D1E6  |. 8B45 C4        MOV EAX,DWORD PTR SS:[EBP-3C]    ; 给 eax 地址处存放假注册码 eax=1234
0045D1E9  |. E8 86B8FAFF    CALL UNPACKED.00408A74
0045D1EE  |. 8945 F0        MOV DWORD PTR SS:[EBP-10],EAX    ; eax=1234 存 ebp-10
0045D1F1  |. 8D55 B8        LEA EDX,DWORD PTR SS:[EBP-48]
0045D1F4  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0045D1F7  |. 8B80 04030000  MOV EAX,DWORD PTR DS:[EAX+304]
0045D1FD  |. E8 F695FDFF    CALL UNPACKED.004367F8    ; 取用户名 laoqian
0045D202  |. 8B45 B8        MOV EAX,DWORD PTR SS:[EBP-48]    ; eax=7
0045D205  |. 8D55 BC        LEA EDX,DWORD PTR SS:[EBP-44]
0045D208  |. E8 0BB5FAFF    CALL UNPACKED.00408718
0045D20D  |. 8B45 BC        MOV EAX,DWORD PTR SS:[EBP-44]    ; 给 eax 地址处存放 laoqian
0045D210  |. 8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]    ; eax=7
0045D213  |. E8 DCB5FAFF    CALL UNPACKED.004087F4
0045D218  |. 8D45 B4        LEA EAX,DWORD PTR SS:[EBP-4C]
0045D21B  |. 50             PUSH EAX
0045D21C  |. 8D55 B0        LEA EDX,DWORD PTR SS:[EBP-50]
0045D21F  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0045D222  |. 8B80 08030000  MOV EAX,DWORD PTR DS:[EAX+308]
0045D228  |. E8 CB95FDFF    CALL UNPACKED.004367F8
0045D22D  |. 8B45 B0        MOV EAX,DWORD PTR SS:[EBP-50]    ; eax=8,12345678
0045D230  |. B9 04000000    MOV ECX,4
0045D235  |. BA 05000000    MOV EDX,5
0045D23A  |. E8 5977FAFF    CALL UNPACKED.00404998    ; 取假注册码的后 4 位：5678
0045D23F  |. 8B55 B4        MOV EDX,DWORD PTR SS:[EBP-4C]    ; edx=5678
0045D242  |. 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
0045D245  |. E8 FE74FAFF    CALL UNPACKED.00404748    ; 合并 laoqian5678 为字符串
0045D24A  |. 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]    ; 存 eax
0045D24D  |. E8 EE74FAFF    CALL UNPACKED.00404740    ; 取“假用户名加假注册码后四位”的长度
0045D252  |. 8945 F8        MOV DWORD PTR SS:[EBP-8],EAX    ; eax=0xB，存的长度
0045D255  |. 8D45 CD        LEA EAX,DWORD PTR SS:[EBP-33]
0045D258  |. 8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]    ; edx=laoqian5678 为字符串，原来是 5678
0045D25B  |. E8 F0BBFAFF    CALL UNPACKED.00408E50
0045D260  |. BB DE040000    MOV EBX,4DE              ; 令 ebx＝0x4de(1246）
0045D265  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]    ; 取假用户名加假注册码后四位的长度 B 给 eax
0045D268  |. 48             DEC EAX                  ; eax-1
0045D269  |. 85C0           TEST EAX,EAX             ; 测试
0045D26B  |. 7C 37          JL SHORT UNPACKED.0045D2A4
0045D26D  |. 40             INC EAX                  ; eax+1 还原
0045D26E  |. 8945 EC        MOV DWORD PTR SS:[EBP-14],EAX    ; 取假用户名加假注册码后四位的长度 B 赋值存 ebp-4
0045D271  |. 33C9           XOR ECX,ECX              ; 清零 ecx
0045D273  |. 8D45 CD        LEA EAX,DWORD PTR SS:[EBP-33]    ; 装入“假用户名加假注册码后四位”laoqian5678
0045D276  |> 8BD1           /MOV EDX,ECX             ; ecx=edx
0045D278  |. 0FAFD1         |IMUL EDX,ECX            ; edx＝edx*ecx 整数乘法
0045D27B  |. 03DA           |ADD EBX,EDX             ; ebx＝ebx+edx
0045D27D  |. 33D2           |XOR EDX,EDX             ; edx＝0
0045D27F  |. 8A10           |MOV DL,BYTE PTR DS:[EAX]     ; 取依次 eax“laoqian5678”字符串的第 n 个 ASCII 值
0045D281  |. 0FAFD1         |IMUL EDX,ECX            ; edx＝edx*ecx
0045D284  |. 03DA           |ADD EBX,EDX             ; ebx＝ebx+edx
0045D286  |. 8B55 F8        |MOV EDX,DWORD PTR SS:[EBP-8]    ; 取假用户名加假注册码后四位的长度给 edx＝B
0045D289  |. 4A             |DEC EDX                 ; edx＝edx－1
0045D28A  |. 83FA 00        |CMP EDX,0               ; 比较是否小于 0
0045D28D  |. 7C 0E          |JL SHORT UNPACKED.0045D29D     ； 循环到 0045D276
0045D28F  |> 8D1C19         /LEA EBX,DWORD PTR DS:[ECX+EBX]     ; ebx＝ebx+ecx
0045D292  |. 0FB630        ||MOVZX ESI,BYTE PTR DS:[EAX]       ; 依次传送 eax“laoqian5678”字符串的第 n 个 ASCII 值给 esi
0045D295  |. 03DE          ||ADD EBX,ESI            ; ebx＝ebx+esi
0045D297  |. 4A             ||DEC EDX                ; edx＝edx－1
0045D298  |. 83FA FF        ||CMP EDX,-1             ； 比较是否小于-1
0045D29B  |. 75 F2          |\JNZ SHORT UNPACKED.0045D28F     ; 循环 0045D28F
0045D29D  |> 41             |INC ECX                 ; ecx+1
0045D29E  |. 40             |INC EAX                 ; eax+1 地址
0045D29F  |. FF4D EC        |DEC DWORD PTR SS:[EBP-14]        ; “假用户名加假注册码后四位”的长度-1
```
