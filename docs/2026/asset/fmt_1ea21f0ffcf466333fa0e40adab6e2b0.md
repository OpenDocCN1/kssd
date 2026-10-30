
# foxit reader 3.2.1.0401 launch action 分析

```asm
00476061  |.  E8 1ABD0900   CALL Foxit\_Re.00511D80                   ; \Foxit\_Re.00511D80

00476066  |.  BF 4820AB00   MOV EDI,Foxit\_Re.00AB2048                ;  newwindow

0047606B  |.  C74424 68 000>MOV DWORD PTR SS:[ESP+68],0

00476073  |.  8BCF          MOV ECX,EDI

00476075  |.  897C24 0C     MOV DWORD PTR SS:[ESP+C],EDI

00476079  |.  85C9          TEST ECX,ECX

0047607B  |.  74 10         JE SHORT Foxit\_Re.0047608D

0047607D  |.  83C9 FF       OR ECX,FFFFFFFF

00476080  |.  33C0          XOR EAX,EAX

00476082  |.  F2:AE         REPNE SCAS BYTE PTR ES:[EDI]

00476084  |.  F7D1          NOT ECX

00476086  |.  49            DEC ECX

00476087  |.  894C24 10     MOV DWORD PTR SS:[ESP+10],ECX

0047608B  |.  EB 08         JMP SHORT Foxit\_Re.00476095

0047608D  |>  C74424 10 000>MOV DWORD PTR SS:[ESP+10],0

00476095  |>  8B0E          MOV ECX,DWORD PTR DS:[ESI]

00476097  |.  8D5424 0C     LEA EDX,DWORD PTR SS:[ESP+C]

0047609B  |.  52            PUSH EDX

0047609C  |.  E8 5FB40500   CALL Foxit\_Re.004D1500

004760A1  |.  85C0          TEST EAX,EAX

004760A3  |.  74 57         JE SHORT Foxit\_Re.004760FC

004760A5  |.  8BC8          MOV ECX,EAX

004760A7  |.  E8 64A20500   CALL Foxit\_Re.004D0310

004760AC  |.  85C0          TEST EAX,EAX

004760AE  |.  74 4C         JE SHORT Foxit\_Re.004760FC

004760B0  |.  B9 11000000   MOV ECX,11

004760B5  |.  33C0          XOR EAX,EAX

004760B7  |.  8D7C24 1C     LEA EDI,DWORD PTR SS:[ESP+1C]

004760BB  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]

004760BD  |.  8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]

004760C1  |.  C74424 1C 440>MOV DWORD PTR SS:[ESP+1C],44

004760C9  |.  85C0          TEST EAX,EAX

004760CB  |.  74 05         JE SHORT Foxit\_Re.004760D2

004760CD  |.  83C0 0C       ADD EAX,0C

004760D0  |.  EB 05         JMP SHORT Foxit\_Re.004760D7

004760D2  |>  B8 9845C000   MOV EAX,Foxit\_Re.00C04598

004760D7  |>  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]

004760DB  |.  8D5424 1C     LEA EDX,DWORD PTR SS:[ESP+1C]

004760DF  |.  51            PUSH ECX                                 ; /pProcessInfo

004760E0  |.  52            PUSH EDX                                 ; |pStartupInfo

004760E1  |.  6A 00         PUSH 0                                   ; |CurrentDir = NULL

004760E3  |.  6A 00         PUSH 0                                   ; |pEnvironment = NULL

004760E5  |.  6A 00         PUSH 0                                   ; |CreationFlags = 0

004760E7  |.  6A 00         PUSH 0                                   ; |InheritHandles = FALSE

004760E9  |.  6A 00         PUSH 0                                   ; |pThreadSecurity = NULL

004760EB  |.  6A 00         PUSH 0                                   ; |pProcessSecurity = NULL

004760ED  |.  50            PUSH EAX                                 ; |CommandLine

004760EE  |.  6A 00         PUSH 0                                   ; |ModuleFileName = NULL

004760F0  |.  FF15 C4059900 CALL DWORD PTR DS:[<&KERNEL32.CreateProc>; \(初始 cpu 选择)

004760F6  |.  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]

004760FA  |.  EB 26         JMP SHORT Foxit\_Re.00476122

004760FC  |>  8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]

00476100  |.  85C0          TEST EAX,EAX

00476102  |.  74 05         JE SHORT Foxit\_Re.00476109

00476104  |.  83C0 0C       ADD EAX,0C

00476107  |.  EB 05         JMP SHORT Foxit\_Re.0047610E

00476109  |>  B8 9845C000   MOV EAX,Foxit\_Re.00C04598

0047610E  |>  6A 05         PUSH 5                                   ; /IsShown = 5

00476110  |.  6A 00         PUSH 0                                   ; |DefDir = NULL

00476112  |.  6A 00         PUSH 0                                   ; |Parameters = NULL

00476114  |.  50            PUSH EAX                                 ; |FileName

00476115  |.  68 EC20AB00   PUSH Foxit\_Re.00AB20EC                   ; |o

0047611A  |.  6A 00         PUSH 0                                   ; |hWnd = NULL

0047611C  |.  FF15 44079900 CALL DWORD PTR DS:[<&SHELL32.ShellExecut>; \ShellExecuteW
```

寄存器情况：

```text
EAX 019F9B9C UNICODE "c:\windows\system32\notepad.exe f:\test.bat"

ECX 0012F560

EDX 0012F570

EBX 00000000

ESP 0012F52C

EBP 01A32E80

ESI 0012F5E4

EDI 0012F5B4

EIP 004760F0 Foxit\_Re.004760F0

C 0  ES 0023 32位 0(FFFFFFFF)

P 1  CS 001B 32位 0(FFFFFFFF)

A 0  SS 0023 32位 0(FFFFFFFF)

Z 0  DS 0023 32位 0(FFFFFFFF)

S 0  FS 003B 32位 7FFDF000(FFF)

T 0  GS 0000 NULL

D 0

O 0  LastErr ERROR\_SUCCESS (00000000)

EFL 00010206 (NO,NB,NE,A,NS,PE,GE,G)

MM0 0059 5A5B 0059 5A5B

MM1 85B8 C2C8 BF80 0B1B

MM2 0000 0001 0000 0001

MM3 0000 0000 0000 0000

MM4 BFFF FFE0 0000 0000

MM5 8000 0000 0000 0000

MM6 0000 0000 0000 0000

MM7 CA12 D2D2 D2D2 D000
```

当参数`/NewWindow true`设置了时，会调用`CreateProcess`来执行程序。而API参数`CommandLine`是个变量。

这里是`c:\windows\system32\notepad.exe f:\test.bat`。

此时，即向Launch Action传递了参数。

---

```text
<<

/Type /Action

/S /Launch

/F (c:\\windows\\system32\\notepad.exe f:\\test.bat)

/NewWindow true

>>
```

```asm
0040A0E1   > \8B8424 880000>MOV EAX,DWORD PTR SS:[ESP+88]

0040A0E8   .  85C0          TEST EAX,EAX

0040A0EA   .  B8 BC81B300   MOV EAX,Foxit\_Re.00B381BC                    ;  UNICODE "open"

0040A0EF   .  75 05         JNZ SHORT Foxit\_Re.0040A0F6

0040A0F1   .  B8 B081B300   MOV EAX,Foxit\_Re.00B381B0                    ;  UNICODE "print"

0040A0F6   >  8B8C24 840000>MOV ECX,DWORD PTR SS:[ESP+84]

0040A0FD   .  8B9424 800000>MOV EDX,DWORD PTR SS:[ESP+80]

0040A104   .  6A 01         PUSH 1                                       ; /IsShown = 1

0040A106   .  51            PUSH ECX                                     ; |DefDir

0040A107   .  52            PUSH EDX                                     ; |Parameters

0040A108   .  57            PUSH EDI                                     ; |FileName

0040A109   .  50            PUSH EAX                                     ; |Operation

0040A10A   .  6A 00         PUSH 0                                       ; |hWnd = NULL

0040A10C   .  FF15 7497A000 CALL DWORD PTR DS:[<&SHELL32.ShellExecuteW>] ; \ShellExecuteW
```

这时`FileName`为`c:\windows\system32\notepad.exe f:\test.bat`，参数空。很明显程序不能运行。

```text
<<

/Type /Action

/S /Launch

/Win

<<

/D()/F(c:\\windows\\system32\\notepad.exe)/O()/P(f:\\test.bat)

>>

>>
```

```asm
0040A0E1   > \8B8424 880000>MOV EAX,DWORD PTR SS:[ESP+88]

0040A0E8   .  85C0          TEST EAX,EAX

0040A0EA   .  B8 BC81B300   MOV EAX,Foxit\_Re.00B381BC                    ;  UNICODE "open"

0040A0EF   .  75 05         JNZ SHORT Foxit\_Re.0040A0F6

0040A0F1   .  B8 B081B300   MOV EAX,Foxit\_Re.00B381B0                    ;  UNICODE "print"

0040A0F6   >  8B8C24 840000>MOV ECX,DWORD PTR SS:[ESP+84]

0040A0FD   .  8B9424 800000>MOV EDX,DWORD PTR SS:[ESP+80]

0040A104   .  6A 01         PUSH 1                                       ; /IsShown = 1

0040A106   .  51            PUSH ECX                                     ; |DefDir

0040A107   .  52            PUSH EDX                                     ; |Parameters

0040A108   .  57            PUSH EDI                                     ; |FileName

0040A109   .  50            PUSH EAX                                     ; |Operation

0040A10A   .  6A 00         PUSH 0                                       ; |hWnd = NULL

0040A10C   .  FF15 7497A000 CALL DWORD PTR DS:[<&SHELL32.ShellExecuteW>] ; \ShellExecuteW
```

寄存器：

```text
EAX 00B381BC UNICODE "open"

ECX 00CBB7F8 Foxit\_Re.00CBB7F8

EDX 01ACB5CC UNICODE "f:\test.bat"

EBX 01AF7F18

ESP 0012EEA4

EBP 0012F5E4

ESI 00000001

EDI 01ACB79C UNICODE "c:\windows\system32\notepad.exe"

EIP 0040A10C Foxit\_Re.0040A10C
```

堆栈：

```text
0012EEA4   00000000  |hWnd = NULL

0012EEA8   00B381BC  |Operation = "open"

0012EEAC   01ACB79C  |FileName = "c:\windows\system32\notepad.exe"

0012EEB0   01ACB5CC  |Parameters = "f:\test.bat"

0012EEB4   00CBB7F8  |DefDir = ""

0012EEB8   00000001  \IsShown = 1
```
