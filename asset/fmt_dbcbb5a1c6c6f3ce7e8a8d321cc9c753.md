
```asm
.text:00043838                MOVL    R1, 0x9D6C
.text:00043840                ADD     R0, SP, #0x894+Caption
.text:00043844                BL      sub_14580
.text:00043848                MOVL    R1, 0x9DD5
.text:00043850                MOVL    R0, 0x478
.text:00043858                ADD     R0, SP, R0
.text:0004385C                BL      sub_14580
.text:00043860                MOVL    R3, 0x10040     ; uType
.text:00043868                ADD     R2, SP, #0x894+Caption  ; lpCaption
.text:0004386C                MOVL    R1, 0x478
.text:00043874                ADD     R1, SP, R1      ; lpText
.text:00043878                MOV     R0, #0          ; hWnd
.text:0004387C                BL      MessageBoxW
.text:00043880
.text:00043880 loc_43880                      ; CODE XREF: sub_43720+40j
.text:00043880                                ; sub_43720+1F0j
.text:00043880                LDR     R0, =0x80004005
.text:00043884                MOVL    R12, 0x878
.text:0004388C                ADD     SP, SP, R12
.text:00043890                LDMFD   SP!, {R4-R9,PC}
.text:00043894 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00043894
.text:00043894 loc_43894                      ; CODE XREF: sub_43720+110j
.text:00043894                MOV     R1, R4          ; lpValueName
.text:00043898                BL      RegDeleteValueW
.text:0004389C                BL      sub_2EE98       ; 从下面的跟踪和分析可以看出这个函数就是计算注册码的主函数，关键的地方。
.text:000438A0                CMP     R0, #0          ; 关键跳转，上面函数的返回值如果是 0 就是非注册版，跳转到下面的计算使用时间函数
.text:000438A4                BEQ     loc_438C4       ; 为什么是这样的呢，分析下面的时间判断函数就知道了
.text:000438A8                LDR     R1, =unk_56444
.text:000438AC                MOV     R2, #1
.text:000438B0                ADD     R0, R1, #0x1C
.text:000438B4
.text:000438B4 loc_438B4                      ; CODE XREF: sub_43720+19Cj
.text:000438B4                STR     R2, [R1],#4
.text:000438B8                CMP     R1, R0
.text:000438BC                BNE     loc_438B4
.text:000438C0                B       loc_4390C
.text:000438C4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:000438C4
.text:000438C4 loc_438C4                      ; CODE XREF: sub_43720+184j
.text:000438C4                BL      sub_43B80       ; 取时间函数
.text:000438C8                LDR     R1, =unk_56444
.text:000438CC                MOV     R3, R0
.text:000438D0                ADD     R0, R1, #0x1C
.text:000438D4
.text:000438D4 loc_438D4                      ; CODE XREF: sub_43720+1BCj
.text:000438D4                STR     R3, [R1],#4
.text:000438D8                CMP     R1, R0
.text:000438DC                BNE     loc_438D4
.text:000438E0                CMP     R3, #0          ; 修改下面的跳转就能去掉过期后的提示信息。
.text:000438E4                BNE     loc_4390C       ; 比较上面函数的返回值，如果是 0 就是试用到期了
.text:000438E8                ADD     R0, SP, #0x894+var_884
.text:000438EC                BL      sub_36204
.text:000438F0                BL      GetActiveWindow
.text:000438F4                MOV     R1, R0
.text:000438F8                MOV     R2, #0
.text:000438FC                ADD     R0, SP, #0x894+var_884
.text:00043900                BL      sub_399DC       ; 提示试用期已完请注册窗口
.text:00043904                ADD     R0, SP, #0x894+var_884
.text:00043908                BL      sub_3624C
.text:0004390C
.text:0004390C loc_4390C                      ; CODE XREF: sub_43720+BCj
.text:0004390C                                ; sub_43720+1A0j ...
.text:0004390C                CMP     R8, #0
.text:00043910                BLE     loc_43880
.text:00043914                MOV     R0, R9
.text:00043918                MOVL    R12, 0x878
.text:00043920                ADD     SP, SP, R12
.text:00043924                LDMFD   SP!, {R4-R9,PC}
.text:00043924 ; End of function sub_43720
```

先不要到注册计算的关键函数里面去，因为那是最后才知道的。先看看计算时间的函数：

```asm
.text:00043B80 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:00043B80
.text:00043B80
.text:00043B80 sub_43B80                      ; CODE XREF: sub_43720:loc_438C4p
.text:00043B80
.text:00043B80 lpData          = -0x58
.text:00043B80 cbData          = -0x54
.text:00043B80 lpSecurityAttributes= -0x50
.text:00043B80 phkResult       = -0x4C
.text:00043B80 lpdwDisposition = -0x48
.text:00043B80 hKey            = -0x44
.text:00043B80 var_40          = -0x40
.text:00043B80 Type            = -0x3C
.text:00043B80 var_38          = -0x38
.text:00043B80 var_34          = -0x34
.text:00043B80 var_2E          = -0x2E
.text:00043B80 SystemTime      = -0x24
.text:00043B80
.text:00043B80                STMFD   SP!, {R4-R7,LR}  ; 取时间函数
.text:00043B84                SUB     SP, SP, #0x44   ; lpData
.text:00043B88                LDR     R4, =aSoftwareMicros
.text:00043B8C                ADD     R0, SP, #0x58+hKey
.text:00043B90                MOV     R3, #loc_20000
.text:00043B94                STR     R0, [SP,#0x58+lpData]
.text:00043B98                MOV     R7, #0
.text:00043B9C                ORR     R3, R3, #0x19   ; samDesired
.text:00043BA0                STR     R7, [SP,#0x58+hKey]
.text:00043BA4                MOV     R2, #0          ; ulOptions
.text:00043BA8                MOV     R1, R4          ; lpSubKey
.text:00043BAC                MOV     R0, #0x80000001  ; hKey
.text:00043BB0                BL      RegOpenKeyExW
.text:00043BB4                CMP     R0, #0
.text:00043BB8                BNE     loc_43CFC
.text:00043BBC                ADD     R0, SP, #0x58+var_40
.text:00043BC0                LDR     R5, =aProxySharpv1_9
.text:00043BC4                ADD     R1, SP, #0x58+var_34
.text:00043BC8                STR     R0, [SP,#0x58+cbData]
.text:00043BCC                STR     R1, [SP,#0x58+lpData]
.text:00043BD0                MOV     R6, #0x10
.text:00043BD4                LDR     R0, [SP,#0x58+hKey]  ; hKey
.text:00043BD8                ADD     R3, SP, #0x58+Type  ; lpType
.text:00043BDC                MOV     R2, #0          ; lpReserved
.text:00043BE0                STR     R6, [SP,#0x58+var_40]
.text:00043BE4                MOV     R1, R5          ; lpValueName
.text:00043BE8                BL      RegQueryValueExW
.text:00043BEC                CMP     R0, #0          ; 注册表值为空？
.text:00043BF0                BNE     loc_43C7C       ; 取时间在注册表中生成新的值，第一次？
.text:00043BF4                ADD     R0, SP, #0x58+SystemTime  ; lpSystemTime
.text:00043BF8                BL      GetLocalTime
.text:00043BFC                LDRH    R0, [SP,#0x58+SystemTime]
.text:00043C00                LDRH    R3, [SP,#0x58+var_34]
.text:00043C04                MOV     R0, R0,LSL#16
.text:00043C08                LDRH    R5, [SP,#0x58+SystemTime.wMonth]
.text:00043C0C                MOV     R1, R0,LSR#16
.text:00043C10                LDRH    R6, [SP,#0x58+var_34+2]
.text:00043C14                MOV     R2, R3,LSL#16
.text:00043C18                MOV     R0, #0x16C
.text:00043C1C                SUB     R2, R1, R2,LSR#16
.text:00043C20                ORR     R0, R0, #1
.text:00043C24                MUL     R4, R2, R0
.text:00043C28                MOV     R0, R5,LSL#16
.text:00043C2C                MOV     R1, R0,LSR#16
.text:00043C30                MOV     R3, R6,LSL#16
.text:00043C34                SUB     R2, R1, R3,LSR#16
.text:00043C38                MOV     R0, #0x1E
.text:00043C3C                MLA     R3, R2, R0, R4
.text:00043C40                LDRH    R0, [SP,#0x58+var_2E]
```
