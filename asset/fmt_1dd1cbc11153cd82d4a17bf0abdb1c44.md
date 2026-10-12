
## Foxit Reader 3.1.4.1125 `launch` action 处理分析

```
0047689E  |. 6A 00         PUSH 0                                      ; |ModuleFileName = NULL
004768A0  |. FF15 64E59800 CALL DWORD PTR DS:[<&KERNEL32.CreateProcessA>]      ; \\CreateProcessA
004768A6  |. 8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]
004768AA  |. EB 26         JMP SHORT Foxit_Re.004768D2
004768AC  |> 8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]
004768B0  |. 85C0          TEST EAX,EAX
004768B2  |. 74 05         JE SHORT Foxit_Re.004768B9
004768B4  |. 83C0 0C       ADD EAX,0C
004768B7  |. EB 05         JMP SHORT Foxit_Re.004768BE
004768B9  |> B8 DC9CBF00   MOV EAX,Foxit_Re.00BF9CDC
004768BE  |> 6A 05         PUSH 5                                      ; /IsShown = 5
004768C0  |. 6A 00         PUSH 0                                      ; |DefDir = NULL
004768C2  |. 6A 00         PUSH 0                                      ; |Parameters = NULL
004768C4  |. 50           PUSH EAX                                    ; |FileName
004768C5  |. 68 8410AB00   PUSH Foxit_Re.00AB1084                      ; |open
004768CA  |. 6A 00         PUSH 0                                      ; |hWnd = NULL
004768CC  |. FF15 F0E69800 CALL DWORD PTR DS:[<&SHELL32.ShellExecuteA>]        ; \\ShellExecuteA
```

当没有 `/NewWindow true` 时，外部程序是用 `ShellExecuteA` 运行的，此时参数=NULL.固定。

当存在参数 `/NewWindow true` 时，外部程序是用 `CreateProcessA` 运行的，

`CommandLine` 即为 `/F` 中的参数经过 `/`，`\\` 字符变换后的命令行参数。

------------------------------------------------------------------------------------------------------------------------------------------------

`foxit reader 3.1.4.1125` 在处理 `launch action` 时存在的问题。

### 第一种情况

```
<<
/Type /Action
/S /Launch
/Win
<<
/D()/F(c:\\\\windows\\\\system32\\\\notepad.exe)/O()/P(f:\\\\test.bat)
>>
>>
```

这样是不行的。

```
00476040  /$ 6A FF         PUSH -1
00476042  |. 68 106B9400   PUSH Foxit_Re.00946B10                   ;  SE 处理程序安装
00476047  |. 64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0047604D  |. 50           PUSH EAX
0047604E  |. 64:8925 00000>MOV DWORD PTR FS:[0],ESP
00476055  |. 83EC 58       SUB ESP,58
00476058  |. 56           PUSH ESI
00476059  |. 8D4424 04     LEA EAX,DWORD PTR SS:[ESP+4]
0047605D  |. 57           PUSH EDI
0047605E  |. 8BF1          MOV ESI,ECX
00476060  |. 50           PUSH EAX                                  ; /Arg1
00476061  |. E8 1ABD0900   CALL Foxit_Re.00511D80                   ; \\Foxit_Re.00511D80
00476066  |. BF 4820AB00   MOV EDI,Foxit_Re.00AB2048                ;  newwindow
0047606B  |. C74424 68 000>MOV DWORD PTR SS:[ESP+68],0
00476073  |. 8BCF          MOV ECX,EDI
00476075  |. 897C24 0C     MOV DWORD PTR SS:[ESP+C],EDI
00476079  |. 85C9          TEST ECX,ECX
0047607B  |. 74 10         JE SHORT Foxit_Re.0047608D
0047607D  |. 83C9 FF       OR ECX,FFFFFFFF
00476080  |. 33C0          XOR EAX,EAX
00476082  |. F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
00476084  |. F7D1          NOT ECX
00476086  |. 49           DEC ECX
00476087  |. 894C24 10     MOV DWORD PTR SS:[ESP+10],ECX
0047608B  |. EB 08         JMP SHORT Foxit_Re.00476095
0047608D  |> C74424 10 000>MOV DWORD PTR SS:[ESP+10],0
00476095  |> 8B0E          MOV ECX,DWORD PTR DS:[ESI]
00476097  |. 8D5424 0C     LEA EDX,DWORD PTR SS:[ESP+C]
0047609B  |. 52           PUSH EDX
0047609C  |. E8 5FB40500   CALL Foxit_Re.004D1500
004760A1  |. 85C0          TEST EAX,EAX
004760A3  |. 74 57         JE SHORT Foxit_Re.004760FC
004760A5  |. 8BC8          MOV ECX,EAX
004760A7  |. E8 64A20500   CALL Foxit_Re.004D0310
004760AC  |. 85C0          TEST EAX,EAX
004760AE  |. 74 4C         JE SHORT Foxit_Re.004760FC
004760B0  |. B9 11000000   MOV ECX,11
004760B5  |. 33C0          XOR EAX,EAX
004760B7  |. 8D7C24 1C     LEA EDI,DWORD PTR SS:[ESP+1C]
004760BB  |. F3:AB         REP STOS DWORD PTR ES:[EDI]
004760BD  |. 8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]
004760C1  |. C74424 1C 440>MOV DWORD PTR SS:[ESP+1C],44
004760C9  |. 85C0          TEST EAX,EAX
004760CB  |. 74 05         JE SHORT Foxit_Re.004760D2
004760CD  |. 83C0 0C       ADD EAX,0C
004760D0  |. EB 05         JMP SHORT Foxit_Re.004760D7
004760D2  |> B8 9845C000   MOV EAX,Foxit_Re.00C04598
004760D7  |> 8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]
004760DB  |. 8D5424 1C     LEA EDX,DWORD PTR SS:[ESP+1C]
004760DF  |. 51           PUSH ECX                                  ; /pProcessInfo
004760E0  |. 52           PUSH EDX                                  ; |pStartupInfo
004760E1  |. 6A 00         PUSH 0                                   ; |CurrentDir = NULL
004760E3  |. 6A 00         PUSH 0                                   ; |pEnvironment = NULL
004760E5  |. 6A 00         PUSH 0                                   ; |CreationFlags = 0
004760E7  |. 6A 00         PUSH 0                                   ; |InheritHandles = FALSE
004760E9  |. 6A 00         PUSH 0                                   ; |pThreadSecurity = NULL
004760EB  |. 6A 00         PUSH 0                                   ; |pProcessSecurity = NULL
004760ED  |. 50           PUSH EAX                                  ; |CommandLine
004760EE  |. 6A 00         PUSH 0                                   ; |ModuleFileName = NULL
004760F0  |. FF15 C4059900 CALL DWORD PTR DS:[<&KERNEL32.CreateProc>; \\(初始 cpu 选择)
004760F6  |. 8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]
004760FA  |. EB 26         JMP SHORT Foxit_Re.00476122
004760FC  |> 8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]
00476100  |. 85C0          TEST EAX,EAX
00476102  |. 74 05         JE SHORT Foxit_Re.00476109
00476104  |. 83C0 0C       ADD EAX,0C
00476107  |. EB 05         JMP SHORT Foxit_Re.0047610E
00476109  |> B8 9845C000   MOV EAX,Foxit_Re.00C04598
0047610E  |> 6A 05         PUSH 5                                   ; /IsShown = 5
00476110  |. 6A 00         PUSH 0                                   ; |DefDir = NULL
00476112  |. 6A 00         PUSH 0                                   ; |Parameters = NULL
00476114  |. 50           PUSH EAX                                  ; |FileName
00476115  |. 68 EC20AB00   PUSH Foxit_Re.00AB20EC                   ; |o
0047611A  |. 6A 00         PUSH 0                                   ; |hWnd = NULL
0047611C  |. FF15 44079900 CALL DWORD PTR DS:[<&SHELL32.ShellExecut>; \\ShellExecuteW
```

寄存器值：

```
EAX 019F94E4 UNICODE "c:\\windows\\system32\\notepad.exe"
ECX 00000000
EDX 005DCDE0 Foxit_Re.005DCDE0
EBX 00000000
ESP 0012F53C
EBP 01A32E20
ESI 0012F5E4
EDI 00AB2052 Foxit_Re.00AB2052
EIP 0047611C Foxit_Re.0047611C
C 0  ES 0023 32 位 0(FFFFFFFF)
P 1  CS 001B 32 位 0(FFFFFFFF)
A 1  SS 0023 32 位 0(FFFFFFFF)
Z 0  DS 0023 32 位 0(FFFFFFFF)
S 0  FS 003B 32 位 7FFDF000(FFF)
T 0  GS 0000 NULL
D 0
O 0  LastErr ERROR_SUCCESS (00000000)
EFL 00010216 (NO,NB,NE,A,NS,PE,GE,G)
MM0 BF83 D600 85F0 0294
MM1 859F 7778 BF80 0B1B
MM2 0000 0001 0000 0001
MM3 0000 0000 0000 0000
MM4 BFFF FFE0 0000 0000
MM5 8000 0000 0000 0000
MM6 0000 0000 0000 0000
MM7 CA12 D2D2 D2D2 D000
```

当 `EIP=0047611C` 时，程序最后调用了 `ShellExecuteW` 来运行 `Open Action`.而 `ShellExecuteW` 的 `Parameters = NULL` 固定了

所以最终只能运行一个程序，而不能给程序传递参数。

### 第二种情况

```
<<
/Type /Action
/S /Launch
/F (c:\\\\windows\\\\system32\\\\notepad.exe f:\\\\test.bat)
/NewWindow true
>>
```

```
0051571B  |. /74 20         JE SHORT Foxit_Re.0051573D
0051571D  |> |66:3D 2F00    /CMP AX,2F                              ;  /
00515721  |. |75 04         |JNZ SHORT Foxit_Re.00515727
00515723  |. |6A 5C         |PUSH 5C                                 ;  \\
00515725  |. |EB 01         |JMP SHORT Foxit_Re.00515728
```
