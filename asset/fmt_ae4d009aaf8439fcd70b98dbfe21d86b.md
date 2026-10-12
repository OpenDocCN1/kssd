
```
004152AA    2BF2                 SUB ESI,EDX                               ; 00416B7B  

004152AC    B9 0A000000          MOV ECX,0A  

004152B1    BA C93EEF5E          MOV EDX,5EEF3EC9  

004152B6    AD                   LODS DWORD PTR DS:[ESI]  

004152B7    4A                   DEC EDX  

004152B8    03C2                 ADD EAX,EDX  

004152BA    42                   INC EDX  

004152C7    33C2                 XOR EAX,EDX  

004152C9    4A                   DEC EDX  

004152CA    C1C2 08              ROL EDX,8  

004152CD    AB                   STOS DWORD PTR ES:[EDI]  

004152CE   ^ E2 E6              LOOPD SHORT PESpin.004152B6               ; 复制 28h 字节  

004152D0    B9 11000000          MOV ECX,11  

004152D5    8DB5 BA5E4000        LEA ESI,DWORD PTR SS:[EBP+405EBA]         ; 0041520E  

004152DB    F3:A4                REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]    ; 复制 11h byte  

004152EE    93                   XCHG EAX,EBX  

004152EF    B9 0A000000          MOV ECX,0A  

004152F4    8BBD 4D5F4000        MOV EDI,DWORD PTR SS:[EBP+405F4D]  

004152FA    2BBD 525F4000        SUB EDI,DWORD PTR SS:[EBP+405F52]  

00415300    F3:AB                REP STOS DWORD PTR ES:[EDI]              ; 00416B7B (破坏)  

...  

004135F6    8BB5 43454000        MOV ESI,DWORD PTR SS:[EBP+404543]        ; [00413897] = C160,   真正的 IAT 信息放在这  

004135FC    03B5 DF4E4000        ADD ESI,DWORD PTR SS:[EBP+404EDF]        ; PESpin.00400000  

    0040C160  00 00 00 00 00 00 00 00 00 00 00 00 C8 C4 00 00  ................  ; 真正的 IAT, 6 个 DLL       
    0040C170  2C C0 00 00  
                 00 00 00 00 00 00 00 00 00 00 00 00  ,...............  
    0040C180  80 C6 00 00 B0 C0 00 00  
                                     00 00 00 00 00 00 00 00  ................  
    0040C190  00 00 00 00 D2 C6 00 00 00 C0 00 00  
                                     00 00 00 00  00 00 00 00  ................  
    0040C1A0  00 00 00 00 00 00 00 00 F4 C6 00 00 10 C0 00 00  ................  
    0040C1B0  00 00 00 00 00 00 00 00 00 00 00 00 44 C7 00 00  ............D...  
    0040C1C0  9C C0 00 00  
                 00 00 00 00 00 00 00 00 00 00 00 00  ................  
    0040C1D0  90 C7 00 00 18 C0 00 00  
                                     00 00 00 00 00 00 00 00  ................  ; 结束的标志  
    0040C1E0  00 00 00 00 2E 1F 01 00 10 1F 01 00                  ............  
00413613    3BB5 DF4E4000        CMP ESI,DWORD PTR SS:[EBP+404EDF]        ; PESpin.00400000  

00413625   /75 0F                JNZ SHORT PESpin.00413636               ; 应该跳  

00413636    817E 10 101F0100     CMP DWORD PTR DS:[ESI+10],11F10          ; DLL 处理完的标志  

...  

0041366A   /FF6424 FC            JMP DWORD PTR SS:[ESP-4]               ; 0041366F(没完), 004138AB(处理完了)  

0041366F    8B5E 0C              MOV EBX,DWORD PTR DS:[ESI+C]           ; 指向 DLL Name  

00413672    039D DF4E4000        ADD EBX,DWORD PTR SS:[EBP+404EDF]  

00413678    8BFB                 MOV EDI,EBX  

0041367A    E8 8B130000          CALL PESpin.00414A0A                   ; 取反解密, F8  

00413686    53                   PUSH EBX  

00413687    50                   PUSH EAX  

00413688    FFB5 A24F4000        PUSH DWORD PTR SS:[EBP+404FA2]          ; KERNEL32.LoadLibraryA  

0041368E    814424 04 14000000   ADD DWORD PTR SS:[ESP+4],14            ; 返回地址 00413698  

00413696    C3                   RETN  

00413698    85C0                 TEST EAX,EAX                             ; KERNEL32.7C570000  

0041369A    0F84 AD090000        JE PESpin.0041404D  

004136A0    E8 01000000          CALL PESpin.004136A6  

004136A6    59                   POP ECX                                  ; PESpin.004136A5  

004136A7    50                   PUSH EAX                                 ; DLL Base  

004136A8    51                   PUSH ECX  

004136A9    55                   PUSH EBP  

004136AA    810424 66394000      ADD DWORD PTR SS:[ESP],PESpin.00403966   ; 00412CBA  

004136B1    814424 04 22000>     ADD DWORD PTR SS:[ESP+4],22              ; 返回地址 004136C7  

004136B9    C3                   RETN                                     ; Call 00412CBA  

;=======================================================================================================================  

00412CBA    分析 DLL 的 Export 一段子程序  

00412CBD    59                   POP ECX                                    ; PESpin.004136C7  

00412CBE    5A                   POP EDX                                    ; DLL Base  

00412CBF    41                   INC ECX  

00412CC0    51                   PUSH ECX                                   ; 返回地址 004136C8  

00412CCA    57                   PUSH EDI                                   ; PESpin.0040C4C8  

00412CCB    53                   PUSH EBX                                   ; DLL Name  

00412CCC    8995 FE394000        MOV DWORD PTR SS:[EBP+4039FE],EDX         ; [00412D52]=7C570000  

00412CD2    8BDA                 MOV EBX,EDX                                ; KERNEL32.7C570000  

00412CD4    0352 3C              ADD EDX,DWORD PTR DS:[EDX+3C]             ; DLL PE 头  

00412CD7    FF72 7C              PUSH DWORD PTR DS:[EDX+7C]                ; DLL Export Size  

00412CDA    8F85 F6394000        POP DWORD PTR SS:[EBP+4039F6]             ; 00412D4A  

00412CE0    8B52 78              MOV EDX,DWORD PTR DS:[EDX+78]             ; DLL Export RVA  

00412CE3    03D3                 ADD EDX,EBX  

00412CE5    52                   PUSH EDX  

00412CE6    8F85 F2394000        POP DWORD PTR SS:[EBP+4039F2]             ; 00412D46  

00412CF5    FF72 20              PUSH DWORD PTR DS:[EDX+20]                ; AddressOfNames  

00412CF8    5F                   POP EDI  

00412CF9    03FB                 ADD EDI,EBX  

00412CFB    57                   PUSH EDI  

00412CFC    8F85 023A4000        POP DWORD PTR SS:[EBP+403A02]             ; 00412D56  

00412D02    FF72 18              PUSH DWORD PTR DS:[EDX+18]                ; NumberofNames  

00412D05    8F85 D93A4000        POP DWORD PTR SS:[EBP+403AD9]             ; 00412E2D  

00412D1A    FF72 1C              PUSH DWORD PTR DS:[EDX+1C]                ; AddressOfFunctions                      

00412D1D    5F                   POP EDI  

00412D1E    03FB                 ADD EDI,EBX  

00412D20    57                   PUSH EDI  

00412D21    8F85 063A4000        POP DWORD PTR SS:[EBP+403A06]             ; 00412D5A  

00412D27    FF72 24              PUSH DWORD PTR DS:[EDX+24]                ; AddressOfNameOrdinals  

00412D2A    5F                   POP EDI  

00412D2B    03FB                 ADD EDI,EBX  

00412D2D    57                   PUSH EDI  

00412D2E    8F85 FA394000        POP DWORD PTR SS:[EBP+4039FA]             ; 00412D4E  

00412D34    FF72 10              PUSH DWORD PTR DS:[EDX+10]                ; Base  

00412D37    8F85 203A4000        POP DWORD PTR SS:[EBP+403A20]             ; 00412D74  

00412D40    5B                   POP EBX                                    ; PESpin.0040C4C8  

00412D41    5F                   POP EDI  

00412D42    C3                   RETN                                       ; 004136C8  

;===========================================================================================================================  

004136C8    2BD2                 SUB EDX,EDX  

004136F7    800B 00              OR BYTE PTR DS:[EBX],0                    ; 破坏 DLL Name  

004136FA    74 0D                JE SHORT PESpin.00413709                 ; 出口  

004136FC    8813                 MOV BYTE PTR DS:[EBX],DL  
```
