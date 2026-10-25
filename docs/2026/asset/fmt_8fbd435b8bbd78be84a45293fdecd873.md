
## 反汇编代码

```
004142C7    . 52                         PUSH EDX
004142C8    . FFD7                       CALL EDI
004142CA    . 6A 01                      PUSH 1                                                    ; /Result = 1
004142CC    . 55                         PUSH EBP                                                  ; |hWnd
004142CD    . C705 C4A44200 01000000     MOV DWORD PTR DS:[42A4C4],1                               ; |
004142D7    . FF15 34124200              CALL DWORD PTR DS:[<&USER32.EndDialog>]                   ; EndDialog
004142DD    . A1 A8A44200                MOV EAX,DWORD PTR DS:[42A4A8]
004142E2    . 68 C8A14200                PUSH hostscan.0042A1C8                                    ; /lParam = 42A1C8
004142E7    . 68 03110000                PUSH 1103                                                 ; |wParam = 1103
004142EC    . 68 01040000                PUSH 401                                                  ; |Message = WM_USER+1
004142F1    . 50                         PUSH EAX                                                  ; |hWnd => E025A
004142F2    . FF15 10124200              CALL DWORD PTR DS:[<&USER32.SendMessageA>]                ; SendMessageA
004142F8    . 5F                         POP EDI
004142F9    . 5E                         POP ESI
004142FA    . 5D                         POP EBP
004142FB    . B8 01000000                MOV EAX,1
00414300    . 5B                         POP EBX
00414301    . 83C4 64                    ADD ESP,64
00414304    . C2 1000                    RETN 10
00414307    > 68 A8A24200                PUSH hostscan.0042A2A8                                    ; /IniFileName = "E:Program FilesHostScanlanguagehseng.ini"
0041430C    . 68 C8000000                PUSH C8                                                   ; |BufSize = C8 (200.)
00414311    . 68 70A34200                PUSH hostscan.0042A370                                    ; |ReturnBuffer = hostscan.0042A370
00414316    . 68 408A4200                PUSH hostscan.00428A40                                    ; |Default = "Key is Wrong,Please check the key!!!"
0041431B    . 68 308A4200                PUSH hostscan.00428A30                                    ; |Key = "KeyWrongMessage"
00414320    . 68 90634200                PUSH hostscan.00426390                                    ; |Section = "Run"
00414325    . FF15 9C104200              CALL DWORD PTR DS:[<&KERNEL32.GetPrivateProfileStringA>]  ; GetPrivateProfileStringA
0041432B    . 6A 40                      PUSH 40                                                   ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
0041432D    . 68 E8784200                PUSH hostscan.004278E8                                    ; |Title = "Tip"
00414332    . 68 70A34200                PUSH hostscan.0042A370                                    ; |Text = "Key is Wrong,Please check the key!!!"
00414337    . 55                         PUSH EBP                                                  ; |hOwner
00414338    . FF15 3C124200              CALL DWORD PTR DS:[<&USER32.MessageBoxA>]                 ; MessageBoxA
0041433E    . 5F                         POP EDI
0041433F    . 5E                         POP ESI
00414340    . 5D                         POP EBP
00414341    . 33C0                       XOR EAX,EAX
00414343    . 5B                         POP EBX
00414344    . 83C4 64                    ADD ESP,64
00414347    . C2 1000                    RETN 10
```

## 注册表信息

注册成功后，软件在注册表中建立下列键值：

`[HKEY_CURRENT_USER\Software\NetSeek]`

"`language`"=`dword:00000010`

"`Start`"=`dword:00000001`

"`Username`"="`coldeye`" 用户名

`[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\hostscan.file\shellEx]`

"`DropHandler`"="`6DC31698@6@N<@;BAIC`" 变换后的注册码
