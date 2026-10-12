
```
  004C9E10  |.  C645 FC 0D    mov     byte ptr [ebp-4], 0D

  004C9E14  |.  8D8D 2CFBFFFF lea     ecx, dword ptr [ebp-4D4]

  004C9E1A  |.  E8 5281F3FF   call    00401F71

  004C9E1F  |.  C645 FC 0C    mov     byte ptr [ebp-4], 0C

  004C9E23  |.  8D8D 20FBFFFF lea     ecx, dword ptr [ebp-4E0]

  004C9E29  |.  E8 E4400A00   call    0056DF12

  004C9E2E  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

  004C9E32  |.  8D8D 64FCFFFF lea     ecx, dword ptr [ebp-39C]

  004C9E38  |.  E8 D5400A00   call    0056DF12

  004C9E3D  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

  004C9E41  |.  8D8D 1CFFFFFF lea     ecx, dword ptr [ebp-E4]

  004C9E47  |.  E8 C6400A00   call    0056DF12

  004C9E4C  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

  004C9E50  |.  8D4D BC       lea     ecx, dword ptr [ebp-44]

  004C9E53  |.  E8 BA400A00   call    0056DF12

  004C9E58  |.  C645 FC 02    mov     byte ptr [ebp-4], 2

  004C9E5C  |.  8D8D 60FCFFFF lea     ecx, dword ptr [ebp-3A0]

  004C9E62  |.  E8 AB400A00   call    0056DF12

  004C9E67  |.  C645 FC 01    mov     byte ptr [ebp-4], 1

  004C9E6B  |.  8D8D 78FCFFFF lea     ecx, dword ptr [ebp-388]

  004C9E71  |.  E8 9C400A00   call    0056DF12

  004C9E76  |.  C645 FC 00    mov     byte ptr [ebp-4], 0

  004C9E7A  |.  8D8D 24FBFFFF lea     ecx, dword ptr [ebp-4DC]

  004C9E80  |.  E8 8D400A00   call    0056DF12

  004C9E85  |.  C745 FC FFFFF>mov     dword ptr [ebp-4], -1

  004C9E8C  |.  8D4D 08       lea     ecx, dword ptr [ebp+8]

  004C9E8F  |.  E8 7E400A00   call    0056DF12

  004C9E94  |.  8B85 30FAFFFF mov     eax, dword ptr [ebp-5D0]

  004C9E9A  |.  E9 19060000   jmp     004CA4B8

  004C9E9F  |>  8D55 C0       lea     edx, dword ptr [ebp-40]

  004C9EA2  |.  52            push    edx                          ; /pFileInformation

  004C9EA3  |.  8B85 BCFBFFFF mov     eax, dword ptr [ebp-444]             ; |

  004C9EA9  |.  50            push    eax                          ; |hFile

  004C9EAA  |.  FF15 80E45900 call    dword ptr [<&KERNEL32.GetFileInf>; \GetFileInformationByHandle

  004C9EB0  |.  8985 54FCFFFF mov     dword ptr [ebp-3AC], eax

  004C9EB6  |.  8B8D BCFBFFFF mov     ecx, dword ptr [ebp-444]

  004C9EBC  |.  51            push    ecx                          ; /hObject

  004C9EBD  |.  FF15 C0E45900 call    dword ptr [<&KERNEL32.CloseHandl>; \CloseHandle

  004C9EC3  |.  8D55 C4       lea     edx, dword ptr [ebp-3C]

  004C9EC6  |.  8955 B4       mov     dword ptr [ebp-4C], edx

  004C9EC9  |.  837D B4 00    cmp     dword ptr [ebp-4C], 0

  004C9ECD  |.  74 22         je      short 004C9EF1

  004C9ECF  |.  6A FF         push    -1

  004C9ED1  |.  8B45 B4       mov     eax, dword ptr [ebp-4C]

  004C9ED4  |.  50            push    eax

  004C9ED5  |.  8D8D 2CFAFFFF lea     ecx, dword ptr [ebp-5D4]

  004C9EDB  |.  E8 631B0A00   call    0056BA43

  004C9EE0  |.  8985 C8F9FFFF mov     dword ptr [ebp-638], eax

  004C9EE6  |.  8B8D C8F9FFFF mov     ecx, dword ptr [ebp-638]

  004C9EEC  |.  8B11          mov     edx, dword ptr [ecx]

  004C9EEE  |.  8955 B0       mov     dword ptr [ebp-50], edx

  004C9EF1  |>  68 C82B5E00   push    005E2BC8                       ;   %

  004C9EF6  |.  8D85 28FAFFFF lea     eax, dword ptr [ebp-5D8]

  004C9EFC  |.  50            push    eax

  004C9EFD  |.  8D4D B0       lea     ecx, dword ptr [ebp-50]

  004C9F00  |.  E8 231C0A00   call    0056BB28                       ;   可能是规定注册时间格式

  004C9F05  |.  8985 7CF9FFFF mov     dword ptr [ebp-684], eax

  004C9F0B  |.  8B8D 7CF9FFFF mov     ecx, dword ptr [ebp-684]

  004C9F11  |.  898D 78F9FFFF mov     dword ptr [ebp-688], ecx

  004C9F17  |.  C645 FC 17    mov     byte ptr [ebp-4], 17

  004C9F1B  |.  8B95 78F9FFFF mov     edx, dword ptr [ebp-688]

  004C9F21  |.  52            push    edx

  004C9F22  |.  8D8D 78FCFFFF lea     ecx, dword ptr [ebp-388]

  004C9F28  |.  E8 2C410A00   call    0056E059

  004C9F2D  |.  C645 FC 11    mov     byte ptr [ebp-4], 11

  004C9F31  |.  8D8D 28FAFFFF lea     ecx, dword ptr [ebp-5D8]

  004C9F37  |.  E8 D63F0A00   call    0056DF12

  004C9F3C  |.  8D45 B8       lea     eax, dword ptr [ebp-48]

  004C9F3F  |.  50            push    eax

  004C9F40  |.  E8 4A1B0A00   call    0056BA8F

  004C9F45  |.  51            push    ecx

  004C9F46  |.  8BCC          mov     ecx, esp

  004C9F48  |.  89A5 24FAFFFF mov     dword ptr [ebp-5DC], esp

  004C9F4E  |.  898D C4F9FFFF mov     dword ptr [ebp-63C], ecx

  004C9F54  |.  8B95 C4F9FFFF mov     edx, dword ptr [ebp-63C]

  004C9F5A  |.  8B45 B8       mov     eax, dword ptr [ebp-48]

  004C9F5D  |.  8902          mov     dword ptr [edx], eax

  004C9F5F  |.  8D8D 20FAFFFF lea     ecx, dword ptr [ebp-5E0]

  004C9F65  |.  51            push    ecx

  004C9F66  |.  8D4D B0       lea     ecx, dword ptr [ebp-50]

  004C9F69  |.  E8 F27AF7FF   call    00441A60                       ;   用以上结果计算出新结果，作用不清楚

  004C9F6E  |.  8985 74F9FFFF mov     dword ptr [ebp-68C], eax

  004C9F74  |.  8B95 74F9FFFF mov     edx, dword ptr [ebp-68C]

  004C9F7A  |.  8B02          mov     eax, dword ptr [edx]

  004C9F7C  |.  8985 50FCFFFF mov     dword ptr [ebp-3B0], eax

  004C9F82  |.  8B85 50FCFFFF mov     eax, dword ptr [ebp-3B0]

  004C9F88  |.  99            cdq

  004C9F89  |.  B9 80510100   mov     ecx, 15180

  004C9F8E  |.  F7F9          idiv    ecx

  004C9F90  |.  85C0          test    eax, eax

  004C9F92  |.  0F8E D6000000 jle     004CA06E

  004C9F98  |.  8B95 B4F9FFFF mov     edx, dword ptr [ebp-64C]

  004C9F9E  |.  C782 14010000>mov     dword ptr [edx+114], 0

  004C9FA8  |.  C785 1CFAFFFF>mov     dword ptr [ebp-5E4], 0

  004C9FB2  |.  C645 FC 10    mov     byte ptr [ebp-4], 10

  004C9FB6  |.  8D8D 20FFFFFF lea     ecx, dword ptr [ebp-E0]

  004C9FBC  |.  E8 B07FF3FF   call    00401F71

  004C9FC1  |.  C645 FC 0F    mov     byte ptr [ebp-4], 0F

  004C9FC5  |.  8D8D C0FBFFFF lea     ecx, dword ptr [ebp-440]

  004C9FCB  |.  E8 A17FF3FF   call    00401F71

  004C9FD0  |.  C645 FC 0E    mov     byte ptr [ebp-4], 0E

  004C9FD4  |.  8D8D 84FCFFFF lea     ecx, dword ptr [ebp-37C]

  004C9FDA  |.  E8 927FF3FF   call    00401F71

  004C9FDF  |.  C645 FC 0D    mov     byte ptr [ebp-4], 0D

  004C9FE3  |.  8D8D 2CFBFFFF lea     ecx, dword ptr [ebp-4D4]

  004C9FE9  |.  E8 837FF3FF   call    00401F71

  004C9FEE  |.  C645 FC 0C    mov     byte ptr [ebp-4], 0C

  004C9FF2  |.  8D8D 20FBFFFF lea     ecx, dword ptr [ebp-4E0]

  004C9FF8  |.  E8 153F0A00   call    0056DF12                           ;下面这几个 CALL 应该是储存上面所有运算出字符串的结果,写入 RWJunk.dll 文件中

  004C9FFD  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

  004CA001  |.  8D8D 64FCFFFF lea     ecx, dword ptr [ebp-39C]

  004CA007  |.  E8 063F0A00   call    0056DF12

  ...............................算了，还是省略掉吧，代码太多看着就头晕哦，呵呵

  004CA438  |.  E8 347BF3FF   call    00401F71

  004CA43D  |.  C645 FC 0C    mov     byte ptr [ebp-4], 0C

  004CA441  |.  8D8D 20FBFFFF lea     ecx, dword ptr [ebp-4E0]

  004CA447  |.  E8 C63A0A00   call    0056DF12                       ; 下面这几个 CALL 应该是储存上面所有运算出字符串的结果

  004CA44C  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

  004CA450  |.  8D8D 64FCFFFF lea     ecx, dword ptr [ebp-39C]

  004CA456  |.  E8 B73A0A00   call    0056DF12

  004CA45B  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

  004CA45F  |.  8D8D 1CFFFFFF lea     ecx, dword ptr [ebp-E4]

  004CA465  |.  E8 A83A0A00   call    0056DF12

  004CA46A  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

  004CA46E  |.  8D4D BC       lea     ecx, dword ptr [ebp-44]
```
