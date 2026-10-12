
```
00408908  |.  83C5 3A       ADD EBP,3A  

0040890B  |.  55            PUSH EBP  

0040890C  |>  53            PUSH EBX  

0040890D  |.  E8 7E420000   CALL xxxxx.0040CB90                     ；这个 CALL 进行网络连接连接上返回一数值到 EAX 中  

00408912  |.  83C4 0C       ADD ESP,0C  

00408915  |.  83F8 FF       CMP EAX,-1                               ；只要不等于-1 即 FFFFFFFF 就发送数据  

00408918  |.  74 0F         JE SHORT xxxxx.00408929  

0040891A  |.  6A 00         PUSH 0                                   ; /Flags = 0  

0040891C  |.  56            PUSH ESI                                 ; |DataSize  

0040891D  |.  57            PUSH EDI                                 ; |Data  

0040891E  |.  53            PUSH EBX                                 ; |Socket  

0040891F  |.  FF15 A4954300 CALL DWORD PTR DS:[<&WS2_32.#19>]       ; \send  

00408925  |.  3BC6          CMP EAX,ESI  

00408927  |.  74 1D         JE SHORT xxxxx.00408946  

00408929  |>  57            PUSH EDI  

0040892A  |.  E8 B50C0200   CALL xxxxx.004295E4  

0040892F  |.  83C4 04       ADD ESP,4  

00408932  |.  53            PUSH EBX                                 ; /Socket  

00408933  |.  FF15 A0954300 CALL DWORD PTR DS:[<&WS2_32.#3>]        ; \closesocket  

00408939  |.  5F            POP EDI  

0040893A  |.  5E            POP ESI  

0040893B  |.  5D            POP EBP  

0040893C  |.  B8 F9FFFFFF   MOV EAX,-7  

00408941  |.  5B            POP EBX  

00408942  |.  59            POP ECX  

00408943  |.  C2 1400       RETN 14  

00408946  |>  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+24]  

0040894A  |.  85C0          TEST EAX,EAX  

0040894C  |.  75 1A         JNZ SHORT xxxxx.00408968  

0040894E  |.  57            PUSH EDI  

0040894F  |.  E8 900C0200   CALL xxxxx.004295E4  

00408954  |.  83C4 04       ADD ESP,4  

00408957  |.  53            PUSH EBX                                 ; /Socket  

00408958  |.  FF15 A0954300 CALL DWORD PTR DS:[<&WS2_32.#3>]        ; \closesocket  

0040895E  |.  5F            POP EDI  

0040895F  |.  5E            POP ESI  

00408960  |.  5D            POP EBP  

00408961  |.  33C0          XOR EAX,EAX  

00408963  |.  5B            POP EBX  

00408964  |.  59            POP ECX  

00408965  |.  C2 1400       RETN 14  

00408946  |>  \8B4424 24    MOV EAX,DWORD PTR SS:[ESP+24]  

0040894A  |.  85C0          TEST EAX,EAX  

0040894C  |.  75 1A         JNZ SHORT xxxxx.00408968  

0040894E  |.  57            PUSH EDI  

0040894F  |.  E8 900C0200   CALL xxxxx.004295E4  

00408954  |.  83C4 04       ADD ESP,4  

00408957  |.  53            PUSH EBX                                 ; /Socket  

00408958  |.  FF15 A0954300 CALL DWORD PTR DS:[<&WS2_32.#3>]        ; \closesocket  

0040895E  |.  5F            POP EDI  

0040895F  |.  5E            POP ESI  

00408960  |.  5D            POP EBP  

00408961  |.  33C0          XOR EAX,EAX  

00408963  |.  5B            POP EBX  

00408964  |.  59            POP ECX  

00408965  |.  C2 1400       RETN 14  

00408968  |>  8D48 03       LEA ECX,DWORD PTR DS:[EAX+3]  

0040896B  |.  B8 56555555   MOV EAX,55555556  

00408970  |.  F7E9          IMUL ECX  

00408972  |.  8BF2          MOV ESI,EDX  

00408974  |.  C1EE 1F       SHR ESI,1F  

00408977  |.  03F2          ADD ESI,EDX  

00408979  |.  C1E6 02       SHL ESI,2  

0040897C  |.  56            PUSH ESI  

0040897D  |.  E8 670C0200   CALL xxxxx.004295E9  

00408982  |.  8BE8          MOV EBP,EAX  

00408984  |.  8BCE          MOV ECX,ESI  

00408986  |.  8BD1          MOV EDX,ECX  

00408988  |.  C1E9 02       SHR ECX,2  

0040898B  |.  33C0          XOR EAX,EAX  

0040898D  |.  8BFD          MOV EDI,EBP  

0040898F  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]  

00408991  |.  8BCA          MOV ECX,EDX  

00408993  |.  83E1 03       AND ECX,3  

00408996  |.  F3:AA         REP STOS BYTE PTR ES:[EDI]  

00408998  |.  8B4424 14     MOV EAX,DWORD PTR SS:[ESP+14]  

0040899C  |.  8B48 14       MOV ECX,DWORD PTR DS:[EAX+14]  

0040899F  |.  51            PUSH ECX  

004089A0  |.  56            PUSH ESI  

004089A1  |.  55            PUSH EBP  

004089A2  |.  53            PUSH EBX  

004089A3  |.  E8 68410000   CALL xxxxx.0040CB10  

004089A8  |.  83C4 14       ADD ESP,14  

004089AB  |.  3BC6          CMP EAX,ESI  

004089AD  |.  74 27         JE SHORT xxxxx.004089D6  

004089AF  |.  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+1C]  

004089B3  |.  52            PUSH EDX  

004089B4  |.  E8 2B0C0200   CALL xxxxx.004295E4  

004089B9  |.  55            PUSH EBP  

004089BA  |.  E8 250C0200   CALL xxxxx.004295E4  

004089BF  |.  83C4 08       ADD ESP,8  

004089C2  |.  53            PUSH EBX                                 ; /Socket  

004089C3  |.  FF15 A0954300 CALL DWORD PTR DS:[<&WS2_32.#3>]        ; \closesocket  

004089C9  |.  5F            POP EDI  

004089CA  |.  5E            POP ESI  

004089CB  |.  5D            POP EBP  

004089CC  |.  B8 F9FFFFFF   MOV EAX,-7  

004089D1  |.  5B            POP EBX  

004089D2  |.  59            POP ECX  

004089D3  |.  C2 1400       RETN 14  

004089D6  |>  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+24]  

004089DA  |.  8B4C24 20     MOV ECX,DWORD PTR SS:[ESP+20]  

004089DE  |.  50            PUSH EAX  

004089DF  |.  56            PUSH ESI  

004089E0  |.  51            PUSH ECX  

004089E1  |.  55            PUSH EBP  

004089E2  |.  E8 49120000   CALL xxxxx.00409C30                     ；这里跟进是进行返回数据初解密的  

004089E7  |.  83C4 10       ADD ESP,10  

004089EA  |.  53            PUSH EBX                                 ; /Socket  

004089EB  |.  FF15 A0954300 CALL DWORD PTR DS:[<&WS2_32.#3>]        ; \closesocket  

004089F1  |.  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+1C]  

004089F5  |.  52            PUSH EDX  

004089F6  |.  E8 E90B0200   CALL xxxxx.004295E4  

004089FB  |.  55            PUSH EBP  

004089FC  |.  E8 E30B0200   CALL xxxxx.004295E4  

00408A01  |.  83C4 08       ADD ESP,8  

00408A04  |.  5F            POP EDI  

00408A05  |.  5E            POP ESI  

00408A06  |.  5D            POP EBP  

00408A07  |.  33C0          XOR EAX,EAX  

00408A09  |.  5B            POP EBX  

00408A0A  |.  59            POP ECX  

00408A0B  \.  C2 1400       RETN 14  
```

`4089E2` 根进 `CALL` 后的数据

```
00409C30  /$  8B5424 0C     MOV EDX,DWORD PTR SS:[ESP+C]  

00409C34  |.  57            PUSH EDI  

00409C35  |.  8BCA          MOV ECX,EDX  

00409C37  |.  33FF          XOR EDI,EDI  

00409C39  |.  33C0          XOR EAX,EAX  

00409C3B  |.  81E1 03000080 AND ECX,80000003  

00409C41  |.  79 05         JNS SHORT xxxxx.00409C48  

00409C43  |.  49            DEC ECX  

00409C44  |.  83C9 FC       OR ECX,FFFFFFFC  

00409C47  |.  41            INC ECX  

00409C48  |>  83F9 01       CMP ECX,1  

00409C4B  |.  75 05         JNZ SHORT xxxxx.00409C52               ；这里必须跳去解密字符串  

00409C4D  |.  83C8 FF       OR EAX,FFFFFFFF  

00409C50  |.  5F            POP EDI  

00409C51  |.  C3            RETN  

00409C52  |>  85D2          TEST EDX,EDX  

00409C54  |.  56            PUSH ESI  

00409C55  |.  8B7424 0C     MOV ESI,DWORD PTR SS:[ESP+C]  

00409C59  |.  7E 0E         JLE SHORT xxxxx.00409C69  

00409C5B  |.  EB 03         JMP SHORT xxxxx.00409C60  

00409C5D  |   8D49 00       LEA ECX,DWORD PTR DS:[ECX]  

00409C60  |>  800430 C4     /ADD BYTE PTR DS:[EAX+ESI],0C4  

00409C64  |.  40            |INC EAX  

00409C65  |.  3BC2          |CMP EAX,EDX  

00409C67  |.^ 7C F7         \JL SHORT xxxxx.00409C60  

00409C69  |>  33C9          XOR ECX,ECX  

00409C6B  |.  85D2          TEST EDX,EDX  

00409C6D  |.  7E 72         JLE SHORT xxxxx.00409CE1  

00409C6F  |.  55            PUSH EBP  

00409C70  |.  8B6C24 14     MOV EBP,DWORD PTR SS:[ESP+14]  

00409C74  |.  53            PUSH EBX  

00409C75  |>  3B7C24 20     /CMP EDI,DWORD PTR SS:[ESP+20]  

00409C79  |.  7D 64         |JGE SHORT xxxxx.00409CDF  

00409C7B  |.  8BC1          |MOV EAX,ECX  

00409C7D  |.  25 03000080   |AND EAX,80000003  
```
