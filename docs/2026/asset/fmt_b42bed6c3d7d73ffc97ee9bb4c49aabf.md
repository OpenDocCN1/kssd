
``` 
// 初始化key
.text:C70FE9E8             ; R1:KEY,R2:IV
.text:C70FE9E8             ; int __fastcall AES_initkey_sub_CDB519E8(_DWORD, _DWORD, _DWORD)
.text:C70FE9E8             var_48 = -0x48
.text:C70FE9E8             var_38 = -0x38
.text:C70FE9E8             var_34 = -0x34
.text:C70FE9E8             var_30 = -0x30
.text:C70FE9E8             var_2C = -0x2C
.text:C70FE9E8             var_24 = -0x24
.text:C70FE9E8             var_14 = -0x14
.text:C70FE9E8             var_10 = -0x10
.text:C70FE9E8
.text:C70FE9E8             ; __unwind {
// FBF89000
.text:C70FE9E8 F0 B5       PUSH            {R4-R7,LR}
.text:C70FE9EA 03 AF       ADD             R7, SP, #0xC
.text:C70FE9EC 4D F8 04 BD PUSH.W          {R11}
.text:C70FE9F0 8E B0       SUB             SP, SP, #0x38
.text:C70FE9F2 6C 46       MOV             R4, SP
.text:C70FE9F4 6F F3 03 04 BFC.W           R4, #0, #4
.text:C70FE9F8 A5 46       MOV             SP, R4
.text:C70FE9FA 06 46       MOV             R6, R0
.text:C70FE9FC 19 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC70FEA06)
.text:C70FE9FE 14 46       MOV             R4, R2
.text:C70FEA00 1A 4A       LDR             R2, =(off_C719AA44 - 0xC70FEA0C)
.text:C70FEA02 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C70FEA04 0D 46       MOV             R5, R1
.text:C70FEA06 18 49       LDR             R1, =(off_C719AA40 - 0xC70FEA12)
.text:C70FEA08 7A 44       ADD             R2, PC                  ; off_C719AA44
.text:C70FEA0A 00 68       LDR             R0, [R0]                ; __stack_chk_guard
.text:C70FEA0C 28 23       MOVS            R3, #0x28 ; '('
.text:C70FEA0E 79 44       ADD             R1, PC                  ; off_C719AA40
.text:C70FEA10 00 68       LDR             R0, [R0]
.text:C70FEA12 0D 90       STR             R0, [SP,#0x48+var_14]
.text:C70FEA14 08 68       LDR             R0, [R1]                ; unk_257D1632
.text:C70FEA16 11 68       LDR             R1, [R2]                ; unk_257B6842
.text:C70FEA18 15 4A       LDR             R2, =(unk_C71A71F0 - 0xC70FEA1E)
.text:C70FEA1A 7A 44       ADD             R2, PC                  ; unk_C71A71F0
.text:C70FEA1C 00 92       STR             R2, [SP,#0x48+var_48]
.text:C70FEA1E C0 22       MOVS            R2, #0xC0
.text:C70FEA20 FF F7 C6 FE BL              initkey_sub_C5D8E7B0
.text:C70FEA24 13 48       LDR             R0, =(off_C719AA34 - 0xC70FEA2E)
.text:C70FEA26 14 49       LDR             R1, =(off_C719AA38 - 0xC70FEA30)
.text:C70FEA28 14 4A       LDR             R2, =(off_C719AA3C - 0xC70FEA34)
.text:C70FEA2A 78 44       ADD             R0, PC                  ; off_C719AA34
.text:C70FEA2C 79 44       ADD             R1, PC                  ; off_C719AA38
.text:C70FEA2E 03 68       LDR             R3, [R0]                ; unk_231A4F2F
.text:C70FEA30 7A 44       ADD             R2, PC                  ; off_C719AA3C
.text:C70FEA32 05 96       STR             R6, [SP,#0x48+var_34]
.text:C70FEA34 09 68       LDR             R1, [R1]
.text:C70FEA36 06 95       STR             R5, [SP,#0x48+var_30]
.text:C70FEA38 10 68       LDR             R0, [R2]
.text:C70FEA3A CD E9 07 43 STRD.W          R4, R3, [SP,#0x48+var_2C]
.text:C70FEA3E 09 91       STR             R1, [SP,#0x48+var_24]
.text:C70FEA40 04 A9       ADD             R1, SP, #0x48+var_38
.text:C70FEA42 00 F0 1F F8 BL              AES_initkey_loc_CB175A84
.text:C70FEA46 0E 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC70FEA4E)
.text:C70FEA48 0D 99       LDR             R1, [SP,#0x48+var_14]
.text:C70FEA4A 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C70FEA4C 00 68       LDR             R0, [R0]                ; __stack_chk_guard
.text:C70FEA4E 00 68       LDR             R0, [R0]
.text:C70FEA50 40 1A       SUBS            R0, R0, R1
.text:C70FEA52 01 BF       ITTTT EQ
.text:C70FEA54 A7 F1 10 04 SUBEQ.W         R4, R7, #-var_10
.text:C70FEA58 A5 46       MOVEQ           SP, R4
.text:C70FEA5A 5D F8 04 BB POPEQ.W         {R11}
.text:C70FEA5E F0 BD       POPEQ           {R4-R7,PC}
// AES 加密
.text:C70FFB10             ; R0:初始化的key,R1:数据,R2:大小
.text:C70FFB10             AES_Encdata_sub_C290EB10
.text:C70FFB10             ; __unwind {
// FBF89000
.text:C70FFB10 F0 B5       PUSH            {R4-R7,LR}
.text:C70FFB12 03 AF       ADD             R7, SP, #0xC
.text:C70FFB14 2D E9 00 0F PUSH.W          {R8-R11}
.text:C70FFB18 8B B0       SUB             SP, SP, #0x2C
.text:C70FFB1A 00 EE 10 2A VMOV            S0, R2
.text:C70FFB1E 9F ED 64 1B VLDR            D1, =0.1
.text:C70FFB22 88 46       MOV             R8, R1
.text:C70FFB22             ; End of function AES_Encdata_sub_C290EB10
.text:C70FFB22
.text:C70FFB24             ; START OF FUNCTION CHUNK FOR sub_C7100108
.text:C70FFB24
.text:C70FFB24             loc_C70FFB24
.text:C70FFB24 00 F1 B0 04 ADD.W           R4, R0, #0xB0
.text:C70FFB28 B8 EE 40 0B VCVT.F64.U32    D0, S0
.text:C70FFB2C 6F F0 71 03 MOV             R3, #0xFFFFFF8E
.text:C70FFB30 41 F6 AE 26 MOVW            R6, #0x1AAE
.text:C70FFB34 30 EE 01 0B VADD.F64        D0, D0, D1
.text:C70FFB38 BC EE C0 0B VCVT.U32.F64    S0, D0
.text:C70FFB3C 10 EE 10 2A VMOV            R2, S0
.text:C70FFB40 03 92       STR             R2, [SP,#0x50+var_44]
.text:C70FFB42 5D 4A       LDR             R2, =(off_C719DA5C - 0xC70FFB4C)
.text:C70FFB44 00 90       STR             R0, [SP,#0]
.text:C70FFB46 00 20       MOVS            R0, #0
.text:C70FFB48 7A 44       ADD             R2, PC                  ; off_C719DA5C
.text:C70FFB4A 08 90       STR             R0, [SP,#0x50+var_30]
.text:C70FFB4C 38 20       MOVS            R0, #0x38 ; '8'
.text:C70FFB4E 15 68       LDR             R5, [R2]                ; unk_C719E810
.text:C70FFB50 5B 4A       LDR             R2, =(off_C719DA60 - 0xC70FFB56)
.text:C70FFB52 7A 44       ADD             R2, PC                  ; off_C719DA60
.text:C70FFB54 D2 F8 00 B0 LDR.W           R11, [R2]               ; unk_C719E850
.text:C70FFB58 58 4A       LDR             R2, =(off_C719DA5C - 0xC70FFB5E)
.text:C70FFB5A 7A 44       ADD             R2, PC                  ; off_C719DA5C
.text:C70FFB5C
.text:C70FFB5C             loc_C70FFB5C                            ; CODE XREF: sub_C7100108-51A↓j
.text:C70FFB5C 12 68       LDR             R2, [R2]
.text:C70FFB5E 01 92       STR             R2, [SP,#4]
.text:C70FFB60 CD F8 10 80 STR.W           R8, [SP,#0x50+var_40]
.text:C70FFB64 05 94       STR             R4, [SP,#0x50+var_3C]
.text:C70FFB66 2A E0       B               loc_C70FFBBE
.text:C70FFB68
.text:C70FFB68             loc_C70FFB68
.text:C70FFB68 5A 48       LDR             R0, =(dword_C719E9F4 - 0xC70FFB6E)
.text:C70FFB6A 78 44       ADD             R0, PC                  ; dword_C719E9F4
.text:C70FFB6C 00 68       LDR             R0, [R0]
.text:C70FFB6E 5A 48       LDR             R0, =(off_C719E9F8 - 0xC70FFB74)
.text:C70FFB70 78 44       ADD             R0, PC                  ; off_C719E9F8
.text:C70FFB72 00 68       LDR             R0, [R0]                ; dword_C709A0A0
.text:C70FFB74 0A 90       STR             R0, [SP,#0x50+var_28]
.text:C70FFB76 59 48       LDR             R0, =(off_C719E9FC - 0xC70FFB7C)
.text:C70FFB78 78 44       ADD             R0, PC                  ; off_C719E9FC
.text:C70FFB7A 00 68       LDR             R0, [R0]
.text:C70FFB7C 09 90       STR             R0, [SP,#0x50+var_2C]
.text:C70FFB7E 2E 20       MOVS            R0, #0x2E ; '.'
.text:C70FFB80 1D E0       B               loc_C70FFBBE
.text:C70FFB82
.text:C70FFB82             loc_C70FFB82
.text:C70FFB82 57 48       LDR             R0, =(off_C719EA04 - 0xC70FFB8A)
.text:C70FFB84 57 49       LDR             R1, =(dword_C71A72F8 - 0xC70FFB8C)
.text:C70FFB86 78 44       ADD             R0, PC                  ; off_C719EA04
.text:C70FFB88 79 44       ADD             R1, PC                  ; dword_C71A72F8
.text:C70FFB8A 00 68       LDR             R0, [R0]
.text:C70FFB8C 0A 68       LDR             R2, [R1]
.text:C70FFB8E 56 49       LDR             R1, =(dword_C719EA08 - 0xC70FFB96)
.text:C70FFB90 00 2A       CMP             R2, #0
.text:C70FFB92 79 44       ADD             R1, PC                  ; dword_C719EA08
.text:C70FFB94 09 68       LDR             R1, [R1]
.text:C70FFB96 05 D1       BNE             loc_C70FFBA4
.text:C70FFB98 00 F0 AA F8 BL              getdec_enc_func_sub_CDB52CF0 ; getdecencfunc
.text:C70FFB9C 02 46       MOV             R2, R0
.text:C70FFB9E 53 48       LDR             R0, =(dword_C71A72F8 - 0xC70FFBA4)
.text:C70FFBA0 78 44       ADD             R0, PC                  ; dword_C71A72F8
.text:C70FFBA2 02 60       STR             R2, [R0]
.text:C70FFBA4
.text:C70FFBA4             loc_C70FFBA4
.text:C70FFBA4 00 99       LDR             R1, [SP,#0]
.text:C70FFBA6 50 46       MOV             R0, R10
.text:C70FFBA8 90 47       BLX             R2                      ; 加解密
.text:C70FFBAA 02 99       LDR             R1, [SP,#0x50+var_48]
.text:C70FFBAC A9 F1 02 00 SUB.W           R0, R9, #2
.text:C70FFBB0 6F F0 71 03 MOV             R3, #0xFFFFFF8E
.text:C70FFBB4 10 31       ADDS            R1, #0x10
.text:C70FFBB6 08 91       STR             R1, [SP,#0x50+var_30]
.text:C70FFBB8 06 9C       LDR             R4, [SP,#0x50+var_38]
.text:C70FFBBA 04 F1 10 08 ADD.W           R8, R4, #0x10
.text:C70FFBBE
.text:C70FFBBE             loc_C70FFBBE
.text:C70FFBBE A9 7E       LDRB            R1, [R5,#(dword_C719E828+2 - 0xC719E810)]
.text:C70FFBC0 59 43       MULS            R1, R3
.text:C70FFBC2
.text:C70FFBC2             loc_C70FFBC2
.text:C70FFBC2 81 46       MOV             R9, R0
.text:C70FFBC4
.text:C70FFBC4             loc_C70FFBC4
.text:C70FFBC4 B9 F1 36 0F CMP.W           R9, #0x36 ; '6'
.text:C70FFBC8 2C DD       BLE             loc_C70FFC24
.text:C70FFBCA C8 B2       UXTB            R0, R1
.text:C70FFBCC FC 28       CMP             R0, #0xFC
.text:C70FFBCE F9 D0       BEQ             loc_C70FFBC4
.text:C70FFBD0 B9 F1 38 0F CMP.W           R9, #0x38 ; '8'
.text:C70FFBD4 0F DB       BLT             loc_C70FFBF6
.text:C70FFBD6 4F F0 29 00 MOV.W           R0, #0x29 ; ')'
.text:C70FFBDA F2 D0       BEQ             loc_C70FFBC2
.text:C70FFBDC 01 99       LDR             R1, [SP,#4]
.text:C70FFBDE 91 F8 25 00 LDRB.W          R0, [R1,#0x25]
.text:C70FFBE2 91 F8 26 10 LDRB.W          R1, [R1,#0x26]
.text:C70FFBE6 48 43       MULS            R0, R1
.text:C70FFBE8 C0 B2       UXTB            R0, R0
.text:C70FFBEA 4C 28       CMP             R0, #0x4C ; 'L'
.text:C70FFBEC C9 D1       BNE             loc_C70FFB82
.text:C70FFBEE B5 D7       BVC             loc_C70FFB5C
.text:C70FFBF0 0C 59       LDR             R4, [R1,R4]
.text:C70FFBF2 24 8B       LDRH            R4, [R4,#0x18]
.text:C70FFBF4 0F 41       ASRS            R7, R1
.text:C70FFBF6
.text:C70FFBF6             loc_C70FFBF6
.text:C70FFBF6 40 F6 87 20 MOVW            R0, #0xA87
.text:C70FFBFA 49 46       MOV             R1, R9
.text:C70FFBFC 60 F0 8E FF BL              nop_sub_CEF29B1C
.text:C70FFC00 03 99       LDR             R1, [SP,#0x50+var_44]
.text:C70FFC02 6F F0 71 03 MOV             R3, #0xFFFFFF8E
.text:C70FFC06 08 9A       LDR             R2, [SP,#0x50+var_30]
.text:C70FFC08 8A 42       CMP             R2, R1
.text:C70FFC0A 28 BF       IT CS
.text:C70FFC0C 89 F0 10 00 EORCS.W         R0, R9, #0x10
.text:C70FFC10 02 92       STR             R2, [SP,#0x50+var_48]
.text:C70FFC12 CD E9 06 84 STRD.W          R8, R4, [SP,#0x50+var_38]
.text:C70FFC16 D2 E7       B               loc_C70FFBBE
.text:C70FFC18
.text:C70FFC18             loc_C70FFC18
.text:C70FFC18 80 FD BA EE STC             p14, c14, [R0,#0x2E8]
.text:C70FFC1C 2B AC       ADD             R4, SP, #0xAC
.text:C70FFC1E E6 1E       SUBS            R6, R4, #3
.text:C70FFC20 A7 61       STR             R7, [R4,#0x18]
.text:C70FFC20             ; END OF FUNCTION CHUNK FOR sub_C7100108
.text:C70FFC22 D2          DCB             0xD2
.text:C70FFC23 FA          DCB             0xFA
.text:C70FFC24             ; START OF FUNCTION CHUNK FOR sub_C7100108
.text:C70FFC24
.text:C70FFC24             loc_C70FFC24
.text:C70FFC24 B9 F1 2E 0F CMP.W           R9, #0x2E ; '.'
.text:C70FFC28 23 DB       BLT             loc_C70FFC72
.text:C70FFC2A B9 F1 31 0F CMP.W           R9, #0x31 ; '1'
.text:C70FFC2E BF F6 9B AF BGE.W           loc_C70FFB68
.text:C70FFC32 BB F8 5C 00 LDRH.W          R0, [R11,#(word_C719E8AC - 0xC719E850)]
.text:C70FFC36 B0 42       CMP             R0, R6
.text:C70FFC38 EE D0       BEQ             loc_C70FFC18
.text:C70FFC3A 23 48       LDR             R0, =(dword_C71A72F4 - 0xC70FFC40)
.text:C70FFC3C 78 44       ADD             R0, PC                  ; dword_C71A72F4
.text:C70FFC3E 02 68       LDR             R2, [R0]
.text:C70FFC40 3A B9       CBNZ            R2, loc_C70FFC52
.text:C70FFC42 DD E9 09 10 LDRD.W          R1, R0, [SP,#0x50+var_2C]
.text:C70FFC46 00 F0 53 F8 BL              getdec_enc_func_sub_CDB52CF0 ; getdecencfunc
.text:C70FFC4A 02 46       MOV             R2, R0
.text:C70FFC4C 1F 48       LDR             R0, =(dword_C71A72F4 - 0xC70FFC52)
.text:C70FFC4E 78 44       ADD             R0, PC                  ; dword_C71A72F4
.text:C70FFC50 02 60       STR             R2, [R0]
.text:C70FFC52
.text:C70FFC52             loc_C70FFC52
.text:C70FFC52 DD F8 18 A0 LDR.W           R10, [SP,#0x50+var_38]
.text:C70FFC56 07 99       LDR             R1, [SP,#0x50+var_34]
.text:C70FFC58 50 46       MOV             R0, R10
.text:C70FFC5A 90 47       BLX             R2                      ; 解密
.text:C70FFC5C 1C 48       LDR             R0, =(unk_C719EA00 - 0xC70FFC64)
.text:C70FFC5E 49 46       MOV             R1, R9
.text:C70FFC60 78 44       ADD             R0, PC                  ; unk_C719EA00
.text:C70FFC62 00 68       LDR             R0, [R0]
.text:C70FFC64 40 F6 3E 20 MOVW            R0, #0xA3E
.text:C70FFC68 60 F0 58 FF BL              nop_sub_CEF29B1C
.text:C70FFC6C 6F F0 71 03 MOV             R3, #0xFFFFFF8E
.text:C70FFC70 A5 E7       B               loc_C70FFBBE
.text:C70FFC72
.text:C70FFC72             loc_C70FFC72
.text:C70FFC72 00 20       MOVS            R0, #0
.text:C70FFC74 B9 F1 29 0F CMP.W           R9, #0x29 ; ')'
.text:C70FFC78 08 90       STR             R0, [SP,#0x50+var_30]
.text:C70FFC7A 4F F0 37 00 MOV.W           R0, #0x37 ; '7'
.text:C70FFC7E DD E9 04 84 LDRD.W          R8, R4, [SP,#0x50+var_40]
.text:C70FFC82 3F F4 9C AF BEQ.W           loc_C70FFBBE
.text:C70FFC86 0F 48       LDR             R0, =(off_C719DA60 - 0xC70FFC90)
.text:C70FFC88 44 F2 EA 11 MOVW            R1, #0x41EA
.text:C70FFC8C 78 44       ADD             R0, PC                  ; off_C719DA60
.text:C70FFC8E 00 68       LDR             R0, [R0]                ; unk_C719E850
.text:C70FFC90 B0 F8 46 00 LDRH.W          R0, [R0,#(word_C719E896 - 0xC719E850)]
.text:C70FFC94
.text:C70FFC94             loc_C70FFC94
.text:C70FFC94 88 42       CMP             R0, R1
.text:C70FFC96 FD D1       BNE             loc_C70FFC94
.text:C70FFC98 05 98       LDR             R0, [SP,#0x50+var_3C]   ; 加密结束
.text:C70FFC9A 10 22       MOVS            R2, #0x10
.text:C70FFC9C 07 99       LDR             R1, [SP,#0x50+var_34]
.text:C70FFC9E 0B B0       ADD             SP, SP, #0x2C ; ','
.text:C70FFCA0 BD E8 00 0F POP.W           {R8-R11}
.text:C70FFCA4 BD E8 F0 40 POP.W           {R4-R7,LR}
```
