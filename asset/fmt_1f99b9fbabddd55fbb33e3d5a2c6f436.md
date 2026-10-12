
# OSR DDK

> 原文：[`www.pediy.com/kssd/pediy03/forum1234.htm`](https://www.pediy.com/kssd/pediy03/forum1234.htm)

| Name | Company | Code |
| :--- | :--- | :--- |
| 1212 | 1212 | 9274-872EA9A7 |

`http://www.driverdevelop.com/dlit.php?id=6`

`osrLicense.dll`

```asm
100016F0   public ValidateSerialNum
100016F0   ValidateSerialNum proc near
100016F0
100016F0   var_10C     = byte ptr -10Ch
100016F0   var_CC      = byte ptr -0CCh
100016F0   var_8C      = byte ptr -8Ch
100016F0   arg_0       = dword ptr  4
100016F0   arg_4       = dword ptr  8
100016F0   arg_8       = dword ptr  0Ch
100016F0
100016F0             sub   esp, 10Ch
100016F6             push  esi
100016F7             mov   esi, [esp+110h+arg_0]
100016FE             push  edi
100016FF             push  esi
10001700             call  GetOSRVersion
10001705             add   esp, 4
10001708             cmp   eax, 2
1000170B             jz    short loc_10001746
1000170D             mov   edi, esi
1000170F             or    ecx, 0FFFFFFFFh
10001712             xor   eax, eax
10001714             repne scasb
10001716             not   ecx
10001718             dec   ecx
10001719             jnz   short loc_10001729
1000171B             pop   edi
1000171C             mov   eax, 0FFFFFFFEh
10001721             pop   esi
10001722             add   esp, 10Ch
10001728             retn
10001729
10001729 loc_10001729:
10001729             mov   edi, esi
1000172B             or    ecx, 0FFFFFFFFh
1000172E             xor   eax, eax
10001730             repne scasb
10001732             not   ecx
10001734             dec   ecx
10001735             cmp   ecx, 0Dh
10001738             jz    short loc_10001746
1000173A             pop   edi
1000173B             or    eax, 0FFFFFFFFh
1000173E             pop   esi
1000173F             add   esp, 10Ch
10001745             retn
10001746
10001746 loc_10001746:
10001746             push  esi
10001747             call  GetOSRProduct
1000174C             add   esp, 4
1000174F             test  eax, eax
10001751             jnz   short loc_10001761
10001753             pop   edi
10001754             mov   eax, 0FFFFFFFBh
10001759             pop   esi
1000175A             add   esp, 10Ch
10001760             retn
10001761
10001761 loc_10001761:
10001761             push  esi
10001762             call  GetOSRVersion
10001767             add   esp, 4
1000176A             test  eax, eax
1000176C             jnz   short loc_1000177C
1000176E             pop   edi
1000176F             mov   eax, 0FFFFFFFCh
10001774             pop   esi
10001775             add   esp, 10Ch
1000177B             retn
1000177C
1000177C loc_1000177C:
1000177C             push  esi
1000177D             call  GetOSRRelease
10001782             add   esp, 4
10001785             cmp   eax, 64h
10001788             jge   short loc_10001798
1000178A             pop   edi
1000178B             mov   eax, 0FFFFFFFDh
10001790             pop   esi
10001791             add   esp, 10Ch
10001797             retn
10001798
10001798 loc_10001798:
10001798             mov   edi, esi
1000179A             or    ecx, 0FFFFFFFFh
1000179D             xor   eax, eax
1000179F             repne scasb
100017A1             not   ecx
100017A3             dec   ecx
100017A4             test  ecx, ecx
100017A6             jle   short loc_100017BD
100017A8             mov   eax, ecx
100017AA             add   esi, 5
100017AD             lea   edi, [esp+114h+var_CC]
100017B1             shr   ecx, 2
100017B4             repe  movsd
100017B6             mov   ecx, eax
100017B8             and   ecx, 3
100017BB             repe  movsb
100017BD
100017BD loc_100017BD:
100017BD             mov   edi, [esp+114h+arg_4]
100017C4             or    ecx, 0FFFFFFFFh
100017C7             xor   eax, eax
100017C9             lea   edx, [esp+114h+var_8C]
100017D0             repne scasb
100017D2             not   ecx
100017D4             sub   edi, ecx
100017D6             push  ebx
100017D7             mov   eax, ecx
100017D9             mov   esi, edi
100017DB             mov   edi, edx
100017DD             lea   edx, [esp+118h+var_8C]
100017E4             shr   ecx, 2
100017E7             repe  movsd
100017E9             mov   ecx, eax
100017EB             xor   eax, eax
100017ED             and   ecx, 3
100017F0             push  6B64646Fh
100017F5             repe  movsb
100017F7             mov   edi, [esp+11Ch+arg_8]
100017FE             or    ecx, 0FFFFFFFFh
10001801             repne scasb
10001803             not   ecx
10001805             sub   edi, ecx
10001807             mov   esi, edi
10001809             mov   ebx, ecx
1000180B             mov   edi, edx
1000180D             or    ecx, 0FFFFFFFFh
10001810             repne scasb
10001812             mov   ecx, ebx
10001814             dec   edi
10001815             shr   ecx, 2
10001818             repe  movsd
1000181A             mov   ecx, ebx
1000181C             and   ecx, 3
1000181F             repe  movsb
10001821             lea   edi, [esp+11Ch+var_8C]
10001828             or    ecx, 0FFFFFFFFh
1000182B             repne scasb
1000182D             not   ecx
1000182F             dec   ecx
10001830             lea   eax, [esp+11Ch+var_8C]
10001837             push  ecx
10001838             push  eax
10001839             call  sub_10001340
1000183E             push  eax
1000183F             lea   ecx, [esp+128h+var_10C]
10001843             push  offset a08x  ; "%08X"
10001848             push  ecx
10001849             call  ds:wsprintfA
1000184F             add   esp, 18h
10001852             lea   esi, [esp+118h+var_CC]
10001856             lea   eax, [esp+118h+var_10C]
1000185A
1000185A loc_1000185A:
1000185A             mov   dl, [eax]
1000185C             mov   bl, [esi]
1000185E             mov   cl, dl
10001860             cmp   dl, bl
10001862             jnz   short loc_10001893
10001864             test  cl, cl
10001866             jz    short loc_1000187E
10001868             mov   dl, [eax+1]
1000186B             mov   bl, [esi+1]
1000186E             mov   cl, dl
10001870             cmp   dl, bl
10001872             jnz   short loc_10001893
```
