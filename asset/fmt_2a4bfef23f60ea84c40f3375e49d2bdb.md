
```
`  .    .    .    0046A4F1  |. /0F8A AA020000      jpe     0046A7A1                    ;  结束    
  0046A4F7  |. |807F 2D 00        cmp     byte ptr [edi+2D], 0          ;  edi+2d = 009853b5
  0046A4FB  |. |0F84 A0020000     je      0046A7A1                    ;  结束    
  0046A501  |. |B9 7050A900       mov     ecx, 00A95070    
  0046A506  |. |E8 55BA0A00       call    00515F60                    ;  随机数    
  0046A50B  |. |D95424 28        fst     dword ptr [esp+28]          ;  返回到这里    
  0046A50F  |. |D81D 14E36500     fcomp   dword ptr [65E314]    
  0046A515  |. |DFE0             fstsw   ax    
  0046A517  |. |F6C4 05         test    ah, 5    
  0046A51A  |. |0F8A 81020000    jpe     0046A7A1    
  0046A520  |. |6A 00          push    0    
  0046A522  |. |B9 6853AB00       mov     ecx, 00AB5368             ;  ASCII "D k"    
  0046A527  |. |E8 B4980E00       call    00553DE0    
  0046A52C  |. |8BF0             mov     esi, eax    
  0046A52E  |. |69F6 485F0000    imul    esi, esi, 5F48    
  0046A534  |. |81C6 001C7100    add     esi, 00711C00    
  0046A53A  |. |83BE B41F0000 FF  cmp     dword ptr [esi+1FB4], -1    
  0046A541  |. |0F84 5A020000    je      0046A7A1    
  0046A547  |. |DD05 70A9AB00    fld     qword ptr [ABA970]    
  0046A54D  |. |8D4E 04         lea     ecx, dword ptr [esi+4]    
  0046A550  |. |D82D 2454AB00    fsubr   dword ptr [AB5424]    
  0046A556  |. |D80D 10E36500    fmul    dword ptr [65E310]    
  0046A55C  |. |D95C24 24        fstp    dword ptr [esp+24]    
  0046A560  |. |D905 6C9E7000    fld     dword ptr [709E6C]    
  0046A566  |. |D95C24 18        fstp    dword ptr [esp+18]    
  0046A56A  |. |E8 61F81000       call    00579DD0    
  0046A56F  |. |D87424 18        fdiv    dword ptr [esp+18]    
  0046A573  |. |DB05 1854AB00    fild    dword ptr [AB5418]    
  0046A579  |. |DB86 F83C0000    fild    dword ptr [esi+3CF8]    
  0046A57F  |. |DEC2             faddp   st(2), st    
  0046A581  |. |D9C0             fld     st    
  0046A583  |. |DEE2             fsubrp  st(2), st    
  0046A585  |. |D9C9             fxch    st(1)    
  0046A587  |. |D80D 34026400    fmul    dword ptr [640234]    
  0046A58D  |. |D905 AC2C6500    fld     dword ptr [652CAC]    
  0046A593  |. |D85C24 28        fcomp   dword ptr [esp+28]    
  0046A597  |. |DFE0             fstsw   ax    
  0046A599  |. |F6C4 41         test    ah, 41    
  0046A59C  |. |75 07          jnz     short 0046A5A5    
  0046A59E  |. |8305 7050A900 01  add     dword ptr [A95070], 1    
  0046A5A5  |> |D9EE             fldz    
  0046A5A7  |. |D94424 24        fld     dword ptr [esp+24]    
  0046A5AB  |. |D8D1             fcom    st(1)    
  0046A5AD  |. |DFE0             fstsw   ax    
  0046A5AF  |. |F6C4 41         test    ah, 41    
  0046A5B2  |. |0F85 18010000    jnz     0046A6D0    
  0046A5B8  |. |A1 7050A900       mov     eax, dword ptr [A95070]    
  0046A5BD  |. |8BD0             mov     edx, eax    
  0046A5BF  |. |81E2 FF000000    and     edx, 0FF    
  0046A5C5  |. |D90495 7450A900  fld     dword ptr [edx*4+A95074]    
  0046A5CC  |. |83C0 01         add     eax, 1    
  0046A5CF  |. |DED9             fcompp    
  0046A5D1  |. |A3 7050A900       mov     dword ptr [A95070], eax    
  0046A5D6  |. |DFE0             fstsw   ax    
  0046A5D8  |. |F6C4 41         test    ah, 41    
  0046A5DB  |. |0F85 F1000000    jnz     0046A6D2    
  0046A5E1  |. |DDD9             fstp    st(1)    
  0046A5E3  |. |DDD8             fstp    st    
  0046A5E5  |> |8B87 241E0000    mov     eax, dword ptr [edi+1E24]    
  0046A5EB  |. |DD40 74          fld     qword ptr [eax+74]    
  0046A5EE  |. |83C0 64         add     eax, 64    
  0046A5F1  |. |DD40 08          fld     qword ptr [eax+8]    
  0046A5F4  |. |DD00             fld     qword ptr [eax]    
  0046A5F6  |. |DCC8             fmul    st, st    
  0046A5F8  |. |D9C1             fld     st(1)    
  0046A5FA  |. |DECA             fmulp   st(2), st    
  0046A5FC  |. |DEC1             faddp   st(1), st    
  0046A5FE  |. |D9C1             fld     st(1)    
  0046A600  |. |DECA             fmulp   st(2), st    
  0046A602  |. |DEC1             faddp   st(1), st    
  0046A604  |. |DC1D 08E36500    fcomp   qword ptr [65E308]    
  0046A60A  |. |DFE0             fstsw   ax    
  0046A60C  |. |F6C4 41         test    ah, 41    
  0046A60F  |. |0F85 8A010000    jnz     0046A79F    
  0046A615  |. |80BF 9D000000 00  cmp     byte ptr [edi+9D], 0    
  0046A61C  |. |0F85 7D010000    jnz     0046A79F    
  0046A622  |. |80BF 9E000000 00  cmp     byte ptr [edi+9E], 0    
  0046A629  |. |0F85 70010000    jnz     0046A79F    
  0046A62F  |. |DD05 68A9AB00    fld     qword ptr [ABA968]    
  0046A635  |. |D81D 48546400    fcomp   dword ptr [645448]    
  0046A63B  |. |DFE0             fstsw   ax    
  0046A63D  |. |F6C4 05         test    ah, 5    
  0046A640  |. |0F8A 59010000    jpe     0046A79F    
  0046A646  |. |391D B0309800    cmp     dword ptr [9830B0], ebx    
  0046A64C  |. |0F85 4D010000    jnz     0046A79F    
  0046A652  |. |833D B4309800 05 cmp     dword ptr [9830B4], 5    
  0046A659  |. |0F85 40010000    jnz     0046A79F    
  0046A65F  |. |D80D 6C9E7000    fmul    dword ptr [709E6C]    
  0046A665  |. |D81D 04E36500    fcomp   dword ptr [65E304]    
  0046A66B  |. |DFE0             fstsw   ax    
  0046A66D  |. |F6C4 41         test    ah, 41    
  0046A670  |. |0F85 2B010000    jnz     0046A7A1    
  0046A676  |. |D905 00E36500    fld     dword ptr [65E300]    
  0046A67C  |. |D81D 2854AB00    fcomp   dword ptr [AB5428]    
  0046A682  |. |DFE0             fstsw   ax    
  0046A684  |. |F6C4 05         test    ah, 5    
  0046A687  |. |0F8A 14010000    jpe     0046A7A1    
  0046A68D  |. |D905 94FD6300    fld     dword ptr [63FD94]    
  0046A693  |. |D85F 28          fcomp   dword ptr [edi+28]    
  0046A696  |. |DFE0             fstsw   ax    
  0046A698  |. |F6C4 41         test    ah, 41    
  0046A69B  |. |0F85 00010000    jnz     0046A7A1    
  0046A6A1  |. |807F 2C 00       cmp     byte ptr [edi+2C], 0    
  0046A6A5  |. |0F85 F6000000    jnz     0046A7A1    
  0046A6AB  |. |C647 2C 01       mov     byte ptr [edi+2C], 1    
  0046A6AF  |. |E8 0E241B00       call    0061CAC2    
  0046A6B4  |. |25 03000080       and     eax, 80000003    
  0046A6B9  |. |79 05          jns     short 0046A6C0    
  0046A6BB  |. |48              dec     eax    
  0046A6BC  |. |83C8 FC         or      eax, FFFFFF    
  0046A6BF  |. |40              inc     eax    
  0046A6C0  |> |83F8 03         cmp     eax, 3                  ;  Switch (cases 0..3)    
  0046A6C3  |. |0F87 D8000000    ja      0046A7A1    
  0046A6C9  |. |FF2485 BCAA74600 jmp     dword ptr [eax*4+46A7BC]    
  0046A6D0  |> |DDD8             fstp    st    
  0046A6D2  |> |D8D1             fcomp   st(1)    
  0046A6D4  |. |DFE0             fstsw   ax    
  0046A6D6  |. |F6C4 05         test    ah, 5    
  0046A6D9  |. |0F8A BE000000    jpe     0046A79D    
  0046A6DF  |. |A1 7050A900       mov     eax, dword ptr [A95070]    
  0046A6E4  |. |8BC8             mov     ecx, eax    
  0046A6E6  |. |81E1 FF000000    and     ecx, 0FF    
  0046A6EC  |. |D9048D 7450A900  fld     dword ptr [ecx*4+A95074]    
  0046A6F3  |. |83C0 01         add     eax, 1    
  0046A6F6  |. |DED9             fcompp    
  0046A6F8  |. |A3 7050A900       mov     dword ptr [A95070], eax    
  0046A6FD  |. |DFE0             fstsw   ax    
  0046A6FF  |. |F6C4 41         test    ah, 41    
  0046A702  |. |0F85 97000000    jnz     0046A79F    
  0046A708  |. .^|E9 D8FEFFFF      jmp     0046A5E5    
  0046A70D  |> |DD05 F8E26500    fld     qword ptr [65E2F8]          ;  Case 0 of switch 0046A6C0    
  0046A713  |. |DD9F C4290000    fstp    qword ptr [edi+29C4]    
  0046A719  |. |E9 83000000       jmp     0046A7A1    
  0046A71E  |> |DD87 40100000    fld     qword ptr [edi+1040]       ;  Case 1 of switch 0046A6C0    
  0046A724  |. |DC0D F0E26500    fmul    qword ptr [65E2F0]    
  0046A72A  |. |DD9F 28100000    fstp    qword ptr [edi+1028]    
  0046A730  |. |DD87 38100000    fld     qword ptr [edi+1038]    
  0046A736  |. |D9E8             fld1    
  0046A738  |. |D8D1             fcom    st(1)    
  0046A73A  |. |DFE0             fstsw   ax    
  0046A73C  |. |DDD9             fstp    st(1)    
  0046A73E  |. |F6C4 41         test    ah, 41    
  0046A741  |. |74 08          je      short 0046A74B    
  0046A743  |. |DDD8             fstp    st    
  0046A745  |. |DD87 38100000    fld     qword ptr [edi+1038]    
  0046A74B  |> |DD9F 38100000    fstp    qword ptr [edi+1038]    
  0046A751  |. |EB 4E          jmp     short 0046A7A1    
  0046A753  |> |DD05 E8E26500    fld     qword ptr [65E2E8]          ;  Case 2 of switch 0046A6C0    
  0046A759  |. |DD9F 9C1A0000    fstp    qword ptr [edi+1A9C]    
  0046A75F  |. |DD87 AC1A0000    fld     qword ptr [edi+1AAC]    
  0046A765  |. |D9E8             fld1    
  0046A767  |. |D8D1             fcom    st(1)    
  0046A769  |. |DFE0             fstsw   ax    
  0046A76B  |. |DDD9             fstp    st(1)    
  0046A76D  |. |F6C4 41         test    ah, 41    
  0046A770  |. |74 08          je      short 0046A77A    
  0046A772  |. |DDD8             fstp    st    
  0046A774  |. |DD87 AC1A0000    fld     qword ptr [edi+1AAC]    
  0046A77A  |> |DD9F AC1A0000    fstp    qword ptr [edi+1AAC]    
  0046A780  |. |EB 1F          jmp     short 0046A7A1    
  0046A782  |> |8B87 542B0000    mov     eax, dword ptr [edi+2B54]  ;  Case 3 of switch 0046A6C0    
  0046A788  |. |83C0 FF         add     eax, -1                ;  暴最高档    
  0046A78B  |. |83F8 03         cmp     eax, 3    
  0046A78E  |. |7D 05          jge     short 0046A795    
  0046A790  |. |B8 03000000       mov     eax, 3    
  0046A795  |> |8987 542B0000    mov     dword ptr [edi+2B54], eax    
  0046A79B  |. |EB 04          jmp     short 0046A7A1    
  0046A79D  |> |DDD8             fstp    st    
  0046A79F  |> |DDD8             fstp    st    
  0046A7A1  |> \8B4C24 54        mov     ecx, dword ptr [esp+54]  ;  Default case of switch 0046A6C0    
  0046A7A5  |.  81F9 00020000     cmp     ecx, 200    
  0046A7AB  |.  74 05          je      short 0046A7B2    
  0046A7AD  |.  E8 2EBF1800       call    005F66E0    
  0046A7B2  |>  5F              pop     edi    
  0046A7B3  |.  5E              pop     esi    
  0046A7B4  |.  5B              pop     ebx    
  0046A7B5  |.  8BE5              mov     esp, ebp    
  0046A7B7  |.  5D              pop     ebp    
  0046A7B8  \.  C2 0800           retn    8
```
