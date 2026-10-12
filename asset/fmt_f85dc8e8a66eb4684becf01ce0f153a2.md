
# TMPGEnc MPEG Editor 1.3.2.98

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-532.htm`](https://www.pediy.com/kssd/pediy08/pediy8-532.htm)

## 前言

在看雪论坛里看到有悬赏 100 大洋破解“小日本” `TMPGEnc` `MPEG` `Editor` 软件(当然我不是为了 100 大洋，只想尽自己能力试着分析下)要求破解在看雪是不被允许的请这类朋友尊重一下自己论坛的 ID 啦 ^_^!!,特下来试着调试一下，看看我自己这个菜鸟到底能把这个软件跟踪那一部份(想试下自己的功力呵呵。。其实我很菜!!请不要拍砖)

安装运行马上要输入注册码，否则不给用!。。只好郁闷退出了!用 `PEID` 查显示 Nothing found \[Overlay\] \* ???没有找到$&^#%$@。。选择深度扫描显示为 `ASPack` 2.x (without poly) -> Alexey Solodnikov \[Overlay\] 哦。。!知道一些普了

## OD 载入

```
00A6C000 >   60               PUSHAD                          ; //壳的入口
00A6C001    E8 00000000      CALL    TMPGEncM.00A6C006
00A6C006    5E               POP     ESI
00A6C007    83C6 09          ADD     ESI,9
00A6C00A    E9 20170000      JMP     TMPGEncM.00A6D72F
```

来到 `00A6C001` 再看 `ESP=0012FFA4`，好了用 `ESP` 定律下 `HR 0012FFA4` 运行 `F9`

```
00A6D906    68 0120A600      PUSH    TMPGEncM.00A62001                  ; //断在这里
00A6D90B    C3               RETN
00A6D90C    55               PUSH    EBP
```

返回到 `00A62001`

```
00A62001    90               NOP                               ; //返回到这里
00A62002    60               PUSHAD                        ; //难道这个是第 2 个壳的入口
00A62003    E8 03000000      CALL    TMPGEncM.00A6200B                 ; //F7 跟进
\====================================================================================
00A6200B    5D               POP     EBP                       ; TMPGEncM.00A62008
00A6200C    45               INC     EBP
00A6200D    55               PUSH    EBP
00A6200E    C3               RETN
```

返回到 `00A62009`

```
00A62009   /EB 04           JMP     SHORT TMPGEncM.00A6200F
00A6200B   |5D              POP     EBP
00A6200C   |45              INC     EBP
00A6200D   |55              PUSH    EBP
00A6200E   |C3              RETN
00A6200F   \E8 01000000     CALL    TMPGEncM.00A62015                 ; //跟进!!
```

跟进 `00A62015`

```
00A62015    5D               POP     EBP                       ; //来这里
00A62016    BB ECFFFFFF      MOV     EBX,-14
00A6201B    03DD             ADD     EBX,EBP
00A6201D    81EB 00206600    SUB     EBX,TMPGEncM.00662000
00A62023    83BD 22040000 00 CMP     DWORD PTR SS:[EBP+422],0
00A6202A    899D 22040000    MOV     DWORD PTR SS:[EBP+422],EBX
00A62030    0F85 65030000    JNZ     TMPGEncM.00A6239B
00A62036    8D85 2E040000    LEA     EAX,DWORD PTR SS:[EBP+42E]
00A6203C    50               PUSH    EAX
00A6203D    FF95 4C0F0000    CALL    DWORD PTR SS:[EBP+F4C]
00A62043    8985 26040000    MOV     DWORD PTR SS:[EBP+426],EAX
00A62049    8BF8             MOV     EDI,EAX
00A6204B    8D5D 5E          LEA     EBX,DWORD PTR SS:[EBP+5E]
00A6204E    53               PUSH    EBX
00A6204F    50               PUSH    EAX
00A62050    FF95 480F0000    CALL    DWORD PTR SS:[EBP+F48]
00A62056    8985 4C050000    MOV     DWORD PTR SS:[EBP+54C],EAX
00A6205C    8D5D 6B          LEA     EBX,DWORD PTR SS:[EBP+6B]
00A6205F    53               PUSH    EBX
00A62060    57               PUSH    EDI
00A62061    FF95 480F0000    CALL    DWORD PTR SS:[EBP+F48]
00A62067    8985 50050000    MOV     DWORD PTR SS:[EBP+550],EAX
00A6206D    8D45 77          LEA     EAX,DWORD PTR SS:[EBP+77]
00A62070    FFE0             JMP     EAX                         ; //F8 过跳走一定要跳
```

跳来到这里 `00A6208B`

```
00A6208B    8B9D 30050000    MOV     EBX,DWORD PTR SS:[EBP+530]
00A62091    0BDB             OR      EBX,EBX
00A62093    74 0A            JE      SHORT TMPGEncM.00A6209F
00A62095    8B03             MOV     EAX,DWORD PTR DS:[EBX]
00A62097    8785 34050000    XCHG    DWORD PTR SS:[EBP+534],EAX
00A6209D    8903             MOV     DWORD PTR DS:[EBX],EAX
00A6209F    8DB5 68050000    LEA     ESI,DWORD PTR SS:[EBP+568]
00A620A5    833E 00          CMP     DWORD PTR DS:[ESI],0
00A620A8    0F84 21010000    JE      TMPGEncM.00A621CF
00A620AE    6A 04            PUSH    4
00A620B0    68 00100000      PUSH    1000
00A620B5    68 00180000      PUSH    1800
00A620BA    6A 00            PUSH    0
00A620BC    FF95 4C050000    CALL    DWORD PTR SS:[EBP+54C]
00A620C2    8985 56010000    MOV     DWORD PTR SS:[EBP+156],EAX
00A620C8    8B46 04          MOV     EAX,DWORD PTR DS:[ESI+4]
00A620CB    05 0E010000      ADD     EAX,10E
00A620D0    6A 04            PUSH    4
00A620D2    68 00100000      PUSH    1000
00A620D7    50               PUSH    EAX
00A620D8    6A 00            PUSH    0
00A620DA    FF95 4C050000    CALL    DWORD PTR SS:[EBP+54C]
00A620E0    8985 52010000    MOV     DWORD PTR SS:[EBP+152],EAX
00A620E6    56               PUSH    ESI
00A620E7    8B1E             MOV     EBX,DWORD PTR DS:[ESI]
00A620E9    039D 22040000    ADD     EBX,DWORD PTR SS:[EBP+422]
00A620EF    FFB5 56010000    PUSH    DWORD PTR SS:[EBP+156]
00A620F5    FF76 04          PUSH    DWORD PTR DS:[ESI+4]
00A620F8    50               PUSH    EAX
00A620F9    53               PUSH    EBX
00A620FA    E8 6D050000      CALL    TMPGEncM.00A6266C
00A620FF    B3 00            MOV     BL,0
00A62101    80FB 00          CMP     BL,0
00A62104    75 5E            JNZ     SHORT TMPGEncM.00A62164
00A62106    FE85 EC000000    INC     BYTE PTR SS:[EBP+EC]
00A6210C    8B3E             MOV     EDI,DWORD PTR DS:[ESI]
00A6210E    03BD 22040000    ADD     EDI,DWORD PTR SS:[EBP+422]
00A62114    FF37             PUSH    DWORD PTR DS:[EDI]
00A62116    C607 C3          MOV     BYTE PTR DS:[EDI],0C3
00A62119    FFD7             CALL    EDI
00A6211B    8F07             POP     DWORD PTR DS:[EDI]
00A6211D    50               PUSH    EAX
00A6211E    51               PUSH    ECX
00A6211F    56               PUSH    ESI
00A62120    53               PUSH    EBX
00A62121    8BC8             MOV     ECX,EAX
00A62123    83E9 06          SUB     ECX,6
00A62126    8BB5 52010000    MOV     ESI,DWORD PTR SS:[EBP+152]
00A6212C    33DB             XOR     EBX,EBX
00A6212E    0BC9             OR      ECX,ECX
00A62130    74 2E            JE      SHORT TMPGEncM.00A62160
00A62132    78 2C            JS      SHORT TMPGEncM.00A62160
00A62134    AC               LODS    BYTE PTR DS:[ESI]
00A62135    3C E8            CMP     AL,0E8
00A62137    74 0A            JE      SHORT TMPGEncM.00A62143
00A62139    EB 00            JMP     SHORT TMPGEncM.00A6213B
00A6213B    3C E9            CMP     AL,0E9
00A6213D    74 04            JE      SHORT TMPGEncM.00A62143
00A6213F    43               INC     EBX
00A62140    49               DEC     ECX
00A62141   ^ EB EB          JMP     SHORT TMPGEncM.00A6212E             ; //F4 过
00A62143    8B06             MOV     EAX,DWORD PTR DS:[ESI]
00A62145   /EB 0A           JMP     SHORT TMPGEncM.00A62151
00A62147   |803E 00         CMP     BYTE PTR DS:[ESI],0
00A6214A  ^|75 F3          JNZ     SHORT TMPGEncM.00A6213F
00A6214C   |24 00          AND     AL,0
00A6214E   |C1C0 18        ROL     EAX,18
00A62151   \2BC3          SUB     EAX,EBX
00A62153    8906             MOV     DWORD PTR DS:[ESI],EAX
00A62155    83C3 05          ADD     EBX,5
00A62158    83C6 04          ADD     ESI,4
00A6215B    83E9 05          SUB     ECX,5
00A6215E  ^ EB CE          JMP     SHORT TMPGEncM.00A6212E             ; //F4 过
00A62160    5B               POP     EBX
00A62161    5E               POP     ESI
```
