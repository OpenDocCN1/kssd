
```text
  004074F6   >  8A4F 02                MOV     CL, BYTE PTR [EDI+2]
  004074F9   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  004074FC   .  884C24 14              MOV     BYTE PTR [ESP+14], CL            ; k[3] -> [esp+14]
  00407500   .  837F F4 03             CMP     DWORD PTR [EDI-C], 3
  00407504   .  7D 0A                  JGE     SHORT 00407510
  00407506   .  68 57000780            PUSH    80070057
  0040750B   .  E8 009BFFFF            CALL    00401010
  00407510   >  8A4F 03                MOV     CL, BYTE PTR [EDI+3]
  00407513   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  00407516   .  884C24 15              MOV     BYTE PTR [ESP+15], CL            ; k[4] -> [esp+15]
  0040751A   .  837F F4 04             CMP     DWORD PTR [EDI-C], 4
  0040751E   .  7D 0A                  JGE     SHORT 0040752A
  00407520   .  68 57000780            PUSH    80070057
  00407525   .  E8 E69AFFFF            CALL    00401010
  0040752A   >  8A4F 04                MOV     CL, BYTE PTR [EDI+4]             ; k[5] -> cl
  0040752D   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  00407530   .  837F F4 05             CMP     DWORD PTR [EDI-C], 5
  00407534   .  7D 0A                  JGE     SHORT 00407540
  00407536   .  68 57000780            PUSH    80070057
  0040753B   .  E8 D09AFFFF            CALL    00401010
  00407540   >  8A5F 05                MOV     BL, BYTE PTR [EDI+5]
  00407543   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  00407546   .  885C24 1A              MOV     BYTE PTR [ESP+1A], BL            ; k[6] -> [esp+1A]
  0040754A   .  837F F4 06             CMP     DWORD PTR [EDI-C], 6
  0040754E   .  7D 0A                  JGE     SHORT 0040755A
  00407550   .  68 57000780            PUSH    80070057
  00407555   .  E8 B69AFFFF            CALL    00401010
  0040755A   >  8A5F 06                MOV     BL, BYTE PTR [EDI+6]
  0040755D   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  00407560   .  885C24 1B              MOV     BYTE PTR [ESP+1B], BL            ; k[7] -> [esp+1B]
  00407564   .  837F F4 07             CMP     DWORD PTR [EDI-C], 7
  00407568   .  7D 0A                  JGE     SHORT 00407574
  0040756A   >  68 57000780            PUSH    80070057
  0040756F   .  E8 9C9AFFFF            CALL    00401010
  00407574   >  8A5F 07                MOV     BL, BYTE PTR [EDI+7]             ; k[8] -> bl
```

到这里用户名和注册码的预处理基本完毕，可以在内存中看一下，熟悉一下各自的位置，下面就要逐一计算比较了。

```text
  00407577   .  0FB67C24 16            MOVZX   EDI, BYTE PTR [ESP+16]           ; r1
  0040757C   .  0FB6D2                 MOVZX   EDX, DL                          ; k[1]
  0040757F   .  83EA 30                SUB     EDX, 30
  00407582   .  3BFA                   CMP     EDI, EDX                         ; 比较1：k[1]-0x30 == r1
  00407584   .  75 48                  JNZ     SHORT 004075CE                   ; 如果不相等，似乎也没有一下跳到出错处哦
  00407586   .  0FB65424 13            MOVZX   EDX, BYTE PTR [ESP+13]           ; k[2]
  0040758B   .  0FB67C24 12            MOVZX   EDI, BYTE PTR [ESP+12]           ; r2
  00407590   .  83EA 30                SUB     EDX, 30
  00407593   .  3BFA                   CMP     EDI, EDX                         ; 比较2：k[2]-0x30 == r2
  00407595   .  75 37                  JNZ     SHORT 004075CE
  00407597   .  0FB65424 14            MOVZX   EDX, BYTE PTR [ESP+14]           ; k[3]
  0040759C   .  0FB67C24 17            MOVZX   EDI, BYTE PTR [ESP+17]           ; r3
  004075A1   .  83EA 30                SUB     EDX, 30
  004075A4   .  3BFA                   CMP     EDI, EDX                         ; 比较3：k[3]-0x30 == r3
  004075A6   .  75 26                  JNZ     SHORT 004075CE
  004075A8   .  0FB65424 15            MOVZX   EDX, BYTE PTR [ESP+15]           ; k[4]
  004075AD   .  0FB67C24 18            MOVZX   EDI, BYTE PTR [ESP+18]           ; r4
  004075B2   .  83EA 30                SUB     EDX, 30
  004075B5   .  3BFA                   CMP     EDI, EDX                         ; 比较4：k[4]-0x30 == r4
  004075B7   .  75 15                  JNZ     SHORT 004075CE
  004075B9   .  99                     CDQ
  004075BA   .  BF 0A000000            MOV     EDI, 0A
  004075BF   .  F7FF                   IDIV    EDI                              ; EAX % 0x0A = 4，即上面求的和
  004075C1   .  0FB6C2                 MOVZX   EAX, DL                          ; 上面的余数
  004075C4   .  0FB6D1                 MOVZX   EDX, CL                          ; 比较5：k[5]-0x30 = 4
  004075C7   .  83EA 30                SUB     EDX, 30
  004075CA   .  3BC2                   CMP     EAX, EDX
  004075CC   .  74 3A                  JE      SHORT 00407608                   ; 完成最后一个比较，就成功了！
```

如果上面的条件没有满足，会跳到这里再经过一次判断：

```text
  004075CE   >  807C24 19 38           CMP     BYTE PTR [ESP+19], 38            ; k[1] == 0x38
  004075D3   .  0F85 85000000          JNZ     0040765E                         ; 这次可是真的跳到失败处了
  004075D9   .  807C24 13 33           CMP     BYTE PTR [ESP+13], 33            ; k[2] == 0x33
  004075DE   .  75 7E                  JNZ     SHORT 0040765E
  004075E0   .  807C24 14 39           CMP     BYTE PTR [ESP+14], 39            ; k[3] == 0x39
  004075E5   .  75 77                  JNZ     SHORT 0040765E
  004075E7   .  8A5424 15              MOV     DL, BYTE PTR [ESP+15]            ; k[4] == 0x31
  004075EB   .  B0 31                  MOV     AL, 31
  004075ED   .  3AD0                   CMP     DL, AL
  004075EF   .  75 6D                  JNZ     SHORT 0040765E
  004075F1   .  80F9 33                CMP     CL, 33                           ; k[5] == 0x33
  004075F4   .  75 68                  JNZ     SHORT 0040765E
  004075F6   .  384424 1A              CMP     BYTE PTR [ESP+1A], AL            ; k[6] == 0x31
  004075FA   .  75 62                  JNZ     SHORT 0040765E
  004075FC   .  807C24 1B 34           CMP     BYTE PTR [ESP+1B], 34            ; k[7] == 0x34
  00407601   .  75 5B                  JNZ     SHORT 0040765E
  00407603   .  80FB 36                CMP     BL, 36                           ; k[8] == 0x36
  00407606   .  75 56                  JNZ     SHORT 0040765E                   ; 天啊，居然有万能注册码'83913146'
  00407608   >  6A 00                  PUSH    0                                ; 下面不用说了，提示注册成功并且保存注册信息
  0040760A   .  6A 00                  PUSH    0
  0040760C   .  68 E8FE4200            PUSH    0042FEE8                         ;  ASCII "Registration has succeeded!"
  00407611   .  E8 CD100200            CALL    004286E3
  00407616   .  8B7E 70                MOV     EDI, DWORD PTR [ESI+70]
  00407619   .  E8 8A1E0200            CALL    004294A8
  0040761E   .  8B40 04                MOV     EAX, DWORD PTR [EAX+4]
  00407621   .  57                     PUSH    EDI                              ; /Arg3
  00407622   .  68 F4FD4200            PUSH    0042FDF4                         ; |Arg2 = 0042FDF4 ASCII "username"
  00407627   .  68 ECFD4200            PUSH    0042FDEC                         ; |Arg1 = 0042FDEC ASCII "Option"
  0040762C   .  8BC8                   MOV     ECX, EAX                         ; |
  0040762E   .  E8 100E0200            CALL    00428443                         ; \FileTree.00428443
  00407633   .  8B7E 74                MOV     EDI, DWORD PTR [ESI+74]
  00407636   .  E8 6D1E0200            CALL    004294A8
  0040763B   .  8B40 04                MOV     EAX, DWORD PTR [EAX+4]
  0040763E   .  57                     PUSH    EDI                              ; /Arg3
  0040763F   .  68 D8FD4200            PUSH    0042FDD8                         ; |Arg2 = 0042FDD8 ASCII "registration\_code"
  00407644   .  68 ECFD4200            PUSH    0042FDEC                         ; |Arg1 = 0042FDEC ASCII "Option"
  00407649   .  8BC8                   MOV     ECX, EAX                         ; |
  0040764B   .  E8 F30D0200            CALL    00428443                         ; \FileTree.00428443
```
