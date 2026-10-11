
```
0047E031   .  51            PUSH ECX                                        ; /Arg4  

0047E032   .  8D8D 3CFFFFFF LEA ECX,DWORD PTR SS:[EBP-C4]                   ; |  

0047E038   .  50            PUSH EAX                                        ; |Arg3  

0047E039   .  8D95 5CFFFFFF LEA EDX,DWORD PTR SS:[EBP-A4]                   ; |  

0047E03F   .  51            PUSH ECX                                        ; |Arg2  

0047E040   .  52            PUSH EDX                                        ; |Arg1  

0047E041   .  C785 74FFFFFF>MOV DWORD PTR SS:[EBP-8C],1                     ; |  

0047E04B   .  C785 6CFFFFFF>MOV DWORD PTR SS:[EBP-94],2                     ; |  

0047E055   .  C785 3CFFFFFF>MOV DWORD PTR SS:[EBP-C4],4008                  ; |  

0047E05F   .  FF15 24114000 CALL DWORD PTR DS:[<&msvbvm60.rtcMidCharVar>]   ; \rtcMidCharVar  

0047E065   .  8D85 5CFFFFFF LEA EAX,DWORD PTR SS:[EBP-A4]                  

0047E06B   .  8D8D 4CFFFFFF LEA ECX,DWORD PTR SS:[EBP-B4]                  

0047E071   .  50            PUSH EAX                                        

0047E072   .  51            PUSH ECX                                        

0047E073   .  FFD7          CALL EDI                                        ;  <&msvbvm60.rtcUpperCaseVar>  

0047E075   .  8B3D F8114000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaStrVarVal>]  ;  msvbvm60.__vbaStrVarVal  

0047E07B   .  8D95 4CFFFFFF LEA EDX,DWORD PTR SS:[EBP-B4]                  

0047E081   .  8D45 9C       LEA EAX,DWORD PTR SS:[EBP-64]                  

0047E084   .  52            PUSH EDX                                        

0047E085   .  50            PUSH EAX                                        

0047E086   .  FFD7          CALL EDI                                        ;  <&msvbvm60.__vbaStrVarVal>  

0047E088   .  50            PUSH EAX                                        

0047E089   .  FFD6          CALL ESI                                        

0047E08B   .  66:8BD0       MOV DX,AX                                        

0047E08E   .  8D8D 7CFFFFFF LEA ECX,DWORD PTR SS:[EBP-84]                  

0047E094   .  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]                  

0047E097   .  51            PUSH ECX                                        

0047E098   .  50            PUSH EAX                                        

0047E099   .  66:8995 A4FEF>MOV WORD PTR SS:[EBP-15C],DX                  

0047E0A0   .  FFD7          CALL EDI                                        ;  <&msvbvm60.__vbaStrVarVal>  

0047E0A2   .  50            PUSH EAX                                        

0047E0A3   .  FFD6          CALL ESI                                        

0047E0A5   .  66:8B95 A4FEF>MOV DX,WORD PTR SS:[EBP-15C]                  

0047E0AC   .  33C9          XOR ECX,ECX                                      

0047E0AE   .  66:3BC2       CMP AX,DX        〈---这里比较的是第 2 组的，把 DX 的值记起来。  

0047E0B1   .  8D45 9C       LEA EAX,DWORD PTR SS:[EBP-64]                  

0047E0B4   .  0F95C1        SETNE CL                                        

0047E0B7   .  F7D9          NEG ECX                                        

0047E0B9   .  8BF9          MOV EDI,ECX                                      

0047E0BB   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]                  

0047E0BE   .  50            PUSH EAX                                        

0047E0BF   .  51            PUSH ECX                                        

0047E0C0   .  6A 02         PUSH 2                                           

0047E0C2   .  FF15 54124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeStrList>] ;  msvbvm60.__vbaFreeStrList  

0047E0C8   .  8D95 4CFFFFFF LEA EDX,DWORD PTR SS:[EBP-B4]                  

0047E0CE   .  8D85 5CFFFFFF LEA EAX,DWORD PTR SS:[EBP-A4]                  

0047E0D4   .  52            PUSH EDX                                        

0047E0D5   .  8D8D 6CFFFFFF LEA ECX,DWORD PTR SS:[EBP-94]                  

0047E0DB   .  50            PUSH EAX                                        

0047E0DC   .  8D95 7CFFFFFF LEA EDX,DWORD PTR SS:[EBP-84]                  

0047E0E2   .  51            PUSH ECX                                        

0047E0E3   .  8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]                  

0047E0E6   .  52            PUSH EDX                                        

0047E0E7   .  50            PUSH EAX                                        

0047E0E8   .  6A 05         PUSH 5                                           

0047E0EA   .  FF15 44104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeVarList>] ;  msvbvm60.__vbaFreeVarList  

0047E0F0   .  83C4 24       ADD ESP,24                                       

0047E0F3   .  66:85FF       TEST DI,DI                                       

0047E0F6   .  0F85 A3010000 JNZ 扩展名仓.0047E29F                              

0047E0FC   .  B8 01000000   MOV EAX,1                                        

0047E101   .  66:03C3       ADD AX,BX                                        

0047E104   .  0F80 29020000 JO 扩展名仓.0047E333                               

0047E10A   .  8BD8          MOV EBX,EAX                                      

0047E10C   .  ^ E9 D6FCFFFF  JMP 扩展名仓.0047DDE7                             

0047E111   >  C785 D4FEFFFF>MOV DWORD PTR SS:[EBP-12C],4                     

0047E11B   .  C785 D8FEFFFF>MOV DWORD PTR SS:[EBP-128],1                     

0047E125   .  33DB          XOR EBX,EBX                                      

0047E127   >  66:3B9D D4FEF>CMP BX,WORD PTR SS:[EBP-12C]                     

0047E12E   .  0F8F 72010000 JG 扩展名仓.0047E2A6                               

0047E134   .  0FBFFB        MOVSX EDI,BX                                     

0047E137   .  83FF 05       CMP EDI,5                                        

0047E13A   .  72 11         JB SHORT 扩展名仓.0047E14D                          

0047E13C   .  FF15 40114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaGenerateBoundsEr>;  msvbvm60.__vbaGenerateBoundsError  

0047E142   .  83FF 05       CMP EDI,5                                        

0047E145   .  72 06         JB SHORT 扩展名仓.0047E14D                          

0047E147   .  FF15 40114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaGenerateBoundsEr>;  msvbvm60.__vbaGenerateBoundsError  

0047E14D   >  8B4D BC       MOV ECX,DWORD PTR SS:[EBP-44]                    

0047E150   .  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]                    

0047E153   .  8B0CB9        MOV ECX,DWORD PTR DS:[ECX+EDI*4]  〈---- 那些值  

0047E156   .  8B04BA        MOV EAX,DWORD PTR DS:[EDX+EDI*4]                

0047E159   .  03C8          ADD ECX,EAX       〈---- 加起来  

0047E15B   .  66:8BC3       MOV AX,BX                                        

0047E15E   .  0F80 CF010000 JO 扩展名仓.0047E333                               

0047E164   .  66:05 0100    ADD AX,1       〈---- 递加 1 ，就是第 1 次是 1 ，第 2 次就是 2 咯，……  

0047E168   .  0F80 C5010000 JO 扩展名仓.0047E333                               

0047E16E   .  66:6BC0 07    IMUL AX,AX,7     〈---- 再乘于 7 ，第 1 次乘 1 ，第 2 次就是乘 2 咯，……  

0047E172   .  0F80 BB010000 JO 扩展名仓.0047E333                               

0047E178   .  0FBFD0        MOVSX EDX,AX                                     

0047E17B   .  03CA          ADD ECX,EDX       〈---- 再加起来  

0047E17D   .  0F80 B0010000 JO 扩展名仓.0047E333                               

0047E183   .  FF15 64114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI2I4>]          ;  msvbvm60.__vbaI2I4  

0047E189   .  8BF8          MOV EDI,EAX                                      

0047E18B   .  66:83FF 30    CMP DI,30     〈-----下面 4 个比较是判断是否在数字和大写字母的范围内，是的话就直接跳走了。  

0047E18F   .  7C 06         JL SHORT 扩展名仓.0047E197                          

0047E191   .  66:83FF 39    CMP DI,39                                      

0047E195   .  7E 60         JLE SHORT 扩展名仓.0047E1F7                         

0047E197   >  66:83FF 41    CMP DI,41                                      

0047E19B   .  7C 06         JL SHORT 扩展名仓.0047E1A3                          

0047E19D   .  66:83FF 5A    CMP DI,5A                                      

0047E1A1   .  7E 54         JLE SHORT 扩展名仓.0047E1F7                         

0047E1A3   >  66:8BC7       MOV AX,DI                                        

0047E1A6   .  66:B9 5B00    MOV CX,5B                                        

0047E1AA   .  66:99         CWD                                            

0047E1AC   .  66:F7F9       IDIV CX     〈----- 否则将 MOD 5B ，取余数  

0047E1AF   .  8BFA          MOV EDI,EDX                                      

0047E1B1   .  66:83FF 1A    CMP DI,1A                                      

0047E1B5   .  7D 06         JGE SHORT 扩展名仓.0047E1BD                         

0047E1B7   .  66:83C7 41    ADD DI,41                                      

0047E1BB   .  EB 34         JMP SHORT 扩展名仓.0047E1F1                        

0047E1BD   >  66:83FF 23    CMP DI,23                                      

0047E1C1   .  7F 06         JG SHORT 扩展名仓.0047E1C9                          

0047E1C3   .  66:83C7 16    ADD DI,16                                      

0047E1C7   .  EB 28         JMP SHORT 扩展名仓.0047E1F1                        

0047E1C9   >  66:83FF 28    CMP DI,28                                      

0047E1CD   .  7D 06         JGE SHORT 扩展名仓.0047E1D5                         

0047E1CF   .  66:83C7 0F    ADD DI,0F                                      

0047E1D3   .  EB 1C         JMP SHORT 扩展名仓.0047E1F1                        

0047E1D5   >  66:83FF 30    CMP DI,30                                      

0047E1D9   .  7D 06         JGE SHORT 扩展名仓.0047E1E1                         

0047E1DB   .  66:83C7 1E    ADD DI,1E                                      

0047E1DF   .  EB 10         JMP SHORT 扩展名仓.0047E1F1                        

0047E1E1   >  66:83FF 39    CMP DI,39                                      

0047E1E5   .  7E 10         JLE SHORT 扩展名仓.0047E1F7                         

0047E1E7   .  66:83FF 41    CMP DI,41                                      

0047E1EB   .  7D 0A         JGE SHORT 扩展名仓.0047E1F7                         

0047E1ED   .  66:83C7 11    ADD DI,11                                      

0047E1F1   >  0F80 3C010000 JO 扩展名仓.0047E333                               

0047E1F7   >  8B15 18514A00 MOV EDX,DWORD PTR DS:[4A5118]                   

0047E1FD   .  66:8BCB       MOV CX,BX                                        

0047E200   .  66:83C1 01    ADD CX,1                                        

0047E204   .  8995 44FFFFFF MOV DWORD PTR SS:[EBP-BC],EDX                  

0047E20A   .  0F80 23010000 JO 扩展名仓.0047E333                               

0047E210   .  8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]                  

0047E213   .  C745 94 01000>MOV DWORD PTR SS:[EBP-6C],1                    

0047E21A   .  0FBFD1        MOVSX EDX,CX                                     

0047E21D   .  50            PUSH EAX                                        ; /Arg4  
```
