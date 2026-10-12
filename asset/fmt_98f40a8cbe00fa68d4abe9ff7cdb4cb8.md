
```
代码:
00440C3C   push ebp
00440C3D   mov ebp,esp
00440C3F   add esp,-480
00440C45   mov eax,dword ptr ss:\[ebp+10\]
00440C48   mov edx,dword ptr ss:\[ebp+C\]
00440C4B   sub edx,110                     ;  Switch (cases 110..112)
00440C51   je short 00440C62               ;  NEWHAND.00440C62
00440C53   dec edx
00440C54   je short 00440CB2               ;  NEWHAND.00440CB2
00440C56   dec edx
00440C57   je 00440CF7                     ;  NEWHAND.00440CF7
00440C5D   jmp 00440D0E                    ;  NEWHAND.00440D0E
00440C62   mov byte ptr ss:\[ebp\-480\],0     ;  Case 110 (WM\_INITDIALOG) of switch 00440C4B
00440C69   mov byte ptr ss:\[ebp\-440\],0
00440C70   lea eax,dword ptr ss:\[ebp\-440\]
00440C76   lea ecx,dword ptr ss:\[ebp\-480\]
00440C7C   push eax                        ; /Arg6
00440C7D   push ecx                        ; |Arg5
00440C7E   push 0A                         ; |Arg4 = 0000000A
00440C80   push 1                          ; |Arg3 = 00000001
00440C82   lea eax,dword ptr ss:\[ebp\-400\]  ; |
00440C88   push 4B9073                     ; |Arg2 = 004B9073 ASCII 0A,"NewHand v%"
00440C8D   push eax                        ; |Arg1
00440C8E   call 004A6C2C                   ; \\NEWHAND.004A6C2C
00440C93   add esp,18
00440C96   lea edx,dword ptr ss:\[ebp\-400\]
00440C9C   push edx                        ; /Text
00440C9D   push 0E75                       ; |ControlID = E75 (3701.)
00440CA2   mov ecx,dword ptr ss:\[ebp+8\]    ; |
00440CA5   push ecx                        ; |hWnd
00440CA6   call 004AF58E                   ; \\SetDlgItemTextA
00440CAB   call 00546360                   ;  // 原指令 mov eax,1
00440CB0   jmp short 00440D10              ;  NEWHAND.00440D10
00440CB2   mov edx,eax                     ;  Case 111 (WM\_COMMAND) of switch 00440C4B
00440CB4   and dx,0FFFF
00440CB9   cmp dx,1
00440CBD   je short 00440CC5               ;  NEWHAND.00440CC5
00440CBF   cmp dx,2
00440CC3   jnz short 00440CD2              ;  NEWHAND.00440CD2
00440CC5   push 0                          ; /Result = 0
00440CC7   mov ecx,dword ptr ss:\[ebp+8\]    ; |
00440CCA   push ecx                        ; |hWnd
00440CCB   call 004AF3EA                   ; \\EndDialog
00440CD0   jmp short 00440D0E              ;  NEWHAND.00440D0E
00440CD2   call 00546206                   ;  // 原指令 cmp dx,0e76
00440CD7   jnz short 00440D0E              ;  NEWHAND.00440D0E
00440CD9   push 0                          ; /IsShown = 0
00440CDB   push 4B91A4                     ; |DefDir = "."
00440CE0   push 0                          ; |Parameters = NULL
00440CE2   push 4B917F                     ; |FileName = "http://home.t-online.de/home/NewHand"
00440CE7   push 4B917A                     ; |Operation = "open"
00440CEC   mov eax,dword ptr ss:\[ebp+8\]    ; |
00440CEF   push eax                        ; |hWnd
00440CF0   call 004AF342                   ; \\ShellExecuteA
00440CF5   jmp short 00440D0E              ;  NEWHAND.00440D0E
00440CF7   and eax,0FFF0                   ;  Case 112 (WM\_SYSCOMMAND) of switch 00440C4B
00440CFC   cmp eax,0F060
00440D01   jnz short 00440D0E              ;  NEWHAND.00440D0E
00440D03   push 0                          ; /Result = 0
00440D05   mov edx,dword ptr ss:\[ebp+8\]    ; |
00440D08   push edx                        ; |hWnd
00440D09   call 004AF3EA                   ; \\EndDialog
00440D0E   xor eax,eax                     ;  Default case of switch 00440C4B
00440D10   mov esp,ebp
00440D12   pop ebp
00440D13   retn 10
00440D16   nop
00440D17   nop
00440D18   nop
00440D18   push 0                          ; /lParam = NULL
00440D1A   mov eax,dword ptr ds:\[4D3B80\]   ; |
00440D1F   push 440C3C                     ; |DlgProc = NEWHAND.00440C3C //借用一下这里
00440D24   push eax                        ; |hOwner => NULL
00440D25   mov edx,dword ptr ds:\[4D3B78\]   ; |
00440D2B   push 4B91A6                     ; |pTemplate = "DIA\_ABOUT"
00440D30   push edx                        ; |hInst => NULL
00440D31   call 004AF3C6                   ; \\DialogBoxParamA
00440D36   retn
```

接下来为刚才处理过的 `OD1.10` 中文版增加一个 `1000h` 长度的 `section`，作为编写运行代码（数据）的空间

下面是 DIY 我们自己需要的代码，对于修改消息流的指令就不贴了，已有注释，代码很乱，边改边写的，没办法了，下面看到的 API 都是从 `OD1.10` 里面找到的。。。代码区定为 va: `546000` - `5463FF`
