
```text
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121  122  123  124  125 |
 .text:C71057A4             encbody_hmac_sha256_sub_CDF417A4
 .text:C71057A4
 .text:C71057A4             var_48= -0x48
 .text:C71057A4             var_44= -0x44
 .text:C71057A4             var_40= -0x40
 .text:C71057A4             var_3C= -0x3C
 .text:C71057A4             var_38= -0x38
 .text:C71057A4             var_34= -0x34
 .text:C71057A4             index= -0x30
 .text:C71057A4             var_2C= -0x2C
 .text:C71057A4             var_28= -0x28
 .text:C71057A4             bodydata1= -0x24
 .text:C71057A4             bodydata= -0x20
 .text:C71057A4             arg_1B4= 0x1BC
 .text:C71057A4             arg_290= 0x298
 .text:C71057A4             arg_2E0= 0x2E8
 .text:C71057A4
 .text:C71057A4             ; __unwind {  // FBF89000
 .text:C71057A4 F0 B5       PUSH            {R4-R7,LR}
 .text:C71057A6 03 AF       ADD             R7, SP, #0xC
 .text:C71057A8 2D E9 00 0F PUSH.W          {R8-R11}
 .text:C71057AC 8B B0       SUB             SP, SP, #0x2C
 .text:C71057AE 01 91       STR             R1, [SP,#0x48+var_44]
 .text:C71057B0 06 46       MOV             R6, R0
 .text:C71057B2 74 49       LDR             R1, =(off_C719DAAC - 0xC71057C2)
 .text:C71057B4 06 F1 48 04 ADD.W           R4, R6, #0x48 ; 'H'
 .text:C71057B8 06 F1 40 05 ADD.W           R5, R6, #0x40 ; '@'
 .text:C71057BC 41 20       MOVS            R0, #0x41 ; 'A'
 .text:C71057BE 79 44       ADD             R1, PC                  ; off_C719DAAC
 .text:C71057C0 04 92       STR             R2, [SP,#0x48+var_38]
 .text:C71057C2 D1 F8 00 B0 LDR.W           R11, [R1]               ; unk_C719EEA0
 .text:C71057C6 6C 49       LDR             R1, =(off_C719DAB4 - 0xC71057CC)
 .text:C71057C8 79 44       ADD             R1, PC                  ; off_C719DAB4
 .text:C71057CA D1 F8 00 80 LDR.W           R8, [R1]                ; unk_C719EEF0
 .text:C71057CE 6B 49       LDR             R1, =(off_C719DAB4 - 0xC71057D4)
 .text:C71057D0 79 44       ADD             R1, PC                  ; off_C719DAB4
 .text:C71057D2 09 68       LDR             R1, [R1]                ; unk_C719EEF0
 .text:C71057D4 00 91       STR             R1, [SP,#0x48+var_48]
 .text:C71057D6 6A 49       LDR             R1, =(off_C719DAB4 - 0xC71057DC)
 .text:C71057D8 79 44       ADD             R1, PC                  ; off_C719DAB4
 .text:C71057DA 09 68       LDR             R1, [R1]                ; unk_C719EEF0
 .text:C71057DC 05 91       STR             R1, [SP,#0x48+var_34]
 .text:C71057DE 64 49       LDR             R1, =(off_C719DAB4 - 0xC71057E4)
 .text:C71057E0 79 44       ADD             R1, PC                  ; off_C719DAB4
 .text:C71057E2 09 68       LDR             R1, [R1]                ; unk_C719EEF0
 .text:C71057E4 03 91       STR             R1, [SP,#0x48+var_3C]
 .text:C71057E6 63 49       LDR             R1, =(off_C719DAB4 - 0xC71057EC)
 .text:C71057E8 79 44       ADD             R1, PC                  ; off_C719DAB4
 .text:C71057EA 09 68       LDR             R1, [R1]                ; unk_C719EEF0
 .text:C71057EC 02 91       STR             R1, [SP,#0x48+var_40]
 .text:C71057EE
 .text:C71057EE             loc_C71057EE
 .text:C71057EE 81 46       MOV             R9, R0
 .text:C71057F0
 .text:C71057F0             loc_C71057F0
 .text:C71057F0 08 98       LDR             R0, [SP,#0x48+var_28]
 .text:C71057F2 01 30       ADDS            R0, #1
 .text:C71057F4 06 90       STR             R0, [SP,#0x48+index]
 .text:C71057F6
 .text:C71057F6             loc_C71057F6
 .text:C71057F6 B9 F1 43 0F CMP.W           R9, #0x43 ; 'C'
 .text:C71057FA 06 DA       BGE             loc_C710580A
 .text:C71057FC B9 F1 3F 0F CMP.W           R9, #0x3F ; '?'
 .text:C7105800 0A DA       BGE             loc_C7105818
 .text:C7105802 B9 F1 3E 0F CMP.W           R9, #0x3E ; '>'
 .text:C7105806 16 D1       BNE             loc_C7105836
 .text:C7105808 AD E0       B               loc_C7105966
 .text:C710580A
 .text:C710580A             loc_C710580A
 .text:C710580A B9 F1 4B 0F CMP.W           R9, #0x4B ; 'K'
 .text:C710580E 2A DA       BGE             loc_C7105866
 .text:C7105810 B9 F1 4A 0F CMP.W           R9, #0x4A ; 'J'
 .text:C7105814
 .text:C7105814             loc_C7105814
 .text:C7105814 17 D1       BNE             loc_C7105846
 .text:C7105816 3B E0       B               loc_C7105890
 .text:C7105818
 .text:C7105818             loc_C7105818
 .text:C7105818 B9 F1 41 0F CMP.W           R9, #0x41 ; 'A'
 .text:C710581C 80 F2 8B 80 BGE.W           loc_C7105936
 .text:C7105820 B8 F8 5A 00 LDRH.W          R0, [R8,#(word_C719EF4A - 0xC719EEF0)]
 .text:C7105824 B8 F8 2E 10 LDRH.W          R1, [R8,#(word_C719EF1E - 0xC719EEF0)]
 .text:C7105828 48 43       MULS            R0, R1
 .text:C710582A 48 F2 E2 51 MOVW            R1, #0x85E2
 .text:C710582E 80 B2       UXTH            R0, R0
 .text:C7105830 88 42       CMP             R0, R1
 .text:C7105832 00 F0 69 80 BEQ.W           loc_C7105908
 .text:C7105836
 .text:C7105836             loc_C7105836
 .text:C7105836 9B F8 02 00 LDRB.W          R0, [R11,#(byte_C719EEA2 - 0xC719EEA0)]
 .text:C710583A 18 28       CMP             R0, #0x18
 .text:C710583C 23 D0       BEQ             loc_C7105886
 .text:C710583E 9E 28       CMP             R0, #0x9E
 .text:C7105840 2B 08       LSRS            R3, R5, #0x20 ; ' '
 .text:C7105842 CC AB       ADD             R3, SP, #0x48+arg_2E0
 .text:C7105844 81 93       STR             R3, [SP,#0x48+arg_1B4]
 .text:C7105846
 .text:C7105846             loc_C7105846
 .text:C7105846 30 46       MOV             R0, R6
 .text:C7105848 31 46       MOV             R1, R6
 .text:C710584A D0 47       BLX             R10                     ; sha256_transform,R1:64字节数据
 .text:C710584C D4 E9 00 01 LDRD.W          R0, R1, [R4]
 .text:C7105850 89 F0 7F 09 EOR.W           R9, R9, #0x7F
 .text:C7105854 10 F5 00 70 ADDS.W          R0, R0, #0x200
 .text:C7105858 41 F1 00 01 ADC.W           R1, R1, #0
 .text:C710585C
 .text:C710585C             loc_C710585C
 .text:C710585C C4 E9 00 01 STRD.W          R0, R1, [R4]
 .text:C7105860 00 20       MOVS            R0, #0
 .text:C7105862 28 60       STR             R0, [R5]
 .text:C7105864 C7 E7       B               loc_C71057F6
 .text:C7105866
 .text:C7105866             loc_C7105866
 .text:C7105866 B9 F1 4E 0F CMP.W           R9, #0x4E ; 'N'
 .text:C710586A 1B DD       BLE             loc_C71058A4
 .text:C710586C DD E9 09 10 LDRD.W          R1, R0, [SP,#0x48+bodydata1]
 .text:C7105870 4F F0 3C 09 MOV.W           R9, #0x3C ; '<'
 .text:C7105874 08 70       STRB            R0, [R1]                ; bodydata
 .text:C7105876 28 68       LDR             R0, [R5]                ; offset
 .text:C7105878 01 30       ADDS            R0, #1
 .text:C710587A 28 60       STR             R0, [R5]                ; offset+1 index
 .text:C710587C 40 28       CMP             R0, #0x40 ; '@'         ; 判断是否拷贝结束
 .text:C710587E 08 BF       IT EQ
 .text:C7105880 4F F0 4B 09 MOVEQ.W         R9, #0x4B ; 'K'
 .text:C7105884 B7 E7       B               loc_C71057F6 |
```
