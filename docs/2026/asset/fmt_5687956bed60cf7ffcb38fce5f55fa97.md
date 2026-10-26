
### 0040B700
```
0040B700   |.  74 34           je short WO.0040B736

0040B702   |.  8D47 08         lea eax,dword ptr ds:[edi+8]

0040B705   |.  50              push eax                                      ; /pVar

0040B706   |.  FF15 4C414100   call dword ptr ds:[<&KERNEL32.InterlockedDecr>; \InterlockedDecrement
```
......

### 跟进 0040B6E9 call dword ptr ds:[ecx+38]
```
01653C11     B8 68A26601       mov eax,ComLicen.0166A268

01653C16     E8 59F30000       call ComLicen.01662F74

01653C1B     83EC 10           sub esp,10

01653C1E     56                push esi

01653C1F     8D4D 0C           lea ecx,dword ptr ss:[ebp+C]

01653C22     FF75 0C           push dword ptr ss:[ebp+C]

01653C25     E8 1FC70000       call ComLicen.01660349

01653C2A     8B45 0C           mov eax,dword ptr ss:[ebp+C]                  ; ASCII "1-3-1-4"

01653C2D     8D4D F0           lea ecx,dword ptr ss:[ebp-10]

01653C30     51                push ecx

01653C31     8D4D EC           lea ecx,dword ptr ss:[ebp-14]

01653C34     51                push ecx

01653C35     33F6              xor esi,esi

01653C37     8D4D E8           lea ecx,dword ptr ss:[ebp-18]

01653C3A     8975 E4           mov dword ptr ss:[ebp-1C],esi

01653C3D     8975 E8           mov dword ptr ss:[ebp-18],esi

01653C40     8975 EC           mov dword ptr ss:[ebp-14],esi

01653C43     8975 F0           mov dword ptr ss:[ebp-10],esi

01653C46     8B40 F8           mov eax,dword ptr ds:[eax-8]

01653C49     51                push ecx

01653C4A     8D4D E4           lea ecx,dword ptr ss:[ebp-1C]

01653C4D     51                push ecx

01653C4E     40                inc eax

01653C4F     68 34E36601       push ComLicen.0166E334                        ; ASCII "%ld-%ld-%ld-%ld"

01653C54     50                push eax

01653C55     8D4D 0C           lea ecx,dword ptr ss:[ebp+C]

01653C58     8975 FC           mov dword ptr ss:[ebp-4],esi

01653C5B     E8 51C80000       call ComLicen.016604B1

01653C60     50                push eax

01653C61     E8 23FD0000       call ComLicen.01663989                        ; 格式化读入，用于验证比较

01653C66     83C4 18           add esp,18

01653C69     83F8 04           cmp eax,4

01653C6C     75 1A             jnz short ComLicen.01653C88

01653C6E     FF75 10           push dword ptr ss:[ebp+10]

01653C71     8B45 08           mov eax,dword ptr ss:[ebp+8]

01653C74     FF75 F0           push dword ptr ss:[ebp-10]

01653C77     8B08              mov ecx,dword ptr ds:[eax]

01653C79     FF75 EC           push dword ptr ss:[ebp-14]

01653C7C     FF75 E8           push dword ptr ss:[ebp-18]

01653C7F     FF75 E4           push dword ptr ss:[ebp-1C]

01653C82     50                push eax

01653C83     FF51 30           call dword ptr ds:[ecx+30]                    ; 算法验证，跟进

01653C86     8BF0              mov esi,eax

01653C88     8D4D 0C           lea ecx,dword ptr ss:[ebp+C]

01653C8B     E8 F2EEFFFF       call ComLicen.01652B82

01653C90     8B4D F4           mov ecx,dword ptr ss:[ebp-C]

01653C93     8BC6              mov eax,esi

01653C95     5E                pop esi

01653C96     64:890D 00000000  mov dword ptr fs:[0],ecx

01653C9D     C9                leave

01653C9E     C2 0C00           retn 0C
```

### 跟进 01653C83 call dword ptr ds:[ecx+30]
```
01653D1D     50                push eax                                      ; ASCII "D:\Program Files\WinUtilities\UserData\User.lcs"

01653D1E     8D46 14           lea eax,dword ptr ds:[esi+14]

01653D21     68 FF000000       push 0FF

01653D26     50                push eax

01653D27     68 28126701       push ComLicen.01671228

01653D2C     68 68E36601       push ComLicen.0166E368                        ; ASCII "Name"

01653D31     68 60E36601       push ComLicen.0166E360                        ; ASCII "License"

01653D36     FF15 90B06601     call dword ptr ds:[<&KERNEL32.GetPrivateProfi>; KERNEL32.GetPrivateProfileStringA

01653D3C     FF76 10           push dword ptr ds:[esi+10]

01653D3F     8D86 13010000     lea eax,dword ptr ds:[esi+113]

01653D45     68 FF000000       push 0FF

01653D4A     50                push eax

01653D4B     68 28126701       push ComLicen.01671228

01653D50     68 58E36601       push ComLicen.0166E358                        ; ASCII "Code"

01653D55     68 60E36601       push ComLicen.0166E360                        ; ASCII "License"

01653D5A     FF15 90B06601     call dword ptr ds:[<&KERNEL32.GetPrivateProfi>; KERNEL32.GetPrivateProfileStringA

01653D60     FF15 A4B26601     call dword ptr ds:[<&WINMM.timeGetTime>]      ; WINMM.timeGetTime

01653D66     6A 0A             push 0A

01653D68     33D2              xor edx,edx

01653D6A     59                pop ecx

01653D6B     F7F1              div ecx

01653D6D     83FA 09           cmp edx,9

01653D70     0F87 8B4E0000     ja ComLicen.01658C01

01653D76     FF2495 11946501   jmp dword ptr ds:[edx*4+1659411]

01653D7D     33C0              xor eax,eax

01653D7F     8D4D 08           lea ecx,dword ptr ss:[ebp+8]

01653D82     50                push eax

01653D83     50                push eax

01653D84     50                push eax

01653D85     50                push eax

01653D86     51                push ecx

01653D87     50                push eax

01653D88     50                push eax

01653D89     68 54E36601       push ComLicen.0166E354                        ; ASCII "c:\"

01653D8E     8945 08           mov dword ptr ss:[ebp+8],eax

01653D91     FF15 8CB06601     call dword ptr ds:[<&KERNEL32.GetVolumeInform>; KERNEL32.GetVolumeInformationA

01653D97     8B45 08           mov eax,dword ptr ss:[ebp+8]

01653D9A     6A 0A             push 0A

01653D9C     33D2              xor edx,edx

01653D9E     59                pop ecx

01653D9F     F7F1              div ecx

01653DA1     83FA 09           cmp edx,9

01653DA4     0F87 FB060000     ja ComLicen.016544A5

01653DAA     FF2495 39946501   jmp dword ptr ds:[edx*4+1659439]
```
......

```
01656E31     8D46 14           lea eax,dword ptr ds:[esi+14]                 ; 取用户名

01656E34     57                push edi

01656E35     50                push eax

01656E36     8D85 E8FDFFFF     lea eax,dword ptr ss:[ebp-218]

01656E3C     50                push eax

01656E3D     E8 FEC50000       call ComLicen.01663440

01656E42     6A 14             push 14

01656E44     8D45 EC           lea eax,dword ptr ss:[ebp-14]

01656E47     6A 00             push 0

01656E49     50                push eax

01656E4A     E8 41C90000       call ComLicen.01663790

01656E4F     83C4 18           add esp,18

01656E52     8D45 FC           lea eax,dword ptr ss:[ebp-4]

01656E55     50                push eax

01656E56     8D45 F8           lea eax,dword ptr ss:[ebp-8]

01656E59     50                push eax

01656E5A     8D45 F4           lea eax,dword ptr ss:[ebp-C]

01656E5D     50                push eax

01656E5E     8D45 F0           lea eax,dword ptr ss:[ebp-10]
```
