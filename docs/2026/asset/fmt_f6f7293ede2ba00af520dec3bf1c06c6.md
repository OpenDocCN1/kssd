
``` 
0048DBD3  |.  BA B4DC4800      mov     edx, Easyresu.0048DCB4                ;  ASCII "five"
0048DBD8  |.  E8 3F6BF7FF      call    Easyresu.0040471C
0048DBDD  |>  837D F8 00       cmp     dword ptr ss:[ebp-8], 0
0048DBE1  |.  76 0B            jbe     short Easyresu.0048DBEE
0048DBE3  |.  8D45 E0          lea     eax, dword ptr ss:[ebp-20]
0048DBE6  |.  8B55 EC          mov     edx, dword ptr ss:[ebp-14]
0048DBE9  |.  E8 726BF7FF      call    Easyresu.00404760
0048DBEE  |>  8B45 EC          mov     eax, dword ptr ss:[ebp-14]
0048DBF1  |.  8B55 F4          mov     edx, dword ptr ss:[ebp-C]
0048DBF4  |.  E8 CB6EF7FF      call    Easyresu.00404AC4
0048DBF9  |.  75 0D            jnz     short Easyresu.0048DC08
0048DBFB  |.  8D45 F8          lea     eax, dword ptr ss:[ebp-8]
0048DBFE  |.  BA C4DC4800      mov     edx, Easyresu.0048DCC4                ;  ASCII "88"
0048DC03  |.  E8 586BF7FF      call    Easyresu.00404760
0048DC08  |>  8B45 F0          mov     eax, dword ptr ss:[ebp-10]
0048DC0B  |.  8B55 F4          mov     edx, dword ptr ss:[ebp-C]
0048DC0E  |.  E8 B16EF7FF      call    Easyresu.00404AC4
0048DC13  |.  75 21            jnz     short Easyresu.0048DC36
0048DC15  |.  B8 8C2D4900      mov     eax, Easyresu.00492D8C
0048DC1A  |.  BA D0DC4800      mov     edx, Easyresu.0048DCD0                ;  ASCII "etion"
0048DC1F  |.  E8 F86AF7FF      call    Easyresu.0040471C
0048DC24  |.  33C0             xor     eax, eax
0048DC26  |.  8905 A02D4900    mov     dword ptr ds:[492DA0], eax
0048DC2C  |.  C705 A42D4900 00>mov     dword ptr ds:[492DA4], 3FF00000
0048DC36  |>  33C0             xor     eax, eax
0048DC38  |.  5A               pop     edx                                   ;  0012FDBC
0048DC39  |.  59               pop     ecx                                   ;  0012FDBC
0048DC3A  |.  59               pop     ecx                                   ;  0012FDBC
0048DC3B  |.  64:8910          mov     dword ptr fs:[eax], edx
0048DC3E  |.  68 58DC4800      push    Easyresu.0048DC58
0048DC43  |>  8D45 D8          lea     eax, dword ptr ss:[ebp-28]
0048DC46  |.  BA 0A000000      mov     edx, 0A
0048DC4B  |.  E8 9C6AF7FF      call    Easyresu.004046EC
0048DC50  .  C3               retn
```

`12345678----CALL--->12316811` 取其7位值与后段取的7位值比较 `2316181---->对应的值2311681`   ---合起来---->`123456782311681`

----------------------------------------------------------------------------------------------

## 保证主程序不会退出的循环

```
00459A25   >  33C0             xor     eax, eax
00459A27   . |55               push    ebp
00459A28   . |68 459A4500      push    Easyresu.00459A45
00459A2D   . |64:FF30          push    dword ptr fs:[eax]
00459A30   . |64:8920          mov     dword ptr fs:[eax], esp
00459A33   . |8B45 FC          mov     eax, dword ptr ss:[ebp-4]
00459A36   . |E8 D1FDFFFF      call    Easyresu.0045980C
00459A3B   . |33C0             xor     eax, eax
00459A3D   . |5A               pop     edx                                   ;  0012FFB4
00459A3E   . |59               pop     ecx                                   ;  0012FFB4
00459A3F   . |59               pop     ecx                                   ;  0012FFB4
00459A40   . |64:8910          mov     dword ptr fs:[eax], edx
00459A43   . |EB 15            jmp     short Easyresu.00459A5A
00459A45   .^|E9 F2A3FAFF      jmp     Easyresu.00403E3C
00459A4A   . |8B55 FC          mov     edx, dword ptr ss:[ebp-4]
00459A4D   . |8B45 FC          mov     eax, dword ptr ss:[ebp-4]
00459A50   . |E8 4B000000      call    Easyresu.00459AA0
00459A55   . |E8 4AA7FAFF      call    Easyresu.004041A4
00459A5A   > |8B45 FC          mov     eax, dword ptr ss:[ebp-4]
00459A5D   . |80B8 9C000000 00 cmp     byte ptr ds:[eax+9C], 0
00459A64   .^74 BF            je      short Easyresu.00459A25
```
