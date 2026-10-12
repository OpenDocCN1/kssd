
```
004369FA  |.  D1E0          SHL EAX,1   

004369FC  |.  2BC1          SUB EAX,ECX   

004369FE  |.  80C2 30       ADD DL,30   

00436A01  |.  885424 14     MOV BYTE PTR SS:[ESP+14],DL   

00436A05  |.  8B4C24 11     MOV ECX,DWORD PTR SS:[ESP+11]   

00436A09  |.  81E1 FF000000 AND ECX,0FF   

00436A0F  |.  8D14CD 000000>LEA EDX,DWORD PTR DS:[ECX*8]   

00436A16  |.  2BD1          SUB EDX,ECX   

00436A18  |.  8D1492        LEA EDX,DWORD PTR DS:[EDX+EDX*4]   

00436A1B  |.  2BC2          SUB EAX,EDX   

00436A1D  |.  99            CDQ   

00436A1E  |.  8BC8          MOV ECX,EAX   

00436A20  |.  33CA          XOR ECX,EDX   

00436A22  |.  2BCA          SUB ECX,EDX   

00436A24  |.  8D0449        LEA EAX,DWORD PTR DS:[ECX+ECX*2]   

00436A27  |.  8D04C0        LEA EAX,DWORD PTR DS:[EAX+EAX*8]   

00436A2A  |.  D1E0          SHL EAX,1   

00436A2C  |.  2BC1          SUB EAX,ECX   

00436A2E  |.  E9 F5000000   JMP UNIRFANV.00436B28   

00436A33  |>  8A4424 15     MOV AL,BYTE PTR SS:[ESP+15]   

00436A37  |.  8A5424 16     MOV DL,BYTE PTR SS:[ESP+16]   

00436A3B  |.  8A4C24 14     MOV CL,BYTE PTR SS:[ESP+14]   

00436A3F  |.  884424 16     MOV BYTE PTR SS:[ESP+16],AL   

00436A43  |.  8A4424 11     MOV AL,BYTE PTR SS:[ESP+11]   

00436A47  |.  885424 18     MOV BYTE PTR SS:[ESP+18],DL   

00436A4B  |.  8A5424 12     MOV DL,BYTE PTR SS:[ESP+12]   

00436A4F  |.  884424 12     MOV BYTE PTR SS:[ESP+12],AL   

00436A53  |.  8B4424 16     MOV EAX,DWORD PTR SS:[ESP+16]   

00436A57  |.  884C24 15     MOV BYTE PTR SS:[ESP+15],CL   

00436A5B  |.  25 FF000000   AND EAX,0FF   

00436A60  |.  885424 13     MOV BYTE PTR SS:[ESP+13],DL   

00436A64  |.  8BC8          MOV ECX,EAX   

00436A66  |.  C1E1 06       SHL ECX,6   

00436A69  |.  2BC8          SUB ECX,EAX   

00436A6B  |.  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+18]   

00436A6F  |.  25 FF000000   AND EAX,0FF   

00436A74  |.  8D04C0        LEA EAX,DWORD PTR DS:[EAX+EAX*8]   

00436A77  |.  C1E0 02       SHL EAX,2   

00436A7A  |.  2BC1          SUB EAX,ECX   

00436A7C  |.  B9 09000000   MOV ECX,9   

00436A81  |.  99            CDQ   

00436A82  |.  33C2          XOR EAX,EDX   

00436A84  |.  2BC2          SUB EAX,EDX   

00436A86  |.  8D04C0        LEA EAX,DWORD PTR DS:[EAX+EAX*8]   

00436A89  |.  C1E0 02       SHL EAX,2   

00436A8C  |.  99            CDQ   

00436A8D  |.  F7F9          IDIV ECX   

00436A8F  |.  80C2 30       ADD DL,30   

00436A92  |.  885424 17     MOV BYTE PTR SS:[ESP+17],DL   

00436A96  |.  8B4424 14     MOV EAX,DWORD PTR SS:[ESP+14]   

00436A9A  |.  25 FF000000   AND EAX,0FF   

00436A9F  |.  83C0 20       ADD EAX,20   

00436AA2  |.  8D14C5 000000>LEA EDX,DWORD PTR DS:[EAX*8]   

00436AA9  |.  2BD0          SUB EDX,EAX   

00436AAB  |.  8D0490        LEA EAX,DWORD PTR DS:[EAX+EDX*4]   

00436AAE  |.  8D0C40        LEA ECX,DWORD PTR DS:[EAX+EAX*2]   

00436AB1  |.  8B4424 13     MOV EAX,DWORD PTR SS:[ESP+13]   

00436AB5  |.  25 FF000000   AND EAX,0FF   

00436ABA  |.  8D1480        LEA EDX,DWORD PTR DS:[EAX+EAX*4]   

00436ABD  |.  C1E2 03       SHL EDX,3   

00436AC0  |.  2BD0          SUB EDX,EAX   

00436AC2  |.  8D0451        LEA EAX,DWORD PTR DS:[ECX+EDX*2]   

00436AC5  |.  99            CDQ   

00436AC6  |.  33C2          XOR EAX,EDX   

00436AC8  |.  2BC2          SUB EAX,EDX   

00436ACA  |.  8D0CC5 000000>LEA ECX,DWORD PTR DS:[EAX*8]   

00436AD1  |.  2BC8          SUB ECX,EAX   

00436AD3  |.  8D0488        LEA EAX,DWORD PTR DS:[EAX+ECX*4]   

00436AD6  |.  B9 09000000   MOV ECX,9   

00436ADB  |.  8D0440        LEA EAX,DWORD PTR DS:[EAX+EAX*2]   

00436ADE  |.  99            CDQ   

00436ADF  |.  F7F9          IDIV ECX   

00436AE1  |.  8B4424 10     MOV EAX,DWORD PTR SS:[ESP+10]   

00436AE5  |.  25 FF000000   AND EAX,0FF   

00436AEA  |.  80C2 30       ADD DL,30   

00436AED  |.  885424 14     MOV BYTE PTR SS:[ESP+14],DL   

00436AF1  |.  8D14C5 000000>LEA EDX,DWORD PTR DS:[EAX*8]   

00436AF8  |.  2BD0          SUB EDX,EAX   

00436AFA  |.  8D0490        LEA EAX,DWORD PTR DS:[EAX+EDX*4]   

00436AFD  |.  8B5424 11     MOV EDX,DWORD PTR SS:[ESP+11]   

00436B01  |.  81E2 FF000000 AND EDX,0FF   

00436B07  |.  8BCA          MOV ECX,EDX   

00436B09  |.  C1E1 04       SHL ECX,4   

00436B0C  |.  03CA          ADD ECX,EDX   

00436B0E  |.  D1E0          SHL EAX,1   

00436B10  |.  8D0C89        LEA ECX,DWORD PTR DS:[ECX+ECX*4]   

00436B13  |.  2BC1          SUB EAX,ECX   

00436B15  |.  99            CDQ   

00436B16  |.  33C2          XOR EAX,EDX   

00436B18  |.  2BC2          SUB EAX,EDX   

00436B1A  |.  8D14C5 000000>LEA EDX,DWORD PTR DS:[EAX*8]   

00436B21  |.  2BD0          SUB EDX,EAX   

00436B23  |.  8D0490        LEA EAX,DWORD PTR DS:[EAX+EDX*4]   

00436B26  |.  D1E0          SHL EAX,1   

00436B28  |>  99            CDQ   

00436B29  |.  B9 09000000   MOV ECX,9   

00436B2E  |.  C64424 19 00  MOV BYTE PTR SS:[ESP+19],0   

00436B33  |.  F7F9          IDIV ECX   

00436B35  |.  80C2 30       ADD DL,30   

00436B38  |.  885424 11     MOV BYTE PTR SS:[ESP+11],DL   

00436B3C  |.  8D5424 10     LEA EDX,DWORD PTR SS:[ESP+10]   

00436B40  |.  52            PUSH EDX   

00436B41  |.  E8 B7FC0700   CALL UNIRFANV.004B67FD   

00436B46  |.  83C4 04       ADD ESP,4   

00436B49  |.  33C9          XOR ECX,ECX   

00436B4B  |.  3BE8          CMP EBP,EAX------------------------------->真假注册码比较(16 进制)   

00436B4D  |.  5F            POP EDI   

00436B4E  |.  5E            POP ESI   

00436B4F  |.  0F94C1        SETE CL   

00436B52  |.  5D            POP EBP   

00436B53  |.  8BC1          MOV EAX,ECX   

00436B55  |.  5B            POP EBX   

00436B56  |.  83C4 14       ADD ESP,14   

00436B59  .  C3            RETN
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

### `004C1986`

```
004C1986  /$  55            PUSH EBP\----------------------------------->注册码的 16 进制数   

004C1987  |.  8BEC          MOV EBP,ESP   

004C1989  |.  33C0          XOR EAX,EAX   

004C198B  |.  837D 10 0A    CMP DWORD PTR SS:[EBP+10],0A   

004C198F  |.  75 08         JNZ SHORT UNIRFANV.004C1999------------>NO   

004C1991  |.  3945 08       CMP DWORD PTR SS:[EBP+8],EAX   

004C1994  |.  7D 03         JGE SHORT UNIRFANV.004C1999------------>JMP   

004C1996  |.  6A 01         PUSH 1   

004C1998  |.  58            POP EAX   

004C1999  |>  50            PUSH EAX                     ; /Arg4-------->00000000   

004C199A  |.  FF75 10       PUSH DWORD PTR SS:[EBP+10]    ; |Arg3-------->0000000A(转换后的进制)   

004C199D  |.  FF75 0C       PUSH DWORD PTR SS:[EBP+C]     ; |Arg2-------->0076EB6C(转换后存放的地址)   

004C19A0  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]     ; |Arg1-------->00039280(要转换的 16 进制数)   

004C19A3  |.  E8 82FFFFFF   CALL UNIRFANV.004C192A         ; UNIRFANV.004C192A--------------->产生数字串   

004C19A8  |.  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]   

004C19AB  |.  83C4 10       ADD ESP,10   

004C19AE  |.  5D            POP EBP   

004C19AF  .  C3            RETN
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

### `004C192A`----------> 将 `39280(H)` 转化为 10 进制存放

```
004C192A  /$  55            PUSH EBP   

004C192B  |.  8BEC          MOV EBP,ESP   

004C192D  |.  837D 14 00    CMP DWORD PTR SS:[EBP+14],0   

004C1931  |.  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]   

004C1934  |.  53            PUSH EBX
```
