
```
/sbin/.magisk  /proc/mounts
.text:C71006DE B6 F7 27 FB BL              DecString_loc_CEE73D30  ; r
.text:C71006E2 01 46       MOV             R1, R0                  ; modes
.text:C71006E4 20 46       MOV             R0, R4                  ; filename
.text:C71006E6 B4 F7 E0 EF BLX             fopen
.text:C71006EA 83 46       MOV             R11, R0
.text:C71006EC BB F1 00 0F CMP.W           R11, #0
.text:C71006F0 08 D0       BEQ             loc_C7100704
.text:C71006F2 58 46       MOV             R0, R11                 ; stream
.text:C71006F4 B5 F7 22 E9 BLX             feof
.text:C71006F8 98 B1       CBZ             R0, loc_C7100722
.text:C7100732 40 46       MOV             R0, R8                  ; s
.text:C7100734 4F F4 80 71 MOV.W           R1, #0x100              ; n
.text:C7100738 5A 46       MOV             R2, R11                 ; stream
.text:C710073A B4 F7 BC EF BLX             fgets
.text:C710073E 40 46       MOV             R0, R8                  ; s
.text:C7100740 04 95       STR             R5, [SP,#0x138+var_128]
.text:C7100742 CD E9 02 55 STRD.W          R5, R5, [SP,#0x138+var_130]
.text:C7100746 B4 F7 98 EF BLX             strlen
.text:C710074A 04 46       MOV             R4, R0
.text:C710074C 14 F1 10 0F CMN.W           R4, #0x10
.text:C7100750 30 D2       BCS             loc_C71007B4
.text:C7100752 0B 2C       CMP             R4, #0xB
.text:C7100754 06 D2       BCS             loc_C7100764
.text:C7100756 01 9E       LDR             R6, [SP,#0x138+var_134]
.text:C7100758 60 00       LSLS            R0, R4, #1
.text:C710075A 00 2C       CMP             R4, #0
.text:C710075C 8D F8 08 00 STRB.W          R0, [SP,#0x138+var_130]
.text:C7100760 0E D1       BNE             loc_C7100780
.text:C7100762 12 E0       B               loc_C710078A
.text:C7100764
.text:C7100764             loc_C7100764
.text:C7100764 04 F1 10 00 ADD.W           R0, R4, #0x10
.text:C7100768 20 F0 0F 05 BIC.W           R5, R0, #0xF
.text:C710076C 28 46       MOV             R0, R5
.text:C710076E 56 F0 8B FE BL              malloc_sub_CF198488
.text:C7100772 06 46       MOV             R6, R0
.text:C7100774 45 F0 01 00 ORR.W           R0, R5, #1
.text:C7100778 04 96       STR             R6, [SP,#0x138+var_128]
.text:C710077A 00 25       MOVS            R5, #0
.text:C710077C CD E9 02 04 STRD.W          R0, R4, [SP,#0x138+var_130]
.text:C7100780
.text:C7100780             loc_C7100780
.text:C7100780 30 46       MOV             R0, R6
.text:C7100782 41 46       MOV             R1, R8
.text:C7100784 22 46       MOV             R2, R4
.text:C7100786 B4 F7 E4 EF BLX             __aeabi_memcpy
.text:C710078A
.text:C710078A             loc_C710078A
.text:C710078A 35 55       STRB            R5, [R6,R4]
.text:C710078C 50 46       MOV             R0, R10
.text:C710078E 49 46       MOV             R1, R9
.text:C7100790 F8 F7 66 FE BL              memcmp_sub_CDB40460
.text:C7100794 04 46       MOV             R4, R0
.text:C7100796 9D F8 08 00 LDRB.W          R0, [SP,#0x138+var_130]
.text:C710079A C0 07       LSLS            R0, R0, #0x1F
.text:C710079C 1C BF       ITT NE
.text:C710079E 04 98       LDRNE           R0, [SP,#0x138+var_128]
.text:C71007A0 56 F0 B0 FE BLNE            free_sub_BDDF1504
.text:C71007A4 00 2C       CMP             R4, #0
.text:C71007A6 A8 D0       BEQ             loc_C71006FA
.text:C71007A8 58 46       MOV             R0, R11                 ; stream
.text:C71007AA B5 F7 C8 E8 BLX             feof
.text:C71007AE 00 28       CMP             R0, #0
.text:C71007B0 BF D0       BEQ             loc_C7100732
```

### 检测xposed

1. `getSystemClassLoader`
2. `de.robv.android.xposed.XposedBridge`
3. `loadClass`

### 检测多开

1. `io.va.exposed64`
2. `createPackageContext`

### 检测frida
