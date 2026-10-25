
# 某 Windows Mobile? SmartPhone 手机软件的破解

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-316.htm`](https://www.pediy.com/kssd/pediy08/pediy8-316.htm)

某 Windows Mobile? SmartPhone 手机软件的破解

此软件是 SmartPhone 手机上*软件，所以隐去软件相关信息。

### 使用工具
`IDA 4.9`

`UltraEdit-32 11.20`

本人只是对 Crack 感兴趣，没有其他目的，如果软件作者看到，请不要见怪。

本想用 `VS2005` 来动态调试跟踪，但是我怎么也下不了断点，只好放弃。

在手机上安装该软件，按说明运行该软件，该软件用*天的试用期，到期后会有一些小的限制，现在把手机的时间向后调整一个月。

再次运行该程序，首先出来的是试用期已过，请注册提示信息。根据这个提示来找到关键点破解这个软件。先用签名除去软件去掉该软件的数字签名,

现在祭起我们的 `IDA`，分析完成后双击 `Exports` 中的 `start` 项来程序的入口：

```
.text:0004F438                MOV     R12, SP
.text:0004F43C                STMFD   SP!, {R4-R7,R11,R12,LR}
.text:0004F440                ADD     R11, SP, #0x1C
.text:0004F444                SUB     SP, SP, #4
.text:0004F448                MOV     R7, R0
.text:0004F44C                MOV     R6, R1
.text:0004F450                MOV     R5, R2
.text:0004F454                MOV     R4, R3
.text:0004F458                BL      _cinit
.text:0004F45C                MOV     R3, R4          ; nShowCmd
.text:0004F460                MOV     R2, R5          ; lpCmdLine
.text:0004F464                MOV     R1, R6          ; hPrevInstance
.text:0004F468                MOV     R0, R7          ; hInstance
.text:0004F46C                BL      WinMain         //进入 winmain 函数
```

### Winmain

```
.text:00043940                STMFD   SP!, {R4-R6,LR}
.text:00043944                SUB     SP, SP, #0xAE0
.text:00043948                MOV     R4, R0
.text:0004394C                MOV     R0, #8
.text:00043950                MOV     R1, #0x110
.text:00043954                STR     R0, [SP,#0xAF0+var_AC8]
.text:00043958                ADD     R0, SP, #0xAF0+var_AC8  ; LPINITCOMMONCONTROLSEX
.text:0004395C                STR     R1, [SP,#0xAF0+var_AC8.dwICC]
.text:00043960                BL      InitCommonControlsEx
.text:00043964                MOV     R1, #0          ; dwCoInit
.text:00043968                MOV     R0, #0          ; pvReserved
.text:0004396C                BL      CoInitializeEx
.text:00043970                LDR     R5, =unk_563C0
.text:00043974                MOV     R3, #0
.text:00043978                MOV     R2, R4
.text:0004397C                MOV     R1, #0
.text:00043980                MOV     R0, R5
.text:00043984                BL      sub_43ED0
.text:00043988                STR     R4, [R5,#4]
.text:0004398C                LDR     R4, =off_55A2C
.text:00043990                ADD     R2, SP, #0xAF0+var_AE0
.text:00043994                MOV     R1, #0
.text:00043998                LDR     R0, [R4]
.text:0004399C                BL      sub_43720       ; 程序进入点
```

### 程序进入点

```
.text:00043720  ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:00043720
.text:00043720
.text:00043720  sub_43720                               ; CODE XREF: WinMain+5Cp
.text:00043720
.text:00043720  lpData          = -0x894
.text:00043720  cbData          = -0x890
.text:00043720  hKey            = -0x88C
.text:00043720  var_888         = -0x888
.text:00043720  var_884         = -0x884
.text:00043720  Caption         = -0x81C
.text:00043720
.text:00043720                STMFD   SP!, {R4-R9,LR}
.text:00043724                LDR     R12, =0xFFFFF788
.text:00043728                ADD     SP, SP, R12
.text:0004372C                MOV     R5, R0
.text:00043730                MOV     R6, R1
.text:00043734                MOV     R7, R2
.text:00043738                LDR     R4, =unk_56464
.text:0004373C                MOV     R9, #0
.text:00043740                STR     R9, [R7]
.text:00043744                MOV     R8, #5
.text:00043748
.text:00043748  loc_43748                               ; CODE XREF: sub_43720+88j
.text:00043748                MOV     R2, R5          ; lpName
.text:0004374C                MOV     R1, #0          ; bInitialOwner
.text:00043750                MOV     R0, #0          ; lpsa
.text:00043754                BL      CreateMutexW
.text:00043758                CMP     R0, #0
.text:0004375C                STR     R0, [R4]
.text:00043760                BEQ     loc_43880
.text:00043764                BL      GetLastError
.text:00043768                CMP     R0, #0xB7       ; 此程序第一次运行时不出现主窗口，第二次才会出现，所以这里有个判断。
.text:0004376C                BNE     loc_437E0
.text:00043770                MOV     R1, R6          ; lpWindowName
.text:00043774                MOV     R0, R5          ; lpClassName
.text:00043778                BL      FindWindowW
.text:0004377C                CMP     R0, #0
.text:00043780                BNE     loc_437C0
.text:00043784                MOV     R0, #0x1F4      ; dwMilliseconds
.text:00043788                BL      Sleep
.text:0004378C                MOV     R1, R6          ; lpWindowName
.text:00043790                MOV     R0, R5          ; lpClassName
.text:00043794                BL      FindWindowW
.text:00043798                CMP     R0, #0
.text:0004379C                BNE     loc_437C0
.text:000437A0                SUB     R8, R8, #1
.text:000437A4                CMP     R8, #0
.text:000437A8                BGT     loc_43748
.text:000437AC                LDR     R0, =0x80004005
.text:000437B0                MOVL    R12, 0x878
.text:000437B8                ADD     SP, SP, R12
.text:000437BC                LDMFD   SP!, {R4-R9,PC}  ; lpData
.text:000437C0  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:000437C0
.text:000437C0  loc_437C0                               ; CODE XREF: sub_43720+60j
.text:000437C0                                         ; sub_43720+7Cj
.text:000437C0                MOV     R1, #0x400
.text:000437C4                MOV     R3, #0          ; lParam
.text:000437C8                MOV     R2, #0          ; wParam
.text:000437CC                ORR     R1, R1, #0xB    ; Msg
.text:000437D0                BL      SendMessageW
.text:000437D4                MOV     R3, #1
.text:000437D8                STR     R3, [R7]
.text:000437DC                B       loc_4390C
.text:000437E0  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:000437E0
.text:000437E0  loc_437E0                               ; CODE XREF: sub_43720+4Cj
.text:000437E0                ADD     R0, SP, #0x894+hKey  ; 第一次会到这里
.text:000437E4                LDR     R1, =aInit      ; lpSubKey
.text:000437E8                MOV     R3, #loc_20000
.text:000437EC                STR     R0, [SP,#0x894+lpData]
.text:000437F0                ORR     R3, R3, #6      ; samDesired
.text:000437F4                MOV     R2, #0          ; ulOptions
.text:000437F8                MOV     R0, #0x80000002  ; hKey
.text:000437FC                BL      RegOpenKeyExW
.text:00043800                LDR     R4, =aTest
.text:00043804                ADD     R0, SP, #0x894+var_888
.text:00043808                MOV     R3, #4
.text:0004380C                STR     R0, [SP,#0x894+lpData]
.text:00043810                STR     R3, [SP,#0x894+cbData]
.text:00043814                MOV     R2, #0          ; Reserved
.text:00043818                LDR     R0, [SP,#0x894+hKey]  ; hKey
.text:0004381C                MOV     R3, #4          ; dwType
.text:00043820                MOV     R1, R4          ; lpValueName
.text:00043824                BL      RegSetValueExW
.text:00043828                CMP     R0, #5
.text:0004382C                LDR     R0, [SP,#0x894+hKey]  ; hKey
.text:00043830                BNE     loc_43894       ; 下面是检查数字签名，这里跳转了。
.text:00043834                BL      RegCloseKey
```



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



```text
.text:00043C44                MOV     R1, R0,LSL#16

.text:00043C48                LDRH    R0, [SP,#0x58+SystemTime.wDay] ; 取系统时间中的 day 值

.text:00043C4C                SUB     R2, R3, R1,LSR#16

.text:00043C50                MOV     R1, R0,LSL#16

.text:00043C54                ADDS    R3, R2, R1,LSR#16

.text:00043C58                BMI     loc_43CE8

.text:00043C5C                CMP     R3, #0x14       ; 比较使用时间是否超过 20 天

.text:00043C60                BGE     loc_43CE8       ; 超过 20 天就跳转

.text:00043C64                LDR     R0, [SP,#0x58+hKey] ; hKey

.text:00043C68                RSB     R7, R3, #0x14

.text:00043C6C                BL      RegCloseKey

.text:00043C70                MOV     R0, R7

.text:00043C74                ADD     SP, SP, #0x44

.text:00043C78                LDMFD   SP!, {R4-R7,PC} ; lpData
```

.text:00043C7C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00043C7C

.text:00043C7C `loc_43C7C`                               ; CODE XREF: `sub_43B80+70j`

```
.text:00043C7C                ADD     R0, SP, #0x58+var_34 ; lpSystemTime  第一次运行该程序时注册表中没有时间标志就到这里建立
.text:00043C80                BL      GetLocalTime    ; 取系统时间
.text:00043C84                STR     R7, [SP,#0x58+lpSecurityAttributes]
.text:00043C88                ADD     R3, SP, #0x58+var_38
.text:00043C8C                STR     R7, [SP,#0x58+cbData]
.text:00043C90                ADD     R0, SP, #0x58+hKey
.text:00043C94                STR     R3, [SP,#0x58+lpdwDisposition]
.text:00043C98                STR     R0, [SP,#0x58+phkResult]
.text:00043C9C                MOV     R3, #0          ; lpClass
.text:00043CA0                MOV     R2, #0          ; Reserved
.text:00043CA4                STR     R7, [SP,#0x58+lpData]
.text:00043CA8                MOV     R1, R4          ; lpSubKey
.text:00043CAC                MOV     R0, #0x80000001 ; hKey
.text:00043CB0                BL      RegCreateKeyExW  ; 生成时间标志注册项
.text:00043CB4                CMP     R0, #0
.text:00043CB8                MOVNE   R0, #0x14       ; 0X14  20 天的试用期
.text:00043CBC                ADDNE   SP, SP, #0x44
.text:00043CC0                LDMNEFD SP!, {R4-R7,PC}
.text:00043CC4                STR     R6, [SP,#0x58+cbData]
.text:00043CC8                ADD     R0, SP, #0x58+var_34
.text:00043CCC                STR     R0, [SP,#0x58+lpData]
.text:00043CD0                MOV     R3, #3          ; dwType
.text:00043CD4                MOV     R2, #0          ; Reserved
.text:00043CD8                MOV     R1, R5          ; lpValueName
.text:00043CDC                LDR     R0, [SP,#0x58+hKey] ; hKey
.text:00043CE0                BL      RegSetValueExW  ; 设置时间标志值
.text:00043CE4                MOV     R7, #0x14       ; 赋第一次总的可使用的天数
```

.text:00043CE8
.text:00043CE8 `loc_43CE8`                               ; CODE XREF: `sub_43B80+D8j`
.text:00043CE8                                         ; `sub_43B80+E0j`

```
.text:00043CE8                LDR     R0, [SP,#0x58+hKey] ; hKey
.text:00043CEC                BL      RegCloseKey
.text:00043CF0                MOV     R0, R7          ; 赋剩余天数值
.text:00043CF4                ADD     SP, SP, #0x44
.text:00043CF8                LDMFD   SP!, {R4-R7,PC}
```

.text:00043CFC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00043CFC

.text:00043CFC `loc_43CFC`                               ; CODE XREF: `sub_43B80+38j`

```
.text:00043CFC                LDR     R7, [SP,#0x58+var_38]
.text:00043D00                MOV     R0, R7          ; ; 赋剩余天数值
.text:00043D04                ADD     SP, SP, #0x44
.text:00043D08                LDMFD   SP!, {R4-R7,PC}
```

.text:00043D08 ; End of function `sub_43B80`

其实我们根据的是提示注册信息框，这个应该是第一个应该到的地方，去看看：

```text
.text:000399DC ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:000399DC
.text:000399DC
.text:000399DC sub_399DC                               ; CODE XREF: sub_361CC+20p
.text:000399DC                                         ; sub_36DF4+84p ...
.text:000399DC
.text:000399DC dwInitParam     = -0x14
.text:000399DC
.text:000399DC                STMFD   SP!, {R4-R6,LR}
.text:000399E0                SUB     SP, SP, #4      ; dwInitParam
.text:000399E4                MOV     R6, R1
.text:000399E8                MOV     R4, R2
.text:000399EC                LDR     R5, =unk_563C0
.text:000399F0                MOV     R2, R0
.text:000399F4                ADD     R1, R0, #0x10
.text:000399F8                MOV     R0, R5
.text:000399FC                BL      sub_132CC
.text:00039A00                LDR     R0, [R5,#8]     ; hModule
.text:00039A04                MOV     R2, #5          ; lpType
.text:00039A08                MOV     R1, #0x6B       ; lpName
.text:00039A0C                BL      FindResourceW
.text:00039A10                MOV     R1, R0          ; hResInfo
.text:00039A14                LDR     R0, [R5,#8]     ; hModule
.text:00039A18                BL      LoadResource
.text:00039A1C                LDR     R3, =sub_14054  ; lpDialogFunc
.text:00039A20                MOV     R1, R0          ; hDialogTemplate
.text:00039A24                LDR     R0, [R5,#8]     ; hInstance
.text:00039A28                MOV     R2, R6          ; hWndParent
.text:00039A2C                STR     R4, [SP,#0x14+dwInitParam]
.text:00039A30                BL      DialogBoxIndirectParamW  ; 试用期完，请注册信息窗口。
.text:00039A34                ADD     SP, SP, #4
.text:00039A38                LDMFD   SP!, {R4-R6,PC}
.text:00039A38 ; End of function sub_399DC
```

好了，从下逆推，应该到关键的注册码启动时计算比较的地方去了：

```text
.text:0002EE98 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:0002EE98
.text:0002EE98
.text:0002EE98 sub_2EE98                               ; CODE XREF: sub_3586C+20p
.text:0002EE98                                         ; sub_3A394+74p ...
.text:0002EE98
.text:0002EE98 var_10          = -0x10
.text:0002EE98 var_C           = -0xC
.text:0002EE98
.text:0002EE98                STMFD   SP!, {R4,LR}
.text:0002EE9C                SUB     SP, SP, #8
.text:0002EEA0                LDR     R0, =off_514B8
.text:0002EEA4                LDR     R1, [R0]
.text:0002EEA8                ADD     R0, SP, #0x10+var_C
.text:0002EEAC                STR     R1, [SP,#0x10+var_C]
.text:0002EEB0                STR     R1, [SP,#0x10+var_10]
.text:0002EEB4                ADD     R1, SP, #0x10+var_10
.text:0002EEB8                BL      sub_2EC68       ; 取注册表中的注册码
.text:0002EEBC                ADD     R1, SP, #0x10+var_10
.text:0002EEC0                ADD     R0, SP, #0x10+var_C
.text:0002EEC4                BL      sub_2EEEC       ; 计算和比较注册码核心代码
.text:0002EEC8                MOV     R4, R0          ; 计算函数返回值
.text:0002EECC                ADD     R0, SP, #0x10+var_10
.text:0002EED0                BL      sub_1559C
.text:0002EED4                ADD     R0, SP, #0x10+var_C
.text:0002EED8                BL      sub_1559C
.text:0002EEDC                MOV     R0, R4          ; 函数返回值
.text:0002EEE0                ADD     SP, SP, #8
.text:0002EEE4                LDMFD   SP!, {R4,PC}
.text:0002EEE4 ; End of function sub_2EE98
```

计算和比较注册码核心代码:

```text
.text:0002EEEC ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:0002EEEC
.text:0002EEEC
.text:0002EEEC sub_2EEEC                               ; CODE XREF: sub_2ADA4+2CCp
.text:0002EEEC                                         ; sub_2EE98+2Cp ...
.text:0002EEEC
.text:0002EEEC var_6C          = -0x6C
.text:0002EEEC var_68          = -0x68
.text:0002EEEC var_64          = -0x64
.text:0002EEEC var_60          = -0x60
.text:0002EEEC var_5C          = -0x5C
```



```
.text:0002EEEC  var_54          = -0x54
.text:0002EEEC  var_50          = -0x50
.text:0002EEEC  var_4C          = -0x4C
.text:0002EEEC  var_44          = -0x44
.text:0002EEEC  var_3C          = -0x3C
.text:0002EEEC  var_34          = -0x34
.text:0002EEEC  var_2C          = -0x2C
.text:0002EEEC
.text:0002EEEC                  STMFD   SP!, {R4-R11,LR}  ; 计算注册码
.text:0002EEF0                  SUB     SP, SP, #0x48
.text:0002EEF4                  MOV     R5, R1
.text:0002EEF8                  MOV     R4, R0
.text:0002EEFC                  STR     R5, [SP,#0x6C+var_68]
.text:0002EF00                  BL      sub_1A3E4       ; 获取序列号
.text:0002EF04                  MOV     R0, R4
.text:0002EF08                  BL      sub_1A30C
.text:0002EF0C                  MOV     R0, R5
.text:0002EF10                  BL      sub_1A3E4
.text:0002EF14                  MOV     R0, R5
.text:0002EF18                  BL      sub_1A30C
.text:0002EF1C                  LDR     R3, [R4]
.text:0002EF20                  LDR     R0, [R3,#-8]
.text:0002EF24                  CMP     R0, #0
.text:0002EF28                  BEQ     loc_2F780
.text:0002EF2C                  LDR     R0, [R5]
.text:0002EF30                  LDR     R1, [R0,#-8]
.text:0002EF34                  CMP     R1, #0
.text:0002EF38                  BEQ     loc_2F780
.text:0002EF3C                  ADD     R2, SP, #0x6C+var_6C
.text:0002EF40                  MOV     R1, R3
.text:0002EF44                  ADD     R0, SP, #0x6C+var_44
.text:0002EF48                  BL      sub_26934
.text:0002EF4C                  MOV     R1, #4
.text:0002EF50                  ADD     R0, SP, #0x6C+var_44
.text:0002EF54                  BL      sub_4D978
.text:0002EF58                  LDR     R1, =aMixfWtk
.text:0002EF5C                  ADD     R2, SP, #0x6C+var_6C+1
.text:0002EF60                  ADD     R0, SP, #0x6C+var_34
.text:0002EF64                  BL      sub_26934
.text:0002EF68                  LDRB    R3, [SP,#0x6C+var_6C+1]
.text:0002EF6C                  MOV     R1, #0
.text:0002EF70                  ADD     R0, SP, #0x6C+var_64
.text:0002EF74                  STRB    R3, [SP,#0x6C+var_64]
.text:0002EF78                  BL      sub_1A6D4
.text:0002EF7C                  LDRB    R3, [SP,#0x6C+var_6C+1]
.text:0002EF80                  MOV     R1, #0
.text:0002EF84                  ADD     R0, SP, #0x6C+var_54
.text:0002EF88                  STRB    R3, [SP,#0x6C+var_54]
.text:0002EF8C                  BL      sub_1A6D4
.text:0002EF90                  LDR     R3, [SP,#0x6C+var_3C]
.text:0002EF94                  LDR     R0, [SP,#0x6C+var_2C]
.text:0002EF98                  ADD     R1, SP, #0x6C+var_34
.text:0002EF9C                  MOV     R2, #0
.text:0002EFA0                  CMP     R0, R3
.text:0002EFA4                  MOVL    R3, 0xFFFFFFFF
.text:0002EFA8                  BCC     loc_2EFBC
.text:0002EFAC                  ADD     R0, SP, #0x6C+var_64
.text:0002EFB0                  BL      sub_251E8
.text:0002EFB4                  ADD     R0, SP, #0x6C+var_54
.text:0002EFB8                  B       loc_2EFC8
.text:0002EFBC  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002EFBC
.text:0002EFBC  loc_2EFBC                               ; CODE XREF: sub_2EEEC+BCj
.text:0002EFBC                  ADD     R0, SP, #0x6C+var_54
.text:0002EFC0                  BL      sub_251E8
.text:0002EFC4                  ADD     R0, SP, #0x6C+var_64
.text:0002EFC8
.text:0002EFC8  loc_2EFC8                               ; CODE XREF: sub_2EEEC+CCj
.text:0002EFC8                  ADD     R1, SP, #0x6C+var_44
.text:0002EFCC                  MOV     R2, #0
.text:0002EFD0                  MOVL    R3, 0xFFFFFFFF
.text:0002EFD4                  BL      sub_251E8
.text:0002EFD8                  LDR     R10, =unk_500B0
.text:0002EFDC                  MOV     R0, #0
.text:0002EFE0                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002EFE4                  MOV     R8, R0
.text:0002EFE8                  LDR     R9, =0x2E8BA2E9
.text:0002EFEC
.text:0002EFEC  loc_2EFEC                               ; CODE XREF: sub_2EEEC+228j
.text:0002EFEC                                         ; sub_2EEEC+314j ...
.text:0002EFEC                  CMP     R8, R11
.text:0002EFF0                  BCS     loc_2F47C
.text:0002EFF4                  MOV     R0, R8,ASR#1
.text:0002EFF8                  ADD     R1, R8, R0,LSR#30
.text:0002EFFC                  MOV     R2, R1,ASR#2
.text:0002F000                  SUB     R3, R8, R2,LSL#2
.text:0002F004                  CMP     R3, #3
.text:0002F008                  BHI     loc_2F474
.text:0002F00C                  MOV     R0, R3,LSL#1
.text:0002F010                  ADD     R0, R0, PC
.text:0002F014                  LDRH    R0, [R0,#4]
.text:0002F018                  ADD     PC, PC, R0
.text:0002F018  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F01C                  DCW loc_2F024 - off_2F020
.text:0002F01E                  DCW loc_2F118 - off_2F020
.text:0002F020  off_2F020       DCW loc_2F250 - off_2F020  ; DATA XREF: sub_2EEEC+130o
.text:0002F020                                         ; sub_2EEEC+132o ...
.text:0002F022                  DCW loc_2F388 - off_2F020
.text:0002F024  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F024
.text:0002F024  loc_2F024                               ; CODE XREF: sub_2EEEC+12Cj
.text:0002F024                                         ; DATA XREF: sub_2EEEC+130o
.text:0002F024  ; jumptable 0002F018 entry 0
.text:0002F024                  LDR     R6, [SP,#0x6C+var_60]
.text:0002F028                  CMP     R11, R8
.text:0002F02C                  BCC     loc_2F050
.text:0002F030                  CMP     R6, #0
.text:0002F034                  BEQ     loc_2F050
.text:0002F038                  ADD     R0, SP, #0x6C+var_64
.text:0002F03C                  BL      sub_2BAD4
.text:0002F040                  LDR     R6, [SP,#0x6C+var_60]
.text:0002F044                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F048                  ADD     R7, R6, R8,LSL#1
.text:0002F04C                  B       loc_2F054
.text:0002F050  ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F050
.text:0002F050  loc_2F050                               ; CODE XREF: sub_2EEEC+140j
.text:0002F050                                         ; sub_2EEEC+148j
.text:0002F050                  MOV     R7, R10
.text:0002F054
.text:0002F054  loc_2F054                               ; CODE XREF: sub_2EEEC+160j
.text:0002F054                  LDR     R2, =__rt_udiv
.text:0002F058                  MOV     R1, R8
.text:0002F05C                  LDR     R4, [SP,#0x6C+var_4C]
.text:0002F060                  LDR     R3, [R2]
.text:0002F064                  MOV     R0, R4
.text:0002F068                  MOV     LR, PC
.text:0002F06C                  MOV     PC, R3
.text:0002F070                  MOV     R5, R1
.text:0002F074                  CMP     R4, R5
.text:0002F078                  BCC     loc_2F0A4
.text:0002F07C                  LDR     R0, [SP,#0x6C+var_50]
.text:0002F080                  CMP     R0, #0
.text:0002F084                  BEQ     loc_2F0A4
.text:0002F088                  ADD     R0, SP, #0x6C+var_54
.text:0002F08C                  BL      sub_2BAD4
.text:0002F090                  LDR     R3, [SP,#0x6C+var_50]
.text:0002F094                  LDR     R6, [SP,#0x6C+var_60]
.text:0002F098                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F09C                  ADD     R5, R3, R5,LSL#1
.text:0002F0A0                  B       loc_2F0A8
```



```
.text:0002F0A4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F0A4  

.text:0002F0A4 loc\_2F0A4                               ; CODE XREF: sub\_2EEEC+18Cj  
.text:0002F0A4                                         ; sub\_2EEEC+198j  
.text:0002F0A4                 MOV     R5, R10  

.text:0002F0A8  

.text:0002F0A8 loc\_2F0A8                               ; CODE XREF: sub\_2EEEC+1B4j  
.text:0002F0A8                 CMP     R11, R8  
.text:0002F0AC                 BCC     loc\_2F0CC  
.text:0002F0B0                 CMP     R6, #0  
.text:0002F0B4                 BEQ     loc\_2F0CC  
.text:0002F0B8                 ADD     R0, SP, #0x6C+var\_64  
.text:0002F0BC                 BL      sub\_2BAD4  
.text:0002F0C0                 LDR     R3, \[SP,#0x6C+var\_60\]  
.text:0002F0C4                 ADD     R4, R3, R8,LSL#1  
.text:0002F0C8                 B       loc\_2F0D0  

.text:0002F0CC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F0CC  

.text:0002F0CC loc\_2F0CC                               ; CODE XREF: sub\_2EEEC+1C0j  
.text:0002F0CC                                         ; sub\_2EEEC+1C8j  
.text:0002F0CC                 MOV     R4, R10  

.text:0002F0D0  

.text:0002F0D0 loc\_2F0D0                               ; CODE XREF: sub\_2EEEC+1DCj  
.text:0002F0D0                 SMULL   R0, R1, R8, R9  
.text:0002F0D4                 MOV     R2, #0xB  
.text:0002F0D8                 MOV     R0, R1,ASR#1  
.text:0002F0DC                 ADD     R1, R0, R0,LSR#31  
.text:0002F0E0                 LDRH    R0, \[R7\]  
.text:0002F0E4                 MUL     R2, R1, R2  
.text:0002F0E8                 MOV     R1, R0,LSL#16  
.text:0002F0EC                 LDRH    R0, \[R5\]  
.text:0002F0F0                 SUB     R3, R8, R2  
.text:0002F0F4                 RSB     R2, R3, R1,LSR#16  
.text:0002F0F8                 MOV     R1, R0,LSL#16  
.text:0002F0FC                 SUB     R2, R2, R1,LSR#16  
.text:0002F100                 MOV     R0, R2,LSL#16  
.text:0002F104                 MOV     R1, R0,LSR#16  
.text:0002F108                 STRH    R1, \[R4\]  
.text:0002F10C                 ADD     R8, R8, #1  
.text:0002F110                 LDR     R11, \[SP,#0x6C+var\_5C\]  
.text:0002F114                 B       loc\_2EFEC  

.text:0002F118 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F118  

.text:0002F118 loc\_2F118                               ; CODE XREF: sub\_2EEEC+12Cj  
.text:0002F118                                         ; DATA XREF: sub\_2EEEC+132o  
.text:0002F118 ; jumptable 0002F018 entry 1  
.text:0002F118                 LDR     R6, \[SP,#0x6C+var\_60\]  
.text:0002F11C                 CMP     R11, R8  
.text:0002F120                 BCC     loc\_2F144  
.text:0002F124                 CMP     R6, #0  
.text:0002F128                 BEQ     loc\_2F144  
.text:0002F12C                 ADD     R0, SP, #0x6C+var\_64  
.text:0002F130                 BL      sub\_2BAD4  
.text:0002F134                 LDR     R6, \[SP,#0x6C+var\_60\]  
.text:0002F138                 LDR     R11, \[SP,#0x6C+var\_5C\]  
.text:0002F13C                 ADD     R7, R6, R8,LSL#1  
.text:0002F140                 B       loc\_2F148  

.text:0002F144 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F144  

.text:0002F144 loc\_2F144                               ; CODE XREF: sub\_2EEEC+234j  
.text:0002F144                                         ; sub\_2EEEC+23Cj  
.text:0002F144                 MOV     R7, R10  

.text:0002F148  

.text:0002F148 loc\_2F148                               ; CODE XREF: sub\_2EEEC+254j  
.text:0002F148                 LDR     R2, =\_\_rt\_udiv  
.text:0002F14C                 MOV     R1, R8  
.text:0002F150                 LDR     R4, \[SP,#0x6C+var\_4C\]  
.text:0002F154                 LDR     R3, \[R2\]  
.text:0002F158                 MOV     R0, R4  
.text:0002F15C                 MOV     LR, PC  
.text:0002F160                 MOV     PC, R3  
.text:0002F164                 MOV     R5, R1  
.text:0002F168                 CMP     R4, R5  
.text:0002F16C                 BCC     loc\_2F198  
.text:0002F170                 LDR     R0, \[SP,#0x6C+var\_50\]  
.text:0002F174                 CMP     R0, #0  
.text:0002F178                 BEQ     loc\_2F198  
.text:0002F17C                 ADD     R0, SP, #0x6C+var\_54  
.text:0002F180                 BL      sub\_2BAD4  
.text:0002F184                 LDR     R3, \[SP,#0x6C+var\_50\]  
.text:0002F188                 LDR     R6, \[SP,#0x6C+var\_60\]  
.text:0002F18C                 LDR     R11, \[SP,#0x6C+var\_5C\]  
.text:0002F190                 ADD     R5, R3, R5,LSL#1  
.text:0002F194                 B       loc\_2F19C  

.text:0002F198 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F198  

.text:0002F198 loc\_2F198                               ; CODE XREF: sub\_2EEEC+280j  
.text:0002F198                                         ; sub\_2EEEC+28Cj  
.text:0002F198                 MOV     R5, R10  

.text:0002F19C  

.text:0002F19C loc\_2F19C                               ; CODE XREF: sub\_2EEEC+2A8j  
.text:0002F19C                 CMP     R11, R8  
.text:0002F1A0                 BCC     loc\_2F204  
.text:0002F1A4                 CMP     R6, #0  
.text:0002F1A8                 BEQ     loc\_2F204  
.text:0002F1AC                 ADD     R0, SP, #0x6C+var\_64  
.text:0002F1B0                 BL      sub\_2BAD4  
.text:0002F1B4                 LDR     R3, \[SP,#0x6C+var\_60\]  
.text:0002F1B8                 SMULL   R0, R1, R8, R9  
.text:0002F1BC                 MOV     R2, #0xB  
.text:0002F1C0                 ADD     R4, R3, R8,LSL#1  
.text:0002F1C4                 MOV     R0, R1,ASR#1  
.text:0002F1C8                 ADD     R1, R0, R0,LSR#31  
.text:0002F1CC                 LDRH    R0, \[R5\]  
.text:0002F1D0                 MUL     R2, R1, R2  
.text:0002F1D4                 MOV     R1, R0,LSL#16  
.text:0002F1D8                 LDRH    R0, \[R7\]  
.text:0002F1DC                 SUB     R3, R8, R2  
.text:0002F1E0                 ADD     R2, R3, R1,LSR#16  
.text:0002F1E4                 MOV     R1, R0,LSL#16  
.text:0002F1E8                 ADD     R2, R2, R1,LSR#16  
.text:0002F1EC                 MOV     R0, R2,LSL#16  
.text:0002F1F0                 MOV     R1, R0,LSR#16  
.text:0002F1F4                 STRH    R1, \[R4\]  
.text:0002F1F8                 ADD     R8, R8, #1  
.text:0002F1FC                 LDR     R11, \[SP,#0x6C+var\_5C\]  
.text:0002F200                 B       loc\_2EFEC  

.text:0002F204 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F204  

.text:0002F204 loc\_2F204                               ; CODE XREF: sub\_2EEEC+2B4j  
.text:0002F204                                         ; sub\_2EEEC+2BCj  
.text:0002F204                 SMULL   R0, R1, R8, R9  
.text:0002F208                 MOV     R2, #0xB  
.text:0002F20C                 MOV     R4, R10  
.text:0002F210                 MOV     R0, R1,ASR#1  
.text:0002F214                 ADD     R1, R0, R0,LSR#31  
.text:0002F218                 LDRH    R0, \[R5\]  
.text:0002F21C                 MUL     R2, R1, R2  
.text:0002F220                 MOV     R1, R0,LSL#16  
.text:0002F224                 LDRH    R0, \[R7\]  
.text:0002F228                 SUB     R3, R8, R2  
.text:0002F22C                 ADD     R2, R3, R1,LSR#16  
.text:0002F230                 MOV     R1, R0,LSL#16  
.text:0002F234                 ADD     R2, R2, R1,LSR#16  
.text:0002F238                 MOV     R0, R2,LSL#16  
.text:0002F23C                 MOV     R1, R0,LSR#16  
.text:0002F240                 STRH    R1, \[R4\]  
.text:0002F244                 ADD     R8, R8, #1  
.text:0002F248                 LDR     R11, \[SP,#0x6C+var\_5C\]  
.text:0002F24C                 B       loc\_2EFEC  

.text:0002F250 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
```



```
.text:0002F250 loc_2F250                               ; CODE XREF: sub_2EEEC+12Cj
.text:0002F250                                         ; DATA XREF: sub_2EEEC:off_2F020o
.text:0002F250 ; jumptable 0002F018 entry 2
.text:0002F250                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F254                         CMP     R11, R8
.text:0002F258                         BCC     loc_2F27C
.text:0002F25C                         CMP     R6, #0
.text:0002F260                         BEQ     loc_2F27C
.text:0002F264                         ADD     R0, SP, #0x6C+var_64
.text:0002F268                         BL      sub_2BAD4
.text:0002F26C                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F270                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F274                         ADD     R7, R6, R8,LSL#1
.text:0002F278                         B       loc_2F280
.text:0002F27C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F27C
.text:0002F27C loc_2F27C                               ; CODE XREF: sub_2EEEC+36Cj
.text:0002F27C                                         ; sub_2EEEC+374j
.text:0002F27C                         MOV     R7, R10
.text:0002F280
.text:0002F280 loc_2F280                               ; CODE XREF: sub_2EEEC+38Cj
.text:0002F280                         LDR     R2, =__rt_udiv
.text:0002F284                         MOV     R1, R8
.text:0002F288                         LDR     R4, [SP,#0x6C+var_4C]
.text:0002F28C                         LDR     R3, [R2]
.text:0002F290                         MOV     R0, R4
.text:0002F294                         MOV     LR, PC
.text:0002F298                         MOV     PC, R3
.text:0002F29C                         MOV     R5, R1
.text:0002F2A0                         CMP     R4, R5
.text:0002F2A4                         BCC     loc_2F2D0
.text:0002F2A8                         LDR     R0, [SP,#0x6C+var_50]
.text:0002F2AC                         CMP     R0, #0
.text:0002F2B0                         BEQ     loc_2F2D0
.text:0002F2B4                         ADD     R0, SP, #0x6C+var_54
.text:0002F2B8                         BL      sub_2BAD4
.text:0002F2BC                         LDR     R3, [SP,#0x6C+var_50]
.text:0002F2C0                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F2C4                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F2C8                         ADD     R5, R3, R5,LSL#1
.text:0002F2CC                         B       loc_2F2D4
.text:0002F2D0 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F2D0
.text:0002F2D0 loc_2F2D0                               ; CODE XREF: sub_2EEEC+3B8j
.text:0002F2D0                                         ; sub_2EEEC+3C4j
.text:0002F2D0                         MOV     R5, R10
.text:0002F2D4
.text:0002F2D4 loc_2F2D4                               ; CODE XREF: sub_2EEEC+3E0j
.text:0002F2D4                         CMP     R11, R8
.text:0002F2D8                         BCC     loc_2F33C
.text:0002F2DC                         CMP     R6, #0
.text:0002F2E0                         BEQ     loc_2F33C
.text:0002F2E4                         ADD     R0, SP, #0x6C+var_64
.text:0002F2E8                         BL      sub_2BAD4
.text:0002F2EC                         LDR     R3, [SP,#0x6C+var_60]
.text:0002F2F0                         SMULL   R0, R1, R8, R9
.text:0002F2F4                         MOV     R2, #0xB
.text:0002F2F8                         ADD     R4, R3, R8,LSL#1
.text:0002F2FC                         MOV     R0, R1,ASR#1
.text:0002F300                         ADD     R1, R0, R0,LSR#31
.text:0002F304                         LDRH    R0, [R5]
.text:0002F308                         MUL     R2, R1, R2
.text:0002F30C                         MOV     R1, R0,LSL#16
.text:0002F310                         LDRH    R0, [R7]
.text:0002F314                         SUB     R3, R8, R2
.text:0002F318                         RSB     R2, R3, R1,LSR#16
.text:0002F31C                         MOV     R1, R0,LSL#16
.text:0002F320                         ADD     R2, R2, R1,LSR#16
.text:0002F324                         MOV     R0, R2,LSL#16
.text:0002F328                         MOV     R1, R0,LSR#16
.text:0002F32C                         STRH    R1, [R4]
.text:0002F330                         ADD     R8, R8, #1
.text:0002F334                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F338                         B       loc_2EFEC
.text:0002F33C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F33C
.text:0002F33C loc_2F33C                               ; CODE XREF: sub_2EEEC+3ECj
.text:0002F33C                                         ; sub_2EEEC+3F4j
.text:0002F33C                         SMULL   R0, R1, R8, R9
.text:0002F340                         MOV     R2, #0xB
.text:0002F344                         MOV     R4, R10
.text:0002F348                         MOV     R0, R1,ASR#1
.text:0002F34C                         ADD     R1, R0, R0,LSR#31
.text:0002F350                         LDRH    R0, [R5]
.text:0002F354                         MUL     R2, R1, R2
.text:0002F358                         MOV     R1, R0,LSL#16
.text:0002F35C                         LDRH    R0, [R7]
.text:0002F360                         SUB     R3, R8, R2
.text:0002F364                         RSB     R2, R3, R1,LSR#16
.text:0002F368                         MOV     R1, R0,LSL#16
.text:0002F36C                         ADD     R2, R2, R1,LSR#16
.text:0002F370                         MOV     R0, R2,LSL#16
.text:0002F374                         MOV     R1, R0,LSR#16
.text:0002F378                         STRH    R1, [R4]
.text:0002F37C                         ADD     R8, R8, #1
.text:0002F380                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F384                         B       loc_2EFEC
.text:0002F388 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F388
.text:0002F388 loc_2F388                               ; CODE XREF: sub_2EEEC+12Cj
.text:0002F388                                         ; DATA XREF: sub_2EEEC+136o
.text:0002F388 ; jumptable 0002F018 entry 3
.text:0002F388                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F38C                         CMP     R11, R8
.text:0002F390                         BCC     loc_2F3B4
.text:0002F394                         CMP     R6, #0
.text:0002F398                         BEQ     loc_2F3B4
.text:0002F39C                         ADD     R0, SP, #0x6C+var_64
.text:0002F3A0                         BL      sub_2BAD4
.text:0002F3A4                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F3A8                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F3AC                         ADD     R7, R6, R8,LSL#1
.text:0002F3B0                         B       loc_2F3B8
.text:0002F3B4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F3B4
.text:0002F3B4 loc_2F3B4                               ; CODE XREF: sub_2EEEC+4A4j
.text:0002F3B4                                         ; sub_2EEEC+4ACj
.text:0002F3B4                         MOV     R7, R10
.text:0002F3B8
.text:0002F3B8 loc_2F3B8                               ; CODE XREF: sub_2EEEC+4C4j
.text:0002F3B8                         LDR     R2, =__rt_udiv
.text:0002F3BC                         MOV     R1, R8
.text:0002F3C0                         LDR     R4, [SP,#0x6C+var_4C]
.text:0002F3C4                         LDR     R3, [R2]
.text:0002F3C8                         MOV     R0, R4
.text:0002F3CC                         MOV     LR, PC
.text:0002F3D0                         MOV     PC, R3
.text:0002F3D4                         MOV     R5, R1
.text:0002F3D8                         CMP     R4, R5
.text:0002F3DC                         BCC     loc_2F408
.text:0002F3E0                         LDR     R0, [SP,#0x6C+var_50]
.text:0002F3E4                         CMP     R0, #0
.text:0002F3E8                         BEQ     loc_2F408
.text:0002F3EC                         ADD     R0, SP, #0x6C+var_54
.text:0002F3F0                         BL      sub_2BAD4
.text:0002F3F4                         LDR     R3, [SP,#0x6C+var_50]
.text:0002F3F8                         LDR     R6, [SP,#0x6C+var_60]
.text:0002F3FC                         LDR     R11, [SP,#0x6C+var_5C]
.text:0002F400                         ADD     R5, R3, R5,LSL#1
.text:0002F404                         B       loc_2F40C
```



# Disassembly

```text
.text:0002F408 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F408  
.text:0002F408 loc_2F408                               ; CODE XREF: sub_2EEEC+4F0j  
.text:0002F408                                         ; sub_2EEEC+4FCj  
.text:0002F408                 MOV     R5, R10  
.text:0002F40C  
.text:0002F40C loc_2F40C                               ; CODE XREF: sub_2EEEC+518j  
.text:0002F40C                 CMP     R11, R8  
.text:0002F410                 BCC     loc_2F430  
.text:0002F414                 CMP     R6, #0  
.text:0002F418                 BEQ     loc_2F430  
.text:0002F41C                 ADD     R0, SP, #0x6C+var_64  
.text:0002F420                 BL      sub_2BAD4  
.text:0002F424                 LDR     R3, [SP,#0x6C+var_60]  
.text:0002F428                 ADD     R4, R3, R8,LSL#1  
.text:0002F42C                 B       loc_2F434  
.text:0002F430 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F430  
.text:0002F430 loc_2F430                               ; CODE XREF: sub_2EEEC+524j  
.text:0002F430                                         ; sub_2EEEC+52Cj  
.text:0002F430                 MOV     R4, R10  
.text:0002F434  
.text:0002F434 loc_2F434                               ; CODE XREF: sub_2EEEC+540j  
.text:0002F434                 SMULL   R0, R1, R8, R9  
.text:0002F438                 MOV     R2, #0xB  
.text:0002F43C                 MOV     R0, R1,ASR#1  
.text:0002F440                 ADD     R1, R0, R0,LSR#31  
.text:0002F444                 LDRH    R0, [R5]  
.text:0002F448                 MUL     R2, R1, R2  
.text:0002F44C                 MOV     R1, R0,LSL#16  
.text:0002F450                 LDRH    R0, [R7]  
.text:0002F454                 SUB     R3, R8, R2  
.text:0002F458                 SUB     R2, R3, R1,LSR#16  
.text:0002F45C                 MOV     R1, R0,LSL#16  
.text:0002F460                 ADD     R2, R2, R1,LSR#16  
.text:0002F464                 MOV     R0, R2,LSL#16  
.text:0002F468                 MOV     R1, R0,LSR#16  
.text:0002F46C                 STRH    R1, [R4]  
.text:0002F470                 LDR     R11, [SP,#0x6C+var_5C]  
.text:0002F474  
.text:0002F474 loc_2F474                               ; CODE XREF: sub_2EEEC+11Cj  
.text:0002F474                 ADD     R8, R8, #1  
.text:0002F478                 B       loc_2EFEC  
.text:0002F47C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F47C  
.text:0002F47C loc_2F47C                               ; CODE XREF: sub_2EEEC+104j  
.text:0002F47C                 MOV     R0, #0  
.text:0002F480                 LDR     R7, [SP,#0x6C+var_60]  
.text:0002F484                 LDR     R8, [SP,#0x6C+var_50]  
.text:0002F488                 MOV     R5, R0  
.text:0002F48C                 LDR     R9, [SP,#0x6C+var_4C]  
.text:0002F490                 MOV     R6, R0  
.text:0002F494  
.text:0002F494 loc_2F494                               ; CODE XREF: sub_2EEEC+698j  
.text:0002F494                                         ; sub_2EEEC+6BCj ...  
.text:0002F494                 LDR     R2, =0x66666667  
.text:0002F498                 CMP     R6, R11  
.text:0002F49C                 BCS     loc_2F700  
.text:0002F4A0                 SMULL   R0, R1, R6, R2  
.text:0002F4A4                 MOV     R0, R1,ASR#1  
.text:0002F4A8                 ADD     R1, R0, R0,LSR#31  
.text:0002F4AC                 ADD     R2, R1, R1,LSL#2  
.text:0002F4B0                 SUB     R3, R6, R2  
.text:0002F4B4                 CMP     R3, #3  
.text:0002F4B8                 BHI     loc_2F6AC  
.text:0002F4BC                 MOV     R0, R3,LSL#1  
.text:0002F4C0                 ADD     R0, R0, PC  
.text:0002F4C4                 LDRH    R0, [R0,#4]  
.text:0002F4C8                 ADD     PC, PC, R0  
.text:0002F4C8 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F4CC                 DCW loc_2F4D4 - off_2F4D0  
.text:0002F4CE                 DCW loc_2F5AC - off_2F4D0  
.text:0002F4D0 off_2F4D0       DCW loc_2F62C - off_2F4D0 ; DATA XREF: sub_2EEEC+5E0o  
.text:0002F4D0                                         ; sub_2EEEC+5E2o ...  
.text:0002F4D2                 DCW loc_2F668 - off_2F4D0  
.text:0002F4D4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F4D4  
.text:0002F4D4 loc_2F4D4                               ; CODE XREF: sub_2EEEC+5DCj  
.text:0002F4D4                                         ; DATA XREF: sub_2EEEC+5E0o  
.text:0002F4D4 ; jumptable 0002F4C8 entry 0  
.text:0002F4D4                 CMP     R11, R6  
.text:0002F4D8                 BCC     loc_2F504  
.text:0002F4DC                 CMP     R7, #0  
.text:0002F4E0                 BEQ     loc_2F504  
.text:0002F4E4                 ADD     R0, SP, #0x6C+var_64  
.text:0002F4E8                 BL      sub_2BAD4  
.text:0002F4EC                 LDR     R7, [SP,#0x6C+var_60]  
.text:0002F4F0                 LDR     R11, [SP,#0x6C+var_5C]  
.text:0002F4F4                 LDR     R8, [SP,#0x6C+var_50]  
.text:0002F4F8                 ADD     R0, R7, R6,LSL#1  
.text:0002F4FC                 LDR     R9, [SP,#0x6C+var_4C]  
.text:0002F500                 B       loc_2F508  
.text:0002F504 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F504  
.text:0002F504 loc_2F504                               ; CODE XREF: sub_2EEEC+5ECj  
.text:0002F504                                         ; sub_2EEEC+5F4j  
.text:0002F504                 MOV     R0, R10  
.text:0002F508  
.text:0002F508 loc_2F508                               ; CODE XREF: sub_2EEEC+614j  
.text:0002F508                 LDRH    R0, [R0]  
.text:0002F50C                 MOV     R1, R0,LSL#16  
.text:0002F510                 MOV     R2, R1,LSR#16  
.text:0002F514                 MUL     R0, R2, R6  
.text:0002F518                 LDR     R2, =__rt_udiv  
.text:0002F51C                 MOV     R1, #0x64  
.text:0002F520                 LDR     R3, [R2]  
.text:0002F524                 MLA     R5, R0, R1, R5  
.text:0002F528                 MOV     R0, R9  
.text:0002F52C                 MOV     R1, R6  
.text:0002F530                 MOV     LR, PC  
.text:0002F534                 MOV     PC, R3  
.text:0002F538                 MOV     R4, R1  
.text:0002F53C                 CMP     R9, R4  
.text:0002F540                 BCC     loc_2F588  
.text:0002F544                 CMP     R8, #0  
.text:0002F548                 BEQ     loc_2F588  
.text:0002F54C                 ADD     R0, SP, #0x6C+var_54  
.text:0002F550                 BL      sub_2BAD4  
.text:0002F554                 LDR     R8, [SP,#0x6C+var_50]  
.text:0002F558                 LDR     R9, [SP,#0x6C+var_4C]  
.text:0002F55C                 ADD     R6, R6, #1  
.text:0002F560                 ADD     R0, R8, R4,LSL#1  
.text:0002F564                 LDR     R7, [SP,#0x6C+var_60]  
.text:0002F568                 LDRH    R0, [R0]  
.text:0002F56C                 LDR     R11, [SP,#0x6C+var_5C]  
.text:0002F570                 MOV     R1, R0,LSL#16  
.text:0002F574                 MOV     R2, R1,LSR#16  
.text:0002F578                 MUL     R0, R2, R5  
.text:0002F57C                 ADD     R0, R0, #0xEA00  
.text:0002F580                 ADD     R5, R0, #0x60  
.text:0002F584                 B       loc_2F494  
.text:0002F588 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0002F588  
.text:0002F588 loc_2F588                               ; CODE XREF: sub_2EEEC+654j  
.text:0002F588                                         ; sub_2EEEC+65Cj  
.text:0002F588                 MOV     R0, R10  
.text:0002F58C                 LDRH    R0, [R0]  
.text:0002F590                 ADD     R6, R6, #1  
.text:0002F594                 MOV     R1, R0,LSL#16  
.text:0002F598                 MOV     R2, R1,LSR#16
```



```
.text:0002F59C                  MUL     R0, R2, R5
.text:0002F5A0                  ADD     R0, R0, #0xEA00
.text:0002F5A4                  ADD     R5, R0, #0x60
.text:0002F5A8                  B       loc_2F494
.text:0002F5AC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F5AC
.text:0002F5AC loc_2F5AC                               ; CODE XREF: sub_2EEEC+5DCj
.text:0002F5AC                                         ; DATA XREF: sub_2EEEC+5E2o
.text:0002F5AC ; jumptable 0002F4C8 entry 1
.text:0002F5AC                  MOV     R1, R6
.text:0002F5B0                  ADD     R0, SP, #0x6C+var_64
.text:0002F5B4                  BL      sub_2F9A8
.text:0002F5B8                  LDRH    R1, [R0]
.text:0002F5BC                  MOV     R2, R1,LSL#16
.text:0002F5C0                  MOV     R0, R2,LSR#16
.text:0002F5C4                  LDR     R2, =__rt_udiv
.text:0002F5C8                  MUL     R1, R0, R6
.text:0002F5CC                  LDR     R3, [R2]
.text:0002F5D0                  ADD     R0, R1, R1,LSL#2
.text:0002F5D4                  ADD     R4, R5, R0,LSL#2
.text:0002F5D8                  LDR     R0, [SP,#0x6C+var_4C]
.text:0002F5DC                  MOV     R1, R6
.text:0002F5E0                  MOV     LR, PC
.text:0002F5E4                  MOV     PC, R3
.text:0002F5E8                  ADD     R0, SP, #0x6C+var_54
.text:0002F5EC                  BL      sub_2F9A8
.text:0002F5F0                  LDR     R7, [SP,#0x6C+var_60]
.text:0002F5F4                  LDRH    R1, [R0]
.text:0002F5F8                  ADD     R6, R6, #1
.text:0002F5FC                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F600                  MOV     R2, R1,LSL#16
.text:0002F604                  LDR     R1, =0x55555556
.text:0002F608                  MOV     R0, R2,LSR#16
.text:0002F60C                  LDR     R8, [SP,#0x6C+var_50]
.text:0002F610                  SMULL   R1, R3, R0, R1
.text:0002F614                  LDR     R9, [SP,#0x6C+var_4C]
.text:0002F618                  MOV     R2, #0x190
.text:0002F61C                  ADD     R0, R3, R3,LSR#31
.text:0002F620                  MUL     R1, R0, R4
.text:0002F624                  MUL     R5, R1, R2
.text:0002F628                  B       loc_2F494
.text:0002F62C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F62C
.text:0002F62C loc_2F62C                               ; CODE XREF: sub_2EEEC+5DCj
.text:0002F62C                                         ; DATA XREF: sub_2EEEC:off_2F4D0o
.text:0002F62C ; jumptable 0002F4C8 entry 2
.text:0002F62C                  MOV     R1, R6
.text:0002F630                  ADD     R0, SP, #0x6C+var_64
.text:0002F634                  BL      sub_2F9A8
.text:0002F638                  LDR     R7, [SP,#0x6C+var_60]
.text:0002F63C                  LDRH    R1, [R0]
.text:0002F640                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F644                  MOV     R2, R1,LSL#16
.text:0002F648                  LDR     R8, [SP,#0x6C+var_50]
.text:0002F64C                  MOV     R0, R2,LSR#16
.text:0002F650                  LDR     R9, [SP,#0x6C+var_4C]
.text:0002F654                  MUL     R1, R0, R6
.text:0002F658                  MOV     R2, #0x58
.text:0002F65C                  ADD     R6, R6, #1
.text:0002F660                  MLA     R5, R1, R2, R5
.text:0002F664                  B       loc_2F494
.text:0002F668 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F668
.text:0002F668 loc_2F668                               ; CODE XREF: sub_2EEEC+5DCj
.text:0002F668                                         ; DATA XREF: sub_2EEEC+5E6o
.text:0002F668 ; jumptable 0002F4C8 entry 3
.text:0002F668                  MOV     R1, R6
.text:0002F66C                  ADD     R0, SP, #0x6C+var_64
.text:0002F670                  BL      sub_2F9A8
.text:0002F674                  LDR     R7, [SP,#0x6C+var_60]
.text:0002F678                  LDRH    R1, [R0]
.text:0002F67C                  ADD     R6, R6, #1
.text:0002F680                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F684                  MOV     R2, R1,LSL#16
.text:0002F688                  LDR     R1, =0x55555556
.text:0002F68C                  MOV     R0, R2,LSR#16
.text:0002F690                  LDR     R8, [SP,#0x6C+var_50]
.text:0002F694                  SMULL   R1, R3, R0, R1
.text:0002F698                  LDR     R9, [SP,#0x6C+var_4C]
.text:0002F69C                  ADD     R0, R3, R3,LSR#31
.text:0002F6A0                  ADD     R1, R0, R5
.text:0002F6A4                  ADD     R5, R1, #6
.text:0002F6A8                  B       loc_2F494
.text:0002F6AC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F6AC
.text:0002F6AC loc_2F6AC                               ; CODE XREF: sub_2EEEC+5CCj
.text:0002F6AC                  MOV     R1, R6
.text:0002F6B0                  ADD     R0, SP, #0x6C+var_64
.text:0002F6B4                  BL      sub_2F9A8
.text:0002F6B8                  MOV     R4, R0
.text:0002F6BC                  MOV     R1, #0
.text:0002F6C0                  ADD     R0, SP, #0x6C+var_54
.text:0002F6C4                  BL      sub_2F9A8
.text:0002F6C8                  LDR     R7, [SP,#0x6C+var_60]
.text:0002F6CC                  MOV     R3, R0
.text:0002F6D0                  LDRH    R0, [R4]
.text:0002F6D4                  LDR     R11, [SP,#0x6C+var_5C]
.text:0002F6D8                  ADD     R6, R6, #1
.text:0002F6DC                  MOV     R1, R0,LSL#16
.text:0002F6E0                  LDRH    R0, [R3]
.text:0002F6E4                  MOV     R4, R1,LSR#16
.text:0002F6E8                  LDR     R8, [SP,#0x6C+var_50]
.text:0002F6EC                  MOV     R1, R0,LSL#16
.text:0002F6F0                  LDR     R9, [SP,#0x6C+var_4C]
.text:0002F6F4                  MOV     R2, R1,LSR#16
.text:0002F6F8                  MLA     R5, R4, R2, R5
.text:0002F6FC                  B       loc_2F494
.text:0002F700 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F700
.text:0002F700 loc_2F700                               ; CODE XREF: sub_2EEEC+5B0j
.text:0002F700                  LDR     R1, [SP,#0x6C+var_68]
.text:0002F704                  EOR     R0, R5, R5,ASR#31
.text:0002F708                  SUB     R4, R0, R5,ASR#31
.text:0002F70C                  LDR     R0, [R1]        ; wchar_t *
.text:0002F710                  BL      _wtol
.text:0002F714                  CMP     R4, R0          ; 注册码比较
.text:0002F718                  ADD     R0, SP, #0x6C+var_54
.text:0002F71C                  MOV     R1, #1
.text:0002F720                  BNE     loc_2F758       ; 关键跳转，注册码不同就到赋失败标志的地方
.text:0002F724                  BL      sub_1A6D4
.text:0002F728                  MOV     R1, #1
.text:0002F72C                  ADD     R0, SP, #0x6C+var_64
.text:0002F730                  BL      sub_1A6D4
.text:0002F734                  MOV     R1, #1
.text:0002F738                  ADD     R0, SP, #0x6C+var_34
.text:0002F73C                  BL      sub_1A6D4
.text:0002F740                  MOV     R1, #1
.text:0002F744                  ADD     R0, SP, #0x6C+var_44
.text:0002F748                  BL      sub_1A6D4
.text:0002F74C                  MOV     R0, #1          ; 赋注册成功标志 1
.text:0002F750                  ADD     SP, SP, #0x48
.text:0002F754                  LDMFD   SP!, {R4-R11,PC}
.text:0002F758 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F758
.text:0002F758 loc_2F758                               ; CODE XREF: sub_2EEEC+834j
.text:0002F758                  BL      sub_1A6D4
.text:0002F75C                  MOV     R1, #1
.text:0002F760                  ADD     R0, SP, #0x6C+var_64
```



```text
.text:0002F764                 BL      sub_1A6D4  

.text:0002F768                 MOV     R1, #1  

.text:0002F76C                 ADD     R0, SP, #0x6C+var_34  

.text:0002F770                 BL      sub_1A6D4  

.text:0002F774                 MOV     R1, #1  

.text:0002F778                 ADD     R0, SP, #0x6C+var_44  

.text:0002F77C                 BL      sub_1A6D4  

.text:0002F780  

.text:0002F780 loc_2F780                               ; CODE XREF: sub_2EEEC+3Cj  

.text:0002F780                                         ; sub_2EEEC+4Cj  

.text:0002F780                 MOV     R0, #0          ; 赋注册失败标志 0  

.text:0002F784                 ADD     SP, SP, #0x48  

.text:0002F788                 LDMFD   SP!, {R4-R11,PC}  

.text:0002F788 ; End of function sub_2EEEC
```

看看这个沉长的注册码计算过程，想想靠 `IDA` 的静态分析是根本跟踪不出算法来的，只好放弃算法跟踪。但是再好的算法如果只靠一个注册标志来保护程序是根本不行的。虽然这个程序有多处效验注册码也是枉然，看看有多少地方效验：

`p sub_2ADA4+2CC BL sub_2EEEC`  
`Down p sub_2EE98+2C BL sub_2EEEC`  ; 计算注册码  
`Down p sub_36394+9C BL sub_2EEEC`  
`Down p sub_39274+1D8 BL sub_2EEEC`

现在看看怎么破解这个软件，比较和判断的地方：

```text
.text:0002F714                 CMP     R4, R0          ; 注册码比较  
.text:0002F718                 ADD     R0, SP, #0x6C+var_54  
.text:0002F71C                 MOV     R1, #1  
.text:0002F720                 BNE     loc_2F758       ; 关键跳转，注册码不同就到赋失败标志的地方
```

看得出来，只要不然上面的跳转跳走就可以完成注册了，因为程序的启动和运行期比较都是使用这个比较函数且只返回标志，所以破解了这里就全部的破解了这个软件。（模块化编程的弊端？）

光标指到要修改的行，到 `IDA` 的 Hex 窗口中，看看二进制代码是：

`.text:0002F720 BNE loc_2F758` 的二进制代码

`.text:0002F720 0C 00 00 1A`

现在用 `UltraEdit` 打开原程序，搜索 `0C 00 00 1A`（为防止重复代码，可以选择长些的数据进行搜索）：

`001eb20h: 0C 00 00 1A`

根据 `Strong ARM` 指令的 `Opcodes for pocket pcs` 知道 `0c` 代表着跳转的距离，修改 `0c` 为 `00` 就是代表跳转长度为 `0` 这样就不会跳转走了

用 `UltraEdit` 保存程序，复制这个修改的程序到手机的安装目录中，运行破解的程序在注册窗口中把注册码窗口用任意数据填满，确定 `ok`

现在重新运行程序，到关于窗口中看看，呵呵注册成功，再把手机的时间向后调整 `100` 年，运行程序还是成功:)

破解中参考的资料有：

`Strong ARM (SA1110) Opcodes for pocket pcs` http://www.pocketwind.com/crack/armopcodes.htm

`StrongArm` 汇编语言手册 http://www.pocketwind.com/crack.htm

纯 `IDA` 静态分析，不爽，也可能存在错误，请大虾指正。希望能看到 `wince` 下的像 `OD` 般的调试器的出现。

fxyang  
2006.6.4

`4dd466d0e1e0cc123afbe504da2462ca`
