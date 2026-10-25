
# Instant Source 注册算法分析+注册器源码

标 题:IrfanView

发信人:东南破佛

时 间:2003年9月23日 02:57

详细信息:

Instant Source是一个IE插件，可以直接看到网页中的源码和Script。

网上能找到现成的注册器，不过破解前我没去搜索 @\_@

这个软件没加壳，没暗桩，明码比较注册码，非常适合刚入门的cracker来练练手，作者真是大善人啊。

我在论坛上搜索了一下，似乎还没有人写过这个软件的破解，可能觉得太简单了，而不屑一写吧。

本着回馈论坛，提携菜鸟，我不入地狱，谁入地狱的大无畏精神，特写此手记。

Instant Source没有可执行文件，只有一个`isrc.Dll`，所以可以用`OllyDbg`非常强大的`Attach`功能来跟踪。

进入IE后，打开Instant source的注册窗口，随便输些东东，记住弹出的对话框说些什么：

"Registration code or user name is invalid. Please check all fields and try again!"

当然不用记这么多，记一些比较有代表的短语就够了。

打开`OllyDbg`, `Attach` IE, `View->Executable modules(Alt-E)`, 找到`isrc.dll`，进入其领空，

在弹出菜单中选`Search for->All referenced text string`, 查找刚才对话框的内容，

找到其判断注册码的地方：

```
0258AA7F  |.  52            PUSH EDX                                        ; /String2
0258AA80  |.  50            PUSH EAX                                        ; |String1
0258AA81  |.  FF15 64515902 CALL NEAR DWORD PTR DS:[<&KERNEL32.lstrcpyA>]   ; lstrcpyA
0258AA87  |.  8B3D 5C515902 MOV EDI, DWORD PTR DS:[<&KERNEL32.lstrcatA>]    ;  KERNEL32.lstrcatA
0258AA8D  |.  8D4C24 2C     LEA ECX, DWORD PTR SS:[ESP+2C]
0258AA91  |.  8D5424 38     LEA EDX, DWORD PTR SS:[ESP+38]
0258AA95  |.  51            PUSH ECX                                        ; /StringToAdd
0258AA96  |.  52            PUSH EDX                                        ; |ConcatString
0258AA97  |.  FFD7          CALL NEAR EDI                                   ; lstrcat
0258AA99  |.  8D4424 20     LEA EAX, DWORD PTR SS:[ESP+20]
0258AA9D  |.  8D4C24 38     LEA ECX, DWORD PTR SS:[ESP+38]
0258AAA1  |.  50            PUSH EAX                                        ; /StringToAdd
0258AAA2  |.  51            PUSH ECX                                        ; |ConcatString
0258AAA3  |.  FFD7          CALL NEAR EDI                                   ; lstrcat
0258AAA5  |.  8D5424 08     LEA EDX, DWORD PTR SS:[ESP+8]
0258AAA9  |.  8D4424 38     LEA EAX, DWORD PTR SS:[ESP+38]
0258AAAD  |.  52            PUSH EDX                                        ; /StringToAdd
0258AAAE  |.  50            PUSH EAX                                        ; |ConcatString
0258AAAF  |.  FFD7          CALL NEAR EDI                                   ; lstrcat
0258AAB1  |.  8D8C24 A00000>LEA ECX, DWORD PTR SS:[ESP+A0]
0258AAB8  |.  8D5424 6C     LEA EDX, DWORD PTR SS:[ESP+6C]
0258AABC  |.  51            PUSH ECX
0258AABD  |.  68 805D5902   PUSH isrc.02595D80                              ;  ASCII "2I$q1[05)&Ew\*\_p["
0258AAC2  |.  52            PUSH EDX                                        ;  name
0258AAC3  |.  E8 E8FDFFFF   CALL isrc.0258A8B0            ;  \*\*\*\*\* 根据name算注册码 \*\*\*\*\*
0258AAC8  |.  83C4 0C       ADD ESP, 0C
0258AACB  |.  8D4424 38     LEA EAX, DWORD PTR SS:[ESP+38]
0258AACF  |.  8D8C24 A00000>LEA ECX, DWORD PTR SS:[ESP+A0]
0258AAD6  |.  50            PUSH EAX                                        ; /input sn
0258AAD7  |.  51            PUSH ECX                                        ; |real sn
0258AAD8  |.  FF15 84515902 CALL NEAR DWORD PTR DS:[<&KERNEL32.lstrcmpiA>]  ; lstrcmpiA
0258AADE  |.  85C0          TEST EAX, EAX
0258AAE0  |.  6A 40         PUSH 40                                         ; /Style = MB\_OK|MB\_ICONASTERISK|MB\_APPLMODAL
0258AAE2  |.  75 31         JNZ SHORT isrc.0258AB15                         ; |
0258AAE4  |.  68 A4A85902   PUSH isrc.0259A8A4                              ; |Title = "Registration"
0258AAE9  |.  68 64A85902   PUSH isrc.0259A864                              ; |Text = "Registration succeeded. Thank you for choosing Instant Source!"
0258AAEE  |.  56            PUSH ESI                                        ; |hOwner
0258AAEF  |.  FF15 88535902 CALL NEAR DWORD PTR DS:[<&USER32.MessageBoxA>]  ; MessageBoxA
0258AAF5  |.  8D5424 38     LEA EDX, DWORD PTR SS:[ESP+38]
0258AAF9  |.  8D4424 6C     LEA EAX, DWORD PTR SS:[ESP+6C]
0258AAFD  |.  52            PUSH EDX
0258AAFE  |.  50            PUSH EAX
0258AAFF  |.  E8 8CFEFFFF   CALL isrc.0258A990
0258AB04  |.  83C4 08       ADD ESP, 8
0258AB07  |.  B8 01000000   MOV EAX, 1
0258AB0C  |.  5F            POP EDI
0258AB0D  |.  5E            POP ESI
0258AB0E  |.  81C4 CC000000 ADD ESP, 0CC
0258AB14  |.  C3            RETN
0258AB15  |>  68 50A85902   PUSH isrc.0259A850                              ; |Title = "Registration error"
0258AB1A  |.  68 FCA75902   PUSH isrc.0259A7FC                              ; |Text = "Registration code or user name is invalid. Please check all fields and try again!"
0258AB1F  |.  56            PUSH ESI                                        ; |hOwner
0258AB20  |.  FF15 88535902 CALL NEAR DWORD PTR DS:[<&USER32.MessageBoxA>]  ; MessageBoxA
0258AB26  |.  68 2C010000   PUSH 12C                                        ; /Timeout = 300. ms
0258AB2B  |.  FF15 58515902 CALL NEAR DWORD PTR DS:[<&KERNEL32.Sleep>]      ; Sleep
0258AB31  |.  5F            POP EDI
0258AB32  |.  33C0          XOR EAX, EAX
0258AB34  |.  5E            POP ESI
0258AB35  |.  81C4 CC000000 ADD ESP, 0CC
0258AB3B  .  C3            RETN
0258AB3C      90            NOP
```

可以看到程序先把输入的4段注册码合并起来，然后在`0258AAC3`处的`Call 0258A8B0`是关键，它会根据`name`算注册码，接着用真的注册码与输入的注册码相比较。

看看`0258A8B0`：

```
0258A8B0  /$  53            PUSH EBX
0258A8B1  |.  8B5C24 0C     MOV EBX, DWORD PTR SS:[ESP+C]
0258A8B5  |.  55            PUSH EBP
0258A8B6  |.  56            PUSH ESI
0258A8B7  |.  57            PUSH EDI
0258A8B8  |.  8B3D 28515902 MOV EDI, DWORD PTR DS:[<&KERNEL32.lstrlenA>]    ;  KERNEL32.lstrlenA
0258A8BE  |.  53            PUSH EBX                                        ; /"2I$q1[05)&Ew\*\_p["
0258A8BF  |.  FFD7          CALL NEAR EDI                                   ; lstrlen
0258A8C1  |.  8BF0          MOV ESI, EAX
0258A8C3  |.  8B4424 14     MOV EAX, DWORD PTR SS:[ESP+14]
0258A8C7  |.  50            PUSH EAX                                        ; /name
0258A8C8  |.  897424 1C     MOV DWORD PTR SS:[ESP+1C], ESI                  ; |
0258A8CC  |.  FFD7          CALL NEAR EDI                                   ; lstrlenA
0258A8CE  |.  8BE8          MOV EBP, EAX
0258A8D0  |.  85ED          TEST EBP, EBP
0258A8D2  |.  75 0C         JNZ SHORT isrc.0258A8E0
0258A8D4  |.  8B4424 1C     MOV EAX, DWORD PTR SS:[ESP+1C]
0258A8D8  |.  5F            POP EDI
0258A8D9  |.  5E            POP ESI
0258A8DA  |.  5D            POP EBP
0258A8DB  |.  C600 00       MOV BYTE PTR DS:[EAX], 0
0258A8DE  |.  5B            POP EBX
0258A8DF  |.  C3            RETN
0258A8E0  |>  8B7C24 1C     MOV EDI, DWORD PTR SS:[ESP+1C]
0258A8E4  |.  53            PUSH EBX                                        ; /"2I$q1[05)&Ew\*\_p["
0258A8E5  |.  57            PUSH EDI                                        ; |String1
0258A8E6  |.  FF15 64515902 CALL NEAR DWORD PTR DS:[<&KERNEL32.lstrcpyA>]   ; lstrcpyA
0258A8EC  |.  3BEE          CMP EBP, ESI
0258A8EE  |.  8BC5          MOV EAX, EBP
0258A8F0  |.  7F 02         JG SHORT isrc.0258A8F4
0258A8F2  |.  8BC6          MOV EAX, ESI
0258A8F4  |>  33C9          XOR ECX, ECX
0258A8F6  |.  894424 1C     MOV DWORD PTR SS:[ESP+1C], EAX
0258A8FA  |.  85C0          TEST EAX, EAX
0258A8FC  |.  7E 39         JLE SHORT isrc.0258A937
0258A8FE  |.  EB 04         JMP SHORT isrc.0258A904
0258A900  |>  8B7424 18     /MOV ESI, DWORD PTR SS:[ESP+18]
0258A904  |>  8BC1           MOV EAX, ECX
0258A906  |.  8B5C24 14     |MOV EBX, DWORD PTR SS:[ESP+14]
0258A90A  |.  99            |CDQ
0258A90B  |.  F7FE          |IDIV ESI
0258A90D  |.  8BC1          |MOV EAX, ECX
0258A90F  |.  8BF2          |MOV ESI, EDX
0258A911  |.  99            |CDQ
0258A912  |.  F7FD          |IDIV EBP
0258A914  |.  33C0          |XOR EAX, EAX
0258A916  |.  8A041A        |MOV AL, BYTE PTR DS:[EDX+EBX]                  ;  al=name[edx]
0258A919  |.  33D2          |XOR EDX, EDX
0258A91B  |.  8A143E        |MOV DL, BYTE PTR DS:[ESI+EDI]                  ;  dl=table[esi]   table="2I$q1[05)&Ew\*\_p["
0258A91E  |.  BB 19000000   |MOV EBX, 19
```
