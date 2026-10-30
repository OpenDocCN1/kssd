
```text
.text:00043C74                 ADD     SP, SP, #0x44
.text:00043C78                 LDMFD   SP!, {R4-R7,PC} ; lpData
.text:00043C7C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00043C7C
.text:00043C7C loc_43C7C                               ; CODE XREF: sub_43B80+70j
.text:00043C7C                 ADD     R0, SP, #0x58+var_34 ; lpSystemTime  第一次运行该程序时注册表中没有时间标志就到这里建立
.text:00043C80                 BL      GetLocalTime    ; 取系统时间
.text:00043C84                 STR     R7, [SP,#0x58+lpSecurityAttributes]
.text:00043C88                 ADD     R3, SP, #0x58+var_38
.text:00043C8C                 STR     R7, [SP,#0x58+cbData]
.text:00043C90                 ADD     R0, SP, #0x58+hKey
.text:00043C94                 STR     R3, [SP,#0x58+lpdwDisposition]
.text:00043C98                 STR     R0, [SP,#0x58+phkResult]
.text:00043C9C                 MOV     R3, #0          ; lpClass
.text:00043CA0                 MOV     R2, #0          ; Reserved
.text:00043CA4                 STR     R7, [SP,#0x58+lpData]
.text:00043CA8                 MOV     R1, R4          ; lpSubKey
.text:00043CAC                 MOV     R0, #0x80000001 ; hKey
.text:00043CB0                 BL      RegCreateKeyExW ; 生成时间标志注册项
.text:00043CB4                 CMP     R0, #0
.text:00043CB8                 MOVNE   R0, #0x14       ; 0X14  20天的试用期
.text:00043CBC                 ADDNE   SP, SP, #0x44
.text:00043CC0                 LDMNEFD SP!, {R4-R7,PC}
.text:00043CC4                 STR     R6, [SP,#0x58+cbData]
.text:00043CC8                 ADD     R0, SP, #0x58+var_34
.text:00043CCC                 STR     R0, [SP,#0x58+lpData]
.text:00043CD0                 MOV     R3, #3          ; dwType
.text:00043CD4                 MOV     R2, #0          ; Reserved
.text:00043CD8                 MOV     R1, R5          ; lpValueName
.text:00043CDC                 LDR     R0, [SP,#0x58+hKey] ; hKey
.text:00043CE0                 BL      RegSetValueExW  ; 设置时间标志值
.text:00043CE4                 MOV     R7, #0x14       ; 赋第一次总的可使用的天数
.text:00043CE8
.text:00043CE8 loc_43CE8                               ; CODE XREF: sub_43B80+D8j
.text:00043CE8                                         ; sub_43B80+E0j
.text:00043CE8                 LDR     R0, [SP,#0x58+hKey] ; hKey
.text:00043CEC                 BL      RegCloseKey
.text:00043CF0                 MOV     R0, R7          ; 赋剩余天数值
.text:00043CF4                 ADD     SP, SP, #0x44
.text:00043CF8                 LDMFD   SP!, {R4-R7,PC}
.text:00043CFC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00043CFC
.text:00043CFC loc_43CFC                               ; CODE XREF: sub_43B80+38j
.text:00043CFC                 LDR     R7, [SP,#0x58+var_38]
.text:00043D00                 MOV     R0, R7          ; ; 赋剩余天数值
.text:00043D04                 ADD     SP, SP, #0x44
.text:00043D08                 LDMFD   SP!, {R4-R7,PC}
.text:00043D08 ; End of function sub_43B80
```

其实我们根据的是提示注册信息框，这个应该是第一个应该到的地方，去看看：

```text
.text:000399DC ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:000399DC
.text:000399DC sub_399DC                               ; CODE XREF: sub_361CC+20p
.text:000399DC                                         ; sub_36DF4+84p ...
.text:000399DC
.text:000399DC dwInitParam     = -0x14
.text:000399DC
.text:000399DC                 STMFD   SP!, {R4-R6,LR}
.text:000399E0                 SUB     SP, SP, #4      ; dwInitParam
.text:000399E4                 MOV     R6, R1
.text:000399E8                 MOV     R4, R2
.text:000399EC                 LDR     R5, =unk_563C0
.text:000399F0                 MOV     R2, R0
.text:000399F4                 ADD     R1, R0, #0x10
.text:000399F8                 MOV     R0, R5
.text:000399FC                 BL      sub_132CC
.text:00039A00                 LDR     R0, [R5,#8]     ; hModule
.text:00039A04                 MOV     R2, #5          ; lpType
.text:00039A08                 MOV     R1, #0x6B       ; lpName
.text:00039A0C                 BL      FindResourceW
.text:00039A10                 MOV     R1, R0          ; hResInfo
.text:00039A14                 LDR     R0, [R5,#8]     ; hModule
.text:00039A18                 BL      LoadResource
.text:00039A1C                 LDR     R3, =sub_14054  ; lpDialogFunc
.text:00039A20                 MOV     R1, R0          ; hDialogTemplate
.text:00039A24                 LDR     R0, [R5,#8]     ; hInstance
.text:00039A28                 MOV     R2, R6          ; hWndParent
.text:00039A2C                 STR     R4, [SP,#0x14+dwInitParam]
.text:00039A30                 BL      DialogBoxIndirectParamW ; 试用期完，请注册信息窗口。
.text:00039A34                 ADD     SP, SP, #4
.text:00039A38                 LDMFD   SP!, {R4-R6,PC}
.text:00039A38 ; End of function sub_399DC
```

好了，从下逆推，应该到关键的注册码启动时计算比较的地方去了：

```text
.text:0002EE98 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:0002EE98
.text:0002EE98 sub_2EE98                               ; CODE XREF: sub_3586C+20p
.text:0002EE98                                         ; sub_3A394+74p ...
.text:0002EE98
.text:0002EE98 var_10          = -0x10
.text:0002EE98 var_C           = -0xC
.text:0002EE98
.text:0002EE98                 STMFD   SP!, {R4,LR}
.text:0002EE9C                 SUB     SP, SP, #8
.text:0002EEA0                 LDR     R0, =off_514B8
.text:0002EEA4                 LDR     R1, [R0]
.text:0002EEA8                 ADD     R0, SP, #0x10+var_C
.text:0002EEAC                 STR     R1, [SP,#0x10+var_C]
.text:0002EEB0                 STR     R1, [SP,#0x10+var_10]
.text:0002EEB4                 ADD     R1, SP, #0x10+var_10
.text:0002EEB8                 BL      sub_2EC68       ; 取注册表中的注册码
.text:0002EEBC                 ADD     R1, SP, #0x10+var_10
.text:0002EEC0                 ADD     R0, SP, #0x10+var_C
.text:0002EEC4                 BL      sub_2EEEC       ; 计算和比较注册码核心代码
.text:0002EEC8                 MOV     R4, R0          ; 计算函数返回值
.text:0002EECC                 ADD     R0, SP, #0x10+var_10
.text:0002EED0                 BL      sub_1559C
.text:0002EED4                 ADD     R0, SP, #0x10+var_C
.text:0002EED8                 BL      sub_1559C
.text:0002EEDC                 MOV     R0, R4          ; 函数返回值
.text:0002EEE0                 ADD     SP, SP, #8
.text:0002EEE4                 LDMFD   SP!, {R4,PC}
.text:0002EEE4 ; End of function sub_2EE98
```

计算和比较注册码核心代码:

```text
.text:0002EEEC ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:0002EEEC
.text:0002EEEC sub_2EEEC                               ; CODE XREF: sub_2ADA4+2CCp
.text:0002EEEC                                         ; sub_2EE98+2Cp ...
.text:0002EEEC
.text:0002EEEC var_6C          = -0x6C
.text:0002EEEC var_68          = -0x68
.text:0002EEEC var_64          = -0x64
.text:0002EEEC var_60          = -0x60
.text:0002EEEC var_5C          = -0x5C
.text:0002EEEC var_54          = -0x54
.text:0002EEEC var_50          = -0x50
.text:0002EEEC var_4C          = -0x4C
.text:0002EEEC var_44          = -0x44
.text:0002EEEC var_3C          = -0x3C
.text:0002EEEC var_34          = -0x34
.text:0002EEEC var_2C          = -0x2C
.text:0002EEEC
.text:0002EEEC                 STMFD   SP!, {R4-R11,LR} ; 计算注册码
.text:0002EEF0                 SUB     SP, SP, #0x48
.text:0002EEF4                 MOV     R5, R1
.text:0002EEF8                 MOV     R4, R0
.text:0002EEFC                 STR     R5, [SP,#0x6C+var_68]
.text:0002EF00                 BL      sub_1A3E4       ; 获取序列号
.text:0002EF04                 MOV     R0, R4
.text:0002EF08                 BL      sub_1A30C
.text:0002EF0C                 MOV     R0, R5
.text:0002EF10                 BL      sub_1A3E4
.text:0002EF14                 MOV     R0, R5
.text:0002EF18                 BL      sub_1A30C
.text:0002EF1C                 LDR     R3, [R4]
.text:0002EF20                 LDR     R0, [R3,#-8]
```
