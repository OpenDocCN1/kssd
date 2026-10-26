
```asm
004E0D3F  |.  B9 200F4E00   mov ecx,OfficeMa.004E0F20                ;  regnosec
004E0D44  |.  BA 000F4E00   mov edx,OfficeMa.004E0F00                ;  reg
004E0D49  |.  8B30          mov esi,dword ptr ds:[eax]
004E0D4B  |.  FF56 04       call dword ptr ds:[esi+4]
004E0D4E  |.  6A 00         push 0
004E0D50  |.  8B15 F4885700 mov edx,dword ptr ds:[5788F4]            ;  OfficeMa.0057B694
004E0D56  |.  8B12          mov edx,dword ptr ds:[edx]
004E0D58  |.  8D45 B8       lea eax,dword ptr ss:[ebp-48]
004E0D5B  |.  B9 340F4E00   mov ecx,OfficeMa.004E0F34                ;  信息库文件\xxklist.edo
004E0D60  |.  E8 2F41F2FF   call OfficeMa.00404E94
004E0D65  |.  8B55 B8       mov edx,dword ptr ss:[ebp-48]
004E0D68  |.  A1 F08C5700   mov eax,dword ptr ds:[578CF0]
004E0D6D  |.  8B00          mov eax,dword ptr ds:[eax]
004E0D6F  |.  B9 10270000   mov ecx,2710
004E0D74  |.  E8 FF9BFCFF   call OfficeMa.004AA978
004E0D79  |.  A1 5C8D5700   mov eax,dword ptr ds:[578D5C]
004E0D7E  |.  8338 00       cmp dword ptr ds:[eax],0
004E0D81  |.  74 0C         je short OfficeMa.004E0D8F
004E0D83  |.  A1 5C8D5700   mov eax,dword ptr ds:[578D5C]
004E0D88  |.  8B00          mov eax,dword ptr ds:[eax]
004E0D8A  |.  E8 B5BAF8FF   call OfficeMa.0046C844
004E0D8F  |>  A1 F88A5700   mov eax,dword ptr ds:[578AF8]
004E0D94  |.  8B00          mov eax,dword ptr ds:[eax]
004E0D96  |.  E8 41F2F8FF   call OfficeMa.0046FFDC
004E0D9B  |.  6A 01         push 1
004E0D9D  |.  6A 00         push 0
004E0D9F  |.  6A 00         push 0
004E0DA1  |.  68 4C0F4E00   push OfficeMa.004E0F4C                   ;  endo.exe
004E0DA6  |.  68 580F4E00   push OfficeMa.004E0F58                   ;  open
004E0DAB  |.  8BC3          mov eax,ebx
004E0DAD  |.  E8 1A4CF7FF   call OfficeMa.004559CC
004E0DB2  |.  50            push eax                                 ; |hWnd
004E0DB3  |.  E8 A8BAF5FF   call <jmp.&shell32.ShellExecuteA>        ; \ShellExecuteA
004E0DB8  |.  EB 27         jmp short OfficeMa.004E0DE1
004E0DBA  |>  6A 00         push 0
004E0DBC  |.  68 600F4E00   push OfficeMa.004E0F60                   ;  警告
004E0DC1  |.  68 680F4E00   push OfficeMa.004E0F68                   ;  注册码错误!请重新输入!
```

***

跟进算法`call OfficeMa.004E1334`，来到这里

```asm
004E1334  /$  55            push ebp
004E1335  |.  8BEC          mov ebp,esp
004E1337  |.  81C4 1CFFFFFF add esp,-0E4
004E133D  |.  53            push ebx
004E133E  |.  56            push esi
004E133F  |.  33DB          xor ebx,ebx
004E1341  |.  899D 1CFFFFFF mov dword ptr ss:[ebp-E4],ebx
004E1347  |.  899D 20FFFFFF mov dword ptr ss:[ebp-E0],ebx
004E134D  |.  894D F4       mov dword ptr ss:[ebp-C],ecx             ;  ebp-C=987654321
004E1350  |.  8955 F8       mov dword ptr ss:[ebp-8],edx             ;  ebp-8=123456789
004E1353  |.  8945 FC       mov dword ptr ss:[ebp-4],eax             ;  ebp-4=机器码
004E1356  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
004E1359  |.  E8 D23CF2FF   call OfficeMa.00405030
004E135E  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8]
004E1361  |.  E8 CA3CF2FF   call OfficeMa.00405030
004E1366  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]
004E1369  |.  E8 C23CF2FF   call OfficeMa.00405030                   ;  eax=987654321
004E136E  |.  33C0          xor eax,eax                              ;  eax清零
004E1370  |.  55            push ebp
004E1371  |.  68 65144E00   push OfficeMa.004E1465
004E1376  |.  64:FF30       push dword ptr fs:[eax]
004E1379  |.  64:8920       mov dword ptr fs:[eax],esp
004E137C  |.  C645 F3 00    mov byte ptr ss:[ebp-D],0
004E1380  |.  33F6          xor esi,esi
004E1382  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]             ;  机器码送eax
004E1385  |.  E8 BE3AF2FF   call OfficeMa.00404E48
004E138A  |.  8BD0          mov edx,eax
004E138C  |.  85D2          test edx,edx
004E138E  |.  7E 2A         jle short OfficeMa.004E13BA
004E1390  |.  B9 01000000   mov ecx,1                                ;  ecx=987654321
004E1395  |.  8D85 24FFFFFF lea eax,dword ptr ss:[ebp-DC]
004E139B  |>  8B5D FC       /mov ebx,dword ptr ss:[ebp-4]
004E139E  |.  0FB65C0B FF   |movzx ebx,byte ptr ds:[ebx+ecx-1]       ;  机器码的ascii码逐位送ebx
004E13A3  |.  8918          |mov dword ptr ds:[eax],ebx              ;  ebx送ds:[eax]
004E13A5  |.  6918 39590500 |imul ebx,dword ptr ds:[eax],55939       ;  ebx*55939
004E13AB  |.  03F3          |add esi,ebx                             ;  esi+ebx
004E13AD  |.  81C6 16930600 |add esi,69316                           ;  esi+69316
004E13B3  |.  41            |inc ecx                                 ;  ecx+1
004E13B4  |.  83C0 04       |add eax,4                               ;  eax+4
004E13B7  |.  4A            |dec edx                                 ;  edx-1
004E13B8  |.^ 75 E1         \jnz short OfficeMa.004E139B             ;  循环15次，即机器码位数
004E13BA  |>  33C0          xor eax,eax                              ;  最后循环计算出来的esi值为1375D293，转成十进制为326488723，即第一部分注册码
004E13BC  |.  8945 EC       mov dword ptr ss:[ebp-14],eax
004E13BF  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]             ;  机器码送eax
004E13C2  |.  E8 813AF2FF   call OfficeMa.00404E48
004E13C7  |.  8BD0          mov edx,eax                              ;  eax=机器码位数
004E13C9  |.  85D2          test edx,edx
004E13CB  |.  7E 2E         jle short OfficeMa.004E13FB
004E13CD  |.  B9 01000000   mov ecx,1
004E13D2  |.  8D85 24FFFFFF lea eax,dword ptr ss:[ebp-DC]
004E13D8  |>  8B5D FC       /mov ebx,dword ptr ss:[ebp-4]
004E13DB  |.  0FB65C0B FF   |movzx ebx,byte ptr ds:[ebx+ecx-1]       ;  机器码的ascii码逐位送ebx
004E13E0  |.  8918          |mov dword ptr ds:[eax],ebx
004E13E2  |.  6918 49860C00 |imul ebx,dword ptr ds:[eax],0C8649      ;  ebx*0C8649
004E13E8  |.  035D EC       |add ebx,dword ptr ss:[ebp-14]           ;  ebx+ss:[ebp-14]
004E13EB  |.  81C3 16930600 |add ebx,69316                           ;  ebx+69316
004E13F1  |.  895D EC       |mov dword ptr ss:[ebp-14],ebx           ;  ebx送ss:[ebp-14]
004E13F4  |.  41            |inc ecx                                 ;  ecx+1
004E13F5  |.  83C0 04       |add eax,4                               ;  eax+4
004E13F8  |.  4A            |dec edx                                 ;  edx-1
004E13F9  |.^ 75 DD         \jnz short OfficeMa.004E13D8             ;  循环15次，即机器码位数
004E13FB  |>  8D95 20FFFFFF lea edx,dword ptr ss:[ebp-E0]            ;  最后循环计算出来的ebx值为2D0D88A3，转成十进制为755861667，即第二部分注册码
004E1401  |.  8BC6          mov eax,esi
004E1403  |.  E8 7486F2FF   call OfficeMa.00409A7C
004E1408  |.  8B95 20FFFFFF mov edx,dword ptr ss:[ebp-E0]
004E140E  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8]
004E1411  |.  E8 763BF2FF   call OfficeMa.00404F8C
004E1416  |.  75 22         jnz short OfficeMa.004E143A              ;  判断第一部分注册码是否正确，爆破点1
004E1418  |.  8D95 1CFFFFFF lea edx,dword ptr ss:[ebp-E4]
004E141E  |.  8B45 EC       mov eax,dword ptr ss:[ebp-14]
004E1421  |.  E8 5686F2FF   call OfficeMa.00409A7C
004E1426  |.  8B95 1CFFFFFF mov edx,dword ptr ss:[ebp-E4]
004E142C  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]
004E142F  |.  E8 583BF2FF   call OfficeMa.00404F8C
004E1434  |.  75 04         jnz short OfficeMa.004E143A              ;  判断第二部分注册码是否正确，爆破点2
004E1436  |.  C645 F3 01    mov byte ptr ss:[ebp-D],1
004E143A  |>  33C0          xor eax,eax
004E143C  |.  5A            pop edx
004E143D  |.  59            pop ecx
004E143E  |.  59            pop ecx
004E143F  |.  64:8910       mov dword ptr fs:[eax],edx
004E1442  |.  68 6C144E00   push OfficeMa.004E146C
004E1447  |>  8D85 1CFFFFFF lea eax,dword ptr ss:[ebp-E4]
004E144D  |.  BA 02000000   mov edx,2
004E1452  |.  E8 5D37F2FF   call OfficeMa.00404BB4
004E1457  |.  8D45 F4       lea eax,dword ptr ss:[ebp-C]
004E145A  |.  BA 03000000   mov edx,3
```
