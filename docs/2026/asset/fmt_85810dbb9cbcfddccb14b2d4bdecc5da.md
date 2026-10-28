
``` 
01014ACA    833D 1CC70101 08    CMP     DWORD PTR DS:[101C71C], 8               ;

01014AD1    75 07               JNZ     SHORT 01014ADA                          ; 不显示则不处理直接MoveWindow

01014AD3    8305 08C90101 1E    ADD     DWORD PTR DS:[101C908], 1E              ; 如果显示,则编辑框位置下移1C

01014ADA    6A 01               PUSH    1                                       ; bRepaint=TRUE;

01014ADC    A1 10C90101         MOV     EAX, DWORD PTR DS:[101C910]

01014AE1    2B05 08C90101       SUB     EAX, DWORD PTR DS:[101C908]

01014AE7    57                  PUSH    EDI                                     ;

01014AE8    56                  PUSH    ESI                                     ; 保存寄存器现场

01014AE9    8BF8                MOV     EDI, EAX

01014AEB    A1 0CC90101         MOV     EAX, DWORD PTR DS:[101C90C]

01014AF0    2B05 04C90101       SUB     EAX, DWORD PTR DS:[101C904]

01014AF6    8BF0                MOV     ESI, EAX                                ;

01014AF8    57                  PUSH    EDI                                     ; rc.bottom-rc.top

01014AF9    56                  PUSH    ESI                                     ; rc.right-rc.left

01014AFA    FF35 08C90101       PUSH    DWORD PTR DS:[101C908]                  ; rc.right

01014B00    FF35 04C90101       PUSH    DWORD PTR DS:[101C904]                  ; rc.left

01014B06    FF35 38980001       PUSH    DWORD PTR DS:[1009838]                  ; hWndEdit

01014B0C    FF15 20120001       CALL    DWORD PTR DS:[<&USER32.MoveWindow>]     ; MoveWindow

01014B12    5E                  POP     ESI

01014B13    5F                  POP     EDI

01014B14    58                  POP     EAX                                     ; 还原寄存器现场

;原有代码

01014B15    83FF 40             CMP     EDI, 40

01014B18    8995 F0FDFFFF       MOV     DWORD PTR SS:[EBP-210], EDX             ; 被修改的代码还原

01014B1E  - E9 A0E0FEFF         JMP     01002BC3                                ; 交给原程序处理
```

现在保存所有的修改,然后运行程序,"工具栏"已经能正常工作了,并能设置检查标志.是不是一切都OK了呢?

现在如果我们点击"自动换行"或者"状态栏"菜单,编辑框跑到工具栏下面去,这个是因为记事本对"自动换行"和"状态栏"两个菜单命令的处理,也是默认把编辑框放到客户区最高处的,如果工具栏此是时显示状态的,则会出现上面的BUG

![](http://bbs.pediy.com/upload/2006/4/image/toolbarautonewline.jpg)

所以必须处理这两个命令消息.

### 处理菜单"自动换行"(ID=20)消息

; 跟踪WM\_COMMAND消息处理过程

```
01002BBE   .- E9 FE1D0100       JMP     010149C1                                ; 这是以前修改的不用理它,一会儿会跳回来

01002BC3      90                NOP

01002BC4      90                NOP

01002BC5      90                NOP

01002BC6      90                NOP

01002BC7   .  0F8F F9060000     JG      010032C6

01002BCD   .  0F84 DB060000     JE      010032AE

01002BD3   .  83FF 15           CMP     EDI, 15                                 ;  Switch (cases 1..21)

01002BD6   .  0F8F CE020000     JG      01002EAA                                ;  因为20>15此处必跳
```

;................

;................

;01002BD6跳到此处

```
01002EAA   > \83FF 1A           CMP     EDI, 1A                                 ; 20>1A 此处必跳

01002EAD   .  0F8F 66010000     JG      01003019
```

;................

;................

;01002EAD跳到此处

```
01003019   > \6A 1B             PUSH    1B                                       ;

0100301B   .  5B                POP     EBX                                      ; EBX=1B

0100301C   .  2BFB              SUB     EDI, EBX                                 ; wParam-=1B

0100301E   .  0F84 F7010000     JE      0100321B                                 ;

01003024   .  83EF 05           SUB     EDI, 5                                   ; wParam-=5;

01003027   .  0F84 11010000     JE      0100313E                                 ; wParam==20(是"自动换行"菜单") 跳
```

;................

;................

;01003027跳到此处,"自动换行"菜单的处理代码

```
0100313E   > \A1 50980001       MOV     EAX, DWORD PTR DS:[1009850]             ;  Case 20 of switch 01002BD3

01003143   .  F7D8              NEG     EAX

01003145   .  1BC0              SBB     EAX, EAX

01003147   .  25 00001000       AND     EAX, 100000

0100314C   .  05 04012050       ADD     EAX, 50200104

01003151   .  50                PUSH    EAX                                     ; /Arg1

01003152   .  E8 EA290000       CALL    01005B41                                ; \01005B41,处理自动换行菜单
```

;................

;................

;跟到这里,发现用户选择了"自动换行"菜单,记事本会在这里重新创建编辑框,搞不懂为什么要这么做....

```
01005C11  |.  57                PUSH    EDI                                     ; /lParam

01005C12  |.  FF35 80AB0001     PUSH    DWORD PTR DS:[100AB80]                  ; |hInst = 01000000

01005C18  |.  6A 0F             PUSH    0F                                      ; |hMenu = 0000000F

01005C1A  |.  FF35 30980001     PUSH    DWORD PTR DS:[1009830]                  ; |hParent = 009509CA ('无标题 - 记事本',class='Notepad')

01005C20  |.  FF75 D4           PUSH    DWORD PTR SS:[EBP-2C]                   ; |Height  高度

01005C23  |.  FF75 D0           PUSH    DWORD PTR SS:[EBP-30]                   ; |Width

01005C26  |.  57                PUSH    EDI                                     ; |Y       最高位置,默认是0

01005C27  |.  57                PUSH    EDI                                     ; |X = 0

01005C28  |.  FF75 08           PUSH    DWORD PTR SS:[EBP+8]                    ; |Style

01005C2B  |.  68 94130001       PUSH    01001394                                ; |WindowName = ""

01005C30  |.  68 94170001       PUSH    01001794                                ; |Class = "Edit"

01005C35  |.  68 00020000       PUSH    200                                     ; |ExtStyle = WS\_EX\_CLIENTEDGE

01005C3A  |.  FF15 E0110001     CALL    DWORD PTR DS:[<&USER32.CreateWindowExW>>; \CreateWindowExW
```

;................

;................

分析以上代码,我们只需要更改创建编辑框时的两个参数,编辑框高度和编辑框的最高位置,在这里是这两句

;需要修改的代码

```
01005C20  |.  FF75 D4           PUSH    DWORD PTR SS:[EBP-2C]                   ; |高度

01005C26  |.  57                PUSH    EDI                                     ; |最高位置,默认是0
```

如果当前工具栏状态是"显示",则需要更改它们,如果不是,则不需要更改.必须修改跳转到我们的判断代码

;在这里我们修改 01005C1A,因为必须在高度入栈之前修改它们.多余字节以NOP填充

```
01005C1A      - E9 91F00000       JMP     01014CB0

01005C1F        90                NOP

01005C20    |.  FF75 D4           PUSH    DWORD PTR SS:[EBP-2C]                   ; 高度

01005C23        FF75 D0           PUSH    DWORD PTR SS:[EBP-30]

01005C26        57                PUSH    EDI                                     ; 最高位置

01005C27        57                PUSH    EDI

01005C28        FF75 08           PUSH    DWORD PTR SS:[EBP+8]

01005C2B    |.  68 94130001       PUSH    01001394                                ; |WindowName = ""

01005C30    |.  68 94170001       PUSH    01001794                                ; |Class = "Edit"

01005C35    |.  68 00020000       PUSH    200                                     ; |ExtStyle = WS\_EX\_CLIENTEDGE

01005C3A    |.  FF15 E0110001     CALL    DWORD PTR DS:[<&USER32.CreateWindowExW>>; \CreateWindowExW
```

;................

;................

;01005C1A跳到此处,用于判断是否需要更改高度与最高位置

```
01014CB0      A3 84C80101         MOV     DWORD PTR DS:[101C884], EAX             ; 保存EAX现场,这里不能PUSH EAX

01014CB5      FF35 30980001       PUSH    DWORD PTR DS:[1009830]                  ; 被修改的代码还原

01014CBB      833D 1CC70101 08    CMP     DWORD PTR DS:[101C71C], 8               ; 开始判断工具栏显示状态

01014CC2      75 0F               JNZ     SHORT 01014CD3                          ; 不相等则跳

01014CC4      90                  NOP

01014CC5      90                  NOP

01014CC6      90                  NOP

01014CC7      90                  NOP

01014CC8      836D D4 1C          SUB     DWORD PTR SS:[EBP-2C], 1C               ; 显示则高度减去1C

01014CCC      B8 1C000000         MOV     EAX, 1C                                 ; 最高位置+1C(往下移)

01014CD1      EB 02               JMP     SHORT 01014CD5

01014CD3      33C0                XOR     EAX, EAX                                ; 不显示则不修改直接入栈

01014CD5      FF75 D4             PUSH    DWORD PTR SS:[EBP-2C]
```
