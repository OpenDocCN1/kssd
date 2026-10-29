
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187

```text
.text:C71166C8             vmpsing_sub_CDF526C8
.text:C71166C8
.text:C71166C8             var_A4 = -0xA4
.text:C71166C8             var_A0 = -0xA0
.text:C71166C8             var_9C = -0x9C
.text:C71166C8             var_98 = -0x98
.text:C71166C8             var_94 = -0x94
.text:C71166C8             var_90 = -0x90
.text:C71166C8             var_8C = -0x8C
.text:C71166C8             var_88 = -0x88
.text:C71166C8             var_84 = -0x84
.text:C71166C8             var_80 = -0x80
.text:C71166C8             var_7C = -0x7C
.text:C71166C8             var_78 = -0x78
.text:C71166C8             var_74 = -0x74
.text:C71166C8             var_70 = -0x70
.text:C71166C8             var_6C = -0x6C
.text:C71166C8             var_68 = -0x68
.text:C71166C8             var_64 = -0x64
.text:C71166C8             var_60 = -0x60
.text:C71166C8             var_5C = -0x5C
.text:C71166C8             var_58 = -0x58
.text:C71166C8             var_54 = -0x54
.text:C71166C8             var_50 = -0x50
.text:C71166C8             var_4C = -0x4C
.text:C71166C8             var_48 = -0x48
.text:C71166C8             var_44 = -0x44
.text:C71166C8             var_40 = -0x40
.text:C71166C8             var_3C = -0x3C
.text:C71166C8             var_34 = -0x34
.text:C71166C8             var_24 = -0x24
.text:C71166C8             var_20 = -0x20
.text:C71166C8             var_1C = -0x1C
.text:C71166C8
.text:C71166C8             ; __unwind { // FBF89000
.text:C71166C8 F0 B5       PUSH            {R4-R7,LR}
.text:C71166CA 03 AF       ADD             R7, SP, #0xC
.text:C71166CC 2D E9 00 0F PUSH.W          {R8-R11}
.text:C71166D0 A3 B0       SUB             SP, SP, #0x8C
.text:C71166D2 6C 46       MOV             R4, SP
.text:C71166D4 6F F3 03 04 BFC.W           R4, #0, #4
.text:C71166D8 A5 46       MOV             SP, R4
.text:C71166DA 06 46       MOV             R6, R0
.text:C71166DC 59 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC71166E6)
.text:C71166DE 91 46       MOV             R9, R2
.text:C71166E0 0D 46       MOV             R5, R1
.text:C71166E2 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C71166E4 00 68       LDR             R0, [R0]                ; __stack_chk_guard
.text:C71166E6 00 68       LDR             R0, [R0]
.text:C71166E8 22 90       STR             R0, [SP,#0xA8+var_20]
.text:C71166EA F9 F7 9D F9 BL              DecString_loc_C5D9FA28
.text:C71166EE 56 48       LDR             R0, =(off_C719AEA0 - 0xC71166F8)
.text:C71166F0 56 49       LDR             R1, =(off_C719AEA4 - 0xC71166FC)
.text:C71166F2 57 4A       LDR             R2, =(off_C719AEA8 - 0xC7116702)
.text:C71166F4 78 44       ADD             R0, PC                  ; off_C719AEA0
.text:C71166F6 57 4B       LDR             R3, =(off_C719AEAC - 0xC7116708)
.text:C71166F8 79 44       ADD             R1, PC                  ; off_C719AEA4
.text:C71166FA DF F8 5C C1 LDR.W           R12, =(off_C719AEB0 - 0xC711670A)
.text:C71166FE 7A 44       ADD             R2, PC                  ; off_C719AEA8
.text:C7116700 DF F8 58 E1 LDR.W           LR, =(off_C719AEB4 - 0xC7116710)
.text:C7116704 7B 44       ADD             R3, PC                  ; off_C719AEAC
.text:C7116706 FC 44       ADD             R12, PC                 ; off_C719AEB0
.text:C7116708 DF F8 54 81 LDR.W           R8, =(off_C719AEB8 - 0xC7116716)
.text:C711670C FE 44       ADD             LR, PC                  ; off_C719AEB4
.text:C711670E 00 68       LDR             R0, [R0]
.text:C7116710 09 68       LDR             R1, [R1]                ; unk_DC0C5BCB
.text:C7116712 F8 44       ADD             R8, PC                  ; off_C719AEB8
.text:C7116714 12 68       LDR             R2, [R2]                ; unk_19559DFE
.text:C7116716 1B 68       LDR             R3, [R3]
.text:C7116718 DC F8 00 40 LDR.W           R4, [R12]
.text:C711671C 02 95       STR             R5, [SP,#0xA8+var_A0]
.text:C711671E DE F8 00 50 LDR.W           R5, [LR]                ; unk_E10C62D9
.text:C7116722 01 96       STR             R6, [SP,#0xA8+var_A4]
.text:C7116724 4F 4E       LDR             R6, =(off_C719AEBC - 0xC711672E)
.text:C7116726 D8 F8 00 C0 LDR.W           R12, [R8]
.text:C711672A 7E 44       ADD             R6, PC                  ; off_C719AEBC
.text:C711672C CD F8 0C 90 STR.W           R9, [SP,#0xA8+var_9C]
.text:C7116730 36 68       LDR             R6, [R6]                ; unk_D748D587
.text:C7116732 04 90       STR             R0, [SP,#0xA8+var_98]
.text:C7116734 4C 48       LDR             R0, =(off_C719AEC0 - 0xC711673A)
.text:C7116736 78 44       ADD             R0, PC                  ; off_C719AEC0
.text:C7116738 D0 F8 00 E0 LDR.W           LR, [R0]
.text:C711673C 05 91       STR             R1, [SP,#0xA8+var_94]
.text:C711673E 4B 49       LDR             R1, =(off_C719AEC4 - 0xC7116746)
.text:C7116740 4F 48       LDR             R0, =(off_C719AED8 - 0xC7116748)
.text:C7116742 79 44       ADD             R1, PC                  ; off_C719AEC4
.text:C7116744 78 44       ADD             R0, PC                  ; off_C719AED8
.text:C7116746 09 68       LDR             R1, [R1]                ; unk_DB4E0EF9
.text:C7116748 06 92       STR             R2, [SP,#0xA8+var_90]
.text:C711674A 49 4A       LDR             R2, =(off_C719AEC8 - 0xC7116750)
.text:C711674C 7A 44       ADD             R2, PC                  ; off_C719AEC8
.text:C711674E 12 68       LDR             R2, [R2]
.text:C7116750 07 93       STR             R3, [SP,#0xA8+var_8C]
.text:C7116752 48 4B       LDR             R3, =(off_C719AECC - 0xC7116758)
.text:C7116754 7B 44       ADD             R3, PC                  ; off_C719AECC
.text:C7116756 1B 68       LDR             R3, [R3]
.text:C7116758 08 94       STR             R4, [SP,#0xA8+var_88]
.text:C711675A 47 4C       LDR             R4, =(off_C719AED0 - 0xC7116760)
.text:C711675C 7C 44       ADD             R4, PC                  ; off_C719AED0
.text:C711675E 24 68       LDR             R4, [R4]                ; unk_1BDC845B
.text:C7116760 09 95       STR             R5, [SP,#0xA8+var_84]
.text:C7116762 46 4D       LDR             R5, =(off_C719AED4 - 0xC7116768)
.text:C7116764 7D 44       ADD             R5, PC                  ; off_C719AED4
.text:C7116766 2D 68       LDR             R5, [R5]                ; unk_2A21A091
.text:C7116768 CD F8 28 C0 STR.W           R12, [SP,#0xA8+var_80]
.text:C711676C D0 F8 00 C0 LDR.W           R12, [R0]               ; unk_F0F4C315
.text:C7116770 0B 96       STR             R6, [SP,#0xA8+var_7C]
.text:C7116772 44 4E       LDR             R6, =(off_C719AEDC - 0xC711677A)
.text:C7116774 44 48       LDR             R0, =(off_C719AEE0 - 0xC711677C)
.text:C7116776 7E 44       ADD             R6, PC                  ; off_C719AEDC
.text:C7116778 78 44       ADD             R0, PC                  ; off_C719AEE0
.text:C711677A 36 68       LDR             R6, [R6]                ; unk_15F9058B
.text:C711677C CD F8 30 E0 STR.W           LR, [SP,#0xA8+var_78]
.text:C7116780 D0 F8 00 E0 LDR.W           LR, [R0]
.text:C7116784 0D 91       STR             R1, [SP,#0xA8+var_74]
.text:C7116786 41 49       LDR             R1, =(off_C719AEE4 - 0xC711678E)
.text:C7116788 45 48       LDR             R0, =(off_C719AEF8 - 0xC7116790)
.text:C711678A 79 44       ADD             R1, PC                  ; off_C719AEE4
.text:C711678C 78 44       ADD             R0, PC                  ; off_C719AEF8
.text:C711678E 09 68       LDR             R1, [R1]
.text:C7116790 0E 92       STR             R2, [SP,#0xA8+var_70]
.text:C7116792 3F 4A       LDR             R2, =(off_C719AEE8 - 0xC7116798)
.text:C7116794 7A 44       ADD             R2, PC                  ; off_C719AEE8
.text:C7116796 D2 F8 00 80 LDR.W           R8, [R2]
.text:C711679A 0F 93       STR             R3, [SP,#0xA8+var_6C]
.text:C711679C 3D 4B       LDR             R3, =(off_C719AEEC - 0xC71167A2)
.text:C711679E 7B 44       ADD             R3, PC                  ; off_C719AEEC
.text:C71167A0 D3 F8 00 90 LDR.W           R9, [R3]                ; unk_E618CE8B
.text:C71167A4 10 94       STR             R4, [SP,#0xA8+var_68]
.text:C71167A6 3C 4C       LDR             R4, =(off_C719AEF0 - 0xC71167AC)
.text:C71167A8 7C 44       ADD             R4, PC                  ; off_C719AEF0
.text:C71167AA D4 F8 00 A0 LDR.W           R10, [R4]               ; unk_1C361051
.text:C71167AE 11 95       STR             R5, [SP,#0xA8+var_64]
.text:C71167B0 3A 4D       LDR             R5, =(off_C719AEF4 - 0xC71167B6)
.text:C71167B2 7D 44       ADD             R5, PC                  ; off_C719AEF4
.text:C71167B4 D5 F8 00 B0 LDR.W           R11, [R5]
.text:C71167B8 CD F8 48 C0 STR.W           R12, [SP,#0xA8+var_60]
.text:C71167BC D0 F8 00 C0 LDR.W           R12, [R0]               ; unk_EC9A6DA7
.text:C71167C0 13 96       STR             R6, [SP,#0xA8+var_5C]
.text:C71167C2 38 4E       LDR             R6, =(off_C719AEFC - 0xC71167CA)
.text:C71167C4 38 48       LDR             R0, =(off_C719AF00 - 0xC71167CC)
.text:C71167C6 7E 44       ADD             R6, PC                  ; off_C719AEFC
.text:C71167C8 78 44       ADD             R0, PC                  ; off_C719AF00
.text:C71167CA 36 68       LDR             R6, [R6]                ; unk_DDD3C1D1
.text:C71167CC CD F8 50 E0 STR.W           LR, [SP,#0xA8+var_58]
.text:C71167D0 D0 F8 00 E0 LDR.W           LR, [R0]                ; unk_320DA0F5
.text:C71167D4 15 91       STR             R1, [SP,#0xA8+var_54]
.text:C71167D6 35 49       LDR             R1, =(off_C719AF04 - 0xC71167DC)
.text:C71167D8 79 44       ADD             R1, PC                  ; off_C719AF04
.text:C71167DA 0A 68       LDR             R2, [R1]                ; unk_2C6D8FB1
.text:C71167DC 34 49       LDR             R1, =(off_C719AF08 - 0xC71167E6)
.text:C71167DE CD F8 58 80 STR.W           R8, [SP,#0xA8+var_50]
.text:C71167E2 79 44       ADD             R1, PC                  ; off_C719AF08
.text:C71167E4 0B 68       LDR             R3, [R1]                ; unk_1BABC103
.text:C71167E6 33 49       LDR             R1, =(off_C719AF0C - 0xC71167F0)
.text:C71167E8 CD F8 5C 90 STR.W           R9, [SP,#0xA8+var_4C]
.text:C71167EC 79 44       ADD             R1, PC                  ; off_C719AF0C
.text:C71167EE 0C 68       LDR             R4, [R1]                ; unk_EEFDC92D
.text:C71167F0 31 49       LDR             R1, =(off_C719AF10 - 0xC71167FA)
.text:C71167F2 CD F8 60 A0 STR.W           R10, [SP,#0xA8+var_48]
.text:C71167F6 79 44       ADD             R1, PC                  ; off_C719AF10
.text:C71167F8 0D 68       LDR             R5, [R1]
.text:C71167FA 30 49       LDR             R1, =(off_C719AF14 - 0xC7116804)
.text:C71167FC CD F8 64 B0 STR.W           R11, [SP,#0xA8+var_44]
.text:C7116800 79 44       ADD             R1, PC                  ; off_C719AF14
.text:C7116802 08 68       LDR             R0, [R1]                ; unk_FFC5974B
.text:C7116804 2E 49       LDR             R1, =(off_C719AF18 - 0xC7116812)
.text:C7116806 CD F8 68 C0 STR.W           R12, [SP,#0xA8+var_40]
.text:C711680A 0D F1 74 0C ADD.W           R12, SP, #0xA8+var_34
.text:C711680E 79 44       ADD             R1, PC                  ; off_C719AF18
.text:C7116810 09 68       LDR             R1, [R1]
.text:C7116812 CD E9 1B 6E STRD.W          R6, LR, [SP,#0xA8+var_3C]
.text:C7116816 8C E8 3C 00 STM.W           R12, {R2-R5}
.text:C711681A 6A 46       MOV             R2, SP
.text:C711681C 21 90       STR             R0, [SP,#0xA8+var_24]
.text:C711681E 4F F4 AA 60 MOV.W           R0, #0x550
.text:C7116822 F9 F7 AF FD BL              VM_Entrance_loc_CDF4C384 ; R:传入不同数字代走不同逻辑
.text:C7116826 27 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC711682E)
.text:C7116828 22 99       LDR             R1, [SP,#0xA8+var_20]
.text:C711682A 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
.text:C711682C 00 68       LDR             R0, [R0]                ; __stack_chk_guard
.text:C711682E 00 68       LDR             R0, [R0]
.text:C7116830 40 1A       SUBS            R0, R0, R1
.text:C7116832 01 BF       ITTTT EQ
.text:C7116834 A7 F1 1C 04 SUBEQ.W         R4, R7, #-var_1C
.text:C7116838 A5 46       MOVEQ           SP, R4
.text:C711683A BD E8 00 0F POPEQ.W         {R8-R11}
.text:C711683E F0 BD       POPEQ           {R4-R7,PC}
```
