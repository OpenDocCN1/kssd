
## 第三步破解:

```
/////////////////////////////////////////////////////////////////////////////////////////////

00439D91   |.  81EC 0C000000     sub esp,0C
00439D97   |.  6A 00             push 0
00439D99   |.  6A 00             push 0
00439D9B   |.  6A 00             push 0
00439D9D   |.  68 04000080       push 80000004
00439DA2   |.  6A 00             push 0
00439DA4   |.  68 250E4100       push 3_.00410E25                                 ;  ASCII "SOFTWARE\microsoft\windows help\test"
00439DA9   |.  68 01030080       push 80000301
00439DAE   |.  6A 00             push 0                                           ;  储存注册码的注册键
00439DB0   |.  68 03000000       push 3
00439DB5   |.  68 03000000       push 3
00439DBA   |.  BB 98060000       mov ebx,698
00439DBF   |.  E8 4F510000       call 3_.0043EF13                                 ;  假码
00439DC4   |.  83C4 28           add esp,28
00439DC7   |.  8945 FC           mov dword ptr ss:[ebp-4],eax
00439DCA   |.  8B1D FC03CA00     mov ebx,dword ptr ds:[CA03FC]
00439DD0   |.  85DB              test ebx,ebx
00439DD2   |.  74 09             je short 3_.00439DDD
00439DD4   |.  53                push ebx
00439DD5   |.  E8 2D510000       call 3_.0043EF07
00439DDA   |.  83C4 04           add esp,4
00439DDD   |>  8B45 FC           mov eax,dword ptr ss:[ebp-4]
00439DE0   |.  A3 FC03CA00       mov dword ptr ds:[CA03FC],eax
00439DE5   |.  E8 9F010000       call 3_.00439F89                                 ;  关键算法F7
00439DEA   |.  8945 FC           mov dword ptr ss:[ebp-4],eax                     ;  这里可以看到真码
00439DED   |.  8B1D 0004CA00     mov ebx,dword ptr ds:[CA0400]
00439DF3   |.  85DB              test ebx,ebx
00439DF5   |.  74 09             je short 3_.00439E00
00439DF7   |.  53                push ebx
00439DF8   |.  E8 0A510000       call 3_.0043EF07
00439DFD   |.  83C4 04           add esp,4
00439E00   |>  8B45 FC           mov eax,dword ptr ss:[ebp-4]
00439E03   |.  A3 0004CA00       mov dword ptr ds:[CA0400],eax
00439E08   |.  8B1D 0404CA00     mov ebx,dword ptr ds:[CA0404]
00439E0E   |.  85DB              test ebx,ebx
00439E10   |.  74 09             je short 3_.00439E1B
00439E12   |.  53                push ebx
00439E13   |.  E8 EF500000       call 3_.0043EF07
00439E18   |.  83C4 04           add esp,4
00439E1B   |>  B8 F6914000       mov eax,3_.004091F6
00439E20   |.  A3 0404CA00       mov dword ptr ds:[CA0404],eax
00439E25   |.  A1 0004CA00       mov eax,dword ptr ds:[CA0400]
00439E2A   |.  50                push eax
00439E2B   |.  FF35 FC03CA00     push dword ptr ds:[CA03FC]
00439E31   |.  E8 0337FEFF       call 3_.0041D539
00439E36   |.  83C4 08           add esp,8
00439E39   |.  83F8 00           cmp eax,0
00439E3C   |.  0F84 E6000000     je 3_.00439F28
00439E42   |.  68 02000080       push 80000002
00439E47   |.  6A 00             push 0
00439E49   |.  68 01000000       push 1
00439E4E   |.  6A 00             push 0
00439E50   |.  6A 00             push 0
00439E52   |.  6A 00             push 0
00439E54   |.  68 01000100       push 10001
00439E59   |.  68 07050106       push 6010507
00439E5E   |.  68 06050152       push 52010506
00439E63   |.  68 03000000       push 3
00439E68   |.  BB 20030000       mov ebx,320
00439E6D   |.  E8 A1500000       call 3_.0043EF13
00439E72   |.  83C4 28           add esp,28
00439E75   |.  68 40924000       push 3_.00409240                                 ;  UNICODE "0123456789"
00439E7A   |.  FF35 0404CA00     push dword ptr ds:[CA0404]
00439E80   |.  E8 B436FEFF       call 3_.0041D539
00439E85   |.  83C4 08           add esp,8
00439E88   |.  83F8 00           cmp eax,0
00439E8B   |.  0F85 23000000     jnz 3_.00439EB4
00439E91   |.  68 42924000       push 3_.00409242                                 ;  UNICODE "123456789"
00439E96   |.  FF35 0404CA00     push dword ptr ds:[CA0404]
00439E9C   |.  E8 9836FEFF       call 3_.0041D539
00439EA1   |.  83C4 08           add esp,8
00439EA4   |.  83F8 00           cmp eax,0
00439EA7   |.  0F84 07000000     je 3_.00439EB4
00439EAD   |.  B8 01000000       mov eax,1
00439EB2   |.  EB 02             jmp short 3_.00439EB6
00439EB4   |>  33C0              xor eax,eax
00439EB6   |>  85C0              test eax,eax
00439EB8   |.  0F84 3A000000     je 3_.00439EF8
00439EBE   |.  6A 00             push 0
00439EC0   |.  6A 00             push 0
00439EC2   |.  6A 00             push 0
00439EC4   |.  68 01030080       push 80000301
00439EC9   |.  6A 00             push 0
00439ECB   |.  68 10000000       push 10
00439ED0   |.  68 04000080       push 80000004
00439ED5   |.  6A 00             push 0
00439ED7   |.  68 4A0E4100       push 3_.00410E4A
00439EDC   |.  68 03000000       push 3
00439EE1   |.  BB 00030000       mov ebx,300
00439EE6   |.  E8 28500000       call 3_.0043EF13
00439EEB   |.  83C4 28           add esp,28
00439EEE   |.  6A 00             push 0
00439EF0   |.  E8 FA4F0000       call 3_.0043EEEF
00439EF5   |.  83C4 04           add esp,4
00439EF8   |>  6A 00             push 0
00439EFA   |.  6A 00             push 0
00439EFC   |.  6A 00             push 0
00439EFE   |.  68 01030080       push 80000301
00439F03   |.  6A 00             push 0
00439F05   |.  68 00000000       push 0
00439F0A   |.  68 04000080       push 80000004
00439F0F   |.  6A 00             push 0
00439F11   |.  68 580E4100       push 3_.00410E58
00439F16   |.  68 03000000       push 3
00439F1B   |.  BB 00030000       mov ebx,300
00439F20   |.  E8 EE4F0000       call 3_.0043EF13
00439F25   |.  83C4 28           add esp,28
00439F28   |>  68 01030080       push 80000301
00439F2D   |.  6A 00             push 0
00439F2F   |.  68 03000000       push 3
00439F34   |.  68 01000000       push 1
00439F39   |.  BB 00000000       mov ebx,0
00439F3E   |.  B8 01000000       mov eax,1
00439F43   |.  E8 E34F0000       call 3_.0043EF2B
00439F48   |.  83C4 10           add esp,10
00439F4B   |.  6A 00             push 0
00439F4D   |.  68 00000000       push 0
00439F52   |.  6A FF             push -1
00439F54   |.  6A 13             push 13
00439F56   |.  68 E6000116       push 160100E6
00439F5B   |.  68 01000152       push 52010001
00439F60   |.  E8 C04F0000       call 3_.0043EF25
00439F65   |.  83C4 18           add esp,18
00439F68   |.  6A 00             push 0
00439F6A   |.  68 00000000       push 0
00439F6F   |.  6A FF             push -1
00439F71   |.  6A 13             push 13
00439F73   |.  68 E8000116       push 160100E8
00439F78   |.  68 01000152       push 52010001
00439F7D   |.  E8 A34F0000       call 3_.0043EF25
00439F82   |.  83C4 18           add esp,18
00439F85   |.  8BE5              mov esp,ebp

==========================================================================================

00439F89   /$  55                push ebp
00439F8A   |.  8BEC              mov ebp,esp
00439F8C   |.  81EC 34000000     sub esp,34
00439F92   |.  68 04000000       push 4
00439F97   |.  E8 714F0000       call 3_.0043EF0D
00439F9C   |.  83C4 04           add esp,4
00439F9F   |.  8945 FC           mov dword ptr ss:[ebp-4],eax
00439FA2   |.  8BD8              mov ebx,eax
00439FA4   |.  C703 00000000     mov dword ptr ds:[ebx],0
00439FAA   |.  68 0A000600       push 6000A
00439FAF   |.  6A 00             push 0
00439FB1   |.  53                push ebx
00439FB2   |.  6A 01             push 1
00439FB4   |.  BB F8000000       mov ebx,0F8
00439FB9   |.  B8 05000000       mov eax,5
00439FBE   |.  E8 684F0000       call 3_.0043EF2B
00439FC3   |.  83C4 10           add esp,10
00439FC6   |.  68 04000000       push 4
00439FCB   |.  E8 3D4F0000       call 3_.0043EF0D
00439FD0   |.  83C4 04           add esp,4
00439FD3   |.  8945 F8           mov dword ptr ss:[ebp-8],eax
00439FD6   |.  8BD8              mov ebx,eax
00439FD8   |.  C703 00000000     mov dword ptr ds:[ebx],0
00439FDE   |.  68 0A000600       push 6000A
```
