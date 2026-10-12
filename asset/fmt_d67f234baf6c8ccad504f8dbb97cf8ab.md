
```text
.text:00406A3A                     ; sub_406580+4ADj
.text:00406A3A 8B 85 04 FF FF FF      mov   eax, [ebp+var_FC]
.text:00406A40 85 C0               test  eax, eax
.text:00406A42 C6 45 FC 15         mov   byte ptr [ebp+var_4], 15h
.text:00406A46 74 12               jz    short loc_406A5A
.text:00406A48 FF 08               dec   dword ptr [eax]
.text:00406A4A 83 38 00            cmp   dword ptr [eax], 0
.text:00406A4D 75 0B               jnz   short loc_406A5A
.text:00406A4F 8D 8D 00 FF FF FF     lea   ecx, [ebp+var_100]
.text:00406A55 E8 60 F4 00 00        call  atr_StringBase__Delete_void_
.text:00406A5A
.text:00406A5A                     loc_406A5A:                       ; CODE XREF: sub_406580+4C6j
                                   ; sub_406580+4CDj
.text:00406A5A 8B 85 3C FF FF FF     mov   eax, [ebp+var_C4]
.text:00406A60 85 C0               test  eax, eax
.text:00406A62 C6 45 FC 14         mov   byte ptr [ebp+var_4], 14h
.text:00406A66 74 12               jz    short loc_406A7A
.text:00406A68 FF 08               dec   dword ptr [eax]
.text:00406A6A 83 38 00            cmp   dword ptr [eax], 0
.text:00406A6D 75 0B               jnz   short loc_406A7A
.text:00406A6F 8D 8D 38 FF FF FF     lea   ecx, [ebp+var_C8]
.text:00406A75 E8 40 F4 00 00        call  atr_StringBase__Delete_void_
.text:00406A7A
.text:00406A7A                     loc_406A7A:                       ; CODE XREF: sub_406580+4E6j
                                   ; sub_406580+4EDj
.text:00406A7A 8B 85 0C FF FF FF     mov   eax, [ebp+var_F4]
.text:00406A80 85 C0               test  eax, eax
.text:00406A82 C6 45 FC 13         mov   byte ptr [ebp+var_4], 13h
.text:00406A86 74 12               jz    short loc_406A9A
.text:00406A88 FF 08               dec   dword ptr [eax]
.text:00406A8A 83 38 00            cmp   dword ptr [eax], 0
.text:00406A8D 75 0B               jnz   short loc_406A9A
.text:00406A8F 8D 8D 08 FF FF FF     lea   ecx, [ebp+var_F8]
.text:00406A95 E8 20 F4 00 00        call  atr_StringBase__Delete_void_
.text:00406A9A
.text:00406A9A                     loc_406A9A:                       ; CODE XREF: sub_406580+506j
                                   ; sub_406580+50Dj
.text:00406A9A 8B 85 7C FF FF FF     mov   eax, [ebp+var_84]
.text:00406AA0 85 C0               test  eax, eax
.text:00406AA2 C6 45 FC 12         mov   byte ptr [ebp+var_4], 12h
.text:00406AA6 74 12               jz    short loc_406ABA
.text:00406AA8 FF 08               dec   dword ptr [eax]
.text:00406AAA 83 38 00            cmp   dword ptr [eax], 0
.text:00406AAD 75 0B               jnz   short loc_406ABA
.text:00406AAF 8D 8D 78 FF FF FF     lea   ecx, [ebp+var_88]
.text:00406AB5 E8 00 F4 00 00        call  atr_StringBase__Delete_void_
.text:00406ABA
.text:00406ABA                     loc_406ABA:                       ; CODE XREF: sub_406580+526j
                                   ; sub_406580+52Dj
.text:00406ABA 8B 85 74 FF FF FF     mov   eax, [ebp+var_8C]
.text:00406AC0 85 C0               test  eax, eax
.text:00406AC2 C6 45 FC 11         mov   byte ptr [ebp+var_4], 11h
.text:00406AC6 74 12               jz    short loc_406ADA              ; 51
.text:00406AC8 FF 08               dec   dword ptr [eax]
.text:00406ACA 83 38 00            cmp   dword ptr [eax], 0
.text:00406ACD 75 0B               jnz   short loc_406ADA              ; 51
.text:00406ACF 8D 8D 70 FF FF FF     lea   ecx, [ebp+var_90]
.text:00406AD5 E8 E0 F3 00 00        call  atr_StringBase__Delete_void_
.text:00406ADA
.text:00406ADA                     loc_406ADA:                       ; CODE XREF: sub_406580+546j
                                   ; sub_406580+54Dj
.text:00406ADA 6A 33               push  33h                           ; 51
.text:00406ADC 8D 8D 10 FF FF FF     lea   ecx, [ebp+var_F0]
.text:00406AE2 E8 F1 F3 00 00        call  atr_StringBase__atr_StringBase_int_
.text:00406AE7 8D 8D 10 FF FF FF     lea   ecx, [ebp+var_F0]
.text:00406AED 51                   push  ecx
.text:00406AEE 6A 08                 push  8                             ; 8
.text:00406AF0 8D 8D 20 FF FF FF     lea   ecx, [ebp+var_E0]
.text:00406AF6 C6 45 FC 1D         mov   byte ptr [ebp+var_4], 1Dh
.text:00406AFA E8 D9 F3 00 00        call  atr_StringBase__atr_StringBase_int_
.text:00406AFF 83 EC 08             sub   esp, 8
.text:00406B02 8B D4               mov   edx, esp
.text:00406B04 89 65 08            mov   [ebp+arg_0], esp
.text:00406B07 8D 85 20 FF FF FF     lea   eax, [ebp+var_E0]
.text:00406B0D 50                   push  eax
.text:00406B0E 6A 12                 push  12h                           ; 18
.text:00406B10 8D 8D 30 FF FF FF     lea   ecx, [ebp+var_D0]
.text:00406B16 C6 45 FC 1E         mov   byte ptr [ebp+var_4], 1Eh
.text:00406B1A 89 55 AC            mov   [ebp+var_54], edx
.text:00406B1D E8 B6 F3 00 00        call  atr_StringBase__atr_StringBase_int_
.text:00406B22 83 EC 08             sub   esp, 8
.text:00406B25 8B CC               mov   ecx, esp
.text:00406B27 89 65 08            mov   [ebp+arg_0], esp
.text:00406B2A 8D 95 30 FF FF FF     lea   edx, [ebp+var_D0]
.text:00406B30 52                   push  edx
.text:00406B31 89 4D A8            mov   [ebp+var_58], ecx
.text:00406B34 6A 24                 push  24h                           ; 36
.text:00406B36 8D 8D 40 FF FF FF     lea   ecx, [ebp+var_C0]
.text:00406B3C C6 45 FC 1F         mov   byte ptr [ebp+var_4], 1Fh
.text:00406B40 E8 93 F3 00 00        call  atr_StringBase__atr_StringBase_int_
.text:00406B45 83 EC 08             sub   esp, 8
.text:00406B48 8B C4               mov   eax, esp
.text:00406B4A 89 65 08            mov   [ebp+arg_0], esp
.text:00406B4D 8D 8D 40 FF FF FF     lea   ecx, [ebp+var_C0]
.text:00406B53 51                   push  ecx
.text:00406B54 68 8E 00 00 00        push  8Eh                           ; 142
.text:00406B59 8D 8D 50 FF FF FF     lea   ecx, [ebp+var_B0]
.text:00406B5F C6 45 FC 20         mov   byte ptr [ebp+var_4], 20h
.text:00406B63 89 45 A4            mov   [ebp+var_5C], eax
.text:00406B66 E8 6D F3 00 00        call  atr_StringBase__atr_StringBase_int_
.text:00406B6B 83 EC 08             sub   esp, 8
.text:00406B6E 8B D4               mov   edx, esp
.text:00406B70 89 65 08            mov   [ebp+arg_0], esp
.text:00406B73 8D 85 50 FF FF FF     lea   eax, [ebp+var_B0]
```
