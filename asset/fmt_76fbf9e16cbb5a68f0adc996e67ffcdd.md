
`043EC990 |. 5B pop ebx`  

`043EC991 |. C9 leave`  

`043EC992 \. C2 0800 retn 8`  

跟进 `043EC947 |. FF53 24 call dword ptr [ebx+24]`：

`043ED50A /. 55 push ebp`  

`043ED50B |. 8BEC mov ebp， esp`  

`043ED50D |. 56 push esi`  

`043ED50E |. 57 push edi`  

`043ED50F |. 8B7D 0C mov edi， dword ptr [ebp+C]`  

`043ED512 |. 8BF1 mov esi， ecx`  

`043ED514 |. 66:837F 02 02 cmp word ptr [edi+2]， 2`  

`043ED519 0F85 08010000 jnz 043ED627`  

`043ED51F |. 0FBF07 movsx eax， word ptr [edi]`  

`043ED522 |. 83E8 7D sub eax， 7D ; ; Switch (cases 7D..AF)`  

`043ED525 0F84 E1000000 je 043ED60C`  

`043ED52B |. 83E8 03 sub eax， 3`  

`043ED52E |. 0F84 B8000000 je 043ED5EC`  

`043ED534 |. 48 dec eax`  

`043ED535 |. 0F84 91000000 je 043ED5CC`  

`043ED53B |. 48 dec eax`  

`043ED53C |. 48 dec eax`  

`043ED53D |. 74 70 je short 043ED5AF`  

`043ED53F |. 83E8 09 sub eax， 9`  

`043ED542 |. 74 4E je short 043ED592`  

`043ED544 |. 83E8 1D sub eax， 1D`  

`043ED547 |. 74 29 je short 043ED572`  

`043ED549 |. 83E8 06 sub eax， 6`  

`043ED54C |. 0F85 D5000000 jnz 043ED627`  

`043ED552 |. 8D46 14 lea eax， dword ptr [esi+14] ; ; Case AF of switch 043ED522`  

`043ED555 |. 68 E8DB4004 push 0440DBE8`  

`; ; ASCII "Receive notify game avatar event item message package"`  

`043ED55A |. 50 push eax`  

`043ED55B |. E8 49B50000 call 043F8AA9`  

`043ED560 |. 59 pop ecx`  

`043ED561 |. 59 pop ecx`  

`043ED562 |. FF75 08 push dword ptr [ebp+8] ; /Arg2`  

`043ED565 |. 8BCE mov ecx， esi ; |`  

`043ED567 |. 57 push edi ; |Arg1`  

`043ED568 |. E8 F9080000 call 043EDE66 ; \ScatProt.1000DE66`  

`043ED56D |. E9 B5000000 jmp 043ED627`  

`043ED572 |> 8D46 14 lea eax， dword ptr [esi+14] ; ; Case A9 of switch 043ED522`  

`043ED575 |. 68 B4DB4004 push 0440DBB4`  

`; ; ASCII "Receive notify update game avatar message package"`  

`043ED57A |. 50 push eax`  

`043ED57B |. E8 29B50000 call 043F8AA9`  

`043ED580 |. 59 pop ecx`  

`043ED581 |. 59 pop ecx`  

`043ED582 |. FF75 08 push dword ptr [ebp+8] ; /Arg2`  

`043ED585 |. 8BCE mov ecx， esi ; |`  

`043ED587 |. 57 push edi ; |Arg1`  

`043ED588 |. E8 FD070000 call 043EDD8A ; \ScatProt.1000DD8A`  

`043ED58D |. E9 95000000 jmp 043ED627`  

`043ED592 |> 8D46 14 lea eax， dword ptr [esi+14] ; ; Case 8C of switch 043ED522`  

`043ED595 |. 68 8CDB4004 push 0440DB8C`  

`; ; ASCII "Receive notify system message package"`  

`043ED59A |. 50 push eax`  

`043ED59B |. E8 09B50000 call 043F8AA9`  

`043ED5A0 |. 59 pop ecx`  

`043ED5A1 |. 59 pop ecx`  

`043ED5A2 |. FF75 08 push dword ptr [ebp+8] ; /Arg2`  

`043ED5A5 |. 8BCE mov ecx， esi ; |`  

`043ED5A7 |. 57 push edi ; |Arg1`  

`043ED5A8 |. E8 49070000 call 043EDCF6 ; \ScatProt.1000DCF6`  

`043ED5AD |. EB 78 jmp short 043ED627`  

`043ED5AF |> 8D46 14 lea eax， dword ptr [esi+14] ; ; Case 83 of switch 043ED522`  

`043ED5B2 |. 68 64DB4004 push 0440DB64`  

`; ; ASCII "Receive notify invite to play package"`  

`043ED5B7 |. 50 push eax`  

`043ED5B8 |. E8 ECB40000 call 043F8AA9`  

`043ED5BD |. 59 pop ecx`  

`043ED5BE |. 59 pop ecx`  

`043ED5BF |. FF75 08 push dword ptr [ebp+8] ; /Arg2`  

`043ED5C2 |. 8BCE mov ecx， esi ; |`  

`043ED5C4 |. 57 push edi ; |Arg1`  

`043ED5C5 |. E8 AE060000 call 043EDC78 ; \ScatProt.1000DC78`  

`043ED5CA |. EB 5B jmp short 043ED627`  

`043ED5CC |> 8D46 14 lea eax， dword ptr [esi+14] ; ; Case 81 of switch 043ED522`  

`043ED5CF |. 68 3CDB4004 push 0440DB3C`  

`; ; ASCII "Receive notify table message package"`  

`043ED5D4 |. 50 push eax`  

`043ED5D5 |. E8 CFB40000 call 043F8AA9`  

`043ED5DA |. 59 pop ecx`  

`043ED5DB |. 59 pop ecx`  

`043ED5DC |. FF75 14 push dword ptr [ebp+14]`  

`043ED5DF |. 8BCE mov ecx， esi`  

`043ED5E1 |. FF75 08 push dword ptr [ebp+8]`  

`043ED5E4 |. 57 push edi`  

`043ED5E5 |. E8 3D050000 call 043EDB27`  

`043ED5EA |. EB 3B jmp short 043ED627`  

`043ED5EC |> 8D46 14 lea eax， dword ptr [esi+14] ; ; Case 80 of switch 043ED522`  

`043ED5EF |. 68 18DB4004 push 0440DB18`  

`; ; ASCII "Receive notify room message package"`  

`043ED5F4 |. 50 push eax`  

`043ED5F5 |. E8 AFB40000 call 043F8AA9`  

`043ED5FA |. 59 pop ecx`  

`043ED5FB |. 59 pop ecx`  

`043ED5FC |. FF75 14 push dword ptr [ebp+14]`  

`043ED5FF |. 8BCE mov ecx， esi`  

`043ED601 |. FF75 08 push dword ptr [ebp+8]`  

`043ED604 |. 57 push edi`  

`043ED605 |. E8 D4030000 call 043ED9DE`  

`043ED60A |. EB 1B jmp short 043ED627`  

`043ED60C 8D46 14 lea eax， dword ptr [esi+14]`  

`043ED60F |. 68 F4DA4004 push 0440DAF4 ; ; ASCII "Receive notify room event package"`  

`043ED614 |. 50 push eax`  

`043ED615 |. E8 8FB40000 call 043F8AA9`  

`043ED61A |. 59 pop ecx`  

`043ED61B |. 59 pop ecx`  

`043ED61C |. FF75 08 push dword ptr [ebp+8]`  

`043ED61F |. 8BCE mov ecx， esi`  

`043ED621 |. 57 push edi`  

`043ED622 |. E8 45000000 call 043ED66C`  

`043ED627 |> 5F pop edi ; ; Default case of switch 043ED522`  

`043ED628 |. 5E pop esi`  

`043ED629 |. 5D pop ebp`  

`043ED62A \. C2 1400 retn 14`  

跟进 `043ED622 |. E8 45000000 call 043ED66C`：

`043ED66C /$ 55 push ebp`  

`043ED66D |. 8BEC mov ebp， esp`  

`043ED66F |. B8 9C510000 mov eax， 519C`  

`043ED674 |. E8 271F0100 call 043FF5A0`  

`043ED679 |. 53 push ebx`  

`043ED67A |. 56 push esi`  

`043ED67B |. 894D FC mov dword ptr [ebp-4]， ecx`  

`043ED67E |. 57 push edi`  

`043ED67F |. 33DB xor ebx， ebx`  

`043ED681 |. 8B75 0C mov esi， dword ptr [ebp+C]`  

`043ED684 |. B9 64140000 mov ecx， 1464`  

`043ED689 |. 33C0 xor eax， eax`  

`043ED68B |. 8DBD 66AEFFFF lea edi， dword ptr [ebp+FFFFAE66]`  

`043ED691 |. 66:899D 64AEF>mov word ptr [ebp+FFFFAE64]， bx`  

`043ED698 |. F3:AB rep stos dword ptr es:[edi]`  

`043ED69A |. 8D8D 64AEFFFF lea ecx， dword ptr [ebp+FFFFAE64]`  

`043ED6A0 |. 66:AB stos word ptr es:[edi]`  

`043ED6A2 |. 8B06 mov eax， dword ptr [esi]`  

`043ED6A4 |. 51 push ecx`  

`043ED6A5 |. 8BCE mov ecx， esi`  

`043ED6A7 |. FF50 1C call dword ptr [eax+1C]`  

`043ED6AA |. 8B06 mov eax， dword ptr [esi]`  

`043ED6AC |. 8D8D 66AEFFFF lea ecx， dword ptr [ebp+FFFFAE66]`  

`043ED6B2 |. 51 push ecx`  

`043ED6B3 |. 8BCE mov ecx， esi`  

`043ED6B5 |. FF50 1C call dword ptr [eax+1C]`  

`043ED6B8 |. 66:8B85 66AEF>mov ax， word ptr [ebp+FFFFAE66]`  

`043ED6BF |. 66:3D 5A00 cmp ax， 5A`  

`043ED6C3 |. 66:8985 66AEF>mov word ptr [ebp+FFFFAE66]， ax`  

`043ED6CA |. 7C 09 jl short 043ED6D5`  

`043ED6CC |. 66:C785 66AEF>mov word ptr [ebp+FFFFAE66]， 5A`  

`043ED6D5 |> 66:399D 66AEF>cmp word ptr [ebp+FFFFAE66]， bx`  

`043ED6DC |. 895D 0C mov dword ptr [ebp+C]， ebx`  

`043ED6DF |. 0F8E CA020000 jle 043ED9AF`  

`043ED6E5 |. 8DBD F6AEFFFF lea edi， dword ptr [ebp+FFFFAEF6]`  

`043ED6EB |> 8B06 /mov eax， dword ptr [esi]`  

`043ED6ED |. 8D8F 72FFFFFF |lea ecx， dword ptr [edi-8E]`  

`043ED6F3 |. 51 |push ecx`  

`043ED6F4 |. 8BCE |mov ecx， esi`  

`043ED6F6 |. FF50 18 |call dword ptr [eax+18]`  

`043ED6F9 |. 8B06 |mov eax， dword ptr [esi]`  

`043ED6FB |. 8D8F 76FFFFFF |lea ecx， dword ptr [edi-8A]`  

`043ED701 |. 51 |push ecx`  

`043ED702 |. 8BCE |mov ecx， esi`  

`043ED704 |. FF50 1C |call dword ptr [eax+1C]`  

`043ED707 |. 8B06 |mov eax， dword ptr [esi]`  

`043ED709 |. 8D8F 78FFFFFF |lea ecx， dword ptr [edi-88]`  

`043ED70F |. 51 |push ecx`  

`043ED710 |. 8BCE |mov ecx， esi`  

`043ED712 |. FF50 1C |call dword ptr [eax+1C]`  

`043ED715 |. 8B06 |mov eax， dword ptr [esi]`  

`043ED717 |. 8D8F 7AFFFFFF |lea ecx， dword ptr [edi-86]`  

`043ED71D |. 51 |push ecx`  

`043ED71E |. 8BCE |mov ecx， esi`  

`043ED720 |. FF50 14 |call dword ptr [eax+14]`  

`043ED723 |. 8B16 |mov edx， dword ptr [esi]`  

`043ED725 |. 8D87 7BFFFFFF |lea eax， dword ptr [edi-85]`  

`043ED72B |. 50 |push eax`  

`043ED72C |. 8BCE |mov ecx， esi`  

`043ED72E |. FF52 14 |call dword ptr [edx+14]`  

`043ED731 |. 8B06 |mov eax， dword ptr [esi]`  

`043ED733 |. 8D9F 7CFFFFFF |lea ebx， dword ptr [edi-84]`  

`043ED739 |. 53 |push ebx`  

`043ED73A |. 8BCE |mov ecx， esi`  

`043ED73C |. FF50 1C |call dword ptr [eax+1C]`  

`043ED73F |. 66:8B03 |mov ax， word ptr [ebx]`  

`043ED742 |. 66:3D 1000 |cmp ax， 10`  

`043ED746 |. 73 05 |jnb short 043ED74D`  

`043ED748 |. 0FBFC0 |movsx eax， ax`  

`043ED74B |. EB 03 |jmp short 043ED750`
