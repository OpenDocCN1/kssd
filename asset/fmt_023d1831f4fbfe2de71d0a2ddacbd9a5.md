
```
00506BD5   |.  B3 01               mov bl,1  

00506BD7   |>  33C0               xor eax,eax  

00506BD9   |.  5A                pop edx  

00506BDA   |.  59                pop ecx  

00506BDB   |.  59                pop ecx  

00506BDC   |.  64:8910          mov dword ptr fs:[eax],edx  

00506BDF   |.  68 016C5000      push WinSSM_e.00506C01  

00506BE4   |>  8D45 F0          lea eax,dword ptr ss:[ebp-10]  

00506BE7   |.  BA 04000000      mov edx,4  

00506BEC   |.  E8 AFDDEFFF      call WinSSM_e.004049A0  

00506BF1   |.  8D45 08          lea eax,dword ptr ss:[ebp+8]  

00506BF4   |.  E8 83DDEFFF      call WinSSM_e.0040497C  

00506BF9   \.  C3               retn  

———————————————————————————————————————————  

call WinSSM_e.00506754  

———————————————————————————————————————————  

00506754   /$  55               push ebp  

00506755   |.  8BEC             mov ebp,esp  

00506757   |.  83C4 E0          add esp,-20  

0050675A   |.  53               push ebx  

0050675B   |.  56               push esi  

0050675C   |.  57               push edi  

0050675D   |.  33DB             xor ebx,ebx  

0050675F   |.  895D EC          mov dword ptr ss:[ebp-14],ebx                 ;  试炼码  

00506762   |.  895D E8          mov dword ptr ss:[ebp-18],ebx  

00506765   |.  895D E4          mov dword ptr ss:[ebp-1C],ebx  

00506768   |.  895D E0          mov dword ptr ss:[ebp-20],ebx  

0050676B   |.  8BF9             mov edi,ecx  

0050676D   |.  8955 FC          mov dword ptr ss:[ebp-4],edx                ;  注册姓名  

00506770   |.  8BF0             mov esi,eax  

00506772   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506775   |.  E8 BEE6EFFF      call WinSSM_e.00404E38  

0050677A   |.  33C0             xor eax,eax  

0050677C   |.  55               push ebp  

0050677D   |.  68 ED685000      push WinSSM_e.005068ED  

00506782   |.  64:FF30          push dword ptr fs:[eax]  

00506785   |.  64:8920          mov dword ptr fs:[eax],esp  

00506788   |.  8D45 EC          lea eax,dword ptr ss:[ebp-14]  

0050678B   |.  BA 04695000      mov edx,WinSSM_e.00506904                   ;  ASCII "gf258369gf"  

00506790   |.  E8 7FE2EFFF      call WinSSM_e.00404A14  

00506795   |.  837D EC 00       cmp dword ptr ss:[ebp-14],0  

00506799   |.  75 0D            jnz short WinSSM_e.005067A8  

0050679B   |.  8D45 E0          lea eax,dword ptr ss:[ebp-20]  

0050679E   |.  8B55 FC          mov edx,dword ptr ss:[ebp-4]  

005067A1   |.  E8 6EE2EFFF      call WinSSM_e.00404A14  

005067A6   |.  EB 5D            jmp short WinSSM_e.00506805  

005067A8   |>  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067AB   |.  E8 A0E4EFFF      call WinSSM_e.00404C50  

005067B0   |.  8BD8             mov ebx,eax  

005067B2   |.  8D45 E8          lea eax,dword ptr ss:[ebp-18]  

005067B5   |.  50               push eax  

005067B6   |.  8BCB             mov ecx,ebx  

005067B8   |.  D1F9             sar ecx,1  

005067BA   |.  79 03            jns short WinSSM_e.005067BF  

005067BC   |.  83D1 00          adc ecx,0  

005067BF   |>  BA 01000000      mov edx,1  

005067C4   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067C7   |.  E8 DCE6EFFF      call WinSSM_e.00404EA8  

005067CC   |.  8D45 E4          lea eax,dword ptr ss:[ebp-1C]               ;  取'gf258369gf'前 5 位  

005067CF   |.  50               push eax  

005067D0   |.  8BC3             mov eax,ebx  

005067D2   |.  D1F8             sar eax,1  

005067D4   |.  79 03            jns short WinSSM_e.005067D9  

005067D6   |.  83D0 00          adc eax,0  

005067D9   |>  8BCB             mov ecx,ebx  

005067DB   |.  2BC8             sub ecx,eax  

005067DD   |.  8BD3             mov edx,ebx  

005067DF   |.  D1FA             sar edx,1  

005067E1   |.  79 03            jns short WinSSM_e.005067E6  

005067E3   |.  83D2 00          adc edx,0  

005067E6   |>  42               inc edx  

005067E7   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067EA   |.  E8 B9E6EFFF      call WinSSM_e.00404EA8  

005067EF   |.  FF75 E8          push dword ptr ss:[ebp-18]                 ;  前 5 位  

005067F2   |.  FF75 FC          push dword ptr ss:[ebp-4]                  ;  注册姓名  

005067F5   |.  FF75 E4          push dword ptr ss:[ebp-1C]                 ;  后 5 位  

005067F8   |.  8D45 E0          lea eax,dword ptr ss:[ebp-20]  

005067FB   |.  BA 03000000      mov edx,3  

00506800   |.  E8 0BE5EFFF      call WinSSM_e.00404D10  

00506805   |>  C745 F0 00000000 mov dword ptr ss:[ebp-10],0  

0050680C   |.  C745 F4 00000000 mov dword ptr ss:[ebp-C],0  

00506813   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506816   |.  E8 35E4EFFF      call WinSSM_e.00404C50  

0050681B   |.  3B46 4C          cmp eax,dword ptr ds:[esi+4C]  

0050681E   |.  7F 0D            jg short WinSSM_e.0050682D  

00506820   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506823   |.  E8 28E4EFFF      call WinSSM_e.00404C50  

00506828   |.  3B46 50          cmp eax,dword ptr ds:[esi+50]  

0050682B   |.  7D 0C            jge short WinSSM_e.00506839  

0050682D   |>  8BC7             mov eax,edi  

0050682F   |.  E8 48E1EFFF      call WinSSM_e.0040497C  

00506834   |.  E9 91000000      jmp WinSSM_e.005068CA  

00506839   |>  8B45 E0          mov eax,dword ptr ss:[ebp-20]              ;  Name=gf258+注册姓名+369gf  

0050683C   |.  E8 0FE4EFFF      call WinSSM_e.00404C50  

00506841   |.  8BD8             mov ebx,eax                                ;  求 Name 长度  

00506843   |.  EB 37            jmp short WinSSM_e.0050687C  

00506845   |>  8B46 68          /mov eax,dword ptr ds:[esi+68]             ;  常数 1B382F6B 记为 Key  

00506848   |.  8B56 6C          |mov edx,dword ptr ds:[esi+6C]             ;  0  

0050684B   |.  0345 F0          |add eax,dword ptr ss:[ebp-10]             ;  Key+Serail  

0050684E   |.  1355 F4          |adc edx,dword ptr ss:[ebp-C]              ;  带进位加法  

00506851   |.  52               |push edx                                   ;  入栈  

00506852   |.  50               |push eax  

00506853   |.  8B45 E0          |mov eax,dword ptr ss:[ebp-20]              ;  Name=gf258+注册姓名+369gf  

00506856   |.  0FB64418 FF      |movzx eax,byte ptr ds:[eax+ebx-1]          ;  取字符 Name[i]  

0050685B   |.  50               |push eax  

0050685C   |.  B8 59040000      |mov eax,459                                ;  常数 459  

00506861   |.  5A               |pop edx                                    ;  Name[i]  

00506862   |.  8BCA             |mov ecx,edx  

00506864   |.  33D2             |xor edx,edx  

00506866   |.  F7F1             |div ecx                                    ;  459/Name[i]  

00506868       8BC2             mov eax,edx                                 ;  取余  

0050686A   |.  33D2             |xor edx,edx  

0050686C   |.  290424           |sub dword ptr ss:[esp],eax                 ;  Key+Serail-459/Name[i]取余  

0050686F   |.  195424 04        |sbb dword ptr ss:[esp+4],edx  

00506873   |.  58               |pop eax  

00506874   |.  5A               |pop edx  

00506875   |.  8945 F0          |mov dword ptr ss:[ebp-10],eax              ;  Key+Serail-459/Name[i]取余  

00506878   |.  8955 F4          |mov dword ptr ss:[ebp-C],edx  

0050687B   |.  4B               |dec ebx                                    ;  i=i-1  

0050687C   |>  8B45 E0          mov eax,dword ptr ss:[ebp-20]              ;  fg258+注册姓名+369gf  

0050687F   |.  E8 CCE3EFFF      |call WinSSM_e.00404C50                    ;  求 Name 长度
```
