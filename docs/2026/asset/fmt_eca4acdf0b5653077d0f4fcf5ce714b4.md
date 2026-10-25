
```assembly
           call dword ptr [edx+000000A0]
:0048B6F2
DBE2                    fclex
:0048B6F4
85C0                    test
eax, eax
:0048B6F6 7D12
   jge 0048B70A
:0048B6F8 68A0000000
     push 000000A0
\*
Possible StringData Ref from Data Obj ->"N?f??"
     |
:0048B6FD 6898044100
   push 00410498
:0048B702 56
           push esi
:0048B703 50
                 push eax
\*
Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h
   |
:0048B704 FF15A4104000            Call
dword ptr [004010A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B6F6(C)
|
\*
Possible StringData Ref from Data Obj ->"JJgClockV32SnCode"
       |
:0048B70A BA381D4100
     mov edx, 00411D38
:0048B70F 8D4DAC
           lea ecx, dword ptr [ebp-54]
\*
Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h
 |
:0048B712 FF1524124000            Call
dword ptr [00401224]
:0048B718 8B4338
     mov eax, dword ptr [ebx+38]
:0048B71B 8B08
               mov ecx, dword ptr [eax]
:0048B71D
8D55A8                  lea edx,
dword ptr [ebp-58]
:0048B720 52
       push edx
:0048B721 8D55AC
         lea edx, dword ptr [ebp-54]
:0048B724 52
                     push
edx
:0048B725 50
   push eax
:0048B726 FF5120
     call [ecx+20]
:0048B729 DBE2
           fclex
:0048B72B 85C0
               test eax, eax
:0048B72D
7D12                    jge
0048B741
:0048B72F 6A20
   push 00000020
\*
Possible StringData Ref from Data Obj ->"}zLH?g0KRKeyString"
         |
:0048B731 687C1C4100
       push 00411C7C
:0048B736 8B4B38
           mov ecx, dword ptr [ebx+38]
:0048B739
51                      push
ecx
:0048B73A 50
   push eax
\*
Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h
   |
:0048B73B FF15A4104000            Call
dword ptr [004010A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B72D(C)
|
:0048B741
8B55A4                  mov edx,
dword ptr [ebp-5C]
:0048B744 52
       push edx
:0048B745 8B45A8
         mov eax, dword ptr [ebp-58]
:0048B748 50
                     push
eax
\* Reference To:
MSVBVM60.\_\_vbaStrCmp, Ord:0000h
                     |
:0048B749
FF1534114000            Call dword ptr [00401134]
:0048B74F
8BF0                    mov
esi, eax
:0048B751 F7DE
   neg esi
:0048B753 1BF6
       sbb esi, esi
:0048B755 46
             inc esi
:0048B756 F7DE
                 neg esi
:0048B758
8D4DA8                  lea ecx,
dword ptr [ebp-58]
:0048B75B 51
       push ecx
:0048B75C 8D55A4
         lea edx, dword ptr [ebp-5C]
:0048B75F 52
                     push
edx
:0048B760 8D45AC
 lea eax, dword ptr [ebp-54]
:0048B763 50
             push eax
:0048B764 6A03
                 push 00000003
\*
Reference To: MSVBVM60.\_\_vbaFreeStrList, Ord:0000h
 |
:0048B766 FF152C124000            Call
dword ptr [0040122C]
:0048B76C 83C410
     add esp, 00000010
:0048B76F 8D4D9C
           lea ecx, dword ptr [ebp-64]
\*
Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h
 |
:0048B772 FF15EC124000            Call
dword ptr [004012EC]
:0048B778 6685F6
     test si, si
:0048B77B 7440
           je 0048B7BD
   ====>跳下去！
:0048B77D
A1B8F24900              mov eax, dword ptr
[0049F2B8]
:0048B782 85C0
     test eax, eax
:0048B784 7510
           jne 0048B796
:0048B786 68B8F24900
             push 0049F2B8
\*
Possible StringData Ref from Data Obj ->""
   |
:0048B78B 680CC14000
 push 0040C10C
\*
Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h
                       |
:0048B790
FF1514124000            Call dword ptr [00401214]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B784(C)
|
:0048B796
8B35B8F24900            mov esi, dword ptr [0049F2B8]
:0048B79C
8B0E                    mov
ecx, dword ptr [esi]
:0048B79E 56
         push esi
:0048B79F FF9104070000
       call dword ptr [ecx+00000704]
:0048B7A5 DBE2
                 fclex
:0048B7A7
85C0                    test
eax, eax
:0048B7A9 7D12
   jge 0048B7BD
:0048B7AB 6804070000
     push 00000704
:0048B7B0 68E4E74000
       push 0040E7E4
:0048B7B5 56
               push esi
:0048B7B6 50
                     push
eax
\* Reference To:
MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h
                       |
:0048B7B7
FF15A4104000            Call dword ptr [004010A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0048B77B(C),
:0048B7A9(C)
|
:0048B7BD 8B13
       mov edx, dword ptr [ebx]
:0048B7BF 53
                 push ebx
:0048B7C0
FF9234030000            call dword ptr [edx+00000334]
:0048B7C6
50                      push
eax
:0048B7C7 8D459C
 lea eax, dword ptr [ebp-64]
:0048B7CA 50
             push eax
\*
Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h
 |
:0048B7CB FF15D0104000            Call
dword ptr [004010D0]
:0048B7D1 8BF0
       mov esi, eax
:0048B7D3 8B0E
             mov ecx, dword ptr [esi]
:0048B7D5
8D55AC                  lea edx,
dword ptr [ebp-54]
:0048B7D8 52
       push edx
:0048B7D9 56
             push esi
:0048B7DA FF5150
               call [ecx+50]
:0048B7DD
DBE2                    fclex
:0048B7DF
85C0                    test
eax, eax
:0048B7E1 7D0F
   jge 0048B7F2
:0048B7E3 6A50
         push 00000050
\*
Possible StringData Ref from Data Obj ->"N?f??"
     |
:0048B7E5 6878F54000
   push 0040F578
:0048B7EA 56
           push esi
:0048B7EB 50
                 push eax
\*
Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h
   |
:0048B7EC FF15A4104000            Call
dword ptr [004010A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B7E1(C)
|
:0048B7F2
8B45AC                  mov eax,
dword ptr [ebp-54]
                 ====>EAX=121
:0048B7F5
50                      push
eax
\* Reference To:
MSVBVM60.rtcR8ValFromBstr, Ord:0245h
                     |
:0048B7F6
FF15F0124000            Call dword ptr [004012F0]
:0048B7FC
DD9D00FFFFFF            fstp qword ptr [ebp+FFFFFF00]
         ====>ST=121.00000000000000000
:0048B802
8B0B                    mov
ecx, dword ptr [ebx]
:0048B804 53
         push ebx
:0048B805 FF9130030000
       call dword ptr [ecx+00000330]
:0048B80B 50
                   push eax
:0048B80C
8D5598                  lea edx,
dword ptr [ebp-68]
:0048B80F 52
       push edx
\*
Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h
 |
:0048B810 FF15D0104000            Call
dword ptr [004010D0]
:0048B816 8BF0
       mov esi, eax
:0048B818 8B06
             mov eax, dword ptr [esi]
:0048B81A
8D4DA8                  lea ecx,
dword ptr [ebp-58]
:0048B81D 51
       push ecx
:0048B81E 56
             push esi
:0048B81F FF5050
               call [eax+50]
:0048B822
DBE2                    fclex
:0048B824
85C0                    test
eax, eax
:0048B826 7D0F
   jge 0048B837
:0048B828 6A50
         push 00000050
\*
Possible StringData Ref from Data Obj ->"N?f??"
     |
:0048B82A 6878F54000
   push 0040F578
:0048B82F 56
           push esi
:0048B830 50
                 push eax
\*
Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h
   |
:0048B831 FF15A4104000            Call
dword ptr [004010A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B826(C)
|
:0048B837
8B55A8                  mov edx,
dword ptr [ebp-58]
                 ====>EDX=83
:0048B83A
52                      push
edx
\* Reference To:
MSVBVM60.rtcR8ValFromBstr, Ord:0245h
                     |
:0048B83B
FF15F0124000            Call dword ptr [004012F0]
:0048B841
DD9DF8FEFFFF            fstp qword ptr [ebp+FFFFFEF8]
         ====>ST=83.000000000000000000
:0048B847
8B03                    mov
eax, dword ptr [ebx]
:0048B849 53
         push ebx
:0048B84A FF902C030000
       call dword ptr [eax+0000032C]
:0048B850 50
                   push eax
:0048B851
```
