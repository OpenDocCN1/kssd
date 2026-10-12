
## 启动验证

右键超级字串参考查找 ASCII.查找`\\SOFTWARE\\WinASO\\Disk Cleaner`，共有 3 处：

```
004B9264  |.  BA C0924B00   MOV EDX,DiskClea.004B92C0          ;  \software\winaso\disk cleaner
004BD8F8  |.  BA 74DA4B00   MOV EDX,DiskClea.004BDA74          ;  \software\winaso\disk cleaner
004BDE2E  |.  BA D0DE4B00   MOV EDX,DiskClea.004BDED0          ;  \software\winaso\disk cleaner
```

### 代码段 1

```
004BD538  /.  55           PUSH EBP
004BD539  |.  8BEC        MOV EBP,ESP
004BD53B  |.  B9 07000000   MOV ECX,7
004BD540  |>  6A 00        /PUSH 0
004BD542  |.  6A 00       |PUSH 0
004BD544  |.  49           |DEC ECX
004BD545  |.^ 75 F9        \\JNZ SHORT DiskClea.004BD540
004BD547  |.  51           PUSH ECX
004BD548  |.  53           PUSH EBX
004BD549  |.  56           PUSH ESI
004BD54A  |.  57           PUSH EDI
004BD54B  |.  8BF0         MOV ESI,EAX
004BD54D  |.  33C0         XOR EAX,EAX
004BD54F  |.  55           PUSH EBP
004BD550  |.  68 5AD94B00  PUSH DiskClea.004BD95A
004BD555  |.  64:FF30      PUSH DWORD PTR FS:\[EAX\]
004BD558  |.  64:8920      MOV DWORD PTR FS:\[EAX\],ESP
004BD55B  |.  8D55 FC      LEA EDX,DWORD PTR SS:\[EBP-4\]
004BD55E  |.  8B86 8C030000 MOV EAX,DWORD PTR DS:\[ESI+38C\]
004BD564  |.  E8 7B04F9FF  CALL DiskClea.0044D9E4               ;  //将注册码长度送入 EAX
004BD569  |.  8D45 F4      LEA EAX,DWORD PTR SS:\[EBP-C\]
004BD56C  |.  50           PUSH EAX                               ; /Arg1
004BD56D  |.  33C9         XOR ECX,ECX                            ; |
004BD56F  |.  BA 70D94B00  MOV EDX,DiskClea.004BD970             ; |
004BD574  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ; |//将注册码送入 EAX
004BD577  |.  E8 F021F8FF  CALL DiskClea.0043F76C                ; \\DiskClea.0043F76C
004BD57C  |.  8B55 F4      MOV EDX,DWORD PTR SS:\[EBP-C\]         ;  //将注册码送入 EDX
004BD57F  |.  8D45 FC      LEA EAX,DWORD PTR SS:\[EBP-4\]
004BD582  |.  E8 8974F4FF  CALL DiskClea.00404A10
004BD587  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD58A  |.  85C0         TEST EAX,EAX                           ;  //比较 EAX
004BD58C  |.  74 05        JE SHORT DiskClea.004BD593             ;  //若相等则跳
004BD58E  |.  83E8 04      SUB EAX,4
004BD591  |.  8B00         MOV EAX,DWORD PTR DS:\[EAX\]           ;  //将注册码长度送入 EAX
004BD593  |>  83F8 10      CMP EAX,10                             ;  //将注册码长度与 10h 比较
004BD596  |.  74 1E        JE SHORT DiskClea.004BD5B6             ;  //若相等则跳
004BD598  |.  6A 40        PUSH 40
004BD59A  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD59F  |.  68 88D94B00  PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD5A4  |.  8BC6         MOV EAX,ESI
004BD5A6  |.  E8 397FF9FF  CALL DiskClea.004554E4
004BD5AB  |.  50           PUSH EAX                               ; |hOwner
004BD5AC  |.  E8 AFA1F4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD5B1  |.  E9 89030000  JMP DiskClea.004BD93F
004BD5B6  |>  BB 01000000  MOV EBX,1                              ;  //将 1 送入 EBX
004BD5BB  |>  8D45 F8      /LEA EAX,DWORD PTR SS:\[EBP-8\]
004BD5BE  |.  50           |PUSH EAX                               ; /Arg1
004BD5BF  |.  B9 01000000  |MOV ECX,1                              ; |//将 1 送入 ECX
004BD5C4  |.  8BD3         |MOV EDX,EBX                              ; |//将 EBX 送入 EDX
004BD5C6  |.  8B45 FC      |MOV EAX,DWORD PTR SS:\[EBP-4\]           ; |//将注册码送入 EAX
004BD5C9  |.  E8 7A23F8FF  |CALL DiskClea.0043F948               ; \\//将注册码 ASC 码 16 进制送入 ECX
004BD5CE  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD5D1  |.  BA FCD94B00  |MOV EDX,DiskClea.004BD9FC
004BD5D6  |.  E8 AD77F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'0'比较
004BD5DB  |.  0F84 AD000000 |JE DiskClea.004BD68E                 ;  //若相等则跳
004BD5E1  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD5E4  |.  BA 08DA4B00  |MOV EDX,DiskClea.004BDA08
004BD5E9  |.  E8 9A77F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'1'比较
004BD5EE  |.  0F84 9A000000 |JE DiskClea.004BD68E                 ;  //若相等则跳
004BD5F4  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD5F7  |.  BA 14DA4B00  |MOV EDX,DiskClea.004BDA14
004BD5FC  |.  E8 8777F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'2'比较
004BD601  |.  0F84 87000000 |JE DiskClea.004BD68E                 ;  //若相等则跳
004BD607  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD60A  |.  BA 20DA4B00  |MOV EDX,DiskClea.004BDA20
004BD60F  |.  E8 7477F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'3'比较
004BD614  |.  74 78        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD616  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD619  |.  BA 2CDA4B00  |MOV EDX,DiskClea.004BDA2C
004BD61E  |.  E8 6577F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'4'比较
004BD623  |.  74 69        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD625  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD628  |.  BA 38DA4B00  |MOV EDX,DiskClea.004BDA38
004BD62D  |.  E8 5677F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'5'比较
004BD632  |.  74 5A        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD634  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD637  |.  BA 44DA4B00  |MOV EDX,DiskClea.004BDA44
004BD63C  |.  E8 4777F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'6'比较
004BD641  |.  74 4B        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD643  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD646  |.  BA 50DA4B00  |MOV EDX,DiskClea.004BDA50
004BD64B  |.  E8 3877F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'7'比较
004BD650  |.  74 3C        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD652  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD655  |.  BA 5CDA4B00  |MOV EDX,DiskClea.004BDA5C
004BD65A  |.  E8 2977F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'8'比较
004BD65F  |.  74 2D        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD661  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]
004BD664  |.  BA 68DA4B00  |MOV EDX,DiskClea.004BDA68
004BD669  |.  E8 1A77F4FF  |CALL DiskClea.00404D88                ;  //将注册码与'9'比较
004BD66E  |.  74 1E        |JE SHORT DiskClea.004BD68E            ;  //若相等则跳
004BD670  |.  6A 40        |PUSH 40
004BD672  |.  68 74D94B00  |PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD677  |.  68 88D94B00  |PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD67C  |.  8BC6         |MOV EAX,ESI
004BD67E  |.  E8 617EF9FF  |CALL DiskClea.004554E4
004BD683  |.  50           |PUSH EAX                               ; |hOwner
004BD684  |.  E8 D7A0F4FF  |CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD689  |.  E9 B1020000  |JMP DiskClea.004BD93F
004BD68E  |>  43           |INC EBX                                 ;  //EBX=EBX+1
004BD68F  |.  83FB 11      |CMP EBX,11                              ;  //将 EBX 与 11h 比较
004BD692  |.^ 0F85 23FFFFFF \\JNZ DiskClea.004BD5BB              ;  //若不相等则跳,以上为验证注册码是否存在非数字
004BD698  |.  33FF         XOR EDI,EDI                              ;  //将 EDI 清零
004BD69A  |.  BB 01000000  MOV EBX,1                                 ;  //将 1 送入 EBX
004BD69F  |>  8D45 F8      /LEA EAX,DWORD PTR SS:\[EBP-8\]
004BD6A2  |.  50           |PUSH EAX                                 ; /Arg1
004BD6A3  |.  8BD3         |MOV EDX,EBX                                ; |//将 EBX 送入 EDX
004BD6A5  |.  03D2         |ADD EDX,EDX                                ; |//EDX=EDX+EDX
004BD6A7  |.  03D2         |ADD EDX,EDX                                ; |//EDX=EDX+EDX
004BD6A9  |.  42           |INC EDX                                    ; |//EDX=EDX+1
004BD6AA  |.  B9 04000000  |MOV ECX,4                                 ; |//将 4 送入 ECX
004BD6AF  |.  8B45 FC      |MOV EAX,DWORD PTR SS:\[EBP-4\]             ; |//将注册码送入 EAX
004BD6B2  |.  E8 9122F8FF  |CALL DiskClea.0043F948               ; \\//从注册码第 5 位开始,每 4 位算 1 组
004BD6B7  |.  8B45 F8      |MOV EAX,DWORD PTR SS:\[EBP-8\]             ;  //1 组 4 位注册码
004BD6BA  |.  E8 EDBBF4FF  |CALL DiskClea.004092AC               ;  //4 位注册码转为 16 进制送入 EAX
004BD6BF  |.  03F8         |ADD EDI,EAX                               ;  //EDI=EDI+EAX
004BD6C1  |.  43           |INC EBX                                     ;  //EBX=EBX+1
004BD6C2  |.  83FB 04      |CMP EBX,4                                   ;  //将 EBX 与 4 比较
004BD6C5  |.^ 75 D8        \\JNZ SHORT DiskClea.004BD69F          ;  //若不相等则跳
004BD6C7  |.  83FF 32      CMP EDI,32                                   ;  //将 EDI 即后 3 组的和与 32h 比较
004BD6CA  |.  7D 1E        JGE SHORT DiskClea.004BD6EA              ;  //若大于等于则跳
004BD6CC  |.  6A 40        PUSH 40
004BD6CE  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD6D3  |.  68 88D94B00  PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD6D8  |.  8BC6         MOV EAX,ESI
004BD6DA  |.  E8 057EF9FF  CALL DiskClea.004554E4
004BD6DF  |.  50           PUSH EAX                                 ; |hOwner
004BD6E0  |.  E8 7BA0F4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD6E5  |.  E9 55020000  JMP DiskClea.004BD93F
004BD6EA  |>  8D45 F0      LEA EAX,DWORD PTR SS:\[EBP-10\]
004BD6ED  |.  50           PUSH EAX
004BD6EE  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD6F3  |.  BA 06000000  MOV EDX,6                               ;  //将 6 送入 EDX
004BD6F8  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD6FB  |.  E8 B421F8FF  CALL DiskClea.0043F8B4
004BD700  |.  FF75 F0      PUSH DWORD PTR SS:\[EBP-10\]
004BD703  |.  8D45 EC      LEA EAX,DWORD PTR SS:\[EBP-14\]
004BD706  |.  50           PUSH EAX
004BD707  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD70C  |.  BA 07000000  MOV EDX,7                               ;  //将 7 送入 EDX
004BD711  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD714  |.  E8 9B21F8FF  CALL DiskClea.0043F8B4
004BD719  |.  FF75 EC      PUSH DWORD PTR SS:\[EBP-14\]
004BD71C  |.  8D45 E8      LEA EAX,DWORD PTR SS:\[EBP-18\]
004BD71F  |.  50           PUSH EAX
004BD720  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD725  |.  BA 05000000  MOV EDX,5                               ;  //将 5 送入 EDX
004BD72A  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD72D  |.  E8 8221F8FF  CALL DiskClea.0043F8B4
004BD732  |.  FF75 E8      PUSH DWORD PTR SS:\[EBP-18\]
004BD735  |.  8D45 E4      LEA EAX,DWORD PTR SS:\[EBP-1C\]
004BD738  |.  50           PUSH EAX
004BD739  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD73E  |.  BA 08000000  MOV EDX,8                               ;  //将 8 送入 EDX
004BD743  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD746  |.  E8 6921F8FF  CALL DiskClea.0043F8B4
004BD74B  |.  FF75 E4      PUSH DWORD PTR SS:\[EBP-1C\]
004BD74E  |.  8D45 F8      LEA EAX,DWORD PTR SS:\[EBP-8\]
004BD751  |.  BA 04000000  MOV EDX,4                               ;  //将 4 送入 EDX
004BD756  |.  E8 A175F4FF  CALL DiskClea.00404CFC                ;  //将注册码第 5-8 位重新排序,为第 6 位第 7 位第 5 位第 8 位
004BD75B  |.  8B45 F8      MOV EAX,DWORD PTR SS:\[EBP-8\]         ;  //将重新排序的第 5-8 位送入 EAX
004BD75E  |.  E8 49BBF4FF  CALL DiskClea.004092AC                ;  //将重新排序的第 5-8 位转为 16 进制送入 EAX
004BD763  |.  8BD8         MOV EBX,EAX                             ;  //将 EAX 送入 EBX
004BD765  |.  8BC3         MOV EAX,EBX                             ;  //将 EBX 送入 EAX
004BD767  |.  B9 17000000  MOV ECX,17                              ;  //将 17h 送入 ECX
004BD76C  |.  99           CDQ                                     ;  //将 EDX 清零
004BD76D  |.  F7F9         IDIV ECX                                 ;  //EDX/ECX,商送 EAX,余送 EDX
004BD76F  |.  85D2         TEST EDX,EDX                              ;  //比较 EDX
004BD771  |.  74 1E        JE SHORT DiskClea.004BD791             ;  //若相等则跳,即余数为 0
004BD773  |.  6A 40        PUSH 40
004BD775  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD77A  |.  68 88D94B00  PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD77F  |.  8BC6         MOV EAX,ESI
004BD781  |.  E8 5E7DF9FF  CALL DiskClea.004554E4
004BD786  |.  50           PUSH EAX                                 ; |hOwner
004BD787  |.  E8 D49FF4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD78C  |.  E9 AE010000  JMP DiskClea.004BD93F
004BD791  |>  8D45 E0      LEA EAX,DWORD PTR SS:\[EBP-20\]
004BD794  |.  50           PUSH EAX
004BD795  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD79A  |.  BA 0B000000  MOV EDX,0B                               ;  //将 0B 送入 EDX
004BD79F  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD7A2  |.  E8 0D21F8FF  CALL DiskClea.0043F8B4
004BD7A7  |.  FF75 E0      PUSH DWORD PTR SS:\[EBP-20\]
004BD7AA  |.  8D45 DC      LEA EAX,DWORD PTR SS:\[EBP-24\]
004BD7AD  |.  50           PUSH EAX
004BD7AE  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD7B3  |.  BA 0C000000  MOV EDX,0C
004BD7B8  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]
004BD7BB  |.  E8 F420F8FF  CALL DiskClea.0043F8B4
004BD7C0  |.  FF75 DC      PUSH DWORD PTR SS:\[EBP-24\]
004BD7C3  |.  8D45 D8      LEA EAX,DWORD PTR SS:\[EBP-28\]
004BD7C6  |.  50           PUSH EAX
004BD7C7  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD7CC  |.  BA 09000000  MOV EDX,9                               ;  //将 9 送入 EDX
004BD7D1  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD7D4  |.  E8 DB20F8FF  CALL DiskClea.0043F8B4
004BD7D9  |.  FF75 D8      PUSH DWORD PTR SS:\[EBP-28\]
004BD7DC  |.  8D45 D4      LEA EAX,DWORD PTR SS:\[EBP-2C\]
004BD7DF  |.  50           PUSH EAX
004BD7E0  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD7E5  |.  BA 0A000000  MOV EDX,0A                               ;  //将 0A 送入 EDX
004BD7EA  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD7ED  |.  E8 C220F8FF  CALL DiskClea.0043F8B4
004BD7F2  |.  FF75 D4      PUSH DWORD PTR SS:\[EBP-2C\]
004BD7F5  |.  8D45 F8      LEA EAX,DWORD PTR SS:\[EBP-8\]
004BD7F8  |.  BA 04000000  MOV EDX,4                               ;  //将 4 送入 EDX
004BD7FD  |.  E8 FA74F4FF  CALL DiskClea.00404CFC                ;  //将注册码第 9-12 位重新排序,为第 11 位第 12 位第 9 位第 10 位
004BD802  |.  8B45 F8      MOV EAX,DWORD PTR SS:\[EBP-8\]         ;  //将重新排序的第 9-12 位送入 EAX
004BD805  |.  E8 A2BAF4FF  CALL DiskClea.004092AC                ;  //将重新排序的第 9-12 位转为 16 进制送入 EAX
004BD80A  |.  8BD8         MOV EBX,EAX                             ;  //将 EAX 送入 EBX
004BD80C  |.  8BC3         MOV EAX,EBX                             ;  //将 EBX 送入 EAX
004BD80E  |.  B9 13000000  MOV ECX,13                              ;  //将 13h 送入 ECX
004BD813  |.  99           CDQ                                     ;  //将 EDX 清零
004BD814  |.  F7F9         IDIV ECX                                 ;  //EDX/ECX,商送 EAX,余送 EDX
004BD816  |.  85D2         TEST EDX,EDX                              ;  //比较 EDX
004BD818  |.  74 1E        JE SHORT DiskClea.004BD838             ;  //若相等则跳,即余数为 0
004BD81A  |.  6A 40        PUSH 40
004BD81C  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD821  |.  68 88D94B00  PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD826  |.  8BC6         MOV EAX,ESI
004BD828  |.  E8 B77CF9FF  CALL DiskClea.004554E4
004BD82D  |.  50           PUSH EAX                                 ; |hOwner
004BD82E  |.  E8 2D9FF4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD833  |.  E9 07010000  JMP DiskClea.004BD93F
004BD838  |>  8D45 D0      LEA EAX,DWORD PTR SS:\[EBP-30\]
004BD83B  |.  50           PUSH EAX
004BD83C  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD841  |.  BA 10000000  MOV EDX,10                              ;  //将 10h 送入 EDX
004BD846  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD849  |.  E8 6620F8FF  CALL DiskClea.0043F8B4
004BD84E  |.  FF75 D0      PUSH DWORD PTR SS:\[EBP-30\]
004BD851  |.  8D45 CC      LEA EAX,DWORD PTR SS:\[EBP-34\]
004BD854  |.  50           PUSH EAX
004BD855  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD85A  |.  BA 0F000000  MOV EDX,0F                               ;  //将 0F 送入 EDX
004BD85F  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD862  |.  E8 4D20F8FF  CALL DiskClea.0043F8B4
004BD867  |.  FF75 CC      PUSH DWORD PTR SS:\[EBP-34\]
004BD86A  |.  8D45 C8      LEA EAX,DWORD PTR SS:\[EBP-38\]
004BD86D  |.  50           PUSH EAX
004BD86E  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD873  |.  BA 0E000000  MOV EDX,0E                               ;  //将 0E 送入 EDX
004BD878  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD87B  |.  E8 3420F8FF  CALL DiskClea.0043F8B4
004BD880  |.  FF75 C8      PUSH DWORD PTR SS:\[EBP-38\]
004BD883  |.  8D45 C4      LEA EAX,DWORD PTR SS:\[EBP-3C\]
004BD886  |.  50           PUSH EAX
004BD887  |.  B9 01000000  MOV ECX,1                               ;  //将 1 送入 ECX
004BD88C  |.  BA 0D000000  MOV EDX,0D                               ;  //将 0D 送入 EDX
004BD891  |.  8B45 FC      MOV EAX,DWORD PTR SS:\[EBP-4\]         ;  //将注册码送入 EAX
004BD894  |.  E8 1B20F8FF  CALL DiskClea.0043F8B4
004BD899  |.  FF75 C4      PUSH DWORD PTR SS:\[EBP-3C\]
004BD89C  |.  8D45 F8      LEA EAX,DWORD PTR SS:\[EBP-8\]
004BD89F  |.  BA 04000000  MOV EDX,4                               ;  //将 4 送入 EDX
004BD8A4  |.  E8 5374F4FF  CALL DiskClea.00404CFC                ;  //将注册码第 13-16 位重新排序,为第 16 位第 15 位第 14 位第 13 位
004BD8A9  |.  8B45 F8      MOV EAX,DWORD PTR SS:\[EBP-8\]         ;  //将重新排序的第 13-16 位送入 EAX
004BD8AC  |.  E8 FBB9F4FF  CALL DiskClea.004092AC                ;  //将重新排序的第 13-16 位转为 16 进制送入 EAX
004BD8B1  |.  8BD8         MOV EBX,EAX                             ;  //将 EAX 送入 EBX
004BD8B3  |.  8BC3         MOV EAX,EBX                             ;  //将 EBX 送入 EAX
004BD8B5  |.  B9 1F000000  MOV ECX,1F                              ;  //将 1Fh 送入 ECX
004BD8BA  |.  99           CDQ                                     ;  //将 EDX 清零
004BD8BB  |.  F7F9         IDIV ECX                                 ;  //EDX/ECX,商送 EAX,余送 EDX
004BD8BD  |.  85D2         TEST EDX,EDX                              ;  //比较 EDX
004BD8BF  |.  74 1B        JE SHORT DiskClea.004BD8DC             ;  //若相等则跳,即余数为 0
004BD8C1  |.  6A 40        PUSH 40
004BD8C3  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD8C8  |.  68 88D94B00  PUSH DiskClea.004BD988                 ;  ASCII "Sorry,that is an invalid license key.Please ensure you have entered the license key exactly as provided."
004BD8CD  |.  8BC6         MOV EAX,ESI
004BD8CF  |.  E8 107CF9FF  CALL DiskClea.004554E4
004BD8D4  |.  50           PUSH EAX                                 ; |hOwner
004BD8D5  |.  E8 869EF4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD8DA  |.  EB 63        JMP SHORT DiskClea.004BD93F
004BD8DC  |>  B2 01        MOV DL,1
004BD8DE  |.  A1 789F4200  MOV EAX,DWORD PTR DS:\[429F78\]
004BD8E3  |.  E8 90C7F6FF  CALL DiskClea.0042A078
004BD8E8  |.  8BD8         MOV EBX,EAX
004BD8EA  |.  BA 02000080  MOV EDX,80000002
004BD8EF  |.  8BC3         MOV EAX,EBX
004BD8F1  |.  E8 22C8F6FF  CALL DiskClea.0042A118
004BD8F6  |.  B1 01        MOV CL,1
004BD8F8  |.  BA 74DA4B00  MOV EDX,DiskClea.004BDA74              ;  ASCII "\\\SOFTWARE\\WinASO\\Disk Cleaner"
004BD8FD  |.  8BC3         MOV EAX,EBX                              ;  //注册信息保存在"\\SOFTWARE\\WinASO\\Disk Cleaner"
004BD8FF  |.  E8 78C8F6FF  CALL DiskClea.0042A17C
004BD904  |.  84C0         TEST AL,AL
004BD906  |.  74 0F        JE SHORT DiskClea.004BD917
004BD908  |.  8B4D FC      MOV ECX,DWORD PTR SS:\[EBP-4\]
004BD90B  |.  BA 9CDA4B00  MOV EDX,DiskClea.004BDA9C              ;  ASCII "DiskCln20"
004BD910  |.  8BC3         MOV EAX,EBX
004BD912  |.  E8 B9CBF6FF  CALL DiskClea.0042A4D0
004BD917  |>  A1 54E34C00  MOV EAX,DWORD PTR DS:\[4CE354\]
004BD91C  |.  C600 01      MOV BYTE PTR DS:\[EAX\],1
004BD91F  |.  6A 40        PUSH 40
004BD921  |.  68 74D94B00  PUSH DiskClea.004BD974                 ;  ASCII "WinASO Disk Cleaner"
004BD926  |.  68 A8DA4B00  PUSH DiskClea.004BDA A8                ;  ASCII "WinASO Disk Cleaner is activated now. Thank you!. If you experience any problems, please contact us at support@winaso.com. Enjoy your product!"
004BD92B  |.  8BC6         MOV EAX,ESI
004BD92D  |.  E8 B27BF9FF  CALL DiskClea.004554E4
004BD932  |.  50           PUSH EAX                                 ; |hOwner
004BD933  |.  E8 289EF4FF  CALL <JMP.&user32.MessageBoxA>         ; \\MessageBoxA
004BD938  |.  8BC6         MOV EAX,ESI
004BD93A  |.  E8 1593FAFF  CALL DiskClea.00466C54
004BD93F  |>  33C0         XOR EAX,EAX
004BD941  |.  5A           POP EDX
004BD942  |.  59           POP ECX
004BD943  |.  59           POP ECX
004BD944  |.  64:8910      MOV DWORD PTR FS:\[EAX\],EDX
004BD947  |.  68 61D94B00  PUSH DiskClea.004BD961
004BD94C  |>  8D45 C4      LEA EAX,DWORD PTR SS:\[EBP-3C\]
004BD94F  |.  BA 0F000000  MOV EDX,0F
004BD954  |.  E8 4370F4FF  CALL DiskClea.0040499C
004BD959  \\.  C3          RETN
004BD95A   .^  E9 6169F4FF JMP DiskClea.004042C0
004BD95F   .^  EB EB      JMP SHORT DiskClea.004BD94C
004BD961   .  5F          POP EDI
004BD962   .  5E          POP ESI
004BD963   .  5B          POP EBX
004BD964   .  8BE5        MOV ESP,EBP
004BD966   .  5D          POP EBP
004BD967   .  C3          RETN
```

### 代码段 2

```
004B9234  /$  55           PUSH EBP
004B9235  |.  8BEC        MOV EBP,ESP
004B9237  |.  6A 00       PUSH 0
004B9239  |.  53          PUSH EBX
004B923A  |.  33C0        XOR EAX,EAX
004B923C  |.  55          PUSH EBP
004B923D  |.  68 AB924B00  PUSH DiskClea.004B92AB
004B9242  |.  64:FF30     PUSH DWORD PTR FS:\[EAX\]
004B9245  |.  64:8920     MOV DWORD PTR FS:\[EAX\],ESP
004B9248  |.  B2 01       MOV DL,1
004B924A  |.  A1 789F4200  MOV EAX,DWORD PTR DS:\[429F78\]
004B924F  |.  E8 240EF7FF  CALL DiskClea.0042A078
004B9254  |.  8BD8        MOV EBX,EAX
004B9256  |.  BA 02000080 MOV EDX,80000002
004B925B  |.  8BC3        MOV EAX,EBX
004B925D  |.  E8 B60EF7FF  CALL DiskClea.0042A118
004B9262  |.  B1 01       MOV CL,1
004B9264  |.  BA C0924B00 MOV EDX,DiskClea.004B92C0          ;  \software\winaso\disk cleaner
004B9269  |.  8BC3        MOV EAX,EBX
004B926B  |.  E8 0C0FF7FF  CALL DiskClea.0042A17C
004B9270  |.  84C0        TEST AL,AL
004B9272  |.  74 1F       JE SHORT DiskClea.004B9293          ;  //关键跳转,改为 JNZ
004B9274  |.  8D4D FC     LEA ECX,DWORD PTR SS:\[EBP-4\]
004B9277  |.  BA E8924B00 MOV EDX,DiskClea.004B92E8           ;  diskcln20
004B927C  |.  8BC3        MOV EAX,EBX
004B927E  |.  E8 7D12F7FF  CALL DiskClea.0042A500
004B9283  |.  8B45 FC     MOV EAX,DWORD PTR SS:\[EBP-4\]
004B9286  |.  E8 69000000 CALL DiskClea.004B92F4
004B928B  |.  84C0        TEST AL,AL
004B928D  |.  75 04       JNZ SHORT DiskClea.004B9293
004B928F  |.  33DB        XOR EBX,EBX
004B9291  |.  EB 02       JMP SHORT DiskClea.004B9295
004B9293  |>  B3 01       MOV BL,1
004B9295  |>  33C0        XOR EAX,EAX
004B9297  |.  5A          POP EDX
004B9298  |.  59          POP ECX
004B9299  |.  59          POP ECX
004B929A  |.  64:8910     MOV DWORD PTR FS:\[EAX\],EDX
004B929D  |.  68 B2924B00 PUSH DiskClea.004B92B2
004B92A2  |>  8D45 FC     LEA EAX,DWORD PTR SS:\[EBP-4\]
004B92A5  |.  E8 CEB6F4FF  CALL DiskClea.00404978
004B92AA  \\.  C3         RETN
004B92AB   .^  E9 10B0F4FF JMP DiskClea.004042C0
004B92B0   .^  EB F0      JMP SHORT DiskClea.004B92A2
004B92B2   .  8BC3        MOV EAX,EBX
004B92B4   .  5B          POP EBX
004B92B5   .  59          POP ECX
004B92B6   .  5D          POP EBP
004B92B7   .  C3          RETN
```

### 代码段 3

```
004BDDF8  /.  55           PUSH EBP
004BDDF9  |.  8BEC        MOV EBP,ESP
004BDDFB  |.  6A 00       PUSH 0
004BDDFD  |.  53          PUSH EBX
004BDDFE  |.  56          PUSH ESI
004BDDFF  |.  57          PUSH EDI
004BDE00  |.  8BF8        MOV EDI,EAX
004BDE02  |.  33C0        XOR EAX,EAX
004BDE04  |.  55          PUSH EBP
004BDE05  |.  68 B8DE4B00  PUSH DiskClea.004BDEB8
004BDE0A  |.  64:FF30     PUSH DWORD PTR FS:\[EAX\]
004BDE0D  |.  64:8920     MOV DWORD PTR FS:\[EAX\],ESP
004BDE10  |.  B2 01       MOV DL,1
004BDE12  |.  A1 789F4200  MOV EAX,DWORD PTR DS:\[429F78\]
004BDE17  |.  E8 5CC2F6FF  CALL DiskClea.0042A078
004BDE1C  |.  8BF0        MOV ESI,EAX
004BDE1E  |.  BA 02000080 MOV EDX,80000002
004BDE23  |.  8BC6        MOV EAX,ESI
004BDE25  |.  E8 EEC2F6FF  CALL DiskClea.0042A118
004BDE2A  |.  B3 01       MOV BL,1
004BDE2C  |.  B1 01       MOV CL,1
004BDE2E  |.  BA D0DE4B00 MOV EDX,DiskClea.004BDED0          ;  \software\winaso\disk cleaner
004BDE33  |.  8BC6        MOV EAX,ESI
004BDE35  |.  E8 42C3F6FF  CALL DiskClea.0042A17C
004BDE3A  |.  84C0        TEST AL,AL
004BDE3C  |.  74 1B       JE SHORT DiskClea.004BDE59          ;  //关键跳转,改为 JNZ
004BDE3E  |.  8D4D FC     LEA ECX,DWORD PTR SS:\[EBP-4\]
004BDE41  |.  BA F8DE4B00 MOV EDX,DiskClea.004BDEF8           ;  diskcln20
004BDE46  |.  8BC6        MOV EAX,ESI
004BDE48  |.  E8 B3C6F6FF  CALL DiskClea.0042A500
004BDE4D  |.  8B45 FC     MOV EAX,DWORD PTR SS:\[EBP-4\]
004BDE50  |.  E8 9FB4FFFF  CALL DiskClea.004B92F4
004BDE55  |.  84C0        TEST AL,AL
004BDE57  |.  74 49       JE SHORT DiskClea.004BDEA2
004BDE59  |>  80FB 01     CMP BL,1
004BDE5C  |.  75 2A       JNZ SHORT DiskClea.004BDE88
004BDE5E  |.  33D2        XOR EDX,EDX
004BDE60  |.  8B87 74030000 MOV EAX,DWORD PTR DS:\[EDI+374\]
004BDE66  |.  E8 99FAF8FF  CALL DiskClea.0044D904
004BDE6B  |.  8B55 FC     MOV EDX,DWORD PTR SS:\[EBP-4\]
004BDE6E  |.  8B87 8C030000 MOV EAX,DWORD PTR DS:\[EDI+38C\]
004BDE74  |.  E8 9BFBF8FF  CALL DiskClea.0044DA14
004BDE79  |.  B2 01       MOV DL,1
004BDE7B  |.  8B87 80030000 MOV EAX,DWORD PTR DS:\[EDI+380\]
004BDE81  |.  E8 7EFAF8FF  CALL DiskClea.0044D904
004BDE86  |.  EB 1A       JMP SHORT DiskClea.004BDEA2
004BDE88  |>  B2 01       MOV DL,1
004BDE8A  |.  8B87 74030000 MOV EAX,DWORD PTR DS:\[EDI+374\]
004BDE90  |.  E8 6FFAF8FF  CALL DiskClea.0044D904
004BDE95  |.  33D2        XOR EDX,EDX
004BDE97  |.  8B87 80030000 MOV EAX,DWORD PTR DS:\[EDI+380\]
004BDE9D  |.  E8 62FAF8FF  CALL DiskClea.0044D904
004BDEA2  |>  33C0        XOR EAX,EAX
004BDEA4  |.  5A          POP EDX
004BDEA5  |.  59          POP ECX
004BDEA6  |.  59          POP ECX
004BDEA7  |.  64:8910     MOV DWORD PTR FS:\[EAX\],EDX
004BDEAA  |.  68 BFDE4B00 PUSH DiskClea.004BDEBF
004BDEAF  |>  8D45 FC     LEA EAX,DWORD PTR SS:\[EBP-4\]
004BDEB2  |.  E8 C16AF4FF  CALL DiskClea.00404978
004BDEB7  \\.  C3         RETN
004BDEB8   .^  E9 0364F4FF JMP DiskClea.004042C0
004BDEBD   .^  EB F0      JMP SHORT DiskClea.004BDEAF
004BDEBF   .  5F          POP EDI
004BDEC0   .  5E          POP ESI
004BDEC1   .  5B          POP EBX
004BDEC2   .  59          POP ECX
004BDEC3   .  5D          POP EBP
004BDEC4   .  C3          RETN
```
