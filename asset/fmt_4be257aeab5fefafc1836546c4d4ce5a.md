
```
  004057A9   >   8A40 01       MOV AL,BYTE PTR DS:[EAX+1]            ;  用户名第二个字符 a 送入 al  

  004057AC   .   884424 0E     MOV BYTE PTR SS:[ESP+E],AL  

  004057B0   .   8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]         ;  用户名送入 eax  

  004057B3   .   8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]  

  004057B6   .   85C9          TEST ECX,ECX  

  004057B8   .   7D 0A         JGE SHORT AVISplit.004057C4  

  004057BA   .   68 57000780   PUSH 80070057  

  004057BF   .   E8 5CC1FFFF   CALL AVISplit.00401920  

  004057C4   >   8B4E 70       MOV ECX,DWORD PTR DS:[ESI+70]  

  004057C7   .   53            PUSH EBX  

  004057C8   .   8A18          MOV BL,BYTE PTR DS:[EAX]  

  004057CA   .   3979 F4       CMP DWORD PTR DS:[ECX-C],EDI  

  004057CD   .   7D 0A         JGE SHORT AVISplit.004057D9  

  004057CF   .   68 57000780   PUSH 80070057  

  004057D4   .   E8 47C1FFFF   CALL AVISplit.00401920  

  004057D9   >   0FB6C2        MOVZX EAX,DL                          ;  把 d 送入 eax 然后高位都用零填充  

  004057DC   .   83C8 53       OR EAX,53                             ;  64 和 53or=77  

  004057DF   .   99            CDQ  

  004057E0   .   BF 0A000000   MOV EDI,0A                            ;  看第一个经验总结  

  004057E5   .   F7FF          IDIV EDI  

  004057E7   .   0FB64424 12   MOVZX EAX,BYTE PTR SS:[ESP+12]        ;  把 a 送入 eax  

  004057EC   .   83C8 41       OR EAX,41                             ;  61 和 41 或  

  004057EF   .   885424 16     MOV BYTE PTR SS:[ESP+16],DL  

  004057F3   .   99            CDQ  

  004057F4   .   F7FF          IDIV EDI  

  004057F6   .   0FB6C3        MOVZX EAX,BL                          ;  把 d 送入 eax  

  004057F9   .   83C8 56       OR EAX,56                             ;  64 和 56 或  

  004057FC   .   885424 12     MOV BYTE PTR SS:[ESP+12],DL           ;  dl 是 07  

  00405800   .   99            CDQ  

  00405801   .   F7FF          IDIV EDI                              ;  再次变为零  

  00405803   .   0FB641 01     MOVZX EAX,BYTE PTR DS:[ECX+1]         ;  把 a 送入 eax  

  00405807   .   83C8 49       OR EAX,49                             ;  a 和 49 或  

  0040580A   .   8BCF          MOV ECX,EDI  

  0040580C   .   885424 17     MOV BYTE PTR SS:[ESP+17],DL  

  00405810   .   99            CDQ  

  00405811   .   F7F9          IDIV ECX  

  00405813   .   33C0          XOR EAX,EAX                           ;  eax 清零  

  00405815   .   33C9          XOR ECX,ECX                           ;  ecx 清零  

  00405817   .   85ED          TEST EBP,EBP  

  00405819   .   885424 18     MOV BYTE PTR SS:[ESP+18],DL  

  0040581D   .   7E 20         JLE SHORT AVISplit.0040583F  

  0040581F   .   90            NOP  

  00405820   >   85C9          TEST ECX,ECX  

  00405822   .   0F8C D2000000 JL AVISplit.004058FA  

  00405828   .   8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]         ;  把用户名送入 edi  

  0040582B   .   3B4F F4       CMP ECX,DWORD PTR DS:[EDI-C]  

  0040582E   .   0F8F C6000000 JG AVISplit.004058FA  

  00405834   .   0FB6140F      MOVZX EDX,BYTE PTR DS:[EDI+ECX]       ;  把 d 送入 edx，第二次把 a 送进去  

  00405838   .   03C2          ADD EAX,EDX  

  0040583A   .   41            INC ECX                               ;  ecx 自加 1  

  0040583B   .   3BCD          CMP ECX,EBP                           ;  ecx 和 ebp 相比 EBP 为 6，而 ecx 为 1，当密码长度为 6 的时候跳转  

  0040583D   .^  7C E1         JL SHORT AVISplit.00405820  

  0040583F   >   8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]         ;  把密码送入 ecx  

  00405842   .   8B51 F4       MOV EDX,DWORD PTR DS:[ECX-C]  

  00405845   .   85D2          TEST EDX,EDX  

  00405847   .   7D 0A         JGE SHORT AVISplit.00405853  

  00405849   .   68 57000780   PUSH 80070057  

  0040584E   .   E8 CDC0FFFF   CALL AVISplit.00401920  

  00405853   >   8A11          MOV DL,BYTE PTR DS:[ECX]              ;  1 送入 dl  

  00405855   .   8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]  

  00405858   .   8379 F4 01    CMP DWORD PTR DS:[ECX-C],1  

  0040585C   .   885424 19     MOV BYTE PTR SS:[ESP+19],DL           ;  把 1 送入栈  

  00405860   .   7D 0A         JGE SHORT AVISplit.0040586C  

  00405862   .   68 57000780   PUSH 80070057  

  00405867   .   E8 B4C0FFFF   CALL AVISplit.00401920  

  0040586C   >   8A49 01       MOV CL,BYTE PTR DS:[ECX+1]            ;  把 2 送入 cl  

  0040586F   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把密码再次送入 edi  

  00405872   .   884C24 13     MOV BYTE PTR SS:[ESP+13],CL  

  00405876   .   837F F4 02    CMP DWORD PTR DS:[EDI-C],2  

  0040587A   .   7D 0A         JGE SHORT AVISplit.00405886  

  0040587C   .   68 57000780   PUSH 80070057  

  00405881   .   E8 9AC0FFFF   CALL AVISplit.00401920  

  00405886   >   8A4F 02       MOV CL,BYTE PTR DS:[EDI+2]            ;  把 3 送入 cl  

  00405889   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把密码送入 edi  

  0040588C   .   884C24 14     MOV BYTE PTR SS:[ESP+14],CL  

  00405890   .   837F F4 03    CMP DWORD PTR DS:[EDI-C],3  

  00405894   .   7D 0A         JGE SHORT AVISplit.004058A0  

  00405896   .   68 57000780   PUSH 80070057  

  0040589B   .   E8 80C0FFFF   CALL AVISplit.00401920  

  004058A0   >   8A4F 03       MOV CL,BYTE PTR DS:[EDI+3]            ;  把 4 送入 cl  

  004058A3   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把密码 123456789 送入 edi  

  004058A6   .   884C24 15     MOV BYTE PTR SS:[ESP+15],CL  

  004058AA   .   837F F4 04    CMP DWORD PTR DS:[EDI-C],4  

  004058AE   .   7D 0A         JGE SHORT AVISplit.004058BA  

  004058B0   .   68 57000780   PUSH 80070057  

  004058B5   .   E8 66C0FFFF   CALL AVISplit.00401920  

  004058BA   >   8A4F 04       MOV CL,BYTE PTR DS:[EDI+4]  

  004058BD   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把假密码送入 edi  

  004058C0   .   837F F4 05    CMP DWORD PTR DS:[EDI-C],5  

  004058C4   .   7D 0A         JGE SHORT AVISplit.004058D0  

  004058C6   .   68 57000780   PUSH 80070057  

  004058CB   .   E8 50C0FFFF   CALL AVISplit.00401920  

  004058D0   >   8A5F 05       MOV BL,BYTE PTR DS:[EDI+5]  

  004058D3   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把假密码送入 edi  

  004058D6   .   885C24 1A     MOV BYTE PTR SS:[ESP+1A],BL  

  004058DA   .   837F F4 06    CMP DWORD PTR DS:[EDI-C],6  

  004058DE   .   7D 0A         JGE SHORT AVISplit.004058EA  

  004058E0   .   68 57000780   PUSH 80070057  

  004058E5   .   E8 36C0FFFF   CALL AVISplit.00401920  

  004058EA   >   8A5F 06       MOV BL,BYTE PTR DS:[EDI+6]            ;  把 7 送入 bl  

  004058ED   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]         ;  把假注册码送入 edi  

  004058F0   .   885C24 1B     MOV BYTE PTR SS:[ESP+1B],BL  

  004058F4   .   837F F4 07    CMP DWORD PTR DS:[EDI-C],7  

  004058F8   .   7D 0A         JGE SHORT AVISplit.00405904  

  004058FA   >   68 57000780   PUSH 80070057  

  004058FF   .   E8 1CC0FFFF   CALL AVISplit.00401920  

  00405904   >   8A5F 07       MOV BL,BYTE PTR DS:[EDI+7]            ;  把 8 送入 bl  

  00405907   .   0FB67C24 16   MOVZX EDI,BYTE PTR SS:[ESP+16]  

  0040590C   .   0FB6D2        MOVZX EDX,DL  

  0040590F   .   83EA 30       SUB EDX,30  

  00405912   .   3BFA          CMP EDI,EDX  

  00405914   .   75 48         JNZ SHORT AVISplit.0040595E  

  00405916   .   0FB65424 13   MOVZX EDX,BYTE PTR SS:[ESP+13]  

  0040591B   .   0FB67C24 12   MOVZX EDI,BYTE PTR SS:[ESP+12]  

  00405920   .   83EA 30       SUB EDX,30  

  00405923   .   3BFA          CMP EDI,EDX  

  00405925   .   75 37         JNZ SHORT AVISplit.0040595E  

  00405927   .   0FB65424 14   MOVZX EDX,BYTE PTR SS:[ESP+14]  

  0040592C   .   0FB67C24 17   MOVZX EDI,BYTE PTR SS:[ESP+17]  

  00405931   .   83EA 30       SUB EDX,30  

  00405934   .   3BFA          CMP EDI,EDX  

  00405936   .   75 26         JNZ SHORT AVISplit.0040595E  

  00405938   .   0FB65424 15   MOVZX EDX,BYTE PTR SS:[ESP+15]  

  0040593D   .   0FB67C24 18   MOVZX EDI,BYTE PTR SS:[ESP+18]  

  00405942   .   83EA 30       SUB EDX,30  

  00405945   .   3BFA          CMP EDI,EDX  
```
