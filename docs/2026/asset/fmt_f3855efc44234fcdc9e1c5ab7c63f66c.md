
结果转换成字符串，随机数+`hmac`

```
   1  7DD6CFBE50FD7930742D168D58099A46D14AE3C7B67341C880 B9BA4EEE79E5FCAEBCE9F68B //组合签名时用到中间50字节
```

## 5.3、加密设备数据与签名组合

压缩`aes`与`base64`加密参与签名的设备数据

```
   1  .text:C7117244             vm_enc_body_sub_C8E49244                ; CODE XREF: getinfo_sub_CDF805D0+6D2↑p
   2  .text:C7117244                                                     ; .text:JNI_OnLoad+18B54↑p
   3  .text:C7117244
   4  .text:C7117244             var_58= -0x58
   5  .text:C7117244             var_54= -0x54
   6  .text:C7117244             var_50= -0x50
   7  .text:C7117244             var_4C= -0x4C
   8  .text:C7117244             var_48= -0x48
   9  .text:C7117244             var_44= -0x44
  10  .text:C7117244             var_40= -0x40
  11  .text:C7117244             var_3C= -0x3C
  12  .text:C7117244             var_38= -0x38
  13  .text:C7117244             var_30= -0x30
  14  .text:C7117244             var_28= -0x28
  15  .text:C7117244             var_20= -0x20
  16  .text:C7117244             var_1C= -0x1C
  17  .text:C7117244             var_18= -0x18
  18  .text:C7117244             arg_0= 8
  19  .text:C7117244
  20  .text:C7117244             ; __unwind { // FBF89000
  21  .text:C7117244 F0 B5       PUSH            {R4-R7,LR}
  22  .text:C7117246 03 AF       ADD             R7, SP, #0xC
  23  .text:C7117248 2D E9 00 07 PUSH.W          {R8-R10}
  24  .text:C711724C 90 B0       SUB             SP, SP, #0x40
  25  .text:C711724E 6C 46       MOV             R4, SP
  26  .text:C7117250 6F F3 03 04 BFC.W           R4, #0, #4
  27  .text:C7117254 A5 46       MOV             SP, R4
  28  .text:C7117256 04 46       MOV             R4, R0
  29  .text:C7117258 29 48       LDR             R0, =(__stack_chk_guard_ptr - 0xC7117262)
  30  .text:C711725A 98 46       MOV             R8, R3
  31  .text:C711725C 15 46       MOV             R5, R2
  32  .text:C711725E 78 44       ADD             R0, PC                  ; __stack_chk_guard_ptr
  33  .text:C7117260 0E 46       MOV             R6, R1
  34  .text:C7117262 00 68       LDR             R0, [R0]                ; __stack_chk_guard
  35  .text:C7117264 00 68       LDR             R0, [R0]
  36  .text:C7117266 0F 90       STR             R0, [SP,#0x58+var_1C]
  37  .text:C7117268 F8 F7 DE FB BL              DecString_loc_C5D9FA28
  38  .text:C711726C 25 48       LDR             R0, =(off_C719AF1C - 0xC7117276)
  39  .text:C711726E 26 49       LDR             R1, =(off_C719AF20 - 0xC711727A)
  40  .text:C7117270 26 4A       LDR             R2, =(off_C719AF24 - 0xC7117280)
  41  .text:C7117272 78 44       ADD             R0, PC                  ; off_C719AF1C
  42  .text:C7117274 26 4B       LDR             R3, =(off_C719AF28 - 0xC7117286)
  43  .text:C7117276 79 44       ADD             R1, PC                  ; off_C719AF20
  44  .text:C7117278 DF F8 98 C0 LDR.W           R12, =(off_C719AF2C - 0xC711728C)
  45  .text:C711727C 7A 44       ADD             R2, PC                  ; off_C719AF24
  46  .text:C711727E DF F8 98 E0 LDR.W           LR, =(off_C719AF30 - 0xC7117290)
  47  .text:C7117282 7B 44       ADD             R3, PC                  ; off_C719AF28
  48  .text:C7117284 DF F8 94 90 LDR.W           R9, =(off_C719AF34 - 0xC7117294)
  49  .text:C7117288 FC 44       ADD             R12, PC                 ; off_C719AF2C
  50  .text:C711728A 00 68       LDR             R0, [R0]                ; unk_1F6B475D
  51  .text:C711728C FE 44       ADD             LR, PC                  ; off_C719AF30
  52  .text:C711728E 09 68       LDR             R1, [R1]                ; unk_E103CAD5
  53  .text:C7117290 F9 44       ADD             R9, PC                  ; off_C719AF34
  54  .text:C7117292 D2 F8 00 A0 LDR.W           R10, [R2]
  55  .text:C7117296 1B 68       LDR             R3, [R3]                ; unk_EF4A43D1
  56  .text:C7117298 02 96       STR             R6, [SP,#0x58+var_50]
  57  .text:C711729A DC F8 00 60 LDR.W           R6, [R12]               ; unk_DFC4E66F
  58  .text:C711729E 01 94       STR             R4, [SP,#0x58+var_54]
  59  .text:C71172A0 DE F8 00 40 LDR.W           R4, [LR]                ; unk_2AD2FC11
  60  .text:C71172A4 03 95       STR             R5, [SP,#0x58+var_4C]
  61  .text:C71172A6 BA 68       LDR             R2, [R7,#arg_0]
  62  .text:C71172A8 D9 F8 00 50 LDR.W           R5, [R9]
  63  .text:C71172AC CD F8 10 80 STR.W           R8, [SP,#0x58+var_48]
  64  .text:C71172B0 05 92       STR             R2, [SP,#0x58+var_44]
  65  .text:C71172B2 1B 4A       LDR             R2, =(off_C719AF38 - 0xC71172B8)
  66  .text:C71172B4 7A 44       ADD             R2, PC                  ; off_C719AF38
  67  .text:C71172B6 12 68       LDR             R2, [R2]                ; unk_25B83385
  68  .text:C71172B8 06 90       STR             R0, [SP,#0x58+var_40]
  69  .text:C71172BA 1A 48       LDR             R0, =(off_C719AF3C - 0xC71172C0)
  70  .text:C71172BC 78 44       ADD             R0, PC                  ; off_C719AF3C
  71  .text:C71172BE 00 68       LDR             R0, [R0]                ; unk_D8E2104D
  72  .text:C71172C0 07 91       STR             R1, [SP,#0x58+var_3C]
  73  .text:C71172C2 19 49       LDR             R1, =(off_C719AF40 - 0xC71172C8)
  74  .text:C71172C4 79 44       ADD             R1, PC                  ; off_C719AF40
  75  .text:C71172C6 09 68       LDR             R1, [R1]
  76  .text:C71172C8 CD E9 08 A3 STRD.W          R10, R3, [SP,#0x58+var_38]
  77  .text:C71172CC CD E9 0A 64 STRD.W          R6, R4, [SP,#0x58+var_30]
  78  .text:C71172D0 CD E9 0C 52 STRD.W          R5, R2, [SP,#0x58+var_28]
  79  .text:C71172D4 6A 46       MOV             R2, SP
  80  .text:C71172D6 0E 90       STR             R0, [SP,#0x58+var_20]
  81  .text:C71172D8 4F F4 3C 70 MOV.W           R0, #0x2F0
  82  .text:C71172DC F9 F7 52 F8 BL              VM_Entrance_loc_CDF4C384 ; R:传入不同数字代走不同逻辑
  83  .text:C71172E0 12 49       LDR             R1, =(__stack_chk_guard_ptr - 0xC71172E8)
  84  .text:C71172E2 00 98       LDR             R0, [SP,#0x58+var_58]
  85  .text:C71172E4 79 44       ADD             R1, PC                  ; __stack_chk_guard_ptr
  86  .text:C71172E6 0F 9A       LDR             R2, [SP,#0x58+var_1C]
  87  .text:C71172E8 09 68       LDR             R1, [R1]                ; __stack_chk_guard
  88  .text:C71172EA 09 68       LDR             R1, [R1]
  89  .text:C71172EC 89 1A       SUBS            R1, R1, R2
```

### 生成`aeskey`

```
   1  7dfd964a-0377-4188-ada7-0758b4f7f63b me5值
```

### AES加密
