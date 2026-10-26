
# Switch table used at `004B428A`

```
004B4295    |.  F9424B00               dd talisman.004B42F9

004B4299    |.  03434B00               dd talisman.004B4303

004B429D    |.  0A434B00               dd talisman.004B430A

004B42A1    |.  11434B00               dd talisman.004B4311

004B42A5    |.  18434B00               dd talisman.004B4318

004B42A9    |.  1F434B00               dd talisman.004B431F

004B42AD    |.  26434B00               dd talisman.004B4326

004B42B1    |.  2D434B00               dd talisman.004B432D

004B42B5    |.  34434B00               dd talisman.004B4334

004B42B9    |.  3B434B00               dd talisman.004B433B

004B42BD    |.  42434B00               dd talisman.004B4342

004B42C1    |.  49434B00               dd talisman.004B4349

004B42C5    |.  50434B00               dd talisman.004B4350

004B42C9    |.  57434B00               dd talisman.004B4357

004B42CD    |.  5E434B00               dd talisman.004B435E

004B42D1    |.  65434B00               dd talisman.004B4365

004B42D5    |.  6C434B00               dd talisman.004B436C

004B42D9    |.  73434B00               dd talisman.004B4373

004B42DD    |.  7A434B00               dd talisman.004B437A

004B42E1    |.  E5424B00               dd talisman.004B42E5

004B42E5    |>  BB F5034401            mov ebx,14403F5                                    ;

Case 2A of switch 004B427E

004B42EA    |.  E9 94000000            jmp talisman.004B4383

004B42EF    |>  BB DBB3A702            mov ebx,2A7B3DB                                    ;

Case 16 of switch 004B427E

004B42F4    |.  E9 8A000000            jmp talisman.004B4383

004B42F9    |>  BB 5A044401            mov ebx,144045A                                    ;

Case 17 of switch 004B427E

004B42FE    |.  E9 80000000            jmp talisman.004B4383

004B4303    |>  BB 9F55F502            mov ebx,2F5559F                                    ;

Case 18 of switch 004B427E

004B4308    |.  EB 79                  jmp short talisman.004B4383

004B430A    |>  BB 357B5F01            mov ebx,15F7B35                                    ;

Case 19 of switch 004B427E

004B430F    |.  EB 72                  jmp short talisman.004B4383

004B4311    |>  BB 71014401            mov ebx,1440171                                    ;

Case 1A of switch 004B427E

004B4316    |.  EB 6B                  jmp short talisman.004B4383

004B4318    |>  BB 05A84401            mov ebx,144A805                                    ;

Case 1B of switch 004B427E

004B431D    |.  EB 64                  jmp short talisman.004B4383

004B431F    |>  BB A528B204            mov ebx,4B228A5                                    ;

Case 1C of switch 004B427E

004B4324    |.  EB 5D                  jmp short talisman.004B4383

004B4326    |>  BB 15B6DA01            mov ebx,1DAB615                                    ;

Case 1D of switch 004B427E

004B432B    |.  EB 56                  jmp short talisman.004B4383

004B432D    |>  BB DB867601            mov ebx,17686DB                                    ;

Case 1E of switch 004B427E

004B4332    |.  EB 4F                  jmp short talisman.004B4383

004B4334    |>  BB 81F43E04            mov ebx,43EF481                                    ;

Case 1F of switch 004B427E

004B4339    |.  EB 48                  jmp short talisman.004B4383

004B433B    |>  BB D7F33E04            mov ebx,43EF3D7                                    ;

Case 20 of switch 004B427E

004B4340    |.  EB 41                  jmp short talisman.004B4383

004B4342    |>  BB BFEE6F04            mov ebx,46FEEBF                                    ;

Case 21 of switch 004B427E

004B4347    |.  EB 3A                  jmp short talisman.004B4383

004B4349    |>  BB A7363104            mov ebx,43136A7                                    ;

Case 22 of switch 004B427E

004B434E    |.  EB 33                  jmp short talisman.004B4383

004B4350    |>  BB BB344E04            mov ebx,44E34BB                                    ;

Case 23 of switch 004B427E

004B4355    |.  EB 2C                  jmp short talisman.004B4383

004B4357    |>  BB F2034004            mov ebx,44003F2                                    ;

Case 24 of switch 004B427E

004B435C    |.  EB 25                  jmp short talisman.004B4383

004B435E    |>  BB 6308B904            mov ebx,4B90863                                    ;

Case 25 of switch 004B427E

004B4363    |.  EB 1E                  jmp short talisman.004B4383

004B4365    |>  BB E344EF02            mov ebx,2EF44E3                                    ;

Case 26 of switch 004B427E

004B436A    |.  EB 17                  jmp short talisman.004B4383

004B436C    |>  BB 3B9EB503            mov ebx,3B59E3B                                    ;

Case 27 of switch 004B427E

004B4371    |.  EB 10                  jmp short talisman.004B4383

004B4373    |>  BB 568F4604            mov ebx,4468F56                                    ;

Case 28 of switch 004B427E

004B4378    |.  EB 09                  jmp short talisman.004B4383

004B437A    |>  BB 7BA28304            mov ebx,483A27B                                    ;

Case 29 of switch 004B427E

004B437F    |.  EB 02                  jmp short talisman.004B4383

004B4381    |>  33DB                   xor ebx,ebx                                        ;

Default case of switch 004B427E

004B4383    |>  8D55 FC                lea edx,dword ptr ss:[ebp-4]

004B4386    |.  8BC6                   mov eax,esi

004B4388    |.  E8 3B3FF5FF            call talisman.004082C8

004B438D    |.  FF75 FC                push dword ptr ss:[ebp-4]

004B4390    |.  68 E0434B00            push talisman.004B43E0

004B4395    |.  8D55 F8                lea edx,dword ptr ss:[ebp-8]

004B4398    |.  8BC3                   mov eax,ebx

004B439A    |.  E8 293FF5FF            call talisman.004082C8

004B439F    |.  FF75 F8                push dword ptr ss:[ebp-8]

004B43A2    |.  8BC7                   mov eax,edi

004B43A4    |.  BA 03000000            mov edx,3

004B43A9    |.  E8 CEFAF4FF            call talisman.00403E7C

004B43AE    |.  33C0                   xor eax,eax

004B43B0    |.  5A                     pop edx

004B43B1    |.  59                     pop ecx

004B43B2    |.  59                     pop ecx

004B43B3    |.  64:8910                mov dword ptr fs:[eax],edx

004B43B6    |.  68 D0434B00            push talisman.004B43D0

004B43BB    |>  8D45 F8                lea eax,dword ptr ss:[ebp-8]

004B43BE    |.  BA 02000000            mov edx,2

004B43C3    |.  E8 9CF7F4FF            call talisman.00403B64

004B43C8    \.  C3                     retn

004B43C9     .^ E9 16F2F4FF            jmp talisman.004035E4

004B43CE     .^ EB EB                  jmp short talisman.004B43BB

004B43D0     .  5F                     pop edi

004B43D1     .  5E                     pop esi

004B43D2     .  5B                     pop ebx

004B43D3     .  59                     pop ecx

004B43D4     .  59                     pop ecx

004B43D5     .  5D                     pop ebp

004B43D6     .  C3                     retn
```

## [分析七]Switch (cases 2B..3F)

```
004BBCE0    /$  55                     push ebp

004BBCE1    |.  8BEC                   mov ebp,esp

004BBCE3    |.  6A 00                  push 0

004BBCE5    |.  6A 00                  push 0

004BBCE7    |.  53                     push ebx

004BBCE8    |.  56                     push esi

004BBCE9    |.  57                     push edi

004BBCEA    |.  8BF9                   mov edi,ecx

004BBCEC    |.  8BF2                   mov esi,edx

004BBCEE    |.  33C0                   xor eax,eax

004BBCF0    |.  55                     push ebp

004BBCF1    |.  68 43BE4B00            push talisman.004BBE43

004BBCF6    |.  64:FF30                push dword ptr fs:[eax]

004BBCF9    |.  64:8920                mov dword ptr fs:[eax],esp

004BBCFC    |.  8BC6                   mov eax,esi
```
