
### 5. 结束

```
00515879      8881 00634000                   mov byte ptr ds:[ecx+406300],al----》不跳，下条指令偏移
量从 406300 开始放       
0051587F      90                              nop
00515880      90                              nop
00515881      FF05 04404000                   inc dword ptr ds:[404004]
00515887      E9 55FCFFFF                     jmp Aspr.005154E1
0051588C      90                              nop
0051588D      90                              nop
0051588E      EB 1D                           jmp short Aspr.005158AD
00515890      33C9                            xor ecx,ecx
00515892      74 00                           je short Aspr.00515894
00515894      EB 10                           jmp short Aspr.005158A6
00515896      F9                              stc
```

### 6. 开始

```
00515897      8B3D 04404000                   mov edi,dword ptr ds:[404004]----》 //有效 int3 的计数器
0051589D      81C7 00614000                   add edi,Aspr.00406100 ---》//跳转类型从 00406100 开始放  
005158A3      BA 0C404000                     mov edx,Aspr.0040400C
005158A8      8B42 04                           mov eax,dword ptr ds:[edx+4]
005158AB      8B5A 08                           mov ebx,dword ptr ds:[edx+8]
005158AE      33D8                              xor ebx,eax
005158B0      74 0F                             je short Aspr.005158C1
005158B2      90                                nop
005158B3      90                                nop
005158B4      C607 E3                           mov byte ptr ds:[edi],0E3
005158B7      E9 20010000                       jmp Aspr.005159DC
005158BC      90                                nop
005158BD      90                                nop
005158BE      90                                nop
005158BF      90                                nop
005158C0      90                                nop
005158C1      8B42 0C                           mov eax,dword ptr ds:[edx+C]
005158C4      8B5A 14                           mov ebx,dword ptr ds:[edx+14]
005158C7      33D8                              xor ebx,eax
005158C9      74 1D                             je short Aspr.005158E8
005158CB      85C0                              test eax,eax
005158CD      74 0C                             je short Aspr.005158DB
005158CF      90                                nop
005158D0      90                                nop
005158D1      C607 7B                           mov byte ptr ds:[edi],7B
005158D4      E9 03010000                       jmp Aspr.005159DC
005158D9      90                                nop
005158DA      90                                nop
005158DB      C607 7A                           mov byte ptr ds:[edi],7A
005158DE      E9 F9000000                       jmp Aspr.005159DC
005158E3      90                                nop
005158E4      90                                nop
005158E5      90                                nop
005158E6      90                                nop
005158E7      90                                nop
005158E8      8B5A 18                           mov ebx,dword ptr ds:[edx+18]
005158EB      33D8                              xor ebx,eax
005158ED      74 61                             je short Aspr.00515950
005158EF      90                                nop
005158F0      90                                nop
005158F1      8B5A 10                           mov ebx,dword ptr ds:[edx+10]
005158F4      33D8                              xor ebx,eax
005158F6      74 1A                             je short Aspr.00515912
005158F8      85C0                              test eax,eax
005158FA      74 0C                             je short Aspr.00515908
005158FC      90                                nop
005158FD      90                                nop
005158FE      C607 77                           mov byte ptr ds:[edi],77
00515901      E9 D6000000                       jmp Aspr.005159DC
00515906      90                                nop
00515907      90                                nop
00515908      C607 76                           mov byte ptr ds:[edi],76
0051590B      E9 CC000000                       jmp Aspr.005159DC
00515910      90                                nop
00515911      90                                nop
00515912      8B5A 1C                           mov ebx,dword ptr ds:[edx+1C]
00515915      33D8                              xor ebx,eax
00515917      74 1A                             je short Aspr.00515933
00515919      85C0                              test eax,eax
0051591B      74 0C                             je short Aspr.00515929
0051591D      90                                nop
0051591E      90                                nop
0051591F      C607 7F                           mov byte ptr ds:[edi],7F
00515922      E9 B5000000                       jmp Aspr.005159DC
00515927      90                                nop
00515928      90                                nop
00515929      C607 7E                           mov byte ptr ds:[edi],7E
0051592C      E9 AB000000                       jmp Aspr.005159DC
00515931      90                                nop
00515932      90                                nop
00515933      85C0                              test eax,eax
00515935      74 0C                             je short Aspr.00515943
00515937      90                                nop
00515938      90                                nop
00515939      C607 75                           mov byte ptr ds:[edi],75
0051593C      E9 9B000000                       jmp Aspr.005159DC
00515941      90                                nop
00515942      90                                nop
00515943      C607 74                           mov byte ptr ds:[edi],74
00515946      E9 91000000                       jmp Aspr.005159DC
0051594B      90                                nop
0051594C      90                                nop
0051594D      90                                nop
0051594E      90                                nop
0051594F      90                                nop
00515950      8B5A 1C                           mov ebx,dword ptr ds:[edx+1C]
00515953      33D8                              xor ebx,eax
00515955      74 34                             je short Aspr.0051598B
00515957      90                                nop
00515958      90                                nop
00515959      8B5A 20                           mov ebx,dword ptr ds:[edx+20]
0051595C      33D8                              xor ebx,eax
0051595E      74 14                             je short Aspr.00515974
00515960      85C0                              test eax,eax
00515962      74 09                             je short Aspr.0051596D
00515964      90                                nop
00515965      90                                nop
00515966      C607 7D                           mov byte ptr ds:[edi],7D
00515969      EB 71                             jmp short Aspr.005159DC
0051596B      90                                nop
0051596C      90                                nop
0051596D      C607 7C                           mov byte ptr ds:[edi],7C
00515970      EB 6A                             jmp short Aspr.005159DC
00515972      90                                nop
00515973      90                                nop
00515974      85C0                              test eax,eax
00515976      74 09                             je short Aspr.00515981
00515978      90                                nop
00515979      90                                nop
0051597A      C607 79                           mov byte ptr ds:[edi],79
0051597D      EB 5D                             jmp short Aspr.005159DC
0051597F      90                                nop
00515980      90                                nop
00515981      C607 78                           mov byte ptr ds:[edi],78
00515984      EB 56                             jmp short Aspr.005159DC
00515986      90                                nop
00515987      90                                nop
00515988      90                                nop
00515989      90                                nop
0051598A      90                                nop
0051598B      8B5A 20                           mov ebx,dword ptr ds:[edx+20]
0051598E      33D8                              xor ebx,eax
00515990      74 17                             je short Aspr.005159A9
```
