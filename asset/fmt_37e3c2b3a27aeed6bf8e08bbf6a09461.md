
```
.text:004895F2 -5F8              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004895F2
.text:004895F8 -5F8              mov     [ebp+var_18], eax
.text:004895FB -5F8              mov     esi, eax
.text:004895FD -5F8              jmp     loc_48923C      ; Jump
.text:004895FD
.text:00489602   ; ---------------------------------------------------------------------------
.text:00489602
.text:00489602   loc_489602:                       ; CODE XREF: sub_488E70+3D3j
.text:00489602 -63C              mov     esi, 1
.text:00489607
.text:00489607   loc_489607:                       ; CODE XREF: sub_488E70+850j
.text:00489607 -63C              mov     eax, 12h
.text:0048960C -63C              cmp     esi, eax        ; Compare Two Operands
.text:0048960E -63C              jg      loc_4896C5      ; Jump if Greater (ZF=0 & SF=OF)
.text:0048960E
.text:00489614 -63C              lea     eax, [ebp+var_1C] ; Load Effective Address
.text:00489617 -63C              lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:0048961D -63C              mov     [ebp+var_F8], eax
.text:00489623 -63C              push    ecx
.text:00489624 -638              lea     edx, [ebp+var_100] ; Load Effective Address
.text:0048962A -638              push    esi
.text:0048962B -634              lea     eax, [ebp+var_E0] ; Load Effective Address
.text:00489631 -634              push    edx
.text:00489632 -630              push    eax
.text:00489633 -62C              mov     [ebp+var_C8], 1
.text:0048963D -62C              mov     [ebp+var_D0], 2
.text:00489647 -62C              mov     [ebp+var_100], 4008h
.text:00489651 -62C              call    ds:rtcMidCharVar ; Indirect Call Near Procedure
.text:00489651
.text:00489657 -62C              lea     edi, [esi-1]    ; Load Effective Address
.text:0048965A -62C              cmp     edi, 12h        ; Compare Two Operands
.text:0048965D -62C              jb      short loc_489665 ; Jump if Below (CF=1)
.text:0048965D
.text:0048965F -62C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048965F
.text:00489665
.text:00489665   loc_489665:                       ; CODE XREF: sub_488E70+7EDj
.text:00489665 -62C              lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:0048966B -62C              lea     edx, [ebp+var_C0] ; Load Effective Address
.text:00489671 -62C              push    ecx
.text:00489672 -628              push    edx
.text:00489673 -624              call    ds:__vbaStrVarVal ; Indirect Call Near Procedure
.text:00489673
.text:00489679 -624              push    eax
.text:0048967A -620              call    ds:rtcR8ValFromBstr ; Indirect Call Near Procedure
.text:0048967A
.text:00489680 -620              call    ds:__vbaFpI4    ; Indirect Call Near Procedure
.text:00489680
.text:00489686 -620              mov     ecx, [ebp+var_4C]
.text:00489689 -620              mov     [ecx+edi*4], eax
.text:0048968C -620              lea     ecx, [ebp+var_C0] ; Load Effective Address
.text:00489692 -620              call    ds:__vbaFreeStr ; Indirect Call Near Procedure
.text:00489692
.text:00489698 -620              lea     edx, [ebp+var_E0] ; Load Effective Address
.text:0048969E -620              lea     eax, [ebp+var_D0] ; Load Effective Address
.text:004896A4 -620              push    edx
.text:004896A5 -61C              push    eax
.text:004896A6 -618              push    2
.text:004896A8 -614              call    ds:__vbaFreeVarList ; Indirect Call Near Procedure
.text:004896A8
.text:004896AE -620              mov     eax, 1
.text:004896B3 -620              add     esp, 0Ch        ; Add
.text:004896B6 -62C              add     eax, esi        ; Add
.text:004896B8 -62C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004896B8
.text:004896BE -62C              mov     esi, eax
.text:004896C0 -62C              jmp     loc_489607      ; Jump
.text:004896C0
.text:004896C5   ; ---------------------------------------------------------------------------
.text:004896C5
.text:004896C5   loc_4896C5:                       ; CODE XREF: sub_488E70+79Ej
.text:004896C5 -63C              mov     edi, 1
.text:004896CA
.text:004896CA   loc_4896CA:                       ; CODE XREF: sub_488E70+BE5j
.text:004896CA -63C              mov     eax, 10h
.text:004896CF -63C              cmp     edi, eax        ; Compare Two Operands
.text:004896D1 -63C              jg      loc_489A5A      ; Jump if Greater (ZF=0 & SF=OF)
.text:004896D1
.text:004896D7 -63C              lea     esi, [edi-1]    ; Load Effective Address
.text:004896DA -63C              cmp     esi, 12h        ; Compare Two Operands
.text:004896DD -63C              jb      short loc_4896E5 ; Jump if Below (CF=1)
.text:004896DD
.text:004896DF -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004896DF
.text:004896E5
.text:004896E5   loc_4896E5:                       ; CODE XREF: sub_488E70+86Dj
.text:004896E5 -63C              mov     eax, edi
.text:004896E7 -63C              add     eax, 1          ; Add
.text:004896EA -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:004896EA
.text:004896F0 -63C              dec     eax             ; Decrement by 1
.text:004896F1 -63C              cmp     eax, 12h        ; Compare Two Operands
.text:004896F4 -63C              mov     [ebp+var_12C], eax
.text:004896FA -63C              jb      short loc_489702 ; Jump if Below (CF=1)
.text:004896FA
.text:004896FC -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004896FC
.text:00489702
.text:00489702   loc_489702:                       ; CODE XREF: sub_488E70+88Aj
.text:00489702 -63C              mov     eax, edi
.text:00489704 -63C              add     eax, 2          ; Add
.text:00489707 -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489707
.text:0048970D -63C              dec     eax             ; Decrement by 1
.text:0048970E -63C              cmp     eax, 12h        ; Compare Two Operands
.text:00489711 -63C              mov     [ebp+var_130], eax
.text:00489717 -63C              jb      short loc_48971F ; Jump if Below (CF=1)
.text:00489717
.text:00489719 -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489719
.text:0048971F
.text:0048971F   loc_48971F:                       ; CODE XREF: sub_488E70+8A7j
.text:0048971F -63C              cmp     esi, 12h        ; Compare Two Operands
.text:00489722 -63C              jb      short loc_48972A ; Jump if Below (CF=1)
.text:00489722
.text:00489724 -63C              call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489724
.text:0048972A
.text:0048972A   loc_48972A:                       ; CODE XREF: sub_488E70+8B2j
.text:0048972A -63C              mov     eax, edi
.text:0048972C -63C              add     eax, 1          ; Add
.text:0048972F -63C              jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:0048972F
.text:00489735 -63C              dec     eax             ; Decrement by 1
```
