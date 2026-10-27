
```asm
0049F551    FF72 08         PUSH DWORD PTR DS:[EDX+8]                ; |ThreadId = 214

0049F554    FF72 04         PUSH DWORD PTR DS:[EDX+4]                ; |ProcessId = 57C

0049F557    FF93 86BB4000   CALL DWORD PTR DS:[EBX+40BB86]           ; \ContinueDebugEvent

0049F55D    E9 F3000000     JMP 0049F655                             ; 跳去ContinueDebugEvent

0049F562    83BB F7C24000 0>CMP DWORD PTR DS:[EBX+40C2F7],1          ; 第二次int3异常

0049F569    0F85 A6000000   JNZ 0049F615

0049F56F    52              PUSH EDX

0049F570    FF83 F7C24000   INC DWORD PTR DS:[EBX+40C2F7]            ; debug step+1

0049F576    8DBB 64C54000   LEA EDI,DWORD PTR DS:[EBX+40C564]

0049F57C    57              PUSH EDI                                 ; /pContext = Explorer.004AA564

0049F57D    FF76 04         PUSH DWORD PTR DS:[ESI+4]                ; |hThread = 00000038 (window)

0049F580    FF93 78BC4000   CALL DWORD PTR DS:[EBX+40BC78]           ; \GetThreadContext

0049F586    8B8F B8000000   MOV ECX,DWORD PTR DS:[EDI+B8]            ; 获取异常地址(4A7781)

0049F58C    51              PUSH ECX

0049F58D    6A 00           PUSH 0                                   ; /pBytesRead = NULL

0049F58F    6A 01           PUSH 1                                   ; |BytesToRead = 1

0049F591    8D83 F6C24000   LEA EAX,DWORD PTR DS:[EBX+40C2F6]        ; |

0049F597    50              PUSH EAX                                 ; |Buffer = Explorer.004AA2F6

0049F598    51              PUSH ECX                                 ; |pBaseAddress = 4A7781

0049F599    FF36            PUSH DWORD PTR DS:[ESI]                  ; |hProcess = 00000034

0049F59B    FF93 63BC4000   CALL DWORD PTR DS:[EBX+40BC63]           ; \ReadProcessMemory

0049F5A1    59              POP ECX

0049F5A2    8D83 F6C24000   LEA EAX,DWORD PTR DS:[EBX+40C2F6]

0049F5A8    8A00            MOV AL,BYTE PTR DS:[EAX]

0049F5AA    34 52           XOR AL,52

0049F5AC    8883 F6C24000   MOV BYTE PTR DS:[EBX+40C2F6],AL

0049F5B2    51              PUSH ECX

0049F5B3    6A 00           PUSH 0                                   ; /pBytesWritten = NULL

0049F5B5    6A 01           PUSH 1                                   ; |BytesToWrite = 1

0049F5B7    8D83 F6C24000   LEA EAX,DWORD PTR DS:[EBX+40C2F6]        ; |

0049F5BD    50              PUSH EAX                                 ; |Buffer = Explorer.004AA2F6

0049F5BE    51              PUSH ECX                                 ; |Address = 4A7781

0049F5BF    FF36            PUSH DWORD PTR DS:[ESI]                  ; |hProcess = 00000034

0049F5C1    FF93 A4BC4000   CALL DWORD PTR DS:[EBX+40BCA4]           ; \WriteProcessMemory

0049F5C7    59              POP ECX

0049F5C8    8D83 04BA4000   LEA EAX,DWORD PTR DS:[EBX+40BA04]        ; 结束地址4A9A04

0049F5CE    41              INC ECX

0049F5CF    3BC8            CMP ECX,EAX

0049F5D1    75 02           JNZ SHORT 0049F5D5                       ; 判断有没有结束，没有结束则跳回去继续

0049F5D3    EB 02           JMP SHORT 0049F5D7

0049F5D5  ^ EB B5           JMP SHORT 0049F58C

0049F5D7    51              PUSH ECX

0049F5D8    8B87 B8000000   MOV EAX,DWORD PTR DS:[EDI+B8]

0049F5DE    8987 B8000000   MOV DWORD PTR DS:[EDI+B8],EAX

0049F5E4    57              PUSH EDI                                 ; /pContext = Explorer.004AA564

0049F5E5    FF76 04         PUSH DWORD PTR DS:[ESI+4]                ; |hThread = 00000038 (window)

0049F5E8    FF93 8DBC4000   CALL DWORD PTR DS:[EBX+40BC8D]           ; \SetThreadContext

0049F5EE    8B87 B8000000   MOV EAX,DWORD PTR DS:[EDI+B8]

0049F5F4    59              POP ECX

0049F5F5    2BC8            SUB ECX,EAX

0049F5F7    51              PUSH ECX                                 ; /RegionSize = 2283

0049F5F8    50              PUSH EAX                                 ; |RegionBase = Explorer.004A7781

0049F5F9    FF36            PUSH DWORD PTR DS:[ESI]                  ; |hProcess = 00000034

0049F5FB    FF93 BEBC4000   CALL DWORD PTR DS:[EBX+40BCBE]           ; \FlushInstructionCache

0049F601    5A              POP EDX

0049F602    68 02000100     PUSH 10002                               ; /ContinueStatus = DBG\_CONTINUE

0049F607    FF72 08         PUSH DWORD PTR DS:[EDX+8]                ; |ThreadId = 214

0049F60A    FF72 04         PUSH DWORD PTR DS:[EDX+4]                ; |ProcessId = 57C

0049F60D    FF93 86BB4000   CALL DWORD PTR DS:[EBX+40BB86]           ; \ContinueDebugEvent

0049F613    EB 40           JMP SHORT 0049F655

0049F615    68 01000180     PUSH 80010001                            ; /DBG\_EXCEPTION\_NOT\_HANDLED

0049F61A    FF72 08         PUSH DWORD PTR DS:[EDX+8]                ; |dwThreadId

0049F61D    FF72 04         PUSH DWORD PTR DS:[EDX+4]                ; |dwProcessId

0049F620    FF93 86BB4000   CALL DWORD PTR DS:[EBX+40BB86]           ; \ContinueDebugEvent

0049F626    EB 2D           JMP SHORT 0049F655

0049F628    68 01000180     PUSH 80010001                            ; /DBG\_EXCEPTION\_NOT\_HANDLED

0049F62D    FF72 08         PUSH DWORD PTR DS:[EDX+8]                ; |dwThreadId

0049F630    FF72 04         PUSH DWORD PTR DS:[EDX+4]                ; |dwProcessId

0049F633    FF93 86BB4000   CALL DWORD PTR DS:[EBX+40BB86]           ; \ContinueDebugEvent

0049F639    EB 1A           JMP SHORT 0049F655

0049F63B    833A 05         CMP DWORD PTR DS:[EDX],5

0049F63E    75 04           JNZ SHORT 0049F644

0049F640    EB 18           JMP SHORT 0049F65A

0049F642    EB 11           JMP SHORT 0049F655

0049F644    68 02000100     PUSH 10002                               ; /ContinueStatus = DBG\_CONTINUE

0049F649    FF72 08         PUSH DWORD PTR DS:[EDX+8]                ; |dwThreadId

0049F64C    FF72 04         PUSH DWORD PTR DS:[EDX+4]                ; |dwProcessId

0049F64F    FF93 86BB4000   CALL DWORD PTR DS:[EBX+40BB86]           ; \ContinueDebugEvent

0049F655  ^ E9 8BFEFFFF     JMP 0049F4E5

0049F65A    61              POPAD

0049F65B    C9              LEAVE

0049F65C    C2 0400         RETN 4

004A5E5B    90              NOP                                      ; 调试完程序结束

004A5E5C    6A 00           PUSH 0                                   ; /Exit Code = 0

004A5E5E    FF95 47BB4000   CALL DWORD PTR SS:[EBP+40BB47]           ; \ExitProcess
```

到这里我们可以总结一下:壳只用了一个`anti-debug`,然后通过调试标志`X`判断是否为子进程。壳只处理`INT3`异常，异常时把从`4A7781`到`4A9A04`处的代码`xor 52`还原出正确代码。

现在再写一段脚本用于直接跳为单进程的方式：

```
var addr

start:

  gpa "GetProcAddress","kernel32.dll"

  bp $RESULT

lbl1:

  run

lbl2:

  mov addr,esp

  add addr,8

  mov addr,[addr]

  mov addr,[addr]

  cmp addr,73756c46

  jne lbl1

  bc $RESULT

  rtu

lbl3:

  mov addr,eip

  add addr,6

  asm addr,"jmp 004A5D7A"

  sto

  sto

  mov [addr],#E803000000#

  sto

  mov [eax],#58#

  run

  ret
```

重新来过，并运行上面的脚本，然后到这里：

```asm
004A5E7A    E8 1E98FFFF     CALL 0049F69D                            ; ;初始化CRC32表
```

跟进看看：

```asm
0049F69D    60              PUSHAD
0049F69E    E8 00000000     CALL 0049F6A3
0049F6A3    5B              POP EBX
0049F6A4    81EB A3164000   SUB EBX,004016A3                         ; 计算重定位值
0049F6AA    B9 00010000     MOV ECX,100
0049F6AF    BA 2083B8ED     MOV EDX,EDB88320
0049F6B4    8D41 FF         LEA EAX,DWORD PTR DS:[ECX-1]
0049F6B7    51              PUSH ECX
0049F6B8    B9 08000000     MOV ECX,8
0049F6BD    D1E8            SHR EAX,1
0049F6BF    73 02           JNB SHORT 0049F6C3
0049F6C1    33C2            XOR EAX,EDX
0049F6C3    49              DEC ECX
0049F6C4  ^ 75 F7           JNZ SHORT 0049F6BD
0049F6C6    59              POP ECX
0049F6C7    8DBB FABD4000   LEA EDI,DWORD PTR DS:[EBX+40BDFA]
0049F6CD    89448F FC       MOV DWORD PTR DS:[EDI+ECX\*4-4],EAX
0049F6D1    49              DEC ECX
0049F6D2  ^ 75 E0           JNZ SHORT 0049F6B4
0049F6D4    61              POPAD
0049F6D5    C3              RETN
```

……

```asm
004A5E99    E8 00000000     CALL 004A5E9E
004A5E9E    90              NOP
004A5E9F    90              NOP
004A5EA0    90              NOP
004A5EA1    90              NOP
```
