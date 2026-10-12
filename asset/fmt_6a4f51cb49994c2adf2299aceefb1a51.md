
```
00552323      |.  8915 00BF5D00       mov dword ptr ds:[5DBF00],edx                      ;  ntdll.KiFastSystemCallRet
00552329      |.  8BC8                mov ecx,eax
0055232B      |.  81E1 FF000000       and ecx,0FF
00552331      |.  890D FCBE5D00       mov dword ptr ds:[5DBEFC],ecx
00552337      |.  C1E1 08             shl ecx,8
0055233A      |.  03CA                add ecx,edx                                      ;  ntdll.KiFastSystemCallRet
0055233C      |.  890D F8BE5D00       mov dword ptr ds:[5DBEF8],ecx
00552342      |.  C1E8 10             shr eax,10
00552345      |.  A3 F4BE5D00         mov dword ptr ds:[5DBEF4],eax
0055234A      |.  6A 01               push 1
0055234C      |.  E8 391D0000         call Babylon.0055408A
00552351      |.  59                  pop ecx                                          ;  kernel32.7C816FD7
00552352      |.  85C0                test eax,eax
00552354      |.  75 08               jnz short Babylon.0055235E
00552356      |.  6A 1C               push 1C
00552358      |.  E8 C3000000         call Babylon.00552420
0055235D      |.  59                  pop ecx                                          ;  kernel32.7C816FD7
0055235E      |>  E8 99350000         call Babylon.005558FC
00552363      |.  85C0                test eax,eax
00552365      |.  75 08               jnz short Babylon.0055236F
00552367      |.  6A 10               push 10
00552369      |.  E8 B2000000         call Babylon.00552420
0055236E      |.  59                  pop ecx                                          ;  kernel32.7C816FD7
0055236F      |>  33F6                xor esi,esi
00552371      |.  8975 FC             mov dword ptr ss:[ebp-4],esi
00552374      |.  E8 1B840000         call Babylon.0055A794
00552379      |.  FF15 0C625700       call near dword ptr ds:[<&KERNEL32.GetCommandLineA>] ; [GetCommandLineA
0055237F      |.  A3 08D85D00         mov dword ptr ds:[5DD808],eax
00552384      |.  E8 1B8C0000         call Babylon.0055AFA4
00552389      |.  A3 7CBE5D00         mov dword ptr ds:[5DBE7C],eax
0055238E      |.  E8 C4890000         call Babylon.0055AD57
00552393      |.  E8 06890000         call Babylon.0055AC9E
00552398      |.  E8 A7380000         call Babylon.00555C44
0055239D      |.  8975 D0             mov dword ptr ss:[ebp-30],esi
005523A0      |.  8D45 A4             lea eax,dword ptr ss:[ebp-5C]
005523A3      |.  50                  push eax                                         ; /pStartupinfo = NULL
005523A4      |.  FF15 18625700       call near dword ptr ds:[<&KERNEL32.GetStartupInfoA>] ; \GetStartupInfoA
005523AA      |.  E8 97880000         call Babylon.0055AC46
005523AF      |.  8945 9C             mov dword ptr ss:[ebp-64],eax
005523B2      |.  F645 D0 01          test byte ptr ss:[ebp-30],1
005523B6      |.  74 06               je short Babylon.005523BE
005523B8      |.  0FB745 D4           movzx eax,word ptr ss:[ebp-2C]
005523BC      |.  EB 03               jmp short Babylon.005523C1
005523BE      |>  6A 0A               push 0A
005523C0      |.  58                  pop eax                                          ;  kernel32.7C816FD7
005523C1      |>  50                  push eax                                         ; /Arg4 = 00000000
005523C2      |.  FF75 9C             push dword ptr ss:[ebp-64]                       ; |Arg3 = 00000001
005523C5      |.  56                  push esi                                         ; |Arg2 = FFFFFFFF
005523C6      |.  56                  push esi                                         ; |/pModule = FFFFFFFF ???
005523C7      |.  FF15 68645700       call near dword ptr ds:[<&KERNEL32.GetModuleHandleA>] ; |\GetModuleHandleA
005523CD      |.  50                  push eax                                         ; |Arg1 = 00000000
005523CE      |.  E8 2DC0EBFF         call Babylon.0040E400                            ; \Babylon.0040E400
005523D3      |.  8945 A0             mov dword ptr ss:[ebp-60],eax
005523D6      |.  50                  push eax
005523D7      |.  E8 95380000         call Babylon.00555C71
005523DC      |.  8B45 EC             mov eax,dword ptr ss:[ebp-14]
005523DF      |.  8B08                mov ecx,dword ptr ds:[eax]
005523E1      |.  8B09                mov ecx,dword ptr ds:[ecx]                       ;  ntdll.7C92E64E
005523E3      |.  894D 98             mov dword ptr ss:[ebp-68],ecx
005523E6      |.  50                  push eax
005523E7      |.  51                  push ecx
005523E8      |.  E8 68550000         call Babylon.00557955
005523ED      |.  59                  pop ecx                                          ;  kernel32.7C816FD7
005523EE      |.  59                  pop ecx                                          ;  kernel32.7C816FD7
005523EF      \.  C3                  retn

55 8B EC 6A FF 68 58 23 58 00 68 6C 5B 55 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58
53 56 57 89 65 E8 FF 15 00 63 57 00 33 D2 8A D4 89 15 00 BF 5D 00 8B C8 81 E1 FF 00 00 00 89 0D
FC BE 5D 00 C1 E1 08 03 CA 89 0D F8 BE 5D 00 C1 E8 10 A3 F4 BE 5D 00 6A 01 E8 39 1D 00 00 59 85
C0 75 08 6A 1C E8 C3 00 00 00 59 E8 99 35 00 00 85 C0 75 08 6A 10 E8 B2 00 00 00 59 33 F6 89 75
FC E8 1B 84 00 00 FF 15 0C 62 57 00 A3 08 D8 5D 00 E8 1B 8C 00 00 A3 7C BE 5D 00 E8 C4 89 00 00
E8 06 89 00 00 E8 A7 38 00 00 89 75 D0 8D 45 A4 50 FF 15 18 62 57 00 E8 97 88 00 00 89 45 9C F6
45 D0 01 74 06 0F B7 45 D4 EB 03 6A 0A 58 50 FF 75 9C 56 56 FF 15 68 64 57 00 50 E8 2D C0 EB FF
89 45 A0 50 E8 95 38 00 00 8B 45 EC 8B 08 8B 09 89 4D 98 50 51 E8 68 55 00 00 59 59 C3

****************************************************************

Microsoft Visual C++ 6.0

.text

.rdata

.data

.tls

.rsrc

004AD06E ACDS>/$  55                  push ebp
004AD06F      |.  8BEC                mov ebp,esp
004AD071      |.  6A FF               push -1
004AD073      |.  68 28014E00         push ACDSee.004E0128
004AD078      |.  68 9C0C4B00         push ACDSee.004B0C9C                           ;  SE handler installation
004AD07D      |.  64:A1 00000000      mov eax,dword ptr fs:[0]
004AD083      |.  50                  push eax
004AD084      |.  64:8925 00000000    mov dword ptr fs:[0],esp
004AD08B      |.  83EC 58             sub esp,58
004AD08E      |.  53                  push ebx
004AD08F      |.  56                  push esi
004AD090      |.  57                  push edi                                       ;  ntdll.7C930738
004AD091      |.  8965 E8             mov dword ptr ss:[ebp-18],esp
004AD094      |.  FF15 A8744D00       call near dword ptr ds:[<&KERNEL32.GetVersion>] ;  kernel32.GetVersion
004AD09A      |.  33D2                xor edx,edx                                    ;  ntdll.KiFastSystemCallRet
004AD09C      |.  8AD4                mov dl,ah
004AD09E      |.  8915 F4584F00       mov dword ptr ds:[4F58F4],edx                  ;  ntdll.KiFastSystemCallRet
004AD0A4      |.  8BC8                mov ecx,eax
004AD0A6      |.  81E1 FF000000       and ecx,0FF
004AD0AC      |.  890D F0584F00       mov dword ptr ds:[4F58F0],ecx
004AD0B2      |.  C1E1 08             shl ecx,8
```
