
```
    │00609B94    .MOV EAX,DWORD PTR SS:[EBP-10]                                             │  
    │00609B97    .CALL Farsight.00442A14                                                     │  
    │00609B9C    .MOV ECX,DWORD PTR SS:[EBP-8]                                              │  
    │00609B9F    .MOV EDX,Farsight.00609C58          ;  ASCII "Sn"                           │  
    │00609BA4    .MOV EAX,DWORD PTR SS:[EBP-10]                                             │  
    │00609BA7    .CALL Farsight.00442A14                                                     │  
    │00609BAC    .MOV BYTE PTR SS:[EBP-9],1                                                 │  
    │00609BB0    .XOR EAX,EAX                                                                │  
    │00609BB2    .POP EDX                                                                    │  
    │00609BB3    .POP ECX                                                                    │  
    │00609BB4    .POP ECX                                                                    │  
    │00609BB5    .MOV DWORD PTR FS:[EAX],EDX                                                │  
    │00609BB8    .PUSH Farsight.00609BCD                                                     │  
    │00609BBD    >MOV EAX,DWORD PTR SS:[EBP-10]                                             │  
    │00609BC0    .CALL Farsight.00403B24                                                     │  
    │00609BC5    .RETN                                                                      │  
    └──────────────────────────────────────────────┘  
```

连这里都一样,不废话了,`F8` 至 `609B33`,再 `F7`,来到,

```
    ┌──────────────────────────────────────────────┐  
    │006099B0  /$PUSH EBP                                                                 │  
    │006099B1  |.MOV EBP,ESP                                                              │  
    │006099B3  |.ADD ESP,-0C                                                               │  
    │006099B6  |.PUSH EBX                                                                  │  
    │006099B7  |.XOR ECX,ECX                                                               │  
    │006099B9  |.MOV DWORD PTR SS:[EBP-C],ECX                                              │  
    │006099BC  |.MOV DWORD PTR SS:[EBP-8],EDX       ;  注册码入栈                         │  
    │006099BF  |.MOV DWORD PTR SS:[EBP-4],EAX       ;  注册名入栈                         │  
    │006099C2  |.MOV EAX,DWORD PTR SS:[EBP-4]       ;  EAX = 注册名                       │  
    │006099C5  |.CALL Farsight.00404EC8                                                     │  
    │006099CA  |.MOV EAX,DWORD PTR SS:[EBP-8]                                              │  
    │006099CD  |.CALL Farsight.00404EC8                                                     │  
    │006099D2  |.XOR EAX,EAX                                                               │  
    │006099D4  |.PUSH EBP                                                                  │  
    │006099D5  |.PUSH Farsight.00609AED                                                    │  
    │006099DA  |.PUSH DWORD PTR FS:[EAX]                                                   │  
    │006099DD  |.MOV DWORD PTR FS:[EAX],ESP                                                │  
    │006099E0  |.XOR EBX,EBX                                                               │  
    │006099E2  |.LEA EDX,DWORD PTR SS:[EBP-C]                                              │  
    │006099E5  |.MOV EAX,DWORD PTR SS:[EBP-8]       ;  EAX = 注册码                       │  
    │006099E8  |.CALL Farsight.00409310            ;  获取有效字符                        │  
    │006099ED  |.CMP DWORD PTR SS:[EBP-C],0                                                │  
    │006099F1  |.JE Farsight.00609AD2                                                        │  
    │006099F7  |.MOV EDX,DWORD PTR SS:[EBP-8]       ;  EDX = 注册码                       │  
    │006099FA  |.MOV EAX,DWORD PTR SS:[EBP-4]       ;  EAX = 注册名                       │  
    │006099FD  |.CALL Farsight.00609128                                                    │  
    │00609A02  |.TEST AL,AL                                                                 │  
    │00609A04  |.JE SHORT Farsight.00609A0D                                                  │  
    │00609A06  |.XOR EBX,EBX                                                                 │  
    │00609A08  |.JMP Farsight.00609AD2                                                       │  
    │00609A0D  |>MOV EDX,DWORD PTR SS:[EBP-8]       ;  EDX = 注册码                       │  
    │00609A10  |.MOV EAX,DWORD PTR SS:[EBP-4]       ;  EAX = 注册名                       │  
    │00609A13  |.CALL Farsight.00608C64            ;                                        │  
    │00609A18  |.TEST AL,AL                                                                 │  
    │00609A1A  |.JE SHORT Farsight.00609A23                                                  │  
    │00609A1C  |.MOV BL,1                                                                    │  
    │00609A1E  |.JMP Farsight.00609AD2                                                       │  
    │00609A23  |>MOV EDX,DWORD PTR SS:[EBP-8]       ;  注册码                              │  
    │00609A26  |.MOV EAX,DWORD PTR SS:[EBP-4]       ;  注册名                              │  
    │00609A29  |.CALL Farsight.00609218            ;  比较注册码第 1 位                    │  
    │00609A2E  |.TEST AL,AL                                                                 │  
    │00609A30  |.JE Farsight.00609AD2               ;  一跳就死                           │  
    │00609A36  |.MOV EDX,DWORD PTR SS:[EBP-8]                                              │  
    │00609A39  |.MOV EAX,DWORD PTR SS:[EBP-4]                                              │  
    │00609A3C  |.CALL Farsight.006092C4            ;  比较注册码第 2 位                    │  
    │00609A41  |.TEST AL,AL                                                                 │  
    │00609A43  |.JE Farsight.00609AD2                                                       │  
    │00609A49  |.MOV EDX,DWORD PTR SS:[EBP-8]                                              │  
    │00609A4C  |.MOV EAX,DWORD PTR SS:[EBP-4]                                              │  
    │00609A4F  |.CALL Farsight.00609378            ;  比较注册码第 3 位                    │  
    │00609A54  |.TEST AL,AL                                                                 │  
    │00609A56  |.JE SHORT Farsight.00609AD2                                                  │  
    │00609A58  |.MOV EDX,DWORD PTR SS:[EBP-8]                                              │  
    │00609A5B  |.MOV EAX,DWORD PTR SS:[EBP-4]                                              │  
    │00609A5E  |.CALL Farsight.00609428            ;  比较注册码第 4 位                    │  
    │00609A63  |.TEST AL,AL                                                                 │  
    │00609A65  |.JE SHORT Farsight.00609AD2                                                  │  
    │00609A67  |.MOV EDX,DWORD PTR SS:[EBP-8]                                              │  
    │00609A6A  |.MOV EAX,DWORD PTR SS:[EBP-4]                                              │  
    │00609A6D  |.CALL Farsight.006094DC            ;  比较注册码第 5 位                    │  
    │00609A72  |.TEST AL,AL                                                                 │  
