
```
007E8B3F    B9 A88D7E00     mov     ecx, 007E8DA8                     ; ASCII "softname"
007E8B44    BA BC8D7E00     mov     edx, 007E8DBC                     ; ASCII "frmMainHint"
007E8B49    E8 F225E4FF     call    0062B140
007E8B4E    8B55 FC         mov     edx, dword ptr [ebp-4]
007E8B51    A1 D0DD8700     mov     eax, dword ptr [87DDD0]
007E8B56    8B00            mov     eax, dword ptr [eax]
007E8B58    E8 9321CCFF     call    004AACF0
007E8B5D    8D55 F8         lea     edx, dword ptr [ebp-8]
007E8B60    8B83 54030000   mov     eax, dword ptr [ebx+354]
007E8B66    E8 9D02CAFF     call    00488E08
007E8B6B    8B55 F8         mov     edx, dword ptr [ebp-8]
007E8B6E    8D83 88030000   lea     eax, dword ptr [ebx+388]
007E8B74    E8 CBC5C1FF     call    00405144
007E8B79    A1 B4DB8700     mov     eax, dword ptr [87DBB4]
007E8B7E    8B00            mov     eax, dword ptr [eax]
007E8B80    8B48 0C         mov     ecx, dword ptr [eax+C]
007E8B83    B2 01           mov     dl, 1
007E8B85    A1 60F04400     mov     eax, dword ptr [44F060]
007E8B8A    E8 8165C6FF     call    0044F110
007E8B8F    8BF0            mov     esi, eax
007E8B91    89B3 78030000   mov     dword ptr [ebx+378], esi
007E8B97    68 D08D7E00     push    007E8DD0                          ; ASCII "quit"
007E8B9C    8D45 F4         lea     eax, dword ptr [ebp-C]
007E8B9F    50              push    eax
007E8BA0    B9 E08D7E00     mov     ecx, 007E8DE0                     ; ASCII "btnQuit"
007E8BA5    8B53 08         mov     edx, dword ptr [ebx+8]
007E8BA8    8BC6            mov     eax, esi
007E8BAA    8B30            mov     esi, dword ptr [eax]
007E8BAC    FF16            call    dword ptr [esi]
007E8BAE    8B55 F4         mov     edx, dword ptr [ebp-C]
007E8BB1    8B83 3C030000   mov     eax, dword ptr [ebx+33C]
007E8BB7    E8 7C02CAFF     call    00488E38
007E8BBC    33C0            xor     eax, eax
007E8BBE    8983 84030000   mov     dword ptr [ebx+384], eax
007E8BC4    A1 04DC8700     mov     eax, dword ptr [87DC04]
007E8BC9    8B00            mov     eax, dword ptr [eax]
007E8BCB    8078 24 00      cmp     byte ptr [eax+24], 0              !!!!!!!!!!!!!!! 是否注册过期? 不等于 0,过期
007E8BCF    74 21           je      short 007E8BF2
007E8BD1    68 E88D7E00     push    007E8DE8                          ; ASCII "TRS_EBoardService"
007E8BD6    68 FC8D7E00     push    007E8DFC                          ; ASCII "TfrmRS_EBoard_Service"
007E8BDB    E8 F0F8C1FF     call    004084D0                          ; jmp 到 user32.FindWindowA
007E8BE0    8983 94030000   mov     dword ptr [ebx+394], eax
007E8BE6    A1 04DC8700     mov     eax, dword ptr [87DC04]
007E8BEB    8B00            mov     eax, dword ptr [eax]
007E8BED    8B70 28         mov     esi, dword ptr [eax+28]
007E8BF0    EB 07           jmp     short 007E8BF9
007E8BF2    E8 B51CF3FF     call    0071A8AC
007E8BF7    8BF0            mov     esi, eax
007E8BF9    85F6            test    esi, esi
007E8BFB    7E 05           jle     short 007E8C02
007E8BFD    83FE 1E         cmp     esi, 1E
007E8C00    7E 04           jle     short 007E8C06
007E8C02    33C0            xor     eax, eax
007E8C04    EB 02           jmp     short 007E8C08
007E8C06    B0 01           mov     al, 1
007E8C08    8883 98030000   mov     byte ptr [ebx+398], al
007E8C0E    85F6            test    esi, esi
007E8C10    7E 05           jle     short 007E8C17
007E8C12    83FE 1E         cmp     esi, 1E
007E8C15    7E 4D           jle     short 007E8C64
007E8C17    8B83 48030000   mov     eax, dword ptr [ebx+348]
007E8C1D    8B40 68         mov     eax, dword ptr [eax+68]
007E8C20    BA FF000000     mov     edx, 0FF
007E8C25    E8 9E62C4FF     call    0042EEC8
007E8C2A    68 1C8E7E00     push    007E8E1C                          ; ASCII "???"
007E8C2F    8D45 F0         lea     eax, dword ptr [ebp-10]
007E8C32    50              push    eax
007E8C33    BA BC8D7E00     mov     edx, 007E8DBC                     ; ASCII "frmMainHint"
007E8C38    B9 288E7E00     mov     ecx, 007E8E28                     ; ASCII "testhint1"
007E8C3D    8B83 78030000   mov     eax, dword ptr [ebx+378]
007E8C43    8B30            mov     esi, dword ptr [eax]
007E8C45    FF16            call    dword ptr [esi]
007E8C47    8B55 F0         mov     edx, dword ptr [ebp-10]
007E8C4A    8B83 48030000   mov     eax, dword ptr [ebx+348]
007E8C50    E8 E301CAFF     call    00488E38
007E8C55    33D2            xor     edx, edx
007E8C57    8B83 38030000   mov     eax, dword ptr [ebx+338]
007E8C5D    8B08            mov     ecx, dword ptr [eax]
007E8C5F    FF51 64         call    dword ptr [ecx+64]
007E8C62    EB 65           jmp     short 007E8CC9
007E8C64    83FE 1E         cmp     esi, 1E
007E8C67    7F 60           jg      short 007E8CC9
007E8C69    BA FF000000     mov     edx, 0FF
007E8C6E    8B83 48030000   mov     eax, dword ptr [ebx+348]
007E8C74    E8 2F03CAFF     call    00488FA8
007E8C79    8D45 EC         lea     eax, dword ptr [ebp-14]
007E8C7C    50              push    eax
007E8C7D    68 3C8E7E00     push    007E8E3C                          ; ASCII "%d"
007E8C82    8D45 E8         lea     eax, dword ptr [ebp-18]
007E8C85    50              push    eax
007E8C86    BA BC8D7E00     mov     edx, 007E8DBC                     ; ASCII "frmMainHint"
007E8C8B    B9 488E7E00     mov     ecx, 007E8E48                     ; ASCII "TestHint"
007E8C90    8B83 78030000   mov     eax, dword ptr [ebx+378]
007E8C96    8B38            mov     edi, dword ptr [eax]
007E8C98    FF17            call    dword ptr [edi]
007E8C9A    8B45 E8         mov     eax, dword ptr [ebp-18]
007E8C9D    8975 E0         mov     dword ptr [ebp-20], esi
007E8CA0    C645 E4 00      mov     byte ptr [ebp-1C], 0
007E8CA4    8D55 E0         lea     edx, dword ptr [ebp-20]
007E8CA7    33C9            xor     ecx, ecx
007E8CA9    E8 0E4BC2FF     call    0040D7BC
007E8CAE    8B55 EC         mov     edx, dword ptr [ebp-14]
007E8CB1    8B83 48030000   mov     eax, dword ptr [ebx+348]
007E8CB7    E8 7C01CAFF     call    00488E38
007E8CBC    B2 01           mov     dl, 1
007E8CBE    8B83 38030000   mov     eax, dword ptr [ebx+338]
007E8CC4    8B08            mov     ecx, dword ptr [eax]
007E8CC6    FF51 64         call    dword ptr [ecx+64]
007E8CC9    33D2            xor     edx, edx
007E8CCB    8B83 34030000   mov     eax, dword ptr [ebx+334]
007E8CD1    8B08            mov     ecx, dword ptr [eax]
007E8CD3    FF51 64         call    dword ptr [ecx+64]
007E8CD6    8BC3            mov     eax, ebx
007E8CD8    E8 2F1E0000     call    007EAB0C
007E8CDD    8B83 3C030000   mov     eax, dword ptr [ebx+33C]
007E8CE3    C780 14020000 0 mov     dword ptr [eax+214], 2
007E8CED    33C0            xor     eax, eax
007E8CEF    5A              pop     edx
007E8CF0    59              pop     ecx
007E8CF1    59              pop     ecx
007E8CF2    64:8910         mov     dword ptr fs:[eax], edx
007E8CF5    68 1F8D7E00     push    007E8D1F
007E8CFA    8D45 E8         lea     eax, dword ptr [ebp-18]
007E8CFD    BA 04000000     mov     edx, 4
007E8D02    E8 0DC4C1FF     call    00405114
007E8D07    8D45 F8         lea     eax, dword ptr [ebp-8]
007E8D0A    E8 E1C3C1FF     call    004050F0
```
