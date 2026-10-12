
```
0040C2C9   |.   66:8945 FE     MOV WORD PTR SS:[EBP-2],AX  

0040C2CD   |.   C645 FD 00     MOV BYTE PTR SS:[EBP-3],0                ;  ESP-3=0  

0040C2D1   |.   66:8B45 FE     MOV AX,WORD PTR SS:[EBP-2]               ;  AX=EBP-2=软件序列号的前四位  

0040C2D5   |.   E8 A6FFFFFF    CALL UN_THINK.     0040C280  

0040C2DA   |.   83E0 7F        AND EAX,7F  

0040C2DD   |.   8D0440         LEA EAX,DWORD PTR DS:[EAX+EAX*2]  

0040C2E0   |.   8D34C5 046160>LEA ESI,DWORD PTR DS:[EAX*8+606104]      ;  ESI 指向的是一张表  

0040C2E7   |.   66:837D FE 01  CMP WORD PTR SS:[EBP-2],1               ;  序列号的前四位大于 1?  

0040C2EC   |.   0F82 86000000  JB UN_THINK.     0040C378                ;  是则继续  

0040C2F2   |.   66:817D FE 0F>CMP WORD PTR SS:[EBP-2],270F             ;  序列号的前四位小于 270FH(9999)?  

0040C2F8   |.   77 7E          JA SHORT UN_THINK.0040C378               ;  不是则继续  

0040C2FA   |.   66:83FF 01     CMP DI,1                                 ;  第 5,6 位大于等于 1?  

0040C2FE   |.   72 78          JB SHORT UN_THINK.0040C378               ;  是则继续  

0040C300   |.   66:83FF 0C     CMP DI,0C                                ;  第 5,6 位小于等于 12?  

0040C304   |.   77 72          JA SHORT UN_THINK.0040C378               ;  不是则继续  

0040C306   |.   66:83FB 01     CMP BX,1                                 ;  第 7,8 位大于等于 1?  

0040C30A   |.   72 6C          JB SHORT UN_THINK.0040C378               ;  是则继续  

0040C30C   |.   0FB7C7         MOVZX EAX,DI                             ;  EAX=第 5、6 位  

0040C30F   |.   66:3B5C46 FE   CMP BX,WORD PTR DS:[ESI+EAX*2-2]         ;  注意这里很有意思哦!第 7,8 位与 ESI 处的表向比较,如果第 5、6 位为 1 则比较第 7,8 位是否小于 1FH(31),如果……看出来了吗?其实第 5,6 位代表月份,第 7,8 位代表日期!  

0040C314   |.   77 62          JA SHORT UN_THINK.0040C378               ;  不是则继续  

0040C316   |.   0FB7C7         MOVZX EAX,DI  

0040C319   |.   48             DEC EAX  

0040C31A   |.   85C0           TEST EAX,EAX  

0040C31C   |.   7E 0E          JLE SHORT UN_THINK.0040C32C  

0040C31E   |.   B9 01000000    MOV ECX,1  

0040C323   |>   66:035C4E FE   /ADD BX,WORD PTR DS:[ESI+ECX*2-2]  

…………  

0040C371   |.   DD18           FSTP QWORD PTR DS:[EAX]  

0040C373   |.   9B             WAIT  

0040C374   |.   C645 FD 01     MOV BYTE PTR SS:[EBP-3],1  

0040C378   |>   8A45 FD        MOV AL,BYTE PTR SS:[EBP-3]               ;  不能到这里!(ESP-3=0)  

0040C37B   |.   5F             POP EDI         

0040C37C   |.   5E             POP ESI  

0040C37D   |.   5B             POP EBX  

0040C37E   |.   59             POP ECX  

0040C37F   |.   59             POP ECX  

0040C380   |.   5D             POP EBP  

0040C381   .   C2 0400        RETN 4  

                                                      <<<<<<<<<<<<<<<<<<<<<<<  

005A5FCE   .   DDD8           FSTP ST  

005A5FD0   .   B3 01          MOV BL,1                                 ;  如果能正常走过上面那个 Call,BL 就置为 1!  

005A5FD2   .   33C0           XOR EAX,EAX  

005A5FD4   .   5A             POP EDX  

005A5FD5   .   59             POP ECX  

005A5FD6   .   59             POP ECX  

005A5FD7   .   64:8910        MOV DWORD PTR FS:[EAX],EDX  

…………  

005A600A   .   8BC3           MOV EAX,EBX  

005A600C   .   5F             POP EDI  

005A600D   .   5E             POP ESI  

005A600E   .   5B             POP EBX  

005A600F   .   8BE5           MOV ESP,EBP  

005A6011   .   5D             POP EBP  

005A6012   .   C3             RETN  
```

▲小结：这段代码就是为了检测，软件序列号前 8 位是否为日期格式。第 1-4 位代表年份，5-6 位代表月份，8-9 位代表日期，并且要注意到大月、小月、闰月的问题。如果嫌麻烦的话，干脆就让第 8-9 位总小于等于 28 就行了；-)

☆☆☆☆☆☆☆☆☆`Call_3`☆☆☆☆☆☆☆☆☆

```
005A6024   $  55             PUSH EBP                              ; F7 跟入我们在这里  
005A6025   .  8BEC           MOV EBP,ESP  
…………  
005A6047   .  64:8920        MOV DWORD PTR FS:[EAX],ESP  
005A604A   .  8D4D FC        LEA ECX,DWORD PTR SS:[EBP-4]  
005A604D   .  33D2           XOR EDX,EDX  
005A604F   .  B8 D4605A00    MOV EAX,UN_THINK.005A60D4             ;  ASCII "Softkey"  
005A6054   .  E8 87E2FFFF    CALL UN_THINK.005A42E0                 
```

`4dd466d0e1e0cc123afbe504da2462ca`
