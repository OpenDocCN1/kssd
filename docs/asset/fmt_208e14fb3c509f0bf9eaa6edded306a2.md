
# 工具栏显示与隐藏逻辑补丁

## 1. 代码实现

以下新加代码用于“显示”、“隐藏”工具栏。

```
01014A6F    0F85 A0000000       JNZ     01014B15                            ; 不是则交张程序处理

; 以下新加代码用于"显示","隐藏"工具栏

01014A75    50                 PUSH    EAX                                  ; 保存 EAX 现场

01014A76    833D 1CC70101 08   CMP     DWORD PTR DS:[101C71C], 8

01014A7D    74 0E              JE      SHORT 01014A8D                        ; 如果当前是显示,则跳到隐藏代码

01014A7F    C705 1CC70101 08000>MOV    DWORD PTR DS:[101C71C], 8            ; 否则显示,置显示状态为显示

01014A89    6A 01              PUSH    1                                     ; nCmdShow=SW_SHOW

01014A8B    EB 0C              JMP     SHORT 01014A99

01014A8D    C705 1CC70101 00000>MOV    DWORD PTR DS:[101C71C], 0

01014A97    6A 00              PUSH    0                                     ; nCmdSHOW=SW_HIDE

01014A99    FF35 18C70101      PUSH    DWORD PTR DS:[101C718]

01014A9F    FF15 B0110001      CALL    DWORD PTR DS:[<&USER32.ShowWindow>]   ; ShowWindow(hWndToolbar,nCmdShow)

01014AA5    FF35 1CC70101      PUSH    DWORD PTR DS:[101C71C]               ; 显示状态=

01014AAB    6A 1D              PUSH    1D                                    ; 菜单 ID

01014AAD    FF35 20C70101      PUSH    DWORD PTR DS:[101C720]               ; hMenu

01014AB3    FF15 48120001      CALL    DWORD PTR DS:[<&USER32.CheckMenuItem>] ; CheckMenuItem

01014AB9    68 04C90101        PUSH    0101C904                              ; /&RECT

01014ABE    FF35 30980001      PUSH    DWORD PTR DS:[1009830]               ; |hWnd

01014AC4    FF15 88110001      CALL    DWORD PTR DS:[<&USER32.GetClientRect>] ; \GetClientRect(hWnd,&RECT)

01014ACA    833D 1CC70101 08   CMP     DWORD PTR DS:[101C71C], 8             ;

01014AD1    75 07              JNZ     SHORT 01014ADA                        ; 不显示则不处理直接 MoveWindow

01014AD3    8305 08C90101 1E   ADD     DWORD PTR DS:[101C908], 1E           ; 如果显示,则编辑框位置下移 1C

01014ADA    6A 01              PUSH    1                                     ; bRepaint=TRUE;

01014ADC    A1 10C90101        MOV     EAX, DWORD PTR DS:[101C910]

01014AE1    2B05 08C90101      SUB     EAX, DWORD PTR DS:[101C908]

01014AE7    57                 PUSH    EDI

01014AE8    56                 PUSH    ESI                                   ; 保存寄存器现场

01014AE9    8BF8               MOV     EDI, EAX

01014AEB    A1 0CC90101        MOV     EAX, DWORD PTR DS:[101C90C]

01014AF0    2B05 04C90101      SUB     EAX, DWORD PTR DS:[101C904]

01014AF6    8BF0               MOV     ESI, EAX                              ;

01014AF8    57                 PUSH    EDI                                   ; rc.bottom-rc.top

01014AF9    56                 PUSH    ESI                                   ; rc.right-rc.left

01014AFA    FF35 08C90101      PUSH    DWORD PTR DS:[101C908]                ; rc.right

01014B00    FF35 04C90101      PUSH    DWORD PTR DS:[101C904]                ; rc.left

01014B06    FF35 38980001      PUSH    DWORD PTR DS:[1009838]                ; hWndEdit

01014B0C    FF15 20120001      CALL    DWORD PTR DS:[<&USER32.MoveWindow>]   ; MoveWindow

01014B12    5E                 POP     ESI

01014B13    5F                 POP     EDI

01014B14    58                 POP     EAX                                   ; 还原寄存器现场

;原有代码

01014B15    83FF 40            CMP     EDI, 40

01014B18    8995 F0FDFFFF      MOV     DWORD PTR SS:[EBP-210], EDX           ; 被修改的代码还原

01014B1E   - E9 A0E0FEFF       JMP     01002BC3                              ; 交给原程序处理
```

现在保存所有的修改，然后运行程序，“工具栏”已经能正常工作了，并能设置检查标志。是不是一切都 `OK` 了呢？

## 2. 问题分析

现在如果我们点击“自动换行”或者“状态栏”菜单，编辑框跑到工具栏下面去。这个是因为记事本对“自动换行”和“状态栏”两个菜单命令的处理，也是默认把编辑框放到客户区最高处的。如果工具栏此时是显示状态的，则会出现上面的 `BUG`。

![](img/dbb7a20dba41111403b98e045ff14e15.jpg)

所以必须处理这两个命令消息。

## 3. 处理“自动换行”菜单消息

### 3.1 跟踪 `WM_COMMAND` 消息处理过程

```
;========================================处理菜单"自动换行"(ID=20)消息=====================================

01002BBE   .- E9 FE1D0100      JMP     010149C1                              ; 这是以前修改的不用理它,一会儿会跳回来

01002BC3      90               NOP

01002BC4      90               NOP

01002BC5      90               NOP

01002BC6      90               NOP

01002BC7   .  0F8F F9060000    JG      010032C6

01002BCD   .  0F84 DB060000    JE      010032AE

01002BD3   .  83FF 15          CMP     EDI, 15                               ;  Switch (cases 1..21)

01002BD6   .  0F8F CE020000    JG      01002EAA                              ;  因为 20>15 此处必跳

;................

;................

;01002BD6 跳到此处

01002EAA   >  \83FF 1A          CMP     EDI, 1A                               ; 20>1A 此处必跳

01002EAD   .  0F8F 66010000    JG      01003019

;................

;................

;01002EAD 跳到此处

01003019   >  \6A 1B            PUSH    1B

0100301B   .  5B                POP     EBX                                   ; EBX=1B

0100301C   .  2BFB              SUB     EDI, EBX                              ; wParam-=1B

0100301E   .  0F84 F7010000    JE      0100321B

01003024   .  83EF 05           SUB     EDI, 5                                ; wParam-=5;

01003027   .  0F84 11010000    JE      0100313E                              ; wParam==20(是"自动换行"菜单") 跳

;................

;................

;01003027 跳到此处,"自动换行"菜单的处理代码

0100313E   >  \A1 50980001      MOV     EAX, DWORD PTR DS:[1009850]          ;  Case 20 of switch 01002BD3

01003143   .  F7D8              NEG     EAX

01003145   .  1BC0              SBB     EAX, EAX

01003147   .  25 00001000       AND     EAX, 100000

0100314C   .  05 04012050       ADD     EAX, 50200104

01003151   .  50                PUSH    EAX                                   ; /Arg1

01003152   .  E8 EA290000       CALL    01005B41                              ; \01005B41,处理自动换行菜单

;................

;................

;跟到这里,发现用户选择了"自动换行"菜单,记事本会在这里重新创建编辑框,搞不懂为什么要这么做....

01005C11  |.  57                PUSH    EDI                                   ; /lParam

01005C12  |.  FF35 80AB0001    PUSH    DWORD PTR DS:[100AB80]                ; |hInst = 01000000

01005C18  |.  6A 0F            PUSH    0F                                      ; |hMenu = 0000000F

01005C1A  |.  FF35 30980001    PUSH    DWORD PTR DS:[1009830]                ; |hParent = 009509CA ('无标题 - 记事本',class='Notepad')

01005C20  |.  FF75 D4          PUSH    DWORD PTR SS:[EBP-2C]                 ; |Height  高度

01005C23  |.  FF75 D0          PUSH    DWORD PTR SS:[EBP-30]                 ; |Width

01005C26  |.  57                PUSH    EDI                                   ; |Y      最高位置,默认是 0

01005C27  |.  57                PUSH    EDI                                   ; |X = 0

01005C28  |.  FF75 08          PUSH    DWORD PTR SS:[EBP+8]                  ; |Style

01005C2B  |.  68 94130001      PUSH    01001394                              ; |WindowName = ""

01005C30  |.  68 94170001      PUSH    01001794                              ; |Class = "Edit"

01005C35  |.  68 00020000      PUSH    200                                   ; |ExtStyle = WS_EX_CLIENTEDGE

01005C3A  |.  FF15 E0110001    CALL    DWORD PTR DS:[<&USER32.CreateWindowExW>] ; \CreateWindowExW
```

### 3.2 修复方案

分析以上代码，我们只需要更改创建编辑框时的两个参数：编辑框高度和编辑框的最高位置。在这里是这两句：

```
;需要修改的代码

01005C20  |.  FF75 D4          PUSH    DWORD PTR SS:[EBP-2C]                 ; |高度

01005C26  |.  57                PUSH    EDI                                   ; |最高位置,默认是 0
```
