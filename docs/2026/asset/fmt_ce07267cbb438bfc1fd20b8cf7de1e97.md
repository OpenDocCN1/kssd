
```
:0048B553 FF15EC124000            Call dword ptr [004012EC]
* 
Reference To: MSVBVM60.__vbaEnd, Ord:0000h
|
:0048B559
FF1544104000            Call dword ptr [00401044]
:0048B55F
BE0A000000              mov esi, 0000000A
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B3FE(C)
|
:0048B564
8B55C0                  mov edx, dword ptr [ebp-40]
:0048B567 52
       push edx
*
Possible StringData Ref from Data Obj ->"SSNHHQXTDCOM"
       |
:0048B568 68441C4100
     push 00411C44
*
Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
 |
:0048B56D FF1534114000            Call dword ptr [00401134]
:0048B573 85C0
       test eax, eax
:0048B575 0F8591000000
       jne 0048B60C
                       ====>跳过去
:0048B57B
B804000280              mov eax, 80020004
:0048B580
898558FFFFFF            mov dword ptr [ebp+FFFFFF58], eax
:0048B586 89B550FFFFFF            mov dword ptr [ebp+FFFFFF50], esi
:0048B58C 898568FFFFFF
   mov dword ptr [ebp+FFFFFF68], eax
:0048B592 89B560FFFFFF
         mov dword ptr [ebp+FFFFFF60], esi
:0048B598
898578FFFFFF            mov dword ptr [ebp+FFFFFF78], eax
:0048B59E 89B570FFFFFF            mov dword ptr [ebp+FFFFFF70], esi
*
Possible StringData Ref from Data Obj ->"lQbR"
     |
:0048B5A4 C78548FFFFFF601C4100    mov dword ptr [ebp+FFFFFF48], 00411C60
:0048B5AE C78540FFFFFF08000000    mov dword ptr [ebp+FFFFFF40], 00000008
:0048B5B8 8D9540FFFFFF
     lea edx, dword ptr [ebp+FFFFFF40]
:0048B5BE 8D4D80
               lea ecx, dword ptr [ebp-80]
*
Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
 |
:0048B5C1 FF156C124000            Call dword ptr [0040126C]
:0048B5C7 8D8550FFFFFF
 lea eax, dword ptr [ebp+FFFFFF50]
:0048B5CD 50
             push eax
:0048B5CE 8D8D60FFFFFF
           lea ecx, dword ptr [ebp+FFFFFF60]
:0048B5D4
51                      push ecx
:0048B5D5 8D9570FFFFFF            lea edx, dword ptr [ebp+FFFFFF70]
:0048B5DB 52
         push edx
:0048B5DC 6A00
             push 00000000
:0048B5DE 8D4580
                 lea eax, dword ptr [ebp-80]
:0048B5E1 50
     push eax
*
Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
                       |
:0048B5E2
FF15CC104000            Call dword ptr [004010CC]
:0048B5E8
8D8D50FFFFFF            lea ecx, dword ptr [ebp+FFFFFF50]
:0048B5EE
51                      push ecx
:0048B5EF 8D9560FFFFFF            lea edx, dword ptr [ebp+FFFFFF60]
:0048B5F5 52
         push edx
:0048B5F6 8D8570FFFFFF
       lea eax, dword ptr [ebp+FFFFFF70]
:0048B5FC 50
                   push eax
:0048B5FD
8D4D80                  lea ecx, dword ptr [ebp-80]
:0048B600 51
       push ecx
:0048B601 6A04
           push 00000004
*
Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
 |
:0048B603 FF1540104000            Call dword ptr [00401040]
:0048B609 83C414
     add esp, 00000014
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B575(C)
|
:0048B60C
8D7338                  lea esi, dword ptr [ebx+38]
*
Possible StringData Ref from Data Obj ->""
   |
:0048B60F 68085C4000
 push 00405C08
*
Reference To: MSVBVM60.__vbaNew, Ord:0000h
                       |
:0048B614
FF1584114000            Call dword ptr [00401184]
:0048B61A
50                      push eax
:0048B61B 8D559C
 lea edx, dword ptr [ebp-64]
:0048B61E 52
             push edx
*
Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
 |
:0048B61F FF15D0104000            Call dword ptr [004010D0]
:0048B625 50
         push eax
:0048B626 56
               push esi
*
Reference To: MSVBVM60.__vbaObjSetAddref, Ord:0000h
 |
:0048B627 FF15DC104000            Call dword ptr [004010DC]
:0048B62D 8D4D9C
     lea ecx, dword ptr [ebp-64]
*
Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
 |
:0048B630 FF15EC124000            Call dword ptr [004012EC]
:0048B636 A1CCF24900
   mov eax, dword ptr [0049F2CC]
:0048B63B 85C0
             test eax, eax
:0048B63D 7515
                   jne 0048B654
:0048B63F
68CCF24900              push 0049F2CC
*
Possible StringData Ref from Data Obj ->""
   |
:0048B644 68046B4000
 push 00406B04
*
Reference To: MSVBVM60.__vbaNew2, Ord:0000h
                       |
:0048B649
FF1514124000            Call dword ptr [00401214]
:0048B64F
A1CCF24900              mov eax, dword ptr [0049F2CC]
* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0048B63D(C)
|
:0048B654
8B08                    mov ecx, dword ptr [eax]
:0048B656 50
         push eax
:0048B657 FF910C030000
       call dword ptr [ecx+0000030C]
:0048B65D 50
                   push eax
:0048B65E
8D559C                  lea edx, dword ptr [ebp-64]
:0048B661 52
       push edx
*
Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
 |
:0048B662 FF15D0104000            Call dword ptr [004010D0]
:0048B668 8BF0
       mov esi, eax
:0048B66A 8B06
             mov eax, dword ptr [esi]
:0048B66C
8D4DAC                  lea ecx, dword ptr [ebp-54]
:0048B66F 51
       push ecx
:0048B670 56
             push esi
:0048B671 FF5050
               call [eax+50]
:0048B674
DBE2                    fclex
:0048B676
85C0                    test eax, eax
:0048B678 7D0F
   jge 0048B689
:0048B67A 6A50
         push 00000050
*
Possible StringData Ref from Data Obj ->"N?f??"
     |
:0048B67C 6878F54000
   push 0040F578
:0048B681 56
           push esi
:0048B682 50
                 push eax
*
Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
   |
:0048B683 FF15A4104000            Call dword ptr [004010A4]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B678(C)
|
:0048B689
8B55AC                  mov edx, dword ptr [ebp-54]
                 ====>EDX=girlboy
      呵呵，“女孩男孩”可是重要参数呀！
:0048B68C
C745AC00000000          mov [ebp-54], 00000000
:0048B693
8D4DA8                  lea ecx, dword ptr [ebp-58]
:0048B696 FFD7
       call edi
:0048B698 8B4338
         mov eax, dword ptr [ebx+38]
:0048B69B 8B10
                   mov edx, dword ptr [eax]
:0048B69D 8D4DA8
     lea ecx, dword ptr [ebp-58]
:0048B6A0 51
                 push ecx
:0048B6A1
50                      push eax
:0048B6A2 FF521C
 call [edx+1C]
:0048B6A5 DBE2
       fclex
:0048B6A7 85C0
           test eax, eax
:0048B6A9 7D12
                 jge 0048B6BD
:0048B6AB
6A1C                    push 0000001C
* Possible
StringData Ref from Data Obj ->"}zLH?g0KRKeyString"
       |
:0048B6AD 687C1C4100
     push 00411C7C
:0048B6B2 8B5338
         mov edx, dword ptr [ebx+38]
:0048B6B5 52
                     push edx
:0048B6B6 50
   push eax
*
Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
   |
:0048B6B7 FF15A4104000            Call dword ptr [004010A4]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B6A9(C)
|
:0048B6BD
8D4DA8                  lea ecx, dword ptr [ebp-58]
*
Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
 |
:0048B6C0 FF15E8124000            Call dword ptr [004012E8]
:0048B6C6 8D4D9C
     lea ecx, dword ptr [ebp-64]
*
Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
 |
:0048B6C9 FF15EC124000            Call dword ptr [004012EC]
:0048B6CF 8B03
       mov eax, dword ptr [ebx]
:0048B6D1 53
                 push ebx
:0048B6D2
FF9014030000            call dword ptr [eax+00000314]
:0048B6D8
50                      push eax
:0048B6D9 8D4D9C
 lea ecx, dword ptr [ebp-64]
:0048B6DC 51
             push ecx
*
Reference To: MSVBVM60.__vbaObjSet, Ord:0000h
 |
:0048B6DD FF15D0104000            Call dword ptr [004010D0]
:0048B6E3 8BF0
       mov esi, eax
:0048B6E5 8B16
             mov edx, dword ptr [esi]
:0048B6E7
8D45A4                  lea eax, dword ptr [ebp-5C]
:0048B6EA 50
       push eax
:0048B6EB 56
             push esi
:0048B6EC FF92A0000000
```
