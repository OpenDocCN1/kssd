
```
0040424A   . 53             PUSH EBX  

0040424B   . 56             PUSH ESI  

0040424C   . 57             PUSH EDI  

0040424D   . 89C3           MOV EBX,EAX  

0040424F   . 89D6           MOV ESI,EDX  

00404251   . 8B79 FC        MOV EDI,DWORD PTR DS:[ECX-4]  

00404254   . 8B56 FC        MOV EDX,DWORD PTR DS:[ESI-4]  

00404257   . 01FA           ADD EDX,EDI  

00404259   . 39CE           CMP ESI,ECX  

0040425B   . 74 17          JE SHORT MoneyBos.00404274  

0040425D   . E8 06030000    CALL MoneyBos.00404568  

00404262   . 89F0           MOV EAX,ESI  

00404264   . 8B4E FC        MOV ECX,DWORD PTR DS:[ESI-4]  

00404267   > 8B13           MOV EDX,DWORD PTR DS:[EBX]  

00404269   . 01FA           ADD EDX,EDI  

0040426B   . E8 38E8FFFF    CALL MoneyBos.00402AA8=================>在这里产生的，请看下面的 CALL  

00404270   . 5F             POP EDI  

\-----------------------------------------------------

\-------------------------------------  

00402AA8  /$ 56             PUSH ESI  

00402AA9  |. 57             PUSH EDI  

00402AAA  |. 89C6           MOV ESI,EAX  

00402AAC  |. 89D7           MOV EDI,EDX  

00402AAE  |. 89C8           MOV EAX,ECX  

00402AB0  |. 39F7           CMP EDI,ESI  

00402AB2  |. 77 13          JA SHORT MoneyBos.00402AC7  

00402AB4  |. 74 2F          JE SHORT MoneyBos.00402AE5  

00402AB6  |. C1F9 02        SAR ECX,2  

00402AB9  |. 78 2A          JS SHORT MoneyBos.00402AE5  

00402ABB  |. F3:A5          REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  

00402ABD  |. 89C1           MOV ECX,EAX  

00402ABF  |. 83E1 03        AND ECX,3  

00402AC2  |. F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>  

00402AC4  |. 5F             POP EDI  

00402AC5  |. 5E             POP ESI  

00402AC6  |. C3             RETN  

00402AC7  |> 8D740E FC      LEA ESI,DWORD PTR DS:[ESI+ECX-4]  

00402ACB  |. 8D7C0F FC      LEA EDI,DWORD PTR DS:[EDI+ECX-4]  

00402ACF  |. C1F9 02        SAR ECX,2  

00402AD2  |. 78 11          JS SHORT MoneyBos.00402AE5  

00402AD4  |. FD             STD  

00402AD5  |. F3:A5          REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>  

00402AD7  |. 89C1           MOV ECX,EAX  

00402AD9  |. 83E1 03        AND ECX,3  

00402ADC  |. 83C6 03        ADD ESI,3  

00402ADF  |. 83C7 03        ADD EDI,3  

00402AE2  |. F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>  

00402AE4  |. FC             CLD  

00402AE5  |> 5F             POP EDI  

00402AE6  |. 5E             POP ESI  

00402AE7  \\. C3            RETN  
```

`4dd466d0e1e0cc123afbe504da2462ca`
