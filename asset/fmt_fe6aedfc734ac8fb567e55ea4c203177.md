
```
053A5AED  |.  0345 DC       |ADD EAX,DWORD PTR SS:[EBP-24]        ; 空间指针 3  
053A5AF0  |.  56           |PUSH ESI  
053A5AF1  |.  50           |PUSH EAX  
053A5AF2  |.  E8 B0EBFFFF   |CALL EXERe.053A46A7                ; uppercase  
053A5AF7  |.  59           |POP ECX  
053A5AF8  |.  50           |PUSH EAX  
053A5AF9  |.  E8 0B1A0000   |CALL EXERe.053A7509                ; Hash 变换  
053A5AFE  |.  59           |POP ECX  
053A5AFF  |.  8945 9C      |MOV DWORD PTR SS:[EBP-64],EAX       ; API Name Hash      
053A5B02  |.  59           |POP ECX  
053A5B03  |>  8B85 7CFFFFFF |MOV EAX,DWORD PTR SS:[EBP-84]       ; Export Table 中的 base  
053A5B09  |.  6A 04        |PUSH 4                             ; Hash 变换只取 4 字节  
053A5B0B  |.  03C7        |ADD EAX,EDI                        ; API 对外的 Ordinal  
053A5B0D  |.  8945 E0      |MOV DWORD PTR SS:[EBP-20],EAX  
053A5B10  |.  8D45 E0      |LEA EAX,DWORD PTR SS:[EBP-20]  
053A5B13  |.  50           |PUSH EAX  
053A5B14  |.  E8 F0190000   |CALL EXERe.053A7509                ; 对 Ordinal 做 Hash 变换  
053A5B19  |.  8945 98      |MOV DWORD PTR SS:[EBP-68],EAX       ; API Ordinal Hash  
053A5B1C  |.  8B45 F0      |MOV EAX,DWORD PTR SS:[EBP-10]       ;  
053A5B1F  |.  59           |POP ECX  
053A5B20  |.  8945 A0      |MOV DWORD PTR SS:[EBP-60],EAX         
053A5B23  |.  59           |POP ECX  
053A5B24  |.  8D45 94      |LEA EAX,DWORD PTR SS:[EBP-6C]  
053A5B27  |.  50           |PUSH EAX  
053A5B28  |.  8D4D B8      |LEA ECX,DWORD PTR SS:[EBP-48]       ; 70FF6C  
053A5B2B  |.  897D 94     |MOV DWORD PTR SS:[EBP-6C],EDI  
053A5B2E  |.  E8 31030000   |CALL EXERe.053A5E64                ; 修正 70FF6C-70FF74 所指的 NameOrdinalHash 列表  
053A5B33  |>  8345 08 04    |ADD DWORD PTR SS:[EBP+8],4  
053A5B37  |.  47           |INC EDI  
053A5B38  |.  3B7D 80      |CMP EDI,DWORD PTR SS:[EBP-80]  
053A5B3B  |.^ 7C 80         \JL SHORT EXERe.053A5ABD  
```

```
structure NameOrdinalHash  
{  
    dword order_in_AddressofFunctions;  
    dword OrdinalHash;  
    dword NameHash;  
    dword order_in_AddressofNameOrdinal;  
}  
```

```
053A5B3D  |>  A1 A0A03B05   /MOV EAX,DWORD PTR DS:[53BA0A0]     ; 都处理好了, 开始等待主线程的命令  
053A5B42  |.  6A 02        |PUSH 2  
053A5B44  |.  F7D0         |NOT EAX  
053A5B46  |.   FFD0        |CALL EAX                           ; Sleep(2ms)  
053A5B48  |.  803D 308C3B05>|CMP BYTE PTR DS:[53B8C30],0         ; 程序结束命令送到这  
053A5B4F  |.  0F85 E1010000 |JNZ EXERe.053A5D36  
053A5B55  |.  A1 94A03B05   |MOV EAX,DWORD PTR DS:[53BA094]     ; 程序没结束  
053A5B5A  |.  F7D0         |NOT EAX  
053A5B5C  |.  FFD0         |CALL EAX                           ; GetTickCount  
053A5B5E  |.  8B0D E0A03B05 |MOV ECX,DWORD PTR DS:[53BA0E0]     ; 053C0087 做的时间标志      
053A5B64  |.  BF E8030000   |MOV EDI,3E8                        ; 1000  
053A5B69  |.  2BC1         |SUB EAX,ECX                        ; 时间差  
053A5B6B  |.  99           |CDQ  
053A5B6C  |.  F7FF         |IDIV EDI  
053A5B6E  |.  33D2         |XOR EDX,EDX  
053A5B70  |.  8BF8         |MOV EDI,EAX                        ; 过了几秒  
053A5B72  |.  8BC1         |MOV EAX,ECX  
053A5B74  |.  B9 2C010000   |MOV ECX,12C                        ; 300  
053A5B79  |.  F7F1         |DIV ECX  
053A5B7B  |.  03D1         |ADD EDX,ECX  
053A5B7D  |.  3BFA         |CMP EDI,EDX                        ; 唯一的一个 Anti  
053A5B7F  |.  7E 0C        |JLE SHORT EXERe.053A5B8D           ; 改成 JMP  ******************************
053A5B81  |.  A1 A8A03B05   |MOV EAX,DWORD PTR DS:[53BA0A8]  
053A5B86  |.  56           |PUSH ESI  
053A5B87  |.  6A FF        |PUSH -1  
053A5B89  |.  F7D0         |NOT EAX  
053A5B8B  |.  FFD0         |CALL EAX                           ; TerminateProcess  
053A5B8D  |>  A1 508C3B05   |MOV EAX,DWORD PTR DS:[53B8C50]     ; 主线程的命令送到这  
053A5B92  |.  3BC6         |CMP EAX,ESI  
053A5B94  |.^ 7E A7         |JLE SHORT EXERe.053A5B3D  
053A5B96  |.  8975 F0      |MOV DWORD PTR SS:[EBP-10],ESI  
053A5B99  |.  8945 E0      |MOV DWORD PTR SS:[EBP-20],EAX      ; 队列中需要处理的 API 个数  
053A5B9C  |>  A1 488C3B05   /MOV EAX,DWORD PTR DS:[53B8C48]     ; 开始地址  
053A5BA1  |.  8B4D F0      ||MOV ECX,DWORD PTR SS:[EBP-10]      ; 已处理过  
053A5BA4  |.  03C1        ||ADD EAX,ECX  
053A5BA6  |.  8B4D A8      ||MOV ECX,DWORD PTR SS:[EBP-58]      ; Dll Name Hash(线程的局部变量)  
053A5BA9  |.  8945 F8      ||MOV DWORD PTR SS:[EBP-8],EAX  
053A5BAC  |.  3908        ||CMP DWORD PTR DS:[EAX],ECX         ; 不相等说明不是本 DLL 的, 队列中下一个 API  
053A5BAE  |.  0F85 6E010000 ||JNZ EXERe.053A5D22  
053A5BB4  |.  8065 0B 00    ||AND BYTE PTR SS:[EBP+B],0  
053A5BB8  |.  8B40 08      ||MOV EAX,DWORD PTR DS:[EAX+8]      ; 0 或 API Address  
053A5BBB  |.  8B55 C0      ||MOV EDX,DWORD PTR SS:[EBP-40]     ; NameOrdinalHash 列表中个数  
053A5BBE  |.  8BF0        ||MOV ESI,EAX  
053A5BC0  |.  B9 000000F0   ||MOV ECX,F0000000  
053A5BC5  |.  8955 C8      ||MOV DWORD PTR SS:[EBP-38],EDX  
053A5BC8  |.  23F1        ||AND ESI,ECX  
053A5BCA  |.  3BF1        ||CMP ESI,ECX  
053A5BCC  |.  74 08       ||JE SHORT EXERe.053A5BD6           ; Fxxxxxxx 重新来过  
053A5BCE  |.  85C0        ||TEST EAX,EAX                       ; 0 or API address  
053A5BD0  |.  0F85 4C010000 ||JNZ EXERe.053A5D22               ; 不等于 0 的话就跳过  
053A5BD6  |>  8365 EC 00    ||AND DWORD PTR SS:[EBP-14],0  
053A5BDA  |.  85D2        ||TEST EDX,EDX  
053A5BDC  |.  0F8E 40010000 ||JLE EXERe.053A5D22  
053A5BE2  |>  8B45 EC      ||/MOV EAX,DWORD PTR SS:[EBP-14]  
053A5BE5  |.  8365 E8 00    |||AND DWORD PTR SS:[EBP-18],0      ; 0, 1 两种情况  
053A5BE9  |.  C1E0 04      |||SHL EAX,4  
053A5BEC  |.  8945 A4      |||MOV DWORD PTR SS:[EBP-5C],EAX  
053A5BEF  |>  8B45 B8      |||/MOV EAX,DWORD PTR SS:[EBP-48]   ; NameOrdianlHash 列表开始地址  
053A5BF2  |.  8B4D A4      ||||MOV ECX,DWORD PTR SS:[EBP-5C]  
053A5BF5  |.  8DBD 5CFFFFFF ||||LEA EDI,DWORD PTR SS:[EBP-A4]  
053A5BFB  |.  8D3401        ||||LEA ESI,DWORD PTR DS:[ECX+EAX]  
053A5BFE  |.  8B45 E8      ||||MOV EAX,DWORD PTR SS:[EBP-18]  
053A5C01  |.  A5           ||||MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  
053A5C02  |.  A5           ||||MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  
053A5C03  |.  A5           ||||MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  
053A5C04  |.  83E8 00      ||||SUB EAX,0                          ;  Switch (cases 0..1)  
053A5C07  |.  A5           ||||MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  
053A5C08  |.  74 0B       ||||JE SHORT EXERe.053A5C15  
053A5C0A  |.  48           ||||DEC EAX  
053A5C0B  |.  75 11       ||||JNZ SHORT EXERe.053A5C1E  
053A5C0D  |.  8B85 60FFFFFF ||||MOV EAX,DWORD PTR SS:[EBP-A0]      ;  Case 1 of switch 053A5C04, 对应 OridnalHash, 没有名字  
053A5C13  |.  EB 06        ||||JMP SHORT EXERe.053A5C1B  
053A5C15  |>  8B85 64FFFFFF ||||MOV EAX,DWORD PTR SS:[EBP-9C]      ;  Case 0 of switch 053A5C04, 对应 NameHash  
053A5C1B  |>  8945 D4      ||||MOV DWORD PTR SS:[EBP-2C],EAX  
053A5C1E  |>  8B55 B4      ||||MOV EDX,DWORD PTR SS:[EBP-4C]      ;  Default case of switch 053A5C04, 是否是指向另一 DLL 的 API  
053A5C21  |.  8065 FF 00    ||||AND BYTE PTR SS:[EBP-1],0  
053A5C25  |.  8365 E4 00    ||||AND DWORD PTR SS:[EBP-1C],0  
053A5C29  |.  8B85 5CFFFFFF ||||MOV EAX,DWORD PTR SS:[EBP-A4]  
053A5C2F  |.  85D2        ||||TEST EDX,EDX  
053A5C31  |.  7E 20       ||||JLE SHORT EXERe.053A5C53  
053A5C33  |.  8B4D AC      ||||MOV ECX,DWORD PTR SS:[EBP-54]  
053A5C36  |>  8B31        ||||/MOV ESI,DWORD PTR DS:[ECX]  
053A5C38  |.  8B79 04      |||||MOV EDI,DWORD PTR DS:[ECX+4]  
053A5C3B  |.  3BF0        |||||CMP ESI,EAX  
```
