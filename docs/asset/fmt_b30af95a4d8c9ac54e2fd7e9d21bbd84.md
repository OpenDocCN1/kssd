
### 01014BD4 - 01014C65

```
01014BD4      A3 08C70101       MOV     DWORD PTR DS:[101C708], EAX           ; cyEdit=HIWORD(lParam)  

01014BD9      833D 1CC70101 08  CMP     DWORD PTR DS:[101C71C], 8  

01014BE0      75 21             JNZ     SHORT 01014C03                       ; if(iCheckFlag!=MF_CHECKED) goto 01014C03  

01014BE2      90                NOP                                           ; 当前没有显示则跳到 01014c03       

01014BE3      90                NOP  

01014BE4      90                NOP  

01014BE5      90                NOP                                           ; 如果现在工具条显示的,执行以下动作  

01014BE6      832D 08C70101 1C SUB     DWORD PTR DS:[101C708], 1C            ; 编辑框高度-=3D  

01014BED      C705 0CC70101 1C000>MOV     DWORD PTR DS:[101C70C], 1C          ; 编辑框最高位置 yEdit=30D  

01014BF7      C705 14C70101 1C000>MOV     DWORD PTR DS:[101C714], 1C          ; 工具条高度 cyToolbar==30D  

01014C01      EB 14             JMP     SHORT 01014C17                       ; 如果 iCheckFlag==0 当前没有显示工具条,  

01014C03      C705 0CC70101 00000>MOV     DWORD PTR DS:[101C70C], 0           ; 则编辑框从客户区最高位置 yEdit=0  

01014C0D      C705 14C70101 00000>MOV     DWORD PTR DS:[101C714], 0           ; 工具条高度设为 0,cyToolbar=0  

01014C17      6A 01             PUSH    1                                    ; /bRepaint=TRUE  

01014C19      FF35 14C70101    PUSH    DWORD PTR DS:[101C714]               ; |cyToolbar  

01014C1F      FF35 04C70101    PUSH    DWORD PTR DS:[101C704]               ; |cxEdit  

01014C25      6A 00             PUSH    0                                    ; |yToolbar  

01014C27      6A 00             PUSH    0                                    ; |xToolbar  

01014C29      FF35 18C70101    PUSH    DWORD PTR DS:[101C718]               ; |hWndToolbar  

01014C2F      FF15 20120001    CALL    DWORD PTR DS:[<&USER32.MoveWindow>]  ; \\MoveWindow 按参数调整窗口大小位置  

01014C35      6A 01             PUSH    1                                    ; /bRepaint=TRUE  

01014C37      FF35 08C70101    PUSH    DWORD PTR DS:[101C708]               ; |cyEdit  

01014C3D      FF35 04C70101    PUSH    DWORD PTR DS:[101C704]               ; |cxEdit  

01014C43      FF35 0CC70101    PUSH    DWORD PTR DS:[101C70C]               ; |yEdit  

01014C49      6A 00             PUSH    0                                    ; |xEdit  

01014C4B      FF35 38980001    PUSH    DWORD PTR DS:[1009838]               ; |hEdit  

01014C51      FF15 20120001    CALL    DWORD PTR DS:[<&USER32.MoveWindow>]  ; \\MoveWindow 按参数调整编辑框窗口大小位置  

01014C57      58                POP     EAX                                  ; 还原 EAX 现场       

01014C58      8B45 10          MOV     EAX, DWORD PTR SS:[EBP+10]           ; 以下动作退出 WNDPROC,平衡堆栈,返回 1  

01014C5B      33F6             XOR     ESI, ESI  

01014C5D      B8 01000000      MOV     EAX, 1  

01014C62      5F                POP     EDI  

01014C63      5E                POP     ESI  

01014C64      C9                LEAVE  

01014C65      C2 1000          RET     10  
```

现在不管怎么样对编辑框操作,都不会有问题了,"工具栏"都老老实实地在那儿,但是,如果我们不想它在那儿怎么办呢?当我们选择 "查看"菜单下的"工具栏"菜单时,让它显示/隐藏工具栏.

实现这个功能,当然要处理,"工具栏"菜单的 `WM_COMMAND` 消息

### 010149C1 - 01014A6C

;=========================================处理菜单"工具栏"消息=========================================

```
010149C1    83FF 1C             CMP     EDI, 1C  

010149C4    0F85 A2000000       JNZ     01014A6C  

010149CA    A1 38980001         MOV     EAX, DWORD PTR DS:[1009838]  

010149CF    A3 80C80101         MOV     DWORD PTR DS:[101C880], EAX  

010149D4    C705 7CC80101 24000>MOV     DWORD PTR DS:[101C87C], 24  

010149DE    C705 84C80101 00000>MOV     DWORD PTR DS:[101C884], 0  

010149E8    C705 88C80101 0000F>MOV     DWORD PTR DS:[101C888], 0FF0000  

010149F2    C705 8CC80101 00AE0>MOV     DWORD PTR DS:[101C88C], 0100AE00  

010149FC    C705 90C80101 01000>MOV     DWORD PTR DS:[101C890], 1  

01014A06    C705 94C80101 00000>MOV     DWORD PTR DS:[101C894], 0  

01014A10    C705 98C80101 00000>MOV     DWORD PTR DS:[101C898], 0  

01014A1A    C705 9CC80101 00000>MOV     DWORD PTR DS:[101C89C], 0  

01014A24    50                PUSH    EAX  

01014A25    68 7CC80101         PUSH    0101C87C  

01014A2A    FF15 1C400101       CALL    DWORD PTR DS:[<&comdlg32.ChooseColorW>] ; comdlg32.ChooseColorW  

01014A30    09C0              OR      EAX, EAX  

01014A32    74 37             JE      SHORT 01014A6B  

01014A34    A1 88C80101         MOV     EAX, DWORD PTR DS:[101C888]  

01014A39    A3 74C80101         MOV     DWORD PTR DS:[101C874], EAX  

01014A3E    FF35 78C80101       PUSH    DWORD PTR DS:[101C878]  

01014A44    FF15 68100001       CALL    DWORD PTR DS:[<&GDI32.DeleteObject>]   ; GDI32.DeleteObject  

01014A4A    FF35 74C80101       PUSH    DWORD PTR DS:[101C874]  

01014A50    FF15 5D400101       CALL    DWORD PTR DS:[101405D]               ; GDI32.CreateSolidBrush  

01014A56    A3 78C80101         MOV     DWORD PTR DS:[101C878], EAX  

01014A5B    6A 01             PUSH    1  

01014A5D    6A 00             PUSH    0  

01014A5F    FF35 38980001       PUSH    DWORD PTR DS:[1009838]  

01014A65    FF15 24120001       CALL    DWORD PTR DS:[<&USER32.InvalidateRect>] ; USER32.InvalidateRect  

01014A6B    58                POP     EAX  

01014A6C    83FF 40             CMP     EDI, 40  

01014A6F    8995 F0FDFFFF       MOV     DWORD PTR SS:[EBP-210], EDX  

01014A75  - E9 49E1FEFF         JMP     01002BC3  
```

更改为：

;原有代码大部分在 `http://bbs.pediy.com//showthread.php?s=&threadid=23277` 有详细说明,此处不作解释

```
010149C1    83FF 1C             CMP     EDI, 1C  

010149C4    0F85 A2000000       JNZ     01014A6C  

010149CA    50                PUSH    EAX  

010149CB    A1 38980001         MOV     EAX, DWORD PTR DS:[1009838]  

010149D0    A3 80C80101         MOV     DWORD PTR DS:[101C880], EAX  

010149D5    C705 7CC80101 24000>MOV     DWORD PTR DS:[101C87C], 24  

010149DF    C705 84C80101 00000>MOV     DWORD PTR DS:[101C884], 0  

010149E9    C705 88C80101 0000F>MOV     DWORD PTR DS:[101C888], 0FF0000  

010149F3    C705 8CC80101 00AE0>MOV     DWORD PTR DS:[101C88C], 0100AE00  

010149FD    C705 90C80101 01000>MOV     DWORD PTR DS:[101C890], 1  

01014A07    C705 94C80101 00000>MOV     DWORD PTR DS:[101C894], 0  

01014A11    C705 98C80101 00000>MOV     DWORD PTR DS:[101C898], 0  

01014A1B    C705 9CC80101 00000>MOV     DWORD PTR DS:[101C89C], 0  

01014A25    68 7CC80101         PUSH    0101C87C  

01014A2A    FF15 1C400101       CALL    DWORD PTR DS:[<&comdlg32.ChooseColorW>] ; comdlg32.ChooseColorW  

01014A30    09C0              OR      EAX, EAX  

01014A32    74 37             JE      SHORT 01014A6B  

01014A34    A1 88C80101         MOV     EAX, DWORD PTR DS:[101C888]  

01014A39    A3 74C80101         MOV     DWORD PTR DS:[101C874], EAX  

01014A3E    FF35 78C80101       PUSH    DWORD PTR DS:[101C878]  

01014A44    FF15 68100001       CALL    DWORD PTR DS:[<&GDI32.DeleteObject>]   ; GDI32.DeleteObject  

01014A4A    FF35 74C80101       PUSH    DWORD PTR DS:[101C874]  

01014A50    FF15 5D400101       CALL    DWORD PTR DS:[101405D]               ; GDI32.CreateSolidBrush  

01014A56    A3 78C80101         MOV     DWORD PTR DS:[101C878], EAX  

01014A5B    6A 01             PUSH    1  

01014A5D    6A 00             PUSH    0  

01014A5F    FF35 38980001       PUSH    DWORD PTR DS:[1009838]  

01014A65    FF15 24120001       CALL    DWORD PTR DS:[<&USER32.InvalidateRect>] ; USER32.InvalidateRect  

01014A6B    58                POP     EAX  

; 判断是否点击的"工具栏"菜单  
01014A6C    83FF 1D             CMP     EDI, 1D                              ; 是否点击的"工具栏"菜单
```
