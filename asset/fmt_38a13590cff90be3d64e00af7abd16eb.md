
```
00481072      68                 db 68                                       ;  CHAR 'h'  

00481073      E4                 db E4  

00481074      2F                 db 2F                                       ;  CHAR '/'  

00481075      4C                 db 4C                                       ;  CHAR 'L'  

00481076      00                 db 00  

00481077      8D                 db 8D  

00481078      4D                 db 4D                                       ;  CHAR 'M'  

00481079      EC                 db EC  

0048107A      E8                 db E8  

0048107B      DB                 db DB  

0048107C      16                 db 16  
```

Ctrl+A 重新分析代码

```
00481036   .  E8 F5ECFEFF      call MainCon.0046FD30  

0048103B   .  8D45 F0          lea eax,dword ptr ss:\[ebp-10\]  

0048103E   .  68 2C154C00      push MainCon.004C152C                        ; /Arg2 = 004C152C ASCII "MachineID"  

00481043   .  50               push eax                                     ; |Arg1  

00481044   .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]              ; |  

00481047   .  E8 64EDFEFF      call MainCon.0046FDB0                        ; \MainCon.0046FDB0  

0048104C   .  85C0             test eax,eax  

0048104E   .  74 0D            je short MainCon.0048105D  

00481050   .  68 E42F4C00      push MainCon.004C2FE4  

00481055   .  8D4D F0          lea ecx,dword ptr ss:\[ebp-10\]  

00481058   .  E8 FD160300      call MainCon.004B275A                        ;  jmp to MFC42.#860_CString::operator=  

0048105D   >  8D4D EC          lea ecx,dword ptr ss:\[ebp-14\]  

00481060   .  68 44154C00      push MainCon.004C1544                        ; /Arg2 = 004C1544 ASCII "UserSN"  

00481065   .  51               push ecx                                     ; |Arg1  

00481066   .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]              ; |  

00481069   .  E8 42EDFEFF      call MainCon.0046FDB0                        ; \MainCon.0046FDB0  

0048106E   .  85C0             test eax,eax                                 //返回到这里  

00481070   .  74 0D            je short MainCon.0048107F  

00481072   .  68 E42F4C00      push MainCon.004C2FE4  

00481077   .  8D4D EC          lea ecx,dword ptr ss:\[ebp-14\]  

0048107A   .  E8 DB160300      call MainCon.004B275A                        ;  jmp to MFC42.#860_CString::operator=  

0048107F   >  8D55 E8          lea edx,dword ptr ss:\[ebp-18\]  

00481082   .  68 38154C00      push MainCon.004C1538                        ; /Arg2 = 004C1538 ASCII "UserName"  

00481087   .  52               push edx                                     ; |Arg1  

00481088   .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]              ; |  

0048108B   .  E8 20EDFEFF      call MainCon.0046FDB0                        ; \MainCon.0046FDB0  

00481090   .  85C0             test eax,eax  

00481092   .  74 0D            je short MainCon.004810A1  

00481094   .  68 E42F4C00      push MainCon.004C2FE4  

00481099   .  8D4D E8          lea ecx,dword ptr ss:\[ebp-18\]  

0048109C   .  E8 B9160300      call MainCon.004B275A                        ;  jmp to MFC42.#860_CString::operator=  

004810A1   >  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  

004810A4   .  E8 07ECFEFF      call MainCon.0046FCB0  

004810A9   .  60               pushad  
```

F8 继续单步执行……

```
0047DCDD   >  \61                popad  

0047DCDE   .  B9 06000000        mov ecx,6  

0047DCE3   .  33C0              xor eax,eax  

0047DCE5   .  8D7D 9C           lea edi,dword ptr ss:\[ebp-64\]  

0047DCE8   .  6A 01             push 1  

0047DCEA   .  F3:AB             rep stos dword ptr es:\[edi\]  

0047DCEC   .  8D45 B4           lea eax,dword ptr ss:\[ebp-4C\]            ;  用户名  

0047DCEF   .  6A 18             push 18  

0047DCF1   .  50                push eax  

0047DCF2   .  8D4D CC           lea ecx,dword ptr ss:\[ebp-34\]  

0047DCF5   .  6A 18             push 18  

0047DCF7   .  8D55 9C           lea edx,dword ptr ss:\[ebp-64\]  

0047DCFA   .  51                push ecx  

0047DCFB   .  52                push edx  

0047DCFC   .  E8 9F60FFFF       call MainCon.00473DA0                       ;  机器码再次转换，重要  

0047DD01   .  83C4 18           add esp,18  

0047DD04   .  60                pushad  

0047DD05   .  6A 04             push 4  

0047DD07   .  6A 00             push 0  

0047DD09   .  6A 00             push 0  

0047DD0B   .  6A FF             push -1  

0047DD0D   .  90                nop  

0047DD0E   .  E8 F8A51000       call MainCon.0058830B  

0047DD13   .  EB 1E             jmp short MainCon.0047DD33  
```

直到异常在

```
7C92EAF0    8B1C24             mov ebx,dword ptr ss:\[esp\]  

7C92EAF3    51                 push ecx  

7C92EAF4    53                 push ebx  
```

多次 Ctrl+F9、F7 返回主模块，来到：

```
0047DB1B      50                 db 50                                       ;  CHAR 'P'  

0047DB1C      8D                 db 8D  

0047DB1D      4D                 db 4D                                       ;  CHAR 'M'  

0047DB1E      0C                 db 0C  

0047DB1F      C6                 db C6  

0047DB20      45                 db 45                                       ;  CHAR 'E'  

0047DB21      FC                 db FC  

0047DB22      02                 db 02  

0047DB23      E8                 db E8  

0047DB24      EC                 db EC  

0047DB25      4C                 db 4C                                       ;  CHAR 'L'  

0047DB26      03                 db 03  

0047DB27      00                 db 00  

0047DB28      8D                 db 8D  

0047DB29      4D                 db 4D                                       ;  CHAR 'M'  

0047DB2A      94                 db 94  

0047DB2B      C6                 db C6  

0047DB2C      45                 db 45                                       ;  CHAR 'E'  
```

Ctrl+A 重新分析代码

```
0047DADF   >  \B9 06000000       mov ecx,6  

0047DAE4   .  33C0               xor eax,eax  

0047DAE6   .  8D7D CC            lea edi,dword ptr ss:\[ebp-34\]  

0047DAE9   .  33DB               xor ebx,ebx  

0047DAEB   .  F3:AB              rep stos dword ptr es:\[edi\]  

0047DAED   .  8B4D 14           mov ecx,dword ptr ss:\[ebp+14\]  

0047DAF0   .  8B11              mov edx,dword ptr ds:\[ecx\]  

0047DAF2   .  8B42 F8           mov eax,dword ptr ds:\[edx-8\]  

0047DAF5   .  85C0              test eax,eax  

0047DAF7   .  0F8E 9F000000    jle MainCon.0047DB9C  

0047DAFD   >  8D4D 0C           lea ecx,dword ptr ss:\[ebp+C\]              ;  循环转换注册码  

0047DB00   .  E8 614C0300       call MainCon.004B2766                       ;  jmp to MFC42.#540_CString::CString  

0047DB05   .  8B4D 14           mov ecx,dword ptr ss:\[ebp+14\]  

0047DB08   .  6A 01             push 1  

0047DB0A   .  8D45 94           lea eax,dword ptr ss:\[ebp-6C\]  

0047DB0D   .  53                push ebx  

0047DB0E   .  50                push eax  

0047DB0F   .  C745 FC 01000000 mov dword ptr ss:\[ebp-4\],1  

0047DB16   .  E8 97510300       call MainCon.004B2CB2                       ;  jmp to MFC42.#4278_CString::Mid  

0047DB1B   .  50                push eax                                    ；返回到这里  

0047DB1C   .  8D4D 0C           lea ecx,dword ptr ss:\[ebp+C\]  

0047DB1F   .  C645 FC 02        mov byte ptr ss:\[ebp-4\],2  

0047DB23   .  E8 EC4C0300       call MainCon.004B2814                       ;  jmp to MFC42.#858_CString::operator=  

0047DB28   .  8D4D 94           lea ecx,dword ptr ss:\[ebp-6C\]  

0047DB2B   .  C645 FC 01        mov byte ptr ss:\[ebp-4\],1  

0047DB2F   .  E8 1A4C0300       call MainCon.004B274E                       ;  jmp to MFC42.#800_CString::~CString  

0047DB34   .  8B55 0C           mov edx,dword ptr ss:\[ebp+C\]  

0047DB37   .  83C9 FF           or ecx,FFFFFFFF  

0047DB3A   .  8BFA              mov edi,edx  

0047DB3C   .  33C0              xor eax,eax  

0047DB3E   .  F2:AE             repne scas byte ptr es:\[edi\]  

0047DB40   .  F7D1              not ecx  

0047DB42   .  8D75 10           lea esi,dword ptr ss:\[ebp+10\]  

0047DB45   .  2BF9              sub edi,ecx  

0047DB47   .  8BC1              mov eax,ecx  

0047DB49   .  8975 E8           mov dword ptr ss:\[ebp-18\],esi  

0047DB4C   .  8BF7              mov esi,edi  
```
