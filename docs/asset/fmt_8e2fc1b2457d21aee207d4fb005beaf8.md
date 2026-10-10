
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
