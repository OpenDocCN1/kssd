
# Assembly Listing: Registration and Blacklist Logic

## 1. Registry Key Check: `zlb`

```text
005BAB6C   .  B1 01             MOV CL,1

005BAB6E   .  BA C0B15B00       MOV EDX,unpacked.005BB1C0        ;  ASCII "\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\zlb"

005BAB73   .  8BC3             MOV EAX,EBX

005BAB75   .  E8 8A2AE8FF      CALL unpacked.0043D604

005BAB7A   .  84C0             TEST AL,AL

005BAB7C   .  74 1F            JE SHORT unpacked.005BAB9D

005BAB7E   .  8D4D D4          LEA ECX,DWORD PTR SS:\[EBP-2C\]

005BAB81   .  BA F8B15B00       MOV EDX,unpacked.005BB1F8        ;  ASCII "dream"

005BAB86   .  8BC3             MOV EAX,EBX

005BAB88   .  E8 E72EE8FF      CALL unpacked.0043DA74            ;  从注册表中取出保存的注册码的信息

005BAB8D   .  8B55 D4          MOV EDX,DWORD PTR SS:\[EBP-2C\]

005BAB90   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAB93   .  05 00040000      ADD EAX,400

005BAB98   .  E8 EB98E4FF      CALL unpacked.00404488

005BAB9D   >  B1 01            MOV CL,1
```

## 2. Registry Key Check: `ActiveX Compatibility`

```text
005BAB9F   .  BA 08B25B00       MOV EDX,unpacked.005BB208        ;  ASCII "\\SOFTWARE\\Microsoft\\Internet Explorer\\ActiveX Compatibility"

005BABA4   .  8BC3             MOV EAX,EBX

005BABA6   .  E8 592AE8FF      CALL unpacked.0043D604

005BABAB   .  84C0             TEST AL,AL

005BABAD   .  74 1C            JE SHORT unpacked.005BABCB

005BABAF   .  BA 4CB25B00       MOV EDX,unpacked.005BB24C        ;  ASCII "{6BF52A52-394A-11D3-B153-00C04F79FAA6}"

005BABB4   .  8BC3             MOV EAX,EBX

005BABB6   .  E8 A530E8FF      CALL unpacked.0043DC60

005BABBB   .  84C0             TEST AL,AL

005BABBD   .  74 0C            JE SHORT unpacked.005BABCB

005BABBF   .  BA 4CB25B00       MOV EDX,unpacked.005BB24C        ;  ASCII "{6BF52A52-394A-11D3-B153-00C04F79FAA6}"

005BABC4   .  8BC3             MOV EAX,EBX

005BABC6   .  E8 5D2BE8FF      CALL unpacked.0043D728

005BABCB   >  BA 01000080       MOV EDX,80000001

005BABD0   .  8BC3             MOV EAX,EBX

005BABD2   .  E8 C529E8FF      CALL unpacked.0043D59C
```

## 3. Registry Key Check: `polly`

```text
005BABD7   .  B1 01            MOV CL,1

005BABD9   .  BA 7CB25B00       MOV EDX,unpacked.005BB27C        ;  ASCII "\\SoftWare\\Microsoft\\Windows\\CurrentVersion\\polly"

005BABDE   .  8BC3             MOV EAX,EBX

005BABE0   .  E8 1F2AE8FF      CALL unpacked.0043D604

005BABE5   .  84C0             TEST AL,AL

005BABE7   .  74 1F            JE SHORT unpacked.005BAC08

005BABE9   .  8D4D D0          LEA ECX,DWORD PTR SS:\[EBP-30\]

005BABEC   .  BA B8B25B00       MOV EDX,unpacked.005BB2B8        ;  ASCII "whole"

005BABF1   .  8BC3             MOV EAX,EBX

005BABF3   .  E8 7C2EE8FF      CALL unpacked.0043DA74            ;  从注册表中取出保存用户名的信息

005BABF8   .  8B55 D0          MOV EDX,DWORD PTR SS:\[EBP-30\]

005BABFB   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BABFE   .  05 04040000      ADD EAX,404

005BAC03   .  E8 8098E4FF      CALL unpacked.00404488

005BAC08   >  8BC3             MOV EAX,EBX

005BAC0A   .  E8 5D29E8FF      CALL unpacked.0043D56C

005BAC0F   .  8BC3             MOV EAX,EBX

005BAC11   .  E8 EE89E4FF      CALL unpacked.00403604
```

## 4. DES Decryption and Blacklist Check

```text
005BAC16   .  8D4D CC          LEA ECX,DWORD PTR SS:\[EBP-34\]

005BAC19   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC1C   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC22   .  BA C8B25B00       MOV EDX,unpacked.005BB2C8        ;  ASCII "zlbpolly"

005BAC27   .  E8 44C6EBFF      CALL unpacked.00477270            ;  用 DES_DE( )解密用户名，将用户名还原出来

005BAC2C   .  8B55 CC          MOV EDX,DWORD PTR SS:\[EBP-34\]

005BAC2F   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC32   .  05 04040000      ADD EAX,404

005BAC37   .  E8 4C98E4FF      CALL unpacked.00404488

005BAC3C   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC3F   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC45   .  BA DCB25B00       MOV EDX,unpacked.005BB2DC        ;  丁世龙

005BAC4A   .  E8 0D9CE4FF      CALL unpacked.0040485C

005BAC4F   .  74 69            JE SHORT unpacked.005BACBA

005BAC51   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC54   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC5A   .  BA ECB25B00       MOV EDX,unpacked.005BB2EC        ;  ASCII "8348006"

005BAC5F   .  E8 F89BE4FF      CALL unpacked.0040485C

005BAC64   .  74 54            JE SHORT unpacked.005BACBA

005BAC66   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC69   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC6F   .  BA FCB25B00       MOV EDX,unpacked.005BB2FC        ;  ASCII "zhaoguihu"

005BAC74   .  E8 E39BE4FF      CALL unpacked.0040485C

005BAC79   .  74 3F            JE SHORT unpacked.005BACBA

005BAC7B   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC7E   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC84   .  BA 10B35B00       MOV EDX,unpacked.005BB310        ;  ASCII "wwwjjj"

005BAC89   .  E8 CE9BE4FF      CALL unpacked.0040485C

005BAC8E   .  74 2A            JE SHORT unpacked.005BACBA

005BAC90   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAC93   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BAC99   .  BA 20B35B00       MOV EDX,unpacked.005BB320        ;  吴小龙

005BAC9E   .  E8 B99BE4FF      CALL unpacked.0040485C

005BACA3   .  74 15            JE SHORT unpacked.005BACBA

005BACA5   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BACA8   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BACAE   .  BA 30B35B00       MOV EDX,unpacked.005BB330        ;  刘光军

005BACB3   .  E8 A49BE4FF      CALL unpacked.0040485C

005BACB8   .  75 1A            JNZ SHORT unpacked.005BACD4

005BACBA   >  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BACBD   .  E8 EAFCFFFF      CALL unpacked.005BA9AC

005BACC2   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BACC5   .  05 04040000      ADD EAX,404

005BACCA   .  BA 40B35B00       MOV EDX,unpacked.005BB340

005BACCF   .  E8 B497E4FF      CALL unpacked.00404488
```

## 5. MD5 Hashing and DES Encryption

```text
005BACD4   >  8D55 C0          LEA EDX,DWORD PTR SS:\[EBP-40\]     ;  上面的名字都是一些黑名单，对比后跳到这里继续对用户名处理

005BACD7   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BACDA   .  8B80 04040000    MOV EAX,DWORD PTR DS:\[EAX+404\]

005BACE0   .  E8 8FD1EBFF      CALL unpacked.00477E74              ;  MD5(用户名)

005BACE5   .  8B45 C0          MOV EAX,DWORD PTR SS:\[EBP-40\]

005BACE8   .  8D4D C4          LEA ECX,DWORD PTR SS:\[EBP-3C\]

005BACEB   .  8B55 FC          MOV EDX,DWORD PTR SS:\[EBP-4\]

005BACEE   .  8B92 08040000    MOV EDX,DWORD PTR DS:\[EDX+408\]

005BACF4   .  E8 8FC3EBFF      CALL unpacked.00477088              ;  DES_EN()，Key＝机器码

005BACF9   .  8B45 C4          MOV EAX,DWORD PTR SS:\[EBP-3C\]

005BACFC   .  8D55 C8          LEA EDX,DWORD PTR SS:\[EBP-38\]

005BACFF   .  E8 70D1EBFF      CALL unpacked.00477E74              ;  MD5( )

005BAD04   .  8B55 C8          MOV EDX,DWORD PTR SS:\[EBP-38\]

005BAD07   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAD0A   .  05 10040000      ADD EAX,410

005BAD0F   .  E8 7497E4FF      CALL unpacked.00404488

005BAD14   .  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]

005BAD17   .  8B80 00040000    MOV EAX,DWORD PTR DS:\[EAX+400\]

005BAD1D   .  E8 EE99E4FF      CALL unpacked.00404710              ;  判断注册表中保存注册码信息的长度，应该有四种可能：10，14，16，12 分别对应注册码的长度是 16、20、24、32

005BAD22   .  83E8 0A          SUB EAX,0A                           ;  Switch (cases A..10)

005BAD25   .  74 7E            JE SHORT unpacked.005BADA5

005BAD27   .  83E8 02          SUB EAX,2

005BAD2A   .  74 17            JE SHORT unpacked.005BAD43

005BAD2C   .  83E8 02          SUB EAX,2

005BAD2F   .  0F84 52010000    JE unpacked.005BAE87

005BAD35   .  83E8 02          SUB EAX,2

005BAD38   .  0F84 2B020000    JE unpacked.005BAF69

005BAD3E   .  E9 E6030000      JMP unpacked.005BB129

005BAD43   >  8B45 FC          MOV EAX,DWORD PTR SS:\[EBP-4\]      ;  Case C of switch 005BAD22

005BAD46   .  05 0C040000      ADD EAX,40C                          ;  如果长度位 12，即当时填入 32 位假码，就跳到这里
```
