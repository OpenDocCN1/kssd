
```
011903F1    .  83C4 14           add esp,14  
011903F4    .  68 94E64600       push lavss1\_.0046E694              ;  UNICODE "4817241"  
011903F9    .  68 78E64600       push lavss1\_.0046E678              ;  UNICODE "visionzero"  
011903FE    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
01190403    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
01190408    .  8B1D 80104000     mov ebx,dword ptr ds:\[<&msvbvm60.\_>;  msvbvm60.\_\_vbaStrCat  
0119040E    .  FFD3              call ebx                            ;  <&msvbvm60.\_\_vbaStrCat>  
01190410    .  8BD0             mov edx,eax  
01190412    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190415    .  FFD7             call edi  
01190417    .  50               push eax  
01190418    .  68 02000080       push 80000002  
0119041D    .  E8 6EEEFEFF       call lavss1\_.0117F290  
01190422    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190425    .  FF15 D4124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStr  
0119042B    .  8B06             mov eax,dword ptr ds:\[esi\]  
0119042D    .  56               push esi  
0119042E    .  FF90 0C030000     call dword ptr ds:\[eax+30C\]  
01190434    .  50               push eax  
01190435    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
01190438    .  51               push ecx  
01190439    .  FF15 C4104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaObjSet  
0119043F    .  8985 44FFFFFF     mov dword ptr ss:\[ebp-BC\],eax  
01190445    .  8B10             mov edx,dword ptr ds:\[eax\]  
01190447    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
0119044A    .  51               push ecx  
0119044B    .  50               push eax  
0119044C    .  FF92 A0000000     call dword ptr ds:\[edx+A0\]  
01190452    .  DBE2             fclex  
01190454    .  85C0             test eax,eax  
01190456    .  7D 18            jge short lavss1\_.01190470  
01190458    .  68 A0000000       push 0A0  
0119045D    .  68 D4564600       push lavss1\_.004656D4  
01190462    .  8B95 44FFFFFF     mov edx,dword ptr ss:\[ebp-BC\]  
01190468    .  52               push edx  
01190469    .  50               push eax  
0119046A    .  FF15 98104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj  
01190470    >  8B45 D8          mov eax,dword ptr ss:\[ebp-28\]  
01190473    .  50               push eax  
01190474    .  68 20A24600       push lavss1\_.0046A220              ;  UNICODE "visionone"  
01190479    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
0119047E    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
01190483    .  FFD3             call ebx  
01190485    .  8BD0             mov edx,eax  
01190487    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
0119048A    .  FFD7             call edi  
0119048C    .  50               push eax  
0119048D    .  68 02000080       push 80000002  
01190492    .  E8 F9EDFEFF       call lavss1\_.0117F290  
01190497    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
0119049A    .  51               push ecx  
0119049B    .  8D55 DC          lea edx,dword ptr ss:\[ebp-24\]  
0119049E    .  52               push edx  
0119049F    .  6A 02             push 2  
011904A1    .  FF15 30124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStrList  
011904A7    .  83C4 0C          add esp,0C  
011904AA    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
011904AD    .  FF15 D0124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeObj  
011904B3    .  8B06             mov eax,dword ptr ds:\[esi\]  
011904B5    .  56               push esi  
011904B6    .  FF90 FC020000     call dword ptr ds:\[eax+2FC\]  
011904BC    .  50               push eax  
011904BD    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
011904C0    .  51               push ecx  
011904C1    .  FF15 C4104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaObjSet  
011904C7    .  8985 44FFFFFF     mov dword ptr ss:\[ebp-BC\],eax  
011904CD    .  8B10             mov edx,dword ptr ds:\[eax\]  
011904CF    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
011904D2    .  51               push ecx  
011904D3    .  50               push eax  
011904D4    .  FF92 A0000000     call dword ptr ds:\[edx+A0\]  
011904DA    .  DBE2             fclex  
011904DC    .  85C0             test eax,eax  
011904DE    .  7D 18            jge short lavss1\_.011904F8  
011904E0    .  68 A0000000       push 0A0  
011904E5    .  68 D4564600       push lavss1\_.004656D4  
011904EA    .  8B95 44FFFFFF     mov edx,dword ptr ss:\[ebp-BC\]  
011904F0    .  52               push edx  
011904F1    .  50               push eax  
011904F2    .  FF15 98104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj  
011904F8    >  8B45 D8          mov eax,dword ptr ss:\[ebp-28\]  
011904FB    .  50               push eax  
011904FC    .  68 50A24600       push lavss1\_.0046A250              ;  UNICODE "visiontwo"  
01190501    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
01190506    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
0119050B    .  FFD3             call ebx  
0119050D    .  8BD0             mov edx,eax  
0119050F    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190512    .  FFD7             call edi  
01190514    .  50               push eax  
01190515    .  68 02000080       push 80000002  
0119051A    .  E8 71EDFEFF       call lavss1\_.0117F290  
0119051F    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
01190522    .  51               push ecx  
01190523    .  8D55 DC          lea edx,dword ptr ss:\[ebp-24\]  
01190526    .  52               push edx  
01190527    .  6A 02             push 2  
01190529    .  FF15 30124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStrList  
0119052F    .  83C4 0C          add esp,0C
```
