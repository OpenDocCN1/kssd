
## 添加系统菜单“置顶”功能

```asm
01004667   .  53            PUSH EBX                                 ; /lParam
01004668   .  56            PUSH ESI                                 ; |hInst
01004669   .  53            PUSH EBX                                 ; |hMenu
0100466A   .  53            PUSH EBX                                 ; |hParent
0100466B   .  FF35 709A0001 PUSH DWORD PTR DS:[1009A70]              ; |Height = 0
01004671   .  BE 94130001   MOV ESI,notepad8.01001394                ; |
01004676   .  FF35 749A0001 PUSH DWORD PTR DS:[1009A74]              ; |Width = 0
0100467C   .  FF35 7C9A0001 PUSH DWORD PTR DS:[1009A7C]              ; |Y = 0
01004682   .  FF35 789A0001 PUSH DWORD PTR DS:[1009A78]              ; |X = 0
01004688   .  68 0000CF00   PUSH 0CF0000                             ; |Style = WS_OVERLAPPED|WS_MINIMIZEBOX|WS_MAXIMIZEBOX|WS_SYSMENU|WS_THICKFRAME|WS_CAPTION
0100468D   .  56            PUSH ESI                                 ; |WindowName => ""
0100468E   .  68 20900001   PUSH notepad8.01009020                   ; |Class = "Notepad"
01004693   .  53            PUSH EBX                                 ; |ExtStyle
01004694   .  FF15 E0110001 CALL DWORD PTR DS:[<&USER32.CreateWindow>; \CreateWindowExW
0100469A   .- E9 C4FA0000   JMP notepad8.01014163                       ;注意是更改notepad主窗口的系统菜单，程序里调用CreateWindow的地方还有创建edit控件时，edit控件的系统菜单是在编辑区右键弹出的那个
```

.............

```asm
01014163    60              PUSHAD
01014164    6A 00           PUSH 0
01014166    50              PUSH EAX
01014167    FF15 CC110001   CALL DWORD PTR DS:[<&USER32.GetSystemMen>; USER32~1.GetSystemMenu
0101416D    A3 40420101     MOV DWORD PTR DS:[1014240],EAX        ;保存系统菜单句柄备后用
01014172    68 30420101     PUSH notepad8.01014230                            ;“置顶”字符
01014177    6A 1F           PUSH 1F
01014179    6A 00           PUSH 0
0101417B    50              PUSH EAX
0101417C    FF15 C8300101   CALL DWORD PTR DS:[<&USER32.AppendMenuA>>; USER32~1.AppendMenuA
01014182    61              POPAD
01014183    8BD0            MOV EDX,EAX
01014185    3BD3            CMP EDX,EBX
01014187    8915 30980001   MOV DWORD PTR DS:[1009830],EDX
0101418D  - E9 1205FFFF     JMP notepad8.010046A4
```

### 添加对菜单项的消息处理

首先弹出菜单时，我们需要根据当前窗口是否已置顶来设置菜单状态，这需要处理 `WM_INITMENUPOPUP` 消息

```asm
01014192    81FE 17010000   CMP ESI,117               ;esi里是消息码，117是WM_INITMENUPOPUP
01014198    0F85 B2000000   JNZ notepad8.01014250
0101419E    50              PUSH EAX
0101419F    8B45 10         MOV EAX,DWORD PTR SS:[EBP+10]              ;窗口句柄
010141A2    3B05 40420101   CMP EAX,DWORD PTR DS:[1014240]     ；[1014240] 里保存的系统菜单句柄
010141A8    58              POP EAX
010141A9    0F85 A1000000   JNZ notepad8.01014250
010141AF    60              PUSHAD
010141B0    0FB605 47870001 MOVZX EAX,BYTE PTR DS:[1008747]  ；[1008747] 里保存着窗口是否置顶的状态
010141B7    C1E0 03         SHL EAX,3                                       ; checked和unchecked 对应0和8,so .....
010141BA    50              PUSH EAX
010141BB    6A 1F           PUSH 1F
010141BD    FF35 40420101   PUSH DWORD PTR DS:[1014240]
010141C3    FF15 48120001   CALL DWORD PTR DS:[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010141C9    61              POPAD
010141CA  ^ E9 23FFFFFF     JMP notepad8.010140F2
```

### 添加系统菜单选择“置顶”时的消息处理

然后添加系统菜单选择“置顶”时消息处理，基本照搬上面的代码

```asm
01014250    81FE 12010000   CMP ESI,112                           ;112为WM_SYSCOMMAND,系统菜单项发出的
01014256  ^ 0F85 96FEFFFF   JNZ notepad8.010140F2
0101425C    837D 10 1F      CMP DWORD PTR SS:[EBP+10],1F    ;‘置顶“的ID
01014260  ^ 0F85 8CFEFFFF   JNZ notepad8.010140F2
01014266    803D 47870001 0>CMP BYTE PTR DS:[1008747],1      
0101426D    74 38           JE SHORT notepad8.010142A7
0101426F    60              PUSHAD
01014270    C605 47870001 0>MOV BYTE PTR DS:[1008747],1
01014277    6A 03           PUSH 3
01014279    6A 00           PUSH 0
0101427B    6A 00           PUSH 0
0101427D    6A 00           PUSH 0
0101427F    6A 00           PUSH 0
01014281    6A FF           PUSH -1
01014283    FF75 08         PUSH DWORD PTR SS:[EBP+8]
01014286    FF15 B8300101   CALL DWORD PTR DS:[<&USER32.SetWindowPos>; USER32~1.SetWindowPos
0101428C    FF75 08         PUSH DWORD PTR SS:[EBP+8]
0101428F    FF15 64120001   CALL DWORD PTR DS:[<&USER32.GetMenu>]    ; USER32~1.GetMenu
01014295    6A 08           PUSH 8
01014297    6A 1C           PUSH 1C
01014299    50              PUSH EAX
0101429A    FF15 48120001   CALL DWORD PTR DS:[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010142A0    61              POPAD
010142A1  ^ E9 4CFEFFFF     JMP notepad8.010140F2
010142A6    90              NOP
010142A7    C605 47870001 0>MOV BYTE PTR DS:[1008747],0
010142AE    60              PUSHAD
010142AF    6A 03           PUSH 3
010142B1    6A 00           PUSH 0
010142B3    6A 00           PUSH 0
010142B5    6A 00           PUSH 0
010142B7    6A 00           PUSH 0
010142B9    6A FE           PUSH -2
010142BB    FF75 08         PUSH DWORD PTR SS:[EBP+8]
010142BE    FF15 B8300101   CALL DWORD PTR DS:[<&USER32.SetWindowPos>; USER32~1.SetWindowPos
010142C4    FF75 08         PUSH DWORD PTR SS:[EBP+8]
010142C7    FF15 64120001   CALL DWORD PTR DS:[<&USER32.GetMenu>]    ; USER32~1.GetMenu
010142CD    6A 00           PUSH 0
010142CF    6A 1C           PUSH 1C
010142D1    50              PUSH EAX
010142D2    FF15 48120001   CALL DWORD PTR DS:[<&USER32.CheckMenuIte>; USER32~1.CheckMenuItem
010142D8    61              POPAD
010142D9  ^ E9 14FEFFFF     JMP notepad8.010140F2
```

至此改造彻底结束，收工，good luck!

## 2009.12.04

试用发现自动换行等功能不对，是消息处理逻辑有误，重新更正。

## 2009.12.08

试用发现如果修改了 `txt` 文件内容，托盘右键菜单“退出”时不会保存文件更改，重新更正。

```asm
01014133    83FF 1E         CMP EDI,1E                                   ;1E即为托盘右键菜单项“退出”的ID
01014136  - 0F85 DDEEFEFF   JNZ notepad9.01003019
0101413C    60              PUSHAD
0101413D    68 00410101     PUSH notepad9.01014100
01014142    6A 02           PUSH 2
01014144    FF15 41300101   CALL DWORD PTR DS:[<&shell32.Shell_Notif>; shell32.Shell_NotifyIconA
0101414A    FF35 00420101   PUSH DWORD PTR DS:[1014200]
01014150    FF15 CC300101   CALL DWORD PTR DS:[<&USER32.DestroyMenu>>; user32.DestroyMenu
01014156    61              POPAD
01014157    56              PUSH ESI                                                       ;下面这段代码是点击窗口菜单"文件“---”退出“的消息处理代码的复制
01014158    56              PUSH ESI
01014159    6A 10           PUSH 10
0101415B    52              PUSH EDX
0101415C    EB 71           JMP SHORT notepad9.010141CF

.....

010141CF    FF15 A4120001   CALL DWORD PTR DS:[<&USER32.PostMessageW>; user32.PostMessageW
010141D5  - E9 8DF1FEFF     JMP notepad9.01003367
```

并将点击窗口菜单“文件”→“退出”的消息处理改为

```asm
01002E8F   >-\E9 A8120100   JMP notepad9.0101413C                    ;  Case 7 of switch 01002BBE
```

## 2010.01.06

今天发现用 `CTRL+C`、`CTRL+V` 不正常，调试发现原来在处理这些消息时最后调用了 `01002e94` 的 `PostMessage()` 函数，而我误删了，现恢复，重新上传附件 `notepad9.rar`。

## 2010.01.07

### 添加字体颜色功能

由于 `notepad9.exe` 里原添加输入表的节太小，无法再添加新的输入函数，所以直接在 `notepad1.exe` 的基础上改的，添加 `gdi32.dll` 的 `SetTextColor()` 函数和 `comdlg32.dll` 的 `ChooseColorW()` 函数。

如何改变 `EDIT` 控件里的字体颜色，查找 MSDN 资料如下：

> Each time an edit control is about to be painted to the screen, Windows sends the parent window a `WM_CTLCOLOR` message. The `wParam` parameter contains the handle to the display context of the edit control, the low-order word of the `lParam` parameter identifies the edit control, and the high-order word of `lParam` contains `CTLCOLOR_EDIT`.

就是 `EDIT` 控件每次显示时会发送 `WM_CTLCOLOR` 消息（好像应该是 `WM_CTLCOLOREDIT` 消息），其中 `wParam` 里是 `HDC`，`lParam` 的低位是控件 ID。

`CHOOSECOLOR` 结构如下：

```c
typedef struct {
  DWORD        lStructSize;
  HWND         hwndOwner;
  HWND         hInstance;
  COLORREF     rgbResult;
  COLORREF   * lpCustColors;
  DWORD        Flags;
  LPARAM       lCustData;
  LPCCHOOKPROC lpfnHook;
  LPCTSTR      lpTemplateName;
} CHOOSECOLOR, *LPCHOOSECOLOR;
```

我在 `109790` 的地方存放了这个结构，大小 `24h`。

紧跟着存放了 `lpCustColors` 指向的用户自定义颜色，有 16 个 RGB 数据，总共大小 `40h`。

代码如下
