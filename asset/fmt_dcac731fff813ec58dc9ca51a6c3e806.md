
```asm
001B:004AF6DB   8365FC00                AND       DWORD PTR [EBP-04],00
001B:004AF6DF   897DEC                  MOV       [EBP-14],EDI
001B:004AF6E2   8B8FB8000000            MOV       ECX,[EDI+000000B8]
001B:004AF6E8   8D87B8000000            LEA       EAX,[EDI+000000B8]
001B:004AF6EE   894DE8                  MOV       [EBP-18],ECX
001B:004AF6F1   8B4E1C                  MOV       ECX,[ESI+1C]
001B:004AF6F4   8908                    MOV       [EAX],ECX
001B:004AF6F6   8B06                    MOV       EAX,[ESI]
001B:004AF6F8   8D4DD4                  LEA       ECX,[EBP-2C]
001B:004AF6FB   51                      PUSH      ECX
001B:004AF6FC   8BCE                    MOV       ECX,ESI
001B:004AF6FE   FF9084000000            CALL      [EAX+00000084]  //取 key、name 字符串
001B:004AF704   C7450801000000          MOV       DWORD PTR [EBP+08],00000001//置标准位
001B:004AF70B   EB27                    JMP       004AF734
001B:004AF70D   B831F74A00              MOV       EAX,004AF731
001B:004AF712   C3                      RET
001B:004AF713   8B75E4                  MOV       ESI,[EBP-1C]
001B:004AF716   6808F10000              PUSH      0000F108
001B:004AF71B   6A30                    PUSH      30
001B:004AF71D   8BCE                    MOV       ECX,ESI
001B:004AF71F   8B06                    MOV       EAX,[ESI]
001B:004AF721   FF5010                  CALL      [EAX+10]
001B:004AF724   8BCE                    MOV       ECX,ESI
001B:004AF726   E8CF3C0000              CALL      004B33FA
001B:004AF72B   B831F74A00              MOV       EAX,004AF731
001B:004AF730   C3                      RET
001B:004AF731   8B7DEC                  MOV       EDI,[EBP-14]
001B:004AF734   8B45E8                  MOV       EAX,[EBP-18]
001B:004AF737   8B4DF4                  MOV       ECX,[EBP-0C]
001B:004AF73A   8987B8000000            MOV       [EDI+000000B8],EAX
001B:004AF740   8B4508                  MOV       EAX,[EBP+08]
001B:004AF743   5F                      POP       EDI
001B:004AF744   5E                      POP       ESI
001B:004AF745   64890D00000000          MOV       FS:[00000000],ECX
001B:004AF74C   5B                      POP       EBX
001B:004AF74D   C9                      LEAVE
001B:004AF74E   C20400                  RET       0004  ret to 00427DCA
--------------------------------------------------
001B:00427DAB   90                      NOP
001B:00427DAC   397D42                  CMP       [EBP+42],EDI
001B:00427DAF   004F7D                  ADD       [EDI+7D],CL
001B:00427DB2   42                      INC       EDX
001B:00427DB3   00447D42                ADD       [EDI*2+EBP+42],AL
001B:00427DB7   004F7D                  ADD       [EDI+7D],CL
001B:00427DBA   42                      INC       EDX
001B:00427DBB   009090909056            ADD       [EAX+56909090],DL
001B:00427DC1   8BF1                    MOV       ESI,ECX
001B:00427DC3   6A01                    PUSH      01
001B:00427DC5   E8E5780800              CALL      004AF6AF  //见上面，取用户注册信息
001B:00427DCA   8BCE                    MOV       ECX,ESI
001B:00427DCC   E81FFEFFFF              CALL      00427BF0  //登录、注册、验证、显示结果，网上验证的过程可能启用了多线程！见下面；
001B:00427DD1   85C0                    TEST      EAX,EAX
001B:00427DD3   7429                    JZ        00427DFE  //跳则不发布，不跳则发布信息
001B:00427DD5   8D8688000000            LEA       EAX,[ESI+00000088]
001B:00427DDB   B910A35100              MOV       ECX,0051A310
001B:00427DE0   50                      PUSH      EAX
001B:00427DE1   E8EC990800              CALL      004B17D2
001B:00427DE6   8D8E8C000000            LEA       ECX,[ESI+0000008C]
001B:00427DEC   51                      PUSH      ECX
001B:00427DED   B90CA35100              MOV       ECX,0051A30C
001B:00427DF2   E8DB990800              CALL      004B17D2
001B:00427DF7   8BCE                    MOV       ECX,ESI
001B:00427DF9   E80E950800              CALL      004B130C
001B:00427DFE   5E                      POP       ESI
001B:00427DFF   C3                      RET
===================================================

如此跟踪，很快找到核心位置 `00427DCC` `CALL 00427BF0`

`g 00427BF0`

----------------

```asm
001B:00427BF0   6AFF                    PUSH      FF
001B:00427BF2   6886694D00              PUSH      004D6986
001B:00427BF7   64A100000000            MOV       EAX,FS:[00000000]
001B:00427BFD   50                      PUSH      EAX
001B:00427BFE   64892500000000          MOV       FS:[00000000],ESP
001B:00427C05   81ECE4000000            SUB       ESP,000000E4
001B:00427C0B   53                      PUSH      EBX
001B:00427C0C   55                      PUSH      EBP
001B:00427C0D   56                      PUSH      ESI
001B:00427C0E   8BF1                    MOV       ESI,ECX
001B:00427C10   C78694000000FFFFFFFF    MOV       DWORD PTR [ESI+00000094],FFFFFFFF  //标志位初始化
001B:00427C1A   C7869800000001000000    MOV       DWORD PTR [ESI+00000098],00000001
001B:00427C24   A1A4105100              MOV       EAX,[005110A4]
001B:00427C29   8944240C                MOV       [ESP+0C],EAX
001B:00427C2D   6804FA0000              PUSH      0000FA04
001B:00427C32   8D4C2410                LEA       ECX,[ESP+10]
001B:00427C36   C78424FC000000000000    MOV       DWORD PTR [ESP+000000FC],00000000
001B:00427C41   E816A00800              CALL      004B1C5C
001B:00427C46   6A00                    PUSH      00
001B:00427C48   6A00                    PUSH      00
001B:00427C4A   6A00                    PUSH      00
001B:00427C4C   6A00                    PUSH      00
001B:00427C4E   56                      PUSH      ESI
001B:00427C4F   68E06F4200              PUSH      00426FE0
001B:00427C54   E826A30800              CALL      004B1F7F
001B:00427C59   6A00                    PUSH      00
001B:00427C5B   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C5F   E82CB4FFFF              CALL      00423090
001B:00427C64   6807FA0000              PUSH      0000FA07
001B:00427C69   8D8C248C000000          LEA       ECX,[ESP+0000008C]
001B:00427C70   C68424FC00000001        MOV       BYTE PTR [ESP+000000FC],01
001B:00427C78   E8DF9F0800              CALL      004B1C5C
001B:00427C7D   56                      PUSH      ESI
001B:00427C7E   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C82   E8A9B5FFFF              CALL      00423230
001B:00427C87   8B4C240C                MOV       ECX,[ESP+0C]
001B:00427C8B   51                      PUSH      ECX
001B:00427C8C   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C90   E8E7850800              CALL      004B027C
001B:00427C95   8B2D3C084E00            MOV       EBP,[004E083C]
001B:00427C9B   FFD5                    CALL      EBP
001B:00427C9D   8BD8                    MOV       EBX,EAX
001B:00427C9F   8B8698000000            MOV       EAX,[ESI+00000098]
001B:00427CA5   85C0                    TEST      EAX,EAX
001B:00427CA7   747D                    JZ        00427D26
001B:00427CA9   57                      PUSH      EDI
001B:00427CAA   8B3D58074E00            MOV       EDI,[USER32!PeekMessageA]
001B:00427CB0   6A01                    PUSH      01
001B:00427CB2   6A00                    PUSH      00
001B:00427CB4   6A00                    PUSH      00
001B:00427CB6   8D542420                LEA       EDX,[ESP+20]
001B:00427CBA   6A00                    PUSH      00
001B:00427CBC   52                      PUSH      EDX
001B:00427CBD   FFD7                    CALL      EDI  //[USER32!PeekMessageA]
001B:00427CBF   85C0                    TEST      EAX,EAX
001B:00427CC1   7429                    JZ        00427CEC
001B:00427CC3   8D442414                LEA       EAX,[ESP+14]\
001B:00427CC7   50                      PUSH      EAX
001B:00427CC8   FF1550074E00            CALL      [USER32!TranslateMessage]
001B:00427CCE   8D4C2414                LEA       ECX,[ESP+14]
001B:00427CD2   51                      PUSH      ECX
001B:00427CD3   FF1554074E00            CALL      [USER32!DispatchMessageA]
001B:00427CD9   6A01                    PUSH      01
001B:00427CDB   6A00                    PUSH      00
001B:00427CDD   6A00                    PUSH      00
001B:00427CDF   8D542420                LEA       EDX,[ESP+20]
```
