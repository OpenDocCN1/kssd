
```asm
.text:0002F0DC                 ADD     R1, R0, R0,LSR#31
.text:0002F0E0                 LDRH    R0, [R7]
.text:0002F0E4                 MUL     R2, R1, R2
.text:0002F0E8                 MOV     R1, R0,LSL#16
.text:0002F0EC                 LDRH    R0, [R5]
.text:0002F0F0                 SUB     R3, R8, R2
.text:0002F0F4                 RSB     R2, R3, R1,LSR#16
.text:0002F0F8                 MOV     R1, R0,LSL#16
.text:0002F0FC                 SUB     R2, R2, R1,LSR#16
.text:0002F100                 MOV     R0, R2,LSL#16
.text:0002F104                 MOV     R1, R0,LSR#16
.text:0002F108                 STRH    R1, [R4]
.text:0002F10C                 ADD     R8, R8, #1
.text:0002F110                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F114                 B       loc_2EFEC
.text:0002F118 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F118
.text:0002F118 loc_2F118                               ; CODE XREF: sub_2EEEC+12Cj
.text:0002F118                                         ; DATA XREF: sub_2EEEC+132o
.text:0002F118 ; jumptable 0002F018 entry 1
.text:0002F118                 LDR     R6, [SP,#0x6C+var_60]
.text:0002F11C                 CMP     R11, R8
.text:0002F120                 BCC     loc_2F144
.text:0002F124                 CMP     R6, #0
.text:0002F128                 BEQ     loc_2F144
.text:0002F12C                 ADD     R0, SP, #0x6C+var_64
.text:0002F130                 BL      sub_2BAD4
.text:0002F134                 LDR     R6, [SP,#0x6C+var_60]
.text:0002F138                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F13C                 ADD     R7, R6, R8,LSL#1
.text:0002F140                 B       loc_2F148
.text:0002F144 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F144
.text:0002F144 loc_2F144                               ; CODE XREF: sub_2EEEC+234j
.text:0002F144                                         ; sub_2EEEC+23Cj
.text:0002F144                 MOV     R7, R10
.text:0002F148
.text:0002F148 loc_2F148                               ; CODE XREF: sub_2EEEC+254j
.text:0002F148                 LDR     R2, =__rt_udiv
.text:0002F14C                 MOV     R1, R8
.text:0002F150                 LDR     R4, [SP,#0x6C+var_4C]
.text:0002F154                 LDR     R3, [R2]
.text:0002F158                 MOV     R0, R4
.text:0002F15C                 MOV     LR, PC
.text:0002F160                 MOV     PC, R3
.text:0002F164                 MOV     R5, R1
.text:0002F168                 CMP     R4, R5
.text:0002F16C                 BCC     loc_2F198
.text:0002F170                 LDR     R0, [SP,#0x6C+var_50]
.text:0002F174                 CMP     R0, #0
.text:0002F178                 BEQ     loc_2F198
.text:0002F17C                 ADD     R0, SP, #0x6C+var_54
.text:0002F180                 BL      sub_2BAD4
.text:0002F184                 LDR     R3, [SP,#0x6C+var_50]
.text:0002F188                 LDR     R6, [SP,#0x6C+var_60]
.text:0002F18C                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F190                 ADD     R5, R3, R5,LSL#1
.text:0002F194                 B       loc_2F19C
.text:0002F198 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F198
.text:0002F198 loc_2F198                               ; CODE XREF: sub_2EEEC+280j
.text:0002F198                                         ; sub_2EEEC+28Cj
.text:0002F198                 MOV     R5, R10
.text:0002F19C
.text:0002F19C loc_2F19C                               ; CODE XREF: sub_2EEEC+2A8j
.text:0002F19C                 CMP     R11, R8
.text:0002F1A0                 BCC     loc_2F204
.text:0002F1A4                 CMP     R6, #0
.text:0002F1A8                 BEQ     loc_2F204
.text:0002F1AC                 ADD     R0, SP, #0x6C+var_64
.text:0002F1B0                 BL      sub_2BAD4
.text:0002F1B4                 LDR     R3, [SP,#0x6C+var_60]
.text:0002F1B8                 SMULL   R0, R1, R8, R9
.text:0002F1BC                 MOV     R2, #0xB
.text:0002F1C0                 ADD     R4, R3, R8,LSL#1
.text:0002F1C4                 MOV     R0, R1,ASR#1
.text:0002F1C8                 ADD     R1, R0, R0,LSR#31
.text:0002F1CC                 LDRH    R0, [R5]
.text:0002F1D0                 MUL     R2, R1, R2
.text:0002F1D4                 MOV     R1, R0,LSL#16
.text:0002F1D8                 LDRH    R0, [R7]
.text:0002F1DC                 SUB     R3, R8, R2
.text:0002F1E0                 ADD     R2, R3, R1,LSR#16
.text:0002F1E4                 MOV     R1, R0,LSL#16
.text:0002F1E8                 ADD     R2, R2, R1,LSR#16
.text:0002F1EC                 MOV     R0, R2,LSL#16
.text:0002F1F0                 MOV     R1, R0,LSR#16
.text:0002F1F4                 STRH    R1, [R4]
.text:0002F1F8                 ADD     R8, R8, #1
.text:0002F1FC                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F200                 B       loc_2EFEC
.text:0002F204 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F204
.text:0002F204 loc_2F204                               ; CODE XREF: sub_2EEEC+2B4j
.text:0002F204                                         ; sub_2EEEC+2BCj
.text:0002F204                 SMULL   R0, R1, R8, R9
.text:0002F208                 MOV     R2, #0xB
.text:0002F20C                 MOV     R4, R10
.text:0002F210                 MOV     R0, R1,ASR#1
.text:0002F214                 ADD     R1, R0, R0,LSR#31
.text:0002F218                 LDRH    R0, [R5]
.text:0002F21C                 MUL     R2, R1, R2
.text:0002F220                 MOV     R1, R0,LSL#16
.text:0002F224                 LDRH    R0, [R7]
.text:0002F228                 SUB     R3, R8, R2
.text:0002F22C                 ADD     R2, R3, R1,LSR#16
.text:0002F230                 MOV     R1, R0,LSL#16
.text:0002F234                 ADD     R2, R2, R1,LSR#16
.text:0002F238                 MOV     R0, R2,LSL#16
.text:0002F23C                 MOV     R1, R0,LSR#16
.text:0002F240                 STRH    R1, [R4]
.text:0002F244                 ADD     R8, R8, #1
.text:0002F248                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F24C                 B       loc_2EFEC
.text:0002F250 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F250
.text:0002F250 loc_2F250                               ; CODE XREF: sub_2EEEC+12Cj
.text:0002F250                                         ; DATA XREF: sub_2EEEC:off_2F020o
.text:0002F250 ; jumptable 0002F018 entry 2
.text:0002F250                 LDR     R6, [SP,#0x6C+var_60]
.text:0002F254                 CMP     R11, R8
.text:0002F258                 BCC     loc_2F27C
.text:0002F25C                 CMP     R6, #0
.text:0002F260                 BEQ     loc_2F27C
.text:0002F264                 ADD     R0, SP, #0x6C+var_64
.text:0002F268                 BL      sub_2BAD4
.text:0002F26C                 LDR     R6, [SP,#0x6C+var_60]
.text:0002F270                 LDR     R11, [SP,#0x6C+var_5C]
.text:0002F274                 ADD     R7, R6, R8,LSL#1
.text:0002F278                 B       loc_2F280
.text:0002F27C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0002F27C
.text:0002F27C loc_2F27C                               ; CODE XREF: sub_2EEEC+36Cj
.text:0002F27C                                         ; sub_2EEEC+374j
.text:0002F27C                 MOV     R7, R10
.text:0002F280
.text:0002F280 loc_2F280                               ; CODE XREF: sub_2EEEC+38Cj
.text:0002F280                 LDR     R2, =__rt_udiv
.text:0002F284                 MOV     R1, R8
.text:0002F288                 LDR     R4, [SP,#0x6C+var_4C]
.text:0002F28C                 LDR     R3, [R2]
.text:0002F290                 MOV     R0, R4
.text:0002F294                 MOV     LR, PC
.text:0002F298                 MOV     PC, R3
.text:0002F29C                 MOV     R5, R1
.text:0002F2A0                 CMP     R4, R5
.text:0002F2A4                 BCC     loc_2F2D0
```
